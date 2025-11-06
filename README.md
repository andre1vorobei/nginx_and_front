# Микросервисная архитектура - Социальная сеть

Пет-проект микросервисной архитектуры для социальной сети с аутентификацией и управлением профилями пользователей.


## Примечание

Фронтенд часть, вынесенную в сабмодуль, разрабатывал другой человек, можете зайти на его github и оценить другие его работы ;)

## Архитектура

Проект состоит из следующих компонентов:

- **Nginx** - Reverse proxy и API Gateway, обеспечивающий единую точку входа для всех сервисов
- **Frontend Service** - Next.js приложение для пользовательского интерфейса
- **Auth Service** - Go REST API для аутентификации и регистрации пользователей
- **Profile Service** - Go REST API для управления профилями пользователей

### Схема взаимодействия

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │ HTTPS (443)
       │
┌──────▼──────────────────────────────────────┐
│           Nginx (API Gateway)               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │   /      │  │/api/auth │  │/api/prof │  │
│  │          │  │          │  │  iles    │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  │
└───────┼─────────────┼─────────────┼────────┘
        │             │             │
┌───────▼─────┐ ┌─────▼─────┐ ┌─────▼──────┐
│  Frontend   │ │   Auth    │ │  Profile   │
│  Service    │ │  Service  │ │  Service   │
│  (Next.js)  │ │  (Go)     │ │  (Go)      │
│  :3000      │ │  :8080    │ │  :8081     │
└─────────────┘ └───────────┘ └────────────┘
```

## Технологический стек

### Инфраструктура
- **Docker & Docker Compose** - контейнеризация и оркестрация сервисов
- **Nginx** - reverse proxy, SSL/TLS терминация, маршрутизация запросов
- **Docker Network** - изолированная сеть для межсервисного взаимодействия

### Frontend
- **Next.js 15.3.1** - React фреймворк
- **TypeScript** - типизация
- **Tailwind CSS** - стилизация
- **React 19** - UI библиотека

### Backend Services
- **Go 1.23.1** - основной язык для микросервисов
- **Gorilla Mux** - HTTP роутинг
- **PostgreSQL** - база данных (для каждого сервиса отдельная БД)
- **JWT** - токены для аутентификации
- **Bcrypt** - хеширование паролей
- **Flyway** - миграции БД
- **Logrus** - структурированное логирование

## Структура проекта

```
nginx_and_front/
├── docker-compose.yml          # Оркестрация Nginx и Frontend
├── conf.d/
│   └── default.conf            # Конфигурация Nginx
├── certs/                      # SSL/TLS сертификаты
│   ├── fullchain.crt
│   └── abc.com.key
└── social/
    └── frontend/               # Next.js приложение (сабмодуль)
        ├── src/
        ├── public/
        └── package.json
```

**Примечание:** Auth Service и Profile Service находятся в отдельных репозиториях и должны быть запущены в той же Docker сети `my-app-network`.

## Установка и запуск

### Предварительные требования

- Docker & Docker Compose
- Go 1.23+ (для локальной разработки backend сервисов)
- Node.js 20+ (для локальной разработки frontend)
- SSL сертификаты (для production)

### Настройка

1. **Клонируйте репозитории:**

```bash
git clone <repository-url>
cd nginx_and_front
git submodule update --init --recursive
```

2. **Подготовьте SSL сертификаты:**

Поместите SSL сертификаты в директорию `certs/`:
- `fullchain.crt` - полная цепочка сертификатов
- `abc.com.key` - приватный ключ

**Для разработки** можно использовать самоподписанные сертификаты или отключить SSL.

3. **Создайте Docker сеть:**

```bash
docker network create my-app-network --subnet=172.20.0.0/16
```

4. **Настройте Auth Service и Profile Service:**

Убедитесь, что оба сервиса настроены и готовы к запуску:
- Auth Service должен быть доступен как `auth_service:8080` в Docker сети
- Profile Service должен быть доступен как `profiles_service:8081` в Docker сети

**Важно:** `JWTSKEY` в обоих сервисах должен совпадать для корректной работы JWT токенов.

### Запуск через Docker Compose

1. **Запустите Auth Service и Profile Service** в той же Docker сети `my-app-network`

2. **Запустите Nginx и Frontend:**

```bash
docker-compose up -d
```

Это запустит:
- Nginx на портах 80 (HTTP) и 443 (HTTPS)
- Frontend Service на порту 3000 (внутренний)

3. **Проверьте работу:**

- Frontend: `https://your-domain.com` или `https://158.160.71.16`
- Auth API: `https://your-domain.com/api/auth/`
- Profiles API: `https://your-domain.com/api/profiles`

