# 3-1. Composition API

## 🔥 Vue Composition API vs React Hooks

Vue 3의 Composition API는 React Hooks와 매우 비슷한 개념이지만, 더 직관적이고 강력합니다! React 개발자라면 금방 익숙해질 거예요.

## 🎯 1. 기본 개념 비교

### React Hooks 방식
```jsx
import { useState, useEffect, useMemo, useCallback } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [posts, setPosts] = useState([]);

  // 사용자 데이터 로드
  useEffect(() => {
    async function loadUser() {
      setLoading(true);
      try {
        const userData = await fetchUser(userId);
        setUser(userData);
      } catch (error) {
        console.error('Failed to load user:', error);
      } finally {
        setLoading(false);
      }
    }
    
    loadUser();
  }, [userId]);

  // 게시글 데이터 로드
  useEffect(() => {
    if (user) {
      fetchUserPosts(user.id).then(setPosts);
    }
  }, [user]);

  // 계산된 값
  const userStats = useMemo(() => {
    if (!user || !posts.length) return null;
    return {
      totalPosts: posts.length,
      avgLikes: posts.reduce((sum, post) => sum + post.likes, 0) / posts.length,
      lastPostDate: Math.max(...posts.map(post => new Date(post.createdAt)))
    };
  }, [user, posts]);

  // 콜백 함수
  const handleLike = useCallback((postId) => {
    setPosts(prevPosts => 
      prevPosts.map(post => 
        post.id === postId 
          ? { ...post, likes: post.likes + 1 }
          : post
      )
    );
  }, []);

  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      {userStats && (
        <div>
          <p>Posts: {userStats.totalPosts}</p>
          <p>Avg Likes: {userStats.avgLikes.toFixed(1)}</p>
        </div>
      )}
      {posts.map(post => (
        <div key={post.id}>
          <h3>{post.title}</h3>
          <p>Likes: {post.likes}</p>
          <button onClick={() => handleLike(post.id)}>Like</button>
        </div>
      ))}
    </div>
  );
}
```

### Vue Composition API 방식
```vue
<template>
  <div>
    <div v-if="loading">Loading...</div>
    <div v-else-if="!user">User not found</div>
    <div v-else>
      <h1>{{ user.name }}</h1>
      
      <div v-if="userStats">
        <p>Posts: {{ userStats.totalPosts }}</p>
        <p>Avg Likes: {{ userStats.avgLikes.toFixed(1) }}</p>
        <p>Last Post: {{ userStats.lastPostDate.toLocaleDateString() }}</p>
      </div>
      
      <div v-for="post in posts" :key="post.id" class="post">
        <h3>{{ post.title }}</h3>
        <p>{{ post.content }}</p>
        <p>Likes: {{ post.likes }}</p>
        <button @click="handleLike(post.id)">👍 Like</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch, onMounted } from 'vue'

export default {
  props: {
    userId: {
      type: [String, Number],
      required: true
    }
  },
  
  setup(props) {
    // 반응형 상태 (useState 역할)
    const user = ref(null)
    const loading = ref(false)
    const posts = ref([])
    
    // 계산된 속성 (useMemo 역할)
    const userStats = computed(() => {
      if (!user.value || !posts.value.length) return null
      
      const totalPosts = posts.value.length
      const avgLikes = posts.value.reduce((sum, post) => sum + post.likes, 0) / totalPosts
      const lastPostDate = new Date(Math.max(...posts.value.map(post => new Date(post.createdAt))))
      
      return { totalPosts, avgLikes, lastPostDate }
    })
    
    // 함수 정의 (useCallback 역할 - Vue에서는 자동으로 최적화됨)
    const loadUser = async () => {
      loading.value = true
      try {
        const userData = await fetchUser(props.userId)
        user.value = userData
      } catch (error) {
        console.error('Failed to load user:', error)
      } finally {
        loading.value = false
      }
    }
    
    const loadUserPosts = async () => {
      if (user.value) {
        try {
          const postsData = await fetchUserPosts(user.value.id)
          posts.value = postsData
        } catch (error) {
          console.error('Failed to load posts:', error)
        }
      }
    }
    
    const handleLike = (postId) => {
      const post = posts.value.find(p => p.id === postId)
      if (post) {
        post.likes++
      }
    }
    
    // 사이드 이펙트 (useEffect 역할)
    watch(() => props.userId, async (newUserId) => {
      if (newUserId) {
        await loadUser()
      }
    }, { immediate: true })
    
    watch(user, async (newUser) => {
      if (newUser) {
        await loadUserPosts()
      }
    })
    
    // 마운트 시 실행 (useEffect with empty deps 역할)
    onMounted(() => {
      console.log('Component mounted')
    })
    
    // 템플릿에서 사용할 항목들 반환
    return {
      user,
      loading,
      posts,
      userStats,
      handleLike
    }
  }
}
</script>

<style scoped>
.post {
  border: 1px solid #ddd;
  padding: 16px;
  margin: 16px 0;
  border-radius: 8px;
}
</style>
```

