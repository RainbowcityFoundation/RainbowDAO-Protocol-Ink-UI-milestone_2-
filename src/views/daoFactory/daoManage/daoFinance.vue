<template>
  <div class="daoFinance">
    <div class="no-account" style="margin: 20px 60px;font-size: 20px" v-show="!account">
      Please connect first
    </div>
    <div class="opera-btn" v-show="account">
      <div class="my-GCCoin-balance">
        My Governance coin balance:
        <strong>
          {{ myGCBalance? parseInt(myGCBalance.replace(/,/g, '')) / 10 ** parseInt(coinInfo.decimals) : 0 }}
          {{ coinInfo.name }}(decimal:{{coinInfo.decimals}})
        </strong>
      </div>
      <div class="rainbow-button" @click=" allowance(),isShowDeposit=true">
        Deposit
      </div>
      <div class="rainbow-button" @click="isShowWithdraw=true">
        Withdraw
      </div>
    </div>
    <div class="finance-box">
      <div class="left">
        <div class="coin-list">
          <div class="item">
            <img src="../../../assets/daoImgs/title_icon1.png" alt="">
            <div class="coin-data">
              <div class="name">
                Total income
              </div>
              <div class="number">
                {{ parseInt(income) / 10 ** parseInt(coinInfo.decimals) }} {{ coinInfo.name }}
              </div>
            </div>
          </div>
          <div class="item">
            <img src="../../../assets/daoImgs/title_icon2.png" alt="">
            <div class="coin-data">
              <div class="name">
                Used
              </div>
              <div class="number">
                {{ parseInt(expenditure) / 10 ** parseInt(coinInfo.decimals) }} {{ coinInfo.name }}
              </div>
            </div>
          </div>
          <div class="item">
            <img src="../../../assets/daoImgs/title_icon3.png" alt="">
            <div class="coin-data">
              <div class="name">
                Remaining
              </div>
              <div class="number">
                {{ daoGCBalance? parseInt(daoGCBalance.replace(/,/g, '')) / 10 ** parseInt(coinInfo.decimals) :0}} {{ coinInfo.name }}
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="right">
        <div class="table-title" style="line-height: 20px;padding: 0">
          AMOUNT CHANGE
        </div>
        <canvas id='chart' width="1008" height="490" style="width: 1000px!important;"></canvas>
