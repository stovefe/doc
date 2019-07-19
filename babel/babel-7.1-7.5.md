# Babel 7.1 ~ 7.5 추가된 문법 정리

[바벨(Babel)](https://babeljs.io/)의 메이저 버전이 7로 변경되었고 현재 7.5 버전까지 업데이트가 되었다.
이렇게 바벨 버전이 업데이트 되면서 버그 수정과 더불어 tc39 proposal 중 일부가 추가적으로 지원되고 있다.
아직은 스테이지 상태이기 때문에 확정은 아니지만 아래의 문법은 어떠한 형태로든 제공 할 것으로 보인다.

7.5 버전까지 추가 된 문법은 크게 클래스(class)관련 private property, method, static과 함수(function) 관련 partial application과 pipeline operator이다. 각각 어떤 문법인지 확인해보고 장단점을 확인해보자.

## Class Private Property, Method, Static (stage 3)

### Proposals

- Stage 3 - [Class field declarations for JavaScript](https://github.com/tc39/proposal-class-fields#private-fields)
- Stage 3 - [Private methods and getter/setters for JavaScript classes](https://github.com/tc39/proposal-private-methods)
- Stage 3 - [Static class features](https://github.com/tc39/proposal-static-class-features#static-private-methods)

### 설명

class private 관련 문법 proposal은 자바스크립트 class의 private property, method, static, getter/setter를 지원하는 것이다. 현재의 자바스크립트 class에는 private 개념이 없기 때문에 private을 구현하기 위해서는,

1. 코딩 컨벤션으로 의도를 드러냄(변수니 메서드에 prefix 또는 suffix에 "\_", "\$" 등을 추가해서 사용)
2. [module reveal pattern](https://shlrur.github.io/develog/2018/09/11/2-module/)을 이용
3. es6에서 새로 추가된 [`Symbol`, `WeakMap`, `WeakSet`](https://yookeun.github.io/javascript/2015/03/28/javascript-private/) 자료 형을 활용 하여 구현

하지만, 이런 구현 방식은 몇몇 단점이 있다.

1번 방법의 경우 코딩 컨벤션으로 의도를 드러낼 뿐 코드 상의 강제성을 띄울 수 없다.
그렇기 때문에 사용자가 의도적으로 값을 변경 한다고 해도 이를 방어하기가 어렵고, 값이 노출 되는 단점이 있다.
2,3 번의 방식의 단점은 코드의 가독성이 떨어지고, 코드가 정황해질 수 있으며 코드의 의도를 명확하게 드러내기 힘들다는 데 있다.

현재 논의되고 있는 class private 관련 문법은 이러한 한계를 네이티브 문법으로 제공함으로써 위에서 지적했던 부분을 극복하는데 있다.

먼저, 추가된 문법은 아래와 같다.

```js
class A {
  // private property
  #privateA = 'private A';

  // private getter
  get #a() {
    return this.#privateA;
  }

  // private setter
  set #a(value) {
    this.#privateA = value;
  }

  // private method
  #privateMethodA() {
    this.#a++;
  }

  // private static property
  static #privateStaticA = 'private static A';

  // prviate static method
  static #privateStaticMethodA() {
    return A.#privateStaticA;
  }
}
```

문법 자체는 간단하다. 프로퍼티나 메서드가 private이 되게 하려면 에 `#`을 붙여주면 된다. `#`이 붙은 프로퍼티나 메서드는 외부에서 접근이 불가능 하게 된다.

### 타입스크립트와의 비교

실제로 타입스크립트에서는 클래스의 `private`, `protected` 문법을 제공하지만, 이는 실제로 컴파일 단계에서의 에러는 잡아 내지만 런타임 에러를 막아주진 않는다([구현하지 못해서이기 때문은 아니다](https://github.com/Microsoft/TypeScript/issues/564#issuecomment-291754335)). 아래의 typescript 예제를 확인해보자.

### 예제

#### [babel](https://codesandbox.io/s/misty-bash-20hmc)

#### [typescript](https://codesandbox.io/s/elegant-mcclintock-gncw7)

#### [compiled](https://babeljs.io/repl#?babili=false&browsers=&build=&builtIns=false&spec=false&loose=false&code_lz=MYGwhgzhAEAKBOBLAbmALgUwMLitA3gFDTQDEADvAPbkCC0AvNAESUrobSU3S3MDcxLtXIAhRi3IBXAEYhEwYT1EDCQ0gFsMaABZUAJrQAUASgJCS8bVPgA7aLsQQAdBRG1BJAL5qSW3QaipuYkltZ2DjpOztxintA-Qv56-ljBRKHQVmg29o4ubjQeQj6JwFS2EGhciky2GADucEiomDiQEKaChOWVVCAYziBUAOZG5MDOyYGmADQ1JoJAA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=stage-1&prettier=false&targets=&version=7.5.4&externalPlugins=)

### 참고

실제로 바벨은 이 private syntax(#)을 지원하기 위해 컴파일시 `WeakSet`과 `WeakMap`을 이용하여 구현하고 있다. 위의 compiled 예제를 확인해보면 이것을 확인 할 수 있다.

## Partial Application (stage 1)

[Partial Application Syntax for ECMAScript](https://github.com/tc39/proposal-partial-application)

### 설명

[Partial Application](https://blog.rhostem.com/posts/2017-04-20-curry-and-partial-application#partial-application)은 보통 함수형 프로그래밍에서 사용하는 기법으로 함수의 인자 중 일부분을 미리 적용한 함수를 리턴하는 함수이다. 자바스크립트의 [bind](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) 메서드나 미리 적용된 함수를 리턴하는 함수를 만들어 partial application을 표한 할 수 있지만 몇가지 단점이 있다.

먼저, 아래의 예제를 보자.

```js
function add(x, y) {
  return x + y;
}

// 1. Function#bind
const addOne = add.bind(null, 1);
addOne(2); // 3

// 2. arrow functions
const addTen = x => add(x, 10);
addTen(2); // 12
```

위의 예제 중 1번은 `Function.prototype.bind` 메서드를 사용하여 partial application을 구현한 예제이다.
2번 예제는 arrow function을 이용해 partial application을 구현한 예제이다.

이 두가지 방법은 몇가지 단점이 있다.

- `bind` 메서드의 경우 항상 첫 번째 parameter로 `this` 값을 전달해야 한다.
- `bind` 메서드의 경우 순차적으로만 인자가 전달된다.
- `bind` 메서드의 경우 템플릿 리터럴과 같이 사용하기 어렵다.
- arrow function의 경우 코드가 길어지고 장황해지는 경향이 있다.

하지만, partial application 문법을 이용하면 좀더 명확히 의도를 드러내고, 유연하게 함수를 표현 할 수 있다.

partial application 문법은 `?` 토큰을 함수 인자에 포함하여 표현한다. 위의 예제였던 `add` 함수를 partial application을 이용하여 작성하면 아래와 같다.

```js
function add(x, y) {
  return x + y;
}

const addTen = add(?, 10);
```

기본적으로 partial application 문법은 함수 인자에만 사용 하도록 되어 있기 때문에 몇가지 제약사항이 있다. 아래 예제를 확인해보자.

```js
// valid
f(x, ?)           // partial application from left
f(?, x)           // partial application from right
f(?, x, ?)        // partial application for any arg
o.f(x, ?)         // partial application from left
o.f(?, x)         // partial application from right
o.f(?, x, ?)      // partial application for any arg
super.f(?)        // partial application allowed for call on |SuperProperty|

// invalid
f(x + ?)          // `?` not in top-level Arguments of call
x + ?             // `?` not in top-level Arguments of call
?.f()             // `?` not in top-level Arguments of call
new f(?)          // `?` not supported in `new`
super(?)          // `?` not supported in |SuperCall|
```

위의 예제에서도 보이지만 partial application proposal은 기본적으로 함수의 인자로 사용 될 때에만 사용 가능하다는 점에 유의하자.

### Pipeline Operator

[The Pipeline Operator](https://github.com/tc39/proposal-pipeline-operator)

Pipeline Operator는 F#, OCaml, Elixir, Elm, Julia, Hack과 LiveScript 언어에서 지원하는 문법과 비슷한데, 함수형 프로그래밍 언어에서 유연하게 함수 합성을 해주는 문법이다.

기존에는 중첩된 함수를 사용, 라이브러리를 사용하거나 `compose`, `pipe` 등의 함수를 만들어 합성하는 방법을 사용 했다.

- [lodash flow](https://lodash.com/docs/4.17.14#flow)
- [ramda pipe](https://ramdajs.com/docs/#pipe)
- [ramda compose](https://ramdajs.com/docs/#compose)

```js
function square(n) {
  return n * n;
}
function add10(a) {
  return a + 10;
}

// 중첩 함수
square(add10(10));

// loadsh
_.flow([add10, square])(10);
// => 9

// ramda
R.pipe([add10, square])(10);

R.compose(
  square,
  add
)(1, 2);

// custom compose function
const compose = (...fns) => fns.reduce((f, g) => (...args) => f(g(...args)));

compose(
  square,
  add10
)(1, 2);
```

Pipeline operator는 `|>`을 기반으로 한다. 먼저 위의 예제를 pipeline operator로 변경하면 아래와 같다.

```js
function square(n) {
  return n * n;
}
function add10(a) {
  return a + 10;
}

10 |> add10 |> square;
```

이 pipeline operator는 위 함수의 문법 설탕(syntax sugar)으로 생각해도 좋다. `|>` 키워드를 제외하고 구현에 대한 구체적인 방법은 확정되지 않았지만 제안되고 있는 방식은 5가지가 있다. 그중에서 지금 바벨 플러그인은 3가지 옵션을 제공한다.

먼저 예제를 확인해보자.

#### Minimal

가장 기본적인 형태의 문법을 제공한다. pipeline operator에서는 point free 스타일의 함수만 사용 할 수 있다.

```js
let result = 'hello' |> doubleSay |> capitalize |> exclaim;
```

#### F# pipeline

F# 스타일의 문법을 제공한다. 여기에서는 point free 스타일과 arrow 함수를 사용할 수 있다.
이 제안에서는 함수 파라미터를 여러개 사용하거나 순서를 변경 할 수 있다.

```js
let newScore = person.score
  |> double
  |> n => add(7, n)
  |> n => boundScore(0, 100, n);
```

#### Smart pipeline(7.3.0)

이 제안은 `#` 토큰을 사용하여 함수에 파라미터를 설정 할 수 있다.

```js
let newScore = person.score
  |> double
  |> add(7, #)
  |> boundScore(0, 100, #);
```

#### Partial(7.4.0)

또한, partial application proposal과 같이 사용 할 수도 있다.

```js
let newScore = person.score
  |> double
  |> add(7, ?)
  |> boundScore(0, 100, ?);
```
