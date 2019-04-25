<template>
    <div :class="'shudan-' + type" :style="styles"></div>
</template>

<script>
const { vertexEquals } = require('./helper');

export default {
    props: {
        v1: {
            type: Array
        },

        v2: {
            type: Array
        },

        type: {
            type: String
        },

        vertexSize: {
            type: Number
        }
    },

    computed: {
        styles: function() {
            let { v1, v2, vertexSize } = this;
            if (vertexEquals(v1, v2)) return;

            let [pos1, pos2] = [v1, v2].map(v => v.map(x => x * vertexSize));
            let [dx, dy] = pos1.map((x, i) => pos2[i] - x);
            let [left, top] = pos1.map(
                (x, i) => (x + pos2[i] + vertexSize) / 2
            );

            let angle = (Math.atan2(dy, dx) * 180) / Math.PI;
            let length = Math.sqrt(dx * dx + dy * dy);

            console.log('left = ' + left);
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
