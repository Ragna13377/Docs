# Содержание

- [1. Установка и TSconfig](#1-установка-и-tsconfig)
- [2. Базовые типы](#2-базовые-типы)
  - [2. unknown, null, undefined, void, never](#21-unknown-null-undefined-void-never)
- [3. Функции](#3-функции)
  - [3.1 Параметры функции](#31-параметры-функции)
- [4. Массивы и кортежи](#4-массивы-и-кортежи)
- [5. Объекты](#5-объекты)
- [6. Собственные типы, интерфейсы](#6-собственные-типы-интерфейсы)
- [7 Enum](#7-enum)
- [8. DOM и встроенные объекты](#8-dom-и-встроенные-объекты)
- [9. Promise](#9-promise)
- [10. Utility Type](#10-utility-type)
- [11. keyof, infer, in и type guard](#11-keyof-infer-in-и-type-guard)
- [12. Рекурсия](#12-рекурсия)
- [13. Классы](#13-классы)
- [14. Дженерики](#14-дженерики)
- [15. Файлы .d.ts](#15-файлы-dts)
- [16. Type Challenges](#16-type-challenges)
- [17. Полезные ссылки](#17-полезные-ссылки)

## 1. Установка и TSconfig

**Установка Typescript:**

1. `npm install --save-dev typescript` - установка компилятора TypeScript в проект
2. `npm install --save-dev ts-loader` - установка загрузчика для Webpack-сборки
3. `npx tsc --init` - создание конфига `tsconfig.json`
4. `npx tsc` - запуск компиляции проекта
5. `npx tsc --noEmit` - проверка типов без создания JS-файлов

**Преднастройка TSconfig**

`tsconfig.json` - главный конфиг TypeScript-проекта

```json
{
	"compilerOptions": {
		"target": "ES2020",
		"module": "ESNext",
		"strict": true,
		"rootDir": "src",
		"outDir": "dist",
		"sourceMap": true
	},
	"include": ["src/**/*"],
	"exclude": ["node_modules"]
}
```

`"extends": "./<path>.json"` - подключает настройки из другого `tsconfig`

`"include": ["src/**/*"]` - файлы, которые входят в TS-проект

`"exclude": ["node_modules"]` - файлы, которые исключаются из TS-проекта

`"files": ["src/index.ts"]` - точный список файлов для компиляции. Обычно вместо него удобнее использовать `include`

Опции *compilerOptions*:

* `target` - версия JS, в которую компилируется код
* `module` - формат модулей после компиляции (например `ESNext`, `CommonJS`, `NodeNext`)
* `strict` - включает строгую проверку типов
* `rootDir` - папка с исходными файлами
* `outDir` - папка для скомпилированных файлов
* `allowJs` - разрешает добавлять `.js` файлы в TS-проект
* `esModuleInterop` - упрощает импорт CommonJS-модулей
* `declaration` - создает `.d.ts` файлы с описанием типов
* `sourceMap` - создает source map для дебага
* `incremental` - ускоряет повторную сборку за счет сохранения данных последней компиляции в `.tsbuildinfo`
* `noEmit` - проверяет типы без создания выходных файлов
* `noImplicitAny` - запрещает неявный `any`
* `strictNullChecks` - заставляет явно учитывать `null` и `undefined`

[Вернуться к содержанию](#содержание)

## 2. Базовые типы

**Примитивные типы** - базовые типы данных:

```typescript
let stringVar: string = 'text'
let numberVar: number = 1
let booleanVar: boolean = true
let bigintVar: bigint = 1n
let symbolVar: symbol = Symbol('id')
```

* `string`, `number`, `boolean`, `bigint`, `symbol`, `null` и `undefined` соответствуют базовым JS-значениям.

Переменная может принимать несколько типов. Они перечисляются через `|`:

```typescript
let value: string | number | boolean
```

**Литеральный тип** - тип, который принимает только конкретное значение:

```typescript
let status: 'success' | 'error' = 'success'
let code: 200 | 404 | 500 = 200
```

* `any` - может хранить любое значение. Используется, когда тип данных неизвестен, но фактически отключает проверку TypeScript:

```typescript
let value: any = 1
value = 'text'
value.someMethod()
```

>Использование `any` - плохой паттерн и лучше использовать как временную меру. Если тип неизвестен, но проверку типов хочется сохранить, обычно лучше взять `unknown`.

[Вернуться к содержанию](#содержание)

### 2.1 unknown, null, undefined, void, never

* `unknown` - используется, когда тип приходящих данных неизвестен. В отличие от `any`, он требует сначала проверить тип, а уже потом использовать значение:

```typescript
let func = (input: unknown): string => {
    // type guard, проверка типа
	if (typeof input === 'number') {
		return input.toString()
	}

	return String(input)
}
```

* `null` - отсутствие значения.

* `undefined` - значение еще не определено.

При `strictNullChecks` TypeScript просит проверять места, где значение может быть `null` или `undefined`:

```typescript
function getLength(value: string | undefined): number {
	if (value === undefined) return 0
	return value.length
}
```

* `void` - используется, когда функция ничего не возвращает или возвращаемое значение не важно:

```typescript
function log(message: string): void {
	console.log(message)
}
```

* `never` - означает, что значения быть не может. Например: бесконечный цикл, функция всегда выбрасывает ошибку, либо все варианты union уже обработаны.

```typescript
const neverFunc = (): never => {
	throw new Error('Error text')
}

type Status = 'success' | 'error'

function handleStatus(status: Status): string {
	if (status === 'success') return 'OK'
	if (status === 'error') return 'FAIL'

	const neverValue: never = status
	return neverValue
}
```

![Наследование типов](https://pictures.s3.yandex.net/resources/Untitled_2_1701698986.png)

Если тип полученных данных не совпадает с ожидаемым, можно принудительно указать его через **as**  
>Частое использование type assertion нарушает принципы TS, является антипаттерном и используется там, где действительно необходимо

```typescript
const anyVar: any = 1
const numberVar: number = anyVar as number
```

`as` не меняет значение в runtime, а только говорит TypeScript, каким типом считать значение.

[Вернуться к содержанию](#содержание)

## 3. Функции

* function declaration
```typescript
function func(arg: number): number {
	return arg
}
```
* function expression и стрелочная функции:
```typescript
const anonymousFunc = function(arg: number): number {
	return arg
}

const arrowFunc = (arg: number): number => {
  return arg
}
```

* переменная, хранящая только функции:

```typescript
const arrowFunc: (arg: number) => number = (arg) => {
	return arg
}

const anonymousFunc: (arg: number) => number = function(arg) {
	return arg
}
```

[Вернуться к содержанию](#содержание)

### 3.1 Параметры функции

Опциональный параметр указывается через `?`:

```typescript
function greet(name?: string): string {
	return name ? `Hello, ${name}` : 'Hello'
}
```

Значение по умолчанию:

```typescript
function greet(name = 'Guest'): string {
	return `Hello, ${name}`
}
```

Rest-параметры:

```typescript
function sum(...numbers: number[]): number {
	return numbers.reduce((acc, number) => acc + number, 0)
}
```

[Вернуться к содержанию](#содержание)

## 4. Массивы и кортежи

* Однородный массив:

```typescript
const stringArray: string[] = ['a']
const numberArray: Array<number> = [1, 2]
```

* Неоднородный массив:

```typescript
const array: (string | number)[] = ['a', 2]
```

* Кортеж(typle) - массив заданной длины и заданных типов:

```typescript
const tuple: [string, number] = ['a', 1]
```

[Вернуться к содержанию](#содержание)

## 5. Объекты

Типизация объекта при объявлении (количество обязательных полей должно совпадать с типом):

```typescript
const obj: {
	requiredField: string
	optionalField?: string
} = {
	requiredField: 'a',
}
```

Опциональное поле по смыслу имеет тип `тип | undefined`, перед использованием требуется проверка:

```typescript
if (obj.optionalField) {
	obj.optionalField.toUpperCase()
}
```

При неизвестных ключах объекта используется index signature:

```typescript
const dictionary: { [key: string]: string } = {
	name: 'Petr',
	city: 'Moscow',
}
```

[Вернуться к содержанию](#содержание)

## 6. Собственные типы, интерфейсы

**Собственные типы** позволяют дать имя уже существующему типу или собрать новый тип из других типов.  

```typescript
type Id = string

type User = {
	id: Id
	name: string
}
```

Пересечение типов через `&` соединяет несколько объектных типов в один:

```typescript
type Named = { name: string }
type WithAge = { age: number }

type Person = Named & WithAge
```

>Не применимо к примитивным типам: `type Result = string & number // never`

**Интерфейс** чаще используют для описания формы объекта или класса:

```typescript
interface User {
	id: number
	name: string
	age?: number
}
```

Интерфейс можно расширять через `extends`:

```typescript
interface Employee extends User {
	role: string
}
```

Интерфейс можно использовать в классе через `implements`:

```typescript
interface Logger {
	log(message: string): void
}

class ConsoleLogger implements Logger {
	log(message: string): void {
		console.log(message)
	}
}
```

**Отличия `type` от `interface`:**

* `type` поддерживает пересечение (`&`) и объединение (`|`)  
Интерфейсы и типы можно пересекать и объединять. При этом интерфейс трансформируется в тип.  
* `type` нельзя объявить повторно с тем же именем
* `interface` можно расширять через `extends`
* `interface` можно имплементировать классами `implements`  
Тип можно имплементировать, но он трансформируется в интерфейс  
>При этом имплементировать тип являющийся объединением нельзя

* `interface` можно объявить несколько раз, и объявления объединятся
```typescript
// WindowState будет содержать оба поля
interface WindowState {
	title: string
}

interface WindowState {
	isOpen: boolean
}
```

## 7. Enum

**Enum** - перечисляемый тип. Создает тип и JS-объект в runtime:

```typescript
enum Status {
	Disabled = 0,
	Enabled = 1,
}

enum Race {
  Ork = 'Ork',
  Human = 'Human',
}

const status = Status.Enabled
const race = Race.Ork
```

К enum можно обращаться через точечную нотацию:

```typescript
type Automat = {
	automatStatus: Status
}

const machine: Automat = {
	automatStatus: Status.Enabled,
}
```

**Отличие `enum` от `const enum`**

Обычный `enum` остается в JS после компиляции.

```typesctipt
enum Race {
	Ork: 'Ork',
	Human: 'Human'
};
let race = Race.Ork;
```
Компилируется в:  
```javascript
var Race;
(function (Race) {
    Race[Race["Ork"] = 0] = "Ork";
    Race[Race["Ork"] = 1] = "Ork";
    Race[Race["Human"] = 2] = "Human";
    Race[Race["Human"] = 3] = "Human";
})(Race || (Race = {}));
let race = Race.Ork;
```

`const enum` **теряет** часть функциональности при компиляции:
```typescript
const enum Race {
	Ork: 'Ork',
	Human: 'Human'
};
let race = Race.Ork
```
Не компилируется в JS, а заменяется фактическим значением:
```typescript
let race = 0 /* Race.Ork */;
```

Из-за этого у `const enum` есть ограничения:

1. Нельзя использовать его как обычный runtime-объект: `for (let key in Race) {}` или `const enumKey = Race[race]`
2. Нельзя использовать вычисляемые значения:  `const enum Size { small: config.smallSize }`

Для сохранения `const enum` при компиляции, используется флаг `preserveConstEnums` в конфиге.

[Небольшая статья](https://phuoc.ng/collection/this-vs-that/const-enum-vs-enum/)
[When can the TypeScript typechecker not inline via const enum?](https://www.susanpotter.net/software/typescript-enum-versus-const-enum/)

[Вернуться к содержанию](#содержание)

## 8. DOM и встроенные объекты

**DOM-элементы**
  При поиске любого DOM-элемента он может быть не найден:
`const element: HTMLElement | null = document.querySelector('.my-class')` // лучше проверять конкретный тип HTMLButtonElement и т.д.
`if(element) {}` - обязательная проверка, что элемент найден

Типизация событий:
`function handleClick(event: Event){}` // лучше проверять конкретный тип MouseEvent
`myButton.addEventListener('click', handleClick)`

Объект **Map** - коллекция пар ключ-значение
`const mapObject = new Map<string, number>()`

Объект **Set** - коллекция уникальных значений
Используется для удаления дубликатов из массива: `[...new Set(arrayWithDoublesValue)]`
`const setObject: Set<number> = new Set();`

Объект **Date**
`const myDate = new Date()` // текущая дата и время
Сторонние библиотеки для упрощения работы с датами: dayjs (https://day.js.org/)

[Вернуться к содержанию](#содержание)

## 9. Promise

**Promise**
`type MyType = {id: number; name: string}`
`function fetchData(): Promise<MyType>{`
`return fetch(<url>).then((res) => { response.json() as MyType })`

`}`

При использовании синтаксиса async/await:
`async function fetchData(): Promise<string>{...запрос; return data}`
`type ResultType = Awaited<ReturnType<fetchData>>`
`const result: ResultType = await fetchData()`

[Вернуться к содержанию](#содержание)

## 10. Utility Type

**Utility Type** - **нужно ознакомиться со всеми вариантами утилитарных типов**
* **Exclude<T, K>** - убирает типы K из T
   `type Colors = 'black' | 'white' | 'red'`
   `type BlackAndWhite = Exclude<Colors, 'red'>`

* **Extract<T, K>** - достает из T те типы, которые расширяют K
   `type Pass = 123 | 15 | 'pass' | 'token';`
   `type PassNumbers = Extract<Pass, number>;` // `123 | 15`

* **NonNullable<T>** - убирает все null и undefined
   `type ApiResponse = string | undefined | null;`
   `type ApiResponseValue = MyNonNullable<ApiResponse>` // `type ApiResponseValue = string`

`type Account = {`
`login: string;`
`password?: string;`

`}`

* **Required** делает поля объектов обязательными
   `type FullAccount = Required<Account>` // `type FullAccount = { login: string; password: string; }`

* **Partial** делает поля опциональными
   `type PartialAccount = Partial<Account>` // `type FullAccount = { login?: string; password?: string; }`

* **Pick** создаёт новый тип, выбирая из объекта заданные поля
   `type OnlyLogin = Pick<Account, 'login'>` // `type FullAccount = { login: string; }`

* **Omit** исключает заданные поля из объекта
   `type WithOutPassword = Omit<Account, 'password'>` // `type FullAccount = { login: string; }`

* **Record<Key, Value>** - типизирует ключ-значение у объекта
   `type Trafficlights = 'green' | 'yellow' | 'red';`
   `const descriptions: Record<Trafficlights, string> = {}`

* **Parameters и ReturnType** - выводят параметры и возвращаемое значение типа
   `function getHexString(color: 'black' | 'white' | 'red') {`
`if(color === 'black' return '#000'`
`else return '#fff'`

`}`

`type Color = Parameters<typeof getHexString>[0]` // `type Color = " "black" | "white"`
`type Hex = ReturnType<typeof getHexString>` // `type Hex = "#000" | "#fff" " `

[Вернуться к содержанию](#содержание)

## 11. keyof, infer, in и type guard

**keyof** - указывает, какие ключи могут быть у объекта (для создания функций принимающих аргументом только те ключи, которые есть в объекте)
`type Person: { name: string, age: number }`
`function acceptProperty(obj: Person, key: keyof Person) { return obj[key] }` // prop может быть равно **только** "name", "age"
`const nameValue = getProperty(person, 'name');` // nameValue имеет тип string
`const ageValue  = getProperty(person, 'age');` // ageValue имеет тип number

**infer** - извлекает и сохраняет типы для дальнейшего использования
`type ObjectProperty<T> = T extends { a: infer U } ? U : never;` // проверка содержит ли `T` свойство `a`

Если содержит, то `infer U` запоминает извлеченное свойство
`type ExampleObject1 = { a: string };`
`type ExtractedType1 = ObjectProperty<ExampleObject1>;` // Результат: string

`type ExampleObject3 = { b: boolean };`
`type ExtractedType3 = ObjectProperty<ExampleObject3>;` // Результат: never

**Пример с in**
`type FilterByProperty<Obj, Key extends string> = {`
`[K in Key]: K extends keyof Obj ? Obj : never;`

`}[Key];`
`{ name: 'Petr', age: 18 }` // выведем **все свойства объекта, если хотя бы 1 содержится в Key**

Если изменим:
`type FilterByProperty<Obj, Key extends string> = {`
`[K in Key]: K extends keyof Obj ? Obj[K] : never;`

`};`
`{ name: 'Petr', age: never }` - первое свойство принадлежит объекту, второе нет

**Пример вывода объектов, в которых содержится определенное свойство**
`type FilterByProperty<Obj, Key extends string> =`
`Obj extends infer O`
`? { [K in Key]?: any } extends O` // можно сократить `? O extends Record<Key, any>`
`? O`
`: never`

`: never;`
Если смогли извлечь из переданных данных переменную O и она соответствует `{ [K in Key]?: any }` или `Record<Key, any>`, то возвращаем `O`

`type Administrator = { name: string; };`
`type Developer = { name: string; computer: 'MacOS' | 'Windows'; };`

`type Personal = Administrator | Developer;`
`type WithComputers = FilterByProperty<Personal, 'computer'>;` // только в Developer содержится ключ computer

`<something>` **is** `<type>` - **принадлежность к типу**

`function func (obj: any): obj is MyType {...проверка соответствия типу MyType}` - функция - кастомный **typeguard** для проверки принадлежности к выбранному типу
Возвращаемое значение объясняет TS'у, что возвращаемый объект точно принадлежит проверяемогу типу

[Вернуться к содержанию](#содержание)

## 12. Рекурсия

**Рекурсия**
`type Flatten<T extends any[]> = T extends [infer First, ...infer Rest]`
`    ? First extends any[]`
`        ? Flatten<First> | Flatten<Rest>`
`        : [First] | Flatten<Rest>`

`    : [];`
Входные данные: `Flatten<[1, [2, [3, [4]]]]>`
Раскладываем масив на First (1) и Rest (`[ [2, [3, [4]]] ]`), если входные данные не массив, то результат пустой массив `[]`
Если First - массив, то рекурсивно вызываем Flatten для First и Rest, если не массив, то создаем из него массив `[First]` и рекурсивно вызываем Flatten для Rest
Далее раскладываем пока не закончатся вложенные массивы

[Вернуться к содержанию](#содержание)

## 13. Классы

*Создание класса:*
`class MyClass {`
`field1: string` // здесь прописываются все возможные свойства объекта. При использовании неуказанных свойств - ошибка
`field2?: number` // поля в классе также могут быть опциональными
`...`
`constructor(field1: string, field2?: number) {`
`this.field1 = field1;`
`...`

`}`
`method1(arg: string): string {}` // методы класса тоже нужно типизировать
`...`

`}`

*Можно сократить запись, используя модификаторы доступа:*
`class MyClass {`
`constructor(`**public** `field1: string, ...) {}` // внутри `{}` не нужно присваиваине. До конструктора не нужно описывать свойства объекта

`}`

*Геттеры и Сеттеры в TS*
`class MyClass {`
`name?: string` // опциональное поле
`get name(){`
`if(!this.name) {` // обязательная проверка существования поля
`throw new Error('Поля не существует')`

`}`
`return this.name`
`set name(value: string){`
`this.name = value`

`}`

`}`

**Интерфейсы** *объектов и классов*
`interface MyObj {`
`field1: string;`
`field2?: string;`
`field3: string|number;`
`method1(arg1: string): string` // метод
`method2?(arg1: string): string` // опциональный метод

`}`
`const obj: MyObj = {}`

Можно задать тип сразу всем полям, если они одного типа: `[number]: string`

Использование интерфейса в классе - **implements** : `class myClass implements MyClass {}`
Использование нескольких интерфейсов в одном классе: `class myClass implements MyClass1, MyClass2 {}`
*При множественной имплементации у интерфейсов не должно быть совпадающих свойств разного типа*

`console.log()` - использует объект *console* для вывода информации. Для того чтобы не зависеть от логгера (*объекта console*) можно реализовать его интерфейс
`interface ILogger {` // общая договоренность начинать имена интерфейсов с I + название класса/конструктора_объекта
`log(...args: (string|number)[]): void;`
`error(...args: (string|number)[]): void;`

`}`
`function func(text: string, logger: ILogger) {`
`logger.log(text)`

`}`
`func('Petr', console)` // вместо console можно использовать любой логгер подходящий под интерфейс

**Отличие типов от интерфейсов**:
* типы поддерживают пересечение (&) и объединение (|)
  Интерфейсы и типы можно пересекать (&) и объединять (|). При этом интерфейс трансформируется в тип:
  `type MyType = {}`
  `inteface MyInterface = {}`
  `const variable: MyType && MyInterface = ''`
* интерфейсы можно расширять с помощью **extends**:
  `inteface Interface1 {};`
  `inteface Interface2 extends Interface1{}`
* интерфейсы можно имплементировать классами: `class MyClass implements MyInterface`
  Тип можно имплементировать, но он трансформируется в интерфейс!
  Имплементировать тип являющийся объединением нельзя


[Вернуться к содержанию](#содержание)

## 14. Дженерики

**При описании используй пример из https://www.youtube.com/watch?v=-6DWwR_R4Xk 31:14**

`function myFunc`**\<T\>**`(arr: T[]): T | undefined {}` // **дженерик функция**

Тип параметра (\<T\>) заранее неизвестен, но вполне конкретен. Тип станет известен при передаче значения в вызове функции
Несколько параметров: \<T, K\>
Тип указывается перед (). Пример стрелочной функции: `<T>() => {}`, именно для стрелочной функции, чтобы не было конфликта с JSX-синтаксисом лучше писать \<T,\> с висячей запятой

Вызов функции: `myFunc(['Petr', 'Ivan'])` - тип \<T\> автоматически определится как `string`
Такой обобщенный тип не любой (any). Ему соответствует заранее неизвестный, но определенный в момент выполнения кода

`interface MyInterface<T> { name: T; }` // **дженерик интерфейса**

`type MyType<T> = T | undefined` // **дженерик типа**

Для использования таких интерфейса или типа нужно явно указать значение дженерика:
`type Post = { name: string, age: number }`
`function handleResponse(response: MyInterface<Post>): MyType<Post>`

`class MyClass<T> { field: T; method: (x: T, y: T) => T }` // **дженерик класса**

Тип дженерика указавается явно или определяется автоматически:
`new MyClass(0)`
`new MyClass<number>(0)`

Можно **ограничить** возможные типы дженерика:
`type MyType<T` **extends** `string> = { name: T }`
`type VariativeName = 'Petr' | 'Ivan'` // enum расширяется типом string

`type EnumType = MyType<VariativeName>`
`type StringType = MyType<string>`

Ограничить можно определенным типами объектов (например включающими поле name):
`type MyType = { name: string }`
`function myFunc<T extends MyType>(data: T): T {}`

Ограничить можно определенным классом и его наследниками:
`class Parent {}`
`class Child extends Parent {}`
`function MyFunc<T extends Parent>(arg: T){ return arg }`

`const parent = MyFunc(new Parent())` // передали и вернули класс Parent
`const parent = MyFunc(new Child())` // передали и вернули наследуемый от класса Parent класс Child

Если не использовать дженерик, а напрямую указать зависимость от класса Parent, то передавая класс наследник возвращенный тип будет Parent:
`function myFunc(arg: Parent){ return arg }`
`const child = myFunc(new Child())` // вернули тип класса Parent

**Значения по умолчанию**
`function myFunc<T extend string = string>(data: string): T | undefined {}`
`myFunc('Petr')` // в этом случае тип дженерика можно не указывать явно (string | undefined)

**Условные типы**
`Current extends Base ? True : False` // Если Current наследует Base, то тип True, иначе False

`type Secret<T> = T extends undefined ? null : T;` // TS проверяет каждый тип в отдельности
`type Result = Secret<number | undefined>;` // `number | null`

`type Secret<T> = T extends undefined ? null : T;` // проверяет свойства по отдельности
`type Result = Secret<number | undefined>;` // в Result `number | null`

[Вернуться к содержанию](#содержание)

## 15. Файлы .d.ts

Нужны для подсказок типов, используемых в обычном JS-скрипте, разработчикам и IDE
Для установки типов для библиотеки `npm install @types/library-name --save-dev`
Файл .d.ts называется также, как и типизируемый скрипт!
В описании нужно описать модуль: `declare module 'my-module' {...описание типов/интерфейсов/функций и т.д.}`
Также все описываемые типы/модули/... нужно экспортировать для использования вне файла по тем же принципам как происходит export/import в JS.
Модули могут быть:
* Амбиентные - используются когда нужно добавить типы для существующих переменных в глобальной области видимости
* Глобальные - доступны глобально `declare global {}`
* namespace - `namespace MyNamespace{ export MyType: string}` - использование в коде: `MyNamespace.MyType`
  устаревший подход, все делается через export/import первых двух типов

[Вернуться к содержанию](#содержание)

## 16 Type Challenges

```typescript
type MyPick<T, K extends keyof T> = {
    [key in K]: T[key]
}

type MyReadonly<T> = {
    readonly [P in keyof T]: T[P]
}

type TupleToObject<T extends readonly PropertyKey[]> = {
     [P in T[number]]: P
}

type First<T extends unknown[]> = T extends [] ? never : T[0]

type Length<T extends readonly unknown[]> = T['length']

type MyExclude<T, U> = T extends U ? never : T

type If<T extends Boolean, K, U> = T extends true ? K : U

type Concat<T extends unknown[], K extends unknown[]> = [...T, ...K]

type IsEqual<T, U> =
  (<G>() => G extends T ? 1 : 2) extends
  (<G>() => G extends U ? 1 : 2)
    ? true
    : false

type Includes<T extends readonly unknown[], U> =
  T extends readonly [infer First, ...infer Rest]
    ? IsEqual<First, U> extends true
      ? true
      : Includes<Rest, U>
    : false

type Push<T extends unknown[], K> = [...T, K]
type Unshift<T extends unknown[], U> = [U, ...T]

type MyParameters<T> = T extends (...args: infer U) => any ? U : never

type MyAwaited<T> = T extends PromiseLike<infer U> ? U extends PromiseLike<any> ? MyAwaited<U> : U : never
```

[Вернуться к содержанию](#содержание)

## 17. Полезные ссылки

[Еще раз очень подробно о Typescript](https://www.youtube.com/watch?v=LWtHl__oEWc)  
[Типизация React/Axios/Redux](https://www.youtube.com/watch?v=ApIStAZa8Vc)

[Вернуться к содержанию](#содержание)
