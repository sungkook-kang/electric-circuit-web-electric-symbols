# Electric Symbols 프로젝트 설정 가이드

## 📋 프로젝트 개요

**목적:** JointJS+를 활용한 전기 회로 심볼 개발
**주요 목표:** 릴레이, 접촉기, 스위치 등 전기 부품 심볼 컴포넌트 제작

---

## 🚀 Phase 1: 기본 환경 설정

### 1.1 의존성 설치

```bash
# 프로젝트 디렉토리로 이동
cd electric-symbols

# 기본 패키지 설치
npm install

# TypeScript 및 개발 도구 설치
npm install -D typescript @types/node vue-tsc

# Tailwind CSS 설치
npm install -D @nuxtjs/tailwindcss
```

### 1.2 JointJS+ 라이브러리 설치

```bash
# 상위 디렉토리의 JointJS+ 라이센스 파일 복사
copy ..\client\joint-plus.tgz .\

# JointJS+ 설치
npm install ./joint-plus.tgz
```

---

## 🔧 Phase 2: Nuxt 설정

### 2.1 nuxt.config.ts 업데이트

**파일:** `nuxt.config.ts`

```typescript
// https://nuxt.com/docs/api/configuration/nuxt-config
export default defineNuxtConfig({
  compatibilityDate: '2025-01-01',

  devtools: {
    enabled: true
  },

  modules: [
    '@nuxtjs/tailwindcss',
  ],

  typescript: {
    strict: true,
    typeCheck: true,
  },

  css: [
    '~/assets/css/main.css'
  ],

  vite: {
    optimizeDeps: {
      include: ['@joint/plus'],
    },
    ssr: {
      noExternal: ['@joint/plus'],
    },
  },

  future: {
    compatibilityVersion: 4,
  },
})
```

### 2.2 Tailwind 설정

**파일:** `tailwind.config.js` (새로 생성)

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./app/components/**/*.{js,vue,ts}",
    "./app/layouts/**/*.vue",
    "./app/pages/**/*.vue",
    "./app/plugins/**/*.{js,ts}",
    "./app/app.vue",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

---

## 📁 Phase 3: 프로젝트 구조 생성

### 3.1 디렉토리 생성

```bash
# app 디렉토리 내부에 생성
cd app

mkdir components
mkdir components\symbols
mkdir composables
mkdir utils
mkdir types
mkdir assets
mkdir assets\css
mkdir pages

cd ..
```

### 3.2 메인 CSS 파일 생성

**파일:** `app/assets/css/main.css`

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* JointJS 캔버스 기본 스타일 */
.joint-paper {
  @apply bg-gray-50;
}

.joint-element {
  cursor: move;
}

/* 커스텀 유틸리티 클래스 */
@layer utilities {
  .canvas-container {
    @apply w-full h-screen overflow-hidden relative;
  }

  .symbol-editor {
    @apply flex flex-col h-screen;
  }

  .toolbar {
    @apply bg-white border-b border-gray-200 p-4;
  }

  .canvas-wrapper {
    @apply flex-1 relative overflow-hidden;
  }
}
```

---

## 📝 Phase 4: TypeScript 타입 정의

### 4.1 JointJS+ 타입 정의

**파일:** `app/types/joint.d.ts`

```typescript
declare module '@joint/plus' {
  import * as joint from '@joint/core'

  export = joint
  export as namespace joint

  export namespace dia {
    class Graph extends joint.dia.Graph {}
    class Paper extends joint.dia.Paper {}
    class Element extends joint.dia.Element {}
    class Link extends joint.dia.Link {}
  }

  export namespace shapes {
    namespace standard {
      class Rectangle extends dia.Element {}
      class Circle extends dia.Element {}
      class Ellipse extends dia.Element {}
      class Path extends dia.Element {}
      class Polygon extends dia.Element {}
      class Polyline extends dia.Element {}
      class Image extends dia.Element {}
      class Link extends dia.Link {}
    }
  }

