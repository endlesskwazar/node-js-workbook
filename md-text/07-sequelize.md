# Зміст

${toc}

# ORM

В об'єктно-орієнтованому програмуванні об'єкти в програмі представляють об'єкти з реального світу. Як приклад можна навести адресну книгу, яка містить список людей разом з кількома телефонами і кількома адресами. В термінах об'єктно-орієнтованого програмування вони будуть представлені об'єктами класу «Людина», які міститимуть такі атрибути: ім'я, список (або масив) телефонів і список адрес.

Суть проблеми полягає в перетворенні таких об'єктів у форму, в якій вони можуть бути збережені у файлах або базах даних, і які легко можуть бути витягнуті в подальшому, зі збереженням властивостей об'єктів і відношень між ними. Ці об'єкти називають «постійними» (англ. persistent).

ORM (англ. Object-relational mapping, Об'єктно-реляційна проекція) — технологія програмування, яка зв'язує бази даних з концепціями об'єктно-орієнтованих мов програмування, створюючи «віртуальну об'єктну базу даних».

![](../resources/img/5/1.jpg)

# Sequilize

Sequelize - це promise-based Node.js ORM, який підтримує Postgres, MySQL, MariaDB, SQLite та Microsoft SQL Server. З можливостей:

- транзакції
- зв'язки
- leazy, eager завнтаження
- репліка читання

Встановлення:

Sequilize доступний через npm:

```bash
npm install --save sequelize
```

Також потрібно встановити пакет в залежності від БД:

```bash
npm install --save pg pg-hstore # Postgres
npm install --save mysql2
npm install --save mariadb
npm install --save sqlite3
npm install --save tedious # Microsoft SQL Server
```

# Quick demo

В прикладі використовується mariadb 10. Скрипт створення БД наступний:

```sql
create database sq_demo_1;
use sq_demo_1;

create table users(
id int primary key auto_increment,
  name varchar(255) not null
);
```

![](../resources/img/5/2.png)

index.js
```js
const { Sequelize, Model, DataTypes } = require('sequelize');
const sequelize = new Sequelize('mariadb://root@localhost:3306/sq_demo_1');

class User extends Model {}
User.init({
  id: {
      type: DataTypes.INTEGER,
      autoIncrement: true,
      primaryKey: true
  },
  name: DataTypes.STRING
}, { sequelize, modelName: 'user', timestamps: false });

sequelize.sync()
  .then(() => User.create({
    name: 'alex'
  }))
  .then(user => {
    console.log(user.toJSON());
  });
```

![](../resources/img/5/3.png)

# Connection string & Dialect

Connection string - строка підключення, яка містить всі необхідні параметри для підключення до СУБД(адресу, ім'я користувача, пароль і т.д.)

**mysql**

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mysql'
})
```

**mariadb**

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mariadb',
  dialectOptions: {connectTimeout: 1000} // mariadb connector option
})
```

або

```js
const sequelize = new Sequelize('mariadb://user:password@example.com:9821/database')
```

**sqlite**

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  // sqlite! now!
  dialect: 'sqlite',

  // the storage engine for sqlite
  // - default ':memory:'
  storage: 'path/to/database.sqlite'
})
```

**pgsql**

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  // gimme postgres, please!
  dialect: 'postgres'
})
```

**mssql**

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mssql',
  dialectOptions: {
    options: {
      useUTC: false,
      dateFirst: 1,
    }
  }
})
```

# Model definition

Для визначення відображень між моделлю та таблицею використовуйте метод define або init:

```js
class Project extends Model {}
Project.init({
  title: Sequelize.STRING,
  description: Sequelize.TEXT
}, { sequelize, modelName: 'project' });

class Task extends Model {}
Task.init({
  title: Sequelize.STRING,
  description: Sequelize.TEXT,
  deadline: Sequelize.DATE
}, { sequelize, modelName: 'task' })
```

Кожен стовпець повинен мати тип даних:

```js
Sequelize.STRING                      // VARCHAR(255)
Sequelize.STRING(1234)                // VARCHAR(1234)
Sequelize.STRING.BINARY               // VARCHAR BINARY
Sequelize.TEXT                        // TEXT
Sequelize.TEXT('tiny')                // TINYTEXT
Sequelize.CITEXT                      // CITEXT      PostgreSQL and SQLite only.

