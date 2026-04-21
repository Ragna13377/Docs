## Axios

Установка: `npm install axios`

Для получения типизаци: `const axios = require('axios').default;`  
[Разбор Axios](https://www.youtube.com/watch?v=ltn9QoBCJkU)  

Установка: `npm install axios-mock-adapter --save-dev`  

[Очень коротко про мокирование Axios](https://www.youtube.com/watch?v=upM6p0eIdw8)  
[Документация на ГХ](https://github.com/ctimmerm/axios-mock-adapter)  

>**Офтоп**
MSW для мокирования запросов в Next не работает [пруф 1](https://x.com/kettanaito/status/1749496339556094316), [пруф 2](https://github.com/mswjs/msw/issues/1644)  
Axios-mock-adapter работает только с axios запросами. Мокировать нужно вручную, либо через малоизвестные библиотеки.  
React Query (поддерживает fetch и axios) имеет встроенные обертки над запросами (отмена запроса, статус pending, повторный запрос и т.д.). 
В Next встроено кэширование для fetch запросов. Для axios автоматическое кэширование не поддерживается, но можно использовать экспериментальный `cache` из React.  
_Нужно оценить решение для применении axios в Next проекте_  
[Сравнение React Query и tRPC с Next](https://www.youtube.com/watch?v=51pf_nCJpwg)

## 1. Запросы: 

* `axios.request(config)`
* `axios.get(url[, config]`
* `axios.head(url[, config])`
* `axios.options(url[, config])`
* `axios.delete(url[, config])`
* `axios.post(url[, data[, config]])`
* `axios.put(url[, data[, config]])`
* `axios.patch(url[, data[, config]])`

**Примеры** реализации запроса: 

* Классический способ
```javascript
axios({
	method: 'post',
	url,
	data: {
		name: 'Petr',
		age: 18
	}
})
	.then(() => {})
	.catch(() => {})
	.finally(() => {})
```

* Упрощенный способ
```javascript
axios.post(url, data, config)
	.then((response) => {})
	.catch((error) => {})
	.finally(() => {})
```  

* Использование async/await
```javascript
const getPost = async () => {
	try {
		const res = await axios({})
	} catch (error) {}
}
```

**Типизация запросов**  
При типизации используется generic: `post<T = any, R = AxiosResponse<T>, D=any>(url: string, data: D, config: AxiosRequsetConfig<D>): Promise<R>`  
Результат типизации с конфигом:  
`axios.post<TResponseData, AxiosResponse<TResponseData>, TConfig>`  
Результат типизации без конфига:  
`axios.get<TResponseData>`  

**Установка параметров по умолчанию** для всех запросов:  
```javascript
axios.defaults.baseURL = baseURL;
axios.defaults.withCredentials = true;
axios.defaults.headers.get = {}; // для headers нужно указывать метод запросов (common - все методы)
...
```

**Создание нескольких экземпляров axios**  
```javascript
const apiNameAxios = axios.create({
	baseURL,
	headers
})

apiNameAxios.get(url)
```

Параметры конфигурации запроса:
* `url: 'url'` - адрес сервера для запроса
* `baseURL: 'baseURL'` - базовый URL добавляется перед URL, если URL не является абсолютным
* `method: 'get'` - метод запроса (по умолчанию GET) 
* `transformRequest: [function (data, headers) { return data }]` - преобразования данных запроса перед отправкой. **Применимо только** к PUT, POST, PATCH, DELETE  
	Возвращает String, Buffer, ArrayBuffer, FormData, Stream 
* `transformResponse: [ function (data) { return data }]` - преобразования данных ответа перед их передачей в then, catch
* `headers: {'X-Requested-With': 'XMLHttpRequest'}` - пользовательские заголовки для запроса
* `params: { id: 132 } - URL-параметры, передаваемые вместе с запросом (нулевые или неопределенные не отображаются в URL)
* `paramsSerializer: function (params) { return Qs.stringify(params, {arrayFormat: 'brackets'}) }` - кастомная функция сериализации параметров  
	[QS](https://www.npmjs.com/package/qs), [jquery.param](http://api.jquery.com/jquery.param/)
* `data: { name: 'Petr' }` - данные передаваемые в POST, PUT, PATCH, DELETE запросах
	Если отсутствует параметр transformResponse, то данные должны быть в форматах: String, Object, ArrayBuffer, ArrayBufferView, URLSearchParams, (FormData, Blob, File - для браузера)  
	Альтернативный синтаксис: `data: 'Name=Petr Petrovich&Age=23'`
* `timeout: 1000` - время ожидания запроса (по умолчанию 0)
* `withCredentials: false - указание для включения в запрос coockies, аутентификационных заголовков и SSL-сертификатов (по умолчанию false) 
* `auth: {username: 'Petr', password: '12456'}` - устанавливает заголовок для базовой Authorization  
**Для Bearer токенов** необходимо использовать пользовательский заголовок Authorization
* `responseType: 'json'` - указывает тип данных, которые возвращает сервер (arraybuffer, document, json (по умолчанию), text, stream, blob (для браузера))
* `responseEncoding: 'utf8'` - указывает кодировку ответа сервера (только в Node.js). По умолчанию utf8  
_Игнорируется для `responseType: 'stream'`_  
* `xsrfCookieName: 'XSRF-TOKEN'` - имя xsrf-cookie, которое может установить бэкенд для защиты от кроссайтовых атак (по умолчанию XSRF-TOKEN)
* `xsrfHeaderName: 'X-XSRF-TOKEN'` - заголовка http содержащее xsrf-токен (по умолчанию X-XSRF-TOKEN)
* `onUploadProgress: function (progressEvent) {}` - обрабатывает события прогресса загрузки при отправке данных (только для браузера)  
	* `progressEvent.loaded` - количество переданных байт
	* `progressEvent.total` - общий размер данных
* `onDownloadProgress: function (progressEvent) {}` - аналгоично onUploadProgress, но для отслеживания загруски при получения ответа с данными (только для браузера) 
* `maxContentLength: 2000` - максимальный размер содержимого ответа http в байтах (для Node.js)
* `maxBodyLength: 2000` - максимальный размер содержимого запроса в байтах (для Node.js)
* `maxRedirects: 5` - максимальное количество перенаправлений в node.js (если 0, то перенаправления не будут выполняться, 5 - по умолчанию)  
* `socketPath: null` - определяет UNIX сокет в node.js для отправки запроса к docker (null по умолчанию)
* `proxy: { protocol: 'https', host: '127.0.0.1', port: '8000', auth: {}}` - определяет прокси  
Если используется совместно с socketPath, то socketPath имеет больший приоритет
* `httpAgent: new http.Agent({ keepAlive: true })` или `httpsAgent: new https.Agent({ keepAlive: true })` - определяют агента, который будет использоваться при выполнении http/https запросов  
Используется для добавления параметров не включенных по умолчанию, например keepAlive
* `cancelToken: new CancelToken(function (cancel) {})` - указывает токен для отмены запроса **Устарел**
* `decompress: true` - указывает нужно ли распаковывать сжатое тело ответа (по умолчанию - true).
* `adapter: function(config) {}` - настройка обработки запросов для упрощения тестирования 

### Адаптер и тестирование
```javascript
const customAdaptrer = async (config) => {
	return {
		data: 'Response data',
		status: 200,
		statusText: 'OK',
		headers: {},
		config: config,
		request: {}
	}
}
axios({
	method: 'get',
	url: url,
	adapter: customAdaptrer
})
.then(() => {})
.catch(() => {})
```
Обязательные поля в adapter: 
* data (даныне ответа) 
* status (код статуса)
* statusText (описание кода статуса)
* headers (заголовки) 
* config (исходная конфигурация запроса)  
Необязательные: 
* request (объект запроса)

[axios-mock-adapter](https://www.npmjs.com/package/axios-mock-adapter)  
```javascript
const MockAdapter = require("axios-mock-adapter");

const mock = new MockAdapter(axios);
// Пример мокирования запроса с параметром
mock.onGet(url, { params: { searchText: "Petr" }).reply(200, {
	users: [{ name: "Petr", age: 18 }],
});

axios.get(url, { params: { searchText: "Petr" }).then(function (response) {
  console.log(response.data);
});
```
* mock.restore() - восстанавливает оригинальные обработчики запросов axios (прекращение использование мока, выполняются настоящие HTTP-запросы)
* mock.resetHandlers() - сбрасывает настройки мока, но мок-адаптер остается активным и может быть перенастроен. Используется для переопределения мока во время теста.
* mock.reset() - сбрасывает настройки мока и историю запросов. Используется между тестами, гарантируя чистое состояние для нового теста.
* mock.onGet(url).networkError() - устанавливает мок, вызывающий сетевую ошибку при каждом запросе
* mock.onGet(url).networkErrorOnce() - аналогично networkError, но срабатывает 1 раз. Следующий запрос выдаст мок по запросу, если он настроен
* mock.onGet(url).timeout() - устанавливает мок, вызывающий ошибку таймаута при каждом запросе
* mock.onGet(url).timeoutOnce() - аналогично timeoutЮ но срабатывает 1 раз. Следующий запрос выдаст мок по запросу, если он настроен
* mock.onGet(url).reply(200, users) - устанавливает мок на ответ по запросу
* mock.onGet(url).replyOnce(200, users) - аналогично reply, но срабатывает 1 раз  
`mock.onGet(url).reply(200, users).onGet(postUrl).reply(200, posts);` - доступен вызов по цепочке. 
* mock.onGet / mock.onPost и т.д. - срабатывание на конкретный метод запроса
* mock.onAny - срабатывание мока на любой метод запроса  
`mock.onGet(url).reply(200).onAny().reply(500)` - метод GET - выдаст успешный ответ, все остальные ошибку 500
* mock.onGet(url).reply(200).onAny().passThrough() - pathThrow позволяет отправлять в сеть запросы, не обработанных моком. Без использования будет ошибка  
Альтернативно можно передать параметр **onNoMatch** при создании мока для отправки в сеть запросов, не обработынных моком  
`var mock = new MockAdapter(axiosInstance, { onNoMatch: "passthrough" });`  
`onNoMatch: "throwException"` - вызовет ошибку при отправке запросов, не обработанных моком
* mock.resetHistory() - очищает историю запросов  
Для получения истории запросов нужно обратиться к `mock.history` - например `mock.history.post[1].data`

## 2. Ответы

Поля ответа: 
* data: {} - ответ сервера
* status: 200 - код статуса ответа
* statusText: 'OK' - текстовое описание кода статуса ответа
* headers: {} - заголовки
* config: {} - конфигурация axios для отправки запроса
* request: {} - параметры самого запроса в одном объекте

## 3. Перехват запросов 

`transformRequest/transformResponse` позволяют обработать передаваемые данные и заголовки до/после запроса/ответа.  
interceptors позволяют обработать config до/после отправки запроса/ответа. Используется для логирования, добавления заголовков, обработки ошибок и изменении конфига  
_Для исключения повторений в обработке ошибок, можно реализовать ее в interseptors вместо блоков catch каждого запроса_
```javascript
axios.interceptors.request.use(function (config) { 
		return config 
	}, function (error) {
		return Promise.reject(error);
	}
)
axios.interceptors.response.use(function (config) {
 		return config 
	}, function (error) {
		return Promise.reject(error);
	}
)
```

Удаление перехватчика: 
```javascript
const myInterceptor = axios.interceptors.request.use(function () {/*...*/});
axios.interceptors.request.eject(myInterceptor);
```

## 4. Отмена запроса

axios поддерживает AbortController  
```javascript
const controller = new AbortController();

axios.get('/foo/bar', {
   signal: controller.signal
})
.catch(err) {
	// два способа отловить отмену запроса AbortController
	if(signal.aborted) {}
	if(error.name = 'AbortError'
}

controller.abort()
```

Альтернативный способ:  
```javascript
const CancelToken = axios.CancelToken;
const source = CancelToken.source();

axios.get(url, {
  cancelToken: source.token
}).catch(function (thrown) {
	if (axios.isCancel(thrown)) { ...обработка отмены запроса}
	else { ...обработка ошибки } 
});

source.cancel('Необязательное сообщение');
```

Можно указать использовать оба способа в одном запросе

## 5. Обработка ошибок

**Пример** обработки ошибок  
_Поля ошибки: data, status, headers, request, message, config_  
```javascript
axios.get(url)
	.catch(function (error) {
		if (error.response) { ...запрос вызвал ответ со статусом вне диапазона 2хх }
		else if (error.request) { ...запрос был сделан, но ответ не получен }
		else { ...ошибка при настройке запроса }
	}
```
Альтернативно можно проверять вызвана ли ошибка обработкой запроса axios или другим способом: `axios.isAxiosError(error)`
```
const getPost = async () => {
	try {
		const res = await axios({})
	} catch (error) {
		// проверяем ошибку вызванную при работе с axios
		if(axios.isAxiosError(error)) {}
		// проверяем стороннюю ошибку
		else if (error instanceof Error) {}
		// остальные случаи
		else {}
	}
}
```

Параметр конфигурации `validateStatus` позволяет указать какие коды должны вызывать ошибку: 
```javascript
axios.get(url, {
  validateStatus: function (status) {
    return status < 500; // Разрешены ответы с кодом статуса < 500
  }
})
```

## 6. Работа с формами

Axios из коробки поддерживает работу с FormData. 
```javascript
const form = new FormData();
form.append('key', value)
axios.post(url, form
```
Axios автоматически превращает объект в формат FormData:
```javascript
// аналогично putForm, patchForm
axios.postForm(url, {
	key: value
})
```

## Полезные ссылки 

[Документация](https://axios-http.com/ru/docs/intro)  
[JsonPlaceholder](https://jsonplaceholder.typicode.com/guide/)