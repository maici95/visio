<template>
    <div
        id='view'
        @click.right='rightClick'
        @click.left='() => addMenu = false'
    >

        <div style='position: absolute; padding: 20px; z-index: 100;'>
            <b style='background: cyan; border: 2px solid #000; padding: 10px;'>{{ proMode ? 'PRO MODE' : 'EASY MODE' }}</b> || 
            <button @click='() => proMode = !proMode'>
                vaihda
<!--                 {{ proMode ? 'easy mode' : 'pro mode' }} -->
            </button>
            <br><br>
            ESC = poistaa valitun noden<br>
            1 = poistaa sinisen linkin<br>
            2 = poistaa oranssin linkin<br>
            3 = poistaa laatikon
        </div>

        <canvas ref='canvas'>

        </canvas>

        <window-object
            v-for='(o, i) in objects'
            :key='i'
            :x='o.x'
            :y='o.y'
            @updatePosition='loc => updatePosition(loc, i)'
            :params='o.params'
            :returns='o.returns'
            :title='o.title'
            @paramClick='param => handleParamClick(param, i)'
            @returnClick='r => handleReturnClick(r, i)'
            @flowIn='e => handleFlowIn(i)'
            @flowOut='e => handleFlowOut(i)'

            :activeActor='activeActor == i'
            :activeNode='activeNode'
        >
        </window-object>

        <div
            v-if='addMenu'
            class='add-menu'
            :style='{
                marginLeft: addMenuLocation[0]+"px",
                marginTop: addMenuLocation[1]+"px",
            }'
        >

            <div
                class='option'
                v-for='(b, i) in getAddMenuContent'
                :key='"button"+i'
                @click='() => !b.title ? addActor(b.actor) : null'
                :style='{
                    background: b.title ? "#5EBA7D" : "",
                }'
            >
                <div
                    :style='{
                        marginLeft: !b.title ? "10px" : "0"
                    }'
                >
                    {{ b.label }}
                </div>
            </div>
        </div>

    </div>
</template>

<script>
import WindowObject from './components/WindowObject.vue';

class Actor {
    constructor(x, y, title, params, returns) {
        this.x = x;
        this.y = y;
        this.title = title;

        this.params = params ?? [
/*             { label: 'tuote1' },
            { label: 'tuote2' },
            { label: 'a' },
            { label: 'tuote3' } */
        ];
        this.returns = returns ?? [
/*             { label: 'Ostoskori' },
            { label: 'SBC' },
            { label: 'Ostoskori2' } */
        ];

    }

}
Actor;

