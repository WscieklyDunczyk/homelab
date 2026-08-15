# Monitoring
Tworzymy katalogi
```bash
   mkdir -p /opt/appdata/uptime-kuma /opt/docker-compose/monitoring
   chown -R 1000:1000 /opt/appdata /opt/docker-compose
```
Przesyłam plik docker-compose.yml do kontenera LXC z naszego komputera 
```bash
    scp docker-compose.yml <użytkownik_kontenera_LXC>@<IP_KONTENERA>:/opt/docker-compose/monitoring/
```
```bash
    cd /opt/docker-compose/monitoring && docker compose up -d
```
Otwieramy w przeglądarce `http://<IP_KONTENERA>:3001` i zakładamy konto admina i konfigurujemy monitory naszych kontenerów/usług
