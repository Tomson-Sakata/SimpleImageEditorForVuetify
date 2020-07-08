# SimpleImageEditorForVuetify
A simple image editor for Vuetify.

# Requirement
* Vue 2.6.11
* Vuetify 2.2.11

# Usage
1.copy src/components/ImageEditor.vue to your project.
2.Embedded in the components of your project.
    <template>
        <Editor ref="editor" @onClickSaveImage="onClickSaveImage" />
    
    <script>
        import Editor from '@/components/ImageEditor';
        export default {
            components: {
                Editor
            },

        (show editor)
        this.$refs.editor.show( params )

        params: {
            readOnly: boolean, (default: false)
            canAddLayer: boolean, (default: true)
            createNewLayerWhenStart: boolean, (default: false)
            baseImage: String (url or Base64)
            layers: [
                {
                    srcImage: String, (url or Base64)
                    visible: Boolean, (default: true)
                    name: String,
                    canDelete: Boolean, (default: true)
                    canEdit: Boolean, (default: true)
                }
            ]

        (event handler)
        onClickSaveImage (params) 

        params: {
            baseCanvas: canvas,
            layers: [
                {
                    no: Number,
                    canvas: Canvas,
                    context: Canvas's context,
                    params: {
                        srcImage: String,
                        visible: Boolean,
                        name: String,
                        canDelete: Boolean,
                        canEdit: Boolean,
                    }
                }
            ]
        }

        *There is no saving process in this component.
        When saving an image, it is implemented by referring to the canvas for each layer in the parameters of a save button click event.

