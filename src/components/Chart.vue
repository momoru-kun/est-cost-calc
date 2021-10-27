<template>
  <div class="ma-4 pa-4" style="max-width: 100%; overflow-x:auto;">
    <v-row class="justify-end">
    <v-btn
      text
      color="primary"
      @click="load_chart()"
    >
      <v-icon>mdi-refresh</v-icon> Обновить
    </v-btn>
    </v-row>
    <v-row>
      <div ref="chart"></div>
    </v-row>
  </div>
</template>

<script>
export default {
  name: "Chart",
  props: {
    data: { type: Array, default: () => [] }
  },
  methods: {
    load_chart() {
      /* eslint-disable */
      var data = new google.visualization.DataTable();
      data.addColumn('number', 'Трудозатраты, часов');
      data.addColumn('number', 'Бюджет, рублей');

      data.addRows(this.data);
      var options = {
        width: 800,
        height: 500,
        chart: {
          title: 'Стоимость работ к трудозатратам',
        },
        hAxis: {title: 'Трудозатраты, часов'},
        vAxis: {title: 'Бюджет, рублей'}
      }

      var chart = new google.charts.Scatter(this.$refs.chart);
      chart.draw(data, google.charts.Scatter.convertOptions(options));
      /* eslint-enable */
    },
  },
  mounted() {
    this.load_chart()
  }
}
</script>