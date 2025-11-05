# 전기 심볼 제작 가이드

## 📋 개요

JointJS+를 사용하여 전기 회로도용 심볼을 SVG 기반으로 제작하는 범용 가이드입니다.
이 가이드는 릴레이, 접촉기, 스위치, 차단기 등 모든 전기 부품 심볼 제작에 적용할 수 있습니다.

---

## 🎯 심볼 제작 프로세스

```
1. 심볼 설계 → 2. SVG 구조 정의 → 3. Shape 클래스 작성 → 4. 포트 설정 → 5. 동작 메서드 구현
```

---

## 🔧 필수 기능 목록

### 1. Shape 기본 구조

#### 1.1 클래스 정의
```typescript
import { dia, shapes } from '@joint/plus'

export class MySymbolShape extends dia.Element {
  defaults() {
    return {
      ...super.defaults,
      type: 'electrical.MySymbol',
      size: { width: 60, height: 60 },
      attrs: { /* SVG 속성 */ },
      ports: { /* 포트 정의 */ }
    }
  }

  markup = [ /* SVG 요소 배열 */ ]
}
```

#### 1.2 네임스페이스 등록
```typescript
Object.assign(shapes, {
  electrical: {
    ...shapes.electrical,
    MySymbol: MySymbolShape
  }
})
```

---

### 2. SVG 요소 사용법

#### 2.1 사각형 (Rect)
**용도:** 코일, 박스, 외곽선 등

```typescript
// attrs 정의
{
  myRect: {
    x: 10,
    y: 10,
    width: 40,
    height: 30,
    fill: '#ffffff',
    stroke: '#000000',
    strokeWidth: 2,
    rx: 2,  // 모서리 둥글기
    ry: 2
  }
}

// markup 정의
{
  tagName: 'rect',
  selector: 'myRect'
}
```

**주요 속성:**
- `x, y`: 좌상단 좌표
- `width, height`: 크기
- `rx, ry`: 모서리 반경
- `fill`: 채우기 색상
- `stroke`: 테두리 색상
- `strokeWidth`: 테두리 두께

#### 2.2 원 (Circle)
**용도:** 단자, 버튼, 표시등 등

```typescript
// attrs 정의
{
  myCircle: {
    cx: 30,  // 중심 x 좌표
    cy: 30,  // 중심 y 좌표
    r: 5,    // 반지름
    fill: '#000000',
    stroke: '#000000',
    strokeWidth: 1
  }
}

// markup 정의
{
  tagName: 'circle',
  selector: 'myCircle'
}
```

**주요 속성:**
- `cx, cy`: 중심 좌표
- `r`: 반지름
- `fill`: 채우기 색상

#### 2.3 선 (Line/Path)
**용도:** 연결선, 접점, 화살표 등

```typescript
// 직선 (Line)
{
  myLine: {
    x1: 0,
    y1: 20,
    x2: 50,
    y2: 20,
    stroke: '#000000',
    strokeWidth: 2
  }
}

// 복잡한 경로 (Path)
{
  myPath: {
    d: 'M 0 20 L 30 20 L 30 40',  // M=이동, L=직선
    stroke: '#000000',
    strokeWidth: 2,
    strokeLinecap: 'round',
    strokeLinejoin: 'round',
    fill: 'none'
  }
}
```

**Path 명령어:**
- `M x y`: 시작점 이동 (Move)
- `L x y`: 직선 그리기 (Line)
- `H x`: 수평선
- `V y`: 수직선
- `C x1 y1, x2 y2, x y`: 베지어 곡선
- `Z`: 경로 닫기

**주요 속성:**
- `d`: 경로 데이터
- `strokeLinecap`: 선 끝 모양 (`butt`, `round`, `square`)
- `strokeLinejoin`: 선 연결 모양 (`miter`, `round`, `bevel`)
- `fill`: `none`으로 설정하면 선만 그림

#### 2.4 텍스트 (Text)
**용도:** 레이블, 단자 번호, 전압 표시 등

