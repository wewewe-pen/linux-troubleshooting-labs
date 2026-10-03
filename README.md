# Linux: разбор инцидентов и troubleshooting

Учебный репозиторий с практическими Linux-инцидентами, которые я разбирал во время подготовки к DevOps/системному администрированию.

Это не выдуманные «production stories»: сценарии собраны из моих лабораторных работ. Я оформил их как короткие incident reports — симптом, проверка гипотез, причина, исправление и итог.

Главный принцип, который я стараюсь использовать во всех работах: **сначала собрать evidence и определить слой проблемы, а уже потом что-то менять**.

## Что здесь есть

| Раздел | Что разбирал | Основные инструменты |
|---|---|---|
| [Networking](networking/) | маршрутизация, ARP, DNS/NSS, loopback, firewall, binding | `ip`, `ss`, `tcpdump`, `dig`, `getent`, `nft`, `curl` |
| [Processes](processes/) | CWD процесса, zombie, stale PID | `ps`, `/proc`, `readlink`, signals |
| [systemd](systemd/) | dependencies, restart policy, StartLimit, drop-in overrides | `systemctl`, unit files, journal |
| [Permissions](permissions/) | роли/group, traversal, ACL, SGID, sticky bit | `chmod`, ACL, ownership, `sudo -u` |

Полная карта пройденных сценариев: [docs/Карта-инцидентов.md](docs/Карта-инцидентов.md).

## Несколько самых показательных кейсов

### Неверный `/32` route к `1.1.1.1`

`ping 1.1.1.1` не работал, хотя default route выглядел нормально. Оказалось, что более специфичный host route отправлял трафик через недоступный gateway. Это пришлось доказать через routing table, neighbour table и ARP capture.

→ [Читать разбор](networking/01-nevernyy-host-route.md)

### Firewall сломал локальный DNS

`dig` таймаутился на `127.0.0.53`, а capture на `ens33` был пустым. Ошибка оказалась не в DNS-сервере, а в том, что я сначала смотрел не тот interface, а firewall блокировал loopback traffic.

→ [Читать разбор](networking/02-firewall-lokalnyi-dns.md)

### Service работал локально, но был недоступен по LAN

Процесс существовал и локальный `curl` работал, но запрос к LAN-адресу получал `Connection refused`. Через `ss` и аргументы процесса выяснилось, что service слушал только `127.0.0.1`.

→ [Читать разбор](networking/04-service-loopback.md)

### Service active, но endpoint всё равно недоступен

Сначала в unit оказался неверный port. После исправления появился listener на нужном порту, но `curl` всё равно зависал. Второй причиной был firewall без разрешения loopback traffic.

→ [Читать разбор](systemd/02-active-no-endpoint.md)

### Crash loop и StartLimit

Application падала из-за неправильного аргумента, `Restart=on-failure` пытался её восстановить, а затем systemd остановил цикл через StartLimit. Полезный сценарий для разделения root cause, recovery policy и состояния systemd.

→ [Читать разбор](systemd/03-crash-loop-startlimit.md)

### Общая директория: SGID + Default ACL + Sticky bit

Для совместной работы двух accounts одного SGID оказалось недостаточно: группа наследовалась, но `umask 027` мог убрать group write. При этом sticky bit решал отдельную задачу — защиту от удаления чужих entries.

→ [Читать разбор](permissions/03-shared-directory-sgid-acl-sticky.md)

## Как я разбираю проблемы

Обычно порядок примерно такой:

1. Воспроизвести симптом.
2. Понять, на каком слое ломается цепочка.
3. Собрать evidence командами, а не менять настройки наугад.
4. Сформулировать гипотезу.
5. Проверить её отдельной командой или capture.
6. Сделать минимальное исправление.
7. Проверить результат тем же способом, которым воспроизводилась проблема.

Отдельно стараюсь фиксировать неверные первоначальные гипотезы: для меня это полезнее, чем оставлять только «идеальный» путь решения.

## Статус

Репозиторий будет пополняться по мере изучения Linux, Nginx, Docker и CI/CD.
