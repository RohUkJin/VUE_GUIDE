# 2-5. 커스텀 디렉티브 (Custom Directives)

## 🎯 Vue vs React: 커스텀 디렉티브와 훅

Vue의 커스텀 디렉티브는 React의 커스텀 훅과 비슷한 역할을 하지만, DOM 조작에 더 특화되어 있어요!

**🎯 학습 목표**
- React 커스텀 훅 vs Vue 커스텀 디렉티브의 차이점과 활용법 이해
- DOM 조작이 필요한 상황에서 디렉티브의 장점 체험
- 실무에서 자주 사용하는 디렉티브 패턴들 (lazy loading, tooltip 등) 구현
- 재사용 가능한 UI 동작을 디렉티브로 캡슐화하는 기법 습득

**💡 왜 Vue의 디렉티브가 DOM 조작에 더 좋을까요?**
- **직접 DOM 접근**: ref 없이도 바로 DOM 요소에 접근 가능
- **생명주기 훅**: mounted, updated, unmounted 등 세밀한 제어 가능
- **선언적 사용**: HTML 속성처럼 자연스럽게 사용 (`v-tooltip="message"`)
- **인자와 수식어**: `v-directive:arg.modifier` 형태로 유연한 설정 가능

## 🔧 1. 기본 개념 비교

### React 방식 (Custom Hooks)
```jsx
import { useEffect, useRef } from 'react';

// 커스텀 훅: 클릭 외부 감지
function useClickOutside(callback) {
  const ref = useRef();
  
  useEffect(() => {
    function handleClickOutside(event) {
      if (ref.current && !ref.current.contains(event.target)) {
        callback();
      }
    }
    
    document.addEventListener('mousedown', handleClickOutside);
    return () => {
      document.removeEventListener('mousedown', handleClickOutside);
    };
  }, [callback]);
  
  return ref;
}

// 커스텀 훅: 자동 포커스
function useAutoFocus() {
  const ref = useRef();
  
  useEffect(() => {
    if (ref.current) {
      ref.current.focus();
    }
  }, []);
  
  return ref;
}

// 사용
function Modal({ onClose }) {
  const modalRef = useClickOutside(onClose);
  const inputRef = useAutoFocus();
  
  return (
    <div ref={modalRef} className="modal">
      <input ref={inputRef} placeholder="입력하세요..." />
      <button onClick={onClose}>닫기</button>
    </div>
  );
}
```

### Vue 방식 (Custom Directives)
```vue
<template>
  <div>
    <!-- 커스텀 디렉티브 사용 -->
    <div v-click-outside="closeModal" class="modal">
      <input v-auto-focus placeholder="입력하세요..." />
      <button @click="closeModal">닫기</button>
    </div>
  </div>
</template>

<script>
export default {
  // 로컬 디렉티브 정의
  directives: {
    // 클릭 외부 감지 디렉티브
    clickOutside: {
      mounted(el, binding) {
        const handler = (event) => {
          if (el && !el.contains(event.target)) {
            binding.value();
          }
        };
        
        document.addEventListener('click', handler);
        el._clickOutsideHandler = handler;
      },
      
      unmounted(el) {
        if (el._clickOutsideHandler) {
          document.removeEventListener('click', el._clickOutsideHandler);
          delete el._clickOutsideHandler;
        }
      }
    },
    
    // 자동 포커스 디렉티브
    autoFocus: {
      mounted(el) {
        el.focus();
      }
    }
  },
  
  methods: {
    closeModal() {
      console.log('모달 닫기');
    }
  }
}
</script>
```

### 글로벌 디렉티브 등록
```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

// 전역 디렉티브 등록
app.directive('click-outside', {
  mounted(el, binding) {
    const handler = (event) => {
      if (el && !el.contains(event.target)) {
        binding.value();
      }
    };
    
    document.addEventListener('click', handler);
    el._clickOutsideHandler = handler;
  },
  
  unmounted(el) {
    if (el._clickOutsideHandler) {
      document.removeEventListener('click', el._clickOutsideHandler);
      delete el._clickOutsideHandler;
    }
  }
})

app.mount('#app')
```