### Локальная разработка

#### Frontend

```bash
cd social/frontend
npm install
npm run dev
```

Frontend будет доступен на `http://localhost:3000`

**Примечание:** Для локальной разработки может потребоваться изменить константы в `src/utils/const.ts` для подключения к локальным сервисам.

#### Backend Services

См. документацию в соответствующих репозиториях:
- [Auth Service README](#) - ссылка на репозиторий Auth Service
- [Profile Service README](#) - ссылка на репозиторий Profile Service

## Конфигурация Nginx

Nginx настроен как API Gateway и выполняет следующие функции:

1. **SSL/TLS терминация** - обработка HTTPS соединений
2. **Маршрутизация запросов:**
   - `/api/auth/*` → `auth_service:8080`
   - `/api/profiles/*` → `profiles_service:8081`
   - `/` → `frontend_service:3000`
3. **HTTP → HTTPS редирект** - автоматическое перенаправление HTTP на HTTPS

Конфигурация находится в `conf.d/default.conf`.

## API Endpoints

Все API endpoints доступны через Nginx на домене с префиксом `/api/`.

### Auth Service (`/api/auth/`)

- `POST /api/auth/register` - регистрация нового пользователя
- `POST /api/auth/login` - авторизация пользователя
- `GET /api/auth/hello` - тестовый endpoint

Подробная документация: см. README Auth Service

### Profile Service (`/api/profiles/`)

- `GET /api/profiles/me` - получить свой профиль
- `GET /api/profiles/{username}` - получить профиль пользователя
- `GET /api/profiles?search={pattern}` - поиск профилей
- `POST /api/profiles/crprofile` - создать профиль
- `POST /api/profiles/subscribe/{username}` - подписаться на пользователя
- `POST /api/profiles/unsubscribe/{username}` - отписаться от пользователя
- `GET /api/profiles/followers/{username}` - получить подписчиков
- `GET /api/profiles/followed/{username}` - получить подписки

Подробная документация: см. README Profile Service

**Все endpoints Profile Service требуют JWT токен в заголовке `Authorization: Bearer <token>`.**

## Сетевая архитектура

Все сервисы работают в Docker сети `my-app-network`:

- **Nginx**: `172.20.0.100`
- **Frontend Service**: `172.20.0.101`
- **Auth Service**: должен быть в той же сети (имя контейнера: `auth_service`)
- **Profile Service**: должен быть в той же сети (имя контейнера: `profiles_service`)

## Безопасность

- **SSL/TLS** - все соединения шифруются через HTTPS
- **JWT токены** - аутентификация через JWT с общим секретным ключом
- **Bcrypt** - пароли хешируются перед сохранением
- **CORS** - настроен в backend сервисах
- **Reverse Proxy** - Nginx скрывает внутреннюю структуру сервисов

## Интеграция сервисов

### Поток аутентификации

1. Пользователь регистрируется/авторизуется через Frontend
2. Frontend отправляет запрос на `/api/auth/register` или `/api/auth/login`
3. Nginx проксирует запрос на Auth Service
4. Auth Service возвращает JWT токен
5. Frontend сохраняет токен и использует его для последующих запросов

### Поток работы с профилями

1. Пользователь запрашивает профиль через Frontend
2. Frontend отправляет запрос на `/api/profiles/*` с JWT токеном
3. Nginx проксирует запрос на Profile Service
4. Profile Service валидирует JWT токен (используя общий `JWTSKEY`)
5. Profile Service возвращает данные профиля

## Лицензия

Пет-проект для обучения и портфолио.




