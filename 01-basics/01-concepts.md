# 1-1. Vue vs React 기본 개념

React 개발자로서 Vue를 배우는 것은 생각보다 어렵지 않습니다! 두 프레임워크 모두 **컴포넌트 기반**이고 **선언적 패러다임**을 따르거든요.

## 🎯 핵심 차이점

### 1. 컴포넌트 구조

**React (JSX)**
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

**Vue (SFC - Single File Component)**
```vue
<!-- Vue는 HTML, JS, CSS가 하나의 파일에! -->
<template>
  <div class="counter">
    <h1>Count: {{ count }}</h1>
    <button @click="count++">증가</button>
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)
    return { count }
  }
}
</script>

<style scoped>
.counter {
  padding: 20px;
  text-align: center;
}
/* scoped 스타일: 이 컴포넌트에만 적용! */
</style>
```

### 2. 상태 관리

**React의 useState**
```jsx
const [message, setMessage] = useState('안녕하세요');
const [user, setUser] = useState({ name: '김개발', age: 30 });

// 상태 업데이트
setMessage('새 메시지');
setUser({ ...user, age: 31 }); // 불변성 유지 필요
```

**Vue의 ref와 reactive**
```vue
<script>
import { ref, reactive } from 'vue'

export default {
  setup() {
    // ref: 원시값을 위한 반응형
    const message = ref('안녕하세요');
    
    // reactive: 객체를 위한 반응형
    const user = reactive({ name: '김개발', age: 30 });

    // 상태 업데이트 (더 직관적!)
    message.value = '새 메시지';
    user.age = 31; // 직접 변경 가능!

    return { message, user }
  }
}
</script>
```

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

2. **React의 useState ≈ Vue의 ref/reactive**
   - Vue는 직접 변경 가능, React는 불변성 필요

3. **React의 useEffect ≈ Vue의 watch/watchEffect**
   - 다음 장에서 자세히 다룰 예정

4. **React의 props ≈ Vue의 props**
   - 거의 동일한 개념

## ➡️ 다음 단계

기본 개념을 이해했다면 [1-2. 템플릿 문법](./02-template-syntax.md)으로 넘어가서 Vue 템플릿의 강력한 기능들을 알아보세요!

---

*React 개발자라면 Vue의 문법이 더 직관적이라고 느낄 거예요. 특히 템플릿 부분은 HTML을 아는 사람이라면 누구나 쉽게 이해할 수 있거든요! 🎉* 