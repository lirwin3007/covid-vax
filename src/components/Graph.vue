<script>
import { Line } from 'vue-chartjs';
import regression from 'regression';

export default {
  extends: Line,
  props: {
    data: Array
  },
  data() {
    return {
      options: {
        tooltips: {
          callbacks: {
            label: function(tooltipItem, data) {
              var label = data.datasets[tooltipItem.datasetIndex].label || '';

              if (label) {
                  label += ': ';
              }
              label += tooltipItem.yLabel.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
              return label;
            }
          }
        },
        maintainAspectRatio: false,
        scales: {
          xAxes: [{
            scaleLabel: {
              display: true,
              labelString: 'Date'
            },
            type: 'time',
          }],
          yAxes: [{
            scaleLabel: {
              display: true,
              labelString: 'Number of people'
            },
            ticks: {
              min: 0,
              max: 70000000,
              userCallback: function(value) {
                value = value.toString();
                value = value.split(/(?=(?:...)*$)/);
                value = value.join(',');
                return value;
              }
            },
          }],
        },
      }
    };
  },
  mounted () {
    this.renderChart(this.chartData, this.options);
  },
  methods: {
    createStartDatapoint(time, value) {
      return {
        t: time,
        y: value
      }
    }
  },
  computed: {
    orderedData() {
      return this.data.slice().sort((a, b) => new Date(a.date) - new Date(b.date));
    },
    startDate() {
      return this.orderedData[0].date;
    },
    endDate() {
      var result = new Date(this.prediction[this.prediction.length - 1].t);
      result.setDate(result.getDate() + 3 * 7);
      return result;
    },
    prediction() {
      var model = regression.exponential(this.orderedData.map(dataPoint => [
        ((new Date(dataPoint.date)).getTime() - (new Date("2020-12-01")).getTime()) / 100000,
        dataPoint.firstdose == 0 ? 1 : dataPoint.firstdose / 100000
      ]),
      {
        precision: 25
      });

      const lastDate = this.orderedData[this.orderedData.length - 1];
      var counter = 0;
      var total = lastDate.firstdose;
      var result = [];

      var currentDate = new Date(lastDate.date);
      while (counter < 25 && total < 55980000) {
        const time = (new Date(currentDate) - (new Date("2020-12-01")).getTime()) / 100000;
        var prediction = model.predict(time)[1] * 100000;
        if (prediction > 55980000) {
          prediction = 55980000;
        }
        result.push({
          t: new Date(currentDate),
          y: parseInt(prediction)
        });

        currentDate.setDate(currentDate.getDate() + 1);
        counter += 1;
        total = prediction;
      }

      return result;
    },
    chartData() {
      return {
        datasets: [
          {
            label: "Population (England)",
            backgroundColor: "#F0F0F055",
            borderDash: [10, 10],
            data: [
              this.createStartDatapoint(this.startDate, 55980000),
              this.createStartDatapoint(this.endDate, 55980000)
            ]
          },
          {
            label: "Vaccine Priority Cohorts 1-4 (88% of mortality)",
            borderColor: "#101010AA",
            borderDash: [5, 5],
            fill: false,
            data: [
              this.createStartDatapoint(this.startDate, 12000000),
              this.createStartDatapoint(this.endDate, 12000000)
            ]
          },
          {
            label: "Vaccine Priority Cohorts 1-9 (99% of mortality)",
            borderColor: "#101010AA",
            borderDash: [5, 15],
            fill: false,
            data: [
              this.createStartDatapoint(this.startDate, 27000000),
              this.createStartDatapoint(this.endDate, 27000000)
            ]
          },
          {
            label: "First Dose",
            backgroundColor: "#FFD70055",
            borderColor: "#FFD700FF",
            data: this.orderedData.map(dataPoint => ({
              t: dataPoint.date,
              y: dataPoint.firstdose
            })),
          },
          {
            label: "Protected from severe disease",
            backgroundColor: "#00FF0055",
            borderColor: "#00FF00FF",
            data: this.orderedData.map(dataPoint => {
              var dateFor = new Date(dataPoint.date);
              dateFor.setDate(dateFor.getDate() + 3 * 7);
              return {
                t: dateFor,
                y: parseInt(dataPoint.firstdose * 0.7)
              }
            }),
          },
          {
            label: "[PREDICTION] First Dose",
            backgroundColor: "#80808055",
            borderColor: "#ffe8baFF",
            data: [
              ...this.prediction,
              ...(this.prediction[this.prediction.length - 1].y == 55980000 ?
              [{
                t: this.endDate,
                y: this.prediction[this.prediction.length - 1].y
              }] :
              [])
            ],
          },
          {
            label: "[PREDICTION] Protected from severe disease",
            backgroundColor: "#80808055",
            borderColor: "#b5ffb5FF",
            data: this.prediction.map(dataPoint => {
              var dateFor = new Date(dataPoint.t);
              dateFor.setDate(dateFor.getDate() + 3 * 7);
              return {
                t: dateFor,
                y: parseInt(dataPoint.y * 0.7)
              }
            }),
          },
        ],
      };
    },
  },
}
</script>
