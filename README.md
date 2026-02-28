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
      - DOMAIN=https://vault.lord-mikrotik.ru

      # Настройки безопасности
      - SIGNUPS_ALLOWED=true # Отключить регистрацию
      - INVITATIONS_ALLOWED=true # Разрешить приглашения
```
### 2. Настраиваем nginx и получение сертификата
#### устанавливаем nginx и certbot
```bash
```

```
nano /etc/nginx/sites-available/vault.domain.com
##### Файл /etc/nginx/sites-available/vault.domain.com
```
server {
    server_name vault.lord-mikrotik.ru;

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
####  Получаем сетификат
```
