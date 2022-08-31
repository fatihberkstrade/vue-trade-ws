<template>
  <div class="hello">
    <h3>
      <input type="text" v-model="pair" />
      <button @click="update">Update</button>
    </h3>
    <h3>
      Buy: {{ totalBuy.toFixed(2) }} Threshold Buy:
      {{ thresholdTotalBuy.toFixed(2) }}
    </h3>
    <h3>
      Sells: {{ totalSell.toFixed(2) }} Threshold Sells:
      {{ thresholdTotalSell.toFixed(2) }}
    </h3>
    <h3>
      {{ (totalBuy / totalSell).toFixed(3) }} -
      {{ (thresholdTotalBuy / thresholdTotalSell).toFixed(3) }}
    </h3>
    <h3>{{ minPrice }} - {{ maxPrice }}</h3>
    <h3 class="main-h3">
      <table>
        <tr
          v-for="trade in trades"
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
        <tr v-for="trade in buys" :key="trade.index" class="buy">
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
        <tr v-for="trade in sells" :key="trade.index" class="sell">
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
export default {
  name: "Trades",
  data() {
    return {
      pair: "btc",
      sells: [],
      buys: [],
      thresholdTotalSell: 0,
      thresholdTotalBuy: 0,
      totalSell: 0,
      totalBuy: 0,
      trades: [],
      minPrice: 0,
      maxPrice: 0,
      ws: undefined,
    };
  },
  methods: {
    processData() {
      const that = this;
      let index = 0;

      this.ws.onmessage = function (event) {
        index++;
        var data = JSON.parse(event.data);
        const price = Number(data.p);
        if (that.minPrice === 0) {
          that.minPrice = price;
        }
        if (price > that.maxPrice) {
          that.maxPrice = price;
        } else if (price < that.minPrice) {
          that.minPrice = price;
        }
        const amount = price * Number(data.q);
        var item = {
          amount: amount,
          isSell: data.m,
          price: price,
          index: index,
          time: data.T,
        };
        that.trades.unshift(item);
        that.trades = that.trades.slice(0, 20);
        if (amount > 100000) {
          if (data.m) {
            that.thresholdTotalSell += amount;
            that.sells.unshift(item);
            that.sells = that.sells.slice(0, 20);
          } else {
            that.thresholdTotalBuy += amount;
            that.buys.unshift(item);
            that.buys = that.buys.slice(0, 20);
          }
        } else {
          if (data.m) {
            that.totalSell += amount;
          } else {
            that.totalBuy += amount;
          }
        }
      };
    },
    update() {
      this.sells = [];
      this.buys = [];
      this.thresholdTotalSell = 0;
      this.thresholdTotalBuy = 0;
      this.totalSell = 0;
      this.totalBuy = 0;
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

.main-h3 {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  align-items: baseline;
}
</style>
