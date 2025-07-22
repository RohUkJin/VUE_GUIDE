# 1-4. 이벤트 핸들링

Vue의 이벤트 처리는 React보다 **훨씬 간단하고 직관적**입니다! 특히 이벤트 수식어(modifiers)는 React 개발자라면 "이런 게 있으면 좋겠다"고 생각했던 기능들이 모두 내장되어 있어요.

**🎯 학습 목표**
- React의 수동 이벤트 처리 vs Vue의 선언적 이벤트 처리 비교
- 이벤트 수식어(`.prevent`, `.stop`, `.once` 등)의 강력함 체험
- 키보드/마우스 수식어로 복잡한 상호작용 간단하게 구현
- 실무에서 자주 사용하는 이벤트 패턴들 익히기

**💡 왜 Vue의 이벤트 시스템이 더 편할까요?**
- **선언적**: HTML에서 바로 이벤트 동작 확인 가능
- **수식어**: `.prevent`, `.stop` 등으로 보일러플레이트 제거
- **키 조합**: `.ctrl.enter`, `.esc` 등 복잡한 키 조합 간단히 처리
- **직관적**: 코드만 봐도 어떤 이벤트인지 바로 이해

## 🎯 기본 이벤트 핸들링

### React vs Vue 이벤트 처리

**React 이벤트 처리**
```jsx
import React, { useState } from 'react';

function EventDemo() {
  const [message, setMessage] = useState('');
  const [count, setCount] = useState(0);

  const handleClick = (e) => {
    console.log('버튼 클릭됨!', e.target);
    setCount(count + 1);
  };

  const handleSubmit = (e) => {
    e.preventDefault(); // 기본 동작 방지
    e.stopPropagation(); // 이벤트 버블링 방지
    console.log('폼 제출:', message);
  };

  const handleKeyDown = (e) => {
    if (e.key === 'Enter') {
      console.log('엔터키 눌림');
    }
    if (e.key === 'Escape') {
      setMessage('');
    }
    if (e.ctrlKey && e.key === 's') {
      e.preventDefault();
      console.log('Ctrl+S 눌림');
    }
  };

  const handleMouseEvents = (e) => {
    console.log('마우스 이벤트:', e.type, e.button);
  };

  return (
    <div>
      <button onClick={handleClick}>
        클릭 횟수: {count}
      </button>

      <form onSubmit={handleSubmit}>
        <input
          value={message}
          onChange={(e) => setMessage(e.target.value)}
          onKeyDown={handleKeyDown}
          placeholder="메시지 입력"
        />
        <button type="submit">제출</button>
      </form>

      <div
        onMouseDown={handleMouseEvents}
        onMouseUp={handleMouseEvents}
        onContextMenu={handleMouseEvents}
        style={{ padding: '20px', border: '1px solid #ccc' }}
      >
        마우스 이벤트 테스트
      </div>
    </div>
  );
}
```

**Vue 이벤트 처리**
```vue
<template>
  <div>
    <!-- @ = v-on: 단축문법 -->
    <button @click="handleClick">
      클릭 횟수: {{ count }}
    </button>

    <!-- .prevent와 .stop 수식어로 간편하게! -->
    <form @submit.prevent.stop="handleSubmit">
      <input
        v-model="message"
        @keydown.enter="handleEnter"
        @keydown.esc="handleEscape"
        @keydown.ctrl.s.prevent="handleSave"
        placeholder="메시지 입력"
      />
      <button type="submit">제출</button>
    </form>

    <!-- 마우스 수식어로 더 명확하게 -->
    <div
      @mousedown.left="handleLeftMouse"
      @mousedown.right.prevent="handleRightMouse"
      @mousedown.middle="handleMiddleMouse"
      class="mouse-area"
    >
      마우스 이벤트 테스트
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const message = ref('');
    const count = ref(0);

    const handleClick = (e) => {
      console.log('버튼 클릭됨!', e.target);
      count.value++;
    };

    const handleSubmit = () => {
      // prevent와 stop이 이미 적용됨!
      console.log('폼 제출:', message.value);
    };

    const handleEnter = () => {
      console.log('엔터키 눌림');
    };

    const handleEscape = () => {
      message.value = '';
    };

    const handleSave = () => {
      console.log('Ctrl+S 눌림');
    };

    const handleLeftMouse = () => {
      console.log('왼쪽 마우스 버튼');
    };

    const handleRightMouse = () => {
      console.log('오른쪽 마우스 버튼');
    };

    const handleMiddleMouse = () => {
      console.log('가운데 마우스 버튼');
    };

    return {
      message,
      count,
      handleClick,
      handleSubmit,
      handleEnter,
      handleEscape,
      handleSave,
      handleLeftMouse,
      handleRightMouse,
      handleMiddleMouse
    }
  }
}
</script>

<style scoped>
.mouse-area {
  padding: 20px;
  border: 1px solid #ccc;
  background: #f9f9f9;
  user-select: none;
}
</style>
```

