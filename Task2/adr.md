### <a name="_b7urdng99y53"></a>**Название задачи: Автоматизация процессов связанных с кормлением, безопасностью и мониторингом поголовья скота.** 
### <a name="_hjk0fkfyohdk"></a>**Автор: Степан Дильман**
### <a name="_uanumrh8zrui"></a>**Дата: 28-03-2026**
### <a name="_3bfxc9a45514"></a>**Функциональные требования**
|**№**|**Действующие лица или системы**|**Use Case**|**Описание**|
| :-: | :- | :- | :- |
|1|Зоотехник/система|Детектирование беспокойства|фиксировать признаки беспокойного поведения или драк среди животных и оповещать оператора|
|2|Зоотехник/система|Детектирование задавливания|фиксировать признаки задавливания поросят|
|3|Система|Управление кормушками и поилками|управлять кормушками и поилками разных производителей|
|4|Зоотехник/Система|Оценка состояния животных|оценивать состояние животных по внешнему виду и поведению: болезнь, гибель, беспокойство и т.д.|
|5|Система|Мониторинг фильтрации воды|следить за состоянием систем фильтрации воды|
|6|Система|Пересчет поголовья|пересчитывать поголовье|
|7|Система|Управление запасами еды|следить за запасами еды и прогнозировать расход|
|8|Система|Видеомониторинг|поддерживать количество видеокамер для аналитики в реальном времени от разных производителей|
|9|Система/агенты|Архитектура ЦС-агенты|быть построена по принципу «центральный сервер — агенты» с поддержкой задержки до 10 минут|
|10|Система|Метрики|предоставлять базовые метрики для передачи в другие системы|
|11|Система|Пользовательские метрики|поддерживать возможность добавления собственных метрик|
|12|Система|Офлайн-режим|работать без интернета и синхронизироваться при восстановлении связи|
|13|Администратор|Роли и безопасность|иметь разделение ролей и поддерживать современные способы аутентификации и авторизации|
|14|Внешние приложения|API|иметь API для создания мобильного/веб-приложения|

### <a name="_u8xz25hbrgql"></a>**Нефункциональные требования**
|**№**|**Требование**|
| :-: | :- |
|1|обеспечивать достаточно высокую отказоустойчивость 99,95%|
|2|быть расширяемой, то есть иметь возможность разработать новый функционал без изменений существующего|
|3|иметь высокую производительность — от момента возникновения нештатной ситуации, зафиксированной с помощью видеоаналитики, должно проходить не более 5 секунд до момента оповещения|
|4|позволять системе видеоаналитики реагировать в реальном времени (миллисекунды)|
### <a name="_qmphm5d6rvi3"></a>**Решение**

