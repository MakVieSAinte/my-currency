<template>
  <div class="bloc">
    <div class="wrapper">
      <header>Convertisseur de devises</header>
      <form @submit.prevent="getExchangeRate">
        <div class="amount">
          <label for="amount">Montant à convertir</label>
          <input type="number" id="amount" v-model.number="amount" min="0.01" step="0.01" required>
        </div>
        <div class="drop-list">
          <div class="from">
            <label>De :</label>
            <div class="select-box">
              <img :src="fromFlag" alt="Drapeau devise source">
              <select v-model="fromCurrency" @change="loadFlag('from')">
                <option v-for="(country, code) in countryList" :key="code" :value="code">{{ code }}</option>
              </select>
            </div>
          </div>
          <div class="icon" @click="swapCurrencies" title="Inverser les devises">

            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#ea400c" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-arrow-right-left"><path d="m16 3 4 4-4 4"/><path d="M20 7H4"/><path d="m8 21-4-4 4-4"/><path d="M4 17h16"/></svg>

          </div>
          <div class="to">
            <label>Vers :</label>
            <div class="select-box">
              <img :src="toFlag" alt="Drapeau devise cible">
              <select v-model="toCurrency" @change="loadFlag('to')">
                <option v-for="(country, code) in countryList" :key="code" :value="code">{{ code }}</option>
              </select>
            </div>
          </div>
        </div>
        <div class="exchange-rate">{{ exchangeRateTxt }}</div>
        <button type="button" @click="onConvertClick">Convertir</button>
      </form>

      <!-- Historique des conversions -->
      <section v-if="history.length" class="section-historique">
        <h2>Historique des conversions</h2>
        <ul>
          <li v-for="(entry, index) in history" :key="index" class="historique-texte">{{ entry }}</li>
        </ul>
        <button @click="clearHistory" class="clear-history">Effacer l'historique</button>
      </section>
    </div>

    <div class="img">
      <span></span>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  name: "CurrencyConverter",
  data() {
    return {
      countryList: { 
        "AED" : "AE",
    "AFN" : "AF",
    "XCD" : "AG",
    "ALL" : "AL",
    "AMD" : "AM",
    "ANG" : "AN",
    "AOA" : "AO",
    "AQD" : "AQ",
    "ARS" : "AR",
    "AUD" : "AU",
    "AZN" : "AZ",
    "BAM" : "BA",
    "BBD" : "BB",
    "BDT" : "BD",
    "XOF" : "BE",
    "BGN" : "BG",
    "BHD" : "BH",
    "BIF" : "BI",
    "BMD" : "BM",
    "BND" : "BN",
    "BOB" : "BO",
    "BRL" : "BR",
    "BSD" : "BS",
    "NOK" : "BV",
    "BWP" : "BW",
    "BYR" : "BY",
    "BZD" : "BZ",
    "CAD" : "CA",
    "CDF" : "CD",
    "XAF" : "CF",
    "CHF" : "CH",
    "CLP" : "CL",
    "CNY" : "CN",
    "COP" : "CO",
    "CRC" : "CR",
    "CUP" : "CU",
    "CVE" : "CV",
    "CYP" : "CY",
    "CZK" : "CZ",
    "DJF" : "DJ",
    "DKK" : "DK",
    "DOP" : "DO",
    "DZD" : "DZ",
    "ECS" : "EC",
    "EEK" : "EE",
    "EGP" : "EG",
    "ETB" : "ET",
    "EUR" : "FR",
    "FJD" : "FJ",
    "FKP" : "FK",
    "GBP" : "GB",
    "GEL" : "GE",
    "GGP" : "GG",
    "GHS" : "GH",
    "GIP" : "GI",
    "GMD" : "GM",
    "GNF" : "GN",
    "GTQ" : "GT",
    "GYD" : "GY",
    "HKD" : "HK",
    "HNL" : "HN",
    "HRK" : "HR",
    "HTG" : "HT",
    "HUF" : "HU",
    "IDR" : "ID",
    "ILS" : "IL",
    "INR" : "IN",
    "IQD" : "IQ",
    "IRR" : "IR",
    "ISK" : "IS",
    "JMD" : "JM",
    "JOD" : "JO",
    "JPY" : "JP",
    "KES" : "KE",
    "KGS" : "KG",
    "KHR" : "KH",
    "KMF" : "KM",
    "KPW" : "KP",
    "KRW" : "KR",
    "KWD" : "KW",
    "KYD" : "KY",
    "KZT" : "KZ",
    "LAK" : "LA",
    "LBP" : "LB",
    "LKR" : "LK",
    "LRD" : "LR",
    "LSL" : "LS",
    "LTL" : "LT",
    "LVL" : "LV",
    "LYD" : "LY",
    "MAD" : "MA",
    "MDL" : "MD",
    "MGA" : "MG",
    "MKD" : "MK",
    "MMK" : "MM",
    "MNT" : "MN",
    "MOP" : "MO",
    "MRO" : "MR",
    "MTL" : "MT",
    "MUR" : "MU",
    "MVR" : "MV",
    "MWK" : "MW",
    "MXN" : "MX",
    "MYR" : "MY",
    "MZN" : "MZ",
    "NAD" : "NA",
    "XPF" : "NC",
    "NGN" : "NG",
    "NIO" : "NI",
    "NPR" : "NP",
    "NZD" : "NZ",
    "OMR" : "OM",
    "PAB" : "PA",
    "PEN" : "PE",
    "PGK" : "PG",
    "PHP" : "PH",
    "PKR" : "PK",
    "PLN" : "PL",
    "PYG" : "PY",
    "QAR" : "QA",
    "RON" : "RO",
    "RSD" : "RS",
    "RUB" : "RU",
    "RWF" : "RW",
    "SAR" : "SA",
    "SBD" : "SB",
    "SCR" : "SC",
    "SDG" : "SD",
    "SEK" : "SE",
    "SGD" : "SG",
    "SKK" : "SK",
    "SLL" : "SL",
    "SOS" : "SO",
    "SRD" : "SR",
    "STD" : "ST",
    "SVC" : "SV",
    "SYP" : "SY",
    "SZL" : "SZ",
    "THB" : "TH",
    "TJS" : "TJ",
    "TMT" : "TM",
    "TND" : "TN",
    "TOP" : "TO",
    "TRY" : "TR",
    "TTD" : "TT",
    "TWD" : "TW",
    "TZS" : "TZ",
    "UAH" : "UA",
    "UGX" : "UG",
    "USD" : "US",
    "UYU" : "UY",
    "UZS" : "UZ",
    "VEF" : "VE",
    "VND" : "VN",
    "VUV" : "VU",
    "YER" : "YE",
    "ZAR" : "ZA",
    "ZMK" : "ZM",
    "ZWD" : "ZW"
       },
      amount: 1,
      fromCurrency: "XAF",
      toCurrency: "USD",
      exchangeRateTxt: "Conversion en cours...",
      lastConvertedAmount: null, // Pour stocker le dernier montant converti
      history: JSON.parse(localStorage.getItem("conversionHistory") || "[]"),
      userClicked: false
    };
  },
  computed: {
    fromFlag() {
      return `https://flagcdn.com/48x36/${this.countryList[this.fromCurrency].toLowerCase()}.png`;
    },
    toFlag() {
      return `https://flagcdn.com/48x36/${this.countryList[this.toCurrency].toLowerCase()}.png`;
    }
  },
  methods: {
    swapCurrencies() {
      [this.fromCurrency, this.toCurrency] = [this.toCurrency, this.fromCurrency];
      this.getExchangeRate();
    },
    getExchangeRate() {
      if (this.amount < 0.01) {
        this.exchangeRateTxt = "Veuillez entrer un montant valide.";
        return;
      }
      this.exchangeRateTxt = "Conversion en cours...";
      const url = `https://v6.exchangerate-api.com/v6/806502e1300f21e45f120893/latest/${this.fromCurrency}`;
      
      fetch(url)
        .then(response => response.json())
        .then(result => {
          const rate = result.conversion_rates[this.toCurrency];
          if (rate) {
            const total = (this.amount * rate).toFixed(2);
            const conversionText = `${this.amount} ${this.fromCurrency} = ${total} ${this.toCurrency}`;
            this.exchangeRateTxt = conversionText;
            this.lastConvertedAmount = total; // Stocker la conversion pour inversion

            if (this.userClicked) {
              this.history.unshift(conversionText);
              localStorage.setItem("conversionHistory", JSON.stringify(this.history));
            }
          } else {
            this.exchangeRateTxt = "Conversion impossible pour cette devise.";
          }
        })
        .catch(() => {
          this.exchangeRateTxt = "Erreur lors de la récupération des taux.";
        });
    },
    clearHistory() {
      this.history = [];
      localStorage.removeItem("conversionHistory");
    },
    onConvertClick() {
      this.userClicked = true;
      this.getExchangeRate();
    },
    reverseConversion() {
      if (this.lastConvertedAmount) {
        this.amount = parseFloat(this.lastConvertedAmount); // Définir l'ancien montant converti
        this.swapCurrencies();
      }
    }
  },
  mounted() {
    this.getExchangeRate();
  }
});
</script>

