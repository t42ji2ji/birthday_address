<template>
    <div class="panel result">
        <div class="row">
            <div class="float-left" id="identicon"></div>
            <div class="col">
                <div>Address: <span class="output" v-text="address"></span></div>
                <div>
                    Private key:
                    <span
                        class="output"
                        v-if="privateKey"
                        v-text="reveal ? privateKey : 'Click to reveal'"
                        @click="revealKey()"
                    ></span>
                </div>
            </div>
            <div class="col-lg-2 col-12">
                <button class="save button-large" :disabled="!privateKey" @click="toggleQrcode">
                    <i class="icon-lock"></i>Show Qrcode
                </button>
            </div>
            <div style="margin-top: 20px" v-if="privateKey && showQrcode">
                <qrcode-vue :value="privateKey" size="200" level="H" />
            </div>
            <!-- <qrcode-vue value="value" :size="size" level="H" /> -->
        </div>
    </div>
</template>

<script>
    import * as blockies from 'blockies';
    import QrcodeVue from 'qrcode.vue';

    export default {
        components: { QrcodeVue },
        props: {
            address: String,
            privateKey: String,
        },
        data: function () {
            return {
                reveal: false,
                size: 128,
                showQrcode: false,
            };
        },
        watch: {
            address(addr) {
                this.reveal = false;
                const id = document.getElementById('identicon');
                id.innerHTML = '';
                if (addr) {
                    id.appendChild(blockies({ seed: addr.toLocaleLowerCase(), scale: 6 }));
                }
            },
        },
        methods: {
            toggleQrcode() {
                this.showQrcode = !this.showQrcode;
            },
            revealKey() {
                this.reveal = true;
            },
        },
    };
</script>

<style lang="sass" scoped>
    @import "../css/variables"
    #identicon
        width: 48px
        height: 48px
        margin: 0 15px
        background-color: $panel-background-alt

    .output
        font-family: $monospace-font
        color: $text-alt
        margin-left: 15px
        word-break: break-all
        font-size: 15px

    .panel > div:not(:last-child)
        margin-bottom: 15px

    .save
        margin-top: 30px
        i
          margin-right: 8px
          top: 2px
          position: relative

    @media screen and (min-width: 992px)
        .save
            margin-top: 0
</style>
