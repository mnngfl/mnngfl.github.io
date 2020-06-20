---
layout: post
title: "[ES6] Generator와 Generator 함수"
categories: [javascript]
comments: true
---

ES6의 표준으로 등장한 제너레이터와 이를 반환하는 제너레이터 함수에 대해 알아 본다.\\
제너레이터는 반복 프로토콜을 따르는 객체로, 일련의 반복 행위를 하도록 정의된다.

<!--more-->

여기서 말하는 "반복 행위"이란 일련의 값을 만들고, 모든 값들을 나타낼 때 까지 잠재적으로 값을 반환하는 행위를 정의하는 방법이라고 풀어서 설명할 수 있다.\\
반복 가능한 객체의 예시로는 `String`, `Array`, `Map`, `Set` 등의 자료형이 있다.

지금부터 이러한 제너레이터의 특징과 이를 생성하고 사용하는 방법에 대해 알아 본다.

<br>

## 살펴보기

제너레이터는 제너레이터 함수에 의해 반환되는 객체이다. 가장 먼저 간단한 코드와 함께 각각 어떠한 기능을 하는 지 간략히 알아보자.

{% highlight js %}
// 제너레이터 함수
function* myGen() {
  yield 1;
  yield 2;
  yield 3;
}

// 제너레이터 객체
const genObj = myGen();
console.log(myGen.prototype); // Generator {}
{% endhighlight %}

<br>

#### 제너레이터 함수

위 코드 블록의 상단 부분에서 myGen 이라는 제너레이터 함수를 선언하였다. 일반 함수를 선언하는 것처럼 `function` 키워드 다음에 바로 `*` 키워드를 사용해 함수를 선언하면 제너레이터 함수가 만들어진다.\\
함수 내부를 보면 `yield`라는 생소한 키워드가 등장한다. 이 키워드는 간단하게 제너레이터 함수로부터 반환되는 단일 값이라고 생각하면 된다.\\
일반 함수에서는 `return`을 통해 값을 반환받게 되지만, 제너레이터 함수는 `yield` 키워드를 통해 비슷한 역할을 하게 된다.\\
단, `yield`는 값을 반환시키기도 하지만 그와 동시에 함수 실행의 중단점을 의미하기도 한다는 걸 알아 두자.

예제에서는 `yield`를 각각 1, 2, 3이라는 값과 함께 사용했다. 처음 이 함수를 실행한다면, 1이라는 값을 반환하고 `yield 1;` 지점에서 중단될 것이다. 다시 돌아가서, 두 번째로 함수를 실행하면 이번엔 2라는 값이 반환되고 함수는 `yield 2;` 지점에서 중단된다. 세 번째 실행도 마찬가지이다.\\
_설명을 위해 '함수를 실행한다'라고 표현했지만, 실제로는 제너레이터 객체를 만든 뒤에 이 객체의 메서드를 이용해야 한다._

정리하면, 제너레이터 함수는 실행을 중단하거나 재개할 수 있게 도움을 주는 함수이다. 실행 중에 빠져나갔다가 나중에 다시 돌아올 수 있다.\\
호출될 때 마다 실행 지점을 기억하고 있다가, 다음 호출 시 이전의 실행이 완료된 지점부터 작업을 수행할 수 있도록 제어가 가능해진다.

<br>

#### 제너레이터 객체

하단 부분에서는 genObj라는 이름의 제너레이터 객체를 선언했다. 제너레이터 함수를 호출하기 위해선 이렇게 제너레이터 객체를 먼저 선언해 주어야 한다.\\
이때 함수에서 `return`값을 내보내거나 콘솔을 찍었다고 해도, 제너레이터 객체 자체는 함수 내부의 코드를 실행하지 않는다.

{% highlight js %}
function* testFunc() {
  console.log("hello generator!");
  yield 10;
  return "bye generator!";
}

const testObj = testFunc(); //중단된 상태, 아직 제너레이터 함수를 실행한 게 아니다!
console.log(testObj); // {<suspended>}
{% endhighlight %}

아직 제너레이터 객체를 만든 것에 불과하므로, 본격적으로 함수 실행을 제어하기 위해서는 제너레이터 객체의 메서드를 활용해야 한다.\\
해당 메서드의 목록은 다음과 같다.

- **next()**: 주로 사용하게 될 메서드로, `yield` 표현식으로부터 산출된 값을 반환한다.
- **return()**: 인수로 주어진 값을 반환하고 제너레이터를 완료시킨다.
- **throw()**: 에러를 발생시켜 제너레이터의 실행을 재개하고, 결과값을 갖는 객체를 반환한다.

