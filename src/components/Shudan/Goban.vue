<script>
import Coord from './Coord.vue';
import Grid from './Grid.vue';
import Vertex from './Vertex.vue';
import ULine from './Line.vue';
import helper from './helper.js';
import { setTimeout } from 'timers';

export default {
    components: {
        Coord,
        Grid,
        Vertex,
        ULine
    },

    props: {
        vertexSize: {
            type: Number,
            default: 16
        },
    
        animate: {
            type: Boolean,
            default: false
        },

        busy: {
            type: Boolean,
            default: false
        },
    
        rangeX: {
            type: Array,
            default: () =>  [0, Infinity]
        },
    
        rangeY: {
            type: Array,
            default: () =>  [0, Infinity]
        },

        signMap: {
            type: Array,
            default: () => []
        },

        coordX: {
            type: Array,
            default: () => [...helper.alpha]
        },

        coordY: {
            type: Array,
            default() {
                const height = this.signMap.length;
                return [...Array(height)].map((_, i) => height - i);
            }
        },

        showCoordinates: {
            type: Boolean,
            default: false
        },

        fuzzyStonePlacement: {
            type: Boolean,
            default: false
        },

        animateStonePlacement: {
            type: Boolean,
            default: false
        },
    
        paintMap: {
            type: Array,
            default: undefined
        },

        heatMap: {
            type: Array,
            default: undefined
        },

        markerMap: {
            type: Array,
            default: undefined
        },
    
        ghostStoneMap: {
            type: Array,
            default: undefined
        },

        lines: {
            type: Array,
            validator(params) {
                const keys = ["type", "v1", "v2"];
                for (let p of params)
                    for (let k of keys)
                        if (!(k in p))
                            return false;
                return true;
            },
            default: undefined
        },

        dimmedVertices: {
            type: Array,
            default: undefined
        },

        selectedVertices: {
            type: Array,
            default: undefined
        }
    },

    data: function () {
        return {
            width: 0,
            height: 0,
            hoshis: [],
            shiftMap: [],
            randomMap: [],
            animatedVertices: [],
            clearAnimatedVerticesHandler: null,
            helper
        };
    },

    watch: {
        signMap: {
            handler(signMap, oldSignMap) {
                let width = signMap.length === 0 ? 0 : signMap[0].length;
                let height = signMap.length;
                if (this.width === width && this.height === height) {
                    if (this.animateStonePlacement
                        && this.fuzzyStonePlacement
                        && this.clearAnimatedVerticesHandler == null
                    ) {
                        this.animatedVertices = helper.diffSignMap(oldSignMap, signMap);
                        this.updateAnimatedVertices();
                    }
                } else { // 尺寸变化
                    this.width = width;
                    this.height = height;
                    this.hoshis = helper.getHoshis(width, height);
                    this.shiftMap = helper.readjustShifts(signMap.map(row => row.map(() => helper.random(8))));
                    this.randomMap = signMap.map(row => row.map(() => helper.random(4)));
                    this.animatedVertices = [];
                    this.clearAnimatedVerticesHandler = null;
                }
            },
            immediate: true
        }
    },

    computed: {
        xs() {
            let { width, rangeX } = this;
            return helper.range(width).slice(rangeX[0], rangeX[1] + 1);
        },

        ys() {
            let { height, rangeY } = this;
            return helper.range(height).slice(rangeY[0], rangeY[1] + 1);
        },

        vertices() {
            const { xs, ys, fuzzyStonePlacement, shiftMap, randomMap,
                    signMap, heatMap, paintMap, markerMap, ghostStoneMap,
                    dimmedVertices, selectedVertices, animatedVertices} = this;
            const result = [];
            ys.forEach(y => { xs.forEach(x => {
                result.push({
                    key: `${x}-${y}`,
                    position: [x, y],
                    shift: fuzzyStonePlacement ? shiftMap && shiftMap[y] && shiftMap[y][x] : 0,
                    random: randomMap && randomMap[y] && randomMap[y][x],
                    sign: signMap && signMap[y] && signMap[y][x],
                    heat: heatMap && heatMap[y] && heatMap[y][x],
                    paint: paintMap && paintMap[y] && paintMap[y][x],
                    marker: markerMap && markerMap[y] && markerMap[y][x],
                    ghostStone: ghostStoneMap && ghostStoneMap[y] && ghostStoneMap[y][x],
                    dimmed: dimmedVertices && dimmedVertices.some(v => helper.vertexEquals(v, [x, y])),
                    selected: selectedVertices && selectedVertices.some(v => helper.vertexEquals(v, [x, y])),
                    animate: animatedVertices && animatedVertices.some(v => helper.vertexEquals(v, [x, y]))
                });
            });});
            return result;
        }
    },

    methods: {
        updateAnimatedVertices() {
            if (this.animatedVertices.length > 0) {
                // 触发落子滑动效果
                for (let [x, y] of this.animatedVertices) {
                    this.$set(this.shiftMap[y], x, helper.random(7) + 1);
                    helper.readjustShifts(this.shiftMap, [x, y]);
                }

                // 延后清除效果(这样后续还可以再触发)
                this.$nextTick(function () {
                    this.clearAnimatedVerticesHandler = setTimeout(
                        () => {
                            this.animatedVertices = [];
                            this.clearAnimatedVerticesHandler = null;
                        },
                        200,
                    );
                });
            }
        }
    }
};
</script>

