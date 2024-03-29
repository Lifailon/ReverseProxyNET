# Reverse Proxy .NET / rpnet

[➡️](https://github.com/Lifailon/ReverseProxyNET/blob/rsa/README.md) Choose language [➡️‍](https://github.com/Lifailon/ReverseProxyNET/blob/rsa/README_RU.md) Выберите язык

Кроссплатформенная утилита командной строки для реализации обратного прокси-сервер на базе .NET. Используется для предоставления доступа всем хостам в сети с одного сетевого интерфейса к удаленным приложениям через протоколы HTTP и TCP доступных через другой сетевой интерфейс без лишних настроек и с поддержкой авторизации.

## 💁 Для чего?

Данная утилита решает несколько задач:

- Во первых, если вы используете VPN сервер в режиме точка-точка, и хотите предоставить доступ удаленной машине к приложениям на других машинах в вашей сети работающих через протокол TCP, например, для протоколов RDP, SSH, RTSP и т.д. Это также выступает альтернативой классического ssh туннелирования, например, через `OpenSSH` или `Putty`.

- Во вторых, если вы используете VPN сервер для доступа к конкретному url ресурсу (HTTP/HTTPS) на своей машине, и хотите предоставить к нему доступ другим машинам в сети без использования Proxy серверов, которые требуют дополнительной настройки на стороне клиента.

- В третьих, если ваша Web-приложение или REST API сервер не поддерживают авторизацию, то вы можете воспользоваться [Base64](https://en.wikipedia.org/wiki/Base64) шифрованием, что обязует передавать в заголовке запроса авторизационные данные для всех клиентов, которые будут подключаться через Proxy. Если вы используете браузер, то будет предоставлена соответствующая форма для прохождения базовой авторизации.

Существует много альтернатив, которые предоставляют схожий функционал по отдельности. Например, `ncat` в Windows (из состава [nmap](https://github.com/nmap/nmap)) и `socat` в Linux для TCP или [ReverseProxy](https://github.com/ilanyu/ReverseProxy) на Golang для перенаправления HTTP/HTTPS трафика. Весь вышеперечисленный функционал реализован в одной утилите `rpnet`.

## 🚀 Установка

### 💻 Windows

- [Скачайте и установите](https://dotnet.microsoft.com/en-us/download/dotnet/8.0/runtime) среду выполнения приложений .NET версии 8.0.

- [Загрузите](https://github.com/Lifailon/ReverseProxyNET/releases/latest) исполняемый файл портативной версии из GitHub репозитория.

### 🐧 Linux

> Протестирован в системе Ubuntu 22.04.

- Установите среду выполнения приложений .NET:

```shell
sudo apt-get install -y dotnet-runtime-8.0
```

- Загрузите исполняемый файл `rpnet` в директорию `/usr/local/bin/` и предоставьте права на выполнение:

```shell
sudo curl -s -L https://github.com/Lifailon/ReverseProxyNET/releases/download/0.0.1-beta/rpnet-0.0.1-beta-linux-x64.1-beta -o /usr/local/bin/rpnet
sudo chmod +x /usr/local/bin/rpnet
```

## 📑 Использование

Получите справку:

```shell
.\rpnet.exe --help

Reverse Proxy server base on .NET.

Parameters:
  -h, --help                       Show help.
  -l, --local <address:port>       Address and port of the interface through which requests will be proxy.
  -r, --remote <address:port/url>  HTTP/HTTPS address of the remote resource to which requests will be proxy.
  -u, --userName <admin>           User name for authorization.
  -p, --password <admin>           User password for authorization.

Examples:
  .\rpnet.exe -l 127.0.0.1:8443 -r 192.168.3.106:80
  .\rpnet.exe -l 127.0.0.1:8443 -r https://kinozal.tv
  .\rpnet.exe -l *:8443 -r https://kinozal.tv
  .\rpnet.exe -local *:8443 -remote https://kinozal.tv -userName proxy -password admin
```

### 📡 TCP

Принимает запросы на интерфейсе с ip-адресом `192.168.3.100` и порту `8443` для перенаправления на удаленный хост с ip-адресом `192.168.3.106`, где запущенно приложение на порту `80`.

```shell
.\rpnet.exe --local 192.168.3.100:8444 --remote 192.168.3.106:80
TCP protocol is used
Listening on 192.168.3.100:8444 for forwarding to 192.168.3.106:80
[13:40:23] 192.168.3.99:29829: [::ffff:192.168.3.106]:80
[13:40:23] 192.168.3.99:29830: [::ffff:192.168.3.106]:80
[13:40:23] 192.168.3.99:29833: [::ffff:192.168.3.106]:80
[13:40:23] 192.168.3.99:29838: [::ffff:192.168.3.106]:80
[13:40:23] 192.168.3.99:29840: [::ffff:192.168.3.106]:80
[13:40:23] 192.168.3.99:29842: [::ffff:192.168.3.106]:80
[13:40:24] 192.168.3.99:29846: [::ffff:192.168.3.106]:80
[13:40:24] 192.168.3.99:29848: [::ffff:192.168.3.106]:80
```

> 💡 Что бы прослушивать все сетевые интерфейсы, используйте символ * вместо локального ip-адреса (требуется запустить консоль с правами администратора).

### 🌐 HTTP

Принимает запросы на интерфейсе с ip-адресом `192.168.3.100` и порту `8443` для перенаправления на удаленный url-ресурс [Кинозал](https://kinozal.tv).

В примере, подключение производится от клиента с ip-адресом `192.168.3.99` с использованием метода `GET`. Указаны всех конечные точки, к которым обращается клиент для загрузки главной страницы.

```shell
.\rpnet.exe --local 192.168.3.100:8443 --remote https://kinozal.tv
HTTP protocol is used
Listening on 192.168.3.100:8443 for forwarding to https://kinozal.tv
Not authorization is used
[12:06:15] 192.168.3.99 GET: /
[12:06:16] 192.168.3.99 GET: /pic/0_kinozal.tv.css?v=3.4
[12:06:16] 192.168.3.99 GET: /pic/jquery-3.6.3.min.js?v=1.1
[12:06:16] 192.168.3.99 GET: /pic/use.js?v=3.7
[12:06:16] 192.168.3.99 GET: /pic/logo3.gif
[12:06:16] 192.168.3.99 GET: /pic/emty.gif
[12:06:17] 192.168.3.99 GET: /pic/radio_ban.jpg
[12:06:17] 192.168.3.99 GET: /pic/knz_love.gif
[12:06:17] 192.168.3.99 GET: /pic/sbg.gif
[12:06:17] 192.168.3.99 GET: /pic/l_portiere.gif
[12:06:17] 192.168.3.99 GET: /pic/r_portiere.gif
[12:06:17] 192.168.3.99 GET: /pic/dw2.png       
[12:06:17] 192.168.3.99 GET: /pic/cat/11.gif
[12:06:17] 192.168.3.99 GET: /pic/cat/8.gif 
[12:06:18] 192.168.3.99 GET: /pic/cat/15.gif
[12:06:18] 192.168.3.99 GET: /pic/cat/17.gif
[12:06:18] 192.168.3.99 GET: /pic/status_icons.png
[12:06:19] 192.168.3.99 GET: /pic/srch_l.png
[12:06:19] 192.168.3.99 GET: /pic/srch_r2.png        
[12:06:19] 192.168.3.99 GET: /pic/flags_all.png?v=1  
[12:06:19] 192.168.3.99 GET: /i/poster/8/6/433686.jpg
[12:06:19] 192.168.3.99 GET: /i/poster/1/9/1357519.jpg
[12:06:19] 192.168.3.99 GET: /i/poster/2/1/975821.jpg
[12:06:19] 192.168.3.99 GET: /pic/cat/24.gif
[12:06:19] 192.168.3.99 GET: /i/poster/2/0/175220.jpg 
[12:06:19] 192.168.3.99 GET: /i/poster/2/0/1255220.jpg
[12:06:20] 192.168.3.99 GET: /pic/favicon.ico
```

### 🔓 Авторизация

Для использования авторизации на стороне клиента, необходимо заполнить соответствующие параметры при запуске сервера. Если клиент передает неверные авторизационные данные, то это будет отображено в логе.

```shell
.\rpnet.exe --local 192.168.3.100:8443 --remote https://kinozal.tv --userName proxy --password admin
HTTP protocol is used
Listening on 192.168.3.100:8443 for forwarding to https://kinozal.tv
Authorization is used
[15:43:39] 192.168.3.100 GET: /
[15:43:39] 192.168.3.100 GET: Authorization error
[15:43:40] 192.168.3.100 GET: /
[15:43:40] 192.168.3.100 GET: Authorization form sent
[15:43:42] 192.168.3.100 GET: /
[15:43:42] 192.168.3.100 GET: Authorization error
[15:43:43] 192.168.3.100 GET: /
[15:43:43] 192.168.3.100 GET: Authorization form sent
[15:43:47] 192.168.3.100 GET: /
[15:43:48] 192.168.3.100 GET: /pic/0_kinozal.tv.css?v=3.4
[15:43:48] 192.168.3.100 GET: /pic/jquery-3.6.3.min.js?v=1.1
[15:43:48] 192.168.3.100 GET: /pic/use.js?v=3.7
[15:43:48] 192.168.3.100 GET: /pic/logo3.gif
[15:43:48] 192.168.3.100 GET: /pic/emty.gif
[15:43:49] 192.168.3.100 GET: /pic/radio_ban.jpg
[15:43:49] 192.168.3.100 GET: /pic/knz_love.gif
[15:43:49] 192.168.3.100 GET: /pic/sbg.gif
[15:43:49] 192.168.3.100 GET: /pic/l_portiere.gif
[15:43:49] 192.168.3.100 GET: /pic/r_portiere.gif
[15:43:49] 192.168.3.100 GET: /pic/dw2.png
[15:43:49] 192.168.3.100 GET: /pic/cat/8.gif
[15:43:50] 192.168.3.100 GET: /pic/cat/49.gif
[15:43:50] 192.168.3.100 GET: /pic/cat/6.gif
[15:43:50] 192.168.3.100 GET: /pic/cat/45.gif
[15:43:50] 192.168.3.100 GET: /pic/cat/13.gif
[15:43:50] 192.168.3.100 GET: /pic/status_icons.png
[15:43:50] 192.168.3.100 GET: /pic/srch_l.png
[15:43:50] 192.168.3.100 GET: /pic/srch_r2.png      
[15:43:50] 192.168.3.100 GET: /pic/flags_all.png?v=1
[15:43:51] 192.168.3.100 GET: /i/poster/0/8/1767008.jpg
[15:43:51] 192.168.3.100 GET: /i/poster/1/7/1995717.jpg
[15:43:51] 192.168.3.100 GET: /pic/cat/15.gif
[15:43:51] 192.168.3.100 GET: /pic/cat/38.gif
[15:43:51] 192.168.3.100 GET: /i/poster/2/0/1255220.jpg
[15:43:51] 192.168.3.100 GET: /i/poster/2/6/2024026.jpg
[15:43:51] 192.168.3.100 GET: /pic/cat/46.gif
[15:43:52] 192.168.3.100 GET: /pic/cat/24.gif
[15:43:52] 192.168.3.100 GET: /pic/favicon.ico
```