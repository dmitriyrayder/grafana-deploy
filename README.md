# 🔐 Безопасный Grafana Мониторинг для Docker

Полнофункциональная система мониторинга Docker контейнеров с акцентом на безопасность.
Включает мониторинг 6 ML моделей и поддержку Grafana + Prometheus + Loki + cAdvisor.

## ⚠️ ВАЖНО: Версия 2.0 - Безопасная конфигурация

**Эта версия содержит исправления всех критических уязвимостей безопасности!**

Подробный отчет по безопасности: [SECURITY.md](./SECURITY.md)

---

## 🎯 Возможности

### Мониторинг
- ✅ **Grafana** - Визуализация метрик и логов
- ✅ **Prometheus** - Сбор и хранение метрик
- ✅ **Loki** - Агрегация логов
- ✅ **cAdvisor** - Метрики Docker контейнеров (CPU, Memory, Network, Disk)
- ✅ **Node Exporter** - Системные метрики хоста
- ✅ **Promtail** - Сбор логов из контейнеров

### Безопасность
- 🔒 Управление секретами через `.env`
- 🔒 Изолированные Docker сети
- 🔒 Порты привязаны к localhost
- 🔒 Непривилегированные контейнеры
- 🔒 Resource limits для всех сервисов
- 🔒 Health checks
- 🔒 Security constraints
- 🔒 Nginx reverse proxy с SSL
- 🔒 Rate limiting
- 🔒 HTTP Security Headers

### ML Модели
- 📊 Мониторинг 6 ML сервисов (service1-6 на портах 8501-8506)
- 📊 Метрики производительности
- 📊 Логирование запросов

---

## 🚀 Быстрый старт

### 1. Требования

- Docker 20.10+
- Docker Compose 2.0+
- Nginx (опционально, для production)
- SSL сертификат (опционально, для production)

### 2. Установка

```bash
# Клонируйте репозиторий
git clone <your-repo-url>
cd grafana-deploy

# Создайте .env файл из шаблона
cp .env.example .env

# ⚠️ ОБЯЗАТЕЛЬНО: Отредактируйте .env и измените ВСЕ пароли!
nano .env
```

**Минимальная конфигурация .env:**
```bash
# Grafana
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=your_strong_password_here_min_16_chars

# Доступ (для production используйте 0.0.0.0 + Nginx)
GRAFANA_HOST=127.0.0.1
PROMETHEUS_HOST=127.0.0.1
LOKI_HOST=127.0.0.1
```

### 3. Запуск

```bash
# Запустите все сервисы
docker-compose up -d

# Проверьте статус
docker-compose ps

# Посмотрите логи
docker-compose logs -f
```

### 4. Доступ

По умолчанию (без Nginx):
- **Grafana**: http://localhost:3000
- **Prometheus**: http://localhost:9090
- **cAdvisor**: http://localhost:8080

С Nginx (production):
- **Grafana**: https://your-domain.com/grafana/
- **Prometheus**: https://your-domain.com/prometheus/
- **ML Services**: https://your-domain.com/service1-6/

**Логин в Grafana:**
- Username: из `.env` (по умолчанию: admin)
- Password: из `.env` (ОБЯЗАТЕЛЬНО ИЗМЕНИТЕ!)

---

## 📁 Структура проекта

```
grafana-deploy/
├── docker-compose.yml          # Основная конфигурация (БЕЗОПАСНАЯ v2.0)
├── .env.example                # Шаблон для секретов
├── .gitignore                  # Игнорирование секретов
├── prometheus.yml              # Конфигурация Prometheus + 6 ML моделей
├── loki-config.yaml            # Конфигурация Loki с security limits
├── promtail-config.yaml        # Конфигурация Promtail
├── nginx-secure.conf           # Безопасный Nginx reverse proxy
├── SECURITY.md                 # Полный отчет по безопасности
├── README.md                   # Этот файл
├── grafana/
│   ├── datasources.yaml        # Datasources для Grafana
│   └── example-dashboard.json  # Пример дашборда
└── backend/
    ├── Dockerfile              # Безопасный Dockerfile
    ├── main.py                 # Тестовое приложение
    └── req.txt                 # Зависимости
```

---

## 📊 Что мониторится?

### 1. Docker контейнеры (cAdvisor)
- CPU usage (%)
- Memory usage (MB)
- Network I/O (bytes)
- Disk I/O (bytes)
- Container status

### 2. Системные метрики хоста (Node Exporter)
- CPU load
- Memory/Swap
- Disk usage
- Network traffic
- System uptime

