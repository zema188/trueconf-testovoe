<template>
    <div class="building__elevator"
        :class="{ called: props.elevator.state === 'called',
                paused: props.elevator.state === 'paused',
                down: props.elevator.direction === -1
        }"
    >
        <div class="building__elevator-cabin"
            ref="cabin"
            :style="{ height: props.floorHeightPercent + '%',
                    bottom: elevator.floor_call * floorHeightPercent  + '%',
                    transition: calcTransition + 's' + ' linear'
            }"
        >
            <span class="building__elevator-current-floor" v-show="props.elevator.state === 'called' "> to {{ props.elevator.floor_call + 1 }}</span>
            {{ props.elevator.current_floor + 1}}
            <img src="@/assets/images/icons/arrow.svg" class="icon"
                v-show="props.elevator.state === 'called'"
            >
        </div>
    </div>
</template>

<script setup>
import { onMounted, watch, computed, ref, onUpdated } from "vue"

const props = defineProps({
    floorHeightPercent: {
        type: Number,
        required: true,
    },
    floorHeight: {
        type: Number,
        required: true,
    },
    elevator: {
        type: Object,
        required: true,
    }
})
const emit = defineEmits(['freeElevatorCall'])
watch(
    () => props.elevator,
    () => {
        if(props.elevator.state === 'free') {
            emit('freeElevatorCall', props.elevator)
        }
    },
    { deep: true }
)

const cabin = ref(null)

const calcTransition = computed(() => {
    return Math.abs(props.elevator.floor_call - props.elevator.current_floor)
})

onMounted(() => {
    // если лифт уже едет
    if(props.elevator.state === 'called') {
        cabin.value.style.bottom = `${props.elevator.current_floor * props.floorHeight}px`
        emit('calledOnMounted', props.elevator)
        setTimeout(() => {
            cabin.value.style.bottom = props.elevator.floor_call * props.floorHeightPercent  + '%'
        },0)
    }
    // если лифт на паузу
    if(props.elevator.state === 'paused') {
        emit('elevatorArrived', props.elevator.id)
    }
})
onUpdated(() => {
})
</script>

<style lang="scss" scoped>

</style>