<template>
<!-- 
xs(Extra Small)・・・主にスマホ対応
sm(Small)・・・主にタブレット
md(Medium) ・・・主にノートパソコンや横表示のタブレット
lg(Large)・・・主にデスクトップ
xl(Extra large)・・・4Kや大きなモニター 


v-if="$vuetify.breakpoint.xs"
-->
    <div>
        <v-toolbar dense elevation="0" class="mb-1">
            <template v-if="!params.readOnly && editTargetLayer != null && editTargetLayer.params.canEdit">
                <!-- システム関連のボタン -->
                <v-sheet v-for="button in systemButtons" :key="button.name" outlined rounded>
                    <v-btn icon @click="onClickSystemButton(button)" small>
                        <v-icon>{{button.icon}}</v-icon>
                    </v-btn>
                </v-sheet>
                <!-- 描画ツール関連のボタン -->
                <v-divider vertical class="ml-2 mr-2" />
                <template v-if="!$vuetify.breakpoint.xs">
                    <v-icon>mdi-drawing-box</v-icon>
                    <v-btn-toggle
                        v-model="activeToolButton"
                        mandatory>
                        <v-btn v-for="button in toolButtons" :key="button.name" icon :value="button.name" small>
                            <v-icon>{{button.icon}}</v-icon>
                        </v-btn>
                    </v-btn-toggle>
                </template>
                <template v-else>
                    <v-menu 
                        transition="slide-y-transition"
                        :close-on-content-click="true" 
                        offset-y>
                        <template v-slot:activator="{ on }">
                            <v-icon v-on="on" >{{activeToolButtonIcon}}</v-icon>
                        </template>
                        <v-list dense class="pa-0">
                            <v-sheet class="grey lighten-2 pa-1" width="100%">
                                <v-icon>mdi-drawing-box</v-icon><span class="body-2">ツール</span>
                            </v-sheet>
                            <v-list-item v-for="button in toolButtons" :key="button.name" @click="activeToolButton = button.name">
                                <v-icon>{{button.icon}}</v-icon>
                            </v-list-item>
                        </v-list>
                    </v-menu>
                </template>
                <!-- 描画ツールのオプション系 -->
                <v-divider vertical class="ml-2 mr-2" />
                <v-card v-if="activeToolButton != 'text'" flat outlined class="mr-2">
                    <v-card-text class="pa-0">
                        <template v-if="!$vuetify.breakpoint.xs">
                            <v-slider
                                style="width:200px"
                                prepend-icon="mdi-format-line-weight"
                                v-model="toolParams.lineWidth" 
                                :min="toolParams.lineWidthMin" 
                                :max="toolParams.lineWidthMax"
                                hide-details
                                :label="''+toolParams.lineWidth"
                                />
                        </template>
                        <template v-else>
                            <v-menu 
                                transition="slide-y-transition"
                                :close-on-content-click="true" 
                                offset-y>
                                <template v-slot:activator="{ on }">
                                    <v-icon v-on="on" >mdi-format-line-weight</v-icon>
                                </template>
                                <v-list dense class="pa-0">
                                    <v-sheet class="grey lighten-2 pa-1" width="100%">
                                        <v-icon>mdi-format-line-weight</v-icon><span class="body-2">先の太さ</span>
                                    </v-sheet>
                                    <v-list-item dense>
                                        <v-slider
                                            style="width:200px"
                                            v-model="toolParams.lineWidth" 
                                            :min="toolParams.lineWidthMin" 
                                            :max="toolParams.lineWidthMax"
                                            hide-details
                                            dense
                                            :label="''+toolParams.lineWidth"
                                            />
                                    </v-list-item>
                                </v-list>
                            </v-menu>
                        </template>
                    </v-card-text>
                </v-card>
                <div v-if="activeToolButton == 'text'" style="width:150px" class="mr-2">
                    <v-card-text class="pa-0">
                        <v-select 
                            v-model="toolParams.fontSize" 
                            :items="fontSizes" 
                            hide-details 
                            style="width:130px" 
                            outlined 
                            dense 
                            suffix="pt"
                            prepend-icon="mdi-format-size">
                        </v-select>
                    </v-card-text>
                </div>
                <v-menu 
                    v-if="activeToolButton != 'erase'"
                    transition="slide-y-transition"
                    :close-on-content-click="false"
                    offset-y>
                    <template v-slot:activator="{ on }">
                        <div 
                            v-on="on" 
                            :style="toolColorStyle" ></div>
                    </template>
                    <v-list dense class="pa-0">
                        <v-sheet class="grey lighten-2 pa-1 mb-1" width="100%">
                            <v-icon>mdi-palette</v-icon><span class="body-2">色選択</span>
                        </v-sheet>
                        <v-list-item>
                            <v-color-picker 
                                v-model="toolParams.lineColor"
                                mode="rgba"
                                hide-inputs
                                width="200" />
                        </v-list-item>
                    </v-list>
                </v-menu>
            </template>
            <template v-else>
                <v-chip color="warning" class="caption">
                    <span v-if="params.readOnly">編集不可</span>
                    <span v-else-if="layers.length == 0">編集レイヤー無し</span>
                    <span v-else-if="editTargetLayer != null && !editTargetLayer.params.canEdit">このレイヤーは編集不可</span>
                </v-chip>
            </template>
            <!-- レイヤー操作 -->
            <v-divider vertical class="ml-2 mr-2" />
            <v-menu
                min-width="300"
                transition="slide-y-transition"
                :close-on-content-click="false"
                offset-y >
                <template v-slot:activator="{ on }">
                    <v-card v-if="editTargetLayer != null" flat outlined class="mr-2" v-on="on">
                        <v-card-text class="pa-1">
                            <v-icon>mdi-layers-outline</v-icon>
                            <span v-if="!$vuetify.breakpoint.xs" class="caption text-no-wrap">{{editTargetLayer.params.name}}</span>
                        </v-card-text>
                    </v-card>
                    <v-card v-else flat outlined class="mr-2" v-on="on">
                        <v-card-text class="pa-1">
                            <v-icon>mdi-layers-outline</v-icon>
                            no layer
                        </v-card-text>
                    </v-card>
                </template>
                <v-list dense class="pa-0">
                    <!-- <v-sheet class="grey lighten-2 pa-1" width="100%">
                        <v-icon>mdi-layers-outline</v-icon><span class="body-2">レイヤー</span>
                    </v-sheet> -->
                    <v-toolbar class="grey lighten-2 pa-0 ma-0" elevation="0" height="32">
                        <v-icon>mdi-layers-outline</v-icon><span class="body-2">レイヤー</span>
                        <v-spacer />
                        <v-btn v-if="!params.readOnly && params.canAddLayer" icon @click="onClickLayerButton('add')" small><v-icon>mdi-plus-circle-outline</v-icon></v-btn>
                    </v-toolbar>
                    <!-- <v-list-item>
                        <v-toolbar height="32">
                            <v-btn v-if="!params.readOnly && params.canAddLayer" icon @click="onClickLayerButton('add')" small><v-icon>mdi-plus-circle-outline</v-icon></v-btn>
                        </v-toolbar>
                    </v-list-item> -->
                    <span v-if="layers.length == 0" class="caption ma-2">
                        <v-icon>mdi-plus-circle-outline</v-icon>でレイヤーを作成出来ます
                    </span>
                    <v-list-item v-for="(layer, index) in layers" :key="index" dense style="border-bottom:solid 1px #dddddd">
                        <v-list-item-content>
                            <v-text-field
                                :prepend-icon="layer == editTargetLayer ? 'mdi-check-circle-outline' : 'mdi-checkbox-blank-circle-outline'"
                                v-model="layer.params.name"
                                hide-details
                                outlined
                                dense
                                single-line                                
                                :background-color="layerSelectorColor(layer)"
                                @click:prepend="editTargetLayer = layer" />
                        </v-list-item-content>
                        <v-btn icon @click="onClickLayerVisible(layer)">
                            <v-icon v-if="layer.params.visible">mdi-eye</v-icon>
                            <v-icon v-else>mdi-eye-off</v-icon>
                        </v-btn>
                        <v-btn icon @click="onClickLayerDelete(layer)">
                            <v-icon v-if="!params.readOnly && layer.params.canDelete" >mdi-delete</v-icon>
                            <v-icon v-else >mdi-delete-off</v-icon>
                        </v-btn>
                    </v-list-item>
                </v-list>
            </v-menu>
            <v-btn v-if="!$vuetify.breakpoint.xs && !params.readOnly && params.canAddLayer" icon @click="onClickLayerButton('add')" small><v-icon>mdi-plus-circle-outline</v-icon></v-btn>
            <!-- 表示 -->
            <template v-if="zooming.enable">
                <v-divider vertical class="ml-2 mr-2" />
                <template v-if="!$vuetify.breakpoint.xs && !$vuetify.breakpoint.sm">
                    <v-slider
                        style="width:200px"
                        prepend-icon="mdi-magnify-minus-outline"
                        append-icon="mdi-magnify-plus-outline"
                        v-model="zooming.zoom" 
                        min="0"
                        max="1"
                        step="0.01"
                        hide-details                        
                        @change="onChangeZoom" />
                </template>
                <template v-else>
                    <v-menu 
                        transition="slide-y-transition"
                        :close-on-content-click="true" 
                        offset-y>
                        <template v-slot:activator="{ on }">
                            <v-icon v-on="on">mdi-magnify</v-icon>
                        </template>
                        <v-list dense class="pa-0">
                            <v-sheet class="grey lighten-2 pa-1" width="100%">
                                <v-icon>mdi-magnify</v-icon><span class="body-2">表示</span>
                            </v-sheet>
                            <v-list-item>
                                <v-slider
                                    style="width:200px"
                                    prepend-icon="mdi-magnify-minus-outline"
                                    append-icon="mdi-magnify-plus-outline"
                                    v-model="zooming.zoom" 
                                    min="0"
                                    max="1"
                                    step="0.01"
                                    hide-details
                                    @change="onChangeZoom" />
                            </v-list-item>
                        </v-list>
                    </v-menu>
                </template>
            </template>
        </v-toolbar>
        <v-divider />
        <v-container fluid class="pt-1">
            <v-row>
                <v-col ref="parent" style="position:relative;" class="pa-0 grey">
                    <!-- 下地 -->
                    <canvas
                        ref="baseCanvas" 
                        style="top:0px; left:0px; width:100%; height:100%; z-index:0" />
                    <!-- 全レイヤーのキャンバスを格納 -->
                    <div ref="layers" />
                    <!-- 作業レイヤーのキャンバスを格納 -->
                    <div ref="tempCanvas" />
                    <!-- テキスト入力 -->
                    <div v-if="editText.isEditting" :style="editTextBoxStyle">
                        <v-textarea
                            ref="editText"
                            v-model="editText.text"
                            background-color="white"
                            hide-details
                            outlined
                            dense
                            autofocus
                            auto-grow
                            clearable
                            append-icon="mdi-check-circle-outline"
                            @click:append="onClickEditTextSet"
                            @click:clear="onClickEditTextClear" />
                    </div>
                </v-col>
            </v-row>
        </v-container>
    </div>