## 🛠️ 이벤트 수식어 (Event Modifiers)

Vue의 가장 강력한 기능 중 하나입니다!

### 기본 이벤트 수식어

```vue
<template>
  <div class="modifier-demo">
    <!-- .prevent: e.preventDefault() -->
    <form @submit.prevent="handleSubmit">
      <button type="submit">제출 (새로고침 안됨)</button>
    </form>

    <!-- .stop: e.stopPropagation() -->
    <div @click="handleDivClick" class="outer">
      <p>외부 div 클릭하면 이벤트 발생</p>
      <button @click.stop="handleButtonClick" class="inner">
        클릭 (이벤트 버블링 방지)
      </button>
    </div>

    <!-- .once: 한 번만 실행 -->
    <button @click.once="handleOnce" class="once-btn">
      한 번만 클릭됨 ({{ onceClicked ? '실행됨' : '대기중' }})
    </button>

    <!-- .self: 자기 자신에서만 실행 -->
    <div @click.self="handleSelfOnly" class="self-area">
      <p>이 p를 클릭해도 핸들러 실행 안됨</p>
      <span>이 span을 클릭해도 실행 안됨</span>
      <button>이 버튼을 클릭해도 실행 안됨</button>
      <br>
      <small>div 배경을 직접 클릭해야 실행됨</small>
    </div>

    <!-- .capture: 캡처링 단계에서 실행 -->
    <div @click.capture="handleCapture" class="capture-area">
      <button @click="handleInnerCapture">
        캡처링 테스트 (capture가 먼저 실행됨)
      </button>
    </div>

    <!-- .passive: passive 리스너로 등록 -->
    <div @scroll.passive="handleScroll" class="scroll-area">
      <div class="scroll-content">
        스크롤해보세요 (passive 최적화 적용)
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const onceClicked = ref(false);

    const handleSubmit = () => {
      console.log('폼 제출됨 (새로고침 없음)');
    };

    const handleDivClick = () => {
      console.log('외부 div 클릭됨');
    };

    const handleButtonClick = () => {
      console.log('버튼 클릭됨 (버블링 방지)');
    };

    const handleOnce = () => {
      console.log('한 번만 실행되는 핸들러');
      onceClicked.value = true;
    };

    const handleSelfOnly = () => {
      console.log('div 자체를 클릭했을 때만 실행');
    };

    const handleCapture = () => {
      console.log('캡처링 단계에서 실행');
    };

    const handleInnerCapture = () => {
      console.log('내부 버튼 클릭');
    };

    const handleScroll = () => {
      console.log('스크롤 이벤트 (passive)');
    };

    return {
      onceClicked,
      handleSubmit,
      handleDivClick,
      handleButtonClick,
      handleOnce,
      handleSelfOnly,
      handleCapture,
      handleInnerCapture,
      handleScroll
    }
  }
}
</script>

<style scoped>
.modifier-demo > * {
  margin: 20px 0;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.outer {
  background: #e8f4f8;
  cursor: pointer;
}

.inner {
  background: #fff;
  margin: 10px;
}

.once-btn {
  background: #f39c12;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 4px;
  cursor: pointer;
}

.self-area {
  background: #ffeaa7;
  min-height: 100px;
  cursor: pointer;
}

.capture-area {
  background: #fd79a8;
  color: white;
}

.scroll-area {
  height: 100px;
  overflow-y: scroll;
  background: #f1f2f6;
}

.scroll-content {
  height: 300px;
  padding: 10px;
  background: linear-gradient(45deg, #667eea 0%, #764ba2 100%);
  color: white;
}
</style>
```

## ⌨️ 키 수식어 (Key Modifiers)

React에서는 복잡한 if문으로 처리해야 하는 키 이벤트를 간단하게!

### 기본 키 수식어

