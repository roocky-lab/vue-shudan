<script>
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
                        y: `${(2 * i + 1) * 0.5}em`,
                        width: `${calcLen(x, xs, width)}em`,
                        height: '1px'
                    };
                }),
                ...xs.map((_, i) => {
                    return {
                        classes: 'shudan-gridline shudan-vertical',
                        x: `${(2 * i + 1) * 0.5}em`,
                        y: `${y}em`,
                        width: '1px',
                        height: `${calcLen(y, ys, height)}em`
                    };
                })
            ];
        },

        points() {
            const { xs, ys, hoshis = [] } = this;
            const convert = val => (2 * val + 1) * 0.5;
            const result = [];
            hoshis.forEach(([x, y]) => {
                const [i, j] = [xs.indexOf(x), ys.indexOf(y)];
                if (i >= 0 && j >= 0) {
                    result.push({
                        x: `calc(${convert(i)}em + .5px)`,
                        y: `calc(${convert(j)}em + .5px)`
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
