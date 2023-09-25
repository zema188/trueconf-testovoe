<template>
    <div class="building__elevator"
        :class="{ called: props.elevator.state === 'called',
                paused: props.elevator.state === 'paused',
        }"
    >
        <div class="building__elevator-cabin"
            :style="{ height: props.floorHeightPercent + '%',
                    bottom: elevator.floor_call * floorHeightPercent  + '%',
                    transition: calcTransition + 's' + ' linear'
            }"
        >
            <span v-show="props.elevator.state === 'called' ">{{ props.elevator.floor_call + 1 }}</span>
            {{ props.elevator.state }}
            {{ props.elevator.current_floor + 1}}
            <img src="@/assets/images/icons/arrow.svg" class="icon"
                v-show="props.elevator.state === 'called'"
            >
        </div>
    </div>
</template>

<script setup>
import { onMounted, watch, computed } from "vue"

const props = defineProps({
    floorHeightPercent: {
        type: Number,
        required: true,
    },
    elevator: {
        type: Object,
        required: true,
    }
})

watch(
    () => props.elevator,
    () => {
        
    },
    { deep: true }
)

const calcTransition = computed(() => {
    return Math.abs(props.elevator.floor_call - props.elevator.current_floor)
})

onMounted(() => {
    if(props.elevator === 'called') {
        true
    }
})

</script>

<style lang="scss" scoped>

</style>