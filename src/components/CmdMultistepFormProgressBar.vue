<template>
    <ol :class="['cmd-multistep-form-progress-bar', {'use-gap': useGap}]">
        <li v-for="(step, index) in multisteps" :key="index" :class="{active : activeLink === index}">
            <!-- begin CmdLink -->
            <CmdLink
                :linkType="step.type"
                :path="step.path"
                :title="step.tooltip"
                @click.stop.prevent="clickedStep($event, index)"
            >
                <span v-if="showStepNumber" class="number">{{ index + 1 }}</span>
                <!-- begin CmdIcon -->
                <CmdIcon v-if="step.iconClass" :iconClass="step.iconClass" :type="step.iconType" />
                <!-- end CmdIcon -->
                <span v-if="step.text">{{ step.text }}</span>
                <span :class="separatorIconClass"></span>
            </CmdLink>
            <!-- end CmdLink -->
        </li>
    </ol>
</template>

<script>
// import functions
import {getRoute} from "../utilities.js"

export default {
    name: 'CmdMultistepFormProgressBar',
    data() {
        return {
            activeLink: 0
        }
    },
    props: {
        /**
         * toggle visibility of step-number in front/left of link-icon and -name
         */
        showStepNumber: {
            type: Boolean,
            default: false
        },
        /**
         * list of multisteps
         */
        multisteps: {
            type: Array,
            required: true
        },
        /**
         * icon-class for separator shown between multisteps
         */
        separatorIconClass: {
            type: String,
            default() {
                return "icon-chevron-one-stripe-right"
            }
        },
        useGap: {
            type: Boolean,
            default: true
        }
    },
    methods: {
        clickedStep(event, index) {
            this.activeLink = index;
            this.$emit('click', {event: event, index: index, stepPath: this.multisteps[index].path})
        },
        getRoute(step) {
            return getRoute(step)
        }
    }
}
</script>

<style>
/* begin cmd-multistep-form-progress-bar ---------------------------------------------------------------------------------------- */

.cmd-multistep-form-progress-bar {
    display: flex;
    justify-content: space-around;
    border: var(--default-border);
    margin: 0;
    border-radius: var(--default-border-radius);

    &.use-gap {
        border: 0;
        gap: var(--default-gap);

        li {
            border: var(--default-border);
            border-color: var(--primary-color-reduced-opacity);
            border-radius: var(--default-border-radius);;

            a, a.active {
                border: 0;
                border-radius: inherit;

                :is(span, [class*="icon-"]) + [class*="icon-"]:last-child {
                    border: 0;
                    right: -1rem;
                    background: none;
                }
            }

            a {
                :is(span, [class*="icon-"]) + [class*="icon-"]:last-child {
                    color: var(--color-scheme-text-color) !important;
                }
            }

            &.active {
                border-color: var(--primary-color);

                & ~li {
                    border-color: var(--border-color);
                }
            }
        }
    }

    li {
        display: flex;
        flex: 1;
        list-style-type: none;
        margin: 0;

        &:first-child, &:first-child > a {
            border-top-left-radius: inherit;
            border-bottom-left-radius: inherit;
        }

        &:last-child, &:last-child > a {
            border-top-right-radius: inherit;
            border-bottom-right-radius: inherit;
        }

        a {
            display: flex;
            text-decoration: none;
            padding: var(--default-padding);
            align-items: center;
            justify-content: center;
            width: 100%;
            background: var(--primary-color-reduced-opacity);

            :is(span, [class*="icon-"]) {
                color: var(--color-white);
                font-size: var(--icon-size-medium) !important;

                & + [class*="icon-"] {
                    &:last-child {
                        border: var(--default-border);
                        border-radius: var(--full-circle);
                        background: var(--color-white);
                        color: var(--color-scheme-text-color);
                        margin: 0;
                        position: absolute;
                        right: 0;
                        transform: translateX(50%);
                        padding: calc(var(--default-padding) / 2);
                        display: flex;
                        justify-content: center;
                        z-index: 1;

                        &::before {
                            font-size: var(--font-size-small);
                        }
                    }
                }
            }

            &:hover, &:active, &:focus {
                background: var(--primary-color);

                :is(span, [class*="icon-"]) {
                    color: var(--color-white);
                }

                .number {
                    color: var(--hyperlink-color);
                }
            }

            .number {
                margin-right: calc(var(--default-margin) / 2);
                line-height: 100%;
                width: 2rem;
                height: 2rem;
                display: flex;
                align-items: center;
                justify-content: center;
                border-radius: var(--full-circle);
                border: var(--default-border);
                border-color: var(--color-white);
                background: var(--color-white);
                color: var(--primary-color-reduced-opacity);
            }
        }

        &:last-child {
            a {
                [class*="icon-"] {
                    &:last-of-type {
                        display: none;
                    }
                }
            }
        }

        &.active {
            a {
                background: var(--primary-color);

                span, [class*="icon-"] {
                    color: var(--color-white);
                }

                &:hover, &:active, &:focus {
                    span, [class*="icon-"] {
                        color: var(--color-white);

                        & + [class*="icon-"] {
                            &:last-child {
                                color: var(--color-scheme-text-color);
                            }
                        }
                    }
                }
            }

            .number {
                background: var(--color-white);
                border-color: var(--color-white);
                color: var(--hyperlink-color) !important;
            }

            & ~ li {
                background: var(--default-background);
                border-left-color: var(--border-color);

                a {
                    color: var(--color-white);

                    span, span[class*='color'] {
                        color: inherit;

                        & + [class*="icon-"] {
                            &:last-child {
                                border-color: var(--border-color);
                                color: var(--color-white);
                                background: var(--secondary-color);
                            }
                        }
                    }

                    &:hover, &:active, &:focus {
                        background: var(--color-white);

                        span, [class*="icon-"] {
                            &:not(:last-child) {
                                color: var(--hyperlink-color);
                            }
                        }

                        .number {
                            border-color: var(--hyperlink-color);
                        }
                    }
                }

                .number {
                    background: none;
                    border-color: var(--border-color);
                }
            }

            & + li {
                border-left-color: var(--border-color);
            }
        }
    }
}
</style>

<style lang="scss">
@import '../assets/styles/variables';

@media only screen and (width < #{$small-max-width}) {
    .cmd-multistep-form-progress-bar {
        flex-direction: column;

        li {
            &:not(:last-child) {
                border-bottom: var(--default-border);
            }

            a {
                :is(span, [class*="icon-"]) {
                    & + [class*="icon-"] {
                        &:last-child {
                            left: auto;
                            right: auto;
                            bottom: 0;
                            transform: translateY(50%);

                            &::before {
                                transform: rotate(90deg);
                            }
                        }
                    }
                }
            }

            &.active {
                & ~ li {
                    border-left: 0;
                    border-bottom-color: var(--border-color);
                }
            }
        }
    }
}

/* end cmd-multistep-form-progress-bar ------------------------------------------------------------------------------------------ */
</style>
