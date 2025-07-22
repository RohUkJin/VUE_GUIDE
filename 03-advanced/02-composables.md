# 3-2. 커스텀 Composables

## 🎭 Vue Composables vs React Custom Hooks

Vue의 Composables는 React의 Custom Hooks와 매우 유사하지만, 더 강력한 반응성 시스템을 기반으로 하여 더욱 편리하고 직관적입니다!

## 🏗️ 1. 기본 패턴 비교

### React Custom Hook
```jsx
// useCounter.js
import { useState, useCallback } from 'react';

function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);

  const decrement = useCallback(() => {
    setCount(prev => prev - 1);
  }, []);

  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);

  return { count, increment, decrement, reset };
}

// 사용
function Counter() {
  const { count, increment, decrement, reset } = useCounter(10);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

### Vue Composable
```javascript
// composables/useCounter.js
import { ref, computed } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)
  
  const increment = () => {
    count.value++
  }
  
  const decrement = () => {
    count.value--
  }
  
  const reset = () => {
    count.value = initialValue
  }
  
  // 추가 계산된 속성
  const isPositive = computed(() => count.value > 0)
  const isEven = computed(() => count.value % 2 === 0)
  
  return {
    count,
    increment,
    decrement,
    reset,
    isPositive,
    isEven
  }
}
```

```vue
<!-- Counter.vue -->
<template>
  <div class="counter">
    <p>Count: {{ count }}</p>
    <p>Is Positive: {{ isPositive ? 'Yes' : 'No' }}</p>
    <p>Is Even: {{ isEven ? 'Yes' : 'No' }}</p>
    
    <div class="buttons">
      <button @click="increment">+</button>
      <button @click="decrement">-</button>
      <button @click="reset">Reset</button>
    </div>
  </div>
</template>

<script>
import { useCounter } from '@/composables/useCounter'

export default {
  setup() {
    const counterState = useCounter(10)
    
    return {
      ...counterState
    }
  }
}
</script>
```

## 🌐 2. 네트워크 요청 Composables

### React: useApi Hook
```jsx
// useApi.js
import { useState, useEffect, useCallback } from 'react';

function useApi(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchData = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch(url, options);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err);
    } finally {
      setLoading(false);
    }
  }, [url, options]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  const refetch = useCallback(() => {
    fetchData();
  }, [fetchData]);

  return { data, loading, error, refetch };
}
```

### Vue: useApi Composable
```javascript
// composables/useApi.js
import { ref, watch, computed, onUnmounted } from 'vue'

