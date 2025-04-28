---
title: "Why? When? use reducer instead of state?"
search: false
tags: [react, state, reducer]
last_modified_at: 2025-04-28T18:10:00
---
React 를 사용하면서 useState 는 당연하다시피 사용하고 필요를 인지하며 사용하고 있습니다.
그에 비해, useReducer 는 사용해본 경험이 많이 적은데요. 이는 reducer 의 활용방법과
장점을 모르기 때문이라고 생각합니다. useState 가 useReducer 로 만든 훅인것은 알고 계셨냐요?
useState 또한 useReducer 의 한가지 방법일 뿐 더 다양하고 효율적인 방법으로 사용할 수 있습니다.
이번 글에서는 그 내용에 대해 나누고자 합니다.

### Should I useState or useReducer? 

그렇다면 우선, useReducer 를 고민할 때는 언제가 있을까요?
저는 보통 이런 경우에 고민을 합니다.

`state 의 변경에 의해 일어나는 복잡성 해소`

어떤 state 가 변경되며, 어떤 특정한 값일 때는 A logic, 어떤 값일 때는 B logic 등등...
이것들이 간단하다면 고민이 없겠지만, 기능이 점점 추가된다면요? A 와 B 를 읽어내는 시간이 점차 늘어나며
이 state 의 변경을 찾아 모두 반영해주기도 해야합니다.

대표적으로 저는 이런 경우에 useState 에서의 useReducer 로 변경을 자주 하는데요.
하지만 이것은 주관적이며 협업을 하는 과정에서 다른 사람과의 공감을 얻기도 힘듭니다.

