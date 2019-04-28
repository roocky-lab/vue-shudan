<template>
    <div
        class="shudan-goban shudan-goban-image"
        :class="{'shudan-busy': busy, 'shudan-coordinates': showCoordinates}"
        :style="{
            'display': 'inline-grid',
            'gridTemplateRows': showCoordinates ? '1em 1fr 1em' : '1fr',
            'gridTemplateColumns': showCoordinates ? '1em 1fr 1em' : '1fr',
            'font-size': `${vertexSize}px`,
            'line-height': '1em'
        }"
    >
        <!-- 上侧及左侧坐标标签 -->
        <CoordX
            v-if="showCoordinates"
            :xs="xs"
            style="grid-area: 1 / 2 / auto / auto;"
            :coordX="coordX"
        ></CoordX>
        <CoordY
            v-if="showCoordinates"
            :height="height"
            :ys="ys"
            style="grid-area: 2 / 1 / auto / auto;"
            :coordY="coordY"
        ></CoordY>

        <!-- 中心区 -->
        <div
            class="shudan-content"
            :style="{
                'position': 'relative',
                'width': `${xs.length}em`,
                'height': `${ys.length}em`,
                'grid-row': showCoordinates ? '2' : '1',
                'grid-column': showCoordinates ? '2' : '1'
            }"
        >
            <!-- 棋盘网线及星位 -->
            <Grid
                :vertexSize="vertexSize"
                :width="width"
                :height="height"
                :xs="xs"
                :ys="ys"
                :hoshis="hoshis"
            ></Grid>
            <!-- 落点区域 -->
            <div
                class="shudan-vertices"
                :style="{
                    'display': 'grid',
                    'grid-template-columns': `repeat(${xs.length}, 1em)`,
                    'grid-template-rows': `repeat(${ys.length}, 1em)`,
                    'position': 'absolute',
                    'top': 0,
                    'left': 0,
                    'right': 0,
                    'bottom': 0,
                    'z-index': 1
                }"
            >
                <!-- XXX： 事件只有单击，不全 -->
                <template v-for="vs in vertexs">
                    <Vertex
                        v-for="(v) in vs"
                        :key="v.key"
                        :position="v.position"
                        :shift="v.shift"
                        :random="v.random"
                        :sign="v.sign"
                        :heat="v.heat"
                        :paint="v.paint"
                        :marker="v.marker"
                        :ghostStone="v.ghostStone"
                        :dimmed="v.dimmed"
                        :selected="v.selected"
                        :animate="v.animate"
                        @vertex-click="$emit('vertex-click', v.position)"
                    ></Vertex>
                </template>
            </div>

            <div
                class="shudan-lines"
                style="position: absolute; top: 0; left: 0; right: 0; bottom: 0; overflow: hidden; pointer-events: none; z-index: 2;"
            >
                <div
                    :style="{
                        'position': 'absolute',
                        'top': `-${rangeY[0]}em`,
                        'left': `-${rangeX[0]}em`,
                        'width': `${width}em`,
                        'height': `${height}em`,
                    }"
                >
                    <XLine
                        v-for="(l, i) in lines"
                        :key="i"
                        :v1="l.v1"
                        :v2="l.v2"
                        :type="l.type"
                        :vertexSize="vertexSize"
                    ></XLine>
                </div>
            </div>
        </div>

        <!-- 下侧及右侧坐标标签 -->
        <CoordY
            v-if="showCoordinates"
            :height="height"
            :ys="ys"
            style="grid-area: 2 / 3 / auto / auto;"
            :coordY="coordY"
        ></CoordY>
        <CoordX
            v-if="showCoordinates"
            :xs="xs"
            style="grid-area: 3 / 2 / auto / auto;"
            :coordX="coordX"
        ></CoordX>
    </div>
</template>


<script>
import { CoordX, CoordY } from './Coord';
import Grid from './Grid.vue';
import Vertex from './Vertex.vue';
import XLine from './Line.vue';
import helper from './helper.js';
import { setTimeout } from 'timers';

