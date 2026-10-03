# Инцидент: drop-in override изменил effective configuration

## Симптом

В основном unit было:

```ini
Restart=on-failure
```

но service вёл себя так, будто restart policy отключена.

## Расследование

Нашёл drop-in:

```text
/etc/systemd/system/sasha-prod-e.service.d/override.conf
```

с содержимым:

```ini
[Service]
Restart=no
```

То есть effective configuration была другой, чем в основном vendor unit.

Полезные проверки:

```bash
systemctl cat sasha-prod-e.service
systemctl show -p Restart sasha-prod-e.service
```

Первая команда показывает, из каких файлов собирается unit, вторая — итоговое property.

## Причина

Administrator drop-in переопределял `Restart=`.

## Исправление

Исправил именно override, а не основной vendor file, затем выполнил:

```bash
sudo systemctl daemon-reload
```

Drop-in применяется к unit автоматически; отдельно `enable` для него не нужен.
