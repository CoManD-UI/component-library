<template>
    <!-- begin edit-mode -->
    <EditComponentWrapper
        v-if="editModeContext"
        ref="editComponentWrapper"
        class="edit-items"
        :showComponentName="false"
        :allowedComponentTypes="[]"
        componentName="CmdImage"
        :componentProps="{image, figcaption}"
        :componentPath="imageComponentPath"
        :allowDeleteComponent="!!imageSource"
        :itemProvider="editModeConfig?.allowAddItem !== false ? itemProvider : null"
    >
        <template v-slot="slotProps">
            <figure :class="['cmd-image flex-container no-gap vertical', textAlign]">
                <!-- begin figcaption above image -->
                <template v-if="figcaption?.show && figcaption?.position === 'top'">
                    <!-- begin CmdFormElement -->
                    <CmdFormElement
                        v-if="slotProps.editing"
                        element="input"
                        type="text"
                        :class="[textAlign, 'edit-mode']"
                        :required="true"
                        labelText="Text figcaption"
                        v-model="editableFigcaptionText"
                    />
                    <!-- end CmdFormElement -->
                    <figcaption v-else-if="figcaption?.text" v-html="figcaption?.text"/>
                </template>
                <!-- end figcaption above image -->

                <!-- begin image-wrapper -->
                <template v-if="slotProps.editing">
                    <!-- begin image with drop-area -->
                    <a href="#" :class="['box drop-area flex-container vertical', { 'allow-drop': allowDrop }]"
                       v-on="dragAndDropHandler"
                       @click.prevent="selectFiles"
                       title="Drag new image to this area to replace old one!">
                        <span class="icon-image"></span>
                        <img :src="imageSource" :alt="image?.alt" :title="image?.tooltip"/>
                    </a>
                    <!-- end image with drop-area -->

                    <!-- begin CmdFormElement -->
                    <CmdFormElement
                        class="hidden"
                        element="input"
                        type="file"
                        labelText="Select new image"
                        :disabled="uploadInitiated"
                        @change="fileSelected"
                        ref="formElement"
                    />
                    <!-- end CmdFormElement -->
                </template>
                <template v-else-if="imageSource">
                    <!-- begin image without drop-area -->
                    <img :src="imageSource" :alt="image?.alt" :title="image?.tooltip"/>
                    <!-- end image without drop-area -->
                </template>
                <!-- end image-wrapper -->

                <!-- begin figcaption below image -->
                <template v-if="figcaption?.show && figcaption?.position !== 'top'">
                    <!-- begin CmdFormElement -->
                    <CmdFormElement
                        v-if="slotProps.editing"
                        element="input"
                        type="text"
                        :class="[textAlign, 'edit-mode']"
                        :required="true"
                        labelText="Text figcaption"
                        :showLabel="false"
                        v-model="editableFigcaptionText"
                        placeholder="figcaption"
                    />
                    <!-- end CmdFormElement -->
                    <figcaption v-else-if="figcaption?.text" v-html="figcaption?.text"/>
                </template>
                <!-- end figcaption below image -->

                <!-- begin show placeholder if no image exists (and component is not edited) -->
                <button v-if="!slotProps.editing && !imageSource"
                        type="button"
                        class="button confirm"
                        @click="onAddItem">
                    <span class="icon-add"></span>
                    <span>Add new image</span>
                </button>
                <!-- end show placeholder if no image exists (and component is not edited) -->
            </figure>
        </template>
    </EditComponentWrapper>
    <!-- end edit-mode -->

    <!-- begin default-view -->
    <!-- begin image with figure/figcaption -->
    <figure v-else-if="useFigureTag" :class="['cmd-image', textAlign]">
        <figcaption v-if="figcaption?.position === 'top' && figcaption?.text && figcaption?.show" v-html="figcaption?.text"/>
        <img :src="imageSource" :alt="image?.alt" :title="image?.tooltip" :loading="image?.lazyLoading !== false ? 'lazy' : null" @load="onImageLoaded"/>
        <figcaption v-if="figcaption?.position !== 'top' && figcaption?.text && figcaption?.show" v-html="figcaption?.text"/>
    </figure>
    <!-- end image with figure/figcaption -->

    <!-- begin image without figure/figcaption -->
    <img v-else class="cmd-image" :src="imageSource" :alt="image?.alt" :title="image?.tooltip" :loading="image?.lazyLoading ? 'lazy' : null" @load="onImageLoaded"/>
    <!-- end image without figure/figcaption -->
    <!-- end default-view -->
