<template>
    <div class="flexible-field-group relative mb-4 pb-1" :id="group.key">
        <div class="w-full shrink">
            <div :class="titleStyle" v-if="group.title">
                <div class="h-8 leading-normal h-full flex items-center box-content"
                    :class="{'border-b border-gray-200 dark:border-gray-700 ': !collapsed}">
                    <button
                        dusk="expand-group"
                        type="button"
                        class="shrink-0 group-control btn border-r border-gray-200 dark:border-gray-700 w-8 h-8 block"
                        :title="__('Expand')"
                        @click.prevent="expand"
                        v-if="collapsed">
                        <icon type="plus" class="align-top" width="16" height="16" />
                    </button>
                    <button
                        dusk="collapse-group"
                        type="button"
                        class="group-control btn border-r border-gray-200 dark:border-gray-700 w-8 h-8 block"
                        :title="__('Collapse')"
                        @click.prevent="collapse"
                        v-else>
                        <icon type="minus" class="align-top" width="16" height="16" />
                    </button>

                    <p class="text-80 font-bold grow px-4">
                      <span class="mr-3 font-semibold">#{{ index + 1 }}</span>
                        {{ group.title }}
                        <span
                            v-if="getGroupSubTitle(group)"
                            class="text-gray-400"
                            v-html="getGroupSubTitle(group)"
                        />
                    </p>

                    <div class="flex" v-if="!readonly">
                        <button
                            dusk="move-up-group"
                            type="button"
                            class="group-control btn border-l border-gray-200 dark:border-gray-700 w-8 h-8 block"
                            :title="__('Move up')"
                            @click.prevent="moveUp">
                            <icon type="arrow-up" class="align-top" width="16" height="16" />
                        </button>
                        <button
                            dusk="move-down-group"
                            type="button"
                            class="group-control btn border-l border-gray-200 dark:border-gray-700 w-8 h-8 block"
                            :title="__('Move down')"
                            @click.prevent="moveDown">
                            <icon type="arrow-down" class="align-top" width="16" height="16" />
                        </button>
                        <button
                            dusk="delete-group"
                            type="button"
                            class="group-control btn border-l border-gray-200 dark:border-gray-700 w-8 h-8 block"
                            :title="__('Delete')"
                            @click.prevent="confirmRemove">
                            <icon type="trash" width="16" height="16" />
                        </button>
                        <delete-flexible-content-group-modal
                            v-if="removeMessage"
                            @confirm="remove"
                            @close="removeMessage=false"
                            :message="field.confirmRemoveMessage"
                            :yes="field.confirmRemoveYes"
                            :no="field.confirmRemoveNo"
                        />
                    </div>

                </div>
            </div>
            <div :class="containerStyle">
                <component
                    v-for="(item, index) in group.fields"
                    :key="index"
                    :is="'form-' + item.component"
                    :resource-name="resourceName"
                    :resource-id="resourceId"
                    :resource="resource"
                    :field="item"
                    :errors="errors"
                    :show-help-text="item.helpText != null"
                    :class="{ 'remove-bottom-border': index == group.fields.length - 1 }"
                />
            </div>
        </div>
    </div>
</template>

<script>
import BehavesAsPanel from '../../../vendor/laravel/nova/resources/js/mixins/BehavesAsPanel.js';

