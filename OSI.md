# модель Open Systems Interconnection

Модель OSI (Open Systems Interconnection) состоит из **7 уровней**, каждый из которых выполняет свою функцию в процессе передачи данных. Ниже приведены уровни модели, возможные проблемы на каждом из них, а также команды и методы диагностики.  

---

### **1. Физический уровень (Physical Layer)**  
**Функция:** Передача битов (0 и 1) через физическую среду (кабель, Wi-Fi, оптоволокно).  
**Возможные проблемы:**  
- Обрыв кабеля  
- Плохой контакт разъема  
- Электромагнитные помехи  
- Неправильная скорость дуплекса  

**Диагностика:**  
✅ **Визуальный осмотр** (кабели, разъемы, индикаторы на сетевых устройствах)  
✅ **Команды:**  
- `ping` (проверка базовой связи)  
- `ip link` (Linux) / `netsh interface show interface` (Windows) – проверка состояния интерфейса  
- `ethtool <интерфейс>` (Linux) – проверка скорости, дуплекса  
- `ifconfig` / `ip a` – проверка поднят ли интерфейс  
✅ **Тестер кабелей** (TDR – Time Domain Reflectometer)  

---

### **2. Канальный уровень (Data Link Layer)**  
**Функция:** Формирование кадров (frames), управление доступом к среде (MAC-адресация).  
**Возможные проблемы:**  
- Ошибки CRC (поврежденные кадры)  
- MAC-адресация (дублирование MAC, неправильная ARP-таблица)  
- Проблемы с коммутацией (закольцовывание, STP)  

**Диагностика:**  
✅ **Проверка ARP-таблицы:**  
- `arp -a` (Windows/Linux)  
- `ip neigh` (Linux)  
✅ **Анализ MAC-адресов:**  
- `show mac address-table` (Cisco)  
- `bridge fdb show` (Linux)  
✅ **Проверка ошибок на интерфейсе:**  
- `ifconfig` / `ip -s link` (статистика ошибок)  
- `ethtool -S <интерфейс>` (статистика драйвера)  
✅ **Анализ STP (Spanning Tree Protocol):**  
- `show spanning-tree` (Cisco)  

---

### **3. Сетевой уровень (Network Layer)**  
**Функция:** Маршрутизация (CLNS, DDP, EIGRP, IGMP, IPsec, IPv4/IPv6, IPX, PIM, RIP, ICMP, OSPF, BGP).
**Возможные проблемы:**  
- Неправильная маршрутизация  
- Блокировка firewall  
- Проблемы с DHCP  
- Недоступность шлюза  

**Диагностика:**  
✅ **Проверка IP-адресации:**  
- `ip addr` / `ifconfig`  
- `ip route` / `route print` (Windows)  
✅ **Проверка маршрутизации:**  
- `traceroute` / `tracert` (трассировка пути)  
- `mtr` (комбинация ping + traceroute)  
✅ **Проверка доступности шлюза:**  
- `ping <шлюз>`  
✅ **Проверка DHCP:**  
- `dhclient -v` (Linux)  
- `ipconfig /release & /renew` (Windows)  
✅ **Проверка Firewall/NAT:**  
- `iptables -L` (Linux)  
- `netsh advfirewall show currentprofile` (Windows)  

---

### **4. Транспортный уровень (Transport Layer)**  
**Функция:** Надежная передача данных (TCP/UDP, порты, ATP, CUDP, DCCP, FCP, IL, MPTCP, RDP, RUDP, SCTP, SPX, SST, µTP ).  
**Возможные проблемы:**  
- Блокировка портов  
- Перегрузка сети (потеря пакетов)  
- Неправильная работа TCP (некорректное завершение сессий)  

**Диагностика:**  
✅ **Проверка открытых портов:**  
- `netstat -tulnp` (Linux)  
- `ss -tuln` (более современная альтернатива)  
- `Test-NetConnection -Port <порт>` (PowerShell)  
✅ **Анализ задержек и потерь:**  
- `ping` (ICMP)  
- `tcptraceroute` (трассировка с TCP)  
✅ **Проверка нагрузки на сеть:**  
- `iftop` (трафик в реальном времени)  
- `nload` (графический мониторинг)  

---

### **5. Сеансовый уровень (Session Layer)**  
**Функция:** Управление сессиями (установка, поддержка, завершение). ADSP, ASP, H.245, ISO-SP (X.225, ISO 8327), iSNS, L2F, L2TP, NetBIOS, PAP, PPTP, RPC, RTCP, SMPP, SCP, SOCKS, ZIP, SDP
**Возможные проблемы:**  
- Разрывы сессий (например, VPN, RDP)  
- Проблемы аутентификации  

**Диагностика:**  
✅ **Логи сессий:**  
- `journalctl -u <сервис>` (systemd)  
- `eventvwr.msc` (Windows Event Viewer)  
✅ **Мониторинг активных сессий:**  
- `who` / `w` (Linux)  
- `query session` (Windows)  

---

### **6. Уровень представления (Presentation Layer)**  
**Функция:** Преобразование данных (кодировки, шифрование, сжатие). AFP, ICA, LPP, NCP, NDR, Tox, XDR, X.25 
**Возможные проблемы:**  
- Несовместимость кодировок  
- Ошибки SSL/TLS  
- Проблемы с шифрованием  

**Диагностика:**  
✅ **Проверка SSL-сертификатов:**  
- `openssl s_client -connect <host>:<port>`  
- `Test-NetConnection -Port 443 -SSL` (PowerShell)  
✅ **Анализ кодировок:**  
- Логи приложений (например, веб-сервера Apache/Nginx)  

---

### **7. Прикладной уровень (Application Layer)**  
**Функция:** Взаимодействие с приложениями (HTTP, Telnet, FTP, TFTP, SMTP, DNS, BOOTP, SNMP, CMOT).  
**Возможные проблемы:**  
- Недоступность сервиса (веб-сайт, почта)  
- Ошибки DNS  
- Проблемы с авторизацией  

**Диагностика:**  
✅ **Проверка DNS:**  
- `nslookup` / `dig`  
- `ping <домен>` (проверка разрешения имени)  
✅ **Проверка HTTP:**  
- `curl -v http://site.com`  
- `telnet site.com 80` (ручная проверка)  
✅ **Проверка почты (SMTP):**  
- `telnet mail.server.com 25`  
✅ **Логи приложений:**  
- `/var/log/nginx/error.log` (Nginx)  
- Event Viewer (Windows)  