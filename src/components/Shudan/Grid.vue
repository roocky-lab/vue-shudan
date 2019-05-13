<script>
const lineWidth = 0.042; // 当font-size === 24px, 0.042em约1px
const convert = val => (2 * val + 1) * 0.5;

export default {
    props: {
        width: {
            type: Number,
            required: true
        },

        height: {
            type: Number,
            required: true
        },

        xs: {
            type: Array,
            required: true
        },

        ys: {
            type: Array,
            required: true
        },

        hoshis: {
            type: Array,
            required: true
        }
    },

    computed: {
        lines() {
            const { width, height, xs, ys } = this;
            const [x, y] = [xs[0], ys[0]].map(i => i === 0 ?  0.5 : 0);
            const calcLen = (val, sets, size) => {
                return sets[sets.length - 1] === size - 1
                    ? (2 * sets.length - 1) * 0.5 - val
                    : sets.length - val;
            };

            return [
                ...ys.map((_, i) => {
                    return {
                        classes: 'shudan-gridline shudan-horizontal',
                        x: `${x}em`,
                        y: `${convert(i)}em`,
                        width: `${calcLen(x, xs, width)}em`,
                        height: `${lineWidth}em`
                    };
                }),
                ...xs.map((_, i) => {
                    return {
                        classes: 'shudan-gridline shudan-vertical',
                        x: `${convert(i)}em`,
                        y: `${y}em`,
                        width: `${lineWidth}em`,
                        height: `${calcLen(y, ys, height)}em`
                    };
                })
            ];
        },

        points() {
            const { xs, ys, hoshis = [] } = this;
            const result = [];
            hoshis.forEach(([x, y]) => {
                const [i, j] = [xs.indexOf(x), ys.indexOf(y)];
                if (i >= 0 && j >= 0) {
                    result.push({
                        x: `${convert(i) + 0.5 * lineWidth}em`,
                        y: `${convert(j) + 0.5 * lineWidth}em`
                    });
                }
            });
            return result;
        }
    }
};
</script>

<template>
<svg class="shudan-grid">
    <rect
        v-for="(l, i) in lines"
        :key="`l${i}`"
        :class="l.classes"
        :x="l.x"
        :y="l.y"
        :width="l.width"
        :height="l.height"
        />
    <circle
        v-for="(p, i) in points"
        :key="`p${i}`"
        class="shudan-hoshi"
        :cx="p.x"
        :cy="p.y"
        r=".1em"
        />
</svg>
</template>

<style scoped>
.shudan-grid {
    position: 'absolute';
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 0;
}
</style>
