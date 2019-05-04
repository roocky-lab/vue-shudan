<template>
<section id="app" style="display: grid; grid-template-columns: 15em auto; column-gap: 1em;">
    <form style="display: flex; flex-direction: column;">
        <p style="margin: 0px 0px 0.5em;">
            Size:
            <button type="button" @click="vertexSize = Math.max(vertexSize - 4, 4)">-</button>
            <button type="button" title="Reset" @click="vertexSize = 24">•</button>
            <button type="button" @click="vertexSize += 4">+</button>
        </p>
        <p style="margin: 0px 0px 0.5em;">
            Stones:
            <button type="button" title="Reset" @click="signMap = rawSignMap">•</button>
        </p>
        <div>
            <template v-for="(c, i) in checkBoxs">
                <label style="display: flex; align-items: center;" :key="i">
                    <input
                        type="checkbox"
                        style="marginRight: .5em;"
                        :value="c.stateKey"
                        v-model="checkedNames"
                    >
                    <span style="user-select: none;">{{ c.text }}</span>
                </label>
            </template>
        </div>
    </form>

    <div>
        <Goban
            :vertex-size="vertexSize"
            :animate="true"
            :busy="isBusy"
            :range-x="showCorner ? [8, 18] : undefined"
            :range-y="showCorner ? [12, 13] : undefined"
            :coord-x="alternateCoordinates ? chineseCoordx : undefined"
            :coord-y="alternateCoordinates ? chineseCoordy : undefined"
            :sign-map="signMap"
            :show-coordinates="showCoordinates"
            :fuzzy-stone-placement="fuzzyStonePlacement"
            :animate-stone-placement="animateStonePlacement"
            :paint-map="showPaintMap ? paintMap : undefined"
            :heat-map="showHeatMap ? heatMap : undefined"
            :marker-map="showMarkerMap ? markerMap : undefined"
            :ghost-stone-map="showGhostStones ? ghostStoneMap : undefined"
            :lines="showLines ? [
                {type: 'line', v1: [15, 6], v2: [12, 15]},
                {type: 'arrow', v1: [10, 4], v2: [5, 7]}
            ] : []"
            :dimmed-vertices="showDimmedStones ? [
                [2, 14], [2, 13], [5, 13], [6, 13],
                [9, 3], [9, 5], [10, 5], [14, 7],
                [13, 13], [13, 14], [18, 13]
            ] : []"
            :selected-vertices="showSelection ? [
                [9, 7], [9, 8], [10, 7], [10, 8]
            ] : []"
            @click="onVertexClick"
        />
    </div>
</section>
</template>

<script>
import { Goban } from '../src/components/Shudan';

const chineseCoordx = [
    '一', '二', '三', '四', '五', '六', '七', '八', '九', '十',
    '十一', '十二', '十三', '十四', '十五', '十六', '十七', '十八', '十九'
];

const chineseCoordy = [...Array(19)].map((_, i) => i);
console.log('chineseCoordy = ' + chineseCoordy);

