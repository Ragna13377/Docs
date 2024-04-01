# Содержание
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

**Компоненты пишутся с Большой буквы** и возвращают JSX-код   
JSX - расширение синтаксиса JS, представляющее HTML-подобную разметку внутри JS  
**Отличия JSX от HTML**:  
1. Все элементы должны иметь закрывающийся тег, в том числе `<input/>`, `<img/>` и т.д.
2. Большинство вещей в записывается в CamelCase.  
Атрибуты записываются в CamelCase. *Исключение data- и aria- атрибуты* (любые с '-')  
Атрибут `class` устанавливается через атрибут `className`, т.к. `class` зарезервирован для создания  класса  
Атрибут `for` устанавливается через атрибут `htmlFor`  
3. Возвращается один корневой элемент: 
	* Если возвращается один элемент и он убирается в одну строчку, то  
	`return <div>...код</div>`
	* Если возвращается один элемент в многострочном коде, то в круглых скобках 
	```typescript
	return (
		<div>
			...код
		</div>
	)
	```
	* Если возвращается несколько элементов, то используется синтаксис `<Fragment></Fragment>` или `<></>`:
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
`<StrictMode></StrictMode>` - стресс-тест, который вызывает дополнительный цикл `setup+clenup` для поиска ошибок
Запускается только в dev-режиме. Цикл `setup->clenup->setup` в dev-режиме должен соответствовать setup в prod-режиме  

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
	function myFunc() { setValue( value + 1) }
	return (
		{/* JSX-код */}
		<button onClick={myFunc}>{value}</button>
	)
}
```
Примеры типизации функционального компонента как стрелочной функции (лучше использовать один вариант в пределах проекта):
1. `const MyComponent = ({prop1, prop2}: CusromPropsType)`, 
	где `CustomPropType` - тип, описывающий передаваемые пропсы  
2. `const MyComponent = ({prop1, prop2}: PropsWithChildren<CusromPropsType>)` - пересечение (Intersection) `CusromPropsType` и `children?: ReactNode | undefined` - устаревший в React 18
3. `const MyComponent: FC<Props> = (props) => { ... }` - устаревший метод  

[Вернуться к содержанию](#содержание)

## 1.3 Классовый компонент (устаревший)

![жизненный цикл классового компонента](https://pictures.s3.yandex.net/resources/Untitled_2_1706861897.png)  
```typescript
{/* если в компоненте не передаются пропсы, то передается пустой объект {} в дженерике */}
class ClassComponent extends React.Component<PropsType, StateType> {
	{/* props передаются в качестве аргументов конструктора */}
	constructor(props) {
		super(props)
		{/* состояние задается в конструкторе, в зарезервированном свойстве state */}
		this.state = {
			value: 0
		}
		this.myFunc = this.myFunc.bind(this) {/* методам необходимо присваивать контекст */}
	}
	{/* вместо хуков используются методы*/}
	{/* состояние не меняется напрямую, любые изменениея происходят через setState */}
	myFunc(){ this.setState({ value: this.state.value + 1 }) }
	
	{/* альтернативный способ создания метода (без привязки контекста в конструкторе) */}
	{/* в обоих случаях методы пренадлежат экземпляру класса, а не прототипу */}
	myFunc = () => { }
	
	// вывод компонента
	render() {
		return (
			{/* JSX-код */}
			<button onClick={this.myFunc}>{this.state.value}</button>
		)
	}
}

```
---
Для взаимодействия с жизненным циклом классового компонента можно использовать методы:
* componentDidMount() - вызывается после монтирования
* componentDidUpdate(prevProps, prevState) - вызывается после обновления внутреннего состояния или пропсов. **Нельзя вызвать при монтировании** Можно работать с реальным DOM-деревом
	prevProps - предыдущие пропсы, prevState - предыдущее состояние  
	Для использования setState внутри метода необходимо обернуть его в условие, чтобы не возник бесконечный цикл
* shouldComponentUpdate(nextProps, nextState) - сравнивает текущие значения props и/или state и решает нужен ли повторный рендеринг (boolean). Используется для оптимизации мест с частым рендерингом    
	вернул true - при изменении props и/или state происходит повторный рендеринг
	вернул false - при изменении props и/или state повторный рендеринг не произойдет
* componentWillUnmount() - вызывается перед размонтированием для устранения утечек памяти (удаление таймеров, отписка от событий, закрытие соединения с сервером)
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

**В функциональных компонентах** аналог - `React.memo(Component, comparator?: (prevProps, nextProps) => {}])`  
Опциональная функция компаратор позволяет указать более детальную проверку ререндера. Работает также как и явное использование `shouldComponentUpdate`  
При отсутствии `React.memo` передача в setState неизменившегося состояния `setState(state)` **НЕ будет** приводить к ререндеру  

[Вернуться к содержанию](#содержание)

# 2. Props

```typescript
<CustomComponent attr1={value} attr2={{value: 10}} attr3="text">Some Text</CustomComponent>
```
Передавать props в компонент можно: 
* между тегами (в `props.children`) `<MyComponent><AnotherComponent /></MyComponent>`  
Приоритет в пробрасывании `props.children` между тегами компонента - выше, чем при указании аналогичного атрибута `children:{...}`
* через одинарные `{}` - переменные `<MyComponent loadintState={isLoading} />`
* через `{{}}` - объекты `<MyComponent personData={{name: 'Petr', age: 18}} />`
* через `""` - текст `<MyComponent title="Some Text"/>`

**Компонент ререндерится при изменнеии переданных props.**  
Деструктуризируя `...props`, можно выделить необходимые для передачи в дочерний компонент propsы
```typescript
const ParentComponent = ({children, ...props}) => {
	return (
		<ChildCompoent key={props.id}>{children}<ChildComponent>
		{/* установка props без присваивания отдельным арибутам */}
		<AnotherComponent {...props} />
	)
}
```

props передаются от родительского компонента к дочернему. Для передачи данных в обратном направлении нужно передать **функцию обратного вызова** в дочерний компонент  
**Важно** передавать функцию `{myFunc}`, а не ее вызов `{myFunc()}` *(неправильный вариант)*
```typescript
funtcion App() {
	const myFunc(){...логика...}
	return (
		<MyComponent getValue={myFunc} />
	)
}
```
## Пример 1

Функцию обратного вызова можно реализовать в родительском компоненте, а в дочернем передавать в нее параметры  
```typescript
funtcion App() {
	return (
		<MyComponent getValue={(param1, param2) => {...логика функции}}></MyComponent>
	)
}

const MyComponent = ({getValue}) => {
	{getValue(param1, param2)}
	...остальная логика компонента
}
```

## Пример 2 
Через функцию обратного вызова можно возвращать children:
```typescript
function App() {
	return (
		<div>
			<MyComponent>
				{/*функция обратного вызова*/}
				{(count, text) => (
						<form>
							<label>{text}</label>
							<p>{count}</p>
						</form>
					)
				}
			</MyComponent>
		</div>
	)
}

