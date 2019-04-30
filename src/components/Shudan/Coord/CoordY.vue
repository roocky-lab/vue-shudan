<template>
    <div class="shudan-coordy" style="text-align: center;">
        <div
            v-for="(t, i) in ys.map(coordY ? coordY : i => height - i)"
            :key="i"
            style="height: 1em;"
        >
            <span :style="shudanCoordSpan" style="display: block;">{{ t }}</span>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        ys: Array,
        height: Number,
        coordY: Function
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
