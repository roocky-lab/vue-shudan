<template>
    <div>
        <svg
            class="shudan-grid"
            style="position: absolute; top: 0px; left: 0px; width: 100%; height: 100%; z-index: 0;"
        >
            <rect
                class="shudan-gridline shudan-horizontal"
                v-for="(l, i) in _xlines"
                :key="'x' + i"
                :x="l.x"
                :y="l.y"
                :width="l.width"
                :height="l.height"
            ></rect>
            <rect
                class="shudan-gridline shudan-vertical"
                v-for="(l, i) in _ylines"
                :key="'y' + i"
                :x="l.x"
                :y="l.y"
                :width="l.width"
                :height="l.height"
            ></rect>
            <circle
                class="shudan-hoshi"
                v-for="(h, i) in _hoshis"
                :key="'h' + i"
                :cx="h.cx"
                :cy="h.cy"
                :r="h.r"
            ></circle>
        </svg>
    </div>
</template>

<script>
export default {
    props: {
        vertexSize: {
            type: Number
        },

        width: {
            type: Number
        },

        height: {
            type: Number
        },

        xs: {
            type: Array
        },

        ys: {
            type: Array
        },

        hoshis: {
            type: Array
        }
    },

    computed: {
        _xlines: function() {
            let { vertexSize, width, height, xs, ys, hoshis } = this;
            let halfVertexSize = vertexSize / 2;
            let fl = Math.floor;

            return ys.map((_, i) => {
                let x = xs[0] === 0 ? halfVertexSize : 0;
                return {
                    x: fl(x),
                    y: fl((2 * i + 1) * halfVertexSize),
                    width:
                        xs[xs.length - 1] === width - 1
                            ? fl((2 * xs.length - 1) * halfVertexSize - x)
                            : fl(xs.length * vertexSize - x),
                    height: 1
                };
            });
        },

        _ylines: function() {
            let { vertexSize, width, height, xs, ys, hoshis } = this;
            let halfVertexSize = vertexSize / 2;
            let fl = Math.floor;

            return xs.map((_, i) => {
                let y = ys[0] === 0 ? halfVertexSize : 0;
                return {
                    x: fl((2 * i + 1) * halfVertexSize),
                    y: fl(y),
                    width: 1,
                    height:
                        ys[ys.length - 1] === height - 1
                            ? fl((2 * ys.length - 1) * halfVertexSize - y)
                            : fl(ys.length * vertexSize - y)
                };
            });
        },

        _hoshis: function() {
            let { vertexSize, width, height, xs, ys, hoshis } = this;
            let halfVertexSize = vertexSize / 2;
            let fl = Math.floor;

            return hoshis.map(([x, y]) => {
                let i = xs.indexOf(x);
                let j = ys.indexOf(y);
                if (i < 0 || j < 0) return;

                return {
                    cx: fl((2 * i + 1) * halfVertexSize) + 0.5,
                    cy: fl((2 * j + 1) * halfVertexSize) + 0.5,
                    r: '.1em'
                };
            });
        }
    }
};
</script>
