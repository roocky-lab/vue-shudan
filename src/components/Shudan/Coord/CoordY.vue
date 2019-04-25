<template>
    <div class="shudan-coordy" style="text-align: center;">
        <div v-for="(t, i) in CoordY" :key="i" style="height: 1em;">
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
        }
    },

    computed: {
        CoordY: function() {
            let { ys, height } = this;
            return ys.map(i => height - i);
        }
    }
};
</script>
