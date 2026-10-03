# Инцидент: firewall сломал локальный DNS

## Симптом

```bash
dig example.com
```

возвращал:

```text
communications error to 127.0.0.53#53: timed out
no servers could be reached
```

При этом capture на `ens33` ничего не показывал:

```bash
sudo tcpdump -i ens33 -nn 'port 53'
```

## Расследование

Ключевая деталь была прямо в ошибке: destination — `127.0.0.53`, то есть loopback.

Значит `ens33` был неправильной точкой наблюдения. Перенёс capture на `lo`:

```bash
sudo tcpdump -i lo -nn 'port 53'
```

Теперь было видно DNS query:

```text
127.0.0.1:ephemeral → 127.0.0.53:53
A? example.com
```

Запросы повторялись, но ответов не было.

После этого проверил firewall. В INPUT chain была `policy drop` и разрешения для established traffic, ICMP и SSH, но loopback не был разрешён.

## Причина

Локальный DNS query шёл через `lo`, а INPUT firewall блокировал новый UDP flow к local resolver.

## Исправление

Добавил разрешение loopback traffic:

```text
iifname "lo" accept
```

После применения ruleset DNS снова заработал.