  export namespace g {
    class Point {
      constructor(x?: number, y?: number)
      x: number
      y: number
    }
    class Rect {
      constructor(x?: number, y?: number, width?: number, height?: number)
      x: number
      y: number
      width: number
      height: number
    }
  }

  export namespace util {}
}
```

### 4.2 전기 심볼 타입 정의

**파일:** `app/types/symbols.ts`

```typescript
/**
 * 전기 심볼 기본 인터페이스
 */
export interface ElectricalSymbol {
  /** 고유 식별자 */
  id: string

  /** 심볼 타입 */
  type: SymbolType

  /** 심볼 이름 */
  name: string

  /** 카테고리 (예: 제어, 보호, 전원 등) */
  category: string

  /** 설명 */
  description?: string

  /** SVG 경로 데이터 */
  svg?: string

  /** 커스텀 속성 */
  properties?: Record<string, any>
}

/**
 * 지원하는 심볼 타입
 */
export type SymbolType =
  | 'relay'           // 릴레이
  | 'contactor'       // 접촉기 (마그네틱)
  | 'switch'          // 스위치
  | 'push_button'     // 푸시버튼
  | 'selector_switch' // 셀렉터 스위치
  | 'fuse'            // 퓨즈
  | 'breaker'         // 차단기 (NFB, MCCB 등)
  | 'terminal'        // 단자대
  | 'lamp'            // 표시등
  | 'motor'           // 모터
  | 'transformer'     // 변압기
  | 'capacitor'       // 커패시터
  | 'resistor'        // 저항
  | 'inductor'        // 인덕터

/**
 * 심볼 라이브러리
 */
export interface SymbolLibrary {
  version: string
  symbols: ElectricalSymbol[]
  metadata?: {
    author?: string
    createdAt?: string
    updatedAt?: string
  }
}

/**
 * 심볼 설정
 */
export interface SymbolConfig {
  /** 폭 */
  width: number

  /** 높이 */
  height: number

  /** 포트 설정 */
  ports?: PortConfig[]

  /** JointJS 속성 */
  attrs?: Record<string, any>

  /** 기본 위치 */
  position?: { x: number; y: number }
}

/**
 * 포트 설정 (연결 지점)
 */
export interface PortConfig {
  /** 포트 ID */
  id: string

  /** 포트 그룹 (입력/출력) */
  group: 'in' | 'out'

  /** 포트 위치 */
  position: { x: number; y: number }

  /** 포트 레이블 */
  label?: string

  /** 포트 속성 */
  attrs?: Record<string, any>
}

/**
 * 릴레이 전용 속성
 */
export interface RelayProperties {
  /** 코일 전압 (예: AC220V, DC24V) */
  coilVoltage: string

  /** 접점 수 (예: '2a2b' = 2개의 a접점, 2개의 b접점) */
  contacts: string

  /** 정격 전류 */
  ratedCurrent: string

  /** 제조사 */
  manufacturer?: string

  /** 모델명 */
  model?: string
}

/**
 * 접촉기(마그네틱) 전용 속성
 */
export interface ContactorProperties {
  /** 코일 전압 */
  coilVoltage: string

  /** 주접점 수 (예: 3극, 4극) */
  mainContacts: number

  /** 보조접점 (예: '2a2b') */
  auxiliaryContacts?: string

  /** 정격 전류 */
  ratedCurrent: string

  /** 정격 용량 (kW) */
  ratedCapacity?: string

  /** 차단 용량 */
  breakingCapacity?: string
}
```

---

## 🎨 Phase 5: 기본 페이지 생성

### 5.1 메인 앱 파일

**파일:** `app/app.vue`

```vue
<template>
  <div>
    <NuxtPage />
  </div>
</template>

