<script>
import { vertexEquals } from './helper.js';

export default {
    props: {
        v1: {
            type: Array,
            required: true
        },

        v2: {
            type: Array,
            required: true
        },

        type: {
            type: String,
            default: 'line'
        },

        vertexSize: {
            type: Number,
            required: true
        },
    },

    computed: {
        styles: function () {
            const { v1, v2, vertexSize } = this;
            if (vertexEquals(v1, v2)) return;
            const [pos1, pos2] = [v1, v2].map(v => v.map(x => x * vertexSize));
            const [dx, dy] = pos1.map((x, i) => pos2[i] - x);
            const [left, top] = pos1.map((x, i) => (x + pos2[i] + vertexSize) / 2);
            const angle = (Math.atan2(dy, dx) * 180) / Math.PI;
            const length = Math.sqrt(dx * dx + dy * dy);

            return {
                position: 'absolute',
                left: `${left}px`,
                top: `${top}px`,
                margin: 0,
                width: `${length}px`,
                transform: `translateX(${-length / 2}px) rotate(${angle}deg)`
            };
        }
    }
};
</script>

<template>
<div :class="`shudan-${type}`" :style="styles" />
</template>
