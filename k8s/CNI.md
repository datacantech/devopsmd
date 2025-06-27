# **Принцип работы CNI в Kubernetes и настройка сети**

## **1. Что такое CNI (Container Network Interface)?**
**CNI** — это стандартный интерфейс для настройки сети в контейнерах. В Kubernetes он отвечает за:
- **Выделение IP-адресов** подам.
- **Настройку маршрутизации** между подами и нодами.
- **Обеспечение сетевой политики** (если поддерживается плагином).

### **🔹 Основные требования CNI в Kubernetes**
- Каждый Pod должен иметь **уникальный IP** (используется плоская сеть).
- Pod'ы должны общаться **между собой без NAT** (если находятся в одном кластере).
- Pod'ы должны иметь доступ **во внешний мир** (через NAT или маршрутизацию).

---

## **2. Как работает сеть в Kubernetes?**
### **🔹 Основные компоненты**
| Компонент | Роль |
|-----------|------|
| **CNI-плагин** (Calico, Flannel, Cilium) | Настраивает сетевые интерфейсы и правила. |
| **kube-proxy** | Обеспечивает балансировку и Service-сеть (iptables/IPVS). |
| **CoreDNS** | DNS-резолвинг для сервисов (`<service>.<namespace>.svc.cluster.local`). |

### **🔹 Основные сетевые модели**
1. **Flat-модель (например, Flannel)**
   - Все Pod'ы находятся в одной большой сети.
   - Использует **overlay-сеть** (VXLAN, IP-in-IP).
   - Подходит для небольших кластеров.

2. **BGP-модель (например, Calico)**
   - Использует **BGP-протокол** для маршрутизации между нодами.
   - Не требует overlay, но сложнее в настройке.

3. **eBPF-модель (например, Cilium)**
   - Использует **eBPF** для ускорения сети и безопасности.
   - Подходит для высоконагруженных кластеров.

---

## **3. Установка CNI-плагина**
### **🔹 1. Flannel (самый простой вариант)**
```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
**Принцип работы:**
- Создает **overlay-сеть** (VXLAN).
- Назначает Pod'ам IP из диапазона `10.244.0.0/16`.
- Не поддерживает Network Policies.

### **🔹 2. Calico (рекомендуемый для production)**
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
**Принцип работы:**
- Использует **BGP** или **IP-in-IP**.
- Поддерживает **Network Policies** (можно ограничивать трафик между подами).
- Можно отключить overlay (`CALICO_IPV4POOL_IPIP: "Never"`).

### **🔹 3. Cilium (eBPF, для высоких нагрузок)**
```bash
helm repo add cilium https://helm.cilium.io/
helm install cilium cilium/cilium --namespace kube-system
```
**Принцип работы:**
- Использует **eBPF** вместо iptables.
- Ускоряет трафик и улучшает безопасность.
- Поддерживает **L7-политики** (например, блокировка HTTP-запросов).

---

## **4. Основные сетевые сценарии в Kubernetes**
### **🔹 1. Общение Pod → Pod (внутри одной ноды)**
- Через **CNI-интерфейс** (`cni0` в Flannel, `calico` в Calico).
- Прямой **bridge-доступ** (без NAT).

### **🔹 2. Общение Pod → Pod (между разными нодами)**
- **Flannel**: через VXLAN-туннель.
- **Calico**: через BGP или IP-in-IP.
- **Cilium**: через eBPF-маршрутизацию.

### **🔹 3. Общение Pod → Service**
- **kube-proxy** создает iptables/IPVS-правила.
- Трафик идет через **ClusterIP** (виртуальный IP сервиса).

### **🔹 4. Общение Pod → Внешний мир**
- Через **NAT** (маскарадинг) на ноде.
- В Calico/Cilium можно настраивать политики.

### **🔹 5. Ingress-трафик (внешний доступ к сервисам)**
- Через **Ingress-контроллер** (Nginx, Traefik).
- Настраивается отдельно от CNI.

---

## **5. Проверка сети**
### **🔹 Проверить IP-адреса подов**
```bash
kubectl get pods -o wide
```
### **🔹 Проверить сетевые интерфейсы на ноде**
```bash
ip a
```
### **🔹 Проверить маршруты**
```bash
ip route
```
### **🔹 Проверить DNS**
```bash
kubectl run -it --rm --image=alpine test -- sh
ping nginx.default.svc.cluster.local
```

---

## **6. Проблемы и их решение**
| Проблема | Решение |
|----------|---------|
| **Pods не получают IP** | Проверить CNI-плагин (`kubectl get pods -n kube-system`). |
| **Pods не могут общаться** | Проверить `kube-proxy` и firewall. |
| **Нет доступа в интернет** | Проверить NAT (`iptables -t nat -L`). |
| **DNS не работает** | Проверить CoreDNS (`kubectl logs -n kube-system coredns-xxxx`). |

---

- **CNI отвечает за сеть Pod'ов** в Kubernetes.
- **Flannel** — самый простой вариант, **Calico** — для production, **Cilium** — для highload.
- **kube-proxy** обеспечивает работу Service'ов.
- **CoreDNS** отвечает за DNS внутри кластера.

### **Схема работы CNI-плагина в Kubernetes (Markdown + Mermaid)**

```mermaid
graph TD
    subgraph Kubernetes Node
        Kubelet -->|1. Создает Pod| ContainerRuntime(Docker/containerd)
        ContainerRuntime -->|2. Вызывает CNI-плагин| CNI(CNI Plugin<br>(Flannel/Calico/Cilium))
        CNI -->|3. Настраивает сеть| PodNetwork[Pod Network]
        CNI -->|4. Обновляет IPAM| IPAM(IP Address Management)
    end

    subgraph CNI Plugin Components
        CNI -->|Конфигурация| CNIConfig(/etc/cni/net.d)
        CNI -->|Бинарные файлы| CNIBin(/opt/cni/bin)
    end

    subgraph Сетевые взаимодействия
        Pod1(Pod A) -->|5. L2/L3-трафик| Pod2(Pod B)
        Pod1 -->|6. Service-трафик| KubeProxy(kube-proxy)
        KubeProxy -->|7. iptables/IPVS| Service(Service IP)
        Pod1 -->|8. Внешний трафик| External(Internet)
    end

    subgraph Внешние компоненты
        Service --> CoreDNS(CoreDNS)
        External -->|NAT| NodeIP(Node IP)
    end
