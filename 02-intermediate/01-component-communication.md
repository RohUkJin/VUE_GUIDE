# 2-1. 컴포넌트 통신 (Component Communication)

## 🔗 Vue 컴포넌트 통신 vs React

Vue와 React 모두 컴포넌트 간 데이터를 주고받는 다양한 방법을 제공합니다. Vue는 더 다양하고 편리한 통신 방법을 제공해요!

## 📤 1. Props Down (부모 → 자식)

### React 방식
```jsx
// Parent.jsx
function Parent() {
  const [user, setUser] = useState({ name: '김개발', age: 30 });
  const [theme, setTheme] = useState('light');
  
  return (
    <div>
      <Child user={user} theme={theme} />
      <Settings onThemeChange={setTheme} />
    </div>
  );
}

// Child.jsx
function Child({ user, theme }) {
  return (
    <div className={`child ${theme}`}>
      <p>{user.name} ({user.age}세)</p>
    </div>
  );
}
```

### Vue 방식
```vue
<!-- Parent.vue -->
<template>
  <div>
    <Child :user="user" :theme="theme" />
    <Settings @theme-change="handleThemeChange" />
  </div>
</template>

<script>
import Child from './Child.vue'
import Settings from './Settings.vue'

export default {
  components: { Child, Settings },
  data() {
    return {
      user: { name: '김개발', age: 30 },
      theme: 'light'
    }
  },
  methods: {
    handleThemeChange(newTheme) {
      this.theme = newTheme;
    }
  }
}
</script>
```

```vue
<!-- Child.vue -->
<template>
  <div :class="`child ${theme}`">
    <p>{{ user.name }} ({{ user.age }}세)</p>
  </div>
</template>

<script>
export default {
  props: {
    user: {
      type: Object,
      required: true,
      validator(value) {
        return value.name && typeof value.age === 'number';
      }
    },
    theme: {
      type: String,
      default: 'light',
      validator(value) {
        return ['light', 'dark'].includes(value);
      }
    }
  }
}
</script>
```

## 📤 2. Events Up (자식 → 부모)

### React 방식
```jsx
// Parent.jsx
function Parent() {
  const [notifications, setNotifications] = useState([]);
  
  const addNotification = (message) => {
    const newNotification = {
      id: Date.now(),
      message,
      timestamp: new Date()
    };
    setNotifications(prev => [newNotification, ...prev]);
  };
  
  const removeNotification = (id) => {
    setNotifications(prev => prev.filter(n => n.id !== id));
  };
  
  return (
    <div>
      <TodoForm onSubmit={addNotification} />
      <NotificationList 
        notifications={notifications}
        onRemove={removeNotification}
      />
    </div>
  );
}
```

### Vue 방식
```vue
<!-- Parent.vue -->
<template>
  <div>
    <TodoForm @submit="addNotification" />
    <NotificationList 
      :notifications="notifications"
      @remove="removeNotification"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      notifications: []
    }
  },
  methods: {
    addNotification(message) {
      const newNotification = {
        id: Date.now(),
        message,
        timestamp: new Date()
      };
      this.notifications.unshift(newNotification);
    },
    
    removeNotification(id) {
      this.notifications = this.notifications.filter(n => n.id !== id);
    }
  }
}
</script>
```

```vue
<!-- TodoForm.vue -->
<template>
  <form @submit.prevent="handleSubmit" class="todo-form">
    <input 
      v-model="todoText" 
      placeholder="할일을 입력하세요..."
      required
    />
    <button type="submit">추가</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      todoText: ''
    }
  },
  methods: {
    handleSubmit() {
      if (this.todoText.trim()) {
        // 부모에게 이벤트 방출
        this.$emit('submit', `새 할일: ${this.todoText}`);
        this.todoText = '';
      }
    }
  }
}
</script>
```

## 🌐 3. Event Bus (형제 컴포넌트 간)

### React 방식 (Context 사용)
```jsx
// EventContext.js
const EventContext = createContext();

export const EventProvider = ({ children }) => {
  const [events, setEvents] = useState([]);
  
  const emitEvent = (type, data) => {
    setEvents(prev => [...prev, { type, data, id: Date.now() }]);
  };
  
  return (
    <EventContext.Provider value={{ events, emitEvent }}>
      {children}
    </EventContext.Provider>
  );
};

// Component A
function ComponentA() {
  const { emitEvent } = useContext(EventContext);
  
  const sendMessage = () => {
    emitEvent('message', 'Hello from A!');
  };
  
  return <button onClick={sendMessage}>Send Message</button>;
}

// Component B
function ComponentB() {
  const { events } = useContext(EventContext);
  const messages = events.filter(e => e.type === 'message');
  
  return (
    <div>
      {messages.map(msg => (
        <p key={msg.id}>{msg.data}</p>
      ))}
    </div>
  );
}
```

