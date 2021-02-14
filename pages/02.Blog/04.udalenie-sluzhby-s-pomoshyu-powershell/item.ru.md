---
title: 'Удаление службы с помощью Powershell'
media_order: powershell_5_dsc.png
date: '04:47 04-11-2017'
publish_date: '04:47 04-11-2017'
taxonomy:
    category:
        - Blog
    tag:
        - Powershell
hero_image: powershell_5_dsc.png
blog_url: /blog
show_sidebar: true
show_breadcrumbs: true
show_pagination: true
---

Подхватил недавно Adware которые насоздавал кучу лишних служб, таких как updater.mail.ru. Конечно, в состав Windows 10 входит отличная утилита SC с помощью которой их легко можно удалить:

>  sc delete updater.mail.ru.

Но удивил тот факт что в составе Powershell есть командлеты для создания, настройки и управления службами, но вот для удаления нет.  Есть костыли использующие WMI

(Get-WmiObject win32_service -Filter ″name=′updater.mail.ru′″).delete()

или прямое удаление ветки реестра относящийся к службе в разделе:

> HKLM:\SYSTEM\CurrentControlSet\Services\

но вот командлета нет. Искал на powershelgallery.com и в стандартной поставке. Отличный пример когда соберусь изучать создание командлетов. наверняка это из-за каких-подводных камней. хотя конечно можно просто обёртку для sc.exe написать, что совсем не интересно.