<!--        <div id='chartjs-tooltip'>-->
<!--          <div id='chartjs-tooltip__text'></div>-->
<!--        </div>-->
      </div>
    </div>
    <div class="finance-detail">
      <div class="title">
        Financial details
      </div>
      <div class="financial-detail-list">
        <div class="item" v-for="(item,index) in transferList" :key="index">
          <div class="index">
            {{ item.transferId }}
          </div>
          <div class="name">
            <div class="income" v-if="item.transferDirection=='2'"> {{ item.fromAddress }}</div>
            <div class="expenditure" v-if="item.transferDirection=='1'"> {{ item.toAddress }}</div>
          </div>
          <div class="line"></div>
          <div class="inOrOut">
            <div class="income" v-if="item.transferDirection=='2'">Income</div>
            <div class="expenditure" v-if="item.transferDirection=='1'">Expenditure</div>
          </div>
          <div class="time">
            {{ new Date(item.transferTime.replace(/,/g, '') * 1) }}
          </div>
          <div class="value">
            <div class="income" v-if="item.transferDirection=='2'">
              {{ parseInt(item.value.replace(/,/g, '')) / 10 ** parseInt(coinInfo.decimals) }} {{ coinInfo.symbol }}
            </div>
            <div class="expenditure" v-if="item.transferDirection=='1'">
              {{ parseInt(item.value.replace(/,/g, '')) / 10 ** parseInt(coinInfo.decimals) }} {{ coinInfo.symbol }}
            </div>
          </div>
        </div>
      </div>
    </div>
    <!--    deposit dialog-->
    <div class="rainbow-dialog-box" v-show="isShowDeposit">
      <div class="mask" @click="isShowDeposit=false">
      </div>
      <div class="rainbow-dialog" @click.stop>
        <div class="dialog-title">
          Deposit
        </div>
        <input type="text" v-model="depositNumber">
        <div class="rainbow-button" @click="approve" v-show="allowanceNumber<1000">
          Approve
        </div>
        <div class="rainbow-button" @click="deposit" >
          Submit
        </div>
      </div>
    </div>
    <!--    withdraw dialog-->
    <div class="rainbow-dialog-box" v-show="isShowWithdraw">
      <div class="mask" @click="isShowWithdraw=false">
      </div>
      <div class="rainbow-dialog" @click.stop>
        <div class="dialog-title">
          Withdraw
        </div>
        <input type="text" v-model="withdrawNumber">

        <div class="rainbow-button" @click="withdraw">
          Submit
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {mapGetters} from "vuex";
export default {
  name: "daoFinance",
  components:{

  },
  data() {
    return {
      isShowWithdraw: false,
      isShowDeposit: false,
      transactionList: [{
        address: "",
        value: 10
      }],
      isReady:false,
      allowanceNumber:0,
      income: 0,
      expenditure: 0,
      depositNumber: 0,
      withdrawNumber: 0,
      options: {
        responsive: true
      },
      datacollection: null,
      chartData: []
    }
  },
  computed: {
    ...mapGetters([
      'isConnected',
      'account'
    ]),
    myGCBalance() {
      return this.$store.state.erc20.myGCBalance
    },
    daoGCBalance() {
      return this.$store.state.erc20.daoGCBalance
    },
    transferList() {
      return this.$store.state.daoVault.transferList
    },
    coinInfo() {
      if (this.$store.state.erc20.coinInfo) {
        return this.$store.state.erc20.coinInfo
      } else {
        return {}
      }
    },
    curDaoAddress() {
      return this.$store.state.daoManage.curDaoAddress
    },
    curDaoControlAddress() {
      return this.$store.state.daoManage.curDaoControlAddress
    },
  },
  watch: {
    account() {
      this.getGovernCoinBalance()
    },
    transferList() {
      this.dealTransferList()
    }
  },
  mounted() {
    this.fillData()
    this.getTransferHistory()
    this.dealTransferList()

    setTimeout(()=>{
      this.dealTransferList()
    },5000)
  },
  methods: {
    dealTransferList() {
      this.income = 0, this.expenditure = 0
      this.chartData = []
      let balance = 0
      let arr = JSON.parse(JSON.stringify(this.transferList))
      arr.reverse()
      this.chartData.push(balance)

      arr.forEach(item => {
        if (item.transferDirection == '2') {
          this.income += parseInt(item.value.replace(/,/g, ''))
          balance += parseInt(item.value.replace(/,/g, ''))
        } else {
          this.expenditure += parseInt(item.value.replace(/,/g, ''))
          balance -= parseInt(item.value.replace(/,/g, ''))
        }
        this.chartData.push(balance / 10**this.coinInfo.decimals)
      })

      this.initChart()
    },
    initChart() {
      if(!document.getElementById('chart')){
        return
      }
      let canvas = document.getElementById('chart'),
          ctx = canvas.getContext('2d'),
          grad = ctx.createLinearGradient(0, 0, 0, window.innerHeight)

      // Create a background gradient.
      grad.addColorStop(0, 'rgba(71, 34, 96, .7)')
      grad.addColorStop(.9, 'rgba(44, 51, 233, .7)')
      grad.addColorStop(1, 'rgba(44, 51, 233, .7)')

      // Create a shadow line.
      let shadowLine = Chart.controllers.line.extend({
        initialize: function () {
          Chart.controllers.line.prototype.initialize.apply(this, arguments)

          var ctx = this.chart.ctx
          var originalStroke = ctx.stroke
          ctx.stroke = function () {
            ctx.save()
            ctx.shadowColor = '#1a1426'
            ctx.shadowOffsetY = 5
            ctx.shadowBlur = 15
            originalStroke.apply(this, arguments)
            ctx.restore()
          }
        }
      })
      Chart.controllers.shadowLine = shadowLine
      let labels= [], dummyData = this.chartData;
      dummyData.forEach(number=>{
        labels.push('')

      })
      let chart = new Chart(ctx, {
        type: 'shadowLine',
        data: {
          labels: labels,
          datasets: [{
            label: '',
            backgroundColor: grad,
            borderColor: 'rgb(193, 130, 255)',
            bezierCurve: true,
            pointBackgroundColor: 'rgba(255, 255, 255, 1)',
            pointBorderColor: 'rgba(255, 255, 255, 1)',
            pointRadius: 7,
            pointHoverRadius: 7,
            data: dummyData,
          }]
        },
        options: {
          title: {
            display: false
          },
          layout: {
            padding: {
              left: 50
            }
          },
          legend: {
            labels: {
              boxWidth: 0,
              fontColor: 'rgba(255, 255, 255)',
              fontSize: 16,
              padding: 16
            },
          },
          scales: {
            xAxes: [{
              type: 'category',
              gridLines: {
                color: 'rgba(64, 71, 91, 1)',
                borderDash: [5, 2]
              },
              ticks: {
                padding: 0
              }
            }],
            yAxes: [{
              gridLines: {
                color: 'rgba(64, 71, 91, 1)',
                borderDash: [5, 2]
              },
              ticks: {
                min: Math.min.apply(null,dummyData),
                suggestedMax: Math.max.apply(null, dummyData) + parseInt(this.myGCBalance) / (10 ** this.coinInfo.decimals)/ 10,
                // Include a dollar sign in the ticks
                callback: function (value, index, values) {
                  return value + '     '
                }
              }
            }]
          },
          // Code mostly copied from http://www.chartjs.org/docs/latest/configuration/tooltip.html#external-custom-tooltips
          tooltips: {
            custom: function (tooltipModel) {
              let tooltipEl = document.getElementById('chartjs-tooltip')
              let tooltipElText = tooltipEl.querySelector('#chartjs-tooltip__text')

              // Hide if no tooltip
              if (tooltipModel.opacity === 0) {
                // tooltipEl.style.opacity = 0
                return
              }

              // Set caret Position
              tooltipEl.classList.remove('above', 'below', 'no-transform', 'active')
              if (tooltipModel.yAlign) {
                tooltipEl.classList.add(tooltipModel.yAlign)
              } else {
                tooltipEl.classList.add('no-transform')
              }

              function getBody(bodyItem) {
                return bodyItem.lines
              }

              if (tooltipModel.body) {
                let titleLines = tooltipModel.title || []
                let bodyLines = tooltipModel.body.map(getBody)

                // `this` will be the overall tooltip
                var position = this._chart.canvas.getBoundingClientRect()

                // Display, position, and set styles for font
                tooltipEl.className += ' active'
                tooltipEl.style.top = tooltipModel.caretY - 34 + 'px'
                tooltipEl.style.height = position.height - tooltipModel.caretY  + 'px'
                tooltipElText.style.width = tooltipModel.width + 'px'
                tooltipEl.style.left = position.left + tooltipModel.caretX + 'px'

                // Hide original tooltip
                tooltipModel.opacity = 0

                // Set text
                tooltipElText.innerHTML = ''

                for (let i = 0; i < bodyLines.length; i++) {
                  var body = bodyLines[i]
                  tooltipElText.innerHTML += body
                }
              }
            }
          }
        }
      })
    },
    getTransferHistory() {
      this.$store.dispatch("daoVault/getTransferHistory", this.curDaoControlAddress.vaultAddr).then(res => {
        this.$store.commit("daoVault/SET_TRANSFERLIST", res)
        this.transferList = res
      })
    },
    approve() {
      this.$store.dispatch("erc20/approve", {
        address: this.curDaoControlAddress.vaultAddr,
        coinAddress: this.curDaoControlAddress.erc20Addr
      }).then(()=>{
        this.$eventBus.$on('message', (message) => {
          if (message.type == "success" && message.message == "Approve Success") {
             this.allowance()
          }
        })
      })
    },
    withdraw() {
      const amount = this.withdrawNumber * (10 ** this.coinInfo.decimals)
      if(amount <= 0 ){
        this.$eventBus.$emit('message', {
          message: "amount should >0",
          type: "error"
        })
        return
      }
      if(amount > this.daoGCBalance.replace(/,/g, '')){
        this.$eventBus.$emit('message', {
          message: "Finance balance not enough",
          type: "error"
        })
        return
      }
      this.$store.dispatch("daoVault/withdraw", {
        address: this.curDaoControlAddress.vaultAddr,
        erc20Addr: this.curDaoControlAddress.erc20Addr,
        amount
      }).then(() => {
        this.$eventBus.$on('message', (message) => {
          if (message.type == "success" && message.message == "Withdraw Success") {
            this.getGovernCoinBalance()
            this.getTransferHistory()
            this.isShowWithdraw = false
          }
        })

      }).catch(err => {
        alert(err)
      })
    },
    getGovernCoinBalance() {
      if (this.curDaoControlAddress.erc20Addr) {
        this.$store.dispatch("erc20/getBalanceOf", {
          toAddr: this.curDaoControlAddress.vaultAddr,
          address: this.curDaoControlAddress.erc20Addr
        }).then(balance => {
          this.$store.commit("erc20/SET_DAOGCBALABCE", balance)
        })
        this.$store.dispatch("erc20/getBalanceOf", {
          toAddr: this.account,
          address: this.curDaoControlAddress.erc20Addr
        }).then(balance => {
          this.$store.commit("erc20/SET_MYGCBALABCE", balance)
        })
      }

    },
    allowance(){
      this.$store.dispatch("erc20/allowance",{
        address:this.curDaoControlAddress.erc20Addr,
        owner:this.account,spender:this.curDaoControlAddress.vaultAddr
      }).then(res=>{
        this.allowanceNumber = res.replace(/,/g, '')
      })
    },

    deposit() {
      const amount = this.depositNumber * (10 ** this.coinInfo.decimals)
      if(amount <= 0 ){
        this.$eventBus.$emit('message', {
          message: "amount should >0",
          type: "error"
        })
        return
      }
      if(amount > this.myGCBalance.replace(/,/g, '')){
        this.$eventBus.$emit('message', {
          message: "You balance not enough",
          type: "error"
        })
        return
      }
      if(this.allowanceNumber < amount){
        this.$eventBus.$emit('message', {
          message: "Please Approve first",
          type: "error"
        })
        return
      }
      this.$store.dispatch("daoVault/deposit", {
        address: this.curDaoControlAddress.vaultAddr,
        erc20Addr: this.curDaoControlAddress.erc20Addr,
        amount:amount
      }).then(() => {
        this.isShowDeposit = false
        this.$eventBus.$on('message', (message) => {
          if (message.type == "success" && message.message == "Deposit Success") {
            this.getTransferHistory()
            this.getGovernCoinBalance()
          }
        })
      }).catch(err => {
        alert(err)
      })
    },
    fillData() {
      this.datacollection = {
        datasets: [
          {
            label: 'Data One',
            backgroundColor: '#fff',
            data: [1, 2]
          }, {
            label: 'Data One',
            backgroundColor: '#fff',
            data: [2, 1]
          }
        ]
      }
    },

  }

}
</script>

