<template>
  <div id="multiLines_plot"> <!-- div that contains svg -->
  </div>
</template>
<script>
let d3 = require('d3');
export default {
  name: "multiLines",
  props: {
    multilines_data: Array, //data passed through root component App.vue
  },
  data(){
    return {
      max:[],
      newall:[],
      margin:{},
      width:0,
      height:0,
    };
  },
  mounted(){
    //set the dimensions and margins of the graph
    this.margin = {top: 30, right: 30, bottom: 30, left: 250};
    this.width = 1280 - this.margin.left - this.margin.right;
    this.height = 500 - this.margin.top - this.margin.bottom;
    //svg
    var svg = d3.select("#multiLines_plot")
    .append("svg")
    .classed("my_svg", true)
    .attr("width", this.width + this.margin.left + this.margin.right)
    .attr("height", this.height + this.margin.top + this.margin.bottom)

    var focus = d3.select("svg.my_svg").append("g") //main group
    .attr("id","g.focus")
    .attr("class", "focus")
    .attr("width", 500)
    .attr("transform", "translate(" + this.margin.left + "," + this.margin.top + ")");
  },
  methods:{
    build_graph(){
      //removing previouses svg elements everytime that new data are passed to the component
      d3.selectAll("path.line").remove();
      d3.selectAll("text.my_text2").remove();
      d3.select("g.focus").selectAll("rect").remove();
      d3.selectAll("g.grid").remove();
      d3.select("g.focus").select(".axis--y").remove();
      d3.select("g.focus").select(".axis--x").remove();
      d3.select("text.xlabel").remove();
      d3.select("text.ylabel").remove();
      //set color palette for lines and items of legend
      var color = d3.scaleOrdinal()
      .range(["#1f77b4", "#ff7f0e", "#2ca02c", "#d62728", "#9467bd", "#8c564b", "#e377c2", "#7f7f7f", "#bcbd22", "#17becf"]);
      //group the data: I want to draw one line per group
      var sumstat = d3.nest() // nest function allows to group the calculation per level of a factor
      .key(function(d) { return d.chemical;})
      .entries(this.newall);
      //list of group names
      var res = sumstat.map(function(d){ return d.key })
      //X axis
      var x = d3.scaleTime()
      .domain(d3.extent(this.newall, function(d) { return d.date; }))
      .range([ 0, 770 ]);
      //append X axis to a group
      d3.select("g.focus").append("g")
      .attr("class", "axis axis--x")
      .attr("width", 600)
      .attr("transform", "translate(0," + this.height + ")")
      .call(d3.axisBottom(x).ticks(5));
      //Y axis
      var y = d3.scaleLinear()
      .domain([0, d3.max(this.newall, function(d){return +d.value;})])
      .range([ this.height, 0 ]);
      //append Y axis to a group
      d3.select("g.focus").append("g")
      .attr("class", "axis axis--y")
      .call(d3.axisLeft(y));
      //Y axis gride
      function make_y_gridlines() {
        return d3.axisLeft(y)
        .ticks(10)
      }
      //add the Y gridlines
      d3.select("g.focus").append("g")
      .attr("class", "grid")
      .call(make_y_gridlines()
      .tickSize(-770)
      .tickFormat("")
    )
    //add lines
    var focus1 = d3.select("g.focus").selectAll(".line")
    .data(sumstat)

    focus1
    .enter()
    .append("path")
    .merge(focus1)
    .attr("class", "line")
    .attr("fill", "none")
    .attr("stroke", function(d){ return color(d.key) })
    .attr("stroke-width", 1.5)
    .attr("id", function(d) { return  d.key })
    .style("opacity",1)
    .attr("clip-path", "url(#clip)")
    .attr("d", function(d){
      return d3.line()
      .curve(d3.curveMonotoneY)
      .x(function(d) { return x(d.date); })
      .y(function(d) {return y(+d.value); })(d.values)
    })
    focus1.exit().remove();
    //legend
    var size = 20;
    //legend rects
    var my_rect = d3.select("g.focus").selectAll("myrect")
    .data(res)

    my_rect
    .enter()
    .append("rect")
    .merge(my_rect)
    .attr("class", function(d){ return d;})
    .attr("id", function(d){return d;})
    .attr("x", 775)
    .attr("y", function(d,i){ return 60 + i*(size+5)}) // 100 is where the first dot appears. 25 is the distance between dots
    .attr("width", 20)
    .attr("margin-right",100)
    .attr("height", 20)
    .style("fill", function(d){ return color(d)})
    .on("click", d => {this.$emit('deselect_chem', d)}) //passing to the root component the name of the chemical clicked to remove
    .on("mouseover", function(d) {
      d3.select(this).style("cursor", "pointer"); //hand cursor
      d3.select(this).style("stroke", 'black'); //black stroke on the bar
    })
    .on("mouseout", function(d) {
      d3.select(this).style("stroke", "none"); //remove black stroke
    })

    my_rect.exit().remove();
    //legend label
    var my_label = d3.select("g.focus").selectAll("mylabels")
    .data(res)

    my_label
    .enter()
    .append("text")
    .classed("my_text2", true)
    .merge(my_label)
    .attr("x", 775 + size*1.2)
    .attr("y", function(d,i){ return 60 + i*(size+5) + (size/2)}) // 100 is where the first dot appears. 25 is the distance between dots
    .style("fill", function(d){ return color(d)})
    .text(function(d){ return d})
    .attr("text-anchor", "left")
    .style("alignment-baseline", "middle")

    my_label.exit().remove();

    //text label for the x axis
    d3.select("svg.my_svg").append("text")
    .classed("xlabel",true)
    .attr("x", 650)
    .attr("y", 500 )
    .style("text-anchor", "middle")
    .style("opacity", 0.8)
    .style("font-size", 15)
    .text("time");
    //text label for the y axis
    d3.select("svg.my_svg").append("text")
    .classed("ylabel",true)
    .attr("transform", "rotate(-90)")
    .attr("y", 200)
    .attr("x", -250)
    .attr("dy", "1em")
    .style("text-anchor", "middle")
    .style("opacity", 0.8)
    .style("font-size", 15)
    .text("reading value");
  }
},
watch:{
  multilines_data:{
    handler(changed_val){
      this.newall = changed_val; //all_data
      this.build_graph(); //this function allows to build the graph everytime that new data are passed
    },
    deep: true
  },
}
};
</script>
<style>
.line {
  fill: none;
}
.grid line {
  stroke: lightgrey;
  stroke-opacity: 0.4;
  shape-rendering: crispEdges;
}
.grid path {
  stroke-width: 0;
}
</style>
