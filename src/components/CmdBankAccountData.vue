<template>
    <div class="cmd-bank-account-data">
        <!-- begin slot -->
        <template v-if="useSlot">
            <slot></slot>
        </template>
        <!-- end slot -->

        <template v-else>
            <!-- begin CmdHeadline -->
            <CmdHeadline v-if="cmdHeadline" v-bind="cmdHeadline"/>
            <!-- end CmdHeadline -->

            <!-- begin account data -->
            <dl>
                <template v-for="(entry, index) in accountData" :key="index">
                    <dt>{{ entry.text }}</dt>
                    <dd v-if="allowCopyByClick && entry.allowCopy">
                        <span>{{ entry.value }}</span>
                        <a href="#" @click.prevent="copyToClipboard(entry.value)" :title="cmdIcon.tooltip">
                            <!-- begin CmdIcon -->
                            <CmdIcon :iconClass="cmdIcon.iconClass" :type="cmdIcon.iconType"/>
                            <!-- end CmdIcon -->
                        </a>
                    </dd>
                    <dd v-else :key="index">{{ entry.value }}</dd>
                </template>
            </dl>
            <!-- end account data -->

            <!-- begin additional information -->
            <p v-if="additionalInformation">{{ additionalInformation }}</p>
            <!-- end additional information -->
        </template>
    </div>
</template>

<script>
export default {
    name: "CmdBankAccountData",
    props: {
        /**
         * activate if you want to use slot instead for given structure
         */
        useSlot: {
            type: Boolean,
            default: false
        },
        /**
         * bank account data
         *
         * must contain: owner, name of bank, IBAN, SWIFT/BIC
         */
        accountData: {
            type: Array,
            required: true
        },
        /**
         * enable if data can be copied by click on icon
         */
        allowCopyByClick: {
            type: Boolean,
            default: true
        },
        /**
         * additional information shown in a paragraph below account data
         */
        additionalInformation: {
            type: String,
            required: false
        },
        /**
         * icon 'copy'
         *
         * @requiredForAccessibility: partial
         */
        cmdIcon: {
            type: Object,
            default() {
                return {
                    iconClass: "icon-file-copy",
                    tooltip: "Copy data to clipboard!",
                    iconType: "auto"
                }
            }
        },
        /**
         * properties for CmdHeadline-component
         */
        cmdHeadline: {
            type: Object,
            required: false
        }
    },
    methods: {
        copyToClipboard(entry) {
            // get data, remove spaces and copy to clipboard
            const output = entry.replace(/\s/g, "")
            navigator.clipboard.writeText(output)
            alert(output + " was copied to clipboard")
        }
    }
}
</script>

<style>
/* begin cmd-bank-account-data ---------------------------------------------------------------------------------------- */
.cmd-bank-account-data {
    dd {
        display: flex;
        gap: calc(var(--default-gap) / 2);

        > span {
            &:first-child {
                white-space: nowrap;
            }
        }
    }
}

/* end cmd-bank-account-data ---------------------------------------------------------------------------------------- */
</style>