### Vue 방식 (Event Bus)
```vue
<!-- eventBus.js -->
<script>
import { createApp } from 'vue'

// 간단한 Event Bus
export const eventBus = createApp({}).config.globalProperties;

// 또는 mitt 라이브러리 사용
// npm install mitt
import mitt from 'mitt'
export const emitter = mitt()
</script>
```

```vue
<!-- ComponentA.vue -->
<template>
  <div class="component-a">
    <h3>Component A</h3>
    <button @click="sendMessage">메시지 보내기</button>
    <button @click="sendData">데이터 보내기</button>
  </div>
</template>

<script>
import { emitter } from './eventBus.js'

export default {
  methods: {
    sendMessage() {
      emitter.emit('message', {
        text: 'Hello from Component A!',
        sender: 'A',
        timestamp: new Date()
      });
    },
    
    sendData() {
      emitter.emit('data', {
        type: 'user-action',
        payload: { action: 'clicked', button: 'send-data' }
      });
    }
  }
}
</script>
```

```vue
<!-- ComponentB.vue -->
<template>
  <div class="component-b">
    <h3>Component B</h3>
    <div class="messages">
      <h4>받은 메시지:</h4>
      <div v-for="msg in messages" :key="msg.id" class="message">
        <strong>{{ msg.sender }}:</strong> {{ msg.text }}
        <small>({{ formatTime(msg.timestamp) }})</small>
      </div>
    </div>
    
    <div class="data-log">
      <h4>데이터 로그:</h4>
      <pre>{{ JSON.stringify(receivedData, null, 2) }}</pre>
    </div>
  </div>
</template>

<script>
import { emitter } from './eventBus.js'

export default {
  data() {
    return {
      messages: [],
      receivedData: []
    }
  },
  
  mounted() {
    // 이벤트 리스너 등록
    emitter.on('message', this.handleMessage);
    emitter.on('data', this.handleData);
  },
  
  beforeUnmount() {
    // 메모리 누수 방지를 위한 이벤트 리스너 제거
    emitter.off('message', this.handleMessage);
    emitter.off('data', this.handleData);
  },
  
  methods: {
    handleMessage(messageData) {
      this.messages.push({
        ...messageData,
        id: Date.now()
      });
    },
    
    handleData(data) {
      this.receivedData.push({
        ...data,
        receivedAt: new Date()
      });
    },
    
    formatTime(date) {
      return date.toLocaleTimeString();
    }
  }
}
</script>
```

## 🎯 4. Provide/Inject (깊은 단계의 Props Drilling 해결)

### React 방식 (Context)
```jsx
// UserContext.js
const UserContext = createContext();

export const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [settings, setSettings] = useState({});
  
  return (
    <UserContext.Provider value={{ user, setUser, settings, setSettings }}>
      {children}
    </UserContext.Provider>
  );
};

// Deep nested component
function DeepChild() {
  const { user, settings } = useContext(UserContext);
  
  return (
    <div>
      <p>User: {user?.name}</p>
      <p>Theme: {settings?.theme}</p>
    </div>
  );
}
```

### Vue 방식 (Provide/Inject)
```vue
<!-- App.vue (최상위 컴포넌트) -->
<template>
  <div>
    <UserProfile />
    <Settings />
  </div>
</template>

<script>
export default {
  data() {
    return {
      user: {
        name: '김개발',
        email: 'dev@example.com',
        role: 'admin'
      },
      settings: {
        theme: 'light',
        language: 'ko',
        notifications: true
      }
    }
  },
  
  provide() {
    return {
      // 반응형 데이터 제공
      user: this.user,
      settings: this.settings,
      // 메서드도 제공 가능
      updateUser: this.updateUser,
      updateSettings: this.updateSettings
    }
  },
  
  methods: {
    updateUser(newUserData) {
      Object.assign(this.user, newUserData);
    },
    
    updateSettings(newSettings) {
      Object.assign(this.settings, newSettings);
    }
  }
}
</script>
```

```vue
<!-- UserProfile.vue (중간 컴포넌트) -->
<template>
  <div class="user-profile">
    <h2>사용자 프로필</h2>
    <UserDetails />
    <UserActions />
  </div>
</template>

<script>
import UserDetails from './UserDetails.vue'
import UserActions from './UserActions.vue'

export default {
  components: { UserDetails, UserActions }
  // 중간 컴포넌트에서는 user 데이터를 직접 사용하지 않아도 됨
  // Props drilling 없이 깊은 자식 컴포넌트가 직접 접근 가능
}
</script>
```