Sequelize.INTEGER                     // INTEGER
Sequelize.BIGINT                      // BIGINT
Sequelize.BIGINT(11)                  // BIGINT(11)

Sequelize.FLOAT                       // FLOAT
Sequelize.FLOAT(11)                   // FLOAT(11)
Sequelize.FLOAT(11, 10)               // FLOAT(11,10)

Sequelize.REAL                        // REAL        PostgreSQL only.
Sequelize.REAL(11)                    // REAL(11)    PostgreSQL only.
Sequelize.REAL(11, 12)                // REAL(11,12) PostgreSQL only.

Sequelize.DOUBLE                      // DOUBLE
Sequelize.DOUBLE(11)                  // DOUBLE(11)
Sequelize.DOUBLE(11, 10)              // DOUBLE(11,10)

Sequelize.DECIMAL                     // DECIMAL
Sequelize.DECIMAL(10, 2)              // DECIMAL(10,2)

Sequelize.DATE                        // DATETIME for mysql / sqlite, TIMESTAMP WITH TIME ZONE for postgres
Sequelize.DATE(6)                     // DATETIME(6) for mysql 5.6.4+. Fractional seconds support with up to 6 digits of precision
Sequelize.DATEONLY                    // DATE without time.
Sequelize.BOOLEAN                     // TINYINT(1)

Sequelize.ENUM('value 1', 'value 2')  // An ENUM with allowed values 'value 1' and 'value 2'
Sequelize.ARRAY(Sequelize.TEXT)       // Defines an array. PostgreSQL only.
Sequelize.ARRAY(Sequelize.ENUM)       // Defines an array of ENUM. PostgreSQL only.

Sequelize.JSON                        // JSON column. PostgreSQL, SQLite and MySQL only.
Sequelize.JSONB                       // JSONB column. PostgreSQL only.

Sequelize.BLOB                        // BLOB (bytea for PostgreSQL)
Sequelize.BLOB('tiny')                // TINYBLOB (bytea for PostgreSQL. Other options are medium and long)

Sequelize.UUID                        // UUID datatype for PostgreSQL and SQLite, CHAR(36) BINARY for MySQL (use defaultValue: Sequelize.UUIDV1 or Sequelize.UUIDV4 to make sequelize generate the ids automatically)

Sequelize.CIDR                        // CIDR datatype for PostgreSQL
Sequelize.INET                        // INET datatype for PostgreSQL
Sequelize.MACADDR                     // MACADDR datatype for PostgreSQL

Sequelize.RANGE(Sequelize.INTEGER)    // Defines int4range range. PostgreSQL only.
Sequelize.RANGE(Sequelize.BIGINT)     // Defined int8range range. PostgreSQL only.
Sequelize.RANGE(Sequelize.DATE)       // Defines tstzrange range. PostgreSQL only.
Sequelize.RANGE(Sequelize.DATEONLY)   // Defines daterange range. PostgreSQL only.
Sequelize.RANGE(Sequelize.DECIMAL)    // Defines numrange range. PostgreSQL only.

Sequelize.ARRAY(Sequelize.RANGE(Sequelize.DATE)) // Defines array of tstzrange ranges. PostgreSQL only.

