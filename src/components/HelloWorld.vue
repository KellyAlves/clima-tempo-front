<template>
  <div>
    <div class="row q-row justify-center">
      <div class="col-5">
        <q-input
          v-model="locale"
          :error="!locale"
          error-message="Selecione uma localidade"
          label="Localidade"
          outlined
          @keyup="filterLocalities"
        >
          <template v-slot:append>
            <q-icon name="search" class="" />
          </template>
        </q-input>
        <q-list bordered class="">
          <q-item
            v-for="locality in filteredLocalities"
            :key="locality.id"
            clickable
            @click="selectLocality(locality)"
          >
            <q-item-section>{{ locality.name }}</q-item-section>
          </q-item>
        </q-list>
      </div>
      <div class="col-2">
        <div>
          <q-select
            v-model="temperature"
            :options="computedOptionsTemp"
            label="Temperatura"
            outlined
          ></q-select>
        </div>
      </div>
      <div class="col-2">
        <div>
          <q-select
            v-model="precipitation"
            :options="computedOptionsPrec"
            label="Chuva"
            outlined
          ></q-select>
        </div>
      </div>
    </div>

    <div v-if="showSearchDiv">
      <q-card
        class=""
        style="width: max-content; margin: 10px auto; padding: 15px 30px"
      >
        <div class="">
          <div class="col-3">
            <q-date v-model="dateModel" mask="YYYY-MM-DD" range minimal />
          </div>
          <div class="col-3 q-mt-lg">
            <q-btn
              color="primary"
              label="Pesquisar"
              @click="checkPeriod"
            ></q-btn>
          </div>
        </div>
      </q-card>
    </div>
    <div v-else>
      <div class="q-pa-md">
        <div class="row justify-center" >
          <div
            class="col-3 q-ma-sm"
            v-for="(forecast, index) in weatherForecasts"
            :key="index"
          >
            <q-card cardProp>
              <q-card-section
                style="
                  text-align: left;
                  background-color: rgba(169, 169, 169, 0.459)
                "
              >
                <div style="font-size: medium">{{ forecast.date }}</div>
                <div style="font-size: small">{{ forecast.text }}</div>
              </q-card-section>

              <q-card-section>
                <div style="display: flex; justify-content: space-around">
                  <div class="div-style" style="color: rgb(27, 127, 160)">
                    <q-img
                      :src="download"
                      spinner-color="white"
                      class="img-style"
                    />
                    {{ forecast.temperature.min }}°C
                  </div>
                  <div class="div-style" style="color: rgb(160, 41, 41)">
                    <q-img
                      :src="upload"
                      spinner-color="white"
                      class="img-style"
                    />
                    {{ forecast.temperature.max }}°C</div>
                </div>
                <div style="display: flex; justify-content: space-around">
                  <div class="div-style">
                    <q-img
                      :src="raindrop"
                      spinner-color="white"
                      class="img-style"
                    />{{ forecast.rain.precipitation }}mm</div>
                  <div class="div-style">
                    <q-img
                      :src="protectionSymbol"
                      spinner-color="white"
                      class="img-style"
                    />{{ forecast.rain.probability }}%</div>
                </div>
              </q-card-section>
            </q-card>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios"
import { ref } from "vue"
import download from '../assets/images/icons/download.png'
import protectionSymbol from '../assets/images/icons/protectionSymbol.png'
import raindrop from '../assets/images/icons/raindrop.png'
import upload from '../assets/images/icons/upload.png'

export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  components: {},

  data() {
    return {
      loading: false,
      locale: "",
      showSearchDiv: false,
      precipitation: "",
      temperature: "",
      localesData: [],
      filteredLocalities: [],
      selectedLocale: "",
      weatherForecasts: {},
      weatherForecastsCheck: {}
    };
  },

  computed: {
    computedOptionsTemp() {
      return [
        { value: "C", label: "°C" },
        { value: "F", label: "°F" },
      ];
    },
    computedOptionsPrec() {
      return [
        { value: "I", label: "INCH" },
        { value: "M", label: "MM" },
      ];
    },
  },

  setup() {
    return {
      dateModel: ref({ from: "2017-02-02", to: "2017-02-06" }),
      protectionSymbol: ref(protectionSymbol),
      raindrop: ref(raindrop),
      upload: ref(upload),
      download: ref(download),
    };
  },

  mounted() {
    this.getLocales();
  },

  methods: {
    async getLocales() {
      try {
        this.loading = true;
        const { data } = await axios.get("http://localhost:3030/locales");
        this.localesData = data;
      } catch (error) {
        console.error(error);
      } finally {
        this.loading = false;
      }
    },

    async checkPeriod() {
      try {
        this.loading = true;
        const { data } = await axios.get(
          `http://localhost:3030/period/${this.selectedLocale}/${this.dateModel.from}/${this.dateModel.to}`
        );
        if (!data) {
          alert(
            "O período selecionado não está disponível para a localidade selecionada"
          );
          return;
        }
        this.searchWeather();
      } catch (error) {
        console.error(error);
      } finally {
        this.loading = false;
      }
    },

    async searchWeather() {
      try {
        this.loading = true;
        this.showSearchDiv = false
        const { data } = await axios.get(
          `http://localhost:3030/weather/${this.selectedLocale}`
        )
        const apiData = data
        this.weatherForecasts = Object.freeze(apiData)
      } catch (error) {
        console.error(error)
      } finally {
        this.loading = false
      }
    },

    filterLocalities() {
      if (this.locale.length > 2) {
        this.filteredLocalities = this.localesData.filter((locale) => {
          const normalizedLocale = this.normalizeText(locale.name)
          const normalizedSelectedLocale = this.normalizeText(this.locale)
          return normalizedLocale.includes(normalizedSelectedLocale)
        })
      } else {
        this.filteredLocalities = []
      }
    },
    selectLocality(locality) {
      this.selectedLocale = locality.id
      this.locale = locality.name + " - " + locality.state
      this.filteredLocalities = []
      this.showSearchDiv = true
    },

    normalizeText(text) {
      return text
        .normalize("NFD")
        .replace(/[\u0300-\u036f]/g, "")
        .toLowerCase()
    },
  },
}
</script>

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


.q-row {
  margin-top: 30px;
  gap: 10px;
  min-width: max-content;
}
.cardProp {
  width: 300px;
  height: 300px;
}
.q-row-card {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px;
  margin: 10px;
  flex-direction: row;
}
.img-style {
  width: 20px;
  display: block;
  margin-right: 5px;
}
.div-style {
  display: flex;
  margin: 10px;
  font-size: larger;
}
</style>
