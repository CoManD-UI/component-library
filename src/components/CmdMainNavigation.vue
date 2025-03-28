<template>
    <div
        :class="[
            'cmd-main-navigation main-navigation-wrapper',
            {
                'hide-sub-navigation' : !showSubNavigations,
                'open-off-canvas': showOffcanvas,
                'persist-on-mobile': persistOnMobile,
                'show-content-overlay': showContentOverlay,
                'off-canvas-right': offcanvasPosition === 'right'
            }
        ]">
        <nav>
            <!-- begin main-level -->
            <ul :class="{'stretch-items' : stretchMainItems}">
                <li class="close-nav" v-if="showOffcanvas">
                    <a href="#" id="close-offcanvas" role="button" @click.prevent="closeOffcanvasNavigation">
                        <!-- begin CmdIcon -->
                        <CmdIcon v-if="closeOffcanvas.iconClass" :iconClass="closeOffcanvas.iconClass" :type="closeOffcanvas.iconType" />
                        <!-- end CmdIcon -->
                        <span :class="{'hidden': !closeOffcanvas.showText}">{{ closeOffcanvas.text }}</span>
                    </a>
                </li>
                <li v-for="(navigationEntry, index) in navigationEntries" :key="index"
                    :class="{'active' : navigationEntry.active, 'open' : openEntry === index, 'has-subentries': navigationEntry?.subentries?.length}">
                    <!-- begin type === href -->
                    <a
                       v-if="navigationEntry.type === 'href'"
                       :href="navigationEntry.path"
                       :title="navigationEntry.tooltip"
                       :target="navigationEntry.target"
                       @click="executeLink($event, navigationEntry, index)"
                       @mouseover="closeAllSubentries()"
                       @focus="closeAllSubentries()"
                    >
                        <!-- begin CmdIcon -->
                        <CmdIcon v-if="navigationEntry.iconClass" :iconClass="navigationEntry.iconClass" :type="navigationEntry.iconType" />
                        <!-- end CmdIcon -->
                        <span v-if="navigationEntry.text">{{ navigationEntry.text }}</span>
                        <span v-if="navigationEntry?.subentries?.length"
                              :class="['subentry-icon', subentriesIconClass]"
                        ></span>
                    </a>
                    <!-- end type === href -->

                    <!-- begin type === router -->
                    <router-link
                        v-if="navigationEntry.type === 'router'"
                        :to="getRoute(navigationEntry)"
                        :title="navigationEntry.tooltip"
                        @click="closeOffcanvasNavigation"
                    >
                        <!-- begin CmdIcon -->
                        <CmdIcon v-if="navigationEntry.iconClass" :iconClass="navigationEntry.iconClass" :type="navigationEntry.iconType" />
                        <!-- end CmdIcon -->
                        <span v-if="navigationEntry.text">{{ navigationEntry.text }}</span>
                        <span v-if="navigationEntry.subentries && navigationEntry.subentries.length > 0"
                              :class="['subentry-icon', subentriesIconClass]"></span>
                    </router-link>
                    <!-- end type === router -->

                    <!-- begin sub-level 1 -->
                    <ul v-if="navigationEntry?.subentries?.length" :aria-expanded="openEntry ? 'true' : 'false'">
                        <li v-for="(navigationSubEntry, subindex) in navigationEntry.subentries" :key="subindex" :class="{'open' : openSubentry === subindex, 'has-subentries': navigationSubEntry?.subentries?.length}">
                            <!-- begin type === href -->
                            <a v-if="navigationSubEntry.type === 'href'"
                               :href="navigationSubEntry.path"
                               :title="navigationSubEntry.tooltip"
                               :target="navigationSubEntry.target"
                               @click="executeLink($event, navigationSubEntry, subindex, 1)"
                               @mouseover="closeAllSubentries(1)"
                               @focus="closeAllSubentries(1)"
                            >
                                <!-- begin CmdIcon -->
                                <CmdIcon v-if="navigationSubEntry.iconClass" :iconClass="navigationSubEntry.iconClass" :type="navigationSubEntry.iconType" />
                                <!-- end CmdIcon -->
                                <span v-if="navigationSubEntry.text">{{ navigationSubEntry.text }}</span>
                                <span v-if="navigationSubEntry.subentries && navigationSubEntry.subentries.length > 0"
                                      :class="['subentry-icon', subentriesIconClass]"
                                ></span>
                            </a>
                            <!-- end type === href -->

                            <!-- begin type === router -->
                            <router-link
                                v-if="navigationSubEntry.type === 'router'"
                                :to="getRoute(navigationSubEntry)"
                                :title="navigationSubEntry.tooltip"
                                @click="closeOffcanvasNavigation"
                            >
                                <!-- begin CmdIcon -->
                                <CmdIcon v-if="navigationSubEntry.iconClass" :iconClass="navigationSubEntry.iconClass" :type="navigationSubEntry.iconType" />
                                <!-- end CmdIcon -->
                                <span v-if="navigationSubEntry.text">{{ navigationSubEntry.text }}</span>
                                <span v-if="navigationSubEntry.subentries && navigationSubEntry.subentries.length > 0"
                                      :class="['subentry-icon', subentriesIconClass]"></span>
                            </router-link>
                            <!-- end type === router -->

                            <!-- begin sub-level 2 -->
                            <ul v-if="navigationSubEntry.subentries?.length" :aria-expanded="openSubentry ? 'true' : 'false'">
                                <li v-for="(navigationSubSubEntry, subsubindex) in navigationSubEntry.subentries"
                                    :key="subsubindex">
                                    <!-- begin type === href -->
                                    <a v-if="navigationSubSubEntry.type === 'href'"
                                       :href="navigationSubSubEntry.path"
                                       :title="navigationSubSubEntry.tooltip"
                                       :target="navigationSubSubEntry.target"
                                       @click="executeLink($event, navigationSubSubEntry)"
                                    >
                                        <!-- begin CmdIcon -->
                                        <CmdIcon v-if="navigationSubSubEntry.iconClass" :iconClass="navigationSubSubEntry.iconClass" :type="navigationSubSubEntry.iconType" />
                                        <!-- end CmdIcon -->
                                        <span v-if="navigationSubSubEntry.text">{{ navigationSubSubEntry.text }}</span>
                                        <span v-if="navigationSubSubEntry.subentries && navigationSubSubEntry.subentries.length > 0"
                                              :class="['subentry-icon', subentriesIconClass]"
                                        ></span>
                                    </a>
                                    <!-- end type === href -->

                                    <!-- begin type === router -->
                                    <router-link v-if="navigationSubSubEntry.type === 'router'"
                                                 :to="getRoute(navigationSubSubEntry)"
                                                 :target="navigationSubSubEntry.target"
                                                 @click="closeOffcanvasNavigation"
                                    >
                                        <!-- begin CmdIcon -->
                                        <CmdIcon v-if="navigationSubSubEntry.iconClass" :iconClass="navigationSubSubEntry.iconClass" :type="navigationSubSubEntry.iconType" />
                                        <!-- end CmdIcon -->
                                        <span v-if="navigationSubSubEntry.text">{{ navigationSubSubEntry.text }}</span>
                                        <span v-if="navigationSubSubEntry.subentries && navigationSubSubEntry.subentries.length > 0"
                                              :class="['subentry-icon', subentriesIconClass]"></span>
                                    </router-link>
                                    <!-- end type === router -->
                                </li>
                            </ul>
                            <!-- end sub-level 2 -->
                        </li>
                    </ul>
                    <!-- begin sub-level 1 -->
                </li>
            </ul>
            <!-- end main-level -->
        </nav>
        <!-- begin off-canvas-button -->
        <a v-if="persistOnMobile === false" href="#" class="button" id="toggle-offcanvas" @click.prevent="toggleOffcanvasNavigation">
            <!-- begin CmdIcon -->
            <CmdIcon :iconClass="buttonOffcanvas.iconClass" :type="buttonOffcanvas.iconType" />
            <!-- end CmdIcon -->
            <span :class="{'hidden' : !buttonOffcanvas.showText}">{{ buttonOffcanvas.text }}</span>
        </a>
        <!-- end off-canvas-button -->
    </div>