```typescript
{
  myText: {
    x: 30,
    y: 20,
    text: 'K1',
    fontSize: 14,
    fontFamily: 'Arial, sans-serif',
    fontWeight: 'bold',
    textAnchor: 'middle',        // 수평 정렬: start, middle, end
    textVerticalAnchor: 'middle', // 수직 정렬: top, middle, bottom
    fill: '#000000'
  }
}
```

**주요 속성:**
- `text`: 표시할 텍스트
- `fontSize`: 글자 크기
- `textAnchor`: 수평 정렬
- `textVerticalAnchor`: 수직 정렬 (JointJS 확장)
- `fill`: 글자 색상

#### 2.5 다각형 (Polygon)
**용도:** 화살표, 삼각형, 특수 모양 등

```typescript
{
  myPolygon: {
    points: '10,0 20,20 0,20',  // x1,y1 x2,y2 x3,y3
    fill: '#ffffff',
    stroke: '#000000',
    strokeWidth: 2
  }
}
```

#### 2.6 타원 (Ellipse)
**용도:** 램프, 특수 부품 등

```typescript
{
  myEllipse: {
    cx: 30,
    cy: 30,
    rx: 20,  // x축 반지름
    ry: 10,  // y축 반지름
    fill: '#ffffff',
    stroke: '#000000',
    strokeWidth: 2
  }
}
```

---

### 3. 상대 좌표 시스템 (ref 속성)

#### 3.1 상대 위치 지정
```typescript
{
  centerRect: {
    refX: '50%',     // Shape 너비의 50% 위치
    refY: '50%',     // Shape 높이의 50% 위치
    refWidth: '80%', // Shape 너비의 80%
    refHeight: '60%' // Shape 높이의 60%
  }
}
```

**장점:**
- Shape 크기 변경 시 자동으로 비율 유지
- 반응형 디자인 가능

#### 3.2 활용 예시
```typescript
{
  // 투명 배경 (Shape 크기 정의용)
  body: {
    refWidth: '100%',
    refHeight: '100%',
    fill: 'transparent'
  },
  // 중앙 레이블
  label: {
    refX: '50%',
    refY: '50%',
    textAnchor: 'middle',
    textVerticalAnchor: 'middle'
  },
  // 상단 단자
  topTerminal: {
    refX: '50%',
    y: 0
  },
  // 하단 단자
  bottomTerminal: {
    refX: '50%',
    refY: '100%'
  }
}
```

---

### 4. 포트 시스템

#### 4.1 포트 그룹 정의
```typescript
ports: {
  groups: {
    // 입력 포트 그룹
    'in': {
      position: 'absolute',
      attrs: {
        circle: {
          r: 4,
          fill: '#ffffff',
          stroke: '#000000',
          strokeWidth: 2,
          magnet: true  // 연결 가능
        }
      }
    },
    // 출력 포트 그룹
    'out': {
      position: 'absolute',
      attrs: {
        circle: {
          r: 4,
          fill: '#ff0000',
          stroke: '#000000',
          strokeWidth: 2,
          magnet: true
        }
      }
    }
  }
}
```

#### 4.2 포트 아이템 정의
```typescript
ports: {
  groups: { /* ... */ },
  items: [
    {
      id: 'port1',           // 포트 고유 ID
      group: 'in',           // 그룹 이름
      args: { x: 0, y: 30 }, // 절대 좌표
      label: {
        text: 'L1',
        position: {
          name: 'left',      // 위치: left, right, top, bottom
          args: { x: -10 }   // 오프셋
        }
      }
    },
    {
      id: 'port2',
      group: 'out',
      args: { x: 60, y: 30 }
    }
  ]
}
```

#### 4.3 포트 위치 옵션
```typescript
// 절대 좌표
position: 'absolute'

// 상대 좌표 (Shape 경계 기준)
position: {
  name: 'left',   // left, right, top, bottom
  args: { y: 10 } // 오프셋
}

// 각도 기준
position: {
  name: 'ellipse',
  args: { angle: 45 }
}
```

---

### 5. 동적 속성 변경

#### 5.1 attr() 메서드
```typescript
// 단일 속성 변경
element.attr('myRect/fill', '#ff0000')
element.attr('myText/text', '새 텍스트')

// 여러 속성 변경
element.attr({
  'myRect/fill': '#ff0000',
  'myRect/stroke': '#000000',
  'myText/text': '변경됨'
})
```

