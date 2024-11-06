Пока что сделал только через дамп, у меня много работы и отпуск с пятницы, остальное попробую когда вернусь.
Сделал скрипт для экспорта

```
#!/bin/bash

# Параметры подключения
SOURCE_DB="thai"                # Имя исходной базы данных
DEST_DB="thai"                  # Имя целевой базы данных
SOURCE_HOST="localhost"         # Хост исходной базы данных
DEST_HOST="localhost"           # Хост целевой базы данных
SOURCE_PORT="5432"              # Порт исходной базы данных (PostgreSQL 16)
DEST_PORT="5434"                # Порт целевой базы данных (PostgreSQL 17)
USER="postgres"                 # Пользователь базы данных
TABLE_NAME="book.tickets"       # Имя таблицы для экспорта

# Файл для временного дампа
DUMP_FILE="table_dump.sql"

# Записываем время начала
start_time=$(date +%s)

echo "Начало миграции таблицы $TABLE_NAME..."

# Экспорт таблицы с помощью pg_dump
echo "Экспортируем таблицу $TABLE_NAME из базы $SOURCE_DB..."
pg_dump -h "$SOURCE_HOST" -p "$SOURCE_PORT" -U "$USER" -d "$SOURCE_DB" -Fc -f "$DUMP_FILE"

# Проверка успешности pg_dump
if [ $? -ne 0 ]; then
    echo "Ошибка: pg_dump завершился с ошибкой."
    exit 1
fi

# Импорт таблицы с помощью pg_restore
echo "Импортируем таблицу $TABLE_NAME в базу $DEST_DB..."
pg_restore -h "$DEST_HOST" -p "$DEST_PORT" -U "$USER" -d "$DEST_DB" "$DUMP_FILE"

# Проверка успешности pg_restore
if [ $? -ne 0 ]; then
    echo "Ошибка: pg_restore завершился с ошибкой."
    end_time=$(date +%s)
    duration=$((end_time - start_time))
    echo "Миграция таблицы $TABLE_NAME завершена за $duration секунд."
    exit 1
fi

# Записываем время окончания
end_time=$(date +%s)

# Рассчитываем и выводим общее время выполнения
duration=$((end_time - start_time))
echo "Миграция таблицы $TABLE_NAME завершена за $duration секунд."

# Удаляем временный файл дампа
rm "$DUMP_FILE"
```

Результат

```
Начало миграции таблицы book.tickets...
Экспортируем таблицу book.tickets из базы thai...
Password:
Импортируем таблицу book.tickets в базу thai...
Password:
Миграция таблицы book.tickets завершена за 169 секунд.
```