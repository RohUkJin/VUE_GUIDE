# 1-1. Vue vs React 기본 개념

React 개발자로서 Vue를 배우는 것은 생각보다 어렵지 않습니다! 두 프레임워크 모두 **컴포넌트 기반**이고 **선언적 패러다임**을 따르거든요.

**🤔 왜 Vue를 배워야 할까요?**
- **더 빠른 개발**: 보일러플레이트가 적어서 같은 기능을 더 적은 코드로 구현
- **직관적인 문법**: HTML/CSS에 익숙하다면 바로 이해 가능
- **점진적 도입**: 기존 프로젝트에 부분적으로 적용 가능
- **뛰어난 성능**: 반응형 시스템이 매우 효율적

## 🎯 핵심 차이점

### 1. 컴포넌트 구조: JavaScript vs HTML-기반

React는 "**JavaScript 중심**"으로 모든 것을 JSX 안에서 처리하지만, Vue는 "**HTML 중심**"으로 웹 개발자에게 친숙한 방식을 제공합니다.

**React (JSX) - JavaScript가 중심**
```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(count + 1);
  };

  return (
    <div className="counter">
      <h1>Count: {count}</h1>
      <button onClick={handleClick}>증가</button>
    </div>
  );
}

export default Counter;
```

**🔍 React 방식의 특징:**
- 모든 것이 JavaScript 함수 안에 있음
- `className` 사용 (class는 JS 예약어)
- 이벤트 핸들러를 별도 함수로 정의 필요
- 상태 변경시 `setCount` 함수 호출 필요

**Vue (SFC - Single File Component) - HTML이 중심**
```vue
<!-- 📝 Template: HTML과 거의 동일한 문법 -->
<template>
  <div class="counter">
    <h1>Count: {{ count }}</h1>
    <button @click="count++">증가</button>
  </div>
</template>

<!-- 🧠 Script: 로직만 집중 -->
<script>
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)
    return { count } // 템플릿에서 사용할 것들을 return
  }
}
</script>

<!-- 🎨 Style: CSS가 분리되어 있어서 관리 쉬움 -->
<style scoped>
.counter {
  padding: 20px;
  text-align: center;
}
/* scoped: 이 컴포넌트에만 적용되는 마법! */
</style>
```

**🔍 Vue SFC의 특징:**
- **역할 분리**: HTML, JS, CSS가 명확하게 구분
- **HTML 친화적**: `class`, `@click` 등 자연스러운 문법
- **직접 변경**: `count++`로 바로 상태 변경 가능
- **스타일 격리**: `scoped` 속성으로 CSS 충돌 자동 방지

**💡 언제 어떤 방식이 좋을까요?**
- **React 방식**: JavaScript에 익숙하고, 모든 것을 코드로 제어하고 싶을 때
- **Vue 방식**: HTML/CSS에 익숙하고, 디자이너와 협업이 많을 때

### 2. 상태 관리: 불변성 vs 직접 변경

React는 **불변성**을 지켜야 하지만, Vue는 **직접 변경**이 가능해서 더 직관적입니다.

**React의 useState - 불변성 원칙**
```jsx
const [message, setMessage] = useState('안녕하세요');
const [user, setUser] = useState({ name: '김개발', age: 30 });

// 상태 업데이트 - 항상 새로운 값/객체를 만들어야 함
setMessage('새 메시지');
setUser({ ...user, age: 31 }); // 스프레드 연산자로 새 객체 생성

// ❌ 이렇게 하면 리렌더링 안됨!
// user.age = 31; (React는 참조가 같으면 변경을 감지 못함)
```

**🔍 React 방식의 특징:**
- **명시적 업데이트**: `setXxx` 함수로만 상태 변경 가능
- **불변성 필수**: 객체/배열 변경시 새로운 참조 생성 필요
- **예측 가능**: 언제 상태가 변경되는지 명확
- **디버깅 용이**: 상태 변경 추적이 쉬움

**Vue의 ref와 reactive - 반응형 시스템**
```vue
<script>
import { ref, reactive } from 'vue'

export default {
  setup() {
    // ref: 원시값(문자열, 숫자, 불린)을 반응형으로 만들기
    const message = ref('안녕하세요');
    
    // reactive: 객체/배열을 반응형으로 만들기
    const user = reactive({ 
      name: '김개발', 
      age: 30,
      hobbies: ['코딩', '게임']
    });

    // 상태 업데이트 - JavaScript처럼 자연스럽게!
    message.value = '새 메시지'; // ref는 .value로 접근
    user.age = 31; // reactive는 직접 변경
    user.hobbies.push('독서'); // 배열도 직접 조작 가능

    return { message, user }
  }
}
</script>
```