## 🎨 2. 실용적인 커스텀 디렉티브들

### 텍스트 하이라이트 디렉티브
```vue
<template>
  <div>
    <input v-model="searchTerm" placeholder="검색어 입력..." />
    
    <div class="search-results">
      <div 
        v-for="item in items" 
        :key="item.id"
        v-highlight="searchTerm"
        class="result-item"
      >
        {{ item.text }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchTerm: '',
      items: [
        { id: 1, text: 'Vue.js는 프로그레시브 프레임워크입니다' },
        { id: 2, text: 'React는 사용자 인터페이스를 만들기 위한 라이브러리입니다' },
        { id: 3, text: 'JavaScript는 웹 개발의 핵심 언어입니다' }
      ]
    }
  },
  
  directives: {
    highlight: {
      mounted(el, binding) {
        this.updateHighlight(el, binding.value);
      },
      
      updated(el, binding) {
        if (binding.value !== binding.oldValue) {
          this.updateHighlight(el, binding.value);
        }
      },
      
      updateHighlight(el, searchTerm) {
        if (!searchTerm) {
          el.innerHTML = el.textContent;
          return;
        }
        
        const text = el.textContent;
        const regex = new RegExp(`(${searchTerm})`, 'gi');
        const highlightedText = text.replace(regex, '<mark>$1</mark>');
        el.innerHTML = highlightedText;
      }
    }
  }
}
</script>

<style scoped>
mark {
  background-color: yellow;
  padding: 2px 4px;
  border-radius: 2px;
}

.result-item {
  padding: 10px;
  border-bottom: 1px solid #eee;
}
</style>
```

### 이미지 지연 로딩 디렉티브
```javascript
// directives/lazy-load.js
export const lazyLoad = {
  mounted(el, binding) {
    const imageObserver = new IntersectionObserver((entries, observer) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const img = entry.target;
          
          // 실제 이미지 로드
          const actualImg = new Image();
          actualImg.onload = () => {
            img.src = binding.value;
            img.classList.remove('lazy');
            img.classList.add('loaded');
          };
          
          actualImg.onerror = () => {
            img.src = '/placeholder-error.png';
            img.classList.add('error');
          };
          
          actualImg.src = binding.value;
          observer.unobserve(img);
        }
      });
    });
    
    // 초기 placeholder 설정
    el.src = el.dataset.placeholder || '/placeholder.png';
    el.classList.add('lazy');
    
    imageObserver.observe(el);
    el._imageObserver = imageObserver;
  },
  
  unmounted(el) {
    if (el._imageObserver) {
      el._imageObserver.disconnect();
    }
  }
}
```

```vue
<template>
  <div class="image-gallery">
    <img
      v-for="image in images"
      :key="image.id"
      v-lazy-load="image.url"
      :data-placeholder="image.placeholder"
      :alt="image.alt"
      class="gallery-image"
    />
  </div>
</template>

<script>
import { lazyLoad } from './directives/lazy-load.js'

export default {
  directives: { lazyLoad },
  
  data() {
    return {
      images: [
        {
          id: 1,
          url: '/images/high-res-1.jpg',
          placeholder: '/images/low-res-1.jpg',
          alt: '이미지 1'
        },
        {
          id: 2,
          url: '/images/high-res-2.jpg',
          placeholder: '/images/low-res-2.jpg',
          alt: '이미지 2'
        }
      ]
    }
  }
}
</script>

<style scoped>
.gallery-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
  transition: opacity 0.3s ease;
}

.gallery-image.lazy {
  opacity: 0.5;
  filter: blur(2px);
}

.gallery-image.loaded {
  opacity: 1;
  filter: none;
}

.gallery-image.error {
  opacity: 0.3;
}
</style>
```

