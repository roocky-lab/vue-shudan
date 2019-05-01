<script>
export default {
    functional: true,

    props: {
        vertexSize: Number,
        width: Number,
        height: Number,
        xs: Array,
        ys: Array,
        hoshis: Array
    },

    render: function (h, _this) {
        let { vertexSize, width, height, xs, ys, hoshis } = _this.props;
        let halfVertexSize = vertexSize / 2;
        let fl = Math.floor;

        return (xs.length > 0 && ys.length > 0 && h('svg',
            {
                class: 'shudan-grid',
                style: {
                    position: 'absolute',
                    top: 0,
                    left: 0,
                    width: '100%',
                    height: '100%',
                    zIndex: 0
                }
            }, [
                // Draw grid lines

                ys.map((_, i) => {
                    let x = xs[0] === 0 ? halfVertexSize : 0;

                    return h('rect', {
                        key: `h${i}`,

                        class: 'shudan-gridline shudan-horizontal',
                        attrs: {
                            x: fl(x),
                            y: fl((2 * i + 1) * halfVertexSize),
                            width: xs[xs.length - 1] === width - 1
                                ? fl((2 * xs.length - 1) * halfVertexSize - x)
                                : fl(xs.length * vertexSize - x),
                            height: 1
                        }
                    });
                }),

                xs.map((_, i) => {
                    let y = ys[0] === 0 ? halfVertexSize : 0;

                    return h('rect', {
                        key: `v${i}`,

                        class: 'shudan-gridline shudan-vertical',
                        attrs: {
                            x: fl((2 * i + 1) * halfVertexSize),
                            y: fl(y),
                            width: 1,
                            height: ys[ys.length - 1] === height - 1
                                ? fl((2 * ys.length - 1) * halfVertexSize - y)
                                : fl(ys.length * vertexSize - y)
                        }
                    });
                }),

                // Draw hoshi points

                hoshis.map(([x, y]) => {
                    let i = xs.indexOf(x);
                    let j = ys.indexOf(y);
                    if (i < 0 || j < 0) return;

                    return h('circle', {
                        key: [x, y].join('-'),

                        class: 'shudan-hoshi',
                        attrs: {
                            cx: fl((2 * i + 1) * halfVertexSize) + .5,
                            cy: fl((2 * j + 1) * halfVertexSize) + .5,
                            r: '.1em'
                        }
                    });
                })
            ]
        ));
    }
};
</script>