### 3. ML Модели (Prometheus)
- Доступность сервисов (up/down)
- HTTP метрики (если экспортируются)
- Process метрики (CPU, Memory)
- Custom метрики (если настроены)

### 4. Логи (Loki + Promtail)
- Логи всех Docker контейнеров
- Логи с метками (container_name, image_name)
- Поиск по логам в Grafana

---

## 🔧 Настройка для production

### 1. Настройка Nginx

```bash
# Скопируйте конфиг
sudo cp nginx-secure.conf /etc/nginx/sites-available/grafana-monitoring

# Создайте симлинк
sudo ln -s /etc/nginx/sites-available/grafana-monitoring /etc/nginx/sites-enabled/

# Проверьте конфигурацию
sudo nginx -t

# Перезапустите Nginx
sudo systemctl reload nginx
```

### 2. Настройка SSL (Let's Encrypt)

```bash
# Установите certbot
sudo apt install certbot python3-certbot-nginx

# Получите сертификат
sudo certbot --nginx -d your-domain.com

# Автообновление (проверьте таймер)
sudo systemctl status certbot.timer
```

### 3. Настройка Firewall

```bash
# Разрешите только HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Заблокируйте прямой доступ к портам мониторинга
sudo ufw deny 3000/tcp
sudo ufw deny 9090/tcp
sudo ufw deny 3100/tcp
sudo ufw deny 8080/tcp

# Включите firewall
sudo ufw enable
```

### 4. Обновите .env для production

```bash
# Измените хосты для работы с Nginx
GRAFANA_HOST=0.0.0.0
PROMETHEUS_HOST=127.0.0.1
LOKI_HOST=127.0.0.1
CADVISOR_HOST=127.0.0.1
```

---

## 🎨 Настройка дашбордов Grafana

### Импорт готовых дашбордов

1. Откройте Grafana → Dashboards → Import
2. Импортируйте дашборды по ID:

**Рекомендуемые дашборды:**
- **Docker** (ID: 193) - Мониторинг контейнеров
- **Node Exporter** (ID: 1860) - Системные метрики
- **cAdvisor** (ID: 14282) - Детальные метрики контейнеров
- **Loki** (ID: 12611) - Просмотр логов

### Создание своего дашборда для ML моделей

```
1. Grafana → Create → Dashboard
2. Add Panel
3. Query Prometheus:
   - up{type="ml-model"}              # Доступность ML сервисов
   - rate(http_requests_total[5m])    # Запросы в секунду
   - container_memory_usage_bytes     # Использование памяти
```

---

## 📈 Примеры запросов Prometheus

### Мониторинг контейнеров
```promql
# CPU usage по контейнерам
rate(container_cpu_usage_seconds_total[5m]) * 100

# Memory usage
container_memory_usage_bytes / 1024 / 1024

# Network traffic
rate(container_network_receive_bytes_total[5m])
```

### Мониторинг ML моделей
```promql
# Доступность сервисов
up{type="ml-model"}

# Процессы
process_cpu_seconds_total{job=~"ml-service.*"}
process_resident_memory_bytes{job=~"ml-service.*"}
```

### Системные метрики
```promql
# CPU Load Average
node_load1

# Free Memory
node_memory_MemFree_bytes / 1024 / 1024 / 1024

# Disk usage
(node_filesystem_size_bytes - node_filesystem_free_bytes) / node_filesystem_size_bytes * 100
```

---

## 🛠️ Управление

### Команды Docker Compose

```bash
# Запуск
docker-compose up -d

# Остановка
docker-compose down

# Перезапуск конкретного сервиса
docker-compose restart grafana

# Просмотр логов
docker-compose logs -f

# Просмотр логов конкретного сервиса
docker-compose logs -f prometheus

# Обновление образов
docker-compose pull
docker-compose up -d

# Очистка (ВНИМАНИЕ: удаляет данные!)
docker-compose down -v
```

### Backup и Restore

```bash
# Backup Grafana
docker-compose exec grafana grafana-cli admin reset-admin-password <new-password>
docker run --rm --volumes-from grafana -v $(pwd):/backup ubuntu tar czf /backup/grafana-backup.tar.gz /var/lib/grafana

# Backup Prometheus
docker run --rm --volumes-from prometheus -v $(pwd):/backup ubuntu tar czf /backup/prometheus-backup.tar.gz /prometheus

# Restore
docker run --rm --volumes-from grafana -v $(pwd):/backup ubuntu tar xzf /backup/grafana-backup.tar.gz -C /
```

