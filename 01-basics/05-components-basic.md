# 1-5. 컴포넌트 기초

Vue 컴포넌트는 React와 매우 유사하지만 **더 직관적인 문법**과 **강력한 기능**들을 제공합니다! 특히 **Slots**는 React의 `children`보다 훨씬 유연하고, **이벤트 시스템**도 더 명확해요.

## 🎯 컴포넌트 정의와 사용

### React vs Vue 컴포넌트 기본

**React 컴포넌트**
```jsx
// UserCard.jsx
import React from 'react';

function UserCard({ user, onEdit, onDelete, className, children }) {
  return (
    <div className={`user-card ${className || ''}`}>
      <div className="user-info">
        <img src={user.avatar} alt={user.name} />
        <h3>{user.name}</h3>
        <p>{user.email}</p>
        <span className={`status ${user.isOnline ? 'online' : 'offline'}`}>
          {user.isOnline ? '온라인' : '오프라인'}
        </span>
      </div>
      
      <div className="user-actions">
        <button onClick={() => onEdit(user.id)}>편집</button>
        <button onClick={() => onDelete(user.id)}>삭제</button>
      </div>
      
      {/* children 표시 */}
      {children && (
        <div className="user-extra">
          {children}
        </div>
      )}
    </div>
  );
}

// 사용법
function App() {
  const users = [
    { id: 1, name: '김개발', email: 'kim@dev.com', avatar: '/avatar1.jpg', isOnline: true },
    { id: 2, name: '박프론트', email: 'park@front.com', avatar: '/avatar2.jpg', isOnline: false }
  ];

  const handleEdit = (id) => console.log('Edit user:', id);
  const handleDelete = (id) => console.log('Delete user:', id);

  return (
    <div>
      {users.map(user => (
        <UserCard
          key={user.id}
          user={user}
          onEdit={handleEdit}
          onDelete={handleDelete}
          className="my-user-card"
        >
          <p>추가 정보가 들어갑니다.</p>
          <button>상세보기</button>
        </UserCard>
      ))}
    </div>
  );
}

export default UserCard;
```

**Vue 컴포넌트**
```vue
<!-- UserCard.vue -->
<template>
  <div :class="['user-card', className]">
    <div class="user-info">
      <img :src="user.avatar" :alt="user.name" />
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
      <span :class="['status', user.isOnline ? 'online' : 'offline']">
        {{ user.isOnline ? '온라인' : '오프라인' }}
      </span>
    </div>
    
    <div class="user-actions">
      <button @click="$emit('edit', user.id)">편집</button>
      <button @click="$emit('delete', user.id)">삭제</button>
    </div>
    
    <!-- slot으로 컨텐츠 투영 -->
    <div v-if="$slots.default" class="user-extra">
      <slot></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UserCard',
  props: {
    user: {
      type: Object,
      required: true
    },
    className: {
      type: String,
      default: ''
    }
  },
  emits: ['edit', 'delete']
}
</script>

<style scoped>
.user-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 20px;
  margin: 10px 0;
  background: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.user-info {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-bottom: 15px;
}

.user-info img {
  width: 50px;
  height: 50px;
  border-radius: 50%;
}

.status.online {
  color: #27ae60;
}

.status.offline {
  color: #e74c3c;
}

.user-actions {
  display: flex;
  gap: 10px;
}

.user-actions button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.user-actions button:first-child {
  background: #3498db;
  color: white;
}

.user-actions button:last-child {
  background: #e74c3c;
  color: white;
}

.user-extra {
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px solid #eee;
}
</style>
```

```vue
<!-- App.vue -->
<template>
  <div>
    <UserCard
      v-for="user in users"
      :key="user.id"
      :user="user"
      @edit="handleEdit"
      @delete="handleDelete"
      className="my-user-card"
    >
      <p>추가 정보가 들어갑니다.</p>
      <button>상세보기</button>
    </UserCard>
  </div>
</template>

<script>
import UserCard from './components/UserCard.vue'

export default {
  components: {
    UserCard
  },
  data() {
    return {
      users: [
        { id: 1, name: '김개발', email: 'kim@dev.com', avatar: '/avatar1.jpg', isOnline: true },
        { id: 2, name: '박프론트', email: 'park@front.com', avatar: '/avatar2.jpg', isOnline: false }
      ]
    }
  },
  methods: {
    handleEdit(id) {
      console.log('Edit user:', id);
    },
    handleDelete(id) {
      console.log('Delete user:', id);
    }
  }
}
</script>
```

## 📨 Props (속성 전달)

### Props 유효성 검사와 기본값

**React의 PropTypes와 defaultProps**
```jsx
import PropTypes from 'prop-types';

function ProductCard({ 
  product, 
  showPrice = true, 
  discount = 0, 
  onAddToCart, 
  onWishlist,
  variant = 'default'
}) {
  return (
    <div className={`product-card product-card--${variant}`}>
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>{product.description}</p>
      
      {showPrice && (
        <div className="price">
          {discount > 0 ? (
            <>
              <span className="original-price">${product.price}</span>
              <span className="discounted-price">
                ${(product.price * (1 - discount)).toFixed(2)}
              </span>
            </>
          ) : (
            <span className="price">${product.price}</span>
          )}
        </div>
      )}
      
      <div className="actions">
        <button onClick={() => onAddToCart(product)}>
          장바구니 추가
        </button>
        <button onClick={() => onWishlist(product)}>
          위시리스트
        </button>
      </div>
    </div>
  );
}

ProductCard.propTypes = {
  product: PropTypes.shape({
    id: PropTypes.oneOfType([PropTypes.string, PropTypes.number]).isRequired,
    name: PropTypes.string.isRequired,
    description: PropTypes.string,
    price: PropTypes.number.isRequired,
    image: PropTypes.string.isRequired
  }).isRequired,
  showPrice: PropTypes.bool,
  discount: PropTypes.number,
  onAddToCart: PropTypes.func.isRequired,
  onWishlist: PropTypes.func.isRequired,
  variant: PropTypes.oneOf(['default', 'compact', 'featured'])
};

ProductCard.defaultProps = {
  showPrice: true,
  discount: 0,
  variant: 'default'
};
```

