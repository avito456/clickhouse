# Выгрузка журнала регистрации 1С в Yandex Clickhouse

Полная документация по ссылкам ниже:

* 📍 Андрей Овсянкин. [Вебинар Сверхбыстрый Журнал Регистрации 1C с помощью Yandex Clickhouse](https://youtu.be/HnZ0Of-YpW0) 
* 📍 Андрей Овсянкин. [Репозиторий с исходниками EvilBeaver/CllickHousePlayground](https://github.com/EvilBeaver/CllickHousePlayground)
* 📍 Евгений Акпаев.  [Репозиторий Exporter (Евakpaevj/OneSTools.EventLog)](https://github.com/akpaevj/OneSTools.EventLog)
* 📍 [Репозиторий Clickhouse Yandex](https://hub.docker.com/r/yandex/clickhouse-server)
* 📍 [ODBC драйвер ClickHouse](https://github.com/ClickHouse/clickhouse-odbc)
* 📍 [runtime .net 5](https://dotnet.microsoft.com/en-us/download/dotnet/5.0)

**Инструкция по установке:**

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

| Параметр | Описание |
|:---------|:---------|
| **ClstFolders** | Массив описаний каталогов рабочих серверов (reg_*) |
| **Folder** | Путь к каталогу рабочего сервера (reg_*) |
| **Templates**  | Шаблоны обрабатываемых наименований информационных баз и правила наименования баз данных логов |
| **Mask** | Регулярное выражение, применяемое к имени информационной базы |
| **Template** | Шаблон имени базы данных хранения логов журнала. Обязательно должен содержать в себе переменную [IBNAME] |

#### ♨️ **Секция Exporter:**
```yaml
"Exporter": {
    "StorageType": 1,
    "LogFolder": "C:\\Users\\akpaev.e.ENTERPRISE\\Desktop\\1Cv8Log",
    "Portion": 10000,
    "TimeZone": "Europe/Moscow",
    "WritingMaxDegreeOfParallelism": 1,
    "CollectedFactor": 2,
    "ReadingTimeout": 1,
    "LoadArchive": false
  }
```

где:

| Параметр | Описание |
|:---------|:--------|
| StorageType | тип хранилища журнала регистрации. Может принимать значения: 1 - Clickhouse 2 - ElasticSearch |
| LogFolder | путь к каталогу журнала регистрации 1С |
| Portion | Размер порции, записываемый в БД за одну итерацию (10000 по умолчанию) |
| TimeZone | часовой пояс (в формате IANA Time Zone Database), в котором записан журнал регистрации. По умолчанию - часовой пояс системы |
| WritingMaxDegreeOfParallelism | количество потоков записи в СУБД. Т.к. в ClickHouse не поддерживаются одновременные BULK операции, то параметр имеет смысл только для ElasticSearch. По умолчанию - 1 |
| CollectedFactor | коэффициент количества элементов, которые могут быть помещены в очередь записи. Предельное количество элементов равно Portion * CollectedFactor. По умолчанию - 2 |
| ReadingTimeout | таймаут сброса данных при достижении конца файла (в секундах). По умолчанию - 1 сек. |
| LoadArchive | Специальный параметр, предназначенный для первоначальной загрузки архивных данных. При установке параметра в true, отключается "live" режим и не выполняется запрос последнего обработанного файла из БД |

#### Использование:
Все приложения могут быть запущены в 2 режимах: как обычное приложение, либо как служба Windows/Linux. Для теста в Вашей среде, достаточно просто выполнить конфигурацию приложения в файле *appsettings.json*, установить **runtime .net 5** (при его отсутствии) и запустить exe/dll. Базы данных в СУБД вручную создавать не нужно, они будут созданы автоматически.

Для запуска приложения как службы необходимо (название службы и путь к исполняемому файлу подставить свои):</br>

**Windows:**</br>
Поместить файлы приложения в каталог и выполнить в консоли команду:
```
sc create EventLogExporter binPath= "C:\elexporter\EventLogExporter.exe"
```
и запустить службу командой:
```
sc start EventLogExporter
```



#### ♨️ **Секция ClickHouse:**
```yaml
"ConnectionStrings": {
    "Default": "Host=localhost;Port=8123;Username=default;password=;Database=database_name;"
  }
```  

По умолчанию: Username = 'default' и Password = ''

Подключиться к web  сервису можно по адресу:
http://localhost:8123



## 🔴 3. Подключение к 1С через ODBC


1. [Скачать и установить ODBC драйвер ClickHouse](https://github.com/ClickHouse/clickhouse-odbc)
2. Подключить как внешний источник к 1С.
3. Использовать анализ журнала регистрации в СКД.