<script setup lang="ts">
useHead({
  title: 'Electric Symbols - JointJS+',
  meta: [
    { charset: 'utf-8' },
    { name: 'viewport', content: 'width=device-width, initial-scale=1' },
    {
      name: 'description',
      content: 'Professional electrical circuit symbols development with JointJS+'
    }
  ],
  link: [
    { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
  ]
})
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
               'Helvetica Neue', Arial, sans-serif;
}
</style>
```

### 5.2 인덱스 페이지

**파일:** `app/pages/index.vue`

```vue
<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100">
    <div class="container mx-auto px-4 py-16">
      <!-- 헤더 -->
      <header class="text-center mb-16">
        <h1 class="text-5xl font-bold text-gray-800 mb-4">
          전기 심볼 개발 프로젝트
        </h1>
        <p class="text-xl text-gray-600">
          JointJS+를 활용한 전문 전기 회로 심볼 컴포넌트 개발
        </p>
      </header>

      <!-- 메뉴 카드 -->
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 max-w-6xl mx-auto">

        <!-- 캔버스 테스트 -->
        <NuxtLink
          to="/canvas-test"
          class="group block p-6 bg-white rounded-xl shadow-md hover:shadow-xl transition-all duration-300 transform hover:-translate-y-1"
        >
          <div class="flex items-center mb-4">
            <div class="w-12 h-12 bg-blue-500 rounded-lg flex items-center justify-center mr-4">
              <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2" />
              </svg>
            </div>
            <h2 class="text-2xl font-semibold text-gray-800 group-hover:text-blue-600 transition">
              캔버스 테스트
            </h2>
          </div>
          <p class="text-gray-600">
            JointJS+ 기본 캔버스 동작 확인 및 그래프 렌더링 테스트
          </p>
        </NuxtLink>

        <!-- 릴레이 심볼 -->
        <NuxtLink
          to="/relay-symbol"
          class="group block p-6 bg-white rounded-xl shadow-md hover:shadow-xl transition-all duration-300 transform hover:-translate-y-1"
        >
          <div class="flex items-center mb-4">
            <div class="w-12 h-12 bg-green-500 rounded-lg flex items-center justify-center mr-4">
              <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z" />
              </svg>
            </div>
            <h2 class="text-2xl font-semibold text-gray-800 group-hover:text-green-600 transition">
              릴레이 심볼
            </h2>
          </div>
          <p class="text-gray-600">
            릴레이 전기 심볼 커스텀 컴포넌트 개발 및 상호작용 테스트
          </p>
        </NuxtLink>

        <!-- 접촉기 심볼 -->
        <NuxtLink
          to="/contactor-symbol"
          class="group block p-6 bg-white rounded-xl shadow-md hover:shadow-xl transition-all duration-300 transform hover:-translate-y-1"
        >
          <div class="flex items-center mb-4">
            <div class="w-12 h-12 bg-purple-500 rounded-lg flex items-center justify-center mr-4">
              <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 9l3 3-3 3m5 0h3M5 20h14a2 2 0 002-2V6a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
              </svg>
            </div>
            <h2 class="text-2xl font-semibold text-gray-800 group-hover:text-purple-600 transition">
              접촉기 심볼
            </h2>
          </div>
          <p class="text-gray-600">
            마그네틱 접촉기(MC) 심볼 개발 및 주접점/보조접점 구현
          </p>
        </NuxtLink>

        <!-- 심볼 라이브러리 -->
        <NuxtLink
          to="/symbol-library"
          class="group block p-6 bg-white rounded-xl shadow-md hover:shadow-xl transition-all duration-300 transform hover:-translate-y-1"
        >
          <div class="flex items-center mb-4">
            <div class="w-12 h-12 bg-yellow-500 rounded-lg flex items-center justify-center mr-4">
              <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10" />
              </svg>
            </div>
            <h2 class="text-2xl font-semibold text-gray-800 group-hover:text-yellow-600 transition">
              심볼 라이브러리
            </h2>
          </div>
          <p class="text-gray-600">
            전체 전기 심볼 라이브러리 관리 및 카탈로그 뷰
          </p>
        </NuxtLink>

      </div>

      <!-- 푸터 정보 -->
      <footer class="mt-16 text-center text-gray-600">
        <p class="mb-2">
          <span class="font-semibold">Tech Stack:</span>
          Nuxt 4 + TypeScript + JointJS+ + Tailwind CSS
        </p>
        <p class="text-sm">
          Professional electrical circuit design tools
        </p>
      </footer>
    </div>
  </div>
