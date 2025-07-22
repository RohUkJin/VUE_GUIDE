# 1-2. 템플릿 문법

Vue의 템플릿 문법은 React의 JSX와 비교해서 훨씬 HTML에 가깝습니다! React 개발자라면 "어? 이게 더 직관적이네!"라고 느낄 거예요.

## 🎯 기본 템플릿 vs JSX

### 텍스트 보간 (Text Interpolation)

**React (JSX)**
```jsx
function Welcome({ name, age, isOnline }) {
  return (
    <div>
      <h1>안녕하세요, {name}님!</h1>
      <p>나이: {age}살</p>
      <p>상태: {isOnline ? '온라인' : '오프라인'}</p>
    </div>
  );
}
```

**Vue (Template)**
```vue
<template>
  <div>
    <!-- 이중 중괄호로 텍스트 보간 -->
    <h1>안녕하세요, {{ name }}님!</h1>
    <p>나이: {{ age }}살</p>
    <p>상태: {{ isOnline ? '온라인' : '오프라인' }}</p>
  </div>
</template>

<script>
export default {
  props: {
    name: String,
    age: Number,
    isOnline: Boolean
  }
}
</script>
```

## 🔗 속성 바인딩 (Attribute Binding)

### 기본 속성 바인딩

**React**
```jsx
function ImageCard({ src, alt, isLarge, className }) {
  return (
    <div className={`card ${className} ${isLarge ? 'large' : ''}`}>
      <img 
        src={src} 
        alt={alt}
        style={{ width: isLarge ? '300px' : '150px' }}
      />
    </div>
  );
}
```

**Vue**
```vue
<template>
  <div :class="['card', className, { large: isLarge }]">
    <!-- v-bind: 또는 : 단축문법 사용 -->
    <img 
      :src="src" 
      :alt="alt"
      :style="{ width: isLarge ? '300px' : '150px' }"
    />
  </div>
</template>

<script>
export default {
  props: {
    src: String,
    alt: String,
    isLarge: Boolean,
    className: String
  }
}
</script>
```

### 클래스와 스타일 바인딩

**React에서는 문자열 조작이나 라이브러리(classnames) 필요**
```jsx
import classNames from 'classnames';

function Button({ isActive, isPrimary, isDisabled, size }) {
  return (
    <button 
      className={classNames('btn', {
        'btn-active': isActive,
        'btn-primary': isPrimary,
        'btn-disabled': isDisabled,
        [`btn-${size}`]: size
      })}
      style={{
        fontSize: size === 'large' ? '18px' : '14px',
        backgroundColor: isPrimary ? '#007bff' : '#6c757d'
      }}
    >
      클릭하세요
    </button>
  );
}
```

**Vue는 객체/배열 방식으로 더 직관적**
```vue
<template>
  <!-- 클래스 바인딩: 객체 방식 -->
  <button 
    :class="{
      'btn': true,
      'btn-active': isActive,
      'btn-primary': isPrimary,
      'btn-disabled': isDisabled,
      [`btn-${size}`]: size
    }"
    :style="{
      fontSize: size === 'large' ? '18px' : '14px',
      backgroundColor: isPrimary ? '#007bff' : '#6c757d'
    }"
  >
    클릭하세요
  </button>

  <!-- 또는 배열 방식도 가능 -->
  <button 
    :class="[
      'btn',
      { 'btn-active': isActive },
      { 'btn-primary': isPrimary },
      `btn-${size}`
    ]"
  >
    버튼
  </button>
</template>

<script>
export default {
  props: {
    isActive: Boolean,
    isPrimary: Boolean,
    isDisabled: Boolean,
    size: String
  }
}
</script>
```

## 🔀 조건부 렌더링

### v-if vs React 조건부 렌더링

