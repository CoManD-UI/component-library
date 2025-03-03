<template>
    <!-- begin default-view -->
    <div v-if="!editModeContext || settingsContext || mainSidebarContext || headlineText"
         :class="['cmd-headline', {'has-pre-headline-text': preHeadlineText, 'has-icon': cmdIcon?.iconClass}, headlineTextAlign]">
        <!-- begin headline with pre-headline-text -->
        <template v-if="preHeadlineText">
            <component v-if="headlineText" :is="headlineTag">
                <!-- begin CmdIcon -->
                <CmdIcon v-if="cmdIcon" v-bind="cmdIcon"/>
                <!-- end CmdIcon -->

                <span class="pre-headline-text-wrapper">
                    <!-- begin pre-headline-text -->
                    <span class="pre-headline-text" v-html="preHeadlineText"></span>
                    <!-- end pre-headline-text -->

                    <!-- begin default headline-text without slot -->
                    <span v-if="headlineText" v-html="headlineText"></span>
                    <!-- end default headline-text without slot -->

                    <!-- begin headline-text with slot -->
                    <span v-else>
                        <!-- begin slot -->
                        <slot></slot>
                        <!-- end slot -->
                    </span>
                    <!-- end headline-text with slot -->
                </span>
            </component>
        </template>
        <!-- end headline with pre-headline-text -->

        <!-- begin headline without pre-headline-text -->
        <component v-else :is="headlineTag">
            <!-- begin CmdIcon -->
            <CmdIcon v-if="cmdIcon" v-bind="cmdIcon"/>
            <!-- end CmdIcon -->

            <!-- begin default headline-text without slot -->
            <span v-if="headlineText" v-html="headlineText"></span>
            <!-- end default headline-text without slot -->

            <!-- begin headline-text with slot -->
            <span v-else>
                <!-- begin slot -->
                <slot></slot>
                <!-- end slot -->
            </span>
            <!-- end headline-text with slot -->
        </component>
        <!-- end headline without pre-headline-text -->
    </div>
    <!-- end default-view -->

    <!-- begin edit-mode -->
    <EditComponentWrapper
        v-else
        ref="editComponentWrapper"
        class="edit-items"
        :showComponentName="false"
        :allowedComponentTypes="[]"
        componentName="CmdHeadline"
        :componentProps="{headlineText, headlineLevel}"
        :componentPath="headlineComponentPath"
        :allowDeleteComponent="!!headlineText"
    >
        <template v-slot="slotProps">
            <!-- begin CmdFormElement -->
            <CmdFormElement
                v-if="slotProps.editing"
                element="input"
                type="text"
                :class="['edit-mode', 'headline', 'h'+ headlineLevel, headlineTextAlign]"
                labelText="Headline"
                :showLabel="false"
                placeholder="Headline"
                v-model="editableHeadlineText"
            />
            <!-- end CmdFormElement -->

            <template v-else-if="headlineText"
                      :class="['cmd-headline', {'has-pre-headline-text': preHeadlineText, 'has-icon': cmdIcon?.iconClass}, headlineTextAlign]">
                <!-- begin CmdIcon -->
                <CmdIcon v-if="cmdIcon" v-bind="cmdIcon"/>
                <!-- end CmdIcon -->

                <template v-if="preHeadlineText">
                    <component v-if="headlineText" :is="headlineTag">
                        <!-- begin CmdIcon -->
                        <CmdIcon v-if="cmdIcon" v-bind="cmdIcon"/>
                        <!-- end CmdIcon -->

                        <!-- begin pre-headline-text -->
                        <span class="pre-headline-text">{{ preHeadlineText }}</span>
                        <!-- end pre-headline-text -->

                        <!-- begin slot -->
                        <slot>{{ headlineText }}</slot>
                        <!-- end slot -->
                    </component>
                </template>
                <component v-else-if="headlineText" :is="headlineTag">
                    <!-- begin slot -->
                    <slot>{{ headlineText }}</slot>
                    <!-- end slot -->
                </component>
            </template>

            <!-- begin show placeholder if no image exists (and component is not edited) -->
            <button v-else-if="componentActive" type="button" class="button confirm" @click="onAddItem">
                <span class="icon-plus"></span>
                <span>Add new headline</span>
            </button>
            <!-- end show placeholder if no image exists (and component is not edited) -->
        </template>
    </EditComponentWrapper>
    <!-- end edit-mode -->
</template>

<script>
import EditMode from "../mixins/EditMode.vue"
import {buildComponentPath, findEditComponentWrapper, updateHandlerProvider} from "../utils/editmode.js"

