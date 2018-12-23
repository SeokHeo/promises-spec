# Promises/A+

> 이 글은 Promises/A+을 한국어로 번역한 문서입니다.

**구현자에 의해, 구현자를 위한 자바스크립트 Promises에 대한 표준**

*promise*는 비동기식 작업의 최종 결과를 보여줍니다. promise가 작용하는 기본적인 방법은 `then` 메소드를 이용하는 것입니다. 사용자들은 promise의 결과값을 받거나 promise가 실행될 수 없는 이유를 받기 위한 콜백들을 등록하게 됩니다.

이 스펙문서는 Promises/A+를 따르는 모든 promise 구현체들에게 기본이 될 수 있도록 `then` 메소드의 행동을 상세하게 기술합니다. 따라서 이 문서는 매우 정확해야 합니다. 때때로 Promises/A+ 조직에서 새로 발견된 예외를 수정하기 위해 이 문서를 개정할 수 있지만, 신중한 검토와 토론 그리고 테스트를 거친 다음에만 문서가 업데이트 될 것입니다.

Promises/A+는 이전 [Promises/A proposal](http://wiki.commonjs.org/wiki/Promises/A)의 조항에 있는 *실제* 행동들의 불충분하거나 문제가 있는 부분들을 보충하여 명확하게 하고자 합니다.

마지막으로, Promises/A+의 핵심은 어떻게 promise를 만들거나 충족시키는지에 대해선 다루지 않습니다. 대신에 `then` 메소드를 제공하는 방법에 대해 집중합니다. 추후 비슷한 문서들에서 이런 주제들을 다룰 순 있습니다.

## 용어

1. "promise"는 `then` 메소드의 규격에 부합하는 객체 또는 함수입니다.
1. "thenable"은 `then` 메소드를 정의하는 객체 또는 함수입니다. 
1. "value"는 허용할 수 있는 모든 자바스크립트 값 입니다. (`undefined`, a thenable, 또는 promise 포함)
1. "exception"은 `throw`를 통해 던져진 값을 의미합니다.
1. "reason"은 왜 promise가 거절됬는지를 나타내는 값을 의미합니다.

## 요구사항

### Promise 상태

promise는 반드시 다음 세가지 상태들 중 하나를 가져야 합니다: 미결(pending), 이행(fulfilled), 또는 거절(rejected).

1. promise가 pending 상태일 때:
    1. fulfilled 상태 또는 rejected 상태로 전환될 수 있습니다.
1. promise가 fulfilled 상태일 때:
    1. 다른 상태로 전환될 수 없습니다.
    1. 반드시 값을 가져야 하며, 변경할 수 없습니다.
1. promise가 rejected 상태일 때:
    1. 다른 상태로 전환될 수 없습니다.
    1. 반드시 reason을 가져야 하며, 변경할 수 없습니다.

여기서 말하는 "변경 할 수 없습니다"는 불변성을 의미하나(i.e. `===`), `deep imuutability`를 뜻하는건 아닙니다.

### `then` 메소드

promise는 결과 값 또는 reason에 접근하기 위해 반드시 `then` 메소드를 제공해야 합니다.

promise의 `then` 메소드는 두 개의 인자들을 받습니다.

```js
promise.then(onFulfilled, onRejected)
```
1. `onFulfilled` 와 `onRejected`는 옵션 인자입니다:
    1. 만약 `onFulfilled`이 함수가 아니라면, 그 인자는 무시됩니다.
    1. 만약 `onRejected`가 함수가 아니라면, 그 인자는 무시됩니다.
1. 만약 `onFulfilled`이 함수일 경우:
    1. `promise`가 이행 되었을 때, 첫번째 인자인 값(value)과 함께 호출되어야 합니다.
    1. `promise`가 이행 되기전에 호출 되어서는 안됩니다.
    1. 1번 이상 호출 되어서는 안됩니다.
1. 만약 `onRejected`가 함수일 경우:
    1. `promise`가 거절(reject)되었을 때, 첫번째 인자인 이유(reason)와 함께 호출 되어야 합니다.
    1. `promise`가 거절 되기전에 호출 되어서는 안됩니다.
    1. 1번 이상 호출 되어서는 안됩니다.
1. `onFulfilled` 또는 `onRejected` 는 [실행 컨텍스트](https://es5.github.io/#x10.3) 스택이 platform code[[3.1](#notes)]를 포함할 때까지 호출 되어서는 안됩니다.
1. `onFulfilled` 와 `onRejected`는 반드시 함수로 호출되어야 합니다. (i.e. `this` 값 없이)[[3.2](#notes)]
1. `then`은 같은 promise에서 여러번 호출 될 수 있습니다.
    1. `promise`가 이행 되었다면, 상대적으로 사용된 모든 `onFulfilled` 콜백들은 `then`의 호출 순서에 따라 실행 되어야 합니다.
    1. `promise`가 거절 되었다면, 상대적으로 사용된 모든 `onRjected` 콜백들은 `then`의 호출 순서에 따라 실행 되어야 합니다.
1. `then`은 반드시 promise를 return해야 합니다[[3.3](#notes)].

    ```js
    promise2 = promise1.then(onFulfilled, onRejected);
    ```
    
    1. 만약 `onFulfilled` 또는 `onRejected` 중 하나가 `x` 값을 반환하면, Promise Resolution Procedure `[[Resolve]](promise2, x)`를 실행합니다.
    1. 만약 `onFulfilled` 또는 `onRejected` 중 하나가 `e` 예외를 던지면, `promise2`는 반드시 이유(reason)로서 `e`와 함께 거절(rejected) 되어야 합니다.
    1. 만약 `onFulfilled`가 함수가 아니고, `promise1`이 이행되었다면, `promise2`는 반드시 `promise1`과 같은 값과 함께 이행 되어야 합니다.
    1. 만약 `onRejected`가 함수가 아니고, `promise1`이 거절되었다면, `promise2`는 반드시 `promise1`과 같은 이유와 함께 거절 되어야 합니다.
    
### The Promise Resolution Procedure

**promise resolution procedure**은 입력값으로 promise와 값(value)를 다루는 방법에 대해서 추상화하는 작업으로, `[[Resolve]](promise, x)`으로 부릅니다. 만약 `x`가 thenable하다면, `x`는 적어도 약속과 같이 행동한다는 가정하에 `x`의 상태를 `promise`로 간주하고록 시도합니다. 그렇지 않다면, `x`의 값(value)과 함께 `promise`를 이행합니다.

thenables에 대한 처리를 통해 `then` 메소드를 따르는 Promises/A+ promise가 구현될 수 있도록 도와줍니다. 또한 Promises/A+로 구현한다면 합리적인 `then`방식을 통해 부적합한 구현을 "평가" 할 수 있습니다.

[Resolve](promise, x)를 실행하려면, 다음 단계를 수행해야 합니다.

1. 만약 `promise`와 `x`가 같은 오브젝트를 참조한다면, reason으로서 `TypeError`와 함께 `promise`를 거절(reject) 해야 합니다.
1. 만약 `x`가 promise라면, 상태를 가지고 있어야 합니다[[3.4](#notes)]:
    1. 만약 `x`가 pending상태라면, `promise`는 `x`가 이행(fulfilled) 또는 거절(reject)될 때 까지 pending 상태로 남아있어야 합니다.
    1. 만약 `x`가 이행(fulfilled)되었다면, 같은 값(value)과 함께 `promise`를 이행되어야 합니다.
    1. 만약 `x`가 거절(rejected)되었다면, 같은 이유(reason)과 함께 `promise`가 거절되어야 합니다.
1. 반대로, `x`가 오브젝트 또는 함수라면,
    1. `then`은 `x.then`이 되어야합니다. [[3.5](#notes)]
    1. 만약 `x.then`의 결과가 `e` 예외를 던진다면, `e`의 이유(reason)과 함께 `promise`를 거절(reject)합니다.
    1. 만약 `then`이 함수라면, `x`를 `this`로서 호출하고, 첫번째 인자에 `resolvePromise`, 두번째 인자에 `rejectPromise`를 넘깁니다:
        1. 만약 `resolvePromise`가 값(value) `y`와 함께 호출된다면, `[[Resolve]](promise, y)`를 실행합니다.
        1. 만약 `rejectPromise`가 이유(reason)와 함께 호출된다면, `r`과 함께 `promise`를 거절합니다.
        1. 만약 `resolvePromise` 와 `rejectPromise`가 모두 호출되거나 같은 인자에 대해 여러번 호출이 일어난다면, 첫번째 호출을 우선시하고, 더 이상의 호출은 무시합니다.
        1. 만약 `then`을 호출했을 때 `e` 에러를 발생시킨다면,
            1. `resolvePromise` 또는 `rejectPromise` 가 이미 호출되었다면, 무시합니다.
            2. 그렇지 않으면, `e`와 함께 `promise`를 거절합니다.
    1. 만약 `then`이 함수가 아니라면, `x`와 함께 `promise`를 이행합니다.
1. 만약 `x`가 오브젝트 또는 함수가 아니라면, `x`와 함께 `promise`를 이행합니다.

만약 하나의 promise가 circular thenable chain에 참여하는 thenable과 함께 할당 된다면, `[[Resolve]](promise, thenable)`의 재귀적 성질은 결국 `[[Resolve]](promise, thenable)`을 다시 호출할 것이고, 위 알고리즘을 따른다면 끊임없이 재귀가 발생할 것입니다. 반드시 필요한 것은 아니지만, 재귀, `TypeError`의 이유(reason)와 함께 거절된 `promise`를 탐지할 수 있으므로 구현하는 것을 권장합니다. [[3.6](#notes)]

## Notes

1. 여기서 언급한 "platform code"는 엔진, 환경, 그리고 prmise 구현 코드를 의미합니다. 실제로, 이 요구상항은 이벤트 루프가 `then`을 호출한 후에 새로운 스택과 함께 `onFulfilled` 와 `onRejected`가 비동기식으로 실행 될 수 있도록 보장합니다. 이것은 [`setTimeout`](https://html.spec.whatwg.org/multipage/webappapis.html#timers), [`setImmediate`](https://dvcs.w3.org/hg/webperf/raw-file/tip/specs/setImmediate/Overview.html#processingmodel), "micro-task" 매커니즘인 [`MutationObserver`](https://dom.spec.whatwg.org/#interface-mutationobserver), [`process.nextTick`](http://nodejs.org/api/process.html#process_process_nexttick_callback)과 같은 "macro-task" 메커니즘으로 구현할 수 있습니다. promise 구현체가 platform code를 고려함에 따라, task-scheduling queue 또는 "trampline" 같이 핸들러로 호출되는 부분을 포함 할 수 있습니다.

1. strict mode의 `this`가 그 안에서 `undefined`가 될 수 있습니다; sloppy mode에서는, 전역 객체가 될 수 있습니다.

1. 구현체는 구현이 모든 요구사항을 충족할 경우 `promise2 === promise1`가 되는것을 허용합니다. 각각의 구현체들은 어떤 조건하에서 `promise2 === promise1`를 만들 수 있는지 명시해야 합니다.

1. 일반적으로, 현재 구현으로부터 만들어진다면, `x`가 promise라는 것을 알 수 있습니다. 이 조항은 promise의 상태를 성택하는데 구체적인 구현방법에 대해 이야기합니다.

1. `x.then` 레퍼런스의 첫번째 저장 절차, 그리고 테스팅하는 방법, 그리고 호출하는 방법은 `x.then`의 프로퍼티에 여러번 접근하는 것을 피할 수 있습니다. 이러한 주의사항은 탐색할 때 값(value)이 변경될 수 있는 프로퍼티에 접근할 때 일관성을 보장하기 위해 매우 중요합니다.

1. 구현은 thenable chain의 깂이에 대해 한계를 설정해서안 *안되며*, 그 한계 이상으로 재귀가 무한하다고 가정해야 합니다. 실제 순환참조들만 `TypeError`로 이어질 수 있습니다; thenables의 무한한 체인을 발견한다면, 무한 재귀는 올바른 행동입니다.

---

<p xmlns:dct="http://purl.org/dc/terms/" xmlns:vcard="http://www.w3.org/2001/vcard-rdf/3.0#">
  <a rel="license"
     href="https://creativecommons.org/publicdomain/zero/1.0/">
    <img src="https://i.creativecommons.org/p/zero/1.0/88x31.png" style="border-style: none;" alt="CC0" />
  </a>
  <br />
  To the extent possible under law,
  <a rel="dct:publisher"
     href="https://github.com/promises-aplus">
    <span property="dct:title">the Promises/A+ organization</span></a>
  has waived all copyright and related or neighboring rights to
  <span property="dct:title">Promises/A+ Promise Specification</span>.
This work is published from:
<span property="vcard:Country" datatype="dct:ISO3166"
      content="US" about="https://github.com/promises-aplus">
  United States</span>.
</p>
