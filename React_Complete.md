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
	- [4.7 useEffectEvent](#47-useeffectevent)
	- [4.8 useLayoutEffect](#48-uselayouteffect)
	- [4.9 useInsertionEffect](#49-useinsertioneffect)
	- [4.10 useMemo](#410-usememo)
	- [4.11 React.memo](#411-reactmemo)
	- [4.12 useCallback](#412-usecallback)
	- [4.13 useDeferredValue](#413-usedeferredvalue)
	- [4.14 useTransition](#414-usetransition)
	- [4.15 useSyncExternalStore](#415-usesyncexternalstore)
	- [4.16 use](#416-use)
		+ [4.16.1 Promise in use](#4161-promise-in-use)
		+ [4.16.2 Context in use](#4162-context-in-use)
	- [4.17 useId](#417-useid)
	- [4.18 useOptimistic](#418-useoptimistic)
	- [4.19 useDebugValue](#419-usedebugvalue)
	- [4.20 Кастомные хуки](#420-кастомные-хуки)
	
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

[Вернуться к содержанию](#содержание)

## 1.2 Функциональный компонент

```typescript
type FunctionalComponentProps = { defaultValue: number }
{/* можно использовать стрелочную функцию */}
function FunctionalComponent({ defaultValue }: FunctionalComponentProps) { 
	{/* хуки описываются ДО основной логики */}
	const [value, setValue] = useState(defaultValue)
	{/* логика */}
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
{/* если в компоненте не передаются пропсы, то передается пустой объект {} в дженерике */}
interface IProps { initialValue: number }
interface IState { value: number }
class ClassComponent extends React.Component<IProps, IState> {
	{/* props передаются в качестве аргументов конструктора */}
	constructor(props: IProps) {
		super(props)
		{/* состояние задается в конструкторе, в зарезервированном свойстве state */}
		this.state = {
			value: props.initialValue
		}
		this.increment = this.increment.bind(this) {/* методам внутри конструктора необходимо присваивать контекст */}
	}
	{/* вместо хуков используются методы*/}
	{/* состояние не меняется напрямую, любые изменениея происходят через setState */}
	increment(){ this.setState({ value: this.state.value + 1 }) }
	{/* альтернативный способ создания метода (без привязки контекста в конструкторе) */}
	{/* в обоих случаях методы пренадлежат экземпляру класса, а не прототипу */}
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
Для взаимодействия с жизненным циклом классового компонента можно использовать методы:
* componentDidMount() - вызывается после монтирования
* componentDidUpdate(prevProps, prevState) - вызывается после обновления внутреннего состояния или пропсов. **Нельзя вызвать при монтировании (первом рендере)** Можно работать с реальным DOM-деревом
	prevProps - предыдущие пропсы, prevState - предыдущее состояние  
	Для использования setState внутри метода необходимо обернуть его в условие, чтобы не возник бесконечный цикл
* shouldComponentUpdate(nextProps, nextState) - сравнивает текущие значения props и/или state и решает нужен ли повторный рендеринг (boolean). Используется для оптимизации мест с частым рендерингом    
	вернул true - при изменении props и/или state происходит повторный рендеринг
	вернул false - при изменении props и/или state повторный рендеринг не произойдет
* componentWillUnmount() - вызывается перед размонтированием для устранения утечек памяти (удаление таймеров, отписка от событий, закрытие соединения с сервером)
* componentDidCatch(error, info) - вызывается после возникновения ошибки в дочернем компоненте. В отличие от `getDerivedStateFromError(error)` в методе можно использовать побочные эффекты
* getSnapshotBeforeUpdate(prevProps, prevState) - вызывается перед применением изменений в `commit phase`. Возвращаемые значения будут переданы в `componentDidUpdate`

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

```typescript
<CustomComponent attr1={value} attr2={{value: 10}} attr3="text">Some Text</CustomComponent>
```
Передавать props в компонент можно: 
* между тегами (в `props.children`) `<MyComponent><AnotherComponent /></MyComponent>`  
Приоритет в пробрасывании `props.children` между тегами компонента - выше, чем при указании аналогичного атрибута `children={...}`
* через одинарные `{}` - переменные `<MyComponent loadintState={isLoading} />`
* через `{{}}` - объекты `<MyComponent personData={{name: 'Petr', age: 18}} />`
* через `""` - текст `<MyComponent title="Some Text"/>`

**Компонент ререндерится при изменнеии переданных props.**  
Деструктуризируя `...props`, можно выделить необходимые для передачи в дочерний компонент пропсы
```typescript
const ParentComponent = ({children, ...props}: ParentComponentProps) => {
	return (
		<ChildCompoent key={props.id}>{children}<ChildComponent>
		{/* установка props без присваивания отдельным арибутам */}
		<AnotherComponent {...props} />
	)
}
```

Пропсы передаются от родительского компонента к дочернему. Для передачи данных в обратном направлении нужно передать **функцию обратного вызова** в дочерний компонент  
**Важно** передавать функцию `{myFunc}`, а не ее вызов `{myFunc()}` _(неправильный вариант)_
```typescript
funсtion App() {
	const myFunc = () => {...логика...}
	return (
		<MyComponent getValue={myFunc} />
	)
}
```

## Пример 1

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

## Пример 2 
Через функцию обратного вызова можно возвращать children:
```typescript
function App() {
	return (
		<MyComponent>
			{/*функция обратного вызова*/}
			{(age, name) => (
					<form>
						<label>{name}</label>
						<p>{age}</p>
					</form>
				)
			}
		</MyComponent>
	)
}

type Props = {
  children: (age: number, name: string) => JSX.Element
}
function MyComponent({children}: Props) {
	const name = 'Petr'
	const age = 18
	return children(age, name)
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
>При передаче больших объемов данных, например массива `useState([0, 1, ..., 999999])`, он будет пересоздаваться при каждом ререндере, даже если содержимое не изменяется, что может повлиять на производительность, поэтому оправдано использование функции 

Напрямую изменять (mutate) состояние нельзя, поэтому любые *мутирующие* методы (sort, reverse и т.д.) должны вызываться на копии  
[Проверка методов на мутации](https://doesitmutate.xyz/)  

Для объектов - `setState(...state, stateArg: newValue)` или для массивов - `setState([...state, newValue])`  
Спред-синтаксис действует поверхностно  
Изменение состояния вложенных полей (при большом уровне вложенности) увеличивает и дублирует код: `setState(...state, innerObj: {...state.innerObj, innerArg: newValue})`  

**Важно** не забывать, что объект - ссылочный тип:  
```typescript
const [state, setState] = useState([{innerArg: 1}, {innerArg: 2}])
function handleClick() {
	const newValue = [...state]
	{/* ! ОШИБКА ! Происходит мутация ссылочного типа (объекта) */}
	newValue[0].innerArg = 2 
	setState(newValue)
	{/* Правильный вариант: */}
	state.map((item) => {
		if(...условие) return {...item, innerArg: 10}
		else return item
	})
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
	innerObj: {
		innerArg: innerValue
	}
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
	3. объединяются макротаска и все, запущенные в ней микротаски, либо синхронные события  
	Следующая за ней (другая) макротаска запустит новый рендер
	4. стоит учесть особенность поведения при указании в таймере нулевой задержки,  
	если к моменту завершения длительной микротаски успеет таймер с 0 задержкой и отличной от 0,  
	то таймер с 0 задержкой вызовет собственный ререндер (**данное поведение может отличаться в разных браузерах**)
	
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
	{/* таймер срабатывает позже - новый ререндер */}
	{/* если асинхронная функция выполняется более 200мс, то таймер добавится в ререндер к двум предыдущим*/}
	setTimeout(() => {setState1(20)}, 200)
	{/* таймер с нулевой задержкой вызвает собственный ререндер в Chrome */}
	{/* в разных браузерах может вести себя по-разному */}
	setTimeout(() => {setState1(30)}, 0)
	{/* асинхронная функция вызывает собственный ререндер */}
	asyncFunc().then(() => {}
}
```
Для многократного изменения состояния **ДО нового** рендера следует использовать **функцию апдейтер**:  
`setState((prev) => prev + 1)`, которая учитывает предыдущее изменение  
Функция встает в очередь на обработку. Во время следующего рендера последовательно вызываются все изменения состояния из очереди  

**Пример**:
```typescript
const [state, setState] = useState(0)
setState(state + 1) // state = 0 + 1
setState(state + 1) // state = 0 + 1
setState(prev => prev + 1) // state = 1 + 1
setState(newValue) // state = newValue
```
В новом рендере состояние будет равно `newValue`, так как все изменения состояния были добавлены в очередь  

---
**React сохраняет состояние компонента, если он НЕ ИЗМЕНЯЕТ позиции в DOM-дереве**  
Для сохранения состояние между ререндерами структура дерева не должна отличаться между рендерами  
(_например: одинаковый компонент сохраняет позицию второго дочернего элемента в родителе_)  
```typescript
<div>
	<p>SomeText</p>
	{theme === 'dark' 
		? (<MyComponent theme='dark' />) 
		: (<MyComponent isFancy='light' />)
	}
</div>
```  
Для сброса состояние компонента можно: 
1. Изменить позицию сбрасываемого компонента
2. Установить в сбрасываемом компоненте атрибут `key`:
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

В React есть:
* высокопроизводительные таски: 
	1. useState
	2. useReducer
	3. useSyncExternalStore 
* низкопроизводительные таски:
	1. useTransition
	2. startTranstion
	3. useDeferredValue

Высокоприоритетные таски прерывают выполнение низкоприоритетных.  
Если компонент начал исполняться, то его нельзя прервать, пока не дойдем до return (render)  
В процессе прервывания низкоприоритетные таски возвращают свой предыдущий state (последний актуальный, т.е. примененный в последнем рендере)

### 4.1.3 flushSync

`flushSync` - заставляет React произвести ререндер сразу после вызова этой функции
Иногда не требуется использовать batching для объединения изменений состояний, _например одно состояние зависит от другого_:
```typescript
const [age, setAge] = useState(0)
const [human, setHuman] = useState({name: 'Petr', age: 18})
handleClick() {
	flushSync(() => {
		setState(21)
	})
	{/* здесь произойдет принудительный ререндер и human получит обновленное значение age */}
	setState2({...human,  age: age})
}
```

### 4.1.4 Ошибки

**Основные ошибки** при создании состояний: 
* Большое количество state, обновляемых одновременно - решение: объединение в один объект state
* Взаимоисключающие state - решение: можно объединить в один state
* Дублирующие state - решение: можно вычислить как переменные на основе других state
* State дублирует передаваемый props - решение: можно использовать, если не требуется обновление состояния при изменении props
* Дублирование state объекта - решение: можно хранить только значимое поле вместо всего объекта, например id
* Большая вложенность в state - решение: можно нормализовать вложенность (сделать плоским объектом)  

[Вернуться к содержанию](#содержание)

## 4.2 useReducer

[Детальный разбор(видео)](https://www.youtube.com/watch?v=rgp_iCVS8ys)  
[отличие useState от useReducer(видео)](https://www.youtube.com/watch?v=3VClygDRSsU)  
[Еще один(видео)](https://www.youtube.com/watch?v=kK_Wqx3RnHk)  
`const [state, dispatch] = useReducer(reducerFuntion, initialState, init?)` - аналогично useState, дополнительный аргумент - функция, объединяющая логику изменения состояния
* state - текущее состояние
* dispatch - функция, запускающая reducerFuntion
* reducerFuntion - функция, объединяющая логику изменения состояния в зависимости от условий  
	**(функция должна быть "чистой" без side-эффектов)**. Возвращает следующее состояние  
* initialState - начальное значение состояния
* init - функция, вычисляющая начальное значение на основе второго аргумента initialState 
	функция инициализации оставленная в компоненте будет вычисляться при каждом ререндере (тяжелые вычисления могу повлиять на производительность)  
	при передаче функции (_НЕ вызовва функции_) в useReducer она вычисляется только при первом рендере

`dispatch({type: string[, payload: data]})`, где type - тип изменения, payload - передаваемые данные  
Используется для объединения логики обновления состояния в одну функцию  
|                                |  useReducer   |        useState         |
|--------------------------------|:-------------:|------------------------:|
| тип состояния                  | Object, Array | Nubmer, String, Boolean |
| количество изменений состояния |      > 2      |         1 или 2         |
| дополнительная обработка       |       да      |           нет           |  

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

Для получения значения обновленного состояния в текущем (до рендера) можно вычислить его вызвав редьюсер  
```typescript
dispatch(action)
const nextState = reducer(state, action)
```
[Вернуться к содержанию](#содержание)

## 4.3 useContext

[детальный разбор(видео)](https://www.youtube.com/watch?v=HYKDUF8X3qI)  
[лучший разбор(видео)](https://www.youtube.com/watch?v=I7dwJxGuGYQ)  
[еще один(видео)](https://www.youtube.com/watch?v=5LrDIWkK_Bc)  
`const context = useContext<TSomeType>(customContext)` - позволяет получить доступ всем **дочерним** компонентам к контексту, не используя props  
(в компоненте Provider доступа к его контексту нет)  
* TSomeType - типизация контекста
* customContext - контекст
Используется: 
* при передача props на более чем 2 уровня вложенности компонентов, при передаче props без использования в промежуточных компонентах (Props Drilling)  
* когда данные редко изменяются (тема, язык), т.к. любое изменение контекста ведет к ререндеру всех дочерних провайдеру контекста компонентов
	для часто меняющихся данных используются библиотеки с глобальным хранилищем: Redux и т.д.    
```typescript
// Context.ts
type StateType = string
type CustomType = {
	state: StateType
	setState: React.Dispatch<React.SetStateAction<StateType>>
}
export const CustomContext = createContext(initialValue)
{/* или с типизацией */}
export const CustomContextWithType = createContext<CustomType | undefined>(undefined)

// ParentComponent.tsx
const [state, setState] = useState<StateType>("Petr")
{/* value - зарезервированный атрибут для передачи значения контекста */}
<CustomContextWithType.Provider value={{state, setState}}>
	<ChildComponent />
</CustomContextWithType.Provider>

// ChildComponent.tsx
export function ChildComponent() {
	const currentContext = useContext(CustomContextWithType)
	... далее можно использовать {state, setState}, но context может быть undefined
}
```

Для исключения undefined в контексте можно использовать кастомный хук:
```typescript
// Context.ts
export function useCustomContext() {
	const currentContext = useContext(CustomContextWithType);
	if(currentContext === undefined) throw new Error('Context is undefined')
	return currentContext
}

// ChildComponent.tsx  
export function ChildComponent() {
	const currentContext = useCustomContext()
	... далее можно использовать {state, setState}
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

При передаче в контекст объект/массив - ссылочный тип, нужно обернуть его в useMemo, функцию - useCallback/useMemo (разница см. 4.9, 4.10)  
useMemo сохранит ссылку на ссылочный тип и не будет создавать его каждый раз с одинаковыми данными
`const myContext = useMemo<ContextProps>(() => ({name: 'Petr', age: 18}),[])`  

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
**Важное замечание**  
[Правильная реализация контекста](https://www.youtube.com/watch?v=16yMmAJSGek)  
При данной реализации все дочерние компоненты (даже не использующие useContext) будут ререндериться при изменении контекста:
```typescript
export const CustomContext = createContext<{state: number, setState: React.Dispatch<React.SetStateAction<number>>} | null>(null)
const [state, setState] = useState(0)
...
<CustomContext.Provider value={{ state, setState }}>
	<ComponentUseContext />
	<ComponentDontUseContext />
</CustomContext.Provider>
```
Правильная реализация. Компонент, не использующий useContext, не будет ререндериться при изменении контекста:
```typescript
// custom-context.tsx
export const CustomContext = createContext<{state: number, setState: React.Dispatch<React.SetStateAction<number>>} | null>(null)

export default function CustomContextProvider({ children }) {
	const [state, setState] = useState(0)
	return <CustomContext.Provider value={{ state, setState }}> {children} </CustomContext.Provider>
}

//MyComponent.tsx
return (
	<CustomContextProvider>
		<ComponentUseContext />
		<ComponentDontUseContext />
	<CustomContextProvider>
)
```
[Вернуться к содержанию](#содержание)

## 4.4 useRef

[детальный разбор(видео)](https://www.youtube.com/watch?v=42BkpGe8oxg)  
[еще один(видео)](https://www.youtube.com/watch?v=t2ypzz6gJm0)  
`const ref = useRef<TSomeType>(initialValue)` - хранение ссылки на DOM-элемент или данные между ререндерами компонента, изменения которых не должы приводить к перерисовке компонента
* TSomeType - типизация значения
* initialValue - исходное значение
	
`useRef` возвращает объект с полем `current`, в котором можно хранить данные между ререндером, не используя state  
```typescript
const [state, setState] = useState(0)
const ref = useRef(0)
const handleCLick = () => {
	useState(prev => prev + 1)
	ref.current += 1
	{/* 0, значение текущего snapshot. Оно изменится после завершения кода обработчика и ререндера */}
	console.log(state)
	{/* 1, значение изменяется сразу, так как не зависит от рендера */}
	console.log(ref.current)
}
```

Начальное значение ref устанавливается один раз при первом рендере, но при передаче функции в начальное значение, она вычисляется при каждом рендере  
`const ref = useRef(MyFunc())`  
Для избежания повторных вычислений нужно добавить условие:
```typescript
const ref = useRef(null)
if(ref.current === null) ref.current = useRef(MyFunc())
```

В ref можно передать любой тип данных, DOM-элемент или функцию-колбек, которая запустится после рендера компонента  
**Вызов колбек рефоф происходит до срабатывания useEffect/useLayoutEffect**    
При использовании колбек ref при ререндере происходит размонтирование ref и передача нового колбека(т.к. функция - ссылочный тип)  
_Проблема_: повторный вызов колбека  
_Решение проблемы_: мемоизация колбека `const callbackRef = useCallback((element) => {console.log(element)})`  
[Подробно про проблемы рефов и решения через колбек рефы](https://www.youtube.com/watch?v=MLWsLn_jeGc)  
[useForkRef - кастомный хук](https://ollylut.medium.com/what-is-useforkref-hook-4be1c85d2d1b "VPN")
```typescript
function App() {
	const myRef = useRef<HTMLInputElement>(null);
	{/* принудительное обновление покажет в консоли null и <input/> */}
	const [, forceUpdate] = useReducer((v) => v+ 1, 0)
	function getValue() {
		return myRef.current.value
	}
	return (
		<>
			<input ref={myRef} />
			<button onClick={getValue} />
			{/* функция-колбэк*/}
			<input ref={(element) => {console.log(element)}} />
			<button onClick={forceUpdate} />
		</>
	)
}
```

При передаче ref как атрибута **функционального** компонента, нужно обернуть этот компонент в `forwardRef`  
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
const MyComponent = forwardRef((props, ref) => { 
	return <input ref={ref} />
})
```
 
В классовых компонентов используется `createRef` внутри конструктора - `this.myRef = React.createRef();`  
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
	{/* внутри функционального компонента создаем ref как в классовом компоненте */}
	const buttonRef = createRef<Button>();
	useEffect(() => {
		{/* type guard и использование кастомного метода компонента через ref */}
		buttonRef.current && buttonRef.current.setFocus()
	})
	return (
		// связывание классового компонента с ref
		<Button ref={buttonRef} onClick={handleClick} />
	)
}
//Class Component
interface ButtonProps {}
export class Button extends React.Component<ButtonProps, {}> {
	private buttonRef: React.RefObject<HTMLButtonElement>;
	constructor(props: ButtonProps) {
		super(props)
		{/* создание ref для связывания с HTMLElement */}
		this.buttonRef = React.createRef()
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

**Не обязательно привязывать переданный ref, к HTMLElement'у дочернего компонента**. Ref служит для передачи функционала между Parent-Child
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
	{/* Для привязки переданного ref к HTMLElementу необходимо создать новый локальный ref, так как в кастомизируемом нет методов HTMLElement */}
	const localRef = useRef<HTMLInputElement>(null);
	const reset = () => {
		setState({name: ''});
	}
	{/* ограничиваем родительский компонент двумя доступными методами */}
	useImperativeHandle(ref, () => ({
		reset,
		{/* Добавляем свойства InputElement к кастомизируемому ref 
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
[Еще один(видео)](https://www.youtube.com/watch?v=0ZJgIjIuY7U)  
`useEffect(callback, deps?)` - выполняет побочные эффекты при монтировании, размонтироваии и изменении состояния компонента
* callback - функция (побочный эффект), выполняющаяся после рендера компонента
* deps - зависимости в формате [dep1, dep2, dep3], если не переданы аргументы, то функция будет выполняться при каждом рендере,   
	если передан пустой массив, то при первом рендере,  
	если в аргументах передать `[prop, state]` prop и/или state, функция выполнится при изменении prop и/или state
	
useEffect не стоит применять для обработки событий пользователя (нужно вынести в обработчик), для преобразования данных для рендеринга (вынести на верхний уровень компонента)  
`useEffect` не следит за изменениями **внутри элементов массива или объекта**. Ререндар произойдет при передаче _нового_ массива или объекта  
Массив/объект/функция - ссылочный тип данных, если передать их как зависимости, то useEffect будет вызваться каждый раз при ререндере.  
Проблема решается передачей элементов массива/объекта/функции (примитивных типов)  
useEffect может возвращать фукнцию очистки, которая выполнится при размонтировании элемента или перед изменением его зависимостей  
```typescript
useEffect(() => {
	... выполняется после рендера
	return () => {
		...выполняется при размонтировании или перед новым рендером
	}
}, [])
```

Strict mode - дважды запускает рендер, что позволяет обнаружить ошибку при повторном запуске Эффекта, если не определена функция очистки  
Функция очистки должна быть вызвана для закрытия соединения с сервером, остановки таймеров, отписки от событий, установки анимаций в начальную фазу  
Например:  
```typescript
const [human, setHuman] = useState('Petr')
const [age, setAge] = useState(0)
useEffect(() => {
	let ignore = false
	fetchHuman(human).then((result) => {
		if(!ignore) setAge(result)
	})
	{/* Для каждого вызова fetch имеет собственную переменную ignore, которая блокирует запись состояния age, если изменилось состояние human */}
	return () => {ignore = true}
}, [human])
```

**Важно помнить, что useEffect срабатывает ПОСЛЕ рендера компонента**, поэтому первый рендер произойдет со старым состоянием  
Также не стоит создавать цепочки из useEffect изменяющие части состояние на основе других состояний - это приводит к лишним ререндерам  

[Вернуться к содержанию](#содержание)

## 4.7 useEffectEvent

[То же что в документации(видео)](https://www.youtube.com/watch?v=NZJUEzn10FI)  
`useEffectEvent(callback)` - разрывает реактивность useEffect от изменяющихся зависимостей, которые не должны приводить к ререндеру
На данный момент является экспериментальным хуком, поэтому требуется переключить `react: 'experimental'` в package.json  
```typescript
import { experimental_useEffectEvent as useEffectEvent } from 'react';
function MyComponent ({url}) {
	const {person} = useContext(PersonContent)
	const age = person.age
	{/* переменная age может изменяться, но не должна приводить к новому запуску useEffect и ререндеру */}
	const onChange = useEffectEvent((changedUrl: string) => {
		customFunction(changedUrl, age)
	})
	{/* оставляем реактивность url в зависимостях, чтобы useEffect реагировал на его изменение */}
	useEffect(() => {
		onChange(url)
	}, [url])
} 
``` 
[Вернуться к содержанию](#содержание)

## 4.8 useLayoutEffect

[Подробный разбор(видео)](https://www.youtube.com/watch?v=9GAt97z8Jc4)  
[Еще один(видео)](https://www.youtube.com/watch?v=wU57kvYOxT4)  
`useLayoutEffect(callback, deps?)` - аналогичен useEffect, НО срабатывает после всех изменений DOM и до того как компонент будет отрисован
Блокирует отрисовку элементов, поэтому не рекомендуется использовать длительные операции  
Используется для изменения расположения или внешнего вида элемента до отрисовки  
* callback - функция, выполняющая до рендера компонента
* deps - зависимости в формате [dep1, dep2, dep3], изменение которых ведет к вызову callback  
```typescript
useLayoutEffect(() => {
	const {height, width} = ref.current.getBoundingClientRect()
	setSize({height, width})
}, [])
```

useLayoutEffect не работает при SSR, так как при серверном рендере не существует макета  
Решение: 
* использовать useEffect
* использовать Suspense и показывать фолбэк, не использующий useLayoutEffect
* проверять состояние isMounted и отображать фолбек, пока не прозойет гидратация  
	(_гидратация - добавление недостающих элементов в код, который был сгенерирован в процессе SSR_)  

[Вернуться к содержанию](#содержание)
	
## 4.9 useInsertionEffect 

Срабатывает до монтирования элементов. useInsertionEffect > useLayoutEffect > useEffect  
Используется при работе CSSinJS  

[Вернуться к содержанию](#содержание)

## 4.10 useMemo

[Подробный разбор(видео)](https://www.youtube.com/watch?v=vpE9I_eqHdM)  
[Разбор документации по шагам с примерами(видео)](https://www.youtube.com/watch?v=oMvW3A_IRsY)  
`useMemo(calculateValue, deps)` - мемоизирует (кэширует) **результат** вычислений. Повторные вычисления производятся при изменении одной из зависимостей хука
* calculateValue - чистая функция, которая должна возвращать результат вычислений
* deps - зависимости в формате [dep1, dep2, dep3], изменение которых ведет к новому вычислению calculateValue   

Если объект/массив передается как props, то их нужно мемоизировать. Ссылка на них сохранится - не будет лишних ререндеров (т.к. `{} !== {}`)  
Мемоизация объекта: `useMemo(() => {name: 'Petr'})`  
При мемоизации функции сохраняется **результат** ее вычислений  
useMemo стоит применять, когда вычисления занимают длительное время. Проверить можно включив CPU Throttling в DevTools и:  
```typescript
console.time('start');
... вычисления
console.timeEnd('end');
```
Не нужно мемоизировать все подряд. Это трата памяти на сохранение всех результатов мемоизации  

[Вернуться к содержанию](#содержание)

## 4.11 React.memo

`React.memo(component[, fn])` - мемоизирует компонент
* component - компонент для мемоизации  
* fn - функция сравнения пропсов, передаваемых в component, для принятия решения о ререндере
Применяется, когда дочерние компоненты не изменяются при изменении состояния родителя (дочерний компонент не будет повторно отрендерен)  
```typescript
import {memo} from "react";

const MyComponent = memo(({ data }) => (
	<div>{data}</div>
)); 
```

Аналог memo с 1 аргументом в классовом компоненте - PureComponent: `class MyComponent extends PureComponent {}`  
Альтернативно в обычном классовом компоненте можно кастомизировать shouldComponentUpdate, который автоматически вызвается в PureComponent (см 1.2)  
shouldComponentUpdate должен вернуть boolean, которое определит нужен ли ререндер компоненту
```typescript
type Props = {propsData: string};
type State = {propsData: string};
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

## 4.12 useCallback

[Качественный разбор(видео)](https://www.youtube.com/watch?v=MxIPQZ64x0I)  
[Еще один(видео)](https://www.youtube.com/watch?v=_AyFP5s69N4)  
`useCallback(fn, deps)` - мемоизирует ссылку на функцию, между ререндерами, пока не изменились зависимости  
* fn - значение функции для мемоизации. React возвращает функцию, НЕ вызвает ее.
* deps - зависимости в формате [dep1, dep2, dep3] 
Используется в связке с `memo`  
При передаче в `memo` функций НЕ обернутых в `useCallback` мемоизаций компонента не произойдет, т.к. функия - ссылочный тип  

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
В отличие от `useMemo` - `useCallback` мемоизирует саму функцию, а не результат ее вычислений. 
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

**Важно помнить**, что функция запоминает свое состояние:
```typescript
const [dataArray, setDataArray] = useState([13,4,11,2,20])
const searchFunc = useCallback((count: number) => {
	{/* зависимости в useCallback не указаны, поэтому функция запомнила свое состояиние */}
	{/* консоль ВСЕГДА будет показывать 0 элемент (13), даже если массив будет отфильтрован */}
	console.log(dataArray[0])
	const filterFunc = () => {...}
}, [])
```

Зависимостей в useCallback должно быть как можно меньше.  
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
2. Создание функции можно перености в useEffect для уменьшения зависимостей:
```typescript
const myFunc = useCallback(() => {}, [deps])
useEffect(() => {myFunc()}, [myFunc])
// устраним лишние зависимости, добавив функцию в useEffect
useEffect(() => {
	const myFunc = () => {}
	myFunc()
}, [myFunc])
```
3. Все функции в кастомном хуке оборачиваются в useCallback, для дальнейшей оптимизации при его использовании  

[Вернуться к содержанию](#содержание)

## 4.13 useDeferredValue

[хороший разбор(видео)](https://www.youtube.com/watch?v=yIpHTYo3PY0)  
[еще один(видео)](https://www.youtube.com/watch?v=jCGMedd6IWA)  
`useDefferedValue(value)` - позволяет отложить обновление части интерфейса  
* value - значение, которое требуется отложить (примитивный типа или объект, созданный вне рендеринга)
При обновлении, React начнет рендер со старым значением, затем будет выполняться повторный рендеринг в фоновом режиме с новым значением  
Любое изменение значение прерывает фоновый рендеринг. Фоновый рендеринг начнется заново после того как значение перестанет изменяться  

Используется при отложенном обновлении результатов. Пользователь видит предыдущие результаты, пока не будут готовы новые  
Например, тяжелые вычисления могут блокировать взаимодействие с остальным интерфейсом. Решение: они помещаются в useDeferredValue для ленивой загрузки  
Для этого `useDefferedValue` комбинируется с `memo`
```typescript
//ParentComponent
const [text, setText] = useState('');
const deferredText = useDeferredValue(text);

return (
	{/* использование useDefferedValue не блокирует отрисовку и ввод текста в input */}
	<input value={text} onChange={(e) => setText(e.target.value)}/>
	<SlowComponent text={deferredText} />
)

//ChildComponent
const SlowComponent = memo(({text}) => {
	...долгие вычисления

	return <p>{text}</p>;
})
```
Для улучшения наглядности интерфейса, устаревшие значения можно выделить стилями
`opacity: query !== deferredQuery ? 0.5 : 1,`

Замена классическим способам оптимизации JS: дебаунс и троттлинг  

[Вернуться к содержанию](#содержание)

## 4.14 useTransition

[Детальный разбор(видео)](https://www.youtube.com/watch?v=1xjSQJWejZM)  
[Еще один(видео)](https://www.youtube.com/watch?v=N5R6NL3UE7I)  
`const [isPending, startTranstion] = useTransition()` - позволяет обновить состояние без блокировки интерфейса  
* isPending - boolean флаг, сообщающий статус перехода
* startTranstion - синхронная функция, помечающая обновление состояний(state), как переход  
Обновление состояний помеченных, как переходы, могут быть прерваны обновлениями обычных состояний
```typescript
const [state, setState] = useState()
const [text, setText] = useState()
const [isPending, startTransition] = useTransition();

function onClick(nextState) {
	startTransition(() => {
			...вычисления
			{/* состояние state помечено как переход */}
			setState(nextState);
	});
}
{/* ввод текста в input может прервать рендер по переходу */}
function onChange(e: ChangeEvent) => {
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
Обновления переходов **нельзя использовать для управления текстовыми выводами**  
Обработка ошибок через ErrorBoundary доступна **только** в экспериментальном режиме

**Отличие от useDefferedValue** - оборачивает состояние, useTransition оборачивает сеттер состояния  

[Вернуться к содержанию](#содержание)

## 4.15 useSyncExternalStore

[Rороткий пример как в документации(видео)](https://www.youtube.com/watch?v=eUJHCW7RBCI)  
[Еще пример(видео)](https://www.youtube.com/watch?v=dtS98IHP7xc)  
`useSyncExternalStore(onStoreChange, getSnapshot, getServerSnapshot?)` - подписка на внешнее хранилище (сторонние библиотеки за пределами React, API браузера).  
При изменении данных во внешнем хранилище происходит ререндер
* onStoreChange(callback) - функция подписки, которая должна возвращать функцию отписки, где callback - вызывается при изменении хранилища
* getSnapshot - функция возвращающая состояние (для клиента)
* getServerSnapshot - функция возвращающая состояние (для сервера SSR)  
`getServerSnapshot` должен возвращать те же данные, что и при первоначальном рендере на клиенте  

```typescript
const subscribe = (listener: () => void) => {
	window.addEventListener('resize', listener);
	return () => {
			window.removeEventListener('resize', listener);
	};
}

const UseSyncExample = () => {
	const innerWidth = useSyncExternalStore(subscribe, () => window.innerWidth)
	return <p>{innerWidth > 350 ? 'Yes' : 'No'}</p>
}
```
[Вернуться к содержанию](#содержание)

## 4.16 use

[Отличный ролик](https://www.youtube.com/watch?v=zdNF9FJWJ8o)  

`use(Promise_or_Context)` - **экспериментальный** хук, позволяющий прочитать значение Promise или Context  
Отличие от остальных хуков: `use` **можно использовать внутри Циклов или Условий**  
Как и остальные хуки, `use` должен использоваться внутри другого компонента или кастомного хука  

### 4.16.1 Promise in use

В серверном компоненте отдается предпочтение `async/await` вместо `use`, т.к. `async/await` рендерит с того момента как вызван `await`, а `use` повторно рендерит компонент после разрешения Promise
При использовании Promise, `use` комбинируется с `<Suspense />` и `<ErrorBoundary>`
Вместо ErrorBoundary альтернативно можно воспользоваться `catch` в Promise, значение которого также возвращается из `use` 
```typescript
// ParentComponent
const fetchFromServer = fetch(url).then(res => res.json)
return (
	<ErrorBoundary fallback={<div>Error</div>}>
		<Suspense fallback={<div>Loading...</div>}>
			<MyComponent request={fetchFromServer}>
		<Suspense>
	</ErrorBoundary>
)

// ChildComponent
{/* ответ с Promise должен быть сериализуем! Объект или Массив. Функции НЕ сериализуемы */}
const data = use(request)
return <p>{data}<p>

//ErrorBoundary
type ErrorBoundaryProps =  {
  fallback: ReactNode;
  children: ReactNode;
}
type ErrorBoundaryState = {
  hasError: boolean;
  error: Error | null;
}
export class ErrorBoundary extends React.Component<ErrorBoundaryProps, ErrorBoundaryState> {
  state: ErrorBoundaryState  = {hasError: false, error: null}
	static getDerivedStateFromError(error: Error): ErrorBoundaryState {
		return {
			hasError: true,
			error
		}
	}
	
	render() {
		if(this.state.hasError) {
			return this.props.fallback
		}
		return this.props.children
	}
}
```
Вызов `use` приостанавливает компонент на время выполнения Promise. Пока не завершится Promise отображается fallback `<Suspense />`  
`<ErrorBoundary />` перехватывает возникшие ошибки.  

### 4.16.2 Context in use

`use(myContext)` аналог `useContext(myContext)`, но его можно использовать внутри условий/циклов  
`use` - ищет ближайший родительский `<Provider />`, не рассматривая провайдеров контекста в самом компоненте  

```typescript
if(isEnabled) {
	const data = use(myContext)
	return <p>{data}<p>
}
```
[Вернуться к содержанию](#содержание)

## 4.17 useId

[Разбор темы(видео)](https://www.youtube.com/watch?v=_vwCKV7f_eA)  
[Более подробный разбор(видео)](https://www.youtube.com/watch?v=GNVI9Pr_RKQ&t=777s)  
`useId()` - генерирует уникальные идентификаторы **(не используется для ключей списков)**  
`:R1:` - пример сгенерированного id. Его **невозможно** использовать в функциях поиска по DOM (querySelector) 
Особенности применения:  
* гарантирует одинаковые id при генерации на сервере и клиенте (в отличие от сторонних библиотек (uuid)), если рендер на сервере и клиенте не имеет отличий   
* подходит для создания уникальных id для связывания форм, label, aria- атрибутов (особенно при заранее неизвестном количестве форм)  
* уникальность id сохраняется в пределах одного React App  

[Вернуться к содержанию](#содержание)

## 4.18 useOptimistic

[Пример с обработкой ошибки(видео)](https://www.youtube.com/watch?v=PPOw-sDeoNw)  
[Пример без обработки ошибки(видео)](https://www.youtube.com/watch?v=M3mGY0pgFk0)  
`const [optimisticState, addOptimistic] = useOptimistic(state, updateFn)` - показывает другое (оптимистичное) состояние интерфейса во время асинхронного действия  
_Пока доступен в экспериментальной версии_  
* optimisticState - результирующее оптимистическое состояние (равно state, если действий не ожидается)
* addOptimistic(optimisticValue) - функция вызывающая `updateFn(state, optimisticValue)`
* state - состояние, которое возвращается изначально и когда никаких действий не ожидается
* updateFn(currentState, optimisticValue) - чистая функция (аргументы: текущее и оптимистичное состояние), 
	которая возвращает результат вычислений как оптимистическое состояние
```typescript
import {experimental_useOptimistic as useOptimistic} form 'react'
const formRef = useRef();
async function formAction(formData) {
	addOptimisticMessage(formData.get('message'));
	formRef.current.reset();
	{/* нужно предусмотреть отмену оптимистичных изменений */}
	try { await sendMessage(formData); }
	catch (error) {
		{/* логика получения не обновившихся данных */} 
	}
	
}
const [optimisticMessages, addOptimisticMessage] = useOptimistic(messages, (state, newMessage) => [
	...state,
	{
		text: newMessage,
		sending: true,
	},
}

return (
	<>
		{optimisticMessages.map((message, index) => (
			<div key={index}> {message.text}
				{!!message.sending && ( <small> Sending... </small> )}
			</div>
		))}
		<form action={formAction} ref={formRef}>
			<input />
		</form>
	</>
)
```	
[Вернуться к содержанию](#содержание)

## 4.19 useDebugValue

[хороший разбор(видео)](https://www.youtube.com/watch?v=pTF86K8JZBQ)  
`useDebugValue(value, format?)` - добавляет метку к пользовательскому хуку в DevTools  
* value - значение отображаемое в devtools (любой тип: массив/объект/строка и т.д.)
* format - функция форматирования с аргументом value  
`useDebugValue(date, (date) => date.toDateString())`
`useDebugValue` находится в теле компонента и вызывается при каждом рендере. Если value - отладочная функция с тяжелыми вычислениями, то ее можно передать в функцию форматирования  
Она будет запускаться только при открытом React DevTools  

[Вернуться к содержанию](#содержание)

## 4.20 Кастомные хуки

Кастомные хуки - начинаются с use и используют внутри базовые хуки. Если базовые хуки не используются, то именование соответствует именованию функций    
Кастомные, также как и встроенные хуки не должны использоваться внутри условных конструкций, циклов и других функциях  
Кастомные должны возвращать одно значение, массив (2 значения) или объект (>2 значений)  
_Каждый вызов кастомного хука не зависит от любого другого вызова того же хука_  

[Неплохие кастомные хуки обертки над useState](https://www.youtube.com/watch?v=3SB278SY73s)  
Cтоит обратить внимание на: 
* useSafeState - обновления состояния при fetch запросах, если не использовать AbortController  
* обертку над localStorage/sessionStorage - проверка доступности localStorage/sessionStorage и преобразования данных, т.к. в них хранится только string
[Кастомные хуки для оптимизации](https://www.youtube.com/watch?v=XOSgHVzHEV4)  
* useLatest - уменьшение количества рендеров при зависимости useCallback от нескольких state  
* useEvent - аналог useLatest, только для функций  
* useWindowEvent -  уменьшение количества рендеров при подписке на события  
[Еще подробнее про useLatest](https://www.youtube.com/watch?v=ILg1zhl92AI)

[Вернуться к содержанию](#содержание)

