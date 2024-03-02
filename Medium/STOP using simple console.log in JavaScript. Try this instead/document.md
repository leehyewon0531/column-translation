## 원문 링크

[STOP using just console.log in JavaScript. Try this instead](https://medium.com/@anirudh.munipalli/stop-using-console-log-in-javascript-try-these-instead-72490d895a24)

## JavaScript에서 단순 console.log를 사용하는 것을 멈추세요! 대신 이걸 사용하세요.

디버깅. 프로그래머들이 피하려고 노력하지만 결국 코드에 더 많은 버그들을 만들 뿐인 것.

버그 없는 코딩은 최고의 프로그래머들도 어려워 하는 것이다. 그것이 당신이 항상 코드를 디버깅 해야 하는 이유이다.

그리고 자바스크립트 코드를 디버깅 하는 최고의 방법 중 하나는 놀라운 `console.log()` 이다. 하지만, 더 좋은 방법이 있다.

그리고 그것이 이 아티클의 핵심이다. 콘솔과 상호작용 하는 더 나은 방법들을 말하는 것이.

정교한 IDE에서 console을 타이핑하면 다양한 자동 완성이 주어진다.

![image](https://github.com/leehyewon0531/column-translation/assets/50830078/669c6e7c-ede6-4f26-b902-a3144fd494c7)

VS Code에서 console을 입력했을 때 나오는 자동완성 옵션

일반적인 `console.log()`를 사용하는 것 대신에, 여기에 더 나은 옵션들이 있다.

이것들을 사용하는 것은 디버깅 프로세스를 더 많이 쉽고 빠르게 만든다.

`console.warn()`과 `console.error()` 이다.

당신의 애플리케이션의 동작을 멈출 수 있는 버그가 있을 때, 다른 `console.log`를 사용해 디버깅 하면 동작하지 않는다.

당신의 콘솔 메시지들은 헷갈릴 수 있다. 당신은 당신이 찾는 메시지를 발견할 수 없다.

하지만, `console.warn()`과 `console.error()`를 사용하는 것은 이것을 극복하는 좋은 방법들이다.

```jsx
console.warn("This is a warning");
```

![image](https://github.com/leehyewon0531/column-translation/assets/50830078/849a1478-f58b-46a5-903d-4c44494f26f2)

```jsx
console.error("This is an error");
```

![image](https://github.com/leehyewon0531/column-translation/assets/50830078/cd510747-592a-4980-8db8-23a79ba32b01)

**Timing operations**

코드가 실행되는 데 얼마만큼의 시간이 걸리는지를 보고 싶은가?

`console.time()`을 사용하라.

먼저, timer를 만들고 고유한 이름을 붙여라.

```jsx
console.time("Loop timer")
```

그 다음, 코드 조각을 실행하라.

```jsx
for(i = 0; i < 10000; i++){
    // Some code here
}
```

그 다음, `timeEnd()`를 호출하라.

```jsx
console.timeEnd("Loop timer")
```

다음은 전체 코드이다.

```jsx
console.time("Loop timer")
for(i = 0; i < 10000; i++){
    // Some code here
}
console.timeEnd("Loop timer")
```

![image](https://github.com/leehyewon0531/column-translation/assets/50830078/e3a39398-7800-4223-8581-56975eb78da4)

이것은 신경망이나 HTML Canvas reading과 같이 시간이 좀 걸리는 CPU-intensive 애플리케이션에서 매우 유용하다.

**코드가 어떻게 거기에 갔는지 추적**

함수가 어떻게 호출되는지를 보고 싶은가?

```jsx
function trace(){
    console.trace()
}
function randomFunction(){
    trace();
}
```

여기, `console.trace()`를 호출하는 `trace()` 를 호출하는 `randomFunction`이라는 함수가 있다.

그래서, 당신이 `randomFunction`을 호출할 때, 당신은 다음과 같은 결과를 얻을 것이다.

![image](https://github.com/leehyewon0531/column-translation/assets/50830078/a53b186d-683c-4059-830e-1d0153e7a393)

그것은 `anonymous`(main JavaScript 코드)가 `randomFunction`을 호출하고, `randomFunction`이 `trace()`를 호출하는 것을 보여준다.

**console 메시지를 그룹핑**

만약 당신이 console 메시지들을 그룹핑하면, 당신은 당신의 콘솔을 더 읽기 쉽게 만들 수 있다.

```jsx
console.log("Test1!");

console.group("My message group");

console.log("Test2!");
console.log("Test2!");
console.log("Test2!");

console.groupEnd()
```

모든 Test2 메시지들은 ‘My message group’에 해당된다.

![image](https://github.com/leehyewon0531/column-translation/assets/50830078/5aecbb59-6702-4a33-8623-5e41dc9fbbd3)

**콘솔 지우기**

당신이 이 튜토리얼을 따라오는 중이었다면, 당신의 콘솔은 꽤 가득차 있을 것이다. 그것을 지워 보자.

```jsx
console.clear();
```

콘솔을 보여주겠다.

![image](https://github.com/leehyewon0531/column-translation/assets/50830078/341401e8-6ebc-4693-9ba3-df682bc44c6a)

별로 보여줄 게 없다. 이제 다시 지워졌으니 계속해 보자.

**테이블**

데이터를 더 좋게 시각화하기 위해 테이블을 추가해 보자.

우리가 2개의 객체를 가지고 있다고 상상해 보자.

```jsx
var person1 = {name: "Weirdo", age : "-23", hobby: "singing"}
var person2 = {name: "SomeName", age : "Infinity", hobby: "programming"}
```

간단히 `console.log`는 데이터를 지저분해 보이게 만들 것이다.

![image](https://github.com/leehyewon0531/column-translation/assets/50830078/1e7a77ed-24ba-409c-9828-0f4f7d1682d7)

테이블은 더 낫다.

```jsx
console.table({person1, person2})
```

![image](https://github.com/leehyewon0531/column-translation/assets/50830078/df0c5445-0f20-4b17-9360-aaa18f47ea62)

JavaScript 콘솔이 이렇게 깔끔하게 보일 수 있다는 걸 전혀 몰랐다. 안 그런가?

**console에서의 CSS?**

그렇다, 당신이 읽은 것이 맞다.

당신은 콘솔에 CSS를 추가할 수 있다.

```jsx
console.log("%c I love JavaScript!", 
  "color: red; background-color: lightblue; border: solid");
```

`%c` sign을 확인하라. 그것이 바로 마법이 있는 곳이다.

![image](https://github.com/leehyewon0531/column-translation/assets/50830078/8e72e389-ac6a-433e-91da-17c0c9ff0755)

이 아티클을 지금까지 읽어주셔서 감사하다. 이것은 다양한 JavaScript API와 특징들에 대한 시리즈의 5번째 아티클이다.

당신은 여기서 다른 아티클들을 볼 수 있다. → https://medium.com/@anirudh.munipalli/list/javascript-apis-and-features-68106ad31255

팔로우하면 내가 더 많은 아티클들을 게시할 때 알림을 받을 수 있다. 당신은 굉장한 독자이다.

**Source**

https://www.w3schools.com/jsref/obj_console.asp

## 단어장

| 단어 | 의미 |
| --- | --- |
| only to | 그러나 결국 ~하다 |
| find hard | 어렵게 “느끼다” |
| which is why | 그렇기 때문에 |
| Except, | 하지만 |
| get mixed up | 헷갈리다, 착각하다 |
| come under | 해당하다 |
| not much | 별로 |
| messy | 지저분한, 어질러놓은 |
| so far | 지금까지 |
| get notify | 알림을 받다 |
