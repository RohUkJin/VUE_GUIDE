# 3-5. 성능 최적화 (Performance Optimization)

## ⚡ Vue vs React: 성능 최적화 전략

Vue는 내장된 반응성 시스템 덕분에 React보다 더 적은 노력으로도 뛰어난 성능을 얻을 수 있습니다!

## 🎯 1. 컴포넌트 최적화 비교

### React 최적화
```jsx
import React, { memo, useMemo, useCallback, useState } from 'react';

// React.memo로 불필요한 리렌더링 방지
const ExpensiveComponent = memo(({ data, onUpdate }) => {
  // useMemo로 비싼 계산 캐싱
  const processedData = useMemo(() => {
    return data.map(item => ({
      ...item,
      processed: expensiveCalculation(item)
    }));
  }, [data]);

  // useCallback으로 함수 참조 안정화
  const handleClick = useCallback((id) => {
    onUpdate(id);
  }, [onUpdate]);

  return (
    <div>
      {processedData.map(item => (
        <ItemComponent 
          key={item.id}
          item={item}
          onClick={handleClick}
        />
      ))}
    </div>
  );
});

// 부모 컴포넌트
function App() {
  const [items, setItems] = useState([]);
  const [filter, setFilter] = useState('');

  // 필터된 데이터도 useMemo로 최적화
  const filteredItems = useMemo(() => {
    return items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);

  const handleUpdate = useCallback((id) => {
    setItems(prev => prev.map(item => 
      item.id === id ? { ...item, updated: true } : item
    ));
  }, []);

  return (
    <div>
      <input 
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="필터..."
      />
      <ExpensiveComponent 
        data={filteredItems}
        onUpdate={handleUpdate}
      />
    </div>
  );
}
```

### Vue 최적화
```vue
<template>
  <div>
    <input 
      v-model="filter" 
      placeholder="필터..."
    />
    
    <!-- Vue는 자동으로 불필요한 렌더링을 방지 -->
    <ExpensiveComponent 
      :data="filteredItems"
      @update="handleUpdate"
    />
  </div>
</template>

<script>
import { ref, computed } from 'vue'
import ExpensiveComponent from './ExpensiveComponent.vue'

export default {
  components: { ExpensiveComponent },
  
  setup() {
    const items = ref([])
    const filter = ref('')
    
    // computed는 자동으로 캐싱됨 (useMemo와 동일한 효과)
    const filteredItems = computed(() => {
      return items.value.filter(item => 
        item.name.toLowerCase().includes(filter.value.toLowerCase())
      )
    })
    
    // Vue에서는 함수를 직접 정의해도 성능 문제 없음
    const handleUpdate = (id) => {
      const item = items.value.find(item => item.id === id)
      if (item) {
        item.updated = true
      }
    }
    
    return {
      filter,
      filteredItems,
      handleUpdate
    }
  }
}
</script>
```

```vue
<!-- ExpensiveComponent.vue -->
<template>
  <div>
    <!-- v-memo로 조건부 메모이제이션 (Vue 3.2+) -->
    <div 
      v-for="item in processedData" 
      :key="item.id"
      v-memo="[item.id, item.updated]"
    >
      <ItemComponent 
        :item="item"
        @click="$emit('update', item.id)"
      />
    </div>
  </div>
</template>

<script>
import { computed } from 'vue'
import ItemComponent from './ItemComponent.vue'

export default {
  components: { ItemComponent },
  
  props: {
    data: {
      type: Array,
      required: true
    }
  },
  
  emits: ['update'],
  
  setup(props) {
    // computed는 자동으로 캐싱되고 의존성 추적
    const processedData = computed(() => {
      return props.data.map(item => ({
        ...item,
        processed: expensiveCalculation(item)
      }))
    })
    
    return {
      processedData
    }
  }
}

function expensiveCalculation(item) {
  // 비싼 계산 로직
  return item.value * Math.random()
}
</script>
```

## 🚀 2. 지연 로딩과 코드 분할

### React Code Splitting
```jsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// 컴포넌트 지연 로딩
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));
const Settings = lazy(() => import('./pages/Settings'));

// 번들 프리로딩
const AdminPanel = lazy(() => 
  import(/* webpackChunkName: "admin" */ './pages/AdminPanel')
);

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<Profile />} />
        <Route path="/settings" element={<Settings />} />
        <Route path="/admin" element={<AdminPanel />} />
      </Routes>
    </Suspense>
  );
}
```