## 🏗️ 2. 로직 재사용과 커스텀 컴포지션

### React: 커스텀 훅
```jsx
// useUser.js
function useUser(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!userId) return;

    async function loadUser() {
      setLoading(true);
      setError(null);
      try {
        const userData = await fetchUser(userId);
        setUser(userData);
      } catch (err) {
        setError(err);
      } finally {
        setLoading(false);
      }
    }

    loadUser();
  }, [userId]);

  const refetch = useCallback(() => {
    loadUser();
  }, [userId]);

  return { user, loading, error, refetch };
}

// useLocalStorage.js
function useLocalStorage(key, defaultValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : defaultValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}
```

### Vue: 컴포지션 함수
```javascript
// composables/useUser.js
import { ref, watch } from 'vue'

export function useUser(userId) {
  const user = ref(null)
  const loading = ref(false)
  const error = ref(null)
  
  const loadUser = async () => {
    if (!userId.value) return
    
    loading.value = true
    error.value = null
    
    try {
      const userData = await fetchUser(userId.value)
      user.value = userData
    } catch (err) {
      error.value = err
    } finally {
      loading.value = false
    }
  }
  
  // userId 변경 감지
  watch(userId, loadUser, { immediate: true })
  
  const refetch = () => loadUser()
  
  return {
    user,
    loading,
    error,
    refetch
  }
}

// composables/useLocalStorage.js
import { ref, watch } from 'vue'

export function useLocalStorage(key, defaultValue) {
  const storedValue = localStorage.getItem(key)
  const value = ref(storedValue ? JSON.parse(storedValue) : defaultValue)
  
  // 값 변경 시 localStorage 업데이트
  watch(value, (newValue) => {
    localStorage.setItem(key, JSON.stringify(newValue))
  }, { deep: true })
  
  return value
}
```

## ⚡ 3. 고급 패턴들

### 제공/주입 패턴 (Provide/Inject)
```vue
<!-- App.vue (최상위) -->
<script>
import { provide, reactive } from 'vue'

export default {
  setup() {
    // 전역 상태
    const globalState = reactive({
      user: null,
      theme: 'light',
      language: 'ko'
    })
    
    // 전역 액션
    const actions = {
      setUser(user) {
        globalState.user = user
      },
      
      toggleTheme() {
        globalState.theme = globalState.theme === 'light' ? 'dark' : 'light'
      },
      
      setLanguage(lang) {
        globalState.language = lang
      }
    }
    
    // 하위 컴포넌트에 제공
    provide('globalState', globalState)
    provide('globalActions', actions)
    
    return { globalState }
  }
}
</script>
```

```vue
<!-- 깊은 하위 컴포넌트 -->
<template>
  <div :class="`theme-${theme}`">
    <h1>{{ user ? `안녕하세요, ${user.name}님!` : '로그인하세요' }}</h1>
    <button @click="toggleTheme">
      {{ theme === 'light' ? '🌙' : '☀️' }} 테마 변경
    </button>
  </div>
</template>

<script>
import { inject, computed } from 'vue'

export default {
  setup() {
    // 최상위에서 제공된 상태와 액션 주입
    const globalState = inject('globalState')
    const globalActions = inject('globalActions')
    
    // 계산된 속성으로 추출
    const user = computed(() => globalState.user)
    const theme = computed(() => globalState.theme)
    
    return {
      user,
      theme,
      toggleTheme: globalActions.toggleTheme
    }
  }
}
</script>
```