</template>

<script>
// import functions
import {getRoute} from "../utilities.js"

export default {
    name: "CmdMainNavigation",
    data() {
        return {
            showOffcanvas: false,
            showSubNavigations: true,
            openEntry: -1,
            openSubentry: -1
        }
    },
    emits: ["offcanvas"],
    props: {
        /**
         * toggle if main-entries (on first-level) should be (horizontally) stretched equally
         */
        stretchMainItems: {
            type: Boolean,
            default: false
        },
        /**
         * set if navigation should persist on mobile and not be collapsed to off-canvas
         *
         * @affectsStyling: true
         */
        persistOnMobile: {
            type: Boolean,
            default: false
        },
        /**
         * toggle if overlay over content should be shown if off-canvas is open
         *
         * @affectsStyling: true
         */
        showContentOverlay: {
            type: Boolean,
            default: true
        },
        /**
         * list of all navigation-entries (incl. sub-entries)
         */
        navigationEntries: {
            type: Array,
            required: true
        },
        /**
         * link shown inside off-canvas-navigation to close itself
         *
         * @requiredForAccessibility: partial
         */
        closeOffcanvas: {
            type: Object,
            default: function () {
                return {
                    iconClass: "icon-cancel",
                    text: "Close navigation",
                    showText: true
                }
            }
        },
        /**
         * position for offcanvas-navigation
         *
         * @allowedValues: "right", "left"
         */
        offcanvasPosition: {
            type: String,
            default: "right",
            validator(value) {
                return value === "right" ||
                    value === "left"
            }
        },
        /**
         * button to open off-canvas-navigation
         *
         * (persistOnMobile-property must be false)
         *
         * @requiredForAccessibility: partial
         */
        buttonOffcanvas: {
            type: Object,
            default: function () {
                return {
                    iconClass: "icon-bars",
                    text: "Open navigation",
                    showText: false
                }
            }
        },
        /**
         * icon to show if a navigation-entry has sub-entries
         */
        subentriesIconClass: {
            type: String,
            default: "icon-chevron-one-stripe-down"
        },
        /**
         * icon to show if a sub-entry has further sub-entries
         */
        subSubentriesIconClass: {
            type: String,
            default: "icon-chevron-one-stripe-right"
        }
    },
    methods: {
        closeAllSubentries(sublevel) {
            if(!sublevel) {
                this.openEntry = -1
            }
            this.openSubentry = -1
        },
        executeLink(event, navigationEntry, index, sublevel) {
            // execute default-link
            if (navigationEntry.target || (navigationEntry.path.length > 1)) {
                this.showOffcanvas = false
                return true
            }

            // toggle subentries (no link execution)
            if(navigationEntry?.subentries?.length) {
                event.preventDefault()
                if(!sublevel) {
                    // assign index for first sublevel (or close if already open)
                    this.openEntry = this.openEntry === index ? -1 : index
                    this.openSubentry = -1 // close all sub-entries
                } else {
                    // assign index for second sublevel (or close if already open)
                    this.openSubentry = this.openSubentry === index ? -1 : index
                }
                return
            }

            // emit event to handle navigation from outside
            if (navigationEntry.path === '#' || navigationEntry.path === '') {
                event.preventDefault()
                this.$emit("click", navigationEntry.path)
            }

            this.showOffcanvas = false
        },
        getRoute(navigationEntry) {
            return getRoute(navigationEntry)
        },
        closeOffcanvasNavigation() {
            this.showOffcanvas = false
            document.getElementById("toggle-offcanvas")?.focus()
        },
        toggleOffcanvasNavigation() {
            if (this.showOffcanvas) {
                // close offcanvas
                this.closeOffcanvasNavigation()
            } else {
                // open offcanvas
                this.showOffcanvas = true
                this.$nextTick(() => document.getElementById("close-offcanvas")?.focus())
            }
        }
    },
    watch: {
        showOffcanvas() {
            this.$emit("offcanvas", { open: this.showOffcanvas})
        }
    }
}
</script>

