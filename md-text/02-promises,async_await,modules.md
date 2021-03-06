# Зміст

${toc}

# Callbacks example

Змоделюємо ситуацію, де є три функції. Виклик другої функції залежить від результату виконання першої, а виклик третьої залежить від виконання другої. Всі три функції повинні приймати callback - функцію як параметр.

Подивимося на код, який це ілюструє:

```js
first(2, function (firstRes, err) {
  if (!err) {
    console.log(`Exec first callback value ${firstRes}`);
    second(firstRes, function(secondRes, err) {
      if (!err){
        console.log(`Exec second callback value ${secondRes}`);
        third(secondRes, function(thirdRes, err){
          if (!err) {
            console.log(`Exec Third callback value ${thirdRes}`);
          }
        })
      }
    });
  }
});

function first(value, callback) {
  console.log('Executing first func');
  callback(value + 2, false);
}

function second(value, callback) {
  console.log('Executing second func');
  callback(value + 2, false);
}

function third(value, callback) {
  console.log('Executing third func');
  callback(value + 2, false);
}
```

![](../resources/img/2/1.png)

Якщо ми змінимо 2-гий параметр, виклику cack у якійсь із функцій ланцюжок викликів функцій припиниться:

```js
...
function second(value, callback) {
  console.log('Executing second func');
  callback(value + 2, true);
}
...
```

![](../resources/img/2/1.png)

Недоліком такого коду є той факт, що із ростом складності код швидко перетворюється в малозрозумілі, багаторазово вкладені блоки. Для такого навіть є назва **"callback hell"**.

# Promises

**Змоделюймо іншу ситуацію**:

Уявіть, що ви дитина. Ваша мама обіцяє вам, що наступного тижня купить вам новий телефон.

Ви не знаєте, чи отримаєте цей телефон до наступного тижня. Ваша мама може дійсно придбати вам новий телефон або незадоволена вашою успішністю в школі не дотримати обіцянку.

**Спочатку реалізуємо це за допомогою callback**:

```js
function byNewPhone(avarageMark, callback) {
  if (avarageMark > 3) {
    const phone = {
      model: "Simens a62"
    }
    callback(phone, null);
  }
  else {
    callback(null, "Mom isn`t happy with your marks!!!");
  }
}

byNewPhone(4, (phone, err) => {
  if(err) {
    console.log("Fuck!!!");
  }
  else {
    console.log("Fuck yeah!!!");
  }
});
```

![](../resources/img/2/7.png)

**А тепер розгляньмо інший спосіб реалізації із використанням Promises**.

Promise - це спеціальний об'єкт, який містить зберігає свій стан:
- pending - очікує на розгляд(pending): Ви не знаєте, чи отримаєте цей телефон до наступного тижня.
- resolved - виконана(resolved): ваша мама дійсно купує вам новий телефон.
- rejected - відхилений(rejected): ви не отримуєте новий телефон.

![](../resources/img/2/8.png)

Переробимо наш приклад, використовуючи проміс:

```js
var isMomHappy = false;

// Promise
var willIGetNewPhone = new Promise(
	function (resolve, reject) {
		if (isMomHappy) {
			var phone = {
				brand: 'Samsung',
				color: 'black'
			};
			resolve(phone); // fulfilled
		} else {
			var reason = new Error('mom is not happy');
			reject(reason); // reject
		}

	}
);

var askMom = function () {
	willIGetNewPhone
		.then(function (phone) {
			console.log(phone);
		})
		.catch(function (error) {
			console.log(error.message);
		});
};

askMom();
```

Щоб виконати якийсь код, коли проміс буде resolved потрібно кинути callback в метод then. Якщо ми хочемо виконати код на rejected потрібно кинути callback в rejected.

Скажімо, ви, обіцяєте своєму другові, що ви покажете новий телефон, коли ваша мама купить його вам.

```js
var showOff = function (phone) {
	return new Promise(
		function (resolve, reject) {
			var message = 'Hey friend, I have a new ' +
				phone.color + ' ' + phone.brand + ' phone';

			resolve(message);
		}
	);
};

var askMom = function () {
	willIGetNewPhone
	.then(showOff) // chain it here
	.then(function (fulfilled) {
			console.log(fulfilled);
		})
		.catch(function (error) {
			console.log(error.message);
		});
};
```

Код повністю:
```js
var isMomHappy = true;

// Promise
var willIGetNewPhone = new Promise(
	function (resolve, reject) {
		if (isMomHappy) {
			var phone = {
				brand: 'Samsung',
				color: 'black'
			};
			resolve(phone); // fulfilled
		} else {
			var reason = new Error('mom is not happy');
			reject(reason); // reject
		}

	}
);

var showOff = function (phone) {
	return new Promise(
		function (resolve, reject) {
			var message = 'Hey friend, I have a new ' +
				phone.color + ' ' + phone.brand + ' phone';

			resolve(message);
		}
	);
};

var askMom = function () {
	willIGetNewPhone
	.then(showOff)
		.then(function (fulfilled) {
			console.log(fulfilled);
		})
		.catch(function (error) {
			console.log(error.message);
		});
};

