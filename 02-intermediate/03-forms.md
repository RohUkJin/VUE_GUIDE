# 2-3. 폼 처리 (Forms)

## 📝 Vue vs React: 폼 처리와 유효성 검사

Vue는 React보다 더 간단하고 강력한 폼 처리 기능을 제공합니다. 특히 `v-model`의 양방향 바인딩은 폼 개발을 훨씬 편하게 만들어줘요!

## 🔧 1. 기본 폼 처리

### React 방식
```jsx
import { useState } from 'react';

function UserForm({ onSubmit }) {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    age: '',
    gender: '',
    interests: [],
    newsletter: false,
    comments: ''
  });
  
  const [errors, setErrors] = useState({});
  
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    
    if (type === 'checkbox') {
      if (name === 'interests') {
        setFormData(prev => ({
          ...prev,
          interests: checked 
            ? [...prev.interests, value]
            : prev.interests.filter(item => item !== value)
        }));
      } else {
        setFormData(prev => ({ ...prev, [name]: checked }));
      }
    } else {
      setFormData(prev => ({ ...prev, [name]: value }));
    }
  };
  
  const validate = () => {
    const newErrors = {};
    
    if (!formData.name.trim()) {
      newErrors.name = '이름은 필수입니다';
    }
    
    if (!formData.email.trim()) {
      newErrors.email = '이메일은 필수입니다';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = '유효한 이메일을 입력하세요';
    }
    
    if (!formData.age || formData.age < 1 || formData.age > 120) {
      newErrors.age = '유효한 나이를 입력하세요';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (validate()) {
      onSubmit(formData);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>이름:</label>
        <input
          type="text"
          name="name"
          value={formData.name}
          onChange={handleChange}
        />
        {errors.name && <span className="error">{errors.name}</span>}
      </div>
      
      <div>
        <label>이메일:</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <label>나이:</label>
        <input
          type="number"
          name="age"
          value={formData.age}
          onChange={handleChange}
        />
        {errors.age && <span className="error">{errors.age}</span>}
      </div>
      
      <button type="submit">제출</button>
    </form>
  );
}
```

