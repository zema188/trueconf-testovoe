<template>
  <main>
    <div class="building">
      <div class="building__floors-list">
        <the-floor
          v-for="(floor, index) in params.floors"
          :key="index"
          :number="index"
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
        >
        </the-elevator>
      </div>
    </div>
  </main>
</template>

<script setup>
import { computed, onMounted, ref, watch } from 'vue'
import TheElevator from './components/TheElevator.vue'
import TheFloor from './components/TheFloor.vue'

const params = {
  floors: 5,
  elevators: 5,
  pause: 3000,
}

let elevators = ref([])

const floorHeightPercent = computed(() => {
  return 100 / params.floors
})

const buildMinHeight = computed(() => {
  return params.floors * 90
})

const callElevator = (numberFloor) => {
  const nearestElevator = findNearestElevator(numberFloor)
  if(nearestElevator) {
    nearestElevator.state = 'called'
    nearestElevator.floor_call = numberFloor
    if(nearestElevator.floor_call > nearestElevator.current_floor) nearestElevator.direction = 1
    else nearestElevator.direction = -1
    const intervalId = setInterval(() => {
      nearestElevator.current_floor = nearestElevator.current_floor + nearestElevator.direction
    }, 1000)

    setTimeout(() => {
      elevatorArrived(nearestElevator.id)
      clearInterval(intervalId)
    }, (Math.abs(nearestElevator.floor_call - nearestElevator.current_floor) * 1000))

  }
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
  if(!elevatorsInfo) {
    for (let i = 0; i < params.elevators; i++) {
      elevators.value.push({id: i, current_floor: 0, state: 'free', floor_call: null, direction: null,})
      localStorage.setItem('elevators', JSON.stringify(elevators.value))
    }
  } else {
    elevators.value = [...JSON.parse(elevatorsInfo)]
  }
}

const elevatorArrived = (idElevator) => {
  const calledElevator = elevators.value.find(elevator => elevator.id === idElevator);

  if (calledElevator) {
    calledElevator.state = 'paused';

    setTimeout(() => {
      calledElevator.state = 'free';
    }, params.pause);
  }
};

watch(
    () => elevators,
    () => {
      localStorage.setItem('elevators', JSON.stringify(elevators.value))
    },
    { deep: true }
)

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
    &__floor-btn {
      cursor: pointer;
      padding-left: 10px;
      display: flex;
      flex-direction: column;
      justify-content: center;
      min-height: 100%;
      width: fit-content;
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
    background-color: red; /* Начальный цвет (красный) */
  }
  50% {
    background-color: green; /* Начальный цвет (красный) */
  }
  100% {
    background-color: red; /* Конечный цвет (зеленый) */
  }
}
</style>
