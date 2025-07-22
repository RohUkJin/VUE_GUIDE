# 2-4. 라이프사이클 (Lifecycle)

## 🔄 Vue vs React: 컴포넌트 라이프사이클

Vue와 React 모두 컴포넌트의 생명주기를 관리하는 방법을 제공합니다. Vue는 더 직관적이고 명확한 라이프사이클 훅을 제공해요!

## 📊 1. 라이프사이클 비교 개요

### React 라이프사이클 (Hooks)
```jsx
import { useState, useEffect, useLayoutEffect } from 'react';

function MyComponent() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  
  // componentDidMount + componentDidUpdate
  useEffect(() => {
    console.log('Component mounted or updated');
  });
  
  // componentDidMount only
  useEffect(() => {
    console.log('Component mounted');
    loadData();
    
    // componentWillUnmount
    return () => {
      console.log('Component will unmount');
      cleanup();
    };
  }, []);
  
  // componentDidUpdate for specific state
  useEffect(() => {
    console.log('Data changed:', data);
  }, [data]);
  
  // Before DOM update (like componentWillUpdate)
  useLayoutEffect(() => {
    console.log('Before DOM update');
  });
  
  const loadData = async () => {
    setLoading(true);
    const result = await fetchData();
    setData(result);
    setLoading(false);
  };
  
  const cleanup = () => {
    // 이벤트 리스너 제거, 타이머 정리 등
  };
  
  return <div>{loading ? 'Loading...' : JSON.stringify(data)}</div>;
}
```

### Vue 라이프사이클 (Options API)
```vue
<template>
  <div>{{ loading ? 'Loading...' : JSON.stringify(data) }}</div>
</template>

<script>
export default {
  data() {
    return {
      data: null,
      loading: true
    }
  },
  
  // 인스턴스 생성 후
  created() {
    console.log('Component created');
    // 여기서 API 호출 등 초기화 작업
    this.loadData();
  },
  
  // DOM 마운트 후
  mounted() {
    console.log('Component mounted');
    // DOM 조작이 필요한 작업
    this.setupEventListeners();
  },
  
  // 데이터 변경 후 DOM 업데이트 전
  beforeUpdate() {
    console.log('Before DOM update');
  },
  
  // DOM 업데이트 후
  updated() {
    console.log('After DOM update');
  },
  
  // 컴포넌트 제거 전
  beforeUnmount() {
    console.log('Before component unmount');
    this.cleanup();
  },
  
  // 컴포넌트 제거 후
  unmounted() {
    console.log('Component unmounted');
  },
  
  // 특정 데이터 변화 감지
  watch: {
    data(newValue, oldValue) {
      console.log('Data changed:', oldValue, '→', newValue);
    }
  },
  
  methods: {
    async loadData() {
      this.loading = true;
      this.error = null;
      
      try {
        const response = await fetch(`/api/users/${this.userId}`);
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        this.user = await response.json();
        
      } catch (error) {
        this.error = error;
        console.error('Failed to load user data:', error);
        
        // 자동 재시도 (최대 3회)
        if (this.retryCount < 3) {
          this.retryCount++;
          setTimeout(() => this.loadUserData(), 2000);
        }
        
      } finally {
        this.loading = false;
      }
    },
    
    async loadUserStats() {
      try {
        const response = await fetch(`/api/users/${this.userId}/stats`);
        if (response.ok) {
          this.userStats = await response.json();
        }
      } catch (error) {
        console.error('Failed to load user stats:', error);
      }
    },
    
    async retryLoad() {
      this.retryCount = 0;
      await this.loadUserData();
      if (this.user) {
        await this.loadUserStats();
      }
    },
    
    cancelRequests() {
      // AbortController를 사용한 요청 취소 로직
    }
  }
}
</script>
```

## 🎯 2. 실용적인 라이프사이클 활용

