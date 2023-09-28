<template>
    <div class="building__elevator"
        :class="{ called: props.elevator.state === 'called',
                paused: props.elevator.state === 'pause',
                down: props.elevator.direction === -1
        }"
    >
        <div class="building__elevator-cabin"
            ref="cabin"
            :style="{ height: props.floorHeightPercent + '%',
                    bottom: props.elevator.position_y + '%',
            }"
        >
            <span class="building__elevator-current-floor" v-show="props.elevator.state === 'called' "> to {{ props.elevator.floor_called + 1 }}</span>
            {{ props.elevator.floor_current + 1}}
            <img src="@/assets/images/icons/arrow.svg" class="icon"
                v-show="props.elevator.state === 'called'"
            >
        </div>
    </div>
</template>

<script setup>
import { onMounted, watch, ref, onUpdated } from "vue"

const props = defineProps({
    elevator: {
        type: Object,
        required: true,
    },
    floorHeightPercent: {
        type: Number,
        required: true,
    },
    floorHeight: {
        type: Number,
        required: true,
    },
})

const emit = defineEmits(['freeElevatorCall'])

const originalHeight = ref(null)

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


onMounted(() => {
    originalHeight.value = props.floorHeight
})

onUpdated(() => {
})
</script>

<style lang="scss" scoped>

</style>