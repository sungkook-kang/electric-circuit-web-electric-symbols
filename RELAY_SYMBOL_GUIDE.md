# 릴레이 심볼 개발 가이드

## 📋 개요

JointJS+를 사용하여 전기 회로도용 릴레이(Relay) 심볼을 SVG 기반으로 구현하는 실전 가이드입니다.

---

## 🎯 구현 목표

1. **코일 심볼**: 사각형 + 단자(A1, A2) + 레이블(K)
2. **a접점 심볼**: NO (Normally Open) - 대각선으로 열림 표현
3. **b접점 심볼**: NC (Normally Closed) - 수평선으로 닫힘 표현
4. **포트 시스템**: Wire 연결을 위한 단자 정의
5. **동적 속성**: 코일 전압, 접점 구성 변경 가능
6. **시각적 피드백**: 코일 여자 시 색상 변화

---

## 🔑 핵심 개념

### JointJS+ Shape 구조

```typescript
export class RelayShape extends dia.Element {
  defaults() {
    return {
      type: 'electrical.Relay',
      size: { width, height },
      attrs: { /* SVG 속성 정의 */ },
      ports: { /* 연결 포트 정의 */ }
    }
  }

  markup = [ /* SVG 요소 구조 */ ]
}
```

### SVG 요소로 심볼 그리기

- **rect**: 사각형 (코일 표현)
- **circle**: 원 (단자 표현)
- **text**: 텍스트 (레이블)
- **path**: 경로 (접점 라인)

---

## 🛠️ Step 1: 코일 심볼 구현

### 1.1 코일 Shape 기본 구조

**파일:** `app/components/symbols/RelayCoilShape.ts`

```typescript
import { dia, shapes } from '@joint/plus'

export class RelayCoilShape extends dia.Element {
  defaults() {
    return {
      ...super.defaults,
      type: 'electrical.RelayCoil',
      size: { width: 60, height: 60 },
      attrs: {
        // 투명 배경 (Shape 영역 정의용)
        body: {
          refWidth: '100%',
          refHeight: '100%',
          fill: 'transparent',
          stroke: 'none'
        },
        // 코일 사각형
        coilRect: {
          x: 10,
          y: 15,
          width: 40,
          height: 30,
          fill: '#ffffff',
          stroke: '#000000',
          strokeWidth: 2
        },
        // K 레이블
        label: {
          refX: '50%',
          refY: '50%',
          text: 'K',
          fontSize: 14,
          fontWeight: 'bold',
          textAnchor: 'middle',
          textVerticalAnchor: 'middle',
          fill: '#000000'
        },
        // A1 단자 (상단)
        terminalA1: {
          refX: '50%',
          y: 0,
          r: 2,
          fill: '#000000'
        },
        // A2 단자 (하단)
        terminalA2: {
          refX: '50%',
          refY: '100%',
          r: 2,
          fill: '#000000'
        }
      },
      ports: {
        groups: {
          coil: {
            position: 'absolute',
            attrs: {
              circle: {
                r: 4,
                fill: '#ffffff',
                stroke: '#000000',
                strokeWidth: 2,
                magnet: true  // 연결 가능하게 설정
              }
            }
          }
        },
        items: [
          {
            id: 'A1',
            group: 'coil',
            args: { x: 30, y: 0 }
          },
          {
            id: 'A2',
            group: 'coil',
            args: { x: 30, y: 60 }
          }
        ]
      }
    }
  }

  markup = [
    {
      tagName: 'rect',
      selector: 'body'
    },
    {
      tagName: 'rect',
      selector: 'coilRect'
    },
    {
      tagName: 'text',
      selector: 'label'
    },
    {
      tagName: 'circle',
      selector: 'terminalA1'
    },
    {
      tagName: 'circle',
      selector: 'terminalA2'
    }
  ]
}

// shapes 네임스페이스에 등록
Object.assign(shapes, {
  electrical: {
    ...shapes.electrical,
    RelayCoil: RelayCoilShape
  }
})
```

### 1.2 코일 동작 메서드 추가

