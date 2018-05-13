<template>
<div>
		<div class="columns"> 
			<div class="column">
				<div class="field is-horizontal">
					<div class="field-label is-normal">
						<label class="label">Sort</label>  
					</div>
					<div class="field-body">	  
						<div class="select">							  
							  <select id="sortby_options" @change="updateSelection" v-model="sortby" class="select">
								<option v-for="(sortby, index) in sortby_options" :value="sortby" v-text="sortby"></option>
							  </select>
						</div>
					</div>
				</div>
			</div>		
			<div class="column">
				<div class="field is-horizontal">
					<div class="field-label is-normal">
						<label class="label">Industry</label>	
					</div>
					<div class="field-body">	  
						<div class="select ">
							<select id="industry_options" @change="updateSelection" v-model="industry">
							<option v-for="(industry, index) in industry_options" :value="industry" v-text="industry"></option>
							</select>
						</div>
					</div>
				</div>
			</div>
			<div class="column">
				<div class="field is-horizontal">
					<div class="field-label is-normal">
						<label class="label">Customer</label>	
					</div>
					<div class="field-body">	  
						<div class="select">
							<select id="customer_options" @change="updateSelection" v-model="customer">
							<option v-for="(customer, index) in customer_options" :value="customer">{{ customer }}</option>
							</select>
						</div>
					</div>
				</div>
			</div>	
			<div class="column">
				<div class="field is-horizontal">
					<div class="field-label is-normal">
						<label class="label">Location</label>	
					</div>
					<div class="field-body">	  
						<div class="select">
							<select id="location_options" @change="updateSelection" v-model="location">
							<option v-for="(location, index) in location_options" :value="location">{{ location }}</option>
							</select>
						</div>
					</div>
				</div>
			</div>	
			<div class="column">
				<div class="field is-horizontal is-pulled-right">
					<div class="field-label is-normal">
						<label class="label">Role</label>	
					</div>
					<div class="field-body is-pulled-right">	  
						<div class="select">
							<select id="filterby_options" @change="updateSelection" v-model="filterby">
							<option v-for="(filterby, index) in filterby_options" :value="filterby">{{ filterby }}</option>
							</select>
						</div>
					</div>
				</div>
			</div>
			
			
			<div class="column">
				<div class="field is-horizontal">
					<div class="field-label is-normal">
						 <label>Limit</label>  
					</div>
					<div class="field-body">	  
						<div class="select">
						  <select id="limit_options" @change="updateSelection" v-model="limit" class="select">
							<option v-for="(limit, index) in limit_options" :value="limit">{{ limit }}</option>
						  </select>	 
						</div>
					</div>
				</div>
			</div>
			
		</div>
 

	  <div class="chart">
		<svg width="960" height="380"></svg>
	  </div>
</div>	  
	 
</template>

<script>
import * as d3 from 'd3';
import d3Tip from 'd3-tip';

// and then
d3.tip = d3Tip;

