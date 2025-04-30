📞 doggo (github.com/mr-karan/d...) — это современный DNS-клиент командной строки, написанный на Go!

🌟 Он предоставляет удобный человеко-читаемый вывод, поддерживает протоколы DoH, DoT, DoQ, DNSCrypt и классические DNS-запросы через UDP/TCP. Также доступны функции JSON-вывода, обратного поиска DNS, поддержки нескольких резолверов и измерения времени ответа.

🔐 Лицензия: GPL-3.0

🖥 [Github]() 

@linuxacademiya

Настроить **DNS over HTTPS (DoH)** в Linux можно несколькими способами. DoH шифрует DNS-запросы, защищая их от перехвата и блокировок.  

---

## **Способы настройки DoH в Linux**

### **1. Через `systemd-resolved` (рекомендуется в Ubuntu, Fedora, Debian)**

`systemd-resolved` поддерживает DoH из коробки.

#### **Шаги:**

1. **Откройте конфиг `resolved.conf`:**
```
sudo nano /etc/systemd/resolved.conf


[Resolve]
DNS=1.1.1.1 8.8.8.8 9.9.9.9
DNSOverTLS=yes
```
# Или для DoH (Cloudflare, Google, Quad9):
DNSOverTLS=opportunistic
# Или явно указать DoH-сервер:
DNS=https://1.1.1.1/dns-query https://8.8.8.8/dns-query

```

sudo systemctl restart systemd-resolved

resolvectl status
```