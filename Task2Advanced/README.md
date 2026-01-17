# Terraform с удалённым состоянием и CI/CD

Проект автоматизирует развёртывание инфраструктуры в Yandex Cloud через Terraform с использованием удалённого хранения состояния в Yandex Object Storage и CI/CD pipeline на GitHub Actions.

## Структура проекта

```
Task2Advanced/
├── modules/
│   └── vm/
│       ├── main.tf          # Ресурсы ВМ, диска и подключения
│       ├── variables.tf     # Входные параметры модуля
│       └── outputs.tf       # Выходные значения модуля
├── envs/
│   ├── dev/
│   │   ├── main.tf          # Конфигурация с backend для dev
│   │   ├── variables.tf
│   │   └── terraform.tfvars.example
│   ├── stage/
│   │   ├── main.tf          # Конфигурация с backend для stage
│   │   ├── variables.tf
│   │   └── terraform.tfvars.example
│   └── prod/
│       ├── main.tf          # Конфигурация с backend для prod
│       ├── variables.tf
│       └── terraform.tfvars.example
├── .github/
│   └── workflows/
│       └── terraform.yml    # CI/CD pipeline
├── .gitignore               # Игнорируемые файлы
├── backend.hcl.example      # Пример конфигурации backend
└── README.md                # Документация
```

## Backend конфигурация

Каждое окружение использует удалённое состояние в Yandex Object Storage (S3-совместимое хранилище):

- **Bucket**: `terraform-state-bucket` (настраивается в backend конфигурации)
- **Endpoint**: `storage.yandexcloud.net`
- **Region**: `ru-central1`
- **Keys**: 
  - `dev/terraform.tfstate`
  - `stage/terraform.tfstate`
  - `prod/terraform.tfstate`

Состояние изолировано для каждого окружения, что предотвращает конфликты при параллельном развёртывании.

## Подготовка инфраструктуры

### 1. Создание бакета в Yandex Object Storage

```bash
# Установите Yandex Cloud CLI
yc storage bucket create \
  --name terraform-state-bucket \
  --default-storage-class standard \
  --max-size 5368709120 \
  --public-read off \
  --public-list off
```

### 2. Создание сервисного аккаунта для Terraform

```bash
# Создание сервисного аккаунта
yc iam service-account create --name terraform-sa

# Назначение ролей
yc resource-manager folder add-access-binding <folder-id> \
  --role editor \
  --subject serviceAccount:<service-account-id>

# Создание статического ключа
yc iam access-key create --service-account-name terraform-sa
```

### 3. Настройка секретов в GitHub

В настройках репозитория (Settings → Secrets and variables → Actions) добавьте:

**Secrets:**
- `YC_CLOUD_ID` - ID облака Yandex Cloud
- `YC_FOLDER_ID` - ID каталога Yandex Cloud
- `YC_SERVICE_ACCOUNT_KEY` - JSON ключ сервисного аккаунта
- `S3_ACCESS_KEY` - Access Key для Object Storage
- `S3_SECRET_KEY` - Secret Key для Object Storage
- `SSH_KEY` - SSH-ключ для доступа к ВМ

**Variables:**
- `VM_NAME` - Имя виртуальной машины
- `VM_CORES` - Количество ядер
- `VM_MEMORY` - Объём RAM в ГБ
- `VM_DISK_SIZE` - Размер диска в ГБ
- `SUBNET_ID` - ID подсети

## CI/CD Pipeline

Pipeline выполняется автоматически при:
- Push в ветки `main` или `develop`
- Pull Request в ветки `main` или `develop`
- Ручной запуск через `workflow_dispatch`

### Этапы pipeline

1. **Checkout code** - получение кода из репозитория
2. **Setup Terraform** - установка Terraform (версия 1.6.0)
3. **Configure Yandex Cloud credentials** - настройка аутентификации
4. **Terraform Init** - инициализация с backend конфигурацией
5. **Terraform Format Check** - проверка форматирования кода
6. **Terraform Validate** - валидация конфигурации
7. **Terraform Plan** - планирование изменений
8. **Terraform Apply** - применение изменений (только при ручном запуске с `action=apply`)

### Безопасность

- **Изоляция окружений**: каждое окружение использует отдельный state файл
- **Секреты**: все чувствительные данные хранятся в GitHub Secrets
- **Manual approval**: для production окружения рекомендуется настроить environment protection rules
- **Переменные**: конфигурация через переменные, без захардкоженных значений

### Ручной запуск

1. Перейдите в Actions → Terraform CI/CD
2. Нажмите "Run workflow"
3. Выберите окружение (dev/stage/prod)
4. Выберите действие (plan/apply)
5. Нажмите "Run workflow"

## Локальное использование

Для локального запуска необходимо настроить backend:

### Вариант 1: Использование файла backend.hcl

```bash
cd envs/dev

# Создайте backend.hcl на основе backend.hcl.example
cp ../../backend.hcl.example backend.hcl
# Отредактируйте backend.hcl, указав свои значения

# Инициализация с backend
terraform init -backend-config=backend.hcl

# Планирование
terraform plan -var-file=terraform.tfvars

# Применение
terraform apply -var-file=terraform.tfvars
```

### Вариант 2: Использование переменных окружения

```bash
cd envs/dev

# Установите переменные окружения
export AWS_ACCESS_KEY_ID="your-s3-access-key"
export AWS_SECRET_ACCESS_KEY="your-s3-secret-key"

# Инициализация с backend
terraform init \
  -backend-config="access_key=$AWS_ACCESS_KEY_ID" \
  -backend-config="secret_key=$AWS_SECRET_ACCESS_KEY"

# Планирование
terraform plan -var-file=terraform.tfvars

# Применение
terraform apply -var-file=terraform.tfvars
```

**Важно**: 
- Backend конфигурация в `main.tf` содержит пустые значения `access_key` и `secret_key` для безопасности
- Всегда используйте `-backend-config` при `terraform init` для передачи ключей
- Не коммитьте файлы `backend.hcl` или `terraform.tfvars` с реальными значениями в репозиторий

## Особенности реализации

- **Удалённое состояние**: состояние хранится в Yandex Object Storage, не в репозитории
- **Изоляция окружений**: отдельные state файлы для dev/stage/prod
- **Автоматизация**: полный цикл CI/CD через GitHub Actions
- **Безопасность**: секреты и переменные через GitHub Secrets/Variables
- **Валидация**: автоматическая проверка форматирования и валидация перед применением
- **Manual approval**: возможность ручного запуска с выбором окружения и действия

## Требования

- Terraform >= 0.13
- Yandex Cloud CLI (для настройки)
- GitHub Actions (для CI/CD)
- Yandex Object Storage bucket для хранения состояния
- Сервисный аккаунт Yandex Cloud с правами editor
