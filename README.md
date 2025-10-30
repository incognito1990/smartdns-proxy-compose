# SmartDNS via Nginx + Dnsmasq (Docker Compose)

## ✅ Требования (до начала)
- **Белый (публичный) IPv4-адрес** на сервере, доступный из интернета.
- **DNS‑запись A** на ваш DoT‑домен (например, `dot.example.com`) указывает на этот публичный IP.
- **Открытый фаервол** для входящих подключений по портам:
  - `80/tcp` — выдача/обновление сертификатов (Certbot standalone),
  - `443/tcp` — SNI‑прокси (Nginx stream),
  - `853/tcp` — DNS‑over‑TLS (DoT).
- Установлены **Docker** и **Docker Compose v2** (команда `docker compose`). Рекомендуемые версии: Docker Engine 20.10+.
- Корректное **время/часовой пояс** (NTP), иначе TLS может не работать.
- (Опционально) Правила безопасности в облаке (Security Groups/NSG) **разрешают** вышеперечисленные порты.

Примеры открытия портов (выберите ваш инструмент фаервола):

**UFW:**
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 853/tcp
sudo ufw reload
```

**firewalld:**
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=853/tcp
sudo firewall-cmd --reload
```

**iptables (минимум):**
```bash
sudo iptables -A INPUT -p tcp --dport 80  -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 853 -j ACCEPT
# не забудьте сохранить правила под вашу ОС
```

---

## 📁 Структура
```
.
├── docker-compose.yml
├── README.md
├── .gitignore
├── letsencrypt/          # сюда Certbot положить полную структуру (live/, archive/, renewal/)
│   └── .gitkeep
└── configs/
    ├── nginx/
    │   └── nginx.conf
    └── dnsmasq/
        ├── dnsmasq.template.conf
        └── dnsmasq.conf           # генерируется из template
```

## 🚀 Быстрый старт
1) Получи сертификаты с помощью Certbot (замени домен на свой реальный DoT‑домен):
```bash
docker run -it --rm   -p 80:80   -v "$(pwd)/letsencrypt:/etc/letsencrypt"   certbot/certbot certonly   --standalone   --agree-tos   --no-eff-email   -d dot.example.com
```
> Монтируем **всю** директорию `letsencrypt` — это сохраняет симлинки `live/`.

2) Подготовь конфигурацию для Dnsmasq:
- открой `configs/dnsmasq/dnsmasq.template.conf`;
- найди все вхождения `<REDIRECT_IP>`;
- замени их на **внешний IP‑адрес** сервера, где работает SmartDNS;
- сохрани файл как `configs/dnsmasq/dnsmasq.conf`.

Пример:
```bash
cp configs/dnsmasq/dnsmasq.template.conf configs/dnsmasq/dnsmasq.conf
sed -i 's|<REDIRECT_IP>|12.34.56.78|g' configs/dnsmasq/dnsmasq.conf  # замени IP-адрес на свой
```

3) Обнови `configs/nginx/nginx.conf`: замени `<YOUR_DOT_DOMAIN>` на свой реальный DoT‑домен.
   - Пути к ключам/сертификатам должны указывать на
   `/etc/letsencrypt/live/<YOUR_DOT_DOMAIN>/{fullchain.pem,privkey.pem}`.

4) Запуск:
```bash
docker compose up -d
```

## 🛡️ Приватность
- Держи `letsencrypt/` в репозитории пустой (только `.gitkeep`), а реальные файлы — на сервере.
- Конфиг `dnsmasq.conf` с твоими IP **не должен** попадать в публичный репозиторий.
