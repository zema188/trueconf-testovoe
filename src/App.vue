<template>
  <main>
    <div class="building">
      <div class="building__floors-list" ref="buildingList" :style="{width: ElevatorsListWidth+'px'}">
        <the-floor
          v-for="(floor, index) in params.floors"
          :key="index"
          :number="index"
          :elevatorsInfo="elevators"
          :queue="queue"
          @callElevator="(numberFloor) => callElevator(numberFloor)"
        >
        </the-floor>
      </div>
      <div class="building__elevators-list" :style="{ minHeight: buildMinHeight + 'px'}">
        <the-elevator
          v-for="elevator in elevators"
          :key="elevator.id"
          :elevator="elevator"
          :floorHeightPercent="floorHeightPercent"
          :floorHeight="floorHeight"
          @freeElevatorCall="(elevator) => freeElevatorCall(elevator)"
        >
        </the-elevator>
      </div>
      <div class="test" style="position: fixed; right: 0; top: 0;">
        {{ queue }}
      </div>
    </div>
  </main>
</template>

<script setup>
import { computed, onMounted, ref } from 'vue'
import TheElevator from './components/TheElevator.vue'
import TheFloor from './components/TheFloor.vue'

//параметры
const params = {
  floors: 5,
  elevators: 3,
  pause: 3000,
}

// массив лифтов
let elevators = ref([])
// очередь вызовов
let queue = ref([])

let buildingList = ref(null)
let documentHeight = ref(window.innerHeight)

window.addEventListener('resize', function() {
  documentHeight.value = window.innerHeight
})

const floorHeightPercent = computed(() => {
  return 100 / params.floors
})

const floorHeight = computed(() => {
  return Math.max(documentHeight.value / params.floors, 90)
})

const buildMinHeight = computed(() => {
  return params.floors * 90
})

const buildHeight = computed(() => {
  return params.floors * floorHeight.value
})

const ElevatorsListWidth = computed(() => {
  
  if (document.documentElement.clientWidth > params.elevators * 96.8 + (params.elevators - 1) * 20 + 50) {
    return document.documentElement.clientWidth;
  } else {
    return params.elevators * 96.8 + (params.elevators - 1) * 20 + 50;
  }

})


// функция которую вызывает этаж для вызова лифта
const callElevator = (numberFloor) => {
  // пропускаем вызов 
  if(queue.value.includes(numberFloor) || cheackElevatorOnFloor(numberFloor) || cheackElevatorToCalled(numberFloor)) {
    return 0
  }

  const nearestElevator = findNearestElevator(numberFloor)

  // если есть свободный лифт вызываем его
  if(nearestElevator) {

    changeStateElevatorToCalled(nearestElevator, numberFloor)

  // если нет свободного лифтра, добавляем вызов в очередь
  } else {
    queue.value.push(numberFloor)
  }
}


// Функция вызова лифтра
const changeStateElevatorToCalled = async (nearestElevator, numberFloor) => {
  nearestElevator.state = 'called';
  nearestElevator.floor_called = numberFloor;

  // Определяем направление движения
  if (nearestElevator.floor_called > nearestElevator.floor_current) nearestElevator.direction = 1;
  if (nearestElevator.floor_called < nearestElevator.floor_current) nearestElevator.direction = -1;

  // Вычисляем дистанцию и время
  const time = 1000 * Math.abs(nearestElevator.floor_called - nearestElevator.floor_current);

  // делаем анимацию
  const animationPromise = new Promise((resolve) => {
    elevatorMove(nearestElevator, time, () => {

      resolve();
    });
  });

  await animationPromise;
  nearestElevator.floor_current = numberFloor

  // отправляем лифт на паузу
  const pausePromise = new Promise((resolve) => {
    pauseElevator(nearestElevator, params.pause, () => {

      resolve();
    });
  });
  
  await pausePromise;
  nearestElevator.state = 'free';
};


