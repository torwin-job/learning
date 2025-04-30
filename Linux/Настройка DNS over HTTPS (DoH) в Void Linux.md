### **Настройка DNS over HTTPS (DoH) в Void Linux**  
Void Linux использует `runit` вместо `systemd`, поэтому настройка DoH немного отличается. Рассмотрим два способа: через **`dnscrypt-proxy`** и **`stubby`**.  

---

## **1. Настройка через `dnscrypt-proxy` (рекомендуется)**
`dnscrypt-proxy` поддерживает DoH и легко настраивается в Void.  

### **Установка и настройка**
1. **Установите `dnscrypt-proxy`:**  
   ```bash
   sudo xbps-install dnscrypt-proxy
   ```

2. **Отредактируйте конфиг (`/etc/dnscrypt-proxy/dnscrypt-proxy.toml`):**  
   ```bash
   sudo nano /etc/dnscrypt-proxy/dnscrypt-proxy.toml
   ```
   Измените следующие параметры:  
   ```toml
   server_names = ['cloudflare', 'google', 'quad9-doh']  # Выбор DoH-серверов
   listen_addresses = ['127.0.0.1:53']  # Слушать локально
   ```

3. **Запустите `dnscrypt-proxy`:**  
   ```bash
   sudo ln -s /etc/sv/dnscrypt-proxy /var/service/
   sudo sv up dnscrypt-proxy
   ```

4. **Настройте `/etc/resolv.conf`:**  
   ```bash
   sudo nano /etc/resolv.conf
   ```
   Замените содержимое на:  
   ```plaintext
   nameserver 127.0.0.1
   options edns0
   ```
   (Если `resolv.conf` перезаписывается DHCP, используйте `dhcpcd` с опцией `nohook resolv.conf`.)

---

## **2. Настройка через `stubby` (альтернативный способ)**
`stubby` — это DNS-стаб-резолвер, поддерживающий DoH.  

### **Установка и настройка**
1. **Установите `stubby`:**  
   ```bash
   sudo xbps-install stubby
   ```

2. **Отредактируйте конфиг (`/etc/stubby/stubby.yml`):**  
   ```bash
   sudo nano /etc/stubby/stubby.yml
   ```
   Пример конфига для Cloudflare и Google:  
   ```yaml
   resolution_type: GET
   round_robin_upstreams: 1
   listen_addresses:
     - 127.0.0.1@53
   upstream_recursive_servers:
     - address_data: 1.1.1.1
       tls_auth_name: "cloudflare-dns.com"
     - address_data: 8.8.8.8
       tls_auth_name: "dns.google"
   ```

3. **Запустите `stubby`:**  
   ```bash
   sudo ln -s /etc/sv/stubby /var/service/
   sudo sv up stubby
   ```

4. **Проверьте работу:**  
   ```bash
   dig @127.0.0.1 google.com
   ```

---

## **3. Проверка DoH**
Убедитесь, что запросы идут через HTTPS:  
```bash
curl -s 'https://1.1.1.1/dns-query?name=google.com&type=A' | jq
```
Или с помощью `tcpdump`:  
```bash
sudo tcpdump -i any port 443 and host 1.1.1.1
```

---

## **Вывод**
- **Проще всего** использовать `dnscrypt-proxy`.  
- **Альтернатива** — `stubby`, если нужен минимализм.  
- **Важно** убедиться, что `/etc/resolv.conf` указывает на `127.0.0.1`.  

Если нужна помощь с конкретным сценарием — спрашивайте! 🚀