flowchart LR
   %% источники
    Git["Git-репозитории<br>(gitlab или bitbucket)"] --"•исходный код app<br>•helm-чарты<br>•пайплайны<br>•ansible"-->  CI["CI/CD система:<br>• gitlab<br>• teamcity<br>• jenkins"]
    CI -->|"сборка образов"| Harbor
    CI --"cекреты<br>(dev/test/prod)"--> Vault
    Harbor --"docker-образы<br>(теги: dev test + app)"--> K8S-CI
    Harbor --"docker-образы<br>(теги: prod + app)"--> K8S-CD

    %% интеграции
    Vault --"инжектинг секретов"--> K8S-CI
    Vault --"инжектинг секретов"--> K8S-CD
    SonarQube -.->|"анализ кода"| CI
    Nexus -.->|"артефакты/пакеты/либы"| CI

    %% Security: Trivy
    CI -->|"сканирование образов"| Trivy
    Trivy -.->|"отчеты об уязвимостях"| CI
    Trivy -.->|"блокировка небезопасных образов"| Harbor

    %% gitops
    Git -->|"helm"| ArgoCD-CI
    Git -->|"helm"| ArgoCD-CD
    ArgoCD-CI --"Деплой в DEV"--> K8S-CI
    ArgoCD-CI --"Деплой в TEST"--> K8S-CI
    ArgoCD-CD --"Деплой в PROD"--> K8S-CD

    %% обсервабилити
    loki -->|"логи"| K8S-CI
    loki -->|"логи"| K8S-CD
    prometheus -->|"метрики"| K8S-CI
    prometheus -->|"метрики"| K8S-CD
    grafana -->|"метрики"| prometheus
    grafana -->|"логи"| loki

    %% Kubernetes CI
    K8S-CI["CI kubernetes cluster"
    namespaces:
    - <>-dev
    - <>-test
    ]

    %% Kubernetes prod
    K8S-CD["CD kubernetes cluster"
    namespaces:
    - <>-prod
    ]

    %% Стили
    style Git fill:#fc6d26,color:white
    style CI fill:#ff9900,color:black
    style ArgoCD-CI fill:#ef7b4d,color:white
    style ArgoCD-CD fill:#ef7b4d,color:white
    style Vault fill:#ffec6e
    style Harbor fill:#60b932
    style K8S-CI fill:#326ce5,color:white
    style K8S-CD fill:#326ce5,color:white
    style SonarQube fill:#4e9ad4,color:white
    style loki fill:#fff
    style grafana fill:#fff
    style prometheus fill:#fff

## описание каждого компонента и его предназначения в представленной диаграмме:  

| **Компонент**        | **Участие в процессе**                                                                 | **Предназначение**                                                                 |
|----------------------|---------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| **Git-репозитории**  | Хранит:<br>• Исходный код приложения<br>• Helm-чарты<br>• CI/CD пайплайны<br>• Ansible-скрипты | Централизованное хранение и управление версиями кода и конфигураций инфраструктуры. |
| **CI/CD система**<br>(GitLab, TeamCity, Jenkins) | Получает код из Git, запускает сборку, тестирование и деплой. Интегрируется с SonarQube, Nexus, Harbor. | Автоматизация процессов сборки, тестирования и развертывания приложений. |
| **Harbor**           | Хранит Docker-образы с тегами (dev, test, prod). Передает образы в K8S-CI и K8S-CD. | Реестр контейнеров для хранения и управления Docker-образами. |
| **Vault**            | Хранит секреты (ключи, пароли, токены) и инжектит их в K8S (CI/CD). | Безопасное хранение и управление секретами. |
| **SonarQube**        | Анализирует код на качество, уязвимости и покрытие тестами. | Контроль качества кода и выявление потенциальных проблем. |
| **Nexus**            | Хранит артефакты (библиотеки, пакеты), используемые в процессе сборки. | Репозиторий бинарных артефактов для зависимостей. |
| **ArgoCD-CI**        | Развертывает приложения в **DEV** и **TEST** окружениях K8S-CI на основе Helm-чартов из Git. | GitOps-инструмент для автоматического деплоя в CI-кластер. |
| **ArgoCD-CD**        | Развертывает приложения в **PROD** окружении K8S-CD на основе Helm-чартов из Git. | GitOps-инструмент для автоматического деплоя в продакшен. |
| **K8S-CI**<br>(CI Kubernetes cluster) | Запускает приложения в неймспейсах `-dev` и `-test`. Получает образы из Harbor и секреты из Vault. | Тестовый кластер для разработки и проверки изменений перед продакшеном. |
| **K8S-CD**<br>(CD Kubernetes cluster) | Запускает приложения в неймспейсе `-prod`. Получает образы из Harbor и секреты из Vault. | Продакшен-кластер для стабильной работы приложений. |
| **Loki**             | Собирает логи из K8S-CI и K8S-CD. | Централизованное хранение и анализ логов. |
| **Prometheus**       | Собирает метрики из K8S-CI и K8S-CD. | Мониторинг состояния кластеров и приложений. |
| **Grafana**          | Визуализирует метрики из Prometheus и логи из Loki. | Дашборды для анализа производительности и диагностики проблем. |
| **Trivy**            | Сканирует Docker-образы из Harbor на уязвимости | Инструмент для статического анализа уязвимостей в образах. |

### дополнительные пояснения:
1. **Поток данных**:
   - **CI-фаза**:  
     `Git → CI → (Harbor, Vault, SonarQube, Nexus) → ArgoCD-CI → K8S-CI`  
   - **CD-фаза**:  
     `Git → ArgoCD-CD → K8S-CD` (использует образы из Harbor и секреты из Vault).  
2. **Наблюдаемость**:  
   - **Loki + Prometheus** собирают данные из кластеров.  
   - **Grafana** предоставляет единый интерфейс для анализа.  
3. **Безопасность**:  
   - **Vault** исключает хранение секретов в коде/конфигах.  
   - **Harbor** обеспечивает контроль доступа к образам.  
