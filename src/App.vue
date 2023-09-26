<template>
  <main>
    <div class="building">
      <div class="building__floors-list">
        <the-floor
          v-for="(floor, index) in params.floors"
          :key="index"
          :number="index"
          :elevatorsInfo="elevators"
          @callElevator="(numberFloor) => callElevator(numberFloor)"
        >
        </the-floor>
      </div>
      <div class="building__elevators-list" :style="{ minHeight: buildMinHeight + 'px'}">
        <the-elevator
          v-for="(elevator, index) in elevators"
          :key="index"
          :floorHeightPercent="floorHeightPercent"
          :elevator="elevator"
          @elevatorArrived="(id) => elevatorArrived(id)"
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
import { computed, onMounted, ref, watch } from 'vue'
import TheElevator from './components/TheElevator.vue'
import TheFloor from './components/TheFloor.vue'

const params = {
  floors: 20,
  elevators: 5,
  pause: 3000,
}

let elevators = ref([])
let queue = ref([])
const floorHeightPercent = computed(() => {
  return 100 / params.floors
})

const buildMinHeight = computed(() => {
  return params.floors * 90
})

const callElevator = (numberFloor) => {
  // пропускаем вызов 
  if(queue.value.includes(numberFloor) || cheackElevatorOnFloor(numberFloor) || cheackElevatorToCalled(numberFloor)) {
    return 0
  }

  const nearestElevator = findNearestElevator(numberFloor)
  // если есть свободный лифт
  if(nearestElevator) {
    nearestElevator.state = 'called'
    nearestElevator.floor_call = numberFloor
    if(nearestElevator.floor_call > nearestElevator.current_floor) nearestElevator.direction = 1
    if(nearestElevator.floor_call < nearestElevator.current_floor) nearestElevator.direction = -1

    const intervalId = setInterval(() => {
      nearestElevator.current_floor = nearestElevator.current_floor + nearestElevator.direction
    }, 1000)

    setTimeout(() => {
      elevatorArrived(nearestElevator.id)
      clearInterval(intervalId)
    }, (Math.abs(nearestElevator.floor_call - nearestElevator.current_floor) * 1000))
  // если нет свободного лифтра
  } else {
    queue.value.push(numberFloor)
  }
}

function cheackElevatorOnFloor(numberFloor) {
  for (const elevator of elevators.value) {
    if (elevator.state === 'free' && elevator.current_floor === numberFloor) {
      return true
    }
  }
  return false
}

function cheackElevatorToCalled(numberFloor) {
  for (const elevator of elevators.value) {
    if (elevator.state === 'called' && elevator.floor_call === numberFloor) {
      return true
    }
  }
  return false
}

function findNearestElevator(floor) {
  const availableElevators = elevators.value.filter(elevator => elevator.state === "free");

  if (availableElevators.length === 0) {
    return null;
  }

  return availableElevators.reduce((closestElevator, elevator) => {
    const distance = Math.abs(elevator.current_floor - floor);
    const closestDistance = Math.abs(closestElevator.current_floor - floor);
    return distance < closestDistance ? elevator : closestElevator;
  });
}

const initBuilding = () => {
  const elevatorsInfo = localStorage.getItem('elevators')
  // const paramsInfo = localStorage.getItem('params')
  // если параметры изменились
  // if(params && params !== paramsInfo) {
  //   for (let i = 0; i < params.elevators; i++) {
  //     elevators.value.push({id: i, current_floor: 0, state: 'free', floor_call: null, direction: null,})
  //     localStorage.setItem('elevators', JSON.stringify(elevators.value))
  //   }
  //   return
  // }
  //если нет в localstorage состояние лифтов
  if(!elevatorsInfo) {
    for (let i = 0; i < params.elevators; i++) {
      elevators.value.push({id: i, current_floor: 0, state: 'free', floor_call: null, direction: null,})
      localStorage.setItem('elevators', JSON.stringify(elevators.value))
    }
  } else { //если есть в localstorage состояние лифтов
    elevators.value = [...JSON.parse(elevatorsInfo)]
  }
}

const elevatorArrived = (idElevator) => {
  const calledElevator = elevators.value.find(elevator => elevator.id === idElevator);

  if (calledElevator) {
    calledElevator.state = 'paused';

    setTimeout(() => {
      calledElevator.state = 'free';
      calledElevator.direction = null;

    }, params.pause);
  }
};

const freeElevatorCall = (elevator) => {
  if(queue.value.length) {
    const floorNumber = queue.value.shift()
    elevator.state = 'called'
    elevator.floor_call = floorNumber
    if(elevator.floor_call > elevator.current_floor) elevator.direction = 1
    if(elevator.floor_call < elevator.current_floor) elevator.direction = -1

    const intervalId = setInterval(() => {
      elevator.current_floor = elevator.current_floor + elevator.direction
    }, 1000)

    setTimeout(() => {
      elevatorArrived(elevator.id)
      clearInterval(intervalId)
    }, (Math.abs(elevator.floor_call - elevator.current_floor) * 1000))
  }
}

const synchronizationLocalStorage = () => {
  localStorage.setItem('elevators', JSON.stringify(elevators.value))
  localStorage.setItem('queue', JSON.stringify(elevators.value))
  localStorage.setItem('params', JSON.stringify(params.value))
}

watch([() => params, elevators, queue], () => {
  synchronizationLocalStorage()
}, { deep: true });

onMounted(() => {
  initBuilding()
})

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
