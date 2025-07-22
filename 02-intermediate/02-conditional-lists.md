# 2-2. 조건부 렌더링 & 리스트 (Conditional Rendering & Lists)

## 🎯 Vue vs React: 조건부 렌더링과 리스트 최적화

Vue는 React보다 더 직관적이고 성능이 좋은 조건부 렌더링과 리스트 처리 방법을 제공합니다!

**🎯 학습 목표**
- Vue의 `v-if`, `v-show` vs React 조건부 렌더링 차이점 완전 정복
- `v-for`의 성능 최적화 기법 (key 사용, 가상 스크롤링 등) 습득
- 복잡한 조건문을 직관적으로 표현하는 Vue만의 방법 학습
- 대용량 리스트 렌더링 최적화 실무 기법 익히기

**💡 왜 Vue의 렌더링이 더 직관적일까요?**
- **선언적 문법**: `v-if`, `v-else-if`, `v-else`로 조건을 명확하게 표현
- **DOM 최적화**: `v-show`는 CSS display만 변경, `v-if`는 실제 DOM 추가/제거
- **리스트 최적화**: Vue의 가상 DOM이 자동으로 최소한의 변경만 적용
- **개발자 경험**: 템플릿에서 바로 렌더링 로직 확인 가능

## 🔄 1. 고급 조건부 렌더링

### React 방식
```jsx
function UserDashboard({ user, permissions, isLoading, error }) {
  if (isLoading) {
    return <div className="loading">로딩 중...</div>;
  }
  
  if (error) {
    return <div className="error">오류: {error.message}</div>;
  }
  
  if (!user) {
    return <div className="no-user">사용자를 찾을 수 없습니다.</div>;
  }
  
  return (
    <div className="dashboard">
      <h1>안녕하세요, {user.name}님!</h1>
      
      {user.role === 'admin' && (
        <div className="admin-panel">
          <h2>관리자 패널</h2>
          {permissions.canManageUsers && <UserManagement />}
          {permissions.canViewAnalytics && <Analytics />}
        </div>
      )}
      
      {user.role === 'moderator' ? (
        <ModeratorTools />
      ) : user.role === 'user' ? (
        <UserContent />
      ) : null}
      
      {user.isPremium && <PremiumFeatures />}
    </div>
  );
}
```

### Vue 방식
```vue
<!-- UserDashboard.vue -->
<template>
  <div>
    <!-- 로딩 상태 -->
    <div v-if="isLoading" class="loading">
      <div class="spinner"></div>
      <p>로딩 중...</p>
    </div>
    
    <!-- 에러 상태 -->
    <div v-else-if="error" class="error">
      <h3>⚠️ 오류가 발생했습니다</h3>
      <p>{{ error.message }}</p>
      <button @click="retryLoad">다시 시도</button>
    </div>
    
    <!-- 사용자 없음 -->
    <div v-else-if="!user" class="no-user">
      <h3>🔍 사용자를 찾을 수 없습니다</h3>
      <button @click="$emit('go-to-login')">로그인하기</button>
    </div>
    
    <!-- 메인 대시보드 -->
    <div v-else class="dashboard">
      <header class="dashboard-header">
        <h1>안녕하세요, {{ user.name }}님! 👋</h1>
        <UserBadge :user="user" />
      </header>
      
      <!-- 관리자 패널 -->
      <section v-if="user.role === 'admin'" class="admin-panel">
        <h2>🛠️ 관리자 패널</h2>
        <div class="admin-tools">
          <UserManagement v-if="permissions.canManageUsers" />
          <Analytics v-if="permissions.canViewAnalytics" />
          <SystemSettings v-if="permissions.canManageSystem" />
        </div>
      </section>
      
      <!-- 역할별 컨텐츠 -->
      <section class="role-content">
        <ModeratorTools v-if="user.role === 'moderator'" />
        <UserContent v-else-if="user.role === 'user'" />
        <GuestContent v-else />
      </section>
      
      <!-- 프리미엄 기능 -->
      <section v-if="user.isPremium" class="premium-section">
        <h2>⭐ 프리미엄 기능</h2>
        <PremiumFeatures />
      </section>
      
      <!-- 조건부 알림 -->
      <div v-show="shouldShowNotification" class="notification">
        <template v-if="user.unreadMessages > 0">
          📬 새 메시지 {{ user.unreadMessages }}개가 있습니다!
        </template>
        <template v-else-if="user.pendingTasks > 0">
          📋 완료하지 않은 작업 {{ user.pendingTasks }}개가 있습니다.
        </template>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    user: Object,
    permissions: Object,
    isLoading: Boolean,
    error: Object
  },
  
  computed: {
    shouldShowNotification() {
      return this.user && (this.user.unreadMessages > 0 || this.user.pendingTasks > 0);
    }
  },
  
  methods: {
    retryLoad() {
      this.$emit('retry-load');
    }
  }
}
</script>
```

