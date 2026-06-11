# ProxRescue

[English](README.md) | [Русский](README_RU.md) | Українська

Інсталятор продуктів Proxmox у режимі відновлення (Rescue Mode) для Hetzner

Опис

Цей скрипт призначений для встановлення продуктів Proxmox (Proxmox Virtual Environment, Proxmox Backup Server, Proxmox Mail Gateway, Proxmox Datacenter Manager) у режимі відновлення на серверах Hetzner. Він дозволяє вибрати продукт для встановлення, налаштувати параметри підключення VNC, обрати режим завантаження UEFI або Legacy BIOS і застосувати типові пост-інсталяційні оптимізації. Крім того, скрипт може запускати встановлену систему Proxmox, дозволяючи підключитися через VNC або noVNC.

Швидкий старт

ProxRescue — це єдиний самодостатній скрипт: клонувати репозиторій не потрібно, потрібен лише один файл.

Запуск безпосередньо в rescue-системі Hetzner:

    bash -c "$(curl -fsSL https://raw.githubusercontent.com/Proxmoxinfo/ProxMoxRescueHelper/refs/heads/main/ProxRescue.sh)"

Щоб передати прапорці командного рядка в цьому варіанті запуску, додайте `_` як заглушку для `$0`:

    bash -c "$(curl -fsSL https://raw.githubusercontent.com/Proxmoxinfo/ProxMoxRescueHelper/refs/heads/main/ProxRescue.sh)" _ -pve -auto -dns 8.8.8.8

Або завантажте скрипт один раз і запускайте його з потрібними прапорцями:

    curl -fsSL -o ProxRescue.sh https://raw.githubusercontent.com/Proxmoxinfo/ProxMoxRescueHelper/refs/heads/main/ProxRescue.sh && chmod +x ProxRescue.sh
    ./ProxRescue.sh -pve -auto -dns 8.8.8.8

Вимоги

Перед запуском скрипта переконайтеся, що в системі встановлені такі пакунки:

    curl
    sshpass
    dialog
    git

Встановлення необхідних пакунків

Якщо необхідні пакунки не встановлені, скрипт спробує встановити їх автоматично.

Використання

Запустіть скрипт із відповідними параметрами для встановлення обраного продукту Proxmox або налаштування системи:

Параметри командного рядка

Встановлення:

    -pve: Встановити Proxmox Virtual Environment.
    -pbs: Встановити Proxmox Backup Server.
    -pmg: Встановити Proxmox Mail Gateway.
    -pdm: Встановити Proxmox Datacenter Manager.

Пост-інсталяція (застосовується автоматично після встановлення):

    -fix-sources: Виправити базові джерела Debian (deb.debian.org).
    -no-sub: Перемкнути Enterprise-репозиторії на no-subscription і прибрати нагадування про підписку.
    -upgrade: Виконати apt update + dist-upgrade (потребує -no-sub).
    -disable-ha: Вимкнути служби HA (лише для single-node PVE).
    -auto: Застосувати всі вищезазначені пост-інсталяційні оптимізації без підтвердження.

Підключення:

    -p, --password PASSWORD: Вказати пароль для VNC-підключення.
    -vport PORT: Вказати порт для noVNC (за замовчуванням 8080).
    -dns DNS_SERVER[,DNS_SERVER...]: Вказати один або кілька DNS-серверів через кому (за замовчуванням: автовизначення з /etc/resolv.conf rescue-системи, fallback 1.1.1.1).
    -uefi: Примусово використовувати режим завантаження UEFI.
    -legacy: Примусово використовувати режим завантаження Legacy BIOS.

Інше:

    -h, --help: Показати довідку та вийти.

Якщо не вказано ні -uefi, ні -legacy, режим завантаження визначається автоматично на основі прошивки rescue-системи.

Якщо -dns не вказано, DNS-сервер визначається автоматично з /etc/resolv.conf rescue-системи (з резервним значенням 1.1.1.1, якщо визначити не вдалося).

Приклади

    Встановити Proxmox Virtual Environment із зазначеним паролем VNC:

       ./ProxRescue.sh -pve -p yourVNCpassword

    Встановити Proxmox VE з усіма пост-інсталяційними оптимізаціями та власним DNS-сервером:

       ./ProxRescue.sh -pve -auto -dns 8.8.8.8

    Встановити Proxmox Backup Server і перемкнутися на no-subscription репозиторії з повним оновленням:

       ./ProxRescue.sh -pbs -no-sub -upgrade

    Встановити Proxmox VE з кількома окремими пост-інсталяційними виправленнями:

       ./ProxRescue.sh -pve -fix-sources -no-sub -upgrade -disable-ha

Головне меню