```

---

### **Пояснение к схеме**

#### **1. Установка CNI-плагина**
- **Где находится CNI?**  
  - Конфигурация: `/etc/cni/net.d/` (например, `10-flannel.conflist`).  
  - Бинарные файлы: `/opt/cni/bin/` (например, `flannel`, `bridge`, `host-local`).  
- **Как устанавливается?**  
  - Через `kubectl apply -f` (например, `kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml`).  

#### **2. Как работает CNI при создании Pod?**
1. **Kubelet** запрашивает у **Container Runtime** (Docker/containerd) создание Pod.  
2. **Container Runtime** вызывает **CNI-плагин** (например, `flannel`).  
3. **CNI-плагин**:  
   - Создает сетевой интерфейс для Pod (например, `eth0`).  
   - Выделяет IP из пула (**IPAM**).  
   - Настраивает маршрутизацию (через `bridge`, `vxlan`, или `BGP`).  
4. **IPAM** сохраняет информацию о выданных IP (например, в `etcd` для Calico).  

#### **3. Как идут сетевые запросы?**
- **Pod → Pod (внутри ноды)**:  
  - Через **CNI-мост** (`cni0` в Flannel).  
  ```bash
  ip link show cni0
  ```
- **Pod → Pod (между нодами)**:  
  - **Flannel**: Через VXLAN-туннель (`flannel.1`).  
  - **Calico**: Через BGP или IP-in-IP.  
- **Pod → Service**:  
  - Трафик идет через **kube-proxy** (iptables/IPVS).  
  ```bash
  iptables -t nat -L | grep <Service-IP>
  ```
- **Pod → Внешний мир**:  
  - Через **NAT** на ноде (`iptables -t nat -A POSTROUTING`).  

#### **4. Куда уходят данные?**
| Действие | Куда уходят данные | Через что? |
|----------|--------------------|------------|
| **Pod → Pod** | Внутри ноды | `cni0` (мост) |
| **Pod → Другая нода** | Через overlay (VXLAN) или BGP | `flannel.1` / `tunl0` |
| **Pod → Service** | В `kube-proxy` | iptables/IPVS |
| **Pod → Интернет** | На внешний интерфейс ноды | NAT (`eth0`) |

---

### **Примеры команд для проверки**
1. **Проверить сетевые интерфейсы Pod:**
   ```bash
   kubectl exec -it <pod-name> -- ip a
   ```
2. **Посмотреть CNI-конфигурацию:**
   ```bash
   cat /etc/cni/net.d/10-flannel.conflist
   ```
3. **Проверить маршруты на ноде:**
   ```bash
   ip route
   ```
4. **Логи CNI-плагина:**
   ```bash
   journalctl -u kubelet -f | grep cni
   ```

---

- **CNI-плагин** — это "мозг" сети в Kubernetes, отвечающий за подключение Pod'ов.  
- **Flannel/Calico/Cilium** работают на уровне **L2/L3**, обеспечивая связность Pod'ов.  
- ** дополняют CNI, обеспечивая работу Service'ов и DNS.  
- Для продакшна лучше **Calico/Cilium** (из-за поддержки Network Policies и производительности).