**Vue의 Props 정의**
```vue
<template>
  <div :class="['product-card', `product-card--${variant}`]">
    <img :src="product.image" :alt="product.name" />
    <h3>{{ product.name }}</h3>
    <p>{{ product.description }}</p>
    
    <div v-if="showPrice" class="price">
      <template v-if="discount > 0">
        <span class="original-price">${{ product.price }}</span>
        <span class="discounted-price">
          ${{ (product.price * (1 - discount)).toFixed(2) }}
        </span>
      </template>
      <span v-else class="price">${{ product.price }}</span>
    </div>
    
    <div class="actions">
      <button @click="$emit('add-to-cart', product)">
        장바구니 추가
      </button>
      <button @click="$emit('wishlist', product)">
        위시리스트
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ProductCard',
  props: {
    // 객체 타입과 필수 여부
    product: {
      type: Object,
      required: true,
      validator(value) {
        // 커스텀 유효성 검사
        return value && 
               typeof value.id !== 'undefined' && 
               typeof value.name === 'string' && 
               typeof value.price === 'number' &&
               typeof value.image === 'string';
      }
    },
    
    // 불린 타입과 기본값
    showPrice: {
      type: Boolean,
      default: true
    },
    
    // 숫자 타입과 기본값, 유효성 검사
    discount: {
      type: Number,
      default: 0,
      validator(value) {
        return value >= 0 && value <= 1;
      }
    },
    
    // 문자열 타입과 여러 가능한 값
    variant: {
      type: String,
      default: 'default',
      validator(value) {
        return ['default', 'compact', 'featured'].includes(value);
      }
    }
  },
  
  emits: {
    // 이벤트 유효성 검사
    'add-to-cart': (product) => {
      return product && typeof product.id !== 'undefined';
    },
    'wishlist': (product) => {
      return product && typeof product.id !== 'undefined';
    }
  }
}
</script>
```

### Prop 타입의 다양한 예시

```vue
<template>
  <div class="advanced-component">
    <!-- props 사용 예시 -->
    <h2>{{ title }}</h2>
    <p>나이: {{ age }}</p>
    <p>활성 상태: {{ isActive ? 'Y' : 'N' }}</p>
    <p>색상: {{ colors.join(', ') }}</p>
    <p>사용자: {{ user.name }} ({{ user.email }})</p>
    <p>콜백: {{ callback ? '제공됨' : '미제공' }}</p>
  </div>
</template>

<script>
export default {
  props: {
    // 기본 타입들
    title: String,
    age: Number,
    isActive: Boolean,
    colors: Array,
    user: Object,
    callback: Function,
    
    // 여러 타입 허용
    id: [String, Number],
    
    // 필수값과 기본값
    message: {
      type: String,
      required: true
    },
    
    status: {
      type: String,
      default: 'pending'
    },
    
    // 객체나 배열의 기본값은 팩토리 함수로
    tags: {
      type: Array,
      default() {
        return [];
      }
    },
    
    config: {
      type: Object,
      default() {
        return {
          theme: 'light',
          language: 'ko'
        };
      }
    },
    
    // 커스텀 유효성 검사
    email: {
      type: String,
      validator(value) {
        return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
      }
    },
    
    // 여러 조건이 있는 복잡한 유효성 검사
    priority: {
      type: String,
      default: 'normal',
      validator(value) {
        return ['low', 'normal', 'high', 'urgent'].includes(value);
      }
    }
  }
}
</script>
```

## 🔄 이벤트 (Events)

### 커스텀 이벤트 발생과 처리

**React의 콜백 함수**
```jsx
function SearchBox({ onSearch, onFilter, onClear }) {
  const [query, setQuery] = useState('');
  const [filters, setFilters] = useState({
    category: '',
    priceRange: [0, 1000],
    inStock: false
  });

  const handleSearch = () => {
    onSearch(query, filters);
  };

  const handleFilterChange = (key, value) => {
    const newFilters = { ...filters, [key]: value };
    setFilters(newFilters);
    onFilter(newFilters);
  };

  const handleClear = () => {
    setQuery('');
    setFilters({ category: '', priceRange: [0, 1000], inStock: false });
    onClear();
  };

  return (
    <div className="search-box">
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="검색어 입력..."
      />
      
      <select
        value={filters.category}
        onChange={(e) => handleFilterChange('category', e.target.value)}
      >
        <option value="">모든 카테고리</option>
        <option value="electronics">전자제품</option>
        <option value="clothing">의류</option>
      </select>
      
      <label>
        <input
          type="checkbox"
          checked={filters.inStock}
          onChange={(e) => handleFilterChange('inStock', e.target.checked)}
        />
        재고 있음만
      </label>
      
      <button onClick={handleSearch}>검색</button>
      <button onClick={handleClear}>초기화</button>
    </div>
  );
}
```

