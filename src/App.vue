<template>
  <main>
    <div class="building">
      <div class="building__floors-list"
        :style="{ width: elevatorsListWidth }"
      >
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
      <div class="building__elevators-list"
        :style="{ minHeight: buildMinHeight + 'px'}"
      >
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
    </div>
  </main>
</template>

<script setup>
import { computed, onMounted, ref } from 'vue'
import TheElevator from './components/TheElevator.vue'
import TheFloor from './components/TheFloor.vue'

const params = { //параметры
  floors: 5,
  elevators: 1,
  pause: 3000,
}

let elevators = ref([]) // массив лифтов

let queue = ref([]) // очередь вызовов

let documentHeight = ref(window.innerHeight) // высота экрана
let documentWidth = ref(window.innerWidth) // ширина экрана

// обновляем высоту экрана
window.addEventListener('resize', () => {
  documentHeight.value = window.innerHeight;
  documentWidth.value = window.innerWidth
});


// высота этажа в процентах
const floorHeightPercent = computed(() => {
  return 100 / params.floors
})


// высота этажа в px
const floorHeight = computed(() => {
  return Math.max(documentHeight.value / params.floors, 90)
})


// минимальная высота блока (здания, лифта, всех этажей)
const buildMinHeight = computed(() => {
  return params.floors * 90
})


// высота блока (здания, лифта, всех этажей) 
const buildHeight = computed(() => {
  return params.floors * floorHeight.value
})


// ширина здания
const elevatorsListWidth = computed(() => {
  const minWidth = params.elevators * 100 + (params.elevators - 1) * 20 + 50;
  return documentWidth.value > minWidth ? 'auto' : minWidth + 'px'; 
});


// функция которую вызывает этаж для вызова лифта
const callElevator = (numberFloor) => {
  // пропускаем вызов если он уже в очереди или к нему уже едет лифт или на этаже уже есть свободный лифт
  if(queue.value.includes(numberFloor) || cheackElevatorOnFloor(numberFloor) || cheackElevatorToCalled(numberFloor)) {
    return 0
  }
  
  const nearestElevator = findNearestElevator(numberFloor) // получаем ближайший лифт
  
  if(nearestElevator) { // если есть свободный лифт вызываем его

    changeStateElevatorToCalled(nearestElevator, numberFloor)
  
  } else { // если нет свободного лифтра, добавляем вызов в очередь
    queue.value.push(numberFloor)
  }
}


// Функция вызова лифтра
const changeStateElevatorToCalled = async (nearestElevator, numberFloor) => {
  nearestElevator.state = 'called'
  nearestElevator.floor_called = numberFloor

  // Определяем направление движения
  if (nearestElevator.floor_called > nearestElevator.floor_current) nearestElevator.direction = 1
  if (nearestElevator.floor_called < nearestElevator.floor_current) nearestElevator.direction = -1

  // Вычисляем время
  const time = 1000 * Math.abs(nearestElevator.floor_called - nearestElevator.floor_current)

  // делаем анимацию движения
  const animationPromise = new Promise((resolve) => {
    elevatorMove(nearestElevator, time, () => {

      resolve();
    })
  })

  await animationPromise;

  // отправляем лифт на паузу
  const pausePromise = new Promise((resolve) => {
    pauseElevator(nearestElevator, params.pause, () => {

      resolve()
    })
  })
  
  await pausePromise
  nearestElevator.state = 'free'
}


// функция анимация движения лифтра
const elevatorMove = (elevator, duration, callback) => {
  const distance = Math.abs(elevator.floor_called * floorHeight.value) / buildHeight.value * 100 - elevator.position_y
  const startValue = elevator.position_y

  let startTime

  const updateValue = (currentTime) => {
    if (!startTime) {
      startTime = currentTime
    }

    const elapsedTime = currentTime - startTime

    elevator.time_to_called_floor = duration - elapsedTime

    if (elapsedTime >= duration) {
      elevator.position_y = startValue + distance
      
      if (elevator.position_y >= elevator.floor_called * floorHeight.value) {
        elevator.floor_current = elevator.floor_called
        
      }
      elevator.time_to_called_floor = 0
      if (callback) {
        callback()

      }
    } else {
      const progress = elapsedTime / duration
      elevator.position_y = startValue + progress * distance

      if(elevator.direction === 1) {
        elevator.floor_current = Math.floor((elevator.position_y / 100) * params.floors) + 1;

      } else {
        elevator.floor_current = Math.floor((elevator.position_y / 100) * params.floors);

      }

      requestAnimationFrame(updateValue)
    }
  }

  requestAnimationFrame(updateValue)
}


// функция пауза лифта
const pauseElevator = (elevator, time_sleep, callback) => {
  elevator.state = 'pause'
  elevator.floor_called = null
  elevator.direction = null
  let startTime = null

  const updateValue = (currentTime) => {
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
const cheackElevatorOnFloor = (numberFloor) => {
  for (const elevator of elevators.value) {
    if (elevator.state === 'free' && elevator.floor_current === numberFloor) {
      return true
    }
  }
  return false
}


// функция проверки есть ли сейчас лифт который едет на этаж
const cheackElevatorToCalled = (numberFloor) => {
  for (const elevator of elevators.value) {
    if (elevator.state === 'called' && elevator.floor_called === numberFloor) {
      return true
    }
  }
  return false
}


//функция возвращает ближайший свободный лифт к переданному этажу
const findNearestElevator = (floor) =>{
  const availableElevators = elevators.value.filter(elevator => elevator.state === "free")

  if (availableElevators.length === 0) {
    return null
  }

  return availableElevators.reduce((closestElevator, elevator) => {
    const distance = Math.abs(elevator.floor_current - floor)
    const closestDistance = Math.abs(closestElevator.floor_current - floor)
    return distance < closestDistance ? elevator : closestElevator
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
  
  if(!elevatorsInfo) { // если нет сохраненной информации о лифтах
    initElevators()
  } else { // если есть сохраненная информация о лифтах
    elevators.value = [...JSON.parse(elevatorsInfo)]
    queue.value = [...JSON.parse(queueInfo)]
    
    // вызываем функцию для проверки состояний лифтов и запуска соответствующих функция
    resumeWorkElevators()
  }
}


// функция для продолжения работы лифтов если страница была загружена и state != free
const resumeWorkElevators = async () => {
  elevators.value.forEach(async (elevator) => {

    if (elevator.state === 'called') { // если лифт в движении

      const animationPromise = new Promise((resolve) => {

        elevatorMove(elevator, elevator.time_to_called_floor, () => {

          resolve()
        })
      })
      await animationPromise

      // отправляем лифт на паузу
      const pausePromise = new Promise((resolve) => {
        pauseElevator(elevator, params.pause, () => {

          resolve()
        })
      })
      
      await pausePromise
      elevator.state = 'free'
      return
    }

    
    if (elevator.state === 'pause') { // если лифт на паузу
    // отправляем лифт на паузу
      const pausePromise = new Promise((resolve) => {
        pauseElevator(elevator, elevator.time_sleep, () => {

          resolve()
        })
      })
      await pausePromise
      elevator.state = 'free'
      return
    }
  })
}

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
window.addEventListener('beforeunload', synchronizationLocalStorage)

</script>

<style lang="scss">
  .building {
    position: relative;
    overflow: auto;
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
