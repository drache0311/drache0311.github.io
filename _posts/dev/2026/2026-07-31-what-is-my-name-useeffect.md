---
title: "What is my name? - useEffect"
search: false
tags: [name function, useEffect]
last_modified_at: 2026-07-31T19:41:00
---

```
Error: something wrong in anonymous function
    at (anonymous) /workspace/Test.tsx:10:13
```
이런 에러 로그를 보신 적 있으신가요? 보고 어떤 생각이 드셨나요? 저는 에러에 대한 명확한 위치나 설명이 표현되어 있지 않아 불편했던 기억이 있습니다.
로그를 보고 해당 라인을 찾아가서 주변 코드들을 보고 원인을 파악하는데 시간을 썼습니다. 여기까지는 늘 하던 일이니까 아무런 의구심이 없었습니다.
하지만 어느날 한 칼럼의 글을 읽게 되었고 이것조차 개선할 방법이 있다는 것을 알게되었습니다. 그리고 지금은 그 효과를 충분히 누리고 있다고 생각합니다.

이 에러 로그에서 느꼈던 불편한 점을 알려면 실제 코드와 함께 봐야합니다.

```typescript
useEffect(() => {
    //...
  throw new Error('something wrong in anonymous function')
    //...
}, [any_dependencies])
```

저 로그를 통해 실제 코드를 보았을 때, 만약 이런 형태의 코드가 있다면 저는 한숨부터 나왔습니다. 이 effect(또는 익명 함수)가 어떤 일을 하는지부터 알아내야 하는데 시간을
써야하고, 왜 effect 를 사용했었을까도 고민하게 합니다. 그런 다음 에러가 발생한 이유를 찾고 수정하려고 합니다. 제가 로그를 보고 불편했던 점은 바로
이 부분 때문이었습니다. 그동안은 늘 과거의 나를 반성하면서 하던 일을 이어갔지만, 이제는 다릅니다.

```typescript
useEffect(function connectSSE()  {
    //...
  throw new Error('something wrong in connecting sse function')
    //...
}, [any_dependencies])
```

차이점이 보이시나요? 외형적으로는 arrow function 에서 named function 으로 변경되었습니다. 그렇다면 어떤 변화점을 가져올까요?
크게 두 가지 변화가 있습니다. 하나는 읽는 사람의 입장에서 에러가 있는 코드의 함수가 무엇을 하는지 이름을 통해 알기가 쉽습니다. 또 하나는 테스트 또는
디버깅시에 바뀌는 에러 로그입니다.

```
Error: something wrong in connecting sse function
    at connectSSE /workspace/Test.tsx:10:13
```

로그에서 at 다음 부분에 함수 이름이 추가되었습니다. 이전에는 빈칸 또는 anonymous 이라는 형태로 많이 보셨을 겁니다. 이제는 함수의 이름이
나옵니다. 우리는 에러 로그만 봐도 시스템이 어떤 일을 하다가 에러가 났는지 더욱 판단이 빨라지고, 절약한 시간만큼 대응도 빨라집니다.
저는 이 이점이 런타임 시점이 아닌 테스트 시점에 더욱 빛을 발한다고 생각합니다. 로그는 테스트시에도 동일하게 나타나며 우리는 서비스중인 시스템에서 나오는
에러보다 테스트에서 보는 에러가 더 많을 것입니다. 또한 처음의 익명함수로 나오는 것보다 이름이 있는 함수로 나오는 것이 코드를 구현하는데 있어 더 도움되리라
생각합니다.

```typescript
// anonymous arrow (what everyone writes)
useEffect(() => {
  document.title = `${count} items`;
}, [count]);

// named function expression (what I'm arguing for)
useEffect(function updateDocumentTitle() {
  document.title = `${count} items`;
}, [count]);
```

더불어 하나 더 있습니다. 불필요한 useEffect 를 찾아낼 수 있거나, 결합도가 높은 useEffect 를 찾아내는데 도움이 될 수 있습니다. 

