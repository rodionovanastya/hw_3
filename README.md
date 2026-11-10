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