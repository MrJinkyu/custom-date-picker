<template>
  <div class="container">
    <input ref="inputRef" type="text" class="datepicker-input" :value="modelValue" @input="handleInput" @keydown="handleKeydown" :placeholder="type === 'dateTime' ? `${format} HH:mm:ss`:format" :maxlength="type === 'dateTime' ? 19 : 10" :readonly="!allowManualInput" @click="showModal = !showModal">
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
          <div @click="onDateClick(day.date)" class="calendar-date" :class="{'is-today':isToday(day.date) && !isReservation(day.date) && !isSelected(day.date), 'is-reservation':isReservation(day.date), 'is-not-current-month': !day.isCurrentMonth, 'is-selected':type === 'dateTime' && isSelected(day.date)}" v-for="day in daysInMonth" :key="day.date.getTime()">
            {{ day.date.getDate() }}
          </div>
        </div>

        <div v-if="props.type === 'dateTime'" class="time-picker">
            <input class="time-input" type="text" v-model="hour"  placeholder="HH"/>:
            <input class="time-input" type="text" v-model="minute" placeholder="mm"/>:
            <input class="time-input" type="text" v-model="second" placeholder="ss"/>
            <button class="time-btn" @click="onSubmitBtnClick">확인</button>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup lang="ts">
interface Props{
  modelValue: string;
  format?: string;
  allowManualInput?: boolean;
  type?: 'date' | 'dateTime';
}

const props = withDefaults(defineProps<Props>(),{
  format: 'YYYY-MM-DD',
  allowManualInput: false,
  type: 'date',
});

const emit = defineEmits(['update:modelValue']);

const showModal = ref(false);
const renderDate = ref(new Date());
const inputRef = ref(null);
const modalRef = ref(null);
const hour = ref('');
const minute = ref('');
const second = ref('');
const selectDate = ref<Date | null>(null);

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
  
    if (props.type === 'date') {
    if (cleaned.length <= 4) {
      return cleaned; // 연도만 입력되면 그대로 반환
    } else if (cleaned.length <= 6) {
      return `${cleaned.slice(0, 4)}-${cleaned.slice(4, 6)}`; // YYYY-MM
    } else {
      return `${cleaned.slice(0, 4)}-${cleaned.slice(4, 6)}-${cleaned.slice(6, 8)}`; // YYYY-MM-DD
    }
  }
  
  if (props.type === 'dateTime') {
    if (cleaned.length <= 4) {
      return cleaned; // 연도만 입력되면 그대로 반환
    } else if (cleaned.length <= 6) {
      return `${cleaned.slice(0, 4)}-${cleaned.slice(4, 6)}`; // YYYY-MM
    } else if (cleaned.length <= 8) {
      return `${cleaned.slice(0, 4)}-${cleaned.slice(4, 6)}-${cleaned.slice(6, 8)}`; // YYYY-MM-DD
    } else if (cleaned.length <= 10) {
      return `${cleaned.slice(0, 4)}-${cleaned.slice(4, 6)}-${cleaned.slice(6, 8)} ${cleaned.slice(8, 10)}`; // YYYY-MM-DD HH
    } else if (cleaned.length <= 12) {
      return `${cleaned.slice(0, 4)}-${cleaned.slice(4, 6)}-${cleaned.slice(6, 8)} ${cleaned.slice(8, 10)}:${cleaned.slice(10, 12)}`; // YYYY-MM-DD HH:mm
    } else {
      return `${cleaned.slice(0, 4)}-${cleaned.slice(4, 6)}-${cleaned.slice(6, 8)} ${cleaned.slice(8, 10)}:${cleaned.slice(10, 12)}:${cleaned.slice(12, 14)}`; // YYYY-MM-DD HH:mm:ss
    }
  }

  return cleaned; // 기본값으로 반환
};

const handleInput = (event: Event) => {
  if(props.allowManualInput === false){
    return ;
  }
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
  if(props.type==='dateTime'){
    if(hour.value === '' && minute.value === '' && second.value === ''){
      hour.value = '12';
      minute.value = '00';
      second.value = '00';
    }
    return `${year}-${month}-${day} ${hour.value}:${minute.value}:${second.value}`;
  }
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

watch([hour,minute,second], () => {
  if (props.modelValue) {
    const [datePart] = props.modelValue.split(' '); // 날짜 부분만 추출 (시, 분, 초 제외)
    

    // 기존 날짜에 새로운 시, 분, 초 값을 합쳐서 emit
    const updatedModelValue = `${datePart} ${hour.value}:${minute.value}:${second.value}`;
    
    emit('update:modelValue', updatedModelValue);
  }
});

const onSubmitBtnClick = () => {
  showModal.value = false;
}

const onDateClick = (date: Date) => {
  if(props.type === 'date'){
    emit('update:modelValue', formatDateToYYYYMMDD(date));
    showModal.value = false;
  }else{
    selectDate.value = date;
    emit('update:modelValue', formatDateToYYYYMMDD(date));
  }
};

const isSelected = (date:Date) => {
  if(!selectDate.value){
    return null;
  }
  return (
    date.getDate() === selectDate.value.getDate() &&
    date.getMonth() === selectDate.value.getMonth() &&
    date.getFullYear() === selectDate.value.getFullYear()
  );
}

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
    let regex;
    if (props.type === 'dateTime') {
      regex = /^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}$/; // YYYY-MM-DD HH:mm:ss
    } else {
      regex = /^\d{4}-\d{2}-\d{2}$/; // YYYY-MM-DD
    }
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

.is-reservation, .is-selected {
  border: 1px solid #357EBD;
  border-radius: 4px;
  font-weight: bold;
}

.time-picker {
  display: flex;
  align-items: center;
  gap: 10px;
  border-top: 1px solid lightgray;
  padding-top: 10px;
  margin-top: 10px;
}

.time-input {
  width: 100%;
  height: 24px;
  text-align: right;
  border: none;
  background-color: whitesmoke;
  border-radius: 4px;
  cursor: pointer;
  outline: none;
}

.time-input:focus{
  outline: 1px solid #357EBD;
}

.time-btn{
  margin-left: 10px;
  width: 100%;
  border: none;
  background-color: #357EBD;
  color: white;
  height: 24px;
  border-radius: 4px;
  cursor: pointer;
  outline: 1px solid #357EBD;
}
</style>