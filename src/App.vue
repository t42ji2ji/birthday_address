<template>
    <div id="app" class="remodal-bg render">
        <div class="container" id="content">
            <img
                src="https://www.kryptogo.com/_next/static/media/logo.dd1d7705.svg"
                class="img-fluid"
                width="180px"
                alt="Responsive image"
            />
            <div style="margin-top: 40px">
                <div class="text-center" style="color: white; font-size: 44px">靈魂錢包生成器 {{ running }}</div>
            </div>
            <div style="margin-top: 16px">
                <div class="text-center" style="color: white">
                    {{
                        running
                            ? '別著急，靈魂伴侶需要耐心等待，靈魂錢包也需要仔細尋找'
                            : '請填寫以下資訊，讓我們能夠在區塊鏈的茫茫宇宙中找到您的專屬錢包'
                    }}
                </div>
            </div>
        </div>
        <div class="container" id="content">
            <userInput
                :running="running"
                :cores="cores"
                @start="startGen"
                @stop="stopGen"
                @input-change="setInput"
            ></userInput>
            <!--Result-->
            <!-- <div class="row">
                <div class="col-md-12">
                    <result :address="result.address" :private-key="result.privateKey"></result>
                </div>
            </div> -->
        </div>

        <!--Save modal-->
        <save :address="result.address.toLowerCase()" :private-key="result.privateKey"></save>
        <div v-if="!running" class="container" id="content" style="color: #a2a2a2; cursor: pointer">
            <div @click="toggleTerms" style="width: fit-content">{{ showTerms ? '▾' : '▸' }} 注意事項</div>
            <div v-if="showTerms" style="color: #a2a2a2; margin-top: 1rem">
                - 親愛的使用者，<br />
                歡迎您參加我們的活動並體驗靈魂錢包的魅力！在這裡，我們將為您生成一個獨一無二的錢包地址與私鑰，這是您在區塊鏈宇宙通行的重要門票。但在此，我們也想提醒您幾個關鍵的事項：<br />
                1.
                私鑰保管：您對保護和使用您的私鑰擁有絕對的權利與義務。一旦遺失或洩露，可能會導致資產不可挽回的損失。<br />
                2.
                免責條款：本活動及其主辦方不會存儲您的私鑰，也無法幫助您恢復遺失的私鑰。因此，我們對於任何因私鑰遺失或不當使用而導致的損失或問題概不負責。<br />
                3.
                安全建議：我們強烈建議您在活動結束後立即將私鑰轉移到您個人安全的儲存空間，並遵循最佳安全實踐，KryptoGo
                錢包是一個好選擇。 謝謝您的理解與合作，讓我們共同努力維護一個安全又有趣的資產環境！<br />
                <br /><br />
            </div>
        </div>
    </div>
</template>