// анимация движения лифтра
function elevatorMove(elevator, duration, callback) {
  const distance = Math.abs(elevator.floor_called * floorHeight.value) / buildHeight.value * 100 - elevator.position_y;
  const startValue = elevator.position_y;

  let startTime;
  let lastUpdateTime = 0; // Время последнего обновления

  function updateValue(currentTime) {
    if (!startTime) {
      startTime = currentTime;
    }

    const elapsedTime = currentTime - startTime;

    // Обновляем time_to_called_floor на каждом кадре
    elevator.time_to_called_floor = duration - elapsedTime;

    if (elapsedTime >= duration) {
      elevator.position_y = startValue + distance;
      elevator.floor_current = Math.round(elevator.position_y / floorHeight.value);
      elevator.time_to_called_floor = 0; // Время до вызова этажа равно 0
      if (callback) {
        callback();
      }
    } else {
      const progress = elapsedTime / duration;
      elevator.position_y = startValue + progress * distance;

      if (currentTime - lastUpdateTime >= 1000) {
        elevator.floor_current += elevator.direction;
        lastUpdateTime = currentTime;
      }

      requestAnimationFrame(updateValue);
    }
  }

  requestAnimationFrame(updateValue);
}


// пауза лифта
function pauseElevator(elevator, time_sleep, callback) {
  elevator.state = 'pause';
  elevator.floor_called = null;
  elevator.direction = null;
  let startTime = null

  function updateValue(currentTime) {
    if (!startTime) {
      startTime = currentTime
    }

    const elapsedTime = currentTime - startTime
    const remainingTime = Math.max(0, time_sleep - elapsedTime)

    elevator.time_sleep = remainingTime

    if (remainingTime <= 0) {
      if (callback) {
        callback()
      }
    } else {
      requestAnimationFrame(updateValue)
    }
  }

  requestAnimationFrame(updateValue)
}


// функция проверки есть ли на этаже свободный лифт
function cheackElevatorOnFloor(numberFloor) {
  for (const elevator of elevators.value) {
    if (elevator.state === 'free' && elevator.floor_current === numberFloor) {
      return true
    }
  }
  return false
}


// функция проверки есть ли сейчас лифт который едет на этаж
function cheackElevatorToCalled(numberFloor) {
  for (const elevator of elevators.value) {
    if (elevator.state === 'called' && elevator.floor_called === numberFloor) {
      return true
    }
  }
  return false
}


//функция возвращает ближайший свободный лифт к переданному этажу
function findNearestElevator(floor) {
  const availableElevators = elevators.value.filter(elevator => elevator.state === "free");

  if (availableElevators.length === 0) {
    return null;
  }

  return availableElevators.reduce((closestElevator, elevator) => {
    const distance = Math.abs(elevator.floor_current - floor);
    const closestDistance = Math.abs(closestElevator.floor_current - floor);
    return distance < closestDistance ? elevator : closestElevator;
  });
}


// инициализируем приложение при запуске
const initBuilding = () => {
  // если параметры изменились
  const paramsInfo = JSON.parse(localStorage.getItem('params'))
  if(paramsInfo) {
    if(paramsInfo.floors !== params.floors || paramsInfo.elevators !== params.elevators) {
      initElevators()
      synchronizationLocalStorage()
      return
    }
  }
  const elevatorsInfo = localStorage.getItem('elevators')
  const queueInfo = localStorage.getItem('queue')
  // если нет сохраненной информации о лифтах
  if(!elevatorsInfo) {
    initElevators()
  } else { //если есть сохраненная информация о лифтах
    elevators.value = [...JSON.parse(elevatorsInfo)]
    queue.value = [...JSON.parse(queueInfo)]
    // вызываем функцию для проверки состояний лифтов и запуска соответствующих функция
    resumeWorkElevators()
  }
}


// функция которую вызывает лифт если при загрузке страницы его state == called
const resumeWorkElevators = async () => {
  elevators.value.forEach(async (elevator) => {

    // если лифт в движении
    if (elevator.state === 'called') {

      const animationPromise = new Promise((resolve) => {

        elevatorMove(elevator, elevator.time_to_called_floor, () => {

          resolve();
        });
      });
      await animationPromise

      // отправляем лифт на паузу
      const pausePromise = new Promise((resolve) => {
        pauseElevator(elevator, params.pause, () => {

          resolve();
        });
      });
      
      await pausePromise;
      elevator.state = 'free';
      return
    }

    // если лифт на паузу
    if (elevator.state === 'pause') {
    // отправляем лифт на паузу
      const pausePromise = new Promise((resolve) => {
        pauseElevator(elevator, elevator.time_sleep, () => {

          resolve();
        });
      });
      await pausePromise;
      elevator.state = 'free';
      return
    }
  });
};