### Vue 방식
```vue
<template>
  <form @submit.prevent="handleSubmit" class="user-form">
    <div class="form-group">
      <label for="name">이름:</label>
      <input
        id="name"
        v-model="form.name"
        type="text"
        :class="{ error: errors.name }"
        @blur="validateField('name')"
      />
      <span v-if="errors.name" class="error-message">{{ errors.name }}</span>
    </div>
    
    <div class="form-group">
      <label for="email">이메일:</label>
      <input
        id="email"
        v-model="form.email"
        type="email"
        :class="{ error: errors.email }"
        @blur="validateField('email')"
      />
      <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
    </div>
    
    <div class="form-group">
      <label for="age">나이:</label>
      <input
        id="age"
        v-model.number="form.age"
        type="number"
        min="1"
        max="120"
        :class="{ error: errors.age }"
        @blur="validateField('age')"
      />
      <span v-if="errors.age" class="error-message">{{ errors.age }}</span>
    </div>
    
    <div class="form-group">
      <label>성별:</label>
      <div class="radio-group">
        <label>
          <input type="radio" v-model="form.gender" value="male" />
          남성
        </label>
        <label>
          <input type="radio" v-model="form.gender" value="female" />
          여성
        </label>
        <label>
          <input type="radio" v-model="form.gender" value="other" />
          기타
        </label>
      </div>
    </div>
    
    <div class="form-group">
      <label>관심사:</label>
      <div class="checkbox-group">
        <label v-for="interest in availableInterests" :key="interest">
          <input type="checkbox" v-model="form.interests" :value="interest" />
          {{ interest }}
        </label>
      </div>
    </div>
    
    <div class="form-group">
      <label>
        <input type="checkbox" v-model="form.newsletter" />
        뉴스레터 구독
      </label>
    </div>
    
    <div class="form-group">
      <label for="comments">추가 의견:</label>
      <textarea
        id="comments"
        v-model="form.comments"
        rows="4"
        placeholder="의견을 입력하세요..."
      ></textarea>
    </div>
    
    <div class="form-actions">
      <button type="button" @click="resetForm">초기화</button>
      <button type="submit" :disabled="!isFormValid">제출</button>
    </div>
  </form>
</template>

<script>
export default {
  data() {
    return {
      form: {
        name: '',
        email: '',
        age: null,
        gender: '',
        interests: [],
        newsletter: false,
        comments: ''
      },
      errors: {},
      availableInterests: ['개발', '디자인', '마케팅', '기획', '영업']
    }
  },
  
  computed: {
    isFormValid() {
      return Object.keys(this.errors).length === 0 && 
             this.form.name && 
             this.form.email && 
             this.form.age;
    }
  },
  
  methods: {
    validateField(fieldName) {
      const newErrors = { ...this.errors };
      delete newErrors[fieldName]; // 기존 에러 제거
      
      switch (fieldName) {
        case 'name':
          if (!this.form.name.trim()) {
            newErrors.name = '이름은 필수입니다';
          }
          break;
          
        case 'email':
          if (!this.form.email.trim()) {
            newErrors.email = '이메일은 필수입니다';
          } else if (!/\S+@\S+\.\S+/.test(this.form.email)) {
            newErrors.email = '유효한 이메일을 입력하세요';
          }
          break;
          
        case 'age':
          if (!this.form.age || this.form.age < 1 || this.form.age > 120) {
            newErrors.age = '유효한 나이를 입력하세요';
          }
          break;
      }
      
      this.errors = newErrors;
    },
    
    validateAll() {
      this.validateField('name');
      this.validateField('email');
      this.validateField('age');
      return this.isFormValid;
    },
    
    handleSubmit() {
      if (this.validateAll()) {
        this.$emit('submit', { ...this.form });
        this.resetForm();
      }
    },
    
    resetForm() {
      this.form = {
        name: '',
        email: '',
        age: null,
        gender: '',
        interests: [],
        newsletter: false,
        comments: ''
      };
      this.errors = {};
    }
  }
}
</script>

<style scoped>
.user-form {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-group input,
.form-group textarea,
.form-group select {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.form-group input.error {
  border-color: #ff6b6b;
  background-color: #fff5f5;
}

.error-message {
  color: #ff6b6b;
  font-size: 12px;
  margin-top: 5px;
  display: block;
}

.radio-group,
.checkbox-group {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
}

.radio-group label,
.checkbox-group label {
  display: flex;
  align-items: center;
  gap: 5px;
  font-weight: normal;
}

.form-actions {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
  margin-top: 30px;
}

.form-actions button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.form-actions button[type="submit"] {
  background-color: #42b883;
  color: white;
}

.form-actions button[type="submit"]:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.form-actions button[type="button"] {
  background-color: #6c757d;
  color: white;
}
</style>
```

## 🛡️ 2. 고급 유효성 검사