```vue
<template>
  <div class="key-demo">
    <h3>키 입력 테스트</h3>
    
    <!-- 특수 키들 -->
    <div class="input-group">
      <label>기본 키들:</label>
      <input
        v-model="basicInput"
        @keyup.enter="handleEnter"
        @keyup.esc="handleEscape"
        @keyup.space="handleSpace"
        @keyup.tab="handleTab"
        @keyup.delete="handleDelete"
        placeholder="Enter, Esc, Space, Tab, Delete 테스트"
      />
    </div>

    <!-- 화살표 키들 -->
    <div class="input-group">
      <label>화살표 키들:</label>
      <input
        v-model="arrowInput"
        @keyup.up="handleUp"
        @keyup.down="handleDown"
        @keyup.left="handleLeft"
        @keyup.right="handleRight"
        placeholder="화살표 키 테스트"
      />
    </div>

    <!-- 조합 키들 -->
    <div class="input-group">
      <label>조합 키들:</label>
      <input
        v-model="comboInput"
        @keyup.ctrl.enter="handleCtrlEnter"
        @keyup.alt.s="handleAltS"
        @keyup.shift.tab="handleShiftTab"
        @keyup.meta.k="handleMetaK"
        placeholder="Ctrl+Enter, Alt+S, Shift+Tab, Meta+K 테스트"
      />
    </div>

    <!-- 정확한 키 조합 (.exact) -->
    <div class="input-group">
      <label>정확한 조합:</label>
      <input
        v-model="exactInput"
        @keyup.ctrl.exact="handleCtrlOnly"
        @keyup.ctrl.shift.exact="handleCtrlShiftOnly"
        placeholder="정확히 Ctrl만, 또는 Ctrl+Shift만"
      />
    </div>

    <!-- 로그 출력 -->
    <div class="log">
      <h4>키 이벤트 로그:</h4>
      <ul>
        <li v-for="(log, index) in keyLogs" :key="index">
          {{ log }}
        </li>
      </ul>
      <button @click="clearLogs">로그 지우기</button>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const basicInput = ref('');
    const arrowInput = ref('');
    const comboInput = ref('');
    const exactInput = ref('');
    const keyLogs = ref([]);

    const addLog = (message) => {
      keyLogs.value.unshift(`${new Date().toLocaleTimeString()}: ${message}`);
      if (keyLogs.value.length > 10) {
        keyLogs.value = keyLogs.value.slice(0, 10);
      }
    };

    // 기본 키 핸들러들
    const handleEnter = () => addLog('Enter 키 눌림');
    const handleEscape = () => {
      addLog('Escape 키 눌림');
      // 모든 입력 필드 초기화
      basicInput.value = '';
      arrowInput.value = '';
      comboInput.value = '';
      exactInput.value = '';
    };
    const handleSpace = () => addLog('Space 키 눌림');
    const handleTab = () => addLog('Tab 키 눌림');
    const handleDelete = () => addLog('Delete 키 눌림');

    // 화살표 키 핸들러들
    const handleUp = () => addLog('위쪽 화살표 키 눌림');
    const handleDown = () => addLog('아래쪽 화살표 키 눌림');
    const handleLeft = () => addLog('왼쪽 화살표 키 눌림');
    const handleRight = () => addLog('오른쪽 화살표 키 눌림');

    // 조합 키 핸들러들
    const handleCtrlEnter = () => addLog('Ctrl + Enter 조합');
    const handleAltS = () => addLog('Alt + S 조합');
    const handleShiftTab = () => addLog('Shift + Tab 조합');
    const handleMetaK = () => addLog('Meta + K 조합');

    // 정확한 조합 핸들러들
    const handleCtrlOnly = () => addLog('정확히 Ctrl만 (다른 키 없음)');
    const handleCtrlShiftOnly = () => addLog('정확히 Ctrl + Shift만');

    const clearLogs = () => {
      keyLogs.value = [];
    };

    return {
      basicInput,
      arrowInput,
      comboInput,
      exactInput,
      keyLogs,
      handleEnter,
      handleEscape,
      handleSpace,
      handleTab,
      handleDelete,
      handleUp,
      handleDown,
      handleLeft,
      handleRight,
      handleCtrlEnter,
      handleAltS,
      handleShiftTab,
      handleMetaK,
      handleCtrlOnly,
      handleCtrlShiftOnly,
      clearLogs
    }
  }
}
</script>

<style scoped>
.key-demo {
  max-width: 600px;
  margin: 20px auto;
  padding: 20px;
}

.input-group {
  margin: 15px 0;
}

.input-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #2c3e50;
}

.input-group input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.input-group input:focus {
  outline: none;
  border-color: #42b883;
  box-shadow: 0 0 5px rgba(66, 184, 131, 0.3);
}

.log {
  margin-top: 30px;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 4px;
}

.log h4 {
  margin-top: 0;
  color: #495057;
}

.log ul {
  list-style: none;
  padding: 0;
  max-height: 200px;
  overflow-y: auto;
}

.log li {
  padding: 5px;
  margin: 2px 0;
  background: white;
  border-radius: 3px;
  font-family: monospace;
  font-size: 12px;
  border-left: 3px solid #42b883;
}

button {
  background: #e74c3c;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
}

button:hover {
  background: #c0392b;
}
</style>
```