</template>
<script>
import {createUuid} from "../utils/common.js"
import {checkAndUploadFile} from "../utils/checkAndUploadFile"
import EditMode from "../mixins/EditMode.vue"
import {updateHandlerProvider} from "../utils/editmode.js"

export default {
    name: "CmdImage",
    mixins: [EditMode],
    data() {
        return {
            mediumMaxWidth: 1023,
            smallMaxWidth: 600,
            currentWindowWidth: window.innerWidth,
            allowedFileExtensions: ["gif", "jpg", "jpeg", "png", "webp", "svg"],
            uploadInitiated: false,
            allowDrop: false,
            showFigcaption: true,
            figcaptionText: null,
            figcaptionPosition: null,
            figcaptionTextAlign: null,
            tooltip: null,
            alternativeText: null,
            newImageSource: null,
            positionOptions: [
                {
                    text: "Above image",
                    value: "top"
                },
                {
                    text: "Below image",
                    value: "bottom"
                }
            ],
            textAlignOptions: [
                {
                    text: "Left",
                    value: "left"
                },
                {
                    text: "Center",
                    value: "center"
                },
                {
                    text: "Right",
                    value: "right"
                }
            ]
        }
    },
    props: {
        /**
         * image-object including source, alternative text, tooltip (not required)
         *
         * {
         *     src: "",
         *     alt: "",
         *     tooltip: ""
         * }
         */
        image: {
            type: Object,
            required: false
        },
        /**
         * activate if image should be wrapped by figure-tag
         */
        useFigureTag: {
            type: Boolean,
            default: true
        },
        /**
         * figcaption-object including visibility, position (top/bottom), text
         */
        figcaption: {
            type: Object,
            required: false
        },
        /**
         * maximum file size (in bytes) for file to upload
         */
        maxFileUploadSize: {
            type: Number,
            default: 500000
        },
        /**
         * minimum image width (in pixels) for file to upload
         */
        minImageWidth: {
            type: Number,
            default: 600
        }
    },
    created() {
        // get current window width on window-resize
        window.addEventListener("resize", this.updateWindowWidth)

        // assign css-breakpoints (as integer to eliminate px-unit) to data-properties to have access to them in vue
        const computedStyle = getComputedStyle(document.documentElement);
        this.mediumMaxWidth = parseInt(computedStyle.getPropertyValue("--medium-max-width"))
        this.smallMaxWidth = parseInt(computedStyle.getPropertyValue("--small-max-width"))
    },
    beforeUnmount() {
        window.removeEventListener("resize", this.updateWindowWidth)
    },
    computed: {
        imageComponentPath() {
            return this.componentPath || ["props", "cmdImage"]
        },
        imageSource() {
            // check if a new image is provided
            if (this.newImageSource) {
                return this.newImageSource
            }

            // if only one src exists
            const imgSrc = this.image?.src

            if (!imgSrc) {
                return null
            }

            if (typeof imgSrc === "string") {
                return imgSrc
            }

            const deviceWidth = this.currentWindowWidth;
            // return image for small-devices (if exists)
            if (imgSrc.small && deviceWidth <= this.smallMaxWidth) {
                return imgSrc.small
            }
            // return image for medium-devices (if exists)
            if (imgSrc.medium && deviceWidth <= this.mediumMaxWidth) {
                return imgSrc.medium
            }
            // else return large (will be used if images for small-and -medium-devices do not exist or if screen resolution is larger than mediumMaxWidth)
            return imgSrc.large
        },
        textAlign() {
            if (this.figcaption?.textAlign) {
                return "text-" + this.figcaption.textAlign
            }
            return ''
        },
        dragAndDropHandler() {
            // register handlers only if drag-and-drop is enabled
            return {
                dragenter: this.dragEnter,
                dragover: this.dragOver,
                dragleave: this.dragLeave,
                drop: this.drop
            }
        },
        editableFigcaptionText: {
            get() {
                return this.figcaptionText == null ? this.figcaption?.text : this.figcaptionText
            },
            set(value) {
                this.figcaptionText = value
            }
        }
    },
    methods: {
        onImageLoaded(event) {
            this.$emit("image-loaded", event)
        },
        itemProvider() {
            const editModeConfig = this.editModeConfig?.itemProviderOverwrite?.()
            return {
                "image": {
                    "src": "/media/images/demo-images/medium/landscape-01.jpg",
                    "alt": "Alternative Text",
                    // add additional keys from editModeConfig
                    ...editModeConfig?.image || {}
                },
                "figcaption": {
                    "text": "Figcaption DE",
                    "position": "bottom",
                    "textAlign": "center",
                    "show": true
                }
            }
        },
        onAddItem() {
            // execute editComponent-function from editComponentWrapper to enter editMode directly on "add"
            this.$refs.editComponentWrapper.editComponent()
        },
        selectFiles() {
            let inputFile = this.$refs.formElement.getDomElement().querySelector("input[type='file']")
            inputFile.click()
        },
        fileSelected(event) {
            if (event.target.files.length > 0) {
                checkAndUploadFile(event.target.files[0], this.allowedFileExtensions, this.minImageWidth, this.maxFileUploadSize, (imageSource) => this.newImageSource = imageSource)
            }
        },
        updateWindowWidth() {
            this.currentWindowWidth = window.innerWidth
        },
        getImage() {
            return {
                image: {...this.image},
                figcaption: {
                    show: this.showFigcaption,
                    position: this.figcaptionPosition,
                    textAlign: this.figcaptionTextAlign,
                    text: this.figcaptionText
                }
            }
        },
        dragEnter(event) {
            this.dragOver(event)
        },
        dragOver(event) {
            if (event.dataTransfer && event.dataTransfer.items && event.dataTransfer.items.length > 0) {
                event.dataTransfer.dropEffect = "none"

                let item = event.dataTransfer.items[0]

                if (item.kind === "file") {
                    event.preventDefault()
                    event.dataTransfer.dropEffect = "copy"
                    this.allowDrop = true
                    // this._handle_dragover(event)
                }
            }
        },
        dragLeave() {
            this.allowDrop = false
        },
        drop(event) {
            this.allowDrop = false
            if (event.dataTransfer && event.dataTransfer.files && event.dataTransfer.files.length > 0) {
                event.preventDefault()
                checkAndUploadFile(event.dataTransfer.files[0], this.allowedFileExtensions, this.minImageWidth, this.maxFileUploadSize, (imageSource) => this.newImageSource = imageSource)
            }
        },
        addHandlerProvider() {
            const itemStructure = {
                "image": {
                    "id": createUuid(),
                    "src": "media/images/demo-images/large/landscape-06.jpg",
                    "srcImageLarge": "media/images/demo-images/large/landscape-06.jpg",
                    "alt": "Alternative Text"
                },
                "figcaption": {
                    "show": true,
                    "position": "bottom",
                    "text": "Figcaption",
                    "textAlign": "center"
                }
            }
            return updateHandlerProvider(this, {
                item() {
                    return itemStructure
                }
            })
        },
        updateHandlerProvider() {
            const figcaptionText = this.figcaptionText
            this.figcaptionText = null // reset data-property

            return updateHandlerProvider(this, {
                update(props) {
                    if (!props.figcaption) {
                        props.figcaption = {}
                    }
                    props.figcaption.text = figcaptionText
                }
            })
        }
    },
    watch: {
        figcaption: {
            handler() {
                this.showFigcaption = this.figcaption?.show
                this.figcaptionText = this.figcaption?.text
                this.figcaptionPosition = this.figcaption?.position
                this.figcaptionTextAlign = this.figcaption?.textAlign
            },
            immediate: true,
            deep: true
        }
    }
}
</script>