**Vue의 $emit**
```vue
<template>
  <div class="search-box">
    <input
      v-model="query"
      @keyup.enter="handleSearch"
      placeholder="검색어 입력..."
    />
    
    <select 
      v-model="filters.category"
      @change="handleFilterChange"
    >
      <option value="">모든 카테고리</option>
      <option value="electronics">전자제품</option>
      <option value="clothing">의류</option>
    </select>
    
    <label>
      <input
        type="checkbox"
        v-model="filters.inStock"
        @change="handleFilterChange"
      />
      재고 있음만
    </label>
    
    <button @click="handleSearch">검색</button>
    <button @click="handleClear">초기화</button>
  </div>
</template>

<script>
export default {
  name: 'SearchBox',
  data() {
    return {
      query: '',
      filters: {
        category: '',
        priceRange: [0, 1000],
        inStock: false
      }
    }
  },
  
  emits: {
    // 이벤트 정의와 유효성 검사
    search: (query, filters) => {
      return typeof query === 'string' && typeof filters === 'object';
    },
    filter: (filters) => {
      return typeof filters === 'object';
    },
    clear: null // 매개변수 없는 이벤트
  },
  
  methods: {
    handleSearch() {
      // 'search' 이벤트 발생
      this.$emit('search', this.query, this.filters);
    },
    
    handleFilterChange() {
      // 'filter' 이벤트 발생 (실시간 필터링)
      this.$emit('filter', this.filters);
    },
    
    handleClear() {
      this.query = '';
      this.filters = {
        category: '',
        priceRange: [0, 1000],
        inStock: false
      };
      
      // 'clear' 이벤트 발생
      this.$emit('clear');
    }
  }
}
</script>
```

```vue
<!-- 부모 컴포넌트에서 사용 -->
<template>
  <div>
    <SearchBox
      @search="handleSearch"
      @filter="handleFilter"
      @clear="handleClear"
    />
    
    <div class="results">
      <!-- 검색 결과 표시 -->
    </div>
  </div>
</template>

<script>
export default {
  methods: {
    handleSearch(query, filters) {
      console.log('검색:', query, filters);
      // 검색 로직
    },
    
    handleFilter(filters) {
      console.log('필터 변경:', filters);
      // 실시간 필터링 로직
    },
    
    handleClear() {
      console.log('검색 초기화');
      // 초기화 로직
    }
  }
}
</script>
```

## 🎯 Slots (컨텐츠 투영)

Vue의 Slots는 React의 `children`보다 훨씬 강력합니다!

### 기본 Slot vs React Children

**React Children**
```jsx
function Card({ children, title, footer }) {
  return (
    <div className="card">
      {title && (
        <div className="card-header">
          <h3>{title}</h3>
        </div>
      )}
      
      <div className="card-body">
        {children}
      </div>
      
      {footer && (
        <div className="card-footer">
          {footer}
        </div>
      )}
    </div>
  );
}

// 사용법 - 복잡한 구조 전달이 어려움
function App() {
  return (
    <Card 
      title="사용자 정보"
      footer={<button>저장</button>}
    >
      <p>카드 내용이 들어갑니다.</p>
      <button>수정</button>
    </Card>
  );
}
```

**Vue Slots**
```vue
<!-- Card.vue -->
<template>
  <div class="card">
    <div class="card-header">
      <!-- 이름있는 슬롯 -->
      <slot name="header">
        <h3>{{ title }}</h3>
      </slot>
    </div>

    <div class="card-body">
      <!-- 기본 슬롯 -->
      <slot>
        <p>기본 내용이 없습니다.</p>
      </slot>
    </div>

    <div v-if="$slots.footer" class="card-footer">
      <!-- 조건부 이름있는 슬롯 -->
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Card',
  props: {
    title: String
  }
}
</script>
```

```vue
<!-- 사용법 - 훨씬 유연함 -->
<template>
  <div>
    <!-- 기본 사용법 -->
    <Card title="사용자 정보">
      <p>카드 내용이 들어갑니다.</p>
      <button>수정</button>
      
      <template #footer>
        <button>저장</button>
        <button>취소</button>
      </template>
    </Card>

    <!-- 더 복잡한 구조 -->
    <Card>
      <template #header>
        <div class="custom-header">
          <h2>커스텀 헤더</h2>
          <span class="badge">NEW</span>
        </div>
      </template>

      <div class="complex-content">
        <img src="/image.jpg" alt="이미지" />
        <div class="text-content">
          <h4>제목</h4>
          <p>설명</p>
        </div>
      </div>

      <template #footer>
        <div class="footer-actions">
          <button class="primary">주요 버튼</button>
          <button class="secondary">보조 버튼</button>
        </div>
      </template>
    </Card>
  </div>
</template>
```

### 스코프드 슬롯 (Scoped Slots)

React에서 render props 패턴과 유사하지만 더 직관적입니다!