Sequelize.GEOMETRY                    // Spatial column.  PostgreSQL (with PostGIS) or MySQL only.
Sequelize.GEOMETRY('POINT')           // Spatial column with geometry type. PostgreSQL (with PostGIS) or MySQL only.
Sequelize.GEOMETRY('POINT', 4326)     // Spatial column with geometry type and SRID.  PostgreSQL (with PostGIS) or MySQL only.
```

Крім типів даних, існує безліч варіантів, які можна встановити в кожному стовпчику.

```js
class Foo extends Model {}
Foo.init({
 // автоматично встановить прапор у true, якщо не встановлено
 flag: { type: Sequelize.BOOLEAN, allowNull: false, defaultValue: true },
 // значення за замовчуванням для дат = = поточний час
 myDate: { type: Sequelize.DATE, defaultValue: Sequelize.NOW },
 // Встановлення allowNull не дозволяє збереження поля без значення
 title: { type: Sequelize.STRING, allowNull: false },
 // Створення двох об'єктів з однаковим значенням призведе до помилки.
 uniqueOne: { type: Sequelize.STRING,  unique: 'compositeIndex' },
 uniqueTwo: { type: Sequelize.INTEGER, unique: 'compositeIndex' },
 // unique властивість - це просто скорочення для створення унікального обмеження.
 someUnique: { type: Sequelize.STRING, unique: true },
 // Це точно так само, як створення індексу в параметрах моделі.
 { someUnique: { type: Sequelize.STRING } },
 { indexes: [ { unique: true, fields: [ 'someUnique' ] } ] },
 // Встановлення первинного ключа
 identifier: { type: Sequelize.STRING, primaryKey: true },
 // autoIncrement
 incrementMe: { type: Sequelize.INTEGER, autoIncrement: true },
 // Ви можете вказати власну назву стовпця за допомогою атрибута "field":
 fieldWithUnderscores: { type: Sequelize.STRING, field: 'field_with_underscores' },
 // Можна створити зовнішні ключі:
 bar_id: {
   type: Sequelize.INTEGER,
   references: {
     // Це посилання на іншу модель
     model: Bar,
     // Це назва стовпця іншої моделі
     key: 'id',
   }
 },
 // Можна додавати коментарі до стовпців лише для MySQL, PostgreSQL та MSSQL
 commentMe: {
   type: Sequelize.INTEGER,
   comment: 'This is a column name that has a comment'
 }
}, {
  sequelize,
  modelName: 'foo'
});
```


# Migrations & seeds

**Міграції** - щось на зразок системи контролю версій для вашої бази даних. Вони дозволяють вашій команді змінювати структуру БД, в той же час залишаючись в курсі змін інших учасників. Міграції зазвичай йдуть рука об руку з будівником структур для більш простого поводження з архітектурою вашої бази даних. Якщо ви коли-небудь просили колегу вручну додати стовпець в його локальну БД, значить ви стикалися з проблемою, яку вирішують міграції БД.

**seeds** - це початкове заповнення бд даними. Seeding - це процес, при якому під час її встановлення в базу даних подається початковий набір даних. Дані можуть бути фіктивними даними або необхідними даними, такими як початковий обліковий запис адміністратора.

Створимо новий node.js - проект і встановимо sequilize і необхідний драйвер БД, в моєму випадку, це mariadb:

```bash
npm init
npm i sequelize
npm i mariadb
```

![](../resources/img/5/4.png)

**Спочатку потрібно створити структуру проекту для sequelize**:

```bash
npx sequelize-cli init
```

Це створить наступні папки:


- config, містить строку підключення
- models, містить всі моделі
- migrations, містить файли міграції
- seeders, містить seed - файли

**Потім налаштуємо строку підключення**:

```json
{
  "development": {
    "username": "root",
    "password": null,
    "database": "sq_demo_2",
    "host": "127.0.0.1",
    "dialect": "mariadb",
    "operatorsAliases": false
  },
  "test": {
    "username": "root",
    "password": null,
    "database": "sq_demo_2",
    "host": "127.0.0.1",
    "dialect": "mariadb",
    "operatorsAliases": false
  },
  "production": {
    "username": "root",
    "password": null,
    "database": "sq_demo_2",
    "host": "127.0.0.1",
    "dialect": "mariadb",
    "operatorsAliases": false
  }
}

