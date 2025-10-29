# SmartDNS via Nginx + Dnsmasq (Docker Compose)

## 📁 Структура
```
.
├── docker-compose.yml
├── README.md
├── .gitignore
├── .env.example
├── letsencrypt/          # сюда Certbot положит полную структуру (live/, archive/, renewal/)
│   └── .gitkeep
└── configs/
    ├── nginx/
    │   └── nginx.conf
    └── dnsmasq/
        └── dnsmasq.conf
```

## 🚀 Быстрый старт
1) Скопируй переменные окружения:
```bash
cp .env.example .env
```
2) Получи сертификаты с помощью Certbot (замени домен при необходимости):
```bash
docker run -it --rm   -p 80:80   -v "$(pwd)/letsencrypt:/etc/letsencrypt"   certbot/certbot certonly   --standalone   --agree-tos   --no-eff-email   -d dot.example.com
```
> Мы монтируем **всю** директорию `letsencrypt` — это сохраняет симлинки `live/`.

3) Обнови `configs/nginx/nginx.conf`: замени `<YOUR_DOT_DOMAIN>` на реальный DoT‑домен (или шаблонизируй приватно переменной `DOT_DOMAIN`).

4) Запуск:
```bash
docker compose up -d
```

## 🛡️ Приватность
- Не коммить реальные домены и IP.
- Держи `letsencrypt/` в репозитории пустой (только `.gitkeep`), а реальные файлы — на сервере.
