<template>
        <div class="app" :class="{appDark: isDark}" >
            <div class="rateWrapper">
                <div class="rate">
                    <div class="chng">
                        <div class="firstrate">
                            <div class="rate__btc" :class="{rate__btcDark: isDark}">
                                BTC {{btc}}$
                            </div>
                            <div v-if="btcGrow" class="arrow-up"></div>
                            <div v-else class="arrow-down"></div>
                        </div>
                        <div class="shbutton">
                            <button @click="switchTo" class="shine-button"><div class="sun"></div></button>
                        </div>
                    </div>
                    <div class="rate__btc" :class="{rate__btcDark: isDark}">
                        <div class="rate__btc">
                            <div class="">
                                ETH {{eth}}$
                            </div>
                            <div v-if="ethGrow" class="arrow-up"></div>
                            <div v-else class="arrow-down"></div>
                        </div>
                    </div>
                </div>
            </div>

            <div class="selects">
                <div class="selectsWrapper">
                    <div class="SelectWrapper">
                        <div id="box" class="selectBorder">
                            <select  class="select" @change="changeOptionFirst">
                                <option
                                    v-for="option in options"
                                    :key="option.value"
                                    :value="option.value"
                                >
                                    {{ option.name }}
                                </option>
                            </select>
                        </div>

                        <div class="ratioWrapper">
                            <div class="ratio">
                                {{selectedFirst.ratio.toPrecision(5)}}
                            </div>
                        </div>

                    </div>
                    <div class="SelectWrapper">
                        <div id="box" class="selectBorder">
                            <select class="select" @change="changeOptionSecond">
                                <option
                                    v-for="option in options"
                                    :key="option.value"
                                    :value="option.value"
                                >
                                    {{ option.name }}
                                </option>
                            </select>
                        </div>
                        <div class="ratioWrapper">
                            <div class="ratio">
                                {{selectedSecond.ratio.toPrecision(5)}}
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="chartView">
                <div class="chartViewWrapper">
                    <div class="btns">
                        <div class="btnWrapper">
                            <button class="btn" :class="{btnActive: btcChart}" @click="changeChartToBtc">BTC</button>
                            <button class="btn eths" :class="{btnActive: !btcChart}" @click="changeChartToEth">ETH</button>

                        </div>
                    </div>
                    <div class="text">Select a graph of exchange rate changes for the last 14 days</div>
                    <div class="charts">
                        <div v-if="btcChart" class="">
                            <div class="chart">
                                <Bar
                                    :chart-options="chartOptions"
                                    :chart-data="chartDataBtc"
                                />
                            </div>
                        </div>
                        <div v-else class="">
                            <div class="chart">
                                <Bar
                                    :chart-options="chartOptions"
                                    :chart-data="chartDataEth"
                                />
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
</template>

<script >
import axios from 'axios';
import { Bar } from 'vue-chartjs';
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js'

ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale)

