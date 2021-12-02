<template>
    <div style="width: 100%; height: 100%;">
        <input type="text" v-model="curve" />
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
        <div style="margin-top: 20px;">
            <input v-model.number="a">
            <br>
            <br>
            <input v-model.number="b">
            <h2>The sum is {{ c }}</h2>
            <h4>Reduced {{ reduced }}</h4>
        </div>

        <svg width="800"
             height="500"
             @click="addPoint" 
             @contextmenu.prevent="points = []" >

            <path stroke="green"
                  fill="none"
                  stroke-width="5"
                  :d="e"
                  transform="translate(0, -450)"   />
            
            <circle v-for="(item, index) in points"
                    :key="index"
                    r="10"
                    :cx="item.x"
                    :cy="item.y"
                    fill="#000"
                    transform="translate(0, -450)"
                    />
        </svg>

        <svg width="800"
             height="500">
             <circle v-for="(item, index) in h.descendants()"
                    :key="index"
                    r="item.r"
                    :cx="item.x"
                    :cy="item.y"
                    fill="#000"
                    transform="translate(0, -450)"
                    />
        </svg>
    </div>


</template>

<script>
    import * as d3 from 'd3'
    import dataset from './dataset'
    

    export default {
        data() {
            return {
                dataset,
                curve: 'curveNatural',
                a: 0,
                b: 0,
                reduced: 0,

                points: [
                    {
                        x: 200,
                        y: 500
                    }
                ],

                width: 100,
                height: 100,
                padding: 2,
                h: d3.hierarchy({}),
                groupOrder: ['region', 'subregion'],
                new_dataset: []
            }
        },
       
        methods: {
            onClick(item) {
                this.curve =
                 this.curve === 'curveStepAfter' ? 'curveStepBefore' : 'curveStepAfter'
                console.log('hey you', item)
            },
            subtract() {
                this.reduced -= .1

                if (this.reduced > .001) {
                    requestAnimationFrame(this.subtract)
                } else {
                    this.reduced = 0
                }
            },
            addPoint(ev) {
                const {
                    layerX: x,
                    layerY: y
                } = ev
                this.points.push({
                    x,y
                })
            },
            updateSize() {
                const {
                    width,
                    height
                } = this.$el.getBoundingClientRect()

                this.width = width
                this.height = height
            }

        },

        computed: {
            lineGenerator() {
                return d3.line()
                         .curve(d3[this.curve])
                         .x(v => v[0])
                         .y(v => v[1])   
            },
            d() {
                return this.lineGenerator(this.dataset)
            },
            c() {
                return this.a + this.b
            },
            newlineGenerator() {
                return d3.line()
                .x(item => item.x)
                .y(item => item.y)
                .curve(d3.curveCardinalClosed)

            },
            e() {
                return this.newlineGenerator(this.points)
            },

            layout() {
                return d3.pack()
                         .size([this.width, this.height])
                         .padding(this.padding)
            },
            nester() {
                const n = d3.nest()
                this.groupOrder.forEach(val => {
                    n.key(node => node[val])
                })

                return n
            },
            nestedData() {
                return {
                    key: 'root',
                    values: this.nester.entries(this.new_dataset)
                }
            }

        },

        watch: {
            c(val) {
                this.reduced = val 
                this.subtract()
            },
            layout() {
                this.layout(this.h)
            },
            nestedData(val) {
                const h = d3.hierarchy(val, z => z.values)
            
                h.sum(z => z.value)
                h.sort((g,i) => d3.ascending(g.value, i.value ))
            
                this.layout(h)
                this.h = h
            }
        },
        async mounted() {
            //1. Update Size
            this.updateSize()
            //2. Load data
            const new_data = await d3.json('./populations.json')
            //3. Apply the data as read only to our dataset
            this.new_dataset = Object.freeze(new_data)
        },
        
    }

</script>

<style scoped>
    path {
        transition: all 500ms ease;

    }

</style>