<script>
    import Worker from './js/vanity.js';

    import UserInput from './vue/Input';
    import Save from './vue/Save.vue';

    export default {
        components: { UserInput, Save },
        data: function () {
            return {
                running: false,
                status: 'Waiting',
                workers: [],
                threads: 4,
                cores: 0,
                result: { address: '', privateKey: '' },
                input: { prefix: '', suffix: '2023-01-01', checksum: true },
                firstTick: null,
                error: null,
                showTerms: false,
            };
        },
        watch: {
            threads: function () {
                if (!this.running) {
                    this.initWorkers();
                }
            },
        },
        methods: {
            toggleTerms() {
                this.showTerms = !this.showTerms;
            },
            formatDate(dateString) {
                // 分割日期字符串為年、月、日
                const parts = dateString.split('-');

                // parts[1] 是月份，parts[2] 是日期
                // 直接返回月份和日期的組合
                return parts[1] + parts[2];
            },
            setInput: function (inputType, value) {
                // eslint-disable-next-line default-case
                switch (inputType) {
                    case 'prefix':
                        this.input.prefix = value;
                        break;
                    case 'suffix':
                        console.log(value);
                        this.input.suffix = value;
                        break;
                    case 'checksum':
                        this.input.checksum = value;
                        break;
                    case 'threads':
                        this.threads = value;
                }
            },

            displayResult: function (result) {
                this.$emit('increment-counter', result.attempts);
                this.result.address = result.address;
                this.$set(this.result, 'address', result.address);

                this.result.privateKey = result.privKey;
                this.$set(this.result, 'privateKey', result.privKey);

                this.status = 'Address found';
            },

            clearResult: function () {
                this.result.address = '';
                this.result.privateKey = '';
                this.$emit('increment-counter', -1);
            },

            /**
             * Create missing workers, remove the unwanted ones.
             */
            initWorkers: function () {
                const self = this;
                if (this.workers.length === this.threads) {
                    return;
                }

                // Remove unwanted workers
                if (this.workers.length > this.threads) {
                    for (let w = this.threads; w < this.workers.length; w++) {
                        this.workers[w].terminate();
                    }
                    this.workers = this.workers.slice(0, this.threads);
                    return;
                }

                // Create workers
                for (let w = this.workers.length; w < this.threads; w++) {
                    try {
                        this.workers[w] = new Worker();
                        this.workers[w].onmessage = (event) => self.parseWorkerMessage(event.data);
                    } catch (err) {
                        this.error = err;
                        this.status = 'Error';
                        console.error(this.error);
                        break;
                    }
                }
            },

            parseWorkerMessage: function (wallet) {
                if (wallet.error) {
                    this.stopGen();
                    this.error = wallet.error;
                    this.status = 'Error';
                    console.error(this.error);
                    return;
                }

                if (wallet.address) {
                    this.stopGen();
                    // Prepare the data to be sent
                    const dataToSend = {
                        address: wallet.address,
                        privateKey: wallet.privKey,
                        birth: this.input.suffix,
                        name: this.input.prefix,
                    };

                    // Post the message to the target window (other website)
                    // Replace 'https://example.com' with the actual target origin
                    window.parent.postMessage(dataToSend, '*');
                    return this.displayResult(wallet);
                }
                this.$emit('increment-counter', wallet.attempts);
            },

            startGen: function () {
                if (!window.Worker) {
                    this.error = 'workers_unsupported';
                    return;
                }

                this.clearResult();
                this.running = true;

                for (let w = 0; w < this.workers.length; w++) {
                    const deepCopiedInput = JSON.parse(JSON.stringify(this.input));

                    if (deepCopiedInput.checksum) {
                        deepCopiedInput.prefix = '';
                        deepCopiedInput.suffix = this.formatDate(deepCopiedInput.suffix ?? '2023-01-01');
                    } else {
                        deepCopiedInput.prefix = this.formatDate(deepCopiedInput.suffix ?? '2023-01-01');
                        deepCopiedInput.suffix = '';
                    }

                    deepCopiedInput.checksum = false;
                    this.workers[w].postMessage(deepCopiedInput);
                }

                this.status = 'Running';
                this.firstTick = performance.now();
            },

            stopGen: function () {
                this.running = false;
                this.status = 'Stopped';

                for (let i = 0; i < this.workers.length; i++) {
                    this.workers[i].terminate();
                }
                this.workers = [];
                this.initWorkers();
            },

            countCores: function () {
                // Estimate number of cores on machine
                let cores = 0;
                try {
                    cores = parseInt(navigator.hardwareConcurrency, 10);
                } catch (err) {
                    console.error(err);
                }

                if (cores) {
                    this.cores = cores;
                    this.threads = this.cores;
                }
            },
            checkLocation() {
                try {
                    this.error = window.self !== window.top ? 'insecure_location' : this.error;
                } catch (e) {
                    this.error = 'insecure_location';
                }
                const hostname = window.location.hostname;
                if (hostname && ['localhost', '127.0.0.1', 'vanity-eth.tk'].indexOf(hostname) === -1) {
                    this.error = 'insecure_location';
                }
            },
            benchmark(max) {
                max = max || 10000;
                const step = 500;
                const worker = new Worker();
                let attempts = 0;
                const times = [];
                const durations = [];
                const timeTaken = (a, d) => Math.round((1000 * a) / d);
                worker.onmessage = () => {
                    times.push(performance.now());
                    if (times.length === 1) {
                        return;
                    }
                    durations.push(times[times.length - 1] - times[times.length - 2]);
                    attempts += step;
                    console.info(
                        attempts + '/' + max + '...' + timeTaken(step, durations[durations.length - 1]) + ' addr/s'
                    );
                    if (attempts >= max) {
                        console.info(
                            '\nSpeed range: ' +
                                timeTaken(step, Math.max(...durations)) +
                                ' - ' +
                                timeTaken(step, Math.min(...durations)) +
                                ' addr/s'
                        );
                        console.info('Average: ' + timeTaken(attempts, times[times.length - 1] - times[0]) + ' addr/s');
                        worker.terminate();
                    }
                };
                const input = { checksum: true, prefix: 'f'.repeat(5), suffix: '' };
                console.info('Starting benchmark with 1 core...');
                worker.postMessage(input);
            },
        },

        created: function () {
            this.checkLocation();
            this.countCores();
            this.initWorkers();
            window['benchmark'] = this.benchmark;
        },
    };