이 관점에 대해 제가 좋아하는 [Kent.C](https://kentcdodds.com) 와 [TkDodo](https://tkdodo.eu/blog) 는 어떻게 생각하는지 찾아보았습니다.
둘은 각각 일종의 규칙(NOT ESLINT RULES)을 정의했습니다.

Kent.C :
- When it's just an independent element of state you're managing: useState
- When one element of your state relies on the value of another element of your state in order to update: useReducer

TkDodo :
- if state updates independently - separate useStates
- for state that updates together, or only one field at a time updates - a single useState object
- for state where user interactions update different parts of the state - useReducer

종합해보면 state 가 독립적인 변경 또는 같이 변경되는 경우 => useState, 다른 상태(user interactions..)에 의존하여 변경되는 경우 => useReducer
로 서로의 규칙이 동일한 것으로 보입니다.
둘이 정의한 state 의 사용조건을 보니 react 공식 문서에서 정의한 그대로인데, 복잡하게 생각하려했던 저의 사고방식을 조금 더 유연하게 해야겠습니다.
그 조건들을 지우다보면 자연스레 reducer 의 사용조건들이 남게 되는 것 같네요.

어쨌거나 useReducer 를 선택하는 조건은 공통적으로 `다른 상태(user interactions..)에 의존하여 변경되는 경우`입니다.

### Is it mandatory?

하지만 위와 같은 상황이라도 useReducer 를 사용하는게 완전한 정답일까요? 여기에 대해서도 위의 둘의 의견을 살펴보면 재미있는 점이 있습니다.
전에 [reduce](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) 를 찾아보았을 때에도,
TkDodo 는 reduce 는 복잡하며 읽기 어려우니, 반복문을 사용하는 것을 선호한다고 했는데요.
이번 reducer 에서도 마찬가지로 정말 복잡한 state 일때만 사용하는 것을 권장하고 있습니다.🤣

`I believe useReducer is still heavily underused. The main thinking around useReducer seems to be that you only need it for "complex state".`

### Tips
reducer 에 대한 여러가지 팁도 있습니다.

#### event driven reducers
밑의 두 예제는 동일하게 동작하지만, dumb-reducer 는 확장성이 뛰어나지 않습니다. 그래서 일반적으로는 액션 이름에 'set'이 포함된 액션은 피하는 것이 좋습니다.
액션을 이벤트로 모델링하는 것은 특히 강조하고 싶은 부분입니다. 왜냐하면 이것이 리듀서의 가장 큰 장점 중 하나이기 때문입니다.
그렇게 하면 애플리케이션 로직을 UI의 여러 부분에 분산시키는 대신, 모든 로직을 리듀서 안에 담을 수 있습니다.
이렇게 하면 상태 전환에 대해 더 쉽게 이해할 수 있을 뿐만 아니라, 로직을 테스트하기도 훨씬 쉬워집니다(사실, 순수 함수는 테스트하기 가장 쉽습니다)
이것은 순수 컴포넌트와도 비슷하네요.
```typescript 
// 🚨-dumb-reducer
const reducer = (state, action) => {
  switch (action.type) {
    // 🚨 dumb reducer that doesn't do anything, logic is in the ui
    case 'set':
      return action.value
  }
}

const App = () => {
  const [count, dispatch] = React.useReducer(reducer, 0)

  return (
    <div>
      Count: {count}
      <button onClick={() => dispatch({ type: 'set', value: count + 1 })}>
        Increment
      </button>
      <button onClick={() => dispatch({ type: 'set', value: count - 1 })}>
        Decrement
      </button>
    </div>
  )
}
```

```typescript
// event-driven-reducer
const reducer = (state, action) => {
  // ✅ ui only dispatches events, logic is in the reducer
  switch (action) {
    case 'increment':
      return state + 1
    case 'decrement':
      return state - 1
  }
}

const App = () => {
  const [count, dispatch] = React.useReducer(reducer, 0)

  return (
    <div>
      Count: {count}
      <button onClick={() => dispatch('increment')}>Increment</button>
      <button onClick={() => dispatch('decrement')}>Decrement</button>
    </div>
  )
}
```
#### passing props to reducers

리듀서의 또 다른 훌륭한 특성은, 그것들을 인라인으로 작성하거나 프로퍼티를 [클로저](https://tkdodo.eu/blog/hooks-dependencies-and-stale-closures#what-are-closures)(closure)로 캡처할 수 있다는 점입니다.
```typescript
const reducer = (data) => (state, action) => {
  // ✅ you'll always have access to the latest
  // server state in here
}

const App = () => {
  const { data } = useQuery(key, queryFn)
  const [state, dispatch] = React.useReducer(reducer(data))
}
```
위와 같은 방식은 서버 상태와 클라이언트 상태를 분리하는 개념과 잘 맞아떨어집니다.

#### articles
useReducer 에 관한 좋은 글들도 추천드립니다 :

[Do Not Mutate State](https://redux.js.org/style-guide/style-guide#do-not-mutate-state)

[Reducers Must Not Have Side Effects](https://redux.js.org/style-guide/style-guide#reducers-must-not-have-side-effects)

[Model Actions as Events, not Setters](https://redux.js.org/style-guide/style-guide#model-actions-as-events-not-setters)


### Conclusion
useReducer 에 대해 궁금했던 언제, 왜 사용하는지를 알아보았는데요.
머리속으로는 이해했지만 실 상황에서 훅/컴포넌트에 reducer 기반으로 작성하는 것은 많은 경험이 뒷받침되어야 할 것 같습니다.
그래도 간단한 toggle, conditional rendering, validation 등 기존의 Frontiers 에서 복잡한 구조로 되어있던 state 변경관리들이
useReducer 를 통해서 어느정도 해소할 수 있어 보여 좋은 시간이 되었습니다.

마지막으로 Kent.C 의 말을 빌려 마무리 하겠습니다.
```
useState, useReducer 는 상황에 따라(case-by-case) 달라진다는 점에 주의하세요.
useReducer와 useState를 함께 사용하는 것이나, 여러 개의 useState와 여러 개의 useReducer를 동시에 사용하는 것도 가능합니다.
"도메인"에 따라 논리적으로 상태를 구분하세요.
만약 여러 상태가 함께 변경된다면, reducer 안에 함께 묶어 관리하는 것이 좋습니다.
반대로 어떤 상태가 다른 상태들과 크게 독립적이라면, 그 상태를 다른 것들과 함께 묶는 것은 리듀서에 불필요한 복잡성과 잡음만 추가하는 것이므로, 독립적으로 따로 관리하는 편이 낫습니다.

그러니까, 단순히 "useState가 X개 이상이면 useReducer로 바꾼다" 같은 규칙은 아니라는 것입니다.
그보다는 더 미묘하고 섬세한 판단이 필요합니다.
하지만 이 글을 통해 그 미묘한 차이를 이해하고, 당신의 상황에 가장 적합한 도구를 선택하는 데 도움이 되기를 바랍니다.

일반적으로는, 처음에는 useState로 시작하는 것을 추천하고, 상태 요소들이 함께 변화해야 할 필요가 느껴질 때 useReducer로 넘어가는 것을 권장합니다.
```

감사합니다.
