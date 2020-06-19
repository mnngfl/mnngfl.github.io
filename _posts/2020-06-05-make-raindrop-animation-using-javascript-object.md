---
layout: post
title: "JavaScript의 객체를 활용한 캔버스 애니메이션"
categories: [javascript]
comments: true
---

자바스크립트와 HTML5의 캔버스를 이용해 객체 기반의 애니메이션을 생성하는 과정을 담았다. 자바스크립트로 비가 내리는 효과의 애니메이션을 만들어 보자! 🌧

<!--more-->

## 캔버스에서의 애니메이션의 기본적인 흐름

애니메이션을 캔버스 위에 그리기 위해선 일반적으로 다음과 같은 과정을 거친다.

**1. `canvas` 지우기**
: 애니메이션을 나타내기 위해서는 이전에 그렸던 도형은 캔버스에서 지워야 한다. 이러한 작업은 `clearRect()` 메서드를 이용함으로써 간단히 이루어질 수 있다.

**2. `canvas` 상태 저장하기**
: 만약 캔버스 상태에 영향을 주는 설정(style, transformations 등)을 변경하였다면 이러한 변경 상태를 저장해야 한다. 그래야 변경한 설정을 새로운 프레임을 그릴 때에도 유지할 수가 있게 된다. `save()` 메서드를 이용한다.

**3. 애니메이션 모양 그리기**
: 실제 프레임이 렌더링되는 단계이다.

**4. `canvas` 상태 복구하기**
: 만약 이전 단계에서 캔버스의 상태를 저장하였다면, 새 프레임을 그리기 전에 해당 설정을 복원하도록 한다.

<br>

## 애니메이션을 제어하는 방법

우리가 그리고자 하는 도형은 `canvas`의 메서드를 직접 이용하거나 사용자 지정 함수 호출에 의해서 화면에 그려질 수 있다. 또한 이러한 그리기 함수들을 일정 시간 주기로 실행시켜야만 시각적 업데이트의 구현이 가능하다.

설정한 시간 동안에 특정 함수를 호출할 수 있도록 하는 함수로는 다음과 같은 것들이 있다.

- setInterval(function, delay)

  일정 시간(ms)마다 `function`으로 전달된 함수를 실행한다.

- setTimeout(function, delay)

  일정 시간(ms)이 지난 뒤 `function` 함수를 실행한다.

- requestAnimationFrame(callback)

  브라우저에게 애니메이션을 수행하고자 한다는 것을 알리고 다음 리페인트를 수행하기 전 브라우저가 지정된 `callback`을 호출해 애니메이션을 이전 상태에서 업데이트하도록 요청한다.\\
  `requestAnimationFrame`(이하 `rAF`)은 일반적으로 1초에 60회 콜백을 요청하여 부드러운 애니메이션이 느껴지도록 하고, 최신 브라우저에서 성능과 배터리 수명 향상을 위해 백그라운드 탭이나 `<iframe>`에서는 실행을 중지하게 된다.

> **rAF 사용을 권장하는 이유**\\
> 애니메이션과 같이 화면에 시각적인 변화가 발생하고 있을 때 이러한 작업이 정확한 시간마다 수행되길 원할 것이다. 애니메이션 업데이트를 위해 `setInterval`이나 `setTimeout`을 사용할 수도 있지만, 이들은 콜백이 프레임의 끝에서 실행될 것이며 종종 프레임이 누락되어 버벅이는 현상이 발생할 수도 있다고 한다. 따라서 프레임 시작과 동시에 실행을 보장하는 `rAF`를 사용하도록 하자.

<br>

## Raindrop 애니메이션 만들어보기

지금부터는 캔버스를 하나 생성하고 그 안에 빗방울이라는 자바스크립트 객체를 여러 개 만들어 이들이 떨어지도록 하는 효과의 애니메이션을 만들어 보자.

<br>

#### 전체적인 흐름 알아 보기

세부 사항 구현에 앞서 어떠한 순서로 구현을 진행할지에 대해 요약해 본다.

- 빗방울 객체 구상하기 (속성, 메서드)
- 캔버스에 빗방울 객체들을 캔버스에 그리기
- 애니메이션 효과를 위해 일정 시간을 주기로 좌표를 갱신하는 함수 실행시키기

