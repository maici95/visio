<template>
    <div
        class='container'
        :style='{
            marginLeft: x+"px",
            marginTop: y+"px",
            outline: activeActor ? "3px solid red" : ""
        }'
    >
        <slot>
        </slot>


        <div
            class='header'
            @drag='dragHandler'
            draggable
            @dragstart='dragStart'    
        >

            <div
                class='flowicon'
                @click='e => $emit("flowIn", e)'
                :style='{
                    outline: activeActor && activeNode == "flowin" ? "4px solid blue" : ""
                }'
            >
            >>
            </div>

            <h1
            >
                {{ title }}

            </h1>

            <div
                class='flowicon'
                @click='e => $emit("flowOut", {})'
                :style='{
                    outline: activeActor && activeNode == "flowout" ? "4px solid blue" : ""
                }'
            >
            >>
            </div>

        </div>

        <div
            class='bottom'
        >
            <div>
                <div
                    v-for='(param, i) in params'
                    :key='"p"+i'
                >
                    <div class='nodecon'>
                        <div
                            class='circle'
                            @click='e => $emit("paramClick", param)'
                            :style='{
                                outline: activeActor && activeNode == "param_"+param.label ? "4px solid blue" : ""
                            }'
                        ></div>
                        <div style='padding: 0 5px;'>{{ param.label }}</div>
                    </div>
                </div>
            </div>

            <div>
                <div
                    v-for='(r, i) in returns'
                    :key='"r"+i'
                >
                    <div class='nodecon'>
                        <div style='padding: 0 5px;'>{{ r.label }}</div>
                        <div
                            class='circle'
                            @click='e => $emit("returnClick", r)'
                            :style='{
                                outline: activeActor && activeNode == "return_"+r.label ? "4px solid blue" : "",
                                background: "green"
                            }'
                        ></div>
                    </div>
                </div>
            </div>

        </div>
    </div>
</template>

<script>

export default {
    props: {
        x: [String, Number],
        y: [String, Number],
        params: Array,
        returns: Array,
        title: String,

        activeActor: Boolean,
        activeNode: String
    },

    data() {
        return {
            offsetX: 0,
            offsetY: 0
        }
    },

    methods: {
        dragHandler(event) {
            if (event.pageX != 0 && event.pageY != 0) {
                this.$emit('updatePosition', [
                    event.pageX - this.offsetX,
                    event.pageY - this.offsetY
                ]);
            }
        },

        dragStart(event) {
            this.offsetX = Math.abs(this.x - event.pageX);
            this.offsetY = Math.abs(this.y - event.pageY);
            event.dataTransfer.setDragImage(event.target, window.outerWidth, window.outerHeight);
        }
    }
}
</script>


<style scoped>
.container {
    position: absolute;
    width: 180px;
    height: auto;
    border: 1px solid #ddd;
    border-radius: 5px;
    background: #F9F9F9;
    outline: 2px solid #000;
}
.bottom {
    display: flex;
    justify-content: space-between;
    padding: 10px 0;
}

.flowicon {
    font-weight: bold;
    cursor: pointer;

}

.flowicon:hover {
    color: cyan;
}

.header {
    display: flex;
    justify-content: space-between;

    border-top-left-radius: 5px;
    border-top-right-radius: 5px;
    padding: 10px;
    background: rgb(200, 200, 200);
}

header:hover {
    background: skyblue;
    cursor: grab;
}

h1 {
    font-size: 14px;
    margin: 0;
    font-weight: 300;
}

.circle {
    width: 13px;
    height: 13px;
    background: red;
    border-radius: 100%;
    margin: 0;
}

.circle:hover {
    outline: 2px solid cyan;
    cursor: pointer;
}

.nodecon {
    display: flex;
    justify-content: space-between;
    margin-bottom: 7px;
}
</style>