```typescript
export class RelayCoilShape extends dia.Element {
  // ... defaults, markup

  /**
   * 코일 여자 상태 설정
   */
  setEnergized(energized: boolean) {
    if (energized) {
      this.attr('coilRect/fill', '#4ade80')  // 녹색
      this.attr('label/fill', '#ffffff')
    } else {
      this.attr('coilRect/fill', '#ffffff')  // 흰색
      this.attr('label/fill', '#000000')
    }
  }

  /**
   * 코일 전압 표시
   */
  setVoltage(voltage: string) {
    this.attr('label/text', `K\n${voltage}`)
  }
}
```

---

## 🛠️ Step 2: a접점 (NO) 심볼 구현

### 2.1 a접점 Shape 정의

**파일:** `app/components/symbols/RelayContactNO.ts`

```typescript
import { dia, shapes } from '@joint/plus'

/**
 * a접점 (Normally Open) 심볼
 * 평상시 열림, 대각선으로 표현
 */
export class RelayContactNO extends dia.Element {
  defaults() {
    return {
      ...super.defaults,
      type: 'electrical.RelayContactNO',
      size: { width: 50, height: 40 },
      attrs: {
        body: {
          refWidth: '100%',
          refHeight: '100%',
          fill: 'transparent',
          stroke: 'none'
        },
        // 좌측 단자 (11)
        terminal11: {
          x: 0,
          refY: '50%',
          r: 2,
          fill: '#000000'
        },
        // 우측 단자 (12)
        terminal12: {
          refX: '100%',
          refY: '50%',
          r: 2,
          fill: '#000000'
        },
        // 접점 라인 (열린 상태 - 대각선)
        contactLine: {
          d: 'M 5 20 L 20 10',  // 대각선으로 열림
          stroke: '#000000',
          strokeWidth: 2,
          strokeLinecap: 'round',
          fill: 'none'
        },
        // 우측 수평선
        rightLine: {
          d: 'M 30 20 L 45 20',
          stroke: '#000000',
          strokeWidth: 2,
          fill: 'none'
        },
        // 11 레이블
        label11: {
          x: 5,
          y: 5,
          text: '11',
          fontSize: 9,
          fill: '#666666'
        },
        // 12 레이블
        label12: {
          refX: '100%',
          y: 5,
          text: '12',
          fontSize: 9,
          textAnchor: 'end',
          fill: '#666666'
        }
      },
      ports: {
        groups: {
          contact: {
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
          { id: '11', group: 'contact', args: { x: 0, y: 20 } },
          { id: '12', group: 'contact', args: { x: 50, y: 20 } }
        ]
      }
    }
  }

  markup = [
    { tagName: 'rect', selector: 'body' },
    { tagName: 'circle', selector: 'terminal11' },
    { tagName: 'circle', selector: 'terminal12' },
    { tagName: 'path', selector: 'contactLine' },
    { tagName: 'path', selector: 'rightLine' },
    { tagName: 'text', selector: 'label11' },
    { tagName: 'text', selector: 'label12' }
  ]

  /**
   * 접점 닫힘 상태로 변경 (코일 여자 시)
   */
  setClosed(closed: boolean) {
    if (closed) {
      // 수평선으로 변경 (닫힘)
      this.attr('contactLine/d', 'M 5 20 L 30 20')
      this.attr('contactLine/stroke', '#ef4444')  // 빨간색
    } else {
      // 대각선으로 변경 (열림)
      this.attr('contactLine/d', 'M 5 20 L 20 10')
      this.attr('contactLine/stroke', '#000000')  // 검은색
    }
  }
}

Object.assign(shapes, {
  electrical: {
    ...shapes.electrical,
    RelayContactNO: RelayContactNO
  }
})
```

---

## 🛠️ Step 3: b접점 (NC) 심볼 구현

### 3.1 b접점 Shape 정의

**파일:** `app/components/symbols/RelayContactNC.ts`