### 데이터 로딩과 에러 처리
```vue
<template>
  <div class="user-profile">
    <!-- 로딩 상태 -->
    <div v-if="loading" class="loading">
      <div class="spinner"></div>
      <p>사용자 정보를 불러오는 중...</p>
    </div>
    
    <!-- 에러 상태 -->
    <div v-else-if="error" class="error">
      <h3>⚠️ 오류가 발생했습니다</h3>
      <p>{{ error.message }}</p>
      <button @click="retryLoad">다시 시도</button>
    </div>
    
    <!-- 성공 상태 -->
    <div v-else-if="user" class="user-content">
      <img :src="user.avatar" :alt="user.name" />
      <h2>{{ user.name }}</h2>
      <p>{{ user.email }}</p>
      
      <!-- 동적으로 로드된 추가 정보 -->
      <div v-if="userStats" class="user-stats">
        <h3>통계</h3>
        <p>게시글: {{ userStats.posts }}개</p>
        <p>팔로워: {{ userStats.followers }}명</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    userId: {
      type: [String, Number],
      required: true
    }
  },
  
  data() {
    return {
      user: null,
      userStats: null,
      loading: false,
      error: null,
      retryCount: 0
    }
  },
  
  async created() {
    // 컴포넌트 생성 시 데이터 로드
    await this.loadUserData();
  },
  
  async mounted() {
    // DOM 마운트 후 추가 데이터 로드
    if (this.user) {
      await this.loadUserStats();
    }
  },
  
  watch: {
    // userId prop 변경 시 데이터 재로드
    async userId(newId, oldId) {
      if (newId !== oldId) {
        await this.loadUserData();
        await this.loadUserStats();
      }
    }
  },
  
  beforeUnmount() {
    // 진행 중인 요청 취소
    this.cancelRequests();
  },
  
  methods: {
    async loadUserData() {
      this.loading = true;
      this.error = null;
      
      try {
        const response = await fetch(`/api/users/${this.userId}`);
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        this.user = await response.json();
        
      } catch (error) {
        this.error = error;
        console.error('Failed to load user data:', error);
        
        // 자동 재시도 (최대 3회)
        if (this.retryCount < 3) {
          this.retryCount++;
          setTimeout(() => this.loadUserData(), 2000);
        }
        
      } finally {
        this.loading = false;
      }
    },
    
    async loadUserStats() {
      try {
        const response = await fetch(`/api/users/${this.userId}/stats`);
        if (response.ok) {
          this.userStats = await response.json();
        }
      } catch (error) {
        console.error('Failed to load user stats:', error);
      }
    },
    
    async retryLoad() {
      this.retryCount = 0;
      await this.loadUserData();
      if (this.user) {
        await this.loadUserStats();
      }
    },
    
    cancelRequests() {
      // AbortController를 사용한 요청 취소 로직
    }
  }
}
</script>
```