export default {
    name: "CmdHeadline",
    mixins: [EditMode],
    data() {
        return {
            editableHeadlineText: null,
            editablePreHeadlineText: null
        }
    },
    props: {
        /**
         * text for headline
         */
        headlineText: {
            type: String,
            required: false
        },
        /**
         * level for headline
         */
        headlineLevel: {
            type: [String, Number],
            default: "2"
        },
        /**
         * small pre-headline-text above main-headline
         *
         * @canContainHtml: true
         */
        preHeadlineText: {
            type: String,
            required: false
        },
        /**
         * text-alignment (has no effect if icon is used)
         *
         * @allowedValues: "left", "center", "right"
         */
        textAlign: {
            type: String,
            default: null,
            validator(value) {
                return value === "left" ||
                    value === "center" ||
                    value === "right"
            }
        },
        /**
         * properties for CmdIcon-component to show icon left/before headline
         */
        cmdIcon: {
            type: Object,
            required: false
        }
    },
    computed: {
        componentActive() {
            return this.editModeContext?.system.isActiveComponent(buildComponentPath(findEditComponentWrapper(this)))
        },
        headlineComponentPath() {
            return this.componentPath || ["props", "cmdHeadline"]
        },
        headlineTag() {
            if (this.headlineLevel) {
                return "h" + this.headlineLevel
            }
            return ""
        },
        headlineTextAlign() {
            if (this.textAlign) {
                return "text-" + this.textAlign
            }
            return ""
        }
    },
    methods: {
        updateHandlerProvider() {
            const headlineText = this.editableHeadlineText
            const preHeadlineText = this.editablePreHeadlineText
            const headlineLevel = this.headlineLevel

            return updateHandlerProvider(this, {
                update(props) {
                    props.headlineText = headlineText
                    props.preHeadlineText = preHeadlineText
                    props.headlineLevel = headlineLevel
                }
            })
        },
        onAddItem() {
            // execute editComponent-function from editComponentWrapper to enter editMode directly on "add"
            this.$refs.editComponentWrapper.editComponent()
        }
    },
    watch: {
        headlineText: {
            handler() {
                this.editableHeadlineText = this.headlineText
            },
            immediate: true
        },
        preHeadlineText: {
            handler() {
                this.editablePreHeadlineText = this.preHeadlineText
            },
            immediate: true
        }
    }
}
</script>

<style>
/* begin cmd-headline ------------------------------------------------------------------------------------------ */
.cmd-headline {
    margin-bottom: var(--default-margin);
    gap: calc(var(--default-gap) / 2);

    &.text-center > * {
        text-align: center;
    }

    &.text-right > * {
        text-align: right;
    }

    &.has-pre-headline-text {
        text-align: inherit;

        > *:first-child {
            display: flex;
            align-items: flex-start;

            span[class*="icon"] {
                bottom: .2rem;
            }
        }

        &.text-center {
            > *:first-child {
                justify-content: center;
            }
        }

        &.text-right {
            > *:first-child {
                justify-content: flex-end;
            }
        }

        &.text-right > * {
            text-align: right;
        }

        &:has(h1) span[class*="icon"] {
            font-size: calc(var(--headline-font-size-h1) * 1.6);
        }

        &:has(h2) span[class*="icon"] {
            font-size: calc(var(--headline-font-size-h2) * 1.8);
        }

        &:has(h3) span[class*="icon"] {
            font-size: calc(var(--headline-font-size-h3) * 1.9);
        }

        &:has(h4) span[class*="icon"] {
            font-size: calc(var(--headline-font-size-h4) * 2);
        }

        &:has(h5) span[class*="icon"] {
            font-size: calc(var(--headline-font-size-h5) * 2.1);
        }

        &:has(h6) span[class*="icon"] {
            font-size: calc(var(--headline-font-size-h6) * 2.2);
        }

        .pre-headline-text-wrapper {
            display: flex;
            flex-direction: column;
        }

        .pre-headline-text {
            font-size: var(--default-font-size);
            font-weight: var(--font-weight-normal);
            line-height: 1;
        }

        &:has(h4, h5, h6) .pre-headline-text {
            font-size: var(--font-size-small);
        }

        &.has-icon {
            &.text-center {
                justify-content: center;

                * {
                    text-align: left;
                }
            }

            &.text-right {
                justify-content: flex-end;

                * {
                    text-align: left;
                }
            }
        }
    }

    p {
        margin-bottom: 0;
    }

    h1, h2, h3, h4, h5, h6 {
        margin: 0;

        &:only-child {
            flex: none;
            width: 100%;
        }
    }
}

.edit-mode label.headline {
    input {
        padding: 0;
        height: auto;
        font-weight: var(--headline-font-weight);
    }

    &.h1 input {
        font-size: var(--headline-font-size-h1);
        text-transform: var(--headline-text-transform);
    }

    &.h2 input {
        font-size: var(--headline-font-size-h2);
    }

    &.h3 input {
        font-size: var(--headline-font-size-h3);
    }

    &.h4 input {
        font-size: var(--headline-font-size-h4);
    }

    &.h5 input {
        font-size: var(--headline-font-size-h5);
    }

    &.h6 input {
        font-size: var(--headline-font-size-h6);
    }
}

/* end cmd-headline ------------------------------------------------------------------------------------------ */
</style>
