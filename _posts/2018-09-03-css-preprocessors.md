---
layout:     post
title:      "CSS-препроцессоры против CSS"
subtitle:   "Не баттл, а унижение"
permalink:  /page/:title
categories: web
---

Свой первый сайт я сделал в 2000 г., около 10 лет писал на обычном CSS и очень его люблю, однако предпочитаю CSS-препроцессоры, т.к. с ними работа идёт быстрее, результат получается более масштабируемым, а порог вхождения в проект увеличивается незначительно. CSS может успешно использоваться в разработке серьезных проектов только при условии его постобработки.

Статья для тех, кому кажется, что обычный CSS способен конкурировать с CSS-препроцессорами и для тех, кто (каким-то чудом) не слышал о возможностях препроцессоров.



## Необходимое введение: компонентный подход

Мы живём в эпоху компонентного подхода: интерфейс проекта делится на компоненты (кнопки, вкладки, записи...), которые можно использовать много раз, вкладывать друг в друга, модифицировать. Проще всего добиться этого — использовать __методологию__ БЭМ, в которой компоненты называются Блоками. Более сложные варианты (CSS-модули, CSS-in-JS), обычно завязаны на __технологии__ (фреймворки, вроде React).

Удобно делить исходный код на несколько папок (по имени компонента), в каждой из которых лежит всё, что нужно компоненту для работы:

```bash
# некий сферический проект в вакууме
📁blocks/
  📁 blog-post/
    📁 bg-img/
    📄 blog-post.scss
    📄 blog-post.js
    📄 blog-post.pug
    📄 tests.js
    📄 readme.md
  📁 btn
  📁 tabs
```

Это позволяет:

- **Безопасно включать и отключать компоненты**. Когда компонент не используется, его стили, скрипты и пр. не загружаются посетителю (зависит от организации сборки).
- **Иметь несколько «сборок» (стилевых и других файлов), отдаваемых в браузер для разных частей проекта**. К примеру, если на сайте есть публичная и приватная части, то стили, скрипты и спрайты приватной части можно подключать отдельно, чтобы не тратить время на их загрузку и обработку для неавторизованных посетителей.
- **Безопасно модифицировать компоненты**. Меняя стили одного компонента, крайне сложно сломать другой компонент (но, если постараться, можно).
- **Быстро находить точку редактирования**. Когда известно название редактируемого класса, находим файл с тем же именем (<kbd>Ctrl + P</kbd> во многих редакторах) и редактируем. Ориентироваться по коду быстрее и проще, т.к. сам файл имеет меньший размер.
- **Использовать компоненты в других проектах**. __Базовые__ стили многих элементов (тех же кнопок) одинаковы от проекта к проекту. Копируем блок из стартовой библиотеки и дописываем нужные проекту стили.
- **Собирать свою библиотеку компонентов**. Компоненты, из которых состоят страницы сайтов, часто повторяются: кнопки, соц. ссылки, список пользователей, карточка товара, поле ввода текста с описанием. Можно один раз написать для компонента относительно универсальную разметку, стили, javascript для применения на любом проекте и не изобретать велосипед.



## CSS-препроцессоры

