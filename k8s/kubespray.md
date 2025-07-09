# Установка Kubernetes с помощью Kubespray: подробное руководство

Kubespray - это популярный инструмент для развертывания готовых к production кластеров Kubernetes с использованием Ansible. Ниже представлен пошаговый процесс установки с детальным описанием конфигурации.

## Подготовительные этапы

### 1. Требования к инфраструктуре
- **Минимальные ноды**:
  - 1 master (рекомендуется 3 для production, 2cpu4RAM70HDD)
  - 1-2 worker ноды (2cpu8RAM70HDD)
- **ОС**: Ubuntu 20.04/22.04, CentOS 7/8, RHEL 7/8, Debian 10/11
- **Доступ**: SSH с ключом на все ноды
- **Права**: sudo без пароля для пользователя ansible
- **Действия**: то есть создаем тем же ансибл пользака, прокидываем ему ключ и редактим судо для него

### 2. Подготовка управляющей машины
```bash
# Установка зависимостей
sudo apt update && sudo apt install -y python3-pip git

# Клонирование Kubespray
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
git checkout release-2.21  # Последняя стабильная версия

# Установка Ansible и Python-зависимостей
pip3 install -r requirements.txt
```

## Конфигурация кластера

### 1. Подготовка инвентарного файла
```bash
# Копирование примера инвентаря
cp -rfp inventory/sample inventory/mycluster

# Генерация инвентарного файла
declare -a IPS=(10.10.1.1 10.10.1.2 10.10.1.3)  # IP ваших серверов
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

### 2. Редактирование `hosts.yaml`
Пример конфигурации для 3 мастеров и 2 воркеров:
```yaml
all:
  hosts:
    node1:
      ansible_host: 10.10.1.1
      ip: 10.10.1.1
      access_ip: 10.10.1.1
    node2:
      ansible_host: 10.10.1.2
      ip: 10.10.1.2
      access_ip: 10.10.1.2
    node3:
      ansible_host: 10.10.1.3
      ip: 10.10.1.3
      access_ip: 10.10.1.3
    node4:
      ansible_host: 10.10.1.4
      ip: 10.10.1.4
      access_ip: 10.10.1.4
    node5:
      ansible_host: 10.10.1.5
      ip: 10.10.1.5
      access_ip: 10.10.1.5
  children:
    kube_control_plane:
      hosts:
        node1:
        node2:
        node3:
    kube_node:
      hosts:
        node1:
        node2:
        node3:
        node4:
        node5:
    etcd:
      hosts:
        node1:
        node2:
        node3:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```

### 3. Основные файлы переменных

#### `inventory/mycluster/group_vars/all/all.yml`
```yaml
# Базовая конфигурация
cluster_name: my-k8s-cluster
supplementary_addresses_in_ssl_keys: [10.10.1.1, 10.10.1.2, 10.10.1.3]
kube_version: v1.25.6  # Укажите нужную версию

# Сетевые настройки
kube_service_addresses: 10.233.0.0/18
kube_pods_subnet: 10.233.64.0/18

# Настройки контейнерного runtime
container_manager: containerd  # или docker, cri-o

# Настройки DNS
dns_mode: coredns
resolvconf_mode: host_resolvconf

# Доступ к кластеру
kubeconfig_localhost: true
kube_api_anonymous_auth: false
```

#### `inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml`
```yaml
# Настройки сети
kube_network_plugin: calico  # или cilium, flannel
calico_ipip_mode: Always
calico_vxlan_mode: Never
calico_typha: true
typha_replicas: 3

# Настройки контроля доступа
authorization_mode: ['Node', 'RBAC']

# Настройки дополнений
helm_enabled: true
metrics_server_enabled: true
local_path_provisioner_enabled: true

# Настройки ingress
ingress_nginx_enabled: true
ingress_nginx_host_network: false
```

## Запуск развертывания

### 1. Проверка доступности нод
```bash
ansible -i inventory/mycluster/hosts.yaml -m ping all
```

### 2. Запуск playbook
```bash
ansible-playbook -i inventory/mycluster/hosts.yaml \
  -u <ssh-user> \
  -b --become-user=root \
  --private-key=~/.ssh/id_rsa \
  cluster.yml
```

### 3. Дополнительные параметры запуска
Для ограничения количества параллельных операций:
```bash
ansible-playbook -i inventory/mycluster/hosts.yaml \
  --fork 10 \
  cluster.yml
```

Для перезапуска только определенных компонентов:
```bash
ansible-playbook -i inventory/mycluster/hosts.yaml \
  --tags=etcd,kube-master \
  cluster.yml
```

## Пост-установочные шаги

### 1. Получение конфигурации
```bash
mkdir ~/.kube
cp inventory/mycluster/artifacts/admin.conf ~/.kube/config
chmod 600 ~/.kube/config
```

### 2. Проверка кластера
```bash
kubectl get nodes
kubectl get pods -A
```

### 3. Установка сетевых плагинов (если не через Kubespray)
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

## Продвинутые настройки

### 1. Настройки etcd
```yaml
# inventory/mycluster/group_vars/etcd.yml
etcd_deployment_type: host
etcd_memory_limit: "4Gi"
etcd_disk_priority: "high"
```

### 2. Настройки контроля ресурсов
```yaml
# inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml
kubelet_custom_flags:
  - "--eviction-hard=memory.available<500Mi,nodefs.available<10%"
  - "--system-reserved=cpu=500m,memory=1Gi"
  - "--kube-reserved=cpu=200m,memory=1Gi"
```

### 3. Настройки хранилища
```yaml
# inventory/mycluster/group_vars/all/addons.yml
nfs_client_provisioner_enabled: true
nfs_client_provisioner_namespace: kube-system
nfs_client_provisioner_archiveOnDelete: "false"
nfs_client_provisioner_server: nfs.example.com
nfs_client_provisioner_path: /exports
```

### 4. Настройки безопасности
```yaml
# inventory/mycluster/group_vars/all/all.yml
kube_basic_auth: false
kube_token_auth: true
kube_api_pwd: "securepassword"
podsecuritypolicy_enabled: true
```

## Обновление кластера

1. Измените `kube_version` в `all.yml`
2. Запустите upgrade playbook:
```bash
ansible-playbook -i inventory/mycluster/hosts.yaml upgrade-cluster.yml
```

## Устранение неполадок

1. **Проверка логов Ansible**:
   ```bash
   tail -f /var/log/ansible.log
   ```

2. **Проверка компонентов на нодах**:
   ```bash
   journalctl -u kubelet -f
   ```

3. **Сброс и повторный запуск**:
   ```bash
   ansible-playbook -i inventory/mycluster/hosts.yaml reset.yml
   ansible-playbook -i inventory/mycluster/hosts.yaml cluster.yml
   ```

## Рекомендации для production

1. Используйте отдельные ноды для etcd (не менее 3)
2. Настройте балансировщик нагрузки для мастер-нод
3. Включите мониторинг (Prometheus оператор)
4. Настройте centralized logging
5. Регулярно делайте бэкапы etcd
6. Используйте политики безопасности Pod и NetworkPolicy