```vue
<!-- DataTable.vue -->
<template>
  <div class="data-table">
    <div class="table-header">
      <slot name="header" :total="filteredData.length" :loading="loading">
        <h3>데이터 테이블 ({{ filteredData.length }}개)</h3>
      </slot>
    </div>

    <div v-if="loading" class="loading">
      로딩 중...
    </div>

    <table v-else>
      <thead>
        <slot name="columns" :columns="columns">
          <tr>
            <th v-for="column in columns" :key="column.key">
              {{ column.label }}
            </th>
          </tr>
        </slot>
      </thead>
      
      <tbody>
        <!-- 각 행에 대해 스코프드 슬롯 -->
        <tr v-for="(item, index) in filteredData" :key="item.id || index">
          <slot 
            name="row" 
            :item="item" 
            :index="index" 
            :isEven="index % 2 === 0"
            :onEdit="() => editItem(item)"
            :onDelete="() => deleteItem(item)"
          >
            <!-- 기본 행 렌더링 -->
            <td v-for="column in columns" :key="column.key">
              {{ item[column.key] }}
            </td>
          </slot>
        </tr>
      </tbody>
    </table>

    <div v-if="!loading && filteredData.length === 0" class="empty">
      <slot name="empty">
        <p>데이터가 없습니다.</p>
      </slot>
    </div>

    <div class="table-footer">
      <slot name="footer" :total="filteredData.length" :page="currentPage">
        <p>총 {{ filteredData.length }}개 항목</p>
      </slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DataTable',
  props: {
    data: {
      type: Array,
      default: () => []
    },
    columns: {
      type: Array,
      default: () => []
    },
    loading: {
      type: Boolean,
      default: false
    }
  },
  
  data() {
    return {
      currentPage: 1
    }
  },
  
  computed: {
    filteredData() {
      return this.data.filter(item => item !== null);
    }
  },
  
  methods: {
    editItem(item) {
      this.$emit('edit', item);
    },
    
    deleteItem(item) {
      this.$emit('delete', item);
    }
  }
}
</script>
```

```vue
<!-- 스코프드 슬롯 사용법 -->
<template>
  <div>
    <DataTable 
      :data="users" 
      :columns="columns"
      :loading="loading"
      @edit="handleEdit"
      @delete="handleDelete"
    >
      <!-- 커스텀 헤더 (스코프드 슬롯) -->
      <template #header="{ total, loading }">
        <div class="custom-header">
          <h2>사용자 목록</h2>
          <div class="stats">
            <span v-if="!loading">총 {{ total }}명</span>
            <button @click="addUser">사용자 추가</button>
          </div>
        </div>
      </template>

      <!-- 커스텀 행 렌더링 -->
      <template #row="{ item, index, isEven, onEdit, onDelete }">
        <td :class="{ 'even-row': isEven }">{{ item.id }}</td>
        <td>
          <div class="user-info">
            <img :src="item.avatar" :alt="item.name" class="avatar" />
            <span>{{ item.name }}</span>
          </div>
        </td>
        <td>{{ item.email }}</td>
        <td>
          <span :class="['status', item.isActive ? 'active' : 'inactive']">
            {{ item.isActive ? '활성' : '비활성' }}
          </span>
        </td>
        <td>
          <div class="actions">
            <button @click="onEdit" class="edit-btn">편집</button>
            <button @click="onDelete" class="delete-btn">삭제</button>
            <button @click="viewProfile(item)" class="view-btn">프로필</button>
          </div>
        </td>
      </template>

      <!-- 빈 상태 커스터마이징 -->
      <template #empty>
        <div class="empty-state">
          <img src="/empty-users.svg" alt="사용자 없음" />
          <h3>아직 사용자가 없습니다</h3>
          <p>첫 번째 사용자를 추가해보세요!</p>
          <button @click="addUser" class="primary">사용자 추가</button>
        </div>
      </template>

      <!-- 커스텀 푸터 -->
      <template #footer="{ total, page }">
        <div class="custom-footer">
          <div class="pagination">
            <button :disabled="page === 1" @click="prevPage">이전</button>
            <span>페이지 {{ page }}</span>
            <button @click="nextPage">다음</button>
          </div>
          <div class="summary">
            총 {{ total }}명의 사용자
          </div>
        </div>
      </template>
    </DataTable>
  </div>
</template>

<script>
export default {
  data() {
    return {
      users: [
        { 
          id: 1, 
          name: '김개발', 
          email: 'kim@dev.com', 
          avatar: '/avatar1.jpg',
          isActive: true 
        },
        { 
          id: 2, 
          name: '박프론트', 
          email: 'park@front.com', 
          avatar: '/avatar2.jpg',
          isActive: false 
        }
      ],
      columns: [
        { key: 'id', label: 'ID' },
        { key: 'name', label: '이름' },
        { key: 'email', label: '이메일' },
        { key: 'isActive', label: '상태' },
        { key: 'actions', label: '작업' }
      ],
      loading: false
    }
  },
  
  methods: {
    handleEdit(user) {
      console.log('편집:', user);
    },
    
    handleDelete(user) {
      console.log('삭제:', user);
    },
    
    addUser() {
      console.log('사용자 추가');
    },
    
    viewProfile(user) {
      console.log('프로필 보기:', user);
    },
    
    prevPage() {
      console.log('이전 페이지');
    },
    
    nextPage() {
      console.log('다음 페이지');
    }
  }
}
</script>
```

## 🎯 실습: 게시판 시스템

Vue 컴포넌트의 모든 기능을 활용한 완전한 게시판을 만들어보세요!

