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
            required: true
        }
    },

    computed: {
        _labels() {
            const { sets = [], labels = [' '] } = this;
            const sub = labels[labels.length - 1];
            return sets.map((t => labels[t] !== undefined ? labels[t] : sub));
        }
    }
};
</script>

<template>
<div :class="`shudan-coord${dir}`">
    <div
        v-for="(t, i) in _labels"
        :key="i"
        >
        <span v-text="t" />
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
