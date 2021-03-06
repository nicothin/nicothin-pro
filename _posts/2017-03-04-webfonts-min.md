---
layout:     post
title:      "Оптимизация шрифтов"
subtitle:   "Как получить наименьший размер шрифтовых файлов"
permalink:  /page/:title
categories: web
---

Столкнулся (в очередной раз) с некорректной работой конвертера [fontsquirrel.com](https://www.fontsquirrel.com) (для некоторых шрифтов он очень заметно деформирует символы шрифта) и решил разобраться каким образом можно самостоятельно получить оптимизированные файлы шрифта для использования при верстке сайтов.

Требования к инструментарию:

*   на выходе должен получаться минимальный размер файлов (скармливать инструменту набор символов, которые нужно оставить в шрифте),
*   отсутствие деформаций символов после конверсии,
*   работа с форматами TTF, WOFF/WOFF2,
*   кроссплатформенность,
*   бесплатность,
*   локальность (не веб-сервис).

Рассматривать буду все в разрезе использования консоли и [nodejs/npm](https://nodejs.org/en/). Если вы еще не начали использовать консольные инструменты, [самое время начать]({{ site.url }}/page/console-windows).

## Оптимизация шрифта

Задача: оставить в шрифте только те символы, которые необходимы на проекте. В моем случае, это цифры, латиница, кириллица (не целиком) и некоторые другие (кавычки, тире и т.п.) [символы](https://gist.github.com/nicothin/758b76f4785f1f8f4b154c3c86b9bc42).

Для этих целей обнаружил три инструмента:

*   [fonttools](https://github.com/fonttools/fonttools) — набор консольных инструментов, в т.ч. инструмент, оставляющий в файле шрифта только символы с указанными utf-кодами.
*   [fontforge](https://fontforge.github.io/en-US/) — бесплатный редактор шрифта (полноценный редактор с GUI).
*   [fontmin](https://github.com/ecomfe/fontmin) — набор китайских инструментов (есть консольные варианты, вебсервис и приложение с GUI).

### [fonttools](https://github.com/fonttools/fonttools)

Лучший из найденных инструментов. Требует установленного [Python](https://www.python.org/downloads/). Ставим инструмент и потом в консоли:

```bash
pyftsubset fonts/Roboto-Black.ttf --output-file=fonts/Roboto-Black--pyftsubset.ttf --unicodes-file=codes.txt
```

*   `fonts/Roboto-Black.ttf` — исходный файл,
*   `fonts/Roboto-Black--pyftsubset.ttf` — результат,
*   `codes.txt` — файл с кодами символов, которые нужно оставить в шрифте ([пример](https://gist.github.com/nicothin/93c8c6e348e4febd22a6d29e8621f377), получить свой набор можно любым [конвертером символов](https://r12a.github.io/apps/conversion/)).

Результат для TTF:

*   [Open Sans](https://nicothin.github.io/webfont_convert_test/fonts/opensans/) 212 Кб → 25 Кб.
*   [Pt Serif](https://nicothin.github.io/webfont_convert_test/fonts/ptserif/) 319 Кб → 62 Кб.
*   [Roboto](https://nicothin.github.io/webfont_convert_test/fonts/roboto/) 158 Кб → 33 Кб.
*   [Roboto Condensed](https://nicothin.github.io/webfont_convert_test/fonts/roboto_condensed/) 156 Кб → 33 Кб.
*   [Roboto Slab](https://nicothin.github.io/webfont_convert_test/fonts/roboto_slab/) 166 Кб → 35 Кб.

После конвертации в WOFF и WOFF2 размеры файлов становятся еще приятнее.

Из обнаруженных минусов:

1.  Пропущенные через инструмент шрифты при открытии стандартным инструментом просмотра шрифтов (по кр. мере, на Windows) выдают ошибку (браузер переваривает их корректно).
2.  Я получил файл нулевой длинны для одного из конвертируемых шрифтов. Странный глюк с неизвестной причиной (пришлось использовать fontforge).

### [fontforge](https://fontforge.github.io/en-US/)

Бесплатное кроссплатформенное ПО для редактирования шрифтов, в т.ч. можно выделить и очистить площадки символов, которые не нужны на проекте, радикально уменьшив размер файла.

Увы, мною не были обнаружены адекватные инструменты автоматизации, позволяющие поточно обработать несколько файлов по одной маске. А выделять символы вручную — минут по 5-7 на файл — не хочется категорически. Если подскажите способ сохранить выделение символов или как-то консольно вызвать fontforge, чтобы от взял ttf-файл, вырезал символы и сохранил, буду благодарен.

### [fontmin](https://github.com/ecomfe/fontmin)

Устанавливаем глобально и потом в консоли можно:

```bash
text="`cat subset.txt`" && fontmin -t "$text" fonts/lato.ttf > fonts/lato--fontmin.ttf
```

*   `subset.txt` — файл с символами, которые хочется оставить в шрифте ([пример](https://gist.github.com/nicothin/758b76f4785f1f8f4b154c3c86b9bc42)),
*   `fonts/lato.ttf` — исходный файл,
*   `fonts/lato--fontmin.ttf` — результат.

Одна беда: fontmin вырежет все пробелы. Либо оставит, но подпортит ширины символов. Поскольку пробельные символы могут сильно отличаться в разных шрифтах, диагноз: **малоюзабельно**.

## Конвертирование шрифта

### [ttf2woff](https://github.com/fontello/ttf2woff)

*   Конвертирует TTF в WOFF.
*   Консольный инструмент. Нужно владеть [азами работы с консолью](https://github.com/nicothin/web-development/tree/master/bash) и иметь установленный [node.js](https://nodejs.org/en/).
*   Настроек не имеет.

Ставим инструмент глобально и потом в консоли:

```bash
ttf2woff fonts/lato.ttf fonts/lato--ttf2woff.woff
```

Указывается путь и имя исходного и результирующего файла.

### [ttf2woff2](https://github.com/nfroidure/ttf2woff2)

*   Конвертирует TTF в WOFF2.
*   Консольный инструмент. Нужно владеть [азами работы с консолью](https://github.com/nicothin/web-development/tree/master/bash) и иметь установленный [node.js](https://nodejs.org/en/).
*   У меня (Windows 10) работает медленно: до пары минут на конверсию одного файла.

Ставим инструмент глобально и потом в консоли:

```bash
cat fonts/lato.ttf | ttf2woff2 >> fonts/lato--ttf2woff2.woff2
```

Указывается путь и имя исходного и результирующего файла.

## Прочие инструменты (веб-сервисы)

### [font2web.com](http://www.font2web.com) — конвертер

*   Работает не со всеми шрифтами (чёрный список обнаружить не удалось, попытка сконвертировать Open Sans привела к скачиванию архива без шрифтов).
*   Настроек не имеет.
*   Не отдаёт шрифты в формате WOFF2.
*   Работает с одним файлом за раз.

### [freefontconverter.com](http://www.freefontconverter.com) — конвертер

*   Отдаёт TTF, WOFF, WOFF2, EOT, SVG.
*   Настроек не имеет.
*   Работает с одним файлом за раз.
*   Получает TTF, который становится немного меньшего размера в результирующем архиве.

### [web-font-generator.com](https://www.web-font-generator.com) — конвертер

*   Отдаёт TTF, WOFF, EOT, SVG.
*   Настроек не имеет.
*   Работает с одним файлом за раз.
*   Показывает превью результата.
*   Умеет конвертировать в base64.
*   Иногда выдает бОльший размер файла, чем у закачанного оригинала.

### [webfont.ru](https://webfont.ru/converter) — конвертер и оптимизатор

*   Отдаёт TTF, WOFF, WOFF2.
*   **Позволяет работать только с имеющимися на проекте шрифтами, свой загрузить нельзя**.
*   Одновременная работа только с одним шрифтом (при переходе от шрифта к шрифту в рамках гарнитуры сохраняются настройки конвертера).
*   Есть настройка с указанием сохраняемых в шрифте диапазонов символов или конкретных символов, что в несколько раз уменьшает вес файла (212Кб → 27Кб для Open Sans [с минимальным набором символов](https://gist.github.com/nicothin/758b76f4785f1f8f4b154c3c86b9bc42)).
*   Умеет конвертировать в base64.
*   **Меняет визуальную плотность, портит полуовалы, деформирует вертикальные размеры букв, меняет высоту строки**.

### [fontsquirrel.com](https://www.fontsquirrel.com) — конвертер и оптимизатор

*   Отдаёт TTF, WOFF, WOFF2, EOT, SVG.
*   Работает не со всеми шрифтами (чёрный список [имеется](https://www.fontsquirrel.com/faq#blacklist), но не опубликован).
*   Множество настроек, в том числе указание диапазонов символов или конкретных символов (212Кб → 34Кб для Open Sans [с минимальным набором символов](https://gist.github.com/nicothin/758b76f4785f1f8f4b154c3c86b9bc42)).
*   С настройками по умолчанию убирает из шрифта все нелатинские символы (в т.ч. кириллические).
*   Работает с несколькими файлами за раз.
*   Умеет конвертировать в base64.
*   **Меняет визуальную плотность, портит полуовалы, деформирует вертикальные размеры букв, меняет высоту строки, для некоторых размеров «съедает» внутрибуквенные просветы**.

## Смотри так же

*   [Как сейчас нужно подключать шрифты]({{ site.url }}/page/web-fonts)
*   [Репозиторий для экспериментов](https://github.com/nicothin/webfont_convert_test)
