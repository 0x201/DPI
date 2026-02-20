# Как обойти DPI

В данном гайде я расскажу как исправить проблему зависающего YouTube и заблокированных сайтов с помощью различных программ обхода блокировок **DPI** (Deep Packet Inspection / Глубокое исследование пакетов).
> [!WARNING]
> В работу DPI погружать не буду, а сразу перейду к делу. Если есть желание узнать об этой технологии, вам [сюда](https://web.archive.org/web/20230331233644/https://habr.com/ru/post/335436/), или [туда](https://ru.wikipedia.org/wiki/Deep_packet_inspection), или [here](https://geneva.cs.umd.edu/papers/geneva_ccs19.pdf).

- [Как обойти DPI](#как-обойти-dpi)
  - [Zapret для YouTube (Linux / Роутеры)](#zapret-для-youtube-linux--роутеры)
  - [Zapret для всех блокировок](#zapret-для-всех-блокировок)
  - [Zapret (Windows)](#zapret-windows)
  - [Zapret-discord-youtube, пока лучший на Windows](#zapret-discord-youtube-пока-лучший-на-windows)
  - [ByeByeDPI (Android)](#byebyedpi-android)
  - [Решение возникших проблем](#решение-возникших-проблем)
    - [Скрипты Zapret не являются исполняемыми](#скрипты-zapret-не-являются-исполняемыми)
    - [Переустановка Bash](#переустановка-bash)
    - [Zapret не запущен](#zapret-не-запущен)
    - [ByeDPI/ByeByeDPI жрет много батареи](#byedpibyebyedpi-жрет-много-батареи)
  - [Источники / Готовые конфиги](#источники--готовые-конфиги)
  - [Поддержка](#поддержка)

## Zapret для YouTube (Linux / Роутеры)
1. Необходимо скачать программу и необходимые компоненты. Сделаем мы это с помощью Git. 

Установка на [Linux](https://git-scm.com/downloads/linux):
- Debian / Ubuntu / Debian подобные:
  `sudo apt-get install git curl ip6tables ipset iptables`

- Либо для Ubuntu:
  `add-apt-repository ppa:git-core/ppa`
  `sudo apt update && apt install git curl ip6tables ipset iptables`

- Arch / Arch подобные:
  `sudo pacman -S git dnsutils curl ip6tables ipset iptables`

-  Red Hat / Fedora / Red Hat подобные:
  `sudo dnf install git curl iptables ipset`

- NixOs:
  `sudo nix-env -i git curl ipset iptables`

- Gentoo:
  ```
  emerge --ask --verbose dev-vcs/git
  emerge --ask --verbose net-misc/curl
  emerge --ask --verbose net-firewall/ipset
  emerge --ask --verbose net-firewall/iptables
  ```

- [OpenWrt](https://gist.github.com/kolyanok/6e93f3ed5f3aefb4d482df8c4463f196) / [Keenetic](https://telegra.ph/Nastrojka-zapret-ot-bol-van-na-Keenetic-08-18):
  ```
  opkg update
  opkg install iptables-mod-extra iptables-mod-nfqueue iptables-mod-filter iptables-mod-ipopt iptables-mod-conntrack-extra ipset curl ip6tables-mod-nat grep git-http curl gzip ipset iptables nano ca-certificates
  mkdir /opt # Если директории нет
  ```

2. Клонируем архив с программой
   `git clone https://github.com/bol-van/zapret /opt/zapret`

3. **(OpenWRT/Keenetic пропускаем логин в рут)** Переходим в папку с Zapret из под рута
   `su -`
    Вводим пароль root\`а
   `cd /opt/zapret`

4. Далее запускаем скрипты установки всех нужных компонентов
   `./install_bin.sh`
   `./install_prereq.sh`

5. Запускаем скрипт установки службы
   `./install_easy.sh`

   И делаем следующее:
    - _Select firewall type_ - Выбираем на свое усмотрение
    - _enable IPV6 support_ - Выбираем отталкиваясь от того какой версии IP вы пользуетесь
    - _select MODE_ - Рекомендую выбирать между tpws и nfqws. Я буду показывать напримере **nfqws**, т.к. работает она лучше.
    - _do you want to edit the options_ - Да, мы хотим изменить, а потому вводим `Y` и изменяем опции:

    ```
    NFQWS_OPT_DESYNC=""
    #NFQWS_OPT_DESYNC_SUFFIX=
    #NFQWS_OPT_DESYNC_HTTP=
    #NFQWS_OPT_DESYNC_HTTP_SUFFIX=
    #NFQWS_OPT_DESYNC_HTTPS=
    #NFQWS_OPT_DESYNC_HTTPS_SUFFIX=
    #NFQWS_OPT_DESYNC_HTTP6=
    #NFQWS_OPT_DESYNC_HTTP6_SUFFIX=
    #NFQWS_OPT_DESYNC_HTTPS6=
    #NFQWS_OPT_DESYNC_HTTPS6_SUFFIX=
    NFQWS_OPT_DESYNC_QUIC=""
    #NFQWS_OPT_DESYNC_QUIC_SUFFIX=
    #NFQWS_OPT_DESYNC_QUIC6=
    #NFQWS_OPT_DESYNC_QUIC6_SUFFIX=
    ```
  
    Всё что за **#** - комменатрии, их не трогаем. 
    Нужны нам лишь `NFQWS_OPT_DESYNC` и *(опционально, в зависимости от того будете вы использовать протокол QUIC или нет) `NFQWS_OPT_DESYNC_QUIC`*. 

    Выбираем способ обхода замедления. Теперь вписываем в `NFQWS_OPT_DESYNC` после `="`:
    `NFQWS_OPT_DESYNC="ваш способ обхода`
    
    Для протокола QUIC:
    
    `NFQWS_OPT_DESYNC_QUIC="ваш способ обхода"`

- После настройки опций сохраняемся и выходим.
- _WAN interface_ - Рекомендую выбрать ANY
- _enable http support_ - Включаем
- _enable https support_ - Включаем
- _enable quic support_ - Можно не включать, опять же в зависимости от того будете ли вы пользоваться протоколом QUIC или нет
- _select filtering_ - Рекомендую выбирать hostlist
- _do you want to auto download ip/host list_ - Скачиваем сам hostlist
- _your choice_ - Рекомендую оставить по умолчанию

6. Далее вставим ссылки из файла [blacklist.txt](./blacklist.txt)
В файл `zapret-hosts-user.txt`:

`nano /opt/zapret/ipser/zapret-hosts-user.txt`
Сохраняемся и выходим.
    
7. **(Если хотите QUIC)** В вашем браузере зайдите в доп. настройки:
- Chrome - `chrome://flags/#enable-quic`
- Vivaldi - `vivaldi://flags/#enable-quic`
- Opera - `opera://flags/#enable-quic`
- Yandex - `browser://flags/#enable-quic`
- FireFox - зайдите в `about:config` и в поиске введите `network.http.http3.enable` , переставьте значение на `true`.

> [!IMPORTANT]
> Желательно **перезагрузить компьютер** или же перезапустить браузер.

## Zapret для всех блокировок
1. Запускаем скрипт для выявления стратегии обхода блокировок:
   `./blockcheck.sh`

-   _specify domain(s) for test_ - По умолчанию выставлен rutracker.org, можно оставить. Но не выставляйте как тестовый домен YouTube или Google, т.к. они замедлены, но не заблокированы полностью.
-   _ip protocol version(s)_ - Выбираем отталкиваясь от того какой версии IP вы пользуетесь.
-   _check http_ - Оставляем.
-   _check https tls 1.2_ - Оставляем.
-   _check https tls 1.3_ - Отключаем.
-   _do not verify server certificate_ - Отключаем.
-   _how many times to repeat each test_ - Выставляем кол-во запросов на сайт с одной стратегией.
-   Выбор между _quick_ _standart_ _force_:
  _Quick_ - Быстрая проверка **(15-30 минут)**.
  _Standart_ - Классическая проверка **(1-1.5 часов)**.
  _Force_ - Максимальная проверка **(более 2-ух часов)**.
- Теперь ожидаем окончание проверки.
- После окончания проверки появится итоговый результат. Выглядит он примерно так **(ЭТО НЕ ГОТОВЫЙ КОНФИГ)**:
  `ipv4 rutracker.org curl_test_https_tls12:nfqws --dpi-desync=fake,split2 --dpi-desync-ttl=3`
  Важна нам часть после :
  - _nfqws_ - это MODE, который вы выбираете в скрипте install_easy.sh, у вас может быть и tpws.
  - Строку `--dpi-desync=fake,split2 --dpi-desync-ttl=3` нам необходимо вставить в опцию `NFQWS_OPT_DESYNC` или `NFQWS_OPT_DESYNC_QUIC`.


2. Теперь после найденной стратегии запускаем скрипт `./install_easy.sh`, доходим до _do you want to edit the options_ и редактируем конфиг.

## Zapret (Windows)
1. Скачиваем [программу](https://github.com/bol-van/zapret-win-bundle) из репозитория.
2. Распаковываем.
3. Зайдём в папку с Zapret и уже там в `zapret-winws`.
4. Запустим файл `preset_russia.cmd`.

> [!WARNING]
> Изначальная конфигурация не будет работать, нужно её составить самому. В этом помогут [неравнодушные люди с форумов и документация](#источники--готовые-конфиги).

## Zapret-discord-youtube, пока лучший на Windows
Тот же Zapret, но с готовыми конфигами.

1. Скачиваем [программу](https://github.com/Flowseal/zapret-discord-youtube/releases) с репозитория.
2. Распаковываем и запускаем `service.cmd` файл.
3. В открывшемся меню выбираем `10. Run Diagnostics`. Данная функция проверит вашу систему на наличие препятствий на пути обхода блокировок. **Ликвидируем** все преграды и переходим к следующему пункту.
4. После прохождения диагностики запускаем `11. Run Tests`. Выбираем пункт который вам нужен:
   - `1. Standart tests` - проверка доступа заблокированного ресурса _(Youtube, Google, Cloudflare(как сайт, а не груда айпишников))_.
   - `2. DPI Checket` - проверка доступа к CDN (Content Delivery Network), это всякие: Cloudflare _(На нём интернет держится)_, Fastly _(Steam, и др.)_, Hetzner _(Arch Linux репозитории)_ и куча других.
5. Далее рекомендую выбрать пункт `All configs`, так как вы пока не знаете какие конфиги работают лучше, а потому придётся потратить время на проверку заведомо плохих. Однако это гарантирует лучший результат.
6. После проверки наблюдаем итоги, по ним ориентируемся при выборе конфига. В меню программы нажимаем `1. Install Service`, выбираем конфигурацию и всё.
> [!WARNING]
    По всем остальным вопросам рекомендую читать официальную документацию, потому что здесь повторяться не хочу, да и я не автор оригинальной программы, а потому не знаю многих тонкостей.

## ByeByeDPI (Android)
1. Скачиваем программу с официального [репозитория](https://github.com/romanvht/ByeByeDPI/releases).
2. Устанавливаем и запускаем.
3. Сразу не подключаемся, а заходим в настройки.
4. Далее выбираем пункт `Подбор стратегий (Beta)`.
5. Слева сверху есть шестерёнка (настройки подбора), можно там поколодовать с тем, что именно будем проверять, как и зачем. Выберите что вам необходимо и запускайте проверку, она обычно занимает 10-15 минут.
6. После перед нами предстаёт список стратегий от лучше к худшей, выбираем, которая по запросам превзошла всех _(то есть 150/150 или какая у вас первая в списке)_.
7. Выходим в главное меню нажимаем кнопку подключения и проверяем, работает - отлично, нет - берём другую стратегию.

Далее можно на свой вкус и цвет настроить программу, например поставить белые или чёрные списки прокисируемых приложений, но тут уже ваше решение.

## Решение возникших проблем
В данном разделе вы скорее всего найдёте решение вашей проблемы.

### Скрипты Zapret не являются исполняемыми
Для того чтобы дать скриптам права на исполнение вводим комманду:
`chmod u+x "путь до скрипта"`

Если нужно наделить других пользователей полномочиями запускать скрипт, то вводим следующее:
`chmod ugo+x "путь до скрипта"`

### Переустановка Bash
- Debian / Ubuntu / Debian подобные:
  `sudo apt-get install bash`

- Arch / Arch подобные:
  `sudo pacman -S base base-devel`

- Red Hat / Fedora / Red Hat подобные:
  `sudo dnf install bash`

- Gentoo:
  `emerge --ask --verbose app-shells/bash`

- NixOs:
  `sudo nix-env -u '*' `

- OpenWrt / Keenetic:
  `opkg install bash`

### Zapret не запущен
Если служба не запущена, то запускаем её этой коммандой:
`sudo systemctl start --now zapret.service`

Для остановки:
`sudo systemctl stop --now zapret.service`

Для отключения автозагрузки:
`sudo systemctl disable --now zapret.service`

Для добавления в автозагрузку:
`sudo systemctl enable --now zapret.service`

Чтобы узнать статус службы вводим:
`sudo systemctl status zapret.service`

Если же такой службы нет, то переходим к [пункту](#zapret-для-youtube-linux--роутеры) 5 и запускаем скрипт, вчитываясь в логи. Так можно найти ошибку.

### ByeDPI/ByeByeDPI жрет много батареи
Решить это почти никак, т.к. по сути программа висит в фоне и весь ваш траффик пропускает через себя, модифицируя его. Но пару действий можно предпринять:

- Можно включить оптимизацию батареи для данного приложения 
  1. Переходим в настройки телефона, ищем пункт _Приложения_, там ищем **ByeDPI**, нажимаем на приложение.
  2. Здесь есть пункт _Расход батареи / Экономия батареи / Использование батареи / App battery usage_
  3. И выбираем пункт _Optimized / Оптимизированый_ и в таком духе.
  
- Могу дать совет, не используйте приложение в фоне постоянно. Посмотрели YouTube, условно говоря, выключили.

## Источники / Готовые конфиги
> [!CAUTION]
> NTC является заблокированным сайтом, как и ветка с ByeDPI на 4PDA **(ТОЛЬКО VPN ИЛИ ПРОКСИ)**. Зайти можно либо с обходом блокировок (DPI), либо с VPN, либо с прокси.

Ссылки на готовые конфиги:
- ByeDPI/ByeByeDPI- [4PDA](https://4pda.to/forum/index.php?showtopic=1092092), [GitHub Issue](https://github.com/dovecoteescapee/ByeDPIAndroid/discussions/64), [NTC](https://ntc.party/t/byedpi-for-android-%D0%BE%D0%B1%D1%81%D1%83%D0%B6%D0%B4%D0%B5%D0%BD%D0%B8%D0%B5/9075)
- Zapret - [GitHub Issue](https://github.com/bol-van/zapret/discussions/200), [NTC](https://ntc.party/t/zapret-%D0%BE%D0%B1%D1%81%D1%83%D0%B6%D0%B4%D0%B5%D0%BD%D0%B8%D0%B5/726)

Списки заблокированных сайтов можете найти здесь:
 - [GitHub Issue Zapret](https://github.com/bol-van/zapret/discussions/200#discussioncomment-10956108)
 - [IPSET by V3nilla](https://github.com/V3nilla/IPSets-For-Bypass-in-Russia)

## Поддержка
Если вы хотите поддержать автора, то поставьте :star: _(в левом верхнем углу)_. И, пожалуйста, старайтесь своими действиями помочь сделать этот мир лучше и **свободнее**.
**Автор: 0x201 :3**