#### 5.2 커스텀 메서드 구현
```typescript
export class MySymbolShape extends dia.Element {
  // ... defaults, markup

  /**
   * 활성화 상태 변경
   */
  setActive(active: boolean) {
    if (active) {
      this.attr('body/fill', '#4ade80')
      this.attr('label/fill', '#ffffff')
    } else {
      this.attr('body/fill', '#ffffff')
      this.attr('label/fill', '#000000')
    }
  }

  /**
   * 전압 설정
   */
  setVoltage(voltage: string) {
    this.attr('voltageLabel/text', voltage)
  }

  /**
   * 상태 색상 변경
   */
  setStatus(status: 'normal' | 'warning' | 'error') {
    const colors = {
      normal: '#10b981',
      warning: '#f59e0b',
      error: '#ef4444'
    }
    this.attr('statusIndicator/fill', colors[status])
  }
}
```

---

### 6. 애니메이션 및 전환 효과

#### 6.1 transition() 메서드
```typescript
// 부드러운 색상 전환
element.transition('attrs/myRect/fill', '#ff0000', {
  duration: 300,
  timingFunction: (t) => t // linear
})

// Path 애니메이션
element.transition('attrs/myPath/d', 'M 0 20 L 50 20', {
  duration: 200,
  timingFunction: (t) => t * t // ease-in
})
```

#### 6.2 타이밍 함수
```typescript
import { util } from '@joint/plus'

// 이징 함수
util.timing.linear
util.timing.quad
util.timing.cubic
util.timing.inout
util.timing.exponential
util.timing.bounce
```

---

### 7. 이벤트 처리

#### 7.1 요소 이벤트
```typescript
// 클릭 이벤트
element.on('change:position', (element, position) => {
  console.log('위치 변경:', position)
})

element.on('change:attrs', (element, attrs) => {
  console.log('속성 변경:', attrs)
})

// Paper 레벨 이벤트
paper.on('element:pointerclick', (elementView) => {
  console.log('요소 클릭:', elementView.model.id)
})

paper.on('element:pointerdblclick', (elementView) => {
  console.log('더블클릭:', elementView.model.id)
})

paper.on('element:pointerdown', (elementView) => {
  console.log('마우스 다운:', elementView.model.id)
})
```

#### 7.2 포트 이벤트
```typescript
paper.on('element:port:add', (elementView, portId) => {
  console.log('포트 추가:', portId)
})

paper.on('link:connect', (linkView) => {
  console.log('연결 완료:', linkView.model.id)
})
```

---

### 8. 데이터 관리

#### 8.1 커스텀 데이터 저장
```typescript
// 데이터 설정
element.set('customData', {
  manufacturer: 'LS Electric',
  model: 'MC-9b',
  voltage: 'AC220V',
  current: '32A'
})

// 데이터 조회
const data = element.get('customData')
console.log(data.voltage) // 'AC220V'
```

#### 8.2 JSON 직렬화
```typescript
// 그래프를 JSON으로 변환
const json = graph.toJSON()

// JSON에서 그래프 복원
graph.fromJSON(json)

// 로컬 스토리지 저장
localStorage.setItem('circuit', JSON.stringify(json))

// 로컬 스토리지 불러오기
const saved = JSON.parse(localStorage.getItem('circuit'))
graph.fromJSON(saved)
```

---

### 9. 그룹 및 계층 구조

#### 9.1 부모-자식 관계
```typescript
// 자식 요소 추가
parent.embed(child)

// 부모 요소 조회
const parent = child.getParentCell()

// 모든 자식 조회
const children = parent.getEmbeddedCells()

// 그룹 이동 시 자식도 함께 이동
parent.position(100, 100) // 자식도 자동으로 이동
```

#### 9.2 Z-Index (레이어 순서)
```typescript
// 맨 앞으로
element.toFront()

// 맨 뒤로
element.toBack()

// Z-Index 직접 설정
element.set('z', 10)
```

---

### 10. 연결선 (Link)

