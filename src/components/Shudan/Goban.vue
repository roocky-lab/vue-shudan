<template>
    <div
        class="shudan-goban shudan-goban-image shudan-coordinates"
        style="display: inline-grid; grid-template-rows: 1em 1fr 1em; grid-template-columns: 1em 1fr 1em; font-size: 24px; line-height: 1em;"
    >
        <CoordX :xs="xs" style="grid-area: 1 / 2 / auto / auto;"></CoordX>
        <CoordY :ys="ys" :height="height" style="grid-area: 2 / 1 / auto / auto;"></CoordY>
        <CoordY :ys="ys" :height="height" style="grid-area: 2 / 3 / auto / auto;"></CoordY>
        <CoordX :xs="xs" style="grid-area: 3 / 2 / auto / auto;"></CoordX>
    </div>
</template>



<script>
import { CoordX, CoordY } from './Coord';
import { range } from './helper.js';

export default {
    components: {
        CoordX,
        CoordY
    },

    data: function() {
        return {
            xs: range(9),
            ys: range(9),
            height: 9
        };
    }
};
</script>

<style scoped>
@import './css/goban.css';
</style>
