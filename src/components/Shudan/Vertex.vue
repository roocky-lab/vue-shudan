<template>
    <div
        :data-x="position[0]"
        :data-y="position[1]"
        :class="vertexClasses"
        style="position: relative;"
        @click="$emit('vertex-click', position)"
    >
        <div
            v-if="!sign && !!marker"
            class="shudan-marker"
            :title="marker.label"
            :style="absoluteStyle(0)"
        ></div>
        <div v-if="!sign && !!ghostStone" class="shudan-ghost" :style="absoluteStyle(1)"></div>

        <div class="shudan-stone" :style="absoluteStyle(2)">
            <div v-if="!!sign" class="shudan-shadow" :style="absoluteStyle()"></div>
            <div v-if="!!sign" :class="innerClasses" :style="absoluteStyle()">{{ sign }}</div>
            <div
                v-if="!!sign && !!marker"
                class="shudan-marker"
                :title="marker.label"
                :style="absoluteStyle()"
            ></div>
        </div>

        <div v-if="!!paint" class="shudan-paint" :style="absoluteStyle(3)"></div>
        <div v-if="!!selected" class="shudan-selection" :style="absoluteStyle(4)"></div>

        <div class="shudan-heat" :style="absoluteStyle(5)"></div>
        <div
            v-if="!!heat"
            class="shudan-heatlabel"
            :style="absoluteStyle(6)"
        >{{ heat.text && heat.text.toString() }}</div>
    </div>
</template>

<script>
export default {
    props: {
        position: {
            type: Array
        },

        shift: {
            type: Number
        },

        /* XXX: 没有shudan-random_xxx样式，这个参数是为什么预留的？ */
        random: {
            type: Number
        },

        sign: {
            type: Number
        },

        heat: {
            type: Object
        },

        paint: {
            type: Number
        },

        marker: {
            type: Object
        },

        ghostStone: {
            type: Number
        },

        dimmed: {
            type: Boolean
        },

        selected: {
            type: Boolean
        },

        /* 触发落子时滑动效果shudan-animate，需要同时开启shudan-shift_xx */
        animate: {
            type: Boolean
        }
    },

    computed: {
        vertexClasses: function() {
            let {
                shift,
                random,
                sign,
                selected,
                heat,
                paint,
                dimmed,
                marker,
                ghostStone,
                animate
            } = this;

            return [
                'shudan-vertex',
                `shudan-random_${random}`,
                `shudan-sign_${sign}`,

                {
                    [`shudan-shift_${shift}`]: !!shift,
                    [`shudan-heat_${!!heat && heat.strength}`]: !!heat,
                    [`shudan-paint_${paint}`]: !!paint,
                    'shudan-dimmed': dimmed,
                    'shudan-selected': selected,
                    'shudan-animate': animate
                },

                marker && marker.type && `shudan-marker_${marker.type}`,
                marker &&
                    marker.type === 'label' &&
                    marker.label &&
                    (marker.label.includes('\n') || marker.label.length >= 3) &&
                    `shudan-smalllabel`,

                ghostStone && `shudan-ghost_${ghostStone.sign}`,
                ghostStone &&
                    ghostStone.type &&
                    `shudan-ghost_${ghostStone.type}`,
                ghostStone && ghostStone.faint && `shudan-ghost_faint`
            ];
        },

        innerClasses: function() {
            let { random, sign } = this;

            return [
                'shudan-inner',
                'shudan-stone-image',
                `shudan-random_${random}`,
                `shudan-sign_${sign}`
            ];
        }
    },

    methods: {
        absoluteStyle: function(zIndex) {
            return {
                position: 'absolute',
                zIndex
            };
        }
    }
};
</script>