이 메서드 중 하나를 예로 들어보자. 아까 선언한 genObj라는 제너레이터 객체에서 `next` 메서드를 3번 호출하게 되면 예상했던대로 1, 2, 3이라는 단일 값들을 차례로 받아볼 수 있어야 한다. 이 때의 결과는 다음과 같다.

{% highlight js %}
console.log(genObj.next()); // { value: 1, done: false }
console.log(genObj.next()); // { value: 2, done: false }
console.log(genObj.next()); // { value: 3, done: false }
console.log(genObj.next()); // { value: undefined, done: true }
{% endhighlight %}

제너레이터 객체의 메서드 호출로 얻어지는 반환 값은 공통적으로 `value`, `done` 두 프로퍼티를 갖는 객체이다. `{ value: VALUE, done: true|false }` 와 같은 형태로 생긴 이 객체는 **iterator result** 라는 이름으로도 불리운다.\\
이들 메서드가 호출될 때마다 제너레이터 함수는 객체에 `value` 프로퍼티에 배열 요소에서 차례대로 꺼내진 값과 `done` 프로퍼티에 요소의 열거가 끝났는지의 여부를 저장하여 반환하고, 내부 처리 상태를 해당 `yield` 표현식까지로 일시 정지 시켜 놓는다. 그리고 계속해서 메서드를 통해 제너레이터를 재개시키거나 종료시킬 수 있는 것이다.

마지막으로 더 이상 실행할 `yield` 표현식이 존재하지 않는 상태에서 `next()`를 호출한다면, `{ value: undefined, done: true }`라는 결과가 반환된다. 이는 제너레이터가 종료되었다는 의미이다.

<br>

## 제너레이터 함수에서도 return을 사용할 수 있을까?

가능하다. 하지만, `yield`로 반환되는 값처럼 `value`, `done` 두 속성을 가지는 객체가 반환된다.\\
또한 리턴문에 다다르면 제너레이터의 상태는 자동으로 종료 상태로 변경된다.

{% highlight js %}
function* generatorWithReturnStatement() {
  yield 100;
  yield 200;
  return 300; // 이 아래의 내용은 무시됨
  yield 400;
}

const generator = generatorWithReturnStatement();
console.log(generator.next()); // { value: 100, done: false }
console.log(generator.next()); // { value: 200, done: false }
console.log(generator.next()); // { value: 300, done: true }
console.log(generator.next()); // { value: undefined, done: true }
{% endhighlight %}

<br>

## Iterator 이면서 Iterable 하다?

