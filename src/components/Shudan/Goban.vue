<script>
import Coord from './Coord.vue';
import Grid from './Grid.vue';
import Vertex from './Vertex.vue';
import ULine from './Line.vue';
import helper from './helper.js';
import { setTimeout } from 'timers';

const defaultVertexSize = 24;
const readjustShifts = function (shiftMap, boardSize, offset) {
    const shift = shiftMap[offset];
    if (!shift) return;

    const table = [
        [[1, 5, 8], offset - 1, [3, 7, 6]],         // 落点偏左，撞到落点左方偏右的棋子
        [[2, 5, 6], offset - boardSize, [4, 7, 8]], // 上撞下
        [[3, 7, 6], offset + 1, [1, 5, 8]],         // 右撞左
        [[4, 7, 8], offset + boardSize, [2, 5, 6]], // 下撞上
    ];
    for (let [directions, nearOffset, removeShifts] of table) {
        const nearShift = shiftMap[nearOffset];
        if (nearShift
            && directions.includes(shift)
            && removeShifts.includes(nearShift))
            shiftMap.splice(nearOffset, 1, 0); // 被撞到中位
    }
};

export default {
    components: {
        Coord,
        Grid,
        Vertex,
        ULine
    },

    props: {
        boardSize: {
            type: Number,
            default: 19
        },

        maxWidth: {
            type: Number,
            default: 480
        },

        maxHeight: {
            type: Number,
            default: 480
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
                const { boardSize } = this;
                return [...Array(boardSize)].map((_, i) => boardSize - i);
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
            clearAnimatedVerticesHandler: null,
            offsetWidth: undefined,
            offsetHeight: undefined,
            fontSize: undefined,
            updatingSize: false
        };
    },

    computed: {
        vertexSize() {
            const { offsetWidth, offsetHeight, fontSize } = this;
            if (!offsetWidth || !offsetHeight || !fontSize) {
                this.updateElementSize();
                return defaultVertexSize;
            }

            const { maxWidth, maxHeight } = this;
            const scale = Math.min(maxWidth / offsetWidth, maxHeight / offsetHeight);
            const vertexSize = Math.max(Math.floor(fontSize * scale), 1);
            return vertexSize;
        },

        shiftMap() {
            const { boardSize } = this;
            const shiftMap = [...Array(boardSize * boardSize)].map(() => helper.random(8));
            shiftMap.forEach((_, i) => readjustShifts(shiftMap, boardSize, i));
            return shiftMap;
        },

        randomMap() {
            const { boardSize } = this;
            return [...Array(boardSize * boardSize)].map(() => helper.random(4));
        },

        xs() {
            this.updateElementSize();
            const { boardSize, rangeX } = this;
            return helper.range(boardSize).slice(rangeX[0], rangeX[1] + 1);
        },

        ys() {
            this.updateElementSize();
            const { boardSize, rangeY } = this;
            return helper.range(boardSize).slice(rangeY[0], rangeY[1] + 1);
        },

        hoshis() {
            const { boardSize} = this;
            return helper.getHoshis(boardSize, boardSize);
        },

        vertices() {
            const { xs, ys, fuzzyStonePlacement, shiftMap, randomMap,
                    signMap, heatMap, paintMap, markerMap, ghostStoneMap,
                    dimmedMap, selectedMap, animatedVertices, boardSize} = this;
            const result = [];
            ys.forEach(y => { xs.forEach(x => {
                const offset = y * boardSize + x;
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
        boardSize: {
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
        },

        showCoordinates() {
            this.updateElementSize();
        }
    },

    methods: {
        updateAnimatedVertices() {
            if (this.animatedVertices.length > 0) {
                // 触发落子滑动效果
                for (let i of this.animatedVertices) {
                    this.$set(this.shiftMap, i, helper.random(7) + 1);
                    readjustShifts(this.shiftMap, this.boardSize, i);
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

        updateElementSize() {
            this.$nextTick(() => {
                const element = this.$refs['goban-root'];
                if (element) {
                    this.offsetWidth = element.offsetWidth;
                    this.offsetHeight = element.offsetHeight;
                    this.fontSize = parseInt(element.style.fontSize) || defaultVertexSize;
                } else {
                    this.updateElementSize();
                }
            });
        }
    }
};
</script>

<template>
<div>
    <div
        ref="goban-root"
        :class="[
            'shudan-goban shudan-goban-image',
            {
                'shudan-busy': busy,
                'shudan-coordinates': showCoordinates
            }
        ]"
        :style="`font-size: ${vertexSize}px;`"
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
                :width="boardSize"
                :height="boardSize"
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
                        width: `${boardSize}em`,
                        height: `${boardSize}em`,
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