### 드래그 앤 드롭 디렉티브
```javascript
// directives/draggable.js
export const draggable = {
  mounted(el, binding) {
    let isDragging = false;
    let startX, startY, initialX, initialY;
    
    const dragStart = (e) => {
      if (binding.modifiers.handle) {
        const handle = el.querySelector(binding.arg || '.drag-handle');
        if (!handle || !handle.contains(e.target)) return;
      }
      
      isDragging = true;
      
      if (e.type === 'mousedown') {
        startX = e.clientX;
        startY = e.clientY;
      } else {
        startX = e.touches[0].clientX;
        startY = e.touches[0].clientY;
      }
      
      const rect = el.getBoundingClientRect();
      initialX = rect.left;
      initialY = rect.top;
      
      el.style.cursor = 'grabbing';
      el.classList.add('dragging');
      
      document.addEventListener('mousemove', dragMove);
      document.addEventListener('mouseup', dragEnd);
      document.addEventListener('touchmove', dragMove);
      document.addEventListener('touchend', dragEnd);
      
      if (binding.value && binding.value.onStart) {
        binding.value.onStart(e, { x: initialX, y: initialY });
      }
    };
    
    const dragMove = (e) => {
      if (!isDragging) return;
      
      e.preventDefault();
      
      let currentX, currentY;
      if (e.type === 'mousemove') {
        currentX = e.clientX;
        currentY = e.clientY;
      } else {
        currentX = e.touches[0].clientX;
        currentY = e.touches[0].clientY;
      }
      
      const deltaX = currentX - startX;
      const deltaY = currentY - startY;
      
      const newX = initialX + deltaX;
      const newY = initialY + deltaY;
      
      // 경계 체크
      if (binding.modifiers.boundary) {
        const container = el.parentElement;
        const containerRect = container.getBoundingClientRect();
        const elRect = el.getBoundingClientRect();
        
        const minX = 0;
        const minY = 0;
        const maxX = containerRect.width - elRect.width;
        const maxY = containerRect.height - elRect.height;
        
        const constrainedX = Math.max(minX, Math.min(maxX, newX));
        const constrainedY = Math.max(minY, Math.min(maxY, newY));
        
        el.style.left = constrainedX + 'px';
        el.style.top = constrainedY + 'px';
      } else {
        el.style.left = newX + 'px';
        el.style.top = newY + 'px';
      }
      
      if (binding.value && binding.value.onMove) {
        binding.value.onMove(e, { x: newX, y: newY, deltaX, deltaY });
      }
    };
    
    const dragEnd = (e) => {
      if (!isDragging) return;
      
      isDragging = false;
      el.style.cursor = '';
      el.classList.remove('dragging');
      
      document.removeEventListener('mousemove', dragMove);
      document.removeEventListener('mouseup', dragEnd);
      document.removeEventListener('touchmove', dragMove);
      document.removeEventListener('touchend', dragEnd);
      
      if (binding.value && binding.value.onEnd) {
        const rect = el.getBoundingClientRect();
        binding.value.onEnd(e, { x: rect.left, y: rect.top });
      }
    };
    
    el.style.position = 'absolute';
    el.style.cursor = 'grab';
    
    el.addEventListener('mousedown', dragStart);
    el.addEventListener('touchstart', dragStart);
    
    el._dragHandlers = { dragStart, dragMove, dragEnd };
  },
  
  unmounted(el) {
    if (el._dragHandlers) {
      el.removeEventListener('mousedown', el._dragHandlers.dragStart);
      el.removeEventListener('touchstart', el._dragHandlers.dragStart);
    }
  }
}
```