#### 10.1 기본 Link 생성
```typescript
import { shapes } from '@joint/plus'

const link = new shapes.standard.Link({
  source: { id: element1.id, port: 'out1' },
  target: { id: element2.id, port: 'in1' },
  attrs: {
    line: {
      stroke: '#000000',
      strokeWidth: 2
    }
  }
})

graph.addCell(link)
```

#### 10.2 Link 스타일
```typescript
{
  attrs: {
    line: {
      stroke: '#000000',
      strokeWidth: 2,
      strokeDasharray: '5,5',  // 점선
      targetMarker: {          // 화살표
        type: 'path',
        d: 'M 10 -5 0 0 10 5 Z',
        fill: '#000000'
      }
    }
  },
  router: {
    name: 'orthogonal'  // 직각 라우팅
  },
  connector: {
    name: 'rounded'     // 둥근 모서리
  }
}
```

---

## 📝 심볼 제작 체크리스트

### 설계 단계
- [ ] IEC/KEC 표준 확인
- [ ] 심볼 구성 요소 분석 (코일, 접점, 단자 등)
- [ ] 단자 번호 규칙 정의 (A1/A2, 11/12, 21/22 등)
- [ ] 포트 위치 결정

### 구현 단계
- [ ] Shape 클래스 생성
- [ ] SVG 요소 정의 (rect, circle, path, text)
- [ ] attrs 속성 설정
- [ ] markup 배열 작성
- [ ] 포트 그룹 및 아이템 정의
- [ ] 커스텀 메서드 구현

### 테스트 단계
- [ ] 렌더링 확인
- [ ] 포트 연결 테스트
- [ ] 상태 변경 테스트
- [ ] 드래그/이동 테스트
- [ ] JSON 직렬화/역직렬화 테스트

---

## 🎓 실전 예제 템플릿

### 기본 심볼 템플릿
```typescript
import { dia, shapes } from '@joint/plus'

export class SymbolTemplate extends dia.Element {
  defaults() {
    return {
      ...super.defaults,
      type: 'electrical.SymbolTemplate',
      size: { width: 60, height: 60 },
      attrs: {
        // 투명 배경
        body: {
          refWidth: '100%',
          refHeight: '100%',
          fill: 'transparent',
          stroke: 'none'
        },
        // 메인 Shape
        mainShape: {
          refX: '20%',
          refY: '20%',
          refWidth: '60%',
          refHeight: '60%',
          fill: '#ffffff',
          stroke: '#000000',
          strokeWidth: 2
        },
        // 레이블
        label: {
          refX: '50%',
          refY: '50%',
          text: 'SYMBOL',
          fontSize: 12,
          textAnchor: 'middle',
          textVerticalAnchor: 'middle',
          fill: '#000000'
        }
      },
      ports: {
        groups: {
          'default': {
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
          { id: 'in', group: 'default', args: { x: 0, y: 30 } },
          { id: 'out', group: 'default', args: { x: 60, y: 30 } }
        ]
      }
    }
  }

  markup = [
    { tagName: 'rect', selector: 'body' },
    { tagName: 'rect', selector: 'mainShape' },
    { tagName: 'text', selector: 'label' }
  ]

  setActive(active: boolean) {
    this.attr('mainShape/fill', active ? '#4ade80' : '#ffffff')
  }
}

Object.assign(shapes, {
  electrical: {
    ...shapes.electrical,
    SymbolTemplate: SymbolTemplate
  }
})
```

---

## 📚 참고 자료

- [JointJS+ API 문서](https://resources.jointjs.com/docs/jointjs)
- [SVG 기본 문법](https://developer.mozilla.org/ko/docs/Web/SVG)
- [IEC 60617 전기 심볼 표준](https://en.wikipedia.org/wiki/IEC_60617)

---

## 🚀 다음 단계

1. **릴레이 심볼 구현**: 코일 + a접점 + b접점
2. **접촉기 심볼 구현**: 주접점 + 보조접점
3. **차단기 심볼 구현**: NFB, MCCB, ACB
4. **스위치 심볼 구현**: 푸시버튼, 셀렉터 스위치
5. **심볼 라이브러리 구축**: 카탈로그 시스템