```vue
<!-- App.vue -->
<template>
  <div class="bulletin-board">
    <header class="app-header">
      <h1>🏢 Vue 게시판</h1>
      <div class="header-stats">
        <span>총 {{ posts.length }}개 게시글</span>
        <span>|</span>
        <span>오늘 {{ todayPosts }}개 작성</span>
      </div>
    </header>

    <main class="main-content">
      <!-- 게시글 작성 폼 -->
      <Card class="post-form-card">
        <template #header>
          <h2>📝 새 게시글 작성</h2>
        </template>

        <PostForm @submit="handlePostSubmit" />
      </Card>

      <!-- 게시글 목록 -->
      <Card class="post-list-card">
        <template #header>
          <div class="list-header">
            <h2>📋 게시글 목록</h2>
            <div class="list-controls">
              <select v-model="sortBy" @change="sortPosts">
                <option value="newest">최신순</option>
                <option value="oldest">오래된순</option>
                <option value="popular">인기순</option>
              </select>
              <input 
                v-model="searchQuery" 
                placeholder="검색..."
                class="search-input"
              />
            </div>
          </div>
        </template>

        <PostList 
          :posts="filteredPosts"
          @like="handlePostLike"
          @delete="handlePostDelete"
          @comment="handleAddComment"
        >
          <!-- 빈 상태 커스터마이징 -->
          <template #empty>
            <div class="empty-board">
              <h3>📭 아직 게시글이 없습니다</h3>
              <p>첫 번째 게시글을 작성해보세요!</p>
            </div>
          </template>
        </PostList>
      </Card>
    </main>

    <!-- 사이드바 -->
    <aside class="sidebar">
      <PostStats :posts="posts" />
    </aside>
  </div>
</template>

<script>
import Card from './components/Card.vue'
import PostForm from './components/PostForm.vue'
import PostList from './components/PostList.vue'
import PostStats from './components/PostStats.vue'

export default {
  name: 'BulletinBoard',
  components: {
    Card,
    PostForm,
    PostList,
    PostStats
  },
  
  data() {
    return {
      posts: [
        {
          id: 1,
          title: 'Vue.js 학습 후기',
          content: 'Vue.js를 배우면서 느낀 점들을 공유합니다.',
          author: '김개발',
          createdAt: new Date('2024-01-15'),
          likes: 5,
          comments: [
            { id: 1, content: '좋은 글이네요!', author: '박프론트' }
          ]
        },
        {
          id: 2,
          title: 'React vs Vue 비교',
          content: '두 프레임워크의 차이점을 분석해봅시다.',
          author: '이풀스택',
          createdAt: new Date('2024-01-14'),
          likes: 8,
          comments: []
        }
      ],
      sortBy: 'newest',
      searchQuery: ''
    }
  },
  
  computed: {
    todayPosts() {
      const today = new Date().toDateString();
      return this.posts.filter(post => 
        post.createdAt.toDateString() === today
      ).length;
    },
    
    filteredPosts() {
      let filtered = this.posts;
      
      // 검색 필터링
      if (this.searchQuery) {
        filtered = filtered.filter(post =>
          post.title.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
          post.content.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
          post.author.toLowerCase().includes(this.searchQuery.toLowerCase())
        );
      }
      
      // 정렬
      return filtered.sort((a, b) => {
        switch (this.sortBy) {
          case 'oldest':
            return a.createdAt - b.createdAt;
          case 'popular':
            return b.likes - a.likes;
          case 'newest':
          default:
            return b.createdAt - a.createdAt;
        }
      });
    }
  },
  
  methods: {
    handlePostSubmit(postData) {
      const newPost = {
        id: Date.now(),
        ...postData,
        createdAt: new Date(),
        likes: 0,
        comments: []
      };
      
      this.posts.unshift(newPost);
      console.log('새 게시글 작성:', newPost);
    },
    
    handlePostLike(postId) {
      const post = this.posts.find(p => p.id === postId);
      if (post) {
        post.likes++;
        console.log('게시글 좋아요:', postId);
      }
    },
    
    handlePostDelete(postId) {
      const index = this.posts.findIndex(p => p.id === postId);
      if (index > -1) {
        this.posts.splice(index, 1);
        console.log('게시글 삭제:', postId);
      }
    },
    
    handleAddComment(data) {
      const post = this.posts.find(p => p.id === data.postId);
      if (post) {
        const newComment = {
          id: Date.now(),
          content: data.content,
          author: data.author,
          createdAt: new Date()
        };
        post.comments.push(newComment);
        console.log('댓글 추가:', newComment);
      }
    },
    
    sortPosts() {
      // computed에서 자동으로 정렬됨
      console.log('정렬 방식 변경:', this.sortBy);
    }
  }
}
</script>

<style scoped>
.bulletin-board {
  display: grid;
  grid-template-columns: 1fr 300px;
  grid-template-rows: auto 1fr;
  grid-template-areas: 
    "header header"
    "main sidebar";
  gap: 20px;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.app-header {
  grid-area: header;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border-radius: 12px;
}

.header-stats {
  display: flex;
  gap: 10px;
  font-size: 14px;
}

.main-content {
  grid-area: main;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.sidebar {
  grid-area: sidebar;
}

.post-form-card,
.post-list-card {
  width: 100%;
}

.list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 15px;
}

.list-controls {
  display: flex;
  gap: 10px;
  align-items: center;
}

.search-input {
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
}

.list-controls select {
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
}

.empty-board {
  text-align: center;
  padding: 40px;
  color: #666;
}

@media (max-width: 768px) {
  .bulletin-board {
    grid-template-columns: 1fr;
    grid-template-areas: 
      "header"
      "main"
      "sidebar";
  }
  
  .list-header {
    flex-direction: column;
    align-items: stretch;
  }
  
  .list-controls {
    justify-content: space-between;
  }
}
</style>
```

