# Vue.js 완전 학습 가이드 🚀

React 개발자를 위한 Vue.js 기초부터 심화까지의 완전한 학습 자료입니다.

## 📚 학습 목차

### 🌱 1단계: 기초 (Basics)
- [1-1. Vue vs React 기본 개념](./01-basics/01-concepts.md)
- [1-2. 템플릿 문법](./01-basics/02-template-syntax.md)
- [1-3. 데이터 바인딩](./01-basics/03-data-binding.md)
- [1-4. 이벤트 핸들링](./01-basics/04-event-handling.md)
- [1-5. 컴포넌트 기초](./01-basics/05-components-basic.md)

### 🌿 2단계: 중급 (Intermediate)
- [2-1. 컴포넌트 간 통신](./02-intermediate/01-component-communication.md)
- [2-2. 조건부 렌더링과 리스트](./02-intermediate/02-conditional-lists.md)
- [2-3. 폼 처리](./02-intermediate/03-forms.md)
- [2-4. 라이프사이클](./02-intermediate/04-lifecycle.md)
- [2-5. 커스텀 디렉티브](./02-intermediate/05-directives.md)

### 🌳 3단계: 고급 (Advanced)
- [3-1. Composition API 심화](./03-advanced/01-composition-api.md)
- [3-2. Composables (재사용 로직)](./03-advanced/02-composables.md)
- [3-3. Pinia (상태 관리)](./03-advanced/03-pinia.md)
- [3-4. Vue Router (라우팅)](./03-advanced/04-vue-router.md)
- [3-5. 성능 최적화](./03-advanced/05-performance.md)

### 🚀 4단계: 실전 프로젝트
- [4-1. Todo 앱 (기초)](./04-projects/01-todo-app.md)
- [4-2. 블로그 시스템 (중급)](./04-projects/02-blog-system.md)
- [4-3. 실시간 채팅앱 (고급)](./04-projects/03-realtime-chat.md)

## 💡 React 개발자를 위한 빠른 비교

| 개념 | React | Vue |
|------|-------|-----|
| **컴포넌트 구조** | JSX | Template + Script + Style |
| **상태 관리** | useState | ref, reactive |
| **사이드 이펙트** | useEffect | watchEffect, watch |
| **계산된 값** | useMemo | computed |
| **조건부 렌더링** | {condition && <div>} | v-if, v-show |
| **리스트 렌더링** | map() | v-for |
| **이벤트 처리** | onClick={handler} | @click="handler" |
| **양방향 바인딩** | value + onChange | v-model |
| **Props** | props | props |
| **컨텐츠 투영** | children | slots |
| **컴포넌트 통신** | props + callbacks | props + emit |
| **전역 상태** | Context API, Redux | Pinia, Vuex |
| **라우팅** | React Router | Vue Router |
| **재사용 로직** | Custom Hooks | Composables |

## 🛠️ 개발 환경 설정

### 1. Node.js 설치
```bash
# Node.js 18+ 버전 필요
node --version
npm --version
```

### 2. Vue 프로젝트 생성
```bash
# Vue CLI 설치
npm install -g @vue/cli

# 새 프로젝트 생성
vue create my-vue-app

# 또는 Vite 사용 (더 빠름)
npm create vue@latest my-vue-app
```

### 3. 개발 서버 실행
```bash
cd my-vue-app
npm run dev
```

## 📖 추천 학습 순서

1. **기초 완주** (1-2주): Vue의 기본 개념과 템플릿 문법 익히기
2. **중급 연습** (2-3주): 컴포넌트 간 통신과 라이프사이클 이해
3. **고급 심화** (3-4주): Composition API와 상태관리 마스터
4. **실전 프로젝트** (4-6주): 실제 애플리케이션 개발

## 🎯 학습 목표

### 기초 과정 완료 후
- ✅ Vue와 React의 차이점을 명확히 이해
- ✅ Vue 템플릿 문법을 자유롭게 사용
- ✅ 컴포넌트 기반 개발 가능
- ✅ 이벤트 핸들링과 데이터 바인딩 마스터

### 중급 과정 완료 후
- ✅ 복잡한 컴포넌트 간 통신 구현
- ✅ 동적 UI 제작 (조건부 렌더링, 리스트)
- ✅ 폼 처리와 유효성 검증
- ✅ 라이프사이클을 활용한 최적화

### 고급 과정 완료 후
- ✅ Composition API로 현대적인 Vue 개발
- ✅ 재사용 가능한 Composables 작성
- ✅ Pinia를 활용한 상태 관리
- ✅ Vue Router로 SPA 개발
- ✅ 성능 최적화 기법 적용

## 🔧 필수 도구

### VS Code 확장 프로그램
- **Vetur** 또는 **Volar**: Vue 파일 지원
- **Vue VSCode Snippets**: 코드 스니펫
- **Auto Rename Tag**: 태그 자동 수정
- **Bracket Pair Colorizer**: 괄호 색상 구분

### 브라우저 개발도구
- **Vue.js devtools**: Vue 컴포넌트 디버깅

### 패키지 관리자
- **npm** 또는 **yarn** 또는 **pnpm**

## 💬 학습 팁

1. **React와 비교하며 학습**: 각 개념을 React와 비교해서 이해하면 더 빠르게 습득 가능
2. **실습 위주**: 문서만 읽지 말고 직접 코드를 작성해보기
3. **공식 문서 활용**: [Vue.js 공식 문서](https://vuejs.org/)는 정말 잘 되어 있음
4. **커뮤니티 참여**: Vue 한국 사용자 그룹에서 질문하고 답변하기
5. **점진적 학습**: Options API → Composition API 순서로 학습

## 🎉 시작하기

준비되셨나요? [1-1. Vue vs React 기본 개념](./01-basics/01-concepts.md)부터 시작해보세요!

---

*React 개발자라면 Vue를 배우는 것이 생각보다 어렵지 않을 거예요. 두 프레임워크 모두 컴포넌트 기반이고 선언적 패러다임을 따르니까요! 💪* 