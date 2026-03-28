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


### <a name="_3bfxc9a45514"></a>**Контекст Мониторинг поведения**

```plantuml
@startuml C3_Monitoring
skinparam shadowing false
skinparam componentStyle rectangle
title C3: Компоненты — Контекст Мониторинг поведения

actor "Зоотехник" as Zootech
actor "Дежурный сотрудник" as LocalOperator

package "Контекст: Мониторинг поведения" {
  
  component "Периферийные устройства мониторинга" as MonPeripheral {
    component "Менеджер видеокамер" as CamManager
    component "Адаптер камер Hikvision" as CamHik
    component "Адаптер камер Dahua" as CamDahua
    component "Адаптер камер Axis" as CamAxis
    component "Менеджер RTSP-потоков" as RTSPManager
    component "Ночной корректор изображения" as NightCorrector
    component "Декодер кадров" as FrameDecoder
  }
  
  component "Обработка и аналитика мониторинга" as MonDataProcessing {
    
    component "Диспетчер видеопотоков" as StreamDispatcher
    
    component "Нейросетевой адаптер паттернов поведения" as NeuralBehaviorAdapter {
      component "Обёртка нейромодели" as NeuralWrapper
      component "Препроцессор изображений" as Preprocessor
      component "Постпроцессор" as Postprocessor
    }
    
    component "Трекер животных" as AnimalTracker {
      component "Детектор" as DetectorInput
      component "Алгоритм SORT" as SORT
      component "Сопоставление треков" as TrackMatcher
    }
    
    component "Классификатор поведения" as BehaviorClassifier {
      component "Агрегатор треков" as TrackAggregator
      component "Детектор драк" as FightDetector
      component "Детектор беспокойства" as AnxietyDetector
      component "Детектор задавливания" as CrushingDetector
      component "Детектор болезней" as DiseaseDetector
      component "Детектор гибели" as DeathDetector
    }
    
    component "Менеджер событий" as EventManager
  }
  
  component "Локальное хранилище мониторинга" as MonLocalStorage {
    component "Буфер видеокадров" as VideoBuffer
    component "Хранилище треков" as TracksDB
    component "Хранилище событий" as EventsStore
    component "Хранилище снимков" as SnapshotsStore
  }
  
  component "Локальная система оповещения" as LocalAlerting
  
  component "Менеджер синхронизации с ЦС" as SyncManager
}

CamManager --> CamHik : управляет
CamManager --> CamDahua : управляет
CamManager --> CamAxis : управляет
CamHik --> RTSPManager : поток RTSP
CamDahua --> RTSPManager : поток RTSP
CamAxis --> RTSPManager : поток RTSP

RTSPManager --> NightCorrector : кадры
NightCorrector --> FrameDecoder : кадры
FrameDecoder --> StreamDispatcher : кадры

StreamDispatcher --> NeuralBehaviorAdapter : кадры
StreamDispatcher --> VideoBuffer : кадры

NeuralBehaviorAdapter --> Preprocessor : кадры
Preprocessor --> NeuralWrapper : тензоры
NeuralWrapper --> "Нейросетевая модель" : вызов
"Нейросетевая модель" --> Postprocessor : детекции
Postprocessor --> DetectorInput : bbox

DetectorInput --> SORT : детекции
SORT --> TrackMatcher : треки
TrackMatcher --> TrackAggregator : треки

TrackAggregator --> FightDetector : треки
TrackAggregator --> AnxietyDetector : треки
TrackAggregator --> CrushingDetector : треки
TrackAggregator --> DiseaseDetector : треки
TrackAggregator --> DeathDetector : треки

FightDetector --> EventManager : событие
AnxietyDetector --> EventManager : событие
CrushingDetector --> EventManager : событие
DiseaseDetector --> EventManager : событие
DeathDetector --> EventManager : событие

TrackAggregator --> TracksDB : треки
EventManager --> EventsStore : события
EventManager --> SnapshotsStore : снимки
VideoBuffer --> SnapshotsStore : кадры

EventManager --> LocalAlerting : событие
LocalAlerting --> Zootech : уведомление
LocalAlerting --> LocalOperator : уведомление

TracksDB --> SyncManager : треки
EventsStore --> SyncManager : события
SnapshotsStore --> SyncManager : снимки

Zootech --> CamManager : настройка
Zootech --> StreamDispatcher : управление
Zootech --> BehaviorClassifier : настройка

@enduml
```

