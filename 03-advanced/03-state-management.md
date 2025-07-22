# 3-3. 상태 관리 (Pinia)

## 🗃️ Pinia vs Redux/Zustand

Pinia는 Vue의 공식 상태 관리 라이브러리로, Redux보다 간단하고 Zustand보다 강력한 기능을 제공합니다!

## 🎯 1. 기본 개념 비교

### Redux 방식
```javascript
// actions/userActions.js
export const SET_USER = 'SET_USER'
export const SET_LOADING = 'SET_LOADING'
export const SET_ERROR = 'SET_ERROR'

export const setUser = (user) => ({
  type: SET_USER,
  payload: user
})

export const fetchUser = (userId) => {
  return async (dispatch) => {
    dispatch(setLoading(true))
    try {
      const response = await fetch(`/api/users/${userId}`)
      const user = await response.json()
      dispatch(setUser(user))
    } catch (error) {
      dispatch(setError(error.message))
    } finally {
      dispatch(setLoading(false))
    }
  }
}

// reducers/userReducer.js
const initialState = {
  user: null,
  loading: false,
  error: null
}

export const userReducer = (state = initialState, action) => {
  switch (action.type) {
    case SET_USER:
      return { ...state, user: action.payload }
    case SET_LOADING:
      return { ...state, loading: action.payload }
    case SET_ERROR:
      return { ...state, error: action.payload }
    default:
      return state
  }
}

// store.js
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import { userReducer } from './reducers/userReducer'

const rootReducer = combineReducers({
  user: userReducer
})

export const store = createStore(rootReducer, applyMiddleware(thunk))
```

### Zustand 방식
```javascript
// stores/useUserStore.js
import { create } from 'zustand'

export const useUserStore = create((set, get) => ({
  user: null,
  loading: false,
  error: null,
  
  setUser: (user) => set({ user }),
  setLoading: (loading) => set({ loading }),
  setError: (error) => set({ error }),
  
  fetchUser: async (userId) => {
    set({ loading: true, error: null })
    try {
      const response = await fetch(`/api/users/${userId}`)
      const user = await response.json()
      set({ user, loading: false })
    } catch (error) {
      set({ error: error.message, loading: false })
    }
  }
}))
```

### Pinia 방식
```javascript
// stores/user.js
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    user: null,
    loading: false,
    error: null
  }),
  
  getters: {
    isLoggedIn: (state) => !!state.user,
    userName: (state) => state.user?.name || 'Guest',
    userInitials: (state) => {
      if (!state.user?.name) return ''
      return state.user.name
        .split(' ')
        .map(word => word[0])
        .join('')
        .toUpperCase()
    }
  },
  
  actions: {
    async fetchUser(userId) {
      this.loading = true
      this.error = null
      
      try {
        const response = await fetch(`/api/users/${userId}`)
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`)
        }
        
        this.user = await response.json()
      } catch (error) {
        this.error = error.message
        throw error
      } finally {
        this.loading = false
      }
    },
    
    logout() {
      this.user = null
      this.error = null
    },
    
    updateUser(userData) {
      if (this.user) {
        this.user = { ...this.user, ...userData }
      }
    }
  }
})
```

## 🏪 2. 복합 Store 구조

### 쇼핑카트 Store
```javascript
// stores/cart.js
import { defineStore } from 'pinia'
import { useUserStore } from './user'

