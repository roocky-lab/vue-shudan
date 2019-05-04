<script>
import { alpha } from './helper';

export default {
    props: {
        dir: {
            type: String,
            required: true,
            validator: val => val === 'x' || val === 'y'
        },

        sets: {
            type: Array,
            required: true
        },

        size: {
            type: Number,
            default: undefined
        },

        labels: {
            type: Array,
            default: undefined
        }
    },

    computed: {
        ticks: function () {
            let { dir, sets, labels, size = Math.max(...sets) + 1 } = this;
            if (labels) {
                return labels.length > 0 ? labels : [' '];
            } else {
                return dir === 'x' ? alpha.split('') : [...Array(size)].map((_, i) => size - i);
            }
        }
    }
};
</script>

<template>
<div :class="`shudan-coord${dir}`">
    <div v-for="(t, i) in sets" :key="i">
        <span>{{ (ticks[t] !== undefined) ? ticks[t] : ticks[ticks.length - 1] }}</span>
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
