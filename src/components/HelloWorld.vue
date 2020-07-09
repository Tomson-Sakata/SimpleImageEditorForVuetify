<template>
    <div>
        <v-container>
            <v-row>
                <v-col>
                    <v-text-field v-model="canvas.width" />                        
                </v-col>
                <v-col>
                    <v-text-field v-model="canvas.height" />                        
                </v-col>
                <v-col>
                    <v-btn @click="onClickCreate">作成</v-btn>
                </v-col>
            </v-row>
            <v-row>
                <v-col>
                    <img :src="testImage1" />
                    <v-btn @click="onClickEdit(testImage1)">編集</v-btn>
                </v-col>
            </v-row>
            <v-row>
                <v-col>
                    <img :src="testImage2" />
                    <v-btn @click="onClickEdit(testImage2)">編集</v-btn>
                </v-col>
            </v-row>
        </v-container>
        <v-dialog v-model="isViewEditor" eager scrollable>
            <v-card>
                <v-card-text class="pa-1">
                    <Editor ref="editor" />
                </v-card-text>
            </v-card>
        </v-dialog>
    </div>
</template>

<script>
    import Editor from '@/components/ImageEditor';
    export default {
        components: {
            Editor
        },
        data: function () {
            return {
                isViewEditor: false,
                // testImage: "static/20200702191048.jpg"
                testImage1: "static/イラスト2.JPG",
                testImage2: "static/icon-72x72.png",
                canvas: {
                    width: 640,
                    height: 480,
                    color: {r:255, g:255, b:255}
                }
            }
        },
        mounted: function () {
                console.log(this.$refs.editor)
        },
        methods: {
            onClickEdit: function (image) {
                const selt = this
                this.isViewEditor = true
                console.log(this.$refs.editor)
                this.$refs.editor.show(
                    {
                        createNewLayerWhenStart: true,
                        baseImage: image
                    }
                )
            },
            onClickCreate: function () {
                const selt = this
                this.isViewEditor = true
                console.log(this.$refs.editor)
                this.$refs.editor.show(
                    {
                        canvas: this.canvas,
                    }
                )
            }
        }
    }
</script>