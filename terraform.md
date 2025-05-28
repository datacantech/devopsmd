## Структура каталога для Terraform-проекта

#### 1. Базовая структура (для небольших проектов)
```
terraform-project/
├── main.tf          # Основная конфигурация инфраструктуры
├── variables.tf     # Определение переменных
├── outputs.tf       # Выходные значения (outputs)
├── providers.tf     # Настройка провайдеров
└── terraform.tfvars # Значения переменных (опционально)
```

---

#### 2. Модульная структура (для средних проектов)
```
terraform-project/
├── modules/         # Reusable модули
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── ec2/
│   │   ├── main.tf
│   │   └── ...
│   └── rds/
│       ├── main.tf
│       └── ...
│
├── environments/    # Окружения (dev/stage/prod)
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── stage/
│   │   └── ...
│   └── prod/
│       └── ...
│
├── main.tf          # Корневой модуль (если не используется environments)
└── ...
```

---

#### 3. Структура с Terragrunt (для сложных проектов)
```
terraform-project/
├── live/            # Реальные окружения
│   ├── dev/
│   │   ├── vpc/
│   │   │   └── terragrunt.hcl
│   │   ├── ec2/
│   │   │   └── terragrunt.hcl
│   │   └── rds/
│   │       └── terragrunt.hcl
│   ├── stage/
│   │   └── ...
│   └── prod/
│       └── ...
│
├── modules/         # Общие модули
│   ├── vpc/
│   ├── ec2/
│   └── rds/
│
└── terragrunt.hcl   # Общий конфиг (опционально)
```

---

### Подробное описание файлов:

#### Основные файлы:
- **`main.tf`** – главная конфигурация ресурсов.
- **`variables.tf`** – входные переменные (`variable "name" {}`).
- **`outputs.tf`** – выходные значения (`output "name" {}`).
- **`providers.tf`** – настройка провайдеров (`provider "aws" {}`).
- **`terraform.tfvars`** / **`*.auto.tfvars`** – значения переменных (можно разделять по средам).

#### Дополнительные файлы:
- **`backend.tf`** – конфигурация remote state (S3, GCS).
- **`versions.tf`** – ограничения версий Terraform и провайдеров.
- **`locals.tf`** – локальные переменные (`locals {}`).

---

### Пример для multi-environment (без Terragrunt):
```
infra/
├── modules/
│   ├── network/
│   └── compute/
│
├── dev/
│   ├── main.tf          # module "network" { source = "../../modules/network" }
│   ├── variables.tf
│   └── dev.tfvars
│
└── prod/
    ├── main.tf
    ├── variables.tf
    └── prod.tfvars
```

### принципы:
1. **Изоляция окружений** – `dev`/`stage`/`prod` должны быть отдельно.
2. **Переиспользование кода** – выносите общую логику в `modules/`.
3. **Масштабируемость** – Terragrunt помогает управлять сложными проектами.

Выбор структуры зависит от масштаба проекта. Для начала подойдет базовая, а по мере роста можно перейти на модули или Terragrunt.

Пояснение:

    Terraform Config (*.tf) – основной код инфраструктуры (ресурсы, переменные, выходы).

    Terraform State (terraform.tfstate) – JSON-файл с текущим состоянием инфраструктуры.

    Terragrunt (terragrunt.hcl) – надстройка над Terraform для управления:

        Удаленным состоянием (хранение state-файла в S3/GCS с блокировкой).

        Повторным использованием модулей.

        Конфигурацией нескольких окружений (dev/stage/prod).

Terragrunt помогает избежать дублирования кода и упрощает управление сложной инфраструктурой.