---

## 🔍 Troubleshooting

### Проблема: Grafana не запускается

```bash
# Проверьте логи
docker-compose logs grafana

# Проверьте права на volume
docker volume inspect grafana-deploy_grafana-data

# Пересоздайте контейнер
docker-compose down
docker-compose up -d grafana
```

### Проблема: Prometheus не собирает метрики

```bash
# Проверьте targets
curl http://localhost:9090/api/v1/targets

# Проверьте конфигурацию
docker-compose exec prometheus promtool check config /etc/prometheus/prometheus.yml

# Перезагрузите конфигурацию
curl -X POST http://localhost:9090/-/reload
```

### Проблема: ML модели не мониторятся

```bash
# Проверьте доступность сервиса
curl http://localhost:8501/metrics

# Если метрики недоступны:
# 1. Добавьте Prometheus client в ваше приложение
# 2. Или используйте только метрики контейнеров через cAdvisor
# 3. Закомментируйте соответствующий job в prometheus.yml
```

### Проблема: Порты заняты

```bash
# Проверьте занятые порты
sudo lsof -i :3000
sudo lsof -i :9090

# Измените порты в .env файле
echo "GRAFANA_HOST=127.0.0.1:3001" >> .env
```

---

## 🔐 Безопасность

### Чеклист перед production

- [ ] Изменены ВСЕ пароли в `.env`
- [ ] `.env` файл НЕ закоммичен в Git
- [ ] SSL сертификаты настроены
- [ ] Nginx reverse proxy настроен
- [ ] Firewall включен
- [ ] Порты доступны только через Nginx
- [ ] Rate limiting работает
- [ ] Backup настроен
- [ ] Логи ротируются

### Рекомендации

1. **Регулярные обновления**
   ```bash
   docker-compose pull
   docker-compose up -d
   ```

2. **Мониторинг безопасности**
   - Проверяйте логи на подозрительную активность
   - Используйте fail2ban для защиты от brute-force
   - Настройте алерты в Grafana

3. **Аудит**
   ```bash
   # Docker Security Bench
   docker run --rm -it --net host --pid host --userns host --cap-add audit_control \
     -v /etc:/etc:ro -v /usr/bin/containerd:/usr/bin/containerd:ro \
     -v /usr/bin/runc:/usr/bin/runc:ro -v /usr/lib/systemd:/usr/lib/systemd:ro \
     -v /var/lib:/var/lib:ro -v /var/run/docker.sock:/var/run/docker.sock:ro \
     docker/docker-bench-security
   ```

Подробнее: [SECURITY.md](./SECURITY.md)

---

## 📚 Дополнительные ресурсы

### Видео
- [Оригинальное видео на YouTube](https://youtu.be/LyocQr7cN-0)

### Документация
- [Grafana Documentation](https://grafana.com/docs/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Loki Documentation](https://grafana.com/docs/loki/latest/)
- [cAdvisor GitHub](https://github.com/google/cadvisor)

### Инструменты
- [Docker Bench Security](https://github.com/docker/docker-bench-security)
- [Trivy Scanner](https://github.com/aquasecurity/trivy)
- [Hadolint](https://github.com/hadolint/hadolint)

---

## 🤝 Contributing

При обнаружении уязвимостей безопасности:
1. НЕ публикуйте детали публично
2. Создайте приватный issue
3. Следуйте responsible disclosure

---

## 📝 Changelog

### v2.0.0 (2025-12-03) - Безопасная версия
- ✅ Исправлены все критические уязвимости безопасности
- ✅ Добавлен мониторинг Docker контейнеров (cAdvisor + Node Exporter)
- ✅ Добавлен мониторинг 6 ML моделей
- ✅ Добавлено управление секретами через .env
- ✅ Добавлен безопасный Nginx конфиг
- ✅ Добавлены resource limits и health checks
- ✅ Улучшен Dockerfile с multi-stage build
- ✅ Добавлена полная документация по безопасности

### v1.0.0 - Начальная версия
- Базовая настройка Grafana + Prometheus + Loki
- ⚠️ **НЕБЕЗОПАСНО - НЕ ИСПОЛЬЗУЙТЕ В PRODUCTION!**

---

## 📧 Контакты

По вопросам безопасности: security@example.com

---

## 📄 Лицензия

MIT License

---

**🔐 Безопасность - приоритет №1!**

Перед развертыванием в production обязательно прочитайте [SECURITY.md](./SECURITY.md)
