<template>
    <div :class="gobanClasses" :style="gobanStyles">
        <CoordX
            v-if="showCoordinates"
            :xs="xs"
            :coordX="coordX"
            style="grid-area: 1 / 2 / auto / auto;"
        ></CoordX>
        <CoordY
            v-if="showCoordinates"
            :ys="ys"
            :coordY="coordY"
            :height="height"
            style="grid-area: 2 / 1 / auto / auto;"
        ></CoordY>

        <div class="shudan-content" :style="contentStyles">
            <Grid
                :vertexSize="vertexSize"
                :xs="xs"
                :ys="ys"
                :width="width"
                :height="height"
                :hoshis="hoshis"
            ></Grid>

            <div class="shudan-vertices" :style="verticesStyles">
                <Vertex
                    v-for="(v) in _vertexs"
                    :key="v.key"
                    :position="v.position"
                    :shift="v.shift"
                    :random="v.random"
                    :sign="v.sign"
                    :heat="v.heat"
                    :paint="v.paint"
                    :marker="v.marker"
                    :dimmed="v.dimmed"
                    :selected="v.selected"
                    :animate="v.animate"
                ></Vertex>
            </div>

            <div
                class="shudan-lines"
                style="position: absolute; top: 0; left: 0; right: 0; bottom: 0; overflow: hidden; pointer-events: none; z-index: 2;"
            >
                <div :style="lineStyles">
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

        <CoordY
            v-if="showCoordinates"
            :ys="ys"
            :coordY="coordY"
            :height="height"
            style="grid-area: 2 / 3 / auto / auto;"
        ></CoordY>
        <CoordX
            v-if="showCoordinates"
            :xs="xs"
            :coordX="coordX"
            style="grid-area: 3 / 2 / auto / auto;"
        ></CoordX>
    </div>
</template>


<script>
import { CoordX, CoordY } from './Coord';
import Grid from './Grid.vue';
import Vertex from './Vertex.vue';
import XLine from './Line.vue';
import { range, vertexEquals } from './helper.js';

export default {
    components: {
        CoordX,
        CoordY,
        Grid,
        Vertex,
        XLine
    },

    props: {
        rangeX: Array,
        rangeY: Array,
        hoshis: Array,
        shiftMap: Array,
        randomMap: Array,
        innerProps: Object,
        vertexSize: Number,
        coordX: Function,
        coordY: Function,
        busy: Boolean,
        signMap: Array,
        paintMap: Array,
        heatMap: Array,
        markerMap: Array,
        ghostStoneMap: Array,
        fuzzyStonePlacement: Boolean,
        showCoordinates: Boolean,
        lines: Array,
        selectedVertices: Array,
        dimmedVertices: Array
    },

    computed: {
        width: function() {
            return this.signMap.length === 0 ? 0 : this.signMap[0].length;
        },

        height: function() {
            return this.signMap.length;
        },

        xs: function() {
            let { width } = this;
            return range(width);
            /*
            XXX:  
            let { width, rangeX } = this;
            return range(width).slice(rangeX[0], rangeX[1] + 1);
            */
        },

        ys: function() {
            let { height } = this;
            return range(height);
            /*
            XXX:
            let { height, rangeY } = this;
            return range(height).slice(rangeY[0], rangeY[1] + 1);
            */
        },

        gobanClasses: function() {
            let { busy, showCoordinates } = this;
            return {
                'shudan-goban': true,
                'shudan-goban-image': true,
                'shudan-busy': busy,
                'shudan-coordinates': showCoordinates
            };
        },

        gobanStyles: function() {
            let { showCoordinates, vertexSize } = this;
            return {
                display: 'inline-grid',
                gridTemplateRows: showCoordinates ? '1em 1fr 1em' : '1fr',
                gridTemplateColumns: showCoordinates ? '1em 1fr 1em' : '1fr',
                fontSize: vertexSize,
                lineHeight: '1em'
            };
        },

        contentStyles: function() {
            let { xs, ys, showCoordinates } = this;
            return {
                position: 'relative',
                width: `${xs.length}em`,
                height: `${ys.length}em`,
                gridRow: showCoordinates ? '2' : '1',
                gridColumn: showCoordinates ? '2' : '1'
            };
        },

        verticesStyles: function() {
            let { xs, ys } = this;
            return {
                display: 'grid',
                gridTemplateColumns: `repeat(${xs.length}, 1em)`,
                gridTemplateRows: `repeat(${ys.length}, 1em)`,
                position: 'absolute',
                top: 0,
                left: 0,
                right: 0,
                bottom: 0,
                zIndex: 1
            };
        },

        _vertexs: function() {
            let {
                xs,
                ys,
                shiftMap,
                randomMap,
                signMap,
                paintMap,
                heatMap,
                markerMap,
                ghostStoneMap,
                fuzzyStonePlacement = false,
                selectedVertices = [],
                dimmedVertices = []
            } = this;

            return ys.map(y =>
                xs.map(x => {
                    let equalsVertex = v => vertexEquals(v, [x, y]);

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
                            ghostStoneMap &&
                            ghostStoneMap[y] &&
                            ghostStoneMap[y][x],
                        dimmed: dimmedVertices.some(equalsVertex),
                        selected: selectedVertices.some(equalsVertex)
                        //   animate: animatedVertices.some(equalsVertex)
                    };
                })
            );
        },

        lineStyles: function() {
            let {
                rangeX = [0, Infinity],
                rangeY = [0, Infinity],
                width,
                height
            } = this;

            return {
                position: 'absolute',
                top: `-${rangeY[0]}em`,
                left: `-${rangeX[0]}em`,
                width: `${width}em`,
                height: `${height}em`
            };
        }
    }
};
</script>


<style scoped>
@import './css/goban.css';
</style>
    