export const useCartStore = defineStore('cart', {
  state: () => ({
    items: [],
    discountCode: null,
    shippingMethod: 'standard'
  }),
  
  getters: {
    itemCount: (state) => {
      return state.items.reduce((total, item) => total + item.quantity, 0)
    },
    
    subtotal: (state) => {
      return state.items.reduce((total, item) => 
        total + (item.price * item.quantity), 0
      )
    },
    
    discount: (state) => {
      if (!state.discountCode) return 0
      
      const discountRates = {
        'SAVE10': 0.1,
        'SAVE20': 0.2,
        'WELCOME': 0.15
      }
      
      return discountRates[state.discountCode] || 0
    },
    
    discountAmount() {
      return this.subtotal * this.discount
    },
    
    shippingCost: (state) => {
      const shippingRates = {
        standard: 5000,
        express: 10000,
        overnight: 20000
      }
      
      return shippingRates[state.shippingMethod] || 0
    },
    
    total() {
      return this.subtotal - this.discountAmount + this.shippingCost
    },
    
    // 다른 Store의 상태 사용
    userDiscount() {
      const userStore = useUserStore()
      return userStore.user?.isVip ? 0.05 : 0
    },
    
    finalTotal() {
      return this.total * (1 - this.userDiscount)
    }
  },
  
  actions: {
    addItem(product) {
      const existingItem = this.items.find(item => item.id === product.id)
      
      if (existingItem) {
        existingItem.quantity += 1
      } else {
        this.items.push({
          id: product.id,
          name: product.name,
          price: product.price,
          image: product.image,
          quantity: 1
        })
      }
    },
    
    removeItem(productId) {
      const index = this.items.findIndex(item => item.id === productId)
      if (index > -1) {
        this.items.splice(index, 1)
      }
    },
    
    updateQuantity(productId, quantity) {
      const item = this.items.find(item => item.id === productId)
      if (item) {
        if (quantity <= 0) {
          this.removeItem(productId)
        } else {
          item.quantity = quantity
        }
      }
    },
    
    applyDiscountCode(code) {
      // 실제로는 API로 검증
      const validCodes = ['SAVE10', 'SAVE20', 'WELCOME']
      if (validCodes.includes(code)) {
        this.discountCode = code
        return true
      }
      return false
    },
    
    clearCart() {
      this.items = []
      this.discountCode = null
    },
    
    async checkout() {
      const userStore = useUserStore()
      
      if (!userStore.isLoggedIn) {
        throw new Error('로그인이 필요합니다')
      }
      
      if (this.items.length === 0) {
        throw new Error('장바구니가 비어있습니다')
      }
      
      const orderData = {
        userId: userStore.user.id,
        items: this.items,
        discountCode: this.discountCode,
        shippingMethod: this.shippingMethod,
        total: this.finalTotal
      }
      
      try {
        const response = await fetch('/api/orders', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(orderData)
        })
        
        if (!response.ok) {
          throw new Error('주문 처리 중 오류가 발생했습니다')
        }
        
        const order = await response.json()
        this.clearCart()
        
        return order
      } catch (error) {
        throw error
      }
    }
  }
})
```

### 알림 Store
```javascript
// stores/notifications.js
import { defineStore } from 'pinia'

export const useNotificationStore = defineStore('notifications', {
  state: () => ({
    notifications: []
  }),
  
  getters: {
    unreadCount: (state) => {
      return state.notifications.filter(n => !n.read).length
    },
    
    urgentNotifications: (state) => {
      return state.notifications.filter(n => n.priority === 'urgent' && !n.read)
    }
  },
  
  actions: {
    addNotification(notification) {
      const newNotification = {
        id: Date.now() + Math.random(),
        timestamp: new Date(),
        read: false,
        priority: 'normal',
        ...notification
      }
      
      this.notifications.unshift(newNotification)
      
      // 자동 제거 (선택적)
      if (notification.autoRemove !== false) {
        setTimeout(() => {
          this.removeNotification(newNotification.id)
        }, notification.duration || 5000)
      }
    },
    
    markAsRead(notificationId) {
      const notification = this.notifications.find(n => n.id === notificationId)
      if (notification) {
        notification.read = true
      }
    },
    
    markAllAsRead() {
      this.notifications.forEach(n => n.read = true)
    },
    
    removeNotification(notificationId) {
      const index = this.notifications.findIndex(n => n.id === notificationId)
      if (index > -1) {
        this.notifications.splice(index, 1)
      }
    },
    
    clearAll() {
      this.notifications = []
    }
  }
})
```

## 🔄 3. Store 조합과 플러그인

### Store 조합 예제
```javascript
// stores/dashboard.js
import { defineStore } from 'pinia'
import { useUserStore } from './user'
import { useCartStore } from './cart'
import { useNotificationStore } from './notifications'