**🔍 Vue 방식의 특징:**
- **직접 변경**: JavaScript 객체 다루듯이 자연스럽게 변경
- **자동 감지**: Vue가 변경을 자동으로 감지하고 UI 업데이트
- **적은 코드**: 스프레드 연산자나 복잡한 로직 불필요
- **프록시 기반**: ES6 Proxy로 내부적으로 변경 감지

**🤔 언제 ref vs reactive를 사용할까요?**

```javascript
// ✅ ref 사용하기 좋은 경우
const count = ref(0);              // 숫자
const isLoading = ref(false);      // 불린
const userName = ref('');          // 문자열
const selectedItem = ref(null);    // 단일 객체 참조

// ✅ reactive 사용하기 좋은 경우
const form = reactive({            // 폼 데이터
  email: '',
  password: '',
  rememberMe: false
});

const userProfile = reactive({     // 복잡한 객체
  name: '김개발',
  contacts: {
    email: 'dev@example.com',
    phone: '010-1234-5678'
  },
  hobbies: ['코딩', '게임']
});
```

**⚠️ 주의사항:**
- `ref`는 템플릿에서 `.value` 자동 해제, 스크립트에서는 `.value` 필요
- `reactive`는 구조분해할당시 반응성 잃어버림 (해결법: `toRefs` 사용)

### 3. 템플릿 문법

**React (JSX)**
```jsx
function UserProfile({ user, isLoading }) {
  return (
    <div>
      {isLoading ? (
        <div>로딩 중...</div>
      ) : (
        <div>
          <h1>{user.name}</h1>
          {user.isOnline && <span className="online">온라인</span>}
          <ul>
            {user.hobbies.map(hobby => (
              <li key={hobby.id}>{hobby.name}</li>
            ))}
          </ul>
        </div>
      )}
    </div>
  );
}
```

**Vue (Template)**
```vue
<template>
  <div>
    <!-- 조건부 렌더링: v-if/v-else -->
    <div v-if="isLoading">로딩 중...</div>
    <div v-else>
      <h1>{{ user.name }}</h1>
      <!-- v-show: display none/block 토글 -->
      <span v-show="user.isOnline" class="online">온라인</span>
      
      <!-- 리스트 렌더링: v-for -->
      <ul>
        <li v-for="hobby in user.hobbies" :key="hobby.id">
          {{ hobby.name }}
        </li>
      </ul>
    </div>
  </div>
</template>
```

### 4. 이벤트 처리

**React**
```jsx
function LoginForm() {
  const [form, setForm] = useState({ email: '', password: '' });

  const handleSubmit = (e) => {
    e.preventDefault(); // 기본 동작 방지
    console.log('로그인 시도:', form);
  };

  const handleInputChange = (e) => {
    const { name, value } = e.target;
    setForm({ ...form, [name]: value });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="email"
        value={form.email}
        onChange={handleInputChange}
        placeholder="이메일"
      />
      <input
        name="password"
        type="password"
        value={form.password}
        onChange={handleInputChange}
        placeholder="비밀번호"
      />
      <button type="submit">로그인</button>
    </form>
  );
}
```

**Vue**
```vue
<template>
  <!-- .prevent: e.preventDefault() 자동 적용! -->
  <form @submit.prevent="handleSubmit">
    <!-- v-model: 양방향 바인딩으로 더 간단! -->
    <input
      v-model="form.email"
      placeholder="이메일"
    />
    <input
      v-model="form.password"
      type="password"
      placeholder="비밀번호"
    />
    <button type="submit">로그인</button>
  </form>
</template>

<script>
import { reactive } from 'vue'

export default {
  setup() {
    const form = reactive({
      email: '',
      password: ''
    });

    const handleSubmit = () => {
      console.log('로그인 시도:', form);
    };

    return { form, handleSubmit }
  }
}
</script>
```

## 📋 Vue의 두 가지 API 스타일

Vue는 같은 기능을 두 가지 방식으로 작성할 수 있어요:

### Options API (Vue 2 스타일)
```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p>클릭 수: {{ count }}</p>
    <button @click="increment">클릭!</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      title: 'Options API 예제',
      count: 0
    }
  },
  methods: {
    increment() {
      this.count++;
    }
  },
  computed: {
    doubleCount() {
      return this.count * 2;
    }
  },
  mounted() {
    console.log('컴포넌트가 마운트됨');
  }
}
</script>
```

