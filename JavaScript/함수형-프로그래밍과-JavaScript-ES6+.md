# 함수형 프로그래밍과 JavaScript ES6+

- [평가와 일급, 고차 함수](#평가와-일급-고차-함수)
- [이터러블(Iterable)과 이터레이터(Iterator)](#이터러블(Iterable)과-이터레이터(Iterator))
- [제너레이터(Generator)](#제너레이터(Generator))

## 평가와 일급, 고차 함수

### 평가

코드가 계산되어 값을 만드는 것이다.

```javascript
console.log(1 + 2); // 1 + 2는 3으로 평가된다
console.log(1 + 2 + 4); // 1 + 2는 3으로 평가되고, 3 + 4는 7로 평가된다
console.log([1, 2 + 3]); // 2 + 3은 5로 평가되고, 배열 요소로 1과 5를 가진 배열로 평가된다
```

### 일급

값으로 다룰 수 있으며, 변수에 담을 수 있다. 그리고 함수의 인자 또는 결과로 사용될 수 있다.

```javascript
const a = 10; // 10을 값으로 다루고, 변수 a에 담는다
const add10 = (a) => a + 10; // a + 10은 함수의 결과로 사용된다
const r = add10(a); // a는 add10 함수의 인자로 사용되고, 함수의 결과를 변수 r에 담는다
console.log(r); // 20
```

### 일급 함수

함수를 값으로 다룰 수 있다. 즉, 변수에 함수를 담을 수 있으며, 함수의 인자 또는 결과로 함수가 사용될 수 있다.

JavaScript에서 함수가 일급이라는 성질을 이용해서 많은 조합성을 만들어 낼 수 있으며, 추상화의 좋은 도구로 사용될 수 있다.

```javascript
const add5 = (a) => a + 5; // 함수를 변수 add5에 담는다
console.log(add5); // (a) => a + 5, 함수의 인자로 함수가 사용된다
console.log(add5(5)); // 10

const f1 = () => () => 1; // 함수의 결과로 함수가 사용된다
console.log(f1); // () => () => 1

const f2 = f1();
console.log(f2); // () => 1
console.log(f2()); // 1
```

### 고차 함수

함수를 값으로 다루는 함수이다. 고차 함수는 크게 2가지가 있다.

#### 함수를 인자로 받아서 실행하는 함수

```javascript
const apply1 = (f) => f(1);
const add2 = (a) => a + 2;
console.log(apply1(add2)); // 3, ((a) => a + 2)(1)
console.log(apply1((a) => a - 1)); // 0, ((a) => a - 1)(1)

const times = (f, n) => {
  let i = -1;
  while (++i < n) f(i);
};
times(console.log, 3); // 0 1 2
times((a) => console.log(a + 10), 3); // 10 11 12
```

#### 함수를 만들어 반환하는 함수

`addMaker()` 함수는 `(b) => a + b` 함수를 반환한다. 그리고 `addMaker()` 함수가 호출될 때, `(b) => a + b` 함수는 `a`를 기억하고 있다. 즉, `(b) => a + b` 함수는 클로저이며, `addMaker()` 함수는 클로저를 만들어 반환하는 함수이다.

> 클로저란?
> 
> 자신이 선언될 당시의 환경을 기억하는 함수이다.

```javascript
const addMaker = (a) => (b) => a + b;
const add10 = addMaker(10);
console.log(add10); // (b) => a + b
console.log(add10(5)); // 15
```

## 이터러블(Iterable)과 이터레이터(Iterator)

### 기존과 달라진 ES6에서의 리스트 순회

```javascript
const list = [1, 2, 3];

// ES5
for (var i = 0; i < list.length; i++) {
  console.log(list[i]); // 1 2 3
}

// ES6
for (const a of list) {
  console.log(a); // 1 2 3
}
```

### Array, Set, Map을 통해 알아보는 이터러블/이터레이터 프로토콜(Iterable/Iterator protocol)

ES6에서의 순회를 어떻게 추상화하였을까? `Array`, `Set`, `Map`을 통해 알아보자.

`Array`, `Set`, `Map`은 `for...of`를 통해 순회가 가능하다. 하지만 `Set`과 `Map`은 `Array`처럼 인덱스를 통해 값에 접근이 불가능하다. 그렇다면 이들은 어떻게 순회를 할 수 있었을까?

```javascript
const arr = [1, 2, 3];
for (const a of arr) console.log(a); // 1 2 3

const set = new Set([1, 2, 3]);
for (const a of arr) console.log(a); // 1 2 3

const map = new Map([
  ["a", 1],
  ["b", 2],
  ["c", 3],
]);
for (const a of map) console.log(a); // ["a", 1] ["b", 2] ["c", 3]

console.log(arr[0]); // 1
console.log(set[0]); // undefined
console.log(map[0]); // undefined
```

#### 이터러블/이터레이터 프로토콜(Iterable/Iterator protocol)

**이터러블(Iterable)**이란 이터레이터(Iterator)를 반환하는 `[Symbol.iterator]()` 함수를 가진 객체이다. `Array`, `Set`, `Map`은 `[Symbol.iterator]()` 함수를 가지고 있어 이터러블이라 할 수 있다.

```javascript
console.log(arr[Symbol.iterator]); // ƒ values() { [native code] }
console.log(set[Symbol.iterator]); // ƒ values() { [native code] }
console.log(map[Symbol.iterator]); // ƒ entries() { [native code] }
```

`[Symbol.iterator]()` 함수를 `null`로 할당하면 어떻게 될까? `TypeError`가 발생하며 순회할 수 없다. 즉, `Array`, `Set`, `Map`은 `[Symbol.iterator]()` 함수를 통해 순회하는 것을 알 수 있다.

```javascript
const arr = [1, 2, 3];
arr[Symbol.iterator] = null;
for (const a of arr) console.log(a); // TypeError: arr is not iterable
```

그렇다면 내부적으로 순회는 어떻게 동작했을까? 이는 이터레이터를 통해 내부에서 `next()` 함수를 호출하면서 값을 `value`로 접근하고, `done`이 `true`가 될 때까지 순회하고 있다. 즉, **이터레이터(Iterator)**는 `value`와 `done` 프로퍼티(Property)를 가진 객체를 반환하는 `next()` 함수를 가진 객체이다. 이와 같이 이터러블/이터레이터를 `for...of`, 전개 연산자 등과 함께 동작하도록 한 규약을 **이터러블/이터레이터 프로토콜(Iterable/Iterator protocol)**이라 한다.

```javascript
const arr = [1, 2, 3];
let iterator = arr[Symbol.iterator]();
console.log(iterator.next()); // {value: 1, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: 3, done: false}
console.log(iterator.next()); // {value: undefined, done: true}

let iterator1 = arr[Symbol.iterator]();
console.log(iterator1.next()); // {value: 1, done: false}
for (const a of iterator1) console.log(a); // 2 3

const map = new Map([
  ["a", 1],
  ["b", 2],
  ["c", 3],
]);
let mapIterator = map.keys(); // or values(), entries() 함수는 이터레이터를 반환한다
console.log(mapIterator); // MapIterator {'a', 'b', 'c'}

let mapIterator2 = mapIterator[Symbol.iterator](); // 자기 자신의 이터레이터를 반환한다
console.log(mapIterator2); // MapIterator {'a', 'b', 'c'}

console.log(mapIterator2.next()); // {value: 'a', done: false}
console.log(mapIterator2.next()); // {value: 'b', done: false}
console.log(mapIterator2.next()); // {value: 'c', done: false}
console.log(mapIterator2.next()); // {value: undefined, done: true}
```

### 사용자 정의 이터러블(Iterable)을 통해 알아보기

3부터 1씩 감소하면서 0이 되면 실행을 종료하는 이터러블을 정의한다.

```javascript
const iterable = {
  [Symbol.iterator]() {
    let i = 3;

    return {
      next() {
        return i == 0 ? { done: true } : { value: i--, done: false };
      },
    };
  },
};
let iterator = iterable[Symbol.iterator]();
console.log(iterator.next()); // {value: 3, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: 1, done: false}

for (const a of iterable) console.log(a); // 3 2 1
```

하지만 앞에서 살펴본 예제와 다르게 이터레이터는 `for...of`에서 사용할 수 없으며 `TypeError`가 발생한다.

```javascript
let iterator = iterable[Symbol.iterator]();
console.log(iterator.next()); // {value: 3, done: false}

for (const a of iterator) console.log(a); // TypeError: iterator is not iterable
```

`for...of`에서 이터레이터를 사용할 수 있고, 일부 진행했을 때의 이후로 순회가 가능하도록 하려면 이터레이터가 자기 자신의 이터레이터를 반환하는 `[Symbol.iterator]()` 함수를 가지고 있어야 한다. 이와 같이 구현된 이터러블을 **well-formed iterable**이라고 한다.

```javascript
const iterable = {
  [Symbol.iterator]() {
    let i = 3;

    return {
      next() {
        return i == 0 ? { done: true } : { value: i--, done: false };
      },
      [Symbol.iterator]() {
        return this;
      },
    };
  },
};

let iterator = iterable[Symbol.iterator]();
console.log(iterator === iterator[Symbol.iterator]()); // true

console.log(iterator.next()); // {value: 3, done: false}
for (const a of iterator) console.log(a); // 2 1
```

## 제너레이터(Generator)

### 제너레이터 함수(Generator function)

일반 함수는 하나의 값만을 반환하거나 반환하지 않을 수 있다. 하지만 **제너레이터 함수(Generator function)**는 여러 개의 값을 필요에 따라 하나씩 반환(`yield`)할 수 있다. `function*`로 정의할 수 있으며, 이는 실행을 처리하는 제너레이터(Generator)를 반환한다.

제너레이터의 `next()` 함수를 호출하면 `yield`를 만날 때까지 함수를 실행하여 `value`, `done` 프로퍼티(Property)를 가진 객체를 반환하고 함수의 실행을 멈춘다. 함수의 실행이 끝나면 `done`이 `true`가 된다.

```javascript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

let generator = gen();
console.log(generator); // Object [Generator] {}
console.log(generator.next()); // { value: 1, done: false }
console.log(generator.next()); // { value: 2, done: false }
console.log(generator.next()); // { value: 3, done: false }
console.log(generator.next()); // { value: undefined, done: true }
```

### 제너레이터(Generator)와 이터러블(Iterable)

**제너레이터(Generator)**는 이터레이터(Iterator)처럼 `next()` 함수를 호출할 수 있는 것을 보아 **이터러블(Iterable)**이다. 따라서 자기 자신의 이터레이터를 반환하는 `[Symbol.iterator]()` 함수를 가지고 있으므로 `for...of`, 전개 연산자, 구조 분해 할당을 사용할 수 있다. 이때, `done`이 `true`일 때의 `value`는 무시하는 것을 주의해야 한다.

이러한 성질을 이용하여 어떠한 상태나 값을 순회할 수 있는 형태로 만들 수 있다.

```javascript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  return 5;
}

let generator = gen();
console.log(generator === generator[Symbol.iterator]()); // true

for (const a of gen()) console.log(a); // 1 2 3 4

console.log([...gen()]); // [1, 2, 3, 4]

const [a, b, ...rest] = gen();
console.log(a); // 1
console.log(b); // 2
console.log(rest); // [3, 4]
```

### 예시를 통해 제너레이터(Generator) 알아보기

`i`부터 `n`까지의 홀수만을 반환하는 `odds()` 함수를 정의한다. 이는 `infinity()` 함수를 통해 `i`부터 1씩 증가하는 무한수열을 만들 수 있다. `next()` 함수를 호출해야 실행하기 때문에 무한 루프에 빠지지 않는다. 그리고 `infinity()` 함수로부터 반환된 이터러블을 `limit()` 함수에 전달하여 `n`까지 순회하고 실행을 종료할 수 있도록 한다.

이와 같이 여러 제너레이터 함수들을 조합하여 정의할 수 있다.

```javascript
function* odds(i, n) {
  for (const a of limit(infinity(i), n)) {
    if (a % 2) yield a;
  }
}

function* infinity(i = 1) {
  while (true) yield i++;
}

function* limit(iterable, n) {
  for (const a of iterable) {
    yield a;
    if (a == n) return;
  }
}

console.log([...odds(1, 10)]); // [1, 3, 5, 7, 9]
```