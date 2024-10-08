# Custom Datepicker Component

이 컴포넌트는 수동 입력과 모달 기반 날짜 선택 기능을 제공하는 커스텀 Vue 데이트피커로 <br/>
vue3 + typescript 문법으로 작성되었습니다.

<br/>

## 주요 기능
- 모달 캘린더를 통해 날짜 선택 가능.
- 수동으로 날짜 입력 가능(옵션).
- 모달에서 이전/다음 달로 이동 가능.
- 수동 입력 시 날짜 자동 포맷팅(YYYY-MM-DD).
- 오늘 날짜와 선택한(예약된) 날짜 하이라이트 표시.
- 날짜 포맷 및 수동 입력 허용 여부를 커스터마이즈할 수 있는 props 제공.

<br/>

## 사용법

1. CustomDatepicker.vue 컴포넌트 파일을 Vue 프로젝트에 복사합니다.
2. 사용할 컴포넌트에서 임포트합니다.

```js
import CustomDatepicker from './CustomDatepicker.vue';


<template>
  <CustomDatepicker v-model="selectedDate" />
</template>

<script setup lang="ts">
import { ref } from 'vue';

const selectedDate = ref('');
</script>
```

<br/>

### Props

| Prop 이름        | 타입      | 기본값        | 설명                                              |
|------------------|-----------|---------------|---------------------------------------------------|
| `modelValue`     | `String`  | 필수          | 선택된 날짜를 바인딩하는 값입니다.                 |
| `format`         | `String`  | `'YYYY-MM-DD'`| 수동 입력 시 사용할 날짜 포맷입니다.               |
| `allowManualInput`| `Boolean` | `false`       | `true`로 설정하면 수동으로 날짜를 입력할 수 있습니다. |


<br/>

### Emit 이벤트

| 이벤트 이름        | 설명                                                            |
|--------------------|-----------------------------------------------------------------|
| `update:modelValue`| 날짜가 업데이트될 때(수동 입력 또는 모달에서 선택) 발생합니다.  |


<br/>

### 메서드
- `onDateClick(date: Date)`: 모달 캘린더에서 날짜를 선택합니다.
- `prevMonth()`: 모달에서 이전 달로 이동합니다.
- `nextMonth()`: 모달에서 다음 달로 이동합니다.

<br/>

### CSS 클래스 목록
| 클래스 이름 | 설명 |
| --- | --- |
| `.datepicker-input` | 날짜 입력 필드의 스타일입니다. |
| `.modal` | 모달 캘린더의 스타일입니다. |
| `.is-today` | 오늘 날짜에 적용되는 스타일입니다. |
| `.is-reservation` | 예약된 날짜에 적용되는 스타일입니다. |
| `.is-not-current-month` | 현재 월이 아닌 날짜에 적용되는 스타일입니다. |

<br/>

