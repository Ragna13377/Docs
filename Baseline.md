# Содержание

- [Baseline 2023](#baseline-2023)
  - [HTML & CSS](#html--css)
  - [JavaScript & Web API](#javascript--web-api)
- [Baseline 2024](#baseline-2024)
  - [HTML & CSS](#html--css-1)
  - [JavaScript & Web API](#javascript--web-api-1)
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

## Baseline 2023

### HTML & CSS

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

---

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

### JavaScript & Web API

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

### HTML & CSS

У нативного аккордеона `<details>` появилась возможность оставлять открытым только один блок, если нескольким элементам задан одинаковый `name`.
```html
<details name="accordion">
  <summary>First</summary>
  <p>…</p>
</details>
<details name="accordion">
  <summary>Second</summary>
  <p>…</p>
</details>
```

`scrollbar-gutter` - позволяет зарезервировать место для скролбара, при его отсутствии, чтобы избежать сдвига макета.
* stable - место под скроллбар резервируется даже при отсутствии переполнения
* both-edges - аналогично stable, но также резервируется место симметрично с другой стороны экрана
* auto - стандартное поведение

`scrollbar-width` - ширина скролл-бара (`auto`, `thin`, `none`)

---

`backdrop-filter` - накладывает визуальные эффекты (blur, contrast и т.д.) на фон за элементом

---

`font-size-adjust` - подстраивает размер fallback-шрифта так, чтобы он выглядел ближе к основному шрифту по выбранной метрике.  

---

`::target-text` - псевдоэлемент для стилизации текста, к которому браузер проскроллил страницу через text fragments (`#:~:text=...`)

---

CSS relative color syntax - возможность через ключевое слово `from` создавать новый цвет на основе другого цвета, получая доступ к его каналам (`r`, `g`, `b`, `h`, `s`, `l`, `alpha` и т.д.) и изменяя их.
Пример:  
`rgb(from green b r g)`

![](https://developer.chrome.com/static/blog/css-relative-color-syntax/image/a-diagram-the-syntax-rgb-3bffe6c436572.png)

[Примеры](https://developer.chrome.com/blog/css-relative-color-syntax)

---

`@starting-style` - директива для анимирования entry transitions элемента перед его появлением/рендером, в том числе для дискретных свойств (`display`, `visibility`, `mix-blend-mode` и аналогов).  
Для анимации дискретных свойств  дополнительно нужно задать `transition-behavior: allow-discrete` или в шорткате `transition`  

```css
#target {
  display: block;
  opacity: 1;
  transition:
    opacity 0.5s ease,
    display 0.5s ease allow-discrete;
  @starting-style {
    opacity: 0;
  }
}

#target.hidden {
  display: none;
  opacity: 0;
}
```

---

`@property <custom-property-name> { syntax, inherits, initial-value }` - директива для регистрации типизированных кастомных CSS-свойств, где  
* `syntax` - допустимый тип значения ("<angle>", "<color>", "<image>" и т.д.) [Полный список значений](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@property/syntax)
* `inherits` - наследуемость свойства (boolean)
* `initial-value` - значение по умолчанию, если свойство не задано или невалидно

```css
@property --customColor {
  syntax: "<color>";
  inherits: false;
  initial-value: red;
}

div {
  background-color: var(--customColor);
}
```

---

`color-interpolation` - свойство определяющее цветовое пространство `linearGradient` и `radialGradient` в SVG (auto, sRGB, linearRGB)  

---

`light-dark(lightColor, darkColor)` - функция, выбирающая один из двух цветов в зависимости от текущего `color-scheme`  
```css
/* Было */
:root {
  --primary-color: #ccc;
}

@media (prefers-color-scheme: dark) {
  :root {
    --primary-color: #333;
  }
}

/* Стало */
:root {
  /* для работы функции light-dark необходимо задать color-scheme  */
  color-scheme: light dark;
  background-color: light-dark(#ccc, #333);
}
```

---

Новые функции CSS Math:
* `round()` - округление по заданной стратегии и шагу
* `rem()` - остаток от деления со знаком делимого
* `mod()` - остаток по модулю со знаком делителя

---

`offset-position` - стартовая позиция элемента для анимации/движения, заданного через `offset-path`.

---

`align-content` - позволяет выравнивать содержимое блочных и табличных контейнеров по вертикали без перевода контейнера в `flex` или `grid`.

[Вернуться к содержанию](#содержание)

### JavaScript & Web API

Для `Set` добавлены методы операций над множествами (`union`, `intersection`, `difference`, `symmetricDifference`) и методы сравнения множеств (`isSubsetOf`, `isSupersetOf`, `isDisjointFrom`).  
[Примеры](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)

---

Добавлены методы `resize()` и `transfer()` для `ArrayBuffer`

---

`CustomStateSet` - `Set`-like коллекция внутренних состояний custom element, доступная через `ElementInternals.states`.  
Используется для хранения кастомных состояний элемента и их последующего использования в CSS через `:state(...)`.  
[MDN](https://developer.mozilla.org/en-US/docs/Web/API/CustomStateSet)

---

Добавлен `Intl.Segmenter` для правильной разбивки текста на части с учетом языка (на слова, предложения или графемы)  

---

`Object.groupBy(items, callbackFn)`, `Map.groupBy(items, callbackFn)` - методы группировки элементов, где  
* `items` - итерируемая сущность
* `callbackFn(element, index)` - функция группировки. Должна возвращать строку/символ указывающую на группу элемента  
Значения, не соответствующие разрешенным типам, приводятся к строке. 

Возвращают `null`-prototype объект / `Map` с группированными элементами. При этом не происходит глубокого копирования.  
```js
const persons = [
  { name: "Petr", age: 18 },
  { name: "Olga", age: 21 },
  { name: "Pavel", age: 15 },
  { name: "Kate", age: 27 },
];

const groupedPersons = Object.groupBy(persons, (person) => {
  return person.age >= 18 ? "adults" : "children";
});

/*
{
  adults: [
    { name: "Petr", age: 18 },
    { name: "Olga", age: 21 },
    { name: "Kate", age: 27 }
  ],
  children: [
    { name: "Pavel", age: 15 }
  ]
}
*/
console.log(groupedPersons);
```

---

`Array.fromAsync(items, mapFn, thisArg)` - метод для создания нового массива из асинхронно итерируемого, синхронно итерируемого или `array-like` объекта, где  
* `items` - `async iterable`, `iterable` или `array-like` объект
* `mapFn(element, index)` - функция преобразования элементов, может быть асинхронной
* `thisArg` - значение `this` для `mapFn`  

В отличие от `Array.from()`:  
* умеет работать с асинхронными итерируемыми объектами  
* возвращает `Promise`, который зарезолвится в массив
* если передан НЕ асинхронный итерируемый объект, то каждый добавляемый элемент сначала внутренне ожидается (`await`)
* если передан `mapFn`, то результат `mapFn` также внутренне ожидается (`await`)  

В отличие от `Promise.all()`:
* ожидает значения последовательно
* итерируется лениво и не получает следующий элемент, пока текущий не завершился

```js
function* makeIterableOfPromises() {
  for (let i = 0; i < 5; i++) {
    yield new Promise((resolve) => setTimeout(resolve, 100));
  }
}

(async () => {
  let start = Date.now();
  await Array.fromAsync(makeIterableOfPromises());
  let end = Date.now();

  console.log("Array.fromAsync:", end - start, "ms");
  // Array.fromAsync() time: ~500ms

  start = Date.now();
  await Promise.all(makeIterableOfPromises());
  end = Date.now();
  
  console.log("Promise.all:", end - start, "ms");
  // Promise.all() time: ~100ms
})();
```

> `Array.fromAsync()` не всегда корректно закрывает `sync iterable`, если ошибка возникает во время `await` значения, а не во время самой итерации.

```js
function* numbers() {
  try {
    // generator `numbers()` возвращает `Promise`
    yield Promise.resolve(1);
    // второй `Promise` завершается с ошибкой
    yield Promise.reject(new Error("Error"));
  // finally внутри generator не выполняется
  } finally {
    console.log("generator closed");
  }
}

(async () => {
  try {
    await Array.fromAsync(numbers());
  } catch (error) {
    // `Array.fromAsync()` пробрасывает ошибку, но не вызывает закрытие исходного iterator
    console.log("caught:", error.message);
  }
})();

// caught: Error
// "generator closed" не выведется

// При ручном `for...of` + `await` generator закрывается корректно, поэтому `finally` выполняется.
(async () => {
  const result = [];

  try {
    for (const value of numbers()) {
      result.push(await value);
    }
  } catch (error) {
    console.log("caught:", error.message);
  }
})();
// generator closed
// caught: Error
```

---

`Promise.withResolvers()` - метод для "внешнего управления промисом", когда сам `Promise` создается в одном месте, а выполнить или отклонить его нужно позже и в другом месте кода.  
Возвращает объект с ключами:  
* `promise` - новый `Promise`
* `resolve` - функция для выполнения `promise`
* `reject` - функция для отклонения `promise`

Более короткая запись шаблона:  
```js
let resolve, reject;
const promise = new Promise((res, rej) => {
  resolve = res;
  reject = rej;
});
```

Пример использования:  
```js
const { promise, resolve, reject } = Promise.withResolvers();

setTimeout(() => resolve("Done"), 1000);

promise.then((value) => {
  console.log(value);
});
```

---

Остальное:
* Declarative Shadow DOM - [web.dev](https://web.dev/articles/declarative-shadow-dom), [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/template#declarative_shadow_dom)
* Screen Wake Lock API - API для удержания экрана включенным во время активного сценария в веб-приложении. [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Screen_Wake_Lock_API)  


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

`:active-view-transition` - псевдокласс root-элемента для стилизации страницы, пока выполняется `view transition`.  

---

Новые единицы измерения на основе типографики документа:
* `rcap` - cap height шрифта root-элемента
* `rch` - ширина символа `0` в шрифте root-элемента
* `rex` - x-height шрифта root-элемента
* `ric` - ширина идеографического символа (`水`) в шрифте root-элемента

---

JavaScript modules in service workers - поддержка `import` / `export` внутри service worker через регистрацию с `type: "module"`.

---

`animation-composition` - CSS-свойство, определяющее, как несколько анимаций комбинируются, если одновременно влияют на одно и то же свойство:
* `replace` - новая анимация заменяет текущее значение свойства
* `add` - новая анимация добавляется к текущему значению
* `accumulate` - значения анимаций накапливаются

Полезно, при использовании `transform` / `filter`. Не нужно заново переписывать свойство, можно добавить новый эффект поверх уже существующего.

```css
.card {
  transform: translateX(40px);
  animation: float 2s ease-in-out infinite;
  animation-composition: add;
}

@keyframes float {
  from {
    transform: translateY(0);
  }
  to {
    transform: translateY(-20px);
  }
}
```

Без `animation-composition: add` анимация `transform` заменила бы исходный `translateX(40px)`.
С `add` итоговый transform комбинируется: элемент сохраняет смещение по `X` и одновременно анимируется по `Y`.

---

Navigation API - современный Web API для работы с навигацией страницы и history stack. (замена History API)  

В сравнении с History API:
* единое событие `navigate` для разных типов переходов
* можно перехватывать переходы  через `event.intercept()`
* удобнее работать с history entries, а не только с текущим URL
* лучше подходит для SPA, form navigation, scroll restoration и View Transitions

Основные методы:
* `navigate(url)` - переход к новому адресу
* `back()` - назад по истории
* `forward()` - вперед по истории
* `reload()` - перезагрузка текущей записи
* `traverseTo(key)` - переход к конкретной записи history по ключу

`navigate()` и другие методы навигации возвращают объект с промисами:
* `committed` - выполняется, когда URL уже изменен и navigation entry создана
* `finished` - выполняется после полного завершения навигации

`NavigateEvent` - событие Navigate API, срабатывающее для любых типов переходов, в том числе из History API (`History.go`)  
Свойства:  
* `canIntercept` - можно ли перехватить текущую навигацию (`Boolean`)
* `destination` - объект с данными [NavigationDestination](https://developer.mozilla.org/en-US/docs/Web/API/NavigationDestination) о destination URL
* `downloadRequest` - имя скачиваемого файла при download navigation (`<a>` или `<area>`) иначе `null`
* `formData` - данные формы [FormData](https://developer.mozilla.org/en-US/docs/Web/API/NavigateEvent/formData) при POST запросе иначе `null`
  `hashChange` - является ли переход только сменой hash/fragment (`Boolean`)
* `hasUAVisualTransition` - была ли браузером уже выполнена visual transition (`Boolean`)
* `info` - произвольные данные, переданные при навигаци или `undefined`, если данные не переданы
* `navigationType` - тип навигации: `push`, `replace`, `reload`, `traverse`
* `signal` - [AbortSignal](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal), который abort-ится при отмене навигации
* `sourceElement` - [элемент](https://developer.mozilla.org/en-US/docs/Web/API/Element), инициировавший навигаци
* `userInitiated` - была ли навигация запущена пользователем (`Boolean`)  

Методы:  
* `intercept(options)` - перехватывает навигацию и превращает ее в same-document navigation
* `scroll()` - вручную запускает browser-managed scroll restoration

События завершения:
* `navigatesuccess` - срабатывает после успешного завершения навигации
* `navigateerror` - срабатывает при ошибке в процессе навигации

### Февраль 2026

`shape()` - новая функция для задания формы в `clip-path` и траектории в `offset-path`  
В отличие от `path()`, использует более удобный CSS-синтаксис, поддерживает разные CSS units и math functions (`calc()`, `max()`, `abs()`).

---

Trusted Types API — это механизм защиты от DOM-based XSS, который при включенном CSP (`require-trusted-types-for`) запрещает передавать обычные строки в опасные места DOM (innerHTML, outerHTML, insertAdjacentHTML, script.src и т.д.)  
Вместо строки нужно передавать специальный объект: [TrustedHTML](https://developer.mozilla.org/en-US/docs/Web/API/TrustedHTML), [TrustedScript](https://developer.mozilla.org/en-US/docs/Web/API/TrustedScript), [TrustedScriptURL](https://developer.mozilla.org/en-US/docs/Web/API/TrustedScriptURL), созданные через trusted policy    
```js
const target = document.querySelector("#output");

const policy = trustedTypes.createPolicy("secure-policy", {
  createHTML(input) {
    return DOMPurify.sanitize(input);
  },
});

const safeHtml = policy.createHTML(userInput);

const userInput = `<img src=x onerror=alert(1)>`;
// При передаче обычной строки при включенном CSP выкинет ошибку
element.innerHTML = safeHtml;
```

---

Новые методы `Map`: `getOrInsert()`, `getOrInsertComputed()`  
* `getOrInsert(key, defaultValue)` - возвращает значение по ключу. Если ключа нет, вставляет `defaultValue` и возвращает его.  
* `getOrInsertComputed(key, callback(key))` - возвращает значение по ключу. Если ключа нет, вызывает `callback(key)`, вставляет результат и возвращает его.  

```js
// Было
if (!map.has(key)) {
  map.set(key, []);
}
map.get(key).push(value);

// Стало
map.getOrInsert(key, []).push(value);
```

>При использовании `getOrInsert()` значение по умолчанию вычисляется каждый раз, даже когда ключ в Map  


```js
const map = new Map([["bar", "foo"]]);

function createDefaultValue(key) {
  console.log(`Creating default for ${key}`);
  return `default for ${key}`;
}

map.getOrInsert("bar", createDefaultValue("bar"));
// лог будет, хотя значение уже есть

map.getOrInsertComputed("bar", createDefaultValue);
// лог не будет, потому что callback не вызовется
```

---

`dirname ` - атрибут для `<input>`, `textarea`, который при отправке формы добавляет отдельное поле с направлением введенного текста (`ltr` или `rtl`).

### Март 2026

Новое семейство шрифтов `math` для отображения математических выражений

---

Добавлен новый метод `concat()` в `Iterator` для объединения нескольких итерируемых сущностей: `Iterator.concat(it1, it2)`  
```js
// Было
const result = [...iter1, ...iter2];

// Стало
const result = Iterator.concat(iter1, iter2);
```

```js
const map1 = new Map([["a", 1], ["b", 2]]);
const map2 = new Map([["a", 5], ["d", 4]]);

const map = new Map(Iterator.concat(map1, map2, map3)); // Map(5) {'a' => 5, 'b' => 2, 'd' => 4}
```

---

`text-indent: each-line` - отступ первой строки блочного контейнера и каждой строки после принудительного переноса (не влияет на мягкий перенос)
`text-indent: hanging` - отступ для всех строк кроме первой  

`hyphens` - CSS свойство, управляет переносом слов: `none`, `manual`, `auto`.
`hyphenate-character` - CSS свойство, задает символ, который будет использоваться в конце строки при переносе слова.

`contain-intrinsic-size` - задает браузеру примерный размер элемента, пока его содержимое не отрисовано полностью. (шорткат для `contain-intrinsic-width`, `contain-intrinsic-height`)
Уменьшает сдвиг макета.

`image-set()` - CSS-функция для выбора наиболее подходящего изображения в зависимости от характеристик устройства, например плотности пикселей. Аналог `srcset` для CSS-фоновых изображений.

`navigator.storage` - API для проверки quota и persistent storage, доступного объема хранилища и запроса persistent storage, чтобы браузер не удалял данные сайта при нехватке места.

`overflow-block`, `overflow-inline` - media features для определения того, как устройство обрабатывает переполнение viewport по блочной и inline осям.

`update` - media feature, определяющая, как часто устройство может обновлять отображение контента (`fast`, `slow`, `none`).

---

Остальное:  
* Добавлены новые события для акселерометра и гироскопа - [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Device_orientation_events)
* Readable byte streams - [MDN](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStreamBYOBReader)
* Reporting API - [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Reporting_API)
* WebTransport API - low-level HTTP/3 transport API, более гибкая альтернатива WebSockets для streams и datagrams. [MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebTransport_API)

[Вернуться к содержанию](#содержание)
