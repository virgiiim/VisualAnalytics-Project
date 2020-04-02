<template>
  <vue-plotly :data="traces" :layout="layout" :options="options"  />
</template>
<script>
let d3 = require('d3');
import VuePlotly from '@statnett/vue-plotly';
export default {
  name: 'BarPlot',
  components:{
    VuePlotly,
  },
  props: {
    bar_data: Array,
  },
  data() {
    return {
      traces: [],
      layout:{
        width:700,
        heigth:800,
        margin:{'t': 15, 'l':50, 'r':20,'b':30},
        xaxis:{
          title: 'time',
          fixedrange: true //disable zoom
        },
        yaxis:{
          title: 'number of measurements',
          fixedrange: true //disable zoom
        }
      },
      options:{
        'displayModeBar':false
      }
    };
  },
  watch:{
    bar_data(changed_val){
      this.traces = []; //emptying array containing data everytimes that the components is called
      var result = {
        x: changed_val[0], //dates
        y: changed_val[1], //number of measurements
        type: 'bar'
      }
      this.traces.push(result);
    }
  }
};
</script>