### Vue Code Splitting
```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/dashboard',
    name: 'Dashboard',
    // 자동 코드 분할
    component: () => import('@/views/Dashboard.vue')
  },
  {
    path: '/profile',
    name: 'Profile',
    // 청크 이름 지정으로 번들 최적화
    component: () => import(/* webpackChunkName: "user" */ '@/views/Profile.vue')
  },
  {
    path: '/settings',
    name: 'Settings',
    component: () => import(/* webpackChunkName: "user" */ '@/views/Settings.vue')
  },
  {
    path: '/admin',
    name: 'Admin',
    component: () => import(/* webpackChunkName: "admin" */ '@/views/Admin.vue'),
    // 관리자 페이지는 권한 체크 후 프리로드
    meta: { preload: true, requiresRole: 'admin' }
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// 라우트 프리로딩
router.beforeEach((to, from, next) => {
  // 다음 페이지가 프리로드 대상이면 미리 로드
  if (to.meta.preload) {
    to.matched.forEach(record => {
      if (typeof record.components.default === 'function') {
        record.components.default()
      }
    })
  }
  next()
})

export default router
```

### 동적 컴포넌트 로딩
```vue
<template>
  <div class="dynamic-dashboard">
    <div class="widget-container">
      <component
        v-for="widget in enabledWidgets"
        :key="widget.id"
        :is="getWidgetComponent(widget.type)"
        :config="widget.config"
        @error="handleWidgetError"
      />
    </div>
  </div>
</template>

<script>
import { ref, shallowRef, defineAsyncComponent } from 'vue'

export default {
  setup() {
    const enabledWidgets = ref([
      { id: 1, type: 'chart', config: { chartType: 'line' } },
      { id: 2, type: 'table', config: { pageSize: 10 } },
      { id: 3, type: 'calendar', config: { view: 'month' } }
    ])
    
    // 위젯 컴포넌트 캐시
    const widgetComponents = new Map()
    
    const getWidgetComponent = (type) => {
      if (!widgetComponents.has(type)) {
        const component = defineAsyncComponent({
          loader: () => import(`@/components/widgets/${type}Widget.vue`),
          loadingComponent: () => import('@/components/WidgetLoading.vue'),
          errorComponent: () => import('@/components/WidgetError.vue'),
          delay: 200,
          timeout: 5000
        })
        
        widgetComponents.set(type, component)
      }
      
      return widgetComponents.get(type)
    }
    
    const handleWidgetError = (error, widgetId) => {
      console.error(`Widget ${widgetId} error:`, error)
      // 에러 위젯 제거 또는 대체
    }
    
    return {
      enabledWidgets,
      getWidgetComponent,
      handleWidgetError
    }
  }
}
</script>
```

## 🎭 3. 가상화와 대용량 데이터 처리

### 가상 스크롤 구현
```vue
<template>
  <div class="virtual-list" ref="containerRef" @scroll="handleScroll">
    <div :style="{ height: totalHeight + 'px' }" class="scroll-container">
      <div
        :style="{ 
          transform: `translateY(${offsetY}px)`,
          position: 'relative'
        }"
        class="visible-items"
      >
        <div
          v-for="item in visibleItems"
          :key="item.id"
          :style="{ height: itemHeight + 'px' }"
          class="list-item"
        >
          <slot :item="item" :index="item.index" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted } from 'vue'

export default {
  props: {
    items: {
      type: Array,
      required: true
    },
    itemHeight: {
      type: Number,
      default: 50
    },
    containerHeight: {
      type: Number,
      default: 400
    },
    overscan: {
      type: Number,
      default: 5
    }
  },
  
  setup(props) {
    const containerRef = ref()
    const scrollTop = ref(0)
    
    const totalHeight = computed(() => props.items.length * props.itemHeight)
    
    const visibleCount = computed(() => 
      Math.ceil(props.containerHeight / props.itemHeight)
    )
    
    const startIndex = computed(() => 
      Math.max(0, Math.floor(scrollTop.value / props.itemHeight) - props.overscan)
    )
    
    const endIndex = computed(() => 
      Math.min(
        props.items.length - 1,
        startIndex.value + visibleCount.value + props.overscan * 2
      )
    )
    
    const visibleItems = computed(() => {
      const items = []
      for (let i = startIndex.value; i <= endIndex.value; i++) {
        if (props.items[i]) {
          items.push({
            ...props.items[i],
            index: i
          })
        }
      }
      return items
    })
    
    const offsetY = computed(() => startIndex.value * props.itemHeight)
    
    const handleScroll = () => {
      if (containerRef.value) {
        scrollTop.value = containerRef.value.scrollTop
      }
    }
    
    // 스크롤 최적화
    let rafId = null
    const optimizedScroll = () => {
      if (rafId) return
      
      rafId = requestAnimationFrame(() => {
        handleScroll()
        rafId = null
      })
    }
    
    onMounted(() => {
      if (containerRef.value) {
        containerRef.value.addEventListener('scroll', optimizedScroll, { passive: true })
      }
    })
    
    onUnmounted(() => {
      if (containerRef.value) {
        containerRef.value.removeEventListener('scroll', optimizedScroll)
      }
      if (rafId) {
        cancelAnimationFrame(rafId)
      }
    })
    
    return {
      containerRef,
      totalHeight,
      offsetY,
      visibleItems
    }
  }
}
</script>

<style scoped>
.virtual-list {
  height: 100%;
  overflow-y: auto;
}

.scroll-container {
  position: relative;
}

.list-item {
  display: flex;
  align-items: center;
  padding: 0 16px;
  border-bottom: 1px solid #eee;
}
</style>
```