```vue
<!-- AdvancedForm.vue -->
<template>
  <form @submit.prevent="handleSubmit" class="advanced-form">
    <div class="form-section">
      <h3>📋 기본 정보</h3>
      
      <!-- 실시간 유효성 검사 -->
      <FormField
        v-model="form.username"
        label="사용자명"
        :rules="usernameRules"
        @validation="updateFieldValidation('username', $event)"
      />
      
      <!-- 비동기 유효성 검사 -->
      <FormField
        v-model="form.email"
        label="이메일"
        :rules="emailRules"
        :async-validator="checkEmailExists"
        @validation="updateFieldValidation('email', $event)"
      />
      
      <!-- 조건부 필수 필드 -->
      <FormField
        v-model="form.password"
        label="비밀번호"
        type="password"
        :rules="passwordRules"
        :required="isNewUser"
        @validation="updateFieldValidation('password', $event)"
      />
      
      <FormField
        v-if="form.password"
        v-model="form.passwordConfirm"
        label="비밀번호 확인"
        type="password"
        :rules="passwordConfirmRules"
        @validation="updateFieldValidation('passwordConfirm', $event)"
      />
    </div>
    
    <div class="form-section">
      <h3>👤 개인 정보</h3>
      
      <!-- 날짜 유효성 검사 -->
      <FormField
        v-model="form.birthDate"
        label="생년월일"
        type="date"
        :rules="birthDateRules"
        @validation="updateFieldValidation('birthDate', $event)"
      />
      
      <!-- 파일 업로드 -->
      <FileUpload
        v-model="form.profileImage"
        label="프로필 이미지"
        accept="image/*"
        :max-size="5 * 1024 * 1024"
        @validation="updateFieldValidation('profileImage', $event)"
      />
      
      <!-- 동적 필드 -->
      <div class="dynamic-fields">
        <h4>📞 연락처</h4>
        <div
          v-for="(contact, index) in form.contacts"
          :key="index"
          class="contact-item"
        >
          <select v-model="contact.type">
            <option value="phone">전화</option>
            <option value="email">이메일</option>
            <option value="address">주소</option>
          </select>
          <input v-model="contact.value" :placeholder="getContactPlaceholder(contact.type)" />
          <button type="button" @click="removeContact(index)">삭제</button>
        </div>
        <button type="button" @click="addContact">연락처 추가</button>
      </div>
    </div>
    
    <!-- 폼 요약 -->
    <div class="form-summary">
      <h4>📊 폼 상태</h4>
      <p>유효성 검사: {{ validationSummary }}</p>
      <p>변경사항: {{ hasChanges ? '있음' : '없음' }}</p>
    </div>
    
    <div class="form-actions">
      <button type="button" @click="resetForm" :disabled="!hasChanges">
        초기화
      </button>
      <button type="button" @click="saveAsDraft">
        임시저장
      </button>
      <button type="submit" :disabled="!canSubmit">
        {{ isSubmitting ? '제출 중...' : '제출' }}
      </button>
    </div>
  </form>
</template>

<script>
import FormField from './FormField.vue'
import FileUpload from './FileUpload.vue'

export default {
  components: { FormField, FileUpload },
  
  data() {
    return {
      form: {
        username: '',
        email: '',
        password: '',
        passwordConfirm: '',
        birthDate: '',
        profileImage: null,
        contacts: [
          { type: 'phone', value: '' }
        ]
      },
      
      originalForm: {},
      fieldValidations: {},
      isSubmitting: false,
      isNewUser: true
    }
  },
  
  computed: {
    usernameRules() {
      return [
        { required: true, message: '사용자명은 필수입니다' },
        { min: 3, message: '최소 3글자 이상이어야 합니다' },
        { max: 20, message: '최대 20글자까지 가능합니다' },
        { pattern: /^[a-zA-Z0-9_]+$/, message: '영문, 숫자, 언더스코어만 사용 가능합니다' }
      ];
    },
    
    emailRules() {
      return [
        { required: true, message: '이메일은 필수입니다' },
        { type: 'email', message: '유효한 이메일 형식이 아닙니다' }
      ];
    },
    
    passwordRules() {
      return [
        { required: this.isNewUser, message: '비밀번호는 필수입니다' },
        { min: 8, message: '최소 8글자 이상이어야 합니다' },
        { 
          validator: (value) => {
            if (!value) return true;
            return /(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value);
          }, 
          message: '대문자, 소문자, 숫자를 모두 포함해야 합니다' 
        }
      ];
    },
    
    passwordConfirmRules() {
      return [
        { required: !!this.form.password, message: '비밀번호 확인은 필수입니다' },
        { 
          validator: (value) => value === this.form.password, 
          message: '비밀번호가 일치하지 않습니다' 
        }
      ];
    },
    
    birthDateRules() {
      return [
        { required: true, message: '생년월일은 필수입니다' },
        {
          validator: (value) => {
            if (!value) return true;
            const date = new Date(value);
            const now = new Date();
            const age = now.getFullYear() - date.getFullYear();
            return age >= 14 && age <= 100;
          },
          message: '14세 이상 100세 이하만 가입 가능합니다'
        }
      ];
    },
    
    isFormValid() {
      return Object.values(this.fieldValidations).every(field => field.valid);
    },
    
    hasChanges() {
      return JSON.stringify(this.form) !== JSON.stringify(this.originalForm);
    },
    
    canSubmit() {
      return this.isFormValid && this.hasChanges && !this.isSubmitting;
    },
    
    validationSummary() {
      const total = Object.keys(this.fieldValidations).length;
      const valid = Object.values(this.fieldValidations).filter(f => f.valid).length;
      return `${valid}/${total} 필드 유효`;
    }
  },
  
  created() {
    this.originalForm = JSON.parse(JSON.stringify(this.form));
  },
  
  methods: {
    async checkEmailExists(email) {
      if (!email) return true;
      
      // API 호출 시뮬레이션
      await new Promise(resolve => setTimeout(resolve, 500));
      
      // 예시: 특정 이메일은 이미 존재한다고 가정
      const existingEmails = ['test@example.com', 'admin@example.com'];
      return !existingEmails.includes(email);
    },
    
    updateFieldValidation(fieldName, validation) {
      this.fieldValidations = {
        ...this.fieldValidations,
        [fieldName]: validation
      };
    },
    
    addContact() {
      this.form.contacts.push({ type: 'phone', value: '' });
    },
    
    removeContact(index) {
      this.form.contacts.splice(index, 1);
    },
    
    getContactPlaceholder(type) {
      const placeholders = {
        phone: '010-1234-5678',
        email: 'contact@example.com',
        address: '서울시 강남구...'
      };
      return placeholders[type] || '';
    },
    
    async handleSubmit() {
      if (!this.canSubmit) return;
      
      this.isSubmitting = true;
      
      try {
        // API 호출 시뮬레이션
        await new Promise(resolve => setTimeout(resolve, 2000));
        
        this.$emit('submit', { ...this.form });
        alert('제출이 완료되었습니다!');
        this.resetForm();
        
      } catch (error) {
        alert('제출 중 오류가 발생했습니다.');
      } finally {
        this.isSubmitting = false;
      }
    },
    
    saveAsDraft() {
      localStorage.setItem('formDraft', JSON.stringify(this.form));
      alert('임시저장되었습니다.');
    },
    
    loadDraft() {
      const draft = localStorage.getItem('formDraft');
      if (draft) {
        this.form = JSON.parse(draft);
      }
    },
    
    resetForm() {
      this.form = JSON.parse(JSON.stringify(this.originalForm));
      this.fieldValidations = {};
      localStorage.removeItem('formDraft');
    }
  },
  
  mounted() {
    // 페이지 로드 시 임시저장된 데이터 복원
    this.loadDraft();
  }
}
</script>

<style scoped>
.advanced-form {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.form-section {
  margin-bottom: 30px;
  padding: 20px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
}

.form-section h3 {
  margin-top: 0;
  color: #333;
  border-bottom: 2px solid #42b883;
  padding-bottom: 10px;
}

.dynamic-fields {
  margin-top: 20px;
}

.contact-item {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
  align-items: center;
}

.contact-item select {
  width: 120px;
}

.contact-item input {
  flex: 1;
}

.contact-item button {
  background: #ff6b6b;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 4px;
  cursor: pointer;
}

.form-summary {
  background: #f8f9fa;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.form-actions {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
}

.form-actions button {
  padding: 12px 24px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.2s;
}

.form-actions button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.form-actions button[type="submit"] {
  background: #42b883;
  color: white;
}

.form-actions button[type="button"] {
  background: #6c757d;
  color: white;
}
</style>
```