```vue
<template>
  <div class="draggable-demo">
    <h2>드래그 앤 드롭 데모</h2>
    
    <div class="container">
      <!-- 기본 드래그 -->
      <div
        v-draggable="{ onMove: handleDrag, onEnd: handleDragEnd }"
        class="draggable-item basic"
      >
        기본 드래그
      </div>
      
      <!-- 경계 제한 드래그 -->
      <div
        v-draggable.boundary="{ onMove: handleDrag }"
        class="draggable-item boundary"
      >
        경계 제한 드래그
      </div>
      
      <!-- 핸들로만 드래그 -->
      <div
        v-draggable.handle="{ onMove: handleDrag }"
        class="draggable-item handle"
      >
        <div class="drag-handle">📌 핸들</div>
        <div class="content">핸들로만 드래그 가능</div>
      </div>
    </div>
    
    <div class="info">
      <p>마지막 드래그 위치: {{ lastPosition.x }}, {{ lastPosition.y }}</p>
    </div>
  </div>
</template>

<script>
import { draggable } from './directives/draggable.js'

export default {
  directives: { draggable },
  
  data() {
    return {
      lastPosition: { x: 0, y: 0 }
    }
  },
  
  methods: {
    handleDrag(event, position) {
      console.log('Dragging:', position);
    },
    
    handleDragEnd(event, position) {
      this.lastPosition = position;
      console.log('Drag ended at:', position);
    }
  }
}
</script>

<style scoped>
.container {
  position: relative;
  height: 400px;
  border: 2px dashed #ccc;
  margin: 20px 0;
}

.draggable-item {
  width: 150px;
  height: 100px;
  padding: 10px;
  background: #42b883;
  color: white;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  user-select: none;
}

.draggable-item.basic {
  top: 10px;
  left: 10px;
}

.draggable-item.boundary {
  background: #ff6b6b;
  top: 10px;
  left: 180px;
}

.draggable-item.handle {
  background: #ffd43b;
  color: #333;
  top: 10px;
  left: 350px;
  flex-direction: column;
}

.draggable-item.dragging {
  transform: rotate(5deg);
  box-shadow: 0 8px 16px rgba(0,0,0,0.3);
}

.drag-handle {
  background: #333;
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: grab;
  margin-bottom: 5px;
}

.content {
  text-align: center;
  font-size: 12px;
}
</style>
```

### 권한 기반 디렉티브
```javascript
// directives/permission.js
export const permission = {
  mounted(el, binding) {
    const userPermissions = binding.instance.$store?.state?.user?.permissions || [];
    const requiredPermissions = Array.isArray(binding.value) ? binding.value : [binding.value];
    
    const hasPermission = requiredPermissions.every(permission => 
      userPermissions.includes(permission)
    );
    
    if (!hasPermission) {
      if (binding.modifiers.hide) {
        el.style.display = 'none';
      } else if (binding.modifiers.disable) {
        el.disabled = true;
        el.style.opacity = '0.5';
        el.style.cursor = 'not-allowed';
      } else {
        el.remove();
      }
    }
  },
  
  updated(el, binding) {
    // 권한이 변경되었을 때 재검사
    if (binding.value !== binding.oldValue) {
      // 요소 재생성이 필요한 경우의 로직
    }
  }
}
```

```vue
<template>
  <div class="admin-panel">
    <h1>관리자 패널</h1>
    
    <!-- 관리자만 볼 수 있는 버튼 -->
    <button v-permission="'admin'">
      관리자 전용 기능
    </button>
    
    <!-- 여러 권한 중 하나라도 있으면 표시 -->
    <button v-permission="['moderator', 'admin']">
      중재자 또는 관리자 기능
    </button>
    
    <!-- 권한이 없으면 숨김 -->
    <div v-permission.hide="'super_admin'">
      슈퍼 관리자 전용 영역
    </div>
    
    <!-- 권한이 없으면 비활성화 -->
    <button v-permission.disable="'delete_posts'">
      게시글 삭제
    </button>
    
    <!-- 일반 사용자는 항상 볼 수 있음 -->
    <button>일반 기능</button>
  </div>
</template>

<script>
import { permission } from './directives/permission.js'

export default {
  directives: { permission }
}
</script>
```

## 🎯 3. 실습 예제: 툴팁 디렉티브

