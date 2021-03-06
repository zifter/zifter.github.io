---
layout: post
title: Немного про приведение типов или почему C# не кроссплатформенный  
categories: [programming]
tags: [C#, bug, float point, decimal]
comments: true
---

В очередной раз столкнулся с проблемой в математике с плавающей точкой, но в этот раз в C#.

Есть код, который должен посчитать бонус ускорения для времени.  
В конкрентном случае, где был словлен баг, нужно рассчитать бонус в 10% для 550 секунд. 
Имеем следующий код:
~~~cpp
int time = 550;
float bonus = 1.1f;

var resultValue = time / bonus;
var value1 = (int) resultValue;
~~~

И почти такой же, но немного сокращенный, так как нет смысла держать промежуточную переменную:
~~~cpp
int time = 550;
float bonus = 1.1f;

var value2 = (int)(time/ bonus);
~~~
Какой значение в реальном мире мы хотим получить? 500, вроде очевидно.

Казалось бы, ``value1`` и ``value2`` должны быть равны, но нет! 
Значение ``value1`` равно **500**, а вот ``value2`` **499**. 

**Почему так?**

Тип выражения ``time/bonus`` - это ``float``, так как ``bonus`` имеет такой тип и ``(time/bonus).GetType()`` честно возвращает ``System.Single``.

Приведение к ``int`` отбрасывает значимую часть. 
Вo ``float`` значение выглядит как ``500.0000002``, отбрасываем - получаем 500. *Идеально.*

**НО**

Компилятор волен (т.е. зависит от реализации, а точнее от платформы, [section 11.2.2](https://www.ecma-international.org/publications/files/ECMA-ST/ECMA-334.pdf)) промежуточные значения оставлять с более высокой точностью, 
т.е. во втором случае это ``double``, и результат деление в итоге выглядит как ``499.99999998``. 
Отбрасываем значимую часть и получаем 499!

По [ссылке](https://stackoverflow.com/questions/8911440/c-sharp-float-expression-strange-behavior-when-casting-the-result-float-to-int) все, в принципе, хорошо описывается.

И да, для расчета экономических параметров стоит использовать все-таки ``Decimal``.  