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
		- [4.9.1 React.memo](#491-reactmemo)
	- [4.10 useCallback](#410-usecallback)
	- [4.11 useDeferredValue](#411-usedeferredvalue)
	- [4.12 useTransition](#412-usetransition)
	- [4.13 useSyncExternalStore](#413-usesyncexternalstore)
	- [4.14 useId](#414-useid)
	- [4.15 useOptimistic](#415-useoptimistic)
	- [4.16 useDebugValue](#416-usedebugvalue)
	- [4.17 useActionState](#417-useactionstate)
	- [4.18 useFormStatus](#418-useformstatus)
	- [4.19 useEffectEvent](#419-useeffectevent)
	- [4.20 Кастомные хуки](#420-кастомные-хуки)
* [5. Пользовательские компоненты](#5-пользовательские-компоненты) 
	- [5.1 Suspense](#51-suspense)
		- [5.1.1 Lazy Loading](#511-lazy-loading)
	- [5.2 Error Boundary](#52-error-boundary)
	- [5.3 HOC (High Order Component)](#53-hoc-high-order-component)
	- [5.4 Compound component](#54-compound-component)
	- [5.5 Activity](#55-activity)
	- [5.6 ViewTransition](#56-viewtransition)
	- [5.7 Container Presenter](#57-container-presenter-component)
	- [5.8 Render Props](#58-render-props)
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

_Термин Virtual DOM упрощенная модель представления._  
При рендере JSX предварительно преобразуется JSX-transform'ом в вызовы React JSX Runtime (`react/jsx-runtime`), которые создают React elements (RE).  
React element - immutable-объект, описывающий часть UI (`type`, `props`, `key` и т.д.).  
Раньше JSX обычно компилировался в `React.createElement(...)`.  
В современном JSX Transform используются функции JSX Runtime; `React.createElement()` при этом по-прежнему существует как API для ручного создания React elements.  

Каждому RE ставится в соответствие Fiber, т.е. создается Fiber-дерево представлено связанной структурой, похожей на linked list, где корневым элементов Fiber-дерева является root-элемент.
В связанном списке:
* `child` - ссылка на первый дочерний Fiber
* `sibling` - опционально (при наличии) ссылка на следующий Fiber того же уровня
* `return` - ссылка на родительский Fiber

**Fiber** -  внутренний mutable-узел React, представляющий часть render tree.  
Он хранит данные элемента/компонента, состояние, обновления, связи с другими Fiber и информацию о работе, которую React должен обработать во время render/commit phases.

Например, Fiber содержит flags и updateQueue, по которым React определяет необходимые изменения DOM и связанные effects.  
Сам пользовательский запрос к API или подписка не хранятся в Fiber как выполняемая операция; Fiber хранит внутренние данные, связанные с необходимостью обработки соответствующего Effect.  

Состав Fiber [Пруф](https://github.com/react/react/blob/main/packages/react-reconciler/src/ReactFiber.js?utm_source=chatgpt.com):  
```
{
  tag,          // внутренний тип Fiber
  key,          // key React element - уникальный ключ для идентификации (используется для оптимизации обновления списка элементов)
  elementType,  // исходный тип элемента
  type,         // тип компонента / host element
  stateNode,    // связанный host node или экземпляр class component
  
  return,       // родительский Fiber (используется для навигации по "дереву" Fiber)
  child,        // первый дочерний Fiber (используется для навигации по "дереву" Fiber)
  sibling,      // следующий Fiber того же уровня (используется для навигации по "дереву" Fiber)
  index,
  
  pendingProps,   // props , которые компонент должен получить при следующем рендере (временное хранилище до завершения рендера)
  memoizedProps,  // props использованные при последнем рендере (позволяет отслеживать изменения при ререндере)
  memoizedState,  // актуальное состояние Fiber после последнего рендера, используемое для последующих ререндеров
  updateQueue,    // очередь/структура обновлений и effects (содержит список изменений, которые должны быть применены к компоненту)
  dependencies,   // зависимости Fiber, , которые могут вызвать повторный рендеринг, например Context
  
  flags,        // работа/изменения текущего Fiber для commit
  subtreeFlags, // наличие работы в дочернем поддереве
  deletions,    // Fiber-узлы, запланированные к удалению
  
  lanes,        // категории/приоритеты ожидающей работы Fiber
  childLanes,   // ожидающая работа в поддереве
  
  alternate     // соответствующий current/work-in-progress Fiber
  ...
}
```

React elements могут пересоздаваться при каждом render, а React сохраняет соответствующий им логический Fiber, пока элемент остается тем же узлом с точки зрения reconciliation.   
Во время обновления React использует current и work-in-progress версии Fiber, связанные через `alternate`.  
При окончательном unmount соответствующий Fiber больше не является частью активного дерева и со временем может быть освобождён сборщиком мусора.

Общий алгоритм сравнения двух произвольных деревьев может иметь сложность `O(N³)` что неоптимально.  
React использует эвристический алгоритм reconciliation, который снижает сложность примерно до `O(N)`, основанный прежде всего на типе элемента и `key`.    
  
Эвристика (Diffing Algorithm):  
1. Если в одной позиции изменяется тип элемента/компонента (`span => div`), React считает его другим узлом и пересоздает соответствующее поддерево.  
2. Состояние сохраняется, пока React видит тот же компонент в той же позиции render tree.
3. `key` позволяет React различать элементы одного родителя не только по их порядку, но и по указанной идентичности.  
Это помогает сопоставлять элементы между ререндерами и избегать лишнего перемонтирования  

```tsx
// ChildComponent меняет позицию среди children,
// поэтому React перемонтирует его и сбросит локальное состояние/effects
function Parent({ isVisible }: { isVisible: boolean }) {
	if(isVisible) {
		return (
          <>
			<OptionalComponent />
			<ChildComponent />
		  </>
		)
	}
	return (
		<>
			<ChildComponent />
		</>
	);
}

// key позволяет сопоставить ChildComponent,
// даже если его позиция среди children изменилась
function Parent({ isVisible }: { isVisible: boolean }) {
	if(isVisible) {
		return (
          <>
			<OptionalComponent />
			<ChildComponent key='uniqueKey' />
		  </>
		)
	}
	// позиция компонента изменилась со 2 на 1
	return <ChildComponent key='uniqueKey' />
}

// остается вторым дочерним компонентом независимо от условия
function Parent({ isVisible }: { isVisible: boolean }) {
	return (
	  <>
		{isVisible && <OptionalComponent /> }
		<ChildComponent key='uniqueKey' />
	  </>
	)
}

```

Обновление UI происходит в 3 шага:
1. **Trigger Phase** - запуск обновления.  
   Происходит при первоначальном `root.render(...)` или при update состояния компонента.
2. **Render Phase** - React вызывает компоненты, строит новое представление UI и определяет необходимые изменения.  
   При Concurrent Rendering эта фаза может быть прервана, отброшена и запущена заново.
3. **Commit Phase** - React применяет подготовленные изменения к реальному DOM.
   Для конкретного commit эта фаза выполняется непрерывно.  

>**Reconciliation** - процесс сопоставления предыдущего и нового render tree, во время которого React определяет, какие части дерева можно сохранить, обновить или пересоздать.  
>
>Reconciliation выполняется в Render Phase.  
>**Commit Phase** применяет результат Render Phase к DOM.

### Trigger Phase

1. React получает initial render (`root.render(...)`) или state update.
2. Для update определяется `lane` - внутренняя категория приоритета работы.
3. `Lane` отмечается в `fiber.lanes`, а информация о pending work распространяется через `childLanes` вверх по Fiber-дереву до root.
4. React планирует работу root с учётом всех ожидающих lanes.

>**Lanes** - внутренняя система приоритезации обновлений React.
>
>Lanes реализованы как битовая маска.  
Поэтому один Fiber может одновременно содержать несколько pending lanes: новые значения объединяются с уже существующими, а не заменяют их.
Условно:
>```javascript
>SyncLane       = 0001
>TransitionLane = 0100
>
>fiber.lanes    = 0101
>```
>
>`fiber.lanes` содержит pending work самого Fiber.
`childLanes` содержит объединённую информацию о pending work в его дочернем поддереве.  
>
>React может выбрать наиболее срочные lanes для текущего render и оставить менее срочные на потом.  
>
>Например:
>* обычное обновление контролируемого input обрабатывается срочно;
>* state update внутри startTransition относится к Transition work и считается менее срочным;
>* idle/background work может быть отложена ещё сильнее.

### Render Phase  

Render Phase выполняется для выбранных React-ом lanes.  

Если во время работы появляется более срочный update, текущий concurrent render может быть прерван.

1. React подготавливает **WorkInProgress Tree** на основе **Current Tree**.
   Соответствующие Current и WorkInProgress Fiber связаны через поле alternate.  
   React использует double buffering: если alternate ещё не существует, WorkInProgress Fiber создаётся;  
   при следующих render существующая alternate-версия обычно переиспользуется.
2. Выполняется **reconciliation** - React сопоставляет новый render tree с предыдущим и определяет, какие Fiber можно сохранить, обновить, добавить или удалить.
3. На основе результатов reconciliation React устанавливает flags Fiber - битовую маску работы, которую потребуется выполнить в Commit Phase.  
  Например:
   * Placement - вставить новый узел;
   * Update - обновить существующий;
   * ChildDeletion - удалить дочерний Fiber;
   * обработать изменение ref;
   * выполнить связанную effect-работу.
4. При завершении обработки дочерних Fiber их flags и subtreeFlags агрегируются в subtreeFlags родителя.

>`flags` описывает работу самого Fiber.  
`subtreeFlags` показывает, что commit-работа существует где-то в его дочернем поддереве.
>
> Благодаря subtreeFlags Commit Phase может пропускать поддеревья, в которых нет изменений.
> 
> Если во время concurrent Render Phase появляется update с более срочным lane, React может прервать текущий render и обработать более срочную работу.  
> 
> Например, большой список обновляется внутри startTransition.  
> Пока React рассчитывает новый список для значения "a", пользователь вводит следующий символ "b" в контролируемый `<input>`.  
> React может прервать незавершённый Transition-render списка, сначала обработать срочное обновление input, а затем заново рассчитать список уже для "ab".
> 
>После завершения Render Phase WorkInProgress Tree готов к commit.

### Commit Phase

В Commit Phase React применяет подготовленную в Render Phase работу.  

1. **Mutation Phase** - добавляет, удаляет и изменяет необходимые DOM-узлы; применяет связанные изменения ref..
2. **Layout Phase** - выполняет `useLayoutEffect`; вызывает layout lifecycle-методы классовых компонентов: `componentDidMount`, `componentDidUpdate`.

После успешного commit WorkInProgress Tree становится новым Current Tree.

Предыдущий Current Tree обычно сохраняется как `alternate` и может быть переиспользован как WorkInProgress при следующем рендере.  
Т.о. Current и WorkInProgress версии Fiber меняются ролями между обновлениями.

После Commit Phase React отдельно выполняет Passive Effects (useEffect).

**Concurrency** в разрезе reconciliation - возможность React выполнять Render Phase прерываемо.  

Во время concurrent render React может:
* приостановить текущую работу;
* обработать более срочный update;
* затем продолжить или полностью начать предыдущий render заново;
* полностью отбросить незавершенный render, если его результат уже неактуален.

Прерываемыми являются прежде всего неблокирующие concurrent updates:
* state updates, помеченные через `startTransition` / `useTransition`
* background render, создаваемый `useDeferredValue`

Обычные пользовательские updates, например обновление контролируемого input через `useState`, считаются более срочными и могут прерывать Transition/background render.

Приоритет относится не к самому `useState` / `useReducer`, а к конкретному update и контексту, в котором он был создан.

[Более подробно про Concurrency](https://www.youtube.com/watch?v=M1OBMTYsKpo)

**SSR**  
[Про SSR и его преимущества](https://www.youtube.com/watch?v=pj5N-Khihgc)

[Вернуться к содержанию](#содержание)

# 1. Создание компонента

## 1.1 Общее

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
	```tsx
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
	```tsx
	return (
		<>
			<div></div>
			<div></div>
		</>
	)
	```

Компоненты React должны быть **чистыми функциями (pure functions)**:
* при одинаковых входных данных компонент должен возвращать одинаковый результат;
* входные данные (`props`, `context` и другие переданные извне значения) нельзя изменять или мутировать во время render;
* side effects не должны выполняться во время render — их нужно выносить в обработчики событий, Effects и другие подходящие места вне render-логики.

>`<StrictMode></StrictMode>` - включает дополнительные проверки только в development-режиме.
React может дополнительно:
>- повторно вызвать render компонента для поиска нечистой render-логики;
>- выполнить дополнительный цикл `setup → cleanup → setup` для Effects;  
   Цикл `setup->cleanup->setup` в dev-режиме должен соответствовать `setup` в prod-режиме
>- выполнить дополнительный setup/cleanup для callback refs;
>- предупреждать об использовании deprecated API.

При условном рендеринге `(condition) && (statement)` через `&&` следует явно приводить левую часть к boolean, если она может быть числом.
`array.length && <MyComponent />` - при пустом массиве вернёт `0`, и React отрендерит `0`..  

_Решение_: `array.length > 0 && <MyComponent />` или `Boolean(array.length) && <MyComponent />`    

Стили можно импортировать из CSS/SCSS модуля `someStyle.module.scss`  
CSS Module локализует имена классов, генерируя уникальные class names (`ИмяКомпонента_ИмяКласса_хэш`) и предотвращая коллизии между стилями разных компонентов.   
Поэтому внутри CSS Module нет необходимости обеспечивать глобальную уникальность имён через длинные BEM-названия: можно использовать локальные `.root`, `.header`, `.open` и т.д.   

```tsx
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
**Не стоит** объявлять компонент внутри другого компонента.    
При каждом render родителя создаётся новый function object вложенного компонента.  
Для React это новый component type, поэтому вложенный компонент может быть перемонтирован:  
* сбрасывается его state;
* заново выполняются mount/unmount effects;
* повторно обрабатывается его поддерево.  

Это может приводить как к ошибкам со state, так и к лишним затратам при частых ререндерах.  
Компоненты обычно следует объявлять на верхнем уровне модуля.
[Как правильно компоновать компоненты](https://www.youtube.com/watch?v=UWC1XlNJSvc)  

[Вернуться к содержанию](#содержание)

## 1.2 Функциональный компонент

```tsx
type FunctionalComponentProps = { defaultValue: number }
// можно использовать стрелочную функцию
function FunctionalComponent({ defaultValue }: FunctionalComponentProps) {
	// Hooks вызываются на верхнем уровне компонента
	const [value, setValue] = useState(defaultValue)
	// логика
	function myFunc() { setValue( value + 1 ) }
	return (
		<button onClick={myFunc}>{value}</button>
	)
}
```
Примеры типизации функционального компонента:  
```tsx
// 1. Тип props отдельно
type Props = {
	value: number;
	title: string;
};

function MyComponent({ value, title }: Props) {
	return <div>{title}: {value}</div>;
}

// 2. Тип props непосредственно в параметрах компонента
function MyComponent(
		{ value, title }: { value: number; title: string }
) {
	return <div>{title}: {value}</div>;
}

// 3. React.FC - допустимый, но необязательный вариант
const MyComponent: React.FC<Props> = ({ value, title }) => {
	return <div>{title}: {value}</div>;
};
```

[Вернуться к содержанию](#содержание)

## 1.3 Классовый компонент (legacy)

![жизненный цикл классового компонента](https://pictures.s3.yandex.net/resources/Untitled_2_1706861897.png)  
```tsx
// если в компоненте не передаются пропсы, то передается пустой объект {} в дженерике
interface Props { initialValue: number }
interface State { value: number }
class ClassComponent extends React.Component<Props, State> {
	// props передаются в качестве аргументов конструктора	
  	constructor(props: Props) {
		super(props);

		// начальное состояние компонента
		this.state = {
			value: props.initialValue,
		};

		// обычный метод класса теряет this при передаче как callback,
		// поэтому привязываем контекст
		this.increment = this.increment.bind(this);
	}

	// вместо хуков используются методы
	// обычный метод находится в prototype класса
	increment() {
		this.setState((prevState) => ({
			value: prevState.value + 1,
		}));
	}

	// стрелочное class field создаётся на экземпляре
	// и сохраняет this без bind
	decrement = () => {
		this.setState((prevState) => ({
			value: prevState.value - 1,
		}));
	};
      
	// render вызывается React для получения JSX
	render() {
		return (
				<div>
					<p>{this.state.value}</p>
					<button onClick={this.increment}>+</button>
					<button onClick={this.decrement}>-</button>
				</div>
		);
	}
}
```
---

Основные lifecycle-методы:
* `constructor(props)` - создаёт экземпляр компонента, позволяет инициализировать `state` и выполнить `bind` методов.  
  Инициализация `state` по смыслу соответствует `useState`.
* `componentDidMount()` - вызывается после первого commit компонента. Часто используется для подписок, запросов и другой логики после монтирования.   
По смыслу соответствует useEffect с пустым массивом зависимостей.
* `componentDidUpdate(prevProps, prevState)` - вызывается после commit при обновлении компонента.  
`prevProps` - предыдущие пропсы, `prevState` - предыдущее состояние
При первом монтировании не вызывается - вместо него срабатывает `componentDidMount`.  
Если внутри вызывается `setState`, обычно требуется условие, чтобы не создать бесконечный цикл.  
По смыслу соответствует useEffect без зависимостей
* `shouldComponentUpdate(nextProps, nextState)` - позволяет как оптимизацию пропустить render при определённых изменениях props/state.  
Вызывается перед рендером и сравнивает текущие значения props и/или state для разрешения повторного рендеринга (boolean).   
	* true - при изменении props и/или state произойдет повторный рендеринг
	* false - при изменении props и/или state НЕ произойдет повторный рендеринг  
Приблизительный аналог - `React.memo` с функцией сравнения props.
* `getDerivedStateFromProps(props, state)` - редкий метод для вычисления state на основе props перед render.   
* `componentWillUnmount()` - вызывается перед удалением компонента; используется для cleanup: таймеров, подписок, соединений и т.д.    
По смыслу соответствует cleanup-функции `useEffect`.
* `componentDidCatch(error, info)` - вызывается после ошибки в дочернем дереве и позволяет выполнить side effect, например отправить ошибку в систему логирования.  
* `getDerivedStateFromError(error)` - позволяет изменить state при ошибке в дочернем дереве и показать fallback UI.
* `getSnapshotBeforeUpdate(prevProps, prevState)` - вызывается перед применением изменений к DOM; возвращённое значение передаётся третьим аргументом в componentDidUpdate.  
В отличие от `getDerivedStateFromError` позволяет делать внутри side-effects  

---

[useMemo и PureComponent](https://dev.to/nibble/react-memo-and-react-purecomponent-3k7k)  
Для пропуска лишних ререндеров классовый компонент можно наследовать от `React.PureComponent` вместо `React.Component`.
Ререндер происходит, когда:
* изменяется собственный state
* изменяется context/props
* ререндерится родитель

PureComponent в отличие от Component автоматически использует логику аналогичную `shouldComponentUpdate` (поверхностную проверку props и state), чтобы пропускать лишний ререндер самого компонента  
Если стандартного shallow comparison недостаточно, можно явно реализовать `shouldComponentUpdate` и сравнить только необходимые данные.  
Полное глубокое сравнение всех props/state обычно не рекомендуется из-за ресурсозатратности.  

**В функциональных компонентах** близкий аналог `PureComponent` - `React.memo`.   
`React.memo(Component, comparator?: (prevProps, nextProps) => {})` - по умолчанию сравнивает props и позволяет пропустить render при их неизменности.  

Опциональная функция `arePropsEqual(prevProps, nextProps)` задаёт собственную проверку:  
* `true` - props считаются эквивалентными, render можно пропустить;
* `false` - props различаются, компонент ререндерится.

Семантика boolean обратна `shouldComponentUpdate`.

[Вернуться к содержанию](#содержание)

# 2. Props

Props — read-only данные, передаваемые компоненту через JSX.  
```tsx
<CustomComponent attr1={value} attr2={{value: 10}} attr3="text">
	Some Text
</CustomComponent>
```

`children` можно передать как обычный prop или через вложенный JSX:
* `<MyComponent><AnotherComponent /></MyComponent>`  
* `<MyComponent children={...} />`
Если указать оба варианта одновременно, вложенный JSX будет использован как `children`, но смешивать эти способы не следует  
Способы передачи props:
* фигурные скобки `{}` - переменные `<MyComponent loadintState={isLoading} />`
* двойные фигурные скобки `{{}}` - объекты `<MyComponent personData={{name: 'Petr', age: 18}} />`
* кавычки `""` - текст `<MyComponent title="Some Text"/>`

> Изменение переданных `props` является одной из причин повторного render компонента. 

Деструктурируя `...props`, можно выделить необходимые для передачи в дочерний компонент пропсы
```tsx
const ParentComponent = ({children, ...props}: ParentComponentProps) => {
	return (
		<ChildComponent key={props.id}>{children}</ChildComponent>
		{/* передача оставшихся props без перечисления каждого отдельно */}
		<AnotherComponent {...props} />
	)
}
```

Пропсы передаются сверху вниз (от родительского компонента к дочернему).  
Для передачи данных из дочернего компонента в родительский родитель передаёт `callback` (функцию обратного вызова) через props, а дочерний компонент вызывает его с нужными параметрами  
**При передаче callback нужно передавать саму функцию `getValue={myFunc}`, а не результат её вызова `getValue={myFunc()}`.**  
```tsx
function App() {
	const myFunc = () => {...логика...}
	return (
		<MyComponent getValue={myFunc} />
	)
}
```

## Пример

Функцию обратного вызова можно реализовать в родительском компоненте, а в дочернем передавать в нее параметры  
```tsx
function App() {
	return (
		<MyComponent getValue={(param1, param2) => {...логика функции}}></MyComponent>
	)
}

const MyComponent = ({getValue}: MyComponentProps) => {
	return (
		<button onClick={() => getValue(param1, param2)}>Click</button>
	)
}
```

**Частая проблема** - большой уровень вложенности и связанный с этим `Props drilling`.  
**Props Drilling** - передача props через промежуточные компоненты, которым эти данные не нужны, только для передачи их более глубоко вложенному компоненту.   
Одно из решений проблемы: реорганизовать компоненты и поднять их на уровень выше  
Необходимые пропсы можно будет использовать сразу на вложенных компонентах, без передачи через промежуточные компоненты  
```tsx
<OuterComponent>
	<InnerComponent>
		<InnerHeader/>
		<InnerContent text={text} />
	</InnerComponent>
</OuterComponent>
```

>Props в React следует считать read-only.  
**Сам компонент не должен изменять полученные props.**    
При этом если через prop передан изменяемый объект, JavaScript технически позволяет мутировать его вложенные свойства.  
Но делать этого не следует: объект принадлежит внешнему коду, а мутация нарушает принцип чистого компонента. 
>[Видео разбор статьи о модифицировании пропсов](https://www.youtube.com/watch?v=jDHBY6tV2SE)  
>```tsx
>type User = {
>	username: string;
>}
>const Index = (props: {user: User}) => {
> // не приводит к ошибке
>	props.user.username = `${props.user.username} Some Text`;
>	return <div>{props.user.username}</div>;
>}
>```

Плавное введение в клиент/серверные компоненты и передачу пропсов между ними  
[Виды компонентов и передача пропсов между ними](https://www.youtube.com/watch?v=F0ZvDcOuBdo)

[Вернуться к содержанию](#содержание)

# 3. Списки

При создании списков необходимо задавать атрибут `key` с уникальным значением среди siblings в данном списке.  
Индекс итерации не рекомендуется использовать в качестве ключа, т.к. элементы могут добавляться/удаляться, что изменит их ключ.  
>`Crypto.randomUUID()` или пакет `uuid` могут использоваться для создания уникальных id.  
>Но такие id лучше создавать в момент создания данных, а не генерировать заново при каждом рендере.
```tsx
function App() {
	const [array, setArray] = useState([{id: 1}, {id: 2}]) // массив элементов
	return (
		<div className="container">
			{/* index допустим, если список действительно статичен: элементы не добавляются, не удаляются и не меняют порядок, и стабильного ID нет. */}
			{array.map((item, index) => <CustomComponent id={item.id} key={index}>Some Text</CustomComponent>)}
		</div>
	)
}
```
Добавление нового элемента в список:
```tsx
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

`const [state, setState] = useState<TState>(defaultValue)` - управляет состоянием компонента, где
* `state` - текущее состояние
* `setState` - функция сеттер для изменения состояния
* `defaultValue` - начальное значение состояния (можно использовать функцию)  

При изменении state React планирует повторный render компонента.  
По умолчанию при render родителя также рендерится его дочернее дерево, если обновление не было пропущено оптимизациями React.

>При передаче в setState неизменившегося состояния React пропускает ререндер компонента и его дочернего дерева.  
При этом функция компонента может быть вызвана повторно, но результат render будет отброшен.  
>
>Пример:
>```tsx
>const [state, setState] = useState(0)
>const ref = useRef(0)
>
>ref.current += 1
>{/* при выполнении handleClick мы передаем то же состояние - нового рендера не происходит */}
>{/* НО вычисления: current.ref += 1 произойдут */}
>handleClick() {
>	setState(state)
>}
>```
>При вызове `handleClick` новое состояние совпадает с текущим.  
React может повторно вызвать функцию компонента, поэтому `ref.current += 1` успеет выполниться, но результат render будет отброшен и commit не произойдёт

>Если начальное значение `useState` вычисляется дорого, например создается большой массив `useState([0, 1, ..., 999999])`, лучше передавать функцию-инициализатор:
>  * `useState(() => createInitialState())` - функция будет вызвана только при первом рендере
>  * `useState(createInitialState())` - `createInitialState()` будет вычисляться при каждом ререндере компонента  
>Это не переинициализирует state на каждом ререндере, но создает лишние вычисления.

Напрямую изменять (mutate) состояние нельзя, поэтому любые *мутирующие* методы (sort, reverse и т.д.) должны вызываться на копии  
[Проверка методов на мутации](https://doesitmutate.xyz/)  

Для объектов - `setState({...state, stateArg: newValue})` или для массивов - `setState([...state, newValue])`  
Спред-синтаксис действует поверхностно  
Изменение состояния вложенных полей (при большом уровне вложенности) увеличивает и дублирует код: `setState({...state, innerObj: {...state.innerObj, innerArg: newValue}})`  

**Важно** не забывать, что объект - ссылочный тип:  
```tsx
const [state, setState] = useState([{innerArg: 1}, {innerArg: 2}])
function handleClick() {
	// ! ОШИБКА ! Происходит мутация ссылочного типа (объекта)
	const newValue = [...state]
	newValue[0].innerArg = 10 
	setState(newValue)

	// Правильный вариант: использование не мутирующего метода
	const newState = state.map((item) => {
		return (item.innerArg === 1) ? {...item, innerArg: 10} : item
	})
	setState(newState)
}
```

### 4.1.1 Immer

[Immer](https://github.com/immerjs/use-immer) - упрощает изменение состояния с вложенными объектами  
Immer позволяет писать код в мутационном стиле через draft, но создаёт новое immutable state с сохранением неизменённых частей структуры.   
Immer не запрещает использовать стандартный синтаксис. Выбор оправдан, если количество кода уменьшается при изменении состояния  

**Пример с объектами:**  
```tsx
const [state, updateState] = useImmer({
	outerArg: outerValue,
	innerObj: { innerArg: innerValue }
})
updateState((draft) => draft.innerObj.innerArg = newValue)
```
**Пример с массивами:**   
```tsx
const [state, updateState] = useImmer([{innerArg: 1}, {innerArg: 2}])
updateState((draft) => {
	const item = draft.find((s) => s.innerArg === 1)
	item.innerArg = 10
})
```

### 4.1.2 Batching

**Изменение состояние происходит НЕ мгновенно**. Setter запрашивает обновление state для следующего render.
State фиксируется для конкретного render, поэтому обработчики и асинхронные callbacks используют snapshot state того render, в котором они были созданы.  
```tsx
const handleClick = () => {
	setState(newState);
	setTimeout(() => {
	{/* в таймере будет использовано текущее значение state, а не newState */}
		...обработка state
	}, delay)
}
```

**Batching** - объединение нескольких state updates перед следующим render, чтобы избежать лишних промежуточных ререндеров.  

Если setter вызван внутри обработчика событий, то React ждёт завершения кода внутри текущего обработчика перед применением queued state updates.
Последовательность нескольких setter внутри обработчика не важна.  
React не выполняет batching между отдельными намеренными пользовательскими событиями, например двумя отдельными кликами.

_Начиная с React 18 automatic batching также распространяется на асинхронные операции (Promise, setTimeout, native event handlers и др.)._  

[Хорошее погружение в batching](https://www.youtube.com/watch?v=VfQ-qSjIalU)  
[Разбор поведения batching с microtasks/macrotasks и таймерами](https://dzen.ru/a/ZfsEbpisFmwi6p2h)  
_Детали взаимодействия scheduler с microtasks/macrotasks и таймерами могут зависеть от окружения и не являются гарантированным API-контрактом React._  
**Краткий пример по таймерам:**  

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
```tsx
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
	{/* при наличии микротаски `asyncFunc` со сложными вычислениями больше времени задержки таймеров (в данном примере 200мс), три таймера могут вызвать 1 ререндер */}
	setTimeout(() => {setState1(20)}, 200)
	{/* при наличии микротаски `asyncFunc` со сложными вычислениями таймер с нулевой задержкой может вызвать собственный ререндер */}
	{/* в разных браузерах может вести себя по-разному */}
	setTimeout(() => {setState1(30)}, 0)
	{/* асинхронная функция вызывает собственный ререндер */}
	asyncFunc().then(() => {}
}
```

Для многократного изменения состояния до следующего render следует использовать updater function - `setState(prev => prev + 1)`, если новое значение зависит от предыдущего.
Updater-функции помещаются в очередь и во время следующего render последовательно вычисляют новое состояние.  

**Пример**:  
В новом рендере состояние будет равно `newValue`, так как все изменения состояния были добавлены в очередь
```tsx
const [state, setState] = useState(0)

setState(state + 1)          // 0 + 1
setState(state + 1)          // 0 + 1
setState(prev => prev + 1)   // 1 + 1
setState(newValue)           // итоговое значение newValue
```

---

State сохраняется, пока React считает компонент тем же узлом в render tree.  
Изменение identity компонента, в том числе key, может сбросить его state. [Подробнее: Reconciliation](#react-под-капотом)  

> Если React уже начал выполнять тело конкретного function component, выполнение функции синхронно и не прерывается посередине.    
Concurrent Render Phase может приостановиться после завершения текущей Fiber-работы.

### 4.1.3 flushSync

`flushSync(callback)` - принудительно синхронно выполняет React updates внутри callback и применяет необходимые изменения к DOM до выполнения следующего кода.  

Используется редко, когда следующий код должен работать уже с обновлённым DOM, например при интеграции с browser API или сторонней библиотекой. Частое использование может ухудшить производительность.  

```tsx
const [isOpen, setIsOpen] = useState(false)
const inputRef = useRef<HTMLInputElement>(null)

function handleOpen() {
	flushSync(() => {
		setIsOpen(true)
	})

	// DOM уже обновлён: input существует и доступен через ref
	inputRef.current?.focus()
}

return (
		<>
			<button onClick={handleOpen}>Open</button>
			{isOpen && <input ref={inputRef} />}
		</>
)
```

Без flushSync React может отложить применение setIsOpen(true) из-за batching, поэтому на следующей строке <input> ещё может отсутствовать в DOM.  

### 4.1.4 Ошибки

**Основные ошибки** при создании состояний: 
* Большое количество state, обновляемых одновременно - решение: объединение в один объект state
* Взаимоисключающие state - решение: можно объединить в один state
* Дублирующие state - решение: можно вычислить как переменные на основе других state
* State дублирует передаваемый props - решение: можно использовать, если не требуется обновление состояния при изменении props
* Дублирование объекта в state - решение: можно хранить только значимое поле вместо всего объекта, например id
* Большая вложенность в state - решение: можно нормализовать вложенность (сделать плоским объектом)  

>Плохой паттерн: `setState` внутри updater function другого `setState`  
>updater function в React должна быть pure: она должна только вычислить и вернуть новое состояние.  
> [Пруф](https://react.dev/learn/queueing-a-series-of-state-updates#what-happens-if-you-replace-state-after-updating-it)

```tsx
setA(prevA => {
setB(prevB => prevB + 1);
return prevA + 1;
});
```

[Вернуться к содержанию](#содержание)

## 4.2 useReducer

[Детальный видео-разбор](https://www.youtube.com/watch?v=rgp_iCVS8ys)  
[отличие useState от useReducer(видео)](https://www.youtube.com/watch?v=3VClygDRSsU)  
[Еще одно видео-разбор](https://www.youtube.com/watch?v=kK_Wqx3RnHk)  
`const [state, dispatch] = useReducer(reducerFunction, initialState, initFunc?)` - аналогично useState
* `state` - текущее состояние
* `dispatch` - функция, запускающая reducerFunction
* `reducerFunction` - функция (возвращает новое состояние), объединяющая логику изменения состояния в зависимости от условий **(чистая функция без side-эффектов)** 
* `initialState` - начальное значение состояния редьюсера
* `initFunc` - необязательная функция ленивой инициализации, вычисляющая начальное state на основе `initialState`  

Важно передавать функцию, а не результат её вызова — тогда initFunc будет вызвана только при инициализации state.  
Если initFunc объявлена внутри компонента, новый function object будет создаваться при каждом render, хотя сама функция как initializer выполняется только при инициализации state.    
Аналогично useState, значение второго аргумента вычисляется при каждом render до вызова useReducer, поэтому дорогое начальное вычисление лучше выносить в initFunc:  
`const [state, dispatch] = useReducer(reducer, initialArg, initFunc)`

`dispatch({type: string[, payload: data]})`, где type - тип изменения, payload - передаваемые данные  
Используется для объединения логики обновления состояния в одну функцию  

| Критерий | `useReducer` | `useState` |
|---|---|---|
| Логика обновления | Много связанных способов изменения state, удобно объединить их в reducer | Простые и независимые изменения |
| Количество сценариев изменения | Много разных actions / условий перехода | Небольшое количество простых setter-вызовов |
| Расположение логики | Логика изменения вынесена в одну pure reducer-функцию | Логика обычно находится непосредственно в handlers |
| Тип state | Любой: primitive, object, array и т.д. | Любой: primitive, object, array и т.д. |
| Удобство | Лучше структурирует сложные переходы состояния | Проще и требует меньше кода для простых случаев |

Получить обновленное значение состояния в текущем состоянии (до рендера) можно вызвав `reducerFunction`: `const nextState = reducerFunction(state, action)` 

**Типизация useReducer**:
1. Типизация состояния: `type BookState = { price: number, title: string }`
2. Типизация action для reducer
```tsx
export type IsNever<T, True, False> = [T] extends [never] ? True : False;

// Тип события. Type — название, Payload — данные
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
3. Типизация reducerFunction
```tsx
type BookAction =
    | Action<'INCREMENT'>
    | Action<'DECREMENT'>
    | Action<'SET_TITLE', string>; 
``` 
4. Реализация reducerFunction
```tsx
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
5. Инициализация useReducer: `const [state, dispatch] = useReducer(bookReducer, {price: 100, title: 'Book Title'})`
6. Вызов диспатчера в коде
```tsx
function handleClick() {
	dispatch({
		type: 'SET_TITLE',
		payload: 'Another Book Title'
	})
}
```
7. Типизация dispatch при передаче в дочерний компонент: `dispatch: Dispatch<BookAction>`

---

Immer включает в себя useImmerReducer: 
```tsx
const [state, dispatch] = useImmerReducer(reducerFunction, initialState)
function reducerFunction(draft, action) {
	switch (action.type) {
		case 'added': {
			draft.push({ id: action.id, text: action.text});
			break;
		}
	}
}
```

[Вернуться к содержанию](#содержание)

## 4.3 useContext

[детальный разбор(видео)](https://www.youtube.com/watch?v=HYKDUF8X3qI)  
[лучший разбор(видео)](https://www.youtube.com/watch?v=I7dwJxGuGYQ)  
[еще один(видео)](https://www.youtube.com/watch?v=5LrDIWkK_Bc)  

`const context = useContext(SomeContext)` - позволяет получить доступ дочерним компонентам к значению ближайшего родительского контекста, не используя props    
>Изменение значения контекста ведет к обновлению компонентов, использующих этот контекст.  
При этом обычные дочерние компоненты провайдера тоже могут ререндериться из-за ререндера родительского дерева.  

Используется, когда данные нужны глубоко вложенным или нескольким компонентам и их приходится передавать через промежуточные компоненты, которым эти данные сами не нужны.  
Context часто используют для общих данных, необходимых многим компонентам: тема, язык, текущий пользователь и т.д.  
Context можно использовать и для часто изменяющихся данных, но при изменении `value` обновляются все компоненты, читающие этот context. Поэтому для большого или часто изменяющегося shared state могут быть удобнее разделение контекстов или специализированные state-management библиотеки (`Redux`, `Zustand` и т.д.).
```tsx
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
<ThemeContext value={{state, setState}}>
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
```tsx
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
```tsx
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
```tsx
// MyComponent.tsx
export function MyComponent() {
	const [value, setValue] = useState(0);
	// объект создается заново при каждом рендере компонента, что ведет за собой изменение контекста
	const contextValue = { value };
	// useMemo сохранит ссылку на ссылочный тип и не будет создавать его каждый раз с одинаковыми данными
	const contextValue = useMemo<ContextProps>(() => ({ value }), [value]);
	return (
		<ThemeContext value={contextValue}>
			<ChildComponent />
		</ThemeContext>
	)
}
```
**Второе важное замечание**  
[Правильная реализация контекста](https://www.youtube.com/watch?v=16yMmAJSGek)  
Имеются два компонента: использующий контекст `ComponentUseContext` и не использующий контекст `ComponentDontUseContext`
При данной реализации оба компонента **будут ререндериться** при изменении контекста, т.к. изменяется state:
```tsx
export const CustomContext = createContext<{state: number, setState: React.Dispatch<React.SetStateAction<number>>} | undefined>(undefined);

const App = () => {
	const [state, setState] = useState(0)
	return (
		<CustomContext value={{ state, setState }}>
			<ComponentUseContext />
			<ComponentDontUseContext />
		</CustomContext>
	)
};
```

Изоляция ререндеров Provider.  
Компонент, не использующий useContext, не будет ререндериться при изменении контекста:
```tsx
// custom-context.tsx
export const CustomContext = createContext<{state: number, setState: React.Dispatch<React.SetStateAction<number>>} | undefined>(undefined)

export const CustomContextProvider = ({ children }) => {
	const [state, setState] = useState(0)
	return <CustomContext value={{ state, setState }}> {children} </CustomContext>
}

// MyComponent.tsx
const App = () => {
	return (
		<CustomContextProvider>
			<ComponentUseContext />
			<ComponentDontUseContext />
		</CustomContextProvider>
	)
}
```

**Переопределение контекста**
Дочерние компоненты обращаются к ближайшему родительскому контексту, поэтому можно переопределить провайдер для части дерева, обернув ее в провайдер с другим значением value   
```tsx
<CustomContext value={0}> 
	...
	<CustomContext.Provider value={1}> 
		<MyComponent />
	</CustomContext.Provider> 
	...
</CustomContext> 
```

При использовании множественных контекстов их выносят в отдельный компонент обертку  
```tsx
<Component1 value={}>
	<Component2.Provider value={}>
		<Component3.Provider value={}>
			<MyComponent />
		</Component3.Provider>
	</Component2.Provider>
</Component1 >
```

[Вернуться к содержанию](#содержание)

## 4.4 useRef

[детальный разбор(видео)](https://www.youtube.com/watch?v=42BkpGe8oxg)  
[еще один(видео)](https://www.youtube.com/watch?v=t2ypzz6gJm0)  
`const ref = useRef<TSomeType>(initialValue)` - хранит mutable значение между ререндерами, изменение которого не вызывает rerender. Также используется для доступа к DOM-элементам.  
`useRef` возвращает объект с полем `current`, в котором можно хранить данные между ререндером, не используя state  

```tsx
const [state, setState] = useState(0)
const ref = useRef(0)
const handleClick = () => {
	setState(prev => prev + 1)
	ref.current += 1
	// 0, значение текущего snapshot. Оно изменится после завершения кода обработчика и ререндера
	console.log(state)
	// 1, значение изменяется сразу, так как не зависит от рендера
	console.log(ref.current)
}
```

Начальное значение `ref.current` используется только при первом render.  
Если начальное значение вычисляется вызовом функции: `const ref = useRef(MyFunc())`, то MyFunc() будет выполняться при каждом render, хотя React использует результат только для первоначальной инициализации ref.  

Для дорогой ленивой инициализации:
```tsx
const ref = useRef(null)
if(ref.current === null) ref.current = MyFunc()
```

В `ref.current` можно хранить значение любого типа.  
Ref можно передать как ref-объект (`useRef`) или как callback-функцию: `<input ref={(node) => {...}} />` 
  
>Callback ref вызывается в commit phase после изменения DOM и до `useLayoutEffect` / `useEffect`. 

_В React 19 callback ref может возвращать cleanup-функцию, которая вызывается при очистке ref._

Если callback ref объявлен inline, при каждом render создаётся новая функция. React очищает предыдущий ref и устанавливает новый callback ref, поэтому callback может вызываться повторно.    
При необходимости стабильного callback ref его можно мемоизировать: `const callbackRef = useCallback((element) => { console.log(element) }, [])`
[Подробно про проблемы рефов и решения через колбек рефы](https://www.youtube.com/watch?v=MLWsLn_jeGc)  
[useForkRef - кастомный хук](https://ollylut.medium.com/what-is-useforkref-hook-4be1c85d2d1b "VPN")  

```tsx
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
			<input ref={memoCallback} />
			<button onClick={forceUpdate}>forceUpdate</button>
		</>
	)
}
```

>В React 19 `ref` можно передавать в function component как обычный prop.  
`forwardRef` - deprecated

В class component `forwardRef` не требуется: переданный `ref` ссылается на экземпляр класса.
```tsx
function App() {
	const myRef = useRef<HTMLInputElement>(null)

	function getValue() {
		return myRef.current?.value
	}

	return <MyComponent ref={myRef} />
}
// MyComponent
type MyComponentProps = {
	ref?: React.Ref<HTMLInputElement>
}

function MyComponent({ ref }: MyComponentProps) {
	return <input ref={ref} />
}
```
 
В классовых компонентов используется `createRef` внутри конструктора - `this.myRef = React.createRef()`:  
```tsx
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
```tsx
//Functional Component
export const MyComponent = () => {
	// ref на экземпляр классового компонента
	const buttonRef = useRef<Button>(null)
	useEffect(() => {
		// type guard и использование кастомного метода компонента через ref 
		buttonRef.current && buttonRef.current.setFocus()
	})
	return (
		// связывание классового компонента с ref
		<Button ref={buttonRef} />
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

`useImperativeHandle(ref, createHandle, deps?)` - позволяет дочернему компоненту определить собственный API, доступный родителю через `ref.current`:  
методы для управления внутренним состоянием, DOM или другой внутренней логикой.
* `ref` - ref, полученный дочерним компонентом от родителя
* `createHandle` - функция, возвращающая значение, которое будет доступно родителю через `ref.current`
* `deps` -  зависимости `createHandle`; при их изменении React создаёт новый handle и присваивает его `ref`

**Переданный ref не обязательно напрямую привязывать к HTMLElement дочернего компонента.**     
Через `useImperativeHandle` дочерний компонент может сам определить, какие методы или значения будут доступны родителю через `ref.current`, не раскрывая напрямую их реализацию.
```tsx
export type MyRef = {
	reset: () => void
	focus: () => void
}

const ParentComponent = () => {
	const ref = useRef<MyRef>(null)
	return (
		<>
			<ChildComponent ref={ref} />
			{/* Сбрасываем состояние дочернего компонента из родительского без поднятия состояния */}
			<button onClick={() => ref.current?.reset()}>Reset</button>
			<button onClick={() => ref.current?.focus()}>Reset</button>
		</>
	)
}

type ChildProps = {
	ref?: React.Ref<MyRef>
}
function ChildComponent({ ref }: ChildProps) {
	const [state, setState] = useState({name: 'Petr'})
	// Для привязки переданного ref к HTMLElementу необходимо создать новый локальный ref, так как в кастомизируемом нет методов HTMLElement
	const localRef = useRef<HTMLInputElement>(null);
	const reset = useCallback(() => {
		setState({ name: '' })
	}, [])

	// ограничиваем родительский компонент двумя доступными методами
	useImperativeHandle(ref, () => ({
		reset,
		// Добавляем свойства InputElement к кастомизируемому ref 
		focus() {
			localRef.current?.focus();
		}
	}), [reset])
	return <input ref={localRef} />
}
```

[Вернуться к содержанию](#содержание)

## 4.6 useEffect

[В этот раз недостаточно подробно(видео)](https://www.youtube.com/watch?v=-4XpG5_Lj_o)  
[Еще одно(видео)](https://www.youtube.com/watch?v=0ZJgIjIuY7U)  

`useEffect(callback, deps?)` - запускает побочный эффект после commit и используется для синхронизации компонента с внешней системой.  
* callback - функция (побочный эффект), выполняющаяся после рендера компонента
* deps - массив зависимостей
  * [dep1, dep2] - при изменении dependencies
  *	[] - после mount
  * без deps - после каждого commit

>`useEffect` срабатывает **ПОСЛЕ** рендера компонента   
	
`useEffect` не стоит применять для обработки событий пользователя (нужно вынести в обработчик), для преобразования данных для рендеринга (вынести на верхний уровень компонента)  
Также не стоит создавать цепочки из useEffect изменяющие части состояния на основе других состояний - это приводит к лишним ререндерам  

Dependencies сравниваются через `Object.is`. Для объектов, массивов и функций сравнивается ссылка, поэтому новое ссылочное значение может перезапустить Effect даже при одинаковом содержимом.   

`useEffect` может возвращать cleanup-функцию:  
cleanup - функция очистки вызывается при unmount и перед повторным запуском Effect после изменения dependencies.
* без dependencies - cleanup выполняется перед каждым следующим запуском Effect и при unmount;
* с `[deps]` - cleanup выполняется перед повторным запуском Effect после изменения dependencies и при unmount;
* с `[]` - cleanup выполняется при unmount (в development `StrictMode` также выполняется дополнительный цикл `setup → cleanup → setup`).

```tsx
useEffect(() => {
	/* cleanup */
	return () => {}
}, [])
```

`Strict mode` - development React специально выполняет дополнительный setup → cleanup → setup, чтобы проверить корректность cleanup.
Функция очистки должна быть вызвана для закрытия соединения с сервером, остановки таймеров, отписки от событий, установки анимаций в начальную фазу и т.д.  
Например:  
```tsx
const [human, setHuman] = useState('Petr')
const [age, setAge] = useState(18)
useEffect(() => {
	let ignore = false
	fetchHuman(human).then((result) => {
		if(!ignore) setAge(result)
	})
	// Каждый вызов fetch имеет собственную переменную ignore, которая блокирует запись состояния age, если изменилось состояние human
	return () => {ignore = true}
}, [human])
```

[Перейти к useLayoytEffect](#47-uselayouteffect)
[Вернуться к содержанию](#содержание)

### 4.6.1 Случаи неправильного использования useEffect

`useEffect` не стоит использовать в случаях:  

1. Обновление состояние на основе пропсов или другого состояния  

```tsx
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
```tsx
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
```tsx
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
```tsx
const List= ({ items }) => {
	// 🔴 useEffect срабатывает после рендера, все дочерние компоненты получат старые значения
	const [selection, setSelection] = useState(null);
	useEffect(() => {
    setSelection(null);
  }, [items]);
	
	// ✅ Можно сохранить предыдущее состояние, но это вызывает ререндер
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
```tsx
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
```tsx
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
```tsx
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
```tsx
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
```tsx
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
```tsx

// 🔴 Лишняя синхронизация данных вверх через Effect.
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
```tsx
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
```tsx
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

`useLayoutEffect(callback, deps?)` - запускается после commit DOM-изменений, но до browser repaint.
* `callback` - функция, выполняющая до рендера компонента
* `deps` - массив зависимостей изменениение которых ведет к вызову callback  

```tsx
useLayoutEffect(() => {
	const {height, width} = ref.current.getBoundingClientRect()
	setSize({height, width})
}, [])
```

> `useLayoutEffect` выполняется раньше `useEffect`: layout effects запускаются синхронно до repaint, а passive effects (`useEffect`) - позже.
Эффект блокирует отрисовку элементов, поэтому не рекомендуется использовать длительные операции  

Используется для измерения DOM и синхронной корректировки layout до browser repaint, чтобы пользователь не увидел промежуточное состояние.  

`useLayoutEffect` не выполняется при SSR: Effects работают только на клиенте, т.к. на сервере нет layout DOM.  
Решение: 
* использовать `useEffect`
* использовать `Suspense` и показывать фолбэк, не использующий `useLayoutEffect`
* проверять состояние `isMounted` и отображать фолбек, пока не произойдет гидратация  
>Гидратация - подключение React к HTML, который был сгенерирован на сервере  

[Вернуться к содержанию](#содержание)
	
## 4.8 useInsertionEffect 

`useInsertionEffect` выполняется раньше layout Effects и предназначен прежде всего для CSS-in-JS библиотек.  
Последовательность: `useInsertionEffect > useLayoutEffect > useEffect`  
 
>В useInsertionEffect refs ещё не обязательно attached, поэтому нельзя полагаться на DOM/ref как в useLayoutEffect.

[Подробнее в документации](https://react.dev/reference/react/useInsertionEffect)  
[Нестандартное применение (малоприменимо)](https://www.youtube.com/watch?v=ILg1zhl92AI&pp=ygUSdXNlSW5zZXJ0aW9uRWZmZWN0)  

[Вернуться к содержанию](#содержание)

## 4.9 useMemo

[Подробный разбор(видео)](https://www.youtube.com/watch?v=vpE9I_eqHdM)  
[Разбор документации по шагам с примерами(видео)](https://www.youtube.com/watch?v=oMvW3A_IRsY)  

`useMemo(calculateValue, deps)` - локально (только в компоненте, вызвавшем хук) мемоизирует **результат** вычислений.
* `calculateValue` - чистая функция, которая должна возвращать результат вычислений
* `deps` - массив зависимостей изменениение которых ведет к пересчету calculateValue  

`useMemo(() => ({name: 'Petr'}), [])`

`useMemo` используют для кэширования дорогих вычислений или для другой оптимизации (memo, dependencies других hooks).   
Также при передаче объекта/массива как props их нужно мемоизировать (т.к. `{} !== {}`) для сохранения стабильной ссылки на значение.
>При мемоизации функции сохраняется **результат** ее вычислений  

Проверить длительность вычислений можно включив CPU Throttling в DevTools или аналог:  
```tsx
console.time('start');
... вычисления
console.timeEnd('end');
```
>**Не нужно мемоизировать все подряд.** Это трата памяти на сохранение всех результатов мемоизации  

>При включённом React Compiler значения и вычисления во многих случаях мемоизируются автоматически, поэтому необходимость в ручном useMemo уменьшается.   

[Вернуться к содержанию](#содержание)

### 4.9.1 React.memo

`React.memo(component, arePropsEqual?)` - создаёт мемоизированную версию компонента и позволяет пропустить его rerender при неизменившихся props.  
Не является хуком.
* `component` - компонент для мемоизации  
* `arePropsEqual(prevProps, nextProps)`: - функция сравнения пропсов, передаваемых в component, для принятия решения о ререндере  
  Применяется, когда компонент часто ререндерится вместе с родителем, но его props обычно остаются неизменными и сам render достаточно дорогой.  
  * `true` - props считаются эквивалентными, rerender можно пропустить
  * `false` - props отличаются, компонент ререндерится
    
```tsx
import {memo} from "react";

const MyComponent = memo(({ data }: TSomeProps) => (
	<div>{data}</div>
)); 
```

>`memo` сравнивает только props. Собственный state и используемый context всё равно могут вызвать rerender.  

> При включённом React Compiler необходимость в ручном React.memo обычно сильно уменьшается: compiler автоматически применяет эквивалентную оптимизацию.

Аналогом `memo` с одним аргументом является PureComponent в классовом компоненте - `class MyComponent extends PureComponent {}`  
Альтернативно в классовом компоненте можно кастомизировать `shouldComponentUpdate`, который автоматически вызывается в PureComponent (См. [Классовый компонент](#13-классовый-компонент-устаревший))  
`shouldComponentUpdate` должен вернуть boolean, который определит нужен ли ререндер компоненту
```tsx
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
* `fn` - значение функции для мемоизации. 
>Возвращает функцию, НЕ вызвает ее.  
* `deps` - массив зависимостей изменениение которых ведет к вызову `fn`

>Часто используется вместе с memo, а также когда стабильная ссылка на функцию нужна как dependency другого hook или часть API custom hook.   
При передаче в `memo` inline-функцию НЕ обернутых в `useCallback` мемоизация компонента не имеет смысла, т.к. функия - ссылочный тип  

```tsx
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

```tsx
const dataMemo = useMemo(() => {
	return computeFunc(data)
}, [data])

const dataCallback = useCallback(
	(params) => {
		post(url) {}
	}, [url]
)
```

**Важно помнить**, что без использования зависимостей функция замыкается на значениях из render, в котором была создана:
```tsx
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

Зависимости `useCallback` должны включать все reactive values. Уменьшать их количество можно рефактором:
1. Функция-апдейтер помогает избавиться от лишних зависимостей:
```tsx
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
```tsx
const myFunc = useCallback(() => {}, [deps])
useEffect(() => {myFunc()}, [myFunc])
```
Устраним лишние зависимости, добавив функцию в useEffect:
```tsx
useEffect(() => {
	const myFunc = () => {}
	myFunc()
}, [deps])
```

>Функции, которые custom hook возвращает наружу, часто стоит оборачивать в useCallback, чтобы пользователь hook мог при необходимости оптимизировать зависимости и memoized components.  

[Вернуться к содержанию](#содержание)

## 4.11 useDeferredValue

[хороший разбор(видео)](https://www.youtube.com/watch?v=yIpHTYo3PY0)  
[еще один(видео)](https://www.youtube.com/watch?v=jCGMedd6IWA)  

`const deferredValue = useDeferredValue(value, initialValue)` - позволяет отложить обновление некритичной части интерфейса и выполнять её render в фоне, не блокируя более срочные обновления.    
В отличие от debounce/throttle, useDeferredValue не использует фиксированную задержку и откладывает именно render, а не само изменение значения или сетевой запрос
* `value` - текущее значение, для которого React должен предоставить отложенную версию  
При обновлении `value`, React рендерит компонент со старым deferred value, затем запускает фоновом рендеринг с новым значением.  
Если value изменится во время background render, React прервёт устаревший render и начнёт новый с актуальным значением.    
* `initialValue` - необязательное значение, которое useDeferredValue вернёт на первом render вместо value;  
после этого React запустит background render с актуальным value.

При отложенном обновлении пользователь видит предыдущие результаты, пока не будут готовы новые  
Например, если изменение `value` приводит к тяжёлому render дочернего компонента, можно передать ему `deferredValue`.  
Тогда срочное обновление, например ввод в input, произойдёт сразу, а тяжёлая часть интерфейса обновится в background render.
Для этого `useDeferredValue` часто комбинируется с `memo`.
```tsx
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
`const [isPending, startTransition] = useTransition()` - позволяет обновить состояние без блокировки интерфейса.   
Помечает обновления состояния как переход, позволяя React отложить их render и сначала обработать более срочные обновления интерфейса.    
* `isPending` - boolean флаг, сообщающий статус перехода
* `startTransition` - функция, принимающая синхронный или асинхронный callback и помечающая выполненные внутри него state updates как Transition    
Пример использования: навигации по страницам  

>В отличие от хука `useDeferredValue`, который откладывает уже готовое значение, `useTransition` применяется в момент вызова setter состояния  

Render, вызванный Transition-update, может быть прерван более срочным обновлением и запущен заново.   
> В React 19 `startTransition` может принимать как синхронную, так и асинхронную функцию.  
>> Однако state updates после `await` сейчас нужно дополнительно оборачивать в `startTransition`, чтобы они тоже были помечены как Transition.  
```tsx
const MyComponent = () => {
	const [state, setState] = useState(null)
	const [text, setText] = useState('')
	const [isPending, startTransition] = useTransition();

	function onClick() {
		startTransition(() => {
				// вычисления
				const nextState = computeNextState();
				// состояние state помечено как переход
				setState(nextState);
		});
	}
	// срочное обновление интерфейса может прервать render Transition
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

>`startTransition` сразу вызывает переданный callback. Откладывается не выполнение callback, а render state updates, помеченных как Transition.  
Только изменение состояния помечается как переход  

```tsx
console.log(1);
startTransition(() => {
  console.log(2);
  setPage('/about');
});
console.log(3);
// console output: 1 2 3
```

> Transition не следует использовать для state, который должен обновляться немедленно при прямом взаимодействии пользователя, например для значения controlled input/select/radio/checkbox.   
Для тяжёлой зависимой части интерфейса можно использовать `useDeferredValue`.

[Вернуться к содержанию](#содержание)

## 4.13 useSyncExternalStore

[Отличные примеры применения](https://www.youtube.com/watch?v=Y34aQue4DIg)  

`useSyncExternalStore(onStoreChange, getSnapshot, getServerSnapshot?)` - подписка на внешнее хранилище (API браузера, сторонние библиотеки за пределами React).  
При изменении данных во внешнем хранилище происходит ререндер
* onStoreChange(callback) - функция подписки, которая должна возвращать функцию отписки, где callback - вызывается при изменении хранилища
* getSnapshot - функция возвращающая состояние внешнего хранилища (для клиента)
* getServerSnapshot - функция возвращающая состояние внешнего хранилища (для сервера SSR)  
`getServerSnapshot` должен возвращать те же данные, что и при первоначальном рендере на клиенте  

```tsx
const useMatchMedia = (query: string) => {
  const getSnapshot = () => window.matchMedia(query).matches;
  const subscribe = (listener: () => void) => {
    const mediaQueryList = window.matchMedia(query);
    mediaQueryList.addEventListener('change', listener);
    return () => mediaQueryList.removeEventListener('change', listener);
  }
  return useSyncExternalStore(subscribe, getSnapshot);
}

const UseSyncExample = () => {
	const isDesktopScreen = useMatchMedia('(min-width: 1920px)')
	return <p>{isDesktopScreen ? 'Desktop' : 'Mobile'}</p>
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

```tsx
import {useOptimistic} from 'react'
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
	])

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
};
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
* `fn(previousState, formData)` - обычно асинхронная функция, вызываемая при отправке формы или нажатии кнопки  
	* `previousState` - предыдущее состояние формы (изначально `initialState`)
	* `formData` - аргументы формы
* `initialState` - начальное состояние (должно быть сериализуемо)
* `permalink` - уникальный URL страницы, испольуемый формой, для редиректа на другую страницу после отправки формы  
При отправке формы (до загрузки JS бандла) произойдет редирект на `permalink`
* `state` - текущее состояние (во время первого рендера соответствует `initialState`)
* `formAction` - `action` передаваемый в компоненты формы или пропс `formAction` любой кнопки внутри формы, также может быть вызвано вручную внутри `startTransition`
* `isPending` - флаг состояния Transition (перехода)

```tsx
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

```tsx
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

## 4.19 useEffectEvent

`const onEvent = useEffectEvent(callback)` - хук, позволяющий вынести из `useEffect` нереактивную логику, которая должна читать актуальные props/state, но не должна вызывать повторный запуск effect, где
* `callback` - логика EffectEvent  

> Не стоит злоупотреблять хуком, как способом убрать "лишние" зависимости из effect-ов. Нужно использовать осмысленно, где это действительно необходимо  

Effect Event (возвращаемое значение хука):  
* можно вызывать только внутри `useEffect`, `useLayoutEffect`, `useInsertionEffect` или других Effect Event  
* нельзя вызывать во время рендера
* не рекомендуется передавать в другие компоненты или кастомные хуки  

[Вернуться к содержанию](#содержание)

## 4.20 Кастомные хуки

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
```tsx
<Suspense fallback={OuterLoader}>
	<OuterComponent>
	<Suspense fallback={InnerLoader}>
		<InnerComponent />
	</Suspense>
	</OuterComponent>
</Suspense>
```
Для того, чтобы уже отображенные данные не заменялись на `fallback` при изменении запроса, нужно использовать `useTransition` или `useDeferredValue`  
При завершении серверного рендеринга (SSR) компонента с ошибкой, на клиент приходит `fallback`, и повторно запускается рендеринг на клиенте   
Ошибки на клиенте обрабатываются в `ErrorBoundary`  
Для исключения компонента из рендера на сервере, в компонент **передается ошибка**:  
```tsx
<Suspense>
	{
		// window отсутствует в серверной среде
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

```tsx
// Объявление на верхнем уровне модуля
const Modal = lazy(() => import("./Modal.tsx"))

const MyComponent = () => {
	const [isModalDisplayed, setModalDisplayed] = useState<boolean>(false);
	return (
		<>
			<button onClick={() => { setModalDisplayed(true) }}>Загрузить модальное окно</button>
			{isModalDisplayed && (
				<Suspense fallback="Loading...">
					<Modal />
				</Suspense>
			)}
		</>
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
 
```tsx
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
	}
	render() {
		if(this.state.error) {
		// Запасной UI
				return <h1>Something wrong</h1>; 
		}
		// Ошибки нет
			return this.props.children
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

```tsx
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
		// общий функционал всех компонентов оборачиваемых в HOC
		const onButtonClick = () => {
			alert(props.alertText)
		}
		// важное замечание: используем спред-синтаксис после всех генерируемых HOC-компонентом пропсов, чтобы не затерлись исходные передаваемые пропсы
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
      	// передаем пропсы базового компонента и пропсы используемые только внутри HOC-компонента 
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
```tsx
type TCard = {
	title: string;
	name: string;
	age: number;
	subscribers: number;
}
type UserCardProps = {
	card: TCard;
}

const UserCard = ({card}: UserCardProps) => {
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
```tsx
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

const UserCard = ({children, card}: UserCardProps) => {
	return (
		<UserCardContext.Provider value={{children, card}}>
			<div>
				{children}
			</div>
		</UserCardContext.Provider>
	);
}

UserCard.Header = function UserCardHeader() {
	const { card } = useUserCardContext();
	return <header><h2>{card.title}</h2></header>
}
UserCard.Main = function UserCardMain() {
	const { card } = useUserCardContext();
	return (
		<div>
			{card.name}
			<span>{card.age}</span>
		</div>
	);
}
UserCard.Footer = function UserCardFooter() {
	const { card } = useUserCardContext();
	return <footer><p>Subscribers: {card.subscribers}</p></footer>
}
```
Использование компонента без футера с измененным порядком элементов 
```tsx
<UserCard
	card={{
		title: 'Title',
		name: 'Petr',
		age: 18,
		subscribers: 1000,
	}}
>
	<UserCard.Main />
	<UserCard.Header />
</UserCard>
```

[Видео-пример компонента](https://www.youtube.com/watch?v=N_WgBU3S9W8)

[Вернуться к содержанию](#содержание)  

## 5.5 Activity

`<Activity mode>{children}</Activity>` - механизм скрытия части UI с сохранением состояния и пониженным приоритетом обновлений, где  
* `mode` - режим показа/скрытия: `visible` (по умолчанию) / `hidden`  

При скрытии к дочернему компоненту применяется `display: none`, срабатывает cleanup эффектов, но компоненты продолжают ререндериться с низким приоритетом при изменении пропсов  
Когда компонент становится видимым он восстанавливает свое предыдущее состояние и заново монтирует эффекты  
Если `Activity` используется внутри `ViewTransition`, то при скрытии срабатывает exit-анимация, при появлении - start-анимация  
Если дочерний компонент содержит только текст, то при скрытии он не рендерится  

В отличии от условного рендера `{isShowingSidebar && <Modal />}` дочерний компонент:  
* не инициализирует заново свой `state`  
* не теряет введенный текст (на примере `textarea`)  
* заранее рендерит скрытые компоненты, без срабатывания onMount эффектов в режиме скрытия

```jsx
<Activity mode={isShow ? "visible" : "hidden"}>
  <>
	<Modal />
	<textarea />
	<SlowComponent />
  </>	
</Activity>
```

Проблемы с `<video>, <audio>, <iframe>`:  
Компоненты не имеют собственного cleanup и будут продолжать проигрываться в фоне при скрытии.  
Для решения проблемы стоит явно добавить cleranup с паузой воспроизведения  

```jsx
export default function Video() {
	const ref = useRef();
	// используем useLayoutEffect, чтобы избежать задержки при использовании Suspense или ViewTransition
	useLayoutEffect(() => {
		const videoRef = ref.current;

		return () => {
			videoRef.pause()
		}
	}, []);

	return <video ref={ref} src="..."/>
}
```

[Вернуться к содержанию](#содержание)

## 5.6 ViewTransition

`<ViewTransition>{children}</ViewTransition>` - компонент React для анимированных переходов (transition) между состояниями интерфейса. По умолчанию анимация crossfade. _(пока еще в Canary)_    
* `name` - опциональный пропс (string/object) для именования transition. По умолчанию React генерирует для каждого transition уникальное имя  
* `enter`, `exit`, `update`, `share`, `default` - transition при появлении, скрытии, изменении, общий transition между связанными элементами. Значения: `auto` (по умолчанию), `none` - отсутствие transition данного типа, `<classname>` - кастомный CSS класс (string/object)  
`default='none'` - отключает все, не заданные явно, transition
*  `onEnter(instance, types) => {}`, `onExit(instance, types) => {}`, `onShare(instance, types) => {}`, `onUpdate(instance, types) => {}` - event срабатывающий при вызове соответствующего transition, где  
	* `types` - [массив типов анимаций](https://www.w3.org/TR/css-view-transitions-2/#active-view-transition-pseudo-examples)
    * `instance` - объект для доступа к псевдоэлементам transition: `old, new, name, group, imagePair`

```jsx
<ViewTransition
	name="modal"
	enter="fade-in"
	exit="fade-out"
	update="smooth"
	share="morph"
>
	{isOpen ? <Modal /> : null}
</ViewTransition>
```

Позволяет:
* анимировать появление и скрытие элементов
* анимировать изменение layout
* анимировать переход между fallback и готовым контентом (`Suspense` и `Activity`)
* использовать переходы между связанными элементами через name
* применять разные анимации для разных типов переходов  

>Срабатывает только при размещении ДО других DOM элементов
```jsx
//Верно
<ViewTransition>
	<div/>
</ViewTransition>
//Неверно
<div>
	<ViewTransition>
		<div/>
	</ViewTransition>
</div>
```

[Подробнее в документации](https://react.dev/reference/react/ViewTransition)  

[Вернуться к содержанию](#содержание)

## 5.7 Container Presenter component

**Container / Presenter component** - паттерн разделения логики и отображения.  
`Container` отвечает за получение данных, состояние и обработчики, а `Presenter` только за отображение через props.  

Паттерн полезен, когда нужно:  
* отделить UI от data fetching и business logic  
* переиспользовать чистый UI-компонент отдельно от логики    

**Задача:** Нужно отобразить список пользователей. Логику получения данных и состояние загрузки не хочется смешивать с JSX-разметкой списка.  

```tsx
type TUser = {
	id: number;
	name: string;
	email: string;
}

type UserListProps = {
	users: TUser[];
	isLoading: boolean;
}

// Presenter component
const UserList = ({ users, isLoading }: UserListProps) => {
	if (isLoading) return <p>Loading...</p>;
	if (!users.length) return <p>No users</p>;

	return (
		<ul>
			{users.map((user) => (
				<li key={user.id}>
					<strong>{user.name}</strong> - {user.email}
				</li>
			))}
		</ul>
	);
};

// Container component
const UserListContainer = () => {
	const [users, setUsers] = useState<TUser[]>([]);
	const [isLoading, setIsLoading] = useState(true);

	useEffect(() => {
		async function loadUsers() {
			const response = await fetch('/api/users');
			const data = await response.json();
			setUsers(data);
			setIsLoading(false);
		}

		loadUsers();
	}, []);

	return <UserList users={users} isLoading={isLoading} />;
};
```

[Вернуться к содержанию](#содержание)

## 5.8 Render Props

**Render Props** - паттерн, при котором компонент принимает функцию и через нее передает наружу данные или поведение.  
Это позволяет переиспользовать логику, не привязываясь к конкретной разметке.

**Задача:** Нужно переиспользовать логику отслеживания позиции курсора, но отображать результат в разных компонентах по-разному.  

```tsx
type MousePosition = {
	x: number;
	y: number;
}

type MouseTrackerProps = {
	render: (position: MousePosition) => React.ReactNode;
}

const MouseTracker = ({ render }: MouseTrackerProps) => {
	const [position, setPosition] = useState<MousePosition>({ x: 0, y: 0 });

	useEffect(() => {
		function handleMouseMove(event: MouseEvent) {
			setPosition({
				x: event.clientX,
				y: event.clientY,
			});
		}

		window.addEventListener('mousemove', handleMouseMove);
		return () => window.removeEventListener('mousemove', handleMouseMove);
	}, []);

	return <>{render(position)}</>;
};

<MouseTracker render={({ x, y }) => (
	<p>Mouse position: {x}, {y}</p>
  )}
/>
```

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

```tsx
const MyComponent = () => {
	const [mode, setMode] = useState('dark');
	const onValueChange = (e: ChangeEvent<HTMLInputElement>) => {
		setMode(e.target.value);
		// для чекбоксов  - setState(e.target.checked)
	}
	return <input type="radio" value="light" checked={mode === "light"} onChange={onValueChange} />
}
```

Можно использовать одно состояние для нескольких полей ввода.  
Для доступа к состоянию конкретного элемента нужно произвести связывание имен в состоянии и в атрибуте элемента
```tsx
const MyComponent = () => {
	const [state, setState] = useState({checkbox: false, text: ''})
	const handler(event: ChangeEvent<HTMLInputElement>) => {
		const target = event.target;
		const value = target.type === "checkbox" ? target.checked : target.value
		const name = target.name
		setState((prevState) => {
			return {
				...prevState,
				[name]: value
			}
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
```tsx
const MyComponent = () => {
	const [state, setState] = useState({checkbox: false, text: ''})
	const handler = 
			(name: string) => (event: ChangeEvent<HTMLInputElement>) => {
			const target = event.target;
			const value = target.type === "checkbox" ? target.checked : target.value
			setState((prevState) => {
				return {
					...prevState,
					[name]: value
				}
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

В Create React App переменные окружения для клиента должны начинаться с `REACT_APP`: `REACT_APP_API_KEY`  
Использование в файлах: `process.env.REACT_APP_API_KEY`

Основные методы: 
1. в настройках системы Windows (Изменение системных переменных сред) или другой ОС
2. в кроссплатформенной среде cross-env (необходима установка `npm install -D cross-env `) `cross-env REACT_APP_CROSS_ENV=value`
3. в .env файлах (необходима установка `npm install -D dotenv-webpack`): `REACT_APP_API_KEY=8sdf3218652sdfq84531203asasd`  

Подключение в webpack: 
```tsx
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

Директива `use server` служит для обозначения серверных функций и серверных действий. Все компоненты по умолчанию являются серверными.  

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
```tsx
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
```tsx
// серверное действие
'use server'

export const create = async (data) => {
	await db.posts.create(data);
}

// использование в клиентском компоненте
'use client';
import {create} from './actions'
const Index = () => <button onClick={() => create(newPost)}>Create</button>
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
```tsx
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

```tsx
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
```tsx
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

```tsx
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

```tsx
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
```tsx
function TimerUI() {
  const time = useTimer();
  return <div>{time}</div>;
} 
```

[Вернуться к содержанию](#содержание)  

## 10.3 Класс без собственных состояний

```tsx
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

```tsx
// [1] - создаем контекст
export const ApiContext = createContext<ApiInterface>({
    getPosts: () => Promise.resolve([]),
}); 
```

Использование в компонентах:
```tsx
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
