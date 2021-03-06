# Types and grammar


* [Типы](#Типы)
* [Значения](#Значения)
  * [Массивы](#Массивы)
  * [Строки](#Строки)
  * [Числа](#Строки)
  * [Особые значения](#Особые-значения)
    * [Особые числа](#Особые-числа)
  * [Значения против Ссылок](#Значения-против-Ссылок)
* [Нативы](#Нативы)
  * [Обертки](#Обертки)
  * [Подводные камни объектов-оберток](#Подводные-камни-объектов-оберток)
  * [Конструкторы](#Конструкторы)
    * [Конструкторы Function и RegExp](#Конструкторы-Function-и-RegExp)
    * [Date и Error](#Date-и-Error)
    * [Symbol](#Symbol)
  * [Прототипы](#Прототипы)
    * [Использование прототипов по умолчанию](#Использование-прототипов-по-умолчанию)

Тип - это внутренний, встроенный набор характеристик, которые однозначно идентифицируют поведение конкретной величины
и отличают его от других величин, как для движка, так и для разработчика.

Типы
======
В JS 7 встроенных типов:
* `null`
* `undefined`
* `boolean`
* `number`
* `string`
* `object`
* `symbol`

Все типы, кроме `object` называются примитивами

У переменных нет типов, они есть только у значений. Переменные принимают в себя любые значения любых типов.

:warning: `typeof null === 'object'`

:memo: Проверка на null, где `const a = null`;
`(!a && typeof === 'object') // true`

На самом деле функция - 'function object', с внутренним свойством `[[CALL]]`, которое позволяет функции быть вызванной

:memo: Смотри страницу 79 и 164 спецификации
###### здесь и дальше "спецификация" - ECMAScript 2018 9th Edition / June 2018

Вызов не объявленной (`undeclared`) переменной вызывает ошибку. Использование объявленной, но "всплывшей" (var) переменной, в которой нет значения (`undefined`) - ошибку не вызывает.
```
console.log(a); // ReferenceError: a is not defined
```
```
console.log(a); // undefined, поскольку var a объявлена и всплывает наверх функции
var a;
```
или
```
console.log(a); // ReferenceError: a is not defined
let a = 42;
```

При этом возможно использование `undeclared` переменной вместе с `typeof`, результатом будет `undefined`. Если после этого использовать необъявленную переменную, ошибка все равно появится.
```
console.log(typeof pew);
console.log(pew);
// undefined
// ReferenceError: pew is not defined
```


Значения
======

## Массивы
Массивы имеют цифровые индексы, но также принимают строки в качестве ключей, эти значения не учитываются в длине массива. Если строка будет состоять из числового значения, то она будет рассматриваться как обычный индекс массива (и увеличит длину, в случае необходимости):
```
const a = [];
a["10"] = true;
console.log(a); // [empty x 10, true];
```
Конструктор `Array` не требует `new`, поэтому вызовы `new Array(1, 2, 3)` и `Array(1, 2, 3)` идентичны.

##### Проблемы создания массива через конструктор
:exclamation: Если в конструктор `Array` передан только 1 числовой аргумент - будет создан пустой массив этой длины
```
Array(5); // [empty x 5]
```
А если передать в конструктор больше аргумента - будет создан массив из этих элементов:
```
Array(1, 2); // [1, 2]
```
Массив с пустыми слотами называется разреженным.

:exclamation: Пустая ячейка не содержит ничего, даже undefined. Использование delete на массиве удаляет значения, но длина не изменится (ячейка станет `empty`);
Поэтому "пустой" массив не итерируется:
```
Array(5).forEach(console.log) // никакого вывода
```
### Массивоподобные
Например: HTMLCollection, arguments

## Строки
Строки похожи на массивы, но не являются ими. Иммутабельны (поэтому нельзя вызвать метод массива `reverse` через `call`).

Некоторые (`reverse`) методы массива не могут работать с unicode-символами:
```
'foo 𝌆 bar'.split('').reverse().join(''); // "rab �� oof"
```

## Числа
Этот тип включает в себя целый и десятичные числа.
Для числовых значений используется стандарт, основанный на `IEEE 754`, использует `dobule-precision (64-bit)` формат стандарта.

Стандарт позволяет представлять числа в диапазоне `-(2^53 - 1) до 2^53 - 1`.

В ES-6 эти значения доступны в `Number.MIN_SAFE_INTEGER` и `Num ber.MAX_SAFE_INTEGER`.

Для проверки целого числа на безопасность использования `Number.isSafeInteger`, в безопасные числа входят все целые числа в диапазоне `от -(2^53 - 1) до 2^53 - 1` включительно:
```
Number.isSafeInteger(Number.MIN_SAFE_INTEGER) // true
Number.isSafeInteger(Number.MIN_SAFE_INTEGER - 1) // false
Number.isSafeInteger(Number.MAX_SAFE_INTEGER) // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1) // false
```

Десятичная часть числа отделяется с помощью `.`, 0 стоит по умолчанию:
```
const a = .42; // 0.42
const b = 42.; // 42, десятичная часть отброшена, т.к равняется нулю
1.  === 42.0 // true
```

Очень большие и очень маленькие числа по умолчанию выводятся в экспонентной (`toExponential()`) форме:
```
const a = 100 ** 100; // 1e+200
typeof a // 'number'
```

Поскольку числовой литерал может кончаться `.`, то это может привести к ошибке при вызове методов:
```
42.toFixed(1); // SyntaxError: Invalid or unexpected token
```
После первой `.` ожидается не вызов метода, а десятичная часть числа. Вызвать метод можно так:
```
42 .toFixed(3) // "42.000" перед вызовом метода стоит пробел
42..toFixed(3); // "42.000" <- это строка
(42).toFixed(3) // "42.000"
```

Сравнение маленьких чисел затруднено из-за `IEEE-74` с выделением 64 бита памяти для любых чисел:
```
0.1 + 0.2 === 0.3; // false
0.1.toFixed(30) // "0.100000000000000005551115123126"
0.2.toFixed(30) // "0.200000000000000011102230246252"

(0.2 + 0.1).toFixed(30) // "0.300000000000000044408920985006" <- Не 0.3
```

В ES-6 введен `Number.EPSILON` - разница между еденицей и наименьшим числом, большим единицы.

`Number.EPSILON // 2.220446049250313e-16 или 2^-52`;

Его можно использовать для сравнения значений маленьких чисел:
```
const a = 0.1;
const b = 0.2;
const result = 0.3;

a + b === result // false
Math.abs(a + b) // 0.30000000000000004
Math.abs(result - b - a).toFixed(30) // "0.000000000000000027755575615629"

Math.abs(result - b - a) < Number.EPSILON // true
```

В JS также существую 32-битные числа, приведение к которым возможно следующим образом:
`number | 0`
Это побочный результат работы с побитовыми операторами, которые работают только с целыми 32-битными числами.

## Особые значения
### Значения без значения
Тип `undefined` в качестве значения имеет только и только `undefined`
Тип `null` содержит в себе только и только `null`

Их название показывает одновременно тип и значение.

### Undefined
В старых версиях (вне `strict`) `undefined` можно было переназначить на свое значение.
При этом даже в `use strict` можно объявить локальную переменную с именем `undefined`;

#### void operator
Результатом любого выражения с `void` будет `undefined`, существующее значение переменной не изменяется, просто из выражения возвращается `undefined`
```
const a = 42;
console.log(void a, a); // undefined 42
```

По соглашению для получения значения `undefined` с помощью `void` используется выражение `void 0`.
:memo: Нет практической разницы между `void 0`, `void 1` и `undefined`:
```
void 0 // undefined
void 1 // undefined
undefined // undefined
void true === undefined // true
```

Оператор `void` может быть полезен в определенных обстоятельствах, когда нужно убедиться, что выражение не имеет результирующего значения (даже при наличии сайд-эффектов), этого же можно добиться обычным `return` без значения.
Эти две функции выполняются одинаково выполняется одинаково:
```
function someFuncWithVoid() {
  ...
  return void doSomeAction(); // вернет undefined, даже если doSomeAction возвращает значение
};

function someFuncWithoutVoid() {
  ...
  doSomeAction();
  return; // вернет undefined
}
```

### Особые числа
Числовой тип включает в себя несколько специальных значений

#### Не числовые числа
Любая математическая операция, где оба операнда не являются числами (или не могут быть приведены к десятичным/шестнадцатеричным числам) производят не валидное число, а `NaN` - `Not a Number`.

При этом тип `NaN` - `number`
```
const a = 42 / 'pew'; // NaN
typeof a; // "number"
a === a; // false
```

`NaN` единственное значение в JS, не равное самому себе, что позволяет понять, является ли значение NaN,
 поскольку глобальный метод `isNaN` срабатывает и на строки (что логично, ведь они не подходят под определение not a number):
```
const str = 'string';
const realNaN = str / 42;

isNaN(str); // true
isNaN(realNaN); // true

function checkIsNaN(value) {
  return value !== value;
}

checkIsNaN(str); // false
checkIsNaN(realNaN) // true;
```
Такого же результата можно добиться с помощью метода Number.isNaN
```
Number.isNaN(str); // false
Number.isNaN(realNaN) // true;
```

#### Бесконечности (Infinity)
Бывает положительная `Infinity`,

`Number.POSITIVE_INFINITY`

и отрицательная `-Infinity`,

`Number.NEGATIVE_INFINITY`

При обработке больших чисел результат обрабатывается следующим образом:
Если результат больше `Number.MAX_VALUE` но ближе к нему, чем к `Number.POSITIVE_INFINITY`, то число округляется в меньшую сторону
до `Number.MAX_VALUE`, иначе округляется в большую сторону до `Number.POSITIVE_INFINITY`:
```
const a = Number.MAX_VALUE; // 1.7976931348623157e+308
a + a; // Infinity
a + Math.pow(2, 970); // Infinity
a + Math.pow(2, 969); // 1.7976931348623157e+308
a + Math.pow(2, 969) === Number.MAX_VALUE // true
```
Если вы превратили число в Infinity, то превратить его обратно не получится:
```
const a = Number.MAX_VALUE; // 1.7976931348623157e+308
const bigNumber = Math.pow(2, 970); // 9.9792015476736e+291

a + bigNumber; // Infinity
a + bigNumber - bigNumber // Infinity
```

Деление `Infinity` на `-Infinity` даст `NaN`. Деление положительного числа на `Infinity` даст `0`.

#### Нули
В JS есть положительные `0` и отрицательные `-0` нули.
Отрицательный ноль может появляться только от определенных математических операций
```
0 / -3 // -0
0 * -3 // -0
```
Сложение и вычитание не приводят к `-0`

При приведении к строке `-0` превращается в `"0"`.

Сравнение происходит следующим образом
```
-0 == 0 //true
-0 === 0 //true
0 > - 0 // false
```

### Особая эквивалентность (equality)
В ES-6 введен новый метод `Object.is`, который имеет отличное от `==` и `===` сравнение. Пример из [статьи на MDN](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Object/is):
```
Object.is('foo', 'foo');     // true
Object.is(window, window);   // true

Object.is('foo', 'bar');     // false
Object.is([], []);           // false

var test = { a: 1 };
Object.is(test, test);       // true

Object.is(null, null);       // true

// Специальные случаи
Object.is(0, -0);            // false
Object.is(-0, -0);           // true
Object.is(NaN, 0/0);         // true
```

## Значения против Ссылок
Ссылки в JS указывают на значение, а не переменную.

Простые (примитивы: `null, undefined, string, number, boolean, symbol`) значения при присвоении в другую переменную всегда копируются, исходное значение не изменяется.

Сложные - объекты (включая массивы, обертки (boxed objects) и фукнкции) - всегда создают копию ссылки, а не значения, даже при передаче в качестве аргумента функции.
```
function mutate(array) {
	array.push(4, 5);
  
  return array;
}

const arr = [1, 2, 3];
mutate(arr);

arr; // содержимое массива изменилось с [1, 2, 3] на [1, 2, 3, 4, 5]
```

Даже если воспользоваться объектом-оберткой для приведения числа к объекту - обрабатываться он будет все равно, как число-примитив:
```
function mutate(value) {
  value = value + 1; // перезаписывается входящее значение
  return value;
}

const a = 1;
const b = new Number(a); // 1
const c = a; // 1

typeof b // 'object'

mutate(a); // 1
console.log(a); // 1

mutate(b); // 1
console.log(b); // 1

mutate(c); // 1
console.log(c); // 1
```

Происходит это потому, что примитивы неизменяемы. При работе с оберткой Number из нее извлекается примитив для проведения каких-либо операций.

Нативы
======

Являются встроенными функциями, список:
* String()
* Number()
* Boolean()
* Array()
* Object()
* Function()
* RegExp()
* Date()
* Error()
* Symbol()

С помощью них можно создавать объекты-обертки над значениями:
```
  const a = new String( "abc" );
  typeof a; // 'object'
  a instanceof String; // true
```

## Обертки
Для вызова методов у примитивов необходимо обернуть (box/wrap) примитив в объект, JS делает это автоматически:
```
  const a = "abc";
  a.length; // 3
  a.toUpperCase(); // "ABC"
```
Не нужно самостоятельно оборачивать примитивы, т.к движки браузеров оптимизированы для использования примитивов.
Код станет медленее, если вы будете создавать обертки вручную.

Для получения (unboxing) обернутого значения используется `valueOf()`
```
  const a = new String( "abc" );
  const b = new Number( 42 );
  const c = new Boolean( true );
  a.valueOf(); // "abc"
  b.valueOf(); // 42
  c.valueOf(); // true
```
Также анбоксинг используется не явно, в случае необходимости
```
  const a = new String( "abc" );
  'string: ' + a // 'string abc'
```


## Подводные камни объектов-оберток
```
  const a = new Boolean(false);
    if (!a) {
      console.log('Pew'); // никогда не исполнится 
  }
```
Условие никогда не выполнится, поскольку a - это объект (truthy), пусть и с заключенным внутри false, а `!truthy === false`.

Вместо `new String()` `new Number`, etc. можно использовать `Object` без приписки `new`.

## Конструкторы
Создавать значения можно с помощью литеральной формы:
```
  const a = 1;
  const b = 'string';
  const c = true;
  const d = [];
  const e = {};
```
Либо воспользоваться встроенными конструкторами:
```
  const a = new Number(1);
  const e = new Array();
  и проч
```

Следует избегать создания через конструкторы из-за наличия различных исключений и подводных камней. [Например при создании массива](#Проблемы-создания-массива-через-конструктор)

### Конструкторы Function и RegExp
Конструктор Function позволяет динамически задавать параметры/тело функции. Скорее всего, это вам не понадобится.

Конструктор RegExp следует избегать и создавать регулярные выражения через литеральную форму - в таком случае движок JS прекомпилирует и кеширует их до выполнения кода.

### Date и Error
Отсчет времени ведется от 1 января 1970 года 00:00:00 по UTC.

Вызов `new Date` принимает параметры для создания даты, если они не переданы - по умолчанию будет взято текущее время. Вернет объект Date, который имеет встроенные методы.

Вызов `Date` без `new` вернет строковое представление объекта `Date`.

`Error` можно создавать как с, так и без `new`. При создании записывает путь возникновения ошибки в предназначенное только для чтения свойство `.stack`;
Есть несколько специфичных ошибок для разных типов:
```
EvalError(..)
RangeError(..)
ReferenceError(..)
SyntaxError(..)
TypeError(..)
URIError(..)
```
Они автоматически вызываются программой в случае ошибки, например `ReferenceError` при использовании необъявленной переменной.

### Symbol
Symbol - специальные уникальные значения, которые можно использовать в качестве ключей для объекта, не опасаясь конфликтов.
Символы могут быть использованы в качестве свойств объекта, но их значения нельзя увидеть изнутри программы или из инструментов разработки.
Символы не являются объектами.
Попытка создать символ через конструктор с `new` приведет к ошибке:
```
new Symbol() // TypeError: Symbol is not a constructor
```

## Прототипы
Каждый встроенный конструктор имеет свой прототип. Это объект, который описывает поведение конкретного типа. Все используемые встроенные методы у типов берутся из прототипа. По соглашению в документации прототип сокращается до `#`:
```
String.prototype.XYZ пишется как String#XYZ
String.prototype.indexOf - String#indexOf
String#charAt
String#toUpperCase
```
Некоторые прототипы не являются объектами:
```
typeof Function.prototype // 'function'
Array.isArray(Array.prototype) // true
RegExp.prototype.toString(); // "/(?:)/" -- пустой regex
```

### Использование прототипов по умолчанию
У функций, массивов и регулярных выражений прототип является тем же типом, что и значение, поэтому можно (но нужно ли?) использовать их в качестве значений по умолчанию для избежания ошибок. :exclamation: не стоит так делать, это пример:
```
function isThisCool(vals, fn, rx) {
    vals = vals || Array.prototype; // заменит undefined на пустой массив. При его изменении будет изменяться массив в прототипе, поскольку это ссылка
    fn = fn || Function.prototype; // заменит undefined на пустую функцию
    rx = rx || RegExp.prototype; // заменит undefined на пустой регексп
    return rx.test(
        vals
          .map(fn)
          .join('')
    ); // не будет ошибок, подобной "cannot read property map of undefined"
}
```
Подобную запись можно сократить (и обезопасить) с помощью задания дефолтных значений для аргументов (в ES6):
```
function isThisCool(vals = [], fn = () => {}, rx = /(?:)/) {
  return rx.test(
      vals
        .map(fn)
        .join('')
  );
}

```