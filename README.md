# SmartDNS via Nginx + Dnsmasq (Docker Compose)


## 📁 Структура
```
.
├── docker-compose.yml
├── README.md
├── .gitignore
├── letsencrypt/          # сюда Certbot положит полную структуру (live/, archive/, renewal/)
│   └── .gitkeep
└── configs/
    ├── nginx/
    │   └── nginx.conf
    └── dnsmasq/
        ├── dnsmasq.template.conf
        └── dnsmasq.conf           # генерируется из template
```

## 🚀 Быстрый старт
1) Получи сертификаты с помощью Certbot (замени домен при необходимости):
```bash
docker run -it --rm   -p 80:80   -v "$(pwd)/letsencrypt:/etc/letsencrypt"   certbot/certbot certonly   --standalone   --agree-tos   --no-eff-email   -d dot.example.com
```
> Монтируем **всю** директорию `letsencrypt` — это сохраняет симлинки `live/`.

2) Подготовь конфигурацию для Dnsmasq:
- открой `configs/dnsmasq/dnsmasq.template.conf`;
- найди все вхождения `<REDIRECT_IP>`;
- замени их на **внешний IP-адрес** сервера, где работает SmartDNS;
- сохрани файл как `configs/dnsmasq/dnsmasq.conf`.

Пример:
```bash
cp configs/dnsmasq/dnsmasq.template.conf configs/dnsmasq/dnsmasq.conf
sed -i 's|<REDIRECT_IP>|12.34.56.78|g' configs/dnsmasq/dnsmasq.conf  # замени IP-адрес на свой
```

3) Обнови `configs/nginx/nginx.conf`: замени `<YOUR_DOT_DOMAIN>` на реальный DoT‑домен.
   - Пути к ключам/сертификатам должны указывать на `
   /etc/letsencrypt/live/<YOUR_DOT_DOMAIN>/{fullchain.pem,privkey.pem}`.

4) Запуск:
```bash
docker compose up -d
```

## 🛡️ Приватность
- Не коммить реальные домены и IP (оставляй плейсхолдеры или используй локальные оверлеи).
- Держи `letsencrypt/` в репозитории пустой (только `.gitkeep`), а реальные файлы — на сервере.
- Конфиг `dnsmasq.conf` с твоими IP **не должен** попадать в публичный репозиторий.
