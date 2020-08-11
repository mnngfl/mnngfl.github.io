---
layout: post
title: "JavaScript에서 2차원 배열을 사용하는 방법 (다차원 배열)"
categories: [javascript]
comments: true
---

🤷‍♀️ JavaScript를 이용해 알고리즘 문제를 풀다가 2차원 배열을 사용하는 부분에서 막혀 이에 관해 알아보고자 작성하게 되었다.

<!--more-->

### 문제가 발생하였던 부분

작성하였던 코드는 아래와 같이 2차원 배열을 선언하고 반복문 안에서 2차원 배열의 인덱스에 직접 접근해 값을 할당하려고 하였는데, `TypeError`를 마주하였다.

```js
let arr = [[]];

for (let i = 0; i < x; i++) {
  for (let j = 0; j < y; j++) {
    arr[i][j] = ???
  }
}
```

- `TypeError: Cannot set property '0' of undefined`

이 문제는 초기화되지 않은 배열의 공간에 접근하려 해서 발생한 문제인 것으로 보인다. 배열을 잘못된 방법으로 사용하고 있었던 것 같아 아래부터는 2차원 배열과 관련하여 검색 혹은 책을 통해 알아낸 내용을 기술하였다.

<br>

### 자바스크립트에서 배열 생성하기

다차원 배열을 생성하기 전에, 먼저 자바스크립트에서 배열을 생성할 수 있는 방법은 두 가지가 있다. 바로 **리터럴 구문**을 이용하는 방법과 **Array 생성자**를 이용하는 방법이다. 다음 코드의 변수 `arr1`, `arr2`를 선언하는 부분이 이 두 가지 방식을 각각 따르고 있다.

```js
let arr1 = [];
let arr2 = new Array(3);

console.log(arr1); // -> []
console.log(arr2); // -> [empty × 3]
```

<br>

### 2차원 배열 생성하기

자바스크립트에서 다차원 배열을 정의할 수 있는 문법이 따로 제공되고 있지는 않다. 그 대신 배열 안에 배열을 중첩하여 초기화하면 다차원 배열과 비슷하게 작동하도록 구현할 수 있다.

#### 2차원 배열 생성

다음은 3 × 3 크기의 2차원 배열을 만드는 코드이다.

```js
let a = new Array(3); // or Array(3) or []
for (let i = 0; i < 3; i++) {
  a[i] = new Array(3);
}
```

이렇게 생성한 2차원 배열에 값을 대입하려고 하면 다음과 같이 할 수 있다.

```js
for (let count = 1, i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    a[i][j] = count++;
  }
}

console.log(a);
// -> [Array(3), Array(3), Array(3)]
// -> a[0]: [1, 2, 3]
// -> a[1]: [4, 5, 6]
// -> a[2]: [7, 8, 9]
```

2차원 배열의 요소를 읽고 쓰고자 할 때는 [] 연산자를 두 번 사용해야 한다. 만약 행 번호와 열 번호가 i, j인 요소를 찾고자 한다면, `a[i][j]`와 같이 표기하도록 한다.

<br>

#### 2차원 배열의 참조 관계

방금 전 생성한 `a`라는 이름의 2차원 배열의 참조 관계를 나타내면 다음 그림과 같다.

![2차원-배열-참조](https://user-images.githubusercontent.com/47686322/89908526-f1196f80-dc28-11ea-8e6d-ae3705028cfa.png)

이전에 작성한 오류가 발생했던 코드를 예로 들어보자면 `arr = [[]]`라고 선언했을 때 `arr[0]`은 빈 배열 []을 참조하고, `arr[0][0]`은 값이 할당되지 않았으므로 `undefined`를 반환하게 된다.

반복문 안에서 사용하였으니 반복문 안에서 만약 i = 3이고 j = 5였다면, `arr[3][5]`에 값을 할당해야 하는데 `arr[3]`은 초기화가 되지 않아 `undefined`를 가르키므로, `arr[3][5]`에도 당연히 접근이 불가능하다. 그러므로 오류가 발생했던 것이다.

<br>

#### 2차원 배열의 요소 초기화하기

초기화되지 않은 배열 요소의 접근을 위해선 배열 생성 시에 값을 0으로 채우든, 빈 배열([])이나 true 값으로 채우든 초기화해주는 작업이 필요하다.

다음은 모두 행 크기 3, 열 크기 5의 2차원 배열을 생성하는 코드이다.

```js
let matrix = [];
for (let i = 0; i < 5; i++) {
  matrix[i] = Array(5);
}

// ES6
matrix = Array.from(Array(3), () => Array(5));
matrix = Array(3)
  .fill(0)
  .map(() => Array(5).fill(0));
```

<br>

### 다차원 배열 생성

2차원 배열을 생성하는 방법과 동일하게 다차원 배열을 만들 수가 있다.

다음은 3 × 3 × 3 크기의 3차원 배열을 생성하는 코드이다.

```js
// 배열 생성
let a = new Array(3);
for (let i = 0; i < 3; i++) {
  a[i] = new Array(3);
  for (let j = 0; j < 3; j++) {
    a[i][j] = new Array(3);
  }
}

// 값 대입
for (let cnt = 1, i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    for (let k = 0; k < 3; k++) {
      a[i][j][k] = cnt++;
    }
  }
}
```

<br>

### 참조

[MDN - Array](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array){:target="\_blank"}\\
[How can I create a two dimensional array in JavaScript?](https://stackoverflow.com/questions/966225/how-can-i-create-a-two-dimensional-array-in-javascript){:target="\_blank"}\\
[How to create multidimensional array](https://stackoverflow.com/questions/7545641/how-to-create-multidimensional-array){:target="\_blank"}
