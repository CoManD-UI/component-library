<template>
    <!-- begin CmdTooltip -->
    <CmdTooltip
        ref="tooltip"
        class="box"
        :class="validationStatus"
        :relatedId="relatedId"
        :toggle-visibility-by-click="true">

        <!-- begin CmdListOfRequirements -->
        <CmdListOfRequirements
            v-if="cmdListOfRequirements.showRequirements"
            v-bind="cmdListOfRequirements"
        />
        <!-- end CmdListOfRequirements -->
    </CmdTooltip>
    <!-- end CmdTooltip -->
</template>

<script>
export default {
    name: "CmdTooltipForInputElements",
    props: {
        /**
         * related-id for CmdTooltip-component
         */
        relatedId: {
            type: String,
            default: ""
        },
        /**
         * validation-status for CmdTooltip-component
         */
        validationStatus: {
            type: String,
            default: ""
        },
        /**
         * properties for CmdListOfRequirements-component
         */
        cmdListOfRequirements: {
            type: Object,
            default() {
                return {
                    showRequirements: true,
                    validationTooltip: "",
                    inputRequirements: [],
                    inputAttributes: {},
                    showHeadline: true,
                    inputModelValue: "",
                    helplink: {},
                    labelText: ""
                }
            }
        }
    },
    methods: {
        hideTooltip() {
            // close tooltip by calling function from CmdTooltip using $refs
            this.$refs.tooltip.hideTooltip()
        }
    }
}
</script>