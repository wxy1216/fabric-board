<template>
    <div class="canvas-wrapper" id="canvas-wrapper"></div>
</template>
<script>
import { fabric } from "fabric";

export default {
    name: "my-canvas",
    props: {
        color: {
            type: String,
            default: "red",
        },
        drawWidth: {
            type: Number,
            default: 2,
        },
    },
    data() {
        return {
            canvas: null,
            mouseTo: { x: "", y: "" },
            mouseFrom: { x: "", y: "" },
            drawingObject: null, // 当前绘制对象
            drawType: "1",
        };
    },
    methods: {
        initCanvas() {
            let wrapper = document.querySelector("#canvas-wrapper");
            let dom = document.createElement("canvas");
            dom.id = "canvas";
            dom.width = wrapper.clientWidth;
            dom.height = wrapper.clientHeight;
            wrapper.appendChild(dom);
            let canvas = new fabric.Canvas("canvas", { selection: false });
            this.canvas = canvas;
        },
        initZoom() {
            this.canvas.on("mouse:wheel", function (opt) {
                let delta = opt.e.deltaY;
                let zoom = this.getZoom();
                zoom *= 0.999 ** delta;
                if (zoom > 20) {
                    zoom = 20;
                }
                if (zoom < 1) {
                    zoom = 1;
                }
                this.zoomToPoint({ x: opt.e.offsetX, y: opt.e.offsetY }, zoom);
                let vpt = this.viewportTransform;
                // 图像往右，往下移动的极限坐标位置
                let maxX = 0;
                let maxY = 0;
                // 图像往左、往上移动的极限坐标位置
                let minX = this.width - this.width * zoom;
                let minY = this.height - this.height * zoom;
                // 放大缩小后的位置
                let moveX = vpt[4];
                let moveY = vpt[5];
                moveX < minX
                    ? (moveX = minX)
                    : moveX > maxX
                    ? (moveX = maxX)
                    : moveX;
                moveY < minY
                    ? (moveY = minY)
                    : moveY > maxY
                    ? (moveY = maxY)
                    : moveY;
                vpt[4] = moveX;
                vpt[5] = moveY;
                this.requestRenderAll();
                opt.e.preventDefault();
                opt.e.stopPropagation();
            });
        },
        initMove() {
            this.canvas.on("mouse:down", function (opt) {
                let evt = opt.e;
                this.isDragging = true;
                this.selection = false;
                this.lastPosX = evt.clientX;
                this.lastPosY = evt.clientY;
            });
            this.canvas.on("mouse:move", function (opt) {
                if (this.isDragging) {
                    let e = opt.e;
                    let vpt = this.viewportTransform;
                    // console.log(vpt);
                    let zoom = this.getZoom();
                    // 鼠标移动计算出的新位置
                    let moveX = vpt[4] + e.clientX - this.lastPosX;
                    let moveY = vpt[5] + e.clientY - this.lastPosY;
                    // 图像往右，往下移动的极限坐标位置
                    let maxX = 0;
                    let maxY = 0;
                    // 图像往左、往上移动的极限坐标位置
                    let minX = this.width - this.width * zoom;
                    let minY = this.height - this.height * zoom;
                    moveX < minX
                        ? (moveX = minX)
                        : moveX > maxX
                        ? (moveX = maxX)
                        : moveX;
                    moveY < minY
                        ? (moveY = minY)
                        : moveY > maxY
                        ? (moveY = maxY)
                        : moveY;

                    vpt[4] = moveX;
                    vpt[5] = moveY;
                    this.requestRenderAll();
                    this.lastPosX = e.clientX;
                    this.lastPosY = e.clientY;
                }
            });
            this.canvas.on("mouse:up", function (opt) {
                // on mouse up we want to recalculate new interaction
                // for all objects, so we call setViewportTransform
                this.setViewportTransform(this.viewportTransform);
                this.isDragging = false;
            });
        },
        closeMouseEvent() {
            this.canvas.off("mouse:down");
            this.canvas.off("mouse:move");
            this.canvas.off("mouse:up");
            this.canvas.off("mouse:over");
            this.canvas.off("mouse:out");
        },
        drawing() {
            if (this.drawingObject) {
                this.canvas.remove(this.drawingObject);
            }
            let canvasObject = null;
            switch (this.drawType) {
                case "line":
                    canvasObject = new fabric.Line(
                        [
                            this.mouseFrom.x,
                            this.mouseFrom.y,
                            this.mouseTo.x,
                            this.mouseTo.y,
                        ],
                        {
                            stroke: this.color,
                            strokeWidth: this.drawWidth,
                        }
                    );
                    break;
            }
            if (canvasObject) {
                this.canvas.add(canvasObject);
                this.drawingObject = canvasObject;
            }
        },
        startDraw(type) {
            this.drawType = type;
            // 关闭画布放大缩小
            this.closeMouseEvent();
            // 开启画布绘制监听
            this.drawListener();
        },
        endDraw() {
            this.initZoom();
            this.initMove();
        },
        drawListener() {
            let doDrawing = false;
            let moveCount = 0;
            let that = this;
            //绑定画板事件
            this.canvas.on("mouse:down", function (options) {
                // var xy = this.transformMouse(
                //     options.e.offsetX,
                //     options.e.offsetY
                // );
                var xy = that.canvas.getPointer(options.e);
                that.mouseFrom.x = xy.x;
                that.mouseFrom.y = xy.y;
                doDrawing = true;
            });
            this.canvas.on("mouse:up", function (options) {
                // var xy = this.transformMouse(
                //     options.e.offsetX,
                //     options.e.offsetY
                // );
                if (!doDrawing) {
                    return;
                }
                var xy = that.canvas.getPointer(options.e);
                that.mouseTo.x = xy.x;
                that.mouseTo.y = xy.y;
                that.drawing();
                debugger;
                that.drawingObject = null;
                moveCount = 1;
                doDrawing = false;
            });
            this.canvas.on("mouse:move", function (options) {
                if (moveCount % 2 && !doDrawing) {
                    //减少绘制频率
                    return;
                }
                debugger;
                moveCount++;
                // var xy = this.transformMouse(
                //     options.e.offsetX,
                //     options.e.offsetY
                // );
                var xy = that.canvas.getPointer(options.e);
                that.mouseTo.x = xy.x;
                that.mouseTo.y = xy.y;
                that.drawing();
            });
        },
        // transformMouse(mouseX, mouseY) {
        //     debugger;
        // },
    },
    mounted() {
        this.initCanvas();
        this.initZoom();
        this.initMove();
    },
};
</script>
<style scoped>
.canvas-wrapper {
    height: 100%;
    width: 100%;
    background: #ccc;
    display: flex;
    align-items: center;
}
</style>