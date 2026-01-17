# Каталог доменных событий системы "Будущее 2.0"

## Формат описания события

Для каждого события указаны:
- **Название** - уникальное имя события
- **Контекст-источник** - Bounded Context, который публикует событие
- **Семантика** - бизнес-смысл события
- **Минимальный контракт** - обязательные поля события

## Медицинский домен

### Patient Management

#### PatientRegistered
**Контекст-источник:** Patient Management  
**Семантика:** Зарегистрирован новый пациент в системе  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "PatientRegistered",
  "timestamp": "ISO 8601",
  "patientId": "uuid",
  "firstName": "string",
  "lastName": "string",
  "email": "string",
  "phoneNumber": "string",
  "dateOfBirth": "ISO 8601"
}
```
**Подписчики:**
- Analytics Domain (Streaming Marts) - обновление метрик по пациентам
- Financial Domain (Account Management) - создание счёта для нового клиента

---

#### PatientUpdated
**Контекст-источник:** Patient Management  
**Семантика:** Обновлена информация о пациенте  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "PatientUpdated",
  "timestamp": "ISO 8601",
  "patientId": "uuid",
  "changedFields": ["field1", "field2"],
  "newValues": {}
}
```
**Подписчики:**
- Analytics Domain - обновление данных в витринах

---

#### AppointmentCreated
**Контекст-источник:** Patient Management  
**Семантика:** Создана новая запись на приём к врачу  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "AppointmentCreated",
  "timestamp": "ISO 8601",
  "appointmentId": "uuid",
  "patientId": "uuid",
  "doctorId": "uuid",
  "clinicId": "uuid",
  "appointmentDateTime": "ISO 8601",
  "serviceType": "string"
}
```
**Подписчики:**
- Analytics Domain - обновление статистики записей
- Financial Domain - подготовка к созданию платежа

---

#### AppointmentCompleted
**Контекст-источник:** Patient Management  
**Семантика:** Завершён приём пациента  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "AppointmentCompleted",
  "timestamp": "ISO 8601",
  "appointmentId": "uuid",
  "patientId": "uuid",
  "doctorId": "uuid",
  "clinicId": "uuid",
  "serviceCost": "decimal",
  "diagnosisCode": "string"
}
```
**Подписчики:**
- Financial Domain (Payment Processing) - инициация платежа за услугу
- Analytics Domain - обновление метрик завершённых приёмов
- AI Services - запрос на анализ при необходимости

---

#### AppointmentCancelled
**Контекст-источник:** Patient Management  
**Семантика:** Отменена запись на приём  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "AppointmentCancelled",
  "timestamp": "ISO 8601",
  "appointmentId": "uuid",
  "patientId": "uuid",
  "reason": "string"
}
```
**Подписчики:**
- Analytics Domain - обновление статистики отмен

---

### Clinic Management

#### ClinicCreated
**Контекст-источник:** Clinic Management  
**Семантика:** Создана новая клиника  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "ClinicCreated",
  "timestamp": "ISO 8601",
  "clinicId": "uuid",
  "name": "string",
  "address": {
    "city": "string",
    "street": "string"
  },
  "taxId": "string"
}
```
**Подписчики:**
- Analytics Domain - добавление клиники в аналитику

---

#### ScheduleUpdated
**Контекст-источник:** Clinic Management  
**Семантика:** Обновлено расписание работы клиники  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "ScheduleUpdated",
  "timestamp": "ISO 8601",
  "clinicId": "uuid",
  "operatingHours": {
    "dayOfWeek": "integer",
    "startTime": "time",
    "endTime": "time"
  }
}
```
**Подписчики:**
- Patient Management - обновление доступности записей

---

### Staff Management

#### StaffRegistered
**Контекст-источник:** Staff Management  
**Семантика:** Зарегистрирован новый сотрудник  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "StaffRegistered",
  "timestamp": "ISO 8601",
  "staffId": "uuid",
  "firstName": "string",
  "lastName": "string",
  "role": "string",
  "clinicId": "uuid",
  "employeeNumber": "string"
}
```
**Подписчики:**
- Analytics Domain - обновление метрик по персоналу

---

#### StaffRoleChanged
**Контекст-источник:** Staff Management  
**Семантика:** Изменена роль сотрудника  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "StaffRoleChanged",
  "timestamp": "ISO 8601",
  "staffId": "uuid",
  "oldRole": "string",
  "newRole": "string"
}
```
**Подписчики:**
- IAM - обновление прав доступа

---

### Inventory Management

#### EquipmentRegistered
**Контекст-источник:** Inventory Management  
**Семантика:** Зарегистрировано новое оборудование  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "EquipmentRegistered",
  "timestamp": "ISO 8601",
  "equipmentId": "uuid",
  "serialNumber": "string",
  "type": "string",
  "clinicId": "uuid"
}
```
**Подписчики:**
- Analytics Domain - обновление инвентарной статистики

