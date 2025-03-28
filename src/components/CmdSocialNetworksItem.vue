<template>
    <li class="cmd-social-networks-item">
        <a
            :key="network.path"
            :class="['button', network.buttonClass, {disabled: userMustAcceptDataPrivacy && !dataPrivacyAccepted}, 'text-align-' + buttonTextAlign]"
            :href="getUrl(network)"
            @click="preventOnDisabled"
            target="_blank"
            :title="tooltip(network.tooltip)">

            <!-- begin CmdIcon -->
            <CmdIcon v-if="network.iconClass" :iconClass="network.iconClass" :type="network.iconType" />
            <!-- end CmdIcon -->

            <span v-if="network.linkText">{{ network.linkText }}</span>
        </a>
    </li>
</template>

<script>
import {createUuid} from "../utils/common.js"
import {updateHandlerProvider} from "../utils/editmode.js"
import EditMode from "../mixins/EditMode.vue"

export default {
    name: "CmdSocialNetworksItem",
    data() {
        return {
            editablePath: null,
            editableTooltip: null,
            editableLinktext: null
        }
    },
    mixins: [EditMode],
    props: {
        network: {
            type: Object,
            required: true
        },
        /**
         * toggle if user has to accept that anonymous data will be sent while sharing
         */
        userMustAcceptDataPrivacy: {
            type: Boolean
        },
        /**
         * page to share (appended to social-bookmark-url)
         *
         * appendPage-property must be activated
         */
        page: {
            type: String,
            required: false
        },
        /**
         * activate if page to share is not given by json-data
         */
        appendPage: {
            type: Boolean,
            default: true
        },
        /**
         * alignment for buttons
         *
         * @allowedValues: "left", "right"
         */
        buttonTextAlign: {
            type: String,
            default: "left",
            validator(value) {
                return value === "left" ||
                    value === "right"
            }
        },
        /**
         * information if the user has accepted the data privacy (by checking the checkbox)
         */
        dataPrivacyAccepted: {
            type: Boolean
        },
        /**
         * tooltip shown on hovering disabled buttons
         *
         * userMustAcceptDataPrivacy-property must be activated
         */
        tooltipAcceptDataPrivacy: {
            type: String,
            default: "You must accept data privacy conditions!"
        }
    },
    methods: {
        getUrl(network) {
            if (this.userMustAcceptDataPrivacy && this.dataPrivacyAccepted) {
                // if path is not given completely by json-data
                if (this.appendPage) {
                    // if page to share is given by property
                    if (this.page) {
                        return network.path + encodeURIComponent(this.page)
                    }

                    // if current page should be appended to url
                    return network.path + encodeURIComponent(location.href)
                }

                // if path is given completely by json-data
                return network.path
            }
            return "#"
        },
        preventOnDisabled(event) {
            let clickedElement = event.target

            if (clickedElement.tagName !== "A") {
                // get surrounding <a> if inner <span> is clicked
                clickedElement = clickedElement.closest("a")
            }

            // href must be set due to html-validity, so click must be prevented if href contains "#" only (equals button is styled as disabled)
            if (clickedElement.getAttribute("href") === "#") {
                event.preventDefault()
            }
        },
        tooltip(tooltip) {
            if (this.userMustAcceptDataPrivacy) {
                if (this.dataPrivacyAccepted) {
                    return tooltip
                }
                return this.tooltipAcceptDataPrivacy
            }
            return tooltip
        },
        addHandlerProvider() {
            const itemStructure = {
                "id": "social-network-facebook",
                "path": "https://www.facebook.com/sharer/sharer.php?u=http%3A%2F%2Fdevelopment.comand-cms.com%2Fmodule%2Fteam.html",
                "tooltip": "Share this page on facebook",
                "iconClass": "icon-facebook",
                "linkText": "Share"
            }
            return updateHandlerProvider(this, {
                item() {
                    return itemStructure
                }
            })
        },
        updateHandlerProvider() {
            const id = createUuid()
            const path = this.editablePath
            const tooltip = this.editableTooltip
            const linkText = this.editableLinktext
            return updateHandlerProvider(this, {
                update(props) {
                    props.id = id
                    props.path = path
                    props.tooltip = tooltip
                    props.linkText = linkText
                }
            })
        }
    }
}
</script>

<style>
.cmd-social-networks-item {
    .button {
        --social-network-text-color: var(--color-white);
        --social-network-text-color-highlighted: var(--social-network-color);
        --social-network-border-color: var(--social-network-color);
        --social-network-background-color: var(--social-network-color);
        --social-network-background-color-highlighted: var(--color-white);

        background: var(--social-network-background-color);
        border-color: var(--social-network-border-color);

        > span {
            color: var(--social-network-text-color);
        }

        &:hover, &:active, &:focus {
            color: var(--social-network-background-color-highlighted);

            > span {
                color: var(--social-network-text-color-highlighted);
            }
        }

        &.social-network-facebook {
            --social-network-color: #3c5a99;
        }

        &.social-network-x {
            --social-network-color: #000;
        }

        &.social-network-xing {
            --social-network-color: #007575;
        }

        &.social-network-linkedin {
            --social-network-color: #0077b5;
        }

        &.social-network-instagram {
            --social-network-color: #000;
        }
    }
}
</style>