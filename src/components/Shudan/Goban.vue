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
        width: {
            type: Number,
            default: 19
        },

        height: {
            type: Number,
            default: 19
        },

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
                const { height } = this;
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

        dimmedMap: {
            type: Array,
            default: undefined
        },

        selectedMap: {
            type: Array,
            default: undefined
        }
    },

    data: function () {
        return {
            animatedVertices: [],
            clearAnimatedVerticesHandler: null
        };
    },

    computed: {
        size() {
            const { width, height } = this;
            return width * height;
        },

        shiftMap() {
            const { size, width, readjustShifts } = this;
            const shiftMap = [...Array(size)].map(() => helper.random(8));
            shiftMap.forEach((_, i) => readjustShifts(shiftMap, i, width));
            return shiftMap;
        },

        randomMap() {
            const { size } = this;
            return [...Array(size)].map(() => helper.random(4));
        },

        xs() {
            const { width, rangeX } = this;
            return helper.range(width).slice(rangeX[0], rangeX[1] + 1);
        },

        ys() {
            const { height, rangeY } = this;
            return helper.range(height).slice(rangeY[0], rangeY[1] + 1);
        },

        hoshis() {
            const { width, height } = this;
            return helper.getHoshis(width, height);
        },

        vertices() {
            const { xs, ys, fuzzyStonePlacement, shiftMap, randomMap,
                    signMap, heatMap, paintMap, markerMap, ghostStoneMap,
                    dimmedMap, selectedMap, animatedVertices, width} = this;
            const result = [];
            ys.forEach(y => { xs.forEach(x => {
                const offset = y * width + x;
                result.push({
                    offset,
                    shift: fuzzyStonePlacement ? shiftMap && shiftMap[offset] : 0,
                    random: randomMap && randomMap[offset],
                    sign: signMap && signMap[offset],
                    heat: heatMap && heatMap[offset],
                    paint: paintMap && paintMap[offset],
                    marker: markerMap && markerMap[offset],
                    ghostStone: ghostStoneMap && ghostStoneMap[offset],
                    dimmed: dimmedMap && dimmedMap[offset],
                    selected: selectedMap && selectedMap[offset],
                    animate: animatedVertices && animatedVertices.some(v => v === offset)
                });
            });});
            return result;
        }
    },

    watch: {
        size: {
            handler() {
                this.animatedVertices = [];
                this.clearAnimatedVerticesHandler = null;
            },
            immediate: true
        },

        signMap(newSignMap, oldSignMap) {
            const { animateStonePlacement, fuzzyStonePlacement, clearAnimatedVerticesHandler } = this;
            if (animateStonePlacement
                && fuzzyStonePlacement
                && clearAnimatedVerticesHandler == null
            ) {
                const animatedVertices = helper.diffSignMap(oldSignMap, newSignMap);
                if (animatedVertices.length > 0) {
                    this.animatedVertices = animatedVertices;
                    this.updateAnimatedVertices();
                }
            }
        }
    },

    methods: {
        updateAnimatedVertices() {
            if (this.animatedVertices.length > 0) {
                // 触发落子滑动效果
                for (let i of this.animatedVertices) {
                    this.$set(this.shiftMap, i, helper.random(7) + 1);
                    this.readjustShifts(this.shiftMap, i, this.width);
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
        },

        readjustShifts(shiftMap, offset, width) {
            const shift = shiftMap[offset];
            if (!shift) return;

            const table = [
                [[1, 5, 8], offset - 1, [3, 7, 6]],     // 落点偏左，撞到落点左方偏右的棋子
                [[2, 5, 6], offset - width, [4, 7, 8]], // 上撞下
                [[3, 7, 6], offset + 1, [1, 5, 8]],     // 右撞左
                [[4, 7, 8], offset + width, [2, 5, 6]], // 下撞上
            ];

            for (let [directions, nearOffset, removeShifts] of table) {
                const nearShift = shiftMap[nearOffset];
                if (nearShift
                    && directions.includes(shift)
                    && removeShifts.includes(nearShift))
                    shiftMap.splice(nearOffset, 1, 0); // 被撞到中位
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
                :key="v.offset"
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
                @click="$emit('click', v.offset)"
                @mousedown="$emit('mousedown', v.offset)"
                @mouseup="$emit('mouseup', v.offset)"
                @mousemove="$emit('mousemove', v.offset)"
                @mouseenter="$emit('mouseenter', v.offset)"
                @mouseleave="$emit('mouseleave', v.offset)"
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
