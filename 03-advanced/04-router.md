# 3-4. Vue Router

## 🗺️ Vue Router vs React Router

Vue Router는 React Router와 비슷한 기능을 제공하지만, Vue의 반응성 시스템과 더 잘 통합되어 더욱 직관적입니다!

## 🎯 1. 기본 라우팅 설정 비교

### React Router 방식
```jsx
// App.jsx
import { BrowserRouter, Routes, Route, Link, Navigate } from 'react-router-dom'
import Home from './pages/Home'
import About from './pages/About'
import Products from './pages/Products'
import ProductDetail from './pages/ProductDetail'
import NotFound from './pages/NotFound'

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/products">Products</Link>
      </nav>
      
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/products" element={<Products />} />
        <Route path="/products/:id" element={<ProductDetail />} />
        <Route path="/old-about" element={<Navigate to="/about" replace />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  )
}
```

### Vue Router 방식
```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import Home from '@/views/Home.vue'
import About from '@/views/About.vue'
import Products from '@/views/Products.vue'
import ProductDetail from '@/views/ProductDetail.vue'
import NotFound from '@/views/NotFound.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    component: About
  },
  {
    path: '/products',
    name: 'Products',
    component: Products
  },
  {
    path: '/products/:id',
    name: 'ProductDetail',
    component: ProductDetail,
    props: true
  },
  {
    path: '/old-about',
    redirect: '/about'
  },
  {
    path: '/:pathMatch(.*)*',
    name: 'NotFound',
    component: NotFound
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

```vue
<!-- App.vue -->
<template>
  <div id="app">
    <nav>
      <router-link to="/">Home</router-link>
      <router-link to="/about">About</router-link>
      <router-link to="/products">Products</router-link>
    </nav>
    
    <router-view />
  </div>
</template>
```

## 🛡️ 2. 네비게이션 가드와 인증

### React Router의 Protected Route
```jsx
// components/ProtectedRoute.jsx
import { Navigate, useLocation } from 'react-router-dom'
import { useAuth } from '../hooks/useAuth'

function ProtectedRoute({ children, requiredRole }) {
  const { user, loading } = useAuth()
  const location = useLocation()
  
  if (loading) {
    return <div>Loading...</div>
  }
  
  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />
  }
  
  if (requiredRole && !user.roles.includes(requiredRole)) {
    return <Navigate to="/unauthorized" replace />
  }
  
  return children
}

// 사용
<Routes>
  <Route path="/dashboard" element={
    <ProtectedRoute>
      <Dashboard />
    </ProtectedRoute>
  } />
  <Route path="/admin" element={
    <ProtectedRoute requiredRole="admin">
      <AdminPanel />
    </ProtectedRoute>
  } />
</Routes>
```

### Vue Router의 Navigation Guards
```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import { useUserStore } from '@/stores/user'

