<template>
   
        <svg width="800" height="500">
            <circle r="10"
                    v-for="(item, index) in dataset"
                    :cx="item[0]"
                    :cy="item[1]"
                    :key="index"
                    fill="#000"
                    transform="translate(350, 320)" />
        </svg>
   


</template>

<script>
    import dataset from './dataset'
    export default {
        data() {
            return {
                dataset
            }
        }
    }

</script>

<style>

</style>