## VM сервер ubuntu 24.04

1. **Автоматизированная настройка репозитория:**\
```sudo apt install -y postgresql-common```\
```sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh```
2. **Устанавливаем нужные модули PostgreSQL 17 и contrib:**\
```sudo apt install postgresql-17```

3. **Запуск сервиса PostgreSQL:**\
```sudo systemctl start postgresql```
4. **Включение сервиса PostgreSQL:**\
```sudo systemctl enable postgresql```