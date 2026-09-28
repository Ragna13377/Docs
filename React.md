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
    - [5.7 Container / Presentational components](#57-container--presentational-components)
    - [5.8 Render Props](#58-render-props)
* [6. ReactDOM. Элементы и события React](#6-reactdom-элементы-и-события-react) 
* [7. Portals](#7-portals) 
* [8. Переменные окружения в React](#8-переменные-окружения-в-react) 
* [9. Server React: SSR, RSC и Server Functions](#9-server-react-ssr-rsc-и-server-functions)
	- [9.1 SSR и Hydration](#91-ssr-и-hydration)
	- [9.2 React Server Components](#92-react-server-components)
		- [9.2.1 use client](#921-use-client)
		- [9.2.2 use server](#922-use-server)
	- [9.3 Server Functions / Server Actions](#93-server-functions--server-actions)
	- [9.4 cache](#94-cache)
* [10. Интеграция классов с React (редкий кейс)](#10-интеграция-классов-с-react-редкий-кейс) 
	- [10.1 Класс с собственным состоянием](#101-класс-с-собственным-состоянием)
	- [10.2 Класс-сервис без состояния, влияющего на render](#102-класс-сервис-без-состояния-влияющего-на-render)
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

`useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)` - подписка на внешнее хранилище (API браузера, сторонние библиотеки за пределами React).  
При изменении хранилища subscribe должен вызвать переданный callback.  
React повторно вызывает getSnapshot и сравнивает новый и предыдущий snapshot по Object.is. Если snapshot изменился - делает rerender    
Пока store не изменился, getSnapshot должен возвращать то же значение. Для mutable store нужно возвращать cached immutable snapshot, а не создавать новый объект при каждом вызове.  
* `subscribe(callback)` - функция подписки, которая должна возвращать функцию отписки, где callback - вызывается при изменении хранилища
* `getSnapshot` - функция возвращающая состояние внешнего хранилища (для клиента)
* `getServerSnapshot` - функция возвращающая состояние внешнего хранилища (для сервера SSR)  
`getServerSnapshot` должен возвращать те же данные, что и при первоначальном рендере на клиенте  

```tsx
const useMatchMedia = (query: string) => {
	const subscribe = useCallback((listener: () => void) => {
		const mediaQueryList = window.matchMedia(query);

		mediaQueryList.addEventListener('change', listener);

		return () => mediaQueryList.removeEventListener('change', listener);
	}, [query]);

	const getSnapshot = useCallback(
			() => window.matchMedia(query).matches,
			[query],
	);
	return useSyncExternalStore(subscribe, getSnapshot, () => false);
}

const UseSyncExample = () => {
	const isDesktopScreen = useMatchMedia('(min-width: 1920px)')
	return <p>{isDesktopScreen ? 'Desktop' : 'Mobile'}</p>
}
```

`useSyncExternalStore` нужен для согласованной подписки React на внешние mutable sources. React читает store через getSnapshot и проверяет snapshot при изменениях, предотвращая отображение несогласованных версий внешнего состояния. 

[Вернуться к содержанию](#содержание)

## 4.14 useId

[Разбор темы(видео)](https://www.youtube.com/watch?v=_vwCKV7f_eA)  
[Более подробный разбор(видео)](https://www.youtube.com/watch?v=GNVI9Pr_RKQ&t=777s)  

`useId()` - генерирует уникальные идентификаторы **(не используется для ключей списков)**  

Сгенерированный id может содержать символы, требующие экранирования в CSS-селекторе (например, `:`).
Поэтому `querySelector(`#${id}`)` может не сработать без `CSS.escape(id)`.  
Для поиска по id можно использовать `getElementById(id)`. На конкретный формат id от `useId` полагаться нельзя.  

Особенности применения:  
* гарантирует одинаковые id при генерации на сервере и клиенте (в отличие от сторонних библиотек (uuid)), если рендер на сервере и клиенте не имеет отличий   
* подходит для создания уникальных id для связывания форм, label, aria- атрибутов (особенно при заранее неизвестном количестве форм)  
* уникальность id сохраняется в пределах одного React App  

[Вернуться к содержанию](#содержание)

## 4.15 useOptimistic

[Пример с обработкой ошибки(видео)](https://www.youtube.com/watch?v=PPOw-sDeoNw)  
[Пример без обработки ошибки(видео)](https://www.youtube.com/watch?v=M3mGY0pgFk0)  

`const [optimisticState, setOptimistic] = useOptimistic(value, reducer?)` - позволяет временно показать оптимистичное состояние интерфейса во время Action  
* `optimisticState` - текущее оптимистическое состояние (равно `value`, если никакой Action не выполняется)
* `setOptimistic(optimisticValue)` - временно обновляет оптимистическое состояние на время Action
  * если `reducer` не передан, `optimisticValue` становится новым оптимистическим состоянием
  * если `reducer` передан, `optimisticValue` передаётся ему вторым аргументом
* `value` - базовое значение, которое отображается, когда никакой Action не выполняется
* `reducer(currentState, optimisticValue)` - чистая функция:
	* `currentState` - текущее оптимистическое состояние
	* `optimisticValue` - значение, переданное в `setOptimistic`
	* возвращаемое значение становится следующим оптимистическим состоянием

```tsx
function MyComponent({ messages, sendMessage }: TMyComponentProps) {
	const formRef = useRef();

	const [optimisticMessages, addOptimisticMessage] = useOptimistic(
		messages,
		(currentMessages, newMessage) => [
			...currentMessages,
			{
				text: newMessage,
				sending: true,
			},
		],
	);

	async function formAction(formData) {
		addOptimisticMessage(formData.get('message'));
		formRef.current.reset();

		try {
			await sendMessage(formData);
		} catch (error) {
			// здесь можно показать ошибку пользователю
			// optimistic state откатится к messages автоматически
		}
	}

	return (
		<>
			{optimisticMessages.map((message, index) => (
				<div key={index}>
					{message.text}
					{!!message.sending && <small> Sending... </small>}
				</div>
			))}
			<form action={formAction} ref={formRef}>
				<input type="text" name="message" />
				<button type="submit">Send</button>
			</form>
		</>
	);
}
```

[Вернуться к содержанию](#содержание)

## 4.16 useDebugValue

[хороший разбор(видео)](https://www.youtube.com/watch?v=pTF86K8JZBQ)  

`useDebugValue(value, format?)` - добавляет метку к пользовательскому хуку в DevTools (т.к. все хуки не имеют маркировки и расположены последовательно в React DevTools)  
* value - значение отображаемое в devtools (любой тип: массив/объект/строка и т.д.)
* format - функция форматирования с аргументом value
`useDebugValue(date, (date) => date.toDateString())`
`useDebugValue` вызывается на верхнем уровне custom Hook и добавляет его отладочное значение в React DevTools.  

Не стоит передавать тяжёлые вычисления в `value`, т.к. они будут выполняться при каждом render.   
Для дорогого форматирования исходное значение передаётся в `value`, а вычисление — в `format`.  
React DevTools вызывает `format` только при инспектировании компонента.

[Вернуться к содержанию](#содержание)

## 4.17 useActionState

[Самая актуальная инфа в доке](https://react.dev/reference/react/useActionState)  

`const [state, dispatchAction, isPending] = useActionState(reducerAction, initialState, permalink?)` - хук, который позволяет обновлять состояние на основе результата действия формы, где  
* `reducerAction(previousState, actionPayload)` -  синхронная или асинхронная функция, вызываемая при запуске Action. Может выполнять side effects и должна вернуть новое состояние    
	* `previousState` - предыдущее состояние (изначально `initialState`, затем результат предыдущего вызова `reducerAction`)
	* `actionPayload` - значение, переданное в `dispatchAction`; при использовании `dispatchAction` как action формы это будет FormData
* `initialState` - начальное состояние. При использовании Server Function должно быть сериализуемым
* `permalink` - необязательный URL для progressive enhancement при Server Functions.  
Если форма отправлена до загрузки JavaScript, браузер перейдёт на этот URL вместо текущей страницы
* `state` - текущее состояние (при первом render соответствует `initialState`, затем результату `reducerAction`)
* `formAction` - функция запуска Action. Может передаваться в action / formAction; при ручном вызове должна вызываться внутри startTransition
* `isPending` - true, пока выполняется одна или несколько Actions этого `useActionState`

```tsx
import { action  } from "./actions.js";
const Index = () => {
	const [state, dispatchAction, isPending] = useActionState(action, null)
  return (
    <form action={dispatchAction}>
		<button type="submit">Submit</button>
		{isPending ? "Loading..." : state}
	</form>
  );
}
```

## 4.18 useFormStatus

`const { pending, data, method, action } = useFormStatus()` - хук `react-dom`, предоставляет информацию о статусе последней отправки родительской формы, где  
* `pending` - boolean. True - форма в процессе отправки
* `data` - отправляемый `FormData` или `null`, если форма сейчас не отправляется
* `method` - HTTP-метод родительской формы (`'get' | 'post'`)
* `action` - функция, переданная в `action` родительской формы, или `null`

> Хук должен вызываться в компоненте, находящемся ВНУТРИ родительской формы.

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
* `callback` - логика Effect Event, которая при вызове читает актуальные committed props/state  

> Не стоит злоупотреблять хуком, как способом убрать "лишние" зависимости из effect-ов. Нужно использовать осмысленно, где это действительно необходимо  

Effect Event (возвращаемое значение хука):  
* можно вызывать только внутри `useEffect`, `useLayoutEffect`, `useInsertionEffect` или других Effect Event  
* нельзя вызывать во время рендера
* нельзя передавать в другие компоненты или кастомные хуки — Effect Event должен использоваться локально рядом с Effect, которому он принадлежит

[Вернуться к содержанию](#содержание)

## 4.20 Кастомные хуки

Кастомные хуки - функции, начинающиеся с `use` и использующие внутри встроенные или другие кастомные хуки для переиспользования stateful-логики     
Кастомные, также как и встроенные хуки не должны использоваться внутри условных конструкций, циклов и других функциях  
Кастомный хук может возвращать значения любого типа; формат возвращаемого значения определяется его API   
_Если кастомный хук создаёт state через `useState` / `useReducer`, каждый его вызов получает собственный независимый state._

Стоит обратить внимание:
* useMediaQuery — подписка на matchMedia.
* useLocalStorage / useSessionStorage — синхронизация state с Web Storage.
* useEventListener / useWindowEvent — подписка на DOM/window events с cleanup.
* `useLatest` - хранит последнее значение в `ref`, позволяя читать актуальное значение из стабильной подписки или callback без добавления этого значения в зависимости и без пересоздания подписки  

[Библиотека полезных хуков](https://usehooks-ts.com/introduction)  

[Неплохие кастомные хуки обертки над useState](https://www.youtube.com/watch?v=3SB278SY73s)  
[Кастомные хуки для оптимизации](https://www.youtube.com/watch?v=XOSgHVzHEV4)  
[Еще подробнее про useLatest](https://www.youtube.com/watch?v=ILg1zhl92AI)

[Вернуться к содержанию](#содержание)  

# 5. Пользовательские компоненты

## 5.1 Suspense  

[Объяснение преимуществ Suspense](https://www.youtube.com/watch?v=pj5N-Khihgc)  

`<Suspense fallback={<Loader />}><MyComponent /></Suspense>` - отображает `fallback`, если дочернее дерево suspend'ится во время render, и заменяет его основным UI, когда данные/код становятся доступны  
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

При streaming SSR, если дочерний компонент внутри `Suspense` выбросил ошибку на сервере, React отправляет `fallback` ближайшего `Suspense` и повторяет попытку render этого компонента на клиенте.    
Если render на клиенте завершится успешно, ошибка пользователю не показывается. Если компонент снова выбросит ошибку, её обработает ближайший `ErrorBoundary`.   
Ошибки на клиенте обрабатываются в `ErrorBoundary`  

>В React 19.3 компонент можно явно исключить из серверного render через `use(browser())`. На сервере React оставит `fallback` ближайшего `Suspense`, а на клиенте компонент отрендерится нормально.
```tsx
import { Suspense, use } from 'react';
import { browser } from 'react-dom';

function BrowserOnlyComponent() {
	use(browser('Component requires browser APIs'));

	return <div>Client content</div>;
}

function App() {
	return (
		<Suspense fallback={<Loader />}>
			<BrowserOnlyComponent />
		</Suspense>
	);
}
```


>`Suspense` поддерживает источники данных: 
>	1. Загрузка данных с помощью фреймворков, поддерживающих `Suspense` (Next)
>	2. `React.lazy`
>	3. `use` для чтения значения из Promise  

>`Suspense` **НЕ** поддерживает источники данных: 
>	1. Загрузка данных в useEffect
> 2. Загрузка данных в обработчиках событий  

[Вернуться к содержанию](#содержание)  

### 5.1.1 Lazy Loading

`lazy(load)` - позволяет отложить загрузку кода компонента до его первого render. `lazy` возвращает React-компонент, где  
* `load` - функция возвращающая Promise/thenable, разрешающийся в объект с компонентом в `.default`  

React кэширует Promise, возвращённый `load()`, и его resolved value.  
При успешного resolve React рендерит компонент из `.default`. 
Если Promise отклоняется, причина ошибки передаётся ближайшему `ErrorBoundary`.  

Lazy-компонент нужно объявлять на уровне модуля ВНЕ других компонентов.  
Если вызывать `lazy()` внутри компонента, при каждом его render будет создаваться новый тип компонента, из-за чего состояние lazy-компонента может сбрасываться.

> По умолчанию `lazy(() => import(...))` ожидает компонент в `default export`.

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

> Error Boundary не перехватывает ошибки:
> * в обработчиках событий;
> * в обычном асинхронном коде (`setTimeout`, `requestAnimationFrame` и т.д.);
> * во время SSR;
> * в самом Error Boundary.
>
> Исключение: ошибки, выброшенные внутри callback `startTransition`, могут быть обработаны Error Boundary.  
> Ошибки в event handlers и обычной async-логике обрабатываются локально через `try...catch` / `.catch()`.
 
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

переиспользования логики компонентов. HOC является функцией, которая принимает компонент и возвращает новый компонент с дополнительным поведением или props.    
HOC принято называть с префиксом `with`: `withClick`, `withAuth`, `withAnalytics`. 

> В современном функциональном React для переиспользования логики чаще используются custom hooks, но HOC всё ещё встречаются в существующих библиотеках и legacy-коде.

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
const withClick = (
		WrappedComponent: ComponentType<PersonalProps & WrapperGeneratedProps>
): ComponentType<PersonalProps & WrapperProps> => {
	return function WithClick({
		alertText,
		...props
	}: PersonalProps & WrapperProps): ReactElement {
		const onButtonClick = () => {
			alert(alertText);
		};

		return (
			<WrappedComponent
				{...props}
				onButtonClick={onButtonClick}
				buttonText="Button Text"
			/>
		);
	};
};

// Button.tsx
const Button = ({isEnabled, onButtonClick, buttonText}: PersonalProps & WrapperGeneratedProps) => {
	return <button disabled={isEnabled} onClick={onButtonClick}>
		{buttonText}
	</button>
}

// App.tsx
const WithClickButton = withClick(Button)

const App = () => {
	//  передаем оборачиваемый компонент в HOC
	const [isEnabled, setIsEnabled] = useState(false);

	return (
      	// передаем пропсы базового компонента и пропсы используемые только внутри HOC-компонента 
		<WithClickButton isEnabled={isEnabled} alertText="Some Text"/>
	)
}
```

[Видео без типизации](https://www.youtube.com/watch?v=KWT8OKzrMZ4)  
[Подробнее про HOC](https://www.patterns.dev/react/hoc-pattern/)

[Вернуться к содержанию](#содержание)  

## 5.4 Compound component

**Compound component** - паттерн, при котором несколько связанных компонентов образуют единый API и могут совместно использовать состояние или данные, сохраняя гибкую композицию через `children`.   
Например:  
```tsx
<UserCard>
	<UserCard.Header />
	<UserCard.Main />
	<UserCard.Footer />
</UserCard>
```

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
	if(context === null) throw new Error('UserCardContext access error')
	return context
}

const UserCard = ({children, card}: UserCardProps) => {
	return (
		<UserCardContext value={card}>
			<div>{children}</div>
		</UserCardContext>
	);
}

UserCard.Header = function UserCardHeader() {
	const card = useUserCardContext();
	return <header><h2>{card.title}</h2></header>;
}
UserCard.Main = function UserCardMain() {
	const card = useUserCardContext();
	return (
		<div>
			{card.name}
			<span>{card.age}</span>
		</div>
	);
}
UserCard.Footer = function UserCardFooter() {
	const card = useUserCardContext();
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

`<Activity mode="visible | hidden">{children}</Activity>` - позволяет скрывать UI с сохранением его DOM и внутреннего состояния  
* `visible` - компонент отображается и работает как обычно
* `hidden` - DOM скрывается через `display: none`, Effects очищаются, state сохраняется, а обновления скрытого дерева выполняются с пониженным приоритетом

При скрытии к дочернему компоненту применяется `display: none`, срабатывает cleanup эффектов, но компоненты продолжают ререндериться с низким приоритетом при изменении пропсов  
Когда компонент становится видимым он восстанавливает свое предыдущее состояние и заново монтирует эффекты  
Если `Activity` находится внутри `ViewTransition` и переключение `visible` / `hidden` происходит в Transition, при появлении активируется `enter` анимация, а при скрытии - `exit` анимация.  
Если дочерний компонент содержит только текст, то при скрытии он не рендерится  

В отличии от условного рендера `{isShowingSidebar && <Modal />}` дочерний компонент:  
* не инициализирует заново свой `state`  
* не теряет введенный текст (на примере `textarea`)  
* может заранее рендерить скрытые компоненты с низким приоритетом без запуска их Effects  

> При pre-render через `Activity` заранее загружаются только данные из Suspense-совместимых источников (например, Promise через `use`). Fetch внутри `useEffect` заранее не запустится.

```tsx
<Activity mode={isShow ? "visible" : "hidden"}>
  <>
	<Modal />
	<textarea />
	<SlowComponent />
  </>	
</Activity>
```

Проблемы с `<video>, <audio>, <iframe>`:  
При `hidden` DOM не удаляется, поэтому побочные эффекты самого DOM-элемента могут продолжаться. Например, `<video>` может продолжать воспроизводиться после скрытия.   
Для таких элементов нужно явно останавливать side effect в cleanup Effect.  

```tsx
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

`<ViewTransition>{children}</ViewTransition>` - компонент React для анимации изменений интерфейса во время Transition. Использует браузерный View Transition API. По умолчанию применяется crossfade.    
* `name` -  опциональное имя для shared element transition. Для обычных анимаций задавать его не нужно: React автоматически генерирует уникальное имя  
* `enter`, `exit`, `update`, `share`, `default` - настройки анимации для соответствующего типа transition
  * `enter` - ViewTransition появился
  * `exit` - ViewTransition удалился
  * `update` - изменился DOM, размер или позиция существующего ViewTransition
  * `share` - один named ViewTransition исчез, а другой с тем же `name` появился в рамках того же Transition
  * `default` - значение для типов transition, которым не задано отдельное поведение
  Каждый из этих props может принимать:
    * `auto` - стандартная browser-анимация
    * `none` - отключает анимацию
    * `<className>` - CSS-класс View Transition
    * объект `{ [transitionType]: value, default: value }` - позволяет выбрать одно из значений выше в зависимости от Transition Type
* `onEnter(instance, types)`, `onExit(instance, types)`, `onShare(instance, types)`, `onUpdate(instance, types)` - callbacks для программного управления соответствующей анимацией через Web Animations API, где
	* `instance` - объект для доступа к псевдоэлементам transition: `old`, `new`, `name`, `group`, `imagePair`
	* `types` - массив активных Transition Types, добавленных через `addTransitionType`

> Event callback должен возвращать cleanup-функцию для остановки/очистки созданной анимации после завершения или прерывания View Transition.

> Обычный `setState` не активирует `ViewTransition`. Анимация запускается для обновлений внутри Transition (`startTransition`), а также при reveal `Suspense` и обновлениях `useDeferredValue`.

```tsx
const [isOpen, setIsOpen] = useState(false);

const toggleModal = () => {
	startTransition(() => {
		setIsOpen(value => !value);
	});
};

return (
	<>
		<button onClick={toggleModal}>Toggle</button>
		{isOpen && (
			<ViewTransition
				enter="fade-in"
				exit="fade-out"
				default="none"
			>
				<Modal />
			</ViewTransition>
		)}
	</>
);
```

Позволяет:
* анимировать появление и скрытие элементов
* анимировать изменение layout
* анимировать reveal между `Suspense fallback` и готовым контентом
* вместе с `Activity` анимировать `enter` / `exit`, сохраняя state скрываемого компонента
* использовать переходы между связанными элементами через name
* применять разные анимации для разных типов переходов  

> Для `enter` / `exit` `<ViewTransition>` должен быть первым компонентом в добавляемом или удаляемом subtree. Если над ним находится DOM-элемент, `enter` / `exit` для этого ViewTransition не активируются.  

```tsx
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

> Для анимаций нужно учитывать `prefers-reduced-motion`: React автоматически не отключает View Transition для пользователей, предпочитающих уменьшенную анимацию.

[Подробнее в документации](https://react.dev/reference/react/ViewTransition)  

[Вернуться к содержанию](#содержание)

## 5.7 Container / Presentational components

**Container / Presentational** - паттерн разделения логики и отображения.  
`Container` отвечает за получение данных и application/business logic и передаёт данные и callbacks через props.  
`Presentational`-компонент в основном отвечает за отображение UI и может иметь собственное локальное UI-состояние.  

> В современном React отдельный Container-компонент часто заменяется custom hook: логику можно вынести в хук, а UI оставить в компоненте. Сам принцип разделения logic / UI при этом сохраняется.

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
}

// Presentational component
const UserList = ({ users }: UserListProps) => {
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
const UserListContainer = async () => {
	const response = await fetch('https://example.com/api/users');
	const users: TUser[] = await response.json();

	return <UserList users={users} />;
};
```

[Вернуться к содержанию](#содержание)

## 5.8 Render Props

**Render Props** - паттерн, при котором компонент получает функцию через props, вызывает её во время render и передаёт ей свои данные или поведение.  
Возвращаемый этой функцией React-узел определяет, что будет отображено. Это позволяет переиспользовать логику, не привязываясь к конкретной разметке.

> В современном React для переиспользования stateful-логики чаще используются custom hooks. Render Props всё ещё полезен, когда компоненту нужно предоставить consumer'у контроль именно над разметкой.

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

> Render-функция может передаваться не только через prop `render`, но и через `children` (function as children).

[Вернуться к содержанию](#содержание)

# 6. ReactDOM. Элементы и события React

`SyntheticEvent` - обертка над стандартным событием `Event`, которая обеспечивает единообразное поведение в разных браузерах.  
`SyntheticEvent.nativeEvent` - исходный browser `Event`. React event не всегда 1:1 соответствует native event, конкретный mapping не является частью публичного API  
Обычные React event handlers (`onClick`, `onChange` и т.д.) вызываются на target и при распространении события вверх по React-дереву.    
Для обработки события в capture phase используется суффикс `Capture`: `onClickCapture`.  

> Большинство событий в React распространяются вверх по дереву, но есть исключения, например `onScroll` не всплывает.

----

Общие props, поддерживаемые встроенными DOM-компонентами:
* `children` - React узел, который может быть элементом, строкой, числом, порталом или пустым узлом (null, undefined), массивом React узлов.
* `dangerouslySetInnerHTML` - замена `innerHTML` для вставки сырого HTML, уязвимого к XSS  
`dangerouslySetInnerHTML: {__html: `<span>Some HTML</span>`}`
* `ref` - объект из `useRef`, `createRef`, callback-ref для связи ссылки с DOM-элементом
* `suppressContentEditableWarning` -скрывает warning React для элемента, который одновременно имеет `contentEditable={true}` и управляемые React `children`.  
Используется, если содержимое `contentEditable` управляется вручную, например внутри text editor
* `suppressHydrationWarning` - подавляет warning о различиях между server/client HTML на этом элементе.  
Используется как escape hatch для заведомо неизбежных различий (например, timestamp), работает только на один уровень в глубину. Не следует использовать для скрытия обычных hydration bugs     
* `style` - объект `CSSProperties` для применения стилей к элементу  

[Список всех пропсов в официальной документации](https://react.dev/reference/react-dom/components/common#reference)  

Form controls (`input`, `select`, `textarea`) могут быть controlled или uncontrolled.  
* Controlled - текущее значение задаётся React через `value` / `checked` и синхронно обновляется через `onChange`  
* Uncontrolled - текущее значение хранится самим DOM; через `defaultValue` / `defaultChecked` можно задать только начальное значение. При необходимости значение можно прочитать через ref или FormData  
`<input>` считается управляемым, если задан проп `value`.  
`<input type="radio">`, `<input type="checkbox">` - считаются управляемыми, если задан пропс `checked`  
  Управляемому input нужен `onChange`, синхронно обновляющий backing state, либо `readOnly`, если значение намеренно нельзя изменять.    
>`<input type="file">` - всегда неуправляемый компонент.  
  
В элементе `<option>` отсутствует пропс `selected`. Значение передается в `defaultValue` родительского элемента `<select>` (неуправляемый список) или в `value` (управляемый список)

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

ontrolled input не должен переключаться между controlled и uncontrolled.  
Для текстового input `value` должен оставаться строкой на всём времени жизни компонента, поэтому данные `null / undefined` обычно нормализуют: `value={data ?? ''}`.    
Для `checked` значение должно оставаться boolean.  
При получении значения от пользователя режим изменится на управляемый  
Чтобы избежать ошибки с изменением режима, нужно контролировать исходное значение: `{state.requestData ?? ""}`  

[Вернуться к содержанию](#содержание)  

# 7. Portals

[Порталы на практике](https://www.youtube.com/watch?v=V4sHZzX4zh0)  

`createPortal(children, domNode, key?)` - позволяет отрендерить `children` в другом месте DOM, сохраняя их положение в исходном React-дереве    
Портал меняет только физическое расположение узла DOM. JSX, который помещается в портал, действует как обычный дочерний узел (имеет доступ к состояниям родителя и т.д.)  
* `children` - React узел
* `domNode` - существующий DOM-узел, в который рендерится `children`. Если во время update передать другой `domNode`, содержимое portal будет пересоздано
* `key` - опциональный ключ (см. [DOM Diffing](#react-под-капотом))

События от порталов распространяются в соответствии с деревом React, а не DOM  
Использование: создания модальных окон, тултипов   

[Вернуться к содержанию](#содержание)  

# 8. Переменные окружения в React

[Общая информация про переменные окружения](https://www.youtube.com/watch?v=HiRxC7WeNZU)  
[Подробно про создание переменных окружения](https://www.youtube.com/watch?v=wkfWaI_lI48) 
[Текстом про переменные окружения](https://danshin.ms/React-Environment-Variables/) 

React сам по себе не предоставляет API для переменных окружения — способ их загрузки и использования определяется framework / build tool.

>Переменные, доступные клиентскому JavaScript, попадают в browser bundle и не должны содержать секретные данные.

## Vite

Vite автоматически загружает .env-файлы и предоставляет переменные через import.meta.env.

Переменные, доступные клиентскому коду, по умолчанию должны иметь префикс VITE_:

```tsx
VITE_API_URL=https://example.com
DB_PASSWORD=secret

const apiUrl = import.meta.env.VITE_API_URL; // доступно
const password = import.meta.env.DB_PASSWORD; // undefined
```

Пользовательские env-переменные в import.meta.env приходят как строки, поэтому значения других типов необходимо преобразовывать самостоятельно.

Встроенные значения:

```tsx
import.meta.env.MODE      // текущий mode
import.meta.env.DEV       // development
import.meta.env.PROD      // production
import.meta.env.SSR       // выполняется в SSR
import.meta.env.BASE_URL  // base URL приложения
```

Для разных окружений можно использовать:

```tsx
.env
.env.local
.env.development
.env.production
.env.[mode]
.env.[mode].local
```

Например: `vite build --mode staging` загрузит переменные для staging mode.

## Next.js

Next.js автоматически загружает .env* файлы и предоставляет server-side переменные через process.env:  

```tsx
DATABASE_URL=postgres://...
const databaseUrl = process.env.DATABASE_URL;
```

По умолчанию такие переменные доступны только серверному коду.

Для переменной, которая должна быть доступна клиентскому JavaScript, используется префикс `NEXT_PUBLIC_`:

```tsx
NEXT_PUBLIC_API_URL=/api

const apiUrl = process.env.NEXT_PUBLIC_API_URL;
```

`NEXT_PUBLIC_*` значения встраиваются в client bundle во время next build, поэтому:
* они не должны содержать секретные данные;
* после сборки их значения зафиксированы и не изменяются при изменении environment на уже собранном приложении.

Server-only переменные без `NEXT_PUBLIC_` можно читать на сервере во время выполнения приложения.

>.env-файлы с секретами не должны попадать в Git. Обычно локальные значения хранятся в .env.local.

[Вернуться к содержанию](#содержание)  

# 9. Server React: SSR, RSC и Server Functions

SSR и React Server Components (RSC) — разные механизмы, которые могут использоваться вместе.
* SSR (Server-Side Rendering) — React генерирует начальный HTML на сервере, после чего клиентский React выполняет hydration и делает страницу интерактивной.
* RSC (React Server Components) — часть компонентов выполняется только в серверном окружении и их код не отправляется в browser. Результат их render передаётся клиенту в сериализованном формате React Server Components.
* Client Components в RSC-приложении при этом также могут предварительно рендериться на сервере в HTML и затем гидратироваться на клиенте.

[Отличия между серверными и клиентскими компонентами](https://www.youtube.com/watch?v=ePAPd9qzGyM)  
[То же самое, но покороче](https://www.youtube.com/watch?v=Qdkg_mrniLk)  
[Отдельно про передачу серверных компонентов в клиентские](https://www.youtube.com/watch?v=9YuHTGAAyu0)  
[Еще один вариант объяснения, если не хватило предыдущих](https://www.youtube.com/watch?v=rGPpQdbDbwo)

## 9.1 SSR и Hydration

Для SSR React предоставляет API из react-dom/server.

Основные streaming API:
```tsx
renderToPipeableStream() // Node.js
renderToReadableStream() // Web Streams / Edge
```

renderToString() также существует, но имеет меньше возможностей и не поддерживает современный streaming так же полно.

На клиенте серверный HTML гидратируется через:
```tsx
import { hydrateRoot } from 'react-dom/client';

hydrateRoot(
	document.getElementById('root')!,
	<App />,
);
```

Hydration связывает существующий серверный HTML с React-логикой и event handlers, не создавая DOM заново.

>Начальный render на клиенте должен выдавать тот же результат, что и render на сервере. Hydration mismatch следует считать ошибкой и исправлять, а не скрывать через suppressHydrationWarning.

Frameworks, например Next.js, обычно самостоятельно управляют server rendering, streaming и hydration.

[Вернуться к содержанию](#содержание)

## 9.2 React Server Components

React Server Components выполняются в отдельном серверном окружении и не отправляют свой component code в client bundle.

Server Components могут выполняться:
* во время build для статического контента;
* на сервере во время запроса.

Server Components могут:
* использовать async / await прямо в компоненте;
* получать данные из БД, файловой системы и других server-only источников;
* использовать server-only зависимости;
* работать с секретами, если они не передаются клиентскому коду;
* импортировать и рендерить Client Components.

Server Components не могут использовать интерактивные client API:
* локальный state через useState / useReducer;
* Effects;
* event handlers (onClick, onChange и т.д.);
* browser API (window, document, localStorage и т.д.).

```tsx
async function UserPage({ id }: { id: string }) {
	const user = await db.user.findUnique({
		where: { id },
	});

	return <UserProfile user={user} />;
}
```

RSC не следует путать с SSR:

```text
SSR:
React component -> HTML -> hydration в browser

RSC:
Server Component выполняется только на сервере
-> его component code не нужен browser
-> результат используется для построения React tree
```

Server Components не обязательно выполняются только один раз.  
Framework может повторно выполнить их при новом запросе, навигации, обновлении данных и других server renders.

[Вернуться к содержанию](#содержание)  

### 9.2.1 use client

Директива: `use client` создаёт границу между server и client module graph.  

Она должна находиться в начале файла до imports и другого кода (комментарии допустимы).
```tsx
'use client';

import { useState } from 'react';

export function Counter() {
	const [count, setCount] = useState(0);

	return (
		<button onClick={() => setCount(count + 1)}>
			{count}
		</button>
	);
}
```

Модуль с `use client` и его транзитивные зависимости становятся client code.

Поэтому `use client` не нужно добавлять в каждый Client Component — достаточно определить server/client boundary.

Компонент без собственной директивы `use client` также станет Client Component, если он импортирован внутри client module subtree.

`use client` нужен, когда используются:
* state и большинство client Hooks;
* Effects;
* event handlers;
* browser API;
* client-only библиотеки.

Server Component нельзя напрямую импортировать в client module как server-executed компонент.

Но Server Component может быть заранее создан серверным родителем и передан Client Component через children или другой prop:
```tsx
// Server Component
function Page() {
	return (
		<ClientLayout>
			<ServerContent />
		</ClientLayout>
	);
}
```

Здесь ClientLayout не импортирует и не выполняет ServerContent — он получает уже созданный React-узел.

Значения, пересекающие Server → Client boundary через props, должны быть сериализуемыми.

Поддерживаются, например:
* primitives;
* plain objects / arrays;
* Date;
* Map / Set;
* Promise;
* React elements;
* Server Functions.

Нельзя передавать обычные функции, экземпляры пользовательских классов, объекты с null prototype и незарегистрированные через Symbol.for() symbols.  

>Client Component не означает «рендерится только в browser». Framework может предварительно отрендерить Client Component на сервере в HTML, а затем гидратировать его на клиенте.

[Вернуться к содержанию](#содержание)  

### 9.2.2 use server

`use server` НЕ обозначает Server Component.

Для Server Components отдельной директивы нет.

`use server` помечает async Server Function, которую клиентский код может вызвать на сервере.
```tsx
async function createPost(formData: FormData) {
	'use server';

	// server code
}
```
или в начале файла:
```tsx
'use server';

export async function createPost(formData: FormData) {
	// server code
}

export async function deletePost(id: string) {
	// server code
}
```

При module-level 'use server' экспортируемые функции должны быть async Server Functions.

При вызове Server Function с клиента framework отправляет сетевой запрос на сервер, выполняет функцию и при необходимости возвращает сериализуемый результат.

Аргументы и возвращаемые значения Server Function должны быть сериализуемыми.

>Аргументы Server Function всегда нужно считать недоверенными пользовательскими данными. Authentication, authorization и validation должны выполняться внутри серверной функции.

[Вернуться к содержанию](#содержание)  

## 9.3 Server Functions / Server Actions

**Server Function** — async функция, выполняемая на сервере и доступная для вызова из клиентского кода через `use server`.

React раньше называл все такие функции Server Actions. Сейчас терминология разделена:
* **Server Function** — общее название;
* **Server Action** — Server Function, используемая как Action, например через <form action> или внутри Transition.

Framework создаёт ссылку на Server Function. На клиент не отправляется её серверная реализация.
```tsx
// actions.ts
'use server';

export async function createPost(formData: FormData) {
	const title = formData.get('title');

	if (typeof title !== 'string') {
		throw new Error('Invalid title');
	}

	await db.post.create({
		data: { title },
	});
}
```

Server Function можно использовать напрямую как form Action:
```tsx
import { createPost } from './actions';

export function CreatePostForm() {
	return (
		<form action={createPost}>
			<input name="title" />
			<button type="submit">Create</button>
		</form>
	);
}
```
Server Functions также можно вызывать из Client Components:
```tsx
'use client';

import { startTransition } from 'react';
import { deletePost } from './actions';

function DeleteButton({ id }: { id: string }) {
	return (
		<button
			onClick={() => {
				startTransition(() => {
					deletePost(id);
				});
			}}
		>
			Delete
		</button>
	);
}
```
Server Functions должны вызываться как Actions / внутри Transition.
При передаче функции в <form action> или formAction React запускает её как Action автоматически.

Server Functions в первую очередь предназначены для mutations:
* создание / изменение / удаление данных;
* отправка формы;
* выполнение server-side side effects.

Для обычного получения данных внутри Server Component предпочтительнее выполнить запрос непосредственно при render:
```tsx
async function Posts() {
	const posts = await db.post.findMany();

	return <PostList posts={posts} />;
}
```

Server Functions не устраняют сетевой запрос — они позволяют не создавать вручную отдельный API endpoint и клиентский fetch для каждой mutation.

Интегрируются с:
* `useActionState`
* `useOptimistic`
* `<form action>`
* `formAction`
* `startTransition`

[Вернуться к содержанию](#содержание)  

## 9.4 cache

`cache(fn)` — memoization API для React Server Components.
```tsx
import { cache } from 'react';

const getUser = cache(async (id: string) => {
	return db.user.findUnique({
		where: { id },
	});
});
```

Повторные вызовы memoized-функции с теми же аргументами в рамках одного server request используют закэшированный результат:  
```tsx
const user1 = await getUser('123');
const user2 = await getUser('123');
```

Второй вызов может использовать результат первого.

Особенности:
* cache используется только в Server Components;
* cache(fn) обычно объявляется на module scope вне компонентов;
* каждый вызов cache(fn) создаёт новую memoized-функцию со своим отдельным cache;
* результаты кэшируются по аргументам вызова;
* ошибки функции также кэшируются;
* React очищает cache memoized-функций между server requests.

Поэтому:
```tsx
const getUser1 = cache(getUser);
const getUser2 = cache(getUser);
```

создаёт две независимые memoized-функции — их cache не общий.

`cache` полезен для дедупликации повторных вычислений и data fetching между несколькими Server Components в рамках одного server request.

>`cache` не является постоянным application/data cache между запросами. Framework может предоставлять собственные дополнительные механизмы долгоживущего кэширования.

`cache`, `useMemo` и memo решают разные задачи:
* `cache` — memoization server-side функций между несколькими Server Components в рамках server request;
* `useMemo` — memoization вычисления внутри конкретного component instance между его renders;
* `memo` — возможность пропустить render компонента, если его props не изменились.

[Подробное объяснение](https://www.youtube.com/watch?v=A8JGtz2yF9g)  

[Вернуться к содержанию](#содержание)  

# 10. Интеграция классов с React (редкий кейс)

Если класс хранит изменяемое состояние вне React, которое используется при render, его можно рассматривать как внешний store и подписать React на изменения через useSyncExternalStore.

Если возможно, обычное состояние приложения предпочтительнее хранить непосредственно в React через useState / useReducer.  
Интеграция с внешним store нужна в основном для существующего non-React кода, сторонних библиотек и browser API.

## 10.1 Класс с собственным состоянием

Класс должен предоставлять:
* getSnapshot — получение текущего состояния;
* subscribe — подписку на изменения с функцией отписки;
* методы изменения состояния должны уведомлять подписчиков.
```tsx
interface Todo {
	id: string;
	title: string;
}

type Listener = () => void;

class TodosStore {
	private todos: Todo[];
	private listeners = new Set<Listener>();

	constructor(initialTodos: Todo[] = []) {
		this.todos = initialTodos;
	}

	getSnapshot = () => {
		return this.todos;
	};

	subscribe = (listener: Listener) => {
		this.listeners.add(listener);

		return () => {
			this.listeners.delete(listener);
		};
	};

	add = (todo: Todo) => {
		this.todos = [...this.todos, todo];
		this.emitChange();
	};

	remove = (id: string) => {
		this.todos = this.todos.filter(todo => todo.id !== id);
		this.emitChange();
	};

	private emitChange() {
		this.listeners.forEach(listener => listener());
	}
}
```
Класс можно подключить к React через custom hook:  

```tsx
function useTodos(store: TodosStore) {
	return useSyncExternalStore(
		store.subscribe,
		store.getSnapshot,
	);
}
```

Использование:
```tsx
function TodoList({ store }: { store: TodosStore }) {
	const todos = useTodos(store);

	return (
		<>
			{todos.map(todo => (
				<div key={todo.id}>{todo.title}</div>
			))}

			<button
				onClick={() => {
					store.add({
						id: crypto.randomUUID(),
						title: 'New todo',
					});
				}}
			>
				Add
			</button>
		</>
	);
}
```

`getSnapshot` должен возвращать то же значение, пока состояние store не изменилось.  
Поэтому при изменении массива создаётся новый массив, а не мутируется старый.

Для SSR при необходимости также передаётся `getServerSnapshot`.

[Вернуться к содержанию](#содержание)  

## 10.2 Класс-сервис без состояния, влияющего на render

Если класс предоставляет методы, но его внутреннее состояние не должно само вызывать render React, подписка не нужна.

Если экземпляр нужен только в одном месте, его можно использовать напрямую.  
Если один экземпляр сервиса должен быть доступен глубоко в дереве компонентов, его можно передать через Context.

```tsx
interface ApiInterface {
	getPosts(): Promise<string[]>;
}

class Api implements ApiInterface {
	getPosts() {
		return Promise.resolve(['Первый', 'Второй', 'Третий']);
	}
}
```

Если осмысленного значения Context по умолчанию нет, используется null: `const ApiContext = createContext<ApiInterface | null>(null);`

Provider со стабильным экземпляром класса:  
```tsx
function ApiProvider({ children }: PropsWithChildren) {
	const apiRef = useRef<Api | null>(null);

	if (apiRef.current === null) {
		apiRef.current = new Api();
	}

	return (
			<ApiContext value={apiRef.current}>
				{children}
			</ApiContext>
	);
}
```

Для чтения Context удобно создать custom hook:
```tsx
function useApi() {
	const api = useContext(ApiContext);

	if (api === null) {
		throw new Error('useApi must be used within ApiProvider');
	}

	return api;
}
```

Использование:
```tsx
function Posts() {
	const api = useApi();

	useEffect(() => {
		api.getPosts().then(posts => {
			// обработка результата
		});
	}, [api]);

	return ...
}
```

Context здесь используется для передачи зависимости, а не для синхронизации состояния класса с React.

[Вернуться к содержанию](#содержание)

# Дополнительно 

>**P.S.** Если данная информация полезна, то буду рад обратной связи по найденным ошибка в тексте.  
Дальше плавный переход в Next

[Паттерны в React](https://www.patterns.dev/react)  
[Пример рефакторинга кода в React](https://www.youtube.com/watch?v=KJEjJF2BmLw)  
[Повторение базовых принципов написания кода в React](https://www.youtube.com/watch?v=5r25Y9Vg2P4)  

[Вернуться к содержанию](#содержание)  
