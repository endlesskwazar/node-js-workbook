# Зміст

${toc}

# Testing

## Mocha, Chai

**Mocha** - це багатофункціональний фреймворк тестування JavaScript, що працює на Node.js та в браузері, що робить асинхронне тестування простим та цікавим. **Chai** - assert - бібліотека.

Щоб встановити потрібні залежності:

```bash
npm i -D mocha chai chai-http sqlite3 mock-session
```

- sqlite3 знадобиться для бази даних. Ми будемо застосовувати для тестів легковісну реляційну базу даних.

Структура тестів mocha:

```js
describe('test suite title', function() {

    before(function() {
        // runs before all tests in this file regardless where this line is defined.
    });

    after(function() {
        // runs after all tests in this file
    });

    beforeEach(function() {
        // runs before each test in this block
    });

    afterEach(function() {
        // runs after each test in this block
    });

    // test case
    it('test 1', done => {
        //test body
    });

    // test case
    it('test 2', done => {
        //test body
    });
});
```

## Тестування додатку "books"

В якості проекту для тестування візьмемо:

- [node-js-examples](https://github.com/endlesskwazar/node-js-examples)
- branch books

Змінемо кофігурацію бази даних:

config.json:
```json
...
"test": {
    "username": "root",
    "password": null,
    "storage": "db_test.sqlite3",
    "host": "127.0.0.1",
    "dialect": "sqlite",
    "operatorsAliases": false
  },
...
```

Додамо скрипт тестування в package.json:

```json
...
"test": "NODE_ENV=test && npx sequelize-cli --env test db:migrate && mocha --timeout 10000"
...
```

Експортуємо головний додаток для можливості його тестування:

index.js:

```js
...
exports.app = app;
...
```

1. Почнимо із простого тесту, який перевіряє, що неавторизований користувач не може отримати доступ до головної сторінки:

test/home.js:

```js
let chai = require('chai');
let chaiHttp = require('chai-http');
let {app} = require('../');

chai.use(chaiHttp);

describe('/GET / not logged in', () => {
    it('it should be redirected to login page', done => {
        chai.request(app)
        .get('/')
        .end((err, res) => {
            chai.expect(res).to.have.status(200);
            chai.expect(res.text).contain('Login');
            chai.expect(res.text).contain('Password');
            done();
        });
    });
});
```

2. Далі створимо тести, які перевірять систему безпеки:

test/auth.js:

```js
let chai = require('chai');
let chaiHttp = require('chai-http');
let {app} = require('../');
let {User} = require('../models');

chai.use(chaiHttp);

describe('/GET register', () => {
    it('it should displays register form', done => {
        chai.request(app)
        .get('/register')
        .end((err, res) => {
            chai.expect(res).to.have.status(200);
            chai.expect(res.text).contain('Login');
            chai.expect(res.text).contain('Password');
            done();
        });
    });
});

describe('/POST register', () => {
    it('it should display validation errors', done => {
        chai.request(app)
        .post('/register')
        .type('form')
        .send({'email': '', 'password': ''})
        .end((err, res) => {
            chai.expect(res.text).contain('Validation notEmpty on email failed');
            chai.expect(res.text).contain('Validation isEmail on email failed');
            chai.expect(res.text).contain('Validation notEmpty on password failed');
            done();
        });
    });

    it('it should create user in database', done => {
        chai.request(app)
        .post('/register')
        .type('form')
        .send({'email': 'test@test.com', 'password': 'test'})
        .end(async (err, res) => {
            chai.expect(res).to.have.status(200);
            const user = await User.findOne({where: {email: 'test@test.com'}});
            chai.expect(user).to.not.be.null;
            done();
        });
    });
});

describe('/GET login', () => {

    before(() => {
        chai.request(app)
        .post('/register')
        .type('form')
        .send({'email': 'test@test.com', 'password': 'test'});
    });

    it('it should display login form', done => {
        chai.request(app)
        .get('/login')
        .end((err, res) => {
            chai.expect(res).to.have.status(200);
            chai.expect(res.text).contain('Login');
            chai.expect(res.text).contain('Password');
            chai.expect(res.text).contain('Submit');
            done();
        });
    });

    it('it should logged user', done => {
        chai.request(app)
        .post('/login')
        .type('form')
        .send({'email': 'test@test.com', 'password': 'test'})
        .end((err, res) => {
            chai.expect(res).to.redirect;
            done();
        });
    });

});
```

3. Тести для перевірки роботи із колекціями

# Heroku

**Heroku** - хмарна PaaS-платформа, що підтримує ряд мов програмування. Heroku, одна з перших хмарних платформ, з'явилася в червні 2007 року і спочатку підтримувала тільки мову програмування Ruby, але на даний момент список підтримуваних мов також включає в себе Java, Node.js, Scala, Clojure, Python, Go і PHP. На серверах Heroku використовуються операційні системи Debian або Ubuntu.

# Підготування Node.js - проекту до розгортання на Heroku



# Розгортання Node.js - проекту на Heroku



# Домашнє завдання

1. Протестуйте розроблений проект на лабораторній роботі №7.
2. Задеплойте проект на heroku.

> При заливанні проекту з тестами в RADME.md потрібно вказати посилання на heroku разом із ім'ям користувача і його паролем(від розробленого додатку, в не від heroku).

# Контрольні запитання

1. Поясніть процес розгортання node.js - проекту на Heroku.