export function useApi(url, options = {}) {
  const data = ref(null)
  const loading = ref(false)
  const error = ref(null)
  const abortController = ref(null)
  
  const isReady = computed(() => !loading.value && !error.value && data.value !== null)
  
  const execute = async (customUrl = null, customOptions = {}) => {
    // 이전 요청 취소
    if (abortController.value) {
      abortController.value.abort()
    }
    
    abortController.value = new AbortController()
    
    const requestUrl = customUrl || url.value || url
    const requestOptions = {
      ...options,
      ...customOptions,
      signal: abortController.value.signal
    }
    
    loading.value = true
    error.value = null
    
    try {
      const response = await fetch(requestUrl, requestOptions)
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`)
      }
      
      const result = await response.json()
      data.value = result
      
      return result
      
    } catch (err) {
      if (err.name !== 'AbortError') {
        error.value = err
        throw err
      }
    } finally {
      loading.value = false
    }
  }
  
  const refetch = () => execute()
  
  const reset = () => {
    data.value = null
    error.value = null
    loading.value = false
  }
  
  // URL이 반응형인 경우 자동 재실행
  if (typeof url === 'object' && url.value !== undefined) {
    watch(url, (newUrl) => {
      if (newUrl) {
        execute()
      }
    }, { immediate: true })
  } else if (url) {
    // 초기 실행
    execute()
  }
  
  // 컴포넌트 언마운트 시 요청 취소
  onUnmounted(() => {
    if (abortController.value) {
      abortController.value.abort()
    }
  })
  
  return {
    data,
    loading,
    error,
    isReady,
    execute,
    refetch,
    reset
  }
}
```

### 고급 useApi 사용 예제
```vue
<template>
  <div class="api-demo">
    <h2>API 데모</h2>
    
    <!-- 사용자 목록 -->
    <div class="section">
      <h3>사용자 목록</h3>
      <div class="controls">
        <input v-model="searchQuery" placeholder="사용자 검색..." />
        <button @click="refreshUsers">새로고침</button>
      </div>
      
      <div v-if="usersLoading">Loading users...</div>
      <div v-else-if="usersError" class="error">
        Error: {{ usersError.message }}
        <button @click="refreshUsers">재시도</button>
      </div>
      <div v-else-if="users">
        <div v-for="user in users" :key="user.id" class="user-item">
          <h4>{{ user.name }}</h4>
          <p>{{ user.email }}</p>
          <button @click="loadUserDetails(user.id)">상세 보기</button>
        </div>
      </div>
    </div>
    
    <!-- 선택된 사용자 상세 -->
    <div v-if="selectedUserId" class="section">
      <h3>사용자 상세 정보</h3>
      <div v-if="userDetailsLoading">Loading details...</div>
      <div v-else-if="userDetailsError" class="error">
        Error: {{ userDetailsError.message }}
      </div>
      <div v-else-if="userDetails">
        <h4>{{ userDetails.name }}</h4>
        <p>Email: {{ userDetails.email }}</p>
        <p>Phone: {{ userDetails.phone }}</p>
        <p>Website: {{ userDetails.website }}</p>
        
        <!-- 사용자 게시글 -->
        <h5>게시글</h5>
        <div v-if="userPostsLoading">Loading posts...</div>
        <div v-else-if="userPosts">
          <div v-for="post in userPosts" :key="post.id" class="post-item">
            <h6>{{ post.title }}</h6>
            <p>{{ post.body }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue'
import { useApi } from '@/composables/useApi'

export default {
  setup() {
    const searchQuery = ref('')
    const selectedUserId = ref(null)
    
    // 사용자 목록 API
    const usersUrl = computed(() => {
      const baseUrl = 'https://jsonplaceholder.typicode.com/users'
      return searchQuery.value 
        ? `${baseUrl}?name_like=${searchQuery.value}`
        : baseUrl
    })
    
    const {
      data: users,
      loading: usersLoading,
      error: usersError,
      refetch: refreshUsers
    } = useApi(usersUrl)
    
    // 선택된 사용자 상세 정보
    const userDetailsUrl = computed(() => 
      selectedUserId.value 
        ? `https://jsonplaceholder.typicode.com/users/${selectedUserId.value}`
        : null
    )
    
    const {
      data: userDetails,
      loading: userDetailsLoading,
      error: userDetailsError
    } = useApi(userDetailsUrl)
    
    // 선택된 사용자의 게시글
    const userPostsUrl = computed(() =>
      selectedUserId.value
        ? `https://jsonplaceholder.typicode.com/posts?userId=${selectedUserId.value}`
        : null
    )
    
    const {
      data: userPosts,
      loading: userPostsLoading,
      error: userPostsError
    } = useApi(userPostsUrl)
    
    const loadUserDetails = (userId) => {
      selectedUserId.value = userId
    }
    
    return {
      searchQuery,
      selectedUserId,
      users,
      usersLoading,
      usersError,
      refreshUsers,
      userDetails,
      userDetailsLoading,
      userDetailsError,
      userPosts,
      userPostsLoading,
      userPostsError,
      loadUserDetails
    }
  }
}
</script>

