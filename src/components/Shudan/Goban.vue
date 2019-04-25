<template>
    <div
        class="shudan-goban shudan-goban-image shudan-coordinates"
        style="display: inline-grid; grid-template-rows: 1em 1fr 1em; grid-template-columns: 1em 1fr 1em; font-size: 24px; line-height: 1em;"
    >
        <CoordX :xs="xs" style="grid-area: 1 / 2 / auto / auto;"></CoordX>
        <CoordY :ys="ys" :height="height" style="grid-area: 2 / 1 / auto / auto;"></CoordY>
        <CoordY :ys="ys" :height="height" style="grid-area: 2 / 3 / auto / auto;"></CoordY>
        <CoordX :xs="xs" style="grid-area: 3 / 2 / auto / auto;"></CoordX>
        <div
            class="shudan-content"
            style="position: relative; width: 19em; height: 19em; grid-area: 2 / 2 / auto / auto;"
        >
            <Grid
                :vertexSize="vertexSize"
                :xs="xs"
                :ys="ys"
                :width="width"
                :height="height"
                :hoshis="hoshis"
            ></Grid>
            <div
                class="shudan-vertices"
                style="display: grid; grid-template-columns: 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em; grid-template-rows: 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em 1em; position: absolute; top: 0px; left: 0px; right: 0px; bottom: 0px; z-index: 1;"
            >
                <Vertex
                    :position="[3, 3]"
                    :shift="4"
                    :random="1"
                    :sign="0"
                    :selected="false"
                    :heat="{strength:4, text: '66%'}"
                    :paint="0"
                    :dimmed="false"
                ></Vertex>
            </div>

            <div
                class="shudan-lines"
                style="position: absolute; top: 0px; left: 0px; right: 0px; bottom: 0px; overflow: hidden; pointer-events: none; z-index: 2;"
            >
                <div style="position: absolute; top: -0em; left: -0em; width: 19em; height: 19em;">
                    <XLine type="line" :v1="[15, 6]" :v2="[12, 15]" :vertexSize="vertexSize"></XLine>
                    <XLine type="arrow" :v1="[10, 4]" :v2="[5, 7]" :vertexSize="vertexSize"></XLine>
                </div>
            </div>
        </div>
    </div>
</template>


<script>
import { CoordX, CoordY } from './Coord';
import Grid from './Grid.vue';
import Vertex from './Vertex.vue';
import XLine from './Line.vue';
import { range } from './helper.js';

export default {
    components: {
        CoordX,
        CoordY,
        Grid,
        Vertex,
        XLine
    },

    data: function() {
        return {
            vertexSize: 24,
            xs: range(19),
            ys: range(19),
            width: 19,
            height: 19,
            hoshis: [
                [3, 3],
                [3, 15],
                [15, 3],
                [15, 15],
                [9, 9],
                [9, 3],
                [3, 9],
                [9, 15],
                [15, 9]
            ]
        };
    }
};
</script>

<style scoped>
@import './css/goban.css';
</style>
