<template>
    <div
        :class="classes"
        @mouseenter="handleMouseenter"
        @mouseleave="handleMouseleave"
        v-clickoutside="handleClose">
        <div
            :class="[prefixCls + '-rel']"
            v-el:reference
            @click="handleClick"
            @mousedown="handleFocus(false)"
            @mouseup="handleBlur(false)">
            <slot></slot>
        </div>
        <div :class="[prefixCls + '-popper']" :style="styles" transition="fade" v-el:popper v-show="visible">
            <div :class="[prefixCls + '-content']">
                <div :class="[prefixCls + '-arrow']"></div>
                <div :class="[prefixCls + '-inner']" v-if="confirm">
                    <div :class="[prefixCls + '-body']">
                        <i class="ivu-icon ivu-icon-help-circled"></i>
                        <div :class="[prefixCls + '-body-message']"><slot name="title">{{ title }}</slot></div>
                    </div>
                    <div :class="[prefixCls + '-footer']">
                        <i-button type="text" size="small" @click="cancel">{{ cancelText }}</i-button>
                        <i-button type="primary" size="small" @click="ok">{{ okText }}</i-button>
                    </div>
                </div>
                <div :class="[prefixCls + '-inner']" v-if="!confirm">
                    <div :class="[prefixCls + '-title']" v-if="showTitle" v-el:title><slot name="title"><div :class="[prefixCls + '-title-inner']">{{ title }}</div></slot></div>
                    <div :class="[prefixCls + '-body']">
                        <div :class="[prefixCls + '-body-content']"><slot name="content"><div :class="[prefixCls + '-body-content-inner']">{{ content }}</div></slot></div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    import Popper from '../base/popper';
    import iButton from '../button/button.vue';
    import clickoutside from '../../directives/clickoutside';
    import { oneOf } from '../../utils/assist';
    import { t } from '../../locale';

    const prefixCls = 'ivu-poptip';

    export default {
        mixins: [Popper],
        directives: { clickoutside },
        components: { iButton },
        props: {
            trigger: {
                validator (value) {
                    return oneOf(value, ['click', 'focus', 'hover']);
                },
                default: 'click'
            },
            placement: {
                validator (value) {
                    return oneOf(value, ['top', 'top-start', 'top-end', 'bottom', 'bottom-start', 'bottom-end', 'left', 'left-start', 'left-end', 'right', 'right-start', 'right-end']);
                },
                default: 'top'
            },
            title: {
                type: [String, Number]
            },
            content: {
                type: [String, Number],
                default: ''
            },
            width: {
                type: [String, Number]
            },
            confirm: {
                type: Boolean,
                default: false
            },
            okText: {
                type: String,
                default () {
                    return t('i.poptip.okText');
                }
            },
            cancelText: {
                type: String,
                default () {
                    return t('i.poptip.cancelText');
                }
            }
        },
        data () {
            return {
                prefixCls: prefixCls,
                showTitle: true,
                isInput: false
            };
        },
        computed: {
            classes () {
                return [
                    `${prefixCls}`,
                    {
                        [`${prefixCls}-confirm`]: this.confirm
                    }
                ];
            },
            styles () {
                let style = {};

                if (this.width) {
                    style.width = `${this.width}px`;
                }
                return style;
            }
        },
        methods: {
            handleClick () {
                if (this.confirm) {
                    this.visible = !this.visible;
                    return true;
                }
                if (this.trigger !== 'click') {
                    return false;
                }
                this.visible = !this.visible;
            },
            handleClose () {
                if (this.confirm) {
                    this.visible = false;
                    return true;
                }
                if (this.trigger !== 'click') {
                    return false;
                }
                this.visible = false;
            },
            handleFocus (fromInput = true) {
                if (this.trigger !== 'focus' || this.confirm || (this.isInput && !fromInput)) {
                    return false;
                }
                this.visible = true;
            },
            handleBlur (fromInput = true) {
                if (this.trigger !== 'focus' || this.confirm || (this.isInput && !fromInput)) {
                    return false;
                }
                this.visible = false;
            },
            handleMouseenter () {
                if (this.trigger !== 'hover' || this.confirm) {
                    return false;
                }
                this.visible = true;
            },
            handleMouseleave () {
                if (this.trigger !== 'hover' || this.confirm) {
                    return false;
                }
                this.visible = false;
            },
            cancel () {
                this.visible = false;
                this.$emit('on-cancel');
            },
            ok () {
                this.visible = false;
                this.$emit('on-ok');
            },
            getInputChildren () {
                const $input = this.$els.reference.querySelectorAll('input');
                const $textarea = this.$els.reference.querySelectorAll('textarea');
                let $children = null;

                if ($input.length) {
                    $children = $input[0];
                } else if ($textarea.length) {
                    $children = $textarea[0];
                }

                return $children;
            }
        },
        compiled () {
            if (!this.confirm) {
                this.showTitle = this.$els.title.innerHTML != `<div class="${prefixCls}-title-inner"></div>`;
            }
            // if trigger and children is input or textarea,listen focus & blur event
            if (this.trigger === 'focus') {
                const $children = this.getInputChildren();
                if ($children) {
                    $children.addEventListener('focus', this.handleFocus, false);
                    $children.addEventListener('blur', this.handleBlur, false);
                    this.isInput = true;
                }
            }
        },
        beforeDestroy () {
            const $children = this.getInputChildren();
            if ($children) {
                $children.removeEventListener('focus', this.handleFocus, false);
                $children.removeEventListener('blur', this.handleBlur, false);
            }
        }
    };
</script>