export default {
    mixins: [BehavesAsPanel],

    props: ['errors', 'group', 'index', 'field'],

    emits: ['move-up', 'move-down', 'remove'],

    data() {
        return {
            removeMessage: false,
            collapsed: this.group.collapsed,
            readonly: this.group.readonly,
        };
    },

    mounted() {
        const collapsed = localStorage.getItem(this.getCollapsedKey());

        if (collapsed === "true") {
            this.collapse();
        }

        Nova.$on('collapse-all-fields', () => this.collapse());
        Nova.$on('expand-all-fields', () => this.expand());
    },

    computed: {
        titleStyle() {
            let classes = ['border-t', 'border-r', 'border-l', 'border-gray-200', 'dark:border-gray-700', 'rounded-t-lg'];
            if (this.collapsed) {
                classes.push('border-b rounded-b-lg');
            }
            return classes;
        },
        containerStyle() {
            let classes = ['grow', 'border-b', 'border-r', 'border-l', 'border-gray-200', 'dark:border-gray-700', 'rounded-b-lg'];
            if(!this.group.title) {
                classes.push('border-t');
                classes.push('rounded-tr-lg');
            }
            if (this.collapsed) {
                classes.push('hidden');
            }
            return classes;
        }
    },

    methods: {
        /**
         * Move this group up
         */
        moveUp() {
            this.$emit('move-up');
        },

        /**
         * Move this group down
         */
        moveDown() {
            this.$emit('move-down');
        },

        /**
         * Remove this group
         */
        remove() {
            this.$emit('remove');
        },

        /**
         * Confirm remove message
         */
        confirmRemove() {
            if(this.field.confirmRemove){
                this.removeMessage = true;
            } else {
                this.remove()
            }
        },

        /**
         * Expand fields
         */
        expand() {
            this.collapsed = false;
            this.saveCollapsedState(false);
        },

        /**
         * Collapse fields
         */
        collapse() {
            this.collapsed = true;
            this.saveCollapsedState(true);
        },

        /**
         * Save collapsed state
         */
        saveCollapsedState(collapsed) {
            localStorage.setItem(this.getCollapsedKey(), collapsed);
        },

        /**
         * Get the local storage key for collapsed state
         */
        getCollapsedKey() {
            return `resource_${this.resourceId}_group_collapsed_${this.group.key}`;
        },

        /**
         * Adds a secondary title that explains the group's content
         */
        getGroupSubTitle(group) {
            let subTitle = '';
            group.fields.forEach((item) => {
                if (item.value && item.name === 'Title') {
                    subTitle = item.value.replace(/(<([^>]+)>)/gi, "");
                } else if (item.value && ['two-columns', 'two-columns-with-sticky-image'].includes(group.name)) {
                    if (item.name === 'Column left') {
                        subTitle = item.value[0]?.layout;
                    } else if (item.name === 'Column right') {
                        subTitle += ' + ' + item.value[0]?.layout;
                    }
                }
            });

            if(subTitle.length > 38) {
                subTitle = subTitle.substring(0,38) + '..';
            }

            return subTitle ? `(${subTitle})` : '';
        }
    },
}
</script>

<style>
    .group-control:focus {
        outline: none;
    }
    .group-control:hover {
        color: rgb(var(--colors-primary-400));
    }
    .confirm-message{
        position: absolute;
        overflow: visible;
        right: 38px;
        bottom: 0;
        width: auto;
        border-radius: 4px;
        padding: 6px 7px;
        border: 1px solid #B7CAD6;
        background-color: var(--20);
        white-space: nowrap;
    }
    [dir=rtl] .confirm-message{
        right: auto;
        left: 35px;
    }

    .confirm-message .text-danger {
        color: #ee3f22;
    }

    .closebtn {
        /*color: #B7CAD6;*/
    }

    .rounded-l {
        border-top-left-radius: 0.25rem; /* 4px */
        border-bottom-left-radius: 0.25rem; /* 4px */
    }

    .rounded-t-lg {
        border-top-right-radius: 0.5rem; /* 8px */
        border-top-left-radius: 0.5rem; /* 8px */
    }

    .rounded-b-lg {
        border-bottom-left-radius: 0.5rem; /* 8px */
        border-bottom-right-radius: 0.5rem; /* 8px */
    }

    .box-content {
        box-sizing: content-box;
    }

    .grow {
        flex-grow: 1;
    }

    .grow-0 {
        flex-grow: 0;
    }

    .shrink {
        flex-shrink: 1;
    }

    .shrink-0 {
        flex-shrink: 0;
    }
</style>
