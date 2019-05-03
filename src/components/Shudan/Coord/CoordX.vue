<template>
<div class="shudan-coordx" style="display: flex; text-align: center;">
    <div v-for="(t, i) in xs.map(coordX)" :key="i" style="width: 1em;">
        <span :style="shudanCoordSpan" style="display: block;">{{ t }}</span>
    </div>
</div>
</template>

<script>
import { alpha } from '../helper';

export default {
    props: {
        xs: Array,
        coordX: {
            type: Function,
            default: i => alpha[i] || alpha[alpha.length - 1]
        }
    },

    computed: {
        shudanCoordSpan: function () {
            for (let x in this.xs) {
                if (this.coordX(x).length > 1) {
                    return { 'font-size': '0.45em' };
                }

            }
            return {};
        }
    }
};
</script>