## 📋 2. 고성능 리스트 렌더링

### React 방식
```jsx
import { memo, useMemo } from 'react';

const ListItem = memo(({ item, onEdit, onDelete, isSelected }) => {
  return (
    <div className={`list-item ${isSelected ? 'selected' : ''}`}>
      <div className="item-content">
        <h3>{item.title}</h3>
        <p>{item.description}</p>
        <span className="date">{item.date}</span>
      </div>
      <div className="item-actions">
        <button onClick={() => onEdit(item.id)}>수정</button>
        <button onClick={() => onDelete(item.id)}>삭제</button>
      </div>
    </div>
  );
});

function ItemList({ items, filter, selectedIds, onEdit, onDelete }) {
  const filteredItems = useMemo(() => {
    return items.filter(item => {
      if (filter.category && item.category !== filter.category) return false;
      if (filter.search && !item.title.toLowerCase().includes(filter.search.toLowerCase())) return false;
      return true;
    });
  }, [items, filter]);
  
  const sortedItems = useMemo(() => {
    return [...filteredItems].sort((a, b) => {
      switch (filter.sortBy) {
        case 'date': return new Date(b.date) - new Date(a.date);
        case 'title': return a.title.localeCompare(b.title);
        default: return 0;
      }
    });
  }, [filteredItems, filter.sortBy]);
  
  return (
    <div className="item-list">
      {sortedItems.map(item => (
        <ListItem
          key={item.id}
          item={item}
          onEdit={onEdit}
          onDelete={onDelete}
          isSelected={selectedIds.includes(item.id)}
        />
      ))}
    </div>
  );
}
```

