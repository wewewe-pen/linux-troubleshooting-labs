# Инцидент: persistent zombie process

## Симптом

У `report-manager` постоянно оставался child process со state `Z`.

## Evidence

Zombie имел PPID самого manager, при этом parent process продолжал работать.

## Причина

Parent не выполнял reap завершившегося child.

## Временное устранение

После `SIGTERM` parent произошёл reparenting/reaping, и zombie исчез.

## Постоянное исправление

Исправить обработку завершившихся child processes в application: использовать `wait()` / `waitpid()`.

## Вывод

Zombie — это не «живой зависший процесс», а уже завершившийся child, информацию о котором parent ещё не забрал.
