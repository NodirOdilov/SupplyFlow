# SupplyFlow

**Комплексная платформа управления цепочками поставок — от закупки и склада до логистики, качества и прогнозирования спроса под полным контролем вашей организации.**

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Django](https://img.shields.io/badge/Django-4.2-092E20?style=for-the-badge&logo=django&logoColor=white)
![DRF](https://img.shields.io/badge/DRF-3.14-092E20?style=for-the-badge)
![React](https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-7-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![Celery](https://img.shields.io/badge/Celery-5-37814A?style=for-the-badge&logo=celery&logoColor=white)

![Docker Compose](https://img.shields.io/badge/Docker_Compose-3.9-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-1.25-009639?style=for-the-badge&logo=nginx&logoColor=white)
![JWT](https://img.shields.io/badge/Auth-JWT-black?style=for-the-badge)
![OpenAPI](https://img.shields.io/badge/API-OpenAPI_3-6BA539?style=for-the-badge&logo=openapi-initiative&logoColor=white)

---

## Содержание

1. [О проекте](#1-о-проекте)
2. [Ключевые возможности](#2-ключевые-возможности)
3. [Технологический стек](#3-технологический-стек)
4. [Структура репозитория](#4-структура-репозитория)
5. [Архитектура и как это работает](#5-архитектура-и-как-это-работает)
6. [Доменная модель (крупными блоками)](#6-доменная-модель-крупными-блоками)
7. [Сервисы в Docker Compose](#7-сервисы-в-docker-compose)
8. [Быстрый старт (локально, Docker)](#8-быстрый-старт-локально-docker)
9. [Основные команды](#9-основные-команды)
10. [Ручной запуск frontend и backend](#10-ручной-запуск-frontend-и-backend)
11. [Конфигурация и переменные окружения](#11-конфигурация-и-переменные-окружения)
12. [API, очереди и интеграции](#12-api-очереди-и-интеграции)
13. [Мониторинг и эксплуатация](#13-мониторинг-и-эксплуатация)
14. [CI/CD (рекомендуемая схема)](#14-cicd-рекомендуемая-схема)
15. [Безопасность и мультитенантность](#15-безопасность-и-мультитенантность)
16. [Роли компонентов в продакшене](#16-роли-компонентов-в-продакшене)
17. [Лицензия](#17-лицензия)
18. [Поддержка](#18-поддержка)

---

## 1. О проекте

**SupplyFlow** — продуктовая B2B SaaS-платформа для сквозного управления supply chain: поставщики, закупки, отгрузки, склад, контроль качества, прогноз спроса и аналитика KPI в едином контуре.

Платформа рассчитана на три типа пользователей:

| Аудитория | Сценарий |
|-----------|----------|
| **Операционные команды** | Ежедневная работа через веб-интерфейс: заявки, заказы, приёмка, отслеживание поставок |
| **Руководители и закупщики** | Дашборды, согласования, рейтинги поставщиков, контроль сроков и бюджета |
| **Интеграторы** | REST API с OpenAPI-документацией, JWT-аутентификация, webhooks-ready архитектура |

### Что это за тип системы

SupplyFlow — **многосервисная распределённая платформа**, а не монолитный скрипт. Бизнес-логика сосредоточена в Django-приложениях, тяжёлые и периодические операции вынесены в Celery, интерфейс — React SPA за reverse proxy.

| Аспект | Описание |
|--------|----------|
| **Продукт** | SCM-система: закупки, логистика, склад, QC, forecasting, analytics |
| **Архитектура** | Django REST API + React SPA + Celery workers + Nginx |
| **Данные** | PostgreSQL (транзакционные метаданные) + Redis (кэш, брокер, результаты задач) |
| **Изоляция** | Мультитенантность на уровне организации (`Organization`) |

---

## 2. Ключевые возможности

### Поставщики и контракты
- Реестр поставщиков с контактами, рейтингами и историей оценок
- Учёт договоров, автоматический пересчёт performance score (Celery Beat, понедельник 03:00 UTC)

### Закупки (Procurement)
- Заявки на закупку (**Purchase Requisition**) → заказы (**Purchase Order**)
- Многоуровневые workflow согласования (**ApprovalWorkflow**)
- Позиционная детализация заказов (**PurchaseOrderLine**)

### Логистика и отгрузки
- Управление перевозчиками (**Carrier**) и отгрузками (**Shipment**)
- Трекинг событий (**ShipmentTracking**), опрос статусов у carrier API каждые 15 минут
- Географическая визуализация (Leaflet) на фронтенде

### Склад и запасы
- Склады (**Warehouse**), номенклатура (**InventoryItem**), остатки (**StockLevel**)
- Алерты по точкам перезаказа — ежедневная проверка в 06:00 UTC

### Качество (QC)
- Инспекции при приёмке (**QualityInspection**, **InspectionItem**)
- Отчёты о дефектах (**DefectReport**), блокировка приёмки до прохождения QC

### Прогнозирование спроса
- Взвешенное скользящее среднее и линейная регрессия (NumPy / SciPy)
- Ежедневный пересчёт прогнозов в 02:00 UTC, метрики точности (**ForecastAccuracy**)

### Аналитика и KPI
- Снимки метрик дашборда, целевые KPI, правила алертов
- Spend analysis, lead time, on-time delivery, цикл закупки

### Доступ и роли (RBAC)
- Роли: **Admin**, **Manager**, **Buyer**, **Viewer**
- JWT (SimpleJWT), refresh rotation, rate limiting API

---

## 3. Технологический стек

| Слой | Технологии |
|------|------------|
| **Backend** | Python 3.11+, Django 4.2, Django REST Framework 3.14 |
| **Auth** | djangorestframework-simplejwt, email-based User model |
| **Frontend** | React 18, Redux Toolkit, Recharts, Leaflet |
| **БД** | PostgreSQL 15 |
| **Кэш / брокер** | Redis 7 |
| **Очереди** | Celery 5, django-celery-beat, django-celery-results |
| **API docs** | drf-spectacular (Swagger UI + ReDoc) |
| **Прокси** | Nginx 1.25 |
| **Контейнеризация** | Docker, Docker Compose 3.9 |
| **Наблюдаемость** | Sentry SDK (опционально), structured logging |
| **Файлы (prod)** | django-storages + AWS S3 (опционально) |

---

## 4. Структура репозитория

```
SupplyFlow/
├── backend/
│   ├── apps/
│   │   ├── accounts/       # Пользователи, организации, JWT
│   │   ├── suppliers/      # Поставщики, контакты, рейтинги, договоры
│   │   ├── procurement/    # Заявки, заказы, согласования
│   │   ├── shipments/      # Отгрузки, трекинг, перевозчики
│   │   ├── inventory/      # Склады, остатки, движения
│   │   ├── quality/        # Инспекции, дефекты
│   │   ├── forecasting/    # Прогноз спроса
│   │   └── analytics/      # KPI, дашборды, алерты
│   ├── config/             # settings, urls, wsgi, celery
│   ├── middleware/         # Organization, timing, audit headers
│   ├── utils/              # pagination, exceptions, helpers
│   └── manage.py
├── frontend/
│   ├── public/
│   └── src/
│       ├── api/            # HTTP-клиент, endpoints
│       ├── components/     # UI-модули по доменам
│       ├── pages/            # Dashboard, Settings, …
│       ├── store/            # Redux store
│       ├── hooks/            # useAuth, useApi
│       └── styles/
├── nginx/
│   └── nginx.conf          # Reverse proxy, static, media
├── docker-compose.yml
├── .env.example
└── README.md
```

---

## 5. Архитектура и как это работает

```mermaid
flowchart TB
    subgraph Client["Клиент"]
        SPA["React SPA"]
    end

    subgraph Edge["Edge"]
        NGX["Nginx :80"]
    end

    subgraph App["Application"]
        API["Django + Gunicorn :8000"]
        CW["Celery Worker"]
        CB["Celery Beat"]
    end

    subgraph Data["Data Layer"]
        PG[(PostgreSQL)]
        RD[(Redis)]
    end

    SPA --> NGX
    NGX -->|/api/, /admin/| API
    NGX -->|/| SPA
    API --> PG
    API --> RD
    CW --> PG
    CW --> RD
    CB --> RD
```

**Типовой запрос пользователя**

1. Браузер загружает React-приложение через Nginx (`/`).
2. API-вызовы идут на `/api/` → Gunicorn → DRF view → сериализатор → PostgreSQL.
3. `OrganizationMiddleware` прикрепляет `request.organization` из JWT-пользователя — все выборки scoped по тенанту.
4. Долгие операции (прогноз, опрос carrier, пересчёт score) — в Celery через Redis.
5. Статика и медиа отдаются Nginx из volume (`/static/`, `/media/`).

---

## 6. Доменная модель (крупными блоками)

```
Organization ──┬── User (role: admin | manager | buyer | viewer)
               │
               ├── Supplier ── SupplierContact, SupplierRating, Contract
               │
               ├── PurchaseRequisition ──► PurchaseOrder ── PurchaseOrderLine
               │                              └── ApprovalWorkflow
               │
               ├── Shipment ── ShipmentItem, ShipmentTracking
               │      └── Carrier
               │
               ├── Warehouse ── InventoryItem ── StockLevel
               │
               ├── QualityInspection ── InspectionItem, DefectReport
               │
               ├── ForecastConfiguration ── DemandForecast ── ForecastAccuracy
               │
               └── DashboardMetricSnapshot, KPITarget, AlertRule ── AlertEvent
```

Все сущности привязаны к **организации** — это граница мультитенантности и единица биллинга/квот в продуктовой модели.

---

## 7. Сервисы в Docker Compose

| Сервис | Образ / сборка | Назначение | Порты |
|--------|----------------|------------|-------|
| `db` | `postgres:15-alpine` | Транзакционное хранилище | `5432` |
| `redis` | `redis:7-alpine` | Брокер Celery, кэш Django | `6379` |
| `backend` | `./backend` Dockerfile | API, migrate, collectstatic, Gunicorn | `8000` |
| `celery_worker` | backend image | Асинхронные задачи (4 concurrency) | — |
| `celery_beat` | backend image | Периодические задачи (DatabaseScheduler) | — |
| `frontend` | `./frontend` Dockerfile | React dev server | `3000` |
| `nginx` | `nginx:1.25-alpine` | Единая точка входа, static/media | `80` |

**Volumes:** `postgres_data`, `redis_data`, `static_volume`, `media_volume`.

---

## 8. Быстрый старт (локально, Docker)

### Предварительные требования

- Docker Engine 24+ и Docker Compose v2+
- Git
- ~4 GB свободной RAM для полного стека

### Запуск за 5 минут

```bash
git clone https://github.com/NodirOdilov/SupplyFlow.git
cd SupplyFlow

cp .env.example .env
# Отредактируйте DJANGO_SECRET_KEY и пароли БД при необходимости

docker compose up --build -d

docker compose exec backend python manage.py createsuperuser

# Опционально: демо-данные
docker compose exec backend python manage.py loaddata sample_data
```

### Точки входа

| Сервис | URL |
|--------|-----|
| **Веб-приложение** | http://localhost |
| **API** | http://localhost/api/ |
| **Django Admin** | http://localhost/api/admin/ |
| **Swagger UI** | http://localhost/api/docs/ |
| **ReDoc** | http://localhost/api/redoc/ |
| **Frontend (напрямую)** | http://localhost:3000 |
| **Backend (напрямую)** | http://localhost:8000 |

---

## 9. Основные команды

Проект использует **Docker Compose** как единый интерфейс эксплуатации (Makefile не обязателен).

```bash
# Поднять / остановить стек
docker compose up -d
docker compose down

# Логи
docker compose logs -f backend
docker compose logs -f celery_worker

# Миграции
docker compose exec backend python manage.py migrate
docker compose exec backend python manage.py makemigrations

# Суперпользователь
docker compose exec backend python manage.py createsuperuser

# Тесты backend
docker compose exec backend python manage.py test

# Shell Django
docker compose exec backend python manage.py shell_plus

# Пересборка после изменения зависимостей
docker compose build --no-cache backend
docker compose up -d backend celery_worker celery_beat
```

---

## 10. Ручной запуск frontend и backend

Используйте, когда нужна горячая перезагрузка без полного Docker-стека (PostgreSQL и Redis должны быть доступны локально или в контейнерах).

### Backend

```bash
cd backend
python -m venv venv

# Windows
venv\Scripts\activate
# Linux / macOS
source venv/bin/activate

pip install -r requirements.txt

set DJANGO_SETTINGS_MODULE=config.settings.development
set DATABASE_URL=postgres://supplyflow:supplyflow@localhost:5432/supplyflow
set REDIS_URL=redis://localhost:6379/0

python manage.py migrate
python manage.py runserver
```

### Celery

```bash
cd backend
celery -A config.celery worker -l info
celery -A config.celery beat -l info
```

### Frontend

```bash
cd frontend
npm install
set REACT_APP_API_URL=http://localhost:8000/api
npm start
```

Приложение откроется на http://localhost:3000 с проксированием API на backend.

---

## 11. Конфигурация и переменные окружения

Скопируйте `.env.example` → `.env`. Полный перечень — в файле; ключевые параметры:

| Переменная | Назначение | По умолчанию |
|------------|------------|--------------|
| `DJANGO_SECRET_KEY` | Секрет Django (обязателен в prod) | — |
| `DJANGO_DEBUG` | Режим отладки | `True` (dev) |
| `DJANGO_SETTINGS_MODULE` | Модуль настроек | `config.settings.development` |
| `DATABASE_URL` | PostgreSQL DSN | `postgres://…@db:5432/supplyflow` |
| `REDIS_URL` | Redis для кэша и Celery | `redis://redis:6379/0` |
| `ALLOWED_HOSTS` | Разрешённые Host-заголовки | `localhost,127.0.0.1,backend` |
| `CORS_ALLOWED_ORIGINS` | CORS для SPA | `http://localhost,http://localhost:3000` |
| `REACT_APP_API_URL` | Base URL API для фронтенда | `http://localhost/api` |
| `SENTRY_DSN` | Трекинг ошибок (prod) | пусто |
| `AWS_*` | S3 для media/static (prod) | пусто |

**Продакшен:** `DJANGO_DEBUG=False`, `config.settings.production`, управляемые PostgreSQL/Redis, TLS на Nginx, уникальный `DJANGO_SECRET_KEY` ≥ 50 символов.

---

## 12. API, очереди и интеграции

### REST API

Базовый префикс: `/api/`. Аутентификация: `Authorization: Bearer <access_token>`.

| Ресурс | Endpoint | Методы |
|--------|----------|--------|
| Аутентификация | `/api/auth/login/` | POST |
| Пользователи | `/api/accounts/users/` | GET, POST |
| Поставщики | `/api/suppliers/` | GET, POST, PUT, DELETE |
| Рейтинги | `/api/suppliers/{id}/ratings/` | GET, POST |
| Заявки | `/api/procurement/requisitions/` | GET, POST, PUT |
| Заказы | `/api/procurement/orders/` | GET, POST, PUT |
| Согласования | `/api/procurement/approvals/` | GET, POST |
| Отгрузки | `/api/shipments/` | GET, POST, PUT |
| Трекинг | `/api/shipments/{id}/tracking/` | GET, POST |
| Номенклатура | `/api/inventory/items/` | GET, POST, PUT |
| Остатки | `/api/inventory/stock/` | GET |
| QC | `/api/quality/inspections/` | GET, POST, PUT |
| Прогнозы | `/api/forecasting/` | GET, POST |
| Аналитика | `/api/analytics/spend/`, `…/lead-time/` | GET |

Интерактивная схема: **Swagger** (`/api/docs/`), **ReDoc** (`/api/redoc/`), OpenAPI JSON (`/api/schema/`).

### Периодические задачи Celery Beat

| Задача | Расписание | Очередь |
|--------|------------|---------|
| `poll_carrier_updates` | каждые 15 мин | `shipments` |
| `generate_daily_forecasts` | 02:00 UTC ежедневно | `analytics` |
| `check_reorder_levels` | 06:00 UTC ежедневно | `inventory` |
| `compute_supplier_scores` | пн 03:00 UTC | `analytics` |

### Заголовки аудита (ответ API)

- `X-Request-Id` — корреляция запроса
- `X-Organization-Id` — UUID текущей организации
- `X-Request-Duration-Ms` — время обработки (slow > 500 ms логируется)

---

## 13. Мониторинг и эксплуатация

| Область | Подход |
|---------|--------|
| **Логи** | Structured logging в stdout (`[{asctime}] {levelname} …`) — сбор через Docker / Loki / CloudWatch |
| **Ошибки** | `SENTRY_DSN` в production settings |
| **Здоровье** | Healthcheck PostgreSQL (`pg_isready`) и Redis (`PING`) в Compose |
| **Метрики API** | Заголовок `X-Request-Duration-Ms`, throttle 1000 req/h per user |
| **Бэкапы** | Volume `postgres_data` — регулярный snapshot + PITR на managed DB |
| **Масштабирование** | Горизонтально: `celery_worker` replicas; Gunicorn `--workers` по CPU |

```bash
# Проверка состояния контейнеров
docker compose ps

# Health PostgreSQL
docker compose exec db pg_isready -U supplyflow
```

---

## 14. CI/CD (рекомендуемая схема)

В репозитории pipeline описан как **целевая** схема — подключите `.github/workflows/` или GitLab CI по шаблону:

```yaml
# Пример этапов
# 1. lint + test backend (manage.py test)
# 2. test frontend (npm test -- --watchAll=false)
# 3. docker build backend + frontend
# 4. push image → registry
# 5. deploy: migrate + collectstatic + rolling update
```

**Чеклист перед релизом:** миграции применены, секреты в vault, `DEBUG=False`, CORS/ALLOWED_HOSTS согласованы с доменом, SSL termination на Nginx.

---

## 15. Безопасность и мультитенантность

- **JWT** с rotation refresh tokens и blacklist после ротации
- **Пароли:** минимум 10 символов, validators Django
- **Изоляция данных:** каждый запрос scoped через `request.organization`
- **Rate limiting:** 100 req/h (anon), 1000 req/h (authenticated)
- **CSRF** для session-auth; API — Bearer-only
- **Статика:** WhiteNoise + Nginx cache headers
- **Файлы в prod:** S3 с приватными ACL через `django-storages`
- **Заголовки:** `X-Frame-Options`, CORS whitelist, audit correlation IDs

> Не коммитьте `.env` с production-секретами. Используйте `.env.example` как шаблон.

---

## 16. Роли компонентов в продакшене

| Компонент | Роль |
|-----------|------|
| **Nginx** | TLS termination, reverse proxy, gzip, раздача static/media, лимит body 100 MB |
| **Gunicorn** | WSGI, 4 workers (настраивается), timeout 120 s |
| **Django** | Бизнес-логика, ORM, admin, OpenAPI |
| **Celery Worker** | Фон: carrier polling, forecasts, reorder alerts, supplier scores |
| **Celery Beat** | Cron через `django_celery_beat` (расписание в БД) |
| **PostgreSQL** | ACID, связи заказов/остатков/трекинга |
| **Redis** | Broker, result backend routing, Django cache (db 2) |
| **React SPA** | UI, Redux state, charts & maps |

**Рекомендация:** managed PostgreSQL + Redis (ElastiCache / Memorystore), отдельные очереди Celery per domain при росте нагрузки.

---

## 17. Лицензия

Проприетарное программное обеспечение. Все права защищены.

Использование, копирование и распространение — только с письменного разрешения правообладателя.

---

## 18. Поддержка

| Канал | Действие |
|-------|----------|
| **Issues** | [GitHub Issues](https://github.com/NodirOdilov/SupplyFlow/issues) — баги и feature requests |
| **Документация API** | `/api/docs/` после запуска стека |
| **Admin** | `/api/admin/` — операционное управление данными |

При обращении укажите: версию коммита, фрагмент `.env` (без секретов), `X-Request-Id` из ответа API и шаги воспроизведения.

---

<p align="center">
  <strong>SupplyFlow</strong> — прозрачная цепочка поставок от заявки до аналитики.<br>
  <sub>Сделано для команд, которым важны контроль, скорость и измеримый результат.</sub>
</p>