### 무한 스크롤 구현
```vue
<template>
  <div class="infinite-scroll">
    <div
      v-for="item in items"
      :key="item.id"
      class="item"
    >
      <slot :item="item" />
    </div>
    
    <div
      v-if="hasMore"
      ref="loadMoreTrigger"
      class="load-more-trigger"
    >
      <div v-if="loading" class="loading">
        더 많은 항목을 불러오는 중...
      </div>
    </div>
    
    <div v-if="!hasMore && items.length > 0" class="end-message">
      모든 항목을 불러왔습니다.
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue'

export default {
  props: {
    loadMore: {
      type: Function,
      required: true
    },
    hasMore: {
      type: Boolean,
      default: true
    },
    items: {
      type: Array,
      default: () => []
    }
  },
  
  setup(props) {
    const loadMoreTrigger = ref()
    const loading = ref(false)
    let observer = null
    
    const handleIntersection = async (entries) => {
      const entry = entries[0]
      
      if (entry.isIntersecting && props.hasMore && !loading.value) {
        loading.value = true
        
        try {
          await props.loadMore()
        } catch (error) {
          console.error('Failed to load more items:', error)
        } finally {
          loading.value = false
        }
      }
    }
    
    onMounted(() => {
      if (loadMoreTrigger.value) {
        observer = new IntersectionObserver(handleIntersection, {
          rootMargin: '100px' // 100px 전에 미리 로드
        })
        
        observer.observe(loadMoreTrigger.value)
      }
    })
    
    onUnmounted(() => {
      if (observer && loadMoreTrigger.value) {
        observer.unobserve(loadMoreTrigger.value)
        observer.disconnect()
      }
    })
    
    return {
      loadMoreTrigger,
      loading
    }
  }
}
</script>
```

## 🎨 4. 이미지와 미디어 최적화

