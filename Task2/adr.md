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
title C4: Контейнеры (C2) - ЦС АгроПромХ Животноводство

actor "Зоотехник" as Zootech
actor "Администратор" as Admin
actor "Дежурный сотрудник на ферме" as LocalOperator

node "ЦС АгроПромХ (существующая система)" as ExistingCS {
  component "IoT-шлюз" as IoTGateway
  component "Брокер сообщений Kafka" as KafkaBroker
}

node "ЦС АгроПромХ - Животноводство (ваша подсистема)" as ArchMS {
  
  component "Локальная система оповещения" as LocalAlerting
  
  node "Контекст: Кормление" as Feeding {
    component "Периферийные устройства" as Peripheral
    component "Обработка и аналитика данных" as DataProcessing
    component "Локальное хранилище данных" as LocalStorage
    component "Менеджер управления устройствами" as DeviceManager
  }
  
  node "Контекст: Безопасность" as Security {
    component "Периферийные устройства безопасности" as SecPeripheral
    component "Обработка и аналитика безопасности" as SecDataProcessing
    component "Локальное хранилище безопасности" as SecLocalStorage
  }
  
  node "Контекст: Мониторинг поведения" as Monitoring {
    component "Периферийные устройства мониторинга" as MonPeripheral
    node "Обработка и аналитика мониторинга" as MonDataProcessing {
      component "Нейросетевой адаптер паттернов поведения" as NeuralBehaviorAdapter
    }
    component "Локальное хранилище мониторинга" as MonLocalStorage
  }
  
  component "Система аутентификации и авторизации" as AuthZ
  component "API для внешних систем" as ExternalAPI
  component "Пользовательские метрики" as UserDefinedMetrics
  component "Менеджер синхронизации с ЦС" as SyncManager
}

' ===== СВЯЗИ =====

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
LocalOperator --> LocalAlerting : получает уведомления

' Кормление
Peripheral --> LocalStorage : пишет данные
Peripheral --> DeviceManager : получает команды
DeviceManager --> Peripheral : управляет кормушками/поилками
DataProcessing <--> LocalStorage : обмен данными
DataProcessing --> LocalAlerting : создает предупреждения

' Безопасность
SecPeripheral --> SecLocalStorage : пишет данные
SecDataProcessing <--> SecLocalStorage : обмен данными
SecDataProcessing --> LocalAlerting : создает сигналы безопасности

' Мониторинг поведения
MonPeripheral --> NeuralBehaviorAdapter : видеопоток
NeuralBehaviorAdapter --> "Нейросетевая модель\n(предоставлена партнёрами)" : использует
MonDataProcessing <--> MonLocalStorage : обмен данными
MonDataProcessing --> LocalAlerting : создает сигналы мониторинга

' ===== Синхронизация =====
LocalStorage --> SyncManager : отправка данных
SecLocalStorage --> SyncManager : отправка данных
MonLocalStorage --> SyncManager : отправка данных
UserDefinedMetrics --> SyncManager : отправка метрик

SyncManager --> IoTGateway : синхронизация (задержка до 10 минут)

IoTGateway --> KafkaBroker : передает телеметрию
KafkaBroker --> DataProcessing : доставляет данные кормления
KafkaBroker --> SecDataProcessing : доставляет данные безопасности
KafkaBroker --> MonDataProcessing : доставляет данные мониторинга

' ===== API и внешние системы =====
AuthZ --> ExternalAPI : авторизует запросы
SyncManager --> ExternalAPI : предоставляет данные
LocalStorage --> UserDefinedMetrics : базовые метрики
SecLocalStorage --> UserDefinedMetrics : базовые метрики
MonLocalStorage --> UserDefinedMetrics : базовые метрики

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
title C4: Контейнеры (C2) - Управление фермой (простая версия)

actor "Зоотехник" as Zootech
actor "Администратор" as Admin
actor "Дежурный сотрудник" as LocalOperator

node "ЦС АгроПромХ (существующая система)" as ExistingCS {
  component "IoT-шлюз" as IoTGateway
  component "Брокер Kafka" as Kafka
}

node "Система управления фермой" as FarmSystem {
  
  component "Локальное хранилище" as LocalStorage
  
  component "Видеоаналитика" as VideoAnalytics {
    component "Нейросеть" as NeuralNet
  }
  
  component "Управление устройствами" as DeviceControl
  
  component "Аналитика данных" as DataAnalytics
  
  component "Оповещение" as Alerting
  
  component "Аутентификация" as Auth
  
  component "API" as API
  
  component "Синхронизация" as Sync
}

' Пользователи
Zootech --> DeviceControl : управляет кормушками/поилками
Zootech --> DataAnalytics : смотрит аналитику
Zootech --> Alerting : получает уведомления
Zootech --> Auth : входит в систему

Admin --> Auth : управляет правами
LocalOperator --> Alerting : получает уведомления

' Устройства на ферме
DeviceControl --> "Кормушки и поилки" : команды
"Датчики" --> DataAnalytics : телеметрия
"Видеокамеры" --> VideoAnalytics : видеопоток

' Внутренняя логика
VideoAnalytics --> NeuralNet : распознавание
VideoAnalytics --> Alerting : события (драка, задавливание)
VideoAnalytics --> LocalStorage : сохраняет снимки

DataAnalytics --> LocalStorage : сохраняет данные
DataAnalytics --> Alerting : аномалии

DeviceControl --> LocalStorage : сохраняет команды

Alerting --> LocalOperator : SMS/звук (офлайн)
Alerting --> Zootech : уведомления

' Синхронизация с ЦС
LocalStorage --> Sync : данные на отправку
Sync --> IoTGateway : синхронизация

' Существующая ЦС
IoTGateway --> Kafka : телеметрия
Kafka --> DataAnalytics : данные из ЦС

' Внешние системы
Auth --> API : авторизация
Sync --> API : данные для мобильного приложения

@enduml
```