<style lang="scss" scoped>
@import url('https://fonts.googleapis.com/css?family=Roboto');

html,
body {
  background: #2F3547;
  font-family: Roboto;
  overflow: hidden
}

#chart {
  height: 50%;
  width: 50%;
}

#chartjs-tooltip {
  width: 1px;
  position: absolute;
  bottom: 50%;
  transform: translateX(-.5px);
  background: white;
  text-align: center;
  transition: .3s;
  opacity: 0;

  &.active {
    opacity: 1;
  }

  &__text {
    position: absolute;
    top: -32px;
    transform: translateX(-50%);
    padding: 2px 0;
    background: white;
    border-radius: 16px;
    font-size: 12px;
    transition: .3s;
  }
}

.daoFinance {
  .rainbow-dialog {
    input {
      width: 300px;
      height: 50px;
      background: #ffffff;
      border: 1px solid #eaeaea;
      border-radius: 10px;
      padding: 0 20px;
      font-size: 16px;
    }

    .rainbow-button {
      width: 200px;
      height: 50px;
      font-size: 20px;
      margin: 20px auto;
    }
  }

  .opera-btn {
    padding: 30px;
    display: flex;

    .my-GCCoin-balance {
      font-size: 20px;
      line-height: 50px;
    }

    .rainbow-button {
      margin-left: 30px;
      width: 200px;
      height: 50px;
      font-size: 18px;
    }
  }

  .finance-box {
    justify-content: space-between;
    display: flex;

    .left {
      .coin-list {
        .item {
          display: flex;
          padding: 30px 20px;
          align-items: center;

          img {
            width: 30px;
            height: 30px;
          }

          .coin-data {
            margin-left: 10px;

            .name {
              color: #a98ebd;
              font-size: 14px;
            }

            .number {
              font-weight: bold;
              text-align: left;
              color: #333333;
              font-size: 16px;
              line-height: 16px;
              padding: 6px 0 0;
            }
          }
        }
      }
    }

    .right {
      flex: 1;
      width: 300px;

      ::v-deep canvas {
        width: 800px !important;
        height: 500px !important;
      }

    }
  }

  .finance-detail {
    padding: 30px;

    .title {
      width: 192px;
      font-size: 26px;
      font-weight: 700;
      text-align: left;
      color: #333333;
      line-height: 60px;
    }

    .financial-detail-list {
      .item {
        display: flex;
        align-items: center;
        height: 90px;
        padding: 0 20px;
        background: #fbfcfe;
        border: 1px solid #f0f0f0;
        margin-top: 20px;
        border-radius: 10px;
        justify-content: space-between;
        width: 100%;

        &:active {
          border: 1px solid;
          border-image: linear-gradient(90deg, #12c2e9, #c471ed 55%, #f64f59) 1 1;
          box-shadow: 0px 0px 15px 3px rgba(218, 0, 127, 0.25);
        }

        .index {
          width: 20px;
          height: 20px;
          border-radius: 3px;
          text-align: center;
          color: #6919bb;
          line-height: 20px;
          font-weight: bold;
          background: #f0effe;
        }

        .name {
          padding: 0 30px;
          font-weight: bold;
          color: #6919bb;
          font-size: 16px;
          width: 500px;
          overflow: hidden;
        }

        .value {
          width: 160px;
          float: right;
          text-align: right;
          font-size: 18px;
          font-weight: bold;

          .income {
            color: #46B787;
          }

          .expenditure {
            color: #FF3B3B;
          }
        }

        .line {
          width: 1px;
          height: 40px;
          background: #eaeaea;
          margin: 0 10px;
        }

        .inOrOut {
          width: 80px;
          height: 20px;
          margin: 0 10px;

          text-align: center;

          .income {
            background: rgba(94, 219, 166, 0.20);
            border: 1px solid rgba(95, 220, 167, 0.50);
            border-radius: 5px;
            color: #46B787;
          }

          .expenditure {
            background: rgba(255, 59, 59, 0.20);
            border: 1px solid rgba(255, 59, 59, 0.50);
            border-radius: 5px;
            color: #FF3B3B;
          }
        }
      }
    }
  }

}
</style>
