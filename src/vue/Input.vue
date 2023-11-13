<template>
    <div id="input-panel">
        <div v-if="running">
            <div class="spinner">
                <div></div>
                <div></div>
                <div></div>
                <div></div>
            </div>
        </div>
        <form @submit.prevent="startGen" v-if="!running">
            <!-- <div class="error-text" v-if="inputError">Numbers and letters from A to F only</div> -->
            <div class="row">
                <div class="col-12 col-sm-6 col-md-12 col-lg-6">
                    <div style="color: white; margin-bottom: 8px">姓名</div>
                    <input
                        type="text"
                        class="text-input-large"
                        id="input"
                        placeholder="請輸入你的名字（10字以內)"
                        :disabled="running"
                        v-model="prefix"
                        maxlength="10"
                    />
                </div>
                <div class="col-12 col-sm-6 col-md-12 col-lg-6" :class="{ inputColor: !suffixError }">
                    <div style="color: white; margin-bottom: 8px">生日</div>
                    <input type="date" class="text-input-large" id="input" v-model="suffix" :disabled="running" />
                </div>
            </div>

            <div class="controls hide-prerender">
                <label class="checkbox" style="color: white">
                    <input type="checkbox" name="checkbox" checked="" v-model="checksum" :disabled="running" />
                    <i class="left"> </i>
                    生成在結尾
                </label>
            </div>
            <!-- <div class="threads hide-prerender">
                <input
                    type="button"
                    class="square-btn button-large"
                    value="-"
                    @click="threads--"
                    :disabled="running || threads <= 1"
                />
                <input
                    type="button"
                    class="square-btn arrow button-large"
                    value="+"
                    @click="threads++"
                    :disabled="running"
                />
                <h4 v-text="threads"></h4>
                <span>&nbsp;threads</span>
                <span v-if="threads === cores"> (recommended)</span>
            </div> -->
        </form>
        <div class="row" style="justify-content: space-between; padding: 0px 15px; align-items: center">
            <div class="row" style="margin-left: 0px; cursor: pointer">
                <div @click="stopGen" :disabled="!running" style="color: #a2a2a2">
                    {{ running ? '← Back' : 'X&nbsp;&nbsp;&nbsp;&nbsp;Clear' }}
                </div>
            </div>
            <div
                class="gen-btn row button-large"
                :class="{ disabledButton: running || inputError || error }"
                v-show="!running"
            >
                <div>
                    <input type="button" value="Generate" class="hide-render" disabled />
                    <div @click="startGen">生成</div>
                </div>
                <div v-if="!(running || inputError || error)" style="display: flex; align-items: center">
                    <img :src="require('@/assets/images/arrow.png')" width="16px" height="16px" />
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    function mixCase(str) {
        let ret = '';
        for (let i = 0; i < str.length; i++) {
            const l = str.substr(i, 1);
            ret += Math.random() < 0.5 ? l.toUpperCase() : l.toLowerCase();
        }
        return ret;
    }
    export default {
        props: {
            running: Boolean,
            cores: Number,
        },
        data: function () {
            return {
                threads: this.$props.cores || 4,
                prefix: '',
                suffix: '',
                checksum: false,
                error: false,
            };
        },
        computed: {
            prefixError: function () {
                return !this.prefix;
            },
            suffixError: function () {
                return !this.suffix;
            },
            inputError: function () {
                return this.prefixError || this.suffixError;
            },
            example: function () {
                if (this.inputError) {
                    return null;
                }
                const prefix = this.checksum ? this.prefix : mixCase(this.prefix);
                const suffix = this.checksum ? this.suffix : mixCase(this.suffix);
                let random = '';
                for (let i = 0; i < 40 - this.prefix.length - this.suffix.length; i++) {
                    random += mixCase(Math.floor(Math.random() * 16).toString(16));
                }
                return { random, prefix, suffix };
            },
        },
        methods: {
            startGen: function () {
                if (!this.running && !this.inputError && !this.error) {
                    this.$emit('start');
                }
            },
            stopGen: function () {
                if (this.running) {
                    this.$emit('stop');
                } else {
                    this.prefix = '';
                    this.suffix = '';
                    this.checksum = false;
                    this.error = false;
                }
            },
        },
        watch: {
            prefix: function () {
                this.$emit('input-change', 'prefix', this.prefix);
            },
            suffix: function () {
                this.$emit('input-change', 'suffix', this.suffix);
            },
            checksum: function () {
                this.$emit('input-change', 'checksum', this.checksum);
            },
            // threads: function () {
            //     this.$emit('input-change', 'threads', this.threads);
            // },
        },
    };