```vue
<!-- UserDetails.vue (깊은 자식 컴포넌트) -->
<template>
  <div class="user-details">
    <h3>사용자 정보</h3>
    <p><strong>이름:</strong> {{ user.name }}</p>
    <p><strong>이메일:</strong> {{ user.email }}</p>
    <p><strong>권한:</strong> {{ user.role }}</p>
    
    <div class="current-settings">
      <h4>현재 설정</h4>
      <p>테마: {{ settings.theme }}</p>
      <p>언어: {{ settings.language }}</p>
      <p>알림: {{ settings.notifications ? '켜짐' : '꺼짐' }}</p>
    </div>
    
    <button @click="handleUpdateProfile">프로필 수정</button>
  </div>
</template>

<script>
export default {
  // 최상위 컴포넌트에서 provide한 데이터 주입
  inject: ['user', 'settings', 'updateUser'],
  
  methods: {
    handleUpdateProfile() {
      const newName = prompt('새 이름을 입력하세요:', this.user.name);
      if (newName && newName.trim()) {
        this.updateUser({ name: newName.trim() });
      }
    }
  }
}
</script>
```

## 🎯 5. 실습 예제: 쇼핑몰 장바구니 시스템

```vue
<!-- App.vue -->
<template>
  <div class="shopping-app">
    <AppHeader />
    <div class="main-content">
      <ProductList />
      <ShoppingCart />
    </div>
    <AppFooter />
  </div>
</template>

<script>
import AppHeader from './components/AppHeader.vue'
import ProductList from './components/ProductList.vue'
import ShoppingCart from './components/ShoppingCart.vue'
import AppFooter from './components/AppFooter.vue'

export default {
  components: { AppHeader, ProductList, ShoppingCart, AppFooter },
  
  data() {
    return {
      products: [
        { id: 1, name: 'Vue.js 책', price: 30000, image: '📚', stock: 10 },
        { id: 2, name: 'React 책', price: 32000, image: '📖', stock: 5 },
        { id: 3, name: 'JavaScript 책', price: 28000, image: '📝', stock: 8 }
      ],
      cart: [],
      user: {
        name: '김개발',
        points: 50000,
        isVip: true
      }
    }
  },
  
  provide() {
    return {
      // 데이터 제공
      products: this.products,
      cart: this.cart,
      user: this.user,
      
      // 액션 메서드 제공
      addToCart: this.addToCart,
      removeFromCart: this.removeFromCart,
      updateQuantity: this.updateQuantity,
      clearCart: this.clearCart,
      
      // 계산된 값들 제공
      cartTotal: this.cartTotal,
      cartCount: this.cartCount
    }
  },
  
  computed: {
    cartTotal() {
      return this.cart.reduce((total, item) => 
        total + (item.price * item.quantity), 0
      );
    },
    
    cartCount() {
      return this.cart.reduce((count, item) => count + item.quantity, 0);
    }
  },
  
  methods: {
    addToCart(product) {
      const existingItem = this.cart.find(item => item.id === product.id);
      
      if (existingItem) {
        if (existingItem.quantity < product.stock) {
          existingItem.quantity++;
        } else {
          alert('재고가 부족합니다!');
        }
      } else {
        this.cart.push({
          ...product,
          quantity: 1
        });
      }
    },
    
    removeFromCart(productId) {
      this.cart = this.cart.filter(item => item.id !== productId);
    },
    
    updateQuantity(productId, newQuantity) {
      const item = this.cart.find(item => item.id === productId);
      if (item) {
        if (newQuantity <= 0) {
          this.removeFromCart(productId);
        } else {
          const product = this.products.find(p => p.id === productId);
          if (newQuantity <= product.stock) {
            item.quantity = newQuantity;
          } else {
            alert('재고가 부족합니다!');
          }
        }
      }
    },
    
    clearCart() {
      this.cart = [];
    }
  }
}
</script>

<style>
.shopping-app {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.main-content {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 20px;
  margin: 20px 0;
}

@media (max-width: 768px) {
  .main-content {
    grid-template-columns: 1fr;
  }
}
</style>
```

```vue
<!-- components/AppHeader.vue -->
<template>
  <header class="app-header">
    <h1>📚 Vue 서점</h1>
    <div class="user-info">
      <span>안녕하세요, {{ user.name }}님!</span>
      <span class="points">포인트: {{ user.points.toLocaleString() }}P</span>
      <span v-if="user.isVip" class="vip-badge">VIP</span>
    </div>
    <div class="cart-summary">
      <span class="cart-count">🛒 {{ cartCount }}개</span>
      <span class="cart-total">{{ cartTotal.toLocaleString() }}원</span>
    </div>
  </header>
</template>

<script>
export default {
  inject: ['user', 'cartCount', 'cartTotal']
}
</script>

<style scoped>
.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 0;
  border-bottom: 2px solid #42b883;
  margin-bottom: 20px;
}

.user-info {
  display: flex;
  gap: 10px;
  align-items: center;
}

.vip-badge {
  background: gold;
  color: black;
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: bold;
}

.cart-summary {
  display: flex;
  gap: 15px;
  font-weight: bold;
}

.cart-count {
  color: #42b883;
}

.cart-total {
  color: #e74c3c;
}
</style>
```

