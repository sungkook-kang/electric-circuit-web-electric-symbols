# 릴레이 심볼 개발 가이드

## 📋 개요

이 문서는 JointJS+를 사용하여 전기 회로도에 사용되는 릴레이(Relay) 심볼을 개발하는 방법을 설명합니다.

---

## 🎯 학습 목표

1. JointJS+ 커스텀 Shape 생성 방법 이해
2. SVG를 활용한 전기 심볼 디자인
3. 포트(연결점) 설정 및 관리
4. 상호작용 기능 구현 (드래그, 클릭, 속성 변경)
5. 릴레이 특성에 맞는 동적 속성 관리

---

## 📐 릴레이 심볼 구조

### 표준 IEC 릴레이 심볼

릴레이는 크게 두 부분으로 구성됩니다:

1. **코일 부분**: 전자석 역할을 하는 코일 (사각형으로 표현)
2. **접점 부분**:
   - **a접점 (NO, Normally Open)**: 평상시 열림 → 코일 여자 시 닫힘
   - **b접점 (NC, Normally Closed)**: 평상시 닫힘 → 코일 여자 시 열림
   - **c접점 (전환 접점)**: a접점과 b접점의 조합

```
    코일 부분 (Coil)              접점 부분 (Contacts)

         A1                          NO (a접점)
          │                          │ ╱
          │                          │╱    ← 평상시 열림 (Normally Open)
    ┌─────┴─────┐                   │
    │           │
    │     K     │                   NC (b접점)
    │           │                   ─┬─    ← 평상시 닫힘 (Normally Closed)
    └─────┬─────┘                    │
          │
          A2                        COM (공통)
                                     │

    [설명]
    - A1, A2: 코일 단자 (전원 연결)
    - K: 릴레이 코일 표시
    - NO: a접점 (평상시 열림, 코일 여자 시 닫힘)
    - NC: b접점 (평상시 닫힘, 코일 여자 시 열림)
    - COM: 공통 단자

    [동작]
    코일(A1-A2)에 전원 인가 → 전자석 작동 → 접점 전환
```

---

## 🛠️ Step 1: 기본 릴레이 컴포넌트 생성

### 1.1 릴레이 Shape 정의

**파일:** `app/components/symbols/RelayShape.ts`

