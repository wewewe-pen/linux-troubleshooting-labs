# Инцидент: процесс запущен с неправильным CWD

## Симптом

Worker постоянно писал:

```text
CONFIG ERROR: service.conf not found; cwd=/tmp
```

При этом process существовал:

```text
PID 12176
STATE S
PPID 1
```

## Расследование

Проверил current working directory:

```bash
readlink /proc/12176/cwd
```

Результат:

```text
/tmp
```

Config находился в:

```text
/home/sanchous/process_lab_1/service.conf
```

Application искала относительный path `service.conf`, поэтому фактически получалось:

```text
cwd=/tmp + service.conf → /tmp/service.conf
```

Такого файла не существовало.

## Причина

Process был запущен с `cwd=/tmp`, а application рассчитывала на relative path к config, находившемуся в другой directory.

## Исправление

В этой лаборатории config был перемещён в `/tmp`. После этого worker начал писать:

```text
CONFIG OK cwd=/tmp
```