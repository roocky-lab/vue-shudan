<script>
export default {
    props: {
        pos1: {
            type: Array,
            required: true,
            default: () => [0, 0]
        },

        pos2: {
            type: Array,
            required: true,
            default: () => [0, 0]
        },

        type: {
            type: String,
            required: true,
            default: "line"
        }
    },

    computed: {
        styles() {
            const { pos1, pos2, vertexEquals } = this;
            if (vertexEquals(pos1, pos2)) return;
            const [dx, dy] = pos1.map((x, i) => pos2[i] - x);
            const [left, top] = pos1.map((x, i) => (x + pos2[i] + 1) / 2);
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
    },

    methods: {
        vertexEquals: ([x1, y1], [x2, y2]) => x1 === x2 && y1 === y2
    }
};
</script>

<template>
<div
    :class="`shudan-${type}`"
    :style="styles"
    />
</template>