**React**
```jsx
function UserDashboard({ user, isLoading, error }) {
  return (
    <div>
      {isLoading && <div>로딩 중...</div>}
      
      {error && (
        <div className="error">
          <h3>오류 발생!</h3>
          <p>{error.message}</p>
        </div>
      )}
      
      {!isLoading && !error && user && (
        <div>
          <h1>환영합니다, {user.name}님!</h1>
          {user.isPremium ? (
            <div className="premium">
              ⭐ 프리미엄 회원
            </div>
          ) : (
            <button>프리미엄 업그레이드</button>
          )}
        </div>
      )}
      
      {!isLoading && !error && !user && (
        <div>사용자를 찾을 수 없습니다.</div>
      )}
    </div>
  );
}
```

**Vue**
```vue
<template>
  <div>
    <!-- v-if: 조건부 렌더링 -->
    <div v-if="isLoading">로딩 중...</div>
    
    <!-- v-else-if: else if 조건 -->
    <div v-else-if="error" class="error">
      <h3>오류 발생!</h3>
      <p>{{ error.message }}</p>
    </div>
    
    <!-- v-else-if: 또 다른 조건 -->
    <div v-else-if="user">
      <h1>환영합니다, {{ user.name }}님!</h1>
      
      <!-- 중첩된 조건부 렌더링 -->
      <div v-if="user.isPremium" class="premium">
        ⭐ 프리미엄 회원
      </div>
      <button v-else>프리미엄 업그레이드</button>
    </div>
    
    <!-- v-else: 마지막 fallback -->
    <div v-else>사용자를 찾을 수 없습니다.</div>
  </div>
</template>

<script>
export default {
  props: {
    user: Object,
    isLoading: Boolean,
    error: Object
  }
}
</script>
```

### v-show vs v-if

```vue
<template>
  <!-- v-if: DOM에서 완전히 제거/생성 -->
  <div v-if="showExpensiveComponent">
    <ExpensiveComponent />
  </div>

  <!-- v-show: display CSS 속성만 토글 (더 빠름) -->
  <div v-show="showModal" class="modal">
    <!-- 자주 토글되는 요소에 적합 -->
    <ModalContent />
  </div>
</template>
```

## 📋 리스트 렌더링

### v-for vs React map()

**React**
```jsx
function TodoList({ todos, onToggle, onDelete }) {
  return (
    <ul>
      {todos.map((todo, index) => (
        <li 
          key={todo.id} 
          className={todo.completed ? 'completed' : ''}
        >
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => onToggle(todo.id)}
          />
          <span>{todo.text}</span>
          <button onClick={() => onDelete(todo.id)}>삭제</button>
          <small>#{index + 1}</small>
        </li>
      ))}
    </ul>
  );
}
```

**Vue**
```vue
<template>
  <ul>
    <!-- v-for: 배열 순회 -->
    <li 
      v-for="(todo, index) in todos" 
      :key="todo.id"
      :class="{ completed: todo.completed }"
    >
      <input
        type="checkbox"
        :checked="todo.completed"
        @change="onToggle(todo.id)"
      />
      <span>{{ todo.text }}</span>
      <button @click="onDelete(todo.id)">삭제</button>
      <small>#{{ index + 1 }}</small>
    </li>
  </ul>
</template>

<script>
export default {
  props: {
    todos: Array
  },
  emits: ['toggle', 'delete'],
  methods: {
    onToggle(id) {
      this.$emit('toggle', id);
    },
    onDelete(id) {
      this.$emit('delete', id);
    }
  }
}
</script>
```

### 다양한 v-for 사용법

```vue
<template>
  <!-- 배열 순회 -->
  <li v-for="item in items" :key="item.id">{{ item.name }}</li>
  
  <!-- 인덱스와 함께 -->
  <li v-for="(item, index) in items" :key="item.id">
    {{ index }}: {{ item.name }}
  </li>
  
  <!-- 객체 순회 -->
  <li v-for="value in user" :key="value">{{ value }}</li>
  
  <!-- 객체의 키와 값 -->
  <li v-for="(value, key) in user" :key="key">
    {{ key }}: {{ value }}
  </li>
  
  <!-- 키, 값, 인덱스 모두 -->
  <li v-for="(value, key, index) in user" :key="key">
    {{ index }}. {{ key }}: {{ value }}
  </li>
  
  <!-- 숫자 범위 -->
  <span v-for="n in 10" :key="n">{{ n }}</span>
</template>
```