export default {
    components: { Bar },
    data() {
        return {
            cryptos: [],
            exchBtc: [],
            isDark: false,
            btcGrow: true,
            ethGrow: true,
            btcChart: true,
            btnClass: 'btn',
            activeBtnClass: 'btnActive',
            dateExchBtc: [],
            valExchBtc: [],
            exchEth: [],
            dateExchEth: [],
            valExchEth: [],
            chartDataBtc: {
                labels: [], //date
                datasets: [ { label: 'BTC', backgroundColor: ['#F37055FF'], data: [] } ] //value
            },
            chartDataEth: {
                labels: [], //date
                datasets: [ { label: 'ETH', backgroundColor: ['#1edea5'], data: [] } ] //value
            },
            chartOptions: {
                responsive: true
            },
            btc: 0,
            eth: 0,
            usd: 1,
            selectedFirst: { // selectedFirst.value
                ratio: 1,
                value: 0
            },
            selectedSecond: { //selectedSecond.value
                ratio: 1,
                value: 0
            },
            options: [
                {value: 'btc', name: 'BTC'},
                {value: 'eth', name: 'ETH'},
                {value: 'usd', name: 'USD'},
            ],
        };
    },

    methods: {
        changeOptionFirst(event){
            if(event.target.value === 'btc'){
                this.selectedFirst.value = this.btc
            }
            if(event.target.value === 'eth'){
                this.selectedFirst.value = this.eth
            }
            if(event.target.value === 'usd'){
                this.selectedFirst.value = this.usd
            }
            this.selectedSecond.ratio = this.selectedFirst.value / this.selectedSecond.value
        },
        changeOptionSecond(event){
            if(event.target.value === 'btc'){
                this.selectedSecond.value = this.btc
            }
            if(event.target.value === 'eth'){
                this.selectedSecond.value = this.eth
            }
            if(event.target.value === 'usd'){
                this.selectedSecond.value = this.usd
            }
            this.selectedSecond.ratio = this.selectedFirst.value / this.selectedSecond.value;
        },
        changeChartToEth(){
            this.btcChart = false
        },
        changeChartToBtc(){
            this.btcChart = true
        },
        switchTo(){
            this.isDark = !this.isDark
        }
        
    },

    async mounted() {
        axios
            .get('https://api.coingecko.com/api/v3/simple/price?ids=bitcoin,ethereum,usd&vs_currencies=usd')
            .then(response => {
                this.cryptos = response.data;
                this.btc = this.cryptos.bitcoin.usd;
                this.eth = this.cryptos.ethereum.usd;
                this.selectedFirst.value = this.btc;
                this.selectedSecond.value = this.btc;
            })
            .catch(error => {
                console.log(error);
            });
        axios
            .get('https://api.coingecko.com/api/v3/coins/bitcoin/market_chart?vs_currency=usd&days=14&interval=daily')
            .then(response => {
                    this.exchBtc = response.data.prices.flat();
                    for(let i = 0; i < this.exchBtc.length; i++){
                        if(i % 2){
                            this.valExchBtc.push(this.exchBtc[i]) //value
                        }
                        else{
                            this.dateExchBtc.push( String(new Date(this.exchBtc[i])).slice(0, 10) ); //date

                        }
                    }
                    this.chartDataBtc.labels = this.dateExchBtc
                    this.chartDataBtc.datasets[0].data = this.valExchBtc
                    if(this.valExchBtc[13] > this.btc){
                        this.btcGrow = false
                    }
            })
            .catch(error => {
                console.log(error);
            });
        axios
            .get('https://api.coingecko.com/api/v3/coins/ethereum/market_chart?vs_currency=usd&days=14&interval=daily')
            .then(response => {
                this.exchEth = response.data.prices.flat();
                for(let i = 0; i < this.exchEth.length; i++){
                    if(i % 2){
                        this.valExchEth.push(this.exchEth[i]) //value
                    }
                    else{
                        this.dateExchEth.push( String(new Date(this.exchEth[i])).slice(0, 10) ); //date

                    }
                }
                this.chartDataEth.labels = this.dateExchEth
                this.chartDataEth.datasets[0].data = this.valExchEth
                if(this.valExchBtc[13] > this.btc){
                    this.ethGrow = false
                }
            })
            .catch(error => {
                console.log(error);
            });

            
    },
}
</script>

