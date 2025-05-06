# Содержание
* [React под капотом](#react-под-капотом)
* [1. Создание компонента](#1-создание-компонента)  
	- [1.1 Общее](#11-общее)  
	- [1.2 Функциональный компонент](#12-функциональный-компонент)  
	- [1.3 Классовый компонент (устаревший)](#13-классовый-компонент-устаревший)  
* [2. Props](#2-props)  
* [3. Списки](#3-списки)  
* [4. Хуки](#4-хуки)  
	- [4.1 useState](#41-usestate)  
		+ [4.1.1 Immer](#411-immer)  
		+ [4.1.2 Batching](#412-batching)  
		+ [4.1.3 flushSync](#413-flushSync)  
		+ [4.1.4 Ошибки](#413-ошибки)  
	- [4.2 useReducer](#42-usereducer)
	- [4.3 useContext](#43-usecontext)
	- [4.4 useRef](#44-useref)
	- [4.5 useImperativeHandle](#45-useimperativehandle)
	- [4.6 useEffect](#46-useeffect)
		- [4.6.1 Случаи неправильного использования useEffect](#461-случаи-неправильного-использования-useEffect)
	- [4.7 useLayoutEffect](#47-uselayouteffect)
	- [4.8 useInsertionEffect](#48-useinsertioneffect)
	- [4.9 useMemo](#49-usememo)
		- [4.9.1 React.memo](#410-reactmemo)
	- [4.10 useCallback](#411-usecallback)
	- [4.11 useDeferredValue](#412-usedeferredvalue)
	- [4.12 useTransition](#413-usetransition)
	- [4.13 useSyncExternalStore](#414-usesyncexternalstore)
	- [4.14 useId](#413-useid)
	- [4.15 useOptimistic](#417-useoptimistic)
	- [4.16 useDebugValue](#418-usedebugvalue)
	- [4.17 useActionState](#419-useactionstate)
	- [4.18 useFormStatus](#420-useformstatus)
	- [4.19 Кастомные хуки](#421-кастомные-хуки)
* [5. Пользовательские компоненты](#5-пользовательские-компоненты) 
	- [5.1 Suspense](#51-suspense)
		- [5.1.1 Lazy Loading](#511-lazy-loading)
	- [5.2 Error Boundary](#52-error-boundary)
	- [5.3 HOC (High Order Component)](#53-hoc-high-order-component)
	- [5.4 Compound component](#54-compound-component)
* [6. ReactDOM. Элементы и события React](#6-reactdom-элементы-и-события-react) 
* [7. Portals](#7-portals) 
* [8. Переменные окружения в React](#8-переменные-окружения-в-react) 
* [9. Серверный рендеринг](#9-серверный-рендеринг) 
	- [9.1 Директивы](#91-директивы)
		- [9.1.1 use client](#911-use-client)
		- [9.1.2 use server](#912-use-server)
	- [9.2 React Server Component](#92-react-server-component)
	- [9.3 Server Action](#93-server-action)
	- [9.4 cache](#94-cache)
* [10. Проекция состояния класса (редкий кейс)](#10-проекция-состояния-класса-редкий-кейс) 
	- [10.1 Класс с редко изменяемым состоянием](#101-класс-с-редко-изменяемым-состоянием)
	- [10.2 Класс с часто изменяемым состоянием](#102-класс-с-часто-изменяемым-состоянием)
	- [10.3 Класс без собственных состояний](#103-класс-без-собственных-состояний)
* [Дополнительно](#дополнительно) 
	
## React под капотом

[Простыми вещами React под капотом](https://www.youtube.com/watch?v=HDajDASxn-w)  
[Немного покороче](https://www.youtube.com/watch?v=A0W2n2azH5s)  
[Подробно на английском ч1](https://www.youtube.com/watch?v=7YhdqIR2Yzo)  
[Подробно на английском ч2](https://www.youtube.com/watch?v=0ympFIwQFJw)  

_Virtual DOM устаревшее понятие._
При первом рендере React создает дерево React элементов (RE), куда распаковываются из JSX наши компоненты через `React.createElement`.  
Каждому RE ставится в соответствие Fiber, т.е. создается Fiber-дерево (на самом деле связанный список Linked List), где корневым элементов Fiber-дерева является root-элемент.  
В связанном списке:
* родительский fiber имеет ссылку на один дочерний fiber
* каждый дочерний fiber имеет ссылку на родительский fiber
* опционально (при наличии) fiber имеет ссылку на следующий дочерний fiber  

**Fiber** - хранит состояние RE, информацию о "работе"(эффект): запрос данных из API, подписки, манипуляции с DOM, которую необходимо произвести  после рендеринга.  
В его состав входит:
```
{
	stateNode, // ссылка на экземпляр компонента (для функциональных - null, классовых - экземпляр класса, DOM-узел - ссылка на DOM-элемент)
	memoizedProps, // пропсы использованные при последнем рендере (позволяет отслеживать изменения при ререндере)
	memoizedState,  // актуальное состояние после последнего рендера, используемое для последующих ререндеров   
	dependencies, // зависимости, которые могут вызвать повторный рендеринг
	pendingProps, // пропсы, которые компонент должен получить при следующем рендере (временное хранилище до завершения рендера)
	updateQueue, // очередь обновлений состояний компонента (содержит список изменений, которые должны быть применены к компоненту)
	key // уникальный ключ для идентификации (используется для оптимизации обновления списка элементов)
	
	child, // ссылка на первый дочерний узел (используется для навигации по дереву Fiber)
	sibling, // ссылка на следующий узел (используется для навигации по дереву Fiber)
	return, // ссылка на родительский узел (используется для навигации по дереву Fiber)
	...
}
```
RE может пересоздаваться, а Fiber мутирует,сохраняя новые данные о нем. Fiber сохраняет связь с RE на всем жизненном цикле, при размонтировании компонента Fiber уничтожается.  
Для обхода дерева и поиска изменений используются алгоритмы со сложностью `O(N^3)`, что неоптимально. В React используется эвристический подход, что снижает сложность до `O(N)`  
Эвристика (Diffing Algorithm): 
1. разные типы элементов (span => div) относятся к разным деревьям (изменении позиции элементов приводит к ререндеру) 
2. ключи элементов сохраняются между ререндерами (они позволяют исключить лишние ререндеры, если компонент не был изменен, но была изменена позиция)
```typescript
// компонент находится на разных позициях, поэтому он перемонтируется, запуская все собственные эффекты заново
function Parent(isVisible: boolean) {
	if(isVisible) {
		return <>
			<OptionalComponent />
			<ChildComponent />
		</>
	}
	return <ChildComponent />
}
```
Чтобы избежать перемонтирования компонента нужно сохранить его позицию в DOM с использованием условного рендеринга или добавить key  
```typescript
function Parent(isVisible: boolean) {
	if(isVisible) {
		return <>
			<OptionalComponent />
			// содержит уникальный ключ, позволяющий react не перемонтировать его при отсутствии изменений в нем
			<ChildComponent key='uniqueKey' />
		</>
	}
	// позиция компонента изменилась со 2 на 1
	return <ChildComponent key='uniqueKey' />
}

// или 

function Parent(isVisible: boolean) {
		return <>
			{isVisible && <OptionalComponent /> }
			// остается вторым дочерним компонентом независимо от условия
			<ChildComponent key='uniqueKey' />
		</>
	}
}

```

**Reconciliation** - определение изменений, которые нужно внести в DOM при ререндере для синхронизации с текущим состоянием.  
Процесс reconciliation идет в 2 этапа:
1. Render Phase (асинхронная - прерывается)
2. Commit Phase (синхронная - не прерывается)

В фазу рендеринга:
1. Создание нового дерева Fiber **WorkInProgress Tree** на основе текущего **Current Tree**
2. **DOM-Diffing** - сравнение деревьев 
На основе полученных изменений React собирает все необходимые эффекты в Effect List, где они приоретизированы по внутренней логике React.  
WorkInProgress Tree становится Current Tree, а предыдущее Current Tree удаляется.  
_Приоритезация эффектов:_
* Высокий приоритет - анимации, взаимодействие с пользователем (ввод текста, клики) 
* Низкий приоритет - фоновые задачи (логирование, обновление кэша)

В фазу коммита происходит:
1. Mutation - внесение изменений в реальный DOM на основе различий, найденных в фазу рендеринга
2. Layout - выполняются useLayoutEffect, componentDidMount, componentDidUpdate

**Concurrency** в разрезе reconciliation: 
Прервать выполнение можно только в Render Phase. Высокоприоритетная задача, приостанавливает текущую работу.  
При этом уже собранные эффекты остаются в Effect List, WorkInProgress Tree остается частично завершенным.  
После обработки приоритетной задачи React может возобновить рендеринг с того места, где он был приостановлен, а если произошло изменение состояния, то Render Phase начнется заново 

**Concurrency** - конкурентный режим в общем смысле.
Возможность прервать низкоприоритетную задачу более высокоприоритетной задачей, при этом состояние переданное в низкоприоритетную задачу сбрасывается до предыдущего. 
К низкоприоритетным задачам относятнся: useTransition, startTransition, useDefferedValue, к высокоприоритетным: useState, useReducer, useSyncExternalState.
[Более подробно про Concurrency](https://www.youtube.com/watch?v=M1OBMTYsKpo)

**SSR**
[Про SSR и его преимущества](https://www.youtube.com/watch?v=pj5N-Khihgc)

[Вернуться к содержанию](#содержание)

# 1. Создание компонента

## 1.1 Общее

Сниппеты webStorm: 
* rsi - функциональный компонент - стрелочная функция без return
* rsf - функциональный компонент
* rsc - функциональный компонент - стрелочная функция
* rscp - функциональнй компонент - стрелочная функция с PropType
* rsfp - функциональный компонент с PropType
* rcc - классовый компонент
* rccp - классовый компонент с PropType
* rcfc- классовый компонент с PropType и методами жизненного цикла

Компоненты пишутся с большой буквы и возвращают JSX-код.  
С 17 версии нет необходимости импортировать `React from 'react'`, т.к. JSX Transform автоматически проставляет импорты для JSX кода   
JSX - расширение синтаксиса JS, представляющее HTML-подобную разметку внутри JS  
**Отличия JSX от HTML**:  
1. Все элементы должны иметь закрывающийся тег, в том числе `<input/>`, `<img/>` и т.д.
2. Большинство вещей в записывается в CamelCase.  
Атрибуты записываются в CamelCase. *Исключение data- и aria- атрибуты* (любые с '-')  
Атрибут `class` устанавливается через атрибут `className`, т.к. `class` зарезервирован для создания  класса  
Атрибут `for` устанавливается через атрибут `htmlFor`  
3. Возвращается один корневой элемент: 
	* Для возвращата одного элемента, который убирается в одну строку:  
	`return <div>...код</div>`
	* Для возврата одного элемента в многострочном коде: 
	```typescript
	return (
		<div>
			...код
		</div>
	)
	```
	* Для группировки нескольких элементов используется Fragment-синтаксис `<></>`. Fragment не влияет на разметку DOM или стили  
		**В DOM не добавляется группирующий родительский элемент**  
		Если при этом нужно передать аргумент key (в цикле), то используется `<Fragment></Fragment>`  
		[Подробно о семантике HTML и Fragment](https://www.youtube.com/watch?v=duoNlz5uTYk)  
	```typescript
	return (
		<>
			<div></div>
			<div></div>
		</>
	}
	```
	
Компоненты React - **чистые функции**(pure function) и **НЕ ДОЛЖНЫ** изменять передаваемые данные  
Изменяться могут только данные, созданные внутри компонента.  
Компонент должен возвращать **одинаковый результат** при вызове с тем же набором аргументов, иначе это приведет ошибкам  
Любые сайд-эффекты нужно выносить в обработчики событий, useEffect и т.д.  
`<StrictMode></StrictMode>` - стресс-тест, запускающийся только в dev-режиме  
Вызывает дополнительный цикл `setup+clenup` для поиска ошибок "нечистого" рендеринга  
Цикл `setup->clenup->setup` в dev-режиме должен соответствовать setup в prod-режиме  

При использованиии условных конструкций `(condition) && (statement)`, `condition` **должен возвращать boolean** значение  
`[].length && <MyComponent>` - отрендерится 0, а не компонент.   
_Решение_: тернарный оператор, проверка length > 0, приведение к логическому типу  

Стили можно импортировать из CSS/SCSS модуля `someStyle.module.scss`  
CSS модуль позволяет изолировать стили, используя уникальное название: ИмяКомпонента_ИмяКласса_хэш  
Таким образом, игнорируя БЭМ можно в каждом CSS модуле описывать только модификаторы класса элемента (например `open`), не опасаясь перекрестного наложения стиля 

```typescript
import styles from './path/app.module.css'
function App() {
	const myStyle = { marginLeft: 10 }
	return (
		<>
			<div className="my_class1" style={{marginTop: 20}}></div>
			<div className="my_class2" style={myStyle}></div>
			<div className={styles.myStyle}></div>
		</>
	)
}

// someStyle.module.css
.myStyle {
	padding: 10px;
}

```
**НЕЛЬЗЯ** создавать один компонент внутри другого, так как компоненты - ссылочные типы данных. Каждый новый рендер это новый компонент  
[Как правильно компоновать компоненты](https://www.youtube.com/watch?v=UWC1XlNJSvc)  

[Вернуться к содержанию](#содержание)

## 1.2 Функциональный компонент

```typescript
type FunctionalComponentProps = { defaultValue: number }
// можно использовать стрелочную функцию
function FunctionalComponent({ defaultValue }: FunctionalComponentProps) { 
	// хуки описываются ДО основной логики
	const [value, setValue] = useState(defaultValue)
	// логика
	function myFunc() { setValue( value + 1 ) }
	return (
		{/* JSX-код */}
		<button onClick={myFunc}>{value}</button>
	)
}
```
Примеры типизации функционального компонента как стрелочной функции (лучше использовать один вариант в пределах проекта):
1. `const MyComponent = ({prop1, prop2}: CustomPropsType)`, где `CustomPropsType` - тип, передаваемых пропсов  
2. `const MyComponent = ({prop1, prop2}: PropsWithChildren<CustomPropsType>)` - пересечение (Intersection) `CustomPropsType` и `children?: ReactNode | undefined` - устаревший в React 18
3. `const MyComponent: FC<Props> = (props) => { ... }` - устаревший метод  

[Вернуться к содержанию](#содержание)

## 1.3 Классовый компонент (устаревший)

![жизненный цикл классового компонента](https://pictures.s3.yandex.net/resources/Untitled_2_1706861897.png)  
```typescript
// если в компоненте не передаются пропсы, то передается пустой объект {} в дженерике
interface IProps { initialValue: number }
interface IState { value: number }
class ClassComponent extends React.Component<IProps, IState> {
	// props передаются в качестве аргументов конструктора
	constructor(props: IProps) {
		super(props)
		// состояние задается в конструкторе, в зарезервированном свойстве state
		this.state = {
			value: props.initialValue
		}
		this.increment = this.increment.bind(this) {/* методам внутри конструктора необходимо присваивать контекст */}
	}
	// вместо хуков используются методы
	// состояние не меняется напрямую, любые изменениея происходят через setState
	increment(){ this.setState({ value: this.state.value + 1 }) }
	// альтернативный способ создания метода (без привязки контекста в конструкторе)
	// в обоих случаях методы пренадлежат экземпляру класса, а не прототипу
	decrement = () => {
		this.setState((prevState) => ({
			value: prevState.value - 1,
		}))
	}
	
	// вывод компонента
	render() {
		return (
			{/* JSX-код */}
			<div>
				<p>{this.state.value}</p>
				<button onClick={this.increment}>+</button>
				<button onClick={this.decrement}>-</button>
			</div>
		)
	}
}

```
---

Жизненным циклом классового компонента:
* constructor - вызывается при создании компонента. Инициализирует состояние и привязывает методы к контексту  
Аналог useState
* componentDidMount() - вызывается после монтирования. Аналог useEffect с пустым массивом зависимостей.
* componentDidUpdate(prevProps, prevState) - вызывается после обновления внутреннего состояния или пропсов. **При первом рендере(монтировании) срабатывает `componentDidMount`**
prevProps - предыдущие пропсы, prevState - предыдущее состояние  
Для использования setState внутри метода, необходимо обернуть его в условие, чтобы не возник бесконечный цикл  
Аналог useEffect без зависимостей
* shouldComponentUpdate(nextProps, nextState) - вызывается перед рендерингом и сравнивает текущие значения props и/или state для разрешения повторного рендеринга (boolean). Используется для оптимизации мест с частым рендерингом  
	* true - при изменении props и/или state произойдет повторный рендеринг
	* false - при изменении props и/или state НЕ произойдет повторный рендеринг  
Аналог memo с передачей колбек-функции
* getDerivedStateFromProps(props, state) - вызывается перед рендером компонента для изменения состояния компонента на основе новых пропсов 
* componentWillUnmount() - вызывается перед размонтированием для устранения утечек памяти (удаление таймеров, отписка от событий, закрытие соединения с сервером)  
Аналог клинап функция useEffect
* componentDidCatch(error, info) - вызывается после возникновения ошибки в дочернем компоненте. В отличие от `getDerivedStateFromError(error)` в методе можно использовать побочные эффекты
* getSnapshotBeforeUpdate(prevProps, prevState) - вызывается перед применением изменений в DOM. Возвращаемые значения будут переданы в `componentDidUpdate`

---

[useMemo и PureComponent](https://dev.to/nibble/react-memo-and-react-purecomponent-3k7k)  
Для того чтобы дочерние компоненты не ререндерились без необходимости классовый компонент наследуется от `React.PureComponent`
Ререндер происходит, когда:
* изменяется state в родителе
* изменяется context в родителе
* изменяется props в родителе
* ререндерится родитель

PureComponent в отличие от Component автоматически использует `shouldComponentUpdate`, который проверяет props и state и не ререндерит дочерние компоненты, если не было изменений  
Сравнение пропсов и состояния поверхностное. Для глубокого сравнения стандартных механизмов нет - можно явно использовать `shouldComponentUpdate` и конкретизировать проверку ререндера    

**В функциональных компонентах** аналог - `React.memo(Component, comparator?: (prevProps, nextProps) => {})`  
Опциональная функция компаратор позволяет указать более детальную проверку ререндера. Работает также как и явное использование `shouldComponentUpdate`  
При отсутствии `React.memo` передача в setState НЕизменившегося состояния `setState(state)` **НЕ будет** приводить к ререндеру  

[Вернуться к содержанию](#содержание)

# 2. Props

Это механизм передачи данных от родительского компонента к дочернему  
```typescript
<CustomComponent attr1={value} attr2={{value: 10}} attr3="text">
	Some Text
</CustomComponent>
```
Способы передачи props: 
* между тегами (`props.children`) `<MyComponent><AnotherComponent /></MyComponent>`  
Приоритет в передаче `props.children` между тегами компонента - выше, чем при указании аналогичного атрибута `<MyComponent children={...} />`
* фигурные скобки `{}` - переменные `<MyComponent loadintState={isLoading} />`
* двойные фигурные скобки `{{}}` - объекты `<MyComponent personData={{name: 'Petr', age: 18}} />`
* кавычки `""` - текст `<MyComponent title="Some Text"/>`

>Компонент ререндерится при изменении переданных props  

Деструктурируя `...props`, можно выделить необходимые для передачи в дочерний компонент пропсы
```typescript
const ParentComponent = ({children, ...props}: ParentComponentProps) => {
	return (
		<ChildCompoent key={props.id}>{children}</ChildComponent>
		{/* установка props без присваивания отдельным арибутам */}
		<AnotherComponent {...props} />
	)
}
```

Пропсы передаются сверху вниз (от родительского компонента к дочернему). Для передачи данных в обратном направлении нужно передать **функцию обратного вызова** в дочерний компонент  
**Важно** передавать функцию `{myFunc}`, а не ее вызов `{myFunc()}` _(неправильный вариант)_
```typescript
funсtion App() {
	const myFunc = () => {...логика...}
	return (
		<MyComponent getValue={myFunc} />
	)
}
```

## Пример

Функцию обратного вызова можно реализовать в родительском компоненте, а в дочернем передавать в нее параметры  
```typescript
function App() {
	return (
		<MyComponent getValue={(param1, param2) => {...логика функции}}></MyComponent>
	)
}

const MyComponent = ({getValue}: MyComponentProps) => {
	{getValue(param1, param2)}
	...остальная логика компонента
}
```

**Частая проблема** - большой уровень вложенности. Пропсы передаются более чем на 2 уровня вложенности и не используются в промежуточных компонентах **(Props Drilling)**  
Можно реорганизовать компоненты и поднять их на уровень выше  
Необходимые пропсы можно будет использовать сразу на вложенных компонентах, без передачи через промежуточные компоненты  
```typescript
<OuterComponent>
	<InnerComponent>
		<InnerHeader/>
		<InnerContent text={text} />
	</InnerComponent>
</OuterComponent>
```

>Пропсы нельзя изменить (readonly), но можно мутировать. Мутирование нарушает принцип инкапсуляции  
>[Видео разбор статьи о модифицировании пропсов](https://www.youtube.com/watch?v=jDHBY6tV2SE)  
>```
>type User = {
>	username: string;
>}
>const Index (props: {user: User}) => {
> // не приводит к ошибке
>	props.user.username = `${props.user.username} Some Text`;
>	retrun <div>{props.user.username}</div>;
>}
>```

Плавное введение в клиент/серверные компоненты и передачу пропсов между ними  
[Виды компонентов и передача пропсов между ними](https://www.youtube.com/watch?v=F0ZvDcOuBdo)

[Вернуться к содержанию](#содержание)

# 3. Списки

При создании списков необходимо задавать атрибут `key` с уникальным значением для каждого итерируемого элемента (id, индекс в массиве map)  
Индекс итерации не рекомендуется использовать в качестве ключа, т.к. элементы могут добавляться/удаляться, что изменит их ключ.  
>`Crypto.randomUUID()` или `npm пакет uuid` создают уникальные id ключи
```typescript
function App() {
	const [array, setArray] = useState([{id: 1}, {id: 2}]) // массив элементов
	return (
		<div className="container">
			{/* Если количество элементов списка не будет изменяться, то можно указывать индекс итерации в качестве атрибута key */}
			{array.map((item, index) => <CustomComponent id={item.id} key={index}>Some Text</CustomComponent>)}
		</div>
	)
}
```
Добавление нового элемента в список:
```typescript
const [array, setArray] = useState([{id: 1}, {id: 2}])
setArray([...array, {id: 3}])
```
[Вернуться к содержанию](#содержание)

# 4. Хуки

Хуки **НЕЛЬЗЯ** использовать внутри функций, условий, циклов  
[Объяснение почему](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e "VPN")

## 4.1 useState

[детальный разбор(видео)](https://www.youtube.com/watch?v=V9i3cGD-mts)  
[еще один(видео)](https://www.youtube.com/watch?v=O6P86uwfdR0)  

`const [state, setState] = useState<TState>(defaultValue)` - управляет состоянием компонента  
При изменении состояния компонент и все дочерние компоненты ререндерятся, состояние сохраняется между ререндерами  
Используется для хранения локального состояния компонента, которое не передается в большое количество других компонентов (риск Prop Drilling)  
* state - текущее состояние
* setState - функция сеттер для изменения состояния
* defaultValue - начальное значение состояния (можно использовать функцию)  
`const [state, setState] = useState(() => setInitial())` - выполнится только при первом рендере  
`const [state, setState] = useState(setInitial())` - выполнится при **КАЖДОМ** новом рендере  
>При передаче больших объемов данных, например массива `useState([0, 1, ..., 999999])`, он будет пересоздаваться при **каждом** ререндере, что приводит к повторным вычислениям, поэтому оправдано использование функции, т.к. она вызывается лишь единожды 

Напрямую изменять (mutate) состояние нельзя, поэтому любые *мутирующие* методы (sort, reverse и т.д.) должны вызываться на копии  
[Проверка методов на мутации](https://doesitmutate.xyz/)  

Для объектов - `setState(...state, stateArg: newValue)` или для массивов - `setState([...state, newValue])`  
Спред-синтаксис действует поверхностно  
Изменение состояния вложенных полей (при большом уровне вложенности) увеличивает и дублирует код: `setState(...state, innerObj: {...state.innerObj, innerArg: newValue})`  

**Важно** не забывать, что объект - ссылочный тип:  
```typescript
const [state, setState] = useState([{innerArg: 1}, {innerArg: 2}])
function handleClick() {
	{/* ! ОШИБКА ! Происходит мутация ссылочного типа (объекта) */}
	const newValue = [...state]
	newValue[0].innerArg = 10 
	setState(newValue)
	{/* Правильный вариант: использование не мутирующего метода */}
	const newState = state.map((item) => {
		(item.innerArg === 1) ? {...item, innerArg: 10} : item
	})
	setState(newValue)
}
```

### 4.1.1 Immer

[Immer](https://github.com/immerjs/use-immer) - упрощает изменение состояния с вложенными объектами  
Immer позволяет использовать синтаксис мутаций, но под капотом вызваются стандартное изменение состояние(без мутаций)  
Immer не запрещает использовать стандартный синтаксис. Выбор оправдан, если количество кода уменьшается при изменении состояния  

**Пример с объектами:**  
```typescript
const [state, updateState] = useImmer({
	outerArg: outerValue,
	innerObj: { innerArg: innerValue }
})
updateState((draft) => draft.innerObj.innerArg = newValue)
```
**Пример с массивами:**   
```typescript
const [state, updateState] = useImmer([{innerArg: 1}, {innerArg: 2}])
updateState((draft) => {
	const item = draft.find((s) => s.innerArg === 1)
	item.innerArg = 10
})
```
### 4.1.2 Batching

**Изменение состояние происходит НЕ мгновенно**. Если функция сеттер вызвана внутри обработчика событий, то повторный рендер ожидает завершения кода внутри обработчика  
Последовательность команд внутри обработчика не важна. Изменение состояния произойдет в новом рендере  
Состояние фиксируется для текущего рендера. Асинхронные функции также используют текущее состояние  
```typescript
const handleClick = () => {
	setState(newState);
	setTimeout(() => {
	{/* в таймере будет использовано текущее значение state, а не newState */}
		...обработка state
	}, delay)
}
```
При изменении нескольких состояний в одном обработчике, React использует механизм Batching (объединение изменений состояний для ререндера)  
_С 18 версии batching распространяется на асинхронные операции_  
[Хорошее погружение в batching](https://www.youtube.com/watch?v=VfQ-qSjIalU)  
**Batching объединяет**: 
* отдельно все синхронные изменения в 1 ререндер
* отдельно все асинхронные операции в ререндер, но:
	1. объединяются все микротаски  
	2. объединяются все макротаски, запустившиеся в одно время
	3. объединяются макротаска и все запущенные в ней микротаски/синхронные события  
	Следующая за ней (другая) макротаска запустит новый рендер
	4. стоит учесть особенность поведения при указании в таймере нулевой задержки.  
	Если к моменту завершения длительной микротаски успеют завершиться два таймера с 0 и отличной от 0 задержкой,  
	то таймер с 0 задержкой вызовет собственный ререндер (**данное поведение может отличаться в разных браузерах**)
**Краткий пример:**  
[Более подробно разобранный пример](https://dzen.ru/a/ZfsEbpisFmwi6p2h)	 
```typescript
const [state1, setState1] = useState(0)
const [state2, setState2] = useState(0)
const handleClick = () => {
	{/* синхронные изменения состояния вызывают 1 ререндер */}
	setState1(5)
	setState2(5)
	{/* первые 2 таймера срабатывают в одно и то же время - происходит 1 ререндер */}
	setTimeout(() => {setState1(10)}, 100)
	setTimeout(() => {setState2(10)}, 100)
	{/* при отсутствии микротаски `asyncFunc`, таймер с задержкой 200мс срабатывает позже предыдущих - новый ререндер */}
	{/* при наличии микротаски `asyncFunc` со сложными вычислениями больше времени задержки таймеров (в данном примере 200мс), три таймера вызовут 1 ререндер */}
	setTimeout(() => {setState1(20)}, 200)
	{/* при наличии микротаски `asyncFunc` со сложными вычислениями таймер с нулевой задержкой вызвает собственный ререндер в Chrome */}
	{/* в разных браузерах может вести себя по-разному */}
	setTimeout(() => {setState1(30)}, 0)
	{/* асинхронная функция вызывает собственный ререндер */}
	asyncFunc().then(() => {}
}
```
Для многократного изменения состояния **ДО нового** рендера следует использовать **функцию апдейтер** -`setState((prev) => prev + 1)`, которая учитывает предыдущие изменения.  
Функция встает в очередь на обработку. Во время следующего рендера последовательно вызываются все изменения состояния из очереди  
**Пример**:  
В новом рендере состояние будет равно `newValue`, так как все изменения состояния были добавлены в очередь  
```typescript
const [state, setState] = useState(0)
setState(state + 1) // state = 0 + 1
setState(state + 1) // state = 0 + 1
setState(prev => prev + 1) // state = 1 + 1
setState(newValue) // state = newValue
```

---

**React сохраняет состояние компонента, если он НЕ ИЗМЕНЯЕТ позиции в DOM-дереве**  
Для сохранения состояние между ререндерами (принципы DOM Diffing):  
1. структура дерева не должна отличаться между рендерами  
_Например: одинаковый компонент сохраняет позицию второго дочернего элемента в родителе_  
2. Значение ключа `key` не должно меняться
```typescript
<div>
	<p>SomeText</p>
	{theme === 'dark' 
		? (<MyComponent theme='dark' />) 
		: (<MyComponent theme='light' />)
	}
</div>
```  
Для сброса состояние компонента можно: 
1. Изменить позицию сбрасываемого компонента
2. Изменить значение ключа `key` в сбрасываемом компоненте:
```typescript
const [state, setState] = useState({name: 'Petr', age: 18, id: 1}, {name: 'Petr', age: 21, id: 2})
return (
	<>
		<CustomComponent name={state.name} />
		{/* зависимость от уникального параметра в state заставляет ререндерить компонент как новый компонент с новыми данными */}
		<CustomInput name={state.name} key={state.id} />
	</>
)
```
_Реальный пример_: общий чат на несколько адресатов. Введенный текст должен сбрасываться при переключении вкладок пользователей  

**React не ререндерит компонент, если передать неизмененное состояние** `setState(state)`, **НО** произойдут все вычисления в теле компонента  
(При нажатии кнопки ref.current === 2 (+1 при первом рендере, еще +1 после нажатия на кнопку, но ререндера не произойдет)  
```typescript
const [state, setState] = useState(0)
const ref = useRef(0)

ref.current += 1
{/* при выполнении handleClick мы передаем то же состояние - нового рендера не происходит */}
{/* НО вычисления: current.ref += 1 произойдут */}
handleClick() {
	setState(state)
}
```

**Виды тасок в React:**
* высокоприоритетные: 
	1. useState
	2. useReducer
	3. useSyncExternalStore 
* низкоприоритетные:
	1. useTransition
	2. startTranstion
	3. useDeferredValue

Высокоприоритетные таски прерывают выполнение низкоприоритетных
Если компонент начал исполняться (рендер компонента), то его нельзя прервать, пока не дойдем до return  
В процессе прервывания низкоприоритетные таски возвращают свой предыдущий state (последний актуальный, т.е. примененный в последнем рендере)

### 4.1.3 flushSync

`flushSync(callback)` - заставляет React произвести ререндер сразу после выполнения синхронного кода `callback`
Иногда не требуется использовать batching для объединения изменений состояний, _например одно состояние зависит от другого_:
```typescript
const [age, setAge] = useState(0)
const [human, setHuman] = useState({name: 'Petr', age: 18})
handleClick() {
	flushSync(() => {
		setState(21)
	})
	{/* здесь произойдет принудительный ререндер и human получит обновленное значение age */}
	setState2({...human,  name: 'Olga'})
}
```

### 4.1.4 Ошибки

**Основные ошибки** при создании состояний: 
* Большое количество state, обновляемых одновременно - решение: объединение в один объект state
* Взаимоисключающие state - решение: можно объединить в один state
* Дублирующие state - решение: можно вычислить как переменные на основе других state
* State дублирует передаваемый props - решение: можно использовать, если не требуется обновление состояния при изменении props
* Дублирование объекта в state - решение: можно хранить только значимое поле вместо всего объекта, например id
* Большая вложенность в state - решение: можно нормализовать вложенность (сделать плоским объектом)  

[Вернуться к содержанию](#содержание)

## 4.2 useReducer

[Детальный видео-разбор](https://www.youtube.com/watch?v=rgp_iCVS8ys)  
[отличие useState от useReducer(видео)](https://www.youtube.com/watch?v=3VClygDRSsU)  
[Еще одно видео-разбор](https://www.youtube.com/watch?v=kK_Wqx3RnHk)  
`const [state, dispatch] = useReducer(reducerFunction, initialState, initFunc?)` - аналогично useState
* state - текущее состояние
* dispatch - функция, запускающая reducerFunction
* reducerFunction - функция (возвращает новое состояние), объединяющая логику изменения состояния в зависимости от условий **(чистая функция без side-эффектов)** 
* initialState - начальное значение состояния редьюсера
* initFunc - функция инициализации, вычисляющая начальное значение на основе второго аргумента initialState 
Важно передавать функцию, а не вызов функции (результат ее выполнения), тогда вычисления проводятся только при первом рендере  
Если реализовать функцию инициализации внутри компонента, то она будет вычисляться при каждом рендере (тяжелые вычисления повлияют на производительность)  
Аналогично хуку `useState` - `initFunc` запускается единожды, а `initialState` создается при каждом рендере, что может быть ресурсозатратным: `const [state, dispatch] = useReducer(reducer, [], initFunc);`

`dispatch({type: string[, payload: data]})`, где type - тип изменения, payload - передаваемые данные  
Используется для объединения логики обновления состояния в одну функцию  
||useReducer|useState|
|----|----|----|
|Тип состояния|Сложные, многосоставные типы данных(Object, Array, Map и т.д.)|Простые типы: (Nubmer, String, Boolean и т.д.)|
|Количество изменений состояния|Множественные изменения: > 2 состояний|Простые изменения (счетчик, переключатель): 1 или 2|
|Дополнительная обработка|Да. Позволяет включить дополнительную логику обработки состояния|Нет. Не поддерживает встроенную сложную обработку|  

Получить обновленное значение состояния в текущем состоянии (до рендера) можно вызвав `reducerFunction`: `const nextState = reducerFunction(state, action)` 

**Типизация useReduce**:
0. Типизация состояния: `type BookState = { price: number, title: string }`
1. Типизация action для reducer
```typescript
export type IsNever<T, True, False> = [T] extends [never] ? True : False;

{/* Тип события. Type — название, Payload — данные */}
type Action<Type extends string, Payload = never> = IsNever<
  Payload,
    // Событие без payload
  {
    type: Type;
  },
    // Событие с payload
  {
    type: Type;
    payload: Payload;
  }
>; 
```
2. Типизация reducerFunction
```typescript
type BookAction =
    | Action<'INCREMENT'>
    | Action<'DECREMENT'>
    | Action<'SET_TITLE', string>; 
``` 
3. Реализация reducerFunction
```typescript
export const bookReducer: React.Reducer<BookState, BookAction> = (value, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return {...value, price: value.price + 100} ;
    case 'DECREMENT':
      return {...value, price: value.price - 100};
    case 'SET_TITLE':
      return {...value, title: action.payload};
		default: 
			return value
  }
}; 
```
4. Инициализация useReducer: `const [state, dispatch] = useReducer(bookReducer, {price: 100, title: 'Book Title'})`
5. Вызов диспатчера в коде
```typescript
function handleClick() {
	dispatch({
		type: 'SET_TITLE',
		payload: 'Another Book Title'
	})
}
```
6. Типизация dispatch при передаче в дочерний компонент: `dispatch: Dispatch<BookAction>`

---

Immer включает в себя useImmerReducer: 
```typescript
const [state, dispatch] = useImmerReducer(reducerFunction, initialState)
function reducerFunction(draft, action) {
	switch (action.type) {
		case 'added': {
			draft.push({ id: action.id, text: action.text}) break;
	}
}
```

[Вернуться к содержанию](#содержание)

## 4.3 useContext

[детальный разбор(видео)](https://www.youtube.com/watch?v=HYKDUF8X3qI)  
[лучший разбор(видео)](https://www.youtube.com/watch?v=I7dwJxGuGYQ)  
[еще один(видео)](https://www.youtube.com/watch?v=5LrDIWkK_Bc)  

`const context = useContext<TSomeType>(initialContext)` - позволяет получить доступ к контексту всем **дочерним** компонентам, не используя props  
>Изменение значения контекста ведет к ререндеру всех дочерних провайдеру контекста компонентов, использующих контекст  

Используется: 
* при передача props более чем на 2 уровня вложенности компонентов, для исключения проблемы Props Drilling  
* для редко изменяемых данных (тема, язык), т.к. 
Для часто меняющихся данных используются библиотеки с глобальным хранилищем: Redux и т.д.    
```typescript
// Context.ts
type StateType = string
type CustomType = {
	state: StateType
	setState: React.Dispatch<React.SetStateAction<StateType>>
}
export const ThemeContext = createContext<CustomType | undefined>(undefined)

// ParentComponent.tsx
const [state, setState] = useState<StateType>("Petr")  
// value - зарезервированный атрибут для передачи актуального значения контекста
<ThemeContext.Provider value={{state, setState}}>
	<ChildComponent />
</ThemeContext.Provider>

// ChildComponent.tsx
export function ChildComponent() {
	const currentContext = useContext(ThemeContext)
	// проверка на undefined
	if(currentContext) ...выполнение действий с контекстом...
}
```

Для исключения undefined в контексте можно использовать кастомный хук:
```typescript
// Context.ts
export function useThemeContext() {
	const currentContext = useContext(ThemeContext);
	if(currentContext === undefined) throw new Error('Context is undefined')
	return currentContext
}

// ChildComponent.tsx  
export function ChildComponent() {
	const currentContext = useThemeContext()
}
```

Классовый компонент для обращения к контексту использует конструкцю:
```typescript
render() {
	return (
		<CustomContext.Consumer>
			{contextValue => {
				...код
			}}
		</CustomContext.Consumer>
	)
}
```

**Важное замечание**  
При передаче в контекст переменной ссылочного типа (объект, массив и т.д.) для исключения лишних ререндеров, ее нужно обернуть в useCallback/useMemo (см. [useMemo](#410-usememo), [useCallback](#412-usecallback))  
```typescript
// MyComponent.tsx
export function MyComponent() {
	const [value, setValue] = useState(0);
	// объект создается заново при каждом рендере компонента, что ведет за собой изменение контекста
	const contextValue = { value };
	// useMemo сохранит ссылку на ссылочный тип и не будет создавать его каждый раз с одинаковыми данными
	const contextValue = useMemo<ContextProps>(() => ({ value }), [value]);
	return (
		<ThemeContext.Provider value={contextValue}>
			<ChildComponent />
		</ThemeContext.Provider>
	)
}
```
**Второе важное замечание**  
[Правильная реализация контекста](https://www.youtube.com/watch?v=16yMmAJSGek)  
Имеются два компонента: использующий контекст `ComponentUseContext` и не использующий контекст `ComponentDontUseContext`
При данной реализации оба компонента **будут ререндериться** при изменении контекста, т.к. изменяется state:
```typescript
export const CustomContext = createContext<{state: number, setState: React.Dispatch<React.SetStateAction<number>>} | undefined>(undefined);

const App = () => {
	const [state, setState] = useState(0)
	return (
		<CustomContext.Provider value={{ state, setState }}>
			<ComponentUseContext />
			<ComponentDontUseContext />
		</CustomContext.Provider>
	)
};
```

Правильная реализация. Компонент, не использующий useContext, не будет ререндериться при изменении контекста:
```typescript
// custom-context.tsx
export const CustomContext = createContext<{state: number, setState: React.Dispatch<React.SetStateAction<number>>} | undefined>(undefined)

export default сonst CustomContextProvider = ({ children }) => {
	const [state, setState] = useState(0)
	return <CustomContext.Provider value={{ state, setState }}> {children} </CustomContext.Provider>
}

// MyComponent.tsx
const App = () => {
	return (
		<CustomContextProvider>
			<ComponentUseContext />
			<ComponentDontUseContext />
		<CustomContextProvider>
	)
}
```

**Переопределение контекста**
Дочерние компоненты обращаются к ближайшему родительскому контексту, поэтому можно переопределить провайдер для части дерева, обернув ее в провайдер с другим значением value   
```typescript
<CustomContext.Provider value={0}> 
	...
	<CustomContext.Provider value={1}> 
		<MyComponent />
	</CustomContext.Provider> 
	...
</CustomContext.Provider> 
```

При использовании множественных контекстов их выносят в отдельный компонент обертку  
```typescript
<Component1.Provider value={}>
	<Component2.Provider value={}>
		<Component3.Provider value={}>
			<MyComponent />
		</Component3.Provider>
	</Component2.Provider>
</Component1.Provider >
```

[Вернуться к содержанию](#содержание)

## 4.4 useRef

[детальный разбор(видео)](https://www.youtube.com/watch?v=42BkpGe8oxg)  
[еще один(видео)](https://www.youtube.com/watch?v=t2ypzz6gJm0)  
`const ref = useRef<TSomeType>(initialValue)` - хранение ссылки на данные или DOM-элемент между рендерами компонента, изменения которых не должы приводить к перерисовке компонента

`useRef` возвращает объект с полем `current`, в котором можно хранить данные между ререндером, не используя state  
```typescript
const [state, setState] = useState(0)
const ref = useRef(0)
const handleCLick = () => {
	setState(prev => prev + 1)
	ref.current += 1
	// 0, значение текущего snapshot. Оно изменится после завершения кода обработчика и ререндера
	console.log(state)
	// 1, значение изменяется сразу, так как не зависит от рендера
	console.log(ref.current)
}
```

Начальное значение ref устанавливается один раз при первом рендере, но при передаче функции в начальное значение, она вычисляется **при каждом рендере**  
`const ref = useRef(MyFunc())`  
Для избежания повторных вычислений нужно добавить условие:
```typescript
const ref = useRef(null)
if(ref.current === null) ref.current = MyFunc()
```

В ref можно передать любой тип данных, DOM-элемент или функцию-колбек, которая запустится после рендера компонента  
>Вызов колбек рефов происходит до срабатывания `useEffect/useLayoutEffect`  

При использовании колбек-ref при ререндере происходит размонтирование ref и передача нового колбека, т.к. функция - ссылочный тип  
_Проблема_: повторный вызов колбека  
_Решение проблемы_: мемоизация колбека `const callbackRef = useCallback((element) => {console.log(element)})`  
[Подробно про проблемы рефов и решения через колбек рефы](https://www.youtube.com/watch?v=MLWsLn_jeGc)  
[useForkRef - кастомный хук](https://ollylut.medium.com/what-is-useforkref-hook-4be1c85d2d1b "VPN")  

```typescript
function App() {
	const myRef = useRef<HTMLInputElement>(null);
	// Проблема: принудительное обновление покажет в консоли null и <input/> 
	const [, forceUpdate] = useReducer((v) => v + 1, 0)
	function getValue() {
		return myRef.current?.value
	}
	// использование мемоизированного колбека решит проблему повторного вызова колбека
	const memoCallback = useCallback((element: HTMLInputElement) => {console.log(element)}, [])
	return (
		<>
			{/* обычный реф */}
			<input ref={myRef} />
			<button onClick={getValue}>getValue</button>
			{/* колбэк-реф */}
			<input ref={(element) => {console.log(element)}} />
			<button onClick={forceUpdate}>forceUpdate</button>
		</>
	)
}
```

При передаче ref в **функциональный** компонент, нужно обернуть этот компонент в `forwardRef`  
>React 19 не требуется оборачивать компоненты в `forwardRef`  

В *классовом компоненте* не требуется использовать `forwardRef`  
```typescript
function App() {
	const myRef = useRef<HTMLInputElement | null>(null);
	function getValue() {
		return myRef.current?.value
	}
	return (
		<MyComponent ref={myRef} />
	)
}
// MyComponent
type MyComponentProps = {
	props?: {}
  ref: React.RefObject<HTMLInputElement>;
}
const MyComponent = forwardRef(({props, ref}: MyComponentProps) => { 
	return <input ref={ref} />
})
```
 
В классовых компонентов используется `createRef` внутри конструктора - `this.myRef = React.createRef()`:  
```typescript
class MyComponent extends React.Component {
	myRef: RefObject<HTMLInputElement>;
	constructor(props: {}) {
		super(props);
		this.myRef = React.createRef();
	}
	 render() {
		return (
			<input ref={this.myRef}/>
		)
	}
}
```

**Пример передачи ref из функционального компонента в классовый**
```typescript
//Functional Component
export const MyComponent = () => {
	// внутри функционального компонента создаем ref как в классовом компоненте
	const buttonRef = createRef<Button>();
	useEffect(() => {
		// type guard и использование кастомного метода компонента через ref 
		buttonRef.current && buttonRef.current.setFocus()
	})
	return (
		// связывание классового компонента с ref
		<Button ref={buttonRef}} />
	)
}
// Class Component
interface ButtonProps {}
export class Button extends React.Component<ButtonProps, {}> {
	private buttonRef: React.RefObject<HTMLButtonElement>;
	constructor(props: ButtonProps) {
		super(props)
		// создание ref для связывания с HTMLElement
		this.buttonRef = React.createRef<HTMLButtonElement>()
	}
	setFocus() {
		this.buttonRef.current && this.buttonRef.current.focus();
	}
	render() {
		return (
			<button ref={this.buttonRef}>Текст</button>
		)
	}
}
```

[Вернуться к содержанию](#содержание)

## 4.5 useImperativeHandle

[Детальный разбор(видео)](https://www.youtube.com/watch?v=ndVIEMasBl8)  
[Еще один(видео)](https://www.youtube.com/watch?v=zpEyAOkytkU)  

`useImperativeHandle(ref, createHandle, deps?)` - используется для предоставление функционала дочернего компонента родителю без поднятия состояния  
* ref - кастомизируемый переданный ref
* createHandle - добавляемый функционал
* deps - зависимости в формате [dep1, dep2, dep3], изменение которых вызывает обновление функионала

**Не обязательно привязывать переданный ref, к HTMLElement'у дочернего компонента**.  
Ref служит для передачи функционала между Parent-Child  
```typescript
export type MyRef = {
	reset: () => void,
	focus: () => void
}

const ParentComponent = () => {
	const ref = useRef<MyRef>(null)
	return (
		<>
			<ChildComponent ref={ref} />
			{/* Сбрасываем состояние дочернего компонента из родительского без поднятия состояния */}
			<button onClick={() => ref.current?.reset()}>Reset</button>
		</>
	)
}

type ChildProps = {
	props: {}
  ref: Ref<MyRef>
}
const ChildComponent = forwardRef(({props, ref}:  ChildProps) => {
	const [state, setState] = useState({name: 'Petr'})
	// Для привязки переданного ref к HTMLElementу необходимо создать новый локальный ref, так как в кастомизируемом нет методов HTMLElement
	const localRef = useRef<HTMLInputElement>(null);
	const reset = () => {
		setState({name: ''});
	}
	// ограничиваем родительский компонент двумя доступными методами
	useImperativeHandle(ref, () => ({
		reset,
		// Добавляем свойства InputElement к кастомизируемому ref 
		focus() {
			localRef.current?.focus();
		}	
	}))
	return <input ref={localRef} />
})
```

[Вернуться к содержанию](#содержание)

## 4.6 useEffect

[В этот раз недостаточно подробно(видео)](https://www.youtube.com/watch?v=-4XpG5_Lj_o)  
[Еще одно(видео)](https://www.youtube.com/watch?v=0ZJgIjIuY7U)  

`useEffect(callback, deps?)` - выполняет побочные эффекты при монтировании, размонтироваии и изменении состояния компонента
* callback - функция (побочный эффект), выполняющаяся после рендера компонента
* deps - зависимости в формате [dep1, dep2, dep3], если не переданы аргументы, то функция будет выполняться при каждом рендере,   
если передан пустой массив, то только при первом рендере,  
если в аргументах передать `[prop, state]` prop и/или state, функция выполнится при изменении prop и/или state

>`useEffect` срабатывает **ПОСЛЕ** рендера компонента   
	
`useEffect` не стоит применять для обработки событий пользователя (нужно вынести в обработчик), для преобразования данных для рендеринга (вынести на верхний уровень компонента)  
Также не стоит создавать цепочки из useEffect изменяющие части состояния на основе других состояний - это приводит к лишним ререндерам  

`useEffect` не следит за изменениями **внутри элементов массива или объекта**. Ререндер произойдет при передаче _нового_ массива или объекта  
Массив/объект/функция - ссылочный тип данных, если передать их как зависимости, то useEffect будет вызваться каждый раз при ререндере.  
Проблема решается передачей элементов массива/объекта/функции (примитивных типов)  
useEffect может возвращать функцию очистки, которая выполнится при размонтировании элемента или перед изменением его зависимостей  
```typescript
useEffect(() => {
	... выполняется после рендера
	return () => {
		...выполняется при размонтировании или перед новым рендером
	}
}, [])
```

`Strict mode` - дважды запускает рендер, что позволяет обнаружить ошибку при повторном запуске useEffect, если для него не определена функция очистки  
Функция очистки должна быть вызвана для закрытия соединения с сервером, остановки таймеров, отписки от событий, установки анимаций в начальную фазу и т.д.  
Например:  
```typescript
const [human, setHuman] = useState('Petr')
const [age, setAge] = useState(18)
useEffect(() => {
	let ignore = false
	fetchHuman(human).then((result) => {
		if(!ignore) setAge(result)
	})
	{/* Каждый вызова fetch имеет собственную переменную ignore, которая блокирует запись состояния age, если изменилось состояние human */}
	return () => {ignore = true}
}, [human])
```

[Перейти к useLayoytEffect](#47-uselayouteffect)
[Вернуться к содержанию](#содержание)

### 4.6.1 Случаи неправильного использования useEffect

`useEffect` не стоит использовать в случаях:  

1. Обновление состояние на основе пропсов или другого состояния  

```typescript
const [firstName, setFirstName] = useState('Petr');
const [lastName, setLastName] = useState('Tchaikovsky');

// 🔴 Лишние state и effect
const [fullName, setFullName] = useState('');
useEffect(() => {
    setFullName(firstName + ' ' + lastName);
}, [firstName, lastName]);

// ✅ константа также перессчитывается при изменении состояний firstName и lastName
const fullName = firstName + ' ' + lastName;
```

2. Кэширование ресурсоемких процессов  
```typescript
const TasksList = ({taksk, filter}) => {
	const [newTask, setNewTask] = useState('');
	
	// 🔴 Лишние state и effect
	const [filteredTasks, setFilteredTasks] = useState([]);
	useEffect(() => {
		setVisibleTodos(getFilteredTasks(tasks, filter));
	}, [tasks, filter]);
	
	// ✅ Перенос вычислений в процесс рендера
	const filteredTasks = useMemo(() => { return getFilteredTasks(tasks, filter) }, [tasks, filter]);
	//...
}
```

3. Сброс состояния при изменении пропсов  
```typescript
const ProfilePage = ({ userId }) => {
  const [comment, setComment] = useState('');

  // 🔴 Лишний effect
  useEffect(() => {
    setComment('');
  }, [userId]);
	
}

// ✅ Можно разнести компоненты и для каждого id рендерить свой компонент
const ProfilePage = ({ userId }) => <Profile userId={userId} key={userId} />
const Profile = ({ userId }) => {
	const [comment, setComment] = useState('');
	//...
}
```

4. Изменение состояния при изменении пропсов  
```typescript
const List= ({ items }) => {
	// 🔴 useEffect срабатывает после рендера, все дочерние компоненты получат старые значения
	const [selection, setSelection] = useState(null);
	useEffect(() => {
    setSelection(null);
  }, [items]);
	
	// ✅ Можно сохранить предыдущее состояние, но это вызывает лишний ререндер
	const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
	
	// ✅ Отсутствие лишних ререндеров, вычисления происходят во время рендера
	const [selectedId, setSelectedId] = useState(null);
	const selection = items.find(item => item.id === selectedId) ?? null;
}	
```

5. Общий код для обработчиков событий  
```typescript
// 🔴 Лишний effect
useEffect(() => {
	if (product.isDropped) {
		showNotification();
	}
}, [product]);

function handleBuyClick() {
	addToCart(product);
}

function handleCheckoutClick() {
	addToCart(product);
	navigateTo('/checkout');
}

// ✅ Можно вынести логику в общую функцию и переиспользовать ее в хэндлерах
function buyProduct() {
	addToCart(product);
	if (product.isDropped) showNotification();
}

function handleBuyClick() {
	buyProduct();
}

function handleCheckoutClick() {
	buyProduct();
	navigateTo('/checkout');
}
```

6. POST-запросы  
```typescript
// 🔴 Лишний ререндер при пользовательском взаимодействии
const [jsonToSubmit, setJsonToSubmit] = useState(null);
useEffect(() => {
	if (jsonToSubmit !== null) {
		post('/api/register', jsonToSubmit);
	}
}, [jsonToSubmit]);
function handleSubmit(e) {
	e.preventDefault();
	setJsonToSubmit({ firstName, lastName });
}

// ✅ Можно вынести POST-запрос в event handler
function handleSubmit(e) {
	e.preventDefault();
	post('/api/register', { firstName, lastName });
}
```

7. Цепочки useEffect  
```typescript
const [card, setCard] = useState(null);
const [round, setRound] = useState(1);

// 🔴 Цепочки эффектов триггерят несколько ререндеров подряд
const [isGameOver, setIsGameOver] = useState(false);
useEffect(() => {
	if (card !== null) {
		setRound(prev => prev + 1)
	}
}, [card]);
useEffect(() => {
	if (round > 5) {
		setIsGameOver(true);
	}
}, [round]);

useEffect(() => {
	alert('Good game!');
}, [isGameOver]);

function handlePlaceCard(nextCard) {
	if (isGameOver) {
		throw Error('Game already ended.');
	} else {
		setCard(nextCard);
	}
}


// ✅ Нужно перенести логику вычисления в процесс рендера или внутрь обработчика
const isGameOver = round > 5;

function handlePlaceCard(nextCard) {
	if (isGameOver) {
		throw Error('Game already ended.');
	} else {
		setCard(nextCard);
		setRound(round + 1);
		if (round === 5) {
			alert('Good game!');
		}
	}
}
```

8. Инициализация приложения  
```typescript
// 🔴 Нужно избегать effecta с логикой, которая должна выполняться только 1 раз
useEffect(() => {
	loadDataFromLocalStorage();
	checkAuthToken();
}, []);

// ✅ Для большей устойчивости к повторному маунту можно завести флаг начальной загрузки
let didInit = false;

function App() {
	useEffect(() => {
		if (!didInit) {
			didInit = true;
			
			loadDataFromLocalStorage();
			checkAuthToken();		
		}
	}, []);
}

// ✅ Или вынести логику вне компонента
if (typeof window !== 'undefined') {
  checkAuthToken();
  loadDataFromLocalStorage();
}
function App() {
  // ...
}
```

9. Уведомление родительского компонента об изменении состояния  
```typescript
const Toggle = ({ onChange }) => {
	const [isOn, setIsOn] = useState(false);

	// 🔴 Хэндлер запускается слишком поздно
	useEffect(() => {
			onChange(isOn);
	}, [isOn, onChange])

	function handleClick() {
		setIsOn(!isOn);
	}

	// ✅ Вынести обновление стейтов в обработчик события
	function updateToggle(nextIsOn) {
		setIsOn(nextIsOn);
		onChange(nextIsOn);
	}
	function handleClick() {
		updateToggle(!isOn);
	}

	// ✅ Сделать управляемый компонент, убрав внутреннее состояние (в примере: isOn) в родительский компонент и передавать его как пропс
	function handleClick() {
    onChange(!isOn);
  }

  //...
}
```

10. Передача данных родителю  
```typescript

// 🔴 Нарушение принципа однонаправленного потока данных
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useAPI();
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);
  // ...
}

// ✅ Данные должны передаваться сверху вниз от родителя к дочерним компонентам
function Parent() {
  const data = useAPI();
  // ...
  return <Child data={data} />;
}

function Child({ data }) {
  // ...
}
```

11. Подписка на внешний store  
```typescript
// 🔴 Требуется ручное управление синхронизацией с состоянием
const [isOnline, setIsOnline] = useState(true);
useEffect(() => {
	function updateState() {
		setIsOnline(navigator.onLine);
	}

	updateState();

	window.addEventListener('online', updateState);
	window.addEventListener('offline', updateState);
	return () => {
		window.removeEventListener('online', updateState);
		window.removeEventListener('offline', updateState);
	};
}, []);

// ✅ Использование хука useSyncExternalStore
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
	 return useSyncExternalStore(
		subscribe,
		() => navigator.onLine,
		() => true
	) 
}
```

12. Получение данных
Один из случаев оправданного применения `useEffect`.
```typescript
const [results, setResults] = useState([]);
const [page, setPage] = useState(1);
// 🔴 Проблема race condition
useEffect(() => {
	// query может изменяться при прямом переходе с заданным query, браузерной навигации, нажатии на предусмотренные в коде кнопки 
	fetchResults(query, page).then(json => {
		setResults(json);
	});
}, [query, page]);
	
// ✅ Отмена устаревших данных в cleanup
useEffect(() => {
	let ignore = false;
	fetchResults(query, page).then(json => {
		if (!ignore) {
			setResults(json);
		}
	});
	return () => {
		ignore = true;
	};
}, [query, page]);
```

[Статья из документации](https://react.dev/learn/you-might-not-need-an-effect)   

[Вернуться к содержанию](#содержание)

## 4.7 useLayoutEffect

[Подробный разбор(видео)](https://www.youtube.com/watch?v=9GAt97z8Jc4)  
[Еще один(видео)](https://www.youtube.com/watch?v=wU57kvYOxT4)  

`useLayoutEffect(callback, deps?)` - аналогичен `useEffect`, НО срабатывает после всех изменений DOM и до того как компонент будет отрисован
* callback - функция, выполняющая до рендера компонента
* deps - зависимости в формате [dep1, dep2, dep3], изменение которых ведет к вызову callback  
Блокирует отрисовку элементов, поэтому не рекомендуется использовать длительные операции  
Используется для изменения расположения или внешнего вида элемента до отрисовки  
```typescript
useLayoutEffect(() => {
	const {height, width} = ref.current.getBoundingClientRect()
	setSize({height, width})
}, [])
```

`useLayoutEffect` не работает при SSR, так как при серверном рендере не существует макета  
Решение: 
* использовать `useEffect`
* использовать `Suspense` и показывать фолбэк, не использующий `useLayoutEffect`
* проверять состояние `isMounted` и отображать фолбек, пока не прозойет гидратация  
>Гидратация - добавление недостающих элементов в код, который был сгенерирован в процессе SSR  

[Вернуться к содержанию](#содержание)
	
## 4.8 useInsertionEffect 

Срабатывает до монтирования элементов. Последовательность: `useInsertionEffect > useLayoutEffect > useEffect`  
Используется при работе CSS-in-JS  
[Подробнее в документации](https://react.dev/reference/react/useInsertionEffect)  
[Нестандартное применение (малоприменимо)](https://www.youtube.com/watch?v=ILg1zhl92AI&pp=ygUSdXNlSW5zZXJ0aW9uRWZmZWN0)  

[Вернуться к содержанию](#содержание)

## 4.9 useMemo

[Подробный разбор(видео)](https://www.youtube.com/watch?v=vpE9I_eqHdM)  
[Разбор документации по шагам с примерами(видео)](https://www.youtube.com/watch?v=oMvW3A_IRsY)  

`useMemo(calculateValue, deps)` - локально (только в компоненте, вызвавшем хук) мемоизирует (кэширует) **результат** вычислений.  
Повторные вычисления производятся при изменении одной из зависимостей хука  
* calculateValue - чистая функция, которая должна возвращать результат вычислений
* deps - зависимости в формате [dep1, dep2, dep3], изменение которых ведет к новому вычислению calculateValue   

`useMemo` стоит применять, когда вычисления занимают длительное время. 
Также при передаче объекта/массива как props их нужно мемоизировать (т.к. `{} !== {}`). Ссылка на них сохранится - не будет лишних ререндеров
Мемоизация объекта: `useMemo(() => {name: 'Petr'})`  
>При мемоизации функции сохраняется **результат** ее вычислений  

Проверить длительность вычислений можно включив CPU Throttling в DevTools или аналог:  
```typescript
console.time('start');
... вычисления
console.timeEnd('end');
```
>**Не нужно мемоизировать все подряд.** Это трата памяти на сохранение всех результатов мемоизации  

[Вернуться к содержанию](#содержание)

##№ 4.9.1 React.memo

`React.memo(component[, fn])` - мемоизирует компонент. _Не является хуком_
* component - компонент для мемоизации  
* fn - функция сравнения пропсов, передаваемых в component, для принятия решения о ререндере  
Применяется, когда дочерние компоненты не изменяются при изменении состояния родителя (дочерний компонент не будет повторно отрендерен)  
```typescript
import {memo} from "react";

const MyComponent = memo(({ data }: TSomeProps) => (
	<div>{data}</div>
)); 
```

Аналогом `memo` с одним аргументом является PureComponent в классовом компоненте - `class MyComponent extends PureComponent {}`  
Альтернативно в классовом компоненте можно кастомизировать `shouldComponentUpdate`, который автоматически вызывается в PureComponent (см [Классовый компонент](#13-классовый-компонент-устаревший))  
`shouldComponentUpdate` должен вернуть boolean, который определит нужен ли ререндер компоненту
```typescript
type Props = {propsData: string};
type State = {stateData: string};
class Header extends Component<Props, State> {
	constructor(props: Props) {
        super(props);
        this.state = { stateData: 'Text' };
    }
	shouldComponentUpdate(nextProps: Readonly<Props>, nextState: Readonly<State>) {
			// Перерендер произойдет только, если изменится data
			return this.state.stateData !== nextState.stateData;
    }
}
```
[Вернуться к содержанию](#содержание)

## 4.10 useCallback

[Качественный разбор(видео)](https://www.youtube.com/watch?v=MxIPQZ64x0I)  
[Еще один(видео)](https://www.youtube.com/watch?v=_AyFP5s69N4)  

`useCallback(fn, deps)` - мемоизирует ссылку на функцию, между ререндерами, пока не изменились зависимости  
* fn - значение функции для мемоизации. 
>Возвращает функцию, НЕ вызвает ее.  
* deps - зависимости в формате [dep1, dep2, dep3]  

>Используется в связке с `memo`  
При передаче в `memo` функций НЕ обернутых в `useCallback` мемоизация компонента не имеет смысла, т.к. функия - ссылочный тип  

```typescript
const handleClick = useCallback(
	(data) => {
		post(url, {
			data,
			changedData
		})
	},
	[url, changedData]
)
```

>В отличие от `useMemo` - `useCallback` мемоизирует саму функцию, а не результат ее вычислений. 

```typescript
const dataMemo = useMemo(() => {
	return computeFunc(data)
}, [data])

const dataCallback = useCallback(
	(params) => {
		post(url) {}
	}, [url]
)
```

**Важно помнить**, что без использования зависимостей функция запоминает свое состояние:
```typescript
function MyComponent() {
	const [dataArray, setDataArray] = useState([13,4,11,2,20])
	const printFirstElement = useCallback(() => {
		// зависимости в useCallback не указаны, поэтому функция запомнила свое состояние  
		// функция замкнулась на переменной dataArray
		// консоль ВСЕГДА будет показывать 0 элемент (13), даже если массив будет отфильтрован
		console.log(dataArray[0])
	}, [])
	const sortArray = useCallback(() => {
		setDataArray(prevArray => [...prevArray].sort((a, b) => a - b));
	}, []);
	return (
    <div>
      <button onClick={sortArray}>Sort Array</button>
      <button onClick={printFirstElement}>Print First Element</button>
			<button onClick={() => setDataArray([21, 5, 30])}>Update Data</button>
    </div>
  );
}
```

Но самих зависимостей в useCallback должно быть как можно меньше.  
1. Функция-апдейтер помогает избавиться от лишних зависимостей:
```typescript
// без функции-апдейтера
const handleClick = useCallback(
	() => {
		const newState = {age: 18}
		setState([...state, newState])
	}, [state]
)
// с функцией-апдейтером
const handleClick = useCallback(
	() => {
		const newState = {age: 18}
		setState((state) => [...state, newState])
	}, []
)
```
2. Создание функции можно перенести в useEffect для уменьшения зависимостей:
```typescript
const myFunc = useCallback(() => {}, [deps])
useEffect(() => {myFunc()}, [myFunc])
```
Устраним лишние зависимости, добавив функцию в useEffect:
```typescript
useEffect(() => {
	const myFunc = () => {}
	myFunc()
}, [myFunc])
```

>Функции в кастомных хуках должны оборачиваться в useCallback, для дальнейшей оптимизации при их использовании  

[Вернуться к содержанию](#содержание)

## 4.11 useDeferredValue

[хороший разбор(видео)](https://www.youtube.com/watch?v=yIpHTYo3PY0)  
[еще один(видео)](https://www.youtube.com/watch?v=jCGMedd6IWA)  

`const deferredValue = useDeferredValue(value)` - позволяет отложить обновление части интерфейса (замена классическим способам оптимизации JS: дебаунс и троттлинг) без блокировки основного потока рендеринга.  
Возвращает отложенное значение, которое обновляется асинхронно в фоновом режиме  
* value - значение, которое требуется отложить (примитивный тип или объект, созданный вне рендера)  
При обновлении `value`, React рендерит компонент со старым значением, затем начинается фоновом рендеринг с новым значением.  
Любое изменение `value` прерывает фоновый рендеринг. Фоновый рендеринг начнется заново после того как `value` перестанет изменяться  

При отложенном обновлении пользователь видит предыдущие результаты, пока не будут готовы новые  
Например, тяжелые вычисления могут блокировать взаимодействие с остальным интерфейсом. Решение: они помещаются в useDeferredValue для ленивой загрузки  
Для этого `useDeferredValue` комбинируется с `memo`
```typescript
// ParentComponent.tsx
const ParentComponent = () => {
	const [text, setText] = useState('');
	const deferredText = useDeferredValue(text);

	return (
		{/* использование useDeferredValue не блокирует отрисовку и ввод текста в input */}
		<input value={text} onChange={(e) => setText(e.target.value)}/>
		<SlowComponent text={deferredText} />
	)
}

// ChildComponent.tsx
type TChildProps = {
  text: string;
};
const ChildComponent = memo(({text}: TChildProps) => {
	...долгие вычисления

	return <p>{text}</p>;
})
```
Для улучшения наглядности интерфейса, устаревшие значения можно выделить стилями `opacity: query !== deferredQuery ? 0.5 : 1,`

[Вернуться к содержанию](#содержание)

## 4.12 useTransition

[Детальный разбор(видео)](https://www.youtube.com/watch?v=1xjSQJWejZM)  
[Еще один(видео)](https://www.youtube.com/watch?v=N5R6NL3UE7I)  
`const [isPending, startTranstion] = useTransition()` - позволяет обновить состояние без блокировки интерфейса.   
Помечает обновления состояния, как переходы, что дает возможность откладывать низкоприоритетные обновления  
* isPending - boolean флаг, сообщающий статус перехода
* startTransition - синхронная функция, помечающая обновление состояний(state), как переход  
Пример использования: навигации по страницам  
>В отличие от хука `useDefferedValue`, который оборачивает состояние, `useTransition` оборачивает сеттер состояния  

Обновления состояний помеченные, как переходы, _могут быть прерваны обновлениями обычных состояний_  
>**Важно!** Функция передаваемая в `startTranstition` должна быть синхронной.  
```typescript
const MyComponent = () => {
	const [state, setState] = useState(null)
	const [text, setText] = useState('')
	const [isPending, startTransition] = useTransition();

	function onClick(nextState) {
		startTransition(() => {
				// вычисления
				const nextState = computeNextState();
				{/* состояние state помечено как переход */}
				setState(nextState);
		});
	}
	{/* ввод текста в input может прервать рендеринг по переходу */}
	function onChange(e: ChangeEvent<HTMLInputElement>) => {
		setText(e.target.value)
	}
	return (
		<>
			<button onClick={onClick}>Click</button>
			<input value={text} onChange={onChange}/>
		</>
	)
}
```

>Функция переданная в `startTransition` выполняется сразу же (не отложенно)  
Только изменение состояния помечается как переход  

```typescript
console.log(1);
startTransition(() => {
  console.log(2);
  setPage('/about');
});
console.log(3);
// console output: 1 2 3
```

>Обновления переходов **нельзя использовать для управления выводом текста**. Альтернатива useDeferredValue.  

>Обработка ошибок через ErrorBoundary доступна **только** в экспериментальном режиме  

[Вернуться к содержанию](#содержание)

## 4.13 useSyncExternalStore

[Отличные примеры применения](https://www.youtube.com/watch?v=Y34aQue4DIg)  

`useSyncExternalStore(onStoreChange, getSnapshot, getServerSnapshot?)` - подписка на внешнее хранилище (API браузера, сторонние библиотеки за пределами React).  
При изменении данных во внешнем хранилище происходит ререндер
* onStoreChange(callback) - функция подписки, которая должна возвращать функцию отписки, где callback - вызывается при изменении хранилища
* getSnapshot - функция возвращающая состояние внешнего хранилища (для клиента)
* getServerSnapshot - функция возвращающая состояние внешнего хранилища (для сервера SSR)  
`getServerSnapshot` должен возвращать те же данные, что и при первоначальном рендере на клиенте  

```typescript
const useMatchMedia = (query: string) => {
  const getSnapShot = () => window.matchMedia(query).matches;
  const subscribe = (listener: () => void) => {
    const mediaQueryList = window.matchMedia(query);
    mediaQueryList.addEventListener('change', listener);
    return () => mediaQueryList.removeEventListener('change', listener);
  }
  return useSyncExternalStore(subscribe, getSnapShot);
}

const UseSyncExample = () => {
	const isDesktopScreen = useMatchMedia('(min-width: 1920px)')
	return <p>{isDesktopScreen ? : 'Desctop' : 'Mobile'}</p>
}
```

`useSyncExternalStore` обеспечивает синхронное обновление состояния при подписке на внешние хранилища.  
В сравнении с `useEffect`, который может вводить задержки из-за асинхронности эффектов, `useSyncExternalStore` гарантирует минимальную задержку и предсказуемость данных при синхронизации  

[Вернуться к содержанию](#содержание)

## 4.14 useId

[Разбор темы(видео)](https://www.youtube.com/watch?v=_vwCKV7f_eA)  
[Более подробный разбор(видео)](https://www.youtube.com/watch?v=GNVI9Pr_RKQ&t=777s)  

`useId()` - генерирует уникальные идентификаторы **(не используется для ключей списков)**  
`:R1:` - пример сгенерированного id. Его **невозможно** использовать в функциях поиска по DOM (querySelector) 
Особенности применения:  
* гарантирует одинаковые id при генерации на сервере и клиенте (в отличие от сторонних библиотек (uuid)), если рендер на сервере и клиенте не имеет отличий   
* подходит для создания уникальных id для связывания форм, label, aria- атрибутов (особенно при заранее неизвестном количестве форм)  
* уникальность id сохраняется в пределах одного React App  

[Вернуться к содержанию](#содержание)

## 4.15 useOptimistic

[Пример с обработкой ошибки(видео)](https://www.youtube.com/watch?v=PPOw-sDeoNw)  
[Пример без обработки ошибки(видео)](https://www.youtube.com/watch?v=M3mGY0pgFk0)  

`const [optimisticState, addOptimistic] = useOptimistic(state, updateFn)` - показывает другое (оптимистичное) состояние интерфейса во время асинхронного действия  
* optimisticState - результирующее оптимистическое состояние (равно state, если действий не ожидается)
* addOptimistic(optimisticValue) - функция вызывающая `updateFn(state, optimisticValue)`
	* state - состояние, которое возвращается изначально и когда никаких действий не ожидается
	* updateFn(currentState, optimisticValue) - чистая функция (аргументы: текущее и оптимистичное состояние), которая возвращает результат вычислений как оптимистическое состояние
```typescript
import {experimental_useOptimistic as useOptimistic} from 'react'
function MyComponent({ messages, sendMessage }: TMyComponentProps) {
	const formRef = useRef();
	const [previousMessages, setPreviousMessages] = useState(messages);
	async function formAction(formData) {
		// сохраняем текущее значение перед оптимистичным обновлением 
		setPreviousMessages(optimisticMessages);
		// оптимистичное значение, передающееся в updateFn
		addOptimisticMessage(formData.get('message'));
		formRef.current.reset();
		// нужно предусмотреть отмену оптимистичных изменений
		try { await sendMessage(formData); }
		catch (error) {
			// пример реализации отката к предыдущему состоянию
			addOptimisticMessage(previousMessages);
		}
	}
	const [optimisticMessages, addOptimisticMessage] = useOptimistic(messages, (state, newMessage) => [
		...state,
		{
			text: newMessage,
			sending: true,
		},
	]

	return (
		<>
			{optimisticMessages.map((message, index) => (
				<div key={index}> {message.text}
					{!!message.sending && ( <small> Sending... </small> )}
				</div>
			))}
			<form action={formAction} ref={formRef}>
				<input type="text" name="message" />
				<button type="submit">Send</button>
			</form>
		</>
	)
}
```	

[Вернуться к содержанию](#содержание)

## 4.16 useDebugValue

[хороший разбор(видео)](https://www.youtube.com/watch?v=pTF86K8JZBQ)  

`useDebugValue(value, format?)` - добавляет метку к пользовательскому хуку в DevTools (т.к. все хуки не имеют маркировки и расположены последовательно в React DevTools)  
* value - значение отображаемое в devtools (любой тип: массив/объект/строка и т.д.)
* format - функция форматирования с аргументом value
`useDebugValue(date, (date) => date.toDateString())`
`useDebugValue` находится в теле компонента и вызывается при каждом рендере. Если `value` - отладочная функция с тяжелыми вычислениями, то ее можно передать в функцию форматирования `format`  
Она будет запускаться только при открытом React DevTools  

[Вернуться к содержанию](#содержание)

## 4.17 useActionState

[Самая актуальная инфа в доке](https://react.dev/reference/react/useActionState)  

`const [state, formAction, isPending] = useActionState(fn, initialState, permalink?)` - хук, который позволяет обновлять состояние на основе результата действия формы, где  
* `fn(previousState, formData)` - асинхронная функция вызываемая при отправке формы или нажатии кнопки  
	* `previousState` - предыдущее состояние формы (изначально `initialState`)
	* `formData` - аргументы формы
* `initialState` - начальное состояние (должно быть сериализуемо)
* `permalink` - уникальный URL страницы, испольуемый формой, для редиректа на другую страницу после отправки формы  
При отправке формы (до загрузки JS бандла) произойдет редирект на `permalink`
* `state` - текущее состояние (во время первого рендера соответствует `initialState`)
* `formAction` - `action` передаваемый в компоненты формы или пропс `formAction` любой кнопки внутри формы, также может быть вызвано вручную внутри `startTransition`
* `isPending` - флаг состояния Transition (перехода)

```typescript
import { action  } from "./actions.js";
const Index = () => {
	const [state, formAction, isPending] = useActionState(action, null)
  return (
    <form action={formAction}>
			<button type="submit">Submit</button>
			{isPending ? "Loading..." : state}
		</form>
	);
}
```

## 4.18 useFormStatus

`const { pending, data, method, action } = useFormStatus()` - хук `react-dom`, предоставляет информацию о статусе последней отправки родительской формы, где  
* `pending` - boolean. True - форма в процессе отправки
* `data` - отправляемые данные в формате FormData
* `method` - метод отправки
* `action` - функция переданная в `action`

>Хук вызывается в компоненте находящемся ВНУТРИ формы.

```typescript
export const Index = () => {
	return (
		<form action={action}>
			<Submit />
		</form>
	)
}

const Submit = () => {
  const { pending, data } = useFormStatus();
  return (
		<>
    	<button type="submit" disabled={pending}>
      	{pending ? "Submitting..." : "Submit"}
   		</button>
			{pending && data && (
        <p>Requesting data: {data.get('key')}</p>
      )}
		</>
  );
}
```

[Вернуться к содержанию](#содержание)  

## 4.19 Кастомные хуки

Кастомные хуки - начинаются с use и используют внутри базовые хуки    
Кастомные, также как и встроенные хуки не должны использоваться внутри условных конструкций, циклов и других функциях  
Кастомные должны возвращать одно значение, массив (2 значения) или объект (>2 значений)  
_Каждый вызов кастомного хука не зависит от другого вызова того же хука_  

[Неплохие кастомные хуки обертки над useState](https://www.youtube.com/watch?v=3SB278SY73s)  
Cтоит обратить внимание: 
* useSafeState - обновления состояния при fetch запросах без AbortController  
* обертка над localStorage/sessionStorage - проверка доступности localStorage/sessionStorage и преобразования данных, т.к. в них хранится только string
[Кастомные хуки для оптимизации](https://www.youtube.com/watch?v=XOSgHVzHEV4)  
* useLatest - уменьшение количества рендеров при зависимости useCallback от нескольких state  
* useEvent - аналог useLatest, только для функций  
* useWindowEvent -  уменьшение количества рендеров при подписке на события  
[Еще подробнее про useLatest](https://www.youtube.com/watch?v=ILg1zhl92AI)

[Вернуться к содержанию](#содержание)  

# 5. Пользовательские компоненты

## 5.1 Suspense  

[Объяснение преимуществ Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc)  

`<Suspense fallback={<Loader />}><MyComponent /></Suspense>` - позволяет отображать `fallback`, пока дочерние компоненты не закончат загрузку  
Для постепенного раскрытия содержимого по мере загрузки, можно использовать вложенные `Suspense`  
Если `OuterComponent` загружен, а `InnerComponent` продолжает загружаться - будет отображен `OuterComponent` и индикатор загрузки `InnerComponent`:    
```typescript
<Suspense fallback={OuterLoader}>
	<OuterComponent>
	<Suspense fallback={InnerLoader}>
		<InnerComponent>
	<Suspense>
</Suspense>
```
Для того, чтобы уже отображенные данные не заменялись на `fallback` при изменении запроса, нужно использовать `useTransiton` или `useDefferedValue`  
При завершении серверного рендеринга (SSR) компонента с ошибкой, на клиент приходит `fallback`, и повторно запускается рендеринг на клиенте   
Ошибки на клиенте обрабатываются в `ErrorBoundary`  
Для исключения компонента из рендера на сервере, в компонент **передается ошибка**:  
```typescript
<Suspense>
	{
		{/* window отсутствует в серверной среде */}
		if (typeof window === 'undefined') {
			throw Error('Client Side Rendering')
		}
	}
</Suspense>
```

>`Suspense` поддерживает источники данных: 
>	1. Загрузка данных с помощью фреймворков, поддерживающих	 Suspense (Next)
>	2. React.lazy
>	3. use (экспериментальный) для чтения значения из Promise  

>`Suspense` **НЕ** поддерживает источники данных: 
>	1. Загрузка данных в useEffect
> 2. Загрузка данных в обработчиках событий  

[Вернуться к содержанию](#содержание)  

### 5.1.1 Lazy Loading

`lazy(load)` - ленивая загрзка компонентов. Возвращает Promise, содержащий разрешенный модуль с компонентом экспорта по умолчанию  
React кэширует Promise (возвращаемое значение `load()`) и разрешенное значение промиса. Успешно разрешенное значение рендерится, ошибочное передается ближайшему `Error Boundary`.  
Динамический импорт компонентов объявляется на уровне модуля **ВНЕ** других компонентов, т.к. при ререндере не будет происходить повторный вызов `lazy` и лениво загруженный компонент не сбросит свое состояние:  

```typescript
// Объявление на верхнем уровне модуля
const Modal = lazy(() => import("./Modal.tsx"))

const MyComponent = () => {
	const [isModalDisplayed, setModalDisplayed] = useState<boolean>(false);
	return (
		<button onClick={() => { setModalDisplayed(true) }}>Загрузить модальное окно</button>
		{isModalDisplayed && (
			<Suspense fallback="Loading...">
				<Modal />
			</Suspense>
		)}
	)
}
```

[Вернуться к содержанию](#содержание)  

## 5.2 Error Boundary  

Механизм Error Boundary перехватывает ошибки в конструкторах дочерних компоненов, методах жизненного цикла и во время рендеринга: `<ErrorBoundary><MyComponent/></ErrorBoundary>`  
> Ошибки не будут пойманы в обработчиках событий, асинхронном коде, SSR, в самом компоненте Error Boundary  
Для отлова ошибок в обработчиках событий и асинхронном коде используется `try...catch`  
 
* Метод `getDerivedStateFromError(error)` вызывается после возниконовения ошибки в дочернем компоненте на этапе рендеринга  
Принимает ошибку в параметре и возвращает значение для обновления состояния. Используется для рендеринга запасного варианта при ошибке  
* Метод `componentDidCatch(error, info)` вызывается после `getDerivedStateFromError`: после рендера и во время применения сайд-эффектов. Используется для логирования ошибок, где  
	* `error` - ошибка
	* `info` - объект с ключом componentStack, содержащий информацию о компоненте, в котором произошла ошибка

> В функциональных компонентах нет аналога  
 
```typescript
class ErrorBoundary extends React.Component {
	constructor(props) {
			super(props);
			// Начальное состояние: ошибок нет
			this.state = { error: false };
	}
	static getDerivedStateFromError(error) {
			// Ошибка: отображаем запасной UI
			return {error: true};
	}
	componentDidCatch(error, info) {
			// логироваине или иная логика обработки
			console.info(info.componentStack);
			console.error(error);
			// Пользовательский метод обработки ошибки
			logComponentStackToMyService(info.componentStack); 
	render() {
		if(this.state.error) {
		// Запасной UI
				return <h1>Something wrong</h1>; 
		}
		// Ошибки нет
			return {this.props.children}
	}
}
```

[Пример использования react-error-boundary библиотеки](https://www.youtube.com/watch?v=gyqAW0--0Tc)  
[Объяснение методов жизненного цикла в Error Boundary](https://www.youtube.com/watch?v=_FuDMEgIy7I)  

[Вернуться к содержанию](#содержание)  

## 5.3 HOC (High Order Component)

Паттерн используемый во фреймворках для создания компонента высшего порядка, объединяющего логику компонентов схожей функциональности  
HOC-компонент получает аргументом исходный компонент и оборачивает его необходимыми свойствами и функционалом  
Принято называть HOC-компоненты со слова with: `withClick`  

```typescript
// собственные пропсы оборачиваемого компонента
type PersonalProps = {
	isEnabled: boolean;
}
// пропсы генерируемые HOC-компонентом дополняющие пропсы базового компонента
type WrapperGeneratedProps = {
	onButtonClick: () => void;
	buttonText: string;
}
// пропсы используемые только внутри HOC-компонента
type WrapperProps = {
	alertText: string;
}

// withClick.tsx
const withClick = (WrappedComponent: ComponentType<PersonalProps & WrapperGeneratedProps>): ComponentType<PersonalProps & WrapperProps> => {
	return function(props: PersonalProps & WrapperProps): ReactElement {
		{/* общий функционал всех компонентов оборачиваемых в HOC */}
		const onButtonClick = () => {
			alert(props.alertText)
		}
		{/* важное замечание: используем спред-синтаксис после всех генерируемых HOC-компонентом пропсов, чтобы не затерлись исходные передаваемые пропсы */}
		const newProps = {onButtonClick, buttonText: 'Button Text', ...props}
		return <WrappedComponent {...newProps} />
	}
}

// Button.tsx
const Button = ({isEnabled, onButtonClick, buttonText}: PersonalProps & WrapperGeneratedProps) => {
	return <button disabled={isEnabled} onClick={onButtonClick}>
		{buttonText}
	</button>
}

// App.tsx
const App = () => {
	//  передаем оборачиваемый компонент в HOC
	const [isEnabled, setIsEnabled] = useState(false);
	const WithClickButton = withClick(Button)
	return (
		{/* передаем пропсы базового компонента и пропсы используемые только внутри HOC-компонента */}
		<WithClickButton isEnabled={isEnabled} alertText="Some Text"/>
	)
}
```

[Видео без типизации](https://www.youtube.com/watch?v=KWT8OKzrMZ4)  
[Дополнения для HOC](https://reactdev.ru/archive/react16/higher-order-components/#static-methods-must-be-copied-over)  

[Вернуться к содержанию](#содержание)  

## 5.4 Compound component

**Compound component** - паттерн для переиспользуемых компонентов, с возможностью переупорядочивания дочерних компонентов.   
**Задача:** В компоненте `UserCard` элементы Header и Footer должны быть опциональными для возможности переиспользовать в разных частях приложения  
```typescript
type TCard = {
	title: string;
	name: string;
	age: number;
	subscribers: number;
}
type UserCardProps = {
	card: TCard;
}

export default const UserCard = ({card}: UserCardProps) => {
	return (
		<div>
			<header>
				<h2>{card.title}</h2>
			</header>
			<div>
				{card.name}
				<span>{card.age}</span>
			</div>
			<footer>
				<p>Subscribers: {card.subscribers}</p>
			</footer>
		</div>
	);
}
```
Примером решения может быть добавление булевых параметров, отвечающих за отображение отдельных частей (`isHeaderVisible: boolean`).  
Альтернативный метод - создание составного (compound) компонента:
```typescript
// изменение типа
type UserCardProps = PropsWithChildren & {
	card: TCard;
}

const UserCardContext = createContext<UserCardProps | undefined>(undefined);
// вспомогательный хук
function useUserCardContext() {
	const context = useContext(UserCardContext);
	if(!context) throw new Error('UserCardContext access error')
	return context
}

export default const UserCard = ({children, card}: UserCardProps) => {
	return (
		<UserCardContext value={card}>
			<div>
				{children}
			</div>
		</UserCardContext>
	);
}

UserCard.Header = function UserCardHeader() {
	const { card } = useUserCardContext(UserCardContext);
	return <header><h2>{title}</h2></header>
}
UserCard.Main = function UserCardMain() {
	const { card } = useUserCardContext(UserCardContext);
	return (
		<div>
			{card.name}
			<span>{card.age}</span>
		</div>
	);
}
UserCard.Footer = function UserCardFooter() {
	const { card } = useUserCardContext(UserCardContext);
	return <footer><p>Subscribers: {card.subscribers}</p></footer>
}
```
Использование компонента без футера с измененным порядком элементов 
```typescript
<UserCard
	card={{
		title: 'Title';
		name: 'Petr';
		age: 18;
		subscribers: 1000;
	}}
>
	<UserCard.Main />
	<UserCard.Header />
</UserCard>
```

[Видео-пример компонента](https://www.youtube.com/watch?v=N_WgBU3S9W8)

[Вернуться к содержанию](#содержание)  

# 6. ReactDOM. Элементы и события React

`SyntheticEvent` - обертка над стандартным событием `Event`, которая обеспечивает единообразное поведение в разных браузерах.  
`SyntheticEvent.nativeEvent` содержит все методы стандартного события  
События в React регистрируются в фазу всплытия (bubbling). Для регистрации в фазу (capturing) к событию нужно добавить слово Capture: `onClickCapture`  
Типизация события в React: `ChangeEvent<TSomeType>`  

----

Стандартные пропсы поддерживаемые всеми компонентами:  
* `children` - React узел, который может быть элементом, строкой, числом, порталом или пустым узлом (null, undefined), массивом React узлов.
* `dangerouslySetInnerHTML` - замена `innerHTML` для вставки сырого HTML, уязвимого к XSS  
`dangerouslySetInnerHTML: {__html: `<span>Some HTML</span>`}`
* `ref` - объект из `useRef`, `createRef`, callback-ref для связи ссылки с DOM-элементом
* `suppressContentEditableWarning` - скрытие предупреждений об использовании свойства `contentEditable`.  
Применение: при использовании React DND и компонентов с атрибутом contentEditable, т.к. `<input>, <textarea>` при добавлении draggable функционала теряют возможность ввода данных
* `suppressHydrationWarning` - скрытие предупреждение об отличии контента при серверном и клиентском рендере.  
Применение: при использовании Next в клиентских компонентах при ошибках гидратации или в сторонних библиотеках изменяющих данные  
* `style` - объект `CSSProperties` для применения стилей к элементу  

[Список всех пропсов в официальной документации](https://react.dev/reference/react-dom/components/common#reference)  

Компоненты бывают 2х видов: управляемые (с обратным связыванием - через useState) **ИЛИ** неуправляемые (без обратного связывания - через useRef)  
`<input>` считается управляемым, если задан проп `value`.  
`<input type="radio">`, `<input type="checkbox">` - считаются управляемыми, если задан пропс `checked`  
Управляемым инпутам должны быть заданы обработчики изменений.  
>`<input type="file">` - всегда неуправляемый компонент.  
  
В элементе `<option>` отсутствует пропс `selected`. Значение передается в `defaultValue` родительского элемента `<select>` (неуправляемый список) или в `velue` (управляемый список)

В элемент `<textarea>` нельзя передать `chidren`. Для установки начального значения используется `defaultValue` (неуправляемый), или `value` (управляемый)

```typescript
const MyComponent = () => {
	const [mode, setMode] = useState('dark');
	const onValueChange = (e: ChangeEvent<HTMLInputElement>) => {
		setMode(e.target.value);
		// для чекбоксов  - setState(e.target.checked)
	}
	return <input type="radio" value="light" checked={mode === "light"} onChange={onValueChange}
}
```

Можно использовать одно состояние для нескольких полей ввода.  
Для доступа к состоянию конкретного элемента нужно произвести связывание имен в состоянии и в атрибуте элемента
```typescript
const MyComponent = () => {
	const [state, setState] = useState({checkbox: false, text: ''})
	const handler(event: ChangeEvent<HTMLInputElement>) => {
		const target = event.target;
		const value = target.type === "checkbox" ? target.checked : target.value
		const name = target.name
		setState((prevState) => {
			...prevState,
			[name]: value
		})
	}
	
	return (
		<>
			<input type="checkbox" name="checkbox" checked={state.checkbox} onChange={handler}/>
			<input type="text" name="text" value={state.text} onChange={handler}/>
		</>
	)
}
```

**Пример с замыканием**
Возвращаем функцию в которой будет доступна переменная `name` из родительской функции
```typescript
const MyComponent = () => {
	const [state, setState] = useState({checkbox: false, text: ''})
	const handler = 
			(name: string) => (event: ChangeEvent<HTMLInputElement>) => {
			const target = event.target;
			const value = target.type === "checkbox" ? target.checked : target.value
			setState((prevState) => {
				...prevState,
				[name]: value
			})
		}
	
	return (
		<>
			<input type="checkbox" name="checkbox" checked={state.checkbox} onChange={handler('checkbox')}/>
			<input type="text" name="text" value={state.text} onChange={handler('text')}/>
		</>
	)
}
```

При работе с данными от сервера, пришедшее значение может быть null или undefined, что принудительно изменит режим компонента на неуправляемый.  
При получении значения от пользователя режим изменится на управляемый  
Чтобы избежать ошибки с изменением режима, нужно контролировать исходное значение: `{state.requestData ?? ""}`  

[Вернуться к содержанию](#содержание)  

# 7. Portals

[Порталы на практике](https://www.youtube.com/watch?v=V4sHZzX4zh0)  

`createPortal(children, domNode, key?)` - позволяет отрендерить дочерние элементы `children` вне иерархии родительского компонента (в другую часть DOM)  
Портал меняет только физическое расположение узла DOM. JSX, который помещается в портал, действует как обычный дочерний узел (имеет доступ к состояниям родителя и т.д.)  
* `children` - React узел
* `domNode` - DOM-узел, в котором будет рендериться children
* `key` - опциональный ключ (см. [DOM Diffing](#react-под-капотом))

События от порталов распространяются в соответствии с деревом React, а не DOM  
Использование: создания модальных окон, тултипов   

[Вернуться к содержанию](#содержание)  

# 8. Переменные окружения в React

[Общая информация про переменные окружения](https://www.youtube.com/watch?v=HiRxC7WeNZU)  
[Подробно про создание переменных окружения](https://www.youtube.com/watch?v=wkfWaI_lI48) 
[Текстом про переменные окружения](https://danshin.ms/React-Environment-Variables/) 

В React все переменные окружения **должны начинаются** с `REACT_APP`: `REACT_APP_API_KEY`  
Использование в файлах: `process.env.REACT_APP_API_KEY`

Основные методы: 
1. в настройках системы Windows (Изменение системных переменных сред) или другой ОС
2. в кроссплатформенной среде cross-env (необходима установка `npm install -D cross-env `) `cross-env REACT_APP_CROSS_ENV=value`
3. в .env файлах (необходима установка `npm install -D dotenv-webpack`): `REACT_APP_API_KEY=8sdf3218652sdfq84531203asasd`  

Подключение в webpack: 
```typescript
const Dotenv = require('dotenv-webpack');

module.exports = {
    plugins: [
        new Dotenv()
    ]
} 
```

[Вернуться к содержанию](#содержание)  

# 9. Серверный рендеринг

## 9.1 Директивы  

Директива **должна находиться** в самом верху файла - **выше импортов!** (исключение: комментарии)  

Пропсы, передаваемые от серверного компонента клиентскому должны быть сериализуемы.   
_Несериализуемы: функции, классы, объекты с прототипом null, символы не зарегистрированные глобально_

### 9.1.1 use client

Директива `use client` служит для обозначения клиентских компонентов (по умолчанию компоненты серверные). Она служит границей между клиентской и серверной частью приложения   

>Клиентский компонент может импортировать ТОЛЬКО клиентские компоненты (импортируемым компонентам не обязательно наличие директивы). _Импорт серверного компонента приведет к ошибке._  
Для использования серверных компонентов в качестве дочерних, нужно передать их через пропс `children` 

Клиентские компоненты используются при наличии: 
* хуков 
* обработчиков событий
* работы с BOM (cookies, storage, navigation и т.д.)
* сторонних библиотек использующих хуки и BOM

[Примеры в официальной документации](https://react.dev/reference/rsc/use-client)

[Вернуться к содержанию](#содержание)  

### 9.1.2 use server

Директива `use server` служит для обозначения серверных действий. Все компоненты по умолчанию являются серверными.  

Серверные компоненты используются при наличии асинхронных запросов или статических файлов, не использующих клиентский функционал  
Важно следить за передачей чувствительных данных в качестве пропсов на клиент (например, секретные ключи)  
_Существуют экспериментальная экранировка передаваемых пропсов: [experimental_taintObjectReference](https://react.dev/reference/react/experimental_taintObjectReference) и [experimental_taintUniqueValue](https://react.dev/reference/react/experimental_taintUniqueValue)_  
`experimental_taintObjectReference(message, object)`, помогает предотвратить передачу объекта `object` клиенту, отображая сообщение `message` (сравнение по ссылке)  
Не гарантирует защиту от передачи уязвимых данных, т.к. объект можно склонировать и т.д.  
`taintUniqueValue(errMessage, lifetime, value)` - помогает предотвратить передачу уникального значения `value` (string, bigInt, TypedArray) клиенту, отображая сообщение `message`, где  
`lifetime` -  объект, существование которого ограничивает передачу чувствительных данных на клиент  
Не гарантирует защиту от передачи уязвимых данных, т.к. строки можно преобразовать в другой регистр и т.д. 

[Примеры в официальной документации](https://react.dev/reference/rsc/use-server#calling-a-server-action-outside-of-form)

[Вернуться к содержанию](#содержание)  

## 9.2 React Server Component

Серверные компоненты (RSC) позволяют рендерить контент на сервере, не нагружая клиент.  
При использовании серверных компонентов _без сервера_, генерация HTML и обработка данных происходит в процессе сборки.  
**Преимущества:**  
* Ускорение загрузки и освобождение ресурсов браузера (уменьшение FCP и TTI)  за счет исключение клиентского рендера и обработки данных, т.к. серверные компоненты отправляются на клиент уже отрендеренными 
* Улучшение SEO, т.к. происходит отправка полностью отрендеренных страниц 
* Размер клиентского бандла уменьшается, т.к. исключаются тяжелые библиотеки и зависимости, используемые для обработке данных на сервере
* Безопасность при использовании секретных данных (API key), т.к. они не передаются на клиент  
* Упрощение структуры проекта и уменьшение количества запросов, т.к. серверные компоненты могут напрямую обращаться к БД или другим источникам данных без создания отдельного API

Серверные компоненты используются в связке с клиентскими компонентами, которые добавляют интерактивность (обработчики событий, BOM, хуки)  
Клиентские компоненты не всегда рендерятся только на клиенте. Они могут быть предварительно отрендерены на сервере и ререндериться на клиенте, из-за чего может возникнуть warning, что данные рендера отличаются  
_Асинхронными компонентами могут быть только RSC_   
>RSC рендерятся **только один раз**, поэтому их нельзя импортировать в клиентских компонентах (клиентский компонент может перерендериться при изменении state).

[Отличия между серверными и клиентскими компонентами](https://www.youtube.com/watch?v=ePAPd9qzGyM)  
[То же самое, но покороче](https://www.youtube.com/watch?v=Qdkg_mrniLk)  
[Отдельно про передачу серверных компонентов в клиентские](https://www.youtube.com/watch?v=9YuHTGAAyu0)  
[Еще один вариант объяснения, если не хватило предыдущих](https://www.youtube.com/watch?v=rGPpQdbDbwo)

[Вернуться к содержанию](#содержание)  

## 9.3 Server Action

Серверные действия - механизм использования серверного кода в клиентском компоненте без создания api и запроса.  
В клиентский компонент передается не сама функция, а ссылка на нее.  
>Серверные действия должны быть отмечены директивой `use server`

**Преимущества:**
* Упрощение взаимодействия сервер-клиент. Исключает необходимость созать отдельные API роуты.
* Безопасность - функция НЕ отправляется на клиент, передается ссылка на функцию.
* Интеграция с новыми хуками useActionState

Пример Next без серверных действия
```typescript
// создание запроса к серверному API на клиенте
const handleSubmit = () => {
	const response = fetch('/api/addPost', {
		method: 'POST',
		body: JSON.stringify(newPost)
	})
}

// создание API роута на сервере 
const handler = (req, res) => {
	if(req.method === 'POST') {
		await db.posts.create(req.body);
		res.status(200).json({success: true})
	} else res.status(405).end()
}
```
Пример с серверными действиями:
```typescript
// серверное действие
'use server'

export const create = async (data) => {
	await db.posts.create(data);
}

// использование в клиентском компоненте
'use client';
import {create} from './actions'
const Index = () => <button onClick={() => create(newPost)}>Create<button>
```

[Вернуться к содержанию](#содержание)  

# 9.4 cache

`cache(fn)` - кэширование результата (включая ошибки) функции `fn`. Используется **только** в RSC и объявляется снаружи компонента  
Кэш активен короткое время, пока выполняется серверный запрос. React инвалидирует кэш всех мемоизированных функций при завершении запроса  
Каждый вызов `cache`, создает новую функцию, которая имеет собственный кэш.  

Сравнение методов кэширования:
||useMemo|cache|memo|
|----|----|----|----|
|Тип компонента|Клиентский|Серверный|Клиентский|
|Возможность передачи между компонентами|Нет|Да|Да|
|Время жизни кэша|Между рендерами, пока не изменятся deps|Ограничен одним серверным запросом|Между рендерами, пока не изменятся deps|
|Кэшируемые данные|Вычисления|Данные и вычисления|Компонент|

[Подробное объяснение](https://www.youtube.com/watch?v=A8JGtz2yF9g)  

[Вернуться к содержанию](#содержание)  

## 10. Проекция состояния класса (редкий кейс)

Примеры обеспечения взаимодействия класса с функциональными компонентами

## 10.1 Класс с редко изменяемым состоянием

* Собственное состояние класса - `todos`  
* Метод `getList` возвращает состояние класса  
* Методы `add`, `remove` изменяют состояние по условию (например, нажатие кнопки)+
* Хук `useTodos` - интегрирует логику класса с компонентами React
```typescript
export interface Todo {
  id: string;
  title: string;
}

export interface ITodos {
  getList(): Todo[];
  add(todo: Todo): void;
  remove(id: string): void;
}

export class Todos implements ITodos {
  constructor(private todos: Todo[] = []) {}

  getList(): Todo[] {
    return this.todos;
  }
  add(todo: Todo): void {
    this.todos.push(todo);
  }
  remove(id: string): void {
    this.todos = this.todos.filter(todo => todo.id !== id);
  }
} 
```

Для взаимодействия с классом нужно создать функцию, где:
1. Получить инстанс класса в ref
2. Сохранить собственное состояние класса во внешнем состоянии state
3. Описать взаимодействие с методами изменяющими собственное состояние класса, обновляя внешний state 

```typescript
export function useTodos(initialTodos?: Todo[]) {
	// [1] - получаем интстанс
  const todos = useRef<ITodos>(new Todos(initialTodos));
	// [2] - сохраняем состояние класса state
  const buildTodosState = useCallback(() => [...todos.current.getList()], []);
  const [state, setState] = useState(buildTodosState);

  // [3] - методы изменяющие состояние
  const addTodo = useCallback(
    (todo: Todo) => {
      // Вызываем методы класса
      todos.current.add(todo);
      // Обновляем state
      setState(buildTodosState());
    },
    [buildTodosState]
  );

  const removeTodo = useCallback(
    (id: string) => {
      todos.current.remove(id);
      setState(buildTodosState());
    },
    [buildTodosState]
  );

  // Возвращаем доступные методы вместе с состоянием
  return [state, { addTodo, removeTodo }] as const;
	// as const позволяет указать возвращаемое значение как кортеж
	// ограничивает и конкретизирует количество элементов
} 
``` 

Использование хука в компонентах  
Если функция передается в HTML элемент, то не нужно использовать useCallback  
Если функция передается в сам компонент, то:
```typescript
const MyComponent = () => {
	const [todos, { addTodo, removeTodo }] = useTodos();
	const onAdd = useCallback(() => {
    // Пользуемся методами мутации
    addTodo({ id: '123', title: 'Новое todo' });
  }, [addTodo]);
}
```

[Вернуться к содержанию](#содержание)  

## 10.2 Класс с часто изменяемым состоянием

```typescript
class Timer {
  private timer?: number;
  private intervalTime: number;
	// передаваемый извне callback 
  private callback?: () => void;
	// часто меняющееся состояние
  time: number;

  constructor(intervalTime: number = 1000) {
    this.timer = undefined;
		this.intervalTime = intervalTime;
    this.callback = undefined;
    this.time = 0;

    this.updateTime();
  }

  // передаем callback снаружи
  onChange(callback: () => void) {
    this.callback = callback;
  }

  updateTime() {
    this.time = Date.now();
		// Вызываем callback на каждое обновление
    this.callback?.();
  }

  start() {
    this.timer = setInterval(() => {
      this.updateTime();
    }, this.intervalTime);
  }

  stop() {
    clearInterval(this.timer);
  }
} 
```

Для взаимодействия с классом нужно создать функцию, где:
1. Получить инстанс класса в ref
2. Передаем useReducer как callback внутрь класса для принудительного обновления внешнего состояния

```typescript
function useTimer() {
  // [1] - получаем интстанс
  const ref = useRef(new Timer());
	// состояние useReducer используется для ререндер компонента
	// это число, увеличивающееся при каждом вызове forceUpdate (в примере изменении таймера)
	const [_, forceUpdate] = useReducer(v => v + 1, 0);

  useEffect(() => {
    const timer = ref.current;
    timer.start();
		// [2] - передаем useReducer в callback
		timer.onChange(forceUpdate);
    return () => {
      timer.stop();
    };
  }, []);

	// Состояние класса отдаём наружу
  return ref.current.time;
}
```

Использование хука в компонентах:
```typescript
function TimerUI() {
  const time = useTimer();
} 
```

[Вернуться к содержанию](#содержание)  

## 10.3 Класс без собственных состояний

```typescript
export interface ApiInterface {
  getPosts: () => Promise<string[]>;
}

export class Api implements ApiInterface {
  getPosts() {
    return Promise.resolve(['Первый', 'Второй', 'Третий']);
  }
} 
```
Класс включает в себя только полезные методы, либо его состояние не нужно для функционала компонентов
Для взаимодействия с ним
1. Получим инстанс класса в ref
2. Создать контекст (контекст должен реализовывать функционал класса)

```typescript
// [1] - создаем контекст
export const ApiContext = createContext<ApiInterface>({
    getPosts: () => Promise.resolve([]),
}); 
```

Использование в компонентах:
```typescript
// [2] - получаем инстанс
const apiRef = useRef(new Api());
<ApiContext.Provider value={apiRef.current}></ApiContext.Provider>


const api = useContext(ApiContext);
useEffect(() => {
	api.getPosts().then();
}, [api]);
```

[Вернуться к содержанию](#содержание)  

# Дополнительно 

>**P.S.** Если данная информация полезна, то буду рад обратной связи по найденным ошибка в тексте.  
Дальше плавный переход в Next

[Паттерны в React](https://www.patterns.dev/react)  
[Пример рефакторинга кода в React](https://www.youtube.com/watch?v=KJEjJF2BmLw)  
[Повторение базовых принципов написания кода в React](https://www.youtube.com/watch?v=5r25Y9Vg2P4)  

[Вернуться к содержанию](#содержание)  