### 실시간 데이터 업데이트
```vue
<template>
  <div class="real-time-dashboard">
    <h1>📊 실시간 대시보드</h1>
    
    <div class="connection-status">
      <span :class="['status', connectionStatus]">
        {{ connectionStatusText }}
      </span>
      <button v-if="connectionStatus === 'disconnected'" @click="reconnect">
        재연결
      </button>
    </div>
    
    <div class="metrics-grid">
      <MetricCard
        v-for="metric in metrics"
        :key="metric.id"
        :title="metric.title"
        :value="metric.value"
        :trend="metric.trend"
        :last-updated="metric.lastUpdated"
      />
    </div>
    
    <div class="activity-feed">
      <h3>실시간 활동</h3>
      <div class="activities">
        <div
          v-for="activity in recentActivities"
          :key="activity.id"
          class="activity-item"
        >
          <span class="timestamp">{{ formatTime(activity.timestamp) }}</span>
          <span class="message">{{ activity.message }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import MetricCard from './MetricCard.vue'

export default {
  components: { MetricCard },
  
  data() {
    return {
      websocket: null,
      connectionStatus: 'connecting', // connecting, connected, disconnected
      metrics: [],
      recentActivities: [],
      reconnectAttempts: 0,
      maxReconnectAttempts: 5,
      reconnectInterval: null
    }
  },
  
  computed: {
    connectionStatusText() {
      const statusMap = {
        connecting: '연결 중...',
        connected: '연결됨',
        disconnected: '연결 끊김'
      };
      return statusMap[this.connectionStatus];
    }
  },
  
  mounted() {
    this.connectWebSocket();
    
    // 페이지 가시성 변경 시 처리
    document.addEventListener('visibilitychange', this.handleVisibilityChange);
  },
  
  beforeUnmount() {
    this.disconnectWebSocket();
    document.removeEventListener('visibilitychange', this.handleVisibilityChange);
    
    if (this.reconnectInterval) {
      clearInterval(this.reconnectInterval);
    }
  },
  
  methods: {
    connectWebSocket() {
      try {
        this.websocket = new WebSocket('ws://localhost:8080/dashboard');
        
        this.websocket.onopen = () => {
          console.log('WebSocket connected');
          this.connectionStatus = 'connected';
          this.reconnectAttempts = 0;
          
          // 초기 데이터 요청
          this.websocket.send(JSON.stringify({ type: 'get_initial_data' }));
        };
        
        this.websocket.onmessage = (event) => {
          const data = JSON.parse(event.data);
          this.handleWebSocketMessage(data);
        };
        
        this.websocket.onclose = (event) => {
          console.log('WebSocket disconnected:', event.code, event.reason);
          this.connectionStatus = 'disconnected';
          
          // 자동 재연결 시도
          if (this.reconnectAttempts < this.maxReconnectAttempts) {
            this.scheduleReconnect();
          }
        };
        
        this.websocket.onerror = (error) => {
          console.error('WebSocket error:', error);
          this.connectionStatus = 'disconnected';
        };
        
      } catch (error) {
        console.error('Failed to create WebSocket:', error);
        this.connectionStatus = 'disconnected';
      }
    },
    
    disconnectWebSocket() {
      if (this.websocket) {
        this.websocket.close();
        this.websocket = null;
      }
    },
    
    handleWebSocketMessage(data) {
      switch (data.type) {
        case 'initial_data':
          this.metrics = data.metrics;
          this.recentActivities = data.activities.slice(0, 10);
          break;
          
        case 'metric_update':
          const metricIndex = this.metrics.findIndex(m => m.id === data.metric.id);
          if (metricIndex !== -1) {
            this.metrics.splice(metricIndex, 1, data.metric);
          }
          break;
          
        case 'new_activity':
          this.recentActivities.unshift(data.activity);
          if (this.recentActivities.length > 50) {
            this.recentActivities.pop();
          }
          break;
          
        default:
          console.warn('Unknown message type:', data.type);
      }
    },
    
    scheduleReconnect() {
      this.reconnectAttempts++;
      const delay = Math.min(1000 * Math.pow(2, this.reconnectAttempts), 30000);
      
      console.log(`Reconnecting in ${delay}ms (attempt ${this.reconnectAttempts}/${this.maxReconnectAttempts})`);
      
      setTimeout(() => {
        if (this.connectionStatus === 'disconnected') {
          this.connectionStatus = 'connecting';
          this.connectWebSocket();
        }
      }, delay);
    },
    
    reconnect() {
      this.reconnectAttempts = 0;
      this.connectionStatus = 'connecting';
      this.connectWebSocket();
    },
    
    handleVisibilityChange() {
      if (document.hidden) {
        // 페이지가 숨겨졌을 때 (탭 변경 등)
        console.log('Page hidden, reducing activity');
      } else {
        // 페이지가 다시 보일 때
        console.log('Page visible, resuming activity');
        if (this.connectionStatus === 'disconnected') {
          this.reconnect();
        }
      }
    },
    
    formatTime(timestamp) {
      return new Date(timestamp).toLocaleTimeString('ko-KR');
    }
  }
}
</script>
```

## 🎣 3. Composition API에서의 라이프사이클

```vue
<template>
  <div class="composition-example">
    <h2>사용자 검색</h2>
    
    <input 
      v-model="searchQuery" 
      placeholder="사용자명을 입력하세요..."
      @keyup.enter="searchUsers"
    />
    
    <div v-if="loading">검색 중...</div>
    
    <div v-else-if="users.length > 0" class="user-list">
      <div
        v-for="user in users"
        :key="user.id"
        class="user-item"
        @click="selectUser(user)"
      >
        <img :src="user.avatar" :alt="user.name" />
        <div>
          <h4>{{ user.name }}</h4>
          <p>{{ user.email }}</p>
        </div>
      </div>
    </div>
    
    <div v-else-if="searchQuery && !loading">
      검색 결과가 없습니다.
    </div>
  </div>
</template>

<script>
import { ref, watch, onMounted, onUnmounted, nextTick } from 'vue'

export default {
  setup() {
    // 반응형 데이터
    const searchQuery = ref('')
    const users = ref([])
    const loading = ref(false)
    const searchController = ref(null)
    
    // 검색 함수
    const searchUsers = async () => {
      if (!searchQuery.value.trim()) {
        users.value = []
        return
      }
      
      // 이전 요청 취소
      if (searchController.value) {
        searchController.value.abort()
      }
      
      searchController.value = new AbortController()
      loading.value = true
      
      try {
        const response = await fetch(`/api/search/users?q=${encodeURIComponent(searchQuery.value)}`, {
          signal: searchController.value.signal
        })
        
        if (!response.ok) throw new Error('Search failed')
        
        const data = await response.json()
        users.value = data.users
        
      } catch (error) {
        if (error.name !== 'AbortError') {
          console.error('Search error:', error)
          users.value = []
        }
      } finally {
        loading.value = false
      }
    }
    
    const selectUser = (user) => {
      console.log('Selected user:', user)
      // 사용자 선택 처리
    }
    
    // 라이프사이클 훅들
    onMounted(() => {
      console.log('Component mounted')
      
      // 포커스 설정
      nextTick(() => {
        const input = document.querySelector('input')
        if (input) input.focus()
      })
    })
    
    onUnmounted(() => {
      console.log('Component unmounted')
      
      // 진행 중인 요청 취소
      if (searchController.value) {
        searchController.value.abort()
      }
    })
    
    // 디바운스된 검색
    let searchTimeout = null
    watch(searchQuery, (newQuery) => {
      if (searchTimeout) {
        clearTimeout(searchTimeout)
      }
      
      searchTimeout = setTimeout(() => {
        searchUsers()
      }, 300)
    })
    
    return {
      searchQuery,
      users,
      loading,
      searchUsers,
      selectUser
    }
  }
}
</script>
```