export default {
    components: {WindowObject},

    data() {
        return {
            objects: [],

            activeActor: {},
            activeNode: '',

            connections: [],

            nodeConnections: [],

            addMenu: true,
            addMenuLocation: [200, 200],

            proMode: true,

            proContent: [
                { label: 'Aloitus', title: true },
                { label: 'Pyyntö', actor: new Actor(0, 0, 'Pyyntö', [], [{ label: 'Tieto' }]) },

                { label: 'Parserit', title: true },
                { label: 'Hae tunniste', actor: new Actor(0, 0, 'Hae tunniste', [{ label: 'Tieto' }], [{ label: 'Tunniste'}])},

                { label: 'Palvelimet', title: true },
                { label: 'Palvelin 1', actor: new Actor(0, 0, 'Palvelin 1', [], [{ label: 'Palvelin' }]) },
                { label: 'Palvelin 2', actor: new Actor(0, 0, 'Palvelin 2', [], [{ label: 'Palvelin' }]) },
                { label: 'Palvelin 3', actor: new Actor(0, 0, 'Palvelin 3', [], [{ label: 'Palvelin' }]) },
                { label: 'Palvelin 4', actor: new Actor(0, 0, 'Palvelin 4', [], [{ label: 'Palvelin' }]) },

                { label: 'Hakijat', title: true },
                { label: 'Kuskit', actor: new Actor(0, 0, 'Hae kuskit', [{ label: 'Palvelin' }], [{ label: 'Kuskit' }])},
                { label: 'Materiaalit', actor: new Actor(0, 0, 'Hae materiaalit', [{ label: 'Palvelin' }], [{ label: 'Materiaalit' }])},
                { label: 'Keikat', actor: new Actor(0, 0, 'Hae keikat', [{ label: 'Palvelin' }], [{ label: 'Keikat' }])},

                { label: 'Etsijät', title: true },
                { label: 'Etsi kuski', actor: new Actor(0,0, 'Etsi kuski', [{label: 'Kuskit'},{label: 'Tieto'}] ,[{ label: 'Kuski'}]) },
                { label: 'Etsi materiaali', actor: new Actor(0,0, 'Etsi Materiaali', [{label: 'Materiaalit'},{label: 'Tieto'}] ,[{ label: 'Materiaali'}]) },

//                { label: 'Kuskit', actor: new Actor(0,0, 'Etsi kuski', [{label: 'Tieto'}] ,[{ label: 'Kuski'}]) }


                { label: 'Castit', title: true },
                { label: 'Keikka', actor: new Actor(0, 0, 'Keikka', [{ label: 'Tunniste' }, { label: 'Kohde' }, { label: 'Materiaali' }, { label: 'Tila' },{ label: 'Tieto' }], [{ label: 'Tunniste' }, { label: 'Kohde' }, { label: 'Materiaali' }, { label: 'Tila' }, { label: 'Tieto'}])},
                { label: '', actor: new Actor(0, 0, '', [{ label: 'Tieto' }], [])},
                { label: '', actor: new Actor(0, 0, '', [{ label: 'Tieto' }], [])},
                { label: '', actor: new Actor(0, 0, '', [{ label: 'Tieto' }], [])},

                { label: 'Toiminnot', title: true },
                { label: 'Lisää keikka', actor: new Actor(0, 0, 'Lisää keikka', [{label: 'Palvelin'}, { label: 'Tieto'}], [{ label: 'Tunniste' }])},
                { label: 'Poista keikka', actor: new Actor(0, 0, 'Poista keikka', [{label: 'Palvelin'},{ label: 'Tunniste'}], [{ label: 'Vastaus' }])},
                { label: 'Päivitä keikka', actor: new Actor(0, 0, 'Päivitä keikka', [{label: 'Palvelin'},{ label: 'Tunniste'}, { label: 'Tieto'}], [{ label: 'Vastaus' }])},
                { label: 'Hae keikka', actor: new Actor(0, 0, 'Hae keikka', [{label: 'Palvelin'},{ label: 'Tunniste'}], [{ label: 'Keikka' }])},


                { label: 'Palautukset', title: true },
                { label: 'Palauta vastaus', actor: new Actor(0, 0, 'Palauta vastaus', [{ label: 'Vastaus'}], []), },
                { label: 'Palauta tunniste', actor: new Actor(0, 0, 'Palauta tunniste', [{ label: 'Vastaus'}, { label: 'Tunniste'}], [])},
                { label: 'Palauta tietoa', actor: new Actor(0, 0, 'Palauta tietoa', [{ label: 'Vastaus'}, { label: 'Tietoa'}], [])},
/* 
                { label: '', actor: new Actor(0, 0, '', [], [])},
                { label: '', actor: new Actor(0, 0, '', [], [])},
                { label: '', actor: new Actor(0, 0, '', [], [])},
                { label: '', actor: new Actor(0, 0, '', [], [])},
                { label: '', actor: new Actor(0, 0, '', [], [])},
                { label: '', actor: new Actor(0, 0, '', [], [])},
                { label: '', actor: new Actor(0, 0, '', [], [])},
                { label: '', actor: new Actor(0, 0, '', [], [])}, */
                                            
            ],

            addMenuContent: [
                { label: 'Aloitus', title: true },
                { label: 'Ihminen', actor: new Actor(0, 0, 'Ihminen', [], [{ label: 'Minä'}])  },

                { label: 'Minä', title: true },
                { label: 'Lompakko', actor: new Actor(0, 0, 'Lompakko', [{ label: 'Minä'}], [{ label: 'Rahaa'}, {label: 'Kortti'} ]) },

                { label: 'Siirtymät', title: true },
                { label: 'Kotiin', actor: new Actor(0, 0, 'Siirry kotiin') },
                { label: 'Prismaan', actor: new Actor(0, 0, 'Siirry Prismaan') },
                { label: 'K- Rautaan', actor: new Actor(0, 0, 'Siirry K-Rautaan') },
                { label: 'Lentokentälle', actor: new Actor(0, 0, 'Siirry Lentokentälle') },
                { label: 'Kassalle', actor: new Actor(0, 0, 'Siirry kassalle', [{ label: 'Ostoskori'}, { label: 'Tuote' }], [{ label: 'Tuotteet'} ]), },

                { label: 'Etsi', title: true },
                { label: 'Tomaatti', actor: new Actor(0, 0, 'Etsi tuote', [],  [{ label: 'Tomaatti'}]) },
                { label: 'Mutteri', actor: new Actor(0, 0, 'Etsi tuote', [],  [{ label: 'Mutteri'}]) },
                { label: 'Vasara', actor: new Actor(0, 0, 'Etsi tuote', [],  [{ label: 'Vasara'}]) },
                { label: 'Naula', actor: new Actor(0, 0, 'Etsi tuote', [],  [{ label: 'Naula'}]) },
                { label: '2x4', actor: new Actor(0, 0, 'Etsi tuote', [],  [{ label: '2x4'}]) },
                { label: 'Teippi 10m', actor: new Actor(0, 0, 'Etsi tuote', [],  [{ label: 'Teippi 10m'}]) },
                { label: 'Siima 100m', actor: new Actor(0, 0, 'Etsi tuote', [],  [{ label: 'Siima 100m'}]) },
                { label: 'Uimarengas', actor: new Actor(0, 0, 'Etsi tuote', [],  [{ label: 'Uimarengas'}]) },

                { label: 'Toiminnot', title: true },
                { label: 'Lisää tuote koriin', actor: new Actor(0, 0, 'Lisää tuote koriin', [{ label: 'Ostoskori' }, { label: 'Tuote'}], [{ label: 'Ostoskori' }]),  },
                { label: 'Ota ostoskori', actor: new Actor(0, 0, 'Ota ostoskori', [], [{ label: 'Ostoskori'}]),  },
                { label: 'Palauta ostoskori', actor: new Actor(0, 0, 'Palauta ostoskori', [{ label: 'Ostoskori'}], []),  },
                { label: 'Maksa ostokset', actor: new Actor(0, 0, 'Maksaostokset', [{ label: 'Tuotteet'}, { label: '€' }], [{ label: 'Tuotteet'}]),  },
                { label: 'Pakkaa ostokset', actor: new Actor(0, 0, 'Pakkaa ostokset', [{ label: 'Tuotteet'}], [{ label: 'Tuotteet'}]),  },
                { label: 'Osta lentolippu', actor: new Actor(0, 0, 'Osta lentolippu', [{ label: '€'}], [{ label: 'Lentolippu'}]),  },
                { label: 'Nouse lentokoneeseen', actor: new Actor(0, 0, 'Nouse lentokoneeseen', [{ label: 'Lentolippu'}], []),  },

                { label: 'Konvertorit', title: true },
                { label: 'Esine rahaksi', actor: new Actor(0, 0, 'Esine rahaksi', [{ label: 'Maksuväline'}], [{ label: '€'}] )}
            ],

        }
    },

    computed: {
        getAddMenuContent() {
            if (!this.proMode) {
                return this.addMenuContent;
            }
            return this.proContent;
        }
    },

    watch: {
        objects: {
            handler() {
                this.draw();
            },
            deep: true
        },
        connections() {
            this.draw();
        },
        nodeConnections() {
            this.draw();
        }
    },

    mounted() {
        window.addEventListener('keyup', event => {
            if (event.key == 'Escape') {
                this.activeActor = {};
                this.activeNode = '';
            }
            if (event.key == '1') {
                this.connections.pop();
            }
            if (event.key == '2') {
                this.nodeConnections.pop();
            }
            if (event.key == '3') {
                this.objects.pop();
            }
        });

        const c = this.$refs.canvas;
        const ctx = c.getContext('2d');

        c.height = window.innerHeight;
        c.width = window.innerWidth;

        ctx.strokeStyle = '333';
        ctx.lineWidth = 2;

    },

    methods: {
        addActor(actor) {
            const a = {...actor};
            a.x = this.addMenuLocation[0];
            a.y = this.addMenuLocation[1];

            this.objects.push(a);
            this.addMenu = false;
        },

        reset() {
            this.activeActor = {};
            this.activeNode = '';
        },

        rightClick(event) {
            event.preventDefault();
//            this.objects.push(new Actor(event.pageX, event.pageY, 'Lisää tuote koriin'));
            this.addMenu = true;
            this.addMenuLocation = [
                event.pageX,
                event.pageY,
            ];
        },

        updatePosition(loc, i) {    
            this.objects[i].x = loc[0];
            this.objects[i].y = loc[1];
        },

        handleParamClick(param, i) {
            if (this.activeNode.match('return_')) {
                this.nodeConnections.push([
                    [this.activeActor, this.activeNode],
                    [i, param.label],
                ]);

                this.activeActor = {};
                this.activeNode = '';
            } else {

                this.activeActor = i;
                this.activeNode = 'param_'+param.label;
            }
        },

        handleReturnClick(r, i) {
            this.activeActor = i;
            this.activeNode = 'return_'+r.label;
        },

        handleFlowIn(i) {
            if (this.activeNode == 'flowout') {
                this.connections.push([this.activeActor, i]);
                this.reset();
                return;
            }

            if (this.activeNode != 'flowin') {
                this.activeActor = i;
                this.activeNode = 'flowin';
                return;
            }

        },

        handleFlowOut(i) {
            this.activeActor = i;
            this.activeNode = 'flowout';
        },

        draw() {
            const c = this.$refs.canvas;
            const ctx = c.getContext('2d');ctx;

            ctx.clearRect(0, 0, 2000, 2000);

            ctx.fillStyle = 'blue';
            ctx.strokeStyle = 'blue';

            this.connections.forEach(c => {
                ctx.beginPath();
                ctx.moveTo(
                    this.objects[c[0]].x + 190,
                    this.objects[c[0]].y + 20
                );

                ctx.quadraticCurveTo(
                    (this.objects[c[0]].x + 190 + this.objects[c[1]].x + 20) / 2,
                    //(this.objects[c[0]].y + 20 + this.objects[c[1]].y + 20) / 2,
                    this.objects[c[0]].y + 20,

                    this.objects[c[1]].x + 20 - 30,
                    this.objects[c[1]].y + 20
                );

/*                 ctx.lineTo(
                    this.objects[c[1]].x + 20 - 30,
                    this.objects[c[1]].y + 20
                ); */
                ctx.stroke();

                ctx.beginPath();
                ctx.arc(
                    this.objects[c[0]].x + 190,
                    this.objects[c[0]].y + 20,
                    6,
                    0,
                    Math.PI * 2
                );
                ctx.fill();

                ctx.arc(
                    this.objects[c[1]].x + 20 - 30,
                    this.objects[c[1]].y + 20,
                    6,
                    0,
                    Math.PI * 2
                );
                ctx.fill();

            });

            ctx.fillStyle = 'orange';
            ctx.strokeStyle = 'orange';

            this.nodeConnections.forEach(c => {
                ctx.beginPath();

                const from = this.objects[c[0][0]];
                const to = this.objects[c[1][0]];

                const offset1 = to.params.findIndex(p => p.label == c[1][1]);
                const offset2 = from.returns.findIndex(r => 'return_'+r.label == c[0][1]);

                ctx.moveTo(
                    from.x + 190,
                    from.y + 20 + (offset2 * 20) + 35,
                );

                ctx.quadraticCurveTo(
                    (from.x + 190 + to.x + 20 - 30) / 2,
                    //(this.objects[c[0]].y + 20 + this.objects[c[1]].y + 20) / 2,
                    from.y + 20 + (offset2 * 20) + 35,

                    to.x + 20 - 30,
                    to.y + 20 + (offset1 * 20) + 35
                );

/*                 ctx.lineTo(
                    to.x + 20 - 30,
                    to.y + 20 + (offset1 * 20) + 35
                ); */
                ctx.stroke();

                ctx.beginPath();
                ctx.arc(
                    to.x + 20 - 30,
                    to.y + 20 + (offset1 * 20) + 35,
                    6,
                    0,
                    Math.PI * 2
                );
                ctx.fill();

                ctx.beginPath();
                ctx.arc(
                    from.x + 190,
                    from.y + 20 + (offset2 * 20) + 35,
                    6,
                    0,
                    Math.PI * 2
                );
                ctx.fill();

            });


        }

    }

}
</script>

