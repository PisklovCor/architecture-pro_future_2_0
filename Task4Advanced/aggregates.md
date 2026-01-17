# Описание агрегатов системы "Будущее 2.0"

## Определение агрегата

Агрегат - это кластер связанных объектов, которые рассматриваются как единое целое. Агрегат имеет корневой объект (Aggregate Root), который является единственной точкой доступа к агрегату извне.

## Медицинский домен

### 1. Patient (Пациент)
**Bounded Context:** Patient Management

**Aggregate Root:** Patient

**Границы агрегата:**
- Patient (корневой объект)
- PersonalInfo (персональная информация)
- ContactInfo (контактная информация)
- InsuranceInfo (информация о страховке)

**Инварианты:**
- У пациента должен быть уникальный идентификатор
- Email и телефон должны быть валидными
- Дата рождения не может быть в будущем
- Пациент не может быть удалён, если у него есть активные записи

**Ключи:**
- Primary Key: patientId (UUID)
- Unique: email, phoneNumber
- Index: lastName, dateOfBirth

**События:**
- PatientRegistered(patientId, personalInfo, timestamp)
- PatientUpdated(patientId, changes, timestamp)
- PatientDeactivated(patientId, reason, timestamp)

**Бизнес-правила:**
- При регистрации создаётся событие PatientRegistered
- Обновление персональных данных требует валидации
- Деактивация возможна только при отсутствии активных записей

---

### 2. MedicalRecord (Медицинская карта)
**Bounded Context:** Patient Management

**Aggregate Root:** MedicalRecord

**Границы агрегата:**
- MedicalRecord (корневой объект)
- Diagnosis (диагнозы)
- TreatmentHistory (история лечения)
- Notes (заметки врачей)

**Инварианты:**
- Медицинская карта должна быть привязана к пациенту
- Каждая запись должна иметь дату и врача
- Результаты исследований не хранятся в этом агрегате (для аналитики)

**Ключи:**
- Primary Key: recordId (UUID)
- Foreign Key: patientId
- Index: patientId, date, doctorId

**События:**
- MedicalRecordCreated(recordId, patientId, doctorId, timestamp)
- DiagnosisAdded(recordId, diagnosis, doctorId, timestamp)
- TreatmentUpdated(recordId, treatmentInfo, timestamp)

**Бизнес-правила:**
- Создание карты автоматически при первой записи пациента
- Добавление диагноза требует наличия активной записи
- История лечения неизменяема после завершения лечения

---

### 3. Appointment (Запись на приём)
**Bounded Context:** Patient Management

**Aggregate Root:** Appointment

**Границы агрегата:**
- Appointment (корневой объект)
- AppointmentDetails (детали записи)
- StatusHistory (история статусов)

**Инварианты:**
- Запись должна быть привязана к пациенту и врачу
- Время приёма не может быть в прошлом при создании
- Врач не может иметь две записи в одно время
- Пациент не может иметь две записи в одно время

**Ключи:**
- Primary Key: appointmentId (UUID)
- Foreign Key: patientId, doctorId, clinicId
- Unique: (doctorId, appointmentDateTime), (patientId, appointmentDateTime)
- Index: appointmentDateTime, status

**События:**
- AppointmentCreated(appointmentId, patientId, doctorId, clinicId, datetime, timestamp)
- AppointmentConfirmed(appointmentId, timestamp)
- AppointmentCompleted(appointmentId, result, timestamp)
- AppointmentCancelled(appointmentId, reason, timestamp)

**Бизнес-правила:**
- При создании записи проверяется доступность врача
- Завершение приёма автоматически создаёт событие для финансового домена
- Отмена записи возможна не позднее чем за 24 часа до приёма

---

### 4. Clinic (Клиника)
**Bounded Context:** Clinic Management

**Aggregate Root:** Clinic

**Границы агрегата:**
- Clinic (корневой объект)
- Address (адрес)
- ContactInfo (контактная информация)
- OperatingHours (часы работы)

**Инварианты:**
- Клиника должна иметь уникальное название
- Адрес должен быть валидным
- Часы работы должны быть корректными