export default {
    components: {
        CoordX,
        CoordY,
        Grid,
        Vertex,
        XLine
    },

    props: {
        vertexSize: Number,
        animate: Boolean,
        busy: Boolean,
        rangeX: {
            type: Array,
            default: function() {
                return [0, Infinity];
            }
        },
        rangeY: {
            type: Array,
            default: function() {
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
            default: function() {
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

    data: function() {
        return {
            width: 0,
            height: 0,
            shiftMap: [],
            randomMap: [],
            hoshis: [],
            animatedVertices: [],
            clearAnimatedVerticesHandler: null
        };
    },

    watch: {
        signMap: {
            handler: function(signMap, oldSignMap) {
                let width = signMap.length === 0 ? 0 : signMap[0].length;
                let height = signMap.length;
                if (this.width === width && this.height === height) {
                    if (
                        this.animateStonePlacement &&
                        this.fuzzyStonePlacement &&
                        this.clearAnimatedVerticesHandler == null
                    ) {
                        this.animatedVertices = helper.diffSignMap(
                            oldSignMap,
                            signMap
                        );
                        this.$nextTick(this.updateAnimatedVertices);
                    }
                } else {
                    this.width = width;
                    this.height = height;
                    this.hoshis = helper.getHoshis(width, height);
                    this.shiftMap = helper.readjustShifts(
                        signMap.map(row => row.map(_ => helper.random(8)))
                    );
                    this.randomMap = signMap.map(row =>
                        row.map(_ => helper.random(4))
                    );
                }
            },
            immediate: true
        }
    },

    computed: {
        xs: function() {
            let { width, rangeX } = this;
            return helper.range(width).slice(rangeX[0], rangeX[1] + 1);
        },

        ys: function() {
            let { height, rangeY } = this;
            return helper.range(height).slice(rangeY[0], rangeY[1] + 1);
        },

        vertexs: function() {
            let { xs, ys, updateVertex } = this;
            return ys.map(y => {
                return xs.map(x => {
                    return updateVertex(x, y);
                });
            });
        }
    },

    methods: {
        updateVertex: function(x, y) {
            let {
                fuzzyStonePlacement,
                shiftMap,
                randomMap,
                signMap,
                heatMap,
                paintMap,
                markerMap,
                ghostStoneMap,
                dimmedVertices,
                selectedVertices,
                animatedVertices
            } = this;
            let equalsVertex = v => helper.vertexEquals(v, [x, y]);

            return {
                key: [x, y].join('-'),
                position: [x, y],

                shift: fuzzyStonePlacement
                    ? shiftMap && shiftMap[y] && shiftMap[y][x]
                    : 0,
                random: randomMap && randomMap[y] && randomMap[y][x],
                sign: signMap && signMap[y] && signMap[y][x],
                heat: heatMap && heatMap[y] && heatMap[y][x],
                paint: paintMap && paintMap[y] && paintMap[y][x],
                marker: markerMap && markerMap[y] && markerMap[y][x],
                ghostStone:
                    ghostStoneMap && ghostStoneMap[y] && ghostStoneMap[y][x],
                dimmed: dimmedVertices.some(equalsVertex),
                selected: selectedVertices.some(equalsVertex),
                animate: animatedVertices.some(equalsVertex)
            };
        },

        updateAnimatedVertices: function() {
            let {
                animateStonePlacement,
                clearAnimatedVerticesHandler,
                animatedVertices
            } = this;

            if (
                animateStonePlacement &&
                !clearAnimatedVerticesHandler &&
                animatedVertices.length > 0
            ) {
                // Handle stone animation
                for (let [x, y] of animatedVertices) {
                    this.$set(this.shiftMap[y], x, helper.random(7) + 1);
                    helper.readjustShifts(this.shiftMap, [x, y]);
                }

                // Clear animation classes
                this.clearAnimatedVerticesHandler = setTimeout(
                    function(that) {
                        that.animatedVertices = [];
                        that.clearAnimatedVerticesHandler = null;
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
@import './css/goban.css';
</style>
