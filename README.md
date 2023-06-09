# Karma8 pure PHP demo

Cервис проверки почтовых адресов и отправки писем.

Простейшая реализация, proof of concept. Нормальная реализация в https://github.com/karma8-demo/laravel.git

### Системные требования

* PHP 8.2
    * ext-apcu
    * ext-pdo_sqlite
    * ext-zend-opcache

### Установка

```bash
git clone https://github.com/karma8-demo/pure.git
cd pure
mv database/database.example.sqlite database/database.sqlite
```

### Использование

Выполнить команды для загрузки почтовых адресов в очереди

```bash
php app/commands/emails-promote.php
php app/commands/emails-check.php
php app/commands/emails-send.php
```

Запустить воркеры для проверки почтовых адресов и отправки писем

```bash
php app/workers/email-check.php
php app/workers/email-send.php
```

Логи выводятся на экран.

## Ключевые моменты

* Реализация на чистом PHP
* БД SQLite с некоторыми ограничениями:
    * Возможны блокировки БД или отдельных таблиц
* Поле emails.email, дублирующее users.email, заменено на внешний ключ user_id
* Добавлены индексы по всем полям для выборок
* Запуск всех команд и воркеров только вручную, без планировщика
* Выбранные адреса загружаются в очереди *checks* и *sends*
* Дублирование задач в очередяз исключается при помощи уникального индекса на поле jobs.payload
* Проверенным адресом присваивается случайное значение valid=true|false
* Confirmed-адреса не проверяются: им сразу присваивается checked=true и valid=true (экономия стоимости проверки)
* Время отправки письма записывается в поле users.notifiedts, для исключения повторной отправки
* Очереди реализованы в общей БД с данными

## Возможные улучшения

* Замена SQLite на внешнюю БД (MariaDB, MySQL, PostgreSQL и т.д.) и перенос очередей, например, в Redis для избежания блокировок БД при записи адресов и увеличения производительности
* Перенос поля confirmed в таблицу emails
* Реализация многопоточных или множественных воркеров
* Множественные адреса для одного пользователя
* Не спамить: слать письма только на confirmed адреса - в таком случае не нужны дополнительные проверки адресов
* Использование Postfix или другого MTA для ускорения обработки очереди отправки писем
* Использование transactional-писем c шаблонами вместо непосредственной отправки
