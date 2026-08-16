# Jellyfin i arr stack

Jako że mam już swój gotowy dysk z mediami i gotową strukturą katalogów podpinam go w kontenerze. Możemy to zrobić w GUI Proxmox.

Struktura katalogów

```
    /
    - movies
    - anime
    - shows
    - transmission
```

Każdy katalog zawiera odpowiadające mu media, a folder transmission jest katalogiem dla klienta torrent

- Zaznacz kontener → **Resources** → **Add** →
- **Mount Point** → `Disk/directory` → `Path: /mnt/storage` (gdzie zostanie podpięty dysk w kontenerze)
→ **Add**. Kontener się zrestartuje automatycznie po dodaniu.

> [!IMPORTANT]
> Uwaga: jeśli dysk nie pojawia się na liście w tym oknie
> zrób to przez **>_ Shell** hosta zamiast GUI:
> ```bash
> pct set <VMID> -mp0 /dev/sda1,mp=/mnt/storage
> pct reboot <VMID>
> ```

Jeżeli nie ma katalogów i mamy pusty dysk:

```bash
   mkdir -p /mnt/storage/{movies,shows,anime,transmission}
   chown -R 1000:1000 /mnt/storage
   chmod -R 750 /mnt/storage
```

## GPU Passthrough

Media serwer potrzebuje dostępu do karty graficznej (dedykowanej lub zintegrowanej) do transkodowania filmów.

**GPU passthrough** — tak jak TUN wyżej. Przez **>_ Shell** hosta: `nano /etc/pve/lxc/<VMID>.conf` (`ls -la /dev/dri` w kontenerze pokazuje faktyczne GID-y właściciela urządzeń — zweryfikuj, mogą się różnić na innym sprzęcie):

```
   lxc.cgroup2.devices.allow: c 226:0 rwm
   lxc.cgroup2.devices.allow: c 226:128 rwm
   lxc.mount.entry: /dev/dri/card0 dev/dri/card0 none bind,optional,create=file
   lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
   lxc.idmap: u 0 100000 65536
   lxc.idmap: g 0 100000 44
   lxc.idmap: g 44 44 1
   lxc.idmap: g 45 100045 948
   lxc.idmap: g 993 993 1
   lxc.idmap: g 994 100994 64542
```

Zapisz i reset kontenera

---

Dodajemy katalogi dla poszczególnych usług

```bash
   mkdir -p /opt/appdata/{gluetun,transmission,prowlarr,sonarr,radarr,jellyfin,jellyfin/custom-cont-init.d} /opt/docker-compose/media
   chown -R 1000:1000 /opt/appdata /opt/docker-compose
```

## Przesłanie pliku docker na kontener
Polecenie wykonane w konsoli z komputera na którym mamy plik

```bash
    scp docker-compose.yml <użytkownik_kontenera_LXC>@<IP_KONTENERA>:/opt/docker-compose/media/
```

> W pliku docker-compose.yml są wykorzystane zmienne aby ukryć wrażliwe dane dostawcy VPN
> Zmienne zapisujemy w pliku .env z odpowiadającymi im wartościami

```bash
    scp .env.example <użytkownik_kontenera_LXC>@<IP_KONTENERA>:/opt/docker-compose/media/
```

Później w konsoli kontenera edytujemy zawartość pliku **.env**. Wpisujemy dane naszego VPN

```bash
    cd /opt/docker-compose/media
    cp .env.example .env
    nano .env # Uzupełnij dane WireGuard
```

### Uruchomienie usług

```bash
    cd /opt/docker-compose/media
    docker compose up -d
```

Po uruchomieniu usługi powinny być dostępne w przeglądarce pod adresem ip kontenera wraz z odpowiadającym portem dla danej usługi.

- Prowlarr (`:9696`) → dodaj indeksery, połącz z Sonarr/Radarr.
- Sonarr (`:8989`) → root foldery `/data/shows` i `/data/anime`, download client Transmission
- Radarr (`:7878`) → root foldery `/data/movies`, download client Transmission
- Jellyfin (`:8096`) → dodaj biblioteki wskazujące na `/data/shows`, `/data/anime`, `/data/movies`, `/data/animacje`; Zakładki Kokpit -> Odtwarzanie -> Transkodowanie, wybierz w **Akceleracja sprzętowa Intel Quicksync(QSV)**, Urządzenie QSV wstaw **/dev/dri/renderD128**
