# Jellyfin i arr stack
Jako że mam już swój gotowy dysk z mediami i gotową strukturą katalogów podpinam go w kontenerze. Możemy to zrobić w GUI Proxmox.
Struktura katalogów
```bash
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

   > Uwaga: jeśli dysk nie pojawia się na liście wyboru w tym oknie (GUI
   > zrób to przez **>_ Shell** hosta zamiast GUI:
   > ```bash
   > pct set 102 -mp0 /dev/sda1,mp=/mnt/storage
   > pct reboot 102
   > ```

Jeżeli nie mam katalogów i mamy pusty dysk:
```bash
   mkdir -p /mnt/storage/{movies,shows,anime,transmission}
   chown -R 1000:1000 /mnt/storage
   chmod -R 750 /mnt/storage
```
Media serwer potrzebuje dostępu do karty graicznej (dedykowanej lub zintegrowanej) do transkodowania filmów.
**GPU passthrough (QSV)** — tak jak TUN wyżej. Przez **>_ Shell** hosta: `nano /etc/pve/lxc/<IVMID>.conf` (`ls -la /dev/dri` w kontenerze pokazuje faktyczne GID-y właściciela urządzeń — zweryfikuj, mogą się różnić na innym sprzęcie):
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
Dodajemy katalogi dla poszczególnych usług
```bash
   mkdir -p /opt/appdata/{gluetun,transmission,prowlarr,sonarr,radarr,jellyfin,jellyfin/custom-cont-init.d} /opt/docker-compose/media
   chown -R 1000:1000 /opt/appdata /opt/docker-compose
```