### <a name="_3bfxc9a45514"></a>**Контекст Безопасность**

```plantuml
@startuml C3_Security
skinparam shadowing false
skinparam componentStyle rectangle
title C3: Компоненты — Контекст Безопасность

actor "Зоотехник" as Zootech
actor "Дежурный сотрудник" as LocalOperator

package "Контекст: Безопасность" {
  
  component "Периферийные устройства безопасности" as SecPeripheral {
    component "Менеджер датчиков" as SensorManager
    component "Адаптер датчиков температуры" as TempSensor
    component "Адаптер датчиков влажности" as HumidSensor
    component "Адаптер датчиков газа" as GasSensor
    component "Адаптер датчиков дыма" as SmokeSensor
    component "Адаптер датчиков протечки" as LeakSensor
    component "Менеджер систем фильтрации" as FilterManager
    component "Адаптер фильтрации воды" as WaterFilterAdapter
    component "Менеджер опроса датчиков" as PollingManager
  }
  
  component "Обработка и аналитика безопасности" as SecDataProcessing {
    component "Сборщик телеметрии" as TelemetryCollector
    component "Анализатор показателей" as MetricsAnalyzer
    component "Детектор аномалий" as AnomalyDetector
    component "Анализатор систем фильтрации" as FilterAnalyzer
    component "Детектор критических значений" as CriticalDetector
    component "Менеджер правил" as RulesManager
    component "Логика безопасности" as SecurityLogic
    component "Генератор сигналов" as SignalGenerator
  }
  
  component "Локальное хранилище безопасности" as SecLocalStorage {
    component "Хранилище телеметрии" as TelemetryStore
    component "Хранилище аномалий" as AnomalyStore
    component "Хранилище состояния фильтров" as FilterStateStore
    component "Журнал событий" as EventLog
  }
  
  component "Локальная система оповещения" as LocalAlerting
  
  component "Менеджер синхронизации с ЦС" as SyncManager
}

SensorManager --> TempSensor : управляет
SensorManager --> HumidSensor : управляет
SensorManager --> GasSensor : управляет
SensorManager --> SmokeSensor : управляет
SensorManager --> LeakSensor : управляет

TempSensor --> PollingManager : показания
HumidSensor --> PollingManager : показания
GasSensor --> PollingManager : показания
SmokeSensor --> PollingManager : показания
LeakSensor --> PollingManager : показания

FilterManager --> WaterFilterAdapter : управляет
WaterFilterAdapter --> PollingManager : состояние

PollingManager --> TelemetryCollector : телеметрия

TelemetryCollector --> TelemetryStore : сохраняет
TelemetryCollector --> MetricsAnalyzer : передаёт

MetricsAnalyzer --> AnomalyDetector : показатели
MetricsAnalyzer --> FilterAnalyzer : показатели фильтров

AnomalyDetector --> AnomalyStore : аномалии
AnomalyDetector --> CriticalDetector : аномалии

FilterAnalyzer --> FilterStateStore : состояние
FilterAnalyzer --> CriticalDetector : отклонения

RulesManager --> SecurityLogic : правила
CriticalDetector --> SecurityLogic : критические значения

SecurityLogic --> SignalGenerator : сигнал тревоги
SignalGenerator --> EventLog : запись

SignalGenerator --> LocalAlerting : сигнал тревоги
LocalAlerting --> Zootech : уведомление
LocalAlerting --> LocalOperator : уведомление

TelemetryStore --> SyncManager : телеметрия
AnomalyStore --> SyncManager : аномалии
FilterStateStore --> SyncManager : состояние
EventLog --> SyncManager : события

Zootech --> RulesManager : настройка правил
Zootech --> SensorManager : конфигурация датчиков
Zootech --> FilterManager : управление фильтрацией

@enduml
```

### <a name="_3bfxc9a45514"></a>**Контекст Кормление**

