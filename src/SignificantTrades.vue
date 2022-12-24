<template>
  <div class="hello">
    <h3 class="main-div">
      <table>
        <thead>
          <tr>
            <th>Symbol</th>
            <th>Amount</th>
            <th>Price</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="trade in thresholdBuys" :key="trade.index" class="buy">
            <td>
              {{ trade.pair }}
            </td>
            <td>
              {{ trade.amount.toFixed(pricePrecision) }}
            </td>
            <td>
              {{ trade.price.toFixed(pricePrecision) }}
            </td>
            <td>{{ ((Date.now() - trade.time) / 1000).toFixed(0) }}s</td>
          </tr>
        </tbody>
      </table>
      <table>
        <thead>
          <tr>
            <th>Symbol</th>
            <th>Amount</th>
            <th>Price</th>
          </tr>
        </thead>
        <tr
          v-for="trade in displayTrades"
          :key="trade.index"
          :class="{ sell: trade.isSell, buy: !trade.isSell }"
        >
          <td>
            {{ trade.pair }}
          </td>
          <td>
            {{ trade.amount.toFixed(pricePrecision) }}
          </td>
          <td>
            {{ trade.price.toFixed(pricePrecision) }}
          </td>
          <td>{{ ((Date.now() - trade.time) / 1000).toFixed(0) }}s</td>
        </tr>
      </table>
      <table>
        <thead>
          <tr>
            <th>Symbol</th>
            <th>Amount</th>
            <th>Price</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="trade in thresholdSells" :key="trade.index" class="sell">
            <td>
              {{ trade.pair }}
            </td>
            <td>
              {{ trade.amount.toFixed(pricePrecision) }}
            </td>
            <td>
              {{ trade.price.toFixed(pricePrecision) }}
            </td>
            <td>{{ ((Date.now() - trade.time) / 1000).toFixed(0) }}s</td>
          </tr>
        </tbody>
      </table>
    </h3>
  </div>
</template>

<script>
import _ from "lodash";
import axios from "axios";
export default {
  name: "Trades",
  data() {
    return {
      trades: [],
      symbols: [],
      pricePrecision: 2,
    };
  },
  computed: {
    displayTrades() {
      return this.trades.slice(0, 20);
    },
    thresholdTotalBuy() {
      let total = 0;
      this.trades.forEach((element) => {
        if (!element.isSell && element.amount >= element.threshold) {
          total += element.amount;
        }
      });
      return total;
    },
    thresholdTotalSell() {
      let total = 0;
      this.trades.forEach((element) => {
        if (element.isSell && element.amount >= element.threshold) {
          total += element.amount;
        }
      });
      return total;
    },

    thresholdBuys() {
      let returnArray = [];
      this.trades.forEach((trade) => {
        if (!trade.isSell && trade.amount > trade.threshold) {
          returnArray.push(trade);
        }
      });
      return returnArray;
    },
    thresholdSells() {
      let returnArray = [];
      this.trades.forEach((trade) => {
        if (trade.isSell && trade.amount > trade.threshold) {
          returnArray.push(trade);
        }
      });
      return returnArray;
    },
  },
  mounted() {
    const that = this;
    axios
      .get("https://fapi.binance.com/fapi/v1/exchangeInfo")
      .then((response) => {
        let symbols = _.filter(
          response.data.symbols,
          (e) =>
            e.contractType === "PERPETUAL" &&
            (e.pair === "BTCUSDT" || e.pair === "ETHUSDT")
        );
        symbols.forEach((item) => {
          const ws = new WebSocket(
            `wss://fstream.binance.com/ws/${item.pair.toLowerCase()}@aggTrade`
          );
          ws.onmessage = function (event) {
            var data = JSON.parse(event.data);
            const price = Number(data.p);
            const amount = price * Number(data.q);
            var item = {
              pair: data.s.substring(0, data.s.length - 4),
              amount: amount,
              isSell: data.m,
              price: price,
              time: data.T,
              threshold:
                data.s === "BTCUSDT"
                  ? 100000
                  : data.s === "ETHUSDT"
                  ? 50000
                  : 1000,
            };
            that.trades.unshift(item);
            that.trades = that.trades.slice(0, 1000);
            console.log(that.trades[0]);
          };
        });
      });
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
</style>