---

### AI Services

#### AIAnalysisCompleted
**Контекст-источник:** AI Services  
**Семантика:** Завершён анализ медицинских данных с помощью ИИ  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "AIAnalysisCompleted",
  "timestamp": "ISO 8601",
  "analysisId": "uuid",
  "patientId": "uuid",
  "researchId": "uuid",
  "analysisType": "string",
  "confidence": "decimal",
  "recommendations": ["string"]
}
```
**Подписчики:**
- Patient Management - добавление рекомендаций в медицинскую карту
- Analytics Domain - обновление метрик по использованию ИИ

---

#### ResearchProcessed
**Контекст-источник:** AI Services  
**Семантика:** Обработано медицинское исследование  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "ResearchProcessed",
  "timestamp": "ISO 8601",
  "researchId": "uuid",
  "patientId": "uuid",
  "researchType": "string",
  "status": "string"
}
```
**Подписчики:**
- Patient Management - обновление статуса исследования

---

## Финансовый домен

### Account Management

#### AccountCreated
**Контекст-источник:** Account Management  
**Семантика:** Создан новый банковский счёт  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "AccountCreated",
  "timestamp": "ISO 8601",
  "accountId": "uuid",
  "customerId": "uuid",
  "accountNumber": "string",
  "accountType": "string",
  "initialBalance": "decimal"
}
```
**Подписчики:**
- Analytics Domain - обновление метрик по счетам

---

#### AccountBalanceUpdated
**Контекст-источник:** Account Management  
**Семантика:** Изменён баланс счёта  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "AccountBalanceUpdated",
  "timestamp": "ISO 8601",
  "accountId": "uuid",
  "oldBalance": "decimal",
  "newBalance": "decimal",
  "transactionId": "uuid",
  "changeReason": "string"
}
```
**Подписчики:**
- Analytics Domain - обновление финансовых метрик в реальном времени
- Credit Management - проверка кредитного лимита

---

#### TransactionCompleted
**Контекст-источник:** Account Management  
**Семантика:** Завершена транзакция по счёту  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "TransactionCompleted",
  "timestamp": "ISO 8601",
  "transactionId": "uuid",
  "accountId": "uuid",
  "amount": "decimal",
  "transactionType": "string",
  "description": "string"
}
```
**Подписчики:**
- Analytics Domain - обновление транзакционной статистики
- Financial Reporting - расчёт финансовых показателей

---

### Credit Management

#### CreditContractCreated
**Контекст-источник:** Credit Management  
**Семантика:** Создан новый кредитный договор  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "CreditContractCreated",
  "timestamp": "ISO 8601",
  "contractId": "uuid",
  "customerId": "uuid",
  "accountId": "uuid",
  "contractNumber": "string",
  "creditAmount": "decimal",
  "interestRate": "decimal",
  "termMonths": "integer",
  "monthlyPayment": "decimal"
}
```
**Подписчики:**
- Analytics Domain - обновление кредитной статистики
- Financial Reporting - включение в финансовую отчётность
- Account Management - резервирование средств

---

#### CreditLimitChanged
**Контекст-источник:** Credit Management  
**Семантика:** Изменён кредитный лимит  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "CreditLimitChanged",
  "timestamp": "ISO 8601",
  "contractId": "uuid",
  "oldLimit": "decimal",
  "newLimit": "decimal",
  "reason": "string"
}
```
**Подписчики:**
- Analytics Domain - обновление метрик

---

#### PaymentDue
**Контекст-источник:** Credit Management  
**Семантика:** Наступил срок платежа по кредиту  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "PaymentDue",
  "timestamp": "ISO 8601",
  "contractId": "uuid",
  "customerId": "uuid",
  "paymentAmount": "decimal",
  "dueDate": "ISO 8601",
  "paymentNumber": "integer"
}
```
**Подписчики:**
- Payment Processing - автоматическое списание платежа
- Analytics Domain - обновление метрик просрочек

---