### Composition API (Vue 3 권장, React Hooks와 유사)
```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p>클릭 수: {{ count }}</p>
    <button @click="increment">클릭!</button>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  setup() {
    const title = ref('Composition API 예제');
    const count = ref(0);

    const increment = () => {
      count.value++;
    };

    const doubleCount = computed(() => count.value * 2);

    onMounted(() => {
      console.log('컴포넌트가 마운트됨');
    });

    return {
      title,
      count,
      increment,
      doubleCount
    }
  }
}
</script>
```

## 🚀 Vue의 장점 (React 개발자 관점)

### 1. **더 적은 보일러플레이트**
- `useState`와 `useEffect` 없이도 반응형 시스템
- 양방향 바인딩으로 폼 처리가 간단

### 2. **직관적인 템플릿 문법**
- HTML에 가까운 문법
- 디자이너와의 협업이 쉬움
- 조건부 렌더링과 리스트가 더 명확

### 3. **스타일 스코핑**
- CSS-in-JS 없이도 컴포넌트 스타일 격리
- `<style scoped>` 자동으로 스타일 충돌 방지

### 4. **내장된 디렉티브**
- `v-if`, `v-for`, `v-model` 등 유용한 기능들
- 커스텀 디렉티브로 DOM 조작 쉽게

## 🎯 실습: HelloWorld 컴포넌트

직접 만들어보세요!

```vue
<template>
  <div class="hello-world">
    <h1>{{ greeting }}</h1>
    
    <div class="user-input">
      <input 
        v-model="name" 
        placeholder="이름을 입력하세요"
        @keyup.enter="addToList"
      />
      <button @click="addToList">추가</button>
    </div>

    <ul v-if="names.length > 0">
      <li v-for="(item, index) in names" :key="index">
        {{ item }}
        <button @click="removeFromList(index)">삭제</button>
      </li>
    </ul>
    
    <p v-else>아직 추가된 이름이 없습니다.</p>
  </div>
</template>

<script>
import { ref, computed } from 'vue'

export default {
  setup() {
    const name = ref('');
    const names = ref([]);

    const greeting = computed(() => 
      names.value.length > 0 
        ? `안녕하세요, ${names.value.join(', ')}님!`
        : '안녕하세요!'
    );

    const addToList = () => {
      if (name.value.trim()) {
        names.value.push(name.value.trim());
        name.value = '';
      }
    };

    const removeFromList = (index) => {
      names.value.splice(index, 1);
    };

    return {
      name,
      names,
      greeting,
      addToList,
      removeFromList
    }
  }
}
</script>

<style scoped>
.hello-world {
  max-width: 400px;
  margin: 50px auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.user-input {
  margin: 20px 0;
}

.user-input input {
  padding: 8px;
  margin-right: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.user-input button {
  padding: 8px 16px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  padding: 8px;
  margin: 4px 0;
  background: #f9f9f9;
  border-radius: 4px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

li button {
  background: #e74c3c;
  color: white;
  border: none;
  padding: 4px 8px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}
</style>
```

## 🔥 핵심 포인트

1. **React의 JSX ≈ Vue의 Template**
   - 둘 다 선언적, Vue가 더 HTML에 가까움
   - Vue는 디자이너와 협업이 더 쉬움

2. **React의 useState ≈ Vue의 ref/reactive**
   - Vue는 직접 변경 가능 (`user.age++`), React는 불변성 필요 (`setUser({...user, age: user.age + 1})`)
   - Vue는 보일러플레이트 코드가 훨씬 적음

3. **React의 useEffect ≈ Vue의 watch/watchEffect**
   - Vue는 의존성 자동 추적, React는 수동 의존성 배열 관리
   - 다음 장에서 자세히 다룰 예정

4. **React의 props ≈ Vue의 props**
   - 거의 동일한 개념, Vue는 타입 검증과 기본값이 더 편리

**💡 React에서 Vue로 마이그레이션하면?**
- ✅ **개발 속도 향상**: 보일러플레이트 50% 감소
- ✅ **학습 곡선 완화**: HTML/CSS 지식 활용 가능
- ✅ **디버깅 편의**: Vue DevTools가 매우 직관적
- ✅ **팀 협업**: 백엔드/디자이너도 쉽게 이해

## ➡️ 다음 단계

기본 개념을 이해했다면 [1-2. 템플릿 문법](./02-template-syntax.md)으로 넘어가서 Vue 템플릿의 강력한 기능들을 알아보세요!

---

*React 개발자라면 Vue의 문법이 더 직관적이라고 느낄 거예요. 특히 템플릿 부분은 HTML을 아는 사람이라면 누구나 쉽게 이해할 수 있거든요! 🎉* 