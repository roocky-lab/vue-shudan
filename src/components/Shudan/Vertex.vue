<template>
<div
    :data-x="position[0]"
    :data-y="position[1]"
    style="position: relative;"
    :class="[
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
        marker && marker.type === 'label'
            && marker.label
            && (marker.label.includes('\n') || marker.label.length >= 3)
            && `shudan-smalllabel`,

        ghostStone && `shudan-ghost_${ghostStone.sign}`,
        ghostStone && ghostStone.type && `shudan-ghost_${ghostStone.type}`,
        ghostStone && ghostStone.faint && `shudan-ghost_faint`
    ]"
    @click="$emit('click', position)"
    @mousedown="$emit('mousedown', position)"
    @mouseup="$emit('mouseup', position)"
    @mousemove="$emit('mousemove', position)"
    @mouseenter="$emit('mouseenter', position)"
    @mouseleave="$emit('mouseleave', position)"
>
    <div
        key="marker"
        v-if="!sign && !!marker"
        class="shudan-marker"
        :title="marker.label"
        :style="absoluteStyle(0)"
    />
    <div
        key="ghost"
        v-if="!sign && !!ghostStone"
        class="shudan-ghost"
        :style="absoluteStyle(1)"
    />

    <div key="stone" class="shudan-stone" :style="absoluteStyle(2)">
        <div key="shadow" v-if="!!sign" class="shudan-shadow" :style="absoluteStyle()" />
        <div
            key="inner"
            v-if="!!sign"
            :class="['shudan-inner', 'shudan-stone-image',
                     `shudan-random_${random}`, `shudan-sign_${sign}`]"
            :style="absoluteStyle()"
        >
            {{ sign }}
        </div>
        <div
            key="marker"
            v-if="!!sign && !!marker"
            class="shudan-marker"
            :title="marker.label"
            :style="absoluteStyle()"
        />
    </div>

    <div key="paint" v-if="!!paint" class="shudan-paint" :style="absoluteStyle(3)" />
    <div key="selection" v-if="!!selected" class="shudan-selection" :style="absoluteStyle(4)" />

    <div key="heat" class="shudan-heat" :style="absoluteStyle(5)" />
    <div key="heatlabel" v-if="!!heat" class="shudan-heatlabel" :style="absoluteStyle(6)">{{ heat.text && heat.text.toString() }}</div>
</div>
</template>

<script>
export default {
    props: {
        position: Array,
        shift: Number,
        random: Number, // XXX: 没有shudan-random_*样式，这个参数是为什么预留的？
        sign: Number,
        selected: Boolean,
        heat: Object,
        paint: Number,
        dimmed: Boolean,
        marker: Object,
        ghostStone: Object,
        animate: Boolean, // 触发落子效果
    },

    methods: {
        absoluteStyle: zIndex => ({
            position: 'absolute',
            zIndex
        })
    }
};
</script>