## 🧩 3. 재사용 가능한 폼 컴포넌트

```vue
<!-- FormField.vue -->
<template>
  <div class="form-field">
    <label v-if="label" :for="fieldId">
      {{ label }}
      <span v-if="required" class="required">*</span>
    </label>
    
    <div class="input-wrapper">
      <input
        :id="fieldId"
        :type="type"
        :value="modelValue"
        :placeholder="placeholder"
        :disabled="disabled"
        :class="fieldClasses"
        @input="handleInput"
        @blur="handleBlur"
        @focus="handleFocus"
      />
      
      <div v-if="isValidating" class="loading-spinner">⏳</div>
      <div v-else-if="isValid" class="validation-icon success">✅</div>
      <div v-else-if="hasError" class="validation-icon error">❌</div>
    </div>
    
    <div v-if="hasError" class="error-messages">
      <div v-for="error in errors" :key="error" class="error-message">
        {{ error }}
      </div>
    </div>
    
    <div v-if="hint" class="hint">{{ hint }}</div>
  </div>
</template>

<script>
export default {
  props: {
    modelValue: {
      type: [String, Number],
      default: ''
    },
    label: String,
    type: {
      type: String,
      default: 'text'
    },
    placeholder: String,
    hint: String,
    required: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    },
    rules: {
      type: Array,
      default: () => []
    },
    asyncValidator: {
      type: Function,
      default: null
    }
  },
  
  data() {
    return {
      errors: [],
      isValidating: false,
      isTouched: false,
      validationTimeout: null
    }
  },
  
  computed: {
    fieldId() {
      return `field-${Math.random().toString(36).substr(2, 9)}`;
    },
    
    hasError() {
      return this.isTouched && this.errors.length > 0;
    },
    
    isValid() {
      return this.isTouched && this.errors.length === 0 && !this.isValidating;
    },
    
    fieldClasses() {
      return {
        'form-input': true,
        'error': this.hasError,
        'success': this.isValid,
        'validating': this.isValidating
      };
    }
  },
  
  watch: {
    modelValue: {
      handler() {
        this.debounceValidation();
      },
      immediate: true
    }
  },
  
  methods: {
    handleInput(event) {
      this.$emit('update:modelValue', event.target.value);
    },
    
    handleBlur() {
      this.isTouched = true;
      this.validate();
    },
    
    handleFocus() {
      this.errors = [];
    },
    
    debounceValidation() {
      if (this.validationTimeout) {
        clearTimeout(this.validationTimeout);
      }
      
      this.validationTimeout = setTimeout(() => {
        if (this.isTouched) {
          this.validate();
        }
      }, 300);
    },
    
    async validate() {
      this.errors = [];
      const value = this.modelValue;
      
      // 동기 규칙 검사
      for (const rule of this.rules) {
        if (rule.required && (!value || value.toString().trim() === '')) {
          this.errors.push(rule.message || '필수 항목입니다');
          break;
        }
        
        if (value && rule.min && value.length < rule.min) {
          this.errors.push(rule.message || `최소 ${rule.min}글자 이상이어야 합니다`);
        }
        
        if (value && rule.max && value.length > rule.max) {
          this.errors.push(rule.message || `최대 ${rule.max}글자까지 가능합니다`);
        }
        
        if (value && rule.pattern && !rule.pattern.test(value)) {
          this.errors.push(rule.message || '형식이 올바르지 않습니다');
        }
        
        if (value && rule.type === 'email' && !/\S+@\S+\.\S+/.test(value)) {
          this.errors.push(rule.message || '유효한 이메일을 입력하세요');
        }
        
        if (rule.validator && !rule.validator(value)) {
          this.errors.push(rule.message || '유효하지 않은 값입니다');
        }
      }
      
      // 비동기 검사
      if (this.errors.length === 0 && this.asyncValidator && value) {
        this.isValidating = true;
        
        try {
          const isValid = await this.asyncValidator(value);
          if (!isValid) {
            this.errors.push('이미 사용 중인 값입니다');
          }
        } catch (error) {
          this.errors.push('검증 중 오류가 발생했습니다');
        } finally {
          this.isValidating = false;
        }
      }
      
      // 검증 결과 전달
      this.$emit('validation', {
        valid: this.errors.length === 0 && !this.isValidating,
        errors: this.errors
      });
    }
  }
}
</script>

<style scoped>
.form-field {
  margin-bottom: 20px;
}

.form-field label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #333;
}

.required {
  color: #ff6b6b;
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.form-input {
  width: 100%;
  padding: 10px;
  border: 2px solid #e0e0e0;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.form-input:focus {
  outline: none;
  border-color: #42b883;
  box-shadow: 0 0 0 3px rgba(66, 184, 131, 0.1);
}

.form-input.error {
  border-color: #ff6b6b;
}

.form-input.success {
  border-color: #51cf66;
}

.form-input.validating {
  border-color: #ffd43b;
}

.validation-icon {
  position: absolute;
  right: 10px;
  font-size: 16px;
}

.loading-spinner {
  position: absolute;
  right: 10px;
}

.error-messages {
  margin-top: 5px;
}

.error-message {
  color: #ff6b6b;
  font-size: 12px;
  margin-bottom: 2px;
}

.hint {
  margin-top: 5px;
  font-size: 12px;
  color: #666;
}
</style>
```

