# ООП, прототипы и контекст `this`

<p class="reading-time">Чтение: 12 минут</p>

> **Подтверждено проектами:** урок 17 содержит функции-конструкторы, `prototype`, `new`, замыкания, `this`, `bind`, `call` и `apply`.

## Объект и прототип

JavaScript ищет свойство сначала в самом объекте, затем идет по цепочке прототипов.

```js
function Car(brand) {
  this.brand = brand;
}

Car.prototype.drive = function () {
  return `${this.brand} едет`;
};

const car = new Car("Audi");
car.drive();
```

Оператор `new` создает объект, связывает его с `Constructor.prototype`, вызывает функцию с новым `this` и возвращает объект.

Класс дает более удобный синтаксис над тем же прототипным механизмом:

```js
class User {
  constructor(name) {
    this.name = name;
  }

  introduce() {
    return `Я ${this.name}`;
  }
}
```

## Как определяется `this`

`this` зависит от способа вызова обычной функции:

- `obj.method()` дает `this === obj`;
- `fn()` в strict mode дает `undefined`;
- `fn.call(obj)` и `fn.apply(obj)` задают контекст явно;
- `fn.bind(obj)` возвращает новую привязанную функцию;
- `new Fn()` дает новый объект;
- стрелочная функция не имеет собственного `this` и берет его снаружи.

```js
const user = { name: "Лена" };

function greet(prefix) {
  return `${prefix}, ${this.name}`;
}

greet.call(user, "Привет");
greet.apply(user, ["Здравствуйте"]);
const bound = greet.bind(user, "Добрый день");
```

## Инкапсуляция через замыкание

```js
function createCounter() {
  let value = 0;
  return {
    increment: () => ++value,
    read: () => value,
  };
}
```

`value` живет после завершения `createCounter`, потому что внутренние функции держат ссылку на лексическое окружение.

## Принципы ООП

- **инкапсуляция** скрывает детали реализации;
- **наследование** переиспользует поведение через цепочку прототипов;
- **полиморфизм** позволяет объектам отвечать на один интерфейс по-разному;
- **абстракция** оставляет важный контракт и прячет детали.

!!! warning "Частая ошибка"
    Передача `obj.method` как callback теряет объект слева от точки. Используйте обертку `() => obj.method()` или привяжите функцию через `bind`.

## Мини-задача

Создайте класс `Cart` с приватным списком товаров, методами `add`, `remove`, `total` и защитой от отрицательного количества.

[Вопросы по JavaScript](interview/javascript.md){ .md-button }
