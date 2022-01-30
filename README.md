# Выгрузка журнала регистрации 1С в Yandex Clickhouse

[📍 см. Сверхбыстрый Журнал Регистрации 1C с помощью Yandex Clickhouse](https://youtu.be/HnZ0Of-YpW0)
[📍 akpaevj/OneSTools.EventLog](https://github.com/akpaevj/OneSTools.EventLog)
[📍 Репозиторий с исходниками EvilBeaver/CllickHousePlayground](https://github.com/EvilBeaver/CllickHousePlayground)
[📍 Репозиторий Clickhouse Yandex](https://hub.docker.com/r/yandex/clickhouse-server)

## 🔴 1. Установка andex Clickhouse + WEB интерфейс в Docker

```yaml
version: "3.3"
services:
  clickhouse:
    image: yandex/clickhouse-server
    ulimits:
        nofile:
          soft: 262144
          hard: 262144 
    ports:
      - "8123:8123"
    volumes:
      - "clickhouse-data:/var/lib/clickhouse"
  click-ui:
    image: spoonest/clickhouse-tabix-web-client
    environment: 
      CH_NAME: 1C_Logs
      CH_HOST: localhost:8123
    depends_on:
        - clickhouse
    ports:
      - "8080:80"
volumes:
  clickhouse-data:      
```

## 🔴 2. Установка 

🔘 2.1 Скачиваем дистрибутив [(akpaevj/OneSTools.EventLog)](https://github.com/akpaevj/OneSTools.EventLog/releases)