### Vue 방식
```vue
<!-- ItemList.vue -->
<template>
  <div class="item-list">
    <!-- 필터 및 정렬 컨트롤 -->
    <div class="list-controls">
      <div class="filters">
        <select v-model="localFilter.category">
          <option value="">모든 카테고리</option>
          <option v-for="category in availableCategories" :key="category" :value="category">
            {{ category }}
          </option>
        </select>
        
        <input 
          v-model="localFilter.search" 
          placeholder="검색..." 
          class="search-input"
        />
      </div>
      
      <div class="sort-options">
        <label>
          <input type="radio" v-model="localFilter.sortBy" value="date" />
          날짜순
        </label>
        <label>
          <input type="radio" v-model="localFilter.sortBy" value="title" />
          제목순
        </label>
        <label>
          <input type="radio" v-model="localFilter.sortBy" value="category" />
          카테고리순
        </label>
      </div>
    </div>
    
    <!-- 리스트 통계 -->
    <div class="list-stats">
      <span>총 {{ filteredAndSortedItems.length }}개 항목</span>
      <span v-if="selectedIds.length > 0">
        ({{ selectedIds.length }}개 선택됨)
      </span>
    </div>
    
    <!-- 대용량 리스트 가상화 (필요시) -->
    <div class="items-container" ref="container">
      <transition-group name="list" tag="div">
        <ListItem
          v-for="item in paginatedItems"
          :key="item.id"
          :item="item"
          :is-selected="selectedIds.includes(item.id)"
          @edit="$emit('edit', $event)"
          @delete="$emit('delete', $event)"
          @toggle-select="toggleSelect"
        />
      </transition-group>
    </div>
    
    <!-- 페이지네이션 -->
    <div v-if="totalPages > 1" class="pagination">
      <button 
        @click="currentPage = Math.max(1, currentPage - 1)"
        :disabled="currentPage === 1"
      >
        이전
      </button>
      
      <span>{{ currentPage }} / {{ totalPages }}</span>
      
      <button 
        @click="currentPage = Math.min(totalPages, currentPage + 1)"
        :disabled="currentPage === totalPages"
      >
        다음
      </button>
    </div>
    
    <!-- 빈 상태 -->
    <div v-if="filteredAndSortedItems.length === 0" class="empty-state">
      <div v-if="localFilter.search || localFilter.category">
        <h3>🔍 검색 결과가 없습니다</h3>
        <p>다른 조건으로 검색해보세요.</p>
        <button @click="clearFilters">필터 초기화</button>
      </div>
      <div v-else>
        <h3>📝 아직 항목이 없습니다</h3>
        <p>첫 번째 항목을 추가해보세요!</p>
        <button @click="$emit('create-new')">새 항목 추가</button>
      </div>
    </div>
  </div>
</template>

<script>
import ListItem from './ListItem.vue'

export default {
  components: { ListItem },
  
  props: {
    items: {
      type: Array,
      required: true
    },
    filter: {
      type: Object,
      default: () => ({ category: '', search: '', sortBy: 'date' })
    },
    selectedIds: {
      type: Array,
      default: () => []
    },
    itemsPerPage: {
      type: Number,
      default: 20
    }
  },
  
  data() {
    return {
      localFilter: { ...this.filter },
      currentPage: 1
    }
  },
  
  computed: {
    availableCategories() {
      const categories = new Set(this.items.map(item => item.category));
      return Array.from(categories).sort();
    },
    
    filteredAndSortedItems() {
      let filtered = this.items.filter(item => {
        // 카테고리 필터
        if (this.localFilter.category && item.category !== this.localFilter.category) {
          return false;
        }
        
        // 검색 필터
        if (this.localFilter.search) {
          const searchTerm = this.localFilter.search.toLowerCase();
          return item.title.toLowerCase().includes(searchTerm) ||
                 item.description.toLowerCase().includes(searchTerm);
        }
        
        return true;
      });
      
      // 정렬
      return filtered.sort((a, b) => {
        switch (this.localFilter.sortBy) {
          case 'date':
            return new Date(b.date) - new Date(a.date);
          case 'title':
            return a.title.localeCompare(b.title);
          case 'category':
            return a.category.localeCompare(b.category);
          default:
            return 0;
        }
      });
    },
    
    totalPages() {
      return Math.ceil(this.filteredAndSortedItems.length / this.itemsPerPage);
    },
    
    paginatedItems() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.filteredAndSortedItems.slice(start, end);
    }
  },
  
  watch: {
    filter: {
      handler(newFilter) {
        this.localFilter = { ...newFilter };
      },
      deep: true
    },
    
    localFilter: {
      handler(newFilter) {
        this.$emit('filter-change', newFilter);
        this.currentPage = 1; // 필터 변경시 첫 페이지로
      },
      deep: true
    }
  },
  
  methods: {
    toggleSelect(itemId) {
      this.$emit('toggle-select', itemId);
    },
    
    clearFilters() {
      this.localFilter = {
        category: '',
        search: '',
        sortBy: 'date'
      };
    }
  }
}
</script>

<style scoped>
.list-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 8px;
}

.filters {
  display: flex;
  gap: 10px;
}

.sort-options {
  display: flex;
  gap: 15px;
}

.list-stats {
  margin-bottom: 15px;
  font-size: 14px;
  color: #666;
}

.items-container {
  min-height: 300px;
}

/* 리스트 애니메이션 */
.list-enter-active,
.list-leave-active {
  transition: all 0.3s ease;
}

.list-enter-from {
  opacity: 0;
  transform: translateY(-20px);
}

.list-leave-to {
  opacity: 0;
  transform: translateX(20px);
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
  margin-top: 20px;
}

.pagination button {
  padding: 8px 16px;
  border: 1px solid #ddd;
  background: white;
  border-radius: 4px;
  cursor: pointer;
}

.pagination button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.empty-state {
  text-align: center;
  padding: 60px 20px;
  color: #666;
}

.empty-state h3 {
  margin-bottom: 10px;
  color: #333;
}

.empty-state button {
  margin-top: 15px;
  padding: 10px 20px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
</style>
```

