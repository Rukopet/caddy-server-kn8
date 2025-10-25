# Caddy Gateway для KN8

API Gateway на Caddy для проксирования запросов в kn8-core.

## Архитектура

```
Client → Caddy (:8080) → kn8-core (:4000)
         Gateway          Bounded Contexts
```

## Что делает

- 🌐 Статический веб-сервер (SPA support)
- 🔄 Reverse proxy `/api/*` → kn8-core
- 📊 JSON логирование
- ✅ Health check проксирование

## Быстрый старт

### Docker

```bash
docker build -t caddy-gateway .

docker run -d \
  -p 8080:8080 \
  -e KN8_CORE_URL=http://kn8-core:4000 \
  caddy-gateway
```

### Docker Compose (с kn8-core)

Добавьте kn8-core в `docker-compose.yml`:

```yaml
services:
  caddy:
    build: .
    ports:
      - "8080:8080"
    environment:
      - KN8_CORE_URL=http://kn8-core:4000
    depends_on:
      - kn8-core

  kn8-core:
    image: your-registry/kn8-core:latest
    ports:
      - "4000:4000"
    environment:
      - PORT=4000
```

Запуск:
```bash
docker-compose up -d
```

## Environment Variables

| Переменная | Описание | Default |
|------------|----------|---------|
| `PORT` | Порт Caddy | `8080` |
| `KN8_CORE_URL` | URL kn8-core сервиса | `http://localhost:4000` |

## Роутинг

| Path | Назначение | Проксирование |
|------|-----------|---------------|
| `/api/*` | API запросы | → kn8-core |
| `/health` | Health check | → kn8-core |
| `/*` | Статика (SPA) | Локально из `/srv` |

## Production (HTTPS)

Для автоматического HTTPS обновите Caddyfile:

```caddyfile
your-domain.com {
    log {
        output stdout
        format json
    }
    
    handle /api/* {
        reverse_proxy kn8-core:4000
    }
    
    handle /health {
        reverse_proxy kn8-core:4000
    }
    
    handle {
        root * /srv
        file_server
        try_files {path} /index.html
    }
}
```

Caddy автоматически получит SSL от Let's Encrypt.
