<script>
export default {
    props: {
        vertexSize: {
            type: Number,
            required: true
        },

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
        _lines() {
            const { vertexSize, width, height, xs, ys } = this;
            const halfVertexSize = vertexSize / 2;
            const [x, y] = [xs[0], ys[0]].map(i => i === 0 ?  halfVertexSize : 0);
            const calcLen = (val, sets, size) => {
                return sets[sets.length - 1] === size - 1
                    ? Math.floor((2 * sets.length - 1) * halfVertexSize - val)
                    : Math.floor(sets.length * vertexSize - val);
            };

            return [
                ...ys.map((_, i) => {
                    return {
                        classes: 'shudan-gridline shudan-horizontal',
                        x: Math.floor(x),
                        y: Math.floor((2 * i + 1) * halfVertexSize),
                        width: calcLen(x, xs, width),
                        height: 1
                    };
                }),
                ...xs.map((_, i) => {
                    return {
                        classes: 'shudan-gridline shudan-vertical',
                        x: Math.floor((2 * i + 1) * halfVertexSize),
                        y: Math.floor(y),
                        width: 1,
                        height: calcLen(y, ys, height)
                    };
                }),
            ];
        },

        _points() {
            const { vertexSize, xs, ys, hoshis = [] } = this;
            const halfVertexSize = vertexSize / 2; 
            const convert = val => Math.floor((2 * val + 1) * halfVertexSize) + .5;
            const result = [];
            hoshis.forEach(([x, y]) => {
                const [i, j] = [xs.indexOf(x), ys.indexOf(y)];
                if (i >= 0 && j >= 0) {
                    result.push({
                        x: convert(i),
                        y: convert(j),
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
        v-for="(l, i) in _lines"
        :key="`l${i}`"
        :class="l.classes"
        :x="l.x"
        :y="l.y"
        :width="l.width"
        :height="l.height"
        />
    <circle
        v-for="(p, i) in _points"
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
