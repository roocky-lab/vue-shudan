<script>
export default {
    props: {
        dir: {
            type: String,
            required: true,
            validator: v => v === 'x' || v === 'y'
        },

        sets: {
            type: Array,
            required: true
        },

        labels: {
            type: Array,
            required: true,
            default: () => [' ']
        }
    }
};
</script>

<template>
<div :class="`shudan-coord${dir}`">
    <div
        v-for="(t, i) in sets"
        :key="i"
        >
        <span
            v-text="labels[t] != undefined
                ? labels[t]
                : labels[labels.length - 1]"
            />
    </div>
</div>
</template>

<style scoped>
.shudan-coordx {
    display: flex;
    text-align: center;
}
.shudan-coordx div {
    width: 1em;
}

.shudan-coordy {
    text-align: center;
}
.shudan-coordy div {
    height: 1em;
}

.shudan-coordx span,
.shudan-coordy span {
    display: block;
}
</style>
