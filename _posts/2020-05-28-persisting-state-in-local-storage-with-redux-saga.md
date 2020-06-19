---
layout: post
title: "Redux Saga를 이용해 Local Storage에 저장해보기"
categories: [react]
comments: true
---

Redux saga와 함께 애플리케이션의 state를 브라우저 내의 로컬 스토리지에 저장하여 영구적으로 보관하는 방법에 대해 알아 본다. 🗃

<!--more-->

<br>

## 개요

예제를 위한 프로젝트는 redux-saga의 [shopping cart 예제](https://github.com/redux-saga/redux-saga/tree/master/examples/shopping-cart){:target="\_blank"}를 기반으로 한 장바구니 프로젝트이다. 이 포스트에서 작성한 코드가 담긴 전체 코드는 [깃허브](https://github.com/mnngfl/my-react-projects/tree/master/redux-saga-shopping-cart){:target="\_blank"}에서 확인 가능하다.

추가적으로 내가 구현해 본 "좋아요" 기능은 의류 쇼핑몰 h&m의 장바구니와 즐겨찾기 메뉴를 참고하여 비슷하게 만들어 봤다.

<br>

### 구현할 기능

- 개별 상품에 좋아요/좋아요 취소를 할 수 있는 기능
- "좋아요"를 누른 상품 ID의 목록을 배열로 저장하여 좋아요 상태를 브라우저의 로컬 스토리지 저장소에 유지
- 장바구니 내의 "좋아요" 버튼을 누르면 해당 상품은 장바구니에서 삭제

이 내용을 기반으로 기존 프로젝트에 "좋아요" 기능을 추가적으로 붙여 보았다. 과정은 아래의 내용과 같다.

<br>

## 구현

### 1. favorites 리듀서 작성

`favorites` 리듀서로 처리할 기능은 세 가지이다.

1. "좋아요" 상태 불러오기 -> ADD_TO_FAVORITES
2. "좋아요" 추가하기 -> REMOVE_FROM_FAVORITES
3. "좋아요" 삭제하기 -> RECEIVE_FAVORITES

이에 따라 Action, Reducer, Action Create를 내용에 맞게 작성해 주었다.

**/actions/index.js**

```js
export const GET_ALL_FAVORITES = "GET_ALL_FAVORITES";
export const RECEIVE_FAVORITES = "RECEIVE_FAVORITES";
export const ADD_TO_FAVORITES = "ADD_TO_FAVORITES";
export const REMOVE_FROM_FAVORITES = "REMOVE_FROM_FAVORITES";

export function getAllFavorites() {
  return {
    type: GET_ALL_FAVORITES,
  };
}

export function receiveFavorites(products) {
  return {
    type: RECEIVE_FAVORITES,
    products,
  };
}

export function addToFavorites(productId) {
  return {
    type: ADD_TO_FAVORITES,
    productId,
  };
}

export function removeFromFavorites(productId) {
  return {
    type: REMOVE_FROM_FAVORITES,
    productId,
  };
}
```

**/reducers/favorites.js**

```js
const initialState = {
  likedIds: [],
};

export default function (state = initialState, action) {
  const { productId } = action;
  switch (action.type) {
    case RECEIVE_FAVORITES:
      return {
        ...state,
        likedIds: action.products,
      };
    case ADD_TO_FAVORITES: {
      // productId의 중복을 피하기 위해 기존 상태를 Set 자료형으로 복사하여 항목을 추가
      const copy = new Set(state.likedIds);
      copy.add(productId);
      return {
        ...state,
        likedIds: Array.from(copy),
      };
    }
    case REMOVE_FROM_FAVORITES: {
      const copy = [...state.likedIds];
      const idx = copy.indexOf(productId);
      if (idx > -1) copy.splice(idx, 1);
      return {
        ...state,
        likedIds: copy,
      };
    }
    default:
      return state;
  }
}

export function getLikedIds(state) {
  return state.likedIds;
}
```

<br>

### 2. 브라우저로부터 이전 "좋아요" 상태 불러오기

다음으로는 service 코드를 수행할 api 함수를 작성한다.

**/services/index.js**

```js
...
getAllFavorites() {
  try {
    const products = localStorage.getItem("FAVORITE_ITEMS");
    if (products === null) {
      return undefined;
    }
    return JSON.parse(products);
  } catch (error) {
    return undefined;
  }
}

saveFavorites(state) {
  try {
    const products = JSON.stringify(state);
    localStorage.setItem("FAVORITE_ITEMS", products);
  } catch { }
}
...
```

이렇게 작성한 함수는 브라우저의 로컬 스토리지로부터 각각 "FAVORITE_ITEMS"라는 key에 해당하는 값을 불러오거나 저장하도록 한다.

<br>

## 3. saga 수정하기

기존 saga 파일에 다음 내용을 추가한다.

**/sagas/index.js**

```js
...
export function* getAllFavorites() {
  const products = yield call(api.getAllFavorites);
  yield put(actions.receiveFavorites(products));
}

export function* saveFavorites() {
  const state = yield select(getFavorites);
  yield call(api.saveFavorites, state);
}

export function* watchSaveFavorites() {
  while (true) {
    yield take([actions.ADD_TO_FAVORITES, actions.REMOVE_FROM_FAVORITES]);
    yield fork(saveFavorites);
  }
}

// rootSaga
export default function* root() {
  yield all([
    ...
    fork(getAllFavorites),
    fork(watchSaveFavorites),
    ...
  ]);
}
```

rootSaga는 여러 개의 saga를 `rootSaga`라는 단일 진입점으로 불러와 sagaMiddleware를 실행시키게끔 한다.
이렇게 `rootSaga`에 추가한 `fork(getAllFavorites)`,`fork(watchSaveFavorites)`라는 코드는 애플리케이션 실행 시에 `all()` 헬퍼 내의 다른 여러 함수와 함께 병렬적으로 실행되어진다.

다른 함수들에 대해 각각 간단한 설명을 붙이자면, `getAllFavorites` 함수는 위에서 작성한 api 함수를 호출하여 로컬 스토리지에 "FAVORITE_ITEMS" 키에 해당하는 값이나 `undefined`라는 상태를 애플리케이션의 상태로서 저장할 수 있게 한다.

또 `watchSaveFavorites` 함수는 watcher 함수로써, 좋아요 추가/삭제 액션이 있을 때마다 saveFavorites 함수를 호출하는 역할을 한다.

<br>

### 4. 일정 시간마다 모아서 요청 처리하기

지금까지의 과정만으로도 좋아요/좋아요 취소 동작은 정상적으로 작동하지만, 짧은 시간 내에 "좋아요" 버튼을 과도하게 반복해서 누른다면 문제가 되지 않을까 하는 의문이 떠오르게 되었다.
"좋아요" 기능 같은 경우 현재 구현한 대로라면 버튼을 누를 때마다 저장 액션을 호출하여 현재 상태를 로컬 스토리지에 반복해서 저장하고 있었다.

관련 내용을 찾아 보던 중 로컬 스토리지의 특징 중 하나로, 다음과 같은 사항이 있었다.

> 모든 로컬 스토리지 호출은 "동기적"으로 일어난다.

그렇기 때문에 만약 복잡한 어플리케이션에서 부하가 큰 로컬 스토리지 동작을 여러 번 요청한다면, 이는 앱을 느리게 하는 주요 원인이 될 수도 있다. 따라서 액션이 요청될 때 마다 로컬 스토리지에 접근하는 것이 아닌 특정 ms 주기마다 마지막 이벤트가 일어난 시점에만 "좋아요" 상태를 저장하기 위해 `Debouncing` 개념을 saga에 적용시켰다.
redux saga 공식 문서 내에도 이미 Debouncing 기능과 관련된 built-in 헬퍼가 존재했다.

그리하여 이전에 작성하였던 코드를 아래와 같이 수정해 주었다.

**/sagas/index.js**

```js
export function* saveFavorites() {
  // delay 헬퍼를 통해 아래의 작업을 실행하기까지 500ms만큼 지연시킨다
  yield delay(500);
  const state = yield select(getFavorites);
  yield call(api.saveFavorites, state);
}

export function* watchSaveFavorites() {
  let task;

  while (true) {
    yield take([actions.ADD_TO_FAVORITES, actions.REMOVE_FROM_FAVORITES]);
    // 이전 task가 남아있는 경우 현재 수행중인 task를 cancel 헬퍼를 이용해 종료시킨다
    if (task) {
      yield cancel(task);
    }
    task = yield fork(saveFavorites);
  }
}
```

이전에는 "좋아요" 버튼을 누르는 즉각 변경된 state가 로컬 스토리지에 반영이 되었다. 반면 변경된 코드를 실행하면 마지막으로 요청이 일어난 뒤 0.5초가 지난 후에만 상태를 읽어와 로컬 스토리지에 저장할 수 있게 된다.

<br>

### Debouncing 적용 비교

**적용 전**
![not-using-debouncing](https://user-images.githubusercontent.com/47686322/83240933-0f003800-a1d5-11ea-8834-cc04e8140585.gif)

**적용 후**
![using-debouncing](https://user-images.githubusercontent.com/47686322/83240974-1c1d2700-a1d5-11ea-9ea5-f0fb78625e11.gif)

<br>

## 참조

[Persisting Redux State to Local Storage
](https://medium.com/@jrcreencia/persisting-redux-state-to-local-storage-f81eb0b90e7e){:target="\_blank"}\\
[Redux-saga docs: Recipes](https://redux-saga.js.org/docs/recipes/){:target="\_blank"}
