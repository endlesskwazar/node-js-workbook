# React. Fetch data from retome server

Компоненти класу ES6 React мають методи життєвого циклу. Метод життєвого циклу **render ()** є обов'язковим для виведення елемента React.

Існує ще один метод життєвого циклу, який ідеально підходить для отримання даних: **componentDidMount ()**. Коли цей метод запускається, компонент вже був намальований один раз методом render (), але він буде повторно відображатися, коли дані будуть отримані в локальний стан **setState ()**. Після цього локальний стан може бути використаний у методі render () для його відображення або для передачі його як реквізит.

Метод життєвого циклу **componentDidMount ()** - найкраще місце для отримання даних. Але як отримати дані зрештою? Екосистема React - це гнучкий фреймворк, тому ви можете вибрати власне рішення для отримання даних. Найпростіше, можна використати нативний API - **Fetch API**.

```js
import React, { Component } from 'react';
class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      data: null,
    };
  }
  componentDidMount() {
    fetch('https://api.mydomain.com')
      .then(response => response.json())
      .then(data => this.setState({ data }));
  }
  ...
}
export default App;
```

Звичайно, потрібні отримані у локальний стан. Але що ще? Є ще дві властивості, які можна зберігати в стані: **стан завантаження** та **стан помилки**. І те й інше покращить вашу користувацьку роботу для кінцевих користувачів вашої програми.

Стан завантаження слід використовувати, щоб вказати, що відбувається асинхронний запит. Між обома методами render () отримані дані відкладені через надходження асинхронно. Таким чином, ви можете додати показник завантаження під час очікування. У вашому методі життєвого циклу вам доведеться перемикати властивість з false на true і коли дані отримані з true у false.

```js
...
class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      hits: [],
      isLoading: false,
    };
  }
  componentDidMount() {
    this.setState({ isLoading: true });
    fetch(API + DEFAULT_QUERY)
      .then(response => response.json())
      .then(data => this.setState({ hits: data.hits, isLoading: false }));
  }
  ...
}
export default App;
```


У вашому методі render () ви можете використовувати умовне відображення React для відображення або індикатора завантаження, або отриманих даних.

```js
...
class App extends Component {
  ...
  render() {
    const { hits, isLoading } = this.state;
    if (isLoading) {
      return <p>Loading ...</p>;
    }
    return (
      <ul>
        {hits.map(hit =>
          <li key={hit.objectID}>
            <a href={hit.url}>{hit.title}</a>
          </li>
        )}
      </ul>
    );
  }
}
```


Другим станом, який ви можете зберегти у своєму локальному стані є стан помилок. Коли у вашій програмі виникає помилка, немає нічого гіршого, ніж давати кінцевому користувачеві ніяких вказівок про помилку.

```js
...
class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      hits: [],
      isLoading: false,
      error: null,
    };
  }
  ...
}
```

При використанні обіцянок блок catch () зазвичай використовується після блоку then () для обробки помилок. Ось чому його можна використовувати для Fetch API.

```js
...
class App extends Component {
  ...
  componentDidMount() {
    this.setState({ isLoading: true });
    fetch(API + DEFAULT_QUERY)
      .then(response => response.json())
      .then(data => this.setState({ hits: data.hits, isLoading: false }))
      .catch(error => this.setState({ error, isLoading: false }));
  }
  ...
}
```


На жаль, Fetch не використовує свій блок catch() для кожного помилкового коду статусу. Наприклад, коли HTTP 404 трапляється, він не запускається у блоці catch. Але ви можете змусити його запустити цей блок, видавши помилку, коли ваша відповідь не відповідає очікуваним даним.

```js
...
class App extends Component {
  ...
  componentDidMount() {
    this.setState({ isLoading: true });
    fetch(API + DEFAULT_QUERY)
      .then(response => {
        if (response.ok) {
          return response.json();
        } else {
          throw new Error('Something went wrong ...');
        }
      })
      .then(data => this.setState({ hits: data.hits, isLoading: false }))
      .catch(error => this.setState({ error, isLoading: false }));
  }
  ...
}
```


І останнє, але не менш важливе, ви можете показати повідомлення про помилку в методі render () як умовне відображення знову.

```js
...
class App extends Component {
  ...
  render() {
    const { hits, isLoading, error } = this.state;
    if (error) {
      return <p>{error.message}</p>;
    }
    if (isLoading) {
      return <p>Loading ...</p>;
    }
    return (
      <ul>
        {hits.map(hit =>
          <li key={hit.objectID}>
            <a href={hit.url}>{hit.title}</a>
          </li>
        )}
      </ul>
    );
  }
}
```

repo - [node-js-examples](https://github.com/endlesskwazar/node-js-examples)
branch - react-fetch

# React hooks

**Hooks** - це функції, які дозволяють «зачепити» React - state та функції життєвого циклу функціональних компонентів. Hooks не працюють у класах - вони дозволяють використовувати React без класів.

State є невід'ємною частиною React. Він дозволяє нам оголошувати змінні, які містять дані, які, в свою чергу, будуть використовуватися в нашому додатку. За допомогою класів state зазвичай визначається наступним чином:

```js
class Example extends React.Component{
  constructor(props){
    super(props);
    this.state = {
      count: 0
    }
  }
}
```

## state hook

```js
import React, { useState } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

useState повертає пару: поточне значення стану і функцію, яка дозволяє оновити її. Ви можете викликати цю функцію з обробника подій або в іншому місці. Він подібний до цього .setState у класі, але він не об'єднує старе та нове стан.

Ви можете скористатися useState більше одного разу в одному компоненті:

```js
function ExampleWithManyStates() {
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
}
```

## effect hook

useEffect, додає можливість виконувати side effects в функції. Він служить тій самій меті, що і componentDidMount, componentDidUpdate і componentWillUnmount в класах React, але уніфікований в один API.

```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>`
    </div>
  );
}
```

## hooks rules

Hooks є функціями JavaScript, але вони накладають два додаткових правила:

- Викликати лише на верхньому рівні. Не викликайте hooks в циклах, умовах або вкладених функціях.
- Викликайте тільки з компонентів функції React. Не икликайте Hooks з регулярних функцій JavaScript.

# React router

# React context api

# Домашнє завдання

# Контрольні запитання

1. віаві
2. віа
3. івуа
4. іва
5. іва