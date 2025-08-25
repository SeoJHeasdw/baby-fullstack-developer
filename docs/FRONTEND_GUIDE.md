
# 🖥️ 프론트엔드 개발 가이드 (Vue.js + TypeScript + Tailwind CSS)

> Vue.js 3를 이용한 HR 교육자료 관리 웹 인터페이스 개발 가이드

## 📋 목차
1. [개발 환경 설정](#개발-환경-설정)
2. [프로젝트 생성 및 기본 설정](#프로젝트-생성-및-기본-설정)
3. [핵심 컴포넌트 구현](#핵심-컴포넌트-구현)
4. [API 연동](#api-연동)
5. [CursorAI 활용 팁](#cursorai-활용-팁)
6. [빌드 및 배포](#빌드-및-배포)

---

## 🛠️ 개발 환경 설정

### Vue 프로젝트 생성
```bash
# 프로젝트 루트에서
cd hr-education-system

# Vue 3 프로젝트 생성
npm create vue@latest frontend

# 설정 선택
✅ TypeScript
✅ Router  
✅ Pinia
❌ 나머지 옵션들

cd frontend
npm install

# 추가 패키지 설치
npm install -D tailwindcss postcss autoprefixer @tailwindcss/forms
npm install axios dayjs lucide-vue-next @vueuse/core
npx tailwindcss init -p
```

---

## ⚙️ 프로젝트 생성 및 기본 설정

### Tailwind CSS 설정
```javascript
// tailwind.config.js
export default {
  content: ["./index.html", "./src/**/*.{vue,js,ts,jsx,tsx}"],
  theme: {
    extend: {
      colors: {
        primary: {
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
        }
      }
    },
  },
  plugins: [require('@tailwindcss/forms')],
}
```

```css
/* src/style.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer components {
  .btn-primary {
    @apply bg-primary-600 hover:bg-primary-700 text-white font-medium py-2 px-4 rounded-lg transition-colors;
  }
  
  .btn-secondary {
    @apply bg-gray-200 hover:bg-gray-300 text-gray-800 font-medium py-2 px-4 rounded-lg transition-colors;
  }
  
  .card {
    @apply bg-white rounded-lg shadow-md border border-gray-200 p-6;
  }
}
```

### 환경 변수 설정
```env
# .env.local
VITE_API_BASE_URL=http://localhost:8000
VITE_APP_TITLE=HR 교육자료 관리 시스템
```

### 기본 타입 정의
```typescript
// src/types/material.ts
export interface Material {
  id: number
  title: string
  description?: string
  category_id?: number
  file_name: string
  original_file_name: string
  file_size: number
  mime_type: string
  tags?: string[]
  is_featured: boolean
  view_count: number
  download_count: number
  created_at: string
}

// src/types/category.ts
export interface Category {
  id: number
  name: string
  description?: string
  icon?: string
  color?: string
  is_active: boolean
}
```

---

## 🌐 API 연동

### API 서비스 설정
```typescript
// src/services/api.ts
import axios from 'axios'

const api = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 30000,
})

export const materialAPI = {
  // 교육자료 목록 조회 (로그인 불필요)
  getAll: (params?: any) => api.get('/api/materials', { params }),
  
  // 교육자료 상세 조회
  getById: (id: number) => api.get(`/api/materials/${id}`),
  
  // 교육자료 다운로드
  download: async (id: number, filename: string) => {
    const response = await api.get(`/api/materials/${id}/download`, {
      responseType: 'blob'
    })
    
    // 파일 다운로드 처리
    const url = window.URL.createObjectURL(new Blob([response.data]))
    const link = document.createElement('a')
    link.href = url
    link.download = filename
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
    window.URL.revokeObjectURL(url)
  },
  
  // 관리자 기능 (향후 SSO 연동)
  create: (formData: FormData) => api.post('/api/materials', formData, {
    headers: { 'Content-Type': 'multipart/form-data' }
  }),
}

export const categoryAPI = {
  getAll: () => api.get('/api/categories')
}

export default api
```

### Pinia 스토어
```typescript
// src/stores/materials.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'
import { materialAPI } from '@/services/api'
import type { Material } from '@/types/material'

export const useMaterialsStore = defineStore('materials', () => {
  const materials = ref<Material[]>([])
  const loading = ref(false)
  const error = ref<string | null>(null)

  const fetchMaterials = async (params?: any) => {
    try {
      loading.value = true
      error.value = null
      const response = await materialAPI.getAll(params)
      materials.value = response.data
    } catch (err: any) {
      error.value = err.message || '자료를 불러오는데 실패했습니다.'
    } finally {
      loading.value = false
    }
  }

  const downloadMaterial = async (material: Material) => {
    try {
      await materialAPI.download(material.id, material.original_file_name)
      // 다운로드 카운트 업데이트 (로컬)
      const index = materials.value.findIndex(m => m.id === material.id)
      if (index !== -1) {
        materials.value[index].download_count += 1
      }
    } catch (err: any) {
      error.value = '다운로드에 실패했습니다.'
      throw err
    }
  }

  return { materials, loading, error, fetchMaterials, downloadMaterial }
})
```

---

## 🧩 핵심 컴포넌트 구현

### 메인 레이아웃
```vue
<!-- src/App.vue -->
<template>
  <div id="app">
    <nav class="bg-white shadow-sm border-b">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between h-16">
          <div class="flex items-center">
            <router-link to="/" class="text-xl font-bold text-gray-900">
              {{ appTitle }}
            </router-link>
          </div>
          <div class="flex items-center space-x-4">
            <router-link to="/" class="text-gray-700 hover:text-gray-900">홈</router-link>
            <router-link to="/materials" class="text-gray-700 hover:text-gray-900">교육자료</router-link>
            <router-link to="/admin" class="btn-primary">관리자</router-link>
          </div>
        </div>
      </div>
    </nav>
    
    <main>
      <router-view />
    </main>
  </div>
</template>

<script setup lang="ts">
const appTitle = import.meta.env.VITE_APP_TITLE
</script>
```

### 교육자료 목록 페이지
```vue
<!-- src/views/MaterialsView.vue -->
<template>
  <div class="max-w-7xl mx-auto px-4 py-8">
    <h1 class="text-3xl font-bold mb-8">교육자료</h1>
    
    <!-- 검색 및 필터 -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-8">
      <input
        v-model="searchQuery"
        @input="handleSearch"
        placeholder="자료 검색..."
        class="border rounded-lg px-4 py-2"
      />
      <select v-model="selectedCategory" @change="handleFilter" class="border rounded-lg px-4 py-2">
        <option value="">전체 카테고리</option>
        <option v-for="category in categories" :key="category.id" :value="category.id">
          {{ category.name }}
        </option>
      </select>
      <button @click="clearFilters" class="btn-secondary">필터 초기화</button>
    </div>

    <!-- 로딩 상태 -->
    <div v-if="loading" class="text-center py-12">
      <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-primary-600 mx-auto"></div>
      <p class="mt-2 text-gray-600">자료를 불러오는 중...</p>
    </div>

    <!-- 자료 목록 -->
    <div v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      <MaterialCard
        v-for="material in materials"
        :key="material.id"
        :material="material"
        @download="handleDownload"
      />
    </div>

    <!-- 빈 상태 -->
    <div v-if="!loading && materials.length === 0" class="text-center py-12">
      <p class="text-gray-500">검색 결과가 없습니다.</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import { useMaterialsStore } from '@/stores/materials'
import { useCategoriesStore } from '@/stores/categories'
import MaterialCard from '@/components/MaterialCard.vue'

const materialsStore = useMaterialsStore()
const categoriesStore = useCategoriesStore()

const searchQuery = ref('')
const selectedCategory = ref('')

const materials = computed(() => materialsStore.materials)
const categories = computed(() => categoriesStore.categories)
const loading = computed(() => materialsStore.loading)

const handleSearch = () => {
  fetchMaterials()
}

const handleFilter = () => {
  fetchMaterials()
}

const clearFilters = () => {
  searchQuery.value = ''
  selectedCategory.value = ''
  fetchMaterials()
}

const fetchMaterials = () => {
  const params: any = {}
  if (searchQuery.value) params.search = searchQuery.value
  if (selectedCategory.value) params.category_id = selectedCategory.value
  materialsStore.fetchMaterials(params)
}

const handleDownload = async (material: any) => {
  try {
    await materialsStore.downloadMaterial(material)
  } catch (error) {
    alert('다운로드에 실패했습니다.')
  }
}

onMounted(async () => {
  await Promise.all([
    materialsStore.fetchMaterials(),
    categoriesStore.fetchCategories()
  ])
})
</script>
```

### 교육자료 카드 컴포넌트
```vue
<!-- src/components/MaterialCard.vue -->
<template>
  <div class="card hover:shadow-lg transition-shadow">
    <div class="mb-4">
      <h3 class="text-lg font-semibold text-gray-900 mb-2">{{ material.title }}</h3>
      <p class="text-sm text-gray-600">{{ material.description || '설명이 없습니다.' }}</p>
    </div>
    
    <div class="flex items-center justify-between text-xs text-gray-500 mb-4">
      <span>{{ formatFileSize(material.file_size) }}</span>
      <span>{{ formatDate(material.created_at) }}</span>
    </div>
    
    <div class="flex items-center justify-between">
      <div class="text-sm text-gray-500">
        <span class="mr-4">👁 {{ material.view_count }}</span>
        <span>⬇ {{ material.download_count }}</span>
      </div>
      
      <button @click="$emit('download', material)" class="btn-primary">
        다운로드
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { Material } from '@/types/material'
import { formatFileSize, formatDate } from '@/utils/format'

defineProps<{
  material: Material
}>()

defineEmits<{
  download: [material: Material]
}>()
</script>
```

### 유틸리티 함수
```typescript
// src/utils/format.ts
export const formatFileSize = (bytes: number): string => {
  if (bytes === 0) return '0 Bytes'
  const k = 1024
  const sizes = ['Bytes', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
}

export const formatDate = (date: string): string => {
  return new Date(date).toLocaleDateString('ko-KR')
}
```

### 라우터 설정
```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      name: 'Home',
      component: () => import('@/views/HomeView.vue')
    },
    {
      path: '/materials',
      name: 'Materials', 
      component: () => import('@/views/MaterialsView.vue')
    },
    {
      path: '/admin',
      name: 'Admin',
      component: () => import('@/views/AdminView.vue')
    }
  ]
})

export default router
```

---

## 🤖 CursorAI 활용 팁

### 효과적인 프롬프트 예시

#### 컴포넌트 생성
```
"Vue 3 + TypeScript로 파일 업로드 드래그앤드롭 컴포넌트 만들어줘.
Tailwind CSS 사용하고 업로드 진행률도 표시해야 해."
```

#### 스타일링 요청
```
"이 카드 컴포넌트를 더 현대적으로 만들어줘:
[컴포넌트 코드]
호버 효과와 그림자 추가하고 반응형으로 만들어줘."
```

#### 기능 구현
```
"Vue 3에서 무한 스크롤 기능을 구현해줘. 
교육자료 목록에서 스크롤 끝에 도달하면 자동으로 더 불러오기."
```

---

## 🚀 빌드 및 배포

### 개발 서버 실행
```bash
# 프론트엔드 서버 실행
npm run dev  # http://localhost:5173

# 백엔드와 함께 테스트할 때
# 터미널 1: 백엔드 실행 (포트 8000)
# 터미널 2: 프론트엔드 실행 (포트 5173)
```

### 프로덕션 빌드
```bash
# 빌드
npm run build

# 빌드 결과 미리보기
npm run preview
```

### Docker 배포 (선택사항)
```dockerfile
# Dockerfile
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## 📝 다음 단계

1. ✅ **개발 서버 실행 확인**
   ```bash
   npm run dev
   ```

2. ✅ **백엔드 연동 테스트**
   - 교육자료 목록 조회
   - 다운로드 기능 확인

3. 🔄 **관리자 기능 추가** (필요시)
   - 파일 업로드 컴포넌트
   - 관리자 인증 (SSO 연동 준비)

---

**프론트엔드 개발 완료! 🎉**  
이제 백엔드와 연결하여 전체 시스템을 테스트해보세요!