</template>

<script>
    class Point {
        constructor (x, y) {
            this.x = x
            this.y = y
        }
    }
    export default {
        data: function () {
            return {
                params: {
                    readOnly: false,
                    canAddLayer: true,
                    createNewLayerWhenStart: false
                },
                systemButtons: [
                    {name: "save", icon:"mdi-content-save"},
                ],
                toolButtons: [
                    {name: "free", icon:"mdi-gesture"},
                    {name: "line", icon:"mdi-chart-line-variant"},
                    {name: "box", icon:"mdi-rectangle-outline"},
                    {name: "circle", icon:"mdi-checkbox-blank-circle-outline"},
                    {name: "erase", icon:"mdi-eraser"},
                    {name: "text", icon:"mdi-format-color-text"},
                ],
                layerButtons: [
                    {name: "add", icon:"mdi-plus-circle-outline"},
                ],
                activeToolButton: "free",
                baseCanvas: null,                   // 下地(初めに画像を描画した後は何もしないので、コンテキストは不要)                
                layers: [],                         // レイヤー
                editTargetLayer: null,              // 編集対象のレイヤー
                tempLayer: {                        // 作業用のレイヤー
                    canvas: null,
                    context: null
                },
                state: "idle",
                clickPoint: null,                   // タッチ開始された座標
                lastPoint: null,                    // 最終座標
                scaling: 1,                         // スケーリング
                zooming: {
                    enable: true,
                    zoom: 0,
                    minRate: 10,
                    maxRate: 100,
                    minWidth: 0,
                    maxWidth: 100,
                    minHeight: 0,
                    maxHeight: 100,
                    minScaling: 0,
                    maxScaling: 1
                },
                editText: {
                    isEditting: false,
                    point: null,
                    text: ""
                },
                toolParams: {
                    lineWidthMin: 1,
                    lineWidthMax: 100,
                    lineWidth: 1,
                    lineColor: {
                        r:0, g:0, b:0, a:1
                    },
                    fontSize: 14
                },
                fontSizes: [],
            }
        },
        watch: {
            activeToolButton: function (value, old) {
                if (value == "text") {
                } else if (old == "text") {
                    this.state = "idle"
                }
            },
        },
        computed: {
            activeToolButtonIcon: function () {
                return this.toolButtons[this.toolButtons.findIndex((v)=>v.name == this.activeToolButton)].icon
            },
            toolColorStyle: function () {
                return {
                    background: "rgba("+this.toolParams.lineColor.r+","+this.toolParams.lineColor.g+","+this.toolParams.lineColor.b+","+this.toolParams.lineColor.a+")",
                    cursor: "pointer",
                    height: "24px",
                    width: "24px",
                    borderRadius: "4px"
                }
            },
            editTextBoxStyle: function () {
                return {
                    position: "absolute",
                    top: this.editText.point.viewPoint.y + "px",
                    left: this.editText.point.viewPoint.x + "px",
                    zIndex: 9999
                }
            },
        },
        created: function () {
            for (var s=1; s<=50; s++) {
                this.fontSizes.push(s)
            }
        },
        mounted: function () {
            this.baseCanvas = this.$refs.baseCanvas
        },
        methods: {
            layerSelectorColor: function (layer) {
                if (layer == this.editTargetLayer) {
                    return "teal accent-1"
                } else {
                    return "white"
                }
            },
            onClickLayerVisible: function (layer) {
                if (layer.params.visible) {
                    this.hideLayer(layer)
                } else {
                    layer.canvas.style.width = layer.originalWidth
                    layer.params.visible = true
                }
            },
            onClickLayerDelete: function (targetLayer) {
                this.deleteLayer(targetLayer)
            },
            onMouseDown: function (e) {
                if (this.editTargetLayer == null || ( this.state != "idle" && this.activeToolButton != "text" )) {
                    return
                }
                console.log("onMouseDown", e)
                this.mouseDown(
                    this.editTargetLayer,
                    this.getPoint(this.editTargetLayer, e.clientX, e.clientY),
                    this.getViewPoint(this.editTargetLayer, e.clientX, e.clientY)
                )
            },
            onTouchStart: function (e) {
                if (this.editTargetLayer == null || ( this.state != "idle" && this.activeToolButton != "text" )) {
                    return
                }
                console.log("onTouchStart", e)
                if (e.changedTouches.length == 1) {
                    e.preventDefault()
                    this.mouseDown(
                        this.editTargetLayer,
                        this.getPoint(this.editTargetLayer, e.changedTouches[0].clientX, e.changedTouches[0].clientY),
                        this.getViewPoint(this.editTargetLayer, e.changedTouches[0].clientX, e.changedTouches[0].clientY)
                    )
                }
            },
            onMouseMove: function (e) {
                if (this.state == "idle") {
                    return
                }
                // console.log(e)
                this.mouseMove(
                    this.editTargetLayer,
                    this.getPoint(this.editTargetLayer, e.clientX, e.clientY),
                    this.getViewPoint(this.editTargetLayer, e.clientX, e.clientY),
                    e.shiftKey
                )
            },
            onTouchMove: function (e) {
                if (this.state == "idle") {
                    return
                }
                // console.log("onTouchMove", e)
                this.mouseMove(
                    this.editTargetLayer,
                    this.getPoint(this.editTargetLayer, e.changedTouches[0].clientX, e.changedTouches[0].clientY),
                    this.getViewPoint(this.editTargetLayer, e.changedTouches[0].clientX, e.changedTouches[0].clientY,
                    e.shiftKey)
                )
            },
            onMouseUp: function (e) {
                if (this.state == "idle") {
                    return
                }
                this.mouseUp(
                    this.editTargetLayer,
                    this.getPoint(this.editTargetLayer, e.clientX, e.clientY),
                    this.getViewPoint(this.editTargetLayer, e.clientX, e.clientY)
                )
            },
            onTouchEnd: function (e) {
                if (this.state == "idle") {
                    return
                }
                this.mouseUp(
                    this.editTargetLayer,
                    this.getPoint(this.editTargetLayer, e.changedTouches[0].clientX, e.changedTouches[0].clientY),
                    this.getViewPoint(this.editTargetLayer, e.changedTouches[0].clientX, e.changedTouches[0].clientY)
                )
            },
            onClickSystemButton: function (button) {
                switch (button.name) {
                    case "save":
                        this.save()
                        break;
                }
            },
            onClickLayerButton: function (name) {
                switch (name) {
                    case "add":
                        this.addLayer()
                        break;
                }
            },
            onClickEditTextSet: function () {
                this.drawText(this.tempLayer)
                this.editText.isEditting = false
                this.endDraw(this.editTargetLayer)
            },
            onClickEditTextClear: function () {
                this.editText.isEditting = false
            },
            onChangeZoom: function () {
                console.log(this.zooming.zoom)
                const self = this
                const width = (this.zooming.minWidth + ((this.zooming.maxWidth - this.zooming.minWidth) * this.zooming.zoom))
                const height = (this.zooming.minHeight + ((this.zooming.maxHeight - this.zooming.minHeight) * this.zooming.zoom))
                const scaling = (this.zooming.minScaling + ((this.zooming.maxScaling - this.zooming.minScaling) * this.zooming.zoom))
                this.baseCanvas.style.width = width + "px"
                this.baseCanvas.style.height = height + "px"
                this.scaling = scaling
                this.setCanvasStyle(this.tempLayer.canvas)
                this.layers.forEach(function (layer){
                    self.setCanvasStyle(layer.canvas)
                })
            },
            mouseDown: function (layer, point, viewPoint) {
                this.clickPoint = point
                this.startDraw(layer)
                switch (this.activeToolButton) {
                    case "free":
                    case "line":
                    case "box":
                    case "circle":
                    case "erase":
                        this.lastPoint = point
                        break;
                    case "text":
                        if (this.editText.isEditting == true) {
                            return
                        }
                        console.log("text input start", viewPoint)
                        this.editText.text = ""
                        this.editText.isEditting = true
                        this.editText.point = {point: point, viewPoint:viewPoint}
                        break;
                }
            },
            mouseUp: function (layer, point, viewPoint) {
                switch (this.activeToolButton) {
                    case "text":
                        return
                }
                this.endDraw(this.editTargetLayer)
            },
            mouseMove: function (layer, point, viewPoint, isShiftKeyDowned) {
                // console.log(isShiftKeyDowned)
                switch (this.activeToolButton) {
                    case "free":
                        layer.context.beginPath()
                        layer.context.lineWidth = this.toolParams.lineWidth // this.scaling
                        layer.context.lineCap = "round"
                        layer.context.moveTo(this.lastPoint.x, this.lastPoint.y)
                        layer.context.lineTo(point.x, point.y)
                        layer.context.strokeStyle = this.getColor(this.toolParams.lineColor)
                        layer.context.stroke()
                        break;
                    case "line":
                        this.tempLayer.context.beginPath()
                        this.tempLayer.context.clearRect(0, 0, this.tempLayer.canvas.width, this.tempLayer.canvas.height)
                        this.tempLayer.context.lineWidth = this.toolParams.lineWidth // this.scaling
                        this.tempLayer.context.lineCap = "round"
                        this.tempLayer.context.moveTo(this.clickPoint.x, this.clickPoint.y)
                        this.tempLayer.context.lineTo(point.x, point.y)
                        this.tempLayer.context.strokeStyle = this.getColor(this.toolParams.lineColor)
                        this.tempLayer.context.stroke()
                        break;
                    case "box":
                        this.tempLayer.context.beginPath()
                        this.tempLayer.context.clearRect(0, 0, this.tempLayer.canvas.width, this.tempLayer.canvas.height)
                        this.tempLayer.context.lineWidth = this.toolParams.lineWidth // this.scaling
                        this.tempLayer.context.lineCap = "round"
                        if (!isShiftKeyDowned) {
                            const width = point.x - this.clickPoint.x
                            const height = point.y - this.clickPoint.y
                            this.tempLayer.context.rect(this.clickPoint.x, this.clickPoint.y, width, height)
                        } else {
                            const width = point.x - this.clickPoint.x
                            this.tempLayer.context.rect(this.clickPoint.x, this.clickPoint.y, width, width)
                        }
                        this.tempLayer.context.strokeStyle = this.getColor(this.toolParams.lineColor)
                        this.tempLayer.context.stroke()
                        break;
                    case "circle":
                        this.tempLayer.context.beginPath()
                        this.tempLayer.context.clearRect(0, 0, this.tempLayer.canvas.width, this.tempLayer.canvas.height)
                        this.tempLayer.context.lineWidth = this.toolParams.lineWidth // this.scaling
                        this.tempLayer.context.lineCap = "round"
                        if (!isShiftKeyDowned) {
                            const radiusX =  Math.abs(this.clickPoint.x - point.x) // this.scaling
                            const radiusY =  Math.abs(this.clickPoint.y - point.y) // this.scaling
                            this.tempLayer.context.ellipse(this.clickPoint.x, this.clickPoint.y, radiusX, radiusY, 0, Math.PI*2, false)
                        } else {
                            const radius =  Math.abs(this.clickPoint.x - point.x) // this.scaling
                            // const radius =  
                            //     Math.abs(
                            //         Math.sqrt(
                            //             (this.clickPoint.x - point.x) * (this.clickPoint.x - point.x) + 
                            //             (this.clickPoint.y - point.y) * (this.clickPoint.y - point.y)
                            //         )
                            //     )
                            this.tempLayer.context.arc(this.clickPoint.x, this.clickPoint.y, radius, 0, Math.PI*2, false)
                        }
                        this.tempLayer.context.strokeStyle = this.getColor(this.toolParams.lineColor)
                        this.tempLayer.context.stroke()
                        break;
                    case "erase":
                        layer.context.globalCompositeOperation = "destination-out"
                        try {
                            layer.context.beginPath()
                            layer.context.lineWidth = this.toolParams.lineWidth // this.scaling
                            layer.context.lineCap = "round"
                            layer.context.moveTo(this.lastPoint.x, this.lastPoint.y)
                            layer.context.lineTo(point.x, point.y)
                            layer.context.strokeStyle = this.getColor(this.toolParams.lineColor)
                            layer.context.stroke()
                        } finally {
                            layer.context.globalCompositeOperation = "source-over"
                        }
                        break;
                }
                this.lastPoint = point
            },
            startDraw: function (layer) {
                this.tempLayer.context.clearRect(0, 0, this.tempLayer.canvas.width, this.tempLayer.canvas.height)
                this.tempLayer.canvas.style.zIndex = (layer.canvas.style.zIndex * 1) + 1
                console.log("zindex", layer.canvas.style.zIndex, this.tempLayer.canvas.style.zIndex)
                this.state = "drawing"
            },
            endDraw: function (layer) {
                this.state = "idle"
                layer.context.drawImage(
                    this.tempLayer.canvas,
                    0,
                    0)
                this.tempLayer.context.clearRect(0, 0, this.tempLayer.canvas.width, this.tempLayer.canvas.height)
                console.log("copied edit canvas to post canvas")
            },
            deleteLayer: function (layer) {
                console.log(layer)
                layer.canvas.remove()
                const isEdittingLayer = (layer == this.editTargetLayer)
                const edittingLayerIndex = this.layers.findIndex((v)=>v.no == this.editTargetLayer.no)
                const index = this.layers.findIndex((v)=>v.no == layer.no)
                this.layers.splice(index, 1)
                if (isEdittingLayer) {
                    if (this.layers.length == 0) {
                        this.editTargetLayer = null
                    } else {
                        if (edittingLayerIndex == 0) {
                            this.editTargetLayer = this.layers[0]
                        } else {
                            this.editTargetLayer = this.layers[edittingLayerIndex - 1]
                        }
                    }
                }
            },
            drawText: function (layer) {

                const self = this
                const fontSize = this.toolParams.fontSize   // (this.toolParams.fontSize / this.scaling)

                layer.context.font = fontSize + "px sans-self"
                layer.context.fillStyle = this.getColor(this.toolParams.lineColor)

                const texts = this.editText.text.split("\n")
                console.log(texts)
                const rowHeight = fontSize

                texts.forEach(function (text, i){
                    layer.context.fillText(text, self.editText.point.point.x, self.editText.point.point.y + rowHeight * i)
                })

            },
            copyLayer: function (sourceLayer, destLayer) {
                const image = sourceLayer.context.getImageData(0, 0, sourceLayer.canvas.width, sourceLayer.canvas.height)
                destLayer.context.putImageData(image, 0, 0)
            },
            hideLayer: function (layer) {
                layer.originalWidth = layer.canvas.style.width
                layer.canvas.style.width = "0%"
                layer.params.visible = false
            },
            save: function () {
                this.$emit(
                    "onClickSaveImage",
                    {
                        baseCanvas: this.baseCanvas,
                        layers: this.layers
                    }
                )
            },
            getColor: function (rgba) {
                return "rgba("+this.toolParams.lineColor.r+","+this.toolParams.lineColor.g+","+this.toolParams.lineColor.b+","+this.toolParams.lineColor.a+")"
            },
            getPoint: function (layer, x, y) {
                const vx = x - layer.canvas.getBoundingClientRect().left
                const vy = y - layer.canvas.getBoundingClientRect().top
                // const cx = vx / ( this.editCanvas.clientWidth / this.editCanvas.width )
                // const cy = vy / ( this.editCanvas.clientHeight / this.editCanvas.height )
                const cx = vx / this.scaling
                const cy = vy / this.scaling

                return new Point(cx, cy)
            },
            getViewPoint: function (layer, x, y) {
                const vx = x - layer.canvas.getBoundingClientRect().left
                const vy = y - layer.canvas.getBoundingClientRect().top
                return new Point(vx, vy)
            },
            // params: {
            //     srcImage: url / base64,
            //     name: name,
            //     visible: boolean,
            //     canEdit: boolean,
            //     canDelete: boolean
            // }
            addLayer: function (layerParams) {

                if (layerParams == null) {
                    layerParams = {
                        srcImage: null,
                        name: null,
                        visible: null,
                        canEdit: null,
                        canDelete: null
                    }
                }

                if (layerParams.name == null) layerParams.name = "layer(" + (this.layers.length + 1)+")"
                if (layerParams.visible == null) layerParams.visible = true
                if (layerParams.canEdit == null) layerParams.canEdit = true
                if (layerParams.canDelete == null) layerParams.canDelete = true

                const canvas = this.createCanvas()

                this.$refs.layers.appendChild(canvas)

                const context = canvas.getContext("2d")

                const layer = {
                    no: this.layers.length + 1,
                    canvas: canvas,
                    context: context,
                    params: layerParams
                }

                this.layers.splice(0, 0, layer)

                var index = this.layers.length * 2 - 1

                this.layers.forEach(function(layer){
                    layer.canvas.style.zIndex = index
                    index-=2
                })

                this.editTargetLayer = layer

                if (layerParams.srcImage != null) {
                    const layerImage = new Image()
                    layerImage.src = layerParams.srcImage
                    layerImage.onload = function () {
                        layer.context.drawImage(layerImage, 0, 0, layerImage.naturalWidth, layerImage.naturalHeight)
                    }
                }

                if (!layer.params.visible) {
                    this.hideLayer(layer)
                }

                console.log("added new layer", layer)
            },
            createCanvas: function () {
                const canvas = document.createElement("canvas")
                canvas.width = this.baseCanvas.width
                canvas.height = this.baseCanvas.height
                canvas.addEventListener("mousedown", this.onMouseDown)
                canvas.addEventListener("touchstart", this.onTouchStart)
                canvas.addEventListener("mousemove", this.onMouseMove)
                canvas.addEventListener("touchmove", this.onTouchMove)
                canvas.addEventListener("mouseup", this.onMouseUp)
                canvas.addEventListener("touchend", this.onTouchEnd)
                this.setCanvasStyle(canvas)
                return canvas
            },
            setCanvasStyle: function (canvas) {
                canvas.style.position = "absolute"
                canvas.style.top = "0px"
                canvas.style.left = "0px"
                canvas.style.width = this.baseCanvas.style.width
                canvas.style.height = this.baseCanvas.style.height
            },
            // params = {
            //     baseImage: "",
            //     layers: [],
            //     readOnly: false,
            //     canAddLayer: false,
            //     createNewLayerWhenStart: true
            // }
            show: function (params) {
                console.log("show", params)

                const self = this

                if (params.readOnly != null) {
                    this.params.readOnly = params.readOnly
                } else {
                    this.params.readOnly = false
                }

                if (params.canAddLayer != null) {
                    this.params.canAddLayer = params.canAddLayer
                } else  {
                    this.params.canAddLayer = true
                }

                if (params.createNewLayerWhenStart != null) {
                    this.params.createNewLayerWhenStart = params.createNewLayerWhenStart
                } else  {
                    this.params.createNewLayerWhenStart = false
                }

                this.layers = []
                this.baseCanvas.style.width = "100%"
                this.baseCanvas.style.height = "100%"
                this.$refs.layers.innerHTML = ""
                this.$refs.tempCanvas.innerHTML = ""

                this.editTargetLayer = null
 
                const initializer = function(canvasWidth, canvasHeight, color, baseImage){

                    console.log("initializer", canvasWidth, canvasHeight, baseImage)

                    self.baseCanvas.width = canvasWidth
                    self.baseCanvas.height = canvasHeight

                    const canvasContext = self.baseCanvas.getContext("2d")
                    if (baseImage != null) {
                        canvasContext.drawImage(baseImage, 0, 0, canvasWidth, canvasHeight)
                    } else if (color != null) {
                        canvasContext.fillStyle = "rgb("+color.r+","+color.g+","+color.b+")"
                        canvasContext.fillRect(0, 0, canvasWidth, canvasHeight)
                    }

                    const rateW = self.baseCanvas.width / self.baseCanvas.clientWidth 
                    const rateH = self.baseCanvas.height / self.baseCanvas.clientHeight

                    console.log(self.baseCanvas.clientWidth, self.baseCanvas.clientHeight, self.baseCanvas.width, self.baseCanvas.height, rateW, rateH)

                    if (rateW <= 1) { // || rateH < 1) {
                        self.zooming.minWidth = canvasWidth
                        self.zooming.minHeight = canvasHeight
                        self.zooming.maxWidth = self.baseCanvas.clientWidth
                        self.zooming.maxHeight = ( self.baseCanvas.clientWidth / canvasWidth ) * canvasHeight
                        self.zooming.minScaling = 1
                        self.zooming.maxScaling = self.baseCanvas.clientWidth / canvasWidth
                        self.zooming.zoom = 0
                        self.zooming.enable = true
                        self.baseCanvas.style.width = canvasWidth + "px"
                        self.baseCanvas.style.height = canvasHeight + "px"
                        self.scaling = 1
                        console.log("zooming", self.zooming)
                    } else {                        
                        self.scaling = self.baseCanvas.clientWidth / self.baseCanvas.width
                        self.zooming.minWidth = self.baseCanvas.clientWidth * 0.1
                        self.zooming.minHeight = self.baseCanvas.clientHeight * 0.1
                        self.zooming.maxWidth = self.baseCanvas.clientWidth
                        self.zooming.maxHeight = self.baseCanvas.clientHeight
                        self.zooming.maxScaling = self.scaling
                        self.zooming.minScaling = self.zooming.minWidth / self.baseCanvas.width
                        self.baseCanvas.style.width = self.baseCanvas.clientWidth + "px"
                        self.baseCanvas.style.height = self.baseCanvas.clientHeight + "px"
                        self.zooming.zoom = 1
                        self.zooming.enable = true
                    }

                    // console.log("scaling", self.scaling)

                    self.tempLayer.canvas = self.createCanvas()
                    self.tempLayer.context = self.tempLayer.canvas.getContext("2d")

                    self.$refs.tempCanvas.appendChild(self.tempLayer.canvas)

                    if (params.layers != null) {
                        for (var i=0; i<params.layers.length; i++) {
                            const layerParams = params.layers[i]

                            self.addLayer(layerParams)
                        }
                    }

                    self.editTargetLayer = self.layers[0]

                    if (!self.params.readOnly && self.params.createNewLayerWhenStart) {
                        self.addLayer()
                    }
                }

                if (params.canvas != null) {
                    setTimeout(function () {
                        initializer(
                            params.canvas.width, 
                            params.canvas.height,
                            params.canvas.color)
                    }, 200)
                } else if (params.baseImage != null) {
                    const image = new Image()
                    image.src = params.baseImage
                    image.onload = function () {
                        console.log("onload")
                        initializer(image.naturalWidth, image.naturalHeight, null, image)
                    }
                } else {
                    setTimeout(function () {
                        initializer(640, 480, {r:255,g:255,b:255})
                    }, 200)
                }

            },
        }
    }
</script>