```plantuml
@startuml C3_Feeding
skinparam shadowing false
skinparam componentStyle rectangle
title C3: Компоненты — Контекст Кормление

actor "Зоотехник" as Zootech
actor "Дежурный сотрудник" as LocalOperator

package "Контекст: Кормление" {
  
  component "Периферийные устройства" as Peripheral {
    component "Менеджер кормушек" as FeederManager
    component "Адаптер кормушек DeLaval" as FeederDeLaval
    component "Адаптер кормушек GEA" as FeederGEA
    component "Адаптер кормушек Big Dutchman" as FeederBigDutchman
    component "Менеджер поилок" as DrinkerManager
    component "Адаптер поилок DeLaval" as DrinkerDeLaval
    component "Адаптер поилок GEA" as DrinkerGEA
    component "Датчики уровня корма" as FeedLevelSensor
    component "Датчики расхода воды" as WaterFlowSensor
    component "Датчики веса" as WeightSensor
    component "Командно-исполнительный блок" as CommandExecutor
  }
  
  component "Менеджер управления устройствами" as DeviceManager {
    component "Диспетчер команд" as CommandDispatcher
    component "Валидатор команд" as CommandValidator
    component "Очередь команд" as CommandQueue
    component "Протокольный адаптер" as ProtocolAdapter
    component "Менеджер состояния устройств" as DeviceStateManager
  }
  
  component "Обработка и аналитика данных" as DataProcessing {
    component "Сборщик данных с устройств" as DataCollector
    component "Анализатор расхода корма" as FeedConsumptionAnalyzer
    component "Анализатор расхода воды" as WaterConsumptionAnalyzer
    component "Детектор отклонений" as DeviationDetector
    component "Прогнозатор запасов" as StockForecaster
    component "Генератор рекомендаций" as RecommendationGenerator
    component "Менеджер расписаний" as ScheduleManager
  }
  
  component "Локальное хранилище данных" as LocalStorage {
    component "Хранилище расхода корма" as FeedConsumptionStore
    component "Хранилище расхода воды" as WaterConsumptionStore
    component "Хранилище весов" as WeightStore
    component "Хранилище запасов" as StockStore
    component "Хранилище поголовья" as AnimalCountStore
    component "Журнал команд" as CommandLog
  }
  
  component "Локальная система оповещения" as LocalAlerting
  
  component "Менеджер синхронизации с ЦС" as SyncManager
  
  component "Пользовательские метрики" as UserDefinedMetrics
}

FeederManager --> FeederDeLaval : управляет
FeederManager --> FeederGEA : управляет
FeederManager --> FeederBigDutchman : управляет

DrinkerManager --> DrinkerDeLaval : управляет
DrinkerManager --> DrinkerGEA : управляет

FeederDeLaval --> CommandExecutor : команды
FeederGEA --> CommandExecutor : команды
FeederBigDutchman --> CommandExecutor : команды
DrinkerDeLaval --> CommandExecutor : команды
DrinkerGEA --> CommandExecutor : команды

FeedLevelSensor --> DataCollector : уровень корма
WaterFlowSensor --> DataCollector : расход воды
WeightSensor --> DataCollector : вес животных

CommandExecutor --> CommandDispatcher : статус команд

CommandDispatcher --> CommandValidator : команда
CommandValidator --> CommandQueue : валидная команда
CommandQueue --> ProtocolAdapter : команда на отправку
ProtocolAdapter --> FeederManager : управление
ProtocolAdapter --> DrinkerManager : управление

CommandDispatcher --> CommandLog : запись
DeviceStateManager --> CommandDispatcher : состояние устройств

DataCollector --> FeedConsumptionStore : данные
DataCollector --> WaterConsumptionStore : данные
DataCollector --> WeightStore : данные

DataCollector --> FeedConsumptionAnalyzer : данные
DataCollector --> WaterConsumptionAnalyzer : данные

FeedConsumptionAnalyzer --> DeviationDetector : отклонения
WaterConsumptionAnalyzer --> DeviationDetector : отклонения

FeedConsumptionAnalyzer --> StockForecaster : остатки
StockForecaster --> StockStore : прогноз

DeviationDetector --> RecommendationGenerator : проблемы
RecommendationGenerator --> ScheduleManager : корректировка

DeviationDetector --> LocalAlerting : критическое отклонение
LocalAlerting --> Zootech : уведомление
LocalAlerting --> LocalOperator : уведомление

FeedConsumptionStore --> SyncManager : данные
WaterConsumptionStore --> SyncManager : данные
WeightStore --> SyncManager : данные
StockStore --> SyncManager : данные
AnimalCountStore --> SyncManager : данные
CommandLog --> SyncManager : логи

FeedConsumptionStore --> UserDefinedMetrics : метрики
WaterConsumptionStore --> UserDefinedMetrics : метрики
WeightStore --> UserDefinedMetrics : метрики
StockStore --> UserDefinedMetrics : метрики

Zootech --> AnimalCountStore : ввод поголовья
Zootech --> ScheduleManager : настройка расписаний
Zootech --> FeederManager : конфигурация
Zootech --> DrinkerManager : конфигурация
Zootech --> DeviceStateManager : мониторинг
Zootech --> RecommendationGenerator : просмотр рекомендаций

@enduml
```