### 지연 로딩 이미지 컴포넌트
```vue
<template>
  <div class="lazy-image" :class="{ loading, loaded, error }">
    <img
      v-if="loaded"
      :src="src"
      :alt="alt"
      @load="onLoad"
      @error="onError"
    />
    
    <div v-else-if="loading" class="placeholder">
      <div v-if="placeholder" class="placeholder-content">
        <img :src="placeholder" alt="" />
      </div>
      <div v-else class="loading-spinner">
        로딩 중...
      </div>
    </div>
    
    <div v-else-if="error" class="error-state">
      <div class="error-icon">❌</div>
      <p>이미지를 불러올 수 없습니다</p>
      <button @click="retry">다시 시도</button>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue'

export default {
  props: {
    src: {
      type: String,
      required: true
    },
    alt: {
      type: String,
      default: ''
    },
    placeholder: {
      type: String,
      default: ''
    },
    threshold: {
      type: Number,
      default: 0.1
    }
  },
  
  setup(props) {
    const loading = ref(false)
    const loaded = ref(false)
    const error = ref(false)
    const imageRef = ref()
    let observer = null
    
    const loadImage = () => {
      loading.value = true
      error.value = false
      
      const img = new Image()
      
      img.onload = () => {
        loaded.value = true
        loading.value = false
      }
      
      img.onerror = () => {
        error.value = true
        loading.value = false
      }
      
      img.src = props.src
    }
    
    const handleIntersection = (entries) => {
      const entry = entries[0]
      
      if (entry.isIntersecting) {
        loadImage()
        if (observer) {
          observer.disconnect()
          observer = null
        }
      }
    }
    
    const retry = () => {
      error.value = false
      loadImage()
    }
    
    onMounted(() => {
      // Intersection Observer를 사용한 지연 로딩
      observer = new IntersectionObserver(handleIntersection, {
        threshold: props.threshold
      })
      
      if (imageRef.value) {
        observer.observe(imageRef.value)
      }
    })
    
    onUnmounted(() => {
      if (observer) {
        observer.disconnect()
      }
    })
    
    return {
      loading,
      loaded,
      error,
      imageRef,
      retry
    }
  }
}
</script>

<style scoped>
.lazy-image {
  position: relative;
  display: inline-block;
  overflow: hidden;
}

.placeholder {
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f5f5f5;
  min-height: 200px;
}

.placeholder-content img {
  filter: blur(2px);
  opacity: 0.7;
}

.loading-spinner {
  padding: 20px;
  color: #666;
}

.error-state {
  text-align: center;
  padding: 20px;
  background: #fff5f5;
  color: #d63031;
}

.error-icon {
  font-size: 24px;
  margin-bottom: 10px;
}

.lazy-image.loading {
  animation: pulse 1.5s ease-in-out infinite;
}

.lazy-image.loaded img {
  animation: fadeIn 0.3s ease-in-out;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.7; }
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
</style>
```

### 반응형 이미지 컴포넌트
```vue
<template>
  <picture class="responsive-image">
    <source
      v-for="source in sources"
      :key="source.media"
      :media="source.media"
      :srcset="source.srcset"
      :type="source.type"
    />
    <LazyImage
      :src="fallbackSrc"
      :alt="alt"
      :placeholder="placeholder"
    />
  </picture>
</template>

<script>
import { computed } from 'vue'
import LazyImage from './LazyImage.vue'

export default {
  components: { LazyImage },
  
  props: {
    baseSrc: {
      type: String,
      required: true
    },
    alt: {
      type: String,
      default: ''
    },
    placeholder: {
      type: String,
      default: ''
    },
    sizes: {
      type: Array,
      default: () => [320, 640, 1024, 1440]
    },
    formats: {
      type: Array,
      default: () => ['webp', 'jpg']
    }
  },
  
  setup(props) {
    const sources = computed(() => {
      const result = []
      
      // 각 포맷과 크기 조합으로 소스 생성
      props.formats.forEach(format => {
        const srcset = props.sizes
          .map(size => `${getImageUrl(props.baseSrc, size, format)} ${size}w`)
          .join(', ')
        
        result.push({
          media: `(min-width: ${Math.min(...props.sizes)}px)`,
          srcset,
          type: `image/${format}`
        })
      })
      
      return result
    })
    
    const fallbackSrc = computed(() => {
      return getImageUrl(props.baseSrc, 640, 'jpg')
    })
    
    const getImageUrl = (baseSrc, width, format) => {
      // 이미지 CDN이나 서버에서 제공하는 변환 API 사용
      const baseUrl = baseSrc.replace(/\.[^.]+$/, '')
      return `${baseUrl}_${width}w.${format}`
    }
    
    return {
      sources,
      fallbackSrc
    }
  }
}
</script>
```

## 🎯 5. 전체 애플리케이션 성능 모니터링