При запуску скрипта без параметрів відображається головне меню:

    1. Вибір дисків для QEMU
    2. Встановлення Proxmox (PVE, PBS, PMG, PDM)
    3. Запуск встановленої системи в QEMU
    4. Перемикання режиму завантаження (UEFI / Legacy BIOS)
    5. Змінити пароль VNC
    6. Змінити DNS-сервер(и)
    7. Перезавантаження
    8. Вихід

Поточний режим завантаження (автовизначений або встановлений вручну) відображається у верхній частині меню.

Можливості

    Перевірка оновлень (Self-Update):
        Скрипт показує поточну версію під час запуску та в --help.
        Під час запуску він перевіряє репозиторій GitHub на наявність нової версії і, якщо вона доступна,
        пропонує завантажити її та автоматично перезапуститися з тими самими аргументами.

    Автоматичне встановлення продуктів Proxmox:
        Вибір з Proxmox Virtual Environment, Proxmox Backup Server, Proxmox Mail Gateway або Proxmox Datacenter Manager.
        Автоматичне завантаження останньої версії обраного продукту або вибір старішої версії зі списку.
        Завантаження ISO через HTTPS (з резервним переходом на HTTP) і перевірка контрольної суми SHA256.

    Пост-інсталяційні оптимізації:
        Виправлення базових джерел Debian на deb.debian.org.
        Перемикання Enterprise-репозиторіїв на канал no-subscription та видалення нагадування про підписку (веб- та мобільний інтерфейс).
        Виконання apt update + dist-upgrade.
        Вимкнення служб High Availability на single-node інсталяціях PVE.
        Кожну оптимізацію можна застосувати інтерактивно, через окремі прапорці або всі одразу за допомогою -auto.

    Налаштування VNC:
        Встановлення власного пароля VNC для безпечного доступу або зміна його пізніше з меню.
        Вказання порту noVNC, щоб уникнути конфліктів з наявними сервісами.

    Режим завантаження (UEFI / Legacy BIOS):
        Автовизначення на основі прошивки rescue-системи.
        Можна примусово задати через прапорці -uefi / -legacy або перемкнути інтерактивно з меню.

    Налаштування DNS:
        DNS-сервер(и) автоматично визначаються з /etc/resolv.conf rescue-системи (усі валідні записи nameserver, при цьому stub-резолвер systemd-resolved 127.0.0.53 розв'язується через /run/systemd/resolve/resolv.conf), з резервним значенням 1.1.1.1, якщо нічого не вдалося визначити.
        Можна перевизначити через прапорець -dns з одним або кількома IP через кому (наприклад, -dns 8.8.8.8,1.1.1.1), з перевіркою та резервним значенням 1.0.0.1, якщо не вказано жодного валідного IP.
        Усі визначені/вказані DNS-сервери записуються в /etc/resolv.conf на встановленій системі.

    Налаштування мережі:
        Автоматичне визначення та налаштування мережевих параметрів (міст vmbr0).
        Введіть пароль root, заданий під час встановлення, щоб застосувати мережеву конфігурацію до встановленої системи Proxmox.

    Керування перезавантаженням:
        Можливість перезавантажити сервер після встановлення або зміни конфігурації.
        Гарантоване коректне завершення роботи QEMU та noVNC перед перезавантаженням.

    Інтеграція NoVNC:
        Автоматичне налаштування та запуск noVNC для веб-доступу через VNC.
        Коректна зупинка сесій noVNC, коли вони більше не потрібні.

    Вибір дисків:
        Ручний вибір дисків для передачі в QEMU через інтерактивний діалог.

    Інтерактивне меню:
        Зручний інтерфейс меню для вибору параметрів встановлення та налаштування.
        Можливість запуску скрипта неінтерактивно з параметрами командного рядка.

Примітки

    Скрипт автоматично завершує всі сесії noVNC та надсилає команду quit монітору QEMU під час виходу/запуску.
    Потрібен KVM (/dev/kvm), наявність якого перевіряється під час запуску.
    Під час роботи інсталятора Proxmox (всередині сесії noVNC/VNC) НЕ налаштовуйте і не змінюйте мережеві
    параметри/IP (залиште їх за замовчуванням/DHCP або як попередньо заповнено інсталятором). Налаштування
    мережі виконується автоматично скриптом після встановлення (configure_network) — зміна IP вручну на
    цьому етапі порушить подальше автоматичне налаштування мережі та пост-інсталяційні кроки.

Подяки

Частина логіки пост-інсталяційних оптимізацій (перемикання репозиторіїв та видалення нагадування про підписку) адаптована з [community-scripts.org](https://community-scripts.org).

Ліцензія MIT

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

Спільноти та підтримка

    Telegram: Proxmox_UA
    GitHub: https://github.com/Proxmoxinfo/ProxMoxRescueHelper
    Website: proxmox.info

Цей скрипт призначений для встановлення продуктів Proxmox у режимі відновлення на серверах Hetzner.