```vue
<!-- components/PostForm.vue -->
<template>
  <form @submit.prevent="handleSubmit" class="post-form">
    <div class="form-group">
      <label for="title">제목:</label>
      <input
        id="title"
        v-model="form.title"
        type="text"
        required
        :class="{ error: errors.title }"
        @blur="validateField('title')"
        placeholder="게시글 제목을 입력하세요"
      />
      <span v-if="errors.title" class="error-message">{{ errors.title }}</span>
    </div>

    <div class="form-group">
      <label for="author">작성자:</label>
      <input
        id="author"
        v-model="form.author"
        type="text"
        required
        :class="{ error: errors.author }"
        @blur="validateField('author')"
        placeholder="작성자명을 입력하세요"
      />
      <span v-if="errors.author" class="error-message">{{ errors.author }}</span>
    </div>

    <div class="form-group">
      <label for="content">내용:</label>
      <textarea
        id="content"
        v-model="form.content"
        required
        :class="{ error: errors.content }"
        @blur="validateField('content')"
        placeholder="게시글 내용을 입력하세요"
        rows="6"
      ></textarea>
      <span v-if="errors.content" class="error-message">{{ errors.content }}</span>
    </div>

    <div class="form-actions">
      <button type="button" @click="resetForm" class="reset-btn">
        초기화
      </button>
      <button type="submit" :disabled="!isFormValid" class="submit-btn">
        게시글 작성
      </button>
    </div>
  </form>
</template>

<script>
export default {
  name: 'PostForm',
  
  data() {
    return {
      form: {
        title: '',
        author: '',
        content: ''
      },
      errors: {}
    }
  },
  
  computed: {
    isFormValid() {
      return this.form.title.trim() && 
             this.form.author.trim() && 
             this.form.content.trim() &&
             Object.keys(this.errors).length === 0;
    }
  },
  
  emits: ['submit'],
  
  methods: {
    validateField(field) {
      this.$set(this.errors, field, '');
      
      switch (field) {
        case 'title':
          if (!this.form.title.trim()) {
            this.$set(this.errors, 'title', '제목을 입력해주세요.');
          } else if (this.form.title.length < 5) {
            this.$set(this.errors, 'title', '제목은 5자 이상 입력해주세요.');
          } else if (this.form.title.length > 100) {
            this.$set(this.errors, 'title', '제목은 100자 이하로 입력해주세요.');
          }
          break;
          
        case 'author':
          if (!this.form.author.trim()) {
            this.$set(this.errors, 'author', '작성자명을 입력해주세요.');
          } else if (this.form.author.length < 2) {
            this.$set(this.errors, 'author', '작성자명은 2자 이상 입력해주세요.');
          }
          break;
          
        case 'content':
          if (!this.form.content.trim()) {
            this.$set(this.errors, 'content', '내용을 입력해주세요.');
          } else if (this.form.content.length < 10) {
            this.$set(this.errors, 'content', '내용은 10자 이상 입력해주세요.');
          }
          break;
      }
    },
    
    handleSubmit() {
      // 모든 필드 유효성 검사
      Object.keys(this.form).forEach(field => {
        this.validateField(field);
      });
      
      if (this.isFormValid) {
        this.$emit('submit', { ...this.form });
        this.resetForm();
      }
    },
    
    resetForm() {
      this.form = {
        title: '',
        author: '',
        content: ''
      };
      this.errors = {};
    }
  }
}
</script>

<style scoped>
.post-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-group label {
  font-weight: bold;
  color: #2c3e50;
}

.form-group input,
.form-group textarea {
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
  transition: border-color 0.3s ease;
}

.form-group input:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #42b883;
  box-shadow: 0 0 0 3px rgba(66, 184, 131, 0.1);
}

.form-group input.error,
.form-group textarea.error {
  border-color: #e74c3c;
}

.error-message {
  color: #e74c3c;
  font-size: 12px;
  margin-top: 4px;
}

.form-actions {
  display: flex;
  gap: 12px;
  justify-content: flex-end;
}

.reset-btn,
.submit-btn {
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
}

.reset-btn {
  background: #95a5a6;
  color: white;
}

.reset-btn:hover {
  background: #7f8c8d;
}

.submit-btn {
  background: #42b883;
  color: white;
}

.submit-btn:hover:not(:disabled) {
  background: #369870;
}

.submit-btn:disabled {
  background: #bdc3c7;
  cursor: not-allowed;
}
</style>
```

```vue
<!-- components/PostList.vue -->
<template>
  <div class="post-list">
    <div v-if="posts.length === 0" class="empty-state">
      <slot name="empty">
        <p>게시글이 없습니다.</p>
      </slot>
    </div>
    
    <div v-else class="posts">
      <PostItem
        v-for="post in posts"
        :key="post.id"
        :post="post"
        @like="$emit('like', $event)"
        @delete="$emit('delete', $event)"
        @comment="$emit('comment', $event)"
      />
    </div>
  </div>
</template>

<script>
import PostItem from './PostItem.vue'

export default {
  name: 'PostList',
  components: {
    PostItem
  },
  
  props: {
    posts: {
      type: Array,
      default: () => []
    }
  },
  
  emits: ['like', 'delete', 'comment']
}
</script>

<style scoped>
.post-list {
  min-height: 200px;
}

.posts {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.empty-state {
  text-align: center;
  padding: 40px;
  color: #666;
}
</style>
```

