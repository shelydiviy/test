🎮 CS 1.6 Mirror Server
Версия: 1.1.22
Язык: Go
Лицензия: Non-Commercial Use Only
Статус: 🟢 Активная разработка
📋 Описание
CS 1.6 Mirror Server — это эмулятор сервера Counter-Strike 1.6, который создаёт зеркальные копии игровых серверов с полной интеграцией Steam API. Проект позволяет запускать множественные виртуальные серверы с фейковыми игроками, A2S-запросами и UDP-проксированием трафика.
🔑 Ключевые возможности
Функция
Описание
🖥️ Зеркалирование
Создание множественных серверов с разными настройками
👤 Фейковые игроки
Боты с реальной Steam-авторизацией
📡 A2S Protocol
Полная поддержка Server Query (INFO, PLAYER, RULES)
🔀 UDP Proxy
Прозрачное перенаправление игрового трафика
🔐 Steam Auth
Интеграция с Steam Game Server API
⚙️ Конфигурация
Гибкая настройка через JSON
🗂️ Структура проекта
1234567891011121314151617181920212223242526272829303132333435363738394041424344
mirror_o/
├── 📄 main.go                    # Точка входа, оркестрация всех компонентов
├── 📄 config.json                # Конфигурация серверов
├── 📄 accounts.txt               # Аккаунты для фейковых игроков
├── 📄 tokens.txt                 # Токены авторизации
├── 📄 build.sh / build.bat       # Скрипты сборки
├── 📄 license.md                 # Лицензия
│
├── 📁 a2s/                       # A2S Server Query Protocol
│   └── server.go                 # Обработчик A2S-запросов

⚙️ Архитектура и логика работы
🔄 Общая схема
1234567891011121314151617
┌─────────────────────────────────────────────────────────────────┐
│                        CS 1.6 Mirror Server                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐       │
│  │   Steam CM   │◄──►│  Steam Client │◄──►│   Server     │       │
│  │   Servers    │    │   (WebSocket) │    │   Instance   │       │
│  └──────────────┘ 
📊 Поток данных
mermaid