```typescript
import { dia, shapes, util } from '@joint/plus'
import type { RelayProperties } from '~/types/symbols'

/**
 * 릴레이 커스텀 Shape
 */
export class RelayShape extends dia.Element {
  defaults() {
    return {
      ...super.defaults,
      type: 'electrical.Relay',
      size: { width: 80, height: 120 },
      attrs: {
        // 외곽선 (보이지 않음)
        body: {
          refWidth: '100%',
          refHeight: '100%',
          strokeWidth: 0,
          fill: 'transparent'
        },
        // 코일 사각형
        coil: {
          x: 10,
          y: 10,
          width: 60,
          height: 40,
          fill: '#ffffff',
          stroke: '#000000',
          strokeWidth: 2,
          rx: 2,
          ry: 2
        },
        // 코일 레이블
        coilLabel: {
          x: 40,
          y: 30,
          text: 'K',
          fontSize: 16,
          fontWeight: 'bold',
          textAnchor: 'middle',
          textVerticalAnchor: 'middle',
          fill: '#000000'
        },
        // A1 단자 (코일 상단)
        coilTerminalA1: {
          x: 40,
          y: 10,
          r: 3,
          fill: '#000000'
        },
        // A2 단자 (코일 하단)
        coilTerminalA2: {
          x: 40,
          y: 50,
          r: 3,
          fill: '#000000'
        },
        // A1 레이블
        labelA1: {
          x: 40,
          y: 5,
          text: 'A1',
          fontSize: 10,
          textAnchor: 'middle',
          fill: '#666666'
        },
        // A2 레이블
        labelA2: {
          x: 40,
          y: 58,
          text: 'A2',
          fontSize: 10,
          textAnchor: 'middle',
          fill: '#666666'
        },
        // NO 접점 (a접점) - 열린 상태
        contactNO: {
          d: 'M 25 75 L 35 85 M 35 85 L 55 85',
          stroke: '#000000',
          strokeWidth: 2,
          fill: 'none'
        },
        // NC 접점 (b접점) - 닫힌 상태
        contactNC: {
          d: 'M 25 95 L 55 95',
          stroke: '#000000',
          strokeWidth: 2,
          fill: 'none'
        },
        // 공통 단자선
        commonLine: {
          d: 'M 40 60 L 40 110',
          stroke: '#000000',
          strokeWidth: 1.5,
          fill: 'none'
        },
        // NO 라벨
        labelNO: {
          x: 65,
          y: 80,
          text: 'NO',
          fontSize: 9,
          fill: '#666666'
        },
        // NC 라벨
        labelNC: {
          x: 65,
          y: 95,
          text: 'NC',
          fontSize: 9,
          fill: '#666666'
        }
      },
      // 포트 정의
      ports: {
        groups: {
          'coil': {
            position: 'absolute',
            attrs: {
              circle: {
                r: 4,
                fill: '#ffffff',
                stroke: '#000000',
                strokeWidth: 2,
                magnet: true
              }
            }
          },
          'contact': {
            position: 'absolute',
            attrs: {
              circle: {
                r: 4,
                fill: '#ffffff',
                stroke: '#000000',
                strokeWidth: 2,
                magnet: true
              }
            }
          }
        },
        items: [
          // 코일 포트
          {
            id: 'A1',
            group: 'coil',
            args: { x: 40, y: 10 },
            label: { text: 'A1', position: { name: 'top', args: { y: -10 } } }
          },
          {
            id: 'A2',
            group: 'coil',
            args: { x: 40, y: 50 },
            label: { text: 'A2', position: { name: 'bottom', args: { y: 10 } } }
          },
          // 접점 포트
          {
            id: 'NO',
            group: 'contact',
            args: { x: 25, y: 80 },
            label: { text: 'NO', position: { name: 'left', args: { x: -10 } } }
          },
          {
            id: 'NC',
            group: 'contact',
            args: { x: 25, y: 95 },
            label: { text: 'NC', position: { name: 'left', args: { x: -10 } } }
          },
          {
            id: 'COM',
            group: 'contact',
            args: { x: 40, y: 110 },
            label: { text: 'COM', position: { name: 'bottom', args: { y: 10 } } }
          }
        ]
      },
      // 커스텀 속성
      relayProperties: {
        coilVoltage: 'AC220V',
        contacts: '1a1b',
        ratedCurrent: '5A',
        manufacturer: '',
        model: ''
      } as RelayProperties
    }
  }

  markup = [
    {
      tagName: 'rect',
      selector: 'body'
    },
    {
      tagName: 'rect',
      selector: 'coil'
    },
    {
      tagName: 'text',
      selector: 'coilLabel'
    },
    {
      tagName: 'circle',
      selector: 'coilTerminalA1'
    },
    {
      tagName: 'circle',
      selector: 'coilTerminalA2'
    },
    {
      tagName: 'text',
      selector: 'labelA1'
    },
    {
      tagName: 'text',
      selector: 'labelA2'
    },
    {
      tagName: 'path',
      selector: 'contactNO'
    },
    {
      tagName: 'path',
      selector: 'contactNC'
    },
    {
      tagName: 'path',
      selector: 'commonLine'
    },
    {
      tagName: 'text',
      selector: 'labelNO'
    },
    {
      tagName: 'text',
      selector: 'labelNC'
    }
  ]

  /**
   * 코일 전압 설정
   */
  setCoilVoltage(voltage: string) {
    const props = this.get('relayProperties') as RelayProperties
    props.coilVoltage = voltage
    this.set('relayProperties', props)
  }

  /**
   * 접점 구성 변경
   */
  setContacts(contacts: string) {
    const props = this.get('relayProperties') as RelayProperties
    props.contacts = contacts
    this.set('relayProperties', props)
  }

  /**
   * 릴레이 활성화 상태 표시 (시뮬레이션용)
   */
  setEnergized(energized: boolean) {
    if (energized) {
      this.attr('coil/fill', '#4ade80') // 녹색 - 여자 상태
      this.attr('contactNO/stroke', '#ef4444') // 빨간색 - 닫힘
      this.attr('contactNC/stroke', '#94a3b8') // 회색 - 열림
    } else {
      this.attr('coil/fill', '#ffffff') // 흰색 - 평상시
      this.attr('contactNO/stroke', '#000000') // 검정 - 열림
      this.attr('contactNC/stroke', '#000000') // 검정 - 닫힘
    }
  }
}

// 네임스페이스에 등록
Object.assign(shapes, {
  electrical: {
    ...shapes.electrical,
    Relay: RelayShape
  }
})
```