```vue
<!-- components/PostItem.vue -->
<template>
  <div class="post-item">
    <div class="post-header">
      <div class="post-meta">
        <h3 class="post-title">{{ post.title }}</h3>
        <div class="post-info">
          <span class="author">👤 {{ post.author }}</span>
          <span class="date">📅 {{ formattedDate }}</span>
          <span class="stats">
            ❤️ {{ post.likes }} | 💬 {{ post.comments.length }}
          </span>
        </div>
      </div>
      
      <div class="post-actions">
        <button @click="handleLike" class="like-btn" :class="{ liked: isLiked }">
          ❤️ {{ post.likes }}
        </button>
        <button @click="toggleComments" class="comment-btn">
          💬 {{ post.comments.length }}
        </button>
        <button @click="handleDelete" class="delete-btn">
          🗑️
        </button>
      </div>
    </div>

    <div class="post-content">
      <p>{{ post.content }}</p>
    </div>

    <!-- 댓글 섹션 -->
    <div v-if="showComments" class="comments-section">
      <div class="comments-header">
        <h4>댓글 ({{ post.comments.length }}개)</h4>
      </div>

      <div v-if="post.comments.length > 0" class="comments-list">
        <div 
          v-for="comment in post.comments" 
          :key="comment.id"
          class="comment-item"
        >
          <strong>{{ comment.author }}</strong>
          <p>{{ comment.content }}</p>
        </div>
      </div>

      <div class="comment-form">
        <div class="form-row">
          <input
            v-model="newComment.author"
            placeholder="작성자"
            class="comment-author"
          />
          <input
            v-model="newComment.content"
            placeholder="댓글을 입력하세요..."
            class="comment-input"
            @keyup.enter="addComment"
          />
          <button @click="addComment" class="add-comment-btn">
            추가
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PostItem',
  
  props: {
    post: {
      type: Object,
      required: true
    }
  },
  
  data() {
    return {
      showComments: false,
      isLiked: false,
      newComment: {
        author: '',
        content: ''
      }
    }
  },
  
  computed: {
    formattedDate() {
      return this.post.createdAt.toLocaleDateString('ko-KR', {
        year: 'numeric',
        month: 'short',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      });
    }
  },
  
  emits: ['like', 'delete', 'comment'],
  
  methods: {
    handleLike() {
      this.isLiked = !this.isLiked;
      this.$emit('like', this.post.id);
    },
    
    handleDelete() {
      if (confirm('정말로 이 게시글을 삭제하시겠습니까?')) {
        this.$emit('delete', this.post.id);
      }
    },
    
    toggleComments() {
      this.showComments = !this.showComments;
    },
    
    addComment() {
      if (this.newComment.author.trim() && this.newComment.content.trim()) {
        this.$emit('comment', {
          postId: this.post.id,
          author: this.newComment.author,
          content: this.newComment.content
        });
        
        this.newComment = {
          author: '',
          content: ''
        };
      }
    }
  }
}
</script>

<style scoped>
.post-item {
  border: 1px solid #e1e8ed;
  border-radius: 12px;
  padding: 20px;
  background: white;
  transition: all 0.3s ease;
}

.post-item:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  transform: translateY(-1px);
}

.post-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 15px;
}

.post-title {
  margin: 0 0 8px 0;
  color: #2c3e50;
  font-size: 18px;
}

.post-info {
  display: flex;
  gap: 15px;
  font-size: 13px;
  color: #657786;
}

.post-actions {
  display: flex;
  gap: 8px;
}

.post-actions button {
  padding: 6px 12px;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  font-size: 12px;
  transition: all 0.3s ease;
}

.like-btn {
  background: #fff;
  border: 1px solid #e1e8ed;
}

.like-btn:hover,
.like-btn.liked {
  background: #ffebee;
  border-color: #e91e63;
  color: #e91e63;
}

.comment-btn {
  background: #fff;
  border: 1px solid #e1e8ed;
}

.comment-btn:hover {
  background: #e3f2fd;
  border-color: #2196f3;
  color: #2196f3;
}

.delete-btn {
  background: #fff;
  border: 1px solid #e1e8ed;
}

.delete-btn:hover {
  background: #ffebee;
  border-color: #f44336;
  color: #f44336;
}

.post-content {
  line-height: 1.6;
  color: #2c3e50;
  margin-bottom: 15px;
}

.comments-section {
  margin-top: 20px;
  padding-top: 20px;
  border-top: 1px solid #e1e8ed;
}

.comments-header {
  margin-bottom: 15px;
}

.comments-header h4 {
  margin: 0;
  color: #2c3e50;
  font-size: 14px;
}

.comments-list {
  margin-bottom: 15px;
}

.comment-item {
  padding: 10px;
  background: #f8f9fa;
  border-radius: 8px;
  margin-bottom: 8px;
}

.comment-item strong {
  color: #2c3e50;
  font-size: 13px;
}

.comment-item p {
  margin: 4px 0 0 0;
  font-size: 14px;
  line-height: 1.4;
}

.comment-form .form-row {
  display: flex;
  gap: 8px;
  align-items: stretch;
}

.comment-author {
  flex: 0 0 120px;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
}

.comment-input {
  flex: 1;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
}

.add-comment-btn {
  padding: 8px 16px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
}

.add-comment-btn:hover {
  background: #369870;
}
</style>
```

