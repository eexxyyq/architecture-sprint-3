# Задание 1. Анализ и планирование

## Функциональность монолитного приложения

- **Управление отоплением.** Пользователи могут удалённо включать/выключать отопление в своих домах.
- **Мониторинг температуры.** Система получает данные о температуре с датчиков, установленных в домах. Пользователи могут просматривать текущую температуру в своих домах через веб-интерфейс.

## Анализ архитектуры монолитного приложения:
- **Язык программирования**: Java
- **База данных:** PostgreSQL
- **Архитектура:** Монолитная, все компоненты системы (обработка запросов, бизнес-логика, работа с данными) находятся в рамках одного приложения.
- **Взаимодействие**: Синхронное, запросы обрабатываются последовательно.
- **Масштабируемость**: Ограничена, так как монолит сложно масштабировать по частям.
- **Развёртывание**: Требует остановки всего приложения.

## Определение доменов и границ контекстов
- **Домен "Управление системой отопления на объекте":** Позволяет удаленно включать и отключать систему отопления.
- **Домен "Мониторинг метрик с датчиков, подключенных к системе отопления на объекте":** Позволяет в реальном времени получать данные с датчиков, подключенных к системе отопления
- **Домен "Управляющее воздействие и указание целевых параметров системы":** Позволяет удаленно отправлять системой отопления на объекте через управляющее воздействие (повысить температуру или понизить в зависимости от показаний датчиков)
- **Домен "Аутентификация и авторизация":** Позволяет обеспечить безопасный доступ к системам от авторизованных на это пользователей
- **Домен "Уведомления":** Позволяет отправлять уведомления пользователям о состоянии системы, датчиков. Рекламные пуши и тд.

## Анализ проблем монолитного приложения

Рассмотрим проблемы приложения в его текущем состоянии:

- Синхронное взаимодействие. В случае, когда будет повышенная нагрузка от пользователей, приложение может перестать оперативно реагировать на запросы.
- Низкая отказоустойчивость. В случае отказа 1 участка монолита может остановиться вся работа приложения.
- Высокий риск ошибок. Изменения в одной части работы (например при получении системы отопления) можно непредсказуемо повлиять на работу с датчиками.
- Отсутствие слоя работы с безопасностью доступа. С учетом планов на допуск пользователей к самостоятельной регистрации систем, данный слой необходим
- Длительные циклы разработки и развёртывания. При каждом изменении приходится тестировать всё приложение целиком.
- Трудно управлять командой. В процессе развития приложения разработка станет узким местом, так как разработчики будут мешать друг другу в соседних файлах
- Трудно масштабировать отдельные компоненты системы. Если на одном объекте будет установленно большое количество датчиков температуры, их показания могут создать нагрузку на приложение, из-за которого нельзя будет оперативно обрабатывать показания от других систем.
- Гибкость в выборе технологий. Вероятно, показания отдельных датчиков с их историзмом лучше хранить в noSQL БД.
- Гибкость в расширении функциональности. В текущем состоянии система далека от привычного "Умного дома". Скорее это рубильник и реле для управления только температурой помещения, только размещенное зачем-то в web среде.

Текущее состояние системы не позволяет расширяться и реализовать задуманный план развития системы.

## Визуализация контекста монолитного приложения 

Схема контекста реализована в [Legacy_C4_Context.puml](schemas/Legacy_C4_Context.puml)

# Задание 2. Проектирование микросервисной архитектуры

## 1. Декомпозиция приложения на микросервисы


### Основные микросервисы в рамках доменной логики приложения

#### Микросервис управления пользователями

- создание/удаление аккаунтов пользователей 
- Присваивание пользователю уникального идентификатора
- Хранение основных данных о пользователе
- Участвует в аутентификации и авторизации пользователей в системе
- Присвоение прав пользователю в зависимости от доступов

#### Микросервис управления хабами Умного дома

В данном контексте хаб - это некая база, установленная на объекте. Пользователи подключают как свои системы, так и датчики именно к этому хабу, а хаб уже взаимодействует с нашим приложением

- Регистрация/удаление хаба
- Присвоение уникального идентификатора
- Выдача всех параметров системы для отображения пользователям
- Прием управляющих воздействий от пользователя и направление их к хабам, установленным на объектах

Если создание и разработка хаба слишком дорога для компании, данный сервис выступает как internal API gateway приложения.

#### Микросервис управления датчиками

- Сбор и хранение информации с датчиков

#### Микросервис анализа

- Расчет управляющего воздействия на системы умного дома в зависимости от показаний

#### Микросервис управления системами умного дома

- Регистрация частей системы умного дома
- Хранение агрегатного состояния всех систем умного дома по типу системы (управление отоплением, электроприборами, освещением)

#### Микросервис логирования и мониторинга

- Логи для аналитики системы и разбора инцидентов

#### Микросервис уведомлений

- Пуш уведомления
- СМС
- Письма на почту

### Дополнительные части системы

#### Kafka 

Используется для асинхронного обмена сообщениями между микросервисами.

#### API Gateway

Единая точка входа в приложение, управление маршрутизацией, управление безопасностью (авторизация и аутентификация)

## 2. Взаимодействия между компонентами

### Связи между микросервисами

- Связь всегда асинхронная, кроме процессов регистрации, удаления и запроса данных.
- API Gateway является единственным интерфейсом, через который взаимодействуют пользователь и приложение (кроме уведомлений)

### Kafka и микросервисы

Kafka выступает брокером сообщений, через которые микросервисы получают данные и выдают команды для других микросервисов.

Примеры:

- Хаб, имеющий связь с API gateway (или кафкой, но тут спорно из-за вопросов авторизации и аутентификации, 
хоть Kafka и поддерживает SSL и авторизацию, лучше не выставлять ее наружу), публикует полученные от датчиков системы 
показания в кафку. Микросервис управления датчиками получает данные и сохраняет себе с ведением истории показаний.
Далее данный микросервис публикует команду, содержащую данные со всех датчиков этой системы 
(все датчики имеют общий признак с уникальным идентификатором системы, к которой они подключены). Микросервис анализа и управления системами
получает данную команду, запрашивает у Микросервиса управления системами умного дома агрегат системы и на основе этого агрегата и полученных данных от датчиков рассчитывает по алгоритмам управляющее воздействие, например, поднять температуру котла еще на 5 градусов.
- Если управляющее воздействие не требуется и целевое состояние достигнуто, Микросервис анализа и управления системами
публикует команду на пуш уведомление пользователя о том, что сценарий завершен. Микросервис уведомлений
читает команду и отправляет пуш 

### БД и микросервисы

- Каждый микросервис имеет свою БД
- Хранение показаний датчиков лучше организовать с noSQL БД, как например Mongo
- Микросервис управления системами умного дома так же имеет связь с кешем, где хранит агрегат состояния системы

## 3. Визуализация архитектуры:

- C4 Component - [Future_C4_Component.puml](schemas/Future_C4_Component.puml)
- C4 Container - [Future_C4_Container.puml](schemas/Future_C4_Container.puml)
- C4 Code - [Future_C4_Code.puml](schemas/Future_C4_Code.puml)

# Задание 3. Разработка ER-диаграммы

- ER Diagram [Future_C4_ER.puml](schemas/Future_C4_ER.puml)

