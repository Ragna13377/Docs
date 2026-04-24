# Содержание

- [Baseline 2023](#baseline-2023)
  - [HTML](#html)
  - [CSS](#css)
  - [JS](#js)
- [Baseline 2024](#baseline-2024)
- [Baseline 2025](#baseline-2025)
  - [Январь 2025](#январь-2025)
  - [Февраль 2025](#февраль-2025)
  - [Март 2025](#март-2025)
  - [Апрель 2025](#апрель-2025)
  - [Май 2025](#май-2025)
  - [Июнь 2025](#июнь-2025)
  - [Июль 2025](#июль-2025)
  - [Август 2025](#август-2025)
  - [Сентябрь 2025](#сентябрь-2025)
  - [Октябрь 2025](#октябрь-2025)
  - [Ноябрь 2025](#ноябрь-2025)
  - [Декабрь 2025](#декабрь-2025)
- [Baseline 2026](#baseline-2026)
  - [Январь 2026](#январь-2026)
  - [Февраль 2026](#февраль-2026)
  - [Март 2026](#март-2026)
  - [Апрель 2026](#апрель-2026)
  - [Май 2026](#май-2026)
  - [Июнь 2026](#июнь-2026)
  - [Июль 2026](#июль-2026)
  - [Август 2026](#август-2026)
  - [Сентябрь 2026](#сентябрь-2026)
  - [Октябрь 2026](#октябрь-2026)
  - [Ноябрь 2026](#ноябрь-2026)
  - [Декабрь 2026](#декабрь-2026)

## Baseline 2023

### HTML

`inert` - атрибут, отключающий интерактивность элемента и всего его subtree. [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/inert)  
Блокирует `click`/`focus` события, отключает выделение (`user-select`) и редактирование (`input`, `textarea`, `contenteditable`), убирает subtree из дерева доступности (AOM) и исключает из поиска по странице (`Ctrl+F` и аналоги).     
Используется для:  
* скрытых или находящихся за пределами экрана элементов (сайдбар, часть карусели за экраном)  
* элементов, которые не должны быть интерактивными (блоки элементов на форме)

```html
<div inert>
  <label for="button2">Button 2</label>
  <button id="button2">I am inert</button>
</div>
<!-- inert-элементы не имеют собственной стилизации -->
<style>
  [inert], [inert] * {
    opacity: 0.5;
    pointer-events: none;
    cursor: default;
    user-select: none;
  }
</style>
```  

---

Поддержка `<source>` для `<video>` и `<audio>`. [Source](https://scottjehl.com/posts/using-responsive-video/)  
В отличие от `<picture>`, при использовании `<source>` в тегах `<video>` и `<audio>`:
* нельзя использовать `srcset`, только `src`
* `source` единственный элемент внутри тега, нет фолбека в в виде аналога `img` в `picture`
* браузер не меняет один `<source>` на другой при ресайзе

---

Добавлена поддержка lazy-loading для `iframe`  
`<iframe loading="lazy"></iframe>`

---

Добавлен семантический тег `<search>` для полей поиска и фильтрации  

[Вернуться к содержанию](#содержание)

### CSS

Добавлен нативный nesting в CSS, аналог вложенности в препроцессорах (`SASS`, `LESS` и т.д.).
```css
.outer {
  div {
    .inner {
      /* styles */
      @media (width <= 100px) {
      /*  some styles */
      }
    }
  }
}
```  

---

`Container Query` - возможность применять media-выражения относительно размера контейнера, а не только viewport. [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Containment/Container_queries)  
* `container-type` - ось зависимостей для размеров запросов   
  * `size` - зависимость от inline/block размеров контейнера
  * `inline-size` - зависимость от inline размеров контейнера
  * `normal` - не делает элемент size query container-ом (не создает зависимости от inline/block контейнера)
* `container-name` - имя контекста контейнера  
* `container` - шорткат для `container-name / container-type`  
Добавлены новые единицы измерений, зависящие от размеров контейнера: `cqw, cqh, cqi, cqb, cqmin, cqmax`

```css
.card-container {
  container: card / inline-size;
}

.card-child {
  display: grid;
  grid-template-columns: 1fr 1fr;
}

@container (max-width: 400px) {
  .card-child {
    grid-template-columns: 1fr;
  }
}
```  

---

`:user-valid`, `:user-invalid` - аналоги `:valid` и `:invalid`, которые срабатывают только после взаимодействия пользователя с элементом.  
![](https://web.dev/static/articles/user-valid-and-user-invalid-pseudo-classes/image/form-state-screenshots_856.png)

---

`:dir` - псевдокласс на основе направления текста  

---

Селектор `:has()` - возможность выбирать элемент в зависимости от его потомков или содержимого.  
`div:has(a)` - стиль применится к `div`, внутри которого есть `a`.  

---

Селекторам `:nth-child(An+B [of S]?)` и `:nth-last-child(An+B [of S]?)` добавлена поддержка микросинтаксиса `An+B` и `of S`.
* `An+B` - позволяет выбрать элементы в DOM по их индексу  
  `:nth-child(-n+5)` - элементы с 1 по 5; `:nth-child(2n+3)` - элементы 1, 3, 5 и т.д.
* `of S` - позволяет предварительно фильтровать элементы по нужному селектору  
  `:nth-child(2 of .selected)` - выбор 2 элемента с классом `selected`

[Примеры](https://developer.chrome.com/docs/css-ui/css-nth-child-of-s?hl=ru)

---

Subgrid - добавлено наследование размеров родительского grid при указании `grid-template-columns: subgrid` и/или `grid-template-rows: subgrid`.   

[Примеры](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Grid_layout/Subgrid)

---

Добавлена поддержка новых цветовых пространств и функций в `color()` / `color-mix()`.
```css
--srgb: color(srgb 1 1 1);
--srgb-linear: color(srgb-linear 100% 100% 100% / 50%);
--display-p3: color(display-p3 1 1 1);
--rec2020: color(rec2020 0 0 0);
--a98-rgb: color(a98-rgb 1 1 1 / 25%);
--prophoto: color(prophoto-rgb 0% 0% 0%);
--xyz: color(xyz 1 1 1);
```

Добавлена поддержка функций `sin(), cos(), tan(), asin(), acos(), atan(), atan2()` в `calc()`.

[Примеры использования функций](https://web.dev/articles/css-trig-functions)

---

Новый синтаксис в media-выражениях  
`@media (max-width: 30em)` можно заменить на `@media (width <= 30em)`  
`@media (min-width: 400px) and (max-width: 600px)` можно заменить на `@media (400px <= width <= 600px )`  

---

Добавлена поддержка построения более сложных linear-анимаций. [webdev](https://developer.chrome.com/docs/css-ui/css-linear-easing-function?hl=ru)  
Из доступных на текущий момент визуальных редакторов: [линк](https://linear-easing-generator.netlify.app/)  
_Пока не найден более удобный draw-first редактор_  

---

Новые единицы измерения: `cap`, `rcap`, где  
`cap` - относительно высоты шрифта верхнего регистра текущего элемента  
`rcap` - относительно высоты шрифта верхнего регистра root-элемента  

---

Добавлена единица измерения `lh`, зависящая от `line-height`.  
```css
/* Было */
--line-height: 1.5;
--lh: calc(1em * var(--line-height));
line-height: var(--line-height);

/* Стало */
line-height: 1lh;
```

[Source](https://danburzo.ro/line-height-lh/)  

---

Добавлено свойство `counter-set` - изменение текущего значения счетчика на указанное: `counter-set: timerName 10`  
Если указан несуществующий счетчик, то работает как `counter-reset`

Добавлена поддержка пользовательских стилей счетчиков через `@counter-style`.

`@counter-style` - правило для стилизации счетчиков  
* `system` - алгоритм построения marker representation (`cyclic, numeric, alphabetic, symbolic, additive, fixed, fixed <integer>, extends`)
* `negative` - символы/строки, добавляемые к отрицательным значениям счетчика
* `prefix` - символ/строка, добавляемые перед marker representation
* `suffix` - символ/строка, добавляемые после marker representation
* `range` - диапазон применения стиля; вне диапазона используется `fallback`
* `pad` - минимальная длина marker representation, например двузначное отображение символов: 01)
* `fallback` - резервный стиль используемый при невозможности отобразить текущий стиль
* `symbols` - набор символов, строк или `url(...)`, из которых строится marker representation
* `additive-symbols` - набор пар `вес + символ`, используемых в `system: additive` (например римские цифры)
* `speak-as` - способ озвучивания screen reader-ами `auto, bullets, numbers, words, spell-out, <counter-style-name>`

```css
ol.custom {
  list-style: custom-badge;
}

@counter-style custom-badge {
  system: fixed;
  symbols: "A" "B" "C";
  negative: "(" ")";
  prefix: "[";
  suffix: "] ";
  range: 1 3;
  pad: 2 "0";
  fallback: decimal;
  speak-as: numbers;
}
```

---

Свойство `display` поддерживает комбинацию двух значений (`outer inner`).  
* `display: grid` аналог `display: block grid`  
* `display: inline-block` аналог `display: inline flow-root`  

Новый тип `flow-root` создает `BFC` у block box, в отличие от `inline-block`, который создает `BFC` у inline-level box.  
>**BFC** - это отдельный режим раскладки блока, при котором контейнер лучше изолирует свое содержимое и корректно учитывает `float` внутри себя.

[Вернуться к содержанию](#содержание)

### JS

`Import Maps` - возможность давать короткие имена модулям и мапить их на URL.

```html
<script type="importmap">
{
  "imports": {
    "browser-fs-access": "https://unpkg.com/browser-fs-access@0.33.0/dist/index.modern.js"
  }
}
</script>


<script type="module">
  import { fileOpen } from "browser-fs-access";
</script>
```

---

Добавлена возможность предзагрузки модулей через `rel="modulepreload"`.  В отличие от обычного `preload`, браузер знает, что грузится именно ES module, и корректнее подготавливает его к выполнению.  
`<link rel="modulepreload" href="/app.js">`  

---

`Compression Streams API` - встроенный browser API для сжатия и распаковки потоков данных (`gzip`, `deflate`, `deflate-raw`) через `CompressionStream` и `DecompressionStream`.  
[web.dev](https://web.dev/blog/compressionstreams) | [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Compression_Streams_API)

---

`OPFS` (`Origin Private File System`) - приватная файловая система браузера, доступная только текущему origin. Полезна для быстрого хранения и обработки файловых данных внутри веб-приложения без участия пользовательской файловой системы.  
[web.dev](https://web.dev/articles/origin-private-file-system) | [MDN](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API/Origin_private_file_system)

---

Остальное:
* Constructable Stylesheets - [web.dev](https://web.dev/articles/constructable-stylesheets), [MDN](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleSheet/CSSStyleSheet)
* OffscreenCanvas - [web.dev](https://web.dev/articles/offscreen-canvas), [MDN](https://developer.mozilla.org/en-US/docs/Web/API/OffscreenCanvas)
* Маскирование в CSS - [статья](https://ishadeed.com/article/css-masking/)

[Вернуться к содержанию](#содержание)

## Baseline 2024

[Вернуться к содержанию](#содержание)

## Baseline 2025

### Январь 2025

### Февраль 2025

### Март 2025

### Апрель 2025

### Май 2025

### Июнь 2025

### Июль 2025

### Август 2025

### Сентябрь 2025

### Октябрь 2025

### Ноябрь 2025

### Декабрь 2025

[Вернуться к содержанию](#содержание)

## Baseline 2026

### Январь 2026

### Февраль 2026

### Март 2026

### Апрель 2026

### Май 2026

### Июнь 2026

### Июль 2026

### Август 2026

### Сентябрь 2026

### Октябрь 2026

### Ноябрь 2026

### Декабрь 2026

[Вернуться к содержанию](#содержание)
