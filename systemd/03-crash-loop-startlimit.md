# Инцидент: crash loop и StartLimit

## Симптом

Unit постоянно перезапускался и в итоге получил:

```text
Start request repeated too quickly
```

## Расследование

Изначально в `ExecStart` был argument:

```text
--mode broken
```

Application отвечала:

```text
FATAL: unsupported mode 'broken'
status=2/INVALIDARGUMENT
```

При этом unit имел:

```ini
Restart=on-failure
RestartSec=0.2
```

Systemd корректно пытался восстанавливать process, но application снова падала. После нескольких попыток сработала защита StartLimit.

Когда я просто убрал `--mode`, появилась другая ошибка: argument был обязательным. Значит проблема была не в наличии `--mode`, а в неверном значении.

## Причина

Application получала неправильный argument и завершалась. Restart policy только повторяла запуск, а StartLimit остановил бесконечный crash loop.

## Исправление

Исправил `ExecStart`, сохранив `Restart=on-failure`, затем:

```bash
sudo systemctl daemon-reload
sudo systemctl reset-failed sasha-prod-d.service
sudo systemctl start sasha-prod-d.service
```

## Вывод

```text
application error → root cause
Restart=...       → recovery policy
StartLimit        → защита от crash loop
reset-failed      → очистка failed/rate-limit state
```

`reset-failed` не исправляет причину и сам по себе не запускает service.
