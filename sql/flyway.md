# Знакомство с Flyway

## Описание

Flyway - это утилита которая позволяет делать миграции в базе данных. А что такое миграции? Миграция - это файл который содержит `SQL` код, который нужно запустить в базе данных чтобы изменить ее структуру. То есть мы "мигрируем" изменения в базу данных.

## Как создавать файл и запустить миграцию

Сначала нам надо создать файл с определенным шаблонным именем, пример `V000_db_model_initiate.sql`. В нашем файле уже мы пишем наш `SQL` код.

```sql
CREATE TABLE indicator (
    id BIGINT NOT NULL,
    name VARCHAR(255) NOT NULL,
    unit VARCHAR(255) NOT NULL,
    CONSTRAINT pk_indicator PRIMARY KEY (id),
    CONSTRAINT uk_indicator_name UNIQUE (name)
);
```

Дальше мы должны запуститт миграцию нашего кода в базу данных а делается это с помощью команды `flyway migrate`. Но так же скажу что, чтобы запустить миграцию нам надо указать для `flyway` кофнигурацию чтобы он понимал куда мы мигрируем, в какую базу данных, что за пользователь там, пароль и т.д.
Давай те разберем пример файла `flyway.conf`:

```conf
flyway.url = jdbc:postgresql://localhost:5432/data
flyway.user = data
flyway.password = data
flyway.schemas = data
flyway.defaultSchema = data

flyway.locations=filesystem:./ex00
```

Как я и говорил выше мы указываем пользователя, пароль, схему и т.д, так же указываем где у нас лежат наши файлы `.sql`
