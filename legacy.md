### **Процесс сборки и деплоя приложения (Java/Python + БД) на виртуальной машине (без Kubernetes и Docker Compose)**  

Рассмотрим развёртывание типичного веб-приложения с:  
- **Бэкендом** (Java/Spring Boot или Python/Django/FastAPI)  
- **Фронтендом** (React/Vue/Angular или статические файлы)  
- **Базой данных** (PostgreSQL/MySQL/MongoDB)  

---

## **1. Подготовка виртуальной машины (VM)**
### **Требования к серверу**
- **ОС**: Ubuntu/Debian/CentOS (рекомендуется LTS-версия).  
- **Порты**:  
  - `80` (HTTP) / `443` (HTTPS) — фронтенд.  
  - `8080`/`5000` (бэкенд).  
  - `5432` (PostgreSQL) / `3306` (MySQL) / `27017` (MongoDB).  
- **Ресурсы**:  
  - 2+ ядра CPU, 4+ GB RAM (зависит от нагрузки).  

### **Настройка VM**
1. **Обновление системы**  
   ```bash
   sudo apt update && sudo apt upgrade -y  # для Ubuntu/Debian
   sudo yum update -y                     # для CentOS
   ```

2. **Установка базовых утилит**  
   ```bash
   sudo apt install -y git curl wget unzip
   ```

---

## **2. Установка и настройка базы данных (PostgreSQL)**
### **1. Установка PostgreSQL**
```bash
sudo apt install -y postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### **2. Настройка БД**
```bash
sudo -u postgres psql
```
```sql
CREATE DATABASE myapp_db;
CREATE USER myapp_user WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE myapp_db TO myapp_user;
\q
```

### **3. Проверка подключения**
```bash
psql -h localhost -U myapp_user -d myapp_db
```

---

## **3. Развёртывание бэкенда**
### **Вариант 1: Java (Spring Boot)**
#### **1. Установка Java (OpenJDK 17)**
```bash
sudo apt install -y openjdk-17-jdk
java -version  # проверка
```

#### **2. Клонирование и сборка**
```bash
git clone https://github.com/your-repo/backend.git
cd backend
./mvnw clean package  # для Maven
# или
./gradlew build      # для Gradle
```

#### **3. Запуск**
```bash
java -jar target/myapp-backend.jar --spring.datasource.url=jdbc:postgresql://localhost:5432/myapp_db
```

### **Вариант 2: Python (Django/FastAPI)**
#### **1. Установка Python и зависимостей**
```bash
sudo apt install -y python3 python3-pip python3-venv
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

#### **2. Настройка БД в `settings.py` (Django)**
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'myapp_db',
        'USER': 'myapp_user',
        'PASSWORD': 'secure_password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

#### **3. Запуск**
```bash
python manage.py migrate
python manage.py runserver 0.0.0.0:8000  # Django
# или
uvicorn main:app --host 0.0.0.0 --port 8000  # FastAPI
```

---

## **4. Развёртывание фронтенда**
### **Вариант 1: Статические файлы (React/Vue)**
#### **1. Установка Node.js**
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
node -v  # проверка
```

#### **2. Сборка фронтенда**
```bash
git clone https://github.com/your-repo/frontend.git
cd frontend
npm install
npm run build  # создаёт папку `dist/` или `build/`
```

#### **3. Настройка веб-сервера (Nginx)**
```bash
sudo apt install -y nginx
sudo cp -r frontend/dist/* /var/www/html/
sudo systemctl restart nginx
```

### **Вариант 2: Серверный рендеринг (Next.js/Nuxt.js)**
```bash
npm install
npm run build
npm run start  # запуск на 3000 порту
```

---

## **5. Настройка Nginx как прокси**
Чтобы связать фронтенд и бэкенд:  
```bash
sudo nano /etc/nginx/sites-available/myapp
```
```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        root /var/www/html;
        try_files $uri /index.html;
    }

    location /api {
        proxy_pass http://localhost:8080;  # Java/Python бэкенд
        proxy_set_header Host $host;
    }
}
```
```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t  # проверка конфига
sudo systemctl restart nginx
```

---

## **6. Настройка HTTPS (Certbot)**
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```
(Сертификаты обновляются автоматически.)

---

## **7. Запуск приложения**
### **1. Запуск бэкенда (в screen/tmux)**
```bash
screen -S backend
java -jar backend.jar  # или `python manage.py runserver`
Ctrl+A, D  # детач
```

### **2. Проверка работы**
- Фронтенд: `http://your-domain.com`  
- API: `http://your-domain.com/api`  
- БД: `psql -h localhost -U myapp_user -d myapp_db`  

---

## **8. Дополнительные настройки**
### **Автозапуск бэкенда (systemd)**
```bash
sudo nano /etc/systemd/system/myapp.service
```
```ini
[Unit]
Description=MyApp Backend
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/backend
ExecStart=/usr/bin/java -jar /home/ubuntu/backend/target/backend.jar
Restart=always

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl daemon-reload
sudo systemctl start myapp
sudo systemctl enable myapp
```

### **Бэкапы БД (cron)**
```bash
sudo crontab -e
```
```cron
0 3 * * * pg_dump -U myapp_user -d myapp_db > /backups/db_$(date +\%Y-\%m-\%d).sql
```
---

## **дальнейшее развитие**
оборачиваем в ансибл или что-то ещё и автоматизируем запуск ансибла

---

## **некоторые плюсы и минусы деплоя на VM (без контейнеров)**
| **плюсы**                          | **минусы**                          |
|-------------------------------------|-------------------------------------|
| ✅ Простота (не нужен Docker/K8s)   | ❌ Нет изоляции процессов           |
| ✅ Прямой доступ к ОС              | ❌ Сложнее масштабировать           |
| ✅ Меньше накладных расходов       | ❌ Нет версионности контейнеров     |

---

## **вывод**
1. **Установите БД** (PostgreSQL/MySQL/MongoDB).  
2. **Разверните бэкенд** (Java/Python) и настройте подключение к БД.  
3. **Соберите фронтенд** и разместите его в Nginx.  
4. **Настройте проксирование** API-запросов через Nginx.  
5. **Добавьте HTTPS** (Certbot).  
6. **Настройте автозапуск** (systemd).  

Этот подход подходит для **небольших проектов**, где не нужна сложная оркестрация. Для продакшена с высокой нагрузкой лучше использовать **Docker + Kubernetes**.  
