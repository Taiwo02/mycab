<template>
  <b-card :title="$t('dashboards.product-categories')">
    <div class="dashboard-donut-chart">
      <polar-area-chart :data="polarAreaChartData" shadow />
    </div>
  </b-card>
</template>
<script>
import PolarAreaChart from "../Analcomponents/Charts/PolarArea";
import { polarAreaChartData } from "../Analcomponents/data/charts";

export default {
  components: {
    "polar-area-chart": PolarAreaChart
  },
  data() {
    return {
      polarAreaChartData
    };
  }
};
</script>