// лифт который освободился вызывает эту функцию и если в очереди есть вызовы то даем этот вызов освободившемуся лифту
const freeElevatorCall = (elevator) => {
  if(queue.value.length) {
    const floorNumber = queue.value.shift()
    if(elevator.floor_current !== floorNumber)
    changeStateElevatorToCalled(elevator, floorNumber)
  }
}


// инициализируем данные лифтов
const initElevators = () => {
  for (let i = 0; i < params.elevators; i++) {
    const elevator = {
      id: i,
      floor_current: 0,
      floor_called: null,
      direction: null,
      state: 'free',
      time_sleep: 0,
      time_to_called_floor: 0,
      position_y: 0,
    }
    elevators.value.push(elevator)
  }
}


// синхронизируем localStorage с переменными
const synchronizationLocalStorage = () => {
  localStorage.setItem('elevators', JSON.stringify(elevators.value))
  localStorage.setItem('queue', JSON.stringify(queue.value))
  localStorage.setItem('params', JSON.stringify(params))
}

//при загрузке страницы инициализируем данные
onMounted(() => {
  initBuilding()
})

//при перезагрузке или закрытии страницы сохраняем все в localstorage
window.addEventListener('beforeunload', synchronizationLocalStorage);

</script>

<style lang="scss">
  .building {
    position: relative;
    height: 100vh;
    &__floors-list {
      display: flex;
      flex-direction: column-reverse;
      min-height: 100%;
    }
    &__floor {
      flex: 1;
      min-height: 90px;
      border-top: 1px solid #969696;
      display: flex;
    }
    &__elevators-list {
      position: absolute;
      top: 0;
      bottom: 0;
      height: 100%;
      width: 100%;
      display: flex;
      gap: 20px;
      padding-left: 50px;
      pointer-events: none;
    }
    &__elevator {
      flex: 1;
      max-width: 100px;
      min-width: 100px;
      border-left: 2px solid #969696;
      border-right: 2px solid #969696;
      position: relative;
      pointer-events: all;
      &.paused {
        & .building__elevator-cabin {
          animation: changeColor 3s linear infinite;
        }
      }
      &.down {
        & .building__elevator-cabin {
          & .icon {
              transform: translate(-50%, -50%) rotate(180deg);
          }
        }
      }
    }
    &__elevator-cabin {
      width: 100%;
      background: rgb(114, 114, 243);
      position: absolute;
      left: 0;
      bottom: 0;
      transition: .2s;
      min-height: 90px;
      text-align: center;
      padding-top: 5px;
      & .icon {
        width: 20px;
        height: 20px;
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
      }
    }
    &__elevator-current-floor {
      position: absolute;
      right: 0;
      bottom: 0;
    }
    &__floor-btn {
      cursor: pointer;
      padding-left: 10px;
      display: flex;
      flex-direction: column;
      justify-content: center;
      min-height: 100%;
      width: fit-content;
      &.called {
        & span {
          &::after {
          background: red;
          }
        }
      }
      &.calledInQueue {
        & span {
          &::after {
            background: yellow;
          }
        }
      }
      & span {
        width: 30px;
        height: 30px;
        border: 1px solid rgb(114, 114, 243);
        display: flex;
        justify-content: center;
        align-items: center;
        margin-top: 5px;
        &::after {
          content: "";
          width: 10px;
          height: 10px;
          border-radius: 50%;
          background: rgb(114, 114, 243);
          outline: 1px solid rgb(114, 114, 243);
          outline-offset: 3px;
        }
      }
    }
  }

  @keyframes changeColor {
  0% {
    background-color: rgb(222, 228, 49);
  }
  50% {
    background-color: rgba(176, 181, 31, 0.292);
  }
  100% {
    background-color: rgb(222, 228, 49);
  }
}
</style>