#### CreditRepaid
**Контекст-источник:** Credit Management  
**Семантика:** Полностью погашен кредитный договор  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "CreditRepaid",
  "timestamp": "ISO 8601",
  "contractId": "uuid",
  "customerId": "uuid",
  "totalPaid": "decimal",
  "repaymentDate": "ISO 8601"
}
```
**Подписчики:**
- Analytics Domain - обновление статистики погашений
- Financial Reporting - закрытие договора в отчётности

---

### Payment Processing

#### PaymentInitiated
**Контекст-источник:** Payment Processing  
**Семантика:** Инициирован платёж  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "PaymentInitiated",
  "timestamp": "ISO 8601",
  "paymentId": "uuid",
  "accountId": "uuid",
  "amount": "decimal",
  "purpose": "string",
  "appointmentId": "uuid (optional)"
}
```
**Подписчики:**
- Account Management - резервирование средств
- Analytics Domain - обновление метрик платежей

---

#### PaymentProcessed
**Контекст-источник:** Payment Processing  
**Семантика:** Успешно обработан платёж  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "PaymentProcessed",
  "timestamp": "ISO 8601",
  "paymentId": "uuid",
  "accountId": "uuid",
  "transactionId": "uuid",
  "amount": "decimal",
  "processingTime": "integer (ms)"
}
```
**Подписчики:**
- Account Management - обновление баланса счёта
- Analytics Domain - обновление статистики успешных платежей
- Financial Reporting - включение в отчётность

---

#### PaymentFailed
**Контекст-источник:** Payment Processing  
**Семантика:** Не удалось обработать платёж  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "PaymentFailed",
  "timestamp": "ISO 8601",
  "paymentId": "uuid",
  "accountId": "uuid",
  "amount": "decimal",
  "failureReason": "string",
  "errorCode": "string"
}
```
**Подписчики:**
- Analytics Domain - обновление метрик неудачных платежей
- DLQ Handler - обработка ошибок

---

### Financial Reporting

#### FinancialReportGenerated
**Контекст-источник:** Financial Reporting  
**Семантика:** Сгенерирован финансовый отчёт  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "FinancialReportGenerated",
  "timestamp": "ISO 8601",
  "reportId": "uuid",
  "reportType": "string",
  "period": {
    "startDate": "ISO 8601",
    "endDate": "ISO 8601"
  },
  "totalRevenue": "decimal",
  "totalExpenses": "decimal"
}
```
**Подписчики:**
- Analytics Domain - сохранение отчёта в витрине данных

---

## Аналитический домен

### Streaming Marts

#### AggregateUpdated
**Контекст-источник:** Streaming Marts  
**Семантика:** Обновлена агрегированная метрика  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "AggregateUpdated",
  "timestamp": "ISO 8601",
  "viewId": "uuid",
  "metricName": "string",
  "value": "decimal",
  "dimensions": {}
}
```
**Подписчики:**
- Data Mart - обновление кэша
- Report Builder - уведомление об изменении данных

---

#### MaterializedViewRefreshed
**Контекст-источник:** Streaming Marts  
**Семантика:** Обновлено материализованное представление  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "MaterializedViewRefreshed",
  "timestamp": "ISO 8601",
  "viewId": "uuid",
  "viewName": "string",
  "recordCount": "integer",
  "lastUpdated": "ISO 8601"
}
```
**Подписчики:**
- Data Mart - инвалидация кэша

---

### Report Builder

#### ReportCreated
**Контекст-источник:** Report Builder  
**Семантика:** Создан новый отчёт  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "ReportCreated",
  "timestamp": "ISO 8601",
  "reportId": "uuid",
  "userId": "uuid",
  "reportName": "string",
  "reportType": "string"
}
```
**Подписчики:**
- Analytics Domain - обновление метрик использования отчётов

---

#### QueryCompleted
**Контекст-источник:** Report Builder  
**Семантика:** Завершено выполнение аналитического запроса  
**Минимальный контракт:**
```json
{
  "eventId": "uuid",
  "eventType": "QueryCompleted",
  "timestamp": "ISO 8601",
  "reportId": "uuid",
  "executionId": "uuid",
  "rowCount": "integer",
  "duration": "integer (ms)",
  "dataSource": "string"
}
```
**Подписчики:**
- Analytics Domain - обновление метрик производительности запросов

---

## Принципы проектирования событий

### Именование
- Используется формат: `[Aggregate][Action]` (например, `PatientRegistered`)
- События в прошедшем времени (что произошло)
- Ясные и понятные названия

### Неизменяемость
- События неизменяемы после публикации
- Версионирование через Schema Registry
- Обратная совместимость при изменении схем

### Минимальный контракт
- Содержит только необходимые данные для обработки
- Не включает полные объекты агрегатов
- Идентификаторы для получения дополнительных данных

### Семантика
- События отражают бизнес-факты
- Не содержат команд или инструкций
- Описывают что произошло, а не что нужно сделать
