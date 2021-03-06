---
layout: post
title: "RegExp 객체와 함께 템플릿 리터럴 사용하기"
categories: [javascript]
comments: true
---

일반적으로 문자열을 처리할 때 특정한 패턴과 일치하는 지 판별하고자 한다면 `정규 표현식`을 사용한다. 자바스크립트는 이러한 정규 표현식을 표현하는 `RegExp` 내장 객체를 가지고 있는데, 이 포스팅에서는 정규식 객체를 생성하는 방법과 정규식을 사용하면서 겪었던 문제점에 대한 경험을 기록한다.

<!--more-->

### 정규 표현식의 생성

자바스크립트에서 정규 표현식(이하 정규식)을 이용하려면 우선 `RegExp` 객체를 생성해야 한다. 이를 위한 방법으로는 두 방법이 존재하는데 바로 `RegExp 생성자` 혹은 `정규 표현식 리터럴` 방식이다. 다음 표현식은 모두 동일한 정규식을 생성하고 있다.

```js
/ab+c/i; // 리터럴
new RegExp("ab+c", i); // 생성자
new RegExp(/ab+c/, i);
```

각 표현식에 대해 설명을 덧붙이자면, `리터럴` 표기법에서는 매개변수를 빗금(슬래시)으로 감싼다는 점과 `생성자` 표기법에서는 매개변수를 따옴표로 감싸 표현한다는 점에서 차이점이 존재했다. 하지만 ES5부터는 RegExp 생성자의 매개변수로 리터럴을 사용할 수 있게 되었다. 그래서 위의 코드에서도 `new RegExp(/ab+c/, i)`와 같이 작성할 수 있었던 것이다.

<br>

### 정규 표현식과 컴파일

어떤 때에 리터럴 방식 혹은 생성자 방식을 사용해야 할까? 이를 판단하기 위해 정규식이 어떨 때 컴파일되는지에 대해 알아 본다.

- **리터럴 표기법**\\
  스크립트가 로드될 때 정규식의 컴파일이 일어난다. 만약 정규식이 계속 동일하게 유지되는 경우 이를 이용하면 성능 향상에 도움이 될 수 있다.

- **생성자 표기법**\\
  런타임에 컴파일이 일어난다. 만약 정규식 패턴이 변경되는 상황이 존재하거나 사용자 입력과 같은 외부에서 소스를 가져 오는 경우 이를 이용하도록 한다.

<br>

### 정규 표현식과 템플릿 리터럴

ES6부터 추가된 문자열 표현 구문인 `템플릿 리터럴`은 백틱(\`) 기호로 묶은 문자열이다.

정규식 리터럴과 템플릿 리터럴을 함께 `replace()` 메소드의 매개변수로 전달하여 코드를 실행하였는데, 원하는 대로 작동하지 않았다.

```js
// 원하는 결과: reg 정규식과 불일치하는 단어를 str에서 제거
const reg = "CBD";
const str = "BACDE";
str.replace(/`[^${reg}]`/g, ""); // "BACDE"
```

이를 원하는 대로 작동시키기 위해서는, 정규식 리터럴 구문 대신 정규식 생성자를 이용하도록 수정하여야 한다. 아까 언급하였듯 생성자 구문에서는 문자열과 리터럴 모두 사용하는 것이 가능하다. 따라서 문자열에 해당하는 템플릿 리터럴을 이용하기 위해선 아래와 같이 코드를 수정하면 된다.

```js
const reg = "CBD";
const str = "BACDE";
str.replace(new RegExp(`[^${reg}]`, "g"), ""); // "BCD"
```

<br>

> **마치며**\\
> 이외에도 정규식 패턴으로는 정말 다양한 상황에서 사용될 수 있는 많은 종류의 패턴이 존재한다. 해당 패턴의 종류와 정규식을 활용하는 방법 등에 대해서는 다음 번에 기회가 되면 정리해 보도록 하자 ✊

<br>

### 참조

[MDN - RegExp](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/RegExp){:target="\_blank"}\\
[Stackoverflow - Template Literals with Javascript Replace? [duplicate]](https://stackoverflow.com/questions/51863348/template-literals-with-javascript-replace){:target="\_blank"}