const routes = [
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: () => import('@/views/Dashboard.vue'),
    meta: { requiresAuth: true }
  },
  {
    path: '/admin',
    name: 'Admin',
    component: () => import('@/views/Admin.vue'),
    meta: { 
      requiresAuth: true, 
      requiredRole: 'admin'
    }
  },
  {
    path: '/profile/:userId',
    name: 'Profile',
    component: () => import('@/views/Profile.vue'),
    meta: { requiresAuth: true },
    beforeEnter: (to, from, next) => {
      const userStore = useUserStore()
      const currentUserId = userStore.user?.id
      const requestedUserId = to.params.userId
      
      // 자신의 프로필이거나 관리자인 경우만 허용
      if (currentUserId === requestedUserId || userStore.user?.role === 'admin') {
        next()
      } else {
        next('/unauthorized')
      }
    }
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// 전역 가드
router.beforeEach(async (to, from, next) => {
  const userStore = useUserStore()
  
  // 로딩 중이면 기다림
  if (userStore.loading) {
    await new Promise(resolve => {
      const unwatch = userStore.$subscribe(() => {
        if (!userStore.loading) {
          unwatch()
          resolve()
        }
      })
    })
  }
  
  // 인증이 필요한 페이지 체크
  if (to.meta.requiresAuth && !userStore.isLoggedIn) {
    next({ 
      name: 'Login', 
      query: { redirect: to.fullPath }
    })
    return
  }
  
  // 역할 기반 접근 제어
  if (to.meta.requiredRole && userStore.user?.role !== to.meta.requiredRole) {
    next('/unauthorized')
    return
  }
  
  // 페이지 제목 설정
  if (to.meta.title) {
    document.title = `${to.meta.title} - My App`
  }
  
  next()
})

// 후처리 가드
router.afterEach((to, from) => {
  // 페이지 조회 분석
  if (typeof gtag !== 'undefined') {
    gtag('config', 'GA_MEASUREMENT_ID', {
      page_path: to.path
    })
  }
  
  // 스크롤 위치 복원
  if (to.meta.scrollToTop !== false) {
    window.scrollTo(0, 0)
  }
})
```

## 🎨 3. 고급 라우팅 패턴

### 중첩 라우팅 (Nested Routing)
```javascript
// router/index.js
const routes = [
  {
    path: '/dashboard',
    component: () => import('@/layouts/DashboardLayout.vue'),
    children: [
      {
        path: '',
        name: 'DashboardHome',
        component: () => import('@/views/dashboard/Home.vue')
      },
      {
        path: 'analytics',
        name: 'Analytics',
        component: () => import('@/views/dashboard/Analytics.vue'),
        children: [
          {
            path: 'sales',
            name: 'SalesAnalytics',
            component: () => import('@/views/dashboard/analytics/Sales.vue')
          },
          {
            path: 'users',
            name: 'UserAnalytics',
            component: () => import('@/views/dashboard/analytics/Users.vue')
          }
        ]
      },
      {
        path: 'settings',
        name: 'Settings',
        component: () => import('@/views/dashboard/Settings.vue')
      }
    ]
  }
]
```

```vue
<!-- layouts/DashboardLayout.vue -->
<template>
  <div class="dashboard-layout">
    <aside class="sidebar">
      <nav>
        <router-link to="/dashboard">🏠 홈</router-link>
        <router-link to="/dashboard/analytics">📊 분석</router-link>
        <router-link to="/dashboard/settings">⚙️ 설정</router-link>
      </nav>
    </aside>
    
    <main class="main-content">
      <router-view />
    </main>
  </div>
</template>
```

### 이름있는 뷰 (Named Views)
```javascript
const routes = [
  {
    path: '/user/:id',
    components: {
      default: () => import('@/views/User.vue'),
      sidebar: () => import('@/components/UserSidebar.vue'),
      header: () => import('@/components/UserHeader.vue')
    }
  }
]
```

```vue
<!-- App.vue -->
<template>
  <div class="app">
    <router-view name="header" />
    <div class="content">
      <router-view name="sidebar" />
      <router-view />
    </div>
  </div>
</template>
```

### 동적 라우팅
```javascript
// router/index.js
const routes = [
  {
    path: '/admin/:module',
    name: 'AdminModule',
    component: () => import('@/views/AdminModule.vue'),
    props: route => ({
      module: route.params.module,
      config: route.query
    })
  }
]

// 동적으로 라우트 추가
export function addModuleRoutes(modules) {
  modules.forEach(module => {
    router.addRoute('AdminModule', {
      path: `/admin/${module.name}/:action?`,
      name: `Admin${module.name}`,
      component: module.component,
      meta: {
        title: module.title,
        icon: module.icon,
        permissions: module.permissions
      }
    })
  })
}
```

## 🎯 4. 고급 네비게이션과 상태 관리

### 라우트 기반 상태 관리
```javascript
// composables/useRouteState.js
import { computed, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'

export function useRouteState() {
  const route = useRoute()
  const router = useRouter()
  
  // 쿼리 파라미터 반응형 관리
  const updateQuery = (newQuery) => {
    router.push({
      query: { ...route.query, ...newQuery }
    })
  }
  
  const removeQuery = (key) => {
    const query = { ...route.query }
    delete query[key]
    router.push({ query })
  }
  
  // 페이지네이션
  const currentPage = computed({
    get: () => parseInt(route.query.page) || 1,
    set: (page) => updateQuery({ page: page > 1 ? page : undefined })
  })
  
  // 필터
  const filters = computed({
    get: () => ({
      category: route.query.category || '',
      search: route.query.search || '',
      sortBy: route.query.sortBy || 'name'
    }),
    set: (newFilters) => {
      const query = { ...route.query }
      Object.entries(newFilters).forEach(([key, value]) => {
        if (value) {
          query[key] = value
        } else {
          delete query[key]
        }
      })
      router.push({ query })
    }
  })
  
  return {
    route,
    router,
    updateQuery,
    removeQuery,
    currentPage,
    filters
  }
}
```

### 브레드크럼 네비게이션
```vue
<!-- components/Breadcrumb.vue -->
<template>
  <nav class="breadcrumb">
    <router-link 
      v-for="(crumb, index) in breadcrumbs"
      :key="crumb.path"
      :to="crumb.path"
      :class="{ active: index === breadcrumbs.length - 1 }"
    >
      {{ crumb.title }}
      <span v-if="index < breadcrumbs.length - 1" class="separator">></span>
    </router-link>
  </nav>
</template>

<script>
import { computed } from 'vue'
import { useRoute } from 'vue-router'

export default {
  setup() {
    const route = useRoute()
    
    const breadcrumbs = computed(() => {
      const matched = route.matched.filter(record => record.meta.breadcrumb !== false)
      const crumbs = []
      
      matched.forEach((record, index) => {
        const title = record.meta.title || record.name
        const path = index === matched.length - 1 
          ? route.path 
          : record.path
        
        crumbs.push({ title, path })
      })
      
      return crumbs
    })
    
    return { breadcrumbs }
  }
}
</script>
```

## 🎯 5. 실습 예제: 완전한 SPA 애플리케이션

```vue
<!-- App.vue -->
<template>
  <div id="app">
    <!-- 로딩 바 -->
    <div v-if="isRouteLoading" class="route-loading">
      <div class="loading-bar"></div>
    </div>
    
    <!-- 헤더 -->
    <AppHeader v-if="!route.meta.hideHeader" />
    
    <!-- 메인 컨텐츠 -->
    <main class="main-content">
      <router-view v-slot="{ Component, route: currentRoute }">
        <transition :name="getTransitionName(currentRoute)" mode="out-in">
          <component :is="Component" :key="currentRoute.path" />
        </transition>
      </router-view>
    </main>
    
    <!-- 푸터 -->
    <AppFooter v-if="!route.meta.hideFooter" />
    
    <!-- 모달 라우터 뷰 -->
    <router-view name="modal" />
  </div>
</template>

<script>
import { computed, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'
import { useUiStore } from '@/stores/ui'
import AppHeader from '@/components/AppHeader.vue'
import AppFooter from '@/components/AppFooter.vue'

export default {
  components: {
    AppHeader,
    AppFooter
  },
  
  setup() {
    const route = useRoute()
    const router = useRouter()
    const userStore = useUserStore()
    const uiStore = useUiStore()
    
    const isRouteLoading = computed(() => uiStore.routeLoading)
    
    // 페이지 전환 애니메이션 결정
    const getTransitionName = (currentRoute) => {
      const level = currentRoute.meta.level || 0
      const prevLevel = route.meta.level || 0
      
      if (level > prevLevel) {
        return 'slide-left'
      } else if (level < prevLevel) {
        return 'slide-right'
      } else {
        return 'fade'
      }
    }
    
    // 라우트 변경 시 로딩 상태 관리
    router.beforeEach(() => {
      uiStore.setRouteLoading(true)
    })
    
    router.afterEach(() => {
      uiStore.setRouteLoading(false)
    })
    
    // 페이지 제목 업데이트
    watch(() => route.meta.title, (title) => {
      if (title) {
        document.title = `${title} - My SPA`
      }
    }, { immediate: true })
    
    return {
      route,
      isRouteLoading,
      getTransitionName
    }
  }
}
</script>

<style>
/* 페이지 전환 애니메이션 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-left-enter-active,
.slide-left-leave-active,
.slide-right-enter-active,
.slide-right-leave-active {
  transition: transform 0.3s ease;
}

.slide-left-enter-from {
  transform: translateX(100%);
}

.slide-left-leave-to {
  transform: translateX(-100%);
}

.slide-right-enter-from {
  transform: translateX(-100%);
}

.slide-right-leave-to {
  transform: translateX(100%);
}

.route-loading {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  z-index: 9999;
}

.loading-bar {
  height: 100%;
  background: linear-gradient(90deg, #42b883, #369870);
  animation: loading 2s ease-in-out infinite;
}

@keyframes loading {
  0%, 100% { transform: translateX(-100%); }
  50% { transform: translateX(100%); }
}
</style>
```

### 상품 상세 페이지
```vue
<!-- views/ProductDetail.vue -->
<template>
  <div class="product-detail">
    <nav class="breadcrumb">
      <router-link to="/">홈</router-link>
      <router-link to="/products">상품</router-link>
      <router-link 
        v-if="product?.category"
        :to="`/products?category=${product.category}`"
      >
        {{ product.category }}
      </router-link>
      <span>{{ product?.name }}</span>
    </nav>
    
    <div v-if="loading" class="loading">
      상품 정보를 불러오는 중...
    </div>
    
    <div v-else-if="error" class="error">
      <h2>오류가 발생했습니다</h2>
      <p>{{ error.message }}</p>
      <button @click="retry">다시 시도</button>
    </div>
    
    <div v-else-if="product" class="product-content">
      <div class="product-images">
        <img :src="product.image" :alt="product.name" />
      </div>
      
      <div class="product-info">
        <h1>{{ product.name }}</h1>
        <p class="price">{{ formatPrice(product.price) }}</p>
        <p class="description">{{ product.description }}</p>
        
        <div class="product-actions">
          <button @click="addToCart" class="add-to-cart-btn">
            장바구니에 추가
          </button>
          <button @click="buyNow" class="buy-now-btn">
            바로 구매
          </button>
        </div>
        
        <!-- 관련 상품 -->
        <div class="related-products">
          <h3>관련 상품</h3>
          <div class="related-grid">
            <router-link
              v-for="related in relatedProducts"
              :key="related.id"
              :to="`/products/${related.id}`"
              class="related-item"
            >
              <img :src="related.image" :alt="related.name" />
              <h4>{{ related.name }}</h4>
              <p>{{ formatPrice(related.price) }}</p>
            </router-link>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useProductStore } from '@/stores/products'
import { useCartStore } from '@/stores/cart'

export default {
  props: {
    id: {
      type: String,
      required: true
    }
  },
  
  setup(props) {
    const route = useRoute()
    const router = useRouter()
    const productStore = useProductStore()
    const cartStore = useCartStore()
    
    const loading = ref(false)
    const error = ref(null)
    
    const product = computed(() => productStore.getProductById(props.id))
    const relatedProducts = computed(() => {
      if (!product.value) return []
      return productStore.getProductsByCategory(product.value.category)
        .filter(p => p.id !== product.value.id)
        .slice(0, 4)
    })
    
    const loadProduct = async () => {
      loading.value = true
      error.value = null
      
      try {
        await productStore.fetchProduct(props.id)
        
        if (!product.value) {
          // 상품이 없으면 404 페이지로
          router.replace('/404')
          return
        }
        
      } catch (err) {
        error.value = err
      } finally {
        loading.value = false
      }
    }
    
    const retry = () => {
      loadProduct()
    }
    
    const addToCart = () => {
      cartStore.addItem(product.value)
      // 성공 메시지 표시
    }
    
    const buyNow = () => {
      cartStore.addItem(product.value)
      router.push('/checkout')
    }
    
    const formatPrice = (price) => {
      return new Intl.NumberFormat('ko-KR', {
        style: 'currency',
        currency: 'KRW'
      }).format(price)
    }
    
    // props.id가 변경될 때마다 상품 다시 로드
    watch(() => props.id, loadProduct, { immediate: true })
    
    return {
      loading,
      error,
      product,
      relatedProducts,
      retry,
      addToCart,
      buyNow,
      formatPrice
    }
  }
}
</script>
```

### 검색 결과 페이지
```vue
<!-- views/SearchResults.vue -->
<template>
  <div class="search-results">
    <div class="search-header">
      <h1>검색 결과</h1>
      <p v-if="query">
        "{{ query }}"에 대한 검색 결과 {{ totalResults }}건
      </p>
    </div>
    
    <div class="search-filters">
      <div class="filter-group">
        <label>카테고리:</label>
        <select v-model="filters.category">
          <option value="">전체</option>
          <option v-for="category in categories" :key="category" :value="category">
            {{ category }}
          </option>
        </select>
      </div>
      
      <div class="filter-group">
        <label>가격 범위:</label>
        <select v-model="filters.priceRange">
          <option value="">전체</option>
          <option value="0-50000">5만원 이하</option>
          <option value="50000-100000">5만원-10만원</option>
          <option value="100000-">10만원 이상</option>
        </select>
      </div>
      
      <div class="filter-group">
        <label>정렬:</label>
        <select v-model="filters.sortBy">
          <option value="relevance">관련도순</option>
          <option value="price-low">가격 낮은순</option>
          <option value="price-high">가격 높은순</option>
          <option value="name">이름순</option>
        </select>
      </div>
    </div>
    
    <div v-if="loading" class="loading">
      검색 중...
    </div>
    
    <div v-else-if="results.length === 0" class="no-results">
      검색 결과가 없습니다.
    </div>
    
    <div v-else class="results-grid">
      <ProductCard
        v-for="product in results"
        :key="product.id"
        :product="product"
      />
    </div>
    
    <!-- 페이지네이션 -->
    <Pagination
      v-if="totalPages > 1"
      v-model:current-page="currentPage"
      :total-pages="totalPages"
    />
  </div>
</template>

<script>
import { computed, watch } from 'vue'
import { useRouteState } from '@/composables/useRouteState'
import { useSearchStore } from '@/stores/search'
import ProductCard from '@/components/ProductCard.vue'
import Pagination from '@/components/Pagination.vue'

export default {
  components: {
    ProductCard,
    Pagination
  },
  
  setup() {
    const { route, filters, currentPage } = useRouteState()
    const searchStore = useSearchStore()
    
    const query = computed(() => route.query.q || '')
    const loading = computed(() => searchStore.loading)
    const results = computed(() => searchStore.results)
    const totalResults = computed(() => searchStore.totalResults)
    const totalPages = computed(() => searchStore.totalPages)
    const categories = computed(() => searchStore.availableCategories)
    
    // 검색 실행
    const performSearch = () => {
      if (!query.value) return
      
      searchStore.search({
        query: query.value,
        filters: filters.value,
        page: currentPage.value
      })
    }
    
    // 검색 조건 변경 시 검색 재실행
    watch([query, filters, currentPage], performSearch, { immediate: true })
    
    return {
      query,
      loading,
      results,
      totalResults,
      totalPages,
      categories,
      filters,
      currentPage
    }
  }
}
</script>
```

## 🎯 실습 과제

1. 위의 SPA 라우팅 구조를 직접 구현해보세요
2. 사용자 권한 기반 동적 메뉴를 만들어보세요
3. 모달 라우팅 시스템을 구현해보세요
4. 무한 스크롤 페이지네이션을 추가해보세요
5. PWA를 위한 오프라인 라우팅을 구현해보세요

## 🔗 다음 단계

다음 챕터에서는 Vue 애플리케이션의 성능 최적화에 대해 알아보겠습니다!

---

💡 **팁**: Vue Router는 React Router보다 더 직관적이고 강력한 기능을 제공합니다. 특히 네비게이션 가드와 메타 필드 활용이 매우 편리해요! 