askMom();
```

![](../resources/img/2/9.png)



# Асинхроний JavaScript. async/await

Проміси і callbacks за дефолтом не є асинхронними, хоча в javascript існують і асинхронні. Розгляньмо одну із них setTimeout(). setTimeout() - дозволяє виконати відстрочений код без блокування потоку:

```js
setTimeout(() => {
  console.log("Callback from setTimeout");
}, 3000);

console.log("Other code");
```

![](../resources/img/2/3.png)

Спочатку виконується функція setTimeout, оскільки її виконання відстрочене на 3 секунди вона реєструє переданий callback, і передає керування кодові, який слідує нижче. Як тільки пройде 3 секунди виконається функція callback - цей код асинхронний. Ця модель хороша, коли ми чекаємо відповіді від сервера, читаємо файл і т.д.

Проміси також бувають асинхронні. Створімо такий проміс:

```js
const getResponse = new Promise((resolve, reject) => {
  setTimeout(() => {
    let response = "This is response from server";
    resolve(response);
  }, 2000);
});

getResponse
.then((data) => {
  console.log(data);
})

console.log("Other code");
```

![](../resources/img/2/4.png)

Проблема полягає в тому, що ми, люди, не звикли мислити асинхронно. Було б краще якщо б такий код виглядав так:

```js
response = getResponse();
```

Ідея async/await полягає саме в цьому. 

Давайте спочатку розглянемо приклад:

```js
const getResponse = function() {
  return new Promise((resolve, reject) => {
  setTimeout(() => {
    let response = "This is response from server";
    resolve(response);
  }, 2000);
});
} 

async function fetchData() {
  console.log("starting fetching data");
  const data = await getResponse();
  console.log(data);
}

fetchData();
```

![](../resources/img/2/5.png)

Як бачимо порядок коду став синхронним, якщо ми видалимо async/await ми отримаємо поламану логіку:

```js
const getResponse = function() {
  return new Promise((resolve, reject) => {
  setTimeout(() => {
    let response = "This is response from server";
    resolve(response);
  }, 2000);
});
} 

function fetchData() {
  console.log("starting fetching data");
  const data = getResponse();
  console.log(data);
}

fetchData();
```

![](../resources/img/2/6.png)

- Async функції. Які оголошуються додаванням слова async, наприклад ```js async function doAsyncStuff () {... code}```.
- Ваш код може встати на паузу в очікуванні Async функції з await
- await повертає те, що асинхронна функція віддає при завершенні.
- Await може бути використано тільки всередині async функції.

Єдине, що нам залишилося розглянути, а що якщо трапиться помилка, через те, що від catch на промісі ми відмовилися можна використати блок try...catch:

```js
const getResponse = function() {
  return new Promise((resolve, reject) => {
  setTimeout(() => {
    let response = "This is response from server";
    reject("Error");
  }, 2000);
});
} 

async function fetchData() {
  console.log("starting fetching data");
  try{
      const data = await getResponse();
      console.log(data);
  }
  catch (e){
    console.log(e);
  }
}

fetchData();
```

# Common.js модулі

Довгий час в JavaScript був відсутній синтаксис модулів на рівні мови. Це не було проблемою, тому що перші скрипти були маленькими і простими. У модулях не було необхідності.

Але з часом скрипти ставали складнішими, тому спільнота придумала кілька варіантів організації коду в модулі. З'явилися бібліотеки для динамічного завантаження модулів.

Наприклад:

- AMD - одна з найстаріших модульних систем, спочатку реалізована бібліотекою require.js.
- CommonJS - модульна система, створена для сервера Node.js.
- UMD - ще одна модульна система, пропонується як універсальна, сумісна з AMD і CommonJS.

Ми розглянемо Common.js модулі, тому що вони підтримуються з коробки без ніяких налаштувань.

Створимо наступну структуру директорій:

![](../resources/img/2/10.png)

math.js:
```js
const pi = 3.4;
function sum(a, b){
	return a + b;
}
exports.pi = pi;
exports.sum = sum;
```

Якщо нам потрібно експортувати об'єкт із файлу, потрібно його просто додати поле в стандартний об'єкт exports після чого його можна буде імпортувати за допомогою require.

index.js:
```js
const math = require('./lib/math')
console.log(math.sum(1,2)); //3
console.log(math.pi); //3.4
```

![](../resources/img/2/11.png)

Оскільки require повертає об'єкт ми можемо використати "Object destructuring":

index.js:
```js
const {sum, pi} = require('./lib/math')
console.log(sum(1,2)); //3
console.log(pi); //3.4
```

## Імпортування черех index.js

Уявіть собі наступну структуру проекту:

![](../resources/img/2/12.png)

В директорії controllers в нас багато файлів. Ось виглядає файл, якому б знадобилося деяка порція цих файлів:

```js
const {aboutController} = require('./controllers/about-controller');
const {contactController} = require('./controllers/contact-controller');
const {homeController} = require('./controllers/home-controller');
const {productsController} = require('./controllers/products-controller');
```

Можна спростити ці імпорти, створивши файл index.js імпортувати всі необхідні модулі і їх експортувати єдиним об'єктам. Так ми зробимо використання коду клієнтами зручнішим:

controllers/index.js
```js
const {aboutController} = require('./about-controller');
const {contactController} = require('./contact-controller');
const {homeController} = require('./home-controller');
const {productsController} = require('./products-controller');