## 🎮 이벤트 처리

### 기본 이벤트 vs React

**React**
```jsx
function EventDemo() {
  const [message, setMessage] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('제출:', message);
  };
  
  const handleKeyDown = (e) => {
    if (e.key === 'Enter' && e.ctrlKey) {
      handleSubmit(e);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        onKeyDown={handleKeyDown}
      />
      <button type="submit">제출</button>
    </form>
  );
}
```

**Vue**
```vue
<template>
  <!-- @ = v-on: 단축문법 -->
  <form @submit.prevent="handleSubmit">
    <!-- .prevent: e.preventDefault() 자동 적용! -->
    <input
      v-model="message"
      @keydown.ctrl.enter="handleSubmit"
    />
    <!-- 키 조합도 쉽게! -->
    <button type="submit">제출</button>
  </form>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const message = ref('');
    
    const handleSubmit = () => {
      console.log('제출:', message.value);
    };
    
    return { message, handleSubmit }
  }
}
</script>
```

### 이벤트 수식어 (Event Modifiers)

```vue
<template>
  <!-- .prevent: e.preventDefault() -->
  <form @submit.prevent="handleSubmit">
    <button type="submit">제출 (새로고침 안됨)</button>
  </form>

  <!-- .stop: e.stopPropagation() -->
  <div @click="handleDivClick">
    <button @click.stop="handleButtonClick">
      클릭 (이벤트 버블링 방지)
    </button>
  </div>

  <!-- .once: 한 번만 실행 -->
  <button @click.once="handleOnce">한 번만 클릭됨</button>

  <!-- .self: 자기 자신에서만 실행 -->
  <div @click.self="handleSelfOnly">
    <p>이 p를 클릭해도 핸들러 실행 안됨</p>
  </div>

  <!-- 키 수식어 -->
  <input 
    @keyup.enter="handleEnter"
    @keyup.esc="handleEscape"
    @keyup.space="handleSpace"
    @keyup.ctrl.enter="handleCtrlEnter"
  />

  <!-- 마우스 수식어 -->
  <button 
    @click.left="handleLeftClick"
    @click.right.prevent="handleRightClick"
    @click.middle="handleMiddleClick"
  >
    마우스 버튼별 처리
  </button>
</template>
```

## 🔄 양방향 데이터 바인딩

### v-model vs React 제어 컴포넌트

**React**
```jsx
function ContactForm() {
  const [form, setForm] = useState({
    name: '',
    email: '',
    age: 0,
    newsletter: false,
    gender: '',
    hobbies: [],
    bio: ''
  });

  const handleInputChange = (e) => {
    const { name, value, type, checked } = e.target;
    setForm(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const handleHobbyChange = (hobby) => {
    setForm(prev => ({
      ...prev,
      hobbies: prev.hobbies.includes(hobby)
        ? prev.hobbies.filter(h => h !== hobby)
        : [...prev.hobbies, hobby]
    }));
  };

  return (
    <form>
      <input
        name="name"
        value={form.name}
        onChange={handleInputChange}
        placeholder="이름"
      />
      
      <input
        name="email"
        type="email"
        value={form.email}
        onChange={handleInputChange}
        placeholder="이메일"
      />
      
      <input
        name="age"
        type="number"
        value={form.age}
        onChange={handleInputChange}
      />
      
      <label>
        <input
          name="newsletter"
          type="checkbox"
          checked={form.newsletter}
          onChange={handleInputChange}
        />
        뉴스레터 구독
      </label>
      
      {/* 라디오, 셀렉트, 체크박스 배열 등 더 복잡... */}
    </form>
  );
}
```