## 🖱️ 마우스 수식어 (Mouse Modifiers)

마우스 버튼별로 다른 동작을 쉽게 구현할 수 있습니다!

```vue
<template>
  <div class="mouse-demo">
    <h3>마우스 이벤트 테스트</h3>
    
    <!-- 기본 마우스 버튼들 -->
    <div 
      class="mouse-area primary"
      @click.left="handleLeftClick"
      @click.right.prevent="handleRightClick"
      @click.middle="handleMiddleClick"
    >
      <h4>마우스 버튼 테스트</h4>
      <p>왼쪽 클릭: 좋아요</p>
      <p>오른쪽 클릭: 메뉴 (컨텍스트 메뉴 방지)</p>
      <p>가운데 클릭: 북마크</p>
    </div>

    <!-- 조합 키 + 마우스 -->
    <div 
      class="mouse-area secondary"
      @click.ctrl="handleCtrlClick"
      @click.alt="handleAltClick"
      @click.shift="handleShiftClick"
      @click.meta="handleMetaClick"
    >
      <h4>조합 키 + 마우스</h4>
      <p>Ctrl + 클릭: 새 탭에서 열기</p>
      <p>Alt + 클릭: 다운로드</p>
      <p>Shift + 클릭: 범위 선택</p>
      <p>Meta + 클릭: 특별 동작</p>
    </div>

    <!-- 복잡한 조합 -->
    <div 
      class="mouse-area complex"
      @click.right.ctrl.prevent="handleComplexCombo"
      @click.left.shift.alt="handleAnotherCombo"
    >
      <h4>복잡한 조합</h4>
      <p>Ctrl + 오른쪽 클릭: 개발자 메뉴</p>
      <p>Shift + Alt + 왼쪽 클릭: 고급 기능</p>
    </div>

    <!-- 드래그 앤 드롭 -->
    <div class="drag-demo">
      <div 
        class="draggable"
        draggable="true"
        @dragstart="handleDragStart"
        @drag="handleDrag"
        @dragend="handleDragEnd"
      >
        드래그 가능한 요소
      </div>

      <div 
        class="drop-zone"
        @dragover.prevent="handleDragOver"
        @drop.prevent="handleDrop"
        @dragenter.prevent="handleDragEnter"
        @dragleave="handleDragLeave"
        :class="{ 'drag-active': isDragActive }"
      >
        여기에 드롭하세요
        <p v-if="droppedCount > 0">드롭된 횟수: {{ droppedCount }}</p>
      </div>
    </div>

    <!-- 이벤트 로그 -->
    <div class="event-log">
      <h4>마우스 이벤트 로그:</h4>
      <ul>
        <li v-for="(event, index) in mouseEvents" :key="index">
          {{ event }}
        </li>
      </ul>
      <button @click="clearMouseEvents">로그 지우기</button>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const mouseEvents = ref([]);
    const isDragActive = ref(false);
    const droppedCount = ref(0);

    const addEvent = (message) => {
      mouseEvents.value.unshift(`${new Date().toLocaleTimeString()}: ${message}`);
      if (mouseEvents.value.length > 15) {
        mouseEvents.value = mouseEvents.value.slice(0, 15);
      }
    };

    // 기본 마우스 이벤트
    const handleLeftClick = () => addEvent('❤️ 좋아요!');
    const handleRightClick = () => addEvent('📋 컨텍스트 메뉴 (차단됨)');
    const handleMiddleClick = () => addEvent('🔖 북마크 추가');

    // 조합 키 + 마우스
    const handleCtrlClick = () => addEvent('🆕 Ctrl + 클릭: 새 탭');
    const handleAltClick = () => addEvent('📥 Alt + 클릭: 다운로드');
    const handleShiftClick = () => addEvent('📑 Shift + 클릭: 범위 선택');
    const handleMetaClick = () => addEvent('⭐ Meta + 클릭: 특별 동작');

    // 복잡한 조합
    const handleComplexCombo = () => addEvent('🛠️ Ctrl + 오른쪽 클릭: 개발자 메뉴');
    const handleAnotherCombo = () => addEvent('🔧 Shift + Alt + 왼쪽 클릭: 고급 기능');

    // 드래그 앤 드롭
    const handleDragStart = (e) => {
      addEvent('🏁 드래그 시작');
      e.dataTransfer.setData('text/plain', '드래그된 데이터');
    };

    const handleDrag = () => {
      // 너무 많은 로그를 방지하기 위해 주석 처리
      // addEvent('드래그 중...');
    };

    const handleDragEnd = () => addEvent('🏁 드래그 종료');

    const handleDragOver = () => {
      // preventDefault는 수식어로 처리됨
    };

    const handleDragEnter = () => {
      isDragActive.value = true;
      addEvent('📥 드롭 영역 진입');
    };

    const handleDragLeave = () => {
      isDragActive.value = false;
      addEvent('📤 드롭 영역 나감');
    };

    const handleDrop = (e) => {
      isDragActive.value = false;
      droppedCount.value++;
      const data = e.dataTransfer.getData('text/plain');
      addEvent(`🎯 드롭 완료: ${data}`);
    };

    const clearMouseEvents = () => {
      mouseEvents.value = [];
    };

    return {
      mouseEvents,
      isDragActive,
      droppedCount,
      handleLeftClick,
      handleRightClick,
      handleMiddleClick,
      handleCtrlClick,
      handleAltClick,
      handleShiftClick,
      handleMetaClick,
      handleComplexCombo,
      handleAnotherCombo,
      handleDragStart,
      handleDrag,
      handleDragEnd,
      handleDragOver,
      handleDragEnter,
      handleDragLeave,
      handleDrop,
      clearMouseEvents
    }
  }
}
</script>

<style scoped>
.mouse-demo {
  max-width: 700px;
  margin: 20px auto;
  padding: 20px;
}

.mouse-area {
  margin: 20px 0;
  padding: 20px;
  border-radius: 8px;
  cursor: pointer;
  user-select: none;
  transition: all 0.3s ease;
}

.mouse-area:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.secondary {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  color: white;
}

.complex {
  background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
  color: white;
}

.mouse-area h4 {
  margin-top: 0;
  font-size: 18px;
}

.mouse-area p {
  margin: 8px 0;
  font-size: 14px;
}

.drag-demo {
  display: flex;
  gap: 20px;
  margin: 30px 0;
  align-items: stretch;
}

.draggable {
  flex: 1;
  padding: 20px;
  background: #f39c12;
  color: white;
  border-radius: 8px;
  cursor: move;
  text-align: center;
  font-weight: bold;
  user-select: none;
}

.drop-zone {
  flex: 1;
  padding: 20px;
  border: 3px dashed #bdc3c7;
  border-radius: 8px;
  text-align: center;
  background: #ecf0f1;
  transition: all 0.3s ease;
}

.drop-zone.drag-active {
  border-color: #27ae60;
  background: #d5f4e6;
  transform: scale(1.02);
}

.event-log {
  margin-top: 30px;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 8px;
}

.event-log h4 {
  margin-top: 0;
  color: #495057;
}

.event-log ul {
  list-style: none;
  padding: 0;
  max-height: 300px;
  overflow-y: auto;
}

.event-log li {
  padding: 8px;
  margin: 3px 0;
  background: white;
  border-radius: 4px;
  font-family: monospace;
  font-size: 13px;
  border-left: 4px solid #42b883;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

button {
  background: #e74c3c;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
  transition: background 0.3s ease;
}

button:hover {
  background: #c0392b;
}
</style>
```

