# Product Service - Redis Course Project

Этот проект является практическим примером для курса по Redis от Павла Сорокина. 
Демонстрирует различные подходы к кэшированию, distributed locking и rate limiting с использованием Redis.

[ссылка на видео на youtube](https://www.youtube.com/watch?v=blrqQOS8kOA)

## 🚀 Что включено в проект

### 1. Три реализации CRUD операций
- **DbProductService** - работа только с базой данных (без кэширования)
- **ManualCachingProductService** - ручное управление кэшем через RedisTemplate
- **SpringAnnotationCachingProductService** - кэширование через Spring Cache аннотации

### 2. Distributed Lock Pattern
- **RedisLockManager** - реализация distributed lock с использованием Redis
- Lua скрипт для атомарного освобождения блокировки
- Поддержка TTL для автоматического освобождения блокировок

### 3. Fixed Window Rate Limiting
- **FixedWindowRateLimiter** - реализация rate limiting с фиксированным окном
- Настраиваемые лимиты и размер окна
- Интеграция через фильтр

### 4. База данных и тестовые данные
- PostgreSQL с 40 миллионами тестовых записей
- SQL скрипт для генерации моковых данных
- Автоматическая настройка схемы

### 5. Нагрузочное тестирование
- K6 тесты для всех CRUD операций
- Сравнение производительности разных режимов кэширования
- Настраиваемые параметры нагрузки

## 🛠 Технологический стек

- **Java 21**
- **Spring Boot 3.5.6**
- **Spring Data Redis**
- **Spring Cache**
- **PostgreSQL**
- **Redis**
- **MapStruct** (маппинг объектов)
- **Lombok**
- **K6** (нагрузочное тестирование)
- **Docker & Docker Compose**

## 📋 Требования

- Java 21+
- Docker & Docker Compose
- Maven 3.6+
- K6 (для нагрузочного тестирования)

## 🚀 Быстрый старт

### 1. Клонирование и настройка

```bash
git clone <repository-url>
cd product-service-redis-course-yt
```

### 2. Настройка окружения

Скопируйте файл с переменными окружения:
```bash
cp example.env .env
```

Отредактируйте `.env` файл при необходимости:
```env
# Postgres
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/postgres
SPRING_DATASOURCE_USERNAME=postgres
SPRING_DATASOURCE_PASSWORD=postgres

# Redis
SPRING_REDIS_HOST=localhost
SPRING_REDIS_PORT=6379
SPRING_REDIS_PASSWORD=123
SPRING_REDIS_USERNAME=123
SPRING_REDIS_SSL_ENABLED=false

# Rate limiting
RATE_LIMIT_ENABLED=false
RATE_LIMIT_LIMIT=100
RATE_LIMIT_WINDOW_SIZE=60s
```

### 3. Запуск инфраструктуры

```bash
# Запуск PostgreSQL и Redis
docker-compose -f docker-compose.dev.yaml up -d
```

### 4. Инициализация базы данных

```bash
# Подключение к PostgreSQL и выполнение seed скрипта
docker exec -i $(docker-compose -f docker-compose.dev.yaml ps -q postgres) psql -U postgres -d postgres < infra/base-seed.sql
```

**⚠️ Внимание:** Скрипт создает 40 миллионов записей, это может занять несколько минут!

### 5. Запуск приложения

```bash
# Сборка и запуск
mvn clean package
java -jar target/product-service-0.0.1-SNAPSHOT.jar
```

Или через Maven:
```bash
mvn spring-boot:run
```

Приложение будет доступно по адресу: http://localhost:8080

## 📊 API Endpoints

### Swagger UI
- **URL:** http://localhost:8080/swagger-ui.html

### Основные endpoints:

#### Продукты
- `GET /api/products/{id}?cacheMode={mode}` - Получить продукт по ID
- `POST /api/products` - Создать продукт
- `PUT /api/products/{id}` - Обновить продукт
- `DELETE /api/products/{id}` - Удалить продукт

#### Distributed Lock
- `POST /api/locking/products/{id}/update` - Обновить продукт с блокировкой

#### Параметры cacheMode:
- `NONE_CACHE` - без кэширования (DbProductService)
- `MANUAL` - ручное кэширование (ManualCachingProductService)
- `SPRING` - Spring Cache аннотации (SpringAnnotationCachingProductService)

## 🧪 Нагрузочное тестирование

### Установка K6

**macOS:**
```bash
brew install k6
```

**Ubuntu/Debian:**
```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
echo "deb https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
sudo apt-get update
sudo apt-get install k6
```

### Запуск тестов

Перейдите в директорию с тестами:
```bash
cd k6
```

#### Доступные команды:

```bash
# Показать справку
make help

# Тест получения продукта (по умолчанию MANUAL кэширование)
make get

# Тест с разными режимами кэширования
make get-none-cache    # Без кэширования
make get-manual-cache  # Ручное кэширование
make get-spring-cache  # Spring Cache

# Тест создания продуктов
make post

# Тест обновления продуктов
make put

# Тест удаления продуктов
make delete

# Запуск всех тестов
make all
```

#### Настройка параметров:

```bash
# Изменение URL сервера
make get BASE_URL=http://prod-server:8080

# Изменение режима кэширования
make get CACHE_MODE=SPRING

# Комбинированная настройка
make get BASE_URL=http://localhost:8080 CACHE_MODE=NONE_CACHE
```

### Пример результатов тестирования

```bash
# Тест без кэширования
make get-none-cache
# Результат: ~2000 RPS, высокое время ответа

# Тест с ручным кэшированием
make get-manual-cache  
# Результат: ~8000 RPS, низкое время ответа

# Тест с Spring Cache
make get-spring-cache
# Результат: ~7500 RPS, низкое время ответа
```

## 🔧 Конфигурация

### Режимы кэширования

Приложение поддерживает переключение между тремя режимами кэширования через параметр `cacheMode`:

1. **NONE_CACHE** - Прямые запросы к базе данных
2. **MANUAL** - Ручное управление кэшем через RedisTemplate
3. **SPRING** - Автоматическое кэширование через Spring Cache

### Rate Limiting

Настройка в `application.yml`:
```yaml
rate-limit:
  enabled: true
  limit: 100          # Максимум запросов
  window-size: 60s    # Размер окна
```

### Redis конфигурация

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      password: 123
      username: 123
      ssl:
        enabled: false
```

## 📁 Структура проекта

```
src/main/java/dev/sorokin/productservice/
├── api/                    # REST контроллеры и DTO
├── config/                 # Конфигурация Spring
├── domain/                 # Доменная логика
│   ├── db/                # JPA сущности и репозитории
│   └── service/           # Реализации ProductService
├── locking/               # Distributed Lock
├── ratelimit/             # Rate Limiting
└── ProductServiceRedisCourseYtApplication.java

k6/                        # Нагрузочные тесты
├── config.js             # Конфигурация K6
├── k6_utils.js           # Утилиты для тестов
├── test-*.js             # Тесты для CRUD операций
└── Makefile              # Команды для запуска тестов

infra/                     # Инфраструктура
└── base-seed.sql         # SQL скрипт с тестовыми данными
```

## 🎯 Ключевые особенности

### 1. Сравнение производительности кэширования
Проект позволяет наглядно сравнить производительность разных подходов к кэшированию:
- Без кэширования: ~2000 RPS
- Ручное кэширование: ~8000 RPS  
- Spring Cache: ~7500 RPS

### 2. Distributed Lock
Реализация надежного distributed lock с:
- Атомарным захватом блокировки
- TTL для предотвращения deadlock
- Lua скриптом для безопасного освобождения

### 3. Rate Limiting
Fixed window rate limiter с:
- Настраиваемыми лимитами
- Автоматической очисткой ключей
- Интеграцией в Spring Filter

### 4. Масштабируемые тестовые данные
40 миллионов записей для реалистичного тестирования производительности.

## 🐛 Troubleshooting

### Проблемы с подключением к Redis
```bash
# Проверка статуса Redis
docker-compose -f docker-compose.dev.yaml ps redis

# Логи Redis
docker-compose -f docker-compose.dev.yaml logs redis
```

### Проблемы с PostgreSQL
```bash
# Проверка статуса PostgreSQL
docker-compose -f docker-compose.dev.yaml ps postgres

# Подключение к базе
docker exec -it $(docker-compose -f docker-compose.dev.yaml ps -q postgres) psql -U postgres -d postgres
```

### Проблемы с K6
```bash
# Проверка установки K6
k6 version

# Запуск простого теста
k6 run --vus 1 --duration 1s k6/test-get.js
```

## 📚 Дополнительные ресурсы

- [Spring Data Redis Documentation](https://docs.spring.io/spring-data/redis/docs/current/reference/html/)
- [Spring Cache Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#cache)
- [K6 Documentation](https://k6.io/docs/)
- [Redis Documentation](https://redis.io/documentation)

## 👨‍💻 Автор

**Павел Сорокин** - курс по Redis

---

*Этот проект создан в образовательных целях для демонстрации различных паттернов работы с Redis в Spring Boot приложениях.*