### 고급 반응성 (Advanced Reactivity)
```vue
<template>
  <div class="advanced-demo">
    <h2>고급 반응성 데모</h2>
    
    <!-- 폼 데이터 -->
    <form @submit.prevent="handleSubmit">
      <div v-for="field in formFields" :key="field.key">
        <label>{{ field.label }}:</label>
        <input 
          v-model="formData[field.key]"
          :type="field.type"
          :placeholder="field.placeholder"
        />
      </div>
      
      <button type="submit" :disabled="!isFormValid">
        제출
      </button>
    </form>
    
    <!-- 실시간 검증 결과 -->
    <div class="validation-results">
      <h3>검증 결과:</h3>
      <div v-for="(errors, field) in validationErrors" :key="field">
        <strong>{{ field }}:</strong>
        <ul>
          <li v-for="error in errors" :key="error" class="error">
            {{ error }}
          </li>
        </ul>
      </div>
    </div>
    
    <!-- 디바운스된 검색 -->
    <div class="search-section">
      <h3>실시간 검색:</h3>
      <input v-model="searchQuery" placeholder="검색어 입력..." />
      <div v-if="searchLoading">검색 중...</div>
      <div v-else-if="searchResults.length">
        <div v-for="result in searchResults" :key="result.id">
          {{ result.title }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { 
  ref, 
  reactive, 
  computed, 
  watch, 
  watchEffect,
  debouncedWatch,
  toRefs,
  readonly
} from 'vue'

export default {
  setup() {
    // 복잡한 폼 상태
    const formData = reactive({
      name: '',
      email: '',
      age: null,
      password: '',
      confirmPassword: ''
    })
    
    // 폼 필드 정의
    const formFields = readonly([
      { key: 'name', label: '이름', type: 'text', placeholder: '이름을 입력하세요' },
      { key: 'email', label: '이메일', type: 'email', placeholder: 'email@example.com' },
      { key: 'age', label: '나이', type: 'number', placeholder: '나이를 입력하세요' },
      { key: 'password', label: '비밀번호', type: 'password', placeholder: '비밀번호' },
      { key: 'confirmPassword', label: '비밀번호 확인', type: 'password', placeholder: '비밀번호 확인' }
    ])
    
    // 검증 규칙
    const validationRules = {
      name: [
        { validator: (value) => !!value?.trim(), message: '이름은 필수입니다' },
        { validator: (value) => value?.length >= 2, message: '이름은 2글자 이상이어야 합니다' }
      ],
      email: [
        { validator: (value) => !!value?.trim(), message: '이메일은 필수입니다' },
        { validator: (value) => /\S+@\S+\.\S+/.test(value), message: '유효한 이메일 형식이 아닙니다' }
      ],
      age: [
        { validator: (value) => value !== null && value !== '', message: '나이는 필수입니다' },
        { validator: (value) => value >= 14 && value <= 100, message: '나이는 14-100 사이여야 합니다' }
      ],
      password: [
        { validator: (value) => !!value, message: '비밀번호는 필수입니다' },
        { validator: (value) => value?.length >= 8, message: '비밀번호는 8글자 이상이어야 합니다' },
        { validator: (value) => /(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value), message: '대소문자와 숫자를 포함해야 합니다' }
      ],
      confirmPassword: [
        { validator: (value) => value === formData.password, message: '비밀번호가 일치하지 않습니다' }
      ]
    }
    
    // 실시간 검증
    const validationErrors = ref({})
    
    const validateField = (fieldName, value) => {
      const rules = validationRules[fieldName] || []
      const errors = rules
        .filter(rule => !rule.validator(value))
        .map(rule => rule.message)
      
      if (errors.length > 0) {
        validationErrors.value[fieldName] = errors
      } else {
        delete validationErrors.value[fieldName]
      }
    }
    
    // 모든 필드 변경 감지
    watchEffect(() => {
      Object.keys(formData).forEach(field => {
        validateField(field, formData[field])
      })
    })
    
    // 폼 유효성 체크
    const isFormValid = computed(() => {
      return Object.keys(validationErrors.value).length === 0 &&
             Object.values(formData).every(value => value !== '' && value !== null)
    })
    
    // 디바운스된 검색
    const searchQuery = ref('')
    const searchResults = ref([])
    const searchLoading = ref(false)
    
    // 검색 함수
    const performSearch = async (query) => {
      if (!query.trim()) {
        searchResults.value = []
        return
      }
      
      searchLoading.value = true
      
      try {
        // API 호출 시뮬레이션
        await new Promise(resolve => setTimeout(resolve, 500))
        
        searchResults.value = [
          { id: 1, title: `"${query}"에 대한 결과 1` },
          { id: 2, title: `"${query}"에 대한 결과 2` },
          { id: 3, title: `"${query}"에 대한 결과 3` }
        ]
      } catch (error) {
        console.error('Search failed:', error)
        searchResults.value = []
      } finally {
        searchLoading.value = false
      }
    }
    
    // 디바운스된 검색 (300ms 딜레이)
    watch(searchQuery, (newQuery) => {
      searchLoading.value = true
      debounce(() => performSearch(newQuery), 300)()
    })
    
    // 디바운스 헬퍼 함수
    let debounceTimer = null
    const debounce = (func, delay) => {
      return (...args) => {
        clearTimeout(debounceTimer)
        debounceTimer = setTimeout(() => func.apply(null, args), delay)
      }
    }
    
    // 폼 제출
    const handleSubmit = () => {
      if (isFormValid.value) {
        console.log('Form submitted:', { ...formData })
        alert('폼이 성공적으로 제출되었습니다!')
      }
    }
    
    // 컴포넌트 상태 디버깅
    watchEffect(() => {
      console.log('Form state changed:', {
        formData: { ...formData },
        isValid: isFormValid.value,
        errors: { ...validationErrors.value }
      })
    })
    
    return {
      formData,
      formFields,
      validationErrors,
      isFormValid,
      handleSubmit,
      searchQuery,
      searchResults,
      searchLoading
    }
  }
}
</script>

<style scoped>
.advanced-demo {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

form > div {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.validation-results {
  margin: 20px 0;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 4px;
}

.error {
  color: #dc3545;
  font-size: 14px;
}

.search-section {
  margin-top: 30px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

button {
  background: #42b883;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  background: #ccc;
  cursor: not-allowed;
}
</style>
```