<style scoped>
.api-demo {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.section {
  margin-bottom: 30px;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.controls {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.user-item,
.post-item {
  padding: 15px;
  border: 1px solid #eee;
  border-radius: 4px;
  margin-bottom: 10px;
}

.error {
  color: #dc3545;
  padding: 10px;
  background: #f8d7da;
  border-radius: 4px;
}
</style>
```

## 🔄 3. 상태 관리 Composables

### useLocalStorage
```javascript
// composables/useLocalStorage.js
import { ref, watch, nextTick } from 'vue'

export function useLocalStorage(key, defaultValue, options = {}) {
  const {
    serializer = JSON,
    syncAcrossTabs = true,
    onError = (error) => console.error(error)
  } = options
  
  const storedValue = ref(defaultValue)
  
  // 초기값 로드
  try {
    const item = localStorage.getItem(key)
    if (item !== null) {
      storedValue.value = serializer.parse(item)
    }
  } catch (error) {
    onError(error)
    storedValue.value = defaultValue
  }
  
  // 값 변경 시 localStorage 업데이트
  watch(storedValue, (newValue) => {
    try {
      localStorage.setItem(key, serializer.stringify(newValue))
    } catch (error) {
      onError(error)
    }
  }, { deep: true })
  
  // 다른 탭에서의 변경사항 동기화
  if (syncAcrossTabs) {
    const handleStorageChange = (e) => {
      if (e.key === key && e.newValue) {
        try {
          storedValue.value = serializer.parse(e.newValue)
        } catch (error) {
          onError(error)
        }
      }
    }
    
    window.addEventListener('storage', handleStorageChange)
    
    // 정리 함수는 컴포넌트에서 호출해야 함
    const cleanup = () => {
      window.removeEventListener('storage', handleStorageChange)
    }
    
    return [storedValue, cleanup]
  }
  
  return [storedValue]
}
```

### useSessionStorage
```javascript
// composables/useSessionStorage.js
import { ref, watch } from 'vue'

export function useSessionStorage(key, defaultValue, options = {}) {
  const {
    serializer = JSON,
    onError = (error) => console.error(error)
  } = options
  
  const storedValue = ref(defaultValue)
  
  // 초기값 로드
  try {
    const item = sessionStorage.getItem(key)
    if (item !== null) {
      storedValue.value = serializer.parse(item)
    }
  } catch (error) {
    onError(error)
    storedValue.value = defaultValue
  }
  
  // 값 변경 시 sessionStorage 업데이트
  watch(storedValue, (newValue) => {
    try {
      sessionStorage.setItem(key, serializer.stringify(newValue))
    } catch (error) {
      onError(error)
    }
  }, { deep: true })
  
  const remove = () => {
    sessionStorage.removeItem(key)
    storedValue.value = defaultValue
  }
  
  return [storedValue, remove]
}
```

### useToggle
```javascript
// composables/useToggle.js
import { ref } from 'vue'

export function useToggle(initialValue = false) {
  const value = ref(initialValue)
  
  const toggle = (newValue) => {
    if (typeof newValue === 'boolean') {
      value.value = newValue
    } else {
      value.value = !value.value
    }
  }
  
  const setTrue = () => {
    value.value = true
  }
  
  const setFalse = () => {
    value.value = false
  }
  
  return [value, toggle, setTrue, setFalse]
}
```

## 🎯 4. UI 상호작용 Composables

### useClickOutside
```javascript
// composables/useClickOutside.js
import { onMounted, onUnmounted } from 'vue'

export function useClickOutside(elementRef, callback) {
  const handleClick = (event) => {
    if (elementRef.value && !elementRef.value.contains(event.target)) {
      callback(event)
    }
  }
  
  onMounted(() => {
    document.addEventListener('click', handleClick)
  })
  
  onUnmounted(() => {
    document.removeEventListener('click', handleClick)
  })
}
```

### useKeyboard
```javascript
// composables/useKeyboard.js
import { ref, onMounted, onUnmounted } from 'vue'

export function useKeyboard() {
  const pressedKeys = ref(new Set())
  const lastPressed = ref(null)
  
  const handleKeyDown = (event) => {
    pressedKeys.value.add(event.key)
    lastPressed.value = event.key
  }
  
  const handleKeyUp = (event) => {
    pressedKeys.value.delete(event.key)
  }
  
  const isPressed = (key) => {
    return pressedKeys.value.has(key)
  }
  
  const isComboPressed = (...keys) => {
    return keys.every(key => pressedKeys.value.has(key))
  }
  
  onMounted(() => {
    window.addEventListener('keydown', handleKeyDown)
    window.addEventListener('keyup', handleKeyUp)
  })
  
  onUnmounted(() => {
    window.removeEventListener('keydown', handleKeyDown)
    window.removeEventListener('keyup', handleKeyUp)
  })
  
  return {
    pressedKeys,
    lastPressed,
    isPressed,
    isComboPressed
  }
}
```

### useMouse
```javascript
// composables/useMouse.js
import { ref, onMounted, onUnmounted } from 'vue'

export function useMouse() {
  const x = ref(0)
  const y = ref(0)
  const isMoving = ref(false)
  let moveTimeout = null
  
  const handleMouseMove = (event) => {
    x.value = event.clientX
    y.value = event.clientY
    isMoving.value = true
    
    clearTimeout(moveTimeout)
    moveTimeout = setTimeout(() => {
      isMoving.value = false
    }, 100)
  }
  
  onMounted(() => {
    window.addEventListener('mousemove', handleMouseMove)
  })
  
  onUnmounted(() => {
    window.removeEventListener('mousemove', handleMouseMove)
    clearTimeout(moveTimeout)
  })
  
  return {
    x,
    y,
    isMoving
  }
}
```

## 🔄 5. 비동기 처리 Composables

### useAsyncQueue
```javascript
// composables/useAsyncQueue.js
import { ref, computed } from 'vue'

export function useAsyncQueue() {
  const queue = ref([])
  const processing = ref(false)
  const results = ref([])
  const errors = ref([])
  
  const isIdle = computed(() => !processing.value && queue.value.length === 0)
  const hasErrors = computed(() => errors.value.length > 0)
  
  const add = (asyncFunction, ...args) => {
    queue.value.push({ function: asyncFunction, args, id: Date.now() })
    if (!processing.value) {
      processQueue()
    }
  }
  
  const processQueue = async () => {
    if (processing.value || queue.value.length === 0) return
    
    processing.value = true
    
    while (queue.value.length > 0) {
      const task = queue.value.shift()
      
      try {
        const result = await task.function(...task.args)
        results.value.push({ id: task.id, result })
      } catch (error) {
        errors.value.push({ id: task.id, error })
      }
    }
    
    processing.value = false
  }
  
  const clear = () => {
    queue.value = []
    results.value = []
    errors.value = []
  }
  
  const clearErrors = () => {
    errors.value = []
  }
  
  return {
    queue,
    processing,
    results,
    errors,
    isIdle,
    hasErrors,
    add,
    clear,
    clearErrors
  }
}
```

### useDebounce
```javascript
// composables/useDebounce.js
import { ref, watch } from 'vue'

export function useDebounce(value, delay = 300) {
  const debouncedValue = ref(value.value)
  let timeout = null
  
  watch(value, (newValue) => {
    clearTimeout(timeout)
    timeout = setTimeout(() => {
      debouncedValue.value = newValue
    }, delay)
  })
  
  return debouncedValue
}
```

### useThrottle
```javascript
// composables/useThrottle.js
import { ref, watch } from 'vue'

export function useThrottle(value, delay = 300) {
  const throttledValue = ref(value.value)
  let lastExecution = 0
  
  watch(value, (newValue) => {
    const now = Date.now()
    
    if (now - lastExecution >= delay) {
      throttledValue.value = newValue
      lastExecution = now
    }
  })
  
  return throttledValue
}
```

## 🎯 6. 실습 예제: 종합 대시보드

```vue
<template>
  <div class="dashboard">
    <header class="dashboard-header">
      <h1>📊 종합 대시보드</h1>
      <div class="header-controls">
        <button @click="toggleTheme">
          {{ isDarkMode ? '☀️' : '🌙' }} 테마
        </button>
        <span class="mouse-position">
          마우스: {{ mouseX }}, {{ mouseY }}
        </span>
      </div>
    </header>
    
    <!-- 키보드 단축키 표시 -->
    <div v-if="showHelp" class="help-overlay" @click="toggleHelp(false)">
      <div class="help-content" @click.stop>
        <h3>키보드 단축키</h3>
        <ul>
          <li><kbd>Ctrl</kbd> + <kbd>R</kbd>: 새로고침</li>
          <li><kbd>Ctrl</kbd> + <kbd>H</kbd>: 도움말</li>
          <li><kbd>Escape</kbd>: 도움말 닫기</li>
        </ul>
        <button @click="toggleHelp(false)">닫기</button>
      </div>
    </div>
    
    <!-- 검색 섹션 -->
    <div class="search-section">
      <input 
        v-model="searchQuery" 
        placeholder="검색어 입력... (300ms 지연)"
        class="search-input"
      />
      <p>Debounced: "{{ debouncedSearch }}"</p>
      <p>Throttled: "{{ throttledSearch }}"</p>
    </div>
    
    <!-- API 데이터 섹션 -->
    <div class="data-section">
      <h2>사용자 데이터</h2>
      <div class="controls">
        <button @click="refreshData" :disabled="loading">
          {{ loading ? '로딩 중...' : '새로고침' }}
        </button>
        <button @click="addToQueue">대기열에 추가</button>
        <span v-if="!isQueueIdle">
          처리 중: {{ queueResults.length }} 완료, {{ queueErrors.length }} 에러
        </span>
      </div>
      
      <div v-if="error" class="error">
        Error: {{ error.message }}
      </div>
      
      <div v-else-if="data" class="user-grid">
        <div 
          v-for="user in data.slice(0, 6)" 
          :key="user.id"
          class="user-card"
          @click="selectUser(user)"
        >
          <h4>{{ user.name }}</h4>
          <p>{{ user.email }}</p>
        </div>
      </div>
    </div>
    
    <!-- 선택된 사용자 모달 -->
    <div v-if="selectedUser" class="modal-overlay" @click="selectedUser = null">
      <div class="modal-content" ref="modalRef" @click.stop>
        <h3>{{ selectedUser.name }}</h3>
        <p>Email: {{ selectedUser.email }}</p>
        <p>Phone: {{ selectedUser.phone }}</p>
        <p>Website: {{ selectedUser.website }}</p>
        <button @click="selectedUser = null">닫기</button>
      </div>
    </div>
    
    <!-- 설정 저장 -->
    <div class="settings-section">
      <h2>설정</h2>
      <label>
        <input type="checkbox" v-model="autoRefresh" />
        자동 새로고침 (localStorage에 저장됨)
      </label>
      <br />
      <label>
        새로고침 간격 (초):
        <input 
          type="number" 
          v-model.number="refreshInterval" 
          min="5" 
          max="60"
        />
      </label>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch, onMounted } from 'vue'
import { useApi } from '@/composables/useApi'
import { useLocalStorage } from '@/composables/useLocalStorage'
import { useToggle } from '@/composables/useToggle'
import { useClickOutside } from '@/composables/useClickOutside'
import { useKeyboard } from '@/composables/useKeyboard'
import { useMouse } from '@/composables/useMouse'
import { useDebounce } from '@/composables/useDebounce'
import { useThrottle } from '@/composables/useThrottle'
import { useAsyncQueue } from '@/composables/useAsyncQueue'

export default {
  setup() {
    // 기본 상태
    const selectedUser = ref(null)
    const modalRef = ref(null)
    
    // 설정 (localStorage에 저장)
    const [isDarkMode] = useToggle(false)
    const [autoRefresh] = useLocalStorage('autoRefresh', false)
    const [refreshInterval] = useLocalStorage('refreshInterval', 10)
    
    // UI 상호작용
    const [showHelp, toggleHelp] = useToggle(false)
    const { isPressed, isComboPressed } = useKeyboard()
    const { x: mouseX, y: mouseY } = useMouse()
    
    // 검색
    const searchQuery = ref('')
    const debouncedSearch = useDebounce(searchQuery, 300)
    const throttledSearch = useThrottle(searchQuery, 500)
    
    // API 데이터
    const { 
      data, 
      loading, 
      error, 
      refetch: refreshData 
    } = useApi('https://jsonplaceholder.typicode.com/users')
    
    // 비동기 대기열
    const {
      queue,
      processing,
      results: queueResults,
      errors: queueErrors,
      isIdle: isQueueIdle,
      add: addToQueue
    } = useAsyncQueue()
    
    // 모달 외부 클릭
    useClickOutside(modalRef, () => {
      selectedUser.value = null
    })
    
    // 키보드 단축키
    watch([isPressed, isComboPressed], () => {
      if (isComboPressed('Control', 'h')) {
        toggleHelp()
      }
      if (isComboPressed('Control', 'r')) {
        refreshData()
      }
      if (isPressed('Escape')) {
        if (showHelp.value) {
          toggleHelp(false)
        }
        if (selectedUser.value) {
          selectedUser.value = null
        }
      }
    })
    
    // 자동 새로고침
    let autoRefreshTimer = null
    watch([autoRefresh, refreshInterval], () => {
      if (autoRefreshTimer) {
        clearInterval(autoRefreshTimer)
      }
      
      if (autoRefresh.value) {
        autoRefreshTimer = setInterval(() => {
          refreshData()
        }, refreshInterval.value * 1000)
      }
    }, { immediate: true })
    
    // 테마 토글
    const toggleTheme = () => {
      isDarkMode.value = !isDarkMode.value
      document.body.className = isDarkMode.value ? 'dark-theme' : 'light-theme'
    }
    
    // 사용자 선택
    const selectUser = (user) => {
      selectedUser.value = user
    }
    
    // 대기열에 작업 추가
    const addTaskToQueue = () => {
      const randomDelay = Math.random() * 2000 + 500
      
      addToQueue(async () => {
        await new Promise(resolve => setTimeout(resolve, randomDelay))
        return `Task completed after ${randomDelay.toFixed(0)}ms`
      })
    }
    
    onMounted(() => {
      toggleTheme() // 초기 테마 설정
    })
    
    return {
      // 상태
      selectedUser,
      modalRef,
      searchQuery,
      debouncedSearch,
      throttledSearch,
      
      // 설정
      isDarkMode,
      autoRefresh,
      refreshInterval,
      
      // UI 상호작용
      showHelp,
      toggleHelp,
      mouseX,
      mouseY,
      
      // API
      data,
      loading,
      error,
      refreshData,
      
      // 대기열
      queueResults,
      queueErrors,
      isQueueIdle,
      addToQueue: addTaskToQueue,
      
      // 메서드
      toggleTheme,
      selectUser
    }
  }
}
</script>

<style scoped>
.dashboard {
  min-height: 100vh;
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.dashboard-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 2px solid #42b883;
}

.header-controls {
  display: flex;
  gap: 20px;
  align-items: center;
}

.mouse-position {
  font-family: monospace;
  font-size: 14px;
  color: #666;
}

.help-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.help-content {
  background: white;
  padding: 30px;
  border-radius: 8px;
  max-width: 400px;
}

.help-content kbd {
  background: #f0f0f0;
  padding: 2px 6px;
  border-radius: 3px;
  font-family: monospace;
}

.search-section,
.data-section,
.settings-section {
  margin-bottom: 30px;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.search-input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 10px;
}

.controls {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  align-items: center;
}

.user-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 15px;
}

.user-card {
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 8px;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.user-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 30px;
  border-radius: 8px;
  max-width: 500px;
  width: 90%;
}

.error {
  color: #dc3545;
  padding: 10px;
  background: #f8d7da;
  border-radius: 4px;
  margin-bottom: 20px;
}

button {
  background: #42b883;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.2s;
}

button:hover:not(:disabled) {
  background: #369870;
}

button:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* 다크 테마 */
:global(.dark-theme) .dashboard {
  background: #1a1a1a;
  color: white;
}

:global(.dark-theme) .search-section,
:global(.dark-theme) .data-section,
:global(.dark-theme) .settings-section {
  background: #2d2d2d;
  border-color: #444;
}

:global(.dark-theme) .user-card {
  background: #2d2d2d;
  border-color: #444;
}

:global(.dark-theme) .modal-content,
:global(.dark-theme) .help-content {
  background: #2d2d2d;
  color: white;
}
</style>
```

## 🎯 실습 과제

1. 위의 종합 대시보드를 직접 구현해보세요
2. useInfiniteScroll composable을 만들어보세요
3. useWebSocket composable을 구현해보세요
4. useGeolocation composable을 만들어보세요
5. useDragAndDrop composable을 구현해보세요

## 🔗 다음 단계

다음 챕터에서는 Pinia를 활용한 상태 관리에 대해 알아보겠습니다!

---

💡 **팁**: Vue Composables는 React Custom Hooks보다 더 간단하고 직관적입니다. 특히 반응성 시스템 덕분에 상태 관리가 훨씬 쉬워요! 