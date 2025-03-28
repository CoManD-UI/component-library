<template>
    <transition :name="useTransition ? 'fade' : null">
        <div
            v-if="showSystemMessage"
            :class="['cmd-system-message', 'system-message', 'flex-container', 'vertical', { 'full-width': fullWidth }, validationStatus]"
            :role="validationStatus === 'error' ? 'alert' : 'dialog'"
            :aria-labelledby="htmlId"
        >
            <!-- begin close-icon -->
            <a
                v-if="iconClose.show && iconClose.iconClass"
                href="#"
                class="close-button"
                @click.prevent="hideSystemMessage"
                :title="iconClose.tooltip"
            >
                <!-- begin CmdIcon -->
                <CmdIcon :iconClass="iconClose.iconClass" :type="iconClose.iconType" />
                <!-- end CmdIcon -->
            </a>
            <!-- end close-icon -->

            <!-- begin slot-content -->
            <slot>
                <!-- begin CmdHeadline -->
                <CmdHeadline
                    class="message-headline"
                    :cmdIcon="headlineIcon"
                    :headlineText="systemMessage"
                    :headlineLevel="messageHeadlineLevel"
                    :id="htmlId"
                />
                <!-- end CmdHeadline -->
            </slot>
            <!-- end slot-content -->
        </div>
    </transition>
</template>

<script>
// import mixins
import Identifier from "../mixins/Identifier"

export default {
    name: "CmdSystemMessage",
    mixins: [Identifier],
    data() {
        return {
            showSystemMessage: true
        }
    },
    props: {
        /**
         * activate if transition for hiding message-box should be used
         */
        useTransition: {
            type: Boolean,
            default: true
        },
        /**
         * validation-status for the message-box
         *
         * @allowedValues: error, warning, success, info
         * @affectsStyling: true
         */
        validationStatus: {
            type: String,
            required: true,
            validator(value) {
                return value === "error" ||
                    value === "warning" ||
                    value === "success" ||
                    value === "info"
            }
        },
        /**
         * activate to stretch message-box as wide as parent container (else message-box is as wide as message (+padding))
         */
        fullWidth: {
            type: Boolean,
            default: true
        },
        /**
         * set icon-class for message (will be displayed left from message)
         */
        iconMessage: {
            type: Object,
            default: function() {
                return {
                    show: true,
                    iconClass: "icon-warning",
                    type: "auto"
                }
            }
        },
        /**
         * the system-message-text
         */
        systemMessage: {
            type: String,
            required: false
        },
        /**
         * set headline-level for system-message (given to CmdHeadline-component)
         */
        messageHeadlineLevel: {
            type: [Number, String],
            default() {
                return 6
            }
        },
        /**
         * icon to close system-message
         *
         * @requiredForAccessibility: partial
         */
        iconClose: {
            type: Object,
            default() {
                return {
                    show: true,
                    iconClass: "icon-cancel-circle",
                    tooltip: "Close this message"
                }
            }
        }
    },
    computed: {
        headlineIcon() {
            if(this.iconMessage.show) {
                return {
                    iconClass: this.iconMessage.iconClass,
                    type: this.iconMessage.type
                }
            }
            return null
        }
    },
    methods: {
        hideSystemMessage(event) {
            this.showSystemMessage = false
            this.$emit("hideSystemMessage", event)
        }
    },
    watch: {
        message() {
            this.showSystemMessage = true
        }
    }
}
</script>

<style>
/* begin cmd-system-message ---------------------------------------------------------------------------------------- */
.cmd-system-message {
    display: inline-flex;
    margin: var(--default-margin) 0;
    align-items: center;

    > :last-child {
        margin-bottom: 0;
    }

    ul {
        display: flex;
        flex-direction: column;

        li {
            margin-left: 0;
        }
    }

    > a.close-button {
        display: flex;
        position: absolute;
        width: auto; /* avoids stretching by flex-containers */
        top: .5rem;
        right: .5rem;
        text-decoration: none;
        z-index: 100;
        line-height: 1;
        background: var(--color-white);
        border-radius: var(--full-circle);

        [class*="icon-"] {
            color: var(--default-text-color);
        }

        &:hover, &:active, &:focus {
            background: none;

            [class*="icon-"] {
                color: var(--color-white);
            }
        }
    }

    &.warning {
        > a.close-button {
            border-color: var(--default-text-color);

            [class*="icon-"] {
                color: var(--default-text-color);
            }

            &:hover, &:active, &:focus {
                border-color: var(--hyperlink-color);

                [class*="icon-"] {
                    color: var(--color-white);
                }
            }
        }
    }
}

/* end cmd-system-message ------------------------------------------------------------------------------------------ */
</style>
