<template>
    <div class="building__floor">
        <button class="building__floor-btn"
            :class="{called: cheackForDisabled}"
            @click="callElevator()"
        >
            {{ props.number + 1}}
            <span>
                
            </span>
        </button>
    </div>
</template>

<script setup>
import { computed } from "vue"

const props = defineProps({
    number: {
        type: Number,
        required: true,
    },
    elevatorsInfo: {
        type: Object,
        required: true,
    }
})

const emit = defineEmits(['callElevator'])

function callElevator() {
  emit('callElevator', props.number)
}

const cheackForDisabled = computed(() => {
    return props.elevatorsInfo.filter(elevator => (elevator.state === 'called' && elevator.floor_call === props.number)).length
})
</script>

<style lang="scss" scoped>

</style>