```

**Створимо модель і міграцію**

```bash
npx sequelize-cli model:generate --name Student --attributes name:string
```

ця команда згенерує наступні файли:

migrations/20191011073932-create-student.js
```js
'use strict';
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('Students', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      name: {
        type: Sequelize.STRING
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('Students');
  }
};
```

models/student.js
```js
'use strict';
module.exports = (sequelize, DataTypes) => {
  const Student = sequelize.define('Student', {
    name: DataTypes.STRING
  }, {});
  Student.associate = function(models) {
    // associations can be defined here
  };
  return Student;
};
```

> Зверніт увагу, що sequelize для комінакації використовує лише моделі. Коли міграції це змінна структури таблиці.

**Запустимо міграції**

> Зверніть увагу, на те, що міграції не створюють саму базу даних, а лише таблиці і їх поля. Ми можемо створити базу даних вручну або скориставшить командою db:create. Деякі діалекти як mariadb не підтримують дану команду.

```bash
npx sequelize-cli db:migrate
```

![](../resources/img/5/5.png)

Подивимося на згенеровані таблиці:

![](../resources/img/5/6.png)



**Тепер, уявімо, що нам потрібно додати поле age до моделі Student**:

Створимо пусту міграцію:

```bash
npx sequelize-cli migration:generate --name students_add_age_field
```

![](../resources/img/5/7.png)

Ця команда згенерує пустий файл міграції:

```js
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    /*
      Add altering commands here.
      Return a promise to correctly handle asynchronicity.

      Example:
      return queryInterface.createTable('users', { id: Sequelize.INTEGER });
    */
  },

  down: (queryInterface, Sequelize) => {
    /*
      Add reverting commands here.
      Return a promise to correctly handle asynchronicity.

      Example:
      return queryInterface.dropTable('users');
    */
  }
};
```

Додамо поле age до вже існуючої таблиці Students:

```js
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.addColumn(
      'Students',
      'age',
     Sequelize.INTEGER
    );
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.removeColumn(
      'Students',
      'age',
     Sequelize.INTEGER
    );
  }
};
```

Запустимо міграції:

```bash
npx sequelize-cli db:migrate
```

![](../resources/img/5/8.png)

І подивимося тепер на структуру таблиці Students:

![](../resources/img/5/9.png)

**Повний код додатку** можна знайти на [node-js-examples](https://github.com/endlesskwazar/node-js-examples) гілка sq_demo_2

# CRUD

**CRUD** — (англ. create read update delete) 4 базові функції управління даними «створення, зчитування, зміна і видалення».

**Стосовно бази даних**.

|Операція|SQL|
|-|-|
|Create|Insert|
|Read|Select|
|Update|Update|
|Delete|Delete|

В якості першопочаткового проекту використаємо [node-js-examples](https://github.com/endlesskwazar/node-js-examples) гілка sq_demo_2

## Create

Для створення запису можна скористатися функцією create () - це проміс, який приймає об'єкт, і в якості resolve повертає створений об'єкт із бази даних:

```js
const {Student} = require('./models');

Student.create({
    'name': 'Ivan',
    'age': 33
})
.then(created => {
    console.log(`Id of created student ${created.id}`)
})
.catch(err => {
    console.log(err);
});
```

![](../resources/img/5/10.png)

## Read

Під читанням ми можемо отримати список всіх студентів, одного студента по id і кастомний пошук:

### findByPk

Функція findByPk дозволяє знайти один екземпляр студента використовуючи поле id:

```js
const {Student} = require('./models');

Student.findByPk(1)
.then(student => {
    console.log(`Student with id 1 has name: ${student.name}`)
})
.catch(err => {
    console.log(err);
})
```

![](../resources/img/5/11.png)

### findAll

Функція findAll дозволяє отримати всі записи в таблиці:

```js
const {Student} = require('./models');

Student.findAll()
.then(students => {
    students.forEach(student => {
        console.log(`Student id:${student.id} name:${student.name}`);
    });
})
.catch(err => {
    console.log(err);
})
```

![](../resources/img/5/12.png)

## Update

## Delete

# Relations

## One to One

## One to Many

## Many to Many

# Домашнє завдання

# Контрольні запитання