</script>

<style lang="sass" scoped>
    @import "../css/variables"
    .panel
        min-height: 280px


    .gen-btn
        border-radius: 12px
        width: fit-content
        color: rgba(27, 37, 89, 1)
        padding: 16px 24px
        margin: 0 !important
    .button-large:disabled
        color: #666666
        background-color: none
    .disabledButton
        background-color: #989898
        color: #C7C7C7

    input[type="button"]
        background-color: none
        border: none
    input[type="date"]
        // WebKit-based browsers
        &::-webkit-datetime-edit-text,
        &::-webkit-datetime-edit-month-field,
        &::-webkit-datetime-edit-day-field,
        &::-webkit-datetime-edit-year-field
            color: gray
    .inputColor
        input[type="date"]
            // WebKit-based browsers
            &::-webkit-datetime-edit-text,
            &::-webkit-datetime-edit-month-field,
            &::-webkit-datetime-edit-day-field,
            &::-webkit-datetime-edit-year-field
                color: $text !important
    .error-text
        font-size: 14px
        color: $error

    input.error
        border: 1px solid $error

    .example
        font-size: 14px
        word-break: break-all
        color: $text-alt
        b
            color: $text
        .monospace
            font-family: $monospace-font
    .controls
        margin: 12px 0
        > div
            padding: 5px 0

        .checkbox
            margin-bottom: 4px
            padding-left: 30px
            line-height: 27px
            cursor: pointer
            position: relative
            color: $text
            font-weight: 400
            &:last-child
                margin-bottom: 0
            i
                position: absolute
                bottom: 4px
                left: 17.5em
                display: block
                width: 19px
                height: 19px
                outline: none
                border: 1px solid $border-grey
                &.left
                    position: absolute
                    bottom: 4px
                    left: 0
                    display: block
                    width: 19px
                    height: 19px
                    outline: none
                    border: 1px solid $border-grey
            input
                + i:after
                    content: ''
                    background: url("../assets/images/tick-mark.png") no-repeat
                    top: 4px
                    left: 3px
                    width: 15px
                    height: 15px
                    position: absolute
                    opacity: 0
                position: absolute
                left: -9999px
                &:checked + i:after
                    opacity: 1

        .switch
            position: relative
            width: 40px
            height: 24px
            margin: 0 5px
            input
                visibility: hidden

        .slider
            position: absolute
            cursor: pointer
            top: 0
            left: 0
            right: 0
            bottom: 0
            background-color: $primary
            transition: .2s
            &:before
                position: absolute
                content: ""
                height: 16px
                width: 16px
                left: 4px
                bottom: 4px
                background-color: white
                transition: .2s

    input
        &:checked + .slider
            background-color: $primary
        &:focus + .slider
            box-shadow: 0 0 1px $primary
        &:checked + .slider:before
            transform: translateX(16px)

    .threads
        h4
            display: inline
        input[type=button].square-btn
            display: inline-block
            width: 24px
            height: 24px
            margin: 0 5px 2px 0
            padding: 0
            line-height: 1em

    .justify-content-center
        justify-content: center

    .spinner
        width: 61px
        height: 61px
        margin: 18px
        & > div
            position: absolute
            width: 61px
            height: 61px
            margin: 5px
            border: 3px solid $primary
            border-radius: 50%
            animation: lds-ring 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite
            border-color: $primary transparent transparent transparent
            &:nth-child(1)
                animation-delay: -0.45s
            &:nth-child(2)
                animation-delay: -0.3s
            &:nth-child(3)
                animation-delay: -0.15s

    @keyframes lds-ring
        0%
            transform: rotate(0deg)
        100%
            transform: rotate(360deg)
</style>
