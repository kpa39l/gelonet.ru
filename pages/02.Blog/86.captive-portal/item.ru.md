---
title: 'Captive portal'
date: '22:22 26-11-2014'
publish_date: '22:22 26-11-2014'
taxonomy:
    category:
        - Blog
    tag:
        - Linux
---

Собираю инфу по реализации Captive порталов для WiFI сетей.
Вот, первая пташка.

[Captive portal собственными руками](https://habr.com/ru/sandbox/79059/)

> Каждый из нас подключался к беспроводной сети (аэропорты, кафе и тд) где необходимо согласится с некоторыми условиями или пройти авторизации прежде чем начинать пользоваться интернетом. Такая технология называется captive portal. 
> 
> В мою задачу входило создать captive portal где каждому пользователю раз в 30 минут будет показываться определенный сайт (реклама), и пока он не нажмет «кнопочку» «далее» интернета не будет. 
> 
> Для создания нам понадобится:
> Wifi-точка доступа любая (на практике можно использовать и проводной интернет)
> Маршрутизатор на любой *nix системе (в моем случае это debian wheezy)
> 
> 
> Краткое описание принципа работы
> Любой пакет пришедший на маршрутизатор маркируем
> Любой запрос на 80 порт (пакеты уж маркированны) перенаправляем на нужную нам страницу
> При «авторизации»(нажатии кнопки далее) пользователя добавляем его мак в исключения и разрешаем доступ в интернет
> Скрипт вычисляет у кого прошел лимит времени и удаляет данного клиента.
> 
> 
> 
> У нас имеется маршрутизатор на базе debian, где eth0 — локальная сеть в моем случае (192.168.11.0/24), eth0:1 — интерфейс который смотрит в интернет.
> 
> Настройку интерфейсов мы пропустим, это каждый выполнит сам. Сразу перейдем к настройке iptables, я обычно прописываю iptables в rc.local:
> /etc/rc.local
> IPTABLES="/sbin/iptables"
> EBTABLES="/sbin/ebtables"
> DHCP="67:68"
> SSH="22"
> WWW="80"
> 
> $IPTABLES -t mangle -F
> $IPTABLES -F
> 
> $IPTABLES -A INPUT -i lo -j ACCEPT
> #ssh
> $IPTABLES -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
> # разрешаем серверу и клиентам dns гугла
> $IPTABLES -A INPUT -s 8.8.8.8 -j ACCEPT
> $IPTABLES -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
> 
> # создаем правило интернет
> $IPTABLES -N internet -t mangle
> $IPTABLES -t mangle -A PREROUTING -j internet
> #добавляем разрешенные маки в исключения путем добавления их в Return
> awk 'BEGIN { FS="\t"; } { system("$IPTABLES -t mangle -A internet -m mac --mac-source "$4" -j RETURN"); }' /var/lib/users
> #маркируем все пакеты
> $IPTABLES -t mangle -A internet -j MARK --set-mark 99
> # все маркированные пакеты которые идут на 80 порт отправляем на наш сервер
> $IPTABLES -t nat -A PREROUTING -m mark --mark 99 -p tcp --dport 80 -j DNAT --to-destination 192.168.11.38
> #дроппаем все маркированное
> #$IPTABLES -t filter -A FORWARD -m mark --mark 99 -j DROP
> #dns
> $IPTABLES -t filter -A INPUT -s 8.8.8.8 -j ACCEPT
> #http
> $IPTABLES -t filter -A INPUT -p tcp --dport 80 -j ACCEPT
> #port dns
> $IPTABLES -t filter -A INPUT -p udp --dport 53 -j ACCEPT
> #drop
> $IPTABLES -t filter -A INPUT -m mark --mark 99 -j DROP
> echo "1" > /proc/sys/net/ipv4/ip_forward
> #настройка нат
> $IPTABLES -A FORWARD -i eth0 -o eth0:1 -m state --state ESTABLISHED,RELATED -j ACCEPT
> $IPTABLES -A FORWARD -i eth0:1 -o eth0 -j ACCEPT
> $IPTABLES -t nat -A POSTROUTING -o eth0 -j MASQUERADE
> 
> На этом конфигурация iptables заканчивается и переходим к странице которую получает пользователь: /var/lib/index.php
> <?php
> 
> $server_name = "192";
> $domain_name = "168.11.38";
> $site_name = "Wireless Network";
> 
> // arp
> $arp = "/usr/sbin/arp";
> 
> // файл где хранятся текущие пользователи
> $users = "/var/lib/users";
> 
> // Check if we've been redirected by firewall to here.
> // If so redirect to registration address
> if ($_SERVER['SERVER_NAME']!="$server_name.$domain_name") {
>   header("location:http://$server_name.$domain_name/index.php?add="
>     .urlencode($_SERVER['SERVER_NAME'].$_SERVER['REQUEST_URI']));
>   exit;
> }
> 
> // получаем и добавляем мак пользователя
> $mac = shell_exec("$arp -a ".$_SERVER['REMOTE_ADDR']);
> preg_match('/..:..:..:..:..:../',$mac , $matches);
> @$mac = $matches[0];
> if (!isset($mac)) { exit; }
> 
> if (!isset($_POST['email']) || !isset($_POST['name'])) {
>   // Name or email address not entered therefore display form
>   ?>
>   <h1>Welcome to <?php echo $site_name;?></h1>
>   To access the Internet you must first enter your details:<br><br>
>   <form method='POST'>
>   <table border=0 cellpadding=5 cellspacing=0>
>   <tr><td>Your full name:</td><td><input type='text' name='name'></td></tr>
>   <tr><td>Your email address:</td><td><input type='text' name='email'></td></tr>
>   <tr><td></td><td><input type='submit' name='submit' value='Submit'></td></tr>
>   </table>
>   </form>
>   <?php
> } else {
>     enable_address();
> }
> 
> 
> function enable_address() {
> 
>     global $name;
>     global $email;
>     global $mac;
>     global $users;
> // время добавления пользователя храним в unixtime
>     file_put_contents($users,$_POST['name'].";".$_POST['email'].";"
>         .$_SERVER['REMOTE_ADDR'].";$mac;".date("U")."\n",FILE_APPEND + LOCK_EX);
>     
>     // добавляем правило в iptables
>     exec("sudo iptables -I internet 1 -t mangle -m mac --mac-source $mac -j RETURN");
>     // The following line removes connection tracking for the PC
>     // This clears any previous (incorrect) route info for the redirection
>     exec("sudo rmtrack ".$_SERVER['REMOTE_ADDR']);
> 
>     sleep(1);
>     header("location:http://".$_GET['add']);
>     exit;
> }
> 
> // Function to print page header
> function print_header() {
> 
>   ?>
>   <html>
>   <head><title>Welcome to <?php echo $site_name;?></title>
>   <META HTTP-EQUIV="CACHE-CONTROL" CONTENT="NO-CACHE">
>   <LINK rel="stylesheet" type="text/css" href="./style.css">
>   </head>
> 
>   <body bgcolor=#FFFFFF text=000000>
>   <?php
> }
> 
> // Function to print page footer
> function print_footer() {
>   echo "</body>";
>   echo "</html>";
> 
> }
> 
> ?>
> 
> Данная страница может быть любой, смысл лишь в том, чтобы получить мак пользователя и добавить его в iptables. 
> 
> В текущий момент получилась следующая схема:
> Iptables автоматически редиректит любой запрос на 80 порт на нашу страницу
> пользователь вводит свое имя и почту, скрипт php получает мак пользователя и добавляет в iptables правило исключения для этого мака
> Php скрипт складывает все данные пользователей (имя, почта, время добавления) в файл /var/lib/users
> 
> Осталось только сделать скрипт который будет:
> удалять правило из iptables для мака у которого вышло время
> удалять из файла данные по этому пользователю
> 
> 
> Так как мои знания в программировании довольно плачевны был написан простенький скрипт на перле:
> #!/usr/bin/perl
> 
> $file = "/var/lib/users";
> $curtime = time();
> # время "бана" 30 минут время в секундах так как считаем потом в unixtime
> $bantime = 1800;
> $tmpfile = "out.tmp";
> 
> open(my $data, '<', $file) or die "Could not open '$file' $!\n";
> while (my $line = <$data>) {
>   chomp $line;
> (@field) = split(";", $line);
> 
> 
> #прибавляем время "бана" ко времени когда зашел пользователь (4 поле в нашем случае) получаем время когда нужно удалить пользователя
> if (defined($exptime)) { undef $exptime; }
> $result = $field[4];
> $exptime = $bantime + $result;
> # такая коснтрукция потому что перл не хотел прибавлять $field[4] + $bantime
> 
> # если текущее время больше времени для удаления, удаляем мак и перезаписываем файл
> if ($curtime > $exptime) {
> #printf "BINGO\n";
>         system "iptables -D internet -t mangle -m mac --mac-source $field[3] -j RETURN";
> #       system "rm /var/lib/users";
> 
>                         } 
> else {
>         printf "$field[3]; $exptime\n"; 
>         
>         open SF, ">>$tmpfile";
>         printf SF "$field[0];$field[1];$field[2];$field[3];$field[4]\n";
>         close SF; 
>         system "mv /root/out.tmp /var/lib/users";
>        system "chown www-data. $file"; 
>         }
>         
> }
> 
> 
> 
> >  Все, задача выполенна, модифицируя файлик index.php можем показывать пользователю релкаму, смешные видео, заставлять его оставлять свою почту и т.д.