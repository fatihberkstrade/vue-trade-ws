<template>
  <div class="hello">
    <h3>Buy: {{ totalSell.toFixed(2) }}</h3>
    <h3>Sells: {{ totalBuy.toFixed(2) }}</h3>
    <div class="main-div">
      <table>
        <tr
          v-for="trade in trades"
          :key="trade.index"
          :class="{ sell: trade.sell, buy: !trade.sell }"
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
        </tr>
      </table>
      <table>
        <tr v-for="trade in sells" :key="trade.index" :class="sell">
          <td>
            {{ trade.amount.toFixed(2) }}
          </td>
          <td>
            {{ trade.price.toFixed(2) }}
          </td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: "Trades",
  data() {
    return {
      sells: [],
      buys: [],
      totalSell: 0,
      totalBuy: 0,
      trades: [],
    };
  },
  mounted() {
    const ws = new WebSocket("wss://fstream.binance.com/ws/btcusdt@aggTrade");
    const that = this;
    let index = 0;
    ws.onmessage = function (event) {
      index++;
      var data = JSON.parse(event.data);
      const price = Number(data.p);
      const amount = price * Number(data.q);
      var item = {
        amount: amount,
        isSell: data.m,
        price: price,
        index: index,
      };
      that.trades.unshift(item);
      that.trades = that.trades.slice(0, 20);
      if (amount > 100000) {
        if (data.m) {
          that.totalSell += amount;
          that.sells.unshift(item);
        } else {
          that.totalBuy += amount;
          that.buys.unshift(item);
        }
      }
    };
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
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
}
</style>
