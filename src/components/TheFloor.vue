<template>
    <div class="building__floor">
        <button class="building__floor-btn"
            @click="callElevator()"
        >
            {{ props.number + 1}}
            <span></span>
        </button>
    </div>
</template>

<script setup>
const props = defineProps({
    number: {
        type: Number,
        required: true,
    }
})

const emit = defineEmits(['callElevator'])

function callElevator() {
  emit('callElevator', props.number)
}
</script>

<style lang="scss" scoped>

</style>