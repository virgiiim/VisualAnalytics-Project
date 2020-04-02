<template>
  <vue-plotly :data="traces" :layout="layout" :options="options"/>
</template>
<script>
let d3 = require('d3');
import VuePlotly from '@statnett/vue-plotly';
export default {
  name: 'stacked_area',
  components:{
    VuePlotly,
  },
  props: {
    area_data: Object,
  },
  data() {
    return {
      traces: [],
      layout:{
        width:700,
        heigth:800,
        margin:{'t': 15, 'l':50, 'r':20,'b':65},
        showlegend: true,
        hovermode: false, //disable tooltip
        legend: {
          orientation: 'h',
          traceorder: 'reversed',
          x: -.1,
          y: 1.2,
        },
        xaxis:{
          title: 'time',
          fixedrange: true, //disable zoom
          showgrid: false //disable grid for x axis
        },
        yaxis:{
          title: 'reading value',
          fixedrange: true //disable zoom
        }
      },
      options:{
        'displayModeBar': false,
      }
    };
  },
  watch:{
    area_data(changed_val){
      var tot_location =  ['Somchair','Sakda','Decha','Kannika','Achara','Busarakhan','Chai','Kohsoom','Tansanee','Boonsri'];
      this.traces = []; //emptying array containing data everytimes that the components is called
      //removing from initial object location with no measurements
      for(var key in changed_val){
        if(key != 'year'){
          if (changed_val[key]['value'].length === 0){
            delete changed_val[key];
          }
        }
      }
      //correct_order is used to set always the same color for each location
      var correct_order = ['Tansanee', 'Achara','Decha','Somchair','Busarakhan','Sakda','Kohsoom','Kannika','Chai','Boonsri']
      var location = Object.keys(changed_val); //list of location that has measurements
      location = location.slice(0,-1);
      for(var n=0; n< correct_order.length; n++){
        if (location.includes(correct_order[n]) === true){
          for(var i=0; i<location.length;i++){
            if(location[i] === correct_order[n]){
              var result = {
                x: changed_val[location[i]]['date'], //dates
                y: changed_val[location[i]]['value'], //number of measurements
                name:location[i],
                stackgroup: 'one',
              };
              this.traces.push(result);
            }
          }
        }
        else{
          for(var j=0; j<tot_location.length; j++){
            if(tot_location[j] === correct_order[n]){
              var res2 = {
                x: [changed_val['year'][0]+"-01",changed_val['year'][1]+"-01"], //dates
                y: [0,0], //measurements
                name: tot_location[j],
                visible: 'legendonly', //item disabled on legend
                stackgroup: 'one'
              };
              this.traces.push(res2);
            }
          }
        }
      }
    }
  }
};
</script>
