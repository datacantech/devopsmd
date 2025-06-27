### **Kubeadm: принципы работы и основные команды**

**Kubeadm** — это официальный инструмент от Kubernetes для быстрой инициализации кластера, добавления нод и управления базовыми компонентами. Он не предназначен для промышленных кластеров (для этого используют `kops`, `RKE`, `OpenShift`), но идеально подходит для локальных тестов и обучения.

---

## **🔹 Основные принципы работы**
1. **Минимализм**  
   - Kubeadm настраивает только базовые компоненты (`kube-apiserver`, `kube-controller-manager`, `kube-scheduler`, `kube-proxy`, `CoreDNS` и `etcd`).  
   - Не устанавливает сетевой плагин (CNI) — его нужно ставить отдельно (`Calico`, `Flannel`, `WeaveNet` и др.).  

2. **Идемпотентность**  
   - Команды можно запускать несколько раз без побочных эффектов.  

3. **Декларативный подход**  
   - Конфигурация кластера задается в YAML-файле (`kubeadm-config.yaml`).  

4. **Этапы работы**  
   - `kubeadm init` – создает Control Plane (мастер-ноду).  
   - `kubeadm join` – добавляет worker-ноды в кластер.  
   - `kubeadm upgrade` – обновляет версию кластера.  

---

## **🔹 Основные команды kubeadm**

### **1️⃣ Инициализация кластера (на мастере)**
```bash
sudo kubeadm init \
  --apiserver-advertise-address=<MASTER_IP> \
  --pod-network-cidr=10.244.0.0/16 \  # Для Flannel/Calico
  --control-plane-endpoint=<LOAD_BALANCER_IP>  # Для HA-кластера
```
После выполнения команды:
- Выводится **`kubeadm join`-токен** для добавления worker-нод.
- Создается конфиг для `kubectl`:  
  ```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```

---

### **2️⃣ Добавление worker-нод**
```bash
sudo kubeadm join <MASTER_IP>:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>
```
> **ℹ️ Если токен утерян**, его можно посмотреть на мастере:  
> ```bash
> kubeadm token list
> kubeadm token create --print-join-command
> ```

---

### **3️⃣ Управление кластером**
| Команда | Описание |
|---------|----------|
| `kubeadm reset` | Сброс ноды (удаляет конфиги и данные) |
| `kubeadm token list` | Показать активные join-токены |
| `kubeadm certs check-expiration` | Проверить сроки сертификатов |
| `kubeadm certs renew` | Обновить сертификаты |
| `kubeadm upgrade plan` | Проверить доступные обновления |
| `kubeadm upgrade apply v1.28.0` | Обновить кластер до версии 1.28.0 |

---

### **4️⃣ Настройка сети (CNI)**
После `kubeadm init` нужно установить CNI-плагин, иначе Pod'ы не смогут общаться.  
**Пример для Calico:**
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
**Пример для Flannel:**
```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

---

### **5️⃣ Высокая доступность (HA)**
Для HA-кластера нужно:
1. Настроить **Load Balancer** перед Control Plane.
2. Инициализировать первый мастер:
   ```bash
   kubeadm init --control-plane-endpoint=<LB_IP>
   ```
3. Добавить остальные мастера:
   ```bash
   kubeadm join <LB_IP>:6443 --token <TOKEN> \
     --discovery-token-ca-cert-hash <HASH> \
     --control-plane --certificate-key <CERT_KEY>
   ```

---

## **🔹 Что делает kubeadm под капотом?**
1. **Генерирует сертификаты** (в `/etc/kubernetes/pki`).  
2. **Запускает статические Pod'ы** для компонентов Control Plane (через `/etc/kubernetes/manifests`).  
3. **Создает `kubeconfig`-файлы** для доступа к API.  
4. **Применяет базовые RBAC-правила**.  

---

## **🔹 Пример полного процесса развертывания кластера**
```bash
# На мастере
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# На worker-нодах
sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash <HASH>
```

---

## **🔹 Плюсы и минусы kubeadm**
✅ **Простота** – быстро развернуть тестовый кластер.  
✅ **Официальная поддержка** от Kubernetes.  
✅ **Гибкость** – можно кастомизировать конфиги.  

❌ **Нет автоматического масштабирования** (в отличие от `kops` или `RKE`).  
❌ **Нет встроенного мониторинга/логирования**.  
❌ **Требует ручного управления сертификатами**.  

---

### **Вывод**
Kubeadm — отличный инструмент для локальных кластеров и обучения, но для прода лучше использовать более продвинутые решения (`kops`, `Rancher`, `OpenShift`).