# Модуль Terraform для виртуальных машин Yandex Cloud

Универсальный переиспользуемый модуль Terraform для создания виртуальных машин в Yandex Cloud с поддержкой различных окружений (dev, stage, prod).

## Структура проекта

```
Task1Advanced/
├── modules/
│   └── vm/
│       ├── main.tf          # Ресурсы ВМ, диска и подключения
│       ├── variables.tf     # Входные параметры модуля
│       └── outputs.tf       # Выходные значения модуля
└── envs/
    ├── dev/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── terraform.tfvars.example
    ├── stage/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── terraform.tfvars.example
    └── prod/
        ├── main.tf
        ├── variables.tf
        └── terraform.tfvars.example
```

## Описание модуля

Модуль `vm` создаёт виртуальную машину в Yandex Cloud со следующими возможностями:
- Настройка количества ядер процессора и объёма RAM
- Создание и подключение дополнительного диска
- Настройка сетевого интерфейса с NAT
- Настройка SSH-доступа через публичный ключ
- Гибкая настройка через переменные без захардкоженных значений

## Параметры модуля

### Обязательные параметры

| Параметр | Тип | Описание |
|----------|-----|----------|
| `cores` | number | Количество ядер процессора |
| `memory` | number | Объём RAM в ГБ |
| `disk_size` | number | Размер подключаемого диска в ГБ |
| `subnet_id` | string | ID подсети |
| `ssh_key` | string | SSH-ключ для доступа к ВМ (sensitive) |
| `vm_name` | string | Имя виртуальной машины |

### Опциональные параметры

| Параметр | Тип | Описание | По умолчанию |
|----------|-----|----------|--------------|
| `zone` | string | Зона доступности | `ru-central1-a` |
| `image_id` | string | ID образа для ВМ | `fd8kdq6d0p8sij7h5qe3` (Ubuntu 22.04) |
| `platform_id` | string | Платформа | `standard-v1` |
| `disk_type` | string | Тип диска | `network-ssd` |
| `labels` | map(string) | Метки для ресурсов | `{}` |

## Выходные значения (Outputs)

Модуль возвращает следующие значения:

| Output | Описание |
|--------|----------|
| `vm_id` | ID виртуальной машины |
| `vm_name` | Имя виртуальной машины |
| `vm_external_ip` | Внешний IP-адрес ВМ |
| `vm_internal_ip` | Внутренний IP-адрес ВМ |
| `disk_id` | ID подключаемого диска |
| `disk_name` | Имя подключаемого диска |
| `disk_size` | Размер подключаемого диска в ГБ |
| `zone` | Зона доступности |

## Использование

### Подготовка окружения

1. Установите Terraform (версия >= 0.13)
2. Настройте провайдер Yandex Cloud:
   ```bash
   terraform init
   ```

### Запуск для конкретного окружения

#### Dev окружение

```bash
cd envs/dev
cp terraform.tfvars.example terraform.tfvars
# Отредактируйте terraform.tfvars, указав свои значения
terraform init
terraform plan -var-file=terraform.tfvars
terraform apply -var-file=terraform.tfvars
```

#### Stage окружение

```bash
cd envs/stage
cp terraform.tfvars.example terraform.tfvars
# Отредактируйте terraform.tfvars, указав свои значения
terraform init
terraform plan -var-file=terraform.tfvars
terraform apply -var-file=terraform.tfvars
```

#### Prod окружение

```bash
cd envs/prod
cp terraform.tfvars.example terraform.tfvars
# Отредактируйте terraform.tfvars, указав свои значения
terraform init
terraform plan -var-file=terraform.tfvars
terraform apply -var-file=terraform.tfvars
```

### Пример конфигурации terraform.tfvars

```hcl
cloud_id  = "b1gxxxxxxxxxxxxx"
folder_id = "b1gxxxxxxxxxxxxx"
vm_name   = "dev-vm"
cores     = 2
memory    = 4
disk_size = 20
subnet_id = "e9bxxxxxxxxxxxxx"
ssh_key   = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC..."
zone      = "ru-central1-a"

labels = {
  environment = "dev"
  project     = "future-2-0"
}
```

## Особенности реализации

- **Переиспользуемость**: Модуль не содержит захардкоженных значений окружений, все параметры передаются через переменные
- **Гибкость**: Поддержка различных конфигураций для разных окружений через `.tfvars` файлы
- **Безопасность**: SSH-ключ помечен как sensitive для защиты конфиденциальных данных
- **Масштабируемость**: Легко добавить новые окружения или изменить существующие конфигурации

## Требования

- Terraform >= 0.13
- Провайдер Yandex Cloud
- Настроенные облако и каталог в Yandex Cloud
- Существующая подсеть (subnet_id)

