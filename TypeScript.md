# Содержание

- [1. Установка и TSconfig](#1-установка-и-tsconfig)
- [2. Базовые типы](#2-базовые-типы)
  - [2.1 unknown, null, undefined, void, never](#21-unknown-null-undefined-void-never)
- [3. Функции](#3-функции)
  - [3.1 Параметры функции](#31-параметры-функции)
- [4. Массивы и кортежи](#4-массивы-и-кортежи)
- [5. Объекты](#5-объекты)
- [6. Собственные типы, интерфейсы](#6-собственные-типы-интерфейсы)
- [7. Enum](#7-enum)
- [8. DOM и встроенные объекты](#8-dom-и-встроенные-объекты)
- [9. Promise](#9-promise)
- [10. Дополнительные приемы типизации](#10-дополнительные-приемы-типизации)
- [11. Рекурсия](#11-рекурсия)
- [12. Дженерики](#12-дженерики)
- [13. Utility Type](#13-utility-type)
- [14. Файлы .d.ts](#14-файлы-dts)
- [15. Кастомные типы](#15-кастомные-типы)
- [16. Полезные ссылки](#16-полезные-ссылки)

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

* `type` поддерживает объединение (`|`) и пересечение (`&`)
Интерфейсы и типы можно пересекать и объединять. При этом интерфейс трансформируется в тип.
* `interface` можно расширять через `extends`
* `interface` можно использовать в классе через `implements`
Тип можно имплементировать, но он трансформируется в интерфейс
>При этом имплементировать тип являющийся объединением нельзя

* `type` нельзя объявить повторно с тем же именем
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

```typescript
enum Race {
	Ork = 'Ork',
	Human = 'Human',
}

let race = Race.Ork
```

Компилируется примерно в:

```javascript
var Race;
(function (Race) {
	Race["Ork"] = "Ork";
	Race["Human"] = "Human";
})(Race || (Race = {}));
let race = Race.Ork;
```

`const enum` **теряет** часть функциональности при компиляции:

```typescript
const enum Race {
	Ork = 'Ork',
	Human = 'Human',
}

let race = Race.Ork
```

Не компилируется в JS, а заменяется фактическим значением:

```javascript
let race = "Ork" /* Race.Ork */;
```

Из-за этого у `const enum` есть ограничения:

1. Нельзя использовать его как обычный runtime-объект: `for (let key in Race) {}` или `const enumKey = Race[race]`
2. Значения должны быть вычислимы компилятором: `const enum Size { small = config.smallSize }` Ошибка.

Для сохранения `const enum` при компиляции, используется флаг `preserveConstEnums` в конфиге.

[Небольшая статья](https://phuoc.ng/collection/this-vs-that/const-enum-vs-enum/)
[When can the TypeScript typechecker not inline via const enum?](https://www.susanpotter.net/software/typescript-enum-versus-const-enum/)

[Вернуться к содержанию](#содержание)

## 8. DOM и встроенные объекты

**DOM-элементы**

При поиске DOM-элемента он может быть не найден, поэтому результат может содержать `null`  
Перед использованием элемент нужно проверить, что он найден

```typescript
const element: HTMLElement | null = document.querySelector('.my-class')

if (element) {
   element.innerText = 'text'
}
```

Если нужен конкретный тип элемента, его лучше указать явно: `const button = document.querySelector<HTMLButtonElement>('.button')`

**Типизация событий**

```typescript
function handleClick(event: MouseEvent): void {
	console.log(event.clientX)
}

button?.addEventListener('click', handleClick)
```

**Map** - коллекция пар ключ-значение:

```typescript
const mapObject = new Map<string, number>()
mapObject.set('count', 1)
```

**Set** - коллекция уникальных значений:

```typescript
const setObject = new Set<number>()
setObject.add(1)
```

[Вернуться к содержанию](#содержание)

## 9. Promise

Типизация Promise:

```typescript
type User = {
	id: number
	name: string
}

function fetchData(): Promise<User> {
	return fetch('/user')
		.then((response) => response.json())
		.then((data) => data as User)
}
```

Типизация с синтаксисом  `async/await`:

```typescript
async function fetchName(): Promise<string> {
	const response = await fetch('/name')
	const data = await response.text()

	return data
}
```

`Awaited<T>` достает тип значения из `Promise` на уровне типов: 
```typescript
type ResultType = Awaited<Promise<string>> // string
```  
Часто `Awaited` используют вместе с `ReturnType`, чтобы получить результат async-функции: 
```typescript
type FetchNameResult = Awaited<ReturnType<typeof fetchName>> // string
```  
`Awaited` умеет разворачивать вложенные promise: 
```typescript
type DeepResult = Awaited<Promise<Promise<number>>>  // number
```

[Вернуться к содержанию](#содержание)

## 10. Дополнительные приемы типизации

**keyof** - оператор, который получает union ключей объекта:

```typescript
type Person = {
	name: string
	age: number
}

type PersonKey = keyof Person // 'name' | 'age'
```

Используется для функция, которые принимают аргументом ключи объекта:

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
	return obj[key]
}

const person = { name: 'Petr', age: 18 }

const nameValue = getProperty(person, 'name') // string
const ageValue = getProperty(person, 'age') // number
```

**in** в типах используется как перебор ключей объекта или union-ключей:

```typescript
type Copy<T> = {
    // `P` по очереди становится каждым ключом из `keyof T`
	[P in keyof T]: T[P]
}

type Status = 'success' | 'error'

type StatusMap = {
   [P in Status]: boolean
}

// {
// 	success: boolean
// 	error: boolean
// }
```

**infer** используется **только!** внутри conditional type и позволяет достать внутренний тип:

```typescript
type ArrayItem<T> = T extends (infer Item)[] ? Item : never

type Result = ArrayItem<string[]> // string
```

Пример с объектом:

```typescript
type ObjectProperty<T> = T extends { a: infer U } ? U : never

type First = ObjectProperty<{ a: string }> // string
type Second = ObjectProperty<{ b: boolean }> // never
```

Пример с функцией:

```typescript
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never

type Result = MyReturnType<() => number> // number
```

**type guard** - проверка, после которой TypeScript сужает тип.

Обычные проверки тоже являются type guard:

```typescript
function print(value: string | number) {
	if (typeof value === 'string') {
		return value.toUpperCase()
	}

	return value.toFixed(2)
}
```

Кастомный type guard пишется через `value is Type`:

```typescript
type User = {
	name: string
}

function isUser(value: unknown): value is User {
	return typeof value === 'object' && value !== null && 'name' in value
}

function printUser(value: unknown) {
	if (isUser(value)) {
		console.log(value.name)
	}
}
```

**satisfies** проверяет, что значение подходит под тип, но не затирает более точный выведенный тип самого значения.

```typescript
type Color = 'red' | 'green' | 'blue'
type RGB = [number, number, number]

const palette = {
	red: [255, 0, 0],
	green: '#00ff00',
	blue: [0, 0, 255]
} satisfies Record<Color, string | RGB>

palette.green.toUpperCase()
```

**as const** делает значение максимально узким и readonly  
Используется, когда из обычного значения нужно получить точные литеральные типы

```typescript
const status = 'success' as const // 'success'
const tuple = ['first', 'second'] as const // readonly ['first', 'second']
```

**typeof в типах** достает тип уже существующего значения:

```typescript
const config = {
	apiUrl: '/api',
	retry: 3
}

type Config = typeof config
```

**Discriminated union** - union объектов с общим полем-маркером:

```typescript
// Поле `status` позволяет вычислить какой вариант union сейчас используется.
type Result =
	| { status: 'success'; data: string }
	| { status: 'error'; message: string }

function handleResult(result: Result) {
	if (result.status === 'success') {
		return result.data
	}

	return result.message
}
```

**Template literal types** позволяют собирать строковые типы по шаблону:

```typescript
type Method = 'get' | 'post'
type Entity = 'User' | 'Post'

type ApiMethod = `${Method}${Entity}` // 'getUser' | 'getPost' | 'postUser' | 'postPost'
```

[Вернуться к содержанию](#содержание)

## 11. Рекурсия

**Рекурсия в типах** - ситуация, когда тип вызывает сам себя.

Пример - развернуть вложенные `Promise`:

```typescript
type MyAwaited<T> = T extends PromiseLike<infer U> ? MyAwaited<U> : T

type Result = MyAwaited<Promise<Promise<string>>> // string
```

Объяснение:

* `T extends PromiseLike<infer U>` - проверяем, является ли `T` promise-подобным типом.
* `infer U` достает тип внутри `Promise`.
* Если внутри снова `Promise`, вызываем `MyAwaited<U>` еще раз.
* Если `Promise` больше нет, возвращаем сам `T`.

Пример с массивом/кортежем - расплющить вложенные массивы:

```typescript
type Flatten<T extends unknown[]> =
	T extends [infer First, ...infer Rest]
		? First extends unknown[]
			? [...Flatten<First>, ...Flatten<Rest>]
			: [First, ...Flatten<Rest>]
		: []

type Result = Flatten<[1, [2, [3]], 4]> // [1, 2, 3, 4]
```

Разбор:

* `T extends [infer First, ...infer Rest]` делит кортеж на первый элемент и остаток.
* Если `First` тоже массив, рекурсивно раскрываем его.
* Если `First` не массив, кладем его в результат.
* `[]` - базовый случай, когда элементов больше нет.

[Вернуться к содержанию](#содержание)

## 12. Дженерики

**Дженерик** - параметр типа. Используется, когда тип заранее неизвестен, но между входными и выходными данными есть связь.  
TypeScript автоматически выводит конкретный тип при вызове функции.  

Пример:

```typescript
function first<T>(arr: T[]): T | undefined {
	return arr[0]
}

const firstName = first(['Petr', 'Ivan']) // string | undefined
const firstNumber = first([1, 2, 3]) // number | undefined
```

Дженерик можно указать явно:

```typescript
const value = first<string>(['Petr', 'Ivan'])
```

Дженерик с несколькими параметрами:

```typescript
function pair<T, U>(first: T, second: U): [T, U] {
	return [first, second]
}

const result = pair('id', 1) // [string, number]
```

Дженерик в стрелочной функции:

```typescript
const identity = <T,>(value: T): T => {
	return value
}
```

>Запятая в `<T,>` часто нужна в `.tsx`, чтобы TypeScript не спутал дженерик с JSX-тегом.

Дженерик можно использовать в type и interface:

```typescript
type ApiResponse<T> = {
	data: T
	error?: string
}

interface Box<T> {
	value: T
}

type User = {
	name: string
}

const response: ApiResponse<User> = {
	data: { name: 'Petr' }
}
```

Можно ограничить возможные значения типа дженерика через `extends`:

```typescript
function getLength<T extends { length: number }>(value: T): number {
	return value.length
}

getLength('text')
getLength([1, 2, 3])
// getLength(123) // ошибка, у number нет length
```

Пример с ключами объекта:

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
	return obj[key]
}

const user = { name: 'Petr', age: 18 }

getProperty(user, 'name') // string
getProperty(user, 'age') // number
// getProperty(user, 'email') // ошибка
```

**Значение типа по умолчанию** указывается через `=`.

```typescript
type ApiResponse<T = unknown> = {
	data: T
	error?: string
}

type UnknownResponse = ApiResponse
type StringResponse = ApiResponse<string>
```

**Условные типы** часто пишутся через дженерики:

```typescript
type CurrentOrNull<T> = T extends undefined ? null : T

type First = CurrentOrNull<string> // string
type Second = CurrentOrNull<undefined> // null
```

```typescript
type Secret<T> = T extends undefined ? null : T

// union проверяется по частям
type Result = Secret<number | undefined> // number | null
```

[Вернуться к содержанию](#содержание)

## 13. Utility Type

**Utility Types** - встроенные типовые операции TypeScript. 

**Partial<T>** делает все свойства опциональными:

```typescript
type PartialAccount = Partial<Account>

// {
// 	login?: string
// 	password?: string
// 	role?: 'user' | 'admin'
// }
```

**Required<T>** делает все свойства обязательными:

```typescript
type RequiredAccount = Required<Account>

// {
// 	login: string
// 	password: string
// 	role: 'user' | 'admin'
// }
```

**Readonly<T>** делает свойства только для чтения:

```typescript
type ReadonlyAccount = Readonly<Account>

const account: ReadonlyAccount = {
	login: 'petr',
	role: 'user'
}

// account.login = 'ivan' // ошибка
```

**Pick<T, K>** создает новый тип, выбирая только нужные поля:

```typescript
type PublicAccount = Pick<Account, 'login' | 'role'>

// {
// 	login: string
// 	role: 'user' | 'admin'
// }
```

**Omit<T, K>** создает новый тип, исключая указанные поля:

```typescript
type AccountWithoutPassword = Omit<Account, 'password'>

// {
// 	login: string
// 	role: 'user' | 'admin'
// }
```

**Record<Key, Value>** описывает объект, где набор ключей и тип значений известны заранее:

```typescript
type TrafficLight = 'green' | 'yellow' | 'red'

const descriptions: Record<TrafficLight, string> = {
	green: 'go',
	yellow: 'wait',
	red: 'stop'
}
```

**Exclude<T, U>** убирает из `T` типы, которые подходят под `U`:

```typescript
type Colors = 'black' | 'white' | 'red'
type BlackAndWhite = Exclude<Colors, 'red'> // 'black' | 'white'
```

**Extract<T, U>** оставляет из `T` только типы, которые подходят под `U`:

```typescript
type Pass = 123 | 15 | 'pass' | 'token'
type PassNumbers = Extract<Pass, number> // 123 | 15
```

**NonNullable<T>** убирает `null` и `undefined`:

```typescript
type ApiResponse = string | undefined | null
type ApiResponseValue = NonNullable<ApiResponse> // string
```

**Parameters<T>** достает тип параметров функции в виде кортежа:

```typescript
function getHexString(color: 'black' | 'white' | 'red'): string {
	if (color === 'black') return '#000'
	return '#fff'
}

type Params = Parameters<typeof getHexString> // ['black' | 'white' | 'red']
type Color = Params[0] // 'black' | 'white' | 'red'
```

**ReturnType<T>** достает тип возвращаемого значения функции:

```typescript
type Hex = ReturnType<typeof getHexString> // string
```

**Awaited<T>** достает тип результата из `Promise`:

```typescript
type Result = Awaited<Promise<string>> // string
type DeepResult = Awaited<Promise<Promise<number>>> // number
```

[Вернуться к содержанию](#содержание)

## 14. Файлы .d.ts

`.d.ts` - файл с описанием типов без реализации.  
Нужен, когда JavaScript-код уже есть, а TypeScript должен понимать его типы.

Для популярных библиотек типы обычно ставятся отдельно:

```bash
npm install --save-dev @types/library-name
```

Глобальные типы добавляются через `declare global`:

```typescript
declare global {
	interface Window {
		appVersion: string
	}
}

export {}
```

`export {}` в конце нужен, чтобы файл считался модулем, а `declare global` корректно расширял глобальные типы.

[Вернуться к содержанию](#содержание)

## 15. Кастомные типы

Примеры кастомных типов из [Type Challenges](https://github.com/type-challenges/type-challenges). Практика для TS

**MyPick** - выбирает из объекта только указанные ключи:

```typescript
type MyPick<T, K extends keyof T> = {
	[P in K]: T[P]
}
```

* `K extends keyof T` - `K` может быть только ключом объекта `T`.
* `[P in K]` - перебираем переданные ключи.
* `T[P]` - берем тип значения по текущему ключу.

**MyReadonly** - делает все свойства объекта readonly:

```typescript
type MyReadonly<T> = {
	readonly [P in keyof T]: T[P]
}
```

`readonly` ставится перед ключом в mapped type.

**TupleToObject** - превращает кортеж значений в объект:

```typescript
const tuple = ['tesla', 'model 3', 'model X'] as const

type TupleToObject<T extends readonly PropertyKey[]> = {
	[P in T[number]]: P
}

type Result = TupleToObject<typeof tuple>
// {
// 	tesla: 'tesla'
// 	'model 3': 'model 3'
// 	'model X': 'model X'
// }
```

* `as const` превращает массив в readonly-кортеж с литеральными типами.
* `PropertyKey` - встроенный тип для ключей объекта: `string | number | symbol`.
* `T[number]` достает union всех значений массива или кортежа.
* `[P in T[number]]` перебирает эти значения и делает их ключами объекта.

**First** - достает первый элемент кортежа:

```typescript
type First<T extends readonly unknown[]> = T extends [] ? never : T[0]
```

Если кортеж пустой, возвращаем `never`. Иначе берем элемент по индексу `0`.

**Length** - достает длину кортежа:

```typescript
type Length<T extends readonly unknown[]> = T['length']
```

У кортежа `length` может быть конкретным числом:

```typescript
type Result = Length<readonly ['a', 'b']> // 2
```

**MyExclude** - убирает из union те типы, которые подходят под `U`:

```typescript
type MyExclude<T, U> = T extends U ? never : T

type Result = MyExclude<'a' | 'b' | 'c', 'a'> // 'b' | 'c'
```

Так работает из-за distributive conditional types: union слева проверяется по частям.

**If** - выбирает один из двух типов по boolean-условию:

```typescript
type If<C extends boolean, T, F> = C extends true ? T : F

type Result = If<true, 'yes', 'no'> // 'yes'
```

`C extends boolean` ограничивает первый аргумент только `true` или `false`.

**Concat** - соединяет два кортежа:

```typescript
type Concat<T extends readonly unknown[], U extends readonly unknown[]> = [...T, ...U]

type Result = Concat<[1, 2], [3, 4]> // [1, 2, 3, 4]
```

`...T` и `...U` здесь работают на уровне типов, как spread у массивов в JavaScript.

**Includes** - проверяет, есть ли тип в кортеже:

```typescript
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
```

`IsEqual` выглядит странно, но смысл простой: он сравнивает два типа именно на равенство, а не просто через `extends`.

В `Includes` используется рекурсия:

* `infer First` достает первый элемент.
* `infer Rest` достает остаток кортежа.
* Если `First` равен `U`, возвращаем `true`.
* Если нет, запускаем `Includes<Rest, U>`.
* Если элементы закончились, возвращаем `false`.

**Push** и **Unshift** - добавляют элемент в конец или начало кортежа:

```typescript
type Push<T extends readonly unknown[], U> = [...T, U]
type Unshift<T extends readonly unknown[], U> = [U, ...T]

type First = Push<[1, 2], 3> // [1, 2, 3]
type Second = Unshift<[1, 2], 0> // [0, 1, 2]
```

**MyParameters** - достает параметры функции в виде кортежа:

```typescript
type MyParameters<T extends (...args: any[]) => any> =
	T extends (...args: infer P) => any ? P : never

type Result = MyParameters<(name: string, age: number) => void>
// [name: string, age: number]
```

`infer P` достает тип rest-параметров функции.

**MyAwaited** - достает тип из `Promise`, включая вложенные promise:

```typescript
type MyAwaited<T extends PromiseLike<any>> =
	T extends PromiseLike<infer U>
		? U extends PromiseLike<any>
			? MyAwaited<U>
			: U
		: never

type Result = MyAwaited<Promise<Promise<string>>> // string
```

Здесь вместе работают `infer` и рекурсия: достаем внутренний тип, проверяем его еще раз и повторяем, пока внутри не останется обычное значение.

[Вернуться к содержанию](#содержание)

## 16. Полезные ссылки

[Еще раз очень подробно о Typescript](https://www.youtube.com/watch?v=LWtHl__oEWc)  
[Типизация React/Axios/Redux](https://www.youtube.com/watch?v=ApIStAZa8Vc)

[Вернуться к содержанию](#содержание)