### 성능 측정 Composable
```javascript
// composables/usePerformance.js
import { ref, onMounted, onUnmounted } from 'vue'

export function usePerformance() {
  const metrics = ref({
    fcp: 0, // First Contentful Paint
    lcp: 0, // Largest Contentful Paint
    fid: 0, // First Input Delay
    cls: 0, // Cumulative Layout Shift
    ttfb: 0 // Time to First Byte
  })
  
  const observer = ref(null)
  
  const measureWebVitals = () => {
    // FCP 측정
    const perfObserver = new PerformanceObserver((entryList) => {
      for (const entry of entryList.getEntries()) {
        if (entry.name === 'first-contentful-paint') {
          metrics.value.fcp = entry.startTime
        }
      }
    })
    
    perfObserver.observe({ entryTypes: ['paint'] })
    
    // LCP 측정
    const lcpObserver = new PerformanceObserver((entryList) => {
      const entries = entryList.getEntries()
      const lastEntry = entries[entries.length - 1]
      metrics.value.lcp = lastEntry.startTime
    })
    
    lcpObserver.observe({ entryTypes: ['largest-contentful-paint'] })
    
    // FID 측정
    const fidObserver = new PerformanceObserver((entryList) => {
      for (const entry of entryList.getEntries()) {
        metrics.value.fid = entry.processingStart - entry.startTime
      }
    })
    
    fidObserver.observe({ entryTypes: ['first-input'] })
    
    // CLS 측정
    let clsValue = 0
    const clsObserver = new PerformanceObserver((entryList) => {
      for (const entry of entryList.getEntries()) {
        if (!entry.hadRecentInput) {
          clsValue += entry.value
          metrics.value.cls = clsValue
        }
      }
    })
    
    clsObserver.observe({ entryTypes: ['layout-shift'] })
    
    observer.value = {
      perf: perfObserver,
      lcp: lcpObserver,
      fid: fidObserver,
      cls: clsObserver
    }
  }
  
  const getNavigationTiming = () => {
    const navigation = performance.getEntriesByType('navigation')[0]
    if (navigation) {
      metrics.value.ttfb = navigation.responseStart - navigation.requestStart
    }
  }
  
  const sendMetrics = () => {
    // 성능 데이터를 분석 서비스로 전송
    if (window.gtag) {
      Object.entries(metrics.value).forEach(([name, value]) => {
        window.gtag('event', name, {
          event_category: 'Web Vitals',
          value: Math.round(value),
          non_interaction: true
        })
      })
    }
  }
  
  onMounted(() => {
    if (typeof PerformanceObserver !== 'undefined') {
      measureWebVitals()
      getNavigationTiming()
      
      // 페이지 언로드 시 메트릭 전송
      window.addEventListener('beforeunload', sendMetrics)
    }
  })
  
  onUnmounted(() => {
    if (observer.value) {
      Object.values(observer.value).forEach(obs => obs.disconnect())
    }
    window.removeEventListener('beforeunload', sendMetrics)
  })
  
  return {
    metrics
  }
}
```

