

### **Основные шаги по настройке WireGuard:**

1. Установка WireGuard:
Установите WireGuard на сервер и клиентские устройства
sudo apt install wireguard
2. Генерация ключей:
Создайте пару ключей (публичный и приватный) для сервера и каждого клиента
cd /etc/wireguard
wg genkey | tee privatekey | wg pubkey > publickey
# для сервера
wg genkey | tee /etc/wireguard/server_privatekey | wg pubkey | tee /etc/wireguard/server_publickey 
3. Настройка сервера:
# Создаем директорию для конфигурационных файлов
mkdir -p /etc/wireguard/config
Создайте конфигурационный файл для сервера (например, wg0.conf). 
Укажите IP-адрес сервера, порт, приватный ключ сервера и публичные ключи подключенных клиентов. 
Добавьте правила для перенаправления трафика, если это необходимо. 

# Создаем конфигурационный файл
cat <<EOF > /etc/wireguard/wg0.conf
[Interface]
Address = 10.13.13.1/24
ListenPort = 51820
PrivateKey = $(cat /etc/wireguard/server_privatekey)
SaveConfig = true

PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
EOF

4. Настройка клиентов:
Создайте конфигурационный файл для каждого клиента (например, client1.conf). 

Укажите публичный ключ сервера, IP-адрес сервера, порт и IP-адрес клиента в приватной сети. 
Добавьте приватный ключ клиента в конфигурационный файл. 
# Добавляем клиента
cat <<EOF > /etc/wireguard/wg0.conf
[Interface]
PrivateKey = <приватный_ключ_клиента>
Address = 10.13.13.2/24
DNS = 8.8.8.8, 8.8.4.4

[Peer]
PublicKey = <публичный_ключ_сервера>
Endpoint = <IP_или_домен_сервера>:51820
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
EOF
или
for i in $(seq 1 $CLIENT_COUNT); do
  CLIENT_NAME="client$i"
  wg genkey | tee /etc/wireguard/${CLIENT_NAME}_privatekey | wg pubkey | tee /etc/wireguard/${CLIENT_NAME}_publickey
  CLIENT_IP="10.13.13.$((i+1))"
  cat <<EOF >> /etc/wireguard/wg0.conf
[Peer]
PublicKey = $(cat /etc/wireguard/${CLIENT_NAME}_publickey)
AllowedIPs = $CLIENT_IP/32
EOF
5. Запуск и подключение:
Запустите WireGuard на сервере
systemctl enable wg-quick@wg0
systemctl start wg-quick@wg0
 и клиентах
 wg-quick up wg0
Подключите клиентов к серверу, используя их конфигурационные файлы. 
Проверьте работоспособность VPN, убедившись в изменении IP-адреса
Запуск клиента:

**Проверка работы**
На сервере: wg show
На клиенте: ping 10.13.13.1


пример 

sudo wg genkey | sudo tee /etc/wireguard/server_privatekey | sudo wg pubkey | sudo tee /etc/wireguard/server_publickey 
sudo wg genkey | sudo tee /etc/wireguard/peer_1_privatekey | sudo wg pubkey | sudo tee /etc/wireguard/peer_1_publickey

[Interface]
PrivateKey = GK6Dr3pBlwHZlhXZTKztlKJ0Oic0F4n66d1f0xb/BXI=
Address = 10.13.13.1/24
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
ListenPort = 51820

[Peer]
#peer_1
PublicKey = 6uGbfFZDO2u8tdhhhQmjhomo/ZtONc/WvOE34wt9h2I=
AllowedIPs = 10.13.13.2/32

или
[Interface]
Address = 10.13.13.1/24
ListenPort = 51820
PrivateKey = $(cat /etc/wireguard/server_privatekey)
SaveConfig = true

PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = $(cat /etc/wireguard/peer_1_publickey)
AllowedIPs = 10.13.13.2/32