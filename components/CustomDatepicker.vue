<template>
  <div class="container">
    <input ref="inputRef" type="text" class="datepicker-input" :value="modelValue" @input="handleInput" @keydown="handleKeydown" :placeholder="format" maxlength="10" :readonly="!allowManualInput" @click="showModal = !showModal">
    <Transition name="modal-fade">
      <div class="modal" v-if="showModal" ref="modalRef">
        <div class="modal-header">
          <button class="prevBtn" @click="prevMonth">‹</button>
          <span class="calendar-title">{{ calendarHeaderDate }}</span>
          <button class="nextBtn" @click="nextMonth">›</button>
        </div>
        <div class="modal-body">
          <div class="calendar-day" v-for="day in DAYS_OF_WEEK" :key="day">
            {{ day }}
          </div>
          <div @click="onDateClick(day.date)" class="calendar-date" :class="{'is-today':isToday(day.date) && !isReservation(day.date), 'is-reservation':isReservation(day.date), 'is-not-current-month': !day.isCurrentMonth}" v-for="day in daysInMonth" :key="day.date.getTime()">
            {{ day.date.getDate() }}
          </div>
        </div>
      </div>
    </Transition>
    <p>현재 예약 날짜: {{ modelValue }}</p>
  </div>
</template>

<script setup lang="ts">
interface Props{
  modelValue: string;
  format?: string;
  allowManualInput?: boolean;
}

const props = withDefaults(defineProps<Props>(),{
  format: 'YYYY-MM-DD',
  allowManualInput: false,
});

const emit = defineEmits(['update:modelValue']);

const showModal = ref(false);
const renderDate = ref(new Date());
const inputRef = ref(null);
const modalRef = ref(null);

const DAYS_OF_WEEK = ['일', '월', '화', '수', '목', '금', '토'];

const handleClickOutside = (event: MouseEvent) => {
  const input = inputRef.value as HTMLElement | null;
  const modal = modalRef.value as HTMLElement | null;

  if (input && !input.contains(event.target as Node) && modal && !modal.contains(event.target as Node)) {
    showModal.value = false;
  }
};

onMounted(() => {
  document.addEventListener('click', handleClickOutside);
});

onBeforeUnmount(() => {
  document.removeEventListener('click', handleClickOutside);
});

const calendarHeaderDate = computed(() => {
    const year = renderDate.value.getFullYear();
    const month = String(renderDate.value.getMonth() + 1); // 월을 2자리 숫자로 만듭니다
     return `${month}월 ${year}`;
});

// 해당 월의 첫 날과 마지막 날 계산
const firstDayOfMonth = computed(() => {
  return new Date(renderDate.value.getFullYear(), renderDate.value.getMonth(), 1);
});


const lastDayOfMonth = computed(() => {
  return new Date(renderDate.value.getFullYear(), renderDate.value.getMonth() + 1, 0);
});

// 해당 월의 날짜 배열 생성
const daysInMonth = computed(() => {
  const days = [];
  for (let i = firstDayOfMonth.value.getDay(); i > 0; i--) {
    days.push({ date: new Date(firstDayOfMonth.value.getTime() - i * 24 * 60 * 60 * 1000), isCurrentMonth: false });
  }
  for (let i = 1; i <= lastDayOfMonth.value.getDate(); i++) {
    days.push({ date: new Date(renderDate.value.getFullYear(), renderDate.value.getMonth(), i), isCurrentMonth: true });
  }
  for (let i = 1; days.length % 7 !== 0; i++) {
    days.push({ date: new Date(lastDayOfMonth.value.getTime() + i * 24 * 60 * 60 * 1000), isCurrentMonth: false });
  }
  return days;
});

// 이전, 다음 달로 이동하는 함수들
const prevMonth = () => {
  const newDate = new Date(renderDate.value.getFullYear(), renderDate.value.getMonth() - 1, 1);
  renderDate.value = newDate;
};

const nextMonth = () => {
  const newDate = new Date(renderDate.value.getFullYear(), renderDate.value.getMonth() + 1, 1);
  renderDate.value = newDate;
};


const formatDateInput = (input: string): string => {
  // 숫자만 남기기
  const cleaned = input.replace(/\D/g, '');
  
  if (cleaned.length <= 4) {
    return cleaned; // 연도만 입력되면 그대로 반환
  } else if (cleaned.length <= 6) {
    return `${cleaned.slice(0, 4)}-${cleaned.slice(4, 6)}`; // YYYY-MM
  } else {
    return `${cleaned.slice(0, 4)}-${cleaned.slice(4, 6)}-${cleaned.slice(6, 8)}`; // YYYY-MM-DD
  }
};

