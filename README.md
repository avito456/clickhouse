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

## 🔴 2. Установка (Без использования менеджера - отдельный процесс, обслуживающий ЖР конкретно указанной информационной базы (EventLogExporter.exe/dll))

### 🔹 2.1 Скачиваем дистрибутив и копируем в каталог сервера 1С (например, в 'c:\EventLog') [(akpaevj/OneSTools.EventLog)](https://github.com/akpaevj/OneSTools.EventLog/releases)
### 🔹 2.2 Настраиваем файл конфигурации (appsettings.json):
#### ♨️ **Секция Manager:**
```yaml
"Manager": {
    "ClstFolders": [
      {
        "Folder": "C:\\\\Program Files\\1cv8\\srvinfo\\reg_1541",
        "Templates": [
          {
            "Mask": "upp_main",
            "Template": "[IBNAME]_el"
          }
        ]
      },
    ]
  }
```
где:

**ClstFolders** - массив описаний каталогов рабочих серверов (reg_*)
**Folder** - Путь к каталогу рабочего сервера (reg_*)
**Templates** - шаблоны обрабатываемых наименований информационных баз и правила наименования баз данных логов
**Mask** - регулярное выражение, применяемое к имени информационной базы
**Template** - шаблон имени базы данных хранения логов журнала. Обязательно должен содержать в себе переменную [IBNAME]



