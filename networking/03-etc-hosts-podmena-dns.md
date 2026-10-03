# Инцидент: `/etc/hosts` подменил настоящий DNS

## Симптом

`ping example.com` приводил не к ожидаемому адресу.

```bash
getent hosts example.com
```

возвращал:

```text
203.0.113.77 example.com
```

а прямой DNS query:

```bash
dig +short @1.1.1.1 example.com
```

показывал другие addresses.

## Расследование

Сравнение `getent` и `dig` показало, что системный name resolution и прямой DNS query дают разные результаты.

В `/etc/hosts` нашёлся local override.

Во время исправления я сделал ещё одну ошибку: записал туда:

```text
1.1.1.1 example.com
```

думая, что таким образом задаю DNS server.

После этого `curl -v https://example.com` действительно пытался использовать `1.1.1.1` как IPv4 адрес самого `example.com`.

## Причина

`/etc/hosts` хранит соответствия `hostname ↔ IP`, а не адрес DNS server.

При порядке вроде:

```text
hosts: files dns
```

локальная запись может быть использована до обращения к DNS.

## Исправление

Удалил ложное соответствие `example.com` из `/etc/hosts`.

Проверил:

```bash
getent hosts example.com
dig +short example.com
curl -4 -sS https://example.com >/dev/null && echo OK
```

## Вывод

- `getent` показывает результат системного Name Resolution;
- `dig` проверяет DNS напрямую;
- `/etc/hosts` — локальная таблица `hostname ↔ IP`;
- неправильная запись в `/etc/hosts` может выглядеть как DNS-проблема, хотя DNS вообще ни при чём.