</template>

<script setup lang="ts">
</script>
```

### 5.3 캔버스 테스트 페이지

**파일:** `app/pages/canvas-test.vue`

```vue
<template>
  <div class="symbol-editor">
    <!-- 상단 툴바 -->
    <div class="toolbar">
      <div class="flex items-center justify-between">
        <div class="flex items-center space-x-4">
          <NuxtLink
            to="/"
            class="px-4 py-2 bg-gray-200 hover:bg-gray-300 rounded-lg transition"
          >
            ← 뒤로가기
          </NuxtLink>
          <h1 class="text-2xl font-bold text-gray-800">
            JointJS+ 캔버스 테스트
          </h1>
        </div>
        <div class="flex items-center space-x-2">
          <button
            @click="addRectangle"
            class="px-4 py-2 bg-blue-500 hover:bg-blue-600 text-white rounded-lg transition"
          >
            사각형 추가
          </button>
          <button
            @click="addCircle"
            class="px-4 py-2 bg-green-500 hover:bg-green-600 text-white rounded-lg transition"
          >
            원 추가
          </button>
          <button
            @click="clearCanvas"
            class="px-4 py-2 bg-red-500 hover:bg-red-600 text-white rounded-lg transition"
          >
            전체 삭제
          </button>
        </div>
      </div>
    </div>

    <!-- 캔버스 영역 -->
    <div class="canvas-wrapper">
      <div ref="canvasRef" class="w-full h-full"></div>
    </div>

    <!-- 상태 표시 -->
    <div class="bg-gray-100 border-t border-gray-200 px-4 py-2 text-sm text-gray-600">
      <span class="mr-4">요소 수: {{ elementCount }}</span>
      <span>그리드: 10px</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { dia, shapes } from '@joint/plus'

const canvasRef = ref<HTMLElement | null>(null)
const elementCount = ref(0)

let graph: dia.Graph | null = null
let paper: dia.Paper | null = null

// 사각형 추가
const addRectangle = () => {
  if (!graph) return

  const rect = new shapes.standard.Rectangle({
    position: {
      x: Math.random() * 400 + 100,
      y: Math.random() * 300 + 100
    },
    size: { width: 100, height: 60 },
    attrs: {
      body: {
        fill: '#3b82f6',
        stroke: '#1e40af',
        strokeWidth: 2,
        rx: 5,
        ry: 5
      },
      label: {
        text: 'Rectangle',
        fill: '#ffffff',
        fontSize: 14,
        fontWeight: 'bold'
      }
    }
  })

  graph.addCell(rect)
  updateElementCount()
}

// 원 추가
const addCircle = () => {
  if (!graph) return

  const circle = new shapes.standard.Circle({
    position: {
      x: Math.random() * 400 + 100,
      y: Math.random() * 300 + 100
    },
    size: { width: 80, height: 80 },
    attrs: {
      body: {
        fill: '#10b981',
        stroke: '#059669',
        strokeWidth: 2
      },
      label: {
        text: 'Circle',
        fill: '#ffffff',
        fontSize: 14,
        fontWeight: 'bold'
      }
    }
  })

  graph.addCell(circle)
  updateElementCount()
}

// 캔버스 초기화
const clearCanvas = () => {
  if (!graph) return
  graph.clear()
  updateElementCount()
}

// 요소 수 업데이트
const updateElementCount = () => {
  if (!graph) return
  elementCount.value = graph.getElements().length
}

