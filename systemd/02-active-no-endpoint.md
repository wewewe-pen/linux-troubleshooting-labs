# Инцидент: service active, но endpoint недоступен

## Симптом

```text
service → active
127.0.0.1:18080/health → timeout
```

## Первая причина

В unit обнаружился неправильный port:

```text
ExecStart=... --port 18081
```

вместо ожидаемого `18080`.

После изменения выполнил:

```bash
sudo systemctl daemon-reload
sudo systemctl restart sasha-prod-b.service
```

После этого `ss -ltnp` уже показывал:

```text
127.0.0.1:18080 LISTEN
```

Но `curl` всё равно зависал.

## Второй этап расследования

`tcpdump` показывал исходящий SYN, но connection не устанавливалось.

В `nftables` INPUT chain имел `policy drop`, но правила для loopback не было.

## Вторая причина

Локальный request шёл по цепочке:

```text
curl
↓
OUTPUT
↓
lo
↓
INPUT
↓
application socket
```

SYN мог выйти через OUTPUT, но INPUT policy drop блокировал его при возвращении через loopback.

## Исправление

```bash
sudo nft add rule inet firewall input iifname "lo" accept
```