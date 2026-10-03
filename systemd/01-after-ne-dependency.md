# Инцидент: `After=` оказался не dependency

## Симптом

API запускался без необходимого DB service и завершался.

## Расследование

В unit было:

```ini
[Unit]
After=db.service
```

Сначала это выглядело как зависимость, но `After=` задаёт только ordering: если оба unit запускаются, DB должна быть раньше API.

Само по себе `After=` не притягивает DB к activation.

## Причина

Не было dependency между API и DB.

## Исправление

```ini
[Unit]
Requires=db.service
After=db.service
```

После изменения:

```bash
sudo systemctl daemon-reload
sudo systemctl start <service>
```

## Вывод

```text
After=/Before=   → порядок
Requires=/Wants= → dependency
```

И эти directives относятся к `[Unit]`.