```vue
<!-- components/ProductList.vue -->
<template>
  <div class="product-list">
    <h2>상품 목록</h2>
    <div class="products-grid">
      <ProductCard 
        v-for="product in products" 
        :key="product.id"
        :product="product"
      />
    </div>
  </div>
</template>

<script>
import ProductCard from './ProductCard.vue'

export default {
  components: { ProductCard },
  inject: ['products']
}
</script>

<style scoped>
.products-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
}
</style>
```

```vue
<!-- components/ProductCard.vue -->
<template>
  <div class="product-card">
    <div class="product-image">{{ product.image }}</div>
    <h3>{{ product.name }}</h3>
    <p class="price">{{ product.price.toLocaleString() }}원</p>
    <p class="stock">재고: {{ product.stock }}개</p>
    
    <button 
      @click="handleAddToCart"
      :disabled="product.stock === 0"
      class="add-button"
    >
      {{ product.stock === 0 ? '품절' : '장바구니에 추가' }}
    </button>
  </div>
</template>

<script>
export default {
  props: {
    product: {
      type: Object,
      required: true
    }
  },
  
  inject: ['addToCart'],
  
  methods: {
    handleAddToCart() {
      this.addToCart(this.product);
    }
  }
}
</script>

<style scoped>
.product-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 20px;
  text-align: center;
  transition: transform 0.2s, box-shadow 0.2s;
}

.product-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.product-image {
  font-size: 48px;
  margin-bottom: 10px;
}

.price {
  font-size: 18px;
  font-weight: bold;
  color: #e74c3c;
}

.stock {
  color: #666;
  font-size: 14px;
}

.add-button {
  background: #42b883;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  width: 100%;
  margin-top: 10px;
}

.add-button:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.add-button:not(:disabled):hover {
  background: #369870;
}
</style>
```

```vue
<!-- components/ShoppingCart.vue -->
<template>
  <div class="shopping-cart">
    <h2>장바구니</h2>
    
    <div v-if="cart.length === 0" class="empty-cart">
      장바구니가 비어있습니다 🛒
    </div>
    
    <div v-else>
      <CartItem 
        v-for="item in cart"
        :key="item.id"
        :item="item"
      />
      
      <div class="cart-total">
        <h3>총 합계: {{ cartTotal.toLocaleString() }}원</h3>
        <div class="cart-actions">
          <button @click="clearCart" class="clear-button">
            장바구니 비우기
          </button>
          <button @click="checkout" class="checkout-button">
            결제하기
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import CartItem from './CartItem.vue'

export default {
  components: { CartItem },
  inject: ['cart', 'cartTotal', 'clearCart'],
  
  methods: {
    checkout() {
      if (this.cart.length === 0) {
        alert('장바구니가 비어있습니다!');
        return;
      }
      
      const confirmed = confirm(`총 ${this.cartTotal.toLocaleString()}원을 결제하시겠습니까?`);
      if (confirmed) {
        alert('결제가 완료되었습니다! 감사합니다.');
        this.clearCart();
      }
    }
  }
}
</script>

<style scoped>
.shopping-cart {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 20px;
}

.empty-cart {
  text-align: center;
  padding: 40px;
  color: #666;
  font-style: italic;
}

.cart-total {
  border-top: 2px solid #42b883;
  padding-top: 15px;
  margin-top: 15px;
}

.cart-actions {
  display: flex;
  gap: 10px;
  margin-top: 15px;
}

.clear-button {
  background: #6c757d;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 4px;
  cursor: pointer;
}

.checkout-button {
  background: #28a745;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 4px;
  cursor: pointer;
  flex: 1;
}

.checkout-button:hover {
  background: #218838;
}
</style>
```

## 🎯 실습 과제

1. 위의 쇼핑몰 앱을 직접 만들어보세요
2. 상품 카테고리 필터링 기능을 추가해보세요
3. 위시리스트 기능을 추가해보세요
4. 주문 히스토리 기능을 추가해보세요
5. 검색 기능을 추가해보세요

## 🔗 다음 단계

다음 챕터에서는 조건부 렌더링과 리스트 최적화에 대해 알아보겠습니다!

---

💡 **팁**: Vue의 Provide/Inject는 React의 Context보다 더 간단하고 직관적으로 깊은 컴포넌트 통신을 해결할 수 있어요! 