|**№**|**Принцип решения**|
| :-: | :- |
|1|Используем подход DDD в решении, разделяем решение на контексты.|
|2|Используем существующий брокер сообщенний для взаимодействия между всеми сервисами.|
|3|Используем функциональность существующего решения ЦС АгроПромХ для минимизации дублирования логики.|

### <a name="_bjrr7veeh80c"></a>**Альтернативы**

```plantuml
@startuml C3_Alt
skinparam shadowing false
skinparam componentStyle rectangle
title C3: Компоненты - Управление фермой (упрощённая версия)

actor "Зоотехник" as Zootech
actor "Дежурный сотрудник" as LocalOperator

package "Система управления фермой" {
  
  component "Управление устройствами" as DeviceControl {
    component "Менеджер кормушек" as FeederManager
    component "Менеджер поилок" as DrinkerManager
    component "Адаптеры производителей" as DeviceAdapters
    component "Менеджер датчиков" as SensorManager
  }
  
  component "Видеоаналитика" as VideoAnalytics {
    component "Менеджер камер" as CameraManager
    component "Адаптеры камер" as CameraAdapters
    component "Нейросеть" as NeuralNet
    component "Детектор событий" as EventDetector
  }
  
  component "Аналитика данных" as DataAnalytics {
    component "Сборщик телеметрии" as TelemetryCollector
    component "Анализатор расхода" as ConsumptionAnalyzer
    component "Прогнозатор запасов" as StockForecaster
    component "Детектор аномалий" as AnomalyDetector
  }
  
  component "Локальное хранилище" as LocalStorage {
    component "Хранилище телеметрии" as TelemetryStore
    component "Хранилище событий" as EventsStore
    component "Хранилище снимков" as SnapshotsStore
  }
  
  component "Оповещение" as Alerting {
    component "Диспетчер уведомлений" as NotificationDispatcher
    component "Локальный канал" as LocalChannel
  }
  
  component "Синхронизация" as Sync {
    component "Буфер отправки" as OutboxBuffer
    component "Менеджер соединения" as ConnectionManager
  }
  
  component "API" as API {
    component "REST API" as REST
    component "WebSocket" as WS
  }
  
  component "Аутентификация" as Auth {
    component "Управление ролями" as RoleManager
    component "Провайдеры аутентификации" as AuthProviders
  }
}

DeviceControl --> DeviceAdapters : использует
DeviceAdapters --> FeederManager : управляет
DeviceAdapters --> DrinkerManager : управляет
SensorManager --> TelemetryCollector : телеметрия

VideoAnalytics --> CameraManager : управляет
CameraManager --> CameraAdapters : использует
CameraAdapters --> NeuralNet : видеопоток
NeuralNet --> EventDetector : детекции
EventDetector --> EventsStore : события

TelemetryCollector --> TelemetryStore : сохраняет
TelemetryCollector --> ConsumptionAnalyzer : данные
ConsumptionAnalyzer --> StockForecaster : остатки
ConsumptionAnalyzer --> AnomalyDetector : отклонения
AnomalyDetector --> EventsStore : аномалии

EventsStore --> NotificationDispatcher : события
NotificationDispatcher --> LocalChannel : отправка
LocalChannel --> LocalOperator : SMS/звук

TelemetryStore --> OutboxBuffer : данные
EventsStore --> OutboxBuffer : события
SnapshotsStore --> OutboxBuffer : снимки
OutboxBuffer --> ConnectionManager : отправка

Auth --> REST : авторизация
Auth --> WS : авторизация
REST --> Sync : данные
WS --> Alerting : уведомления в реальном времени

Zootech --> FeederManager : управление
Zootech --> DataAnalytics : просмотр
Zootech --> Auth : вход

@enduml
```