```typescript
import { dia, shapes } from '@joint/plus'

/**
 * b접점 (Normally Closed) 심볼
 * 평상시 닫힘, 수평선으로 표현
 */
export class RelayContactNC extends dia.Element {
  defaults() {
    return {
      ...super.defaults,
      type: 'electrical.RelayContactNC',
      size: { width: 50, height: 40 },
      attrs: {
        body: {
          refWidth: '100%',
          refHeight: '100%',
          fill: 'transparent',
          stroke: 'none'
        },
        terminal21: {
          x: 0,
          refY: '50%',
          r: 2,
          fill: '#000000'
        },
        terminal22: {
          refX: '100%',
          refY: '50%',
          r: 2,
          fill: '#000000'
        },
        // 접점 라인 (닫힌 상태 - 수평선)
        contactLine: {
          d: 'M 5 20 L 45 20',  // 수평선으로 닫힘
          stroke: '#000000',
          strokeWidth: 2,
          strokeLinecap: 'round',
          fill: 'none'
        },
        // 닫힘 표시 (작은 원)
        closedMark: {
          cx: 25,
          cy: 20,
          r: 3,
          fill: '#000000'
        },
        label21: {
          x: 5,
          y: 5,
          text: '21',
          fontSize: 9,
          fill: '#666666'
        },
        label22: {
          refX: '100%',
          y: 5,
          text: '22',
          fontSize: 9,
          textAnchor: 'end',
          fill: '#666666'
        }
      },
      ports: {
        groups: {
          contact: {
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
          { id: '21', group: 'contact', args: { x: 0, y: 20 } },
          { id: '22', group: 'contact', args: { x: 50, y: 20 } }
        ]
      }
    }
  }

  markup = [
    { tagName: 'rect', selector: 'body' },
    { tagName: 'circle', selector: 'terminal21' },
    { tagName: 'circle', selector: 'terminal22' },
    { tagName: 'path', selector: 'contactLine' },
    { tagName: 'circle', selector: 'closedMark' },
    { tagName: 'text', selector: 'label21' },
    { tagName: 'text', selector: 'label22' }
  ]

  /**
   * 접점 열림 상태로 변경 (코일 여자 시)
   */
  setOpen(open: boolean) {
    if (open) {
      // 대각선으로 변경 (열림)
      this.attr('contactLine/d', 'M 5 20 L 20 10 M 30 20 L 45 20')
      this.attr('contactLine/stroke', '#94a3b8')  // 회색
      this.attr('closedMark/opacity', 0)  // 닫힘 표시 숨김
    } else {
      // 수평선으로 유지 (닫힘)
      this.attr('contactLine/d', 'M 5 20 L 45 20')
      this.attr('contactLine/stroke', '#000000')  // 검은색
      this.attr('closedMark/opacity', 1)  // 닫힘 표시 표시
    }
  }
}

Object.assign(shapes, {
  electrical: {
    ...shapes.electrical,
    RelayContactNC: RelayContactNC
  }
})
```

---

## 🎨 Step 4: Vue 컴포넌트 통합

### 4.1 릴레이 캔버스 컴포넌트

**파일:** `app/pages/relay-symbol.vue`

