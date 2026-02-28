# vaultwarden-docker
Meneger passwords vaulwarden on docker with nginx
## Домен `vault.domain.com` предоствлен как пример замените на свой иначе vaultwarden не заработает
### 1.  Создаем папку и файл `docker-compose.yml`

```bash
mkdir /etc/docker/vaultwarden
```

Файл `docker-compose.yml`

```yml                                                                                   
version: '3.8'

services:
  vaultwarden:
    # Использую официальный образ Vaultwarden
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    
    # Порты для доступа
    ports:
      - "127.0.0.1:8080:80" # HTTP порт
      - "3012:3012" # WebSocket для уведомлений
    
    # Тома для данных
    volumes:
      # Основные данные Vaultwarden
      - ./vw-data:/data
      # Логи для отладки
      - ./vw-logs:/var/log/vaultwarden
    
    # Переменные окружения
    environment:
      # ВАЖНО: Смените этот ключ на случайный!
      # Используйте: openssl rand -base64 48
#      - ADMIN_TOKEN=your-admin-token-here

      # Домен вашего сервера
      - DOMAIN=https://vault.domain.com

      # Настройки безопасности
      - SIGNUPS_ALLOWED=true # Отключить регистрацию
      - INVITATIONS_ALLOWED=true # Разрешить приглашения
```
### 1.2 поднимаем docker контейнер командой `docker compose up -d`
### 2. Настраиваем nginx и получаем сертификат
#### устанавливаем nginx и certbot
```bash
apt install certbot python3-certbot-nginx nginx
systemctl enable --now nginx
```

```
nano /etc/nginx/sites-available/vault.domain.com
```
#####  Содержимое файла `vault.domain.com`
```
server {
    server_name vault.domain.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
### 3. Обязательно делаем ссылку без нее не будет работать и перезапускаем nginx
```bash
ln -s /etc/nginx/sites-available/memos.domain.com /etc/nginx/sites-enabled/
systemctl restart nginx
```
####  Получаем сетификат и еще раз перезапускаем nginx
```bash
certbot --nginx -d vault.doamin.com
systemctl restart nginx
```
### 4. Если мы все правильно сделали, то регистрируемся  на нащем сайте `vault.domain.com` , после  редактируем в `docker-compose.yml` параметр `SIGNUPS_ALLOWED` с `true` на `false` что бы запретить решистрацию если надо