export const useDashboardStore = defineStore('dashboard', {
  state: () => ({
    sidebarOpen: false,
    activeTab: 'overview',
    widgets: [
      { id: 'sales', enabled: true, position: 0 },
      { id: 'users', enabled: true, position: 1 },
      { id: 'analytics', enabled: false, position: 2 }
    ]
  }),
  
  getters: {
    enabledWidgets: (state) => {
      return state.widgets
        .filter(w => w.enabled)
        .sort((a, b) => a.position - b.position)
    },
    
    // 다른 Store들의 상태를 조합
    dashboardSummary() {
      const userStore = useUserStore()
      const cartStore = useCartStore()
      const notificationStore = useNotificationStore()
      
      return {
        user: userStore.userName,
        isLoggedIn: userStore.isLoggedIn,
        cartItems: cartStore.itemCount,
        cartTotal: cartStore.total,
        unreadNotifications: notificationStore.unreadCount,
        urgentAlerts: notificationStore.urgentNotifications.length
      }
    }
  },
  
  actions: {
    toggleSidebar() {
      this.sidebarOpen = !this.sidebarOpen
    },
    
    setActiveTab(tab) {
      this.activeTab = tab
    },
    
    toggleWidget(widgetId) {
      const widget = this.widgets.find(w => w.id === widgetId)
      if (widget) {
        widget.enabled = !widget.enabled
      }
    },
    
    reorderWidgets(newOrder) {
      newOrder.forEach((widgetId, index) => {
        const widget = this.widgets.find(w => w.id === widgetId)
        if (widget) {
          widget.position = index
        }
      })
    }
  }
})
```

### 지속성 플러그인
```javascript
// plugins/persistence.js
export function createPersistedStore(options = {}) {
  const {
    key = 'pinia-store',
    storage = localStorage,
    serialize = JSON.stringify,
    deserialize = JSON.parse
  } = options
  
  return ({ store }) => {
    // 초기 상태 복원
    const stored = storage.getItem(`${key}-${store.$id}`)
    if (stored) {
      try {
        const state = deserialize(stored)
        store.$patch(state)
      } catch (error) {
        console.error('Failed to restore state:', error)
      }
    }
    
    // 상태 변경 시 저장
    store.$subscribe((mutation, state) => {
      try {
        storage.setItem(`${key}-${store.$id}`, serialize(state))
      } catch (error) {
        console.error('Failed to persist state:', error)
      }
    })
  }
}

// main.js에서 사용
import { createPinia } from 'pinia'
import { createPersistedStore } from '@/plugins/persistence'

const pinia = createPinia()
pinia.use(createPersistedStore({
  key: 'my-app'
}))
```

### 로깅 플러그인
```javascript
// plugins/logger.js
export function createLoggerPlugin(options = {}) {
  const {
    logActions = true,
    logMutations = true,
    logState = false
  } = options
  
  return ({ store }) => {
    if (logActions) {
      // Action 호출 로깅
      const originalActions = { ...store }
      Object.keys(store).forEach(key => {
        if (typeof store[key] === 'function') {
          const originalAction = store[key]
          store[key] = (...args) => {
            console.group(`🎬 Action: ${store.$id}.${key}`)
            console.log('Arguments:', args)
            console.log('State before:', { ...store.$state })
            
            const result = originalAction.apply(store, args)
            
            if (result instanceof Promise) {
              result
                .then((res) => {
                  console.log('Action completed successfully')
                  console.log('State after:', { ...store.$state })
                  console.groupEnd()
                  return res
                })
                .catch((error) => {
                  console.error('Action failed:', error)
                  console.log('State after:', { ...store.$state })
                  console.groupEnd()
                  throw error
                })
            } else {
              console.log('State after:', { ...store.$state })
              console.groupEnd()
            }
            
            return result
          }
        }
      })
    }
    
    if (logMutations) {
      // 상태 변경 로깅
      store.$subscribe((mutation, state) => {
        console.log(`🔄 Mutation in ${store.$id}:`, mutation)
        if (logState) {
          console.log('New state:', state)
        }
      })
    }
  }
}
```

## 🎯 4. 실습 예제: E-commerce 앱

```vue
<template>
  <div class="ecommerce-app">
    <!-- 헤더 -->
    <AppHeader />
    
    <!-- 메인 컨텐츠 -->
    <main class="main-content">
      <router-view />
    </main>
    
    <!-- 장바구니 사이드바 -->
    <CartSidebar />
    
    <!-- 알림 -->
    <NotificationContainer />
  </div>
