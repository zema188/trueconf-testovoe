<template>
  <main>
    <div class="building">
      <div class="building__floors-list">
        <the-floor
          v-for="(floor, index) in params.floors"
          :key="index"
          :number="index"
          @callElevator="(number) => callElevator(number)"
        >
        </the-floor>
      </div>
      <div class="building__elevators-list" :style="{ minHeight: buildHeight + 'px'}">
        <the-elevator
          v-for="(elevator, index) in params.elevator"
          :key="index"
          :floorHeightPercent="floorHeightPercent"
        >
        </the-elevator>
      </div>
    </div>
  </main>
</template>

<script setup>
import { computed } from 'vue'
import TheElevator from './components/TheElevator.vue'
import TheFloor from './components/TheFloor.vue'

const params = {
  floors: 5,
  elevator: 5
}

const floorHeightPercent = computed(() => {
  return 100 / params.floors
})

const buildHeight = computed(() => {
  return params.floors * 90
})

const callElevator = (number) => {
  console.log(number)
}

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
    }
    &__elevator-cabin {
      width: 100%;
      background: rgb(114, 114, 243);
      position: absolute;
      left: 0;
      bottom: 0;
      transition: .2s;
      min-height: 90px;
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
</style>
