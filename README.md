# Promises/A+

> 이 글은 Promises/A+을 한국어로 번역한 문서입니다.

**구현자에 의해, 구현자를 위한 자바스크립트 Prmises에 대한 표준**

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

The **promise resolution procedure** is an abstract operation taking as input a promise and a value, which we denote as `[[Resolve]](promise, x)`. If `x` is a thenable, it attempts to make `promise` adopt the state of `x`, under the assumption that `x` behaves at least somewhat like a promise. Otherwise, it fulfills `promise` with the value `x`.

This treatment of thenables allows promise implementations to interoperate, as long as they expose a Promises/A+-compliant `then` method. It also allows Promises/A+ implementations to "assimilate" nonconformant implementations with reasonable `then` methods.

To run `[[Resolve]](promise, x)`, perform the following steps:

1. If `promise` and `x` refer to the same object, reject `promise` with a `TypeError` as the reason.
1. If `x` is a promise, adopt its state [[3.4](#notes)]:
   1. If `x` is pending, `promise` must remain pending until `x` is fulfilled or rejected.
   1. If/when `x` is fulfilled, fulfill `promise` with the same value.
   1. If/when `x` is rejected, reject `promise` with the same reason.
1. Otherwise, if `x` is an object or function,
   1. Let `then` be `x.then`. [[3.5](#notes)]
   1. If retrieving the property `x.then` results in a thrown exception `e`, reject `promise` with `e` as the reason.
   1. If `then` is a function, call it with `x` as `this`, first argument `resolvePromise`, and second argument `rejectPromise`, where:
      1. If/when `resolvePromise` is called with a value `y`, run `[[Resolve]](promise, y)`.
      1. If/when `rejectPromise` is called with a reason `r`, reject `promise` with `r`.
      1. If both `resolvePromise` and `rejectPromise` are called, or multiple calls to the same argument are made, the first call takes precedence, and any further calls are ignored.
      1. If calling `then` throws an exception `e`,
         1. If `resolvePromise` or `rejectPromise` have been called, ignore it.
         1. Otherwise, reject `promise` with `e` as the reason.
   1. If `then` is not a function, fulfill `promise` with `x`.
1. If `x` is not an object or function, fulfill `promise` with `x`.

If a promise is resolved with a thenable that participates in a circular thenable chain, such that the recursive nature of `[[Resolve]](promise, thenable)` eventually causes `[[Resolve]](promise, thenable)` to be called again, following the above algorithm will lead to infinite recursion. Implementations are encouraged, but not required, to detect such recursion and reject `promise` with an informative `TypeError` as the reason. [[3.6](#notes)]

## Notes

1. Here "platform code" means engine, environment, and promise implementation code. In practice, this requirement ensures that `onFulfilled` and `onRejected` execute asynchronously, after the event loop turn in which `then` is called, and with a fresh stack. This can be implemented with either a "macro-task" mechanism such as [`setTimeout`](https://html.spec.whatwg.org/multipage/webappapis.html#timers) or [`setImmediate`](https://dvcs.w3.org/hg/webperf/raw-file/tip/specs/setImmediate/Overview.html#processingmodel), or with a "micro-task" mechanism such as [`MutationObserver`](https://dom.spec.whatwg.org/#interface-mutationobserver) or [`process.nextTick`](http://nodejs.org/api/process.html#process_process_nexttick_callback). Since the promise implementation is considered platform code, it may itself contain a task-scheduling queue or "trampoline" in which the handlers are called.

1. That is, in strict mode `this` will be `undefined` inside of them; in sloppy mode, it will be the global object.

1. Implementations may allow `promise2 === promise1`, provided the implementation meets all requirements. Each implementation should document whether it can produce `promise2 === promise1` and under what conditions.

1. Generally, it will only be known that `x` is a true promise if it comes from the current implementation. This clause allows the use of implementation-specific means to adopt the state of known-conformant promises.

1. This procedure of first storing a reference to `x.then`, then testing that reference, and then calling that reference, avoids multiple accesses to the `x.then` property. Such precautions are important for ensuring consistency in the face of an accessor property, whose value could change between retrievals.

1. Implementations should *not* set arbitrary limits on the depth of thenable chains, and assume that beyond that arbitrary limit the recursion will be infinite. Only true cycles should lead to a `TypeError`; if an infinite chain of distinct thenables is encountered, recursing forever is the correct behavior.

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