**Vue**
```vue
<template>
  <form>
    <!-- 텍스트 입력 -->
    <input v-model="form.name" placeholder="이름" />
    
    <!-- 이메일 입력 -->
    <input v-model="form.email" type="email" placeholder="이메일" />
    
    <!-- 숫자 입력 (.number 수식어로 자동 변환) -->
    <input v-model.number="form.age" type="number" />
    
    <!-- 체크박스 -->
    <label>
      <input v-model="form.newsletter" type="checkbox" />
      뉴스레터 구독
    </label>
    
    <!-- 라디오 버튼 -->
    <label>
      <input v-model="form.gender" type="radio" value="male" />
      남성
    </label>
    <label>
      <input v-model="form.gender" type="radio" value="female" />
      여성
    </label>
    
    <!-- 체크박스 배열 -->
    <label>
      <input v-model="form.hobbies" type="checkbox" value="reading" />
      독서
    </label>
    <label>
      <input v-model="form.hobbies" type="checkbox" value="music" />
      음악
    </label>
    <label>
      <input v-model="form.hobbies" type="checkbox" value="sports" />
      운동
    </label>
    
    <!-- 셀렉트 -->
    <select v-model="form.country">
      <option value="">국가 선택</option>
      <option value="kr">한국</option>
      <option value="us">미국</option>
      <option value="jp">일본</option>
    </select>
    
    <!-- 텍스트영역 (.trim 수식어로 자동 공백 제거) -->
    <textarea 
      v-model.trim="form.bio" 
      placeholder="자기소개"
    ></textarea>
  </form>
</template>

<script>
import { reactive } from 'vue'

export default {
  setup() {
    const form = reactive({
      name: '',
      email: '',
      age: 0,
      newsletter: false,
      gender: '',
      hobbies: [],
      country: '',
      bio: ''
    });

    return { form }
  }
}
</script>
```

### v-model 수식어

```vue
<template>
  <!-- .lazy: change 이벤트에서만 동기화 (input 대신) -->
  <input v-model.lazy="message" />
  
  <!-- .number: 자동으로 숫자 타입 변환 -->
  <input v-model.number="age" type="number" />
  
  <!-- .trim: 자동으로 앞뒤 공백 제거 -->
  <input v-model.trim="username" />
  
  <!-- 수식어 조합도 가능 -->
  <input v-model.lazy.trim="description" />
</template>
```

## 🎯 실습: Todo 앱 만들기

React와 Vue로 같은 Todo 앱을 만들어 문법 차이를 체감해보세요!

