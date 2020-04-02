<template>
  <!-- application container build using mdb bootstrap and BootstrapVue -->
  <div class="container-fluid">
    <h1 align="center" >VAST Challenge 2018 MC2 - Like a duck to water</h1>
    <mdb-row class="mb-4">
      <!-- introduction -->
      <mdb-card class="card-body" style="width:550px; margin-left:1em; padding-left:5em; padding-right:5em; padding-bottom:3em; margin-right:0em; margin-top: 1rem; padding-bottom: 0em;">
        <mdb-card-title>Background</mdb-card-title>
        <mdb-card-text class="intro" style="padding-bottom: 0.5em;">Mistford is a mid-size city located to the southwest of the Boonsong Lekagul
          Wildlife Preserve. Mitch Vogel, a post-doc student, has been discovering signs that,
          all over the preserve, the number of nesting pairs of the Rose-Crested Blue Pipit
          is decreasing. Kasios Furniture Company, an industry located at Mistford, was accused
          for environmental damage to the preserve for both dumping toxic waste and polluting the
          air with chemicals from its manufacturing process, but they denied these accusations.
          To find the truth, Mitch and his colleagues have come forward with water sensor readings
          from rivers and streams in the preserve. These samples, ranging from <b>1998</b> to <b>2016</b>,
          were taken from <b>10</b> different <b>locations</b> scattered throughout the area and contain
          measurements of <b>106 chemicals </b> of possible interest. <br>
          The graph on the right represents the numbers of measurements made for
          each chemical all over the time. Select a slice below and then click
          on a bar to start investigation.
        </mdb-card-text>
        <!-- select slice of number of measurements through radio button -->
        <b-form-group label="Number of measurements(m):" label-align="center">
          <b-form-radio-group
          v-model="radio_button.selected"
          :options="radio_button.options"
          name="radio-inline"
          ></b-form-radio-group>
        </b-form-group>
      </mdb-card>
      <!-- cirular barplot -->
      <mdb-card class="card-body" style="width: 36rem; margin-left:0em; margin-right:1em; margin-top: 1rem; padding-right:0em;">
        <circular v-on:childToParent="childClick" :circular_data="radioButton_slice">
        </circular>
      </mdb-card>
    </mdb-row>
    <h3 class="titolo">{{chemical_selected}}</h3>
    <!-- year range-slider -->
    <div class="sticky-top">
      <b-row align-h="center" class="slider_year">
        <p> Select a temporal slice: </p>
        <vue-slider
        v-model="slider_years.value2"
        :adsorb="true"
        :data="slider_years.data"
        :marks="true"
        :width = "900"
        ></vue-slider>
      </b-row>
    </div>
    <mdb-row class="mb-4">
      <!-- single line plot -->
      <mdb-card class="card-body" style="width:550px; margin-left:1em; padding-bottom:3em; margin-right:0em; margin-top: 1rem; padding-bottom: 0em;">
        <mdb-card-title>Mean monthly values</mdb-card-title>
        <singleLine :line_data="singleline_filtered"></singleLine>
      </mdb-card>
      <!-- stacked area chart -->
      <mdb-card class="card-body" style="width: 36rem; margin-left:0em; margin-right:1em; margin-top: 1rem; padding-right:0em;">
        <mdb-card-title>Monthly values by location
          <b-button class="btn btn-light" v-b-popover.hover.top="'Click on a location to deselect it from the plot. Double click on a location to isolate it.'">
            Info
          </b-button>
        </mdb-card-title>
        <AreaChart :area_data="area_filtered"></AreaChart>
      </mdb-card>
    </mdb-row>
    <mdb-row class="mb-4">
      <!-- bar plot  -->
      <mdb-card class="card-body" style="width:550px; margin-left:1em; margin-right:0em; margin-top: 1rem;">
        <mdb-card-title>Total monthly number of measurements </mdb-card-title>
        <barPlot :bar_data="barplot_filtrato"></barPlot>
      </mdb-card>
      <!-- stacked bar plot -->
      <mdb-card class="card-body" style="width:36rem; margin-left:0em; margin-right:1em; margin-top: 1rem; padding-right:0em;">
        <mdb-card-title>Monthly number of measurements by location
          <b-button class="btn btn-light" v-b-popover.hover.top="'Click on a location to deselect it from the plot. Double click on a location to isolate it.'">
            Info
          </b-button></mdb-card-title>
          <barStacked :stackedBar_data="barStacked_filtered"></barStacked>
        </mdb-card>
      </mdb-row>
      <mdb-row class="mb-4">
        <!-- mutilines plot for comparison -->
        <mdb-card class="card-body" style="width: 1000px; margin-left:1em; margin-right:1em; margin-top: 1rem; padding-right:0em;">
          <mdb-card-title>Compare {{chemical_selected}} values with others chemicals ones
            <b-button class="btn btn-light" v-b-popover.hover.top="'Deselect element by clicking on the legend. It is advisable to select four chemicals at a time.'">
              Info
            </b-button></mdb-card-title>
            <mdb-col></mdb-col>
            <!-- select chemicals for comparison -->
            <div class="d-flex justify-content-center">
              <b-form-group label="Select a chemical:" label-align="center">
                <b-form-select v-model="chemical_select.selected" :options="chemical_select.options" size="sm">
                </b-form-select>
              </b-form-group>
            </div>
            <mdb-col></mdb-col>
            <multiLines v-on:deselect_chem="remove_chem" :multilines_data="sel_comparison"></multiLines>
          </mdb-card>
        </mdb-row>
      </div>
    </template>

    <script>
    import Vue from 'vue'
    //import all module from d3
    import * as d3 from 'd3'
    //import crossfilter for data handling
    import crossfilter from 'crossfilter';
    //import BootstrapVue components
    import { BootstrapVue, IconsPlugin } from 'bootstrap-vue'
    Vue.use(BootstrapVue)
    import 'bootstrap/dist/css/bootstrap.css'
    import 'bootstrap-vue/dist/bootstrap-vue.css'
    //import mdb bootstrap components
    import { mdbCard, mdbCardTitle, mdbCardText, mdbRow, mdbCol } from 'mdbvue';
    //import vue-slider component
    import VueSlider from 'vue-slider-component'
    import 'vue-slider-component/theme/default.css'
    //import graph components
    import circular from './components/circular_barplot';
    import singleLine from './components/single_line';
    import AreaChart from './components/stacked_area';
    import barPlot from './components/bar_plot';
    import barStacked from './components/barplot_stacked';
    import multiLines from './components/multiple_lines';

    //crossfilter data management
    let cf; // crossfilter instance
    let dChemicals; // dimension for Chemicals
    export default {
      name: 'app',
      components:{
        VueSlider,
        mdbCard,
        mdbCardTitle,
        mdbRow,
        mdbCol,
        mdbCardText,
        circular,
        singleLine,
        AreaChart,
        barPlot,
        barStacked,
        multiLines,
      },
      data() {
        return {
          radio_button:{
            selected: 'All',
            options: [
              { text: 'All', value: 'All' },
              { text: 'm < 200', value: 'm < 200' },
              { text: '200 < m < 2000', value: '200 < m < 2000' },
              { text: 'm > 2000', value: 'm > 2000' }
            ]
          },
          slider_years:{
            value2:[1998,2016],
            data:[],
            marks:[],
          },
          chemical_select:{
            selected: 'Water temperature',
            options: [],
          },
          chemical_selected : 'Water temperature',
          circularPlot_rep:[],
          res_area:{},
          multilines_rep:{},
          barPlot_rep:{},
          res_barStacked:{},
          stackedBar_rep:{},
          singleLine_rep:{},
          stackedArea_rep:{},
          sel_comparison:[],
          area_filtered:{},
          barStacked_filtered:{},
          bar_plot_selected:[],
          barplot_filtrato:[],
          array_selected:[],
          years:[0,0],
          cnt:0,
          min:0,
          max:0,
          radioButton_slice:[],
          singleline_filtered:[],
          res_obj:[],
          compared_chem:[],
        };
      },
      mounted(){
        //data manipulation for circular barplot
        d3.csv("./data/waterways.csv").then((res) => {
          var reports = res.map((d) => {
            const r = {
              id : +d.id,
              value : +d.value,
              location: d.location,
              sample_date : d["sample date"],
              measure : d.measure
            };
            return r;
          });
          // initialize Crossfilter
          cf = crossfilter(reports);
          dChemicals = cf.dimension(d => d.measure);
          //circularPlot_rep is an array where each element is an object having as key chemical name and as value total number of measurements
          this.circularPlot_rep = dChemicals.group().reduceCount().all();
          this.circularPlot_rep.sort(function(a,b){
            return b.value - a.value;
          });
          this.radioButton_slice = this.circularPlot_rep;
          //extracting all chemicals name to use them in the "comparison select"
          this.circularPlot_rep.map(d=>this.chemical_select.options.push(d.key));
        }),
        //data manipulation for bar plot
        d3.csv("./data/df_misurations.csv").then((res) => {
          var reports = res.map((d) => {
            const r = {
              id : +d.id,
              chemical : d.chemical,
              year: d.date,
              misurations: d.misurations
            };
            return r;
          });
          //barPlot_rep is an object that has one key for each chemical and as value, for each of them, array of dates and array of number of measurements
          var step;
          for (step=0; step<reports.length; step++){
            var elem = reports[step];
            var name = elem.chemical;
            if (!(name in this.barPlot_rep)){
              this.barPlot_rep[name] = [[elem.year],[elem.misurations]];
            } else {
              this.barPlot_rep[name][0].push(elem.year)
              this.barPlot_rep[name][1].push(elem.misurations)
            }
          }
          //Water temperature is the default chemical selected
          this.barplot_filtrato = this.barPlot_rep['Water temperature'];
        })
        //data manipulation for stacked bar plot
        d3.csv("./data/df_misurations_location.csv").then((res) => {
          var reports = res.map((d) => {
            const r = {
              id : +d.id,
              chemical : d.chemical,
              year: d.date,
              location : d.location,
              misurations: d.misurations
            };
            return r;
          });
          //stackedBar_rep is an object that has one key for each chemical and as value, for each of them, array of dates, array of locations and array of number of measurements for each location
          var step;
          for (step=0; step<reports.length; step++){
            var elem = reports[step];
            var name = elem.chemical;
            if (!(name in this.stackedBar_rep)){
              this.stackedBar_rep[name] = [[elem.year],[elem.location],[elem.misurations]];
            } else {
              this.stackedBar_rep[name][0].push(elem.year)
              this.stackedBar_rep[name][1].push(elem.location)
              this.stackedBar_rep[name][2].push(elem.misurations)
            }
          }
          //Water temperature is the default chemical selected
          this.manipulation_stackedBar(this.stackedBar_rep['Water temperature']);
        })
        //data manipulation for stacked area chart
        d3.csv("./data/df_stackedArea.csv").then((res) => {
          var reports = res.map((d) => {
            const r = {
              id : +d.id,
              chemical : d.chemical,
              year: d.year,
              location: d.location,
              value: d.value
            };
            return r;
          });
          //stackedBar_rep is an object that has one key for each chemical and as value, for each of them, array of dates, array of locations and array of measured value for each location
          var step;
          for (step=0; step<reports.length; step++){
            var elem = reports[step];
            var name = elem.chemical;
            if (!(name in this.stackedArea_rep)){
              this.stackedArea_rep[name] = [[elem.year],[elem.location],[elem.value]];
            } else {
              this.stackedArea_rep[name][0].push(elem.year)
              this.stackedArea_rep[name][1].push(elem.location)
              this.stackedArea_rep[name][2].push(elem.value)
            }
          }
          //Water temperature is the default chemical selected
          this.manipulation_stackedArea(this.stackedArea_rep['Water temperature']);
        })
        //data manipulation for single line plot with mean
        d3.csv("./data/single_line.csv").then((res) => {
          var reports = res.map((d) => {
            const r = {
              id : +d.id,
              chemical : d.chemical,
              year: d.date,
              anno: d.year,
              value: d.value
            };
            return r;
          });
          //singleLine_rep is an object that has one key for each chemical and as value, for each of them, array of dates, array of years and array of measured value for each date
          var step;
          for (step=0; step<reports.length; step++){
            var elem = reports[step];
            var name = elem.chemical;
            if (!(name in this.singleLine_rep)){
              this.singleLine_rep[name] = [[d3.timeParse("%Y-%m")(elem.year)],[elem.value],[elem.anno]];
            } else {
              this.singleLine_rep[name][0].push(d3.timeParse("%Y-%m")(elem.year))
              this.singleLine_rep[name][1].push(elem.value)
              this.singleLine_rep[name][2].push(elem.anno)
            }
          }
          //Water temperature is the default chemical selected
          this.manipulation_singleLine(this.singleLine_rep['Water temperature']);
        })
        //data manipulation for multilines graph for comparison
        d3.csv("./data/df_normalized.csv").then((res) => {
          var reports = res.map((d) => {
            const r = {
              id : +d.id,
              chemical : d.chemical,
              year: d.date,
              value: d.value,
              anno: d.year,
            };
            return r;
          });
          //multilines_rep is an object that has one key for each chemical and as value, for each of them, array of dates, array of years and array of measured value for each date
          //the difference with previous object is that, here, values are normalized in order to make comparison among chemicals
          var step;
          for (step=0; step<reports.length; step++){
            var elem = reports[step];
            var name = elem.chemical;
            if (!(name in this.multilines_rep)){
              this.multilines_rep[name] = [[d3.timeParse("%Y-%m")(elem.year)],[elem.value],[elem.anno]];
            } else {
              this.multilines_rep[name][0].push(d3.timeParse("%Y-%m")(elem.year))
              this.multilines_rep[name][1].push(elem.value)
              this.multilines_rep[name][2].push(elem.anno)
            }
          }
          this.manipulation_multiLines(this.multilines_rep['Water temperature'],'Water temperature');
        })
      },
      methods:{
        //childClick is the function that controls behaviour of the whole application: through v-on directive this function is executed when a bar of the circular plot is selected
        childClick(value) {
          this.chemical_selected = value; //selected chemical
          var min_year = Math.min.apply(Math, this.singleLine_rep[value][2]); //min year in which selected chemical has measurements
          var max_year = Math.max.apply(Math, this.singleLine_rep[value][2]); //max year in which selected chemical has measurements
          this.years = [min_year,max_year];
          this.compared_chem = []; //emptying array of compared chemicals to plot (at the beginning) only the selected one
          this.array_selected = [];
          this.array_selected.push(value);
          if (this.chemical_select.selected === value){
            this.manipulation_multiLines(this.multilines_rep[value],value); //changing chemical in multilines plot
          }
          else{
            this.chemical_select.selected = value;
          }
          this.manipulation_stackedArea(this.stackedArea_rep[value]); //changing chemical in stacked area chart
          this.manipulation_singleLine(this.singleLine_rep[value]); //changing chemical in single line plot
          this.manipulation_stackedBar(this.stackedBar_rep[value]); //changing chemical in stacked bar plot
          this.bar_plot_selected = this.barPlot_rep[value]; //changing chemical in bar plot
        },
        //remove_chem removes chemicals from the plot: through v-on directive this function is executed when an item of the legend is clicked
        remove_chem(val){
          if (val != this.chemical_selected){ //chemical selected in the circular plot can't be removed
            this.sel_comparison = this.sel_comparison.filter(function (el) {
              return el.chemical != val;
            });
            this.compared_chem = this.compared_chem.filter(function (el) {
              return el.chemical != val;
            });
          }
          //removing this chemical from array of selected chemicals
          if(val != this.chemical_selected){
            this.array_selected = this.array_selected.filter(function (str) { return str.indexOf(val) === -1; });
          }
        },
        //manipulation_multiLines is a function used to manipulate chemical data to fit the multilines plot: it is called everytime that user wants to make comparison
        manipulation_multiLines(my_object,value){
          if (typeof(my_object) != 'undefined') {
            this.cnt = 0; //used to verify if selected chemical has misuration in the time range considered
            for (var j=0; j<my_object[0].length; j++){
              //every measurement for that chemical is an object with date,value,year and chemical name
              var obj = {};
              obj["date"] = my_object[0][j];
              obj["value"] = my_object[1][j];
              obj["year"] = my_object[2][j];
              obj["chemical"] = value;
              if(my_object[2][j] >= this.years[0] && my_object[2][j] <= this.years[1]){ //only measurements that fall in the selected time range are shown
                this.compared_chem.push(obj);
                this.cnt += 1;
              }
            }
            this.sel_comparison = this.compared_chem; //sel_comparison is given as props to the mutilines component
          }
          else{
            this.cnt += 1;
          }
        },
        //manipulation_singleLine is a function used to manipulate chemical data to fit the single line plot
        manipulation_singleLine(my_object){
          this.slider_years.value2 = [0,0]; //reset time slider
          var lista2=[];
          for (var j=0; j<my_object[0].length; j++){
            //every measurement for that chemical is an object with date,value and year
            var obj = {};
            obj["date"] = my_object[0][j];
            obj["value"] = my_object[1][j];
            obj["year"] = my_object[2][j];
            lista2.push(obj);
          }
          this.res_obj = lista2;
          //handling time range slider for each chemical
          var tot_years=[]; //array of years in which selected chemical has measurements
          this.res_obj.map(d=>tot_years.push(parseInt(d.year)));
          function Unique_years(array){
            var distinct_years= [];
            for(var i=0; i < array.length; i++){
              if(distinct_years.indexOf(array[i]) === -1) {
                distinct_years.push(array[i]);
              }
            }
            return distinct_years;
          }
          this.slider_years.data = Unique_years(tot_years); //distinct years for time range slider
          this.slider_years.value2[0] = Math.min.apply(Math, this.slider_years.data); //min year in range
          this.slider_years.value2[1] = Math.max.apply(Math, this.slider_years.data); //max year in range
          this.singleline_filtered = this.res_obj; //singleline_filtered is given as props to the single line component
        },
        //manipulation_stackedArea is a function used to manipulate chemical data to fit the stacked area chart
        manipulation_stackedArea(my_object){
          //find min and max years to handle null locations in the chart
          this.min = 2016;
          this.max = 0;
          //every measurement for that chemical is an object with date, location, value and year
          var obj={};
          for(var i=0;i<my_object[1].length;i++){
            var year = my_object[0][i];
            var luogo = my_object[1][i];
            var val = my_object[2][i];
            if( parseInt(year.slice(0,4)) < this.min){
              this.min = year.slice(0,4);
            }
            if (parseInt(year.slice(0,4)) > this.max){
              this.max = year.slice(0,4);
            }
            if(!(luogo in obj)){
              obj[luogo] = {'value':[] , 'date':[]};
            }
            obj[luogo]['value'].push(parseFloat(val));
            obj[luogo]['date'].push(year);
          }
          obj['year'] = [this.min,this.max];
          this.res_area = obj;
          this.area_filtered = this.res_area; //area_filtered is given as props to the AreaChart component
        },
        manipulation_stackedBar(my_object){
          //every measurement for that chemical is an object with date, location, number of measurements and year
          var obj={};
          for(var i=0;i<my_object[1].length;i++){
            var year = my_object[0][i];
            var luogo = my_object[1][i];
            var  misuraz = my_object[2][i];
            if(!(luogo in obj)){
              obj[luogo] = {'misuration':[] , 'date':[]};
            }
            obj[luogo]['misuration'].push(parseFloat(misuraz));
            obj[luogo]['date'].push(year);
          }
          obj['year']=[this.min,this.max];
          this.res_barStacked = obj;
          this.barStacked_filtered = this.res_barStacked; //barStacked_filtered is given as props to the barStacked component
        },
      },
      watch:{
        //radio button handler: when a slice is selected function handler is called
        radio_button:{
          handler(changed_val){
            this.radioButton_slice = [];
            if ( changed_val.selected=== 'm > 2000' ){
              this.radioButton_slice = this.circularPlot_rep.slice(0,32); //only chemical in this range are shown in circular plot
            }
            else if (changed_val.selected === '200 < m < 2000'){
              this.radioButton_slice = this.circularPlot_rep.slice(32,65); //only chemical in this range are shown in circular plot
            }
            else if (changed_val.selected === 'm < 200'){
              this.radioButton_slice = this.circularPlot_rep.slice(65,106); //only chemical in this range are shown in circular plot
            }
            else{
              this.radioButton_slice = this.circularPlot_rep; //otherwise all chemical are shown in circular plot
            }
          },
          deep:true,
        },
        //slider year handler: when a time range is selected function handler is called
        slider_years:{ //time range slider
          handler(changed_val){
            this.years[0] = changed_val.value2[0]; //min year in the range
            this.years[1] = changed_val.value2[1]; //max year in the range
            //filter years in single line plot
            this.singleline_filtered = this.res_obj.filter(function (el) { //only measurements in the time range are selected
              return parseInt(el.year) >= changed_val.value2[0] && parseInt(el.year) <= changed_val.value2[1];
            });
            //filter year in multiline plot
            this.compared_chem = []; //emptying list of compared chemical_select
            //pushing in the list all of the chemical in array_selected
            for(var n = 0; n<this.array_selected.length; n++){
              this.manipulation_multiLines(this.multilines_rep[this.array_selected[n]],this.array_selected[n]);
            }
            //filter year in stacked area chart
            this.area_filtered = {};
            var keys= Object.keys(this.res_area);
            keys = keys.slice(0,-1);
            for (var elem=0; elem < keys.length; elem++){
              this.area_filtered[keys[elem]] = {'value':[], 'date':[]};
            }
            for (var i=0; i<keys.length; i++){
              for(var j=0; j<this.res_area[keys[i]]['value'].length; j++){
                if (parseInt(this.res_area[keys[i]]['date'][j].slice(0,4)) >= changed_val.value2[0] && parseInt(this.res_area[keys[i]]['date'][j].slice(0,4)) <= changed_val.value2[1]){
                  this.area_filtered[keys[i]]['value'].push(this.res_area[keys[i]]['value'][j]);
                  this.area_filtered[keys[i]]['date'].push(this.res_area[keys[i]]['date'][j]);
                }
              }
            }
            this.area_filtered['year'] = [changed_val.value2[0],changed_val.value2[1]]; //adding year to object to filter values
            //filter year in bar chart
            this.barplot_filtrato = [];
            this.barplot_filtrato[0] = [];
            this.barplot_filtrato[1] = [];
            for (var x=0; x<this.barPlot_rep[this.chemical_selected][0].length; x++){
              if (parseInt(this.barPlot_rep[this.chemical_selected][0][x].slice(0,4)) >= changed_val.value2[0] && parseInt(this.barPlot_rep[this.chemical_selected][0][x].slice(0,4)) <= changed_val.value2[1]){
                this.barplot_filtrato[0].push(this.barPlot_rep[this.chemical_selected][0][x]);
                this.barplot_filtrato[1].push(this.barPlot_rep[this.chemical_selected][1][x]);
              }
            }
            //filter years in stacked bar chart
            this.barStacked_filtered = {};
            var keys2= Object.keys(this.res_barStacked);
            keys2 = keys2.slice(0,-1);
            for (var elem2=0; elem2 < keys2.length; elem2++){
              this.barStacked_filtered[keys2[elem2]] = {'misuration':[], 'date':[]};
            }
            for (var a=0; a<keys2.length; a++){
              for(var b=0; b<this.res_barStacked[keys2[a]]['misuration'].length; b++){
                if (parseInt(this.res_barStacked[keys2[a]]['date'][b].slice(0,4)) >= changed_val.value2[0] && parseInt(this.res_barStacked[keys2[a]]['date'][b].slice(0,4)) <= changed_val.value2[1]){
                  this.barStacked_filtered[keys2[a]]['misuration'].push(this.res_barStacked[keys2[a]]['misuration'][b]);
                  this.barStacked_filtered[keys2[a]]['date'].push(this.res_barStacked[keys2[a]]['date'][b]);
                }
              }
            }
            this.barStacked_filtered['year'] = [changed_val.value2[0],changed_val.value2[1]]; //adding year to filter values
          },
          deep:true,
        },
        //chemical select handler: when a new chemical is selected function handler is called
        chemical_select:{
          handler(changed_val){
            //if selected chemical is not in array of selected chemical add it
            if( this.array_selected.includes(changed_val.selected) === false ){
              this.compared_chem = [];
              this.array_selected.push(changed_val.selected);
              for(var i = 0; i<this.array_selected.length; i++){
                this.manipulation_multiLines(this.multilines_rep[this.array_selected[i]],this.array_selected[i]);
              }
              //if selected chemical has no measurements show an alert message
              if(this.cnt === 0){
                alert(changed_val.selected+' has no misurations in this time period, try a different time range')
              }
            }
            //if chemical selected is already in the array of selected chemical show an alert message
            else if (changed_val.selected != this.chemical_selected){
              alert(changed_val.selected+ " is alredy selected");
            }
          },
          deep:true,
        },
      }
    };
    </script>

    <style>
    header {
      background-color: #FF6F61;
      margin-right:0;
    }
    h1 {
      display: block;
      font-size: 2em;
      margin-top: 0.67em;
      margin-left: 0;
      margin-right: 0;
    }
    .slider_year{
      margin-bottom: 50px;
    }
    h3 {
      text-align: center;
      margin-bottom: 0em;
    }
    p{
      margin-right: 2em;
    }
    .sticky-top{
      background-color: white;
      padding-top: 1.9em;
      height:5em;
      margin-bottom: 50px;
    }
    .form-group{
      height: 5em;
    }
    .intro{
      text-align: justify;
      line-height: 2;
    }
    .container-fluid{
      font-family: 'Avenir', Helvetica, Arial, sans-serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
      text-align: center;
      background-color: white;
    }
    </style>