<style>
/* begin cmd-image ------------------------------------------------------------------------------------------ */
.cmd-image {
    img {
        display: block;
    }

    figcaption {
        border-bottom-left-radius: var(--default-border-radius);
        border-bottom-right-radius: var(--default-border-radius);
    }

    &.text-center {
        figcaption {
            text-align: center;
        }
    }

    &.text-right {
        figcaption {
            text-align: right;
        }
    }

    .drop-area {
        border: 0;
        align-items: center;
        justify-content: center;
        padding: 0;

        > [class*="icon"] {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translateX(-50%) translateY(-50%);
            font-size: 10rem;
            color: var(--color-white);
            text-shadow: var(--default-text-shadow);
            z-index: 10;
        }

        img {
            opacity: .7;
            transition: var(--default-transition);

            &:hover, :active, :focus {
                opacity: 1;
                transition: var(--default-transition);
            }

            &:not([src]) {
                display: block;
                width: 100%;
                min-height: 30rem;
            }
        }
    }
}

figure.cmd-image:not(:has[src$=".svg"]) {
    display: inline-flex; /* assures figcaption is aligned correctly */
    flex-direction: column;
}

.edit-mode .edit-component-wrapper .cmd-image {
    label.edit-mode input {
        padding: calc(var(--default-padding) / 2);
    }
}

/* end cmd-image ------------------------------------------------------------------------------------------ */
</style>
