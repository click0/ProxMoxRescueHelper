# ProxRescue

[English](README.md) | Русский | [Українська](README_UK.md)

Установщик продуктов Proxmox в режиме восстановления (Rescue Mode) для Hetzner

Описание

Этот скрипт предназначен для установки продуктов Proxmox (Proxmox Virtual Environment, Proxmox Backup Server, Proxmox Mail Gateway, Proxmox Datacenter Manager) в режиме восстановления на серверах Hetzner. Он позволяет выбрать продукт для установки, настроить параметры подключения VNC, выбрать режим загрузки UEFI или Legacy BIOS и применить распространённые пост-установочные оптимизации. Кроме того, скрипт может запускать установленную систему Proxmox, позволяя подключиться через VNC или noVNC.

Быстрый старт

Запуск напрямую в rescue-системе Hetzner:

    bash -c "$(curl -fsSL https://raw.githubusercontent.com/Proxmoxinfo/ProxMoxRescueHelper/refs/heads/main/ProxRescue.sh)"

Чтобы передать флаги командной строки в этом варианте запуска, добавьте `_` в качестве заглушки для `$0`:

    bash -c "$(curl -fsSL https://raw.githubusercontent.com/Proxmoxinfo/ProxMoxRescueHelper/refs/heads/main/ProxRescue.sh)" _ -pve -auto -dns 8.8.8.8

Либо скачайте скрипт один раз и запускайте его с нужными флагами:

    curl -fsSL -o ProxRescue.sh https://raw.githubusercontent.com/Proxmoxinfo/ProxMoxRescueHelper/refs/heads/main/ProxRescue.sh && chmod +x ProxRescue.sh
    ./ProxRescue.sh -pve -auto -dns 8.8.8.8

Требования

Перед запуском скрипта убедитесь, что в системе установлены следующие пакеты:

    curl
    sshpass
    dialog
    git

Установка необходимых пакетов

Если необходимые пакеты не установлены, скрипт попытается установить их автоматически.

Использование

Запустите скрипт с соответствующими параметрами для установки выбранного продукта Proxmox или настройки системы:

Параметры командной строки

Установка:

    -pve: Установить Proxmox Virtual Environment.
    -pbs: Установить Proxmox Backup Server.
    -pmg: Установить Proxmox Mail Gateway.
    -pdm: Установить Proxmox Datacenter Manager.

Пост-установка (применяется автоматически после установки):

    -fix-sources: Исправить базовые источники Debian (deb.debian.org).
    -no-sub: Переключить Enterprise-репозитории на no-subscription и убрать напоминание о подписке.
    -upgrade: Выполнить apt update + dist-upgrade (требует -no-sub).
    -disable-ha: Отключить службы HA (только для single-node PVE).
    -auto: Применить все вышеуказанные пост-установочные оптимизации без подтверждения.

Подключение:

    -p, --password PASSWORD: Указать пароль для VNC-подключения.
    -vport PORT: Указать порт для noVNC (по умолчанию 8080).
    -dns DNS_SERVER[,DNS_SERVER...]: Указать один или несколько DNS-серверов через запятую (по умолчанию: автоопределение из /etc/resolv.conf rescue-системы, fallback 1.1.1.1).
    -uefi: Принудительно использовать режим загрузки UEFI.
    -legacy: Принудительно использовать режим загрузки Legacy BIOS.

Прочее:

    -h, --help: Показать справку и выйти.

Если не указаны ни -uefi, ни -legacy, режим загрузки определяется автоматически на основе прошивки rescue-системы.

Если -dns не указан, DNS-сервер определяется автоматически из /etc/resolv.conf rescue-системы (с резервным значением 1.1.1.1, если определить не удалось).

Примеры

    Установить Proxmox Virtual Environment с указанным паролем VNC:

       ./ProxRescue.sh -pve -p yourVNCpassword

    Установить Proxmox VE со всеми пост-установочными оптимизациями и указанным DNS-сервером:

       ./ProxRescue.sh -pve -auto -dns 8.8.8.8

    Установить Proxmox Backup Server и переключиться на no-subscription репозитории с полным обновлением:

       ./ProxRescue.sh -pbs -no-sub -upgrade

    Установить Proxmox VE с несколькими отдельными пост-установочными исправлениями:

       ./ProxRescue.sh -pve -fix-sources -no-sub -upgrade -disable-ha

Главное меню

