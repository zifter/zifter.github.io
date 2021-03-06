---
layout: post
title: Как в астрономии ведется обмен знаниями
categories: [programming, general]
tags: [python, astronomy]
comments: true
---

Как астрономы и обсерватории получают инорфмацию о взрывах сверхновых, гравитационных волнах и прочих событиях? 

Мне всегда казалось, что это некие пресс релизы с кучей excel файлов, которые рассылаются по почте. 
А если что-то срочное - то звонят по телефону или даже скайпу. Не знаю с чем связано такое глупое заблуждение, 
но я был рад узнать об [International  Virtual  Observatory  Alliance](http://www.ivoa.net/).

Смысл этого сообщества в создании "виртуальной обсерватории" для своевременного обмена 
астрономическими событиями. 
Эти события описываются в 
[определенном формате](http://www.ivoa.net/documents/VOEvent/) (в xml =). 
Для обмена есть свой собственный протокол поверх TCP/IP - [VOEvent Transport Protocol](http://www.ivoa.net/documents/Notes/VOEventTransport/). 

И, конечно же, для всего этого есть [проект на github](https://github.com/jdswinbank/Comet).

Зачем это? И что этот проект дает?
Да все просто - обсерватории мира объединены в одну сеть и если какая-либо регистрирует 
событие о, например, гравитационных волнах, то она посылает информацию в эту сеть.
Это дает возможность другим автоматически навести свой телескоп на предполагаемый 
участок неба и в реальном времени снять другие параметры этого события.

Есть так же и [специализированные проекты](https://github.com/lpsinger/pygcn) - для получения сообщений 
конкретно о гамма-всплесках. Без проблем получилось запустить и получить нотификации о 
событиях. За 15 минут - 3 события.