## 🎯 4. 실습 예제: 다단계 폼

```vue
<!-- MultiStepForm.vue -->
<template>
  <div class="multi-step-form">
    <!-- 진행률 표시 -->
    <div class="progress-bar">
      <div 
        v-for="(step, index) in steps" 
        :key="index"
        :class="['step', { 
          active: currentStep === index,
          completed: currentStep > index,
          error: stepErrors[index]
        }]"
      >
        <div class="step-number">
          <span v-if="currentStep > index">✓</span>
          <span v-else>{{ index + 1 }}</span>
        </div>
        <div class="step-title">{{ step.title }}</div>
      </div>
    </div>
    
    <!-- 현재 단계 내용 -->
    <div class="step-content">
      <component
        :is="currentStepComponent"
        v-model="formData"
        @validation="handleStepValidation"
        @next="nextStep"
        @prev="prevStep"
      />
    </div>
    
    <!-- 네비게이션 -->
    <div class="form-navigation">
      <button 
        v-if="currentStep > 0"
        @click="prevStep"
        class="btn-secondary"
      >
        이전
      </button>
      
      <div class="nav-info">
        {{ currentStep + 1 }} / {{ steps.length }}
      </div>
      
      <button
        v-if="currentStep < steps.length - 1"
        @click="nextStep"
        :disabled="!isCurrentStepValid"
        class="btn-primary"
      >
        다음
      </button>
      
      <button
        v-else
        @click="submitForm"
        :disabled="!canSubmit"
        class="btn-submit"
      >
        {{ isSubmitting ? '제출 중...' : '제출' }}
      </button>
    </div>
  </div>
</template>

<script>
import PersonalInfo from './steps/PersonalInfo.vue'
import ContactInfo from './steps/ContactInfo.vue'
import Preferences from './steps/Preferences.vue'
import Review from './steps/Review.vue'

export default {
  components: {
    PersonalInfo,
    ContactInfo,
    Preferences,
    Review
  },
  
  data() {
    return {
      currentStep: 0,
      formData: {
        // 1단계: 개인정보
        personal: {
          firstName: '',
          lastName: '',
          birthDate: '',
          gender: ''
        },
        // 2단계: 연락처
        contact: {
          email: '',
          phone: '',
          address: {
            street: '',
            city: '',
            zipCode: ''
          }
        },
        // 3단계: 선호사항
        preferences: {
          newsletter: false,
          notifications: [],
          theme: 'light'
        }
      },
      stepValidations: {},
      stepErrors: {},
      isSubmitting: false,
      
      steps: [
        { title: '개인정보', component: 'PersonalInfo' },
        { title: '연락처', component: 'ContactInfo' },
        { title: '선호사항', component: 'Preferences' },
        { title: '검토', component: 'Review' }
      ]
    }
  },
  
  computed: {
    currentStepComponent() {
      return this.steps[this.currentStep].component;
    },
    
    isCurrentStepValid() {
      return this.stepValidations[this.currentStep] !== false;
    },
    
    canSubmit() {
      return Object.keys(this.stepValidations).length === this.steps.length &&
             Object.values(this.stepValidations).every(valid => valid) &&
             !this.isSubmitting;
    }
  },
  
  methods: {
    handleStepValidation(isValid) {
      this.stepValidations = {
        ...this.stepValidations,
        [this.currentStep]: isValid
      };
      
      if (!isValid) {
        this.stepErrors = {
          ...this.stepErrors,
          [this.currentStep]: true
        };
      } else {
        const newErrors = { ...this.stepErrors };
        delete newErrors[this.currentStep];
        this.stepErrors = newErrors;
      }
    },
    
    nextStep() {
      if (this.isCurrentStepValid && this.currentStep < this.steps.length - 1) {
        this.currentStep++;
        this.saveProgress();
      }
    },
    
    prevStep() {
      if (this.currentStep > 0) {
        this.currentStep--;
      }
    },
    
    async submitForm() {
      if (!this.canSubmit) return;
      
      this.isSubmitting = true;
      
      try {
        // API 호출 시뮬레이션
        await new Promise(resolve => setTimeout(resolve, 2000));
        
        alert('등록이 완료되었습니다!');
        this.clearProgress();
        this.resetForm();
        
      } catch (error) {
        alert('등록 중 오류가 발생했습니다.');
      } finally {
        this.isSubmitting = false;
      }
    },
    
    saveProgress() {
      const progress = {
        currentStep: this.currentStep,
        formData: this.formData,
        stepValidations: this.stepValidations
      };
      localStorage.setItem('multiStepFormProgress', JSON.stringify(progress));
    },
    
    loadProgress() {
      const saved = localStorage.getItem('multiStepFormProgress');
      if (saved) {
        const progress = JSON.parse(saved);
        this.currentStep = progress.currentStep || 0;
        this.formData = progress.formData || this.formData;
        this.stepValidations = progress.stepValidations || {};
      }
    },
    
    clearProgress() {
      localStorage.removeItem('multiStepFormProgress');
    },
    
    resetForm() {
      this.currentStep = 0;
      this.formData = {
        personal: { firstName: '', lastName: '', birthDate: '', gender: '' },
        contact: { email: '', phone: '', address: { street: '', city: '', zipCode: '' } },
        preferences: { newsletter: false, notifications: [], theme: 'light' }
      };
      this.stepValidations = {};
      this.stepErrors = {};
    }
  },
  
  mounted() {
    this.loadProgress();
  },
  
  beforeUnmount() {
    this.saveProgress();
  }
}
</script>

<style scoped>
.multi-step-form {
  max-width: 700px;
  margin: 0 auto;
  padding: 20px;
}

.progress-bar {
  display: flex;
  justify-content: space-between;
  margin-bottom: 40px;
  position: relative;
}

.progress-bar::before {
  content: '';
  position: absolute;
  top: 20px;
  left: 0;
  right: 0;
  height: 2px;
  background: #e0e0e0;
  z-index: 1;
}

.step {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  z-index: 2;
}

.step-number {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #e0e0e0;
  color: #666;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  margin-bottom: 8px;
  transition: all 0.3s ease;
}

.step.active .step-number {
  background: #42b883;
  color: white;
}

.step.completed .step-number {
  background: #51cf66;
  color: white;
}

.step.error .step-number {
  background: #ff6b6b;
  color: white;
}

.step-title {
  font-size: 12px;
  color: #666;
  text-align: center;
}

.step.active .step-title {
  color: #42b883;
  font-weight: bold;
}

.step-content {
  min-height: 400px;
  margin-bottom: 30px;
}

.form-navigation {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 0;
  border-top: 1px solid #e0e0e0;
}

.nav-info {
  font-size: 14px;
  color: #666;
}

.btn-primary,
.btn-secondary,
.btn-submit {
  padding: 12px 24px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.2s;
}

.btn-primary {
  background: #42b883;
  color: white;
}

.btn-secondary {
  background: #6c757d;
  color: white;
}

.btn-submit {
  background: #28a745;
  color: white;
}

.btn-primary:disabled,
.btn-submit:disabled {
  background: #ccc;
  cursor: not-allowed;
}
</style>
```

## 🎯 실습 과제

1. 위의 다단계 폼을 직접 만들어보세요
2. 파일 업로드와 미리보기 기능을 추가해보세요
3. 폼 데이터를 로컬스토리지에 자동 저장하는 기능을 구현해보세요
4. 커스텀 유효성 검사 규칙을 만들어보세요
5. 동적으로 필드를 추가/제거하는 기능을 구현해보세요

## 🔗 다음 단계

다음 챕터에서는 Vue의 라이프사이클 훅에 대해 알아보겠습니다!

---

💡 **팁**: Vue의 v-model과 폼 처리는 React보다 훨씬 간단하고 직관적입니다. 특히 양방향 바인딩으로 인해 코드가 크게 줄어들어요! 