```vue
<!-- ListItem.vue -->
<template>
  <div 
    :class="['list-item', { selected: isSelected, 'high-priority': item.priority === 'high' }]"
    @click="$emit('toggle-select', item.id)"
  >
    <div class="item-selector">
      <input 
        type="checkbox" 
        :checked="isSelected"
        @change="$emit('toggle-select', item.id)"
        @click.stop
      />
    </div>
    
    <div class="item-content">
      <div class="item-header">
        <h3>{{ item.title }}</h3>
        <div class="item-meta">
          <span class="category">{{ item.category }}</span>
          <span class="date">{{ formattedDate }}</span>
          <span v-if="item.priority === 'high'" class="priority-badge">
            🔥 높음
          </span>
        </div>
      </div>
      
      <p class="description">{{ item.description }}</p>
      
      <div class="item-tags">
        <span 
          v-for="tag in item.tags" 
          :key="tag"
          class="tag"
        >
          {{ tag }}
        </span>
      </div>
    </div>
    
    <div class="item-actions" @click.stop>
      <button @click="$emit('edit', item.id)" class="edit-btn">
        ✏️ 수정
      </button>
      <button @click="confirmDelete" class="delete-btn">
        🗑️ 삭제
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    item: {
      type: Object,
      required: true
    },
    isSelected: {
      type: Boolean,
      default: false
    }
  },
  
  computed: {
    formattedDate() {
      return new Date(this.item.date).toLocaleDateString('ko-KR');
    }
  },
  
  methods: {
    confirmDelete() {
      if (confirm(`"${this.item.title}"을(를) 정말 삭제하시겠습니까?`)) {
        this.$emit('delete', this.item.id);
      }
    }
  }
}
</script>

<style scoped>
.list-item {
  display: flex;
  align-items: flex-start;
  gap: 15px;
  padding: 20px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  margin-bottom: 10px;
  background: white;
  transition: all 0.2s ease;
  cursor: pointer;
}

.list-item:hover {
  border-color: #42b883;
  box-shadow: 0 2px 8px rgba(66, 184, 131, 0.1);
}

.list-item.selected {
  border-color: #42b883;
  background: #f0fdf4;
}

.list-item.high-priority {
  border-left: 4px solid #ff6b6b;
}

.item-selector {
  margin-top: 5px;
}

.item-content {
  flex: 1;
}

.item-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 10px;
}

.item-header h3 {
  margin: 0;
  font-size: 18px;
  color: #333;
}

.item-meta {
  display: flex;
  gap: 10px;
  font-size: 12px;
  color: #666;
}

.category {
  background: #e3f2fd;
  color: #1976d2;
  padding: 2px 8px;
  border-radius: 12px;
}

.priority-badge {
  background: #ffebee;
  color: #c62828;
  padding: 2px 8px;
  border-radius: 12px;
}

.description {
  margin: 10px 0;
  color: #666;
  line-height: 1.5;
}

.item-tags {
  display: flex;
  gap: 5px;
  flex-wrap: wrap;
}

.tag {
  background: #f5f5f5;
  color: #333;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 12px;
}

.item-actions {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.item-actions button {
  padding: 6px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  white-space: nowrap;
}

.edit-btn {
  background: #e3f2fd;
  color: #1976d2;
}

.delete-btn {
  background: #ffebee;
  color: #c62828;
}

.item-actions button:hover {
  opacity: 0.8;
}
</style>
```

## 🚀 3. 가상 스크롤링 (Virtual Scrolling)

