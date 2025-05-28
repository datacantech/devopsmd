# **Helm: Package Manager для Kubernetes**

Helm — это **менеджер пакетов (чартов)** для Kubernetes, который упрощает деплой приложений, позволяя:
- Шаблонизировать манифесты.
- Управлять версиями.
- Настраивать параметры через `values.yaml`.

---

## **🔹 Основные понятия**
| Термин          | Описание                                                                 |
|-----------------|-------------------------------------------------------------------------|
| **Chart**       | Пакет с файлами для деплоя (аналог `deb`/`rpm` для Kubernetes).         |
| **Release**     | Экземпляр развернутого чарта в кластере (можно обновлять/откатывать).   |
| **Values**      | Переменные для кастомизации чарта (`values.yaml`).                      |
| **Repository**  | Хранилище чартов (как репозиторий Docker).                             |

---

## **🔹 Структура чарта**
```bash
my-chart/
├── Chart.yaml          # Метаданные (имя, версия)
├── values.yaml         # Дефолтные параметры
├── charts/             # Зависимости (subcharts)
└── templates/          # Шаблоны манифестов
    ├── deployment.yaml
    ├── service.yaml
    └── ingress.yaml
```

---

## **🔹 Пример 1: Простой чарт для Nginx**
### **1. Создаем чарт**
```bash
helm create nginx-chart
cd nginx-chart
```

### **2. Редактируем `values.yaml`**
```yaml
replicaCount: 2
image:
  repository: nginx
  tag: "1.25"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
ingress:
  enabled: false
```

### **3. Шаблон `templates/deployment.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-nginx
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: 80
```

### **4. Установка в кластер**
```bash
helm install nginx-release ./nginx-chart
```

---

## **🔹 Пример 2: Чарт с Ingress и ConfigMap**
### **1. Добавляем Ingress (`templates/ingress.yaml`)**
```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
spec:
  rules:
  - host: {{ .Values.ingress.host }}
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: {{ .Release.Name }}-svc
            port:
              number: {{ .Values.service.port }}
{{- end }}
```

### **2. Добавляем ConfigMap (`templates/configmap.yaml`)**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
  nginx.conf: |
    server {
      listen 80;
      server_name {{ .Values.ingress.host }};
      location / {
        root /usr/share/nginx/html;
      }
    }
```

### **3. Обновляем `values.yaml`**
```yaml
ingress:
  enabled: true
  host: "myapp.example.com"
```

### **4. Установка**
```bash
helm upgrade --install nginx-release ./nginx-chart
```

---

## **🔹 Пример 3: Чарт с зависимостями (PostgreSQL + Redis)**
### **1. Добавляем зависимости в `Chart.yaml`**
```yaml
dependencies:
  - name: postgresql
    version: "12.2.0"
    repository: "https://charts.bitnami.com/bitnami"
  - name: redis
    version: "16.8.0"
    repository: "https://charts.bitnami.com/bitnami"
```

### **2. Скачиваем зависимости**
```bash
helm dependency update
```

### **3. Переопределяем их `values`**
```yaml
postgresql:
  auth:
    postgresPassword: "secret"
redis:
  architecture: standalone
```

### **4. Установка**
```bash
helm install my-app .
```

---

## **🔹 Продвинутые техники**
### **1. Условная генерация (`if` в шаблонах)**
```yaml
{{- if eq .Values.environment "prod" }}
apiVersion: v1
kind: ResourceQuota
metadata:
  name: prod-quota
spec:
  hard:
    cpu: "10"
{{- end }}
```

### **2. Хелперы (`_helpers.tpl`)**
```tpl
{{- define "mychart.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name }}
{{- end }}
```
Использование:
```yaml
name: {{ include "mychart.fullname" . }}
```

### **3. Hooks (пред-/пост-установка)**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migrate
  annotations:
    "helm.sh/hook": post-install
spec:
  template:
    spec:
      containers:
      - name: migrate
        image: alpine
        command: ["/bin/sh", "-c", "echo 'DB migrated!'"]
```

---

## **🔹 Полезные команды**
| Команда | Описание |
|---------|----------|
| `helm install my-release ./chart` | Установка чарта. |
| `helm upgrade my-release ./chart` | Обновление. |
| `helm rollback my-release 1` | Откат к версии 1. |
| `helm list` | Список релизов. |
| `helm template ./chart` | Проверка шаблонов без установки. |

Где брать чарты? - Artifact Hub (официальный репозиторий)[https://artifacthub.io/]