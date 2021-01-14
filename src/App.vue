<template>
  <div id="app">
    <h1>Coronavirus Vaccine Progress in England</h1>
    <Graph :data="data" v-if="data" style="height: 65vh; width: 90vw; margin-left:5vw;" :width="100" :height="100" />
    <div v-else>
      Loading
    </div>
    <div style="text-align: left;">
      <div id="credit">
        Created by <b><a href="mailto:coding@louisirwin.co.uk">Louis Irwin</a></b><br>
        Code available open source on <a href="https://github.com/lirwin3007">GitHub</a><br>
        First dose data is sourced from the <a href="https://coronavirus.data.gov.uk/details/developers-guide">government coronavirus data API</a>
      </div>

      <div id="warning">
        <b>WARNING</b><br>
        All other data is the result of very basic modelling. I am in no way a modelling expert, so all other data should be treated the same as you would a <b>wild guess</b>.<br> Please read the notes below and <b>come to your own conclusion</b> as to if the models used are representative of reality.
      </div>

      <div id="notes">
        <div @click="showNotes=!showNotes" style="cursor: pointer;"><b>Modelling Notes {{ !showNotes ? '(Click to expand)' : '' }}</b></div>

        <span v-if="showNotes">
          <br>'Protected from severe disease' is calculated by taking 70% of the total number of people who had received a vaccine 3 weeks ago.
          This is in line with section 4.7 of <a href="https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/951928/uk-covid-19-vaccines-delivery-plan-final.pdf">this government report</a><br><br>

          Predictions are made using exponential regression. i.e. new data is predicted by the exponential curve that best fits the current data.
          This does not take into account many real life effects such as the availability of vaccine and efficiency of distribution.<br><br>

          This prediction is limited to 25 days in an effort to show this method is not reliable predicting far into the future - however even this limit of 25 days probably well over-estimates this models ability to predict anything!<br><br>

          Numbers of people in each priority cohort is taken from <a href="https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/951928/uk-covid-19-vaccines-delivery-plan-final.pdf">this government report</a>
        </span>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import Graph from './components/Graph.vue';

export default {
  name: 'App',
  components: {
    Graph,
  },
  data() {
    return {
      data: null,
      showNotes: false,
    };
  },
  mounted() {
    const baseUrl = "https://api.coronavirus.data.gov.uk/v1";
    const structure = `{"date": "date", "firstdose": "cumPeopleVaccinatedFirstDoseByPublishDate", "seconddose": "cumPeopleVaccinatedSecondDoseByPublishDate"}`;
    const filters = "areaType=nation;areaName=england";
    axios.get(`${baseUrl}/data?structure=${encodeURIComponent(structure)}&filters=${filters}`)
      .then(response => {
        this.data = [
          {
            date: "2020-12-08",
            firstdose: 0,
            seconddose: 0
          },
          ...response.data.data
        ];
      })
      .catch(function (error) {
        alert("Oops... Something went wrong whilst loading the data.")
        console.log(error);
      });
  }
}
</script>

<style>
#app {
  font-family: 'Open Sans', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100vh;
}

a {
  text-decoration-style: dashed;
  text-decoration-color: #5e5e5e;
  color: black;
  font-weight: bold;
}

#credit {
  margin: 1rem;
}

#warning {
  margin: 1rem;
  background-color: rgb(255, 136, 136);
  border: 3px solid rgb(141, 44, 44);
  border-radius: 0.5rem;
  padding: 1rem;
}

#notes {
  margin: 1rem;
  background-color: rgb(136, 164, 255);
  border: 3px solid rgb(44, 54, 141);
  border-radius: 0.5rem;
  padding: 1rem;
}
</style>