```vue
<!-- VirtualList.vue -->
<template>
  <div class="virtual-list" ref="container" @scroll="handleScroll">
    <div class="spacer-before" :style="{ height: offsetBefore + 'px' }"></div>
    
    <div class="visible-items">
      <slot 
        v-for="item in visibleItems" 
        :key="item.id"
        :item="item"
        :index="item.index"
      ></slot>
    </div>
    
    <div class="spacer-after" :style="{ height: offsetAfter + 'px' }"></div>
  </div>
</template>

<script>
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
      default: 300
    },
    overscan: {
      type: Number,
      default: 5
    }
  },
  
  data() {
    return {
      scrollTop: 0
    }
  },
  
  computed: {
    visibleStartIndex() {
      return Math.max(0, Math.floor(this.scrollTop / this.itemHeight) - this.overscan);
    },
    
    visibleEndIndex() {
      const visibleCount = Math.ceil(this.containerHeight / this.itemHeight);
      return Math.min(
        this.items.length - 1, 
        this.visibleStartIndex + visibleCount + this.overscan * 2
      );
    },
    
    visibleItems() {
      return this.items
        .slice(this.visibleStartIndex, this.visibleEndIndex + 1)
        .map((item, index) => ({
          ...item,
          index: this.visibleStartIndex + index
        }));
    },
    
    offsetBefore() {
      return this.visibleStartIndex * this.itemHeight;
    },
    
    offsetAfter() {
      return (this.items.length - this.visibleEndIndex - 1) * this.itemHeight;
    }
  },
  
  methods: {
    handleScroll() {
      this.scrollTop = this.$refs.container.scrollTop;
    }
  }
}
</script>

<style scoped>
.virtual-list {
  height: 100%;
  overflow-y: auto;
}
</style>
```

## 🎯 4. 실습 예제: 대시보드 위젯 시스템