</template>

<script>
import { onMounted } from 'vue'
import { useUserStore } from '@/stores/user'
import { useNotificationStore } from '@/stores/notifications'
import AppHeader from '@/components/AppHeader.vue'
import CartSidebar from '@/components/CartSidebar.vue'
import NotificationContainer from '@/components/NotificationContainer.vue'

export default {
  components: {
    AppHeader,
    CartSidebar,
    NotificationContainer
  },
  
  setup() {
    const userStore = useUserStore()
    const notificationStore = useNotificationStore()
    
    onMounted(async () => {
      // 로그인 상태 복원
      const token = localStorage.getItem('auth-token')
      if (token) {
        try {
          await userStore.fetchUser()
          notificationStore.addNotification({
            type: 'success',
            title: '로그인 성공',
            message: `안녕하세요, ${userStore.userName}님!`
          })
        } catch (error) {
          localStorage.removeItem('auth-token')
          notificationStore.addNotification({
            type: 'error',
            title: '로그인 실패',
            message: '세션이 만료되었습니다. 다시 로그인해주세요.'
          })
        }
      }
    })
    
    return {}
  }
}
</script>
```

### 상품 목록 컴포넌트
```vue
<!-- components/ProductList.vue -->
<template>
  <div class="product-list">
    <div class="filters">
      <select v-model="selectedCategory">
        <option value="">모든 카테고리</option>
        <option v-for="category in categories" :key="category" :value="category">
          {{ category }}
        </option>
      </select>
      
      <input v-model="searchQuery" placeholder="상품 검색..." />
      
      <select v-model="sortBy">
        <option value="name">이름순</option>
        <option value="price-low">가격 낮은순</option>
        <option value="price-high">가격 높은순</option>
      </select>
    </div>
    
    <div v-if="loading" class="loading">상품을 불러오는 중...</div>
    
    <div v-else class="products-grid">
      <ProductCard
        v-for="product in filteredProducts"
        :key="product.id"
        :product="product"
        @add-to-cart="handleAddToCart"
      />
    </div>
    
    <div v-if="!loading && filteredProducts.length === 0" class="empty-state">
      조건에 맞는 상품이 없습니다.
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { useProductStore } from '@/stores/products'
import { useCartStore } from '@/stores/cart'
import { useNotificationStore } from '@/stores/notifications'
import ProductCard from './ProductCard.vue'