<style>
    body{
    }
    * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;

    }
    .app{
        padding: 15px;
        background-color: #efeff3;
        overflow: scroll;
        min-height: 100vh;
        transition: .2s ease-in-out;
    }
    .appDark{
        padding: 15px;
        background-color: #303034;
        min-height: 100vh;
        overflow: scroll;
        transition: .2s ease-in-out;
    }
    .shbutton{
        display: flex;
        justify-content: right;
    }
    .shine-button {
        display: inline-block;
        padding: 10px 15px;
        overflow: hidden;
        top: 3vh;
        right: 10vh;
        position: absolute;
        border: 2px solid #59599b;
        border-radius: 8px;
        font-family: 'Montserrat', sans-serif;
        font-size: 25px;
        color: #000000;
        background-color: #303034;
        transition: .2s ease-in-out;
    }
    .sun::after{
        content: '\1F323';
        color: white;
    }
    .rateWrapper{
        display: flex;
        justify-content: center;
    }
    .rate{
        margin-left: 20px;
        margin-top: 20px;
        margin-bottom: 20px;
        font-size: 150%;

    }
    .rate__btc{
        margin-bottom: 10px;
        display: flex;
    }
    .rate__btcDark{
        color: white;
    }
    .firstrate{
        display: flex;
    }
    .chng{
        display: flex;
        justify-content: space-between;

    }
    .arrow-up::after{
        content: "\2191";
        color: #0fbb0f;
        margin-left: 10px;
    }
    .arrow-down::after{
        content: "\2193";
        color: #ff0000;
        margin-left: 10px;
    }

    .selects{
        display: flex;
        justify-content: space-around;
        align-items: center;
    }
    .selectsWrapper{
        display: flex;
        justify-content: center;
        margin-bottom: 5%;

    }
    .SelectWrapper{
        display: flex;
        margin-left: 30px;
    }
    .SelectWrapper:nth-child(1){
        margin-left: 0;
    }
    .select{
        width: 100%;
        height: 100%;
        text-align: center;
        border: none;
        font-size: 25px;
    }
    #box {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 11ex;
        height: 100%;
    }
    .selectBorder {
        --borderWidth: 2px;
        background: #1D1F20;
        position: relative;
        z-index: 99;
        border-radius: var(--borderWidth);
    }
    .selectBorder::after {
        content: '';
        position: absolute;
        top: calc(-1 * var(--borderWidth));
        left: calc(-1 * var(--borderWidth));
        height: calc(100% + var(--borderWidth) * 2);
        width: calc(100% + var(--borderWidth) * 2);
        background: linear-gradient(60deg, #f79533, #f37055, #ef4e7b, #a166ab, #5073b8, #1098ad, #07b39b, #6fba82);
        box-shadow: 0 0 3px #ef4e7b;
        border-radius: calc(2 * var(--borderWidth));
        z-index: -1;
        animation: animatedgradient 3s ease alternate infinite;
        background-size: 300% 300%;
    }


    @keyframes animatedgradient {
        0% {
            background-position: 0% 50%;
        }
        50% {
            background-position: 100% 50%;
        }
        100% {
            background-position: 0% 50%;
        }
    }
    .ratio{
        display: inline-block;
        margin-left: 10px;
        border: 1px solid #1098ad;
        box-shadow: 0 0 3px #1098ad;
        background-color: #fff;
        text-align: center;
        font-size: 25px;
        min-width: 5ex;
        height: 100%;

    }
    .btns{
        display: flex;
        justify-content: center;
        align-items: center;
        width: 100%;
        margin-bottom: 2%;
    }
    .btnWrapper{

    }
    .btn{
        text-decoration: none;
        display: inline-block;
        
        position: relative;
        color: white;
        border: none;
        -webkit-border-radius: 5px;
        -moz-border-radius: 5px;
        border-radius: 5px;
        background-color: #F37055FF;
        font-weight: 500;
        font-family: 'Montserrat', sans-serif;
        text-align: center;
        letter-spacing: 2px;
        font-size: 20px;
        height: 50px;
        width: 60px;
    }
    .eths{
        background-color: #1edea5;
        margin-left: 4px;
    }
    .btnActive{
        height: 50px;
        width: 60px;
        background-color: #ab3f29;
    }
    .btnActive:nth-child(2){
        background-color: #199170;
    }
    .text{
        text-align: center;
        font-family: 'Montserrat', sans-serif;
        color: #00d013;
        margin-bottom: 5%;
    }
    .charts{
        display: flex;
        justify-content: center;
    }
    .chart{
        width: 34em;
        height: 17em;

    }
    @media (max-width: 550px) {
        .app{
            overflow: scroll;
        }
        .appDark{
            overflow: scroll;
        }
        .shine-button{
            right: 5px;
            top: 5px;
        }
        .selectsWrapper{
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }
        .SelectWrapper{
            display: flex;
            margin-bottom: 15px;
            margin-left: 0;
        }
        .chart{
            width: 20em;
            height: 10em;

        }
    }

</style>