<template>
    <div class="building__elevator">
        <div class="building__elevator-cabin" :style="{ height: props.floorHeightPercent + '%'}"></div>
    </div>
</template>

<script setup>
const props = defineProps({
    floorHeightPercent: {
        type: Number,
        required: true,
    }
})
</script>

<style lang="scss" scoped>

</style>