const rawSignMap = [
    [0, 0, 0, -1, -1, -1, 1, 0, 1, 1, -1, -1, 0, -1, 0, -1, -1, 1, 0],
    [0, 0, -1, 0, -1, 1, 1, 1, 0, 1, -1, 0, -1, -1, -1, -1, 1, 1, 0],
    [0, 0, -1, -1, -1, 1, 1, 0, 0, 1, 1, -1, -1, 1, -1, 1, 0, 1, 0],
    [0, 0, 0, 0, -1, -1, 1, 0, 1, -1, 1, 1, 1, 1, 1, 0, 1, 0, 0],
    [0, 0, 0, 0, -1, 0, -1, 1, 0, 0, 1, 1, 0, 0, 0, 1, 1, 1, 0],
    [0, 0, -1, 0, 0, -1, -1, 1, 0, -1, -1, 1, -1, -1, 0, 1, 0, 0, 1],
    [0, 0, 0, -1, -1, 1, 1, 1, 1, 1, 1, 1, 1, -1, -1, -1, 1, 1, 1],
    [0, 0, -1, 1, 1, 0, 1, -1, -1, 1, 0, 1, -1, 0, 1, -1, -1, -1, 1],
    [0, 0, -1, -1, 1, 1, 1, 0, -1, 1, -1, -1, 0, -1, -1, 1, 1, 1, 1],
    [0, 0, -1, 1, 1, -1, -1, -1, -1, 1, 1, 1, -1, -1, -1, -1, 1, -1, -1],
    [-1, -1, -1, -1, 1, 1, 1, -1, 0, -1, 1, -1, -1, 0, -1, 1, 1, -1, 0],
    [-1, 1, -1, 0, -1, -1, -1, -1, -1, -1, 1, -1, 0, -1, -1, 1, -1, 0, -1],
    [1, 1, 1, 1, -1, 1, 1, 1, -1, 1, 0, 1, -1, 0, -1, 1, -1, -1, 0],
    [0, 1, -1, 1, 1, -1, -1, 1, -1, 1, 1, 1, -1, 1, -1, 1, 1, -1, 1],
    [0, 0, -1, 1, 0, 0, 1, 1, -1, -1, 0, 1, -1, 1, -1, 1, -1, 0, -1],
    [0, 0, 1, 0, 1, 0, 1, 1, 1, -1, -1, 1, -1, -1, 1, -1, -1, -1, 0],
    [0, 0, 0, 0, 1, 1, 0, 1, -1, 0, -1, -1, 1, 1, 1, 1, -1, -1, -1],
    [0, 0, 1, 1, -1, 1, 1, -1, 0, -1, -1, 1, 1, 1, 1, 0, 1, -1, 1],
    [0, 0, 0, 1, -1, -1, -1, -1, -1, 0, -1, -1, 1, 1, 0, 1, 1, 1, 0]
];

const paintMap = [
    [-1, -1, -1, -1, -1, -1, 1, 1, 1, 1, -1, -1, -1, -1, -1, -1, -1, 1, 1],
    [-1, -1, -1, -1, -1, 1, 1, 1, 1, 1, -1, -1, -1, -1, -1, -1, 1, 1, 1],
    [-1, -1, -1, -1, -1, 1, 1, 1, 1, 1, 1, -1, -1, 1, -1, 1, 1, 1, 1],
    [-1, -1, -1, -1, -1, -1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
    [-1, -1, -1, -1, -1, -1, -1, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1],
    [-1, -1, -1, -1, -1, -1, -1, 1, 1, 1, 1, 1, -1, -1, 0, 1, 1, 1, 1],
    [-1, -1, -1, -1, -1, 1, 1, 1, 1, 1, 1, 1, 1, -1, -1, -1, 1, 1, 1],
    [-1, -1, -1, 1, 1, 1, 1, -1, -1, 1, 0, 1, -1, -1, -1, -1, -1, -1, 1],
    [-1, -1, -1, -1, 1, 1, 1, 0, -1, 1, -1, -1, -1, -1, -1, 1, 1, 1, 1],
    [-1, -1, -1, 1, 1, -1, -1, -1, -1, 1, 1, 1, -1, -1, -1, -1, 1, -1, -1],
    [-1, -1, -1, -1, 1, 1, 1, -1, -1, -1, 1, -1, -1, -1, -1, 1, 1, -1, -1],
    [-1, 1, -1, 0, -1, -1, -1, -1, -1, -1, 1, -1, -1, -1, -1, 1, -1, -1, -1],
    [1, 1, 1, 1, -1, 1, 1, 1, -1, 1, 1, 1, -1, -1, -1, 1, -1, -1, -1],
    [1, 1, 1, 1, 1, 1, 1, 1, -1, 1, 1, 1, -1, -1, -1, 1, 1, -1, -1],
    [1, 1, 1, 1, 1, 1, 1, 1, -1, -1, 0, 1, -1, -1, -1, 1, -1, -1, -1],
    [1, 1, 1, 1, 1, 1, 1, 1, 1, -1, -1, 1, -1, -1, 1, -1, -1, -1, -1],
    [1, 1, 1, 1, 1, 1, 1, 1, -1, -1, -1, -1, 1, 1, 1, 1, -1, -1, -1],
    [1, 1, 1, 1, -1, 1, 1, -1, -1, -1, -1, 1, 1, 1, 1, 1, 1, -1, 1],
    [1, 1, 1, 1, -1, -1, -1, -1, -1, -1, -1, -1, 1, 1, 1, 1, 1, 1, 1]
];

const heatMap = (() => {
    let _ = null;
    let O = (strength, text) => ({ strength, text });

    return [
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, O(7), O(9, '80%\n13.5k'), _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, O(3), _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, O(2), _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, O(1, '20%\n111'), _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, O(5, '67%\n2315'), O(4), _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _]
    ];
})();

