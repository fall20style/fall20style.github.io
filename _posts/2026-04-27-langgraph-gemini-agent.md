---
title: "LangGraph.js와 Gemini로 간단한 AI 에이전트 만들기"
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - LangChain
  - LangGraph
  - TypeScript
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

LangGraph.js와 Google의 Gemini 3 Flash 모델을 사용하여 상태 기반(Stateful) AI 에이전트를 구축

## 1. LangGraph.js란?

LangGraph는 대화의 흐름을 그래프로 정의할 수 있게 해주는 도구

기존의 단순한 체인(Chain) 구조와 달리, 순환(Loop)이나 복잡한 분기 로직을 직관적으로 관리

상태(State) 관리에 강점이 있습니다.

## 2. 프로젝트 설정

먼저 필요한 패키지를 설치하고 Google AI API 키를 설정.

```bash
npm install @langchain/google-genai @langchain/langgraph @langchain/core
export GOOGLE_API_KEY='your-api-key'
```

## 3. 핵심 코드 구현

로직은 `index.ts`에서 다음과 같이 정의.

### 상태(State)와 모델 정의

`Annotation.Root`를 통해 대화 기록을 유지하는 리듀서 패턴을 적용합니다.

```typescript
const GraphState = Annotation.Root({
  messages: Annotation<BaseMessage[]>({
    reducer: (x, y) => x.concat(y),
  }),
});

const model = new ChatGoogleGenerativeAI({
  model: "gemini-3-flash-preview",
});
```

### 그래프 구조 설계

워크플로우를 정의하고 노드 간의 관계를 연결.

```typescript
const workflow = new StateGraph(GraphState)
  .addNode("agent", callModel)
  .addEdge(START, "agent")
  .addEdge("agent", END);

const app = workflow.compile();
```

## 4. 실행 결과

에이전트에게 "LangGraph와 Gemini의 장점이 뭐야?"라고 질문하면, Gemini 모델이 현재 상태(메시지 기록)를 파악하여 답변을 생성.

```bash
npx tsx index.ts
```

## 마무리

LangGraph.js를 사용하면 AI 에이전트의 동작 방식을 그래프로 시각화하고 제어하기가 훨씬 수월.

특히 Gemini 3 Flash의 빠른 속도와 결합했을 때 더욱 강력한 실시간 인터랙션을 제공할 수 있음.