---

## 🎨 Step 2: 릴레이 컴포넌트 Vue 통합

### 2.1 릴레이 캔버스 컴포넌트

**파일:** `app/components/symbols/RelayCanvas.vue`

```vue
<template>
  <div class="relay-canvas-wrapper">
    <!-- 캔버스 -->
    <div ref="canvasRef" class="relay-canvas"></div>

    <!-- 컨트롤 패널 -->
    <div class="control-panel">
      <h3 class="text-lg font-semibold mb-4">릴레이 제어</h3>

      <!-- 코일 전압 선택 -->
      <div class="mb-4">
        <label class="block text-sm font-medium text-gray-700 mb-2">
          코일 전압
        </label>
        <select
          v-model="coilVoltage"
          @change="updateCoilVoltage"
          class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
        >
          <option value="AC220V">AC 220V</option>
          <option value="AC110V">AC 110V</option>
          <option value="DC24V">DC 24V</option>
          <option value="DC12V">DC 12V</option>
        </select>
      </div>

      <!-- 접점 구성 -->
      <div class="mb-4">
        <label class="block text-sm font-medium text-gray-700 mb-2">
          접점 구성
        </label>
        <select
          v-model="contacts"
          @change="updateContacts"
          class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
        >
          <option value="1a">1a (NO 1개)</option>
          <option value="1b">1b (NC 1개)</option>
          <option value="1a1b">1a1b (NO 1개, NC 1개)</option>
          <option value="2a2b">2a2b (NO 2개, NC 2개)</option>
          <option value="3a3b">3a3b (NO 3개, NC 3개)</option>
        </select>
      </div>

      <!-- 시뮬레이션 버튼 -->
      <div class="mb-4">
        <button
          @click="toggleEnergized"
          :class="[
            'w-full px-4 py-3 rounded-lg font-semibold transition-all duration-200',
            isEnergized
              ? 'bg-green-500 hover:bg-green-600 text-white'
              : 'bg-gray-200 hover:bg-gray-300 text-gray-800'
          ]"
        >
          {{ isEnergized ? '코일 여자 중' : '코일 평상시' }}
        </button>
      </div>

      <!-- 정보 표시 -->
      <div class="bg-gray-50 p-4 rounded-lg">
        <h4 class="text-sm font-semibold text-gray-700 mb-2">릴레이 정보</h4>
        <div class="text-sm text-gray-600 space-y-1">
          <p><span class="font-medium">코일 전압:</span> {{ coilVoltage }}</p>
          <p><span class="font-medium">접점 구성:</span> {{ contacts }}</p>
          <p><span class="font-medium">상태:</span> {{ isEnergized ? '여자' : '평상시' }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { dia, shapes } from '@joint/plus'
import { RelayShape } from './RelayShape'

const canvasRef = ref<HTMLElement | null>(null)
const coilVoltage = ref('AC220V')
const contacts = ref('1a1b')
const isEnergized = ref(false)

let graph: dia.Graph | null = null
let paper: dia.Paper | null = null
let relayElement: RelayShape | null = null

const updateCoilVoltage = () => {
  if (relayElement) {
    relayElement.setCoilVoltage(coilVoltage.value)
  }
}

const updateContacts = () => {
  if (relayElement) {
    relayElement.setContacts(contacts.value)
  }
}

const toggleEnergized = () => {
  isEnergized.value = !isEnergized.value
  if (relayElement) {
    relayElement.setEnergized(isEnergized.value)
  }
}

onMounted(() => {
  if (!canvasRef.value) return

  // 그래프 생성
  graph = new dia.Graph({}, { cellNamespace: shapes })

  // 페이퍼 생성
  paper = new dia.Paper({
    el: canvasRef.value,
    model: graph,
    width: 600,
    height: 400,
    gridSize: 10,
    drawGrid: {
      name: 'mesh',
      args: {
        color: '#e5e7eb',
      }
    },
    background: {
      color: '#ffffff'
    },
    cellViewNamespace: shapes,
    interactive: { elementMove: true }
  })

  // 릴레이 요소 생성
  relayElement = new RelayShape()
  relayElement.position(260, 140)
  relayElement.setCoilVoltage(coilVoltage.value)
  relayElement.setContacts(contacts.value)

  graph.addCell(relayElement)
})

onUnmounted(() => {
  if (paper) {
    paper.remove()
  }
  if (graph) {
    graph.clear()
  }
})
</script>

<style scoped>
.relay-canvas-wrapper {
  @apply flex gap-4 h-full;
}

.relay-canvas {
  @apply flex-1 bg-white border-2 border-gray-200 rounded-lg;
}

.control-panel {
  @apply w-80 bg-white border-2 border-gray-200 rounded-lg p-6;
}
</style>
```

