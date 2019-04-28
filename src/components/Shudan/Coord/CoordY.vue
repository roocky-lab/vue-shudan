<template>
    <div class="shudan-coordy" style="text-align: center;">
        <div
            v-for="(t, i) in ys.map(coordY ? coordY : i => height - i)"
            :key="i"
            style="height: 1em;"
        >
            <span style="display: block;">{{ t }}</span>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        ys: Array,
        height: Number,
        coordY: Function
    }
};
</script>