**Ключи:**
- Primary Key: clinicId (UUID)
- Unique: name, taxId
- Index: city, region

**События:**
- ClinicCreated(clinicId, name, address, timestamp)
- ClinicUpdated(clinicId, changes, timestamp)
- ScheduleUpdated(clinicId, operatingHours, timestamp)

**Бизнес-правила:**
- Создание клиники требует валидации адреса
- Обновление расписания влияет на доступность записей

---

### 5. Staff (Персонал)
**Bounded Context:** Staff Management

**Aggregate Root:** Staff

**Границы агрегата:**
- Staff (корневой объект)
- PersonalInfo (персональная информация)
- Qualifications (квалификации)
- Roles (роли)

**Инварианты:**
- Сотрудник должен иметь уникальный идентификатор
- Должна быть указана хотя бы одна роль
- Квалификации должны быть валидными

**Ключи:**
- Primary Key: staffId (UUID)
- Unique: employeeNumber, email
- Index: role, clinicId

**События:**
- StaffRegistered(staffId, personalInfo, role, timestamp)
- StaffRoleChanged(staffId, oldRole, newRole, timestamp)
- WorkScheduleUpdated(staffId, schedule, timestamp)

**Бизнес-правила:**
- Изменение роли требует проверки прав доступа
- Обновление расписания влияет на доступность для записей

---

### 6. Equipment (Оборудование)
**Bounded Context:** Inventory Management

**Aggregate Root:** Equipment

**Границы агрегата:**
- Equipment (корневой объект)
- MaintenanceHistory (история обслуживания)
- Location (местоположение)

**Инварианты:**
- Оборудование должно иметь уникальный серийный номер
- Оборудование должно быть привязано к клинике
- Статус оборудования должен быть валидным

**Ключи:**
- Primary Key: equipmentId (UUID)
- Unique: serialNumber
- Foreign Key: clinicId
- Index: status, type

**События:**
- EquipmentRegistered(equipmentId, serialNumber, type, clinicId, timestamp)
- EquipmentMaintained(equipmentId, maintenanceInfo, timestamp)
- EquipmentLocationChanged(equipmentId, newLocation, timestamp)

**Бизнес-правила:**
- Регистрация оборудования требует валидации серийного номера
- Обслуживание обновляет статус оборудования

---

## Финансовый домен

### 7. Account (Счёт)
**Bounded Context:** Account Management

**Aggregate Root:** Account

**Границы агрегата:**
- Account (корневой объект)
- Balance (баланс)
- TransactionHistory (история транзакций - последние N записей)

**Инварианты:**
- Счёт должен быть привязан к клиенту
- Баланс не может быть отрицательным (для дебетовых счетов)
- Каждая транзакция должна изменять баланс
- Баланс = сумма всех транзакций

**Ключи:**
- Primary Key: accountId (UUID)
- Foreign Key: customerId
- Unique: accountNumber
- Index: customerId, status

**События:**
- AccountCreated(accountId, customerId, accountNumber, initialBalance, timestamp)
- AccountBalanceUpdated(accountId, oldBalance, newBalance, transactionId, timestamp)
- TransactionCompleted(accountId, transactionId, amount, type, timestamp)

**Бизнес-правила:**
- Создание счёта требует проверки клиента
- Обновление баланса атомарно с созданием транзакции
- Баланс всегда должен быть консистентным

---

### 8. CreditContract (Кредитный договор)
**Bounded Context:** Credit Management

**Aggregate Root:** CreditContract

**Границы агрегата:**
- CreditContract (корневой объект)
- CreditTerms (условия кредита)
- PaymentSchedule (график платежей)
- PaymentHistory (история платежей)

**Инварианты:**
- Кредитный договор должен быть привязан к клиенту
- Сумма кредита должна быть положительной
- Процентная ставка должна быть в допустимых пределах
- Сумма всех платежей = сумма кредита + проценты
- Нельзя изменить условия активного договора

**Ключи:**
- Primary Key: contractId (UUID)
- Foreign Key: customerId, accountId
- Unique: contractNumber
- Index: customerId, status, dueDate