```vue
<!-- Dashboard.vue -->
<template>
  <div class="dashboard">
    <header class="dashboard-header">
      <h1>📊 대시보드</h1>
      <div class="dashboard-controls">
        <select v-model="selectedPeriod">
          <option value="today">오늘</option>
          <option value="week">이번 주</option>
          <option value="month">이번 달</option>
        </select>
        <button @click="refreshData">🔄 새로고침</button>
      </div>
    </header>
    
    <div class="dashboard-grid">
      <!-- 조건부 위젯 렌더링 -->
      <template v-for="widget in enabledWidgets" :key="widget.id">
        <div 
          v-if="shouldShowWidget(widget)"
          :class="['widget', `widget-${widget.size}`]"
        >
          <!-- 동적 컴포넌트 -->
          <component 
            :is="widget.component"
            :data="getWidgetData(widget)"
            :period="selectedPeriod"
            :loading="widget.loading"
            @refresh="refreshWidget(widget.id)"
          />
        </div>
      </template>
    </div>
    
    <!-- 빈 상태 -->
    <div v-if="enabledWidgets.length === 0" class="empty-dashboard">
      <h2>📋 위젯을 추가해보세요</h2>
      <p>대시보드를 사용자화하려면 위젯을 추가하세요.</p>
      <button @click="showWidgetSelector = true">위젯 추가</button>
    </div>
    
    <!-- 위젯 선택기 -->
    <WidgetSelector 
      v-if="showWidgetSelector"
      :available-widgets="availableWidgets"
      @select="addWidget"
      @close="showWidgetSelector = false"
    />
  </div>
</template>

<script>
import StatsWidget from './widgets/StatsWidget.vue'
import ChartWidget from './widgets/ChartWidget.vue'
import ListWidget from './widgets/ListWidget.vue'
import CalendarWidget from './widgets/CalendarWidget.vue'
import WidgetSelector from './WidgetSelector.vue'

export default {
  components: {
    StatsWidget,
    ChartWidget,
    ListWidget,
    CalendarWidget,
    WidgetSelector
  },
  
  data() {
    return {
      selectedPeriod: 'today',
      showWidgetSelector: false,
      widgets: [
        {
          id: 'stats-1',
          component: 'StatsWidget',
          title: '핵심 지표',
          size: 'small',
          enabled: true,
          loading: false,
          permissions: ['view_stats']
        },
        {
          id: 'chart-1',
          component: 'ChartWidget',
          title: '매출 차트',
          size: 'large',
          enabled: true,
          loading: false,
          permissions: ['view_analytics']
        },
        {
          id: 'list-1',
          component: 'ListWidget',
          title: '최근 주문',
          size: 'medium',
          enabled: true,
          loading: false,
          permissions: ['view_orders']
        },
        {
          id: 'calendar-1',
          component: 'CalendarWidget',
          title: '일정',
          size: 'medium',
          enabled: false,
          loading: false,
          permissions: ['view_calendar']
        }
      ],
      widgetData: {
        'stats-1': { views: 1234, users: 567, revenue: 89012 },
        'chart-1': [
          { date: '2024-01-01', value: 100 },
          { date: '2024-01-02', value: 120 },
          { date: '2024-01-03', value: 90 }
        ],
        'list-1': [
          { id: 1, customer: '김고객', amount: 50000, date: '2024-01-03' },
          { id: 2, customer: '이구매', amount: 30000, date: '2024-01-03' }
        ],
        'calendar-1': [
          { date: '2024-01-05', title: '회의', type: 'meeting' },
          { date: '2024-01-07', title: '데드라인', type: 'deadline' }
        ]
      },
      userPermissions: ['view_stats', 'view_analytics', 'view_orders']
    }
  },
  
  computed: {
    enabledWidgets() {
      return this.widgets.filter(widget => widget.enabled);
    },
    
    availableWidgets() {
      return this.widgets.filter(widget => !widget.enabled);
    }
  },
  
  methods: {
    shouldShowWidget(widget) {
      // 권한 체크
      return widget.permissions.every(permission => 
        this.userPermissions.includes(permission)
      );
    },
    
    getWidgetData(widget) {
      return this.widgetData[widget.id] || [];
    },
    
    refreshWidget(widgetId) {
      const widget = this.widgets.find(w => w.id === widgetId);
      if (widget) {
        widget.loading = true;
        
        // API 호출 시뮬레이션
        setTimeout(() => {
          widget.loading = false;
          // 데이터 업데이트 로직
        }, 1000);
      }
    },
    
    refreshData() {
      this.widgets.forEach(widget => {
        if (widget.enabled) {
          this.refreshWidget(widget.id);
        }
      });
    },
    
    addWidget(widgetId) {
      const widget = this.widgets.find(w => w.id === widgetId);
      if (widget) {
        widget.enabled = true;
      }
      this.showWidgetSelector = false;
    }
  }
}
</script>

<style scoped>
.dashboard {
  padding: 20px;
  max-width: 1400px;
  margin: 0 auto;
}

.dashboard-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
}

.dashboard-controls {
  display: flex;
  gap: 10px;
}

.dashboard-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
  grid-auto-rows: 200px;
}

.widget {
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.1);
  padding: 20px;
  transition: transform 0.2s ease;
}

.widget:hover {
  transform: translateY(-2px);
}

.widget-small {
  grid-column: span 1;
  grid-row: span 1;
}

.widget-medium {
  grid-column: span 1;
  grid-row: span 2;
}

.widget-large {
  grid-column: span 2;
  grid-row: span 2;
}

.empty-dashboard {
  text-align: center;
  padding: 80px 20px;
  color: #666;
}

@media (max-width: 768px) {
  .dashboard-grid {
    grid-template-columns: 1fr;
  }
  
  .widget-large {
    grid-column: span 1;
  }
}
</style>
```

## 🎯 실습 과제

1. 위의 대시보드 시스템을 직접 만들어보세요
2. 위젯 드래그 앤 드롭 재정렬 기능을 추가해보세요
3. 무한 스크롤 기능을 구현해보세요
4. 실시간 데이터 업데이트 기능을 추가해보세요
5. 위젯 설정 커스터마이징 기능을 추가해보세요

## 🔗 다음 단계

다음 챕터에서는 Vue의 폼 처리와 유효성 검사에 대해 알아보겠습니다!

---

💡 **팁**: Vue의 조건부 렌더링과 리스트 처리는 React보다 더 직관적이고 성능이 뛰어나며, 특히 v-show와 v-if의 차이를 잘 활용하면 최적화에 큰 도움이 됩니다! 