<style scoped>
#view {
    width: 100vw;
    height: 100vh;

    background:
        linear-gradient(-90deg, rgba(0,0,0,.05) 1px, transparent 1px),
        linear-gradient(rgba(0,0,0,.05) 1px, transparent 1px), 
        linear-gradient(-90deg, rgba(0, 0, 0, .04) 1px, transparent 1px),
        linear-gradient(rgba(0,0,0,.04) 1px, transparent 1px),
        linear-gradient(transparent 3px, #f2f2f2 3px, #f2f2f2 78px, transparent 78px),
        linear-gradient(-90deg, #aaa 1px, transparent 1px),
        linear-gradient(-90deg, transparent 3px, #f2f2f2 3px, #f2f2f2 78px, transparent 78px),
        linear-gradient(#aaa 1px, transparent 1px),
        #f2f2f2;
    background-size:
        4px 4px,
        4px 4px,
        80px 80px,
        80px 80px,
        80px 80px,
        80px 80px,
        80px 80px,
        80px 80px;
}

canvas {
    position: absolute;
}
</style>

<style>
body {
    font-family: Arial;
    font-size: 12px;
    padding: 0;
    margin: 0;
}

.add-menu {
    position: absolute;
    width: 200px;
    height: auto;
    min-height: 50px;
    border: 1px solid #ddd;
    background: #f5f5f5;
    border-radius: 5px;
    padding: 10px 0;
    outline: 2px solid #000;
}

.option {
    padding: 5px;
    width: calc(100% - 10px);
    cursor: pointer;
    border-bottom: 1px solid #ddd;
}

.option:hover {
    background: cyan;
}



</style>