<br>

#### 빗방울 객체의 생성자

각 빗방울 객체를 나타내기 위해서는 (x, y) 좌표와 길이, 속도라는 특성을 나타내는 프로퍼티가 필요하다. 이를 생성자로 나타내보면 다음과 같다.

```js
function Rain(x, y, l, v) {
  this.x = x;
  this.y = y;
  this.l = l;
  this.v = v;
}
```

다음으로 빗방울의 행동을 정의하기 위해 `Rain` 생성자의 프로토타입을 수정하여 메서드를 추가해 보자. `Rain` 생성자 안쪽에서 `this.메서드명`으로 선언하지 않는 이유는, 해당 방식으로 객체 인스턴스를 생성할 경우 모든 인스턴스에 똑같은 메서드가 추가되기 때문이다. 이렇게 메서드를 포함하는 생성자로 인스턴스를 여러 개 생성하게 되면 똑같은 작업을 하는 메서드가 그만큼 여러 개 생성된다는 것이고 이로 인해 메모리 낭비로 이어지게 된다는 특징이 있다.

따라서 아래와 같이 생성자의 프로토타입 객체에 메서드를 추가하는 방식을 통하여 모든 객체 인스턴스가 추가된 메서드를 사용할 수 있도록 하는 방식이 바람직하다.

```js
Rain.prototype = {
  // 캔버스에 빗방울 객체를 그리는 과정
  draw: function () {
    ctx.beginPath();
    ctx.strokeStyle = "white";
    ctx.lineWidth = "0.7";
    ctx.moveTo(this.x, this.y);
    ctx.lineTo(this.x, this.y + this.l);
    ctx.stroke();
  },
  // 이전 상태로부터 좌표를 업데이트하는 과정
  update: function () {
    this.y += this.vy;

    if (this.y + this.l >= canvas.height) {
      this.x = Math.floor(Math.random() * canvas.width);
      this.y = Math.floor(Math.random() * 80) - 80;
      this.l = Math.floor(Math.random() * 30) + 10;
      this.vy = Math.floor(Math.random() * 10) + 2;
    }
  },
};
```

<br>

#### 캔버스 업데이트와 그리기

`Rain` 생성자의 `update` 메서드는 빗방울 객체의 y좌표를 이동시켜 아래로 떨어지는 효과를 만들어 낸다. 또한 `draw` 메서드는 현재 객체의 x, y, height 프로퍼티를 기반으로 캔버스에 빗방울을 그려내게 된다.

그렇다면 이 두 메서드는 어떤 순서로 호출되어야 할까?\\
결과적으로 말하자면 `update()`이 `draw()`보다 먼저 호출되어야 한다. 매 프레임마다 실행되는 `loop` 함수 안에서 `update()`은 빗방울을 표시할 좌표가 업데이트하고, 그 후 `draw()`을 이용해 그려내야 가장 최신의 상태를 캔버스에 유지할 수 있게 된다.

```js
function loop() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  for (let i = 0; i < rains.length; i++) {
    rains[i].update();
    rains[i].draw();
  }

  window.requestAnimationFrame(loop);
}
```

<br>

#### 지금까지의 코드와 결과 확인

<script src="https://gist.github.com/mnngfl/691c247e8faddd8f9c4546e21f94a574.js"></script>