export default {
  components: { ProductCard },
  
  setup() {
    const productStore = useProductStore()
    const cartStore = useCartStore()
    const notificationStore = useNotificationStore()
    
    const selectedCategory = ref('')
    const searchQuery = ref('')
    const sortBy = ref('name')
    
    const categories = computed(() => productStore.categories)
    const loading = computed(() => productStore.loading)
    
    const filteredProducts = computed(() => {
      let products = productStore.products
      
      // 카테고리 필터
      if (selectedCategory.value) {
        products = products.filter(p => p.category === selectedCategory.value)
      }
      
      // 검색 필터
      if (searchQuery.value) {
        const query = searchQuery.value.toLowerCase()
        products = products.filter(p => 
          p.name.toLowerCase().includes(query) ||
          p.description.toLowerCase().includes(query)
        )
      }
      
      // 정렬
      products = [...products].sort((a, b) => {
        switch (sortBy.value) {
          case 'name':
            return a.name.localeCompare(b.name)
          case 'price-low':
            return a.price - b.price
          case 'price-high':
            return b.price - a.price
          default:
            return 0
        }
      })
      
      return products
    })
    
    const handleAddToCart = (product) => {
      cartStore.addItem(product)
      
      notificationStore.addNotification({
        type: 'success',
        title: '장바구니 추가',
        message: `${product.name}이(가) 장바구니에 추가되었습니다.`,
        duration: 3000
      })
    }
    
    onMounted(() => {
      productStore.fetchProducts()
    })
    
    return {
      selectedCategory,
      searchQuery,
      sortBy,
      categories,
      loading,
      filteredProducts,
      handleAddToCart
    }
  }
}
</script>
```

### 장바구니 사이드바
```vue
<!-- components/CartSidebar.vue -->
<template>
  <transition name="slide">
    <div v-if="isOpen" class="cart-sidebar">
      <div class="cart-header">
        <h2>장바구니</h2>
        <button @click="toggleCart" class="close-btn">×</button>
      </div>
      
      <div class="cart-content">
        <div v-if="items.length === 0" class="empty-cart">
          장바구니가 비어있습니다
        </div>
        
        <div v-else>
          <CartItem
            v-for="item in items"
            :key="item.id"
            :item="item"
            @update-quantity="updateQuantity"
            @remove="removeItem"
          />
          
          <div class="cart-summary">
            <div class="summary-row">
              <span>소계:</span>
              <span>{{ formatPrice(subtotal) }}</span>
            </div>
            
            <div v-if="discount > 0" class="summary-row discount">
              <span>할인 ({{ (discount * 100).toFixed(0) }}%):</span>
              <span>-{{ formatPrice(discountAmount) }}</span>
            </div>
            
            <div class="summary-row">
              <span>배송비:</span>
              <span>{{ formatPrice(shippingCost) }}</span>
            </div>
            
            <div class="summary-row total">
              <span>총합:</span>
              <span>{{ formatPrice(finalTotal) }}</span>
            </div>
            
            <div class="discount-code">
              <input 
                v-model="discountCodeInput" 
                placeholder="할인 코드"
                @keyup.enter="applyDiscount"
              />
              <button @click="applyDiscount">적용</button>
            </div>
            
            <button 
              @click="handleCheckout"
              :disabled="!canCheckout"
              class="checkout-btn"
            >
              결제하기
            </button>
          </div>
        </div>
      </div>
    </div>
  </transition>
  
  <div v-if="isOpen" class="cart-overlay" @click="toggleCart"></div>
</template>

<script>
import { ref, computed } from 'vue'
import { useCartStore } from '@/stores/cart'
import { useUserStore } from '@/stores/user'
import { useNotificationStore } from '@/stores/notifications'
import { useUiStore } from '@/stores/ui'
import CartItem from './CartItem.vue'

