<template>
  <div class="hello">
    <div class="header-div">
      <select v-model="selectedSymbol">
        <option
          :value="symbol"
          @change="update"
          v-for="symbol in symbols"
          :key="symbol.symbol"
        >
          {{ symbol.pair }}
        </option>
      </select>
      <input type="number" name="" id="" v-model="threshold" />
      <div>
        <button @click="descrease" style="width: 50%">-</button>
        <button @click="increase" style="width: 50%">+</button>
      </div>
      <h3 class="buy">{{ totalBuy.toFixed(pricePrecision) }}</h3>
      <div></div>
      <h3 class="sell">{{ totalSell.toFixed(pricePrecision) }}</h3>
      <h3 class="buy">
        {{ thresholdTotalBuy.toFixed(2) }}
      </h3>
      <div></div>
      <h3 class="sell">
        {{ thresholdTotalSell.toFixed(2) }}
      </h3>
      <div></div>
      <h3 :class="{ buy: totalBuy > totalSell, sell: totalSell > totalBuy }">
        {{ (totalBuy / totalSell).toFixed(3) }} -
        {{ (thresholdTotalBuy / thresholdTotalSell).toFixed(3) }}
      </h3>
      <div></div>
      <div></div>
      <h3>{{ minPrice }} - {{ maxPrice }}</h3>
      <div></div>
      <div></div>
      <h3>{{ lastPrice.toFixed(pricePrecision) }}</h3>
      <div></div>
    </div>
    <h3 class="main-div">
      <table>
        <tr v-for="trade in thresholdBuys" :key="trade.index" class="buy">
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
        <tr
          v-for="trade in displayTrades"
          :key="trade.index"
          :class="{ sell: trade.isSell, buy: !trade.isSell }"
        >
          <td>
            {{ trade.amount.toFixed(pricePrecision) }}
          </td>
          <td>
            {{ trade.price.toFixed(pricePrecision) }}
          </td>
        </tr>
      </table>
      <table>
        <tr v-for="trade in thresholdSells" :key="trade.index" class="sell">
          <td>
            {{ trade.amount.toFixed(pricePrecision) }}
          </td>
          <td>
            {{ trade.price.toFixed(pricePrecision) }}
          </td>
          <td>{{ ((Date.now() - trade.time) / 1000).toFixed(0) }}s</td>
        </tr>
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
      pair: "eh",
      trades: [],
      ws: undefined,
      symbols: [],
      selectedSymbol: { pair: "ETHUSDT", symbol: "ETHUSDT" },
      symbol: { pair: "ETHUSDT", symbol: "ETHUSDT" },
      pricePrecision: 2,
      minPrice: 1000000,
      maxPrice: 0,
      threshold: 50000,
      thresholdValue: 50000,
      lastPrice: 0,
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
        if (!element.isSell && element.amount >= this.thresholdValue) {
          total += element.amount;
        }
      });
      return total;
    },
    thresholdTotalSell() {
      let total = 0;
      this.trades.forEach((element) => {
        if (element.isSell && element.amount >= this.thresholdValue) {
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
        if (trade.amount > this.thresholdValue) {
          returnArray.push(trade);
        }
      });
      return returnArray;
    },
    thresholdSells() {
      let returnArray = [];
      this.sells.forEach((trade) => {
        if (trade.amount > this.thresholdValue) {
          returnArray.push(trade);
        }
      });
      return returnArray;
    },
  },
  watch: {
    selectedSymbol(newValue, oldValue) {
      if (newValue) {
        console.log(newValue);
        if (newValue.pair === "BTCUSDT") {
          this.threshold = 100000;
        } else if (newValue.pair === "ETHUSDT") {
          this.threshold = 50000;
        } else {
          this.threshold = 10000;
        }
        this.update();
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
          pair: data.s,
          amount: amount,
          isSell: data.m,
          price: price,
          index: index,
          time: data.T,
        };
        if (price < that.minPrice) {
          that.minPrice = price;
        }
        if (price > that.maxPrice) {
          that.maxPrice = price;
        }
        that.trades.unshift(item);
        that.trades = that.trades.slice(0, 1000);
        that.lastPrice = price;
      };
    },
    increase() {
      this.threshold *= 2;
      this.update();
    },
    descrease() {
      this.threshold /= 2;
      this.update();
    },
    update() {
      this.thresholdValue = this.threshold;
      this.minPrice = 1000000;
      this.maxPrice = 0;
      this.pricePrecision = this.selectedSymbol.pricePrecision;
      this.trades = _.filter(
        this.trades,
        (e) => e.pair === this.selectedSymbol.pair
      );
      this.ws.close();
      this.ws = new WebSocket(
        `wss://fstream.binance.com/ws/${this.selectedSymbol.pair.toLowerCase()}@aggTrade`
      );
      this.processData();
    },
  },
  mounted() {
    this.ws = new WebSocket("wss://fstream.binance.com/ws/ethusdt@aggTrade");
    this.processData();
    axios
      .get("https://fapi.binance.com/fapi/v1/exchangeInfo")
      .then((response) => {
        let symbols = _.filter(
          response.data.symbols,
          (e) => e.contractType === "PERPETUAL" && e.quoteAsset === "USDT"
        );
        symbols = _.orderBy(symbols, (e) => e.baseAsset);
        debugger;
        this.symbols.push(...symbols.filter((e) => e.pair === "BTCUSDT"));
        this.symbols.push(...symbols.filter((e) => e.pair === "ETHUSDT"));
        this.symbols.push(
          ...symbols.filter((e) => e.pair !== "ETHUSDT" && e.pair !== "BTCUSDT")
        );
        this.selectedSymbol = _.filter(
          this.symbols,
          (e) => e.pair === "ETHUSDT"
        )[0];
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

.header-div {
  padding-left: 1vw;
  padding-right: 1vw;
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  grid-template-rows: 1fr 1fr 1fr;
  grid-gap: 1vw;
}
</style>