```vue
<template>
  <div class="h-screen flex flex-col">
    <!-- 툴바 -->
    <div class="bg-white border-b p-4">
      <div class="flex justify-between items-center">
        <h1 class="text-2xl font-bold">릴레이 심볼 개발</h1>
        <div class="flex gap-2">
          <button
            @click="toggleEnergized"
            :class="[
              'px-4 py-2 rounded font-semibold',
              isEnergized
                ? 'bg-green-500 text-white'
                : 'bg-gray-200 text-gray-800'
            ]"
          >
            {{ isEnergized ? '코일 ON' : '코일 OFF' }}
          </button>
        </div>
      </div>
    </div>

    <!-- 캔버스 -->
    <div class="flex-1 relative">
      <div ref="paperEl" class="w-full h-full"></div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { dia, shapes } from '@joint/plus'
import { RelayCoilShape } from '~/components/symbols/RelayCoilShape'
import { RelayContactNO } from '~/components/symbols/RelayContactNO'
import { RelayContactNC } from '~/components/symbols/RelayContactNC'

const paperEl = ref<HTMLElement>()
const isEnergized = ref(false)

let graph: dia.Graph
let paper: dia.Paper
let coil: RelayCoilShape
let contactNO: RelayContactNO
let contactNC: RelayContactNC

onMounted(() => {
  // 그래프 생성
  graph = new dia.Graph({}, { cellNamespace: shapes })

  // 페이퍼 생성
  paper = new dia.Paper({
    el: paperEl.value,
    model: graph,
    width: '100%',
    height: '100%',
    gridSize: 10,
    drawGrid: { name: 'mesh' },
    background: { color: '#f8f9fa' },
    cellViewNamespace: shapes
  })

  // 코일 생성
  coil = new RelayCoilShape()
  coil.position(200, 100)
  coil.setVoltage('AC220V')

  // a접점 생성
  contactNO = new RelayContactNO()
  contactNO.position(300, 120)

  // b접점 생성
  contactNC = new RelayContactNC()
  contactNC.position(300, 180)

  // 그래프에 추가
  graph.addCells([coil, contactNO, contactNC])
})

const toggleEnergized = () => {
  isEnergized.value = !isEnergized.value

  // 코일 상태 변경
  coil.setEnergized(isEnergized.value)

  // 접점 상태 변경
  contactNO.setClosed(isEnergized.value)  // NO: 닫힘
  contactNC.setOpen(isEnergized.value)     // NC: 열림
}
</script>
```

---

## 🧪 Step 5: 테스트 및 검증

### 5.1 기능 테스트

```bash
cd electric-symbols
npm run dev
```

**테스트 항목:**

1. ✅ 코일 사각형이 올바르게 렌더링되는가?
2. ✅ A1, A2 단자가 표시되는가?
3. ✅ a접점이 대각선으로 표시되는가?
4. ✅ b접점이 수평선으로 표시되는가?
5. ✅ "코일 ON" 버튼 클릭 시 색상이 변경되는가?
6. ✅ 코일 ON 시 a접점이 닫히는가? (수평선으로 변경)
7. ✅ 코일 ON 시 b접점이 열리는가? (대각선으로 변경)
8. ✅ 포트에 마우스 오버 시 연결 표시가 나타나는가?

---

## 🎓 학습 포인트

### SVG 좌표계

- **원점 (0, 0)**: 좌상단
- **x 축**: 오른쪽으로 증가
- **y 축**: 아래로 증가

### Path 명령어

- `M x y`: Move to (시작점)
- `L x y`: Line to (직선)
- `H x`: Horizontal line (수평선)
- `V y`: Vertical line (수직선)

### 접점 표현 기법

**a접점 (NO)**
```javascript
// 열림: M 5 20 L 20 10 (대각선)
// 닫힘: M 5 20 L 30 20 (수평선)
```

**b접점 (NC)**
```javascript
// 닫힘: M 5 20 L 45 20 (수평선) + 원(●)
// 열림: M 5 20 L 20 10 M 30 20 L 45 20 (끊김)
```

---

## 📚 다음 단계

1. **릴레이 통합 컴포넌트**: 코일 + 여러 접점을 하나의 Shape로 결합
2. **다양한 접점 구성**: 1a1b, 2a2b, 3a3b 등 동적 생성
3. **Wire 연결**: Link를 사용한 포트 간 연결
4. **XML Export**: 릴레이 설정을 XML로 내보내기
5. **라이브러리 확장**: 타이머 릴레이, 열동 릴레이 등

---

## ✅ 체크리스트

- [ ] RelayCoilShape.ts 파일 생성
- [ ] RelayContactNO.ts 파일 생성
- [ ] RelayContactNC.ts 파일 생성
- [ ] relay-symbol.vue 페이지 생성
- [ ] 개발 서버 실행 및 동작 확인
- [ ] 코일 여자 시 접점 변화 확인
- [ ] 포트 연결 테스트