export default {
	name: 'DearTechPeopleChart',
	
		data: function() {	
			return {
				keys:[],
				limit:15,
				_mydata:[],
				limit_options: [10,15,20,25,50,80,"all"],
				sortby_options: [ "Ranking", "Leadership ranking", "Technical ranking", "_________________",
					"Total employees", "_________________", 
					"Most female (ratio)", "Most Black (ratio)",
					"Most Asian (ratio)", "Most Latinx (ratio)",],
				sortby: "Ranking",
				filterby_options: ["all", "leadership", "technical"],
				filterby: "all",	
				industry_options: ["all",],
				industry: "all",
				customer_options: ["all",],
				customer: "all",
				location_options: ["all",], //"SF based"],
				location: "all",	
				limits:25,
			}
		},

		mounted() {
			this.$nextTick(function () {
				this.load_data()
			})
		},
		
		
		created() {
			this.$on('data_loaded', function () {
				this.init_chart();
				this.wrangle_data();
				this.update_chart();
			});
		},
			
		methods: {
			updateSelection() {
				this.wrangle_data();
				this.update_chart();
			},		

			init_chart: function() {
				var self = this;

				self.margin = {top: 70, right: 20, bottom: 30, left: 30};				
				self.svg = d3.select("svg");
				self.colorS = {
					"white":"#eeeeee",
					"black":"#333333",
					"asian":"#fee8c8",
					"latinx": "#fdbb84",
					"other": "#eded84",
				}
				
				self.width = +self.svg.attr("width") - self.margin.left - self.margin.right;
				self.height = +self.svg.attr("height") - self.margin.top - self.margin.bottom;
				//self.limit = self.height / 39;
				
				self.g = self.svg.append("g").attr("transform", "translate(" + self.margin.left + "," + self.margin.top + ")");
				self.bars = self.g.append("g").attr("class","bars");
				self.defs = self.svg.append("defs");
				
				self.filter = self.defs.append("filter")
				.attr("id", "drop-shadow")
				.attr("height", "130%")
				.attr("width", "130%");

				self.filter.append("feGaussianBlur")
					.attr("in", "SourceAlpha")
					.attr("stdDeviation", 1)
					.attr("result", "blur");

				self.filter.append("feOffset")
					.attr("in", "blur")
					.attr("dx", 1)
					.attr("dy", 1)
					.attr("result", "offsetBlur");

				var feMerge = self.filter.append("feMerge");

				feMerge.append("feMergeNode")
					.attr("in", "offsetBlur")
				feMerge.append("feMergeNode")
					.attr("in", "SourceGraphic");		
				
				self.t = d3.transition().duration(50);
				self.x = d3.scaleLinear().range([0, self.width/3]);
				self.y = d3.scaleBand().range([0, self.height]).padding(0.5);
				var color = d3.scaleOrdinal().range([
					self.colorS["black"],
					self.colorS["asian"],
					self.colorS["latinx"],
					self.colorS["other"],
					self.colorS["white"],
					]);	
										
				self.g.append("text")
					.attr("class", "top")
					.attr("transform", function(d,i) { return "translate(" + (2*self.width/3 - 20)  + ", -10)" })
					.text("\uf222");	
						
				self.g.append("text")
					.attr("class", "top")
					.attr("transform", function(d,i) { return "translate(" + (2*self.width/3 + 8)  + ", -10)" })
					.text("\uf221");
					
				self.g.append("text")
					.attr("class", "top")
					.attr("transform", function(d,i) { return "translate(-10,-10)" })
					.text("RANK");	
					
				self.middle_seperator = self.g.append("line")
					.attr("x1", 2*self.width/3 )
					.attr("y1", -20 )
					.attr("y2", self.height + 10 )
					.attr("x2", 2*self.width/3 )
					.attr("class","line")					
					
				self.g.append("line")
					.attr("class","line")
					.attr("x1", -10 )
					.attr("y1", -5 )
					.attr("y2", -5 )
					.attr("x2", self.width+10)

				self.bottom_line = self.g.append("line")
					.attr("class","line")
					.attr("x1", -10 )
					.attr("y1", self.height + 5)
					.attr("y2", self.height + 5)
					.attr("x2", self.width + 10)


				self.customXAxis = function(g) {
				  g.call(self.xAxis);
				  g.select(".domain").remove();
				  //g.selectAll(".tick:not(:first-of-type) line").attr("stroke", "#666").attr("stroke-dasharray", "2,2");
				 // g.selectAll(".tick text").attr("x", 4).attr("dy", -4);
				  g.selectAll(".tick text").attr("fill", "#999999");
				}
									
				self.xAxis = d3.axisTop(self.x)			
					.ticks(3, ",f")
					.tickSize(2)
					
				self.xAxisGroup = self.g.append("g")
					.attr("class","xAxis")
					.attr("transform","translate(" +  2*self.width/3 + ", -5)");
					
				self.xAxisGroup.call(self.customXAxis);
	
	
				var diversities = ["black","asian", "latinx", "other", "white",];
				self.color = d3.scaleOrdinal().range(["#333333", '#fee8c8','#fdbb84','#eded84',"#eeeeee", ]);
						
				self.legend = self.g.append("g")
					.attr("transform", "translate("+ (2*self.width/3 + 20 )  + ",-45)" );
				
				diversities.forEach(function(diversities, i){
					var legendRow = self.legend.append("g")
						.attr("transform", "translate("+ (i * 60) + ",0)");

					legendRow.append("rect")
						.attr("width", 12)
						.attr("height", 12)
						.attr("fill", self.color(diversities));

					legendRow.append("text")
						.attr("x", 15)
						.attr("y", 10)
						.attr("text-anchor", "start")
						.style("text-transform", "capitalize")
						.text(diversities);
				});		
				
				
				// Tooltip
				self.tip = d3.tip().attr('class', 'd3-tip')
					.html(function(d) {
						var text = "<div>"
						text += "<strong>" + d.data.company_name + "</strong> | " + d.data.hq_location + "<br>";
						text += "<span class='subinf'>" + d.data.sector_1
						if (d.data.sector_2.length > 1) 
							text += ", " + d.data.sector_2;
						text += " | " + d.data.customer_base_1 + "</span>";
						if (d.data.customer_base_2.length > 1) 
							text += ", " + d.data.customer_base_2;
						text += "</span><br/>";
						text += "<table>";				
						text += "<tr>";
						text += "<td><strong>" + d.data.total + "</strong> Employees</td>";
						text += "<td></td>";
						text += "</tr>";
						
						if (this.className.baseVal == "female") {
							text += "<tr class='sep'>"
							text += "<td><span style='color:#92278f'>" 
								+ d.data.total_female + "</span> Females </td>" 
							text += "<td>(<span style='color:#92278f'>" 
								+ d3.format(".0f")(d.data.ratio_female * 100) 
								+ "%</span>) </td>";
							text += "</tr><tr>"
							text += "<td><span style='color:#92278f'>" 
								+ d.data.total_leadership_female 
								+ "</span> Females in leader positions</td>" 
							text += "<td>(<span style='color:#92278f'>" 
								+ d3.format(".0f")(d.data.ratio_leadership_female * 100) 
								+ "%</span> )</td>";
							text += "</tr><tr>"
							text += "<td><span style='color:#92278f'>" 
								+ d.data.total_technical_female 
								+ "</span> Females in tech</td>" 
							text += "<td>(<span style='color:#92278f'>" 
								+ d3.format(".0f")(d.data.ratio_technical_female * 100) 
								+ "%</span> )</td>";				
							text += "</tr><tr  class='sep'>"
							text += "<td><span style='color:#92278f'>" 
								+ d.data.overall_female_black 
								+ "</span> Black females </td>" 
							text += "<td>(<span style='color:#92278f'>" 
								+ d3.format(".0f")(d.data.overall_female_black / d.data.total_female * 100) 
								+ "%</span>)</td>";
							text += "</tr><tr>"		
							text += "<td><span style='color:#92278f'>" 
								+ d.data.overall_female_asian 
								+ "</span> Asian females </td>" 
							text += "<td>(<span style='color:#92278f'>" 
								+ d3.format(".0f")(d.data.overall_female_asian/ d.data.total_female * 100) 
								+ "%</span> )</td>";
							text += "</tr><tr>"		
								text += "<td><span style='color:#92278f'>" 
								+ d.data.overall_female_latinx 
								+ "</span> Latinx females </td>" 
							text += "<td>(<span style='color:#92278f'>" 
								+ d3.format(".0f")(d.data.overall_female_latinx / d.data.total_female * 100) 
								+ "%</span> )</td>";	
							text += "</tr>";
						} else {							
							text += "<tr class='sep'>"
							text += "<td>" + d.data.total_male + " Males </td>" 
							text += "<td>(" + d3.format(".0f")(d.data.total_male / d.data.total * 100) 
								+ "%) </td>";
							text += "</tr><tr>"
							text += "<td>" + d.data.total_leadership_male + " Males in leader positions</td>" 
							text += "<td>(" + d3.format(".0f")( (1-d.data.ratio_leadership_female) * 100) 
								+ "%)</td>";
							text += "</tr><tr>"
							text += "<td>" + d.data.total_technical_male + " Males in tech</td>" 
							text += "<td>(" + d3.format(".0f")( (1-d.data.ratio_technical_female) * 100) 
								+ "%)</td>";				
							text += "</tr><tr  class='sep'>"
							text += "<td>" + d.data.overall_male_black + " Black males </td>" 
							text += "<td>(" + d3.format(".0f")(d.data.overall_male_black / d.data.total_male * 100) 
								+ "%)</td>";
							text += "</tr><tr>"		
							text += "<td>" + d.data.overall_male_asian + " Asian males </td>" 
							text += "<td>(" + d3.format(".0f")(d.data.overall_male_asian/ d.data.total_male * 100) 
								+ "%)</td>";
							text += "</tr><tr>"		
								text += "<td>" + d.data.overall_male_latinx + " Latinx males </td>" 
							text += "<td>(" + d3.format(".0f")(d.data.overall_male_latinx / d.data.total_male * 100) 
								+ "%)</td>";	
							text += "</tr>";
						}			
						text += "</table>";
						text += "</div>";
						return text;
					});
					
				self.tips = self.g.append("g")
				self.tips.call(self.tip);

			},
			
			load_data() {
				var self = this;
				var limit = self.limit;
	
				d3.csv("data/techpeople.csv").then(function(_data, i, columns) {
				
					_data = _data.map(function(d, i, columns) { 
						for(var key in d){
							if (["company_name", "date_pulled", 
								"customer_base_1","customer_base_2",
								"hq_location", "sf_based",
								"sector_1", "sector_2",
								"total_employees"].indexOf(key) == -1)
								d[key] = +d[key]; 
						}

						d["total_leadership_male"] = 
							+d.leadership_male_black +
							+d.leadership_male_asian +
							+d.leadership_male_latinx +
							+d.leadership_male_white;
							
						d["total_leadership_female"] = 
							+d.leadership_female_black  +
							+d.leadership_female_latinx  +
							+d.leadership_female_asian +
							+d.leadership_female_white;
						
						d["total_leadership"] = 
							+d.total_leadership_male +
							+d.total_leadership_female;
							
						d["total_technical_male"] = 
							  +d.technical_male_black 
							+ +d.technical_male_asian 
							+ +d.technical_male_latinx
							+ +d.technical_male_white;
							
						d["total_technical_female"] =
							+ +d.technical_female_black 
							+ +d.technical_female_asian 
							+ +d.technical_female_latinx
							+ +d.technical_female_white;	
						
						d["total_technical"] = 
							  d.total_technical_male
							+ d.total_technical_female
													
						
						d.total = d.total_female + d.total_male;
						d["ratio_female"] = d.total_female / d.total;	
						d["ratio_technical_female"] = d.total_technical_female / d.total_technical;	
						d["ratio_leadership_female"] = d.total_leadership_female / d.total_leadership;
						
						d["total_black"] = d.overall_male_black + d.overall_female_black;
						d["total_asian"] = d.overall_male_asian + d.overall_female_asian;
						d["total_latinx"] = d.overall_male_latinx + d.overall_female_latinx;
						
						d["ratio_black"] = d.total_black / d.total;
						d["ratio_asian"] = d.total_asian / d.total;
						d["ratio_latinx"] = d.total_latinx / d.total;
						
						
						
						if( self.industry_options.indexOf(d.sector_1) == -1 && d.sector_1.length > 1)
							self.industry_options.push(d.sector_1);
							
						if( self.customer_options.indexOf(d.customer_base_1) == -1 && d.sector_1.length > 1)
							self.customer_options.push(d.customer_base_1);

						if( self.location_options.indexOf(d.hq_location) == -1 && d.hq_location.length > 1)
							self.location_options.push(d.hq_location);
															
						return d;
					})
					
					self._mydata = _data;
					//self.y.domain(self.display_data.map(function(d){ return d.company_name; }));
					self.$emit('data_loaded',true)
					
				});
			},
			
			wrangle_data() {
				var self = this;
				var limit = self.limit;

							
				switch (self.sortby) {				   
					case "Most female (ratio)": 
						if (self.filterby == "technical") {
							self.display_data = self._mydata.sort(function(a,b) {
								return d3.descending(a.ratio_technical_female, b.ratio_technical_female); 
							})
						} else if (self.filterby == "leadership") {
							self.display_data = self._mydata.sort(function(a,b) {
								return d3.descending(a.ratio_leadership_female, b.ratio_leadership_female); 
							})
						} else {
							self.display_data = self._mydata.sort(function(a,b) {
								return d3.descending(a.ratio_female, b.ratio_female); 
							})
						}
						break;
						
					case "Most Black (ratio)": 
						if (self.filterby == "technical") {
							self.display_data = self._mydata.sort(function(a,b) {
								return d3.descending((a.technical_male_black + a.technical_female_black)/ a.total_technical,  
								(b.technical_male_black + b.technical_female_black) / b.total_technical, );
							})
						} else if (self.filterby == "leadership") {
							self.display_data = self._mydata.sort(function(a,b) {
								return d3.descending((a.leadership_male_black + a.leadership_female_black)/ a.leadership_total,  
								(b.leadership_male_black + b.leadership_female_black) / b.leadership_total, ); 
							})
						} else {
							self.display_data = self._mydata.sort(function(a,b) {
								return d3.descending(a.ratio_black, b.ratio_black); 						
							})
						}
						break;

					case "Most Asian (ratio)": 
						if (self.filterby == "technical") {
							self.display_data = self._mydata.sort(function(a,b) {
								return d3.descending((a.technical_male_asian + a.technical_female_asian)/ a.total_technical,  
								(b.technical_male_asian + b.technical_female_asian) / b.total_technical, );
							})
						} else if (self.filterby == "leadership") {
							self.display_data = self._mydata.sort(function(a,b) {
								return d3.descending((a.leadership_male_asian + a.leadership_female_asian)/ a.leadership_total,  
								(b.leadership_male_asian + b.leadership_female_asian) / b.leadership_total, ); 
							})
						} else {
							self.display_data = self._mydata.sort(function(a,b) {
								return d3.descending(a.ratio_asian, b.ratio_asian); 						
							})
						}
						break;
						
					case "Most Latinx (ratio)": 
						if (self.filterby == "technical") {
							self.display_data = self._mydata.sort(function(a,b) {
								return d3.descending((a.technical_male_latinx + a.technical_female_latinx)/ a.total_technical,  
								(b.technical_male_latinx + b.technical_female_latinx) / b.total_technical, );
							})
						} else if (self.filterby == "leadership") {
							self.display_data = self._mydata.sort(function(a,b) {
								return d3.descending((a.leadership_male_latinx + a.leadership_female_latinx)/ a.leadership_total,  
								(b.leadership_male_latinx + b.leadership_female_latinx) / b.leadership_total, ); 
							})
						} else {
							self.display_data = self._mydata.sort(function(a,b) {
								return d3.descending(a.ratio_latinx, b.ratio_latinx); 						
							})
						}
						break;			
						
					case "Ranking": 
						self.display_data = self._mydata.sort(function(a,b) {
							return d3.ascending(a.overall_ranking, b.overall_ranking); 
						})
						break;		
						
					case "Technical ranking": 
						self.display_data = self._mydata.sort(function(a,b) {
							return d3.ascending(a.technical_ranking, b.technical_ranking); 
						})
						break;

					case "Leadership ranking": 
						self.display_data = self._mydata.sort(function(a,b) {
							return d3.ascending(a.leadership_ranking, b.leadership_ranking); 
						})
						break;		
						
					case "Total employees": 
						self.display_data = self._mydata.sort(function(a,b) {
							return d3.descending(a.total, b.total); 
						})
						break;		
						
																							   
					default: 
						self.display_data = self._mydata.sort(function(a,b) {
							return d3.ascending(a.overall_ranking, b.overall_ranking); 
						})
				}
				
				if (self.industry != "all") {
					self.display_data = self.display_data.filter( function(d) {
						if (d.sector_1 == self.industry)
							return d;
					})	
				}
				
				if (self.customer != "all") {
					self.display_data = self.display_data.filter( function(d) {
						if (d.customer_base_1 == self.customer)
							return d;
					})	
				}

				/*if (self.location == "SF based") {
					self.display_data = self.display_data.filter( function(d) {
						if (d.sf_based === "TRUE")
							return d;
					})	
				} else */
				
				if (self.location != "all") {
					self.display_data = self.display_data.filter( function(d) {
						if (d.hq_location == self.location)
							return d;
					})	
				}
				
				if (self.limit != "all")
					self.display_data = self.display_data.slice(0, self.limit);
			},
			
			update_chart() {
				
				var self = this;
				
				var bw = 30; //self.y.bandwidth();
				
				self.svg.attr("height", ((bw + 10)* self.display_data.length + self.margin.top + self.margin.bottom));
				
				if (self.filterby == "technical") {
					var keys_left = ["technical_male_black", "technical_male_asian", "technicall_male_latinx", "technical_male_white",  ]
					var keys_right = ["technical_female_black", "technical_female_asian","technical_female_latinx","technical_female_white", ]
					self.x.domain([0, d3.max(self.display_data, function(d) { return Math.max(d.total_technical_male, d.total_technical_female) })]);
					var color = d3.scaleOrdinal().range([self.colorS["black"], self.colorS["asian"], self.colorS["latinx"], self.colorS["white"],]);	
				} else if (self.filterby == "leadership") {
					var keys_left = ["leadership_male_black", "leadership_male_asian", "leadership_male_latinx", "leadership_male_white",  ]
					var keys_right = ["leadership_female_black", "leadership_female_asian","leadership_female_latinx","leadership_female_white", ]
					var color = d3.scaleOrdinal().range([self.colorS["black"], self.colorS["asian"], self.colorS["latinx"], self.colorS["white"],]);	
					self.x.domain([0, d3.max(self.display_data, function(d) { return Math.max(d.total_leadership_male, d.total_leadership_female) })]);
				} else {
					var keys_left = ["overall_male_black", "overall_male_asian", "overall_male_latinx", "overall_male_other", "overall_male_white",  ]
					var keys_right = ["overall_female_black", "overall_female_asian", "overall_female_latinx", "overall_female_other","overall_female_white", ];
					self.x.domain([0, d3.max(self.display_data, function(d) { return d.total; })]);
					var color = d3.scaleOrdinal().range(
						[self.colorS["black"], self.colorS["asian"], self.colorS["latinx"], self.colorS["other"], self.colorS["white"],]
					);	
					self.x.domain([0, d3.max(self.display_data, function(d) { return Math.max(d.total_male, d.total_female) })]);
				}
				
				self.xAxisGroup.call(self.customXAxis);
				self.g.selectAll(".tick").each(function (d) { if ( d === 0 ) { this.remove(); }});


				self.y.domain(self.display_data.map(function(d){ return d.company_name; }));
				color.domain(keys_left);
								
				// left bars - male

				var stack = d3.stack().keys(keys_left);
				self.bars.selectAll(".bars_left").remove();

				
				var bars = self.bars
					.selectAll(".bars_left")
					.data(stack(self.display_data), function(d){
						return  "l" + d.company_name ;
					});
						
				bars.enter()
					.append("g")
					.attr("class","bars_left")	
					.attr("fill", function(d) { return color(d.key); })
					.selectAll("rect")
					.data(function(d) { return d; })
					.enter()
						.append("rect")
						.attr("y", function(d,i ) { return (bw+10) * i + 5}) //self.y(d.data.company_name) })
						.attr("x", function(d) {  return  2*self.width/3 - self.x(d[1]); })
						.attr("width", function(d) { return self.x(d[1]) -self. x(d[0]); })
						.attr("height", bw/1.5 )
						.on("mouseover", self.tip.show)
						.on("mouseout", self.tip.hide);	
				
								
				stack = d3.stack().keys(keys_right);


				// right bars - female
				self.bars.selectAll(".bars_right").remove();

				bars = self.bars
					.selectAll(".bars_right")
					.data(stack(self.display_data), function(d){
						return "r" + d.company_name ;
					})
		
									
				bars.enter()
					.append("g")
					.attr("class","bars_right")
										
					.attr("fill", function(d) { return color(d.key); })
					.selectAll("rect")
					.data(function(d) { return d; })
					.enter()
						.append("rect")
						.attr("class","female")
						.attr("y", function(d,i ) { return (bw+10) * i + 5 }) //self.y(d.data.company_name) })
						.attr("x", function(d) {  return  2*self.width/3 + self.x(d[0]) + 0.5 })
						.attr("width", function(d) { return self.x(d[1]) - self.x(d[0]); })	
						.attr("height", bw/1.5) //self.y.bandwidth );
						.on("mouseover", self.tip.show)
						.on("mouseout", self.tip.hide);							
					
										
				// pie ratio male/female
				
				var arc = d3.arc().innerRadius(5).outerRadius(bw/2 + 2 );
				var pie = d3.pie().sort(null);

				var color_pie = d3.scaleOrdinal().range(["#ffffff","#92278f",  ]);
				if (self.sortby == "Most Black (ratio)") {
					color_pie = d3.scaleOrdinal().range(["#ffffff", self.colorS["black"], ]);
				} else if (self.sortby == "Most Asian (ratio)") {
					color_pie = d3.scaleOrdinal().range(["#ffffff",self.colorS["asian"],   ]);
				} else  if (self.sortby == "Most Latinx (ratio)") {
					color_pie = d3.scaleOrdinal().range(["#ffffff", self.colorS["latinx"],   ]);
				}
				
				self.g.selectAll(".pies").remove()				
				self.g.selectAll('arc')
					.data(self.display_data)
					.enter()
					.append('g')
					.attr("class","pies")
					.attr("transform", function(d, i) {
						//return "translate(220,"+  (self.y(d.company_name)+ self.y.bandwidth()/2 ) +")";
						return "translate(220,"+  ((bw + 10)* i + bw/2 + 2) +")";
					})
					.selectAll('.pieChart')
					.data( function(d) {
						if (self.sortby == "Most Black (ratio)") {
							if (self.filterby == "technical") {
								return pie([d.total_technical, d.technical_male_black + d.technical_female_black]);
							} else if (self.filterby == "leadership") {
								return pie([d.total_leadership, d.leadership_male_black + d.leadership_female_black]);
							} else {
								return pie([d.total, d.total_black])
							}
						} else if (self.sortby == "Most Asian (ratio)") {
							if (self.filterby == "technical") {
								return pie([d.total_technical, d.technical_male_asian + d.technical_female_asian]);
							} else if (self.filterby == "leadership") {
								return pie([d.total_leadership, d.leadership_male_asian + d.leadership_female_asian]);
							} else {
								return pie([d.total, d.total_asian])
							}
						} else if (self.sortby == "Most Latinx (ratio)") {
							if (self.filterby == "technical") {
								return pie([d.total_technical, d.technical_male_latinx + d.technical_female_latinx]);
							} else if (self.filterby == "leadership") {
								return pie([d.total_leadership, d.leadership_male_latinx + d.leadership_female_latinx]);
							} else {
								return pie([d.total, d.total_latinx])
							}
						} else if (self.filterby == "technical") {
							return pie([d.total_technical_male, d.total_technical_female]);
						} else if (self.filterby == "leadership") {
							return pie([d.total_leadership_male, d.total_leadership_female]);
						} else {
							return pie([d.total_male,d.total_female]);
						}						
					})
					.enter()
						.append("path")
						.attr("d",  arc)
						.attr("fill", function(d,i) { return color_pie(i); })
						.attr("stroke", "#333" )
						.attr("stroke-width", 0.5 )
						//.style("filter", "url(#drop-shadow)")
						


				// notes 
				self.g.selectAll("text.ratio_note").remove()
				self.g
					.selectAll("text.ratio_note")
					.data(self.display_data)
					.enter()
					.append("text")
					.attr("class", "ratio_note")
					.attr("transform", function(d, i) { return "translate(242," + ((bw + 10)* i + bw/2 + 5) + ")" })
					.text(function(d) { 
						if (self.sortby == "Most Black (ratio)") {
							if (self.filterby == "technical") {
								return d3.format(".0f")( ((d.technical_male_black + d.technical_female_black) / d.total_technical) * 100) + "% Black" 
							} else if (self.filterby == "leadership") {
								return d3.format(".0f")( ((d.leadership_male_black + d.leadership_female_black) / d.total_leadership) * 100) + "% Black" 
							} else {
								return d3.format(".0f")(d.ratio_black * 100) + "% Black" 
							}
						} else if (self.sortby == "Most Asian (ratio)") {
							if (self.filterby == "technical") {
								return d3.format(".0f")( ((d.technical_male_asian + d.technical_female_asian) / d.total_technical) * 100) + "% asian" 
							} else if (self.filterby == "leadership") {
								return d3.format(".0f")( ((d.leadership_male_asian + d.leadership_female_asian) / d.total_leadership) * 100) + "% asian" 
							} else {
								return d3.format(".0f")(d.ratio_asian * 100) + "% Asian" 
							}
						} else if (self.sortby == "Most Latinx (ratio)") {
							if (self.filterby == "technical") {
								return d3.format(".0f")( ((d.technical_male_latinx + d.technical_female_latinx) / d.total_technical) * 100) + "% latinx" 
							} else if (self.filterby == "leadership") {
								return d3.format(".0f")( ((d.leadership_male_latinx + d.leadership_female_latinx) / d.total_leadership) * 100) + "% latinx" 
							} else {
								return d3.format(".0f")(d.ratio_latinx * 100) + "% latinx" 
							}
						} else if (self.filterby == "technical") {
							return d3.format(".0f")(d.ratio_technical_female * 100) + '% \uf221'
						} else if (self.filterby == "leadership") {
							return d3.format(".0f")(d.ratio_leadership_female * 100) + '% \uf221'
						} 
							return d3.format(".0f")(d.ratio_female * 100) + '% \uf221'
					} );
				

				// rankinks
				
				self.g.selectAll("text.ranking").remove()
				self.g
					.selectAll("text.ranking")
					.data(self.display_data)
					.enter()
					.append("text")
					.attr("class", "ranking")
					//.attr("transform", function(d) { return "translate(0," + (self.y(d.company_name) + self.y.bandwidth() * 0.7) + ")" })
					.attr("transform", function(d, i) { return "translate(0," + ((bw + 10)* i + bw/2 + 8) + ")" })
					.text(function(d) { return d.overall_ranking + "."} );
				
				
				
				// labels
				
				self.g.selectAll("text.label").remove()
				self.g
					.selectAll("text.label")
					.data(self.display_data)
					.enter()
					.append("text")
					.attr("class", "label")
					.attr("transform", function(d,i) { return "translate(30," + ((bw + 10)* i + bw/2+ 8) + ")" })
					//.attr("transform", function(d) { return "translate(30," + (self.y(d.company_name) + self.y.bandwidth() * 0.7) + ")" })
					.text(function(d) { return d.company_name} );		

				self.bottom_line
					.attr("y1", ((bw + 10)* self.display_data.length))
					.attr("y2", ((bw + 10)* self.display_data.length))
					
				self.middle_seperator
					.attr("y2",  ((bw + 10)* self.display_data.length) )
			}
		}
	}
	  
</script>


<style>

</style>
