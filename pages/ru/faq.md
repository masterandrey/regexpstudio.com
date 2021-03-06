---
layout: redirected
redirect_to: https://regex.sorokin.engineer/ru/latest/faq.html
sitemap: false
lang: ru
ref: faq
title: ЧАсто задаваемые ВОпросы
permalink: /ru/faq.html
---

### В. Ничего не работает! Какой-то Access violation выдает!

#### О.
Пожалуйста, проверьте, что вы создали эземпляр объекта.

После того как вы описали что-то вроде:

    var r : TRegExpr; 
    
нужно создать объект: 

    r := TRegExpr.Create

### В. Как использовать TRegExpr в Borland C++ Builder?

Я не могу это сделать, потому что нет заголовочных файлов (.h или .hpp).

#### О.

* Добавьте RegExpr.pas в Ваш bcb-проект
* Откомпилируйте проект. В результате автоматически будет создан hpp-файл (сообщения об ошибках можно проигнорировать)
* Теперь Вы можете использовать класс TRegExpr в своем проекте. Не забывайте добавлять  `\#include "RegExpr.hpp"` в соответствующие cpp-файлы
* Не забудьте заменить в регулярных выражениях все символы `\\` на `\\\\`.

### В. А почему почти все выражения (даже приведенные здесь как примеры) неверно работают в Borland C++ Builder?

#### О.
Символ `\\` воспринимается в C++ как "эскейп"-символ, поэтому его самого нужно
всегда "эскейпить". Когда Вам надоест работать с кошмаром вида
`\\\\w+\\\\\\\\\\\\w+\\\\.\\\\w+`, Вы можете переопределить константу
EscChar (RegExpr.pas), например, если EscChar=`/` - то приведенное выше
выражение нужно записать как `/w+\\/w+/./w+` - немножко
непривычно\\нестандартно, но намного обозримее\\читаемее..

### В. Почему TRegExpr возвращает более одной строки?

Например, выражение <font .\*> возвращает <font, а затем - весь
оставшийся файл включая завершающий </html>...

#### О.
Для обеспечения обратной совместимости, [модификатор
/s](regexp_syntax.html#modifier_s) по умолчанию включен.

Если Вы его выключите, то мета-символ `.` перестанет совпадать с
[разделителями строк](regexp_syntax.html#syntax_line_separators) - и Вы
получите ожидаемый Вами результат.

Кстати, данное конкретное выражение было бы эффективнее переписать как
`<font (\[^\\n>\]\*)>`, (в Match\[1\] Вы получите URL).

### В. Почему TRegExpr возвращает больше чем я ожидал?

Например, выражение `<p>(.+)</p>` примененное к строке
`<p>a</p><p>b</p>` возвращает
`a</p><p>b` а не `a` как я ожидал.

#### О.
По умолчанию, все операторы работают в "жадном" (`greedy`) режиме и
пытаются захватить как можно большую часть входной строки.

Если Вам необходим "не жадный" (`non-greedy`) режим, то Вы можете
использовать либо специальные "не жадные" варианты операторов (`+?` и
т.п.) либо вообще переключить модификатор "жадности" ModifierG. Прим.:
эта возможность появилась начиная с версии 0.940).

### В. Как анализировать HTML с помощью TRegExpr

#### О.
Мне очень часто задают этот вопрос! Должен вас огорчить, корректный
ответ - "никак".

Конечно, вполне возможно (и очень удобно) использовать TRegExpr для
извлечения каких-то специфичных фрагментов HTML, я сам часто этим
пользуюсь, но если Вам нужен полноценный синтаксический разбор, то
возьмите подходящий инструмент - синтаксический анализатор, а не
регулярные выражения!

Мне не хочется занимать здесь место объяснениями почему это так - просто
поверьте или почитайте например книжку Tom Christiansen и Nathan
Torkington `Perl Cookbook` (в русском переводе вышла в серии "Библиотека
программиста" и называется просто "Perl"). Если в двух словах - мало
того что отдельные случаи даже теоретически невозможно обработать с
помощью регулярных выражений, так еще и не стоит забывать, что
регулярные выражения это достаточно ресурсоемкий
[механизм](regexp_syntax.html#mechanism) выполняющий оптимизационный
поиск, в то время как синтаксический анализ обычно линеен и работает
гораздо быстрее. Используйте подходящие инструменты для каждого вида
работ!

### В. Как мне получить все вхождения регулярного выражения а не только первое?

#### О.
Очень просто - напишите цикл, который будет вызывать метод ExecNext и
тем самым переберет все вхождения.

Пример Вы можете посмотреть в реализации метода TRegExpr.Replace или в
исходных текстах модуля
[HyperLinksDecorator.pas](#hyperlinksdecorator.html)

### В. Я хочу проверить вводимые пользователем строки. Но почему-то TRegExpr иногда возвращает True для явно ошибочных строк.

#### О.
Возможно, Вы просто забыли о том, что регулярные выражения ИЩУТ заданный
Вами шаблон во входной строке. Поэтому, если например задать шаблон типа
`\\d{4,4}` то он "сработает" не только для строк, состоящих из четырех
цифр, но и для таких строк как `12345` или даже  `что угодно 1234 и
опять что угодно`. Если Вам необходимо убедиться что во входной строке
ТОЛЬКО искомый шаблон, не забудьте обрамлять выражение метасимволами
начала и конца строки: `^\\d{4,4}$`.

### В. Почему "не жадные" повторители иногда ведут себя "жадно"?

Например, выражение `a+?,b+?`, примененное к строке `aaa,bbb` находит
`aaa,b`, в то время как я ожидал что будет найдено `a,b`, ведь первый
повторитель написан в "нежадной" форме!

#### О.
Это особенность используемой TRegExpr (а также Perl-ом и многими
Unix-скими регулярными выражениями) математики - выполняется только
"простая" оптимизация при поиске и не делается попыток найти наиболее
оптимальный вариант. В некоторых ситуациях это неудобно, однако, в
основном, это можно воспринимать как преимущество а не недостаток,
поскольку это обеспечивает большую скорость работы и предсказуемость
результатов.

Основное правило - вначале TRegExpr пытается найти соответствие
выражению начиная с текущей позиции во входном тексте. И только если это
абсолютно невозможно, передвигается на символ вперед и повторяет все
сначала. Поэтому, для выражения `a,b+?` будет найдено `a,b`, а вот в
случае `a+?,b+?` выражение не рекомендует (с помощью "нежадной" формы
повторителя) но и не запрещает вовсе совпадение более одного символа
`a`, поэтому TRegExpr, пытаясь найти соответствие для текущей позиции,
захватывает все больше этих символов, пока не находит соответствие. На
этом он завершает поиск, не пытаясь выяснить, нельзя ли, сдвинувшись
вперед, найти лучший вариант. Впрочем, строго говоря, нет способа понять
- что значит "лучший"

Вы можете также найти более детальные пояснения в 
[Синтаксис](regexp_syntax.html).

