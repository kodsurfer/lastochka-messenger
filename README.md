# Ласточка — Мессенджер с открытым исходным кодом

Защищённый мессенджер с Telegram-like интерфейсом, построенный на базе [Tinode](https://github.com/tinode/chat) (Go-сервер).  
Серверы в РФ. Compliance с ФЗ-152.

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Platform](https://img.shields.io/badge/platforms-Web%20%7C%20Android%20%7C%20Desktop%20%7C%20iOS-lightgrey)](docs/ARCHITECTURE.md)

---

## Содержание

- [О проекте](#о-проекте)
- [Статус компонентов](#статус-компонентов)
- [Архитектура](#архитектура)
- [Структура репозитория](#структура-репозитория)
- [Быстрый старт](#быстрый-старт)
- [Для контрибьюторов](#для-контрибьюторов)
- [Прозрачность и безопасность](#прозрачность-и-безопасность)
- [Лицензии](#лицензии)

---

## О проекте

**Ласточка** — это опенсорс-мессенджер. Мы не скрываем, что стоит под капотом.

Проект начат одним человеком и сейчас в стадии активной разработки. Фронтенд и Android написаны с нуля; бэкенд — форк Tinode с нашими патчами. Compliance-сервис — отдельный Go-бинарник для работы с ФЗ-152 (не является частью GPL-кода).

Мы открыли код по двум причинам:
1. Хотим, чтобы пользователи могли убедиться: нет скрытых закладок, нет сбора лишних данных.
2. Хотим привлечь разработчиков, которые разделяют ценности приватности и хотят строить что-то реальное.

---

## Статус компонентов

| Компонент | Стек | Путь | Статус |
|-----------|------|------|--------|
| **Web-клиент** | React 18 + TypeScript + Zustand + Tailwind | `lastochka-ui/` | ✅ Рабочий |
| **Android** | Kotlin + Jetpack Compose + Hilt + Room | `lastochka-android/` | ✅ Рабочий |
| **Desktop** | Electron + React + TypeScript | `lastochka-desktop/` | ✅ Сборка Windows |
| **Сервер (Tinode)** | Go (форк), PostgreSQL, Redis, MinIO | `lastochka-server/` | ✅ Docker |
| **iOS** | Swift (форк Tinode iOS) | `lastochka-ios/` | ⏳ В разработке |

---

## Архитектура

```
  [Web]   [Android]   [Desktop]   [iOS]
    │          │           │         │
    └──────────┴───────────┴─────────┘
                     │
              wss:// (TLS 1.3)
                     │
              ┌──────▼──────┐
              │    Nginx     │  TLS-терминация, rate limit
              └──────┬──────┘
                     │
         ┌───────────┴───────────┐
         │                       │
  ┌──────▼──────┐       ┌────────▼────────┐
  │  Tinode     │       │  Compliance     │
  │  (Go, форк) │       │  Service (Go)   │
  │  WebSocket  │       │  (не GPL, ФЗ-152)│
  └──────┬──────┘       └────────┬────────┘
         │                       │
    ┌────┴────┐             ┌────┘
    │         │             │
 PostgreSQL  Redis  MinIO   ComplianceDB
```

Полная архитектура, схемы данных, принятые решения — в [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

---

## Структура репозитория

```
lastochka/
├── lastochka-ui/           # Web-клиент (React 18 + TypeScript + Vite)
├── lastochka-android/      # Android (Kotlin + Jetpack Compose)
├── lastochka-desktop/      # Desktop (Electron + React)
├── lastochka-ios/          # iOS (Swift, форк tinode-ios)
├── lastochka-server/       # Tinode-сервер (Go, форк tinode/chat)
├── docs/
│   └── ARCHITECTURE.md     # Подробная архитектура для аудиторов
├── CONTRIBUTING.md         # Как участвовать в разработке
├── SECURITY.md             # Политика безопасности и раскрытия уязвимостей
├── CODE_OF_CONDUCT.md      # Правила поведения в сообществе
└── LICENSE                 # GPL v3
```

---

## Быстрый старт

### Web-клиент

```bash
cd lastochka-ui
npm install
npm run dev        # http://localhost:5173
npm run build      # production-сборка
```

По умолчанию клиент подключается к `wss://app.lastochka-m.ru`. Для локальной разработки:

```bash
cp .env.example .env.local
# VITE_TINODE_HOST=localhost:6060
# VITE_TINODE_SECURE=false
```

### Android

```bash
cd lastochka-android
# Открыть в Android Studio → Sync Gradle → Run
./gradlew assembleDebug   # APK: app/build/outputs/apk/debug/
./gradlew test            # unit-тесты
```

### Desktop (Electron)

```bash
cd lastochka-desktop
npm install
npm run dev:electron      # dev-режим
npm run build:electron    # сборка installer
```

### Сервер (локально через Docker)

```bash
cd lastochka-server
docker compose -f docker/docker-compose.dev.yml up -d
curl http://localhost:6060/v0/health  # должен вернуть 200
```

---

## Для контрибьюторов

Прочитайте [`CONTRIBUTING.md`](CONTRIBUTING.md) — там подробно о процессе работы, ветвлении, ревью и принципах проекта.

Вкратце:
1. Выберите задачу в **Issues** (метки: `good first issue`, `help wanted`)
2. Напишите в задаче «Беру в работу»
3. Создайте ветку от `develop`: `feature/название` или `fix/название`
4. Откройте PR с описанием: что сделано, как проверить, скриншоты (для UI)
5. Дождитесь ревью

Нет подходящей задачи — создайте Discussion с предложением.

---

## Прозрачность и безопасность

Этот раздел для тех, кто проверяет код на открытость и отсутствие скрытых функций.

### Что сервер получает от клиента

- Учётные данные (логин/пароль) при авторизации — не хранятся в открытом виде
- Сообщения — передаются по WebSocket, хранятся в PostgreSQL на сервере
- IP-адреса — логируются Nginx (стандартное поведение)
- Push-токены — при включении FCM-уведомлений на Android

Клиент **не** отправляет: контакты телефона, геолокацию, метаданные устройства (кроме стандартных HTTP-заголовков).

### Compliance-сервис

Отдельный Go-бинарник (`compliance/`, не в этом репозитории, не GPL). Задача: выполнение требований российского законодательства (ФЗ-152). Хранит:
- Audit-лог действий (append-only)
- Сессионные данные для государственных запросов

Исходный код compliance-сервиса закрыт по юридическим причинам. Это зафиксировано честно, а не скрыто.

### Зависимости

| Компонент | Основные зависимости |
|-----------|---------------------|
| Сервер | [tinode/chat](https://github.com/tinode/chat) (GPL v3), PostgreSQL, Redis |
| Web | tinode-sdk (npm), React, Zustand, Tailwind CSS |
| Android | tinodesdk (jar), Jetpack Compose, Hilt, Room, OkHttp |
| Desktop | Electron, tinode-sdk |

Все зависимости — публичные библиотеки с открытым кодом. Нет проприетарных SDK (кроме FCM для Push, по выбору пользователя).

### Политика безопасности

Уязвимости — в [`SECURITY.md`](SECURITY.md).

---

## Лицензии

| Компонент | Лицензия |
|-----------|----------|
| `lastochka-server` (форк Tinode) | [GPL v3](LICENSE) |
| `lastochka-ui` | Apache 2.0 |
| `lastochka-android` | Apache 2.0 |
| `lastochka-desktop` | Apache 2.0 |
| `lastochka-ios` | Apache 2.0 |
| Документация (`docs/`) | CC BY-SA 4.0 |

Compliance-сервис (`compliance/`) — отдельный проект, не входит в этот репозиторий.

---

## Связь

- **Баги и фичи**: [GitHub Issues](../../issues)
- **Обсуждения**: [GitHub Discussions](../../discussions)
- **Уязвимости**: см. [SECURITY.md](SECURITY.md)
- **Сервер**: `app.lastochka-m.ru`