Инициализация → Загрузка конфига, создание серверов
Steam Connect → Подключение к CM-серверам через WebSocket
Server Logon → Авторизация каждого сервера в Steam
A2S Listen → Запуск UDP-слушателя для Server Query
Proxy Start → Запуск UDP-прокси для игрового трафика
Fake Players → Авторизация ботов и подключение к серверам
Heartbeat → Поддержание соединения с Steam
📁 Описание файлов
🔹 Корневые файлы
Файл
Размер
Описание
main.go
19.5 KB
Точка входа. Оркестрация серверов, прокси, A2S, фейковых игроков. Управление жизненным циклом через context.
config.json
7.9 KB
Конфигурация. Список серверов с настройками: порт, hostname, карта, игроки, регион, VAC.
accounts.txt
0 B
Аккаунты. Логин:пароль для фейковых игроков (Steam авторизация).
tokens.txt
0 B
Токены. Кэш токенов авторизации для ускорения подключения.
build.sh / build.bat
~100 B
Сборка. Скрипты компиляции для Linux/Windows.
license.md
627 B
Лицензия. Non-Commercial Use Only (Wirstaff.inc, 2025).
🔹 Модуль A2S (a2s/)
Файл
Размер
Описание
server.go
13.5 KB
A2S Server. UDP-сервер для обработки Server Query запросов (INFO, PLAYER, RULES). Реализует Challenge-механизм для защиты от спуфинга.
🔹 Модуль Player (player/)
Файл
Размер
Описание
player.go
6.4 KB
Player Class. Логика фейкового игрока: логин в Steam, получение App Ownership Ticket, генерация Auth Session Ticket.
listeners.go
4.5 KB
Event Handlers. Обработка ответов Steam: LogOnResponse, GameConnectTokens, AppOwnershipTicket.
appticket/ticket.go
1.5 KB
Ticket Parser. Парсинг бинарной структуры App Ownership Ticket (SteamID, AppID, expiration).
🔹 Модуль Server (server/)
Файл
Размер
Описание
server.go
7.1 KB
Server Instance. Представление игрового сервера: настройки, список фейковых игроков, отправка данных в Steam (GSServerType, GameServerData).
listeners.go
1.6 KB
Packet Handlers. Обработка пакетов от Steam: LogOnResponse, LoggedOff.
🔹 Модуль Steam (steam/)
Файл
Размер
Описание
client.go
9.4 KB
WebSocket Client. Подключение к Steam CM-серверам, чтение/запись пакетов, heartbeat, обработка дисконнектов.
servers.go
3.5 KB
CM Server List. Получение списка Connection Manager серверов через Steam API, ротация при ошибках.
🔹 Модуль Protocol (protocol/)
Файл
Размер
Описание
msg.go
4.6 KB
Message Types. Базовые интерфейсы сообщений (IMsg, IClientMsg), ClientMsgProtobuf, ClientMsg.
packet.go
2.9 KB
Packet Parser. Парсинг входящих пакетов, определение типа (Proto/Non-Proto).
internal.go
1.0 KB
Internal Types. JobId, Serializer интерфейсы, константы.
doc.go
1.2 KB
Documentation. Описание архитектуры протокола.
steamlang/*.go
~634 KB
Steam Constants. Enum-ы, EMsg коды, структуры сообщений (автогенерация из SteamKit).
protobuf/*.pb.go
~2.5 MB
Protobuf Messages. Автогенерированные protobuf сообщения Steam API.
gamecoordinator/*.go
~4.1 KB
GC Protocol. Сообщения для Game Coordinator (CS:GO, Dota 2).
🔹 Модуль SteamID (steamid/)
Файл
Размер
Описание
steamid.go
3.5 KB
SteamID Utils. Генерация, парсинг, конвертация SteamID (STEAM_X:Y:Z ↔ Uint64).
🔹 Модуль Redirect (redirect/)
Файл
Размер
Описание
handler.go
6.5 KB
Unified Handler. Комбинированный UDP-обработчик: A2S-запросы + редирект игрового трафика на целевой сервер.
🔹 Модуль RWU (rwu/)
Файл
Размер
Описание
rwu.go
1.9 KB
Binary Utils. Утилиты для чтения/записи бинарных данных (ReadUint32, ReadString, WriteBool).
🔹 Модуль Utils (utils/)
Файл
Размер
Описание
utils.go
501 B
Common Utils. Чтение текстовых файлов (accounts.txt, tokens.txt).
🚀 Быстрый старт
Требования
✅ Go 1.20+
✅ Linux/Windows (amd64)
✅ Доступ к Steam API (порт 27036)
Установка
bash
12345678910
# Клонирование репозитория
git clone https://github.com/yourusername/mirror_o.git
cd mirror_o

# Установка зависимостей
go mod tidy

# Сборка
./build.sh          # Linux
build.bat           # Windows
Конфигурация
Отредактируйте config.json:
json
123456789101112131415161718
{
  "start_port": 27015,
  "servers": [
    {
      "count": 1,
      "server_address": "89.223.121.95:27015",
      "players": 1,
      "max_players": 32,
      "bots": 0,
      "hostname": "[CSDM-A] Мой Сервер",

Запуск
bash
1234567891011
# Стандартный запуск
./mirrors

# С кастомным конфигом
./mirrors -config custom_config.json

# С аккаунтами для ботов
./mirrors -accounts accounts.txt

# С кастомным offset прокси

Остановка
Нажмите q + Enter в консоли для graceful shutdown.
📐 Технические детали
Порты
Тип
Порт
Описание
A2S Query
start_port + N
Server Query (UDP)
Game Proxy
start_port + N + 500
Игровой трафик (UDP)
Steam CM
27036
WebSocket подключение
Протоколы
Протокол
Порт
Назначение
A2S
UDP
Server Query (Info, Player, Rules)
Steam
WebSocket (WSS)
Авторизация, heartbeat
Game
UDP
Игровой трафик (прокси)
Steam Integration
12345
Client → WebSocket → CM Server → Auth → Game Server Registration
                          ↓
                   Heartbeat (30s)
                          ↓
            Server Update (AMGameServerUpdate)
⚠️ Важные замечания
[!WARNING]
Лицензия: Только для некоммерческого использования
Steam API: Требуется доступ к публичным CM-серверам
Порты: Убедитесь, что порты не заняты другими сервисами
Файрвол: Откройте UDP порты для A2S и игрового трафика
Боты: Аккаунты в accounts.txt должны иметь лицензию CS 1.6 (AppID: 10)
📊 Статистика проекта
Метрика
Значение
Всего файлов
49
Общий размер
4.75 MB
Язык
Go (~95%)
Protobuf
Автогенерация (~5%)
Строк кода
~15,000+
🤝 Вклад в проект
Fork репозитория
Создайте feature branch (git checkout -b feature/amazing)
Commit изменения (git commit -m 'Add amazing feature')
Push на branch (git push origin feature/amazing)
Откройте Pull Request
📞 Контакты
Автор: Wirstaff.inc
Год: 2025
Лицензия: Non-Commercial Use Only
📝 Changelog
v1.1.22 (2026-02-19)
✅ Интеграция UDP-прокси
✅ Поддержка A2S Challenge
✅ Фейковые игроки с Steam Auth
✅ Graceful shutdown
✅ Контекст-менеджмент для всех горутин