```vue
<template>
  <div class="todo-app">
    <h1>Vue Todo App</h1>
    
    <!-- 할 일 추가 폼 -->
    <form @submit.prevent="addTodo">
      <input
        v-model.trim="newTodo"
        placeholder="새 할 일을 입력하세요"
        required
      />
      <button type="submit">추가</button>
    </form>

    <!-- 필터 버튼들 -->
    <div class="filters">
      <button 
        v-for="filter in filters" 
        :key="filter"
        :class="{ active: currentFilter === filter }"
        @click="currentFilter = filter"
      >
        {{ filter }}
      </button>
    </div>

    <!-- 할 일 목록 -->
    <ul class="todo-list">
      <li 
        v-for="todo in filteredTodos" 
        :key="todo.id"
        :class="{ completed: todo.completed }"
      >
        <input
          type="checkbox"
          v-model="todo.completed"
        />
        <span class="todo-text">{{ todo.text }}</span>
        <button @click="deleteTodo(todo.id)" class="delete-btn">
          삭제
        </button>
      </li>
    </ul>

    <!-- 상태 정보 -->
    <div class="status">
      <p>전체: {{ todos.length }}</p>
      <p>완료: {{ completedCount }}</p>
      <p>남은 할 일: {{ remainingCount }}</p>
    </div>

    <!-- 전체 삭제 버튼 -->
    <button 
      v-if="todos.length > 0"
      @click="clearCompleted"
      class="clear-btn"
    >
      완료된 할 일 삭제
    </button>
  </div>
</template>

<script>
import { ref, computed } from 'vue'

export default {
  setup() {
    const newTodo = ref('');
    const todos = ref([]);
    const currentFilter = ref('전체');
    const filters = ['전체', '진행중', '완료'];

    // 계산된 속성들
    const filteredTodos = computed(() => {
      switch (currentFilter.value) {
        case '진행중':
          return todos.value.filter(todo => !todo.completed);
        case '완료':
          return todos.value.filter(todo => todo.completed);
        default:
          return todos.value;
      }
    });

    const completedCount = computed(() =>
      todos.value.filter(todo => todo.completed).length
    );

    const remainingCount = computed(() =>
      todos.value.filter(todo => !todo.completed).length
    );

    // 메서드들
    const addTodo = () => {
      if (newTodo.value) {
        todos.value.push({
          id: Date.now(),
          text: newTodo.value,
          completed: false
        });
        newTodo.value = '';
      }
    };

    const deleteTodo = (id) => {
      const index = todos.value.findIndex(todo => todo.id === id);
      if (index > -1) {
        todos.value.splice(index, 1);
      }
    };

    const clearCompleted = () => {
      todos.value = todos.value.filter(todo => !todo.completed);
    };

    return {
      newTodo,
      todos,
      currentFilter,
      filters,
      filteredTodos,
      completedCount,
      remainingCount,
      addTodo,
      deleteTodo,
      clearCompleted
    }
  }
}
</script>

<style scoped>
.todo-app {
  max-width: 500px;
  margin: 50px auto;
  padding: 20px;
}

form {
  display: flex;
  margin-bottom: 20px;
}

form input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px 0 0 4px;
}

form button {
  padding: 10px 20px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
}

.filters {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.filters button {
  padding: 8px 16px;
  border: 1px solid #ddd;
  background: white;
  cursor: pointer;
  border-radius: 4px;
}

.filters button.active {
  background: #42b883;
  color: white;
  border-color: #42b883;
}

.todo-list {
  list-style: none;
  padding: 0;
}

.todo-list li {
  display: flex;
  align-items: center;
  padding: 10px;
  border-bottom: 1px solid #eee;
}

.todo-list li.completed .todo-text {
  text-decoration: line-through;
  color: #999;
}

.todo-text {
  flex: 1;
  margin: 0 10px;
}

.delete-btn {
  background: #e74c3c;
  color: white;
  border: none;
  padding: 4px 8px;
  border-radius: 4px;
  cursor: pointer;
}

.status {
  margin: 20px 0;
  padding: 10px;
  background: #f9f9f9;
  border-radius: 4px;
}

.status p {
  margin: 5px 0;
}

.clear-btn {
  width: 100%;
  padding: 10px;
  background: #e74c3c;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
</style>
```

## 🔥 핵심 포인트

1. **Vue 템플릿은 HTML에 더 가까움**
   - JSX보다 디자이너가 이해하기 쉬움
   - 조건부 렌더링이 더 명확 (`v-if`, `v-else`)

2. **디렉티브가 강력함**
   - `v-model`: 양방향 바인딩이 간단
   - `v-for`: 리스트 렌더링이 직관적
   - 이벤트 수식어: `.prevent`, `.stop` 등으로 간편

3. **React 대비 더 적은 코드**
   - 폼 처리가 특히 간단
   - 이벤트 핸들링이 더 직관적

## ➡️ 다음 단계

템플릿 문법을 이해했다면 [1-3. 데이터 바인딩](./03-data-binding.md)으로 넘어가서 Vue의 반응형 시스템을 깊이 알아보세요!

---

*Vue 템플릿 문법은 처음엔 낯설 수 있지만, HTML과 비슷해서 금방 익숙해질 거예요. 특히 `v-model`의 편리함에 감탄하게 될 것입니다! 🚀* 