const handleInput = (event: Event) => {
  showModal.value = true;
  const input = event.target as HTMLInputElement;
  const value = input.value;
  const formattedValue = formatDateInput(value);

  emit('update:modelValue', formattedValue);

  if (formattedValue.length === 10) {
    const parsedDate = parseYYYYMMDDToDate(formattedValue);
    
    if (parsedDate) {
      renderDate.value = parsedDate;
    }else{
      renderDate.value = new Date();
      emit('update:modelValue', ''); 
    }
  }
}

const handleKeydown = (event: KeyboardEvent) => {
  if (event.key === 'Enter' || event.key === 'Escape' || event.key === 'Tab') {
    showModal.value = false; // Enter 키가 눌리면 모달 닫기
  }
};

const formatDateToYYYYMMDD = (date: Date): string => {
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0'); // 월은 0부터 시작하므로 +1
  const day = String(date.getDate()).padStart(2, '0'); // 일자를 2자리로 맞춤

  return `${year}-${month}-${day}`;
};

const parseYYYYMMDDToDate = (dateString: string): Date | null => {
  const regex = /^\d{4}-\d{2}-\d{2}$/;
  
  if (!regex.test(dateString)) {
    return null;
  }

  const [year, month, day] = dateString.split('-').map(Number);
  
  const date = new Date(year, month - 1, day);

  if (date.getFullYear() === year && date.getMonth() === month - 1 && date.getDate() === day) {
    return date;
  } else {
    return null; 
  }
};

const onDateClick = (date: Date) => {
  showModal.value = false;
  emit('update:modelValue', formatDateToYYYYMMDD(date));
};

const isToday = (date:Date) => {
  const today = new Date();
  return (
    date.getDate() === today.getDate() &&
    date.getMonth() === today.getMonth() &&
    date.getFullYear() === today.getFullYear()
  );
}

const isReservation = (date:Date) => {
  if(!props.modelValue){
    return;
  }
  const reservation = parseYYYYMMDDToDate(props.modelValue);
  if(!reservation){
    return null;
  }
  return (
    date.getDate() === reservation.getDate() &&
    date.getMonth() === reservation.getMonth() &&
    date.getFullYear() === reservation.getFullYear()
  );
}

watch(showModal, (newValue) => {
  if (!newValue) { // showModal이 false일 때
    const regex = /^\d{4}-\d{2}-\d{2}$/;
    if (!regex.test(props.modelValue)) {
      emit('update:modelValue', ''); // 형식이 맞지 않으면 빈 문자열로 초기화
      renderDate.value = new Date();
    }
  }
});
</script>

<style scoped>
.container{
  position: relative;
}

.datepicker-input{
  width: 200px;
  padding: 10px;
  text-align: center;
  font-size: 15px;
  border: 1px solid lightgray;
  border-radius: 4px;
  user-select: none;
  cursor: pointer;
}

.is-invalidDate{
  border-color: red;
}

.modal-fade-enter-active, .modal-fade-leave-active {
  transition: all 0.2s ease; 
}
.modal-fade-leave-to {
  opacity: 0;
  transform: translateX(100px); 
}
.modal-fade-enter-from{
  opacity: 0; 
  transform: translateX(100px);
}
.modal-fade-leave-from {
  transform: translateX(0px);
  opacity: 1;
}
.modal-fade-enter-to{
  transform: translateX(0px);
  opacity: 1;
}

.modal {
  font-size: 14px;
  background-color: white;
  z-index: 99;
  position: absolute;
  top: 50px;
  left: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 300px;
  user-select: none;
  border: 1px solid lightgray;
  padding: 10px;
  border-radius: 5px;
}

.modal-header {
  width: 100%;
  display: grid;
  grid-template-columns: 1fr 5fr 1fr;
  gap: 5px;
  align-items: center;
}

.calendar-title {
  font-weight: bold;
  text-align: center;
}

.prevBtn, .nextBtn {
  border: none;
  font-size: 24px;
  font-weight: bold;
  background-color: transparent;
  cursor: pointer;
  padding: 0 10px;
}

.prevBtn{
  text-align: left;
}

.nextBtn{
  text-align: right;
}

.modal-body {
  width: 100%;
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 5px;
  cursor: pointer;
}

.calendar-day, .calendar-date {
  text-align: center;
  padding: 10px;
}

.calendar-date{
  border: 1px solid transparent;
  transition: all 0.2s ease;
}

.calendar-day {
  font-weight: bold;
}

.is-not-current-month {
  color: lightgray;
}

.is-today{
  background-color: #357EBD;
  color: white;
  font-weight: bold;
  border-radius: 4px;
}

.is-reservation {
  border: 1px solid #357EBD;
  border-radius: 4px;
  font-weight: bold;
}

.calendar-date:hover {
  background-color: whitesmoke;
}
</style>