# Выгрузка журнала регистрации 1С в Yandex Clickhouse

[см. Сверхбыстрый Журнал Регистрации 1C с помощью Yandex Clickhouse](https://youtu.be/HnZ0Of-YpW0)

** 1. Установка andex Clickhouse + WEB интерфейс в Docker **

[Репозиторий Clickhouse Yandex:](https://hub.docker.com/r/yandex/clickhouse-server)

[Репозиторий с исходниками:](https://github.com/EvilBeaver/CllickHousePlayground)

```
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





https://github.com/akpaevj/OneSTools.EventLog