## 🎯 4. 실습 예제: 실시간 대시보드

```vue
<template>
  <div class="realtime-dashboard">
    <h1>📊 실시간 대시보드</h1>
    
    <!-- 연결 상태 -->
    <div class="connection-status" :class="connectionStatus">
      {{ connectionStatusText }}
    </div>
    
    <!-- 메트릭 카드들 -->
    <div class="metrics-grid">
      <MetricCard
        v-for="metric in metrics"
        :key="metric.id"
        :title="metric.title"
        :value="metric.value"
        :change="metric.change"
        :trend="metric.trend"
      />
    </div>
    
    <!-- 실시간 차트 -->
    <div class="chart-section">
      <h2>실시간 트래픽</h2>
      <canvas ref="chartCanvas" width="800" height="300"></canvas>
    </div>
    
    <!-- 실시간 로그 -->
    <div class="logs-section">
      <h2>실시간 로그</h2>
      <div class="log-container" ref="logContainer">
        <div
          v-for="log in recentLogs"
          :key="log.id"
          :class="['log-entry', log.level]"
        >
          <span class="timestamp">{{ formatTime(log.timestamp) }}</span>
          <span class="level">{{ log.level.toUpperCase() }}</span>
          <span class="message">{{ log.message }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { 
  ref, 
  reactive, 
  computed, 
  onMounted, 
  onUnmounted, 
  nextTick,
  watch
} from 'vue'
import MetricCard from '@/components/MetricCard.vue'

export default {
  components: { MetricCard },
  
  setup() {
    // WebSocket 연결 상태
    const ws = ref(null)
    const connectionStatus = ref('disconnected') // connected, connecting, disconnected
    
    // 메트릭 데이터
    const metrics = reactive([
      { id: 'users', title: '활성 사용자', value: 0, change: 0, trend: 'stable' },
      { id: 'requests', title: '요청/분', value: 0, change: 0, trend: 'up' },
      { id: 'errors', title: '에러율', value: 0, change: 0, trend: 'down' },
      { id: 'latency', title: '평균 응답시간', value: 0, change: 0, trend: 'stable' }
    ])
    
    // 차트 데이터
    const chartData = reactive({
      labels: [],
      datasets: [{
        label: 'Traffic',
        data: [],
        borderColor: '#42b883',
        tension: 0.1
      }]
    })
    
    // 로그 데이터
    const logs = ref([])
    const maxLogs = 100
    
    // 차트 캔버스 참조
    const chartCanvas = ref(null)
    const logContainer = ref(null)
    let chart = null
    
    // 계산된 속성들
    const connectionStatusText = computed(() => {
      const statusMap = {
        connected: '🟢 연결됨',
        connecting: '🟡 연결 중...',
        disconnected: '🔴 연결 끊김'
      }
      return statusMap[connectionStatus.value]
    })
    
    const recentLogs = computed(() => {
      return logs.value.slice(-50).reverse()
    })
    
    // WebSocket 연결
    const connectWebSocket = () => {
      connectionStatus.value = 'connecting'
      
      ws.value = new WebSocket('ws://localhost:8080/dashboard')
      
      ws.value.onopen = () => {
        connectionStatus.value = 'connected'
        console.log('WebSocket connected')
      }
      
      ws.value.onmessage = (event) => {
        const data = JSON.parse(event.data)
        handleWebSocketMessage(data)
      }
      
      ws.value.onclose = () => {
        connectionStatus.value = 'disconnected'
        console.log('WebSocket disconnected')
        
        // 3초 후 재연결 시도
        setTimeout(connectWebSocket, 3000)
      }
      
      ws.value.onerror = (error) => {
        console.error('WebSocket error:', error)
        connectionStatus.value = 'disconnected'
      }
    }
    
    // 메시지 처리
    const handleWebSocketMessage = (data) => {
      switch (data.type) {
        case 'metrics':
          updateMetrics(data.payload)
          break
          
        case 'chart':
          updateChart(data.payload)
          break
          
        case 'log':
          addLog(data.payload)
          break
          
        default:
          console.warn('Unknown message type:', data.type)
      }
    }
    
    // 메트릭 업데이트
    const updateMetrics = (newMetrics) => {
      newMetrics.forEach(newMetric => {
        const metric = metrics.find(m => m.id === newMetric.id)
        if (metric) {
          const oldValue = metric.value
          metric.value = newMetric.value
          metric.change = newMetric.value - oldValue
          metric.trend = newMetric.trend || 'stable'
        }
      })
    }
    
    // 차트 업데이트
    const updateChart = (data) => {
      const now = new Date()
      const timeLabel = now.toLocaleTimeString()
      
      // 최대 20개 데이터 포인트 유지
      if (chartData.labels.length >= 20) {
        chartData.labels.shift()
        chartData.datasets[0].data.shift()
      }
      
      chartData.labels.push(timeLabel)
      chartData.datasets[0].data.push(data.value)
      
      // Chart.js 업데이트
      if (chart) {
        chart.update('none')
      }
    }
    
    // 로그 추가
    const addLog = (logEntry) => {
      logs.value.push({
        ...logEntry,
        id: Date.now() + Math.random(),
        timestamp: new Date()
      })
      
      // 최대 로그 수 유지
      if (logs.value.length > maxLogs) {
        logs.value.shift()
      }
      
      // 로그 컨테이너 자동 스크롤
      nextTick(() => {
        if (logContainer.value) {
          logContainer.value.scrollTop = logContainer.value.scrollHeight
        }
      })
    }
    
    // 차트 초기화
    const initChart = async () => {
      if (!chartCanvas.value) return
      
      const { Chart, registerables } = await import('chart.js')
      Chart.register(...registerables)
      
      chart = new Chart(chartCanvas.value, {
        type: 'line',
        data: chartData,
        options: {
          responsive: true,
          scales: {
            y: {
              beginAtZero: true
            }
          },
          plugins: {
            legend: {
              display: false
            }
          }
        }
      })
    }
    
    // 시간 포맷팅
    const formatTime = (date) => {
      return date.toLocaleTimeString('ko-KR')
    }
    
    // Mock 데이터 생성 (실제 환경에서는 제거)
    const generateMockData = () => {
      // Mock 메트릭 업데이트
      updateMetrics([
        { id: 'users', value: Math.floor(Math.random() * 1000) + 500, trend: 'up' },
        { id: 'requests', value: Math.floor(Math.random() * 100) + 50, trend: 'stable' },
        { id: 'errors', value: Math.random() * 5, trend: 'down' },
        { id: 'latency', value: Math.floor(Math.random() * 100) + 50, trend: 'stable' }
      ])
      
      // Mock 차트 데이터
      updateChart({ value: Math.floor(Math.random() * 100) + 20 })
      
      // Mock 로그
      const logLevels = ['info', 'warning', 'error']
      const logMessages = [
        'User login successful',
        'API request processed',
        'Database connection established',
        'Warning: High memory usage',
        'Error: Failed to process request',
        'Cache cleared successfully'
      ]
      
      addLog({
        level: logLevels[Math.floor(Math.random() * logLevels.length)],
        message: logMessages[Math.floor(Math.random() * logMessages.length)]
      })
    }
    
    // 생명주기
    onMounted(async () => {
      await initChart()
      connectWebSocket()
      
      // Mock 데이터 생성 (개발용)
      const mockInterval = setInterval(generateMockData, 2000)
      
      // 컴포넌트 언마운트 시 정리
      onUnmounted(() => {
        clearInterval(mockInterval)
        if (ws.value) {
          ws.value.close()
        }
        if (chart) {
          chart.destroy()
        }
      })
    })
    
    return {
      connectionStatus,
      connectionStatusText,
      metrics,
      recentLogs,
      chartCanvas,
      logContainer,
      formatTime
    }
  }
}
</script>

<style scoped>
.realtime-dashboard {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.connection-status {
  padding: 10px 15px;
  border-radius: 20px;
  font-weight: bold;
  margin-bottom: 20px;
  display: inline-block;
}

.connection-status.connected {
  background: #d4edda;
  color: #155724;
}

.connection-status.connecting {
  background: #fff3cd;
  color: #856404;
}

.connection-status.disconnected {
  background: #f8d7da;
  color: #721c24;
}

.metrics-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
  margin-bottom: 30px;
}

.chart-section,
.logs-section {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  margin-bottom: 20px;
}

.log-container {
  height: 300px;
  overflow-y: auto;
  border: 1px solid #e0e0e0;
  border-radius: 4px;
  padding: 10px;
  background: #f8f9fa;
}

.log-entry {
  display: flex;
  gap: 10px;
  padding: 5px 0;
  border-bottom: 1px solid #e0e0e0;
  font-family: monospace;
  font-size: 12px;
}

.log-entry:last-child {
  border-bottom: none;
}

.timestamp {
  color: #666;
  min-width: 80px;
}

.level {
  min-width: 60px;
  font-weight: bold;
}

.level.info { color: #007bff; }
.level.warning { color: #ffc107; }
.level.error { color: #dc3545; }

.message {
  flex: 1;
}
</style>
```

## 🎯 실습 과제

1. 위의 실시간 대시보드를 직접 구현해보세요
2. WebSocket 연결 관리를 위한 composable을 만들어보세요
3. 차트 데이터 관리를 위한 composable을 분리해보세요
4. 로그 필터링 기능을 추가해보세요
5. 알림 시스템을 구현해보세요

## 🔗 다음 단계

다음 챕터에서는 커스텀 Composables 제작에 대해 더 자세히 알아보겠습니다!

---

💡 **팁**: Composition API는 React Hooks보다 더 직관적이고 TypeScript와의 호환성도 뛰어납니다. 로직 재사용과 테스트도 더 쉬워요! 