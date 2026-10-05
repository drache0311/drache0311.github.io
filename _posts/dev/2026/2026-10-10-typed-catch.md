---
title: "typed catch"
search: false
tags: [error, type, catch]
last_modified_at: 2026-10-10T18:18:00
---

[get-a-catch-block-error-message-with-typescript](https://kentcdodds.com/blog/get-a-catch-block-error-message-with-typescript)를 읽고 작성한 문서입니다.

위 글의 핵심 내용은 catch에서 발생한 값의 타입을 무조건 Error라고 가정하지 말고, unknown으로 취급한 뒤 타입을 확인하여 처리하라는 것입니다.

개발을 하면서 프로젝트의 틀을 정리하는 상황을 마주쳤을 때, 좋은 레퍼런스를 먼저 찾아보는 습관을 들여야 하는데 매 번 많은 시간을 들여 고민하고 결론한 답으로 해결하고는 합니다.
이런 방식은 많은 방법을 알고 있고, 그에 따른 장단점을 파악하고 있을 경우 효과적이겠지만 아직 저에게는 향후 생길 리팩토링, 단점들이 명확히 보이진 않아 나중에 후회하며 고치는 경우가 많습니다.
현재 프로젝트에서도 약 1년전 발생하는 fe,be 에러들을 정리할 일이 생겼습니다. 주 내용은 에러가 발생하면 fe 에서 어떻게 핸들링할 것인지 였습니다.
그래도 이번의 경우에는 우연치않게 좋은 길로 걸어나간 것 같습니다. 예전에 했던 작업들이 정말로 효과적이고, 신뢰성이 있는지 확인하는 작업은 소소한 재미인 듯 합니다.

### Error Integration

원 글에서는 에러의 타입을 unknown으로 취급하고, 타입을 확인하여 처리하는 방법을 소개했습니다.
그렇다면, 그 `처리`를 어디서 하면 될까요?
저의 경우, api 에서 발생하는 에러인 경우는 `interceptor` 에서 catch 하여 정의한 `ErrorType` 으로 타입을 변경시켰고, 그 외의 에러는 `ErrorBoundary` 에서 catch 하여 `ErrorType` 타입으로 변경시켰습니다.