**События:**
- CreditContractCreated(contractId, customerId, amount, interestRate, term, timestamp)
- CreditLimitChanged(contractId, oldLimit, newLimit, reason, timestamp)
- PaymentDue(contractId, paymentAmount, dueDate, timestamp)
- PaymentReceived(contractId, paymentId, amount, timestamp)
- CreditRepaid(contractId, totalPaid, timestamp)

**Бизнес-правила:**
- Создание договора требует проверки кредитоспособности клиента
- Изменение лимита возможно только для активных договоров
- Просрочка платежа создаёт событие для обработки
- Полное погашение закрывает договор

---

### 9. Payment (Платеж)
**Bounded Context:** Payment Processing

**Aggregate Root:** Payment

**Границы агрегата:**
- Payment (корневой объект)
- PaymentDetails (детали платежа)
- PaymentMethod (платежный метод)

**Инварианты:**
- Платеж должен быть привязан к счёту
- Сумма платежа должна быть положительной
- Платеж не может быть изменён после обработки
- Статус платежа должен следовать жизненному циклу: Initiated → Processing → Completed/Failed

**Ключи:**
- Primary Key: paymentId (UUID)
- Foreign Key: accountId, appointmentId (опционально)
- Index: status, createdAt, accountId

**События:**
- PaymentInitiated(paymentId, accountId, amount, purpose, timestamp)
- PaymentProcessed(paymentId, transactionId, timestamp)
- PaymentFailed(paymentId, reason, timestamp)

**Бизнес-правила:**
- Инициация платежа проверяет достаточность средств
- Обработка платежа атомарно обновляет баланс счёта
- Неудачный платеж возвращает средства

---

## Аналитический домен

### 10. MaterializedView (Материализованное представление)
**Bounded Context:** Streaming Marts

**Aggregate Root:** MaterializedView

**Границы агрегата:**
- MaterializedView (корневой объект)
- AggregationRules (правила агрегации)
- DataSnapshot (снимок данных)

**Инварианты:**
- Представление должно иметь уникальное имя
- Правила агрегации должны быть валидными
- Данные должны быть актуальными (в пределах TTL)

**Ключи:**
- Primary Key: viewId (UUID)
- Unique: viewName
- Index: lastUpdated, domain

**События:**
- MaterializedViewRefreshed(viewId, viewName, recordCount, timestamp)
- AggregateUpdated(viewId, metricName, value, timestamp)

**Бизнес-правила:**
- Обновление представления происходит асинхронно при получении событий
- Устаревшие данные помечаются как неактуальные

---

### 11. Report (Отчёт)
**Bounded Context:** Report Builder

**Aggregate Root:** Report

**Границы агрегата:**
- Report (корневой объект)
- QueryDefinition (определение запроса)
- Parameters (параметры отчёта)
- ExecutionHistory (история выполнения)

**Инварианты:**
- Отчёт должен иметь уникальное имя в рамках пользователя
- Запрос должен быть валидным SQL
- Параметры должны соответствовать схеме запроса

**Ключи:**
- Primary Key: reportId (UUID)
- Foreign Key: userId
- Unique: (userId, reportName)
- Index: userId, createdAt

**События:**
- ReportCreated(reportId, userId, reportName, timestamp)
- ReportExecuted(reportId, executionId, parameters, timestamp)
- QueryCompleted(reportId, executionId, rowCount, duration, timestamp)

**Бизнес-правила:**
- Создание отчёта требует проверки прав доступа к данным
- Выполнение отчёта проверяет валидность параметров
- Результаты кэшируются для повторных запросов

---

## Принципы проектирования агрегатов

### Размер агрегатов
- Агрегаты должны быть как можно меньше
- Включают только объекты, которые должны изменяться вместе
- История транзакций хранится отдельно (Event Store)

### Консистентность
- **Сильная консистентность** внутри агрегата
- **Eventual consistency** между агрегатами через события
- Транзакции ограничены одним агрегатом

### Доступ
- Доступ к агрегату только через Aggregate Root
- Внешние ссылки только на Aggregate Root (по ID)
- Нельзя напрямую изменять внутренние объекты агрегата

### События
- Все изменения агрегата публикуют события
- События неизменяемы после публикации
- События содержат минимальный набор данных для обработки