При запуске скрипта без параметров отображается главное меню:

    1. Выбор дисков для QEMU
    2. Установка Proxmox (PVE, PBS, PMG, PDM)
    3. Запуск установленной системы в QEMU
    4. Переключение режима загрузки (UEFI / Legacy BIOS)
    5. Изменить пароль VNC
    6. Изменить DNS-сервер(ы)
    7. Перезагрузка
    8. Выход

Текущий режим загрузки (автоопределённый или установленный вручную) отображается в верхней части меню.

Возможности

    Проверка обновлений (Self-Update):
        Скрипт показывает текущую версию при запуске и в --help.
        При запуске он проверяет репозиторий GitHub на наличие новой версии и, если она доступна,
        предлагает скачать её и автоматически перезапуститься с теми же аргументами.

    Автоматическая установка продуктов Proxmox:
        Выбор из Proxmox Virtual Environment, Proxmox Backup Server, Proxmox Mail Gateway или Proxmox Datacenter Manager.
        Автоматическая загрузка последней версии выбранного продукта или выбор более старой версии из списка.
        Загрузка ISO по HTTPS (с резервным переходом на HTTP) и проверка контрольной суммы SHA256.

    Пост-установочные оптимизации:
        Исправление базовых источников Debian на deb.debian.org.
        Переключение Enterprise-репозиториев на канал no-subscription и удаление напоминания о подписке (веб- и мобильный интерфейс).
        Выполнение apt update + dist-upgrade.
        Отключение служб High Availability на single-node установках PVE.
        Каждую оптимизацию можно применить интерактивно, через отдельные флаги или все сразу с помощью -auto.

    Настройка VNC:
        Установка собственного пароля VNC для безопасного доступа или его изменение позже из меню.
        Указание порта noVNC во избежание конфликтов с существующими сервисами.

    Режим загрузки (UEFI / Legacy BIOS):
        Автоопределение на основе прошивки rescue-системы.
        Можно принудительно задать через флаги -uefi / -legacy или переключить интерактивно из меню.

    Настройка DNS:
        DNS-сервер(ы) автоматически определяются из /etc/resolv.conf rescue-системы (все валидные записи nameserver, при этом stub-резолвер systemd-resolved 127.0.0.53 разрешается через /run/systemd/resolve/resolv.conf), с резервным значением 1.1.1.1, если ничего не удалось определить.
        Можно переопределить через флаг -dns с одним или несколькими IP через запятую (например, -dns 8.8.8.8,1.1.1.1), с проверкой и резервным значением 1.0.0.1, если не указан ни один валидный IP.
        Все определённые/указанные DNS-серверы записываются в /etc/resolv.conf на установленной системе.

    Настройка сети:
        Автоматическое определение и настройка сетевых параметров (мост vmbr0).
        Введите пароль root, заданный при установке, чтобы применить сетевую конфигурацию к установленной системе Proxmox.

    Управление перезагрузкой:
        Возможность перезагрузить сервер после установки или изменения конфигурации.
        Гарантированное корректное завершение работы QEMU и noVNC перед перезагрузкой.

    Интеграция NoVNC:
        Автоматическая настройка и запуск noVNC для веб-доступа по VNC.
        Корректная остановка сессий noVNC, когда они больше не нужны.

    Выбор дисков:
        Ручной выбор дисков для передачи в QEMU через интерактивный диалог.

    Интерактивное меню:
        Удобный интерфейс меню для выбора параметров установки и настройки.
        Возможность запуска скрипта неинтерактивно с параметрами командной строки.

Примечания

    Скрипт автоматически завершает все сессии noVNC и отправляет команду quit монитору QEMU при выходе/запуске.
    Требуется KVM (/dev/kvm), наличие которого проверяется при запуске.
    Во время работы инсталлятора Proxmox (внутри сессии noVNC/VNC) НЕ настраивайте и не изменяйте сетевые
    параметры/IP (оставьте их по умолчанию/DHCP или как предзаполнено инсталлятором). Настройка сети
    выполняется автоматически скриптом после установки (configure_network) — изменение IP на этом этапе
    вручную нарушит последующую автоматическую настройку сети и пост-установочные шаги.

Благодарности

Часть логики пост-установочных оптимизаций (переключение репозиториев и удаление напоминания о подписке) адаптирована из [community-scripts.org](https://community-scripts.org).

Лицензия MIT

Copyright (c) 2026 Proxmox UA

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

Сообщества и поддержка

    Telegram: Proxmox_UA
    GitHub: https://github.com/Proxmoxinfo/ProxMoxRescueHelper
    Website: proxmox.info

Этот скрипт предназначен для установки продуктов Proxmox в режиме восстановления на серверах Hetzner.
