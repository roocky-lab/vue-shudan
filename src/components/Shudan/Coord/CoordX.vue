<template>
    <div class="shudan-coordx" style="display: flex; text-align: center;">
        <div v-for="(t, i) in CoordX" :key="i" style="width: 1em;">
            <span style="display: block;">{{ t }}</span>
        </div>
    </div>
</template>

<script>
import { alpha } from '../helper';

export default {
    props: {
        xs: {
            type: Array
        }
    },

    computed: {
        CoordX: function() {
            return this.xs.map(i => alpha[i] || alpha[alpha.length - 1]);
        }
    }
};
</script>
