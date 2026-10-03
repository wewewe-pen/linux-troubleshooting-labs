# Инцидент: service слушал только loopback

## Симптом

Локально service работал:

```bash
curl http://127.0.0.1:8080
```

Ответ:

```text
NETWORK BOSS LAB SERVICE OK
```

Но запрос к LAN IP:

```bash
curl -v http://192.168.254.129:8080
```

возвращал `Connection refused`.

## Расследование

Проверил listening sockets:

```bash
ss -lnt
```

Увидел:

```text
127.0.0.1:8080 LISTEN
```

То есть process существовал, но listener был привязан только к loopback.

Затем нашёл сам process:

```bash
sudo ss -lntp | grep ':8080'
ps -p 5013 -o pid,ppid,cmd
```

В аргументах было:

```text
python3 -m http.server 8080 --bind 127.0.0.1 ...
```

## Причина

Application слушала только `127.0.0.1:8080`. Для LAN IP подходящего listener не существовало.

Firewall здесь не мог исправить binding: разрешающий rule не создаёт socket.

## Исправление

Перезапустил service с:

```text
--bind 0.0.0.0
```

После этого:

```text
0.0.0.0:8080 LISTEN
```

Для remote access также был разрешён TCP/8080 в INPUT.

Финальная проверка с другой машины прошла успешно.

## Вывод

- `127.0.0.1:PORT` — только loopback;
- `0.0.0.0:PORT` — все подходящие local IPv4 addresses;
- `firewall ACCEPT` и наличие listener — разные вещи;
- `Connection refused` не стоит автоматически лечить firewall rules.