Это языки написания стилей, похожие на CSS, но с возможностями, которых не хватает в CSS. Напрямую в браузере не работают, нуждаются в компиляции, результатом которой является CSS-файл. Как это работает: вы запускаете какое-либо ПО (в моём случае — [gulp](https://gulpjs.com/) или [webpack](https://webpack.js.org/)), оно следит за сохранением препроцессорных файлов и собирает (компилирует) из них CSS-файл. Заодно и страницу в браузере автоматически обновляет.

Актуальные CSS-препроцессоры: Sass (2006 г., лидер рынка), LESS (2009 г., весьма хорош), stylus (2010 г., прекрасен, но не разрабатывается) и PostCSS (2013 г., изначально для постпроцессинга, но с ним можно повторить почти всю функциональность настоящих препроцессоров). PostCSS используется почти во всех проектах и в виде постпроцессора.

Далее сравнение CSS-препроцессоров с обычным CSS.



## Разделение на небольшие файлы

Выше есть упоминание компонентного подхода. Действительно удобно, когда в одном стилевом файле описан лишь один компонент (блок), это ускоряет восприятие кода и поиск точки редактирования.

### В CSS есть импорты

```css
/* Два равновозможных варианта использования */
@import url("additional-style.css") screen and (orientation: landscape);
@import "print.css" print;
```

Браузер, встречая импорт, идёт по указанному пути, загружает и обрабатывает CSS-файл.

Дьявол в мелочах:

1. Если в одном CSS-файле написать импорт другого, они будут загружаться последовательно, а не параллельно, как могли бы (увеличение времени до начала рендера на медленных каналах). Это хуже, чем написать в разметке два `<link>`.
2. Любое подключение дополнительного файла к странице — дополнительный запрос к серверу (увеличение нагрузки на аппаратную часть сервера), как следствие — при средней и высокой посещаемости сайта все файлы будут отдаваться в браузер медленнее. Спасти может использование протокола HTTP2 (позволяет за один запрос получать несколько файлов), но [всего 29,6%](https://w3techs.com/technologies/details/ce-http2/all/all) из 10 млн. самых популярных сайтов используют его (данные на середину 2018 г., за последний год распространённость выросла почти в 2 раза).

### В CSS-препроцессорах импорты работают иначе

В CSS-препроцессорах импорты обрабатываются на этапе компиляции в CSS: если мы импортируем препроцессорный файл, его содержимое «вставится» вместо импорта. В итоге — один CSS файл, содержащий стили всех импортированных файлов.

```scss
// SCSS
@import 'btn.scss';    // «вставка» содержимого файла btn.scss или btn.sass
// Можно не указывать расширение файла
@import 'page-header'; // «вставка» содержимого файла page-header.scss, или page-header.sass, или page-header.css

// Если нужен CSS-импорт
// Что писать:                // Во что скомпилируется:
@import 'foo.css';            // @import url(foo.css);
@import 'foo' screen;         // @import "foo" screen;
@import 'http://foo.com/bar'; // @import "http://foo.com/bar";
@import url(foo);             // @import url(foo);
```

### Итог: импортирование в препроцессорах лучше

Без HTTP2 (менее 1/3 всех сайтов) CSS-импорты — почти всегда зло (увеличение количества запросов к серверу).



## Переменные

Это удобно и расширяемо. Записываем в переменную с именем `color-danger` предупреждающий об опасности цвет, используем переменную везде, где нужен такой цвет (цвет текста в сообщении об ошибке под текстовым полем,  бордюр текстового поля, цвет фона сообщения о критической ошибке в нижнем правом углу, фоновый цвет кнопки опасного действия и т.п.). Сменим значение переменной — получим изменения везде, где она применялась.

### В CSS есть custom properties

Это как переменные, но круче, ибо работает прямо в браузере. Если значение «переменной» изменить (по медиа-условию или javascript-ом), изменятся стили на всех селекторах, где применена «переменная». К custom properties применяется наследование и каскад.

Первый минус: custom properties не работают в Internet Explorer всех версий (разработка IE прекращена в пользу Edge, где custom properties работают) и в старых Safari (поддерживаются с 9.1 и 9.3 — для мобильного). Костыли в виде [cssnext](http://cssnext.io/) позволяют в небольшом количестве случаев решить эту проблему (глючно, поскольку плохо работают с медиавыражениями или не работают с ними вове). В любом случае, это потеря тех возможностей, благодаря которым custom properties круче, чем препроцессорные переменные.

Второй минус: custom properties не получится применить в CSS-анимациях, т.к. их значения — строки, а браузер не может (пока?) вычислить плавный переход между двумя строками. Решение этой проблемы возможны с javascript, я уверен.

```css
:root {
  --font-size: 12px;
}

body {
  font-size: var(--font-size);
}

/* rem-имитатор */
@media (min-width: 1024px) {
  :root {
    --font-size: 16px;
  }
}
```

### В CSS-препроцессорах переменные — обычные переменные

Препроцессорные переменные работают только на этапе компиляции. Это никак не мешает присвоить их значения в CSS custom properties, чтобы использовать их же как CSS-переменные.

```scss
// Внимание: хорошей практикой является вынос переменных в отдельный файл.
$font-size: 12px;

body {
  font-size: $font-size;
}
```

### Итог: CSS-переменные круче, если в ТЗ нет IE и старых Safari

Если в техническом задании упомянута поддержка IE или по метрике вы не можете игнорировать старые Safari (особенно актуально для мобильных версий), то CSS custom properties «превращаются в тыкву» — становятся ничуть не лучше препроцессорных переменных.



## Математические операции

Мат. операции и переменные созданы друг для друга. Но мат. операции ещё «гуляют налево» (в CSS).

### В CSS есть `calc()`

И он восхитителен по тем же причинам, по которым прекрасны CSS custom properties. Но `calc()` круче, поскольку [поддерживается в IE](https://caniuse.com/#feat=calc) (хотя и не без багов).

```css
/* если вы писали CSS в 2000-х, тут можно вновь немного поплакать от счастья */
.tabs {
  width: calc(100% - 3em);
}
```

Поскольку это срабатывает в браузере, в момент выполнения предыдущего примера браузер знает чему равно `3em` и ширина действительно будет `100% - 3em`.

```css
:root {
  --margin: 3em;
}

.tabs {
  width: calc(100% - var(--margin));
}
```


### В CSS-препроцессорах есть мат. операции

И `calc()` никто не запрещает использовать (в LESS придется экранировать, иначе внутри скобок произойдёт мат. операция).

```scss
// Внимание: хорошей практикой является вынос переменных в отдельный файл.
$font-size: 12px;

h1 { font-size: $font-size * 2; }
h2 { font-size: $font-size * 1.618; }
```



## Что есть в CSS-препроцессорах, чего нет в CSS

Единственное, в чём функциональность CSS превосходит препроцессоры — custom properties. Это действительно офигенно. Но что такого есть в препроцессорах, чего нет в CSS?

### Вложения

```scss
.promo {
  padding: 1em;

  h1 { // после компиляции превратится в .promo h1 {...}
    font-size: 48px;
  }

  @media (min-width: 1024px) { // после компиляции «вывернется»
    padding: 2em;
  }
}
```

Позволяет не повторять написание селектора, внутри которого нужно что-то стилизовать. Просто пишем вложенные селекторы внутри родительских. Ощутимо ускоряет работу, используется всегда.

### Амперсанд

```scss
.btn {
  color: #000;

  &:hover { // после компиляции превратится в .btn:hover {...}
    color: #808080;
  }

  &__icon { // после компиляции превратится в .btn__icon {...}
    position: absolute;
  }

  .promo & { // после компиляции превратится в .promo .btn {...}
    margin-left: 30px;
  }
}
```

Позволяет не повторять родительский селектор, быстро менять его, дублировать такие блоки. Ускоряет работу, используется всегда.

ВНИМАНИЕ: амперсанд не стоит использовать в местах разделения слов, а только в местах отделения БЭМ-элементов и модификаторов (для псевдоселекторов и псевдоэлементов — без ограничений). Подробнее — см. [Как работать с CSS-препроцессорами и БЭМ](https://nicothin.pro/idiomatic-pre-CSS/).

### Примеси

Набор правил, который можно использовать многократно.

```scss
// Создание примеси
@mixin clearfix {

  &:after {
    content: "";
    display: table;
    clear: both;
  }
}

// Применение примеси
.promo__wrapper {
  @include clearfix;
  padding: 1em;
}
```

В случаях, когда есть набор из нескольких свойств, одинаковых для многих селекторов, можно вынести такой набор в примесь и вызывать её для нужных селекторов. При компиляции на месте вызова примеси появятся её стили. Применяется умеренно.

Раньше активно применялось для добавления наборов правил с вендорными префиксами (но сейчас есть Autoprefixer), сейчас — для добавления правил ячейки модульной сетки, т.к. в примесь можно передать параметры и она вернет разные свойства в зависимости от переданных параметров ([пример](https://github.com/nicothin/NTH-start-project/blob/master/src/scss/mixins/grid-mixins.scss)) и для других целей.

### Условия

```scss
// Параметризированная примесь. При вызове ожидает передачи параметра — цвета.
@mixin mixin($color) {
  color: $color;

  @if (lightness($color) >= 50%) {
    background-color: black;
  }
  @else {
    background-color: white;
  }
}
```

Добавляют логику, используются, чаще всего, в примесях. Вне примесей (да и в целом) используются редко.

### Циклы

```scss
$columns: 12;

@for $i from 1 through $columns {
  .col-#{$i} { width: 100% / $columns * $i; }
}
```

`@for`, `@while`, `@each`. Удобны при использовании типов данных, напоминающих массивы (см. пример ниже в разделе о типах данных). Используются относительно редко.

### Функции

Есть встроенные функции для математических операций (округления, получение наибольшего, генератор случайных чисел и пр.), операций с цветами (осветление, насыщение и пр.), строками, массивами. Удобны при написании фреймворков. Используются умеренно.

```scss
$base-color: #AD141E;

.btn {
  background-color: $base-color;

  &:hover {
    lighten( $base-color, 10% ); // тот же цвет, но на 10% светлее
  }
}
```

В черновике CSS Color Module level 4 есть функции работы с цветом. Когда будут готовы для использования в реальных проектах — не ясно (вероятно, когда IE умрёт). Сейчас (осень 2018) поддержи нет никакой.

Можно добавлять свои функции, [с `rem` и пикселями](https://www.youtube.com/watch?v=Dk2Vyh5PRcQ).

```scss
@function rem($px) {
  @return ($px / 16) + rem;
}

h1 { font-size: rem(32); }
```

Позволяют дописать любую дополнительную логику. Используются относительно редко.

### Расширения (экстенды)

```scss
.error {
  border: 1px #ff0;
  background-color: #fdd;
}

.fatal-error {
  @extend .error; // в CSS будет взят .fatal-error и дописан через запятую к .error
  border-width: 5px;
}
```

Вредная и неочевидная возможность расширять одни селекторы другими. Используется редко. Не используйте это.

### Типы данных

Помимо чисел и строк, во многих препроцессорах есть и составные типы данных (массивы).

```scss
$status-colors: (
  primary: #3971ED,
  success: #27BA6C,
  info:    #03a9f4,
  warning: #FF8833,
  danger:  #ff1a1a
);

.message {

  @each $status, $color in $status-colors {

    &--#{$status} {
      background: $color;
    }
  }
}
```

Удобны при работе с наборами свойств, которые можно перебирать в цикле (цвета, брейкпоинты). Используются относительно редко.



## Предупреждения

Не пытайтесь программировать на CSS-препроцессорах. Особенно, если не умеете программировать! В больших возможностях — большая сила и большая опасность выстрелить себе в ногу. См. [Как работать с CSS-препроцессорами и БЭМ](https://nicothin.pro/idiomatic-pre-CSS/).

Пишите код так, как будто сопровождать его будет склонный к насилию психопат, который знает, где вы живёте ([Стивен Макконнелл](https://ru.wikipedia.org/wiki/%D0%9C%D0%B0%D0%BA%D0%BA%D0%BE%D0%BD%D0%BD%D0%B5%D0%BB%D0%BB,_%D0%A1%D1%82%D0%B8%D0%B2)).



## Итого: преимущества CSS-препроцессоров

- Ускоряют работу.
- Облегчают модификацию проекта.
- Упрощают компонентный подход.



## Итого: недостатки CSS-препроцессоров

- Требуют знание языка, весьма похожего на CSS.
- Требуют компиляции (дополнительного ПО).



## Выводы

1. В реальной жизни сверстать что-то без процессинга стилей (используя только CSS) можно, но это будут проекты уровня «Сайт моего хомячка». Изучайте CSS-препроцессоры.
2. Изучайте новые возможности CSS. Они классные.
3. CSS-препроцессоры — не конкурирующая с CSS технология, но дополняющая её.



## P.S.

Вышесказанное относится (в основном) к проектам, на которых не используются фронтенд-фреймворки. Однако их всё больше и с ними приходят замечательные возможности добавления стилей к нашим DOM-элементам — CSS modules, CSS-in-JS. «На подходе» и веб-компоненты как стек технологий, поддерживаемых браузерами (полная изоляция компонента и пр. плюшки). Подходы часто комбинируются.
