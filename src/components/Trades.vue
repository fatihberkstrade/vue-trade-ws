<template>
  <div class="hello">
    <h3>
      <input type="text" v-model="pair" />
      <button @click="update">Update</button>
    </h3>
    <div class="header-div">
      <h3>Buy: {{ totalBuy.toFixed(2) }}</h3>
      <h3>Sells: {{ totalSell.toFixed(2) }}</h3>
      <h3>
        Threshold Buy:
        {{ thresholdTotalBuy.toFixed(2) }}
      </h3>
      <h3>
        Threshold Sells:
        {{ thresholdTotalSell.toFixed(2) }}
      </h3>
      <h3>
        {{ (totalBuy / totalSell).toFixed(3) }} -
        {{ (thresholdTotalBuy / thresholdTotalSell).toFixed(3) }}
      </h3>
      <h3>{{ minPrice }} - {{ maxPrice }}</h3>
    </div>
    <h3 class="main-div">
      <table>
        <tr
          v-for="trade in displayTrades"
          :key="trade.index"
          :class="{ sell: trade.isSell, buy: !trade.isSell }"
        >
          <td>
            {{ trade.amount.toFixed(2) }}
          </td>
          <td>
            {{ trade.price.toFixed(2) }}
          </td>
        </tr>
      </table>
      <table>
        <tr v-for="trade in thresholdBuys" :key="trade.index" class="buy">
          <td>
            {{ trade.amount.toFixed(2) }}
          </td>
          <td>
            {{ trade.price.toFixed(2) }}
          </td>
          <td>{{ ((Date.now() - trade.time) / 1000).toFixed(0) }}s</td>
        </tr>
      </table>
      <table>
        <tr v-for="trade in thresholdSells" :key="trade.index" class="sell">
          <td>
            {{ trade.amount.toFixed(2) }}
          </td>
          <td>
            {{ trade.price.toFixed(2) }}
          </td>
          <td>{{ ((Date.now() - trade.time) / 1000).toFixed(0) }}s</td>
        </tr>
      </table>
    </h3>
  </div>
</template>

<script>
import _ from "lodash";
export default {
  name: "Trades",
  data() {
    return {
      pair: "btc",
      trades: [],
      ws: undefined,
    };
  },
  computed: {
    displayTrades() {
      return this.trades.slice(0, 20);
    },
    totalBuy() {
      let total = 0;
      this.buys.forEach((element) => {
        total += element.amount;
      });
      return total;
    },
    totalSell() {
      let total = 0;
      this.sells.forEach((element) => {
        total += element.amount;
      });
      return total;
    },
    thresholdTotalBuy() {
      let total = 0;
      this.trades.forEach((element) => {
        if (!element.isSell && element.amount >= 100000) {
          total += element.amount;
        }
      });
      return total;
    },
    thresholdTotalSell() {
      let total = 0;
      this.trades.forEach((element) => {
        if (element.isSell && element.amount >= 100000) {
          total += element.amount;
        }
      });
      return total;
    },
    buys() {
      let returnArray = [];
      this.trades.forEach((trade) => {
        if (!trade.isSell) {
          returnArray.push(trade);
        }
      });
      return returnArray;
    },
    sells() {
      let returnArray = [];
      this.trades.forEach((trade) => {
        if (trade.isSell) {
          returnArray.push(trade);
        }
      });
      return returnArray;
    },
    thresholdBuys() {
      let returnArray = [];
      this.buys.forEach((trade) => {
        if (trade.amount > 100000) {
          returnArray.push(trade);
        }
      });
      return returnArray;
    },
    thresholdSells() {
      let returnArray = [];
      this.sells.forEach((trade) => {
        if (trade.amount > 100000) {
          returnArray.push(trade);
        }
      });
      return returnArray;
    },
    minPrice() {
      var item = _.minBy(this.trades, function (item) {
        return item.price;
      });
      if (item) {
        return item.price;
      } else {
        return 0;
      }
    },
    maxPrice() {
      var item = _.maxBy(this.trades, function (item) {
        return item.price;
      });
      if (item) {
        return item.price;
      } else {
        return 0;
      }
    },
  },
  methods: {
    processData() {
      const that = this;
      let index = 0;

      this.ws.onmessage = function (event) {
        index++;
        var data = JSON.parse(event.data);
        const price = Number(data.p);
        const amount = price * Number(data.q);
        var item = {
          amount: amount,
          isSell: data.m,
          price: price,
          index: index,
          time: data.T,
        };
        that.trades.unshift(item);
        that.trades = that.trades.slice(0, 1000);
      };
    },
    update() {
      this.trades = [];
      this.ws.close();
      this.ws = new WebSocket(
        `wss://fstream.binance.com/ws/${this.pair}usdt@aggTrade`
      );
      this.processData();
    },
  },
  mounted() {
    this.ws = new WebSocket("wss://fstream.binance.com/ws/btcusdt@aggTrade");
    this.processData();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 10px 0 0;
}

.buy {
  color: green;
}
.sell {
  color: red;
}

.main-div {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  align-items: baseline;
}

.header-div {
  padding-left: 10vw;
  padding-right: 10vw;
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-template-rows: 1fr 1fr 1fr;
}
</style>
