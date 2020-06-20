---
layout: post
title: "CRA 프로젝트에서 절대경로 사용하기"
categories: [react]
comments: true
---

CRA로 구성된 리액트 앱에서 절대 경로로 import하는 방법을 알아 본다.

<!--more-->
<br>

## 개요

절대경로를 통해 컴포넌트나 애셋을 불러 오면 코드를 더욱 깔끔하고 읽기 쉽게 만들 수 있다.\\
나는 로컬에 있는 이미지를 리액트 컴포넌트에서 불러오고자 하여 해당 방법에 대해 찾아 보았다.

### 실행 환경

먼저, CRA로 구성한 리액트 프로젝트가 존재한다는 가정 하에 작성한다.

- `create-react-app@3.3.0`

<br>

## 절대경로로 import하는 방법

### 1. 프로젝트 루트 경로에 jsconfig.json 생성

_만약 typescript를 사용한다면 `tsconfig.json` 파일이 이에 해당한다._

{% highlight bash %}
my-app
├── node_modules/
├── public/
├── src/
├── jsconfig.json # 생성
└── package.json
{% endhighlight %}

### 2. jsconfig 파일 내용 작성

{% highlight js %}
{
  "compilerOptions": {
    "baseUrl": "src"
  },
  "include": ["src"]
}
{% endhighlight %}

### 3. 불러오기 단계

이전 단계로 인해 절대경로 import 설정이 적용되었다. 프로젝트 설정이 변경되었으므로 만약 개발 서버가 실행중이라면 종료 후 재시작하고, 리액트 컴포넌트를 수정해 본다.

**현재 프로젝트 구조**

{% highlight bash %}
my-app
├── ...
├── src/
│   ├── actions/
│   ├── assets/
│   ├── components/
│   ├── index.js
│   ├── reducers/
│   ├── sagas/
│   └── services/
├── node_modules/
{% endhighlight %}

지금부터는 `src/components/Product.js` 파일에서 `src/assets/image.jpg` 파일을 불러올 것이다.

**src/components/Product.js**

{% highlight js %}
import React from "react";

const Product = (props) => {
  const { image, title } = props;
  return (
    <div>
      <p>{title}</p>
      <img src={require(`assets/${image}`)} alt={title} />
    </div>
  );
};

export default Product;
{% endhighlight %}

이전 단계에서 설정에 `baseUrl`을 `src/`로 설정했기 때문에, `require('assets/...')`로 간단하게 경로에 접근할 수 있었다.\\
참고로 리액트에서 로컬 이미지를 불러오기 위해 `require()`를 사용하였다.

<br>

## 참조

[Create React App 공식 문서 - Importing a component](https://create-react-app.dev/docs/importing-a-component#absolute-imports){:target="\_blank"}\\
[Stackoverflow - how do i reference a local image in react](https://stackoverflow.com/questions/39999367/how-do-i-reference-a-local-image-in-react){:target="\_blank"}
