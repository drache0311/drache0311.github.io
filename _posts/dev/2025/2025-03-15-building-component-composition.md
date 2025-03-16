---
title: "Building component composition"
search: false
tags: [react, component]
last_modified_at: 2025-03-16T23:51:00
---
# Thinking in React
먼저, 당신이 리액트를 사용하는 서비스를 개발중이라면 한가지 묻고 싶습니다. 리액트스럽게 개발을 하고 계신가요?
제 대답은 `노력중` 이고 얼마전까지는 정확히 `아니요` 였습니다. 그 변화의 영향 중 가장 큰 요소로써 이 글을 통해 리액트를 처음 시작하는 또는 리액트를 
javascript markup 용도로만 사용하시는 분들께 도움이 되었으면 좋겠습니다.

리액트스럽다? 리액트답게? 라는건 어떤 의미일까요? 저는 가장 먼저 `컴포넌트`를 기반한 구조가 떠오릅니다.
컴포넌트에 화면에 보이는 `state`를 정의하고, 그 컴포넌트들을 연결하며 데이터의 흐름을 만들어 낼 것입니다.
이 작업은 단순히 화면구성, 즉 레이아웃을 만드는 것에 그치지 않습니다. 이후의 확장성, 테스트, 렌더링 등 모든 요소에 영향을 미칠 것입니다.
하지만 저는 이것을 알아차리기에 오랜 시간이 걸렸습니다. 반복적인 코드를 줄이기 위하거나 기능을 나누거나 그때마다의 느낌으로 필요성이 느껴질 때마다 컴포넌트를 만들어
사용했습니다. 이것이 점점 누적되며 코드의 유지보수, 테스트 코드의 작성 등에 어려움이 생겼지만 처음엔 문제가 무엇인지 인지하는 것조차 쉽지 않았습니다. 리액트를 간접적으로만 배움으로써
본질을 이해하지 못 한점이 크다고 생각하여 리액트 공식 문서부터 읽기 시작했고 이제는 리액트스러운 생각을 가지고 개발을 하고자 하고 있습니다.

컴포넌트 구성은 리액트 기능에 가장 많은 영향을 받거나 또는 주는 것이라고 해도 과언이 아닙니다.
그럼에도, 익숙해짐에 따라 가장 놓치기 쉬운 것이기도 합니다.

관심사 분리는요? 여전히 관심사를 분리하지만 컴포넌트 구성은 이전과는 다른 방식으로 분리합니다.
[Component-composition-is-great-btw](https://tkdodo.eu/blog/component-composition-is-great-btw)의 글에서 소개한 트윗의 그림을 보면 컴포넌트 구성의 관심사 분리가 어떻게 되고 있는지 이해하기 쉽습니다. 

<a href="https://x.com/mxstbr/status/993455977008594944" target="_blank" align="left">
    <img  width="60%" src="../../static/images/2025/03-15/concerns.png" alt="concerns"/>
</a>

이 말은 곧 `코드 응집력`을 의미합니다. 버튼의 기능(클릭에 관한)이나 버튼의 스타일, 마크업이 한 곳에 모여 이루어집니다.
이것은 한 레이어에 모든 코드를 담은 것보다 컴포넌트 별로 묶는 것이 역시 더 좋습니다. 

그렇다해도 컴포넌트를 언제, 어떻게, 어떤 기준으로 만드는지에 대한건 여전히 어렵습니다.
정말 감사하게도, 컴포넌트를 정의하고 설계하는 방법을 [리액트 공식 문서](https://react.dev/learn/thinking-in-react)에서도 제공하고 있기에 꼭 읽어보시기를 추천드립니다.  
개발을 진행하기에 앞서 먼저 컴포넌트 구성을 생각한다면, 이후의 코드를 더 리액트스럽게 개발할 수 있을 것입니다. 

<p align="center">
    <img src="../../static/images/2025/03-15/thinking-in-react_ui.png" alt="thinking-in-react_ui"/><br/>
    <em>https://react.dev/learn/thinking-in-react</em>
</p>

위에서 소개한 [Tkdodo](https://tkdodo.eu/blog/component-composition-is-great-btw#conditional-rendering) 글에 컴포넌트 구성의 이점을 활용함에 마주치는 어려움도 소개하고 있습니다.
이 문제는 점차 어플리케이션의 컴포넌트 구성을 멈추게하고 지속하기에 어려움을 줍니다.
바로 `Conditional Rendering` 인데, 이것을 해소하는 방법은 글에서 자세히 설명하고 있으니 보시는 것을 추천드립니다.

# Component for props
하지만 Layout 을 잡아도 해당 컴포넌트의 children 은 어떻게 관리해야 더 좋을지 고민이 있었습니다. 밑의 테이블 예제로 레이아웃을 잡은 컴포넌트 뿐만이 아닌
자식 컴포넌트의 관리까지 다루는 방법을 알아보겠습니다.

```tsx
const Component = () => {
    return (
        <table>
            <thead>
                <tr>
                    <th>Header1</th>
                    <th>Header2</th>
                    <th>Header3</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>Item1</td>
                    <td>Item2</td>
                    <td>Item3</td>
                </tr>
            </tbody>
        </table>
    )
}
```
이것도 역시 나쁘지는 않습니다. 여기에 추가로, 헤더에는 Hello 가 앞에 붙으며, 각각에 클래스를 주고 싶다면요?
우리는 레이아웃 컴포넌트의 이점을 알고 있으므로 이 테이블을 레이아웃화 하여 해보겠습니다.
```tsx
const Component = () => {
    return (
        <Table>
            <TableHeader>
                <TableRow>
                    <th className="header">Hello Header1</th>
                    <th className="header middle">Hello Header2</th>
                    <th className="header">Hello Header3</th>
                </TableRow>
            </TableHeader>
            <TableBody>
                <TableRow>
                    <td className="body">Item1</td>
                    <td className="body">Item2</td>
                    <td className="body">Item3</td>
                </TableRow>
            </TableBody>
        </Table>
    )
}
```
준비는 된 것 같아 보입니다! 하지만 아직 컴포넌트를 더 진화시킬 수 있을 것 같아 보입니다. 아직은 읽기 쉽고, 변경이 쉬워보이진 않습니다.
로우가 가진 자식 컴포넌트들에 동일한 속성이나, 값을 부여하는 등 어떤 그룹화가 목적이라면 props로 해결해보는건 어떨까요?
```tsx
const Component = () => {
    return (
        <Table>
            <TableHeader>
                <TableRow
                    className="header"
                    prefix="Hello"
                    group={[
                        <th>Header1</th>,
                        <th className="middle">Header2</th>,
                        <th>Header3</th>       
                    ]}
                />
            </TableHeader>
            <TableBody>
                <TableRow
                    className="body"
                    group={[
                        <td>Item1</td>,
                        <td>Item2</td>,
                        <td>Item3</td>    
                    ]}
                />
            </TableBody>
        </Table>
    )
}
```
훨씬 좋아보입니다! 이제 공통된 요소는 TableRow 가 props 로 넘겨줄 수 있으며, 자식 컴포넌트의 개별 요소는 각각 가져가 관심사 분리도 되어 보입니다.
