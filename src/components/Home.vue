<template>
    <div>
        <a-row type="flex">
            <a-col flex="50px" class="left-content">
                <a-radio-group v-model="value" @change="onChange">
                    <a-radio-button :value="1" :style="radioStyle">
                        <a-icon type="highlight" />
                    </a-radio-button>
                    <a-radio-button :value="2" :style="radioStyle">
                        <a-icon type="arrow-right" />
                    </a-radio-button>
                    <a-radio-button :value="3" :style="radioStyle">
                        <a-icon type="minus" />
                    </a-radio-button>
                    <a-radio-button :value="4" :style="radioStyle">
                        <a-icon type="small-dash" />
                    </a-radio-button>
                    <a-radio-button :value="5" :style="radioStyle">
                        <a-icon type="reload" />
                    </a-radio-button>
                    <a-radio-button :value="6" :style="radioStyle">
                        <a-icon type="arrow-right" />
                    </a-radio-button>
                    <a-radio-button :value="7" :style="radioStyle">
                        <a-icon type="border" />
                    </a-radio-button>
                    <a-radio-button :value="8" :style="radioStyle">
                        <a-icon type="arrow-right" />
                    </a-radio-button>
                    <a-radio-button :value="9" :style="radioStyle">
                        <a-icon type="font-colors" />
                    </a-radio-button>
                    <a-radio-button :value="10" :style="radioStyle">
                        <a-icon type="inbox" />
                    </a-radio-button>
                </a-radio-group>
            </a-col>
            <a-col flex="auto" class="main-content"></a-col>
        </a-row>
    </div>
</template>

<script>
import { fabric } from "fabric";
export default {
    name: "HelloWorld",
    data() {
        return {
            value: "1",
            radioStyle: {
                width: "40px",
                "border-radius": "5px",
                padding: "2px",
            },
        };
    },
    methods: {
        onChange(value) {},
    },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
/* .left-content {
    flex-wrap: wrap;
    -webkit-box-pack: center;
    -ms-flex-pack: center;
    justify-content: center;
    align-content: flex-start;
    display: flex;
} */
.left-content >>> .anticon {
    font-size: 24px;
}
.main-content {
    height: calc(100vh - 12px);
    background: black;
}
</style>
