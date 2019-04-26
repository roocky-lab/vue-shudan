<template>
    <div class="shudan-coordx" style="display: flex; text-align: center;">
        <div v-for="(t, i) in _coordX" :key="i" style="width: 1em;">
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
        },

        coordX: {
            type: Function
        }
    },

    computed: {
        _coordX: function() {
            let {
                xs,
                coordX = i => alpha[i] || alpha[alpha.length - 1]
            } = this;

            return xs.map(coordX);
        }
    }
};
</script>