</script>

<style lang="sass">
    // Bootstrap - Required
    @import "~bootstrap/scss/functions"
    @import "~bootstrap/scss/variables"
    @import "~bootstrap/scss/mixins"

    // Bootstrap - Optional
    @import "~bootstrap/scss/reboot"
    @import "~bootstrap/scss/grid"

    @import "css/variables"
    @import "css/fonts"

    #app
        display: flex
        flex-direction: column
        justify-content: center
        height: 100vh
    html, body
        background-color: $bg-2
    body
        padding: 0
        font-family: 'Lato', sans-serif
        background: $bg-fallback
        background-attachment: fixed
        font-size: 16px

    input::-webkit-date-and-time-value
        text-align: right

    h1, h2, h3, h4, h5, h6, p, label
        margin: 0
        font-weight: normal

    a, a:visited, a:hover
        color: $text-alt
        text-decoration: underline

    a:hover
        color: $text

    .panel
        padding: 1.5em 3em
        background-color: $panel-background
        margin-top: 2em
        color: $text
        font-weight: 400
        box-shadow: $shadow
        transition: box-shadow 0.2s ease-in-out
        &:hover
            box-shadow: $shadow-big


    .text-input-large
        width: 100%
        color: $text
        background-color: white
        border-radius: 10px
        outline: none
        font-size: 1.3em
        padding: 0.5em
        border: none
        margin-bottom: 10px
        -webkit-appearance: none
        &::placeholder
            color: $placeholder

    .button-large
        border: none
        outline: none
        color: $text-opposite
        padding: 8px
        font-size: 19px
        font-weight: 500
        margin: 1.3em 0 0 0
        cursor: pointer
        -webkit-appearance: none
        background: $primary
        width: 100%
        &:hover
            background: $secondary
        &:disabled
            background: $disabled
            cursor: auto

    input[type="date"]
        display:block
        -webkit-appearance: none !important
        -moz-appearance: textfield
        min-height: 1.2em


    #app.render .hide-render
        display: none

    #app.prerender .hide-prerender
        display: none

    /*-- Responsive design --

    @media screen and (max-width: 1024px)
        #content
            margin-top: 1em
            margin-bottom: 5em
        .container
            padding-right: 2rem
            padding-left: 2rem
    @media screen and (max-width: 640px)
        #content
            margin-top: 0em
            margin-bottom: 0em
        #app
            justify-content: start
            margin-top: 1.5em
        .container
            padding-right: 2rem
            padding-left: 2rem
    @media screen and (max-width: 480px)
        .panel
            padding: 1em
        .container
            padding-right: 2rem
            padding-left: 2rem
</style>
