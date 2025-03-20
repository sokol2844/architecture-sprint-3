# Project_template

Тип: Материал
Родитель: Описание проекта для 11 когорты (https://www.notion.so/11-03abbbbc8bcb49ed9b85c9b6d1174056?pvs=21)

Это шаблон для решения проектной работы. Структура этого файла повторяет структуру заданий. Заполняйте его по мере работы над решением.

# Задание 1. Анализ и планирование


### 1. Описание функциональности монолитного приложения

**Управление отоплением:**

- Пользователи могут удалённо включать/выключать отопление в своих домах

**Мониторинг температуры:**

- Пользователи могут просматривать текущую температуру в своих домах
- Система поддерживает собственные датчики

### 2. Анализ архитектуры монолитного приложения

- Язык программирования: Java
- База данных: PostgreSQL
- Архитектура: Монолитная, все компоненты системы (обработка запросов, бизнес-логика, работа с данными) находятся в рамках одного приложения.
- Взаимодействие: Синхронное, запросы обрабатываются последовательно.
- Масштабируемость: Ограничена, так как монолит сложно масштабировать по частям.
- Развёртывание: Требует остановки всего приложения.

### 3. Определение доменов и границы контекстов

- Управление устройствами. Домен включает в себя взаимодействие с разными умными устройствами, такими как отопление, освещение, ворота, системы наблюдения
- Аутентификация. Домен включает методы аутентификации и управления пользователями
- Телеметрия. Домен включает в себя сбор данных с датчиков
- Тех поддержка. Домен включает в себя методы для поддержки пользователей и помощи в подключении
- Нотификации. Домен службы нотификации для уведомлений
- Мониторинг. Домен отвечает за сбор и анализ данных системы

### **4. Проблемы монолитного решения**

- Высокий риск ошибок. Изменения в одной части приложения могут непредсказуемо влиять на другие части.
- Длительные циклы разработки и развёртывания. При каждом изменении приходится тестировать всё приложение целиком. Это замедляет выпуск новых функций.
- Трудно масштабировать отдельные компоненты системы.


Компания планирует сильно раширять свой функционал и возможности, такой переход реализовать в рамках монолитной системы будет тяжело и поддерживать и развивать систему будет сложно и дорого.

### 5. Визуализация контекста системы — диаграмма С4

```markdown
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml

Person(user, "Пользователь", "Управляет системой отопления и получает данные о температуре.")

System(monolith, "Монолит / Умный дом", "Монолитная система, ответственная за управление отоплением и мониторинг температуры")

System_Ext(sensor, "Датчик", "Измеряет и передает температуру в систему для анализа и управления.")

Rel(user, monolith, "Использует приложение для управления системой отопления и мониторинга температуры.")
Rel(monolith, sensor, "Регулировка температуры")
Rel(sensor, monolith, "Передача данных о температуре")
@enduml
```

# Задание 2. Проектирование микросервисной архитектуры

**Диаграмма контейнеров (Containers)**

```markdown
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml



Person(user, "Пользователь")

System_Boundary(smartHome, "Умный дом") {
    Container(apiGateway, "API Gateway", "Кеширование и маршрутизация запросов")

    Container(managmentService, "Сервис управления устройствами", "Java", "Контроль и управление устройствами")
    ContainerDb(managmentDb, "Managment DB", "PostgreSQL", "Хранение информации об устройствах")

    Container(authService, "Сервис аутентификации", "Java", "Аутентификация и управление пользователями")
    ContainerDb(authDb, "Auth DB", "PostgreSQL", "Хранение данных пользователей и ролей")
    
    Container(telemetryService, "Сервис телеметрии", "Java", "Сбор, обработка и анализ данных с устройств")
    ContainerDb(telemetryDb, "Telemetry DB", "PostgreSQL", "Хранение телеметрических данных")

    Container(supportService, "Сервис тех поддержки", "Java", "Обработка запросов в техподдержку")
    ContainerDb(supportDb, "Support DB", "PostgreSQL", "Хранение тикетов поддержки")
    
    Container(notificationService, "Сервис нотификаций", "Java", "Отправка уведомлений пользователям")
    ContainerDb(notificationDb, "Notification DB", "PostgreSQL", "Хранение истории уведомлений")
    
    Container(monitoringService, "Сервис мониторинга", "Python", "Сбор, анализ данных системы и генерация отчетов")
    ContainerDb(monitoringDb, "Monitoring DB", "PostgreSQL", "Хранение аналитических данных")
    
    SystemQueue(kafka, "Kafka", "Брокер сообщений для асинхронного взаимодействия сервисов")
    
}

System_Ext(sensors, "Датчики", "Датчики температуры, умные розетки, камеры наблюдения и тд")

' Взаимодействия между контейнерами
Rel(user, apiGateway, "Взаимодействие через браузер или приложение")

Rel(apiGateway, managmentService, "Запросы на управление устройствами")
Rel(apiGateway, telemetryService, "Запрос данных с датчиков")
Rel(apiGateway, authService, "Аутентификация и авторизация пользователей")
Rel(apiGateway, supportService, "Взаимодействие с тикетами поддержки")
Rel(apiGateway, notificationService, "Получение уведомлений")
Rel(apiGateway, monitoringService, "Запрос аналитики")

' Взаимодействия с датчиками
Rel(managmentService, sensors, "Передача запросов на управление устройствами")
Rel(telemetryService, sensors, "Сбор данных датчиков")

' Взаимодействия с кафкой
Rel(telemetryService, kafka, "Передача телеметрических данных, а также данных любых изменений")
Rel(notificationService, kafka, "Получение необходимой информации для генерации нотификаций")
Rel(managmentService, kafka, "Получение событий и телеметрических данных")
Rel(monitoringService, kafka, "Получение данных для генерации отчетов ")

' Взаимодействия с БД
Rel(managmentService, managmentDb, "Сохранение и получение данных")
Rel(telemetryService, telemetryDb, "Сохранение и получение данных")
Rel(authService, authDb, "Сохранение и получение данных")
Rel(supportService, supportDb, "Сохранение и получение данных")
Rel(notificationService, notificationDb, "Сохранение и получение данных")
Rel(monitoringService, monitoringDb, "Сохранение и получение данных")

@enduml
```

**Диаграмма компонентов (Components)**

Для примера реализации диаграммы компонентов углубимся в контейнер Управления устройствами

```markdown
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml

Person(user, "Пользователь")

System_Boundary(smartHome, "Умный дом") {

    Container(apiGateway, "API Gateway")


    Container_Boundary(managmentContainer, "Управление устройствами") {
        Component(api, "API", "Java")
        Component(registrationComponent, "Регистрация новых устройств", "Java")
        Component(handlersComponent, "Обработчик команд", "Java")
        Component(stateComponent, "Менеджер состояния устройств", "Java")

        ContainerDb(managmentDb, "Managment DB", "PostgreSQL", "Хранение информации об устройствах")
    }

    SystemQueue(kafka, "Kafka", "Брокер сообщений для асинхронного взаимодействия сервисов")
}

Rel(user, apiGateway, "Взаимодействие с системой через UI")
Rel(apiGateway, api, "Запросы к сервису управлением устройств")
Rel(api, registrationComponent, "Запросы на регистрацию новых устройств")
Rel(api, handlersComponent, "Запросы на управление устройствами")
Rel(handlersComponent, kafka, "Публикация событий управления устройствами")
Rel(stateComponent, kafka, "Вычитка изменения состояния устройств")
Rel(handlersComponent, stateComponent, "Получение измененного состояния устройства для завершения события управления устройством")
Rel(registrationComponent, managmentDb, "Хранение данных сервиса")
Rel(handlersComponent, managmentDb, "Хранение данных сервиса")
Rel(stateComponent, managmentDb, "Хранение данных сервиса")
@enduml
```

**Диаграмма кода (Code)**

```markdown
@startuml
title Диаграмма кода, показывающая принцип наследования классов для разных устройств

!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Component.puml

class User {
  +String name
  +String email
  +String id
  +List<Device> devices
}

interface Device {
  +String id
  +{abstract} void getState()
  +{abstract} void setState()
}

class HeatingDevice {
  +String id
  +HeatingDeviceState getState()
  +void setState()
}

class HeatingDeviceState {
  +Number currentTemperature
  +Number targetTemperature
  +Number power
}

class LightDevice {
  +String id
  +LightDeviceState getState()
  +void setState()
}

class LightDeviceState {
  +String currentLightPower
  +String operatingMode
}

User "1" -- "0..*" HeatingDevice : has
User "1" -- "0..*" LightDevice : has
Device <|.. HeatingDevice
HeatingDevice "1" -- "1" HeatingDeviceState : includes
Device <|.. LightDevice
LightDevice "1" -- "1" LightDeviceState : includes
@enduml
```

# Задание 3. Разработка ER-диаграммы

```markdown
@startuml
entity User {
  * id : UUID
  * name : String
  * email : String
  * created_at : DateTime
}

entity House {
  * id : UUID
  * address : String
  * user_id : UUID
}

entity Device {
  * id : UUID
  * type_id : UUID
  * serial_number : String
  * house_id : String
  * status : Boolean
  * created_at : DateTime
}

entity DeviceType {
  * id : UUID
  * name : String
  * description : String
}

entity Module {
  * id : UUID
  * name : String
  * description : String
  * created_at : DateTime
}

entity TelemetryData {
  * id : UUID
  * device_id : UUID
  * timestamp : DateTime
  * temperature : Number
  * power_status : Boolean
}

User }o-o{ House : "владеет"
House }o--|| Device : "содержит"
Device }o--|| DeviceType : "содержит"
Device }o--|| TelemetryData : "генерирует"
Device }o--|| Module : "содержит"
@enduml
```


# ❌ Задание 4. Создание и документирование API