## 🔄 4. 커스텀 훅 (Composables)

```javascript
// composables/useApi.js
import { ref, onUnmounted } from 'vue'

export function useApi() {
  const loading = ref(false)
  const error = ref(null)
  const controllers = ref(new Set())
  
  const request = async (url, options = {}) => {
    const controller = new AbortController()
    controllers.value.add(controller)
    
    loading.value = true
    error.value = null
    
    try {
      const response = await fetch(url, {
        ...options,
        signal: controller.signal
      })
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`)
      }
      
      const data = await response.json()
      return data
      
    } catch (err) {
      if (err.name !== 'AbortError') {
        error.value = err
        throw err
      }
    } finally {
      loading.value = false
      controllers.value.delete(controller)
    }
  }
  
  const cancelAll = () => {
    controllers.value.forEach(controller => controller.abort())
    controllers.value.clear()
  }
  
  // 컴포넌트 언마운트 시 모든 요청 취소
  onUnmounted(() => {
    cancelAll()
  })
  
  return {
    loading,
    error,
    request,
    cancelAll
  }
}
```

```javascript
// composables/useLocalStorage.js
import { ref, watch } from 'vue'

export function useLocalStorage(key, defaultValue) {
  const storedValue = localStorage.getItem(key)
  const initialValue = storedValue ? JSON.parse(storedValue) : defaultValue
  
  const value = ref(initialValue)
  
  // 값 변경 시 localStorage 업데이트
  watch(value, (newValue) => {
    localStorage.setItem(key, JSON.stringify(newValue))
  }, { deep: true })
  
  // 다른 탭에서 값 변경 시 동기화
  window.addEventListener('storage', (e) => {
    if (e.key === key && e.newValue) {
      value.value = JSON.parse(e.newValue)
    }
  })
  
  return value
}
```

## 🎯 5. 실습 예제: 채팅 애플리케이션

```vue
<!-- ChatApp.vue -->
<template>
  <div class="chat-app">
    <div class="chat-header">
      <h2>💬 실시간 채팅</h2>
      <div class="connection-status" :class="connectionStatus">
        {{ connectionStatus === 'connected' ? '연결됨' : '연결 중...' }}
      </div>
    </div>
    
    <div class="chat-messages" ref="messagesContainer">
      <div
        v-for="message in messages"
        :key="message.id"
        :class="['message', { own: message.userId === currentUserId }]"
      >
        <div class="message-header">
          <span class="username">{{ message.username }}</span>
          <span class="timestamp">{{ formatTime(message.timestamp) }}</span>
        </div>
        <div class="message-content">{{ message.content }}</div>
      </div>
      
      <div v-if="typingUsers.length > 0" class="typing-indicator">
        {{ typingText }}
      </div>
    </div>
    
    <div class="chat-input">
      <input
        v-model="newMessage"
        @keyup.enter="sendMessage"
        @input="handleTyping"
        placeholder="메시지를 입력하세요..."
        :disabled="connectionStatus !== 'connected'"
      />
      <button @click="sendMessage" :disabled="!canSend">전송</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      websocket: null,
      connectionStatus: 'connecting',
      messages: [],
      newMessage: '',
      typingUsers: [],
      typingTimeout: null,
      currentUserId: 'user123',
      currentUsername: '김개발'
    }
  },
  
  computed: {
    canSend() {
      return this.newMessage.trim() && this.connectionStatus === 'connected'
    },
    
    typingText() {
      if (this.typingUsers.length === 1) {
        return `${this.typingUsers[0]}님이 입력 중입니다...`
      } else if (this.typingUsers.length > 1) {
        return `${this.typingUsers.length}명이 입력 중입니다...`
      }
      return ''
    }
  },
  
  mounted() {
    this.connectWebSocket()
    this.loadMessageHistory()
  },
  
  beforeUnmount() {
    this.disconnectWebSocket()
  },
  
  updated() {
    // 새 메시지가 추가될 때마다 스크롤을 하단으로
    this.$nextTick(() => {
      this.scrollToBottom()
    })
  },
  
  methods: {
    connectWebSocket() {
      this.websocket = new WebSocket('ws://localhost:8080/chat')
      
      this.websocket.onopen = () => {
        this.connectionStatus = 'connected'
        
        // 채팅방 입장 알림
        this.websocket.send(JSON.stringify({
          type: 'join',
          userId: this.currentUserId,
          username: this.currentUsername
        }))
      }
      
      this.websocket.onmessage = (event) => {
        const data = JSON.parse(event.data)
        this.handleWebSocketMessage(data)
      }
      
      this.websocket.onclose = () => {
        this.connectionStatus = 'disconnected'
        // 자동 재연결 시도
        setTimeout(() => this.connectWebSocket(), 3000)
      }
      
      this.websocket.onerror = (error) => {
        console.error('WebSocket error:', error)
      }
    },
    
    disconnectWebSocket() {
      if (this.websocket) {
        this.websocket.close()
      }
    },
    
    handleWebSocketMessage(data) {
      switch (data.type) {
        case 'message':
          this.messages.push(data.message)
          break
          
        case 'typing':
          if (data.userId !== this.currentUserId) {
            if (!this.typingUsers.includes(data.username)) {
              this.typingUsers.push(data.username)
            }
            
            // 3초 후 타이핑 상태 제거
            setTimeout(() => {
              const index = this.typingUsers.indexOf(data.username)
              if (index > -1) {
                this.typingUsers.splice(index, 1)
              }
            }, 3000)
          }
          break
          
        case 'stop_typing':
          if (data.userId !== this.currentUserId) {
            const index = this.typingUsers.indexOf(data.username)
            if (index > -1) {
              this.typingUsers.splice(index, 1)
            }
          }
          break
      }
    },
    
    sendMessage() {
      if (!this.canSend) return
      
      const message = {
        type: 'message',
        content: this.newMessage,
        userId: this.currentUserId,
        username: this.currentUsername,
        timestamp: Date.now()
      }
      
      this.websocket.send(JSON.stringify(message))
      this.newMessage = ''
      
      // 타이핑 중지 알림
      this.sendStopTyping()
    },
    
    handleTyping() {
      // 타이핑 시작 알림
      if (this.websocket && this.websocket.readyState === WebSocket.OPEN) {
        this.websocket.send(JSON.stringify({
          type: 'typing',
          userId: this.currentUserId,
          username: this.currentUsername
        }))
      }
      
      // 3초 후 타이핑 중지
      if (this.typingTimeout) {
        clearTimeout(this.typingTimeout)
      }
      
      this.typingTimeout = setTimeout(() => {
        this.sendStopTyping()
      }, 3000)
    },
    
    sendStopTyping() {
      if (this.websocket && this.websocket.readyState === WebSocket.OPEN) {
        this.websocket.send(JSON.stringify({
          type: 'stop_typing',
          userId: this.currentUserId,
          username: this.currentUsername
        }))
      }
    },
    
    async loadMessageHistory() {
      try {
        const response = await fetch('/api/messages/recent')
        const data = await response.json()
        this.messages = data.messages || []
      } catch (error) {
        console.error('Failed to load message history:', error)
      }
    },
    
    scrollToBottom() {
      const container = this.$refs.messagesContainer
      if (container) {
        container.scrollTop = container.scrollHeight
      }
    },
    
    formatTime(timestamp) {
      return new Date(timestamp).toLocaleTimeString('ko-KR', {
        hour: '2-digit',
        minute: '2-digit'
      })
    }
  }
}
</script>
```

## 🎯 실습 과제

1. 위의 채팅 애플리케이션을 직접 만들어보세요
2. 메시지 읽음 상태 표시 기능을 추가해보세요
3. 파일 업로드와 이미지 미리보기 기능을 구현해보세요
4. 사용자 온라인 상태 표시 기능을 추가해보세요
5. 메시지 검색 기능을 구현해보세요

## 🔗 다음 단계

다음 챕터에서는 Vue의 커스텀 디렉티브에 대해 알아보겠습니다!

---

💡 **팁**: Vue의 라이프사이클 훅은 React의 useEffect보다 더 명확하고 직관적입니다. 각 단계별로 명확한 용도가 있어서 코드 의도를 파악하기 쉬워요! 