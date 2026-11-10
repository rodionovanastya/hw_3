# Команды для копипаста
```sh
curl http://127.0.0.1:8080/healthz
curl http://127.0.0.1:8080/readyz
curl http://127.0.0.1:8080/
curl -X POST http://127.0.0.1:8080/increment
```

# 4 балла
Запуск:
```sh
docker run -d --name my-counter-container -p 127.0.0.1:8080:8000 -e GREETING="Мое новое приветствие" my-counter
```
Логи:
```sh
docker logs --tail 10 my-counter-container

2026-11-10 15:54:32,186 listening port=8000 storage=sqlite
2026-11-10 15:55:04,849 http 172.17.0.1 "GET /healthz HTTP/1.1" 200 -
2026-11-10 15:56:59,713 http 172.17.0.1 "GET /readyz HTTP/1.1" 200 -
2026-11-10 15:57:03,722 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-10 15:57:09,070 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-10 15:57:10,704 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-10 15:57:12,383 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-10 15:57:16,076 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-10 15:57:19,337 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-10 15:57:22,134 http 172.17.0.1 "GET / HTTP/1.1" 200 -
```
# 6 баллов

compose.yaml:
```sh
services:
  app:
    build: .
    container_name: my-counter-container
    ports:
      - "${APP_PORT:-8000}:8000"
    environment:
      STORAGE: sqlite
      DATA_FILE: /data/counter.db
      GREETING: "${GREETING:-Default greeting}"
    volumes:
      - counter-data:/data

volumes:
  counter-data:
```

После выполнения следующих команд:
```sh
docker compose up --build -d
curl -X POST http://127.0.0.1:8080/increment (3 раза)
```
Имеем:
```sh
{"greeting": "My env greeting", "count": 3, "hostname": "54620c645a25", "storage": "sqlite"}
```
Далее:
```sh
docker compose down
docker compose up -d
curl http://127.0.0.1:8080/
```
Имеем:
```sh
{"greeting": "My env greeting", "count": 3, "hostname": "25b9eb0af385", "storage": "sqlite"}
```

Логи на всякий случай:
```sh
my-counter-container  | 2026-11-10 16:15:45,974 listening port=8000 storage=sqlite
my-counter-container  | 2026-11-10 16:16:00,146 http 172.18.0.1 "POST /increment HTTP/1.1" 200 -
my-counter-container  | 2026-11-10 16:16:01,120 http 172.18.0.1 "POST /increment HTTP/1.1" 200 -
my-counter-container  | 2026-11-10 16:16:02,435 http 172.18.0.1 "POST /increment HTTP/1.1" 200 -

my-counter-container  | 2026-11-10 16:16:50,366 listening port=8000 storage=sqlite
my-counter-container  | 2026-11-10 16:17:08,896 http 172.18.0.1 "GET / HTTP/1.1" 200 -
```

# 8 баллов
в compose.yaml добавли редис, хэлсчек апп и редис:
```sh
services:
  app:
    build: .
    container_name: my-counter-container
    ports:
      - "${APP_PORT:-8000}:8000"
    environment:
      STORAGE: redis
      REDIS_URL: redis://redis:6379/0
      DATA_FILE: /data/counter.db
      GREETING: "${GREETING:-Default greeting}"
    depends_on:
      redis:
        condition: service_healthy
    healthcheck:
      test:
        - CMD
        - python
        - -c
        - "import urllib.request; urllib.request.urlopen('http://127.0.0.1:${APP_PORT:-8000}/healthz', timeout=2)"
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
volumes:
  redis-data:
```
Тестим:
```sh
$ docker compose up -d --build
...
$ curl http://127.0.0.1:8080/readyz
{"status": "ready", "storage": "redis"}
$ curl -X POST http://127.0.0.1:8080/increment (3 раза)
{"greeting": "My env greeting", "count": 3, "hostname": "ba26324b1512", "storage": "redis"}
$ docker compose down
...
$ docker compose up -d
...
$ curl http://127.0.0.1:8080/
{"greeting": "My env greeting", "count": 3, "hostname": "f531c59d83f6", "storage": "redis"}  # то есть N сохранилось
$ docker compose stop redis # отключаем редис
...
$ curl http://127.0.0.1:8080/healthz 
{"status": "alive"} # приложение работает
$ curl http://127.0.0.1:8080/readyz
{"error": "storage_unavailable"} # бд не работает
$ curl -X POST http://127.0.0.1:8080/increment
{"error": "storage_unavailable"} # инкремент не работает
$ docker compose start redis
...
$ curl http://127.0.0.1:8080/
{"greeting": "My env greeting", "count": 3, "hostname": "f531c59d83f6", "storage": "redis"} # запускаем обратно редис, всё сохранилось
$ curl -X POST http://127.0.0.1:8080/increment
{"greeting": "My env greeting", "count": 4, "hostname": "f531c59d83f6", "storage": "redis"}
```
Логи, когда redis был отключен:
```sh
my-counter-container  | 2026-11-10 16:39:58,471 storage_unavailable: ConnectionError
my-counter-container  | 2026-11-10 16:39:58,471 http 172.18.0.1 "GET /readyz HTTP/1.1" 503 -
my-counter-container  | 2026-11-10 16:40:06,715 http 127.0.0.1 "GET /healthz HTTP/1.1" 200 -
my-counter-container  | 2026-11-10 16:40:11,311 storage_unavailable: ConnectionError
my-counter-container  | 2026-11-10 16:40:11,311 http 172.18.0.1 "POST /increment HTTP/1.1" 503 -
```