<template>
    <div class="building__floor">
        <button class="building__floor-btn"
            :class="{called: cheackForDisabled, calledInQueue: isCalled}"
            :disabled="isCalled"
            @click="callElevator()"
        >
            {{ props.number + 1}}
            <span>
                
            </span>
        </button>
    </div>
</template>

<script setup>
import { computed, ref } from "vue"

const props = defineProps({
    number: {
        type: Number,
        required: true,
    },
    elevatorsInfo: {
        type: Object,
        required: true,
    },
    queue: {
        type: Array,
        required: true,
    },
})

// создаем вызов на этаж
const emit = defineEmits(['callElevator'])

//если на этаж вызван лсифт то кнопка загарается красной
const cheackForDisabled = computed(() => {
    return props.elevatorsInfo.filter(elevator => (elevator.state === 'called' && elevator.floor_called === props.number)).length
})

//если этаж в очереди 
const isCalled = computed(() => {
    return props.queue.includes(props.number) && isClicked.value;
});

let isClicked = ref(false);

function callElevator() {
    emit("callElevator", props.number);
    isClicked.value = true
    setTimeout(() => {
        isClicked.value = false
    }, 1000)
}

</script>

<style lang="scss" scoped>

</style>