Это альфа версия, может не работать. Обсуждение https://t.me/itdogchat - топик Podkop dev

# Выпил getdomains
По минимуму
```
rm /etc/hotplug.d/iface/30-vpnroute
sed -i '/getdomains start/d' /etc/crontabs/root
rm /tmp/dnsmasq.d/domains.lst
ip route del default scope link table vpn
```

Может потребоваться удалить правила фаервола касающиеся vpn_subnet, internal итд.

# Установка
Пакет работает на всех архитектурах. Тестировался только на 23.05.

## Вручную
Скачать пакеты podkop_*.ipk и luci-app-podkop_*.ipk. `opkg install` сначала первый, потом второй.

```
/etc/init.d/ucitrack restart
```

## Автоматическая
-

# Использование
Конфиг: /etc/config/podkop

Luci: Services/podkop

## Режимы

### Proxy
Для VLESS и Shadowsocks. Другие протоколы тоже будут, кидайте в чат примеры строк без чувствительных данных.
Для использования этого режима нужен sing-box:
```
opkg update && opkg install sing-box
```

В этом режиме просто копируйте строку в **Proxy String** и из неё автоматически настроится sing-box.

### VPN
Здесь у вас должен быть уже настроен WG/OpenVPN/OpenConnect etc, создана Zone и Forwarding.

Просто выбрать интерфейс из списка.

## Настройка доменов и подсетей
**Domain list enable** - Включить общий список.

**Delist domains from main list enable** - Выключение заданных доменов из общего списка. Задавать списком.

**Subnets list enable** - Включить подсети из общего списка, выбрать из предложенных.

**Custom domains enable** - Добавить свои домены. Задавать списком.

**Custom subnets enable** - Добавить подсети или IP-адреса. Для подсетей задать маску.

# Известные баги
1. ucitrack не рестартится автоматически после установки пакета 

# ToDo
- [ ] Скрипт для автоматической установки.
- [x] Подсети дискорда.
- [ ] Удаление getdomains через скрипт. Кроме туннеля и sing-box.
- [ ] Дополнительная вкладка для ещё одного туннеля. Домены, подсети.
- [ ] Wiki
- [ ] IPv6
- [ ] Исключение для IP, не ходить в туннель\прокси совсем 0x0

# Разработка
Есть два варианта:
- Просто поставить пакет на роутер или виртуалку и прям редактировать через SFTP
- SDK, чтоб собирать пакеты

Для сборки пакетов нужен SDK, один из вариантов скачать прям файл и разархивировать
https://downloads.openwrt.org/releases/23.05.5/targets/x86/64/
Нужен файл с SDK в имени

```
wget https://downloads.openwrt.org/releases/23.05.5/targets/x86/64/openwrt-sdk-23.05.5-x86-64_gcc-12.3.0_musl.Linux-x86_64.tar.xz
tar xf openwrt-sdk-23.05.5-x86-64_gcc-12.3.0_musl.Linux-x86_64.tar.xz
mv openwrt-sdk-23.05.5-x86-64_gcc-12.3.0_musl.Linux-x86_64 SDK
```
Последнее для удобства.

Создаём директорию для пакета
```
mkdir package/utilites && mkdir package/luci
```

Симлинк из репозитория
```
ln -s ~/podkop/podkop package/utilites/podkop
ln -s ~/podkop/luci-app-podkop package/luci/luci-app-podkop
```

Сборка пакета
```
make package/podkop/{clean,compile} V=s
```

Также для luci
```
make package/luci-app-podkop/{clean,compile} V=s
```

При первом make выводится менюшка, можно просто сохранить и всё. Первый раз долго грузит зависимости.

## make зависимости
https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem

Ubuntu
```
sudo apt update
sudo apt install build-essential clang flex bison g++ gawk \
gcc-multilib g++-multilib gettext git libncurses-dev libssl-dev \
python3-distutils rsync unzip zlib1g-dev file wget
```