![rainy-day-1](https://user-images.githubusercontent.com/47686322/85141855-63ea1800-b282-11ea-881d-ed6a54690994.gif)

<br>

## 빗방울이 바닥 충돌 시 파동을 일으키도록 구현

한 단계 더 나아가서 빗방울이 캔버스의 바닥 영역에 다다르면 파동을 일으키는 효과를 추가해보자. 이 부분은 [Codepen - Neon Rain](https://codepen.io/natewiley/pen/NNgqVJ?editors=0010){:target="\_blank"}을 참고하여 작성하였다.

<br>

#### 생성자 프로퍼티 추가

해당 내용을 구현하기 위해서는 추가적인 프로퍼티가 몇 개 더 필요하다. 아래와 같이 생성자를 수정해 주었다.

```js
function Rain(x, y, l, vy, hit, r, vr, a, va) {
  this.x = x;
  this.y = y;
  this.l = l;
  this.vy = vy;
  this.hit = hit; // 바닥에 닿는 지점
  this.r = r; // 파동의 반지름
  this.vr = vr; // 파동의 반지름이 증가하는 속도
  this.a = a; // 파동을 표시할 투명도
  this.va = va; // 투명도가 증가하는 속도
}
```

<br>

#### Rain 생성자의 프로토타입 메서드 수정

이전에는 `draw`와 `update` 메서드만 존재했다. 하지만, 객체 인스턴스를 처음 생성했을 때나 물방울의 프로퍼티 값을 초기화 해줘야 하는 경우가 발생해서 `init` 메서드를 추가로 작성하였다. 이에 해당하는 코드는 다음과 같다.

```js
// min 이상 max 미만의 랜덤 정수 값을 반환하는 함수
function random(min, max) {
  min = Math.ceil(min);
  max = Math.floor(max);
  return Math.floor(min + Math.random() * (max - min));
}

...

Rain.prototype = {
  init: function () {
    this.x = random(0, canvas.width);
    this.y = Math.floor(Math.random() * 80) - 80;
    this.l = random(10, 30);
    this.vy = random(2, 10);
    this.hit = random(canvas.height * 0.8, canvas.height * 0.9);
    this.a = 1;
    this.va = 0.96;
    this.r = 5;
    this.vr = 1;
  },
  draw: function () {
    if (this.y + this.l < this.hit) {
      ctx.beginPath();
      ctx.strokeStyle = "white";
      ctx.lineWidth = "0.7";
      ctx.moveTo(this.x, this.y);
      ctx.lineTo(this.x, this.y + this.l);
      ctx.stroke();
    } else {
      ctx.beginPath();
      ctx.strokeStyle = `rgba(255, 255, 255, ${this.a})`;
      ctx.ellipse(
        this.x,
        this.y,
        this.r * 0.4,
        this.r * 1.2,
        Math.PI / 2,
        0,
        2 * Math.PI
      );
      ctx.stroke();
    }
  },
  update: function () {
    if (this.y + this.l < this.hit) {
      this.y += this.vy;
    } else {
      if (this.a > 0.03) {
        this.r += this.vr;
        if (this.r > 15) {
          this.a *= this.va;
          this.vr *= 0.98;
        }
      } else {
        this.init();
      }
    }
  },
};
```

<br>

마치기 전에, `init` 메서드를 이용한다면 객체 인스턴스 생성 시 인자를 전해주지 않아도 되므로 `setup` 함수를 다음과 같이 수정해 준다. 이로 인해 코드가 한결 간단해졌다.

```js
function setup() {
  ...
  for (let i = 0; i < N_RAIN; i++) {
    rains[i] = new Rain();
    rains[i].init(); // 여기서 프로퍼티 값들이 결정됨
  }
  ...
}
```

<br>

#### 최종 코드와 결과

완성된 최종 코드와 브라우저에서 확인해 본 다음과 같다.

<script src="https://gist.github.com/mnngfl/ef173568bd703cf167ac8f518949b114.js"></script>

![rainy-day-2](https://user-images.githubusercontent.com/47686322/85142099-ca6f3600-b282-11ea-9949-2dafc8b77521.gif)

<br>

## 참조

[MDN - Basic animations](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Basic_animations){:target="\_blank"}\\
[Google Developers - 자바스크립트 실행 최적화](https://developers.google.com/web/fundamentals/performance/rendering/optimize-javascript-execution#%EC%8B%9C%EA%B0%81%EC%A0%81_%EB%B3%80%ED%99%94%EC%97%90_requestanimationframe_%EC%82%AC%EC%9A%A9){:target="\_blank"}\\
[Create a smooth canvas animation](https://spicyyoghurt.com/tutorials/html5-javascript-game-development/create-a-smooth-canvas-animation){:target="\_blank"}\\
[Creating a Bouncing Ball Animation Using JavaScript and Canvas](https://medium.com/dev-compendium/creating-a-bouncing-ball-animation-using-javascript-and-canvas-1076a09482e0){:target="\_blank"}
