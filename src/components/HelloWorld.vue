<template>
    <div>
        <v-btn @click="onClickEdit">編集</v-btn>
        <v-img :src="testImage" aspect-ratio="1" />
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
                testImage: "static/DSC07572.JPG"
            }
        },
        mounted: function () {
                console.log(this.$refs.editor)
        },
        methods: {
            onClickEdit: function () {
                const selt = this
                this.isViewEditor = true
                console.log(this.$refs.editor)
                this.$refs.editor.show({baseImage: this.testImage})
            }
        }
    }
</script>