### 렌더링 성능 프로파일러
```vue
<template>
  <div class="performance-profiler">
    <div class="metrics-display">
      <h3>성능 메트릭</h3>
      <div class="metric" v-for="(value, key) in performanceMetrics" :key="key">
        <span class="metric-name">{{ key.toUpperCase() }}:</span>
        <span class="metric-value" :class="getMetricClass(key, value)">
          {{ formatMetric(key, value) }}
        </span>
      </div>
    </div>
    
    <div class="render-profiler">
      <h3>렌더링 프로파일</h3>
      <div class="profiler-data">
        <p>총 렌더링 시간: {{ totalRenderTime }}ms</p>
        <p>평균 프레임 시간: {{ averageFrameTime }}ms</p>
        <p>느린 프레임: {{ slowFrames }}개</p>
      </div>
    </div>
    
    <div class="memory-usage">
      <h3>메모리 사용량</h3>
      <div v-if="memoryInfo" class="memory-stats">
        <p>사용 중: {{ formatBytes(memoryInfo.usedJSHeapSize) }}</p>
        <p>총 할당: {{ formatBytes(memoryInfo.totalJSHeapSize) }}</p>
        <p>한계: {{ formatBytes(memoryInfo.jsHeapSizeLimit) }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { usePerformance } from '@/composables/usePerformance'

export default {
  setup() {
    const { metrics: performanceMetrics } = usePerformance()
    const frameTimings = ref([])
    const memoryInfo = ref(null)
    let animationId = null
    
    const totalRenderTime = computed(() => {
      return frameTimings.value.reduce((sum, time) => sum + time, 0)
    })
    
    const averageFrameTime = computed(() => {
      return frameTimings.value.length > 0
        ? totalRenderTime.value / frameTimings.value.length
        : 0
    })
    
    const slowFrames = computed(() => {
      return frameTimings.value.filter(time => time > 16.67).length
    })
    
    const measureFramePerformance = () => {
      let lastTime = performance.now()
      
      const measure = (currentTime) => {
        const frameTime = currentTime - lastTime
        frameTimings.value.push(frameTime)
        
        // 최근 100프레임만 유지
        if (frameTimings.value.length > 100) {
          frameTimings.value.shift()
        }
        
        lastTime = currentTime
        animationId = requestAnimationFrame(measure)
      }
      
      animationId = requestAnimationFrame(measure)
    }
    
    const updateMemoryInfo = () => {
      if (performance.memory) {
        memoryInfo.value = {
          usedJSHeapSize: performance.memory.usedJSHeapSize,
          totalJSHeapSize: performance.memory.totalJSHeapSize,
          jsHeapSizeLimit: performance.memory.jsHeapSizeLimit
        }
      }
    }
    
    const getMetricClass = (key, value) => {
      const thresholds = {
        fcp: { good: 1800, poor: 3000 },
        lcp: { good: 2500, poor: 4000 },
        fid: { good: 100, poor: 300 },
        cls: { good: 0.1, poor: 0.25 },
        ttfb: { good: 800, poor: 1800 }
      }
      
      const threshold = thresholds[key]
      if (!threshold) return ''
      
      if (value <= threshold.good) return 'good'
      if (value <= threshold.poor) return 'needs-improvement'
      return 'poor'
    }
    
    const formatMetric = (key, value) => {
      if (key === 'cls') {
        return value.toFixed(3)
      }
      return `${Math.round(value)}ms`
    }
    
    const formatBytes = (bytes) => {
      if (bytes === 0) return '0 Bytes'
      
      const k = 1024
      const sizes = ['Bytes', 'KB', 'MB', 'GB']
      const i = Math.floor(Math.log(bytes) / Math.log(k))
      
      return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
    }
    
    onMounted(() => {
      measureFramePerformance()
      updateMemoryInfo()
      
      // 5초마다 메모리 정보 업데이트
      const memoryInterval = setInterval(updateMemoryInfo, 5000)
      
      onUnmounted(() => {
        if (animationId) {
          cancelAnimationFrame(animationId)
        }
        clearInterval(memoryInterval)
      })
    })
    
    return {
      performanceMetrics,
      totalRenderTime,
      averageFrameTime,
      slowFrames,
      memoryInfo,
      getMetricClass,
      formatMetric,
      formatBytes
    }
  }
}
</script>

<style scoped>
.performance-profiler {
  padding: 20px;
  font-family: monospace;
  background: #f8f9fa;
  border-radius: 8px;
}

.metrics-display,
.render-profiler,
.memory-usage {
  margin-bottom: 20px;
  padding: 15px;
  background: white;
  border-radius: 4px;
}

.metric {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
}

.metric-value.good {
  color: #28a745;
}

.metric-value.needs-improvement {
  color: #ffc107;
}

.metric-value.poor {
  color: #dc3545;
}

.memory-stats p {
  margin: 5px 0;
}
</style>
```

## 🎯 실습 과제

1. 위의 성능 최적화 기법들을 직접 구현해보세요
2. 가상 테이블 컴포넌트를 만들어보세요
3. 이미지 압축과 포맷 변환 서비스를 구현해보세요
4. Service Worker를 활용한 캐싱 전략을 구현해보세요
5. 번들 분석과 최적화 보고서를 만들어보세요

## 🎉 축하합니다!

Vue.js 완전 학습 가이드의 모든 과정을 완료하셨습니다! 🎊

### 🚀 다음 단계 추천

1. **실제 프로젝트 구축**: 배운 내용을 활용해 완전한 웹 애플리케이션을 만들어보세요
2. **오픈소스 기여**: Vue 생태계의 오픈소스 프로젝트에 기여해보세요
3. **성능 최적화**: 실제 서비스에서 성능 병목을 찾고 최적화해보세요
4. **TypeScript 통합**: Vue + TypeScript로 더 안전한 애플리케이션을 개발해보세요
5. **팀 리딩**: 다른 개발자들에게 Vue.js를 가르쳐보세요

### 📚 추가 학습 자료

- [Vue.js 공식 문서](https://vuejs.org/)
- [Pinia 공식 문서](https://pinia.vuejs.org/)
- [Vue Router 공식 문서](https://router.vuejs.org/)
- [Vue 생태계 Awesome 리스트](https://github.com/vuejs/awesome-vue)

---

💡 **마지막 팁**: Vue.js의 강력함은 단순함에 있습니다. 복잡한 React 패턴에 익숙하셨다면, Vue의 직관적인 접근 방식이 얼마나 개발 경험을 향상시키는지 느끼셨을 것입니다. 이제 Vue 마스터가 되어 멋진 웹 애플리케이션을 만들어보세요! 🚀 