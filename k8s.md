# работа кластера (см. k8s)
```mermaid
flowchart TD
    %% Ноды кластера
    subgraph Kubernetes Cluster
        subgraph Control Plane
            API["kube-apiserver (API)"]
            etcd["etcd (хранилище)"]
            CM["kube-controller-manager (Controller Manager)"]
            Scheduler["kube-scheduler (Scheduler)"]
            CCM["cloud-controller-manager (опционально)"]
        end

        subgraph Worker Nodes
            Kubelet1["kubelet (агент)"]
            KubeProxy1["kube-proxy (сеть)"]
            Pod1[("Pod (контейнеры)")]
            Pod2[("Pod (контейнеры)")]
        end
    end

    %% Взаимодействие
    API <-->|"сохранение состояния"| etcd
    API -->|"управление нодами/подами"| Kubelet1
    API -->|"настройка сети"| KubeProxy1

    CM -->|"проверка состояния (ReplicaSet, Deployments)"| API
    Scheduler -->|"выбор ноды для пода"| API
    CCM -.->|"интеграция с облаком (AWS/GCP)"| API

    Kubelet1 -->|"отчет о состоянии"| API
    KubeProxy1 -->|"обновление правил шгзщiptables/IPVS"| API

    %% Стилизация
    style Control Plane fill:#326ce5,color:white,stroke:#333
    style Worker Nodes fill:#60b932,color:white,stroke:#333
    style API fill:#ff9900,color:black
    style etcd fill:#8a2be2,color:white
    style CM fill:#ef7b4d,color:white
    style Scheduler fill:#ef7b4d,color:white
    style CCM fill:#ef7b4d,color:white
```

### описание компонент и их взаимодействия

| **Компонент**               | **Роль**                                                                 | **Взаимодействие**                                                                 |
|-----------------------------|--------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| **kube-apiserver (API)**    | Главный интерфейс управления кластером.                                  | 1. Принимает запросы от `kubectl`, Dashboard, CI/CD.<br>2. Хранит состояние в `etcd`.<br>3. Отправляет команды `kubelet` и `kube-proxy`. |
| **etcd**                    | Распределенное key-value хранилище состояния кластера.                  | Только API сервер напрямую пишет/читает из etcd. Остальные компоненты работают через API. |
| **kube-controller-manager** | Отслеживает состояние объектов (поды, ноды, сервисы) через API.         | Запускает корректирующие действия (например, рестарт пода при падении). |
| **kube-scheduler**          | Выбирает ноду для запуска пода на основе ресурсов и политик.            | Читает незапланированные поды через API и назначает им ноды. |
| **cloud-controller-manager**| Интегрирует кластер с облачными провайдерами (AWS/GCP/Azure).           | Управляет облачными LB, дисками, нодами через API облака. |
| **kubelet**                 | Агент на нодах, управляет жизненным циклом подов.                       | Получает команды от API, отчитывается о состоянии ноды. |
| **kube-proxy**              | Настраивает сетевые правила (iptables/IPVS) для сервисов.               | Обновляет правила на основе изменений в API (сервисы, эндпоинты). |

### Ключевые взаимодействия:
1. **API ↔ etcd**:  
   - Все изменения (создание пода, обновление деплоймента) сохраняются в etcd.  
   - API сервер — единственный компонент, имеющий доступ к etcd.

2. **API ↔ kubelet**:  
   - `kubelet` запрашивает у API манифесты подов для своей ноды.  
   - Отправляет обратно статус (например, `Running`, `CrashLoopBackOff`).

3. **Controller Manager ↔ API**:  
   - Проверяет, совпадает ли текущее состояние (например, количество реплик) с желаемым.  
   - Если нет — создает/удаляет поды через API.

4. **Scheduler ↔ API**:  
   - Находит поды с `pending` статусом и выбирает для них ноду по критериям:  
     - Достаточно ли CPU/RAM?  
     - Есть ли tolerations к taints?  

5. **kube-proxy ↔ API**:  
   - Следит за изменениями сервисов и эндпоинтов.  
   - Обновляет iptables/IPVS правила для маршрутизации трафика.

### пример последовательности создания пода (деплоя)
```mermaid
sequenceDiagram
    participant User as kubectl
    participant API as kube-apiserver
    participant etcd as etcd
    participant Scheduler as kube-scheduler
    participant CM as kube-controller-manager
    participant Kubelet as kubelet

    User->>API: apply deployment.yaml
    API->>etcd: Сохранить состояние
    CM->>API: Проверить ReplicaSet
    API->>etcd: Получить данные
    CM->>API: Создать недостающие поды
    Scheduler->>API: Назначить ноду для пода
    API->>Kubelet: Запустить под
    Kubelet->>API: Под готов (Running)
```

### примечания:
- **Безопасность**: Все коммуникации между компонентами используют TLS (кроме etcd → API, если etcd развернут локально).  
- **Масштабируемость**: API сервер можно масштабировать горизонтально, etcd — только odd-количество нод (3, 5, 7).  
- **Расширения**: Доступны дополнительные контроллеры (например, для CRD) и scheduler’ы (например, на основе машинного обучения).