```vue
<!-- components/PostStats.vue -->
<template>
  <Card class="stats-card">
    <template #header>
      <h3>📊 게시판 통계</h3>
    </template>

    <div class="stats-content">
      <div class="stat-item">
        <div class="stat-label">총 게시글</div>
        <div class="stat-value">{{ totalPosts }}</div>
      </div>

      <div class="stat-item">
        <div class="stat-label">총 댓글</div>
        <div class="stat-value">{{ totalComments }}</div>
      </div>

      <div class="stat-item">
        <div class="stat-label">총 좋아요</div>
        <div class="stat-value">{{ totalLikes }}</div>
      </div>

      <div class="stat-item">
        <div class="stat-label">활성 사용자</div>
        <div class="stat-value">{{ activeUsers }}</div>
      </div>
    </div>

    <div class="popular-posts">
      <h4>🔥 인기 게시글</h4>
      <div v-if="popularPosts.length === 0" class="no-popular">
        아직 인기 게시글이 없습니다.
      </div>
      <div v-else class="popular-list">
        <div 
          v-for="post in popularPosts" 
          :key="post.id"
          class="popular-item"
        >
          <div class="popular-title">{{ truncateTitle(post.title) }}</div>
          <div class="popular-stats">❤️ {{ post.likes }}</div>
        </div>
      </div>
    </div>

    <template #footer>
      <div class="stats-footer">
        <small>마지막 업데이트: {{ lastUpdate }}</small>
      </div>
    </template>
  </Card>
</template>

<script>
import Card from './Card.vue'

export default {
  name: 'PostStats',
  components: {
    Card
  },
  
  props: {
    posts: {
      type: Array,
      default: () => []
    }
  },
  
  computed: {
    totalPosts() {
      return this.posts.length;
    },
    
    totalComments() {
      return this.posts.reduce((sum, post) => sum + post.comments.length, 0);
    },
    
    totalLikes() {
      return this.posts.reduce((sum, post) => sum + post.likes, 0);
    },
    
    activeUsers() {
      const authors = new Set(this.posts.map(post => post.author));
      return authors.size;
    },
    
    popularPosts() {
      return this.posts
        .filter(post => post.likes > 0)
        .sort((a, b) => b.likes - a.likes)
        .slice(0, 5);
    },
    
    lastUpdate() {
      return new Date().toLocaleTimeString('ko-KR');
    }
  },
  
  methods: {
    truncateTitle(title) {
      return title.length > 25 ? title.substring(0, 25) + '...' : title;
    }
  }
}
</script>

<style scoped>
.stats-card {
  height: fit-content;
}

.stats-content {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
  margin-bottom: 25px;
}

.stat-item {
  text-align: center;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 8px;
}

.stat-label {
  font-size: 12px;
  color: #666;
  margin-bottom: 5px;
}

.stat-value {
  font-size: 24px;
  font-weight: bold;
  color: #2c3e50;
}

.popular-posts {
  margin: 20px 0;
}

.popular-posts h4 {
  margin: 0 0 15px 0;
  color: #2c3e50;
  font-size: 16px;
}

.no-popular {
  text-align: center;
  color: #666;
  font-size: 14px;
  padding: 20px;
}

.popular-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.popular-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  background: #fff;
  border: 1px solid #e1e8ed;
  border-radius: 6px;
  font-size: 13px;
}

.popular-title {
  color: #2c3e50;
  font-weight: 500;
}

.popular-stats {
  color: #e91e63;
  font-weight: bold;
}

.stats-footer {
  text-align: center;
  color: #666;
  padding-top: 15px;
  border-top: 1px solid #e1e8ed;
}
</style>
```

```vue
<!-- components/Card.vue -->
<template>
  <div class="card">
    <div v-if="$slots.header" class="card-header">
      <slot name="header"></slot>
    </div>

    <div class="card-body">
      <slot></slot>
    </div>

    <div v-if="$slots.footer" class="card-footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Card'
}
</script>

<style scoped>
.card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  transition: all 0.3s ease;
}

.card:hover {
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}

.card-header {
  padding: 20px 20px 0 20px;
  border-bottom: 1px solid #e1e8ed;
}

.card-header h2,
.card-header h3 {
  margin: 0 0 15px 0;
  color: #2c3e50;
}

.card-body {
  padding: 20px;
}

.card-footer {
  padding: 0 20px 20px 20px;
}
</style>
```

## 🔥 핵심 포인트

1. **Vue 컴포넌트가 더 직관적**
   - 단일 파일 컴포넌트 (SFC)로 HTML, JS, CSS 분리
   - Props 유효성 검사가 더 명확
   - 스타일 스코핑이 기본 제공

2. **이벤트 시스템이 더 명확**
   - `$emit`으로 명시적 이벤트 발생
   - `emits` 옵션으로 이벤트 문서화
   - React의 콜백 지옥 방지

3. **Slots가 children보다 강력**
   - 이름있는 슬롯으로 여러 위치에 컨텐츠 삽입
   - 스코프드 슬롯으로 데이터 전달
   - 조건부 슬롯 렌더링

4. **컴포넌트 간 통신이 명확**
   - Props Down, Events Up 패턴
   - React보다 더 명시적인 인터페이스

## ➡️ 다음 단계

컴포넌트 기초를 마스터했다면 이제 중급 과정으로 넘어가서 더 복잡한 Vue 기능들을 배워보세요!

- [2-1. 컴포넌트 간 통신](../02-intermediate/01-component-communication.md)
- [Vue Router와 Pinia](../03-advanced/)

---

*Vue 컴포넌트는 React보다 더 직관적이고 강력해요! 특히 Slots는 정말 혁신적인 기능입니다. 복잡한 UI 구조를 훨씬 깔끔하게 만들 수 있거든요! 🎨* 