```plantuml
@startuml C2_Containers
skinparam shadowing false
skinparam componentStyle rectangle
skinparam nodeStyle plain
title C2: Контейнеры - ЦС АгроПромХ Животноводство

actor "Зоотехник" as Zootech
actor "Администратор" as Admin
actor "Дежурный сотрудник на ферме" as LocalOperator

rectangle "Облако (ЦС АгроПромХ)" #LightGray {
  component "IoT-шлюз" as IoTGateway <<gRPC/HTTPS>>
  component "Брокер Kafka" as KafkaBroker <<Kafka Protocol>>
}

rectangle "Граница фермы" #LightBlue {

  node "ЦС АгроПромХ - Животноводство (Edge-сервер фермы)" as ArchMS {
    
    component "Локальная система оповещения" as LocalAlerting <<SMS/Telegram/WebSocket>>
    
    node "Контекст: Кормление" as Feeding {
      component "Периферийные устройства" as Peripheral <<Modbus/MQTT/OPC UA>>
      component "Обработка и аналитика данных" as DataProcessing <<Python/Java>>
      component "Локальное хранилище данных" as LocalStorage <<TimescaleDB/InfluxDB>>
      component "Менеджер управления устройствами" as DeviceManager <<Адаптеры производителей>>
    }
    
    node "Контекст: Безопасность" as Security {
      component "Периферийные устройства безопасности" as SecPeripheral <<Modbus/MQTT>>
      component "Обработка и аналитика безопасности" as SecDataProcessing <<Python/Java>>
      component "Локальное хранилище безопасности" as SecLocalStorage <<TimescaleDB>>
    }
    
    node "Контекст: Мониторинг поведения" as Monitoring {
      component "Периферийные устройства мониторинга" as MonPeripheral <<RTSP/ONVIF>>
      node "Обработка и аналитика мониторинга" as MonDataProcessing {
        component "Нейросетевой адаптер паттернов поведения" as NeuralBehaviorAdapter <<ONNX/TensorRT>>
      }
      component "Локальное хранилище мониторинга" as MonLocalStorage <<MinIO/Redis>>
    }
    
    component "Система аутентификации и авторизации" as AuthZ <<Keycloak/OAuth2/JWT>>
    component "API для внешних систем" as ExternalAPI <<REST/GraphQL/WebSocket>>
    component "Пользовательские метрики" as UserDefinedMetrics <<Prometheus/OpenMetrics>>
    component "Менеджер синхронизации с ЦС" as SyncManager <<gRPC/HTTPS+Zstd>>
  }
  
  node "Физические устройства" as Devices {
    database "Кормушки/поилки" as Feeders <<Modbus/MQTT>>
    database "Датчики" as Sensors <<Modbus/MQTT>>
    database "Видеокамеры" as Cameras <<RTSP/ONVIF>>
  }
}

' Пользователи
Zootech --> Peripheral : управляет устройствами
Zootech --> DataProcessing : настраивает/контролирует
Zootech --> SecPeripheral : контролирует безопасность
Zootech --> SecDataProcessing : анализирует безопасность
Zootech --> MonPeripheral : контролирует мониторинг
Zootech --> MonDataProcessing : анализирует мониторинг
Zootech --> LocalAlerting : получает уведомления
Zootech --> AuthZ : выполняет вход/права

Admin --> AuthZ : управляет ролями
LocalOperator --> LocalAlerting : получает уведомления (SMS/Telegram/звук)

' Устройства на ферме
Cameras --> MonPeripheral : RTSP/ONVIF
Sensors --> SecPeripheral : Modbus/MQTT
Feeders --> Peripheral : Modbus/MQTT (телеметрия)
Peripheral --> Feeders : Modbus/MQTT (команды)

' Кормление
Peripheral --> LocalStorage : запись телеметрии
Peripheral --> DeviceManager : получение команд
DeviceManager --> Peripheral : управление устройствами
DataProcessing <--> LocalStorage : SQL/TimescaleDB
DataProcessing --> LocalAlerting : gRPC/внутренний вызов

' Безопасность
SecPeripheral --> SecLocalStorage : запись телеметрии
SecDataProcessing <--> SecLocalStorage : SQL
SecDataProcessing --> LocalAlerting : gRPC/внутренний вызов

' Мониторинг поведения
MonPeripheral --> NeuralBehaviorAdapter : видеокадры (Shared Memory/gRPC)
NeuralBehaviorAdapter --> "Нейросетевая модель" : ONNX/TensorRT
MonDataProcessing <--> MonLocalStorage : S3/Redis
MonDataProcessing --> LocalAlerting : gRPC/внутренний вызов

' Синхронизация с облаком (задержка до 10 минут)
LocalStorage --> SyncManager : данные
SecLocalStorage --> SyncManager : данные
MonLocalStorage --> SyncManager : данные
UserDefinedMetrics --> SyncManager : метрики

SyncManager --> IoTGateway : gRPC/HTTPS (сжатие Zstd)

' Облачная ЦС
IoTGateway --> KafkaBroker : Kafka Protocol
KafkaBroker --> DataProcessing : Kafka Consumer
KafkaBroker --> SecDataProcessing : Kafka Consumer
KafkaBroker --> MonDataProcessing : Kafka Consumer

' API и внешние системы
AuthZ --> ExternalAPI : авторизация (JWT)
SyncManager --> ExternalAPI : предоставляет данные (REST/GraphQL)
LocalStorage --> UserDefinedMetrics : экспорт метрик (Prometheus)
SecLocalStorage --> UserDefinedMetrics : экспорт метрик
MonLocalStorage --> UserDefinedMetrics : экспорт метрик

@enduml
```

|**№**|**Принцип решения**|
| :-: | :- |
|1|Используем подход DDD в решении, разделяем решение на контексты.|
|2|Используем существующий брокер сообщенний для взаимодействия между всеми сервисами.|
|3|Используем функциональность существующего решения ЦС АгроПромХ для минимизации дублирования логики.|

### <a name="_bjrr7veeh80c"></a>**Альтернативы**

