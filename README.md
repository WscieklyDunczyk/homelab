# 🚀 Homelab
> "From Functional Chaos to Enterprise Standards"

![Proxmox](https://img.shields.io/badge/Hypervisor-Proxmox_VE-orange?style=for-the-badge&logo=proxmox&logoColor=white)
![Status](https://img.shields.io/badge/Status-v1.0_Learning_Phase-blue?style=for-the-badge)

![Blueprint](img/blueprint.png)

Dokumentacja mojego domowego laboratorium opartego na dwóch niezależnych jednostkach Mini-PC. Projekt jest poligonem doświadczalnym dla technologii wirtualizacji, konteneryzacji oraz administracji systemami Windows/Linux.

---
## 🏗️ Infrastruktura Sprzętowa
Poniżej 
| Host | Model | CPU | RAM | Rola |
| :--- | :--- | :--- | :--- | :--- |
| **PVE-Loki** | HP ProDesk 400 G5 | 6C/6T | 8GB | Najważniejsze usługi i magazyn danych |
| **PVE-Hel** | HP EliteDesk 800 G2 | 4C/4T | 32GB | Windows Lab i testowanie |

---
### 🎯 Cel i geneza
Celem projektu i to dlaczego w ogóle to zacząłem robić jest nauka i zgłębianie mojego zainteresowania. Równie ważnym powodem dla mnie jest przekonanie że w obecnych czasach dominuje model subskrypcyjny na praktycznie każde usługi w internecie czy dostęp do mediów co skutkuje tym że nie posiadamy niczego na wyłączność. Innym powodem jest zadbanie o prywatność i zabezpieczeniu swoich danych.

---

## 📊 Stan obecny: v1.0
Obecnie lab znajduje się w fazie intensywnego wdrażania usług. Priorytetem jest **dostępność (uptime)** i **funkcjonalność**. 

- ### ✅ Ukończone
	- Instalacja i konfiguracja Proxmox na obu maszynach
	- Uruchomienie wybranych usług na kontenerach LXC i Docker
	- Przygotowanie bezpiecznego, izolowanego środowiska (VLAN) dla Windows Serwer
	- Zdalny dostęp (Tailscale)
	- Lokalny DNS i blokowanie reklam (Pi-hole)
 - ### 🛠️ Zadania w toku
	- Dokumentacja obecnej topologii i konfiguracji usług
 - ### 📊 Kolejne kroki
   - Wdrożenie stroney domowej (Homepage)
   - Rozbudowa głownego serwera i migracja usług
---

## 🛠️ Stos Technologiczny i Usługi

### 🎬 Media i dane
Centralny punkt rozrywki i przechowywania danych, zarządzany głównie przez kontenery LXC i Docker.
* **Jellyfin:** Prywatny serwer strumieniowania wideo.
* **Samba Server:** Sieciowy magazyn plików (NAS) dla urządzeń domowych.
* **"Arr" Stack:** Automatyzacja biblioteki mediów (Sonarr, Radarr, Prowlarr, Transmission, Gluetun).

### 🖥️ Windows Lab
W pełni odizolowane środowisko do nauki administracji systemami Microsoft.
* **Windows Server 2022:** Active Directory, DNS, DHCP, GPO Management.
* **Windows Client:** Stacja robocza do testowania polityk domenowych.

### 🌐 Sieć i przepływ ruchu
Zarządzanie ruchem w sieci, bezpieczeństwo i usprawnienie użytkowania
* **Reverse Proxy (Nginx Proxy Manager):** Obsługuje certyfikaty SSL i kieruje ruch do odpowiednich usług.
* **Pihole:** Lokalny serwer DNS, blokuje reklamy dla urządzeń z niego korzystających 
* **VLAN:** Separacja ruchu laboratorium Windows od reszty sieci, co zwiększa bezpieczeństwo sieci domowej.
* **Tailscale** Bezpieczny dostęp do usług z poza sieci domowej
---

## 🗺️ Schemat Architektury

```mermaid
graph TD
    User((Użytkownik)) --> Pi[Pihole]
    subgraph "PVE-Hel"
        Pi --> NPM[Nginx]
        NPM --> HP[Homepage]
        NPM --> BS[Bookstack]
        subgraph arr[Arr Stack - Docker]
            gluetun[gluetun]
            sonarr[sonarr] --> gluetun
            lidarr[lidarr] --> gluetun
            prowlarr[plowarr] --> gluetun
            transmission[transmission] --> gluetun
        end
        NPM --> arr
        arr --> SMB1[SMB share]
        subgraph "VLAN 10 - Windows Lab"
            WS[Windows Server AD]
            WC[Windows Klient]
            WS --- WC
        end
    end
    subgraph "PVE-Loki"
        NPM --> JF[Jellyfin]
        JF[Jellyfin] --> SMB2[SMB share]
    end
    arr --> SMB2[SMB share]
    VPN((VPN)) --> gluetun
    gluetun --> VPN
```
# 🚀 v2.0
  ### Plany na dalszy rozwój projektu
  - Monitorowanie ruchu w sieci
  - Powiadomienia o incydentach, które wymagają podjęcia działania
  - Klucze SSH zamiast haseł
  - Migracja usług z kontenerów z użytkownikiem root na dedykowanych użytkowników
  - Kopie zapasowe

# 🎞️ Zrzuty ekranu
![Homepage](img/homepage.png)
![PVE-Hel](img/hel_homepage.png)
![Pi-hole](img/pihole_homepage.png)
![Nginx Proxy Manager](img/nginx_homepage.png)
![Jellyfin](img/jellyfin_titlepage.png)
![Sonarr](img/sonarr.png)