---

## 📄 Step 3: 릴레이 페이지 생성

**파일:** `app/pages/relay-symbol.vue`

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
            릴레이 심볼 개발
          </h1>
        </div>
      </div>
    </div>

    <!-- 캔버스 영역 -->
    <div class="canvas-wrapper">
      <RelayCanvas />
    </div>

    <!-- 하단 정보 -->
    <div class="bg-gray-100 border-t border-gray-200 px-4 py-2 text-sm text-gray-600">
      <span class="mr-4">릴레이 (Relay) - 전자석 코일과 접점으로 구성된 전기 스위치</span>
      <span>IEC 표준 준수</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import RelayCanvas from '~/components/symbols/RelayCanvas.vue'

useHead({
  title: '릴레이 심볼 - Electric Symbols'
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
  @apply flex-1 overflow-hidden bg-gray-50 p-4;
}
</style>
```

---

## 🧪 Step 4: 테스트 및 검증

### 4.1 개발 서버 실행

```bash
npm run dev
```

### 4.2 테스트 항목

- [ ] 릴레이 심볼이 캔버스에 정확히 렌더링되는가?
- [ ] 코일 부분과 접점 부분이 명확하게 구분되는가?
- [ ] 포트(A1, A2, NO, NC, COM)가 올바른 위치에 표시되는가?
- [ ] 드래그로 릴레이를 이동할 수 있는가?
- [ ] 코일 전압 선택이 동작하는가?
- [ ] 접점 구성 변경이 동작하는가?
- [ ] "코일 여자" 버튼 클릭 시 시각적 피드백이 표시되는가?
  - 코일이 녹색으로 변경되는가?
  - NO 접점이 빨간색(닫힘)으로 변경되는가?
  - NC 접점이 회색(열림)으로 변경되는가?

---

## 🎓 추가 학습 과제

### 과제 1: 다중 접점 릴레이

현재 구현은 1a1b (NO 1개, NC 1개)만 표시합니다.
2a2b, 3a3b 등 다중 접점을 시각적으로 표현하도록 확장하세요.

**힌트:**
```typescript
// 접점 수에 따라 동적으로 markup 생성
updateContactsDisplay(contactConfig: string) {
  // '2a2b' -> NO 2개, NC 2개
  const matches = contactConfig.match(/(\d+)a(\d+)b/)
  if (matches) {
    const noCount = parseInt(matches[1])
    const ncCount = parseInt(matches[2])
    // 동적으로 접점 생성...
  }
}
```

### 과제 2: 릴레이 간 연결

여러 개의 릴레이를 생성하고, JointJS+ Link를 사용하여 연결하세요.

**힌트:**
```typescript
// 링크 생성
const link = new shapes.standard.Link({
  source: { id: relay1.id, port: 'NO' },
  target: { id: relay2.id, port: 'A1' }
})
graph.addCell(link)
```

### 과제 3: 릴레이 애니메이션

코일이 여자될 때 접점이 서서히 움직이는 애니메이션을 추가하세요.

**힌트:**
```typescript
// transition 사용
relayElement.transition('attrs/contactNO/d', newPath, {
  duration: 300,
  timingFunction: util.timing.easeInOut
})
```

---

## 📚 참고 자료

- [JointJS+ Custom Elements](https://resources.jointjs.com/docs/jointjs/v4.0/joint.html#dia.Element)
- [JointJS+ Ports](https://resources.jointjs.com/docs/jointjs/v4.0/joint.html#dia.Element.ports)
- [IEC 60617 - 전기 심볼 표준](https://en.wikipedia.org/wiki/IEC_60617)

---

## ✅ 완료 체크리스트

- [ ] RelayShape.ts 파일 생성 및 구현
- [ ] RelayCanvas.vue 컴포넌트 생성
- [ ] relay-symbol.vue 페이지 생성
- [ ] 개발 서버에서 동작 확인
- [ ] 모든 테스트 항목 통과
- [ ] 추가 학습 과제 도전 (선택)

---

다음 단계: [접촉기 심볼 개발 가이드](./CONTACTOR_SYMBOL_GUIDE.md)