```javascript
// directives/tooltip.js
export const tooltip = {
  mounted(el, binding) {
    const tooltip = document.createElement('div');
    tooltip.className = 'custom-tooltip';
    tooltip.textContent = binding.value;
    tooltip.style.cssText = `
      position: absolute;
      background: #333;
      color: white;
      padding: 8px 12px;
      border-radius: 4px;
      font-size: 12px;
      white-space: nowrap;
      z-index: 9999;
      opacity: 0;
      pointer-events: none;
      transition: opacity 0.2s ease;
      transform: translateX(-50%);
    `;
    
    document.body.appendChild(tooltip);
    
    const showTooltip = (e) => {
      const rect = el.getBoundingClientRect();
      const tooltipRect = tooltip.getBoundingClientRect();
      
      let top = rect.top - tooltipRect.height - 8;
      let left = rect.left + rect.width / 2;
      
      // 화면 경계 체크
      if (top < 8) {
        top = rect.bottom + 8;
        tooltip.style.transform = 'translateX(-50%)';
      }
      
      if (left - tooltipRect.width / 2 < 8) {
        left = 8 + tooltipRect.width / 2;
      } else if (left + tooltipRect.width / 2 > window.innerWidth - 8) {
        left = window.innerWidth - 8 - tooltipRect.width / 2;
      }
      
      tooltip.style.top = top + window.scrollY + 'px';
      tooltip.style.left = left + 'px';
      tooltip.style.opacity = '1';
    };
    
    const hideTooltip = () => {
      tooltip.style.opacity = '0';
    };
    
    el.addEventListener('mouseenter', showTooltip);
    el.addEventListener('mouseleave', hideTooltip);
    
    el._tooltip = tooltip;
    el._tooltipHandlers = { showTooltip, hideTooltip };
  },
  
  updated(el, binding) {
    if (el._tooltip && binding.value !== binding.oldValue) {
      el._tooltip.textContent = binding.value;
    }
  },
  
  unmounted(el) {
    if (el._tooltip) {
      document.body.removeChild(el._tooltip);
    }
    
    if (el._tooltipHandlers) {
      el.removeEventListener('mouseenter', el._tooltipHandlers.showTooltip);
      el.removeEventListener('mouseleave', el._tooltipHandlers.hideTooltip);
    }
  }
}
```

```vue
<template>
  <div class="tooltip-demo">
    <h2>커스텀 툴팁 데모</h2>
    
    <div class="demo-area">
      <button v-tooltip="'이것은 간단한 툴팁입니다'">
        기본 툴팁
      </button>
      
      <button v-tooltip="dynamicTooltip">
        동적 툴팁
      </button>
      
      <div 
        v-tooltip="'이 영역에 마우스를 올려보세요'"
        class="hover-area"
      >
        호버 영역
      </div>
      
      <input 
        v-tooltip="'여기에 이메일 주소를 입력하세요'"
        placeholder="이메일"
        type="email"
      />
    </div>
    
    <div class="controls">
      <button @click="updateTooltip">툴팁 텍스트 변경</button>
    </div>
  </div>
</template>

<script>
import { tooltip } from './directives/tooltip.js'

export default {
  directives: { tooltip },
  
  data() {
    return {
      dynamicTooltip: '현재 시간: ' + new Date().toLocaleTimeString()
    }
  },
  
  methods: {
    updateTooltip() {
      this.dynamicTooltip = '업데이트됨: ' + new Date().toLocaleTimeString();
    }
  }
}
</script>

<style scoped>
.demo-area {
  display: flex;
  gap: 20px;
  flex-wrap: wrap;
  margin: 20px 0;
}

.hover-area {
  width: 150px;
  height: 100px;
  background: #f0f0f0;
  border: 2px dashed #ccc;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

button, input {
  padding: 10px 15px;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
}

.controls {
  margin-top: 20px;
}
</style>
```

## 🎯 실습 과제

1. 위의 모든 커스텀 디렉티브를 직접 구현해보세요
2. 텍스트 자동 완성 디렉티브를 만들어보세요
3. 무한 스크롤 디렉티브를 구현해보세요
4. 복사하기 디렉티브를 만들어보세요
5. 확대/축소 가능한 이미지 디렉티브를 구현해보세요

## 🔗 다음 단계

축하합니다! 🎉 Vue.js 중급 과정을 완료했습니다!

이제 **심화 과정**으로 넘어가서 다음 내용들을 학습하게 됩니다:
- Composition API 마스터
- 커스텀 Composables 제작
- Pinia를 활용한 상태 관리
- Vue Router로 SPA 구축
- 성능 최적화 기법

---

💡 **팁**: Vue의 커스텀 디렉티브는 DOM 조작이 필요한 재사용 가능한 로직을 깔끔하게 캡슐화할 수 있는 강력한 기능입니다! 