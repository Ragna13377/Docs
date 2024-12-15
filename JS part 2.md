# Содержание
* [1. DOM](#1-dom)
	- [1.1 Поиск элемента на странице](#11-поиск-элемента-на-странице)   
	- [1.2 Относительный поиск элемента](#12-относительный-поиск-элемента)  
	- [1.3 Создание, вставка, удаление элементов](#13-создание-вставка-удаление-элементов)  
	- [1.4 Работа с атрибутами](#14-работа-с-атрибутами)  
	- [1.5 Работа с классами](#15-работа-с-классами)  
	- [1.6 Работа со стилями](#16-работа-со-стилями)  
	- [1.7 Работа с формами](#17-работа-с-формами)  
	- [1.8 Скролл](#18-скролл)  
	- [1.9 Размеры элемента](#19-размеры-элемента)  
	- [1.10 Остальное](#110-остальное)  
	- [1.11 События страницы](#111-события-страницы)  
		- [1.11.1 Распространение событий](#1111-распространение-событий)  
		- [1.11.2  Виды событий](#1112-виды-событий)  
	- [1.12 FormData](#112-formdata)  
* [2. БОМ](#2-бом) 
	- [2.1 window.history](#21-windowhistory)  
	- [2.2 window.location](#22-windowlocation)   
	- [2.3 window.navigator](#23-windownavigator)   
* [3. Drag n Drop](#3-drag-n-drop)  
	- [3.1 DataTransfer](#31-datatransfer)  
	- [3.1 DataTransferItem](#32-datatransferitem)  
	- [3.1 DataTransferItemList](#33-datatransferitemlist)  
* [4. RegExp](#4-regexp)  
	- [4.1 Флаги](#41-флаги)  
	- [4.2 Методы](#42-методы)
	- [4.3 Спецсимволы](#43-спецсимволы)
	- [4.4 Якоря](#44-якоря)
	- [4.5 Квантификаторы](#45-квантификаторы)
	- [4.6 Скобочные группы](#46-скобочные-группы)
* [5. Proxy, Reflect](#5-proxy-reflect)  
	- [5.1 Proxy](#51-proxy)  
	- [5.2 Reflect](#52-reflect)  
* [6. Классы](#6-классы)  
	- [6.1 Наследование классов](#61-наследование-классов)  
	- [6.2 Виды методов и свойств](#62-виды-методов-и-свойств)  
* [7. Дополнительно](#7-дополнительно) 
	- [7.1 Скачивание файла](#71-скачивание-файла)  

## 1. DOM

DOM - древовидная структура для управления HTML-разметкой из JS-кода. Помещается в `document` после срабатывания события `DOMContentLoaded`  
`document.readyState` - этап загрузки DOM в document:
* loading - загружается
* interactive - HTML загружен, но сторонние ресурсы(картинки, скрипты) еще загружаются
* complete - загрузка HTML и всех ресурсов завершена  

DOM состоит из узлов **(Node)**: 
1. HTML-теги **(Element)**
2. текст 
3. комментарии
4. и т.д.

NodeList - живая или статическая коллекция содержащая любые типы узлов DOM (Node)  
HTMLCollection - всегда живая коллекция содержащая только элементы (Element)  
_Живые коллекции/элементы обновляются в реальном времени при изменении элементов_  

[Вернуться к содержанию](#содержание)

## 1.1 Поиск элемента на странице

* `{document/Element}.querySelector(selector)` - возвращает **статический (не живой)** первый элемент с переданным селектором или null
* `{document/Element}.querySelectorAll(selector)` - возвращает **статический (не живой)** NodeList элементов с переданным селектором
* `{document/Element}.getElementsByClassName(name)` - возвращает **живую** HTMLCollection содержащую элементы с переданным именем класса
* `{document/Element}.getElementsByTagName(tagName)` - возвращает **живую** HTMLCollection содержащую элементы с переданным тэгом
* `document.getElementById(id)` - возвращает **статический (не живой)** элемент с переданным id или null
* `document.getElementsByName(name)` - возвращает **живой** NodeList с элементами, имеющими атрибут с переданным именем

[Вернуться к содержанию](#содержание)

## 1.2 Относительный поиск элемента

* `Element.previousElementSibling, Element.nextElementSibling, Node.previousSibling, Node.nextSibling` - возвращает следующий/предыдущий элемент/узел или null
* `{document/Element}.firstElementChild, {document/Element}.lastElementChild, Node.firstChild, Node.lastChild` - возвращает первый/последний дочерний элемент/узел или null
* `{document/Element}.children, Node.childNodes` - возвращает **живую** коллекцию дочерних элементов в формате HTMLCollection/NodeList
* `Element.parentElement, Node.parentNode` - возвращает родительский элемент/узел или null
* `Element.closest(selector)` - возвращает ближайший родительский элемент по DOM дереву (начиная с самого элемента), соответствующий селектору или null
* `Node.getRootNode()` - возвращает корневой узел (HTMLDocument, iframe)

[Вернуться к содержанию](#содержание)

## 1.3 Создание, вставка, удаление элементов

* `document.createElement(tagName)` - создание элемента
* `{document/Element}.append(param1, ... paramN), {document/Element}.prepend(param1, ... paramN)` - вставляет или **перемещает** (если он уже был добавлен) элемент(ы) в начало/конец родителя; вставляет текстовый контент в начало/конец родителя
* `Node.appendChild(aChild)` - вставляет **один** узел в конец родителя. В отличие от `Element.append` возвращает добавленный узел
* `Element.after(node1, ... nodeN), Element.before(node1, ... nodeN)` - вставляет или **перемещает** (если он уже был добавлен) узел(ы) после/до элемента; вставляет текстовый контент после/до элемента
>В `append, prepend, after, before` id элемента можно указывать: `before(some_id)` и `before('#some_id')`  

* `Node.insertBefore(newNode, referenceNode)` - вставляет узел `newNode` до узла `referenceNode`
* `Element.insertAdjacentElement(position, element)` - вставляет элемент в позицию (beforebegin, afterbegin, beforeend, afterend)
* `Element.insertAdjacentHTML(position, HTMLtext)` - вставляет HTML-текст в позицию (beforebegin, afterbegin, beforeend, afterend) **БЕЗ перерендера**
* `Element.insertAdjacentText(position, text)` - вставляет текстовый узел в нужную позицию (beforebegin, afterbegin, beforeend, afterend)
* `Node.textContent` - возвращает текстовый контент всех вложенных узлов без HTML / устанавливает текстовый контент вместо вложенных узлов. **Узел перерендерится**
* `Element.innerText` - аналогично textContent
>Отличие: `innerText` - покажет только видимый текст (без `display: none` дочерних элементов), а `textContent` любой

* `Element.innerHTML()` - возвращает/заменяет HTML-разметку внутри элемента. *Уязвимо для XSS*. **Узел перерендерится**
* `Element.outerHTML()` - возвращает/заменяет HTML-разметку элемента и содержимого. *Уязвимо для XSS*. **Узел перерендерится**
>Методы `innerHTML`, `outerHTML` работаают со строкой из которой создаются элементы  

* `Element.replaceWith(param1, ... paramN)` - заменяет элемент новыми элементами или текстовым контентом
* `{document/Element}.replaceChildren(param1, ... paramN)` - заменяет дочерние узлы новыми узлами или текстовым контентом
* `Node.replaceChild(newChild, oldChild)` - заменяет дочерний узел новым узлом
* `Node.removeChild(child)` - удаляет дочерний узел (возвращает удаленный узел)
* `Element.remove()` - удаляет элемент из DOM
* `Node.contains(otherNode)` - проверка содержит ли узел переданный узел `otherNode` (boolean)
* `Node.isEqualNode(otherNode)` - проверка являются ли узлы одинаковыми
* `Node.isSameNode(otherNode)` - проверка являются ли узлы одним и тем же узлом
* `Node.hasChildNodes()` - проверка содержит ли узел дочерние узлы (boolean)
* `{document/Element}.childElementCount` - возвращает количество дочерних элементов

[Вернуться к содержанию](#содержание)

## 1.4 Работа с атрибутами

* `Element.attributes` - список атрибутов (NamedNodeMap - коллекция объектов Attr)`
* `Element.dataset.dataAttributeName` - возвращает / устанавливает дата атрибут `dataAttributeName`  
Удаление data-атрибута: `delete element.dataset.dataAttributeName`  
>data-атрибуты в CamelCase в js - соответствуют kebab-case в html  

* `Element.setAttribute(name, value)` - устанавливает значение атрибута
* `Element.getAttribute(attributeName)` - возвращает значение атрибута или null
* `Element.getAttributeNames()` - возвращает массив имен атрибутов элемента
* `Element.hasAttribute(name)` - проверка наличия конкретного атрибута (boolean)
* `Element.hasAttributes()` - проверка наличия атрибутов у элемента (boolean)
* `Element.toggleAttribute(name, force)` - переключает значение атрибута (force: boolean - добавляет/удаляет атрибут)
* `Element.removeAttribute(attributeName)` - удаляет атрибут
* `document.createAttribute(name)` - создает объект Attr атрибута
* `setAttributeNode(attribute)` - устанавливает узел Attr атрибута
* `getAttributeNode(attributeName)` - возвращает Attr узел аргумента  
* `removeAttributeNode(attributeNode)` - удаляет атрибут по узлу Attr (предварительно возвращается через `getAttributeNode`)
```javascript
const node = document.getElementById('first')
const attr = document.createAttribute("custom_attribute");
attr.value = "customValue";
node.setAttributeNode(attr);
```
* `HTMLElement.hidden` - boolean. Устанавливает/удаляет атрибут скрытия элемента. (Свойство display CSS обладает большим приоритетом)

[Вернуться к содержанию](#содержание)

## 1.5 Работа с классами

* `Element.tagName`, `Element.id`, `Element.className` - возвращает тэг элемента / идентификатор / атрибут `class` (строка со всеми классами элемента через пробел)
* `Element.matches(selectors)` - проверка соответсвия переданному селектору
* `Element.classList` - возвращает коллекцию [DOMTokenList](https://developer.mozilla.org/en-US/docs/Web/API/DOMTokenList) классов элемента
  * `add(class1, ... classN)` - добавление класса **(класс не должен содержать пробелов)**
  * `remove(class1, ... classN)` - удаление класса
  * `contains(class)` - проверка содержит ли DOMTokenList переданный класс (boolean)
  * `replace(oldClass, newClass)` - заменяет класс (boolean, true - если замена произошла) 
  * `toggle(class[, state])` - переключает класс, где state: true/false = add/remove
	
[Вернуться к содержанию](#содержание)

## 1.6 Работа со стилями

* `Element.cssText` - возвращает/устанавливает свойства с полным перезаписыванием атрибута `style`. **Вставляются инлайново**
* `Element.style.backgroundStyle` - возвращает/устанавливает конкретное свойство. **Вставляются инлайново**.  
В следующих методах `propertyName` записывается как в CSS файле
  * `style.getPropertyValue(propertyName)` - возвращает значение CSS свойства `propertyName`
  * `style.setProperty(propertyName[, value[, priority]])` - устанавливает значение `value` CSS свойства `propertyName`. В `priority` можно передать `important`
  * `style.removeProperty(propertyName)` - удаляет CSS свойство `propertyName`. Возвращает значение удаленного свойства
  * `style.getPropertyPriority(propertyName)` - получает приоритет свойства ('important' или '')
* `Element.computedStyleMap()` - возвращает вычисленные CSS свойства в формате [StylePropertyMapReadOnly](https://developer.mozilla.org/en-US/docs/Web/API/StylePropertyMapReadOnly)

[Вернуться к содержанию](#содержание)

## 1.7 Работа с формами

`forms[id], forms.id, forms[index], forms[name], forms.name` - способы доступа к формам внутри HTMLCollection полученной из `document.forms` (атрибут name / атрибут id / индекс в коллекции)  
`forms.formId.elementName` - доступ по имени к полям формы
* `HTMLFormElement.reportValidity()` - метод проверки валидности всех дочерних полей (boolean)  
Значение `false` генерирует событие `invalid` для всех элементов не прошедших проверку
* `HTMLFormElement.submit()` - отправляет форму  
**В отличии** от нажатия на кнопку не вызывает событие `submit`, валидация не проверяется
* `HTMLFormElement.requestSubmit([submitter])` - отправляет форму также как при нажатии на кнопку (submitter - конкретный элемент, запускающий отправку формы)
* `HTMLFormElement.reset()` - очищает форму, устанавливая в полях значения по умолчанию

[Вернуться к содержанию](#содержание)

## 1.8 Скролл

* `Element.scroll(x-coord, y-coord)` - скролл элемента(страницы) до указанных координат  
* `Element.scrollTo(x-coord, y-coord)` - аналог `Element.scroll`
* `Element.scrollBy(x-coord, y-coord)` - скролл элемента(страницы) относительно текущей позиции
>Альтернативно методам можно передать объект с параметрами (top, left, behavior (плавность))  

* `Element.scrollLeft`, `Element.scrollTop` - возвращает/устанавливает значение прокрутки скролла в элементе
* `Element.scrollIntoView(scrollIntoViewOptions)` - скролл до указанного элемента, где опции - объект с полями: behavior (плавность), block, inline (выравнивание)  
Альтернативно принимает boolean значение, где true/false - скролл до верхней/нижней границы элемента

[Вернуться к содержанию](#содержание)

## 1.9 Размеры элемента 

* `Element.clientHeight`, `Element.clientWidth` - height/width content + padding элемента
* `Element.scrollHeight`, `Element.scrollWidth` - высота/ширина контента в элементе, включая содержимое невидимое из-за прокрутки
* `Element.clientLeft`, `Element.clientTop` - ширина border элемента
* `Element.getClientRects()` - возвращает DOMRect с информацией о размерах элемента (width, height, x, y, top, right, bottom, left) по его границам  
inline элементы могут иметь не строго прямоугольные границы, если текст переносится на другие строчки
* `Element.getBoundingClientRect()` - возвращает DOMRect с информацией о размерах элемента по охватывающему его прямоугольнику
* `document.documentElement.clientWidth, document.documentElement.clientHeight` - ширина/высота без полос прокрутки
* `window.innerWidth, window.innerHeight` - ширина/высота окна вместе с полосами прокрутки

[Вернуться к содержанию](#содержание)

## 1.10 Остальное

* `document.title, document.head, document.body` - возвращают части разметки
* `document.forms, document.links, document.images` - возвращает HTMLCollection всех (`<form>`/`<area> и <a>`/`<image>`) в документе
* `document.cookie` - возвращает/устанавливает cookies страницы
* `Element.focus()` - устанавливает фокус на элементе  
Можно передать объект с полем `preventScroll` для отмены прокрутки к элементу  
* `Element.blur()` - сбрасывает фокус с элемента  
* `document.activeElement` - возвращает элемент, находящийся в фокусе
* `document.getSelection()` - возвращает выделенный текст
* `Element.checkVisibility(options)` - проверяет видим ли объект (boolean). В опциях передается по какому из параметров идет проверка (content-visibility, opacity, visibility)
* `Element.animate(keyframes, options)` - создает анимацию [Animation](https://developer.mozilla.org/en-US/docs/Web/API/Animation) элементу через JS
* `{document/Element}.getAnimations()` - возвращает Animation элемента
* `Node.compareDocumentPosition(otherNode)` - сравнивает позиции узлов в DOM. Возвращает [битовую маску](https://developer.mozilla.org/en-US/docs/Web/API/Node/compareDocumentPosition) с параметром соответствующим позиции (до, после, дочерний и т.д.)  
* `Node.normalize()` - удаляет пустые элементы, выравнивает текст по одной строке без переносов
* `Node.isConnected` - проверка присоединен ли элемент к document (boolean)  
Созданные и не добавленные элементы - не присоединены к document
* `Node.cloneNode([true])` - клонирует Node из HTML (true - глубокое клонирование. Весь контент внутри)   
При клонировании копируются все атрибуты узла. В том числе id, который должен быть уникален в пределах DOM  
Клонирование применяется при работе с `<template>`:
```javascript
document.getElementById('templateID').content.firstElementChild.cloneNode(true)
```

[Вернуться к содержанию](#содержание)

## 1.11 События страницы

### 1.11.1 Распространение событий

Распространение событий в JS проходят через 3 фазы:  
1. Фаза захвата (Capture Phase) - идет поиск целевого элемента от корня DOM
2. Фаза целевого элемента (Target Phase) - cобытие достигло целевого элемента, на котором оно было вызвано
3. Фаза всплытия (Bubble Phase) - cобытие начинает всплывать от целевого элемента к корню DOM, вызывая обработчики на каждом промежуточном элементе  
>На фазах захвата и всплытия событие можно перехватывать  

Способы обработки событий:  
1. `element.addEventListener(event, handler[, options])` - устанавливает на HTMLElement действие `handler`, возникающее при событии `event`, где `options` объект с полями:
	* once - (boolean) вызывает одиночное срабатывание события, после вызова слушатель удаляется
	* passive - (boolean) гарантирует, что preventDefault никогда не сработает   
	(для обработки scroll, тач событий, которые по умолчанию ожидают выполнение пользовательского обработчика)  
	* capture - (boolean) перехват события в фазе захвата родительским элементом     
`element.removeEventListener(event, handler[, options])` - отписывает HTMLElement от действия `handler` по событию `event`, подписанного через `addEventListener`  
_Если при подписке на события были переданы опции, то при удалении их также нужно указывать_  
2. `on-свойство` - (`onclick`, `onlock` и т.д.). Передача null в свойство удаляет обработчик   
>Отличие - `on-свойства` позволяют добавить только одну функцию на каждое событие  

[Вернуться к содержанию](#содержание)

### 1.11.2 Виды событий
 
В `handler` через `this` **можно обратиться к элементу, на котором сработало событие**.  
Также в `handler` передается объект Event, обладающий свойствами (различными в зависимости от типа собития):
* `event.composedPath()` - возвращает массив элементов, через которые прошло событие от корня DOM до целевого элемента
* `event.bubbles` - проверка является ли событие всплывающим (boolean)
* `event.stopPropagation()` - блокирует всплытие по DOM
* `event.cancelable` - проверка является ли событие отменяемым для `preventDefault` (boolean)
* `event.preventDefault()` - блокирует стандартное поведение браузера
* `event.defaultPrevented` - проверка была ли блокировка стандартного поведения через `preventDefault()` (boolean)
* `event.target` - возвращает элемент вызвавший срабатывание события (может быть дочерним к тому, на котором установлен обработчик)
* `event.currentTarget` - возвращает объект на котором установлен обработчик события
* `type` - тип события
* `event.eventPhase` - возвращает текущую фазу события (number)
	* 0 - событие не запущено
	* 1 - CAPTURING_PHASE
	* 2 - AT_TARGET
	* 3 - BUBBLING_PHASE
* `event.stopImmediatePropagation()` - блокирует дальнейшее распространение события в текущей фазе  
_Можно блокировать вызов обработчиков установленных на том же элементе, так как они вызваются в порядке установки на элемент_
* `event.isTrusted` - проверка было ли вызвано событие действиями пользователя (true) или из скрипта (false)
* `event.timeStamp` - возввращает время вызова события в мс  

Далее идут наиболее полезные специальные свойства конкретных элементов (форм, кнопок и т.д.)
* `event.submitter` - возвращает элемент кнопки или другой элемент, запустивший отправку формы
* `event.clientX, event.clientY` - координаты мыши
* `event.which` - номер клавиши мыши использованной в данном событии
* и т.д.

**События:**  
* `сopy, cut,  paste` - событие срабатывает при копировании/вырезании/вставке в браузере
* `DOMContentLoaded` - событие срабатывает при полной загрузке HTML, ресурсы могут продолжать загружаться 
* `load` - событие срабатывает при полной загрузке HTML и всех ресурсов
* `beforeunload` - событие срабатывает перед выгрузкой страницы (в нем используют вызов модального окна, подтверждающего выход со страницы)
_В Google Chrome требуется установка в обработчике `event.returnValue = "string"`_
* `visibilitychange` - событие срабатывает при потере веб-страницей видимости
* `readystatechange` - событие срабатывает при изменении `readyState`
* `focus` - событие срабатывает при фокусе
* `focusin` - событие срабатывает при фокусе **(всплывает)**
* `blur` - событие срабатывает при потере фокуса
* `focusout` - событие срабатывает при потере фокуса **(всплывает)**
* `click` событие срабатывает при клике основной кнопки на элемент, нажатие кнопок space или enter на элементе в фокусе, тап на элементе 
Последовательно вызывает несколько событий, например при клике: `mousedown > mouseup > click` 
* `dbclick` - событие срабатывает при двойном клике
* `auxclick` - событие срабатывает при нажатии НЕ основной кнопки мыши
* `contextmenu` событие срабатывает при открытии контекстного меню
* `input` - событие срабатывает при изменении пользователм содержимого `<input>` `<textarea>` и `<select>`  
`<input type="button">, <input type="sumbit">` не вызывают событие
* `change` - событие срабатывает когда `<input>`, `<textarea>` теряет фокус и его значение изменено, при изменении состояния `<input type="checkbox">, <input type="radio">`, `<select>`
* `beforeinput` - событие срабатывает при изменении содержимого `input` или `textarea`
* `selectstart` - событие срабатывает при начале нового выделения (selection)
* `selectionchange` - событие срабатывает при изменении текущего выделения (selection)
* `reset` - событие срабатывает при очистке полей формы
* `submit` - событие срабатывает при отправке валидной формы
* `formData` - событие срабатывает в момент отправки формы (для возможности обработать отправляемые данные) 
* `invalid` - событие срабатывает при отправке формы с невалидными полями (после события submit)
* `scroll` - событие срабатывает при скролле страницы/элемента (мышь, полоса прокрутки, стрелки и т.д.)
* `scrollend` - событие срабатывает при прокрутке страницы/элемента до конца
* `wheel` - событие срабатывает при прокрутки колеса мыши
* `keydown` - событие срабатывает при нажатии кнопки клавиатуры
* `keyup` - событие срабатывает при отжатии кнопки клавиатуры
* `mousedown` - событие срабатывает при нажатии и удерживании кнопки мыши  
Позволяет обойти по приоритету событие `click`
* `mouseup` - событие срабатывает при отжатии кнопки мыши
* `mouseenter` - событие срабатывает при входе указателя в пределы элемента (не всплывает)
* `mousemove` - событие срабатывает при движении указателя в пределах элемента (не всплывает)
* `mouseover` - событие срабатывает при движении указателя над элементом или его дочерними элементами
* `mouseleave` - событие срабатывает при покидании указателем пределов элемента
* `mouseout` - событие срабатывает при покидании указателем пределов элемента или его дочернего элемента **(событие всплывает)**
* `touchstart` - событие срабатывает при первом касании
* `touchmove` - событие срабатывает во время перемещения при касании
* `touchcancel` - событие срабатывает при прерывании касания (другим касанием, модальным окном, ошибкой, покидании элемента)
* `touchend` - событие срабатывает при окончании касания  
>При касании вызывается последовательность событий `touchstart > touchend > mousedown > mouseup > click`  

* `animationstart` - событие срабатывает при начале анимации
* `animationiteration` - событие срабатывает при окончании одной итерации и начале новой
* `animationcancel` - событие срабатывает при внезапном обрыве анимации
* `animationend` - событие срабатывает при окончании анимации
* `transitionrun` - событие срабатывает при начале transition до delay
* `transitionstart` - событие срабатывает при начале transition после delay
* `transitioncancel` - событие срабатывает при отмене transition
* `transitionend` - событие срабатывает при окончании transition
* `storage` - событие срабатывает при изменении значений в localStorage или sessionStorage
* `pagehide` - событие срабатывает когда страница становится невидимой для пользователя (переход на другую страницу, закрытие влкадки и т.д.)
* `pageshow` - событие срабатывает когда страница становится видимой для пользователя (первичная загрузка страницы, переход с другой вкладки и т.д.)

`window.matchMedia(mediaQuery)` - возможность подписываться на CSS `@media-события` в JS
```javascript
const mediaQuery = window.matchMedia('(max-width: 300px)')
mediaQuery.addEventListener('change', callback)
```

[Вернуться к содержанию](#содержание)

### 1.12 FormData

FormData - специальная коллекция для передачи данных в формате ключ-значение через fetch, XMLHttpRequest  
Тот же формат данных использует `<form>` с типом кодирования `multipart/form-data`  
`FormData([form])` - позволяет программно сериализовать данные с формы без ее отправки   
* `append(name, value[, filename]` - добавление значения по ключу `name` с сохранением предыдущих данных
* `set(name, value[, filename])` - добавление значениz, по ключу `name` с перезаписью предыдущих данных
* `get(name)` - возвращает первое значение с ключом `name` или null (элементы формы могут иметь один и тот же атрибут `name`)
* `getAll(name)` - возвращает массива всех значений с ключом `name` 
* `has(name)` - проверка наличия данных по ключу `name` (boolean)
* `entries()` - возвращает итератор с парами ключ/значение
* `keys()` - возвращает итератор с ключами
* `values()` - возвращает итератор со значениями
* `delete(name)` - удаляет значение по ключу `name`

[Вернуться к содержанию](#содержание)

## 2. БОМ

### 2.1 window.history

`window.history` - управление историей браузера в рамках текущей сессии(вкладки)  
* `History.back()` - перемещение на страницу назад (равносильно go(-1))
* `History.forward()` - перемещение на страницу впереди (равносильно go(1))
* `History.go(delta)` - универсальное перемещение по истории (delta - шаги по истории, go(0) - обновляет текущую страницу)
Переходы по истории асинхронны. Для того чтобы узнать, когда переход произошел нужно подписаться на событие `popstate`
* `History.pushState(state, unused[, url])` - добавляет новую запись в историю сессии 
	* state - объект данных, который сохраняется вместе с историей (копируются в `event.state` события `popstate`)  
	Используется например для сохранения фильтра товаров
	* unused - не используется (лучше передавать пустую строку)
	* url - новый URL в истории
* `History.replaceState(state, unused[, url])` - изменяет текущую запись в истории сессии
* `History.length` - количество записей в истории 

[Вернуться к содержанию](#содержание)

### 2.2 window.location
	
`window.location` - информация о текущем адресе страницы  
* `Location.assign(url)` - переход к странице с новым URL
* `Location.replace(url)` - аналогично assign, но текущая страница **НЕ сохраняется** в истории, переход осуществится на предыдущую  
* `Location.reload()` - перезагрузка текущей страницы

`https://example.org:8080/foo/bar?price=low#cars` - Location.href  
`https://example.org:8080` - Location.origin  
`https` - Location.protocol  
`example.org:8080` - Location.host
`example.org` - Location.hostname  
`8080` - Location.port  
`/foo/bar` - Location.pathname  
`?price=low` - Location.search  
`#cars` - Location.hash  

`URLSearchParams` - API (методы аналогичны FormData) позволяющее формировать строку поисковых параметров  
```javascript
const myParams = new URLSearchParams({min: 100, max: 200})`
// важно отметить, что перед SearchParams в URL стоит знак ?
const url = BASE_URL + `?` + myParams;
```

[Вернуться к содержанию](#содержание)

### 2.3 window.navigator 

`window.navigator` - возващает user agent (напрмер браузер) _разобран частично_
* `Navigator.userAgent` - название браузера
* `Navigator.language` - язык в настройках браузера
* `Navigator.languages` - языки в порядке предпочтения
* `Navigator.cookieEnabled` - проверка поддерживает ли браузер куки и включены ли они (boolean)
* `Navigator.onLine` - проверка подключен ли пользователь в сети (ненадежно) (boolean)
* `Navigator.geolocation` - доступ к геолокации
Также много свойств связанных с размером окна браузера, проверка доступности localStorage и sessionStorage, проверка видимости тулбаров, события связанные со страницей (переходы, закрытие/открытие, скролл, изменение размера) и т.д  

[Вернуться к содержанию](#содержание)

## 3. Drag n Drop 

Интерфейс для работы с перетаскиванием объектов.  
Ключевой атрибут `draggable="true"` должен быть установлен на HTML элементе, который можно перетаскивать

**Cобытия Drag n Drop:**  
* `drag` - событие при перетаскивании элемента (1раз/100мс)
* `dragstart` - событие при начале перетаскивания
* `dragend` - событие при окончании перетаскивания
* `dragenter` - событие при заходе в валидную дроп-зону
* `dragleave` - событие при покидании валидной дроп-зоны
* `dragover` - событие при нахождении элемента над валидной дроп-зоной (1раз/100мс)
* `drop` - событие при сбрасывании перетаскиваемого объекта в валидной дроп-зоне
>Для работы drop необходимо создать событие `dragover` и указать в нем `preventDefault()`

[Вернуться к содержанию](#содержание)

### 3.1 DataTransfer

**DataTransfer** - объект для передачи данных при перетаскивании. Передаваемые данные удаляются по окончанию Drag n Drop

* `DataTransfer.setData(format, data)` - устанавливает данные доступные при перетаскивании, где `format` - MIME или собственный тип
	* `text/plain` - простой текст
	* `text/uri-list` - список URI
	* `text/html` - HTML-код
	* `text/csv` - формат CSV
	* `application/x-my-custom-file-type` - файлы собственного типа
	* `application/json` - JSON
	* `image/*` - изображение, например `image/png`
	* `audio/*` - аудио
	* `video/*` - видео
	* `application/pdf` - pdf
	* `application/x-custom-format` - собственный формат
* `DataTransfer.setDragImage(imgElement, xOffset, yOffset)` - устанавливает изображение, которое будет отображаться при перетаскивании. 
`xOffset, yOffset` - позиция курсора относительно изображения
* `DataTransfer.getData(format)` - получение данных по ключу
* `DataTransfer.clearData([format])` - очистка всех данных или указанного по ключу

* `DataTransfer.items` - объект DataTransferItemList содержащий DataTransferItem (элементы добавленные с помощью `setData`)
* `DataTransfer.files` - файлы задействованные при перетаскивании в формате FileList
* `DataTransfer.types` - строковый массив типов данных в DataTransfer
* `DataTransfer.effectAllowed` - свойтсво определяющее какие типы операций разрешены 
Устанавливается в событии `dragstart`. В других событиях не оказывает эффекта
	* none
	* copy
	* move
	* link
	* copyMove
	* copyLink
	* linkMove
	* all
* `DataTransfer.dropEffect` - свойство определяющее доступные типы операций в drop области
Устанавливается в событии `dragover`. Значения аналогичны `effectAllowed`

[Вернуться к содержанию](#содержание)
	
### 3.2 DataTransferItem 

**DataTransferItem** - элемент в DataTransfer  
* `DataTransferItem.type` - MIME-тип элемента  
* `DataTransferItem.kind` - тип элемента (string или file)  
* `DataTransferItem.getAsString(callbackFn)` - получение данных строкового типа  
* `DataTransferItem.getAsFile()` - получение данных типа File  

[Вернуться к содержанию](#содержание)

### 3.3 DataTransferItemList

**DataTransferItemList** - список объектов DataTransferItem  
* `DataTransferItemList.length` - длина
* `DataTransferItemList.add(data, type), DataTransferItemList.add(file)` - добавление элемента
* `DataTransferItemList.remove(index)` - удаление элемента
* `DataTransferItemList.clear()` - очистка списка

[Вернуться к содержанию](#содержание)

## 4. RegExp

[Хороший разбор Regexp](https://www.youtube.com/watch?v=2CW1wVtnzi4)  
[Проверка regexp](https://regex101.com/)  

RegExp - ругулярное выражение, представляющее собой шаблон для поиска и замены текста, проверки соответствий строки шаблону  
Создание:
1. `new RegExp('pattern', 'flags')` - можно использовать шаблонные строки с переменными  
2. `/pattern/flags`
`pattern` - регулярное выражение  
`flags` - нуобязательные флаги для регулярного выражения   
Для экранирования специальных символов `[]^$(){}.|?+*\` в первом варианте используется используются `\\`, во втором `\`

[Вернуться к содержанию](#содержание)

### 4.1 Флаги

* `g` - множественные совпадения
* `i` - совпадение без учета регистра
* `m` - мультистрочный поиск 
* `s` - меняет интерпритацию символа точки. Точка будет соответствовать любому символу, включая новую строку
* `y` - поиск на позиции lastIndex и не дальше
* `d` - вывод совпадения в массиве с дополнительной информацией (индекс начала и конца совпадения и т.д.)

[Вернуться к содержанию](#содержание)

### 4.2 Методы

* `String.match(regexp)` - возвращает массив с одним совпадением и всеми capture group (скобочными группами), index`ом(позиция совпадения), input`ом(строка поиска) или null  
Если установлен флаг `g`, то возвращает массив всех совпадений
* `String.matchAll(regexp)` - работает только с флагом `g`. Возвращает RegExp String Iterator (массивоподобную сущность) с совпадением и всеми capture group или null  
Для `/(A-Z)(0-9)/g` в строке `'abCDA1234'` вернет `['A1', 'A', '1']`  
* `regexp.test(string)` - (boolean) проверка совпадения строки регулярному выражения.
При установке флага `g` запишет в свойство `RegExp.prototype.lastIndex` номер символа строки, где совпало значение.  
Повторный запуск начнет поиск совпадения с `RegExp.prototype.lastIndex`. Если повторный поиск не дал совпадений lastIndex сбрасывается в 0

[Вернуться к содержанию](#содержание)

### 4.3 Спецсимволы

* `\w` - цифра, латинская буква или _. Эквивалент: `/[A-Za-z0-9_]/`
* `\W` - обратное поиску `\w`. Эквивалент: `/[^A-Za-z0-9_]/`
* `\d` - цифра. Эквивалент: `/[0-9]/`
* `\D` - обратное поиску \d (не цифра). Эквивалент: `/[^0-9]/`
* `\s` - пробел, табуляция, перенос строки. Эквивалент: `[\f\n\r\t\v\u0020\u00a0\u1680\u2000-\u200a\u2028\u2029\u202f\u205f\u3000\ufeff]`
* `\S` - обратное поиску \s. Эквивалент: `[^\f\n\r\t\v\u0020\u00a0\u1680\u2000-\u200a\u2028\u2029\u202f\u205f\u3000\ufeff]`
* `\b` - граница слова (начало, конец строки или любой символ кроме цифры, **латинской** буквы или _)
* `\t` - горизонтальная табуляция
* `\n` - новая строка
* `\v` - вертикальная табуляция
* `\r` - возврат каретки (carriage return)

* `.` - любой символ кроме конца строки (для абсолютно любого символа нужно использовать `/\s\S/`
* `|` - ИЛИ. Пример: (a|b|c)
* `[]` - поиск любого символ из набора/диапазона. Набор: [ABCDEF]. Диапазон: [A-F]. 
* `^` - НЕ - знак исключения, если записан внутри скобок [], (), {}

[Вернуться к содержанию](#содержание)

### 4.4 Якоря

* `^` - начало строки `/^[0-9]/`
* `$` - конец строки `/[0-9]$/`

* `CAT(?=DOG)` - поиск CAT, когда за ним следует DOG в начало, не включая DOG: `CATDOG // CAT`
* `CAT(?!DOG)` - поиск CAT, когда за ним не следует DOG: `CATZ // CAT`
* `(?<=DOG)CAT` - поиск CAT, когда перед ним стоит DOG: `DOGCAT // CAT`
* `(?<!DOG)CAT` - поиск CAT, когда перед ним не стоит DOG: `ZCAT // CAT`

[Вернуться к содержанию](#содержание)

### 4.5 Квантификаторы

* `+` - 1 и более повторений
* `*` - 0 и более повторений
* `?` - 0 или 1 повторение
* `{}` - указывается количество повторений  
`{2}` - 2 повторения. `{2-4}` - от 2 до 4 повторений. `{2,}` -  от 2 и более повторений

[Вернуться к содержанию](#содержание)

### 4.6 Скобочные группы

* `()` - группа захвата. `(cat)` - группа захвата cat  
* `(?<name>pattern)` - именованная группа захвата, где `name` - имя группы, `pattern` - шаблон захвата  
```javascript
const regex = /(?<firstName>\w+) (?<lastName>\w+)/;
const text = 'Petr Ivanov';
const match = text.match(regex);
console.log(match.groups.firstName); // Petr
console.log(match.groups.lastName);  // Ivanov 
```
* `(?:)` НЕ группа захвата 

**Дополнение:**  
```javascript
const str = 'Some "text" in quotes "here"'
const regexp1 = /".+"/g
const matches1 = text.match(regexp1);
console.log(matches1); // ["\"text\" in quotes \"here\""]
// Для исправления и поиска текста в кавычках: 
const regexp2 = /".+?"/g
const matches2 = text.match(regex);
console.log(matches); // ["\"text\"", "\"here\""]
```

[Вернуться к содержанию](#содержание)

## 5. Proxy, Reflect

### 5.1 Proxy  

**Proxy** - специальный объект-обертка над целевым объектом, создающий "ловушки" (traps) для перехвата операций над целевым объектом:   
`let proxy = new Proxy(target, handler)`, где  
* `trap` - метод, перехватывающий операции над целевым объектом
* `target` - целевой объект  
* `handler` - конфигурация с ловушками. Пустой handler перенаправляет все операции на целевой объект

Количество операций, которые можно перехватить ловушками ограничены:  

|Операция|Ловушка|Вызов|
|----|----|----|
|getPrototypeOf|getPrototypeOf|образение к прототипу объекта `Object.getPrototypeOf()`|
|setPrototypeOf|setPrototypeOf|установка прототипа объекта `Object.setPrototypeOf()`|
|isExtensible|isExtensible|при проверке объекта на расширяемость `Object.isExtensible()`|
|preventExtensions|preventExtensions|при попытке сделать объект нерасширяемым `Object.preventExtensions()`|
|getOwnProperty|getOwnPropertyDescriptor|при получении дескриптора свойства `Object.getOwnPropertyDescriptor()`|
|defineProperty|defineOwnProperty|при определении/изменении свойства `Object.defineProperty()`|
|hasProperty|has|проверка существования свойства `in`|
|get|get|чтение свойства|
|set|set|запись свойства|
|delete|deleteProperty|удаление свойства `delete`|
|ownPropertyKeys|ownKeys|при перечислении свойств объекта `keys, values, entries, for...in, Object.getOwnPropertyNames, Object.getOwnPropertySymbols`|
|call|apply|при вызове функции|
|construct|construct|при использовании `new`|

ECMAScript накладывает условия на реализацию многих типов ловушек: [подробнее](https://tc39.es/ecma262/#sec-proxy-object-internal-methods-and-internal-slots)  
**Примеры:**  
* **Ловушка `get(target, property, value, receiver)`** должна возвращать значение свойства, к которому было обращение, если оно является собственным (не унаследованным), неизменяемым (`writable: false`) и неконфигурируемым (`configurable: false`) за исключением случая, когда свойства не существует  
Значение для неконфигурируемого (`configurable: false`) собственного (не унаследованного) свойства, у которого поле `get` равно `undefined` должно возвращать `undefined`  
```javascript
const target = {}
Object.defineProperties(target, {
	immutableProp: {
		value: 'Petr',
		writable: false, // неизменяемое
		configurable: false // неконфигурируемое
	},
	accessorProp: {
		get: undefined, // атрибут [[Get]] равен undefined
		set: (value) => {
			this._accessorProp = value;
		},
		configurable: false // неконфигурируемое
	}
})
const proxy = new Proxy(target, {
	get(target, property) {
		if (property in target) return target[property];
		else return 'defaultValue'
	}
})
console.log(proxy.immutableProp); // Petr
console.log(proxy.accessorProp); // undefined
console.log(proxy.unknownProp); // defaultValue
```  
* **Ловушка `set(target, property, value, receiver)`** должна возвращать boolean.  
Нельзя изменить значение так, чтобы оно отличалось от соответствующего свойства в оригинальном объекте, для собственного (не унаследованного), неизменяемого (`writable: false`), неконфигурируемого (`configurable: false`) свойства    
Изменение значения должно быть заблокировано для собственного (не унаследованного), неконфигурируемого (`configurable: false`) свойства, у которого поле `set` равно undefined попытка  

```javascript
const target = {}
Object.defineProperties(target, {
    immutableProp: {
        value: 'Petr',
        writable: false, // неизменяемое
        configurable: false // неконфигурируемое
    },
    accessorProp: {
        get: () => 'Getting value for accessorProp', 
        set: (value) => undefined, // атрибут [[Set]] равен undefined
        configurable: false // неконфигурируемое
    }
});
const proxy = new Proxy(target, {
	set(target, property, value) {
		const descriptor = Object.getOwnPropertyDescriptor(target, property);
		if (descriptor && !descriptor.configurable && (!descriptor.writable || descriptor.set === undefined)) {
				if (descriptor.value !== value) {
						return false;
				}
		}
		target[property] = value;
		return true;
	}
})
console.log(proxy.immutableProp); // Petr
proxy.immutableDataProp = 21; // попытка изменения значения неизменяемого и неконфигурируемого свойства
console.log(proxy.immutableProp); // Petr
console.log(proxy.accessorProp); // Getting value for accessorProp
proxy.accessorProp = 21; // попытка изменения значения неконфигурируемого свойства с set равным undefined
console.log(proxy.accessorProp); // Getting value for accessorProp
proxy.customProp = 21;
console.log(proxy.customProp); // 21
``` 
* **Ловушка `has(target, property)`** - должна возвращать boolean.  
Свойство не может быть указано несуществующим, если оно существует как неконфигурируемое (`configurable: false`) собственное (не унаследованное) свойство целевого объекта  
Свойство не может быть указано несуществующим, если оно существует как собственное (не унаследованное) свойство целевого объекта, а сам объект является нерасширяемым (`Object.isExtensible() === false`)
```javascript
const target = {};
Object.defineProperties(target, {
    configurableProp: {
        value: 1,
        configurable: false
    },
    extensibleProp: {
        value: 2,
        configurable: true
    }
});
Object.preventExtensions(target);
const proxy = new Proxy(target, {
	has(target, property) {
		const descriptor = Object.getOwnPropertyDescriptor(target, property);
		if (descriptor && !descriptor.configurable) return true
		return descriptor && !Object.isExtensible(target) ? true : false
	}
})
console.log('configurableProp' in proxy); // true, потому что свойство существует и неконфигурируемое
console.log('extensibleProp' in proxy); // true, потому что свойство существует
console.log('nonExistentProp' in proxy); // false, такое свойство не существует
```
* **Ловушка `delete(target, property)`** - должна возвращать boolean.  
Нельзя удалить собственное (не унаследованне), неконфигурируеме (`configurable: false`) свойство  
Нельзя удлить собственное (не унаследованне) свойство в нерасширяемом целевом объекте (`Object.isExtensible() === false`)  
* **Ловушка `ownKeys(target)`** - должна возврашать список ключей (`string` или `symbol`) всех собственных (не унаследованных) свойств целевого объекта   
В списке должны содержаться ключи всех неконфигурируемых (`configurable: false`) собственных (не унаследованных) свойств  
Если целевой объект не расширяем (`Object.isExtensible() === false`), то список должен содержать только ключи собственных свойств целевого объекта 
Ключи в списке не должны повторяться.   
* **Ловушка `getOwnPropertyDescriptor(target, property)`** должна возвращать объект или undefined.  
Свойство не может быть признано несуществующим, если оно существует в целевом объекте как неконфигурируемое (`configurable: false`) собственное (не унаследованное) свойство или собственное свойство.  
Свойство не может быть признано несуществующим, если оно существует как собственное свойство в нерасширяемом объекте.  
A property cannot be reported as non-configurable, unless it exists as a non-configurable own property of the target object.
A property cannot be reported as both non-configurable and non-writable, unless it exists as a non-configurable, non-writable own property of the target object.
* **Ловушка `defineProperty(target, property, descriptor)`** должна возвращать boolean.  
Свойство не может быть добавлено к нерасширяемому объекту (`Object.isExtensible() === false`).  
Неконфигуреруемое  (`configurable: false`) свойство не может стать неизменияемым (`writtable: false`), если для него существует соответствующее собственное (не унаследованное) неконфигурируемое свойства целевого объекта.  
Применение дескриптора к свойству не должно вызывать ошибку, если существует соответствующее собственное (не унаследованное) свойство целевого объекта. 
A property cannot be non-configurable, unless there exists a corresponding non-configurable own property of the target object
* **Ловушка `apply(target, thisArg, argumentsList)`** применима только к целевому объекту, который является вызываемым (имеет метод `call()`)
* **Ловушка `construct(target, argumentsList, newTarget)`** должна возвращать объект.  
Применима только к целевому объекту, у которого есть внутренний метод `construct` (т.е. вызов через `new`)  
* **Ловушка `isExtensible(target)`** должна возвращать boolean, значение которого должно быть таки же, что и примененый `Object.isExtensible()` к целевому объекту
* **Ловушка `preventExtensions(target)`** должна возвращать boolean. True - только в том случае, когда объект не расширяем (`Object.isExtensible() === false`)
* **Ловушка `getPrototypeOf(target)`** должна возвращать объект или null.  
Когда целевой объект не расширяем (`Object.isExtensible() === false`), ловушка возвращает то же значение, что и примененный `Object.getPrototypeOf()` к целевому объекту
* **Ловушка `setPrototypeOf(target, prototype)`** - должна возвращать boolean.  
Когда целевой объект не расширяем (`Object.isExtensible() === false`), то значение `prototype` то же, что и возвращаемое значение метода `Object.getPrototypeOf()` примененного к целевому объекту
>Все условия описаны [здесь](https://tc39.es/ecma262/#sec-proxy-object-internal-methods-and-internal-slots)  

**Отключаемые прокси** - прокси, который может быть отключен от оригинального объекта  
```javascript
const target = { name: 'Petr' }
const {proxy, revoke} = Proxy.revocable(target, handler) 
// отключение прокси
revoke();
proxy.name // ошибка, прокси отключен
```

[Вернуться к содержанию](#содержание)

### 5.2 Reflect   

**Reflect** - встроенный объект, упрощающий работу с Proxy   
Для каждой ловушки в `Proxy` существует метод в `Reflect`, имеющий те же аргументы   

**Пример 1 (сокращение кода - уменьшает вероятность создания ошибок в сложных ловушках)**  
```javascript
let target = {
	name: 'Petr'
};

const proxy = new Proxy(target, {
	get(target, prop, receiver) {
		console.log('Get trap');
		return Reflect.get(target, prop, receiver);
		// Без использования Reflect
		// return target[prop];
	},
	set(target, prop, val, receiver) {
		console.log('Set trap');
		return Reflect.set(target, prop, val, receiver);
		// Без использования Reflect
		// target[prop] = value;
		// return true;
	}
});
console.log(proxy.name)
proxy.name = 'Olga'
```

**Пример 2 (сохранение контекста)**  
Проблема: В объекте `person1` отсутствует поле `name`.  
При обращении к нему свойство ищется в прототипе (`proxyWithoutReflex`), где срабатывает ловушка и возвращается значение из объекта `target`  
Решение проблемы сохранения контекста: `return prop in receiver ? receiver[prop] : target[prop]`, что аналогично применению `Reflect` в `proxyWithReflex`
```javascript
let target = {
	_name: 'Petr',
	get name() {
    return this._name;
  }
};
let proxyWithoutReflex = new Proxy(target, {
  get(target, prop, receiver) {
    return target[prop];
  }
});
let proxyWithReflex = new Proxy(target, {
  get(target, prop, receiver) {
    return Reflect.get(target, prop, receiver);
  }
});

let person1 = {
  __proto__: proxyWithoutReflex,
  _name: "Olga"
};
let person2 = {
  __proto__: proxyWithReflex,
  _name: "Olga"
};
console.log(person1.name) // Petr
console.log(person2.name) // Olga
```  

>Встроенные объекты (например Map, Set, Date, Promise и т.д., **кроме Array**) используют внутренние слоты, которые не может перехватить Proxy  
Аналогичное поведение приватных свойства классов  
>```javascript
>const target = new Map()
>const proxyMap = new Proxy(target, {})
>proxy.set('name', 'Petr'); // ошибка
>```
>Решение проблемы - привязка свойств-функций к контексту оригинального объекта: 
>```javascript
>const target = new Map()
>const proxyMap = new Proxy(target, {
>	get(target, prop, receiver) {
> let value = Reflect.get(target, prop, receiver);
> return typeof value == 'function' ? value.bind(target) : value;
>	}
>})
>proxy.set('name', 'Petr'); // ошибка
>```

[Еще примеры на видео, если не хватило](https://www.youtube.com/watch?v=NQIsue1YeO8)  
[Почитать дополнительно](https://learn.javascript.ru/proxy#reflect)  

[Вернуться к содержанию](#содержание)

## 6. Классы

[Вкратце про классы на видео. Не слишком подробно](https://www.youtube.com/watch?v=gWl8M9Z4fic)  
Класс - шаблон для создания однотипных объектов.  
Внутри классов можно создавать собственные (не доступные в прототипе) свойства, вычисляемые свойства, геттеры/сеттеры, методы.   
Методы в классе являются неперечислимыми (`enumerable: false`):
  
```javascript
class Person {
	// собственное свойство
	role = 'Admin';
	constructor(nameKey, nameValue, age) {
		// вычисляемое свойство
		this[nameKey] = nameValue
		this.age = age
	}
	 get city() {
    return this._city;
  }
	set city(value) {
		this._city = value;
	}
	calculateAge(){
		const currentYear = new Date().getFullYear();
		return currentYear - this.age
	}
}
const nameKey = 'firstName'

const person = new Person(nameKey, 'Petr', 2000)
person.city = 'Moscow'
console.log(person.role);
console.log(person.firstName)
console.log(person.age)
console.log(person.city)
console.log(person.calculateAge())
```

**Class Expression** - классам доступно определение внутри другого выражения, присваивание, возвращение из функции (аналогично Function Expression):  
```javascript
const person = class { method(){} }
const namedPerson = class Person { method(){} }
function makeClass() {
	return class { method() {} }
}
new person().method()
new namedPerson().method()
const functionPerson = makeClass()
new functionPerson().method()
```

`instanseof` - оператор проверяет является ли объект экземпляром конструктора или класса (просматривая цепочку прототипов)  
```javascript
const arr = [];
console.log(arr instanceof Array) // true
console.log(arr instanceof Object) // true т.к. Array наследуется от Object
```

[Вернуться к содержанию](#содержание)

### 6.1 Наследование классов   

Наследование - функционал расширения одного класса другим, когда расширяемый класс получает доступ к методам и свойствам наследуемого класса  
Конструктор в наследующем классе **должен обязательно** вызывать `super()` перед использованием `this`, т.к. конструктор наследующего класса не создает пустого объекта с привязкой `this`  
_Методы наследуемого класса могут быть переопределены_  
Собственные свойства не переопределяются при использовании родительского конструктора
```javascript
class Human {
	planet = 'Earth';
	constructor() {
		this.race = 'human';
	}
	getInfo() {
		console.log('Your race is ' + this.race)
	}
}
class Warrior extends Human {
	planet = 'Mars'
	constructor() {
		super();
		this.strength = 50;
	}
	// переопределение наследуемого метода
	getInfo() {
		// базовая реализация метода наследуемого класса
		super.getInfo()
		// дополнительный функционал текущего класса
		console.log('Your strength is ' + this.strength)
	}
}
const warrior  = new Warrior()
warrior.getInfo()
// собственное свойство не переопределено
console.log(warrior.planet) // Earth
```
>Также стоит отметить, что стрелочные функции не имеют собственного контекста и не могут использовать `super`  
>`getInfo = () => { supet.getInfo() }` приведет к ошибке

**Пример наследования от Class Expression**  
```javascript
function myFunction(name) {
	return class {
		getName() { console.log(name) }
	}
}
class Person extends myFunction('Petr') {}
new Person.getName()
```

**Примеси** - методы, реализующие определенное поведение, функционал которыех можно добавить к другим классам 
```javascript
const mixin = {
	getInfo() {
		console.log('Info')
	}
}
class Person {
	constructor(name) {
		this.name = name
	} 
}
Object.assign(Person.prototype, mixin)
const person = new Person('Petr')
person.getInfo()
```

[Вернуться к содержанию](#содержание)

### 6.2 Виды методов и свойств

**Статические методы и свойства** - методы и свойства принадлежащие классу, но не инстансу  
Статические методы и свойства наследуются.  
```javascript
class Person {
	static role = 'user';
	constructor(name) {
		this.name = name
	}
	static getSecureInfo() {
		console.log('Secure info')
	}
	// статический метод для создания инстанса класса
	static createPerson(name) {
		return new this(name)
	}
}
const person = new Person();
try {
  person.getSecureInfo() // ошибка. отсутствующий метод
}
catch(error) {
  console.log('catch static method error')
}
Person.getSecureInfo() // Secure info
const user = Person.createPerson('Petr');
console.log(user.name)
```

Поля (свойства и методы) класса могут быть **приватными и публичными**. Защищенных полей в JS нет, они эмулируются.  
Все предыдущие примеры рассматривали работу публичных полей.  
Защищенные поля (начинаются с префикса `_`) доступны внутри класса и наследуемых классов (это лишь соглашение).  
Приватные поля (начинаются с префикса `#`) **доступны только** внутри класса.  
```javascript
class Person {
	_city = 'Moscow'  
	#role = 'user'
}
const person = new Person()
console.log(person._city) // Moscow. Доступ есть, но по соглашению, прямое обращение к защищенным полям вне классов не используется
console.log(person.#role) // ошибка обращения к приватному свойству
```

[Вернуться к содержанию](#содержание)

## 7. Дополнительно

### 7.1 Скачивание файла

Способ скачивания файла в браузере:
1. `<a download="filename.png"></a>`  
2. XMLHttpRequest 
3. FetchAPI
4. Экспериментальная технология: [snowSaveFilePicker](https://developer.mozilla.org/en-US/docs/Web/API/Window/showSaveFilePicker#examples  )

Для реализации скачивания файла необходимо:  
1. Программно создать элемент `<a>`  
2. Назначить атрибуты `href, download`
3. Добавить созданный элемент в DOM 
4. Произвести программный клик 
5. Удалить элемент из DOM
```javascript
  const link = document.createElement('a');
  link.href = url;
  link.download = filename;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
```

[Вернуться к содержанию](#содержание)