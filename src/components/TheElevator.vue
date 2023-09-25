<template>
    <div class="building__elevator"
        :class="{ called: props.elevator.state === 'called'}"
    >
        <div class="building__elevator-cabin"
            :style="{ height: props.floorHeightPercent + '%', bottom: elevator.floor_call * floorHeightPercent  + '%'}"
        >
        {{ props.elevator.state === 'called' }}
        </div>
    </div>
</template>

<script setup>
import { onMounted, watch } from "vue"

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
    );
onMounted(() => {
    if(props.elevator === 'called') {
        true
    }
})

</script>

<style lang="scss" scoped>

</style>