<template>
<div
    :class="[
        'shudan-goban shudan-goban-image',
        {
            'shudan-busy': busy,
            'shudan-coordinates': showCoordinates
        }
    ]"
    :style="`fontSize: ${vertexSize}px;`"
    >
    <!-- 中心区 -->
    <div
        class="center shudan-content"
        :style="{
            position: 'relative',
            width: `${xs.length}em`,
            height: `${ys.length}em`,
        }"
        >
        <!-- 棋盘网线及星位 -->
        <Grid
            :width="width"
            :height="height"
            :xs="xs"
            :ys="ys"
            :hoshis="hoshis"
            />

        <!-- 落点区域 -->
        <div
            class="shudan-vertices"
            :style="{
                display: 'grid',
                gridTemplateColumns: `repeat(${xs.length}, 1em)`,
                gridTemplateRows: `repeat(${ys.length}, 1em)`
            }"
            >
            <Vertex
                v-for="v in vertices"
                :key="v.key"
                :shift="v.shift"
                :random="v.random"
                :sign="v.sign"
                :heat="v.heat"
                :paint="v.paint"
                :marker="v.marker"
                :ghost-stone="v.ghostStone"
                :dimmed="v.dimmed"
                :selected="v.selected"
                :animate="v.animate"
                @click="$emit('click', v.position)"
                @mousedown="$emit('mousedown', v.position)"
                @mouseup="$emit('mouseup', v.position)"
                @mousemove="$emit('mousemove', v.position)"
                @mouseenter="$emit('mouseenter', v.position)"
                @mouseleave="$emit('mouseleave', v.position)"
                />
        </div>

        <!-- 指示线 -->
        <div class="shudan-lines">
            <div
                :style="{
                    position: 'absolute',
                    top: `-${rangeY[0]}em`,
                    left: `-${rangeX[0]}em`,
                    width: `${width}em`,
                    height: `${height}em`,
                }"
                >
                <ULine
                    v-for="(l, i) in lines"
                    :key="i"
                    :v1="l.v1"
                    :v2="l.v2"
                    :type="l.type"
                    />
            </div>
        </div>
    </div>

    <!-- 上左右下的刻度标签 -->
    <Coord
        v-if="showCoordinates"
        class="top"
        dir="x"
        :sets="xs"
        :labels="coordX"
        />
    <Coord
        v-if="showCoordinates"
        class="left"
        dir="y"
        :sets="ys"
        :labels="coordY"
        />
    <Coord
        v-if="showCoordinates"
        class="right"
        dir="y"
        :sets="ys"
        :labels="coordY"
        />
    <Coord
        v-if="showCoordinates"
        class="bottom"
        dir="x"
        :sets="xs"
        :labels="coordX"
        />
</div>
</template>

<style scoped>
@import "./css/goban.css";

.shudan-goban {
        line-height: 1em;
        display: inline-grid;
    }
    .shudan-goban.shudan-coordinates {
        grid-template-rows: 1em 1fr 1em;
        grid-template-columns: 1em 1fr 1em;
    }
    .shudan-goban:not(.shudan-coordinates) {
        grid-template-rows: 0 1fr 0;
        grid-template-columns: 0 1fr 0;
}

.top.shudan-coordx {
    grid-area: 1 / 2 / auto / auto;
}

.left.shudan-coordy {
    grid-area: 2 / 1 / auto / auto;
}

.center.shudan-content {
    grid-area: 2 / 2 / auto / auto;
}

.right.shudan-coordy {
    grid-area: 2 / 3 / auto / auto;
}

.bottom.shudan-coordx {
    grid-area: 3 / 2 / auto / auto;
}

.shudan-vertices {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 1;
}

.shudan-lines {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    overflow: hidden;
    pointer-events: none;
    z-index: 2;
}
</style>
