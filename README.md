# 🕊️ Мессенджер «Ласточка»

Защищённый мессенджер с открытым исходным кодом, построенный на базе **Tinode** (Go-сервер) с кастомным фронтендом.

## Возможности

- **Чаты и группы** — личная переписка, групповые чаты, каналы
- **Медиа** — отправка изображений и голосовых сообщений
- **Поиск пользователей** — по логину, email, телефону
- **Тёмная тема** — автоматическое переключение
- **Офлайн-индикатор** — мониторинг подключения с автопереподключением
- **Reply, edit, delete, forward** — полноценное управление сообщениями
- **Typing indicators** — индикаторы набора в реальном времени
- **Модерация** — бан, мут, пин, управление участниками групп

## Платформы

| Платформа | Путь | Статус |
|-----------|------|--------|
| **Web** | `lastochka-ui/` | ✅ Работает |
| **Android** | `lastochka-android-compose/` | ✅ Работает |
| **iOS** | `lastochka-ios/` | ⏳ В разработке |
| **Сервер** | `lastochka-server/` | ✅ Работает |

## Структура репозитория

```
dev/
├── lastochka-ui/               # Web-клиент (React 18 + TypeScript + Tailwind + Zustand)
├── lastochka-android-compose/  # Android-клиент (Kotlin + Jetpack Compose + Hilt + Room)
├── lastochka-ios/              # iOS-клиент (Swift, форк Tinode)
├── lastochka-server/           # Tinode-сервер (Go, форк)
└── README.md                   # Этот файл
```

## Быстрый старт

### Web-клиент

```bash
cd lastochka-ui
npm install
cp .env.example .env.local   # при необходимости
npm run dev                  # http://localhost:5173
npm run build                # production-сборка
```

### Android-клиент

```bash
cd lastochka-android-compose
./gradlew assembleDebug      # сборка APK
./gradlew test               # запуск тестов
```

### Сервер

```bash
cd lastochka-server
go build -o tinode .
./tinode -config tinode.yml
```

### Docker (полный стек)

Полная инструкция по развёртыванию — в [PROJECT_CONTEXT.md](../PROJECT_CONTEXT.md) и [instructions/deploy.md](../instructions/deploy.md).

## Стек технологий

### Web

| Технология | Назначение |
|------------|------------|
| React 18 + TypeScript | UI + типизация |
| Zustand | State management |
| Tailwind CSS | Стилизация |
| Vite | Сборщик |
| tinode-sdk | WebSocket-клиент Tinode |
| lucide-react | Иконки |

### Android

| Технология | Назначение |
|------------|------------|
| Kotlin + Jetpack Compose | UI |
| Hilt | Dependency injection |
| Room | Локальная БД |
| OkHttp | WebSocket + HTTP |
| Coroutines + Flow | Асинхронность |

### Сервер

| Технология | Назначение |
|------------|------------|
| Go 1.21+ | Язык сервера |
| PostgreSQL 16 | Основная БД |
| Redis 7 | Кэш и pub/sub |
| MinIO | S3-совместимое хранилище |

## Архитектура

```
┌─────────────────┐     ┌──────────────────┐
│   Web (React)   │     │  Android (Kotlin)│
└────────┬────────┘     └────────┬─────────┘
         │ WebSocket (wss://)     │
         ▼                        ▼
┌────────────────────────────────────────┐
│       Tinode Server (Go, порт 6060)    │
│  • Auth  • Messaging  • Push  • Media  │
└────────────┬───────────────────────────┘
             │
    ┌────────┼────────┐
    ▼        ▼        ▼
 PostgreSQL  Redis   MinIO (S3)
```

Пользователь подключается через WebSocket → Tinode обрабатывает аутентификацию, сообщения, присутствие → медиа хранятся в MinIO → данные пользователей и сообщения в PostgreSQL → кэш в Redis.

## Лицензия

Серверная часть (форк Tinode) распространяется под **GPL v3**. Клиентские компоненты — см. лицензию в каждом подпроекте.

Подробности — файл [LICENSE](./LICENSE) в корне этого репозитория.

## Участие

Мы приветствуем контрибьюторов! Смотрите [CONTRIBUTING.md](../CONTRIBUTING.md) в корневой директории проекта.

### С чего начать

1. Прочитайте этот README и [CONTRIBUTING.md](../CONTRIBUTING.md)
2. Посмотрите открытые **Issues** — выберите задачу по силам
3. Напишите в комментарии к задаче: «Беру в работу»
4. Создайте ветку от `develop`, работайте, отправляйте MR

## Связь

- **Issues** — баг-репорты, фич-реквесты
- **Discussions** — идеи, обсуждения архитектуры
- **Email** — вопросы к мейнтейнерам