onMounted(() => {
  if (!canvasRef.value) return

  // 그래프 생성
  graph = new dia.Graph({}, { cellNamespace: shapes })

  // 페이퍼 생성
  paper = new dia.Paper({
    el: canvasRef.value,
    model: graph,
    width: '100%',
    height: '100%',
    gridSize: 10,
    drawGrid: {
      name: 'mesh',
      args: {
        color: '#e5e7eb',
      }
    },
    background: {
      color: '#f9fafb'
    },
    cellViewNamespace: shapes,
    interactive: true,
    defaultLink: () => new shapes.standard.Link(),
  })

  // 그래프 변경 이벤트 리스너
  graph.on('add remove', updateElementCount)

  // 초기 요소 추가 (데모용)
  addRectangle()
})

onUnmounted(() => {
  // 정리
  if (paper) {
    paper.remove()
    paper = null
  }
  if (graph) {
    graph.clear()
    graph = null
  }
})
</script>

<style scoped>
.symbol-editor {
  @apply flex flex-col h-screen;
}

.toolbar {
  @apply bg-white border-b border-gray-200 p-4 shadow-sm;
}

.canvas-wrapper {
  @apply flex-1 relative overflow-hidden bg-gray-50;
}
</style>
```

---

## ✅ 검증 체크리스트

### 환경 설정 확인
- [ ] Node.js 및 npm이 설치되어 있는가?
- [ ] 프로젝트 디렉토리가 생성되었는가?
- [ ] `npm install` 실행 완료
- [ ] JointJS+ 라이센스 파일이 복사되었는가?

### 설정 파일 확인
- [ ] `nuxt.config.ts` 업데이트 완료
- [ ] `tailwind.config.js` 생성 완료
- [ ] `app/assets/css/main.css` 생성 완료
- [ ] TypeScript 타입 정의 파일 생성 완료

### 페이지 생성 확인
- [ ] `app/app.vue` 업데이트 완료
- [ ] `app/pages/index.vue` 생성 완료
- [ ] `app/pages/canvas-test.vue` 생성 완료

### 개발 서버 실행
```bash
npm run dev
```

- [ ] 개발 서버가 시작되는가? (http://localhost:3000)
- [ ] 인덱스 페이지가 표시되는가?
- [ ] 캔버스 테스트 페이지에서 도형이 표시되는가?
- [ ] TypeScript 컴파일 오류가 없는가?
- [ ] Tailwind CSS 스타일이 적용되는가?

---

## 🔍 트러블슈팅

### JointJS+ import 오류
```
Cannot find module '@joint/plus'
```
**해결책:**
```bash
rm -rf node_modules .nuxt
npm install
```

### Vite 최적화 오류
**증상:** 개발 서버 시작 시 JointJS+ 관련 오류

**해결책:** `nuxt.config.ts`에 다음 추가
```typescript
vite: {
  optimizeDeps: {
    include: ['@joint/plus'],
    force: true
  }
}
```

### TypeScript 타입 오류
**증상:** `dia.Graph` 또는 `shapes.standard.Rectangle` 타입을 찾을 수 없음

**해결책:** `app/types/joint.d.ts` 파일 확인 및 재시작
```bash
npm run dev
```

---

## 📚 다음 단계

이 설정 가이드를 완료한 후:

1. **릴레이 심볼 개발**: `RELAY_SYMBOL_GUIDE.md` 참조
2. **접촉기 심볼 개발**: `CONTACTOR_SYMBOL_GUIDE.md` 참조
3. **심볼 라이브러리 구축**: 다양한 전기 부품 심볼 추가

---

## 🔗 참고 자료

- [Nuxt 4 문서](https://nuxt.com/docs)
- [JointJS+ 문서](https://resources.jointjs.com/docs/jointjs)
- [Tailwind CSS 문서](https://tailwindcss.com/docs)
- [TypeScript 핸드북](https://www.typescriptlang.org/docs/handbook/intro.html)
