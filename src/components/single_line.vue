<template>
  <div id="single_line">   <!-- div that contains svg -->
  </div>
</template>
<script>
let d3 = require('d3');
export default {
  name: "singleline",
  props: {
    line_data: Array, //data passed through root component App.vue
  },
  data(){
    return {
      margin:{},
      width:0,
      height:0,
      all_data:[], //data: array in which each element is an object
      values:[], //array of values
    };
  },
  mounted(){
    //set the dimensions and margins of the graph
    this.margin = {top: 10, right: 30, bottom: 40, left: 60};
    this.width = 680 - this.margin.left - this.margin.right;
    this.height = 400 - this.margin.top - this.margin.bottom;
    //svg
    var svg = d3.select("#single_line")
    .append("svg")
    .classed("svgs", true)
    .attr("width", this.width + this.margin.left + this.margin.right)
    .attr("height", this.height + this.margin.top + this.margin.bottom)
    .append("g") //main group
    .classed("svg3", true)
    .attr("transform",
    "translate(" + this.margin.left + "," + this.margin.top + ")")
  },
  methods:{
    build_graph(){
      //removing previouses svg elements everytime that new data are passed to the component
      d3.select("path.path_value").remove();
      d3.select("g.x-axis").remove();
      d3.select("g.y-axis").remove();
      d3.select("line").remove();
      d3.select("text.my_text").remove();
      d3.select("text.label_x").remove();
      d3.select("text.label_y").remove();
      //X axis
      var x = d3.scaleTime()
      .domain(d3.extent(this.all_data, function(d) { return d.date; }))
      .range([ 0, this.width ]);
      //append X axis to a group
      d3.select("g.svg3").append("g")
      .classed("x-axis", true)
      .attr("transform", "translate(0," + this.height + ")")
      .call(d3.axisBottom(x));
      //add x label
      d3.select("#single_line").append("text")
      .classed("label_y", true)
      .attr("text-anchor", "end")
      .attr("y", 50)
      .attr("x", 900)
      .attr("dy", ".75em")
      .attr("transform", "rotate(90)")
      .style("opacity", 0.9)
      .style("font-size", 14)
      .text("time");
      //Y axis
      var y = d3.scaleLinear()
      .domain([0, d3.max(this.all_data, function(d) { return +d.value; })])
      .range([ this.height, 0 ]);
      //append Y axis to a group
      d3.select("g.svg3").append("g").append("g")
      .classed("y-axis", true)
      .call(d3.axisLeft(y));
      //add y label
      d3.select("g.svg3").append("text")
      .classed("label_x",true)
      .attr("transform", "rotate(-90)")
      .attr("y", -50)
      .attr("x",-200)
      .attr("dy", "1em")
      .style("text-anchor", "middle")
      .style("opacity", 0.9)
      .style("font-size", 14)
      .text("reading value");
      //add line representing chemical values during time
      d3.select("g.svg3").append("path")
      .datum(this.all_data)
      .classed("path_value",true)
      .attr("fill", "none")
      .attr("stroke", "steelblue")
      .attr("stroke-width", 1.5)
      .attr("d", d3.line()
      .x(function(d) { return x(d.date) })
      .y(function(d) { return y(d.value) })
    )
    //add mean line
    d3.select("g.svg3").append("line")
    .style("stroke", "red")
    .attr("x1", 0)
    .attr("y1", y(d3.mean(this.values)))
    .attr("x2", this.width)
    .attr("y2", y(d3.mean(this.values)));
    //add text label at the end of mean line
    d3.select("g.svg3").append("text")
    .classed("my_text", true)
    .attr("transform", "translate(" + (this.width+3) + "," + y(d3.mean(this.values)) + ")")
    .attr("dy", ".35em")
    .attr("text-anchor", "start")
    .style("fill", "red")
    .style("font-size", 11)
    .text("mean");
  }
},
watch:{
  line_data:{
    handler(changed_val){
      this.values = [];
      this.all_data = changed_val;
      this.all_data.map(d=>this.values.push(parseFloat(d.value))); //creating a list containg all chemical values due to compute mean
      this.build_graph(); //this function allows to build the graph everytime that new data are passed
    },
    deep:true,
  }
}
};
</script>
<style>
#single_line {
  margin-top: 0.67em;
  margin-bottom: 0.67em;
  margin-left:0em;
  margin-right:0em;
  padding-left:0em;
  padding-right:0em;
  width:700px;
}
</style>
