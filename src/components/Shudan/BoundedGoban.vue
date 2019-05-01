<template>
    <Goban
        :vertexSize="vertexSize"
        :animate="innerProps && innerProps.animate"
        :busy="innerProps && innerProps.busy"
        :rangeX="rangeX"
        :rangeY="rangeY"
        :coordX="innerProps && innerProps.coordX"
        :coordY="innerProps && innerProps.coordY"
        :signMap="signMap"
        :showCoordinates="showCoordinates"
        :fuzzyStonePlacement="innerProps && innerProps.fuzzyStonePlacement"
        :animateStonePlacement="innerProps && innerProps.animateStonePlacement"
        :paintMap="innerProps && innerProps.paintMap"
        :heatMap="innerProps && innerProps.heatMap"
        :markerMap="innerProps && innerProps.markerMap"
        :ghostStoneMap="innerProps && innerProps.ghostStoneMap"
        :lines="innerProps && innerProps.lines"
        :dimmedVertices="innerProps && innerProps.dimmedVertices"
        :selectedVertices="innerProps && innerProps.selectedVertices"
        @click="$emit('click', $event)"
        @mousedown="$emit('mousedown', $event)"
        @mouseup="$emit('mouseup', $event)"
        @mousemove="$emit('mousemove', $event)"
        @mouseenter="$emit('mouseenter', $event)"
        @mouseleave="$emit('mouseleave', $event)"
    />
</template>

<script>
import Goban from './Goban.vue';

export default {
    components: {
        Goban
    },

    props: {
        innerProps: Object,
        showCoordinates: Boolean,
        maxWidth: Number,
        maxHeight: Number,
        rangeX: Array,
        rangeY: Array,
        signMap: Array,
        onResized: Function
    },

    data: function () {
        return {
            offsetWidth: 1,
            offsetHeight: 1,
            visibility: 'hidden'
        };
    },

    computed: {
        vertexSize: function () {
            /*
            let { offsetWidth = 1, offsetHeight = 1, maxWidth, maxHeight, vertexSize = 1 } = this;
            console.log('vertexSize = ' + vertexSize);
            let scale = Math.min(maxWidth / offsetWidth, maxHeight / offsetHeight);
            return Math.max(Math.floor(vertexSize * scale), 1);
            */
            return 1;
        }
    },

    mounted: function () {
        this.$nextTick(function () {
            let { offsetWidth, offsetHeight } = this.$el;
            console.log('offsetWidth = ' + offsetWidth);
            console.log('offsetHeight = ' + offsetHeight);
            if (this.offsetWidth !== offsetWidth)
                this.offsetWidth = offsetWidth;
            if (this.offsetHeight !== offsetHeight)
                this.offsetHeight = offsetHeight;

            // this.$nextTick(updateSize);
        });
    }


    /*
console.log('maxHeight = ' + maxHeight);
console.log('maxWidth = ' + maxWidth);
console.log('offsetWidth = ' + offsetWidth);
console.log('offsetHeight = ' + offsetHeight);
console.log('vertexSize = ' + vertexSize);
    */
};
</script>