```plantuml
@startuml C2_Alt
skinparam shadowing false
skinparam componentStyle rectangle
skinparam nodeStyle plain
title C2: Контейнеры - Управление фермой (упрощённая версия)

actor "Зоотехник" as Zootech
actor "Администратор" as Admin
actor "Дежурный сотрудник" as LocalOperator

rectangle "Облако (ЦС АгроПромХ)" #LightGray {
  component "IoT-шлюз" as IoTGateway <<gRPC/HTTPS>>
  component "Брокер Kafka" as Kafka <<Kafka Protocol>>
}

rectangle "Граница фермы" #LightPink {

  node "Система управления фермой (монолит)" as FarmSystem {
    
    component "Локальное хранилище" as LocalStorage <<SQLite/PostgreSQL>>
    
    component "Видеоаналитика" as VideoAnalytics <<Python/Java/C++>> {
      component "Нейросеть" as NeuralNet <<ONNX/TensorRT>>
    }
    
    component "Управление устройствами" as DeviceControl <<Modbus/MQTT/OPC UA>>
    
    component "Аналитика данных" as DataAnalytics <<Python/Java>>
    
    component "Оповещение" as Alerting <<SMS/Telegram/WebSocket>>
    
    component "Аутентификация" as Auth <<OAuth2/JWT/Keycloak>>
    
    component "API" as API <<REST/GraphQL/WebSocket>>
    
    component "Синхронизация" as Sync <<gRPC/HTTPS+Zstd>>
  }
  
  node "Физические устройства" as Devices {
    database "Кормушки/поилки" as Feeders <<Modbus/MQTT>>
    database "Датчики" as Sensors <<Modbus/MQTT>>
    database "Видеокамеры" as Cameras <<RTSP/ONVIF>>
  }
}

' Пользователи
Zootech --> DeviceControl : управляет кормушками/поилками
Zootech --> DataAnalytics : смотрит аналитику
Zootech --> Alerting : получает уведомления
Zootech --> Auth : входит в систему

Admin --> Auth : управляет правами
LocalOperator --> Alerting : получает уведомления (SMS/звук)

' Устройства на ферме
Cameras --> VideoAnalytics : RTSP/ONVIF
Sensors --> DataAnalytics : Modbus/MQTT
Feeders --> DeviceControl : Modbus/MQTT (телеметрия)
DeviceControl --> Feeders : Modbus/MQTT (команды)

' Внутренняя логика
VideoAnalytics --> NeuralNet : ONNX/TensorRT
VideoAnalytics --> Alerting : события (внутренний вызов)
VideoAnalytics --> LocalStorage : сохраняет снимки (S3/файловая система)

DataAnalytics --> LocalStorage : сохраняет данные (SQL)
DataAnalytics --> Alerting : аномалии (внутренний вызов)

DeviceControl --> LocalStorage : сохраняет команды (SQL)

Alerting --> LocalOperator : SMS/GSM-модем (офлайн)
Alerting --> Zootech : Telegram/WebSocket

' Синхронизация с ЦС (задержка до 10 минут)
LocalStorage --> Sync : данные на отправку
Sync --> IoTGateway : gRPC/HTTPS + Zstd

' Облачная ЦС
IoTGateway --> Kafka : Kafka Protocol
Kafka --> DataAnalytics : Kafka Consumer

' Внешние системы
Auth --> API : авторизация (JWT)
Sync --> API : данные для мобильного приложения (REST/GraphQL)

@enduml
```

### <a name="_88stt5hww918"></a>**Недостатки, ограничения, риски**

*Основное решение (распределенное)*

Преимущества:

- Высокая масштабируемость — каждый контекст масштабируется независимо
- Отказоустойчивость — изоляция контекстов локализует сбои
- Расширяемость без изменений существующего — новый функционал добавляется как отдельный контекст

Недостатки и риски:

- Высокая сложность разработки и развёртывания
- Требуются эксперты по распределённым системам и Kafka
- Высокие операционные затраты (кластер из сервисов)
- Долгий вывод на рынок

*Альтернативное решение (монолитное)*

Преимущества:

- Быстрый вывод на рынок
- Низкий порог входа для команды — не требуются эксперты по распределённым системам
- Минимальные операционные затраты — один сервер и одна база данных вместо кластера

Недостатки и риски:

- Низкая отказоустойчивость — падение любого компонента останавливает всю систему
- Плохая расширяемость — изменения затрагивают весь монолит
- Ограниченная масштабируемость
- Невозможно гарантировать 99.95% доступности
