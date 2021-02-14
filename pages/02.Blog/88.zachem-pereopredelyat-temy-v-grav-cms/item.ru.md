---
title: 'Зачем переопределять темы в Grav CMS'
media_order: grav-logo.png
date: '19:27 11.04.2019'
taxonomy:
    category:
        - Blog
    tag:
        - GravCMS
---

Сегодня вышло обновление используемой мной темы оформления [Quark](https://getgrav.org/downloads/themes). В результате чего все изменения внесенные в файлы темы перезписались свежей версией. Чтобы опять не попадать в такую ситуацию ребя с проекта [gravcms.ru](http://gravcms.ru/) написали [короткую инструкцию](http://gravcms.ru/doc/theme) по переопредению стандартных тем. Пока петух не клюнул в это твопрос даже непытался погружаться в этот вопрос.

[Переопределение стандартного шаблона Grav](http://gravcms.ru/doc/theme)

Этот вам очень пригодится, если вы хотите сохранить обновляемость, оригинальной темы, но при этом что-то изменить. Допустим изменить сайтбар.
Создайте папку user/themes/mytheme - здесь будет храниться ваша тема.
Создайте файл /user/themes/mytheme/mytheme.yaml здесь сделаем взаимосвязь темы

> streams:
> 
>  schemes:
>  
>    theme:
>    
>      type: ReadOnlyStream
>      
>      prefixes:
>      
>        '':
>        
>          - user/themes/mytheme
>          
>          - user/themes/antimatter 
>          

_user/themes/mytheme_ - mytheme - это ваша тема
_user/themes/antimatter_ - antimatter - тема которую вы переопределяете

Создайте файл /user/themes/mytheme/blueprints.yaml - укажите основные элементы темы.

> name: MyTheme
> version: 1.0.0
> description: "Extending Antimatter"
> icon: crosshairs
> author:
>  name: Team Grav
>  email: devs@getgrav.org
>  url: http://getgrav.org

Теперь можете пройти в админ панель и указать ваш новый шабон, как основной.

Создайте файл user/themes/mytheme/mytheme.php - это будет новый класс темы.

> <?php
>  namespace Grav\Theme;
>  class Mytheme extends Antimatter
>  {
>      // Some new methods, properties etc.
>  }
> ?> 

mytheme - это ваша новая тема
Antimatter - это тема донор

Теперь вы можете скопировать файл sidebar.html.twig из antimatter/templates/partials в mytheme/templates/partials и спокойной его изменять уже в вашей теме.