<script>
export default {
    props: {
        position: {
            type: Array,
            required: true,
            default: undefined
        },

        sign: {
            type: Number,
            required: true,
            default: 0
        },

        selected: {
            type: Boolean,
            required: true,
            default: false
        },

        dimmed: {
            type: Boolean,
            required: true,
            default: false
        },

        animate: {
            type: Boolean,
            required: true,
            default: false
        },

        shift: {
            type: Number,
            required: true,
            default: undefined
        },

        random: {
            type: Number,
            required: true,
            default: 0
        },

        paint: {
            type: Number,
            default: undefined
        },

        heat: { 
            type: Object,
            default: undefined
        },

        marker: {
            type: Object,
            default: undefined
        },

        ghostStone: {
            type: Object,
            default: undefined
        }
    }
};
</script>

<template>
<div
    :data-x="position[0]"
    :data-y="position[1]"
    :class="[
        'shudan-vertex',
        `shudan-random_${random}`,
        `shudan-sign_${sign}`,
        {
            [`shudan-shift_${shift}`]: !!shift,
            [`shudan-heat_${heat && heat.strength}`]: !!heat && !!heat.strength,
            [`shudan-paint_${paint}`]: !!paint,
            'shudan-dimmed': dimmed,
            'shudan-selected': selected,
            'shudan-animate': animate,
            [`shudan-marker_${marker && marker.type}`]: !!marker && !!marker.type,
            'shudan-smalllabel': !!marker && marker.type === 'label'
                && !!marker.label && (marker.label.includes('\n') || marker.label.length >= 3),
            [`shudan-ghost_${ghostStone && ghostStone.sign}`]: !!ghostStone && !!ghostStone.sign,
            [`shudan-ghost_${ghostStone && ghostStone.type}`]: !!ghostStone && !!ghostStone.type,
            'shudan-ghost_faint': !!ghostStone && !!ghostStone.faint
        }
    ]"
    style="position: relative;"
    @click="$emit('click', position)"
    @mousedown="$emit('mousedown', position)"
    @mouseup="$emit('mouseup', position)"
    @mousemove="$emit('mousemove', position)"
    @mouseenter="$emit('mouseenter', position)"
    @mouseleave="$emit('mouseleave', position)"
    >
    <div
        v-if="!sign && !!marker"
        key="marker"
        class="shudan-marker"
        style="z-index: 0;"
        :title="marker.label"
        />
    <div
        v-if="!sign && !!ghostStone"
        key="ghost"
        class="shudan-ghost"
        style="z-index: 1;"
        />
    <div
        key="stone"
        class="shudan-stone"
        style="z-index: 2;"
        >
        <div
            v-if="!!sign"
            key="shadow"
            class="shudan-shadow"
            />
        <div
            v-if="!!sign"
            key="inner"
            :class="[
                'shudan-inner',
                'shudan-stone-image',
                `shudan-random_${random}`,
                `shudan-sign_${sign}`
            ]"
            v-text="sign"
            />
        <div
            v-if="!!sign && !!marker"
            key="marker"
            class="shudan-marker"
            :title="marker.label"
            />
    </div>

    <div
        v-if="!!paint"
        key="paint"
        class="shudan-paint"
        style="z-index: 3;"
        />
    <div
        v-if="selected"
        key="selection"
        class="shudan-selection"
        style="z-index: 4;"
        />
    <div
        key="heat"
        class="shudan-heat"
        style="z-index: 5;"
        />
    <div
        v-if="!!heat"
        key="heatlabel"
        class="shudan-heatlabel"
        style="z-index: 6;"
        v-text="heat.text && heat.text.toString()"
        />
</div>
</template>

<style scoped>
.shudan-vertex div {
    position: absolute;
}
</style>