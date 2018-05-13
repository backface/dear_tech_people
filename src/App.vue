<template>
  <center>
  <h1 class="title">Dear Tech People</h1> 
  
  <p>A race and gender diversity ranking of 100 top tech companies </p>
  <p>&nbsp;</p>
  <div id="app">
    <DearTechPeopleChart />
  </div>
  <p>A <a href="https://www.vizforsocialgood.com/join-a-project/2018/4/20/dear-tech-people">viz for social good</a> challenge, data: <a href="http://www.deartechpeople.com/">www.deartechpeople.com</a>,  visualized by Michael Aschauer, &lt;<a href="http://backface.io">m.ash.to</a>&gt; 2018</a></p>
  </center>

</template>

<script>
require("./css/bulma.min.css");

import DearTechPeopleChart from './components/DearTechPeopleChart.vue'

export default {
  name: 'app',
  components: {
    DearTechPeopleChart
  },
  created: function() {
        //window.document.title = "Dear Tech People!";
  }
}
</script>

<style>

body, html {
		width:100%;
		font-size:12px
	}
	
	body { padding-top:20px}
	
	a { color:#92278f}
	
	#app {
		width:960px;
	}
	
	.title {text-transform:uppercase}
	
	
	svg { font-size:12px}			
	
	.ranking { 
		font-size: 12px;
		stroke:#ccc;
	}
	.ratio_note { 
		font-size: 12px;
		fill-color:#666666;
		font-weight:bold;
		font-family: FontAwesome;
	}
	.top {
		font-family: FontAwesome;
		font-size: 12px;
		fill:#999999;
	}
	.line {
		stroke:#666666;
		stroke-width:0.5;
	}
	.label {font-size:14px}
	.columns, .field-label {padding:0;margin:0}
	.field-label {padding-right:7px}
	
	.d3-tip { 
		background:rgba(255,255,255,.9);
		padding:5px; 
		border: 1px solid #666 ;
		box-shadow: 5px 5px 5px #888888;
		z-index:10;
	}
	.d3-tip .sep {border-top: 1px solid #ddd}
	.d3-tip td {padding-right:10px;}
	.d3-tip table {margin-top:5px;}
	.d3-tip .subinf {color:#999999}
	
	.select:not(.is-multiple):not(.is-loading)::after { border-color:#666666 }
	
	[v-cloak] {
		display: none;
	}			

</style>
