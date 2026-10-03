# Инцидент: auditor должен читать только новые файлы в `incoming/`

## Требование

Новые файлы, создаваемые dev, должны были:

- наследовать group `p2team`;
- быть readable для auditor;
- оставаться недоступными outsider;
- не быть writable для auditor.

## Первая ошибка

Я добавил обычный Access ACL для auditor на саму directory:

```text
user:p2audit:r-x
```

Это позволяло auditor пройти в `incoming/`, но не давало ему автоматический доступ к новым файлам.

## Причина

Access ACL действует на текущий объект. Для наследования прав новыми объектами нужен Default ACL.

## Рабочая схема

```text
SGID       → новые файлы наследуют p2team
Access ACL → auditor может traverse incoming/
Default ACL→ новые файлы наследуют read для auditor
```

## Вывод

`Access ACL` и `Default ACL` нельзя воспринимать как одно и то же:

```text
Access ACL  → текущий объект
Default ACL → будущие дочерние объекты
```