exports.aboutController = aboutController;
exports.contactController = contactController;
exports.homeController = homeController;
exports.productsController = productsController;
```

Тоді клієнт index.js буде виглядати як:

```js
const {aboutController, homeController} = require('./controllers');
aboutController();
```

# npm

**npm** - менеджер пакетів, що входить до складу Node.js. Знайти пакети для установки можна на [npmjs](https://www.npmjs.com/).

Установка пакету проводиться за допомогою команди:
```bash
npm install <package_name>
//or
npm i <package_name>
```

Наприклад:
```bash
npm i lodash
```

> Зверніть увагу на те, що якщо в робочій директорії куди встановлюється пакет не буде ініціалізований npm - проект установка завершиться з помилкою, хоча пакет буде дійсно встановлений.

![](../resources/img/2/14.png)

Тому краще ініціалізувати проект, використовуючи команду:
```bash
npm init
```

В процесі буде необхідно ввести наступну інформацію:
- package name - ім'я проекту
- version - версія проекту
- description - опис проекту
- entry point - точка входу
- test command - команда для запуску тестів
- git repository - адреса git - репозиторія
- keywords - ключові слова для пошуку проекту
- author - ім'я, нік автора
- license - ліцензія список ліцензій

Тепер ми можемо встановлювати пакети.

В робочій директорії з'явилися наступні директорії і файли:

- node_modules - директорія в якій знаходяться всі пакети(потрібно додавати в .gitignore)
- package.json - файл опису проекту і його залежностей(не потрібно додавати в .gitignore)
- package-lock.json - package-lock.json використовується виключно для блокування залежностей від певного номера версії.(непотрібно додавати в .gitignore)

Можна використати наступні ключі для модифікації доступності об'єкта від оточення використовувати наступні ключі щоб змінити це:

- Для цілей розробки:

```bash
npm i --save-dev
// or
npm i -D
```

```js
"devDependencies": {
    "lodash": "^4.17.15"
  }
```

- Для продакшн

```npm
npm i --save-prod
// or
npm i -P
```

```js
"dependencies": {
  "lodash": "^4.17.15"
}
```

Щоб використати завантажений пакет його достатньо імпортувати:
```js
var _ = require('lodash');
```


## Версії в package.json

Якщо Ви помітили package.json містить перелік пакетів і їх версій від, яких залежить наш проект.

Номер версії є синтаксисом semver, який позначає кожен розділ з різним значенням. semver розбивається на три секції, розділені крапкою.

```
major.minor.patch
1.0.2
```

Перед номером версій можуть ставитися символи:

- '~' - це означає встановити версію 1.0.2 або останню версію патча, таку як 1.0.4.
- '^' - це означає встановити версію 1.0.2 або останню мінорну або виправлену версію, таку як 1.1.0.


## start scripts

npm містить поле під назвою scripts в файлі package.json проекту для того, щоб робити такі команди, як npm test, фактично, виконує вміст поля scripts.test, і npm start, що викликає інструкції з поля scripts. start.

npm test і npm start - це, всього лише, зручні посилання для npm run test і npm run start. За допомогою npm run можна виконати абсолютно будь-який вміст будь-якого поля всередині scripts.

Крім того, npm run чудовий ще й тому, що npm автоматично додає в $ PATH директорію node_modules / .bin, так що ви можете безпосередньо запускати команди з dependencies або devDependencies, без необхідності встановлювати ці модулі глобально. npm-пакети, які ви хотіли б включити у свій воркфлоу, повинні мати всього лише простий інтерфейс командного рядка, і ви зможете написати просту автоматизацію самостійно.

Створимо власний скрипт під назвою my:

```js
...
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "my": "node -v && node index.js"
}
...
```

Запустимо цей скрипт за допомогою команди:
```bash
npm run my
```

![](../resources/img/2/15.png)

# Домашнє завдання

Створіть наступну структуру проекту:
```
--models/index.js
--services/index.js
index.js
```

В models/index.js створіть клас, використовуючи синтаксис ES6. В services/index.js створіть promise, який в залежності від локального флажка повертає масив об'єктів створеного класу або помилку. Файл index.js імпортує проміс із services/index.js і виконує його за допомогою then,catch і async/await.

# Контрольні запитання

1. Що таке "Callback hell"?
2. Що таке Promise? Назвіть стани? Як прослуховувати зміну стану Promise?
3. Що таке асинхронний код?
4. Що таке async/await?
5. Поясніть, як можна імпортувати функцію з одного модуля в інший, використовуючи Common.js.
6. Що таке npm? Як можна ініціалізувати проект і додати до нього залежність?