<style>
/* begin cmd-main-navigation ---------------------------------------------------------------------------------------- */
.cmd-main-navigation {
    &.hide-sub-navigation {
        ul {
            ul {
                display: none;
            }
        }
    }

    nav {
        .stretch-items {
            > li {
                flex: 1
            }
        }

        > ul {
            > li {
                .close-nav {
                    display: none;
                }

                &.open {
                    > ul {
                        display: block;

                        > li {
                            &.open {
                                > ul {
                                    display: block;
                                }
                            }
                        }
                    }
                }
            }
        }

        + div {
            display: none;
        }
    }

    .fade-enter-active, .fade-leave-active {
        left: 0;
    }

    .fade-enter, .fade-leave-to {
        left: -100%;
    }

    header nav > ul, header nav > ul > li {
        border: 0;
    }
}

/* keep outside of .cmd-main-navigation to keep specificity */
.off-canvas-right {
    #toggle-offcanvas {
        margin-right: 0;
        margin-left: auto;
    }
}

#toggle-offcanvas {
    margin-left: 0;
    display: none;
}
</style>

<style lang="scss">
@import '../assets/styles/variables';

@media only screen and (max-width: $medium-max-width) {
    .cmd-main-navigation.main-navigation-wrapper {
        --nav-transition: all .5s linear;

        display: flex;
        background: none; /* overwrite framework-css */
        border: 0; /* overwrite framework-css */

        /* begin offcanvas-navigation */
        &:not(.persist-on-mobile) {
            padding: 0 var(--default-padding);
            transition: var(--nav-transition);

            #toggle-offcanvas {
                display: flex;
                margin-bottom: 0;
                width: auto;
            }

            &.open-off-canvas {
                nav {
                    height: 100%;
                    left: 0;
                    opacity: 1;
                    padding: 0 !important;
                    transition: var(--nav-transition);
                    border-right: var(--default-border);
                    display: block;
                }

                &.show-content-overlay {
                    nav {
                        &::after {
                            content: "";
                            position: fixed;
                            width: 100%;
                            top: 0;
                            left: 0;
                            height: 100dvh;
                            display: block;
                            background: var(--color-black-reduced-opacity);
                        }

                        ul {
                            z-index: 1000; /* keep ul above overlay */
                            height: 100%;
                            background: var(--default-background);
                        }
                    }
                }
            }

            nav {
                position: fixed;
                top: 0;
                left: -100%;
                width: auto;
                height: 100dvh;
                opacity: 0;
                z-index: 1000;
                transition: var(--nav-transition);

                ul {
                    flex-direction: column;
                    position: relative;
                    left: 0;
                    border: 0;

                    li {
                        border-bottom: var(--default-border);
                        border-right: 0;

                        &.close-nav {
                            display: block;
                            border-bottom: var(--default-border);

                            a {
                                display: flex;
                                align-items: center;

                                span {
                                    font-weight: var(--font-weight-bold);

                                    &[class*="icon-"] {
                                        font-size: var(--font-size-small);
                                    }
                                }
                            }

                            > a {
                                text-align: left;
                            }
                        }

                        &.has-subentries.open {
                            &:hover, &:active, &:focus {
                                > ul {
                                    display: block;
                                }
                            }
                        }

                        /* sub-level 1 */
                        ul {
                            li {
                                &:not(:first-child) {
                                    border-top: 0;
                                }

                                &:last-child {
                                    border-bottom: 0;
                                }
                                > a {
                                    padding-left: calc(var(--default-margin) * 2);

                                    span {
                                        & + [class*="icon-"]::before {
                                            display: inline-block;
                                            transform: rotate(0);
                                        }
                                    }
                                }

                                /* sub-level 2 */
                                ul {
                                    li {
                                        a {
                                            padding-left: calc(var(--default-margin) * 4);
                                        }
                                    }
                                }
                            }
                        }

                        &.open {
                            > a span {
                                + [class*="icon-"]::before {
                                    display: inline-block;
                                    transform: rotate(-180deg);
                                }
                            }

                            > ul {
                                height: auto;
                                display: block;
                                opacity: 1;
                                transition: var(--nav-transition);
                            }
                        }
                    }
                }
            }

            &.off-canvas-right {
                &.open-off-canvas {
                    nav {
                        right: 0;
                    }
                }

                nav {
                    right: -100%;
                    left: auto;
                }
            }
        }
    }
}
/* end cmd-main-navigation ------------------------------------------------------------------------------------------ */
</style>