# ===========================================
# БЕЗОПАСНЫЙ DOCKERFILE
# ===========================================
# Многоступенчатая сборка для минимизации размера образа

# Используем slim образ вместо full
FROM python:3.11-slim as builder

# Установка зависимостей для сборки
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Копируем только requirements сначала (кеширование слоев)
COPY req.txt .

# Устанавливаем зависимости
RUN pip install --no-cache-dir --user -r req.txt

# ===========================================
# ФИНАЛЬНЫЙ ОБРАЗ
# ===========================================
FROM python:3.11-slim

# Метаданные
LABEL maintainer="security@example.com"
LABEL description="Secure Python backend application"
LABEL version="1.0"

# Установка curl для healthcheck
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# БЕЗОПАСНОСТЬ: Создаем непривилегированного пользователя
RUN groupadd -r appuser && useradd -r -g appuser -u 1001 appuser

WORKDIR /app

# Копируем зависимости из builder stage
COPY --from=builder /root/.local /home/appuser/.local

# Копируем код приложения
COPY --chown=appuser:appuser main.py .

# БЕЗОПАСНОСТЬ: Устанавливаем правильные права доступа
RUN chmod 755 /app && \
    chmod 644 /app/main.py

# БЕЗОПАСНОСТЬ: Переключаемся на непривилегированного пользователя
USER appuser

# Добавляем локальные бинарники в PATH
ENV PATH=/home/appuser/.local/bin:$PATH

# Переменные окружения
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1

EXPOSE 8080

# БЕЗОПАСНОСТЬ: Healthcheck
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/ || exit 1

# Запуск от непривилегированного пользователя
CMD ["python", "main.py"]
