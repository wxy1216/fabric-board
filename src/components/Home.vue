<template>
    <div>
        <a-row type="flex">
            <a-col flex="50px" class="left-content">
                <template v-for="btn in btns">
                    <a-button
                        @click="onChange($event, btn.value)"
                        :key="btn.value"
                    >
                        <a-icon :type="btn.icon"></a-icon>
                    </a-button>
                </template>
            </a-col>
            <a-col flex="auto" class="main-content">
                <my-canvas ref="canvas"></my-canvas>
            </a-col>
        </a-row>
    </div>
</template>
<script>
import MyCanvas from "./MyCanvas";

export default {
    name: "HelloWorld",
    data() {
        return {
            btns: [
                { icon: "highlight", value: "freeDraw" },
                { icon: "minus", value: "line" },
                { icon: "arrow-right", value: "arrow" },
                { icon: "small-dash", value: "dashLine" },
                { icon: "reload", value: "3" },
                { icon: "border", value: "5" },
                { icon: "font-colors", value: "6" },
                { icon: "inbox", value: "erase" },
                { icon: "save", value: "4" },
            ],
        };
    },
    components: {
        MyCanvas,
    },
    methods: {
        onChange(e, value) {
            e.preventDefault();
            e.stopPropagation();
            this.$refs.canvas.startDraw(value);
        },
    },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.left-content >>> .anticon {
    font-size: 24px;
}
.main-content {
    height: calc(100vh - 12px);
    background: black;
}
.left-btn {
    width: 40px;
    border-radius: 5px;
    padding: 2px;
}
</style>
