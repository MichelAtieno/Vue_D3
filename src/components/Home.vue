<template>
   
        <svg width="800" height="500">
            <path stroke="green"
                  fill="none"
                  stroke-width="5"
                  :d="d"
                  transform="translate(350, 320)"   />
            <circle r="10"
                    v-for="(item, index) in dataset"
                    :cx="item[0]"
                    :cy="item[1]"
                    :key="index"
                    fill="#000"
                    transform="translate(350, 320)"
                    @click="onClick(item)" />
        </svg>
   


</template>

<script>
    import * as d3 from 'd3'
    import dataset from './dataset'
    export default {
        data() {
            return {
                dataset
            }
        },
        methods: {
            onClick(item) {
                console.log('hey you', item)
            }
        },
        computed: {
            lineGenerator() {
                return d3.line()
                         .x(v => v[0])
                         .y(v => v[1])   
            },
            d() {
                return this.lineGenerator(this.dataset)
            }
        }
    }

</script>

<style>

</style>