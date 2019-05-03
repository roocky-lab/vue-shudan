<template>
<div
    class="shudan-goban shudan-goban-image"
    :class="{'shudan-busy': busy, 'shudan-coordinates': showCoordinates}"
    :style="{
        'display': 'inline-grid',
        'gridTemplateRows': showCoordinates ? '1em 1fr 1em' : '1fr',
        'gridTemplateColumns': showCoordinates ? '1em 1fr 1em' : '1fr',
        'fontSize': `${vertexSize}px`,
        'lineHeight': '1em'
    }"
>
    <!-- 上侧及左侧坐标标签 -->
    <CoordX
        v-if="showCoordinates"
        :xs="xs"
        style="grid-area: 1 / 2 / auto / auto;"
        :coord-x="coordX"
    />
    <CoordY
        v-if="showCoordinates"
        :height="height"
        :ys="ys"
        style="grid-area: 2 / 1 / auto / auto;"
        :coord-y="coordY"
    />

    <!-- 中心区 -->
    <div
        class="shudan-content"
        :style="{
            position: 'relative',
            width: `${xs.length}em`,
            height: `${ys.length}em`,
            gridRow: showCoordinates ? '2' : '1',
            gridColumn: showCoordinates ? '2' : '1'
        }"
    >
        <!-- 棋盘网线及星位 -->
        <Grid
            :vertex-size="vertexSize"
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
                gridTemplateRows: `repeat(${ys.length}, 1em)`,
                position: 'absolute',
                top: 0,
                left: 0,
                right: 0,
                bottom: 0,
                zIndex: 1
            }"
        >
            <template v-for="y in ys">
                <template v-for="x in xs">
                    <Vertex
                        :key="[x, y].join('-')"
                        :position="[x, y]"
                        :shift="fuzzyStonePlacement ? shiftMap && shiftMap[y] && shiftMap[y][x] : 0"
                        :random="randomMap && randomMap[y] && randomMap[y][x]"
                        :sign="signMap && signMap[y] && signMap[y][x]"
                        :heat="heatMap && heatMap[y] && heatMap[y][x]"
                        :paint="paintMap && paintMap[y] && paintMap[y][x]"
                        :marker="markerMap && markerMap[y] && markerMap[y][x]"
                        :ghost-stone="ghostStoneMap && ghostStoneMap[y] && ghostStoneMap[y][x]"
                        :dimmed="dimmedVertices && dimmedVertices.some(v => helper.vertexEquals(v, [x, y]))"
                        :selected="selectedVertices && selectedVertices.some(v => helper.vertexEquals(v, [x, y]))"
                        :animate="selectedVertices && animatedVertices.some(v => helper.vertexEquals(v, [x, y]))"
                        @click="$emit('click', $event)"
                        @mousedown="$emit('mousedown', $event)"
                        @mouseup="$emit('mouseup', $event)"
                        @mousemove="$emit('mousemove', $event)"
                        @mouseenter="$emit('mouseenter', $event)"
                        @mouseleave="$emit('mouseleave', $event)"
                    />
                </template>
            </template>
        </div>

        <div
            class="shudan-lines"
            style="position: absolute; top: 0; left: 0; right: 0; bottom: 0; overflow: hidden; pointer-events: none; z-index: 2;"
        >
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
                    :vertex-size="vertexSize"
                />
            </div>
        </div>
    </div>

    <!-- 下侧及右侧坐标标签 -->
    <CoordY
        v-if="showCoordinates"
        :height="height"
        :ys="ys"
        style="grid-area: 2 / 3 / auto / auto;"
        :coord-y="coordY"
    />
    <CoordX
        v-if="showCoordinates"
        :xs="xs"
        style="grid-area: 3 / 2 / auto / auto;"
        :coord-x="coordX"
    />
</div>
</template>


<script>
import { CoordX, CoordY } from './Coord';
import Grid from './Grid.vue';
import Vertex from './Vertex.vue';
import ULine from './Line.vue';
import helper from './helper.js';
import { setTimeout } from 'timers';

export default {
    components: {
        CoordX,
        CoordY,
        Grid,
        Vertex,
        ULine
    },

    props: {
        vertexSize: Number,
        animate: Boolean,
        busy: Boolean,
        rangeX: {
            type: Array,
            default: function () {
                return [0, Infinity];
            }
        },
        rangeY: {
            type: Array,
            default: function () {
                return [0, Infinity];
            }
        },
        coordX: Function,
        coordY: Function,
        signMap: Array,
        showCoordinates: Boolean,
        fuzzyStonePlacement: Boolean,
        animateStonePlacement: Boolean,
        paintMap: {
            type: Array,
            default: function () {
                return [];
            }
        },
        heatMap: Array,
        markerMap: Array,
        ghostStoneMap: Array,
        lines: Array,
        dimmedVertices: Array,
        selectedVertices: Array
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
            handler: function (signMap, oldSignMap) {
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
        xs: function () {
            let { width, rangeX } = this;
            return helper.range(width).slice(rangeX[0], rangeX[1] + 1);
        },

        ys: function () {
            let { height, rangeY } = this;
            return helper.range(height).slice(rangeY[0], rangeY[1] + 1);
        }
    },

    methods: {
        updateAnimatedVertices: function () {
            if (this.animatedVertices.length > 0) {
                // Handle stone animation
                for (let [x, y] of this.animatedVertices) {
                    this.$set(this.shiftMap[y], x, helper.random(7) + 1);
                    helper.readjustShifts(this.shiftMap, [x, y]);
                }

                // Clear animation classes
                this.clearAnimatedVerticesHandler = setTimeout(
                    function (_this) {
                        _this.animatedVertices = [];
                        _this.clearAnimatedVerticesHandler = null;
                    },
                    this.animationDuration || 200,
                    this
                );
            }
        }
    }
};
</script>

<style scoped>
@import "./css/goban.css";
</style>
