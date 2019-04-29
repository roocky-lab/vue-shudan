<template>
    <section id="app" style="display: grid; grid-template-columns: 17em auto; column-gap: 1em;">
        <form style="display: flex; flex-direction: column;">
            <p style="margin: 0px 0px 0.5em;">
                Size:
                <button type="button" @click="vertexSize = Math.max(vertexSize - 4, 4)">-</button>
                <button type="button" title="Reset" @click="vertexSize = 24">•</button>
                <button type="button" @click="vertexSize += 4">+</button>
            </p>
            <p style="margin: 0px 0px 0.5em;">
                Stones:
                <!-- XXX: 复位未实现 -->
                <button type="button" title="Reset" @click="signMap = rawSignMap">•</button>
            </p>

            <label>
                <input type="checkbox" v-model="showCoordinates">Show coordinates
            </label>
            <label>
                <input type="checkbox" v-model="alternateCoordinates">Alternate coordinates
            </label>
            <label>
                <input type="checkbox" v-model="showCorner">Show lower right corner only
            </label>
            <label>
                <input type="checkbox" v-model="showDimmedStones">Dim dead stones
            </label>
            <label>
                <input type="checkbox" v-model="fuzzyStonePlacement">Fuzzy stone placement
            </label>
            <label>
                <input type="checkbox" v-model="animateStonePlacement">Animate stone placement
            </label>
            <label>
                <input type="checkbox" v-model="showMarkerMap">Show markers
            </label>
            <label>
                <input type="checkbox" v-model="showGhostStones">Show ghost stones
            </label>
            <label>
                <input type="checkbox" v-model="showPaintMap">Show paint map
            </label>
            <label>
                <input type="checkbox" v-model="showHeatMap">Show heat map
            </label>
            <label>
                <input type="checkbox" v-model="showLines">Show lines
            </label>
            <label>
                <input type="checkbox" v-model="showSelection">Show selection
            </label>
            <label>
                <input type="checkbox" v-model="isBusy">Busy
            </label>
        </form>
        <div>
            <Goban
                :vertexSize="vertexSize"
                :animate="true"
                :busy="isBusy"
                :rangeX="showCorner ? [8, 18] : undefined"
                :rangeY="showCorner ? [12, 18] : undefined"
                :coordX="alternateCoordinates ? i => chineseCoord[i] : undefined"
                :coordY="alternateCoordinates ? i => i + 1 : undefined"
                :signMap="signMap"
                :showCoordinates="showCoordinates"
                :fuzzyStonePlacement="fuzzyStonePlacement"
                :animateStonePlacement="animateStonePlacement"
                :paintMap="showPaintMap ? paintMap : undefined"
                :heatMap="showHeatMap ? heatMap : undefined"
                :markerMap="showMarkerMap ? markerMap : undefined"
                :ghostStoneMap="showGhostStones ? ghostStoneMap : undefined"
                :lines="showLines ? [
                        {type: 'line', v1: [15, 6], v2: [12, 15]},
                        {type: 'arrow', v1: [10, 4], v2: [5, 7]}
                    ] : []"
                :dimmedVertices="showDimmedStones ? [
                        [2, 14], [2, 13], [5, 13], [6, 13],
                        [9, 3], [9, 5], [10, 5], [14, 7],
                        [13, 13], [13, 14], [18, 13]
                    ] : []"
                :selectedVertices="showSelection ? [
                        [9, 7], [9, 8], [10, 7], [10, 8]
                    ] : []"
                @vertex-click="onVertexClick"
            />
        </div>
    </section>
</template>

<script>
import { Goban } from './components/Shudan';

const chineseCoord = [
    '一',
    '二',
    '三',
    '四',
    '五',
    '六',
    '七',
    '八',
    '九',
    '十',
    '十一',
    '十二',
    '十三',
    '十四',
    '十五',
    '十六',
    '十七',
    '十八',
    '十九'
];

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
        [
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            O(7),
            O(9, '80%\n13.5k'),
            _,
            _,
            _,
            _
        ],
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
        [
            _,
            O(1, '20%\n111'),
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _
        ],
        [
            _,
            O(5, '67%\n2315'),
            O(4),
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _
        ],
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
        [
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            Q,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            L('Long\nlabel with linebreak')
        ],
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
        [
            X('good'),
            x('good'),
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _
        ],
        [
            X('interesting'),
            x('interesting'),
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _
        ],
        [
            X('doubtful'),
            x('doubtful'),
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _,
            _
        ],
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
    name: 'app',
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
            showDimmedStones: true,
            fuzzyStonePlacement: false,
            animateStonePlacement: false,
            showPaintMap: false,
            showHeatMap: false,
            showMarkerMap: true,
            showGhostStones: false,
            showLines: true,
            showSelection: true,
            isBusy: false,

            /* 声明 */
            rawSignMap,
            chineseCoord,
            paintMap,
            heatMap,
            markerMap,
            ghostStoneMap
        };
    },

    methods: {
        onVertexClick: function ([x, y]) {
            let signMap = JSON.parse(JSON.stringify(this.signMap));
            signMap[y][x] = Math.sign(Math.random() - 0.5) || 1;
            this.signMap = signMap;
        }
    }
};
</script>


<style>
body {
    font-family: "Segoe UI", Ubuntu, Helvetica, Arial, sans-serif;
}
/* XXX 
.shudan-coordx span {
    font-size: 0.45em;
}
*/
</style>