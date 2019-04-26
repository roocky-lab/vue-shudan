<template>
    <div class="shudan-coordy" style="text-align: center;">
        <div v-for="(t, i) in _coordY" :key="i" style="height: 1em;">
            <span style="display: block;">{{ t }}</span>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        ys: {
            type: Array
        },

        height: {
            type: Number
        },

        coordY: {
            type: Function
        }
    },

    computed: {
        _coordY: function() {
            let { ys, height, coordY = i => height - i } = this;
            return ys.map(coordY);
        }
    }
};
</script>
