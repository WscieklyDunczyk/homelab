# Pi-hole
Program typu otwarto źródłowego (Open-source) blokuje reklamy i telemetrię na poziomie zapytania (DNS sinkhole) na każdym urządzeniu, które będzie korzystać z niego jako swojego serwera DNS komputery, telefony, a nawet telewizory. 

# 🚀 Konfiguracja

## Przygotowanie kontenera LXC (wspólne dla każdego kontenera)
Dla KAŻDEJ roli (pihole, media, monitoring) tworzę osobny, kontener LXC i instaluje Dockera.
- Kontener **CT**
- Hostname `<rola>`
- `unprivileged container` - ✓
- Template `debian-12-standard`
- IPv4/CIDR: `192.168.1.<x>/24`,
- Gateway: `192.168.1.1`
> Po utworzeniu w właściwościach zaznaczamy jeszcze `Nesting` i `Keyctl` (wymagane do działania Docker)
---
## Instalacja Dockera (w kontenerze)
Prosta komenda
```bash
    curl -fsSL https://get.docker.com | sh
```
> skrypt dodaje repozytorium, instaluje potrzebne pakiety, włącza usługę i ustawia automatyczne uruchamianie po restarcie systemu
## Katalogi appdata
```bash
    mkdir -p /opt/appdata/<usługa> /opt/docker-compose/<rola>
    chown -R 1000:1000 /opt/appdata /opt/docker-compose
    chmod -R 750 /opt/appdata /opt/docker-compose
```
---
## Tailscale - TUN passthrough
Tailscale tworzy wirtualny interfejs sieciowy, przez który przechodzi cały ruch WireGuard. Do stworzenia takiego interfejsu w Linuksie jest potrzebny kernelowy sterownik TUN. Kontener LXC to nie wirtualna maszyna nie ma jądra systemu, dzieli je z hostem Proxmox. Dostęp do urządzeń sprzętowych/kernelowych musi zostać nadany, nie jest automatycznie przydzielany wraz z utworzeniem kontenera LXC. Musimy zrobić to ręcznie w konsoli hosta(proxmox).

```bash
    nano /etc/pve/lxc/<VMID>.conf
```
Dopisać na końcu pliku:
```
    lxc.cgroup2.devices.allow: c 10:200 rwm
    lxc.mount.entry: /dev/net dev/net none bind,create=dir
```
Zapisz i zrestartuj kontener
---

## Instalacja Tailscale
```bash
    curl -fsSL https://tailscale.com/install.sh | sh
    tailscale up --auth-key=<auth key z login.tailscale.com/admin/settings/keys> --hostname=<rola>
```
> [!IMPORTANT]
> Wyłącz wygasanie kluczy w panelu Tailscale **Key Expiry**`

---
W Pi-hole dodajemy jedną opcje przy uruchamianiu Tailscale
```bash
    tailscale up --accept-dns=false
```
> Flaga `--accept-dns=false` jest kluczowa. Zapobiega pętli DNS, w której Pi-hole mógłby próbować pytać samego siebie o zewnętrzne adresy.
Dodaje jeszcze jeden katalog dla konfiguracji Pi-hole

Następnie w panelu Tailscale
- Zakładka **DNS**
- **Add nameserver** -> **Custom**
- Wpisujemy adres IP (Tailscale IP) kontenera Pi-hole
- Włącz opcje **Override DNS servers**

```bash
    mkdir -p /opt/appdata/pihole/etc-dnsmasq.d
```
Przesyłam plik docker-compose.yml do kontenera LXC z naszego komputera 
```bash
    scp docker-compose.yml <użytkownik_kontenera_LXC>@192.168.1.21:/opt/docker-compose/pihole/
```
Uruchamiamy kontener Pi-hole
```bash
    cd /opt/docker-compose/pihole && docker compose up -d
```
Panel konfiguracji Pi-hole powinien być już dostępny w przeglądarce pod adresem
`http://<IP_KONTENERA>/admin`

> [!IMPORTANT]
> Jeżeli połączenie przez adres Tailscale nie działa należy ustawić `Listen on all interfaces, permit all` w zakładce `settings` -> `DNS`
---