export default {
  components: { CartItem },
  
  setup() {
    const cartStore = useCartStore()
    const userStore = useUserStore()
    const notificationStore = useNotificationStore()
    const uiStore = useUiStore()
    
    const discountCodeInput = ref('')
    
    const isOpen = computed(() => uiStore.cartSidebarOpen)
    const items = computed(() => cartStore.items)
    const subtotal = computed(() => cartStore.subtotal)
    const discount = computed(() => cartStore.discount)
    const discountAmount = computed(() => cartStore.discountAmount)
    const shippingCost = computed(() => cartStore.shippingCost)
    const finalTotal = computed(() => cartStore.finalTotal)
    
    const canCheckout = computed(() => {
      return userStore.isLoggedIn && items.value.length > 0
    })
    
    const toggleCart = () => {
      uiStore.toggleCartSidebar()
    }
    
    const updateQuantity = (productId, quantity) => {
      cartStore.updateQuantity(productId, quantity)
    }
    
    const removeItem = (productId) => {
      cartStore.removeItem(productId)
      
      notificationStore.addNotification({
        type: 'info',
        title: '상품 제거',
        message: '장바구니에서 상품이 제거되었습니다.',
        duration: 2000
      })
    }
    
    const applyDiscount = () => {
      if (!discountCodeInput.value.trim()) return
      
      const success = cartStore.applyDiscountCode(discountCodeInput.value.trim())
      
      if (success) {
        notificationStore.addNotification({
          type: 'success',
          title: '할인 적용',
          message: '할인 코드가 적용되었습니다!',
          duration: 3000
        })
        discountCodeInput.value = ''
      } else {
        notificationStore.addNotification({
          type: 'error',
          title: '할인 실패',
          message: '유효하지 않은 할인 코드입니다.',
          duration: 3000
        })
      }
    }
    
    const handleCheckout = async () => {
      if (!canCheckout.value) {
        if (!userStore.isLoggedIn) {
          notificationStore.addNotification({
            type: 'warning',
            title: '로그인 필요',
            message: '결제하려면 먼저 로그인해주세요.',
            duration: 4000
          })
        }
        return
      }
      
      try {
        const order = await cartStore.checkout()
        
        notificationStore.addNotification({
          type: 'success',
          title: '주문 완료',
          message: `주문번호 ${order.id}로 결제가 완료되었습니다.`,
          duration: 5000
        })
        
        toggleCart()
        
      } catch (error) {
        notificationStore.addNotification({
          type: 'error',
          title: '결제 실패',
          message: error.message,
          duration: 5000
        })
      }
    }
    
    const formatPrice = (price) => {
      return new Intl.NumberFormat('ko-KR', {
        style: 'currency',
        currency: 'KRW'
      }).format(price)
    }
    
    return {
      isOpen,
      items,
      subtotal,
      discount,
      discountAmount,
      shippingCost,
      finalTotal,
      canCheckout,
      discountCodeInput,
      toggleCart,
      updateQuantity,
      removeItem,
      applyDiscount,
      handleCheckout,
      formatPrice
    }
  }
}
</script>

<style scoped>
.cart-sidebar {
  position: fixed;
  top: 0;
  right: 0;
  height: 100vh;
  width: 400px;
  background: white;
  box-shadow: -2px 0 10px rgba(0,0,0,0.1);
  z-index: 1000;
  display: flex;
  flex-direction: column;
}

.cart-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  z-index: 999;
}

.cart-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  border-bottom: 1px solid #eee;
}

.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
}

.cart-content {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

.empty-cart {
  text-align: center;
  padding: 40px 20px;
  color: #666;
}

.cart-summary {
  border-top: 1px solid #eee;
  padding-top: 20px;
  margin-top: 20px;
}

.summary-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.summary-row.discount {
  color: #28a745;
}

.summary-row.total {
  font-weight: bold;
  font-size: 18px;
  border-top: 1px solid #eee;
  padding-top: 10px;
}

.discount-code {
  display: flex;
  gap: 10px;
  margin: 20px 0;
}

.discount-code input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.checkout-btn {
  width: 100%;
  padding: 15px;
  background: #28a745;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
}

.checkout-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease;
}

.slide-enter-from,
.slide-leave-to {
  transform: translateX(100%);
}
</style>
```

## 🎯 실습 과제

1. 위의 E-commerce 스토어들을 직접 구현해보세요
2. 주문 내역을 관리하는 Store를 추가해보세요
3. 찜하기 기능을 위한 Store를 만들어보세요
4. 상품 리뷰 시스템을 구현해보세요
5. 실시간 재고 업데이트 기능을 추가해보세요

## 🔗 다음 단계

다음 챕터에서는 Vue Router를 활용한 라우팅에 대해 알아보겠습니다!

---

💡 **팁**: Pinia는 Redux보다 훨씬 간단하고 직관적이면서도, Zustand보다 더 강력한 기능을 제공합니다. TypeScript 지원도 훌륭해요! 