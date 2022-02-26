<template>
    <div
        class="canvas-wrapper"
        id="canvas-wrapper"
        @contextmenu.prevent.stop="endDraw"
    ></div>
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
            drawType: "",
            textbox: null,
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
            let canvas = new fabric.Canvas("canvas", {
                selection: false,
                selectable: false,
            });
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
            this.canvas.off("mouse:wheel");
            this.canvas.isDrawingMode = false;
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
                            evented: false,
                        }
                    );
                    break;
                case "freeDraw":
                    break;
                case "dashLine":
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
                            evented: false,
                            strokeDashArray: [3, 1],
                        }
                    );
                    break;
                case "erase":
                    break;
                case "arrow":
                    canvasObject = new fabric.Path(
                        this.drawArrow(
                            this.mouseFrom.x,
                            this.mouseFrom.y,
                            this.mouseTo.x,
                            this.mouseTo.y,
                            30,
                            30
                        ),
                        {
                            fill: "rgba(255,255,255,0)",
                            stroke: this.color,
                            strokeWidth: this.drawWidth,
                            evented: false,
                        }
                    );
                    break;
                case "rectangle":
                    canvasObject = new fabric.Rect({
                        top: this.mouseFrom.y,
                        left: this.mouseFrom.x,
                        width: this.mouseTo.x - this.mouseFrom.x,
                        height: this.mouseTo.y - this.mouseFrom.y,
                        stroke: this.color,
                        strokeWidth: this.drawWidth,
                        // opacity: 0.6,
                        fill: false,
                    });
                    break;
                case "circle":
                    canvasObject = new fabric.Circle({
                        top: this.mouseFrom.y,
                        left: this.mouseFrom.x,
                        originX: "center",
                        originY: "center",
                        radius: Math.sqrt(
                            Math.pow(this.mouseTo.x - this.mouseFrom.x, 2) +
                                Math.pow(this.mouseTo.y - this.mouseFrom.y, 2)
                        ),
                        stroke: this.color,
                        strokeWidth: this.drawWidth,
                        fill: null,
                    });
                    break;
                case "text":
                    let textbox = new fabric.Textbox("", {
                        left: this.mouseFrom.x - 60,
                        top: this.mouseFrom.y - 20,
                        width: 150,
                        fontSize: 18,
                        borderColor: "#2c2c2c",
                        fill: this.color,
                        hasControls: false,
                    });
                    this.canvas.add(textbox);
                    textbox.enterEditing();
                    textbox.hiddenTextarea.focus();
                    this.textbox = textbox;
                    break;
            }
            if (canvasObject) {
                this.canvas.add(canvasObject);
                this.drawingObject = canvasObject;
            }
        },
        download(url) {
            if (!url) {
                return;
            }
            let a = document.createElement("a");
            a.setAttribute("href", url);
            a.download = "未命名.png";
            a.click();
            a.remove();
        },
        startDraw(type) {
            this.endDraw();
            this.drawType = type;

            if (this.drawType === "freeDraw") {
                this.freeDraw();
            } else if (this.drawType === "erase") {
                this.freeEarse();
            } else if (this.drawType === "save") {
                let url = this.canvas.toDataURL();
                this.download(url);
            } else {
                // 开启画布绘制监听
                this.drawListener();
            }
        },
        endDraw() {
            // 关闭绘制
            if (this.textbox) {
                this.textbox.exitEditing();
                this.textbox = null;
            }
            this.closeMouseEvent();
            // 开启放大缩小
            this.initZoom();
            this.initMove();
        },
        drawListener() {
            let doDrawing = false;
            let moveCount = 0;
            this.canvas.skipTargetFind = true; //画板元素不能被选中
            this.canvas.selection = false; //画板不显示选中
            let that = this;
            //绑定画板事件
            this.canvas.on("mouse:down", function (options) {
                var xy = that.canvas.getPointer(options.e);
                that.mouseFrom.x = xy.x;
                that.mouseFrom.y = xy.y;
                doDrawing = true;
            });
            this.canvas.on("mouse:up", function (options) {
                if (!doDrawing) {
                    return;
                }
                var xy = that.canvas.getPointer(options.e);
                that.mouseTo.x = xy.x;
                that.mouseTo.y = xy.y;
                that.drawing();
                that.drawingObject = null;
                moveCount = 1;
                doDrawing = false;
            });
            this.canvas.on("mouse:move", function (options) {
                if (!doDrawing) {
                    return;
                }
                if (moveCount % 2) {
                    //减少绘制频率
                    moveCount++;
                    return;
                }
                moveCount++;
                var xy = that.canvas.getPointer(options.e);
                that.mouseTo.x = xy.x;
                that.mouseTo.y = xy.y;
                that.drawing();
            });
        },
        freeDraw() {
            //  same as `PencilBrush`
            this.canvas.freeDrawingBrush = new fabric.PencilBrush(this.canvas);
            this.canvas.isDrawingMode = true;
            //  optional
            this.canvas.freeDrawingBrush.width = 10;
            //  undo erasing
            this.canvas.freeDrawingBrush.inverted = true;
        },
        freeEarse() {
            // 暂不支持
            // //  same as `PencilBrush`
            // this.canvas.freeDrawingBrush = new fabric.EraserBrush(this.canvas);
            // this.canvas.isDrawingMode = true;
            // //  optional
            // this.canvas.freeDrawingBrush.width = 10;
            // //  undo erasing
            // this.canvas.freeDrawingBrush.inverted = true;
        },
        drawArrow(fromX, fromY, toX, toY, theta, headlen) {
            theta = typeof theta != "undefined" ? theta : 30;
            headlen = typeof theta != "undefined" ? headlen : 10;
            // 计算各角度和对应的P2,P3坐标
            var angle = (Math.atan2(fromY - toY, fromX - toX) * 180) / Math.PI,
                angle1 = ((angle + theta) * Math.PI) / 180,
                angle2 = ((angle - theta) * Math.PI) / 180,
                topX = headlen * Math.cos(angle1),
                topY = headlen * Math.sin(angle1),
                botX = headlen * Math.cos(angle2),
                botY = headlen * Math.sin(angle2);
            var arrowX = fromX - topX,
                arrowY = fromY - topY;
            var path = " M " + fromX + " " + fromY;
            path += " L " + toX + " " + toY;
            arrowX = toX + topX;
            arrowY = toY + topY;
            path += " M " + arrowX + " " + arrowY;
            path += " L " + toX + " " + toY;
            arrowX = toX + botX;
            arrowY = toY + botY;
            path += " L " + arrowX + " " + arrowY;
            return path;
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
    background: #fff;
    border: 1px solid black;
    align-items: center;
}
</style>