제너레이터 객체는 이터레이터와 이터레이블 프로토콜이 함께 구현되어 있다. 이 두 개념에 대한 설명은 [MDN - Iteration Protocol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#The_iterator_protocol){:target="\_blank"}에 설명되어 있다.\\
해당 글의 내용을 요약하면, 다음과 같다.

- Iterable protocol: 객체의 값들이 반복하며 특정 동작을 처리하도록 반복 동작을 정의하는 프로토콜. `for...of` 구문에서 모든 값의 순회가 가능하다.
- Iterator protocol: 모든 값에 다다를 때 까지 일련의 값이나 잠재적으로 반환할 값을 생성하는 것을 정의하는 프로토콜. `value`와 `done` 두 프로퍼티를 갖는 객체를 반환하는 `next()` 메서드를 구현한다.

이러한 사실을 통해 제너레이터 함수에서도 `for...of` 루프를 이용해 간편하게 값을 얻을 수 있게 된다.\\
아래 예제는 제너레이터 함수를 순회하며 `yield`문을 만날 때마다 값을 반환하도록 한다.

{% highlight js %}
function* myGen() {
  yield 1;
  yield 2;
  yield 3;
}

for (const n of myGen()) {
  console.log(n); // 1 2 3
}
{% endhighlight %}

<br>

## 제너레이터 함수에서 변수와 인자를 사용해보자

제너레이터 함수는 변수를 사용하거나 인자를 사용하여 더 강력한 힘을 발휘하도록 할 수 있다.\\
예를 들어, 호출할 때 마다 제곱수를 반환하는 함수를 만들고 싶다면 다음과 같이 작성한다.\\
이를 `next().value`로 호출하게 되면 1, 4, 9, 16, 25...와 같이 증가하는 패턴을 볼 수 있다.

{% highlight js %}
function* squaredNumber() {
  let n = 0;

  while (true) {
    n++;
    yield n * n;
  }
}

const squaredNumbers = squaredNumber();
console.log(squaredNumbers.next().value); // 1
console.log(squaredNumbers.next().value); // 4
console.log(squaredNumbers.next().value); // 9
console.log(squaredNumbers.next().value); // 16
{% endhighlight %}

제너레이터 함수를 다양한 방식으로도 사용해보자. 위 예제는 `while (true)`문에 의해 호출되는 만큼 걊이 무한으로 증가할 것이다.\\
제너레이터 함수에 인자를 전달하여 반복 횟수를 조절할 수 있다. 아래와 같이 작성하면 반복문이 넘겨진 인자의 수 만큼만 반복이 될 것이다.

{% highlight js %}
function* squaredNumber(max) {
  let n = 0;

  while (n < max) {
    n++;
    yield n * n;
  }
}

for (const n of sqaredNumber(10)) {
  console.log(n); // 1, 4, 9, 16, 25, 36, 49, 64, 81, 100
}
{% endhighlight %}

<br>

## next() 메서드에 인수 전달하기

`next()` 메서드에 인수를 전달하면 어떤 일이 발생할 지와 `yield` 키워드에 대해 좀 더 알아보자.\\
지금까지의 예제에서는 단순히 `yield [표현식]`의 구문으로 사용하였는데, `yield`는 원래 다음과 같은 형태로 사용된다. (대괄호는 생략 가능하다는 의미)

{% highlight js %}
[returnValue] = yield [expression]
{% endhighlight %}

이 때, `expression`은 이터레이터 프로토콜을 통해 제너레이터 함수로부터 반환할 값을 정의한다. (생략하면 `undefined`가 반환)\\
`returnValue`는 `next()` 메서드에 전달되는 값을 검색하여 함수의 실행을 재개한다. 이 값은 선택 사항이다.

<br>

#### next(value)의 사용

`next(value)`의 형태는 제너레이터로 값을 전달하겠다는 의미이다. 전달된 값은 `yield [expression]`의 반환값으로서 할당된다. 즉, 제너레이터가 일시적으로 정지되기 직전의 `yield` 표현식의 값으로써 사용된다.\\
예를 들어 제너레이터 함수에서 아래와 같이 작성했다면, 전달된 값이 variable이라는 변수에 할당된다. 이를 통해 제너레이터의 내부 상태를 외부로부터 변경시킬 수 있게 된다.

{% highlight js %}
variable = yield expression
{% endhighlight %}

`expression`과 별도로, `yield`는 `returnValue`의 값으로써 `next()`에 전달된 값을 할당하게 된다.\\
하지만 `yield`는 이전 반복에서 실행을 재개할 때에만 전달된 값을 할당한다는 특징이 있다. 따라서 첫 번째 반복에서는 아무리 `next()`에 값을 전달해도 `returnValue`에 아무런 값을 지정하지 않는다.

<br>

#### 사용 예제

그럼 이제 실제로 `next()` 메서드를 인수와 함께 입력하여 제너레이터로 이 값을 전달해보자.

{% highlight js %}
function* gen() {
  console.log("before a");
  let a = yield 123;
  console.log("after a");
  yield a * 2;
  console.log("end of function");
}

const g = gen();
console.log(g.next());
// "before a"
// { value: 123, done: false }
console.log(g.next(4));
// "after a"
// { value: 8, done: false }
console.log(g.next());
// "end of function"
// { value: undefined, done: true }
{% endhighlight %}

- 첫 번째 `next()` 호출: `next()` 함수에 아무 값을 전달하지 않았다. 첫 번째 `yield` 문이 있는 곳까지 실행된다. `yield 123`에 의해 123이라는 반환 값을 얻었다.
- 두 번째 `next(4)` 호출: 인수로 4를 전달하였다. 이 값은 제너레이터 함수의 지역 변수 a로 할당된다. 따라서 다음 `yield`문이 있는 곳까지 코드가 실행되어 `yield a * 2`에 의해 8을 반환받게 된다.
- 세 번째 `next()` 호출: 더 이상 `yield`로부터 불러 올 값이 없으니 함수 반복의 종료를 알린다.

<br>

## Reference

[\[MDN\] Generator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator){:target="\_blank"}\\
[\[MDN\] yield](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield){:target="\_blank"}\\
[\[MDN\] Generator.prototype.next()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/next){:target="\_blank"}\\
[How to use Generator Functions in JavaScript - Tutorial](https://www.youtube.com/watch?v=EzdgkEMvrvA){:target="\_blank"}\\
[How does Generator.next() processes its parameter?](https://stackoverflow.com/questions/37354461/how-does-generator-next-processes-its-parameter){:target="\_blank"}\\
[ES6 Generators: First call to next()](https://stackoverflow.com/questions/39732697/es6-generators-first-call-to-next){:target="\_blank"}