type Props = {
  children: (count: number, text: string) => JSX.Element
}
function MyComponent({children}: Props) {
	const text = ''
	const count = 42
	return children(count, text)
}
```

*Частая проблема* - большой уровень вложенности  
props передаются более чем на 2 уровня вложенности, не используясь в промежуточных компонентах (Props Drilling)  
Можно реорганизовать компоненты и поднять их на уровень выше  
Необходимые props можно будет использовать сразу на вложенных компонентах, без передачи через промежуточные компоненты  
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

При создании списков необходимо задавать атрибут `key` с уникальным значением для каждого итерируемого элемента(id, индекс в массиве map)  
Индекс итерации не рекомендуется использовать, так как элементы могут добавляться/удаляться, что изменит их ключ.  
`Crypto.randomUUID()` или npm пакет uuid создают уникальные id ключи
```typescript
function App() {
	const [array, setArray] = useState([{id: 1}, {id: 2}]) // массив элементов
	return (
		<div className="my_class">
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
`const [state, setState] = useState(defaultValue)` - управляет состоянием компонента  
При изменении состояния компонент и все дочерние компоненты ререндерятся, состояние сохраняется между ререндерами  
Используется для хранения локального состояния компонента, которое не передается в большое количество других компонентов (риск Prop Drilling)  
* state - текущее состояние
* setState - функция сеттер для изменения состояния
* defaultValue - начальное значение состояния (можно использовать функцию)  
`const [state, setState] = useState(() => setInitial())` - выполнится  только при первом рендере  
`const [state, setState] = useState(setInitial())` - выполнится при **КАЖДОМ** новом рендере  

Напрямую изменять (mutate) состояние нельзя, поэтому любые *мутирующие* методы (sort, reverse и т.д.) должны вызываться на копии  
Для объектов - `setState(...state, stateArg: newValue)` или для массивов - `setState([...state, newValue])`  
[Проверка методов на мутации](https://doesitmutate.xyz/)  

Спред-синтаксис действует поверхностно  
Изменение состояния вложенных полей (при большом уровне вложенности) увеличивает и дублирует код  
Например: 
```typescript
setState(...state, innerObj: {...state.innerObj, innerArg: newValue})
```
**Важно** не забывать, что объект - ссылочный тип:  
```typescript
const [state, setState] = useState([{innerArg: 1}, {innerArg: 2}])
function handleClick() {
	const newValue = [...state]
	{/* ! ОШИБКА ! Происходит мутация ссылочного типа (объекта) */}
	newValue.innerArg = 2 
	{/* Правильный вариант: */}
	state.map((item) => {
		if(...условие) return {...item, innerArg: 10}
		else return item
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
	то таймер с 0 задержкой вызовет собственный ререндер
	
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
	{/* таймер с нулевой задержкой вызвает собственный ререндер */}
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

Отличие от useDefferedValue - оборачивает состояние, useTransition оборачивает сеттер состояния  

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

`use(Mycontext)` аналог `useContext(myContext)`, но его можно использовать внутри условий/циклов  
`use` - ишет ближайший родительский `<Provider />`, не рассматривая провайдеров контекста в самом компоненте  

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
Кастомные должны возвращать одно значение, массив (2 значения) или объект ( >2 значений)  
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

# 5 Работа с формами и событиями

`SyntheticEvent` - обертка над стандартным событием `Event` со всеми его свойствами, исключающая различия в поведении в разных браузерах  
`SyntheticEvent.nativeEvent` содержит все методы стандартного события  
События в React регистрируются в фазу всплытия (bubbling). Для регистрации в фазу (capturing) к событию нужно добавить слово Capture: `onClickCapture`
Типизация события в React: ChangeEvent<TSomeType>

Компоненты бывают 2х видов: управляемые (с обратным связыванием - через useState) и неуправляемые (без обратного связывания - через useRef)
`<input type="file">` - всегда неуправляемый компонент.  
`<select>` - состояние изменяется через атрибут value на самом `<select>`, а не на дочерних `<option>`  
`<input type="radio">` - для проверки, в атрибуте checked требуется использовать результат установленного значения со значением на элементе
```typescript
const [mode, setMode] = useState('dark');
 const onValueChange = (e: ChangeEvent<HTMLInputElement>) => {
    setMode(e.target.value);
		{/*для чекбоксов  - setState(e.target.checked);*/}
  }
 <input type="radio" value="light" checked={mode === "light"} onChange={onValueChange}
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
		setState({
			...state,
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
Возвращаем функцию в которой будет доступна переменная `name` из родительской
```typescript
const MyComponent = () => {
	const [state, setState] = useState({checkbox: false, text: ''})
	const handler = 
			(name: string) => (event: ChangeEvent<HTMLInputElement>) => {
			const target = event.target;
			const value = target.type === "checkbox" ? target.checked : target.value
			setState({
				...state,
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

При работе с данными от сервера пришедшее значение может быть null или undefined, что принудительно изменит режим компонента на неуправляемый  
При получении значения от пользователя режим изменится на управляемый  
Чтобы избежать ошибки с изменением режима, нужно контролировать исходное значение: `{state.requestData ?? ""}`

# 6. Обработка ошибок. Error Boundary

[абзац про Error Boundary](https://habr.com/ru/articles/461541/)  
[пример использования npm react-error-boundary](https://www.youtube.com/watch?v=gyqAW0--0Tc)  
В отличие от `try...catch` механизм Error Boundary перехватывает ошибки в конструкторах дочерних компоненов, методах жизненного цикла и во время рендеринга  
**Ошибки не будут пойманы в обработчиках событий, асинхронном коде, SSR, в самом компоненте Error Boundary**  
Для отлова ошибок в обработчиках события также используется `try...catch`  
* Метод `getDerivedStateFromError(error)` вызывается после возниконовения ошибки у компонента-потомка на этапе рендеринга  
Принимает ошибку в параметре и возвращает значение для обновления состояния. Используется для рендеринга запасного варианта при ошибке  
* Метод `componentDidCatch(error, info)` вызывается после `getDerivedStateFromError`. Вызывается после рендера и во время применения сайд-эффектов. Используется для логирования ошибок  
	* error - ошибка
	* info - объект с ключом componentStack, содержащий информацию о компоненте, в котором произошла ошибка

```typescript
class ErrorBoundary extends React.Component {
    constructor(props) {
        super(props);
        this.state = { error: false };
    }
    static getDerivedStateFromError(error) {
        // Произошла ошибка!!!
        return {error: true};
    }
		componentDidCatch(error, info) {
        console.info(info.componentStack); // Вывод состояния стека и ошибки в консоль
        console.error(error);
        logComponentStackToMyService(info.componentStack); // Пользовательский метод обрабатывающий ошибку
    }
    render() {
        if(this.state.error) {
            return <h1>Что-то пошло не так.</h1>; // Отображение запасного UI
        }
        // Ошибки нет
            return {this.props.children}
    }
}
```
Необходимые компоненты оборачиваются в ErrorBoundary для отлова ошибок `<ErrorBoundary><MyComponent/></ErrorBoundary>`  
**В функциональных компонентах нет аналога**  

# 7. Переменные окружения в React

[Общая информация про переменные окружения](https://www.youtube.com/watch?v=HiRxC7WeNZU)  
[Подробно про создание переменных окружения](https://www.youtube.com/watch?v=wkfWaI_lI48) 
[Текстом про переменные окружения](https://danshin.ms/React-Environment-Variables/) 

В React все переменные окружения **ДОЛЖНЫ начинаются** с `REACT_APP`: `REACT_APP_SOME_NAME`  
Использование в файлах: `process.env.REACT_APP_API_KEY`

Основные методы: 
1. в настройках системы Windows (Изменение системных переменных сред)
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

# 8 HOC(High Order Component) устаревшая логика

Паттерн используемый во фреймворках для создания компонента высшего порядка, объединяющего логику компонентов схожей функциональности  
HOC-компонент получает аргументом исходный компонент и оборачивает его необходимыми свойствами и функционалом  
Принято называть HOC-компоненты с with: `withClick` и располагать в папке hocs на уровне с папкой components  
```typescript
{/* собственные пропсы оборачиваемого компонента */}
interface PersonalProps {
	isEnabled: boolean
}
{/* пропсы генерируемые HOC-компонентом дополняющие пропсы базового компонента */}
interface WrapperGeneratedProps {
	onButtonClick: () => void
	buttonText: string
}
{/* пропсы используемые только внутри HOC-компонента */}
interface WrapperProps {
	alertText: string
}

//withClick.tsx
function withClick(WrappedComponent: ComponentType<PersonalProps & WrapperGeneratedProps>): ComponentType<PersonalProps & WrapperProps>{
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

//Button.tsx
const Button = ({isEnabled, onButtonClick, buttonText}: PersonalProps & WrapperGeneratedProps) => {
	return <button disabled={isEnabled} onClick={onButtonClick}>{buttonText}</button>
}

//App.tsx
{/* передаем оборачиваемый компонент в HOC */}
const WithClickButton = withClick(Button)
const [isEnabled, setIsEnabled] = useState(false)
return (
	{/* передаем пропсы базового компонента и пропсы используемые только внутри HOC-компонента */}
	<WithClickButton isEnabled={isEnabled} alertText="Some Text"/>
)
```
[Дополнения по HOC](https://reactdev.ru/archive/react16/higher-order-components/#static-methods-must-be-copied-over)
[Видео без типизации] (https://www.youtube.com/watch?v=KWT8OKzrMZ4)

# 9. ReactDOM 

## 9.1 Portals

[Порталы на практике](https://www.youtube.com/watch?v=V4sHZzX4zh0)
`createPortal(children, domNode, key?)` - позволяет отрендерить дочерние элементы вне иерархии родительского компонента (другая часть DOM)  
Портал меняет только физическое расположение узла DOM. JSX, который помещается в портал действует как обычный дочерний узел компонента React (имеет доступ к состояниям родителя и т.д.)  
* children - все что может отобразить React: JSX, компонент, строка, число, массив, которые нужно отрендерить вне родителя
* domNode - HTML-элемент контейнер, в котором нужно отрендерить children
* key - опциональный ключ (см. # 3. Cписки) для портала

События от порталов распространяются в соответствии с деревом React, а не DOM  
Порталы часто используются для создания модальных окон и тултипов   

# 10. Типизация в React

[Замечательное видео по типизации](https://www.youtube.com/watch?v=TPACABQTHvM) 

* типизация children
	* React.ReactNode - общий тип для узлов React-дерева (ReactElement, ReactPortal, JSX, Iterable<ReactNode>, string, number, boolean, null, undefined)
	Возвращается классовым компонентом 
	* ReactElement - объект со свойствами type, props, key
	* ReactPortal - ReactElement со свойством children 
	* JSX.Element - ReactElement<any, any> (например `<div></div>`). Возвращается функциональным компонентом
[Разница между JSX.Element, ReactNode и ReactElement](https://stackoverflow.com/questions/58123398/when-to-use-jsx-element-vs-reactnode-vs-reactelement)  

```typescript
type MyComponentProps = {
	children: React.ReactNode
}
const MyComponent = ({children}: MyComponentProps) => {}

const App = () => {return <MyComponent>Some Text</MyComponent>}
```
* типизация сеттера useState - `React.Dispatch<React.SetStateAction<number>>` (подсказка в IDE при наведении на сеттер useState)
* типизация HTML-элемента  
	`React.ComponentProps<"img">` - доступны все свойства HTMLImageElement
	`React.ComponentPropsWithRef<"img">` - аналогично предыдущему + свойство ref
* типизация стилей CSS - `React.CSSProperties`
* типизация событий Event - `React.MouseEvent<HTMLButtonElement, MouseEvent>`  
	Для упрощения можно создать обработчик событий на компоненте, чтобы увидеть подсказку типа
* типизация useState - `const [state, setState] = useState<MyType | null>(null)`
* типизация useRef - `const ref = useRef<HTMLButtonElement>(null)`
* типизаця forwardRef - `React.forwardRef<HTMLButtonElement, MyProps>((props, ref) => {})`
* конкретизация типа - добавление `as const` позволяет задать тип с конкретными данными
	`const array = ['Petr', 'Olga'] as const` - тип массива НЕ string[], а массив с данными 'Petr', 'Olga'
* дженерик - аналог перегрузки функции (обобщает передаваемый тип значения, который указывается при вызове)
	`const handler = <T,>(value: T): T[] => {...}` - запятая в дженерике в стрелочной функции обязательна
	`function handler<T>(value: T): T[] {...}`
* типизация импорта - `import {type MyType} from ''`. При импорте типов указывается ключевое слово type
	
## 10.1 Utility Types

1. Record<string, number> - соответствует {'width': 100, "height": 500, minWidth: 50}
2. Omit<MyType, 'name'> - удаление из существующего типа(MyType) полей(name)

# 10.2 type vs interface 

[Отличие интерфейсов от типов. Отличный ролик](https://www.youtube.com/watch?v=Idf0zh9f3qQ)  

**Базовый пример:**
```typescript
type TProps = {
	name: string
	age: number
}

interface IProps{
	name: string
	age: number
}
```  
1. **Описание типов** - интерфейсы могут описывать только объекты, типы могут описывать любой тип данных  
```typescript
{/* переопределили примитивный тип string для большей наглядности */}
type Name = string
const guestName: Name = 'Petr'

{/* объект со свойством name */}
interface {
	name: string
}
```
2. **Union Types** - типы поддерживает Union types  
```typescript
type Name = string | string[]
const name: Name = ['Petr', 'Olga']
```
3. **Наследование свойств** - интерфейсы используют extends, типы &  
```typescript
type TProps = {
	name: string
}
type TExtendProps & TProps = {
	age: number
}

interface IProps{
	name: string
}
interface IExtendProps extends IProps {
	age: number
}
```
4. **Utility Types** - синтаксис в типах при использовании Utility Types короче и лаконичнее
```
type TProps = {
	name: string,
	age: number
}
type MyProps = Omit<TProps, 'name'>

interface MyProps extends Omit<TProps, 'name'> {}
```
5. **Кортежи(типизированный массив с конкретным количеством аргументов)** - типы проще описывают кортежи, чем интерфейсы
```typescript
type TProps = [string, number]

{/* синтаксис сложнее */}
interface IProps {
	0: string
	1: number
}
```
6. **Извлечение типов** - типы проще, чем интерфейсы поддерживают извлечение типов
```typescript
const props = {
	name: 'Petr',
	age: 21,
	country: {
		square: 1200,
		people: 1500
	}
}
{/* при изменении переменной изменять тип не нужно */}
type TProps = typeof props
type TCountry = typeof props['country']

{/* при изменении переменной придется изменять и интерфейсы */}
interface IProps {
	name: string
	age: number
}
interface ICountry {
	square: number
	people: number
}
```
7. **Объединение** - интерфейсы с одним названием объединяются, типы с одним названием вызовут ошибку
```typescript
interface IProps {
	name: string
}
/* Получаем общий интерфейс с двумя полями */
interface IProps {
	age: number
}

type TProps = {
	name: string
}
{/* Ошибка. Не может быть двух типов с одним названием */}
type TProps = {
	age: number
}
``` 

**Сводная таблица:**  
|   | interface  | type  |
|---|:----------:|------:|
| Наследование | через extends | через & |
| Описание | Только объекты | Любой тип данных |
| Union types | Не поддерживает | Поддерживает |
| Utility Types | Синтаксис хуже | Синтаксис лучше |
| Кортежи | Синтаксис сложнее | Синтаксис проще |
| Извлечение типов | требуется изменять | не требуется изменять |
| Объединение | можно объединять | нельзя объединять |

# 11 Проекция состояния класса

## 11.1 Класс с редко изменяемым состоянием

Собственное состояние класса - `todos`  
Метод `getList` возвращает состояние класса  
Методы `add`, `remove` изменяют состояние по условию (например, нажатие кнопки)
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
	{/* [1] - получаем интстанс */}
  const todos = useRef<ITodos>(new Todos(initialTodos));
	{/* [2] - сохраняем состояние класса state */}
  const buildTodosState = useCallback(() => [...todos.current.getList()], []);
  const [state, setState] = useState(buildTodosState);

  /* [3] - методы изменяющие состояние */
  const addTodo = useCallback(
    (todo: Todo) => {
      /* Вызываем методы класса */
      todos.current.add(todo);
      /* Обновляем state */
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

  {/* Возвращаем доступные методы вместе с состоянием */}
  return [state, { addTodo, removeTodo }] as const;
	{/* as const позволяет указать возвращаемое значение как кортеж */}
	{/* ограничивает и конкретизирует количество элементов */}
} 
``` 

Использование хука в компонентах  
Если функция передается в HTML-Element, то не нужно использовать useCallback  
Если функция передается в сам компонент, то:
```typescript
const MyComponent = () => {
	const [todos, { addTodo, removeTodo }] = useTodos();
	const onAdd = useCallback(() => {
    /* Пользуемся методами мутации */
    addTodo({ id: '123', title: 'Новое todo' });
  }, [addTodo]);
}
```

# 11.2 Класс с часто изменяемым состоянием

```typescript
class Timer {
  private timer?: number;
  private intervalTime: number;
	{/* передаваемый извне callback */}
  private callback?: () => void;
	{/* часто меняющееся состояние*/}
  time: number;

  constructor(intervalTime: number = 1000) {
    this.timer = undefined;
		this.intervalTime = intervalTime;
    this.callback = undefined;
    this.time = 0;

    this.updateTime();
  }

    /* передаем callback снаружи */
  onChange(callback: () => void) {
    this.callback = callback;
  }

  updateTime() {
    this.time = Date.now();
		{/* Вызываем callback на каждое обновление */}
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
  {/* [1] - получаем интстанс */}
  const ref = useRef(new Timer());
	{/* состояние useReducer используется для ререндер компонента */}
	{/* это число, увеличивающееся при каждом вызове forceUpdate (в примере изменении таймера) */}
	const [_, forceUpdate] = useReducer(v => v + 1, 0);

  useEffect(() => {
    const timer = ref.current;
    timer.start();
		{/* [2] - передаем useReducer в callback */}
		timer.onChange(forceUpdate);
    return () => {
      timer.stop();
    };
  }, []);

	/* Состояние класса отдаём наружу */
  return ref.current.time;
}
```

Использование хука в компонентах:
```typescript
function TimerUI() {
  const time = useTimer();
} 
```

# 11.3 Класс без собственных состояний

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
{/* [1] - создаем контекст */}
export const ApiContext = createContext<ApiInterface>({
    getPosts: () => Promise.resolve([]),
}); 
```

Использование в компонентах:
```typescript
{/* [2] - получаем инстанс */}
const apiRef = useRef(new Api());
<ApiContext.Provider value={apiRef.current}></ApiContext.Provider>


const api = useContext(ApiContext);
useEffect(() => {
	api.getPosts().then();
}, [api]);
```

# 12 Redux

[Redux DevTools](https://github.com/reduxjs/redux-devtools)  
`npm i react-router-dom` - установка  
`npm i -D @types/react-router@next @types/react-router-dom@next` - типизация  
Настройка Redux начинается с создания store(хранилища)
Обязательный ключ - reducer, необязательные middleware и enhancers  
0. Action (действие):
	* Объект с обязательным полем `type`, описывающим тип производимого действия, и необязательными данными (чаще в поле `payload`)
	* передает данные для взаимодействия с хранилищем от компонента как аргумент `store.dispatch`
1. Reducer (редьюсер): 
	* через `store.dispatch` в reducer передается action и состояние хранилища
	* редьюсер создает новый объект хранилища и возвращает его
	* новый объект заменяет старый (иммутабельность)
	* вызываются функции обработчики, установленные через `store.subscribe`
2. Middleware (посредник):
	* получает action до его попадения в reducer
	* выполняет сторонний эффект
	* при необходимости передает action по цепочке дальше
3. Enhancers (усилитель):
	* добавляет или изменяет функциональность хранилища

- reducer - корневой reducer, объединяющий другие reducerы различной логики (логирования, аутентификации и т.д.)
- middleware - функция с аргументом getDefaultMiddleware (мидлвары по умолчанию), к которому добавляем собственные middlewarы  
	getDefaultMiddleware:
	1. middleware проверяющий иммутабельность хранилища (ошибки связанные с изменением хранилища напрямую)
	2. middleware проверяющий сериализуемость данных (ошибки при сохранении функций, промисов и т.д. в хранилище)
	3. middleware проверяющий попытку передать генератор экшенов вместо экшена
	4. middleware (redux-thunk) позволяющий использовать асинхронные экшены
- enhancers - функция с аргументом getDefaultEnhancers (усилители по умолчанию), к которому добавляем собственные enhancerы  
	getDefaultEnhancers:
	1. enhancer (applyMiddleware) модифицирующий `store.dispatch` для работы с middleware
	2. enhancer подключающий Redux DevTools
	3. enhancer (autoBatchEnhancer) улучшающий производительность при работе с RTK Query

```typescript
import { configureStore } from "@reduxjs/toolkit";
import { offline } from '@redux-offline/redux-offline'
import offlineConfig from '@redux-offline/redux-offline/lib/defaults'

import { reducer1 } from "./feature1/reducer1";
import { reducer2 } from "./feature2/reducer2";
import { middleware1 } from "./middlewares/middleware1";
import { middleware2 } from "./middlewares/middleware2";

export const store = configureStore({
	reducer: {
		feature1: reducer1,
		feature2: reducer2,
		auth: authReducer,
		products: productsReducer,
	},
	middleware: (getDefaultMiddleware) => {
			return getDefaultMiddleware().concat(middleware1, middleware2);
  },
  enhancers: (getDefaultEnhancers) => {
        return getDefaultEnhancers().concat(offline(offlineConfig)),
  }
}); 
```
Подключение store к React (все компоненты имеют доступ к store):
```typescript
import {Provider} from 'react-redux'
root.render(
	<Provider store={store}>
		<App />
	</Provider>
)
```
Типизация хуков redux:  
```typescript
//store
import {
	useDispatch as dispatchHook,
	useSelector as selectorHook
} from 'react-redux'
import type { TypedUseSelectorHook } from 'react-redux'

{/* Типизация диспетчера действий (dispatch) */}
type AppDispatch = typeof store.dispatch
{/* Типизация состояния store */}
export type RootState = ReturnType<typeof store.getState>;
{/* определяем какие action мы можем отправить к store и какой тип данных приходит из store */}
const useDispatch: () => AppDispatch = dispatchHook;
const useSelector: TypedUseSelectorHook<RootState> = selectorHook; 
```
Использование типизированных хуков в компонентах:
```typescript
const { someData } = useSelector(store => store.feature1);
const dispatch = useDispatch();

{/* IDE webStorm выдает ошибку аргументов. Передавать нужно сразу payload */}
dispatch(someActionCreator());
```

## 12.1 Создание Action и reducer

### 12.1.1 Классический способ (без Redux Toolkit):

**Создание action**:
1. Определяем константы для описания типов(поле type) action
```typescript
const ADD_BOOK = "ADD_BOOK";
const REMOVE_BOOK = "REMOVE_BOOK"; 
```
2. Типизируем action
```typescript
type TId = {
	id: number
}
type TBook = {
  id: TId;
	title: string;
	author: string;
};

type TAddBookAction = {
	type: typeof ADD_BOOK;
  book: TBook;
};

type TRemoveBookAction = {
	type: typeof REMOVE_BOOK,
  id: TId
} 
```
3. Создаем генератор экшенов (убирает лишнюю реализацию action из компонентов)
```typescript
const addBook = (book: TBook): TAddBookAction => ({
    type: ADD_BOOK,
		book
});

const removeBook = (id: TId): TRemoveBook => ({
    type: REMOBE_BOOK,
    id
}); 
```
**Создание reducer**:
1. Создаем тип данных с которым работает reducer
```typescript
type TBooksState = {
    books: Array<TBook>;
}; 
```
2. Создаем начальное состояние данных reducer
```typescript 
const initialState: TBooksState = {
    books: [];
}; 
```
3. Создаем reducer
```typescript
const bookReducer = (
    state = initialState,
    action: TAddBookAction | TRemoveBookAction
): TBooksState = {
    switch (action.type) {
		case ADD_BOOK:
				return { ...state, books: [...state.books, action.book] };
    case REMOVE_BOOK:
				return { ...state, books: state.books.filter(b => b.id !== action.id) };
    default:
				return state;
    };
} 
```

**Основные правила при написании reducer**  
* ВСЕГДА указывать тип возвращаемого значения reducer, чтобы ошибка не просочилась дальше
* ВСЕГДА формировать новое состояние на основе старого через спред-синтаксис (НЕЛЬЗЯ мутировать состояние)
* ВСЕГДА должен быть default, возвращающий текущее состояние
* reducer - чистая функция и не может использовать глобальные переменные или нечистые функции. ТОЛЬКО переданные в него аргументы

### 12.1.2 Способ с помощью Redux Toolkit:

_Redux дока советует использовать данный подход вместо классического_  
Под капотом используется Immer  

**Создание action**:
```typescript
type TBook = {
  id: number;
	title: string;
	author: string;
};
const addBook = createAction<TBook, 'ADD_BOOK'>('ADD_BOOK');
const removeBook = createAction<string, 'REMOVE_BOOK'>('REMOVE_BOOK'); 
```
* Первый параметр типа - тип данных, передаваемый в action в поле Payload
* Второй параметр типа - значения поля type
* Аргумент - значение поля type

**Создание reducer**:
```typescript
type TBooksState = {
   books: Array<TBook>;
};

const initialState: TBooksState = {
   books: [];
};

export const liveTableReducer = createReducer(initialState, (builder) => {
	builder
		.addCase(addBook, (state, action) => {
				state.books.push(action.payload);
		})
		.addCase(removeBook, (state, action) => {
				state.books = state.books.filter(b => b.id !== action.id);
		})
}); 
```
* initialState - начальное состояние reducer  
* стрелочная функция с аргументом builder - описывает действия производимые с store при получении action  
* каждый метод builderа принимает в аргументах: генератор actionа и функкцию-reducer для обработки

### 12.1.3 Использование Slice

Функция `createSlice` принимает параметры:
* name - уникальное название Slice**
* initialState - начальное состояние store
* reducers - reducerы actionов (тип передаваемых actionом данных указывается параметром в обобщенном типе PayloadAction
После создания слайсов автоматически создадутся генераторы actionов с названиями - ключи объекта reducers

```typescript
type TBooksState = {
    books: Array<TBook>;
};

const initialState: TBooksState = {
    books: []
};

const bookSlice = createSlice({
	name: 'book',
  initialState,
  reducers: {
		addBook: (state, action: PayloadAction<TBook>) => {
			state.books.push(action.payload);
    },
    removeBook: (state, action: PayloadAction<string> => {
			state.books = state.books.filter(b => b.id !== action.id);
    }
  }
});

export const { addBook, removeBook } = bookSlice.actions;
{/* его добавляем в store */}
export const reducer = bookSlice.reducer; 
```
Чтобы кастомизировать генератор actionа в поле (в примере addBook) передается reducer и prepare,  
где reducer - редьюсер, а prepare предварительная функция обработки  
reducer - чистая функция, все обращения к глобальным переменным и изменение данных в prepare  
```typescript
type TBookWithKey = TBook & { key: string };

reducers: {
	addBook:{
		reducer: (state, action: PayloadAction<TBookWithKey>) => {
			state.books.push(action.payload);
		},
		prepare: (book: TBook) => {
			const key = nanoid();
			return { payload: {...book, key } };
		}
	}
},
```
После отправки, созданного через генератор actionа с данными book, он попадает в функцию prepare, которая передает их в reducer  

**Обработка внешних action в Slice**  

Генератор action создан отдельно от Slice (внешний):
```typescript
type TMoveParams = {
	from: number;
  to: number;
}
const moveBook = createAction<TMoveParams, 'MOVE_BOOK'>('MOVE_BOOK'), 
```
Внешний action подключается в поле extraReducers:
```typescript
//createSlice
reducers: {
	extraReducers: (builder) => {
    builder.addCase(moveBook, (state, action) => {
      state.books.splice(
        action.payload.to,
        0,
        state.books.splice(action.payload.from, 1)[0]
      );
    }
}
```

## 12.2 Selectors

Селекторы извлекают данные из store, получая в аргументе стрелочную функцию  
`const books = useSelector(store => store.library.books);`  
Проблемы: 
1. создание стрелочной функции в момент вызова хука может спровоцировать дополнительные ререндеры
2. компоненту доступна структура хранилища store. При изменении структуры придется изменять компоненты  

Решение проблемы в классическом способе: стрелочные функции выносятся в отдельный файл:
```typescript
export const getBooks = (state: RootState) => state.library.books; 

//использование в компоненте
const books = useSelector<RootState, TBooksState>(getBooks); 
```
Решение проблемы с использование Redux Toolkit: в reducers передается selectors со стрелочными функциями
```typescript
createSlice({
	selectors: {
		getBooks: (state) => state.books,
  },
})

export const { getBook } = bookSlice.selectors; 
```

**Мемоизированные селекторы**  
Используем `createSelector` для мемоизации вычислений:
```typescript
//selectors
import { createSelector } from '@reduxjs/toolkit';
export const getBooksById = (id: string) =>
  createSelector(
		{/* state соответствует общему состояние хранилища */}
		{/* выбираем конкретный state указывая нужный reducer */}
    (state: RootState) => state.bookReducer.books,
    (items) => (items ? items.filter((item) => item.id === id) : [])
  );
```

## 12.3 Middleware

Action, которые используются только в middleware и не будут переданы в reducer создаются через `createAction`    
**Их нельзя определить в slice**  
```typescript
export const startTicker = createAction('START_TICKER');
export const stopTicker = createAction('STOP_TICKER'); 
```
Для типизации middleware необходимо знать тип store, для этого создаем корневой reducer:
```typescript
const rootReducer = combineReducers({
    tickerSlice: tickerReducer;
});
export const RootState = ReturnType<typeof rootReducer>; 
```
Создем генератор action для middleware и сам middleware, где  
`(store) => (next) => (action) => {...код middleware}`  
Полученный action передается в следующий middleware или reducer через next
```typescript
{/* типизация Middleware - первый параметр unknown, второй тип данных store */}
export const tickerMiddleware = (): Middleware<unknown, RootState> => {
	return (store) => {
		let timer = 0;
		return (next) => (action) => {
			{/* сравнение полученного action по типу(type) */}
			if (startTicker.match(action)) {
				timer = setInterval(
					() => store.dispatch(updateTicker()), 
					1000
			);
			{/* сравнение полученного action по типу(type) */}
			} else if (stopTicker.match(action)) {
					clearInterval(timer);
			} else {
					next(action);
			}
		}
	}
} 
```
Подключение к store:
```typescript
const store = configureStore({
	reducer: rootReducer,
	middleware: (getDefaultMiddleware) => {
		return getDefaultMiddleware().concat(tickerMiddleware());
	}
}); 
```

## 12.4 Создание асинхронного action

Асинхронные action создаются с помощью `createAsyncThunk`, где 
* первый аргумент - уникальное строковое значение на основе которого формируются типы трех экшенов (ниже)
* второй аргумент асинхронная функция, в которую можно передать данные
```typescript
export const getBooks = createAsyncThunk(
	"books/getBooks",
	async (id: string) => {
		return api.getBooks(id);
	},
); 
```
При обработке асинхронного action может быть отправлено в reducer 3 экшена:
* getBooks.pending - до вызова асинхронной функции
* getBooks.fulfilled - после успешного завершения асинхронной функции с payload - результатом асинхронхронной функции
* getBooks.rejected - после неуспешного завершения асинхронной функции с error - результатом ошибки в формате:
```typescript
 interface SerializedError {
	name?: string
	message?: string
	code?: string
	stack?: string
}
```
Slice для работы с асинхронным action
```typescript
	type TBooksState = {
	books: Array<TBook>;
  loading: boolean;
  error: string | null;
};

const initialState: TBooksState = {
	books: [],
  loading: false,
	error: null
};

const bookSlice = createSlice({
	name: "book",
  initialState,
	reducers: {},
  selectors: {
		getBooksSelector: (state) => state, 
  }, 
  extraReducers: (builder) => {
		builder
			.addCase(getBooks.pending, (state) => {
				state.loading = true;
				state.error = null;
			})
			.addCase(getBooks.rejected, (state, action) => {
				state.loading = false;
				state.error = action.error.message;
			})
			.addCase(getBooks.fulfilled, (state, action) => {
				state.loading = false;
				state.books = action.payload;    
			}
  }
});
export { getBookSelector } = bookSlice.selectors; 
```

# 13. Готовый код Store и slice

```typescript
//store
import { configureStore } from '@reduxjs/toolkit'
import musicSliceReducer from '../slices/musicSlice';
import { AppDispatch, useSelector } from "../store/store";
import { TypedUseSelectorHook, useSelector as selectorHook, useDispatch as dispatchHook } from "react-redux";
export const store = configureStore({
    reducer: {
        music: musicSliceReducer
    }
		{/* опциональны */}
		middleware: {}
		enhancers: {}
});
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch
export const useDispatch: () => AppDispatch = dispatchHook;
export useSelector: TypedUseSelectorHook<RootState> = selectorHook;

//slice
import {createSlice, PayloadAction} from "@reduxjs/toolkit";
import {musicModel} from '../models/Music';
import {music} from '../data/music';

interface musicListState {
    music: musicModel[],
		isLoading: boolean
}

const initialState: musicListState = {
    music,
		isLoading: false
}
{/* асинхронное действие */}
export const fetchMusic = createAsyncThunk(
  "music/getMusic",
  async() => {
		{/* асинхронный запрос */}
    const music = await getMusic();
    return music;
  }
)

const musicSlice = createSlice({
    name: 'music',
    initialState,
    reducers: {
			toggleMusic: {
				reducer: (state, action: PayloadAction<number>) => {
					state.music.map(item => {
					if(item.id === action.payload)
						item.isEnabled = !item.isEnabled
					})
				}
				{/* предварительная функция обработки */}
				{/* для взаимодействия со сторонними библиотеками и глобальными функциями */}
				prepare: (book: TBook) => {
					const key = nanoid();
					return { payload: {...book, key } };
				}
			},
    },
    selectors: {
			selectMusic: (sliceState) => {
					return sliceState.music
			}
    },
		extraReducers: (builder) => {
      builder
        .addCase(fetchMusic.pending, (state) => {
          state.isLoading = true
        })
        .addCase(fetchMusic.fulfilled, (state, action) => {
          state.isLoading = false
          state.tracks = action.payload
        })
        .addCase(fetchMusic.rejected, (state) => {
          state.isLoading = false
        })
    }
})
export const {toggleMusic} =  musicSlice.actions
export const {selectMusic} = musicSlice.selectors
export default musicSlice.reducer;

//component
export const MusicBlock = ({id}: MusicBlockProps) => {
		const dispatch = useDispatch();
		{/* вызов асинхронново action */}
		useEffect(() => {
			dispatch(fetchMusic())
		}, [])
		const { music } = useSelector(store => store.music);
		const onClick = () {
			dispatch(toggleMusic(id))
		}
}
```

## 14 React Router

[React router](https://github.com/remix-run/react-router)

Маршрутизация в браузере  
`npm i react-router-dom`   
`npm i -D @types/react-router-dom`  

Виды роутеров:
* BrowserRouter - роутер, использующий hitstory API
* NatibeRouter - аналог на React Native
* HashRouter  - роутер, использующий hash(#) в URL  
	используется, когда провайдер не позволяет изменить настройку сервера для поддержки HTML5 HistoryAPI
* MemoryRouter - роутер, использующий память(адресная строка не меняется) 
	используется при тестировании
* StaticRouter - серверный роутер SSR

## 14.1 Старая конфигурация Router

Создание Router:
1. В папке pages создаются экраны страниц 
2. Router оборачивает все приложение
```typescript
//index
import {BrowserRouter} from "react-router-dom";

ReactDOM.createRoot(document.getElementById("root")).render(
  <StrictMode>
    <BrowserRouter>
			<App />
		</BrowserRouter>
  </StrictMode>
);
```
3. Routes оборачивает дочерние Route и выбирает подходящий текущему URL  
Route - привязывает path в URL к компоненту (экрану)  
_Порядок экранок не важен_. Вложенные маршруты группируются.  
```typescript
//app
import { BrowserRouter, Routes, Route } from 'react-router-dom';
{/* импорт экранов */}
import { ProfilePage, ProfileForm, ProfileListPage, HomePage, NotFoundPage } from './pages';

export default function App() {
  return (
		<Routes>
			<Route path="*" element={<NotFoundPage />} />
			<Route path="/" element={<HomePage/>}/>
			{/* в родительском route - общий путь, в дочерних НЕ используется часть родительского пути */}
			{/* пропс index указывает родительский путь */}
			<Route path="/profile">
        <Route index element={<ProfilePage />} />
        <Route path="edit" element={<ProfileForm/>} />
        <Route path="archive" element={<ProfileListPage />} />
			</Route>
		</Routes>
  );
} 
```

## 14.3 Outlet

При передаче в родительский Route пропса element, он отобразит в компоненте в `<Outlet />` дочерний компонент, соответствующий текущему URL
```typescript
<Route path="/profile" element={<ProfileLayout/>}>
	<Route index element={<ProfilePage />} />
	<Route path="edit" element={<ProfileForm/>} />
	<Route path="archive" element={<ProfileListPage />} />
</Route>
```
ProfileLayout: 
```typescript
import { Link, Outlet } from "react-router-dom"
<ul>
	<li><Link to="/profile">Профиль</Link></li>
	<li><Link to="/profile/edit">Редактировать профиль</Link>
	<li><Link to="/profile/archive">Архивные часты</Link></li>
</ul>
{/* В Outlet отрендерится дочерний компонент соответствующий текущему URL */}
<Outlet />
```

## 14.4 Декларативная маршрутизация (новая)

Особенности новой маршрутищзации:
1. предзагрузка данных перед рендером роута
2. обработка состояний загрузки
3. централизованная обработка ошибок
4. реализация оптимистичного UI
5. восстановление прокрутки при навигации

Создание маршрутов:
```typescript
//App
import {createBrowserRouter} from "react-router-dom";

export const router = createBrowserRouter([
	{
    path: "*",
    element: <NotFoundPage />,
  },
	{
    path: "/",
    element: <HomePage/>,
  },
	{
    path: "/profile",
    element: <ProfileLayout/>,
		children: [
			{
				index: true,
				element: <ProfilePage />,
			},
			{
				path: "edit",
				element: <ProfileForm/>,
			},
						{
				path: "archive",
				element: <ProfileListPage />,
			},
		],
  },
])
```
Обновление старой конфигурации:
```typescript
//App
import {
  createRoutesFromElements,
  createBrowserRouter,
  Route,
} from "react-router-dom";

export const router = createBrowserRouter(
  createRoutesFromElements(
    <Route path="*" element={<NotFoundPage />} />
		<Route path="/" element={<HomePage/>} />
		<Route path="/profile" element={<ProfileLayout/>}>
			<Route index element={<ProfilePage />} />
			<Route path="edit" element={<ProfileForm/>} />
			<Route path="archive" element={<ProfileListPage />} />
		</Route>
  )
); 
```
Создание Router
```typescript
//index
import {RouterProvider} from "react-router-dom";
import {router} from './app.js';

ReactDOM.createRoot(document.getElementById("root")).render(
  <StrictMode>
    <RouterProvider router={router}>
			<App />
		</RouterProvider>
  </StrictMode>
); 
```

**Асинхронная функция** `loader` загружает данные до рендера компонента.  
Данные можно получить через хук `useLoaderData`  
**Работает только с новым функционалом**
```typescript
import { defer } from "react-router-dom";
export async function loader({ params, request }) {
	const  data = await getData();
	return  defer({data});
};

// компонент-экран
import { useLoaderData } from "react-router-dom";
{/* Кастомное API */}
import { getData } from '../services/api';
export const PageComponent = () => {
	const { data } = useLoaderData();
	return (
		<Suspense fallback={<div>"Loadinng..."</div>}>
			<Await .
				key="myId" 
				resolve={data} 
				errorElement={<div>Error</div>}
				 children={(resolvedData) => <div>{resolvedData.title}</div>}
			/>
		</Suspense>
	)
}

//App
import {createRoutesFromElements, createBrowserRouter, Route} from "react-router-dom";
import {loader as dataLoader} from "./pages/mycomponent";

export const router = createBrowserRouter(
  createRoutesFromElements(
		<Route path="/list" loader={dataLoader} element={<PageComponent  />} />
	)
)
```
Компонент Await см. ниже

**Состояние загрузки**
`useNavigation` - возвращает информацию о текущем состоянии навигации: простой или загрузка  
**Работает только с новым функционалом**
```typescript
import { useNavigation } from "react-router-dom";

function App() {
  const navigation = useNavigation();
} 
```
Объект navigation содержит:
* navigation.state - состояние загрузки данных при маршрутизации(idle, submitting, loading)  
	(простой, данные отправляются формой, загрузка данных)
	каждый запрос GET проходит через состояния: idle->loading->idle  
	каждый запрос POST, PUT, PATCH, DELETE проходит через состояния: idle->submitting->loading->idle  
* navigation.loaction - информация о URL на который переходит пользователь (можно получить через useLocation)
* navigation.formData - данные, отправляемые формой
* navigation.formAction - URL по которому отправлена форма
* navigation.formMethod - метод отправки формы

## 14.5 Навигация

## 14.5.1 Link 

Для замены элемента `<a>` используется `<Link>`, который не перезагружает страницу:
```typescript
import { Link } from "react-router-dom"
<Link to='/'>Главная страница</Link>
```
* to - путь куда ведет ссылка
* relative='path' - префикс пути указанного в to(берет URL текущей страницы)

## 14.5.2 NavLink 

`NavLink` - аналог `Link`, но в пропс className можно передать функцию  
Аргумент функции - объект с полем isActive, которое принимает true, если ссылка совпадает с URL в браузере  
Альтернативно функция передается в children
```typescript
<NavLink
	to={'/login'}
	className={({isActive}) => isActive ? styles.linkActive : 'styles.link'}
/> 
	
<NavLinkto={'/login'}>
	{(isActive) => (<p style={isActive ? styles.linkActive : 'styles.link}>Some Text</p>)}
</NavLink> 
```
## 14.5.3 useNavigate 

`useNavigate()` - возвращает функцию навигации. **НЕ путать с useNavigation**  
- первый аргумент(string) - выполняется навигация на роут
```typescript
const navigate = useNavigate(); 
{/* Абсолютный путь, заменяющий URL в адресной строке */}
navigate('/destination')
{/* Относительный путь, добавляющийся к URL в адресной строке */}
navigate('destination')
```
- первый аргумент(number) - аналог history.go(): `navigate(-1);`
- второй аргумент ({replace: boolean = false, state}), где  
	replace - true(заменяет)/false(добавляет) текущую позицию в историю (аналог replaceState/pushState)  
	_замена требуется, если страница стала неактуальной или не нее не предполагается возвращаться_
	state - состояние страницы, которое не сбрасывается при перезагрузке страницы

## 14.5.4 Navigate 
`<Navigate to={url} replace={true} state={data} />` - перенаправляет пользователя на другую страинцу
Пропсы аналогичны useNavigate и Link
```typescript
<Route path="/profile">
  {!loggedIn ? <Navigate to="/login" /> : <ProfilePage />}
</Route> 
```

## 14.6 Работа с параметрами

## 14.6.1 useParams

`useParams` возвращает объект для каждого параметра в URL
url `item/123` вернет `{id: 123}` для 
```typescript
<Route path="/item" element={<ItemPage/>}>
	<Route path=":id" element={<ItemIdPage />} />
</Route>
```
**Фильтрация и сортировка**
`useSearchParams` - сохраняет состояние в параметрах поиска URL (query)
```typescript
import { useSearchParams } from 'react-router-dom'; 
const [searchParams, setSearchParams] = useSearchParams();

setSearchParams({ filter });
```
searchParams - объект Map, получение значения по ключу: `searchParams.get('filter')`,  
запись:
```typescript
setSearchParams({ [key]: value })` или 
//или
setSearchParams(prevParams => {
	prevParams.set(key, value);
	return prevParams;
});
```

## 14.6.2 useLocation

`useLocation` - возвращает текущее месторасположение в виде объекта  
для `http://localhost:3000/list?filter=item#id`
* pathname - '/list'
* state - null (данные состояния, хранятся в памяти и не пропадают при перезагрузке. Тип любой)
* search - '?filter=item'
* key - '2FDS3A' (уникальный ключ маршрута)
* hash - '#id'

**Проблемы**  
1.Внутреннее состояние (state) компонента сбрасывается при перезагрузке.  
	Нужно заменить зависимости от state на сопадение с динамическим маршрутом
2. При включении модального окна - пропадает контент на фоне из-за того, что сменился url  
	Решение - **множественный маршрут** (использование нескольких Routes)   
	
Используем пропс `location` в Routes. В нем указывается какой в данный момент url использовать для рендера дочерних элементов Route  
`location` содержит объект с теми же полями как в `useLocation`
```typescript
{/* всегда рендерит  компонент Gallery по пути '/' */}
<Routes location={{pathname: '/'}}>
  <Route element={<Gallery/>} />
  <Route path="/img/:id" element={<ImageView />} />
</Routes>
{/* рендерит модальное окно при наличии динамического маршрута */}
<Routes>
  <Route path="/img/:id" element={<Modal><ImageView /></Modal>} />
</Routes> 
```

Для этого в компоненте при помощи useLocation получаем объект  
`const location = useLocation()`  
Передадим его в state компоненту Link  
`<Link to={`/img/${img.id}`} state={{backgroundLocation: location}}>Some Text</Link>`  
Получим location в App
```typescript
const location = useLocation();
const backgroundLocation = location.state?.backgroundLocation;

<Routes location={backgroundLocation || location}>
	<Route path='/' element={<Gallery />} />
	<Route path="/img/:id" element={<ImageView />} />
</Routes >

<Routes>
	<Route path="/img/:id" element={<Modal><ImageView /></Modal>} />
</Routes>
```
3. Исключить конфликт при прямом переходе по url, без модального окна  
Отслеживать наличие модального окна (в примере через backgroundLocation)  

_В примере, если backgroundLocation существует, то второй компонент Routes проходит проверку и перезаписывает путь до `<ImageView />` во первом компоненте `<Routes />`_  
```typescript
const location = useLocation();
const backgroundLocation = location.state?.backgroundLocation;

<Routes location={backgroundLocation || location}>
	<Route path='/' element={<Gallery />} />
	<Route path="/img/:id" element={<ImageView />} />
</Routes >

{backgroundLocation &&
	<Routes>
		<Route path="/img/:id" element={<Modal><ImageView /></Modal>} />
	</Routes>
}
```

**Важно**: если передана `backgroundLocation`, то 

## 14.7 Типизация в router

**Типизация RouteProps**  
RouteProps передает в `<Route />` свойста маршрута: path, element и т.д.  

```typescript
import { Navigate, Route, RouteProps } from 'react-router-dom';
{/* дополняем кастомными свойствами пропс RouteProps */}
interface PrivateRouteProps extends RouteProps {
  isAuthenticated: boolean;
  redirectTo: string;
}

function PrivateRoute({ isAuthenticated, redirectTo, ...rest }: PrivateRouteProps) {
  if (!isAuthenticated) {
    return <Navigate to={redirectTo} replace />;
  }

  return <Route {...rest} />;
} 
```

**Типизация LinkProps**  
LinkProps передает в `<Link />` свойста ссылки: to, replace, state и т.д.  
```typescript
import { Link, LinkProps } from 'react-router-dom';
interface PrivateLinkProps extends LinkProps {
  isEnabled: boolean,
	children: ReactElement
}

function CustomLink({ isEnabled, children, ...rest }: LinkProps) {
  return (
		{isEnabled && 
			<Link {...rest} className="custom-link">
				{children}
			</Link>
		}
  );
} 
```
**Типизация params из хука useParams**  
```typescript
import { useParams } from 'react-router-dom';

interface Params {
    id: string;
}

const UserPage = () => {
    const { id } = useParams<Params>();

    return (
        <div>
            <h1>User Page</h1>
            <p>ID: {id}</p>
        </div>);
};
```

# 15 Авторизация 

## 15.1 C токеном (JWT)

Структура JWT токена разделена точками:
* header - служебная информация: {alg: алгоритм создания токена(чаще HS256), typ: тип токена(JWT)}
* payload - данные 
* signature - подпись, предотвращающая подмену (алгоритмически генерируется на основе header и payload)
Все данные кодируются в строку (чаще Base64Url)

Токены бывают двух видов (access - малое время жизни(15-30мин), refresh - долгое время жизни)  
Если access токен expired, то по refresh токену пользователь получает новую пару токенов  
Если refresh токен expired, то новая авторизация  
![https://pictures.s3.yandex.net/resources/PWD-416_1709134032.png]

### 15.1.1 Запрос авторизации
Алгоритм авторизации:
```typescript
// внешняя функция
async function getUser() {
	const dataUser = await fetchWithRefresh('auth/user', {
		headers: {
				authorization: getCookie('accessToken'),
		},
	});
	return dataUser;
}

// авторизация
async function fetchWithRefresh(endpoint, options) {
	try {
		// делаем запрос за защищёнными данным
		return await request(endpoint, options);
	} catch (error) {
		// при ошибке авторизации получаем 401 или 403 ошибки
		if (error.statusCode === 401 || error.statusCode === 403) {
			// отправляем refresh токен для обновления токенов
			const refreshData = await refreshToken();
			if (!refreshData.success) {
				// при ошибке - новая авторизация
				return Promise.reject(refreshData);
			}
			// при успешном обновлении токенов записываем access в куки, refresh в localStorage
			setCookie('accessToken', refreshData.accessToken);
			localStorage.setItem('refreshToken', refreshData.refreshToken);
			
			// делаем новый запрос с обновленными токенами
			return await request(endpoint, {
				...options,
				headers: {
					...options.headers,
					authorization: getCookie('accessToken'),
				},
			}
		}
	}
}

// запрос обновления токенов
function refreshToken = () => {
	return request('auth/token', {
		method: 'POST',
		body: JSON.stringify({ token: localStorage.getItem('refreshToken') }),
	});
};

// запрос на сервера
async function request(endpoint, options) {
	try {
		const res = await fetch(`${host}/api/${endpoint}`, {
			headers: {
				'content-type': 'application/json',
			},
			...options,
		});
		return await checkResponse(res);
	} catch (error) {
		return Promise.reject(error);
	}
}

// проверка ответа от сервера
function checkResponse(res) {
	return res.ok
	? res.json()
	: res
		.json()
		.then(err => Promise.reject({ ...err, statusCode: res.status }));
}

```

### 15.1.2 Работа с флагами

```typescript
const initialState: TUserState = {
	// проверка токенов авторизации
  isAuthChecked: false,
	// проверка успешности авторизации
	isAuthenticated: false;
	// полезные данные при успешном запросе
  data: null,
	// информация об ошибке
  loginUserError: null,
	// статус отправки запроса
  loginUserRequest: false,
};

//createSlice
extraReducers: (builder) => {
	builder
	.addCase(loginUser.pending, (state) => {
		// отправили запрос к серверу
		state.loginUserRequest = true;
		state.loginUserError = null;
	})
  .addCase(loginUser.rejected, (state, action) => {
		// ответ не успешен, получили ошибку
		state.loginUserRequest = false;
		state.loginUserError = action.payload;
		// проверка токенов завершена (но не успешно)
		state.isAuthChecked = true;
	})
	.addCase(loginUser.fulfilled, (state, action) => {
		// ответ успешный, получили данные
		state.loginUserRequest = false;
		state.data = action.payload.user;
		// проверка токенов завершена (успешно)
		state.isAuthenticated = true;
		// авторизация пройдена
		state.isAuthChecked = true;
	}
}
```
По loginUserRequest выставляется лоадер, ожидающий окончания запроса от сервера
По isAuthChecked проверяется пройдена ли авторизация пользователем
По isAuthenticated проверяется иная логика связанная с проверками токенов

isAuthChecked === false && isAuthenticated === false - проверка авторизации не была начата 
isAuthChecked === true && isAuthenticated === false - токены expired, авторизация не завершена (нужно сделать запрос refresh токена)  
isAuthChecked === true && isAuthenticated === true - токены активны, авторизация пройдена  

### 15.1.3 Сохранение токенов

Access-токен сохраняеют в ooockie, refresh-токен в localStorage  
Куки автоматически прикрепляются к любому запросу на сервер  
Оба варианта хранилища принадлежат домену. С google невозможно обратиться к yandex  

Формат куки: `"name=Petr; path=/; age=18"`  
Установка куки: `document.cookie = "name=Petr; path=/; age=18"` (такая кука доступна 5 минут)  

Функция нормализующая работу с временем жизни куки:
```typescript
// аргументы: имя куки, значение, объект дополнительных свойств
export function setCookie(name: string, value: string, props: { [key: string]: string | number | Date | boolean } = {}) {
	props = { path: '/', ...props };

	// время жизни куки
	let exp = props.expires;
	// если время жизни передано и это число, перессчитываем время, когда куки будут неактуальными
	if (typeof exp == 'number' && exp) {
		const d = new Date();
		d.setTime(d.getTime() + exp * 1000);
		exp = props.expires = d;
	}
	// если время жизни передано и это дата, то преобразуем в формат UTC
	if (exp && exp instanceof Date) {
		props.expires = exp.toUTCString();
	}
	// кодируем value - encodeURIComponent, чтобы исключить возможные ошибки при использовании
	value = encodeURIComponent(value);
	// добавляем в куки переданные дополнительные свойства
	let updatedCookie = name + '=' + value;
	for (const propName in props) {
		updatedCookie += '; ' + propName;
		const propValue = props[propName];
		if (propValue !== true) {
				updatedCookie += '=' + propValue;
		}
	for (const propName in props) {
		updatedCookie += '; ' + propName;
		const propValue = props[propName];
		if (propValue !== true) {
				updatedCookie += '=' + propValue;
		}
}

document.cookie = updatedCookie;
```
### 15.1.4 Повторная авторизация

Функция отправляет запрос серверу для повторной авторизации при наличии токенов  
Токен отправляется в заголовке `authorization`  
Кроме токена отправляется схема аутентификации `Bearer`, для указания серверу, что проверка прав осуществляется по токену  
Вспомогательная функция получения куки
```typescript
export function getCookie(name: string) {
	const matches = document.cookie.match(
		// eslint-disable-next-line no-useless-escape
		new RegExp('(?:^|; )' + name.replace(/([\.$?*|{}\(\)\[\]\\\/\+^])/g, '\\$1') + '=([^;]*)')
	);
return matches ? decodeURIComponent(matches[1]) : undefined;
}
```

```typescript
async function getUser() {
	//Использование функции fetchWithRefresh (рассмотрена ранее)
	const dataUser = await fetchWithRefresh('auth/user', {
			headers: {
					authorization: getCookie('accessToken'),
			},
	});
return dataUser;
} 
```

## 15.2 C помощью сессии

1. Сервер получает авторизационные данные
2. Устанавливает заголовок Set-Coockie (в браузере клиента помещается кука) 
3. Каждый запрос к серверу сопровождается автоматическим прикреплением куки
4. Сервер каждый раз сверяет наличие сессии с идентификатором в куке

Данный способ подходит для приложений работающих только в браузере с минимальной микросервисной архитектурой  
Если нужно запомнить корзину, открытые документы и ихостояние, просмотренные товары - используется сессия  

## 15.3 Защищенный маршрут

Маршрут, по которому может попасть только пользователь с нужным набором прав  
Типичная схема с защищенным маршрутом:  
Route `/list` является защищенным. `<ProtectedRoute><RoutePathComponent/></ProtectedRoute>` передается в пропс element, где  
RoutePathComponent - экран данного Route  
Незащищенный Route, куда осуществляется редирект из зашишенного также оборачивается в `<ProtectedRoute />`, т.к.  
после редиректа (в нем завершении авторизации) нужно осуществить обратный редирект на страницу, которую пользователь открывал изначально  
```typescript
export default function App() {
  return (
		<Provider store={store}>
			<BrowserRouter >
				<Routes>
						<Route path="/login" element={<ProtectedRoute onlyUnAuth><LoginPage/></ProtectedRoute>} />
						<Route path="/list" element={<ProtectedRoute><ListPage /></ProtectedRoute>} />
						<Route path="*" element={<NotFound404/>}/>
				</Routes>
			</BrowserRouter >
		</Provider>
  );
} 
```
Реализация защищенного маршрута: 
- если в хранилище нет токенов - переадресация на незащищенный маршрут (например `/login`)
- если токены есть и они просрочены - переадресация
- если токены есть и они не просрочены - переходим по защищенному маршруту (в примере `/list`)
```typescript
type ProtectedRouteProps = {
	onlyUnAuth?: boolean;
  children: React.ReactElement;
};

export const ProtectedRoute = ({ children }: ProtectedRouteProps) => {
	// проверка статуса авторизации
	const isAuthChecked = useSelector(isAuthCheckedSelector);
	// получение данных о пользователе
	const user = useSelector(userDataSelector);
	// используется при большом количестве защищенных маршрутов
	const location = useLocation();
	
	//здесь можно сделать асинхронный запрос на сервер для авторизации
	
	// пока идет чекайут пользователя показывам прелоадер
	if (!isAuthChecked) { 
    return <Preloader />;
  }
	//если пользователя в хранилише нет и он НЕ на странице авторизации, то делаем редирект
	if (!onlyUnAuth && !user) {
		return <Navigate replace to='/login'/ state={{ from: location }} />;
  }
	//если пользователь на странице авторизации и данные есть обратный редирект
	if (onlyUnAuth && user) { 
		// если пользователь зашел на страницу авторизации по прямой ссылке, то редиректим на главную страницу
		// если передан location.state.from, то обратный редирект
		const from  = location.state?.from || { pathname: '/' };
    return <Navigate replace to={from} />;
  }
  return children;
} 
```

## 15.4 Log Out

Вспомогительная функция удаления куки (установка отрицательного значения во времени жизни куки):  
```typescript
// services/utils.ts
export function deleteCookie(name:string) {
	// Находим куку по ключу token, удаляем её значение, 
	// устанавливаем отрицательное время жизни, чтобы удалить сам ключ token
  setCookie(name, null, { expires: -1 });
}
```
Механизм Log Out. Редирект не нужен, так как ProtectedRoute должен учитывать отсутствие user в хранилище  
```typescript
// utils/api.ts
// запрос на logout серверу
export const logoutApi = () =>
  fetch(`${URL}/auth/logout`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json;charset=utf-8'
    },
    body: JSON.stringify({
      token: localStorage.getItem('refreshToken')
    })
  }).then((res) => checkResponse<TServerResponse<{}>>(res));
	
// services/slices/user/user-slice.ts
// удаление информации о токенах и пользователе
export const logoutUser = createAsyncThunk(
  'user/logoutUser',
  (_, { dispatch }) => {
    logoutApi()
      .then(() => {
        localStorage.clear(); // очищаем refreshToken
        deleteCookie('accessToken'); // очищаем accessToken
        dispatch(userLogout()); // удаляем пользователя из хранилища
      })
      .catch(() => {
        console.log('Ошибка выполнения выхода');
      });
  }
);
```

## 16 Drag and Drop

Установка `npm i react-dnd`  
[Документация](https://react-dnd.github.io/react-dnd/about)  

`useDrag(object, deps)` - добавляет функциональность элементам при перетаскивании  
Возвращает массив из трех значений `[CollectedProps, dragRef, dragPreviewRef]`   
* object - объект c полями: type, item, collect(необязательный)
	* type - тип элементов, которые можно перетащить в целевой элемент
	* item - данные о перетаскиваемом элементе
	* collect(monitor, props) - функция (набор вычислений для работы с пропсами)
* deps - зависимости
* CollectedProps - объект, предоставляющий другим частям компонента доступ к вычислениям функции collect внутри useDrag
* dragRef - ref на перетаскиваемый элемент/компонент
* dragPreviewRef - элемент использующийся в качестве превью при перетаскивании (необязательный)

```typescript
const [{isDrag}, dragRef] = useDrag({
	type: "animal",
	item: {id},
	collect: monitor => ({
			isDrag: monitor.isDragging()
	})
});

return (
	!isDrag && 
	<div ref={dragRef}>
		{content}
	</div>
)
```

Объект monitor созержит 11 методов:
* isDragging - возвращает boolean (true при перетаскивании)

`useDrop(object)` - добавляет функциональность элементам при перетаскивании  
Возвращает массив из двух значений `[CollectedProps, DropTargetRef]`  
* object - объект c полями: type, accept, hover(необязательный), drop(необязательный), collect(необязательный)
	* accept - строка аналогичная свойству type для перетаскиваемого объекта
	* hover(item, monitor) - Срабатывает при попадении перетаскиваемого элемента в зону целевого. item перетаскиваемого объекта
	* drop(item, monitor) - Срабатывает при броске перетаскиваемого элемента в целевой. item перетаскиваемого объекта
	* collect(monitor, props) - аналогичен методу из useDrag
* CollectedProps - объект, предоставляющий другим частям компонента доступ к вычислениям функции collect внутри useDrop
* DropTargetRef - реф указывающий на целевой элемент

```typescript
const [, dropTarget] = useDrop({
	accept: "animal",
	drop(itemId) {
			onDropHandler(itemId);
	},
});

return (
	<div ref={dropTarget}>
		{children}
	</div>
);
```

Опишем компонент контейнер DragNDrop:
```typescript
import { DndProvider } from "react-dnd";
import { HTML5Backend } from "react-dnd-html5-backend";

const [elements, setElements] = React.useState([]);
const [draggedElements, setDraggedElements] = React.useState([]);

// обработчик события "броска" элемента
const handleDrop = (itemId) => {
// удаляем перетаскиваемый элемент из контейнера
	setElements([
			...elements.filter(element => element.id !== itemId.id)
	]);
// добавляем элемент к перенесенным элементам
	setDraggedElements([
		...draggedElements,
		...elements.filter(element => element.id === itemId.id)
	]);
};

return (
	<DndProvider backend={HTML5Backend}>
		<DraggableAnimal data={animal}/>
		<DropTarget onDropHandler={handleDrop} />
	</DndProvider>
)
```
2 варианта реализации api: стандартное, для touch событий  
`import {HTML5Backend} from "react-dnd-html5-backend"`  
`import {touch} from "react-dnd-touch-backend"`  

## 17. WebSocket 

Установка соединения в коде: `const websocket = new WebSocket(url)`  
URL в формате: `ws://my-ws-server.net/ws-connection`
Или в защищенном протоколе: `wss://my-ws-server.net/ws-connection`

При создании экземпляра WebSocket браузер отправляет http-запрос:
```
GET /ws-connection
Host: my-ws-server.net
Origin: https://my-awesome-site.ru
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Key: Iv8io/9s+lYFgZWcXczP8Q==
Sec-WebSocket-Version: 13 
```
origin - адрес, с которым устанавливается соединение  
Connection: Upgrade - заголовок для изменения протокола  
Upgrade: websocket - заголовок с протоколом, на который будет изменение  
Sec-WebSocket-key - случайный ключ безопасности  
Sec-WebSocket-Version - версия протокола  

После установки WebSocket соединения доступны события для обработки в addEventListener или on<Имя События>:
* open - возникает, когда соединение с сервером открыто
* message - возникает, когда браузер получает сообщение от сервера
* error - возникает, когда соединение закрыто по причине ошибки
* close - возникает, когда соединение ожидаемо закрыто со стороны браузера/сервера  

* `websocket.send(data)` - отправка данных серверу в текстовом формате, ArrayBuffer или Blob  
Для бинарных данных выбирается формат: `socket.binaryType = 'arraybuffer'` или `'blob'`  
* `websocket.close(code, reason)` - закрывает соединение (аргументы - специальный код закрытия, описание)  
{коды закрытия](https://datatracker.ietf.org/doc/html/rfc6455#section-7.4.1)
* websocket.readyState - возвращает состояние соединения  
	- 0 - CONNECTING - соединение еще не установлено
	- 1 - OPEN - обмен данными
	- 2 - CLOSING - соединение закрывается
	- 3 - CLOSED - соединение закрыто

## 18 Кэширование

Кэширование данных - `npm i @tanstack/react-query`  
[Документация](https://tanstack.com/query/latest)  
QueryClient используется для взаимодействия с кэшем  
Для установки времени жизни кэша используется параметр `gcTime` в мс  

`useQuery({queryKey, queryFn})` - хук для использования запроса, возвращает isPending, error, data
* queryKey - список параметров запроса (при изменении параметров отправляется новый запрос, иначе из кэша)
* queryFn - функция запроса для получения данных
* isPending - состояние загрузки запроса
* error - объект ошибки
* data - результат успешного запроса
```typescript
import {QueryClient} from '@tanstack/react-query'

const queryClient = new QueryClient()

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <MyComponent />
    </QueryClientProvider>
  )
} 

function Example() {
	const { isPending, error, data } = useQuery({
		queryKey: ['todo', id],
		queryFn: () =>
			fetch(url/id).then((res) =>
				res.json(),
			),
	})
	if (isPending) return 'Loading...'
	if (error) return 'An error has occurred: ' + error.message
 return <p>{data}</p>
}
```

`useMutation({mutationFn, onError, onSuccess, onSettled})` - работа с запросами на изменение (удаление, обновление, добавление)  
* mutationFn - функция совершающая мутацию
* onError(error, variables, context) - функция запускающаяся при ошибке
* onSuccess(data, variables, context) - функция запускающаяся при успехе
* onSettled(data, error, variables, context) - функция запускающаяся при ошибке или успехе
* isPending - состояние загрузки запроса
* isError — состояние ошибки
* error - объект ошибки
* isSuccess — состояние успеха
* mutate - функция мутации, запускающая mutationFn из параметров хука
```typescript
function App() {
  const {isPending, isError, error, isSuccess, mutate} = useMutation({
    mutationFn: (newTodo) => {
      return axios.post('/todos', newTodo)
    },
  })
	
	if (isPending) return 'Добавление задачи...'
	if (isError) return <div>Ошибка: {mutation.error.message}</div>
	if (isSuccess) return <div>Задание добавленно!</div>
	return <button onClick={() => {mutation.mutate({ id: new Date(), title: 'Text' })} />
}
```

`queryClient.invalidateQueries({queryKey})` - инвалидация(признание кэша не актуальным)   
* queryKey - список параметров, **с которых должен начинаться** queryKey инвалидированных запросов
```typescript
queryClient.invalidateQueries({
  queryKey: ['todos', { type: 'done' }],
})

// Запрос будет инвалидирован
const todoListQuery = useQuery({
  queryKey: ['todos', { type: 'done' }],
  queryFn: fetchTodoList,
})

// Запрос НЕ будет инвалидирован
const todoListQuery = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodoList,
}) 
```

Для точного совпадения запросов на инвалидацию используется `exact: true`  
```typescript
queryClient.invalidateQueries({
  queryKey: ['todos'],
  exact: true,
})

// Запрос будет инвалидирован
const todoListQuery = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodoList,
})

// Запрос НЕ будет инвалидирован
const todoListQuery = useQuery({
  queryKey: ['todos', { type: 'done' }],
  queryFn: fetchTodoList,
}) 
```

## 19 React-router-dom

Используется в маленьких приложениях, где нет необходимости подключать redux  
Навигация Form:
```typescript
const formAction = async ({ request }) => {
	// Получаем объект FormData с данными формы
	const formData = await request.formData();
	// преобразование формата FormData
	const data = Object.fromEntries(formData.entries());
	await fakeApiRequest(data);
	return redirect("/posts");
}

import { Form } from "react-router-dom";
<Route path: path:"/create-post" action:{formAction} element:{<Form method:"post" action"/create-post" />} />
```
Навигация модальных окон - состояние модального окна можно хранить в hashUrl вместо redux или useParams  
```typescript
<Link to={{ hash: "#modal-name" }}>Open modal</Link>

//Modal
return (
	location.hash === "#modal-name" && ()
)
```

## 20 TDD

TDD - создание тестов до написаия функционального кода приложения  
Виды тестирований: 
* модульные тесты - проверка компонента при различных входных параметрах (проверка работы компонента)
* интеграционные тесты - проверка работы нескольких связанных компонентов (проверка пришедших данных по запросу)
* функциональные (E2E) тесты - проверка взаимодействия всех модулей (регистрация)

Наиболее популярные способы запуска тестов: Jest или Cypress, а также [(chai](https://www.chaijs.com/), [enzyme](https://enzymejs.github.io/enzyme/)  
[Jest дока](https://jestjs.io/docs/configuration)  
[TS-Jest дока](https://kulshekhar.github.io/ts-jest/docs/getting-started/options/)  
Установка:  
`npm install --save-dev jest` для JS  
`npm install --save-dev ts-jest ts-node` для TS   

Создание конфига: `npx jest --init`  
Дополнительная типизация: `npm install --save-dev @jest/globals`  
`"test": "jest --verbose”` - команда запуска тестов  

Тесты используеют функции:
* `describe(desctiption, fn)` - объединяет несколько тестов, добавляя им описание
* `test(description, fn)` - функция выполнения теста. Принимает в аргументах описание и тест  
* `expect(arg)` - проверка переданного аргумента условиям (в примере `toBe(8)`)  

Условия проверки тестов:  
* `toBe` - сравнивает ожидаемый и полученный результат с помощью Object.is (`{} !== {}`)  
* `toBeNull` - сравнивает результат с null
* `toEqual` - сравнивает рекурсивно ожидаемый и полученный объект/массив
* `arrayContaining` - проверяет содержит ли переданный массив все элементы ожидаемого
* `toHaveBeenCalled` - проверяет была ли вызвана нужная функция при выполнении теста
* `toThrow` - проверяет были вызвана ошибка при выполнении теста

Хуки: 
* beforeEach(fn)/beforeAll(fn) - запускает функцию fn до выполнения каждого/всех теста
* afterEach(fn)/afterAll(fn) - запускает функцию fn после выполнения каждого/всех теста

Пример простого теста для функции умножения:  
```typescript
import { expect, test, describe } from '@jest/globals';

// toBe
import { expect, test } from '@jest/globals';
const multiply = (a: number, b: number): number => a * b 

describe('Арифметические операции', () => {
	test('умножение', () => {
		const result = () => multiply(2,4);
		expect(result).toBe(8);
	}) 
})

// toEqual
const person1 = {name: 'Petr', age: 12};
const person2 = {name: 'Andrey', age: 12};
test('Равенство объектов', () => {
	// тест не пройдет, так как поля в объектах не равны
	expect(person1).toEqual(person2);
});

// toHaveBeenCalled
function runAction(callback, flavour) {
    if (flavour === 'action') {
        callback(flavour);
    }
}
test('Вызов функции', () => {
	const drink = jest.fn();
	runAction(drink, 'action');
	expect(drink).toHaveBeenCalled();
});

//arrayContaining
const received = ['Alice', 'Bob', 'Eve'];
const expected = ['Alice', 'Bob'];
 
test('является ли массив expected подмножеством массива received', () => {
	expect(received).toEqual(expect.arrayContaining(expected));
});

//toThrow`
function myFunc(count: number) { if(count < 0) throw new Error('Ошибка')}

test('Проверка ошибки', () => {
	const result = () => myFunc(-1)
	expect.toThrow('Ош') // частичное совпадение с текстом ошибки
	expect.toThrow('Ошибка') // полное совпадение с текстом ошибки
	expect.toThrow(Error) // любая ошибка
}

// beforeAll и afterAll

beforeAll(() => {})
afterAll(() => {})
```

**Мокирование** - создание функции заглушки, эмулирующей поведение внешних зависимостей  
`jest.spyOn(object, methodName)` - мокирует метод объекта   
Примеры: 
* resize объекта window
* play объекта video
* customMethod объекта customApi

Метод mockImplementation(fn) - позволяет указать конкретную реализацию функционала мока в функции fn 
mockRestore() - удаляет мокирование
```typescript
let alertSpy: jest.SpyInstance;
beforeEach(() => {
	alertSpy = jest.spyOn(window, 'alert').mockImplementation();
}
afterAll(() => {
	alertSpy.mockRestore()
})
```


**Базовый пример тестирования с помощью мока**
```typescript
test('Тестирование с помщью мока', () => {
	const callback = jest.fn((element) => element * 2);
	map([1,2,3], callback)
	// проверка, что callback вызван трижды
	expect(callback).toHaveBeenCalledTimes(3)
	expect(callback.mock.calls).toHaveLength(3);
	// проверка результата итерации
	expect(callback.mock.results[0].value).toBe(2);
})
```

**Мокирование запроса на сервер**:  
1. Мокирование браузерного API - fetch
```typescript
export async function myAsyncFunction(id: number): Promise<number> {
		// функция с fetch запросом
    const responseData = await getData(id);

    const data: number[] = Object.values(responseData)
    const result = data.reduce((acc, value) => acc + value, 0) / data.length;
    return result
}

test('мокирование fetch', async () => {
	global.fetch = jest.fn(() =>
			Promise.resolve({
					json: () => Promise.resolve({
							math: 5,
							programming: 5,
							physics: 5
					}),
			})
	) as jest.Mock;

	const result = await myAsyncFunction(1);

	expect(result).toBe(5);
})
```

2. Мокирование самой функции через mockImplementation
```typescript
jest.mock('../getData');

(getData as jest.Mock).mockImplementation(() => Promise.resolve({
    math: 5,
    programming: 5,
    physics: 5
}));

test('мокирование функции', async () => {

    const result = await myAsyncFunction(1);

    expect(result).toBe(5);
}) 
```
3. Мокирование при помощи spyOn
```typescript
test('Мокирование при помощи spyOn', async () => {
    const getDataMock = jest.spyOn(dataApi, 'getData').mockImplementation(() => Promise.resolve({
        math: 5,
        programming: 5,
        physics: 5
    }));

    const result = await myAsyncFunction(1);
		// позволяет отследить факт вызова мока
    expect(getDataMock).toBeCalled();
    expect(result).toBe(5);
})
```

**Мокирование таймеров**
[статья хабр](https://habr.com/ru/companies/psb/articles/750286/)
При мокировании таймеров используются фейковые таймеры без времени ожидания
```typescript
beforeAll(() => {
	jest.useFakeTimers();
}
afterAll(() => {
	// возвращаем использование реальных таймеров
	jest.useRealTimers();
}
```
Для перематывания времени используется `jest.advanceTimersByTime(msDelay)`  

```typescript
export function timer(callback: () => void) {
    setTimeout(callback, 1000);
} 

test('мокирование таймера', () => {
    // фейковый таймер без времени ожидания
    jest.useFakeTimers();
    // следим за выполнением setTimeout
    jest.spyOn(global, 'setTimeout');

    const callback = jest.fn();

    timer(callback);

    // количество вызвовов setTimeout
    expect(setTimeout).toHaveBeenCalledTimes(1);
    // проверка аргументов, с которыми был вызван setTimeout
    expect(setTimeout).toHaveBeenLastCalledWith(callback, 1000);
});
```

**Mock Service Worker**  - универсальный способ мокирования запросов к серверу  
[Mock Service Worker](https://mswjs.io/docs/getting-started)  
Установка: `npm install msw --save-dev`  

Создаются 2 файла: 
* mocks/handlers.ts с моками сетевых запросов  
```typescript
// mocks/handlers.ts

// модуль для мокирования сетевых запросов
import { http } from 'msw' 
// класс ответа на запрос (важно использовать HttpResponse, т.к. он содержит json/xml/formData и т.д.
import { HttpResponse } from 'msw'

export const handlers = [
    http.get(url, () => {
        return HttpResponse.json({
            math: 5,
            programming: 5,
            physics: 5
        });
    })
]; 
```
* mocks/node.ts с инициацией мок-сервера для перехвата сетевых запросов и подмене их на моки из handlers
```typescript
// mocks/node.ts

import { setupServer } from 'msw/node'
import { handlers } from './handlers'

// инициализация мок-сервера
export const server = setupServer(...handlers)
```

Пример использования: 
```typescript
// включаем мок-сервер до всех тестов
beforeAll(() => { server.listen() })
// выключаем мок-сервер после всех тестов
afterAll(() => { server.close() })
```

**Тест Reducer**  
```typescript
describe('тест reducer', )() => {
	const initialState = [
		{
			id: 1,
			name: 'Petr',
			isAdmin: false
		},
		{
		id: 2,
		name: 'Andrey',
		isAdmin: true
		},
	]
	test('Change rule true', () => {
		const newState = adminSliceReducer(initialState, toggleRule({id: 1})
		const {isAdmin} = newState
		expect(isAdmin).toBe(true)
	})
	test('Change rule false', () => {
		const newState = adminSliceReducer(initialState, toggleRule({id: 2})
		const {isAdmin} = newState
		expect(isAdmin).toBe(false)
	})
})
```
**Тест Async Action**  
```typescript
describe('тест Async Action', )() => {
	test('Запрос админов', async () => {
		 const expectedResult = [
			{
				id: 1,
				name: 'Petr',
				isAdmin: true
			},
			{
				id: 2,
				name: 'Andrey',
				isAdmin: true
			},
		]
		global.fetch = jest.fn(() =>
			Promise.resolve({
					json: () => Promise.resolve(expectedResult),
			})
		) as jest.Mock;
		const store = configureStore({
			reducer: { admins: adminSliceReducer }
		});
		await store.dispatch(fetchAdmins());
		const admins = store.getState().admins;
		expect(admins).toEqual(expectedResult)
	}
}
```
**Тест Selector**  
```typescript
describe('тест Selector', () => {
	test('получение админов', () => {
		const store = configureStore({
			reducer: {
					admins: adminSliceReducer,
			},
			preloadedState: { 
				admins: {
					admins: [
						{
							id: 1,
							name: 'Petr',
							isAdmin: false
						},
						{
							id: 2,
							name: 'Andrey',
							isAdmin: true
						},
					],
					isLoading: false
				}
			}
		})
		const admins = selectLikedTracks(store.getState());
		expect(admins).toEqual([
			{
				id: 2,
				name: 'Andrey',
				isAdmin: true
			}
		])
	})
})
```
**Тест компонента**  
Установка: `npm install --save-dev react-test-renderer @types/react-test-renderer`  
Компонент: 
```typescript
interface LinkProps {
    title: string
    url: string
}

export const Link = ({ title, url }: LinkProps) => {
	const onClick = () => {
			alert('Link Clicked')
	};

	return (
			<a href={url} onClick={onClick}>{title}</a>
	)
}
```
Тест
```typescript
import renderer from 'react-test-renderer';
import { render, screen, userEvent } from '@testing-library/react';

test('Тест рендера компонента', () => {
  const tree = renderer
    .create(<Link title="Title" url="url" />)
    .toJSON();
    expect(tree).toMatchSnapshot();
}); 

test('Тест нажатия на ссылку', () => {
    // мокаем alert
    window.alert = jest.fn();
		
    render(<Link title="Title" url="url" />)

		// Находим элемент ссылки
		const link = screen.getByText("Title");

		// Имитируем нажатие на ссылку
    userEvent.click(link);
        
		// Проверяем, что alert сработал с правильным текстом предупреждения
    expect(window.alert).toHaveBeenCalledWith('Link Clicked');
}); 
```

**Тестирование хука в компоненте**  

```typescript
export const Counter = () => {
	let [count, setCount] = useState(0);

	const decrement = () => setCount((count -= 1));
	const increment = () => setCount((count += 1));

	return (
		<div>
			<p data-testid="count">{count}</p>
			<button data-testid="decrement" type="button" onClick={decrement}>-</button>
			<button data-testid="increment" type="button" onClick={increment}>+</button>
		</div>
	);
} 

import { render, getByTestId, userEvent } from '@testing-library/react';
test("Плюс и минус в счетчике работают без ошибок", () => {
    // Рендерим компонент
  const { container } = render(<Counter />);

    // Находим элемент со значением счетчика
  const countValue = getByTestId(container, "count");

    // Находим кнопку, увеличивающую значение
  const increment = getByTestId(container, "increment");

    // Находим кнопку, уменьшающую значение
  const decrement = getByTestId(container, "decrement");

    // Проверяем, что начальное состояние счетчика равно 0
  expect(countValue.textContent).toBe("0");

    // Увеличиваем значение счетчика, симулируя нажатие на соответствующую кнопку  
  userEvent.click(increment);

    // Проверяем, что состояние счетчика теперь равно 1
  expect(countValue.textContent).toBe("1");

    // Уменьшаем значение счетчика, симулируя нажатие на соответствующую кнопку  
  userEvent.click(decrement);

    // Проверяем, что состояние счетчика теперь равно 0
  expect(countValue.textContent).toBe("0");
}); 
```

**Cypress**  
Установка:  
`npm install cypress --save-dev `  
`npm install ts-node --save-dev `  
Запуск: `"cypress:open": "cypress open"`

Конфиг ts для работы Cypress:  
```typescript
{
  // расширяем основной конфиг всего проекта
  "extends": "../tsconfig.json",
  "compilerOptions": {
    "noEmit": true,
    // явно указываем что будем использовать типы из cypress
    // чтобы избежать пересечения с типами jest, если мы его используем
    "types": ["cypress"]
  },
  "include": [
    "../node_modules/cypress",
    "./**/*.ts"
  ]
} 
```  

Тесты запускаются из `cypress/e2e/**`  
Файл должен соответствовать паттерну: `cypress/e2e/**/*.cy.{js,jsx,ts,tsx}`  
Приложение запускается на dev-server и в параллели запускаем cypress  

**Пример теста**
```typescript
describe('проверяем доступность приложения', function() {
    it('сервис должен быть доступен по адресу localhost:3000', function() {
        cy.visit('http://localhost:3000'); 
				// находим в DOM дереве кнопку с атрибутом data-cy=1
        const button = cy.get(`[data-cy=${1}]`);
				// проверяем что в ней есть текст лайк
        button.contains('лайк');
				// кликаем на кнопку
        button.click();
        // проверяем что текст сменился на дизлайк
        button.contains('дизлайк');
    });
}); 
```

Удаление тестовых атрибутов из prod:
```typescript
// remove-test-attributes-loader.ts
import { loader } from 'webpack';

const removeTestAttributesLoader: loader.Loader = function (source) {
  return source.replace(/(data-test|cy)\s*=\s*(['"])[^'"]*\2/g, '');
};

export default removeTestAttributesLoader;

//webpack.loaders

loader: path.resolve(__dirname, 'remove-test-attributes-loader.ts')
```

**Strict mode**

* дважды отрисовывает компоненты
* повторно запускает эффекты
* проверяет использование legacy API

Чистыми функциями должны быть: 
* тело функции компонента (без обработчиков событий)
* функции, переданные в useState, useReducer, useMemo
* constructor, render, shouldComponentUpdate классовых компонентов

**Helmet** - инструмент для работы с метатегами страниц  
Установка `npm i react-helmet-async`  
HelmetProvider - оборачивает все приложение для создания контекста  
Helmet - импортируется в компоненты страниц, в которых требуется внедрить метатеги  
```typescript
import { HelmetProvider } from 'react-helmet-async';

root.render(
	<HelmetProvider>
		<App />
	</HelmetProvider>
); 

//page.tsx
import { Helmet } from 'react-helmet-async';

export default Page = () => {
	return (
		<>
			<Helmet>
				{ /* Стандартные мета теги */}
				<title>{title}</title>
				<meta name='description' content={description} />
				{ /* Метатеги для ВКонтакте*/}
				<meta property="og:type" content={type} />
				<meta property="og:title" content={title} />
				<meta property="og:description" content={description} />
				{ /* Метатеги для Twitter*/}
				<meta name="twitter:creator" content={name} />}
				<meta name="twitter:card" content={type} />
				<meta name="twitter:title" content={title} />
				<meta name="twitter:description" content={description} />
			</Helmet>
			<h1>Header</h1>
		</>
	)
}
```
Или аналогично: 
```typescript
<Helmet 
	htmlAttributes={{lang: 'English'}} 
	title={title} 
	meta={{description}} 
/>
```

## 21 Динамический импорт

### 21.1 Бандлинг

**Бандлинг** - объединение импортированного кода в один файл  
Асинхронная загрузка модулей:  
```typescript
export const App = () => {
	const [isModalDisplayed, setModalDisplayed] = useState<boolean>(false);
	const [ModalComponent, setModalComponent] = useState<() => JSX.Element>();
	
	const loadModalComponent = async () => {
		const loadResult = await import('../components/modal');
		{/* при export default */}
		setModalComponent(() => loadResult.default); 
		{/* при стандартном экспорте */}
		const {module} = loadResult
		setModalComponent(() => module); 
	}
	return (
		<button onClick={() => {
			setModalDisplayed(true);
			loadModalComponent();
		}}>
			Загрузить модальное окно
		</button>
		{isModalDisplayed && ModalComponent ? <ModalComponent /> : null}
	)	
}
```

### 21.2 Lazy Loading  

Lazy - возвращает Promise, содержащий разрешенный модуль с компонентом экспорта по умолчанию:
```typescript
{/* ленивые компоненты объявляются на верхнем уровне модулей, НЕ внутри других компонентов */}
const Modal = lazy(() => import("./Modal.tsx"))

const MyComponent = () => {
	const [isModalDisplayed, setModalDisplayed] = useState<boolean>(false);
	return (
		<button onClick={() => { setModalDisplayed(true) }}>Загрузить модальное окно</button>
		{isModalDisplayed ? (
			<Suspense fallback="Loading...">
				<Modal />
			</Suspense>
		) : null}
	)
}
```
**Файл использующийся для ленивой загрузке в роутере должен экспортировать Component**: `export const Component = UnnessesaryComponent`  

_В роуте можно передать `<Suspense fallback={}><MyComponent /></Suspense>` в element или вместо element используется lazy_:
```typescript
const router = createBrowserRouter([
  {
    path: "/modal",
    lazy: () => import("./Modal.tsx"),
  },
]); 
```
Также 

Имя чанка определяется в webpack `output: { chunkFilename: 'static/scripts/[name].[contenthash].bundle.js'' }`,  
где `[name]` - абсолютный путь от корня проекта  
Для изменения имени чанка во время динамического импорта в компоненте, используется префикс: `/* webpackChunkName: "chunkName" */`  
```typescript
const loadModalComponent = async () => {
	const loadResult = await import( /* webpackChunkName: "modal" */ '../components/modal');
	setModalComponent(() => loadResult.default); 
}
```
Пример при использовании `lazy`: `const Modal = React.lazy(() => import(/*webpackChunkName: "modal" */ '../components/modal'))`

## 22 Деплой
Имя описываемого процеса: 
`name: Deploy static content to Pages`
On - описание когда запускается процесс (в примере при pull request в ветку main)  
[Список триггеров](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#on)  
```
on:
  pull_request:
    branches:
      - main 
```
jobs объединяет именованные блоки команд  
runs-on - указывает в каком окружении запускать (Ubuntu, Windows и macOS)  
steps - отдельный шаг задачи  
	name - имя шага
	run - команда, запускаемая на шаге
	uses - подключение готового экшена (доступны в [маректплейсе](https://github.com/marketplace?type=actions)
```
jobs: # Секция jobs определяет этапы процесса
	deploy:  # Создаётся одна задача deploy, имя можеть быть любым
		runs-on: ubuntu-latest # Определяет, в каком окружении будут запущены все команды этой задачи
		steps: # Перечень шагов, команд, которые будут выполнены в этом блоке
			- name: Checkout # Шаг для получения доступа к репозиторию
				uses: actions/checkout@v4
			- name: Set up Node
				uses: actions/setup-node@v4
				with:
          node-version: 18
          cache: 'npm'
			- name: Install dependencies
				run: npm ci
jobs:    
	notification:
		needs: deploy # указывает зависимость от другой задачи с именем deploy 
			runs-on: ubuntu-latest
		steps:
			- name: Send message
				run: # команда отправки сообщения
```

Для запуска проекта в GithubPages пробрасывает env переменную в workflow  
```
permissions:
	...
env: 
	PUBLIC_PATH: /github_actions/
jobs: 
	... 
```
В webpack добавляем: 
```
output: {
	publicPath: process.env.PUBLIC_PATH ? process.env.PUBLIC_PATH : '/',
}
```
В роутере обновляем путь 
```
<BrowserRouter basename={process.env.PUBLIC_PATH ? process.env.PUBLIC_PATH : '/'}>
	<App />
</BrowserRouter>
```

## 23 Аналитика

Документация по яндекс метрике и google аналитике 
В ЯП хуки для привязки отслеживаемых событий