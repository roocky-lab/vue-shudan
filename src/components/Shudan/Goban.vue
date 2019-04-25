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
        </div>
    </div>
</template>

<script>
import { CoordX, CoordY } from './Coord';
import Grid from './Grid.vue';
import { range } from './helper.js';

export default {
    components: {
        CoordX,
        CoordY,
        Grid
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