## 🎯 실습: 키보드 반응 게임

Vue의 이벤트 핸들링을 활용한 재미있는 게임을 만들어보세요!

```vue
<template>
  <div class="game-container">
    <h1>⌨️ 키보드 반응 게임</h1>
    
    <!-- 게임 상태 -->
    <div class="game-status">
      <div class="status-item">
        <span class="label">점수:</span>
        <span class="value">{{ score }}</span>
      </div>
      <div class="status-item">
        <span class="label">레벨:</span>
        <span class="value">{{ level }}</span>
      </div>
      <div class="status-item">
        <span class="label">시간:</span>
        <span class="value">{{ timeLeft }}초</span>
      </div>
      <div class="status-item">
        <span class="label">연속:</span>
        <span class="value">{{ combo }}회</span>
      </div>
    </div>

    <!-- 게임 영역 -->
    <div 
      v-if="gameState === 'playing'"
      class="game-area"
      tabindex="0"
      @keydown.prevent="handleKeyPress"
      ref="gameArea"
    >
      <div class="instruction">
        <p>화면에 나타나는 키를 빠르게 누르세요!</p>
        <p class="current-key">{{ currentKey }}</p>
      </div>

      <!-- 타겟 키 표시 -->
      <div class="target-display">
        <div 
          class="key-target" 
          :class="{ 'hit': keyHit, 'miss': keyMiss }"
        >
          {{ currentKey }}
        </div>
      </div>

      <!-- 진행바 -->
      <div class="progress-bar">
        <div 
          class="progress-fill" 
          :style="{ width: `${(timeLeft / maxTime) * 100}%` }"
        ></div>
      </div>
    </div>

    <!-- 시작/종료 화면 -->
    <div v-else class="menu-screen">
      <div v-if="gameState === 'ready'" class="start-screen">
        <h2>🎮 게임 준비</h2>
        <p>키보드 반응속도를 테스트해보세요!</p>
        <div class="difficulty">
          <label>난이도 선택:</label>
          <select v-model="difficulty">
            <option value="easy">쉬움 (3초)</option>
            <option value="normal">보통 (2초)</option>
            <option value="hard">어려움 (1.5초)</option>
            <option value="insane">미친 (1초)</option>
          </select>
        </div>
        <button @click="startGame" class="start-btn">게임 시작</button>
      </div>

      <div v-else-if="gameState === 'gameOver'" class="game-over">
        <h2>🏁 게임 종료!</h2>
        <div class="final-stats">
          <p>최종 점수: <strong>{{ score }}</strong></p>
          <p>도달 레벨: <strong>{{ level }}</strong></p>
          <p>최고 연속: <strong>{{ maxCombo }}</strong></p>
          <p>정확도: <strong>{{ accuracy }}%</strong></p>
        </div>
        <div class="rank">
          <p class="rank-text">등급: {{ rank }}</p>
        </div>
        <button @click="resetGame" class="restart-btn">다시 시작</button>
      </div>
    </div>

    <!-- 게임 로그 -->
    <div class="game-log">
      <h4>게임 로그:</h4>
      <ul>
        <li v-for="(log, index) in gameLogs" :key="index" :class="log.type">
          {{ log.message }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted } from 'vue'

export default {
  setup() {
    const gameState = ref('ready'); // 'ready', 'playing', 'gameOver'
    const score = ref(0);
    const level = ref(1);
    const timeLeft = ref(0);
    const maxTime = ref(3);
    const combo = ref(0);
    const maxCombo = ref(0);
    const difficulty = ref('normal');
    const currentKey = ref('');
    const keyHit = ref(false);
    const keyMiss = ref(false);
    const hitCount = ref(0);
    const missCount = ref(0);
    const gameLogs = ref([]);
    const gameArea = ref(null);

    let gameTimer = null;
    let keyTimer = null;

    const keys = ['A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L', 'Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P'];

    // 계산된 속성들
    const accuracy = computed(() => {
      const total = hitCount.value + missCount.value;
      return total > 0 ? Math.round((hitCount.value / total) * 100) : 0;
    });

    const rank = computed(() => {
      const acc = accuracy.value;
      if (acc >= 95) return '🏆 마스터';
      if (acc >= 85) return '🥇 고수';
      if (acc >= 75) return '🥈 숙련자';
      if (acc >= 60) return '🥉 초보자';
      return '😅 연습생';
    });

    // 로그 추가
    const addLog = (message, type = 'info') => {
      gameLogs.value.unshift({
        message: `${new Date().toLocaleTimeString()}: ${message}`,
        type
      });
      if (gameLogs.value.length > 10) {
        gameLogs.value = gameLogs.value.slice(0, 10);
      }
    };

    // 난이도별 시간 설정
    const setDifficulty = () => {
      const timeMap = {
        easy: 3,
        normal: 2,
        hard: 1.5,
        insane: 1
      };
      maxTime.value = timeMap[difficulty.value];
    };

    // 새로운 키 생성
    const generateNewKey = () => {
      const randomKey = keys[Math.floor(Math.random() * keys.length)];
      currentKey.value = randomKey;
      timeLeft.value = maxTime.value;
      
      // 키 타이머 시작
      keyTimer = setInterval(() => {
        timeLeft.value -= 0.1;
        if (timeLeft.value <= 0) {
          handleKeyMiss();
        }
      }, 100);
    };

    // 키 입력 처리
    const handleKeyPress = (e) => {
      const pressedKey = e.key.toUpperCase();
      
      if (pressedKey === currentKey.value) {
        handleKeyHit();
      } else {
        handleWrongKey(pressedKey);
      }
    };

    // 정확한 키 입력
    const handleKeyHit = () => {
      clearInterval(keyTimer);
      keyHit.value = true;
      
      const baseScore = 10;
      const timeBonus = Math.round(timeLeft.value * 5);
      const comboBonus = combo.value * 2;
      const totalScore = baseScore + timeBonus + comboBonus;
      
      score.value += totalScore;
      combo.value++;
      hitCount.value++;
      
      if (combo.value > maxCombo.value) {
        maxCombo.value = combo.value;
      }
      
      addLog(`✅ ${currentKey.value} 성공! +${totalScore}점 (연속: ${combo.value})`, 'success');
      
      // 레벨업 체크
      if (score.value >= level.value * 100) {
        level.value++;
        addLog(`🎉 레벨 업! 레벨 ${level.value}`, 'level');
        
        // 난이도 증가
        if (maxTime.value > 0.5) {
          maxTime.value -= 0.1;
        }
      }
      
      setTimeout(() => {
        keyHit.value = false;
        generateNewKey();
      }, 300);
    };

    // 키를 놓침
    const handleKeyMiss = () => {
      clearInterval(keyTimer);
      keyMiss.value = true;
      
      combo.value = 0;
      missCount.value++;
      
      addLog(`❌ ${currentKey.value} 실패! 연속 기록 초기화`, 'error');
      
      setTimeout(() => {
        keyMiss.value = false;
        generateNewKey();
      }, 300);
    };

    // 잘못된 키 입력
    const handleWrongKey = (key) => {
      addLog(`⚠️ 잘못된 키: ${key} (정답: ${currentKey.value})`, 'warning');
    };

    // 게임 시작
    const startGame = () => {
      setDifficulty();
      gameState.value = 'playing';
      score.value = 0;
      level.value = 1;
      combo.value = 0;
      maxCombo.value = 0;
      hitCount.value = 0;
      missCount.value = 0;
      gameLogs.value = [];
      
      addLog('🎮 게임 시작!', 'info');
      
      setTimeout(() => {
        if (gameArea.value) {
          gameArea.value.focus();
        }
        generateNewKey();
      }, 100);
      
      // 게임 종료 타이머 (60초)
      gameTimer = setTimeout(() => {
        endGame();
      }, 60000);
    };

    // 게임 종료
    const endGame = () => {
      clearInterval(keyTimer);
      clearTimeout(gameTimer);
      gameState.value = 'gameOver';
      addLog('🏁 게임 종료!', 'info');
    };

    // 게임 리셋
    const resetGame = () => {
      gameState.value = 'ready';
      score.value = 0;
      level.value = 1;
      combo.value = 0;
      maxCombo.value = 0;
      hitCount.value = 0;
      missCount.value = 0;
      timeLeft.value = 0;
      currentKey.value = '';
      gameLogs.value = [];
    };

    // 컴포넌트 정리
    onUnmounted(() => {
      clearInterval(keyTimer);
      clearTimeout(gameTimer);
    });

    return {
      gameState,
      score,
      level,
      timeLeft,
      maxTime,
      combo,
      maxCombo,
      difficulty,
      currentKey,
      keyHit,
      keyMiss,
      gameLogs,
      gameArea,
      accuracy,
      rank,
      startGame,
      resetGame,
      handleKeyPress
    }
  }
}
</script>

<style scoped>
.game-container {
  max-width: 800px;
  margin: 20px auto;
  padding: 20px;
  font-family: 'Arial', sans-serif;
}

.game-status {
  display: flex;
  justify-content: space-around;
  background: #f8f9fa;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.status-item {
  text-align: center;
}

.status-item .label {
  display: block;
  font-size: 12px;
  color: #666;
  margin-bottom: 5px;
}

.status-item .value {
  font-size: 20px;
  font-weight: bold;
  color: #2c3e50;
}

.game-area {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 40px;
  border-radius: 12px;
  text-align: center;
  outline: none;
  margin-bottom: 20px;
}

.instruction {
  margin-bottom: 30px;
}

.current-key {
  font-size: 24px;
  font-weight: bold;
  margin-top: 10px;
}

.target-display {
  margin: 30px 0;
}

.key-target {
  display: inline-block;
  width: 100px;
  height: 100px;
  background: rgba(255, 255, 255, 0.2);
  border: 3px solid white;
  border-radius: 12px;
  line-height: 94px;
  font-size: 48px;
  font-weight: bold;
  transition: all 0.3s ease;
}

.key-target.hit {
  background: #27ae60;
  transform: scale(1.1);
}

.key-target.miss {
  background: #e74c3c;
  animation: shake 0.5s ease-in-out;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-10px); }
  75% { transform: translateX(10px); }
}

.progress-bar {
  width: 100%;
  height: 8px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 4px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: #27ae60;
  transition: width 0.1s linear;
}

.menu-screen {
  text-align: center;
  padding: 40px;
  background: #f8f9fa;
  border-radius: 12px;
  margin-bottom: 20px;
}

.difficulty {
  margin: 20px 0;
}

.difficulty label {
  display: block;
  margin-bottom: 10px;
  font-weight: bold;
}

.difficulty select {
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.start-btn, .restart-btn {
  background: #42b883;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 6px;
  font-size: 16px;
  cursor: pointer;
  transition: background 0.3s ease;
}

.start-btn:hover, .restart-btn:hover {
  background: #369870;
}

.final-stats {
  margin: 20px 0;
}

.final-stats p {
  margin: 10px 0;
  font-size: 16px;
}

.rank {
  margin: 20px 0;
}

.rank-text {
  font-size: 24px;
  font-weight: bold;
  color: #f39c12;
}

.game-log {
  background: #f8f9fa;
  padding: 15px;
  border-radius: 8px;
}

.game-log h4 {
  margin-top: 0;
  color: #495057;
}

.game-log ul {
  list-style: none;
  padding: 0;
  max-height: 200px;
  overflow-y: auto;
}

.game-log li {
  padding: 5px 8px;
  margin: 2px 0;
  border-radius: 3px;
  font-size: 12px;
  font-family: monospace;
}

.game-log li.success {
  background: #d4edda;
  color: #155724;
}

.game-log li.error {
  background: #f8d7da;
  color: #721c24;
}

.game-log li.warning {
  background: #fff3cd;
  color: #856404;
}

.game-log li.level {
  background: #cce5ff;
  color: #004085;
  font-weight: bold;
}

.game-log li.info {
  background: #e2e3e5;
  color: #383d41;
}
</style>
```

## 🔥 핵심 포인트

1. **이벤트 수식어가 React보다 직관적**
   - `.prevent`, `.stop` 등으로 간단하게 제어
   - 조합 키도 쉽게 처리 (`.ctrl.enter`)

2. **마우스 버튼별 처리가 간편**
   - `.left`, `.right`, `.middle`로 명확하게 구분
   - 컨텍스트 메뉴 방지도 간단

3. **키보드 이벤트가 더 명확**
   - 특정 키를 직접 지정 (`.enter`, `.esc`)
   - `.exact`로 정확한 조합 처리

4. **이벤트 처리 코드가 더 적음**
   - React의 복잡한 if문 대신 수식어 활용
   - 가독성과 유지보수성 향상

## ➡️ 다음 단계

이벤트 핸들링을 마스터했다면 [1-5. 컴포넌트 기초](./05-components-basic.md)로 넘어가서 Vue 컴포넌트의 강력한 기능들을 알아보세요!

---

*Vue의 이벤트 수식어는 정말 편리해요! React에서 복잡하게 처리해야 하는 것들을 한 줄로 해결할 수 있거든요. 특히 키보드 이벤트 처리는 Vue가 훨씬 직관적입니다! 🎮* 