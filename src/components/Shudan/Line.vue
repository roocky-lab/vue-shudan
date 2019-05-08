<script>
import { vertexEquals } from "./helper.js";

export default {
    props: {
        type: {
            type: String,
            required: true,
            default: "line"
        },

        v1: {
            type: Array,
            required: true,
            default: () => [0, 0]
        },

        v2: {
            type: Array,
            required: true,
            default: () => [0, 0]
        }
    },

    computed: {
        styles() {
            const { v1, v2 } = this;
            if (vertexEquals(v1, v2)) return;
            const [dx, dy] = v1.map((x, i) => v2[i] - x);
            const [left, top] = v1.map((x, i) => (x + v2[i] + 1) / 2);
            const angle = (Math.atan2(dy, dx) * 180) / Math.PI;
            const length = Math.sqrt(dx * dx + dy * dy);
            return {
                position: 'absolute',
                left: `${left}em`,
                top: `${top}em`,
                margin: 0,
                width: `${length}em`,
                transform: `translateX(${-length / 2}em) rotate(${angle}deg)`
            };
        }
    }
};
</script>

<template>
<div
    :class="`shudan-${type}`"
    :style="styles"
    />
</template>