const markerMap = (() => {
    let _ = null;
    let O = { type: 'circle' };
    let X = { type: 'cross' };
    let T = { type: 'triangle' };
    let Q = { type: 'square' };
    let $ = { type: 'point' };
    let S = { type: 'loader' };
    let L = label => ({ type: 'label', label });
    let A = L('a');
    let B = L('b');
    let C = L('c');

    return [
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, O, O, O, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, X, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, X, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, X, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, T, T, T, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, $, $, $, _, _, _, _, _, _, _, _, _, _, _, S, S, S, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, Q, _, _, _, _, _, _, _, _, _, L('Long\nlabel with linebreak')],
        [_, _, _, _, _, _, _, _, Q, _, _, _, _, _, _, _, _, _, C],
        [_, _, _, _, _, _, _, _, Q, _, _, _, _, _, _, _, _, _, B],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, A]
    ];
})();

const ghostStoneMap = (() => {
    let _ = null;
    let O = t => ({ sign: -1, type: t });
    let X = t => ({ sign: 1, type: t });
    let o = t => ({ sign: -1, type: t, faint: true });
    let x = t => ({ sign: 1, type: t, faint: true });

    return [
        [X(), x(), _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [O(), o(), _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [X('good'), x('good'), _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [X('interesting'), x('interesting'), _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [X('doubtful'), x('doubtful'), _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [X('bad'), x('bad'), _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _],
        [_, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _, _]
    ];
})();

export default {
    name: 'App',
    components: {
        Goban
    },

    data: function () {
        return {
            signMap: rawSignMap,
            vertexSize: 24,
            showCoordinates: false,
            alternateCoordinates: false,
            showCorner: false,
            showDimmedStones: false,
            fuzzyStonePlacement: false,
            animateStonePlacement: false,
            showPaintMap: false,
            showHeatMap: false,
            showMarkerMap: false,
            showGhostStones: false,
            showLines: false,
            showSelection: false,
            isBusy: false,

            rawSignMap,
            chineseCoordx,
            chineseCoordy,
            paintMap,
            heatMap,
            markerMap,
            ghostStoneMap,

            checkedNames: []
        };
    },

    methods: {
        onVertexClick: function ([x, y]) {
            let signMap = JSON.parse(JSON.stringify(this.signMap));
            signMap[y][x] = Math.sign(Math.random() - 0.5) || 1;
            this.signMap = signMap;
        }
    },

    computed: {
        checkBoxs: function () {
            return [
                { stateKey: 'showCoordinates', text: 'Show coordinates' },
                { stateKey: 'alternateCoordinates', text: 'Alternate coordinates' },
                { stateKey: 'showCorner', text: 'Show lower right corner only' },
                { stateKey: 'showDimmedStones', text: 'Dim dead stones' },
                { stateKey: 'fuzzyStonePlacement', text: 'Fuzzy stone placement' },
                { stateKey: 'animateStonePlacement', text: 'Animate stone placement' },
                { stateKey: 'showMarkerMap', text: 'Show markers' },
                { stateKey: 'showGhostStones', text: 'Show ghost stones' },
                { stateKey: 'showPaintMap', text: 'Show paint map' },
                { stateKey: 'showHeatMap', text: 'Show heat map' },
                { stateKey: 'showLines', text: 'Show lines' },
                { stateKey: 'showSelection', text: 'Show selection' },
                { stateKey: 'isBusy', text: 'Busy' }
            ];
        }
    },

    watch: {
        checkedNames: function () {
            let { checkBoxs, checkedNames } = this;
            checkBoxs.map((value) => {
                let { stateKey } = value;
                let newState = checkedNames.indexOf(stateKey) > -1 ? true : false;
                if (this[stateKey] != newState)
                    this[stateKey] = newState;
            });
        }
    }
};
</script>


<style>
body {
    font-family: "Segoe UI", Ubuntu, Helvetica, Arial, sans-serif;
}
</style>
