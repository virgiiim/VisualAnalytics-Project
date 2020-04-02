<template>
  <div id="circular_barplot"> <!-- div that contains svg -->
  </div>
</template>

<script>
let d3 = require('d3');
import * as Radial from './d3-scale-radial.js';
export default {
  name: "circular_plot",
  props: {
    circular_data: Array //data passed through root component App.vue
  },
  data(){
    return {
      all:{},
      margin:{},
      width:0,
      height:0,
      innerRadius:0,
      outerRadius:0,
    };
  },
  mounted(){
    // set the dimensions and margins of the graph
    this.margin = {top: 59, right: 10, bottom: 10, left: 10};
    this.width = 598 - this.margin.left - this.margin.right;
    this.height = 568 - this.margin.top - this.margin.bottom;
    this.innerRadius = 80,
    this.outerRadius = Math.min(this.width, this.height) / 2.5; //the outerRadius goes from the middle of the SVG area to the border
    //svg
    var svg = d3.select("#circular_barplot")
    .append("svg")
    .attr("width", this.width + this.margin.left + this.margin.right)
    .attr("height", this.height + this.margin.top + this.margin.bottom)
    .classed("my_svg2",true)
    .append("g") //main group
    .classed("svg", true)
    .attr("transform",
    "translate(" + (this.width / 2 + this.margin.left) + "," + (this.height / 2 + this.margin.top) +")");
  },
  methods:{
    build_graph(){
      //removing previouses svg elements everytime that new data are passed to the component
      d3.selectAll("g.my_label").remove();
      //X axis
      var x = d3.scaleBand()
      .range([0, 2 * Math.PI])
      .align(0)
      .domain(this.all.map(function(d) { return d.key; })); //the domain of the X axis is the list of chemicals
      //Y axis
      var y = Radial.scaleRadial()
      .range([this.innerRadius, this.outerRadius])
      .domain([0, d3.max(this.all, function(d) { return d.value; })]); //the domain of the Y axis is the list of measurements
      //add tooltip to each bar
      var div = d3.select("#circular_barplot").append("div")
      .attr("class", "tooltip-donut")
      .style("opacity", 0);
      //add the bars for the circular barplot
      var bars = d3.select("g.svg")
      .selectAll("path")
      .data(this.all);
      bars.enter()
      .append("path")
      .merge(bars)
      .attr("fill", function(d) { //different colors based on chemical measurements
        if (d.value > 2000) {
          return "#457371";
        } else if (d.value > 200 && d.value < 2000) {
          return "#03A89E";
        }
        return "#99CDC9";
      })
      .attr("d", d3.arc()     // imagine your doing a part of a donut plot
      .innerRadius(this.innerRadius)
      .outerRadius(function(d) { return y(d.value); })
      .startAngle(function(d) { return x(d.key); })
      .endAngle(function(d) { return x(d.key) + x.bandwidth(); })
      .padAngle(0.035) //controlla lo spazio fra le barre
      .padRadius(this.innerRadius))
      .on("click", d => {this.$emit('childToParent', d.key)}) //passing to the root component the name of the chemical clicked to select
      .on("mouseover", function(d) {
        d3.select(this).style("cursor", "pointer");//hand cursor
        d3.select(this).style("stroke", 'black'); //black stroke on the bar
        div.transition() //tooltip when mouseover
        .duration(50)
        .style("opacity", 0.9);
        div.html(d.key + "</br>" + d.value + " measurements")
        .style("left", (d3.event.pageX -800) + "px")
        .style("top", (d3.event.pageY -200) + "px");
      })
      .on("mouseout", function(d) {
        d3.select(this).style("stroke", "none"); //remove stroke
        div.transition() //remove tooltip when mouseout
        .duration('50')
        .style("opacity", 0);
      })
      bars.exit().remove();
      //add the labels of each bar with the name of the chemical
      var labels = d3.select("g.svg")
      .selectAll("g")
      .data(this.all);
      labels.enter()
      .append("g")
      .classed("my_label", true)
      .merge(labels)
      .attr("text-anchor", function(d) { return (x(d.key) + x.bandwidth() / 2 + Math.PI) % (2 * Math.PI) < Math.PI ? "end" : "start"; })
      .attr("transform", function(d) { return "rotate(" + ((x(d.key) + x.bandwidth() / 2) * 180 / Math.PI - 90) + ")"+"translate(" + (y(d.value)+10) + ",0)"; })
      .append("text")
      .text(function(d){return(d.key)})
      .attr("transform", function(d) { return (x(d.key) + x.bandwidth() / 2 + Math.PI) % (2 * Math.PI) < Math.PI ? "rotate(180)" : "rotate(0)"; })
      .style("font-size", "9px")
      .attr("alignment-baseline", "middle")
      labels.exit().remove();
    }
  },
  watch:{
    circular_data:{
      handler(changed_val){
        this.all = changed_val; //all_data
        this.build_graph(); //this function allows to build the graph everytime that new data are passed
      }
    }
  }
};
</script>
<style>
div.tooltip-donut {
  position: absolute;
  text-align: center;
  padding: .5rem;
  background: #FFFFFF;
  color: #313639;
  border: 1px solid #313639;
  border-radius: 8px;
  pointer-events: none;
  font-size: 1rem;
}
</style>
