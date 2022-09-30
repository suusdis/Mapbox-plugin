
<!-- Html text -->
<template>
<div class="map-container">
    <div v-if="filterEnabled" class="filter-bar">
      <div class="filter">
          <button class="remove" @click="reset()">
            <div class="icon-container">
              <refresh-icon color="grey-dark">
              </refresh-icon>
            </div>
          </button>
        </div>
      <div class="filter" :class="{ open: filter.open }" v-for="filter in filters" :key="filter.name">
        <div class="filter-type" v-if="filter.type === 'Multi-Select' || filter.type === 'Single-Select'">
          <button :class="{ open: filter.open }" @click="openFilter(filter.key)"> 
            <div class="filter-type-indicator" :class="{active: filter.active}"></div>
            <div class="filter-type-name">{{filter.name}}</div> 
            <div class="icon-container">
              <arrow-icon color="grey-dark">
                </arrow-icon>
            </div>
          </button>
            <transition 
                enter-active-class="fadeInUp"
                leave-active-class="fadeOutDown">
                <div class="checklist" v-if="filter.open">
                  <div class="filter-search-container">
                      <div class="search-icon-container">
                          <search-icon color="grey" class="search-icon"></search-icon>
                      </div>
                      <input type="text" class="search" placeholder="Search" v-model="filter.search" autocomplete="off" ref="typeSearch" />
                  </div>
                  <div class="scroll-container" :options="scrollOptions" key="checklist-type">
                      <label class="input-check"  v-for="option in filteredOptions(filter.options, filter.search)" :key="option.option" @click="singleSelect(filter, option); filterClicked(filter.key)">
                        <span v-if="filter.showCount">{{option.option}} ({{option.count}}) </span>
                        <span v-else>{{option.option}}</span>
                        <span class="checkmark" :class="{single: filter.type === 'Single-Select', enabled: option.enabled}"></span>
                      </label>
                  </div>
                  <div class="selection-buttons">
                      <button @click="select(filter.name , true);filterClicked(filter.key)" class="selection-button">Select All</button> 
                      / 
                      <button @click="select(filter.name, false);filterClicked(filter.key)" class="selection-button">Deselect All</button>
                  </div>
                </div>
            </transition>
          </div>
          <div class="filter-type" v-if="filter.type === 'Agenda'">
            <button :class="{ open: filter.open }" @click="openFilter(filter.key)">
              <div class="filter-type-indicator" :class="{active: filter.active}"></div>
              <div class="filter-type-name">{{filter.name}}</div> 
              <div class="icon-container"><arrow-icon color="grey-dark"></arrow-icon></div>
            </button>
            <transition enter-active-class="fadeInUp" leave-active-class="fadeOutDown">
              <div v-if="filter.open" class="date-filter">
                <client-only>
                <v-date-picker
                  value="range"
                  v-model="filter.filterRange"
                  is-range
                  :available-dates='{start: new Date(filter.dateRange.start.year, filter.dateRange.start.month, filter.dateRange.start.day),
                  end: new Date(filter.dateRange.end.year, filter.dateRange.end.month, filter.dateRange.end.day)}'
                  :model-config="modelConfig"
                  :mode="filter.time ? 'dateTime' : 'date'"/>
                </client-only>
              </div>        
            </transition>
          </div>
          <div class="filter-type" v-if="filter.type === 'Range Histogram'">
            <button :class="{ open: filter.open }" @click="openFilter(filter.key)">
            <div class="filter-type-indicator" :class="{active: filter.active}"></div>
            <div class="filter-type-name">{{filter.name}}</div> 
              <div class="icon-container"><arrow-icon color="grey-dark"></arrow-icon></div>
            </button>
              <transition enter-active-class="fadeInUp" leave-active-class="fadeOutDown">
                <div v-if="filter.open" class="dropdown">
                    <client-only>
                    <HistogramSlider
                        :width="275"
                        :bar-height="75"
                        ref="hist"
                        :data="filter.histogramOptions"
                        :colors="[pSBC(0.1, filter.color), pSBC(-0.1, filter.color)]"
                        :primaryColor="filter.color"
                        :labelColor="filter.color"
                        :hideFromTo="true"
                        :forceEdges="true"
                        :barGap="5"
                        :handleSize="22"
                        @finish="histogramRange($event, filter.key); filterClicked(filter.key)"
                        fontFamily="Galyon Regular"
                        :histSliderGap="1"
                        :step="1"
                        v-cloak
                    />
                    <div class="range-number-input">
                      <div class="input-container">
                        <div class="title">Min:</div>
                        <input class="number-input" type="number"  v-model="filter.histogramChart.startText" @change="rangeChange(filter.key);filterClicked(filter.key)"/>
                      </div>
                      <div class="input-container">
                        <div class="title">Max:</div>
                        <input class="number-input" type="number"  v-model="filter.histogramChart.endText" @change="rangeChange(filter.key);filterClicked(filter.key)"/>
                      </div>
                    </div>
                    </client-only>
                </div>
            </transition>
          </div>
        </div>
        
      </div>

    <client-only>
      <MglMap ref="map" class="map" id='map'
	  	container="map" 
        v-if="componentLoaded"
        :access-token="accessToken"
        :map-style="mapStyle"
        :center="center"
        :zoom="zoom"
      >
      <MglMarker v-for="coordinate in pinpoints" :key="coordinate.id"
          :coordinates="[coordinate.longitude, coordinate.latitude]">
          <i slot="marker">
            <div v-if="coordinate.source === 'Color'">
              <span class="dot"
              :style=" {'background-color': coordinate.color + ' !important'}"></span>
            </div>
            <div v-else-if="coordinate.source === 'Picture'">
                    <img v-if="coordinate.picture.enableUpload" :src="'data:image/jpeg;base64,' + coordinate.pictureData" style="height:30px; width:30px;">
                    <img v-else :src="coordinate.pictureData" style="height:30px; width:30px;">
            </div>
            <div v-else-if="coordinate.source === 'SVG'">
                    <img v-if="coordinate.picture.enableUpload" :src="'data:image/svg+xml;base64,' + coordinate.pictureData" style="height:30px; width:30px;">
                    <img v-else :src="coordinate.pictureData" style="height:30px; width:30px;">
            </div>
            <div  v-else>
                <div class="icon-container">
                    <component :is="coordinate.icon" color="black"></component>
                </div>
                
            </div>
          </i>
          <MglPopup :closeButton="false" className='map' v-if="coordinate.popup">  
            <div class="popup-container">
                <div class="popup" v-for="details in coordinate.details" :key="details.id">
                    <div class="popup-title">{{ details.name }}</div>
                    <div class="popup-content">{{ details.value }}</div>     
                </div>
                <div class="actions" v-if="coordinate.enableActions">
                      <div class="actions-title">Actions</div>
                      <div class="actions-content">
                          <div v-for="button in coordinate.actionButtons" :key="button.buttonKey">
                              <div v-for="action in button.actions" :key="action.actionKey">
                                  <button
                                      :class="'action_button'"
                                      @click="actionFunction(button.actions, coordinate.details)"
                                      :style=" { 'background-color': button.color + ' !important'}">
                                          <div class="icon-container">
                                              <component :is="button.icon" color="white-light"></component>
                                          </div>
                                  </button>
                              </div>
                          </div>
                      </div>
                </div>        
            </div>
          </MglPopup>
        </MglMarker>
        <MglGeolocateControl ref="geolocateControl" />
        <MglNavigationControl position="top-right"/>
      </MglMap>
    </client-only>
    <button @click="filterMap()"> Testen </button>
  </div>
  
</template>

<script>
import API from '~/plugins/API';
import Color from '~/mixins/components/charts/colorMixin';
import { mapGetters } from "vuex";
export default { 
  	mixins: [Color],
	props:{
		data: {
			type: Array,
			required: true,
			default: function(){
				return [];
			}
		},
		selection: {
			type: Object,
			required: true,
			default: function(){
				return {};
			}
		},
		title: {
			type: String,
			required: true,
			default: ""
		},
		transformations:{
			type: Array,
			required: true,
			default: function(){
				return [];
			}
		},
		widgetId: {
			type: Number,
			required: true
		},
		itemId: {
			required: false,
			default: null
		},
        currentApp: {
            type : String,
            required: false
        }
	},  
  	data() {
		return {
		accessToken: "pk.eyJ1IjoiY2xhcHBmb3JtIiwiYSI6ImNrdndhMG9uNzA2Y3UzMHI1Y2tjY283bW8ifQ.Q5DHDK0fucyYPvSpDkAEkw",// your access token. Needed if you using Mapbox maps
		mapStyle: '', // your map style // your map style
		img: '',
		center: [ 4.900143252355628, 52.379678412705594],
		// location: [5.0564295882, 52.498345],
		zoom: 13,
		pinpoints: [],
		filters: [],
		filterCopy: [],
		icons:{},
		scrollOptions: {
			wheelSpeed: 0.3
		},
		filtersActive: [],
		componentLoaded: false,
		provincie: ['Noord-Holland','Zuid-Holland'],
		};
  	},
   	methods:{
		filterMap: function(){
			this.$refs.map.map.style.setFilter('postcode_vier', ['all',['match', ['get', 'prov_name'], this.provincie, true, false]]);
		},
		median: function(array= []) {
			array.sort((a, b) => a - b);
			if (array.length % 2) {
				//length of array is odd
				return array[(array.length + 1) / 2 - 1];
			} else {
				//length of array is even
				return 0.5 * [(array[array.length / 2 - 1] + array[array.length / 2])];
			} 
		},
		modus: function(arr ){
			let outArr = []; //output variable (an array containing arrays)
			for (let i = 0; i < arr.length; i++) { 
				let pushed = false; //keep track
				for (let j = 0; j < outArr.length; j++) {
					if (!outArr[j].includes(arr[i])) { //for arrays include function checks for given elements
					outArr[j].push(arr[i]); //push if array doesn't have the variable
					pushed = true;
					break;
					}
				}
				if (!pushed) {
					outArr.push([arr[i]]); //push a brand new array of array
				}
			}
			return outArr.pop()[0]; //return the last one (therefore most frequent) //for this case pop() is similar to outArr[outArr.length-1]
		},
		openFilter(filterKey){
		let filters = JSON.parse(JSON.stringify(this.filters));
		for(let i =0; i< filters.length; i++){
			if(filters[i].key === filterKey){
				filters[i].open = !filters[i].open;
				if(filters[i].open && filters[i].type === 'Range Histogram'){
					let app = this;
					window.setTimeout(function(){
					app.$refs[filterKey][0].update({ from: filters[i].histogramChart.start, to: filters[i].histogramChart.end })
					}, 150)
					
				}
			}
		}
		this.$set(this, 'filters', filters);
		},
		rangeChange: function(filterKey){
		
			let filters = this.filters;
			let filter = filters.filter(function (item){
				return item.key === filterKey
			});
			filter = filter[0];
			if(typeof filter.histogramChart.startText === 'string'){
				let number =  Number(filter.histogramChart.startText);
				if(number < filter.histogramChart.end && number >= filter.histogramOptions[0]){
				filter.histogramChart.start = number;
				this.$refs.hist[0].ionRangeSlider.update({from: number, to: filter.histogramChart.end});
				this.$refs.hist[0].updateBarColor({ from: number, to: filter.histogramChart.end });
				}else{
				filter.histogramChart.startText = filter.histogramChart.start
				}
			}
			if(typeof filter.histogramChart.endText === 'string'){
				let number = Number(filter.histogramChart.endText);
				if(number > filter.histogramChart.start && number <= filter.histogramOptions[filter.histogramOptions.length - 1] ){
				filter.histogramChart.end = number
				this.$refs.hist[0].ionRangeSlider.update({from: filter.histogramChart.start , to: number});
				this.$refs.hist[0].updateBarColor({ from: filter.histogramChart.start , to: number });
				}else{
				filter.histogramChart.endText = filter.histogramChart.end
				}
			}
			for(let i=0; i < filters.length; i++){
				if(filters[i].key === filterKey){
				filters[i] = filter;
				}
			}
			this.$set(this, 'filters', filters);
		},
		// openFilter(filterKey){
		//   let multiple = !this.selection.settings.multiple_filters;
		//   let filters = JSON.parse(JSON.stringify(this.filters));
		//   for(let i =0; i< filters.length; i++){
		//     if(filters[i].key === filterKey){
		//       filters[i].open = !filters[i].open;
		//       if(filters[i].open && filters[i].type === 'Range Histogram'){
		//         let app = this;
		//         window.setTimeout(function(){
		//           app.$refs[filterKey][0].update({ from: filters[i].histogramChart.start, to: filters[i].histogramChart.end })
		//         }, 50)
				
		//       }
		//     }else{
		//       if(multiple){
		//         filters[i].open = false;
		//       }
		//     }
		//   }
		//   this.$set(this, 'filters', filters);
		// },
		filteredOptions(Options, Search){
		if(Options !== null){
				function escapeRegExp(s) {
				return s.replace(/[-/\\^$*+?.()|[\]{}]/g, "\\$&");
			}
			const words = Search
				.split(/\s+/g)
				.map(s => s.trim())
				.filter(s => !!s);
			const hasTrailingSpace = Search.endsWith(" ");
			const regexString = words
			.map((word, i) => {
				if (i + 1 === words.length && !hasTrailingSpace) {
					// The last word - ok with the word being "startswith"-like
					return `(?=.*\\b${escapeRegExp(word)})`;
				} else {
					// Not the last word - expect the whole word exactly
					return `(?=.*\\b${escapeRegExp(word)}\\b)`;
				}
			})
			.join("") + ".+";
			let filtered = Options.filter(item => {
				let regex = new RegExp(regexString, "gi");
				return regex.test(item.option);
			});
			return filtered;
			}else{
				return Options
			}
		
		},
		reset(){
		if (JSON.stringify(this.filters) !== this.filterCopy) {
			this.filters = JSON.parse(this.filterCopy);
			this.MapWidgetData(this.filtersActive)
		}
		},
		select(name, value){
		let filters = JSON.parse(JSON.stringify([...this.filters]));
		let filtersActive = this.filtersActive;
		for(let i=0; i < filters.length; i++){
			if(filters[i].name === name){
			for(let j=0; j < filters[i].options.length; j++){
				filters[i].options[j].enabled = value;
			}
			if(value === true){
				filters[i].active = false;
			}else{
				filters[i].active = true;
			}
			for(let k=0; k < filters[i].filters_on.length; k++){
				let key = filters[i].filters_on[k].key;
				for(let l=0; l < filtersActive.length; l++){
				if(filtersActive[l].key === key){
					filtersActive[l].active = true;
				}
				}
			}
			}
		}
		this.filtersActive = filtersActive;
		this.$set(this, 'filters', filters);
		},
		singleSelect(filter, option){
		let filters = JSON.parse(JSON.stringify(this.filters));
		let filtersActive = this.filtersActive;
		if(filter.type === "Single-Select"){
			for(let i=0; i < filters.length; i++){
			if(filters[i].name === filter.name){
				for(let j=0; j < filters[i].options.length; j++){
				filters[i].active = true;
				if(filters[i].options[j].option !== option.option){
					filters[i].options[j].enabled = false;
				}else{
					filters[i].options[j].enabled = true;
				}
				}
				for(let k=0; k < filters[i].filters_on.length; k++){
				let key = filters[i].filters_on[k].key;
				for(let l=0; l < filtersActive.length; l++){
					if(filtersActive[l].key === key){
					filtersActive[l].active = true;
					}
				}
				}
			}
			}
		}else if(filter.type === "Multi-Select"){
			for(let i=0; i < filters.length; i++){
			if(filters[i].name === filter.name){
				for(let j=0; j < filters[i].options.length; j++){
				filters[i].active = true;
				if(filters[i].options[j].option === option.option){
					filters[i].options[j].enabled = !filters[i].options[j].enabled
				}
				}
				for(let k=0; k < filters[i].filters_on.length; k++){
				let key = filters[i].filters_on[k].key;
				for(let l=0; l < filtersActive.length; l++){
					if(filtersActive[l].key === key){
					filtersActive[l].active = true;
					}
				}
				}
			}
			}
		}
		this.filtersActive = filtersActive;
		this.$set(this, 'filters', filters);
		},
		histogramRange(event, filterKey){
		let filters = JSON.parse(JSON.stringify(this.filters));
		let filtersActive = JSON.parse(JSON.stringify([...this.filtersActive]));
		let startRange = event.from;
		let endRange = event.to;
		for(let i=0; i < filters.length; i ++){
			if(filters[i].key === filterKey){
			filters[i].histogramChart = {
				start: startRange,
				end: endRange,
				startText: startRange,
				endText: endRange
			}
			if(filters[i].histogramChart.start === filters[i].options[0].option && filters[i].histogramChart.end === filters[i].options.slice(-1)[0].option){
			filters[i].active = false
			}else{
			filters[i].active = true
			}
			for(let k=0; k < filters[i].filters_on.length; k++){
				let key = filters[i].filters_on[k].key;
				for(let l=0; l < filtersActive.length; l++){
				if(filtersActive[l].key === key){
					filtersActive[l].active = true;
				}
				}
			}
			}
		}
		this.$set(this, 'filtersActive', filtersActive);
		this.$set(this, 'filters', filters);
		},
		transformUnixcode(timestamp){
		timestamp = timestamp * 1000;
		let date = new Date(timestamp);
		let dateAgenda = {
			year: date.getFullYear(),
			month: date.getMonth(),
			day: date.getDate()
		}
		return dateAgenda
		},
		filterClicked(filterKey){
			for(let i=0; i < this.filters.length; i++){
				if(this.filters[i].key === filterKey){
				this.filters[i].clicked = true;
				let related = this.filters[i].related_filters;
				for(let j=0; j < related.length; j++){
					related[j].active = true;
				}
				this.relatedFilters = related;
				}else{
				this.filters[i].clicked = false;
				}
			}
		},
		async MapWidgetData(filteredQueries){
			let datasetsFilter = [];
			let transformationsFilter = [];
			let pinpoints = [];
			let response;
			let oldPinpoints = JSON.parse(JSON.stringify([...this.pinpoints]));
			let allOptions = this.filteredData;
			let newPinpoints = [];
			let existingPinpoints = [];
			for(let i=0; i < filteredQueries.length; i++){
				datasetsFilter.push(...filteredQueries[i].queries)
			}
			for(let i=0; i < this.transformations.length; i++){
			let id = this.transformations[i].id;
				if(datasetsFilter.includes(id)){
					transformationsFilter.push(this.transformations[i])
				}
			}
			let queries = transformationsFilter;
			for(let i=0; i < queries.length; i++){
				let module = queries[i];
				let id = module.id
				let options = {};
				let dataSet = this.selection.datasets.filter(item => item.datasetID.id === id);
				let limit;
				options[id] = allOptions[id];
				if(dataSet[0].hasOwnProperty("limit") === true){
					limit = parseInt(dataSet[0].limit)
				}else{
					limit = 100
				}
				
				if(module.app === null && module.collection === null){
					response = await API.Collection.Aggregate(this.$store.state.general.auth.token, "",  "", module.transformations, {}, "GPS DATA" , 0, limit, "ID", "ASC", "", this.itemId, module.filter_module_id, module.filter_query_id, options);
					module.total = response.data.total;
				}else{
					// main module moet 0 zijn en de filter_query_id dat is de datasetID
					response = await API.Collection.Aggregate(this.$store.state.general.auth.token, module.app, module.collection, module.transformations, {}, "GPS DATA" ,  0, limit, "ID", "ASC", "", this.itemId, 0, id, options);
					module.total = response.data.total;
				}
				if(response.data.hasOwnProperty("data") === true){
					let data = response.data.data;
					let longitude = dataSet[0].longitude;
					let latitude = dataSet[0].latitude;
					let color = dataSet[0].color;
					let icon = dataSet[0].icon;
					let source = dataSet[0].source;
					let picture = dataSet[0].picture;
                    if(picture.enableUpload == true){
                        let folderPath = ["file_upload", 'pinpoints']
                        let res = await API.File.ReadOne(this.$store.state.general.auth.token, folderPath, picture.full_name);
                        picture = res.data.data
                    } else {
                        picture = picture.url
                    }
					let popup = dataSet[0].togglePopUp;
					let enableActions = dataSet[0].enableActions;
					let actionButtons = dataSet[0].actionButtons;
					for(let j=0; j< data.length; j++){
					let details = JSON.parse(JSON.stringify([...dataSet[0].details]));
					for(let k=0; k< details.length; k++){
						details[k].value = data[j][details[k].key]
					}
					let pinpoint = {
						id: this.generateKey(),
						longitude: data[j][longitude],
						latitude: data[j][latitude],
						color: color,
						icon: icon,
						picture: dataSet[0].picture,
						pictureData: picture,
						details: details,
						dataset: dataSet[0].datasetID.id,
						popup: popup,
						source: source,
						enableActions:enableActions,
						actionButtons:actionButtons
					}
					pinpoints.push(pinpoint);
					}
				}
			}
			for(let i=0; i < this.filtersActive.length; i++){
				this.filtersActive[i].active = false;
			}
			let datasetIDS = filteredQueries[0].queries;
			for(let i=0; i < oldPinpoints.length; i++){
				let id = oldPinpoints[i].dataset;
				if(datasetIDS.includes(id) === false){
				existingPinpoints.push(oldPinpoints[i])
				}
			}
			newPinpoints = [...existingPinpoints, ...pinpoints]
			this.$set(this, 'pinpoints', newPinpoints);
		},
		async updateFilterOptions(filteredQueries){
			// let related_filters = [];
			// let optionKeys = [];
			let filters = [];
			let filter_ID = 0
			let module = JSON.parse(JSON.stringify({...this.activeModule}));
			let data = JSON.parse(JSON.stringify([...this.filters]));
			let allData = data;
			let clickedFilters =  data.filter( item => item.clicked === true);
			let datasetsFilter = [];
			let filterBarFilters = [];
			for (let i = 0; i <  clickedFilters.length; ++i){
				filter_ID = clickedFilters[i].id;
				let filter = {
					filterid: module.selection.filters[i].id,
					filters_on: []
				}
				for(let j = 0; j < clickedFilters[i].filters_on.length; ++j){
					let filterContent = {
						queryid: clickedFilters[i].filters_on[j].query.id,
						optionKeys: clickedFilters[i].filters_on[j].key,
						appId: "",
						collection: ""
					}
					filter.filters_on.push(filterContent);
				}
				filterBarFilters.push(filter);
				
				for (let j = 0; j < clickedFilters[i].transformations.length; ++j){
					for(let d = 0; d < filterBarFilters.length; ++d){
						for(let f = 0; f < filterBarFilters[d].filters_on.length; ++f){
							if(clickedFilters[i].transformations[j].id == filterBarFilters[d].filters_on[f].queryid){
								filterBarFilters[d].filters_on[f].appId = clickedFilters[i].transformations[j].app;
								filterBarFilters[d].filters_on[f].collection = clickedFilters[i].transformations[j].collection;
								break;
							}
						}
					}
				}
			
				if(clickedFilters[i].related_filters.length > 0){
					for (let j = 0; j < clickedFilters[i].related_filters.length; ++j){
						for (let k = 0; k < module.selection.filters.length; ++k){
							if(clickedFilters[i].related_filters[j].id == module.selection.filters[k].id){
								filters.push(module.selection.filters[k]);
							}
						}
					}
				}
			}
			let responseData = [];
			for(let i = 0; i < filters.length; ++i){
				let filterid = filters[i].id;
				let specificFilter = [];
				specificFilter.push(filters[i])
				
				for(let j = 0; j < filters[i].filters_on.length; ++j){
					let app = "";
					let collection = "";
					
					let optionKeys = [];
					let pipeline = [];
					for(let d = 0; d < filterBarFilters.length; ++d){
						
						for(let f = 0; f < filterBarFilters[d].filters_on.length; ++f){
							if(filters[i].filters_on[j].query.id == filterBarFilters[d].filters_on[f].queryid){
								app = filterBarFilters[d].filters_on[f].appId;
								collection = filterBarFilters[d].filters_on[f].collection;
								optionKeys.push(filterBarFilters[d].filters_on[f].optionKeys);
							};
						}
					}
					
					for (let p = 0; p < module.transformations.length; ++p){
						if(module.transformations[p].id == filters[i].filters_on[j].query.id){
							pipeline.push(module.transformations[p]);
							break;
						}
					}
					let item = {
						filter: specificFilter,
						information: module.selection.information,
						name: module.selection.name,
						settings: module.selection.settings,
						optionKeys: optionKeys
					}
					let response = await API.Collection.Aggregate(
							this.$store.state.general.auth.token, 
							app, 
							collection, 
							pipeline, 
							item, 
							"Inner Filter Bar", 
							this.dataSettings.offset, 
							this.dataSettings.limit, 
							this.dataSettings.sorting, 
							this.dataSettings.order, 
							this.dataSettings.search, 
							this.itemId, 
							module.filter_module_id, 
							module.filter_query_id, 
							this.filteredData, 
							this.deepDive);
					
					if(response.data.code === 200 ){
						for (let k = 0; k < response.data.data.length; ++k){
							let it = {
								filterid: filterid,
								data: response.data.data[k] 
							}
							responseData.push(it);
						}
					}
				}
			}
			let filteredData_ = allData;
        	let moduleData = [];
			for(let i = 0; i < responseData.length; ++i){
				for(let d = 0; d < filteredData_.length; ++d){
					let filterid = responseData[i].filterid; // FILTER ID
					
					let data = responseData[i].data; // FILTER DATA
					if(filteredData_[d].name == data.name){
						if(data.options != null){
							let found = false;
							if(moduleData.filter( item => item.name == data).length > 0){
								let old_data = moduleData.filter( item => item.name == data.name);
								let new_options = data.options;
								let old_options = old_data[0].options;
								for (let nw = 0; nw < new_options.length; ++nw) {
									for (let old = 0; old < old_options.length; ++old) {
										if(old_options[old].option == new_options[nw].option){
											found = true;
											let old_count = old_options[old].count;
											let new_count = new_options[nw].count;
											old_options[old].count = old_count + new_count
											break;
										}
									}
									if(!found){
										old_options.push_back(new_options[nw]);
									}
								}
							}else {
								let it = {
									filterid: filterid,
									options: data.options
								}
								moduleData.push(it)
							}
						} else {
							let it = {
								filterid: filterid,
								options: []
							}
							moduleData.push(it)
						}
					}
				}
			}
			
			for (let i = 0; i < filteredData_.length; ++i){
				for (let d = 0; d < moduleData.length; ++d){
					if(filteredData_[i].id == moduleData[d].filterid){
						filteredData_[i].options = moduleData[d].options
					}
				}
			}
			this.$set(this, 'filters', filteredData_);
		},
		async actionFunction(actions, item){
				
			for await (const action of actions){
				if (action.type == "Link"){
                    await this.linkItem(item, action.link, action.linkExternalTab);
                } else if (action.type == "External Link"){
                    await this.externalLinkItem(item, action.to, action.linkExternalTab, action.extern);
                }
			}    
		},
		linkItem: function (item, link, toExternal){
            let itemKey = "";
            for(let i = 0; i < item.length; i++){
                if(item[i].key === link.key){
                    itemKey = item[i].value;
                }
            }
            let url = "/app/" + this.currentApp + "/" + link.page + "/" + itemKey; 
            if(toExternal){
                window.open(url);
            } else {
                this.$router.push(url)
            }
        },
		externalLinkItem: function(item, to, toExternal, linkExtern){
			this.pageloading = true;
			let url = to;
			let text = "";
			if(url.includes("$")){
				let it = url.split( '/' );
				
				for (const [index, path] of Object.entries(it)) {
					if(path.length > 0 ){
						if(path.startsWith("$")){
							let p = path.substring(1);
                        
                            for(let i = 0; i < item.length; i++){
                                if(item[i].key === p){
                                    text += ('/'+ item[i].value)
                                }
                            }
						} else {
						text += ('/' + path);
						}
					}
				}
				let k;
				if(linkExtern){
					k = 'https://' + text;
				} else {
					k = text
				}
				if(toExternal){
					
					window.open(k);
				} else {
					if (linkExtern) {
						window.location.href = k;
					} else {
						this.$router.push(k)
					}
				}
			} else {
				let k;
				if(linkExtern){
					k = 'https://' + url;
				} else {
					k = url
				}
				if(toExternal){
					window.open(k);
				} else {
					if (linkExtern) {
						window.location.href = k;
					} else {
						this.$router.push(k)
					}
				}
				
			}
			this.pageloading = false;
		}
 	},
  	computed: {
		...mapGetters("page", ['getActiveModule']),
		...mapGetters("page", ['getDeepDive']),
		...mapGetters("page", ['getMainModule']),
		...mapGetters("page", ['getTableSettings']),
		deepDive: function(){
            return this.getDeepDive(this.widgetId);
        },
		activeModule: function(){
            return this.getActiveModule(this.widgetId);
        },
		mainModule: function(){
            return this.getMainModule(this.widgetId);
        },
		dataSettings: function(){
            return this.getTableSettings(this.widgetId);
        },
		showClearFilters(){
		if(JSON.stringify(this.filters) !== this.filterCopy){
			return true
		}else{
			return false
		}
		},  
		filterEnabled: function(){
		return this.selection.settings.enable_filters;
		},
		filteredData(){
		// pas de berekening maken als de pagina geladen is
		if(this.componentLoaded){
			let queryOptions = {};
			let data = JSON.parse(JSON.stringify([...this.filters]));
		// per query de keys sorteren en de relevante opties
			for(let i=0; i < data.length; i++ ){
			for(let j=0; j < data[i].filters_on.length; j++){
				let query = data[i].filters_on[j].query.id;
				let options;
				let key = data[i].filters_on[j].key;
				if(queryOptions.hasOwnProperty(query) === false){
					queryOptions[query] = {};
				}
				if(data[i].type === 'Multi-Select' || data[i].type === 'Single-Select'){
				if(data[i].options !== null && data[i].options !== undefined && data[i].options !== ""){
					options = data[i].options.filter(item => {
						return item.enabled === true
					})
				}else{
					options = [];
				}
				}
				if(data[i].type === 'Range Histogram'){
				if(Object.keys(data[i].histogramChart).length > 0){
					options = {
					start: data[i].histogramChart.start,
					end: data[i].histogramChart.end
					};
				}else{
					options = { start: null, end: null}
				}
				}
				if(data[i].type === 'Agenda'){
				if(data[i].filterRange !== null){
					let start;
					let end;
					if(typeof data[i].filterRange.start === 'string'){
						start = ((new Date(data[i].filterRange.start)).getTime() / 1000);
						end = ((new Date(data[i].filterRange.end)).getTime() / 1000);
					}else{
						start = (data[i].filterRange.start.getTime() / 1000);
						end = (data[i].filterRange.end.getTime() / 1000);
					}
					options = { start: start, end: end}
				}else{
					options = { start: null, end: null}
				}
				} 
				queryOptions[query][key] = [{
					options: options,
					type: data[i].type
				}]
			}
			}
			return queryOptions;
		}         
		},
  	},
    async created(){
		// this.mapbox = Mapbox;
		let datasets = this.selection.datasets;
		let queries = this.transformations;
		let pinpoints = [];
		let response;
		let locationStart = this.selection.locationStart;
		let locationDataSets = this.selection.locationStartDatasets;
        // console.log('Mapbar', this.selection);
        let syntaxMapbar = 'mapbox://styles/clappform/';
        let mapbarStyleId = "";
        if(this.selection.hasOwnProperty("mapStyle") === false){
            mapbarStyleId = 'ckvwah9it188014qehjcys4nj';  
        } else {
            mapbarStyleId  = this.selection.mapStyle;
        }
        
        let mapbar = syntaxMapbar + mapbarStyleId
        this.mapStyle = mapbar;
		for(let i=0; i < datasets.length; i++){
			let module = queries.find(item => item.id === datasets[i].datasetID.id);
			let limit;
			if(datasets[i].hasOwnProperty("limit") === true){
			limit = parseInt(datasets[i].limit)
			}else{
			limit = 100
			}
			if(module.app === null && module.collection === null){
			    response = await API.Collection.Aggregate(this.$store.state.general.auth.token, "",  "", module.transformations, {}, "GPS DATA" , 0, limit, "ID", "ASC", "", this.itemId, module.filter_module_id, module.filter_query_id);
				module.total = response.data.total;
			}else{
				response = await API.Collection.Aggregate(this.$store.state.general.auth.token, module.app, module.collection, module.transformations, {}, "GPS DATA" ,  0, limit, "ID", "ASC", "", this.itemId, false, false,);
				module.total = response.data.total;
			}
			if(response.data.hasOwnProperty("data") === true){
			let data = response.data.data;
			let longitude = datasets[i].longitude;
			let latitude = datasets[i].latitude;
			let color = datasets[i].color;
			let icon = datasets[i].icon;
			let source = datasets[i].source;
			let picture = datasets[i].picture;
            if(picture.enableUpload == true){
                let folderPath = ["file_upload", 'pinpoints']
                let res = await API.File.ReadOne(this.$store.state.general.auth.token, folderPath, picture.full_name);
                picture = res.data.data
            } else {
                picture = picture.url
            }
            
			let popup = datasets[i].togglePopUp;
			let enableActions = datasets[i].enableActions;
			let actionButtons = datasets[i].actionButtons;
			for(let j=0; j< data.length; j++){
				let details = JSON.parse(JSON.stringify([...datasets[i].details]));
				for(let k=0; k< details.length; k++){
				details[k].value = data[j][details[k].key]
				}
				let pinpoint = {
				id: this.generateKey(),
				longitude: data[j][longitude],
				latitude: data[j][latitude],
				color: color,
				icon: icon,
                picture: datasets[i].picture,
                pictureData: picture,
				details: details,
				dataset: datasets[i].datasetID.id,
				popup: popup,
				source: source,
				enableActions:enableActions,
				actionButtons:actionButtons
				}
				pinpoints.push(pinpoint);
			}
			}
		}
            // set dynamic start locatuion
		if(locationStart === 'Custom'){
			if(typeof this.selection.latitude === 'string' && typeof this.selection.longitude === 'string'){
			let location = [];
			location.push(parseFloat(this.selection.longitude));
			location.push(parseFloat(this.selection.latitude));
			this.center = location;
			}
		}
		else if((locationStart === 'Modus' || locationStart === 'Median') && locationDataSets.length > 0){
			let startDataSets = locationDataSets.map(function(value){return value.id});
			let locationStartPoints = pinpoints.filter(item => startDataSets.includes(item.dataset))
			let latitudes = locationStartPoints.map(function(value){return value.latitude});
			let longitudes = locationStartPoints.map(function(value){return value.longitude});
			if(locationStart === 'Median'){
			let location = [this.median(longitudes), this.median(latitudes)];
			this.center = location;
			}else if(locationStart === 'Modus'){
			let location = [this.modus(longitudes), this.modus(latitudes)];
			this.center = location;
			}
		}
		if(typeof this.selection.zoom === 'string'){
			this.zoom = parseInt(this.selection.zoom)
		}
		
		// set the filteroptions;
		if(this.filterEnabled){
			let filters = JSON.parse(JSON.stringify([...this.data]));
			let selectionFilter = JSON.parse(JSON.stringify([...this.selection.filters]));
			let module = JSON.parse(JSON.stringify({...this.activeModule}));
			let filtersActive = [];
			for(let i=0; i < filters.length; i++){
			// settings die voor elke filter moeten
			filters[i].active = false;
			filters[i].key = this.generateKey(),
			filters[i].open = false;
			filters[i].clicked = false;
			for(let k=0; k < selectionFilter.length; k++ ){
			if(filters[i].name === selectionFilter[k].name){
				filters[i].filters_on = selectionFilter[k].filters_on
				filters[i].color = selectionFilter[k].color
				filters[i].id = selectionFilter[k].id;
				filters[i].related_filters = selectionFilter[k].related_filters;
				if(selectionFilter[k].hasOwnProperty("showCount") === false){
					filters[i].showCount = true;
				} else {
					filters[i].showCount = selectionFilter[k].showCount;
				}
				filters[i].transformations = module.transformations;
			}
			}
		if(filters[i].type === 'Multi-Select' || filters[i].type === 'Single-Select'){
			filters[i].search = "";
		}
		if(filters[i].type === 'Agenda'){
			filters[i].filterRange = null;
			filters[i].dateRange = {
			start: this.transformUnixcode(filters[i].options[0].option),
			end: this.transformUnixcode(filters[i].options[filters[i].options.length - 1].option),
			}
		}
		if(filters[i].type === 'Range Histogram'){
			let histogramOptions = [];
			let filterOptions = filters[i].options;
			filterOptions.forEach(element =>{
			let items = Array(element.count).fill(element.option);
			for(let j =0; j <items.length; j ++){
				histogramOptions.push(items[j]);
			}
			});
			filters[i].histogramOptions = histogramOptions;
			filters[i].histogramChart = {};
			filters[i].active = false;
		}
		if(filters[i].options === null){
			filters[i].options = [];
		}
		for(let j=0; j < filters[i].options.length; j++){
			filters[i].options[j].enabled = true;
			}
		}
		for(let i=0; i < filters.length; i++){
				// make data object to check which widget needs to be updated
			for(let l=0; l < filters[i].filters_on.length; l++ ){
			let key = filters[i].filters_on[l].key;
			// check if key already exists 
			if(filtersActive.some(item => item.key === key) === true){
				for(let m=0; m< filtersActive.length; m++){
				if(filtersActive[m].key === key){
					filtersActive[m].queries.push(filters[i].filters_on[l].query.id)
				}
				}
			}else{
				filtersActive.push({key: key, queries: [filters[i].filters_on[l].query.id], active: false});
			}
			}
		}
		this.$set(this, 'filterCopy', JSON.stringify(filters));
		this.$set(this, 'filters', filters);
		this.$set(this, 'filtersActive', filtersActive);
		};
        
		this.pinpoints = pinpoints;
		this.componentLoaded = true;
  	},
    watch: {
		filtersActive: {
			handler() {
				let filteredQueries = this.filtersActive.filter(item => item.active === true);
				if(filteredQueries !== 'undefined' && filteredQueries !== undefined && filteredQueries.length > 0){
					this.updateFilterOptions(filteredQueries);
					this.MapWidgetData(filteredQueries);
				}
			},
			deep: true
		}, 
	}
};
</script>

<style scoped lang="scss">
.map-container{
  flex: 1;
  border-radius: 6px;
  position: relative;
  padding-bottom: 15px;
}
.map{
    width: 100%;
    height: 500px;
    background-color: #fff;
    border-radius: 14px;
}
.popup-container{
  display: flex;
  .popup{
    display:flex;
    flex-direction: column;
    padding: 0 5px;
    .popup-title{
      font-size: 12px;
      font-family: "Galyon Bold";
      padding-bottom: 4px;
    }
    .popup-content{
        font-family: "Galyon Regular";
        font-size: 11px;   
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
    }
  }
}
h3 {
  font-family: "Galyon Regular";
  left: 25px;
  font-size: 14px;
  color: #637381;
  font-size: 18px;
  padding: 5px;
}
.dot {
  height: 13px;
  width: 13px;
  /* background-color: rgb(59, 112, 236); */
  border-radius: 50%;
  display: inline-block;
  border: 2px solid white;
  //box-shadow: 2px 2px 5px 1px rgb(42, 42, 42);
}
.icon-container {
    width: 15px;
    height: 15px;
}
.smallDot {
  height: 10px;
  width: 10px;
  background-color: rgb(255, 255, 255);
  border-radius: 50%;
  display: inline-block;
}
.filter-bar {
  display: flex;
  background-color: $color-white-light;
  border-radius: 10px;
  position: absolute;
  box-shadow: rgba(0, 0, 0, 0.10) 0px 3px 8px;
  left: 50px;
  top: 20px;
  z-index: 10;
  .filter {
    position: relative;
    display: flex;
    // & ~ .filter {
    //     margin-left: 15px;
    // }
    &:first-child{
      border-radius: 10px 0 0 10px;
    }
    &:last-child{
      border-radius: 0 10px 10px 0;
    }
    &:active {
      background-color: $color-white-dark;
    }
    &:hover {
      background-color: $color-white;
    }
    &.open {
      background-color: rgba($color-grey, 0.4);
      &:hover {
        background-color: $color-grey-light;
      }
      &:active {
        background-color: rgba($color-grey-dark, 0.4);
      }
    }
  }
  .filter-type{
    display: inline;
  }
  button {
    display: flex;
    align-items: center;
    background-color: transparent;
    border: none;
    outline: 0;
    padding: 8px 8px;
    font-family: "Galyon Regular";
    font-size: 12px;
    color: $color-grey-dark;
    &.remove {
      padding: 0 0 0 5px;
      @include icon-size(15px, 15px);
    }
    .filter-type-indicator{
        width: 5px;
        height: 5px;
        border-radius: 50%;
        background-color: transparent;
      &.active{
        background-color: $color-success;
      }
    }
    .filter-type-name{
      padding: 0 8px;
    }
    .icon-container {
      width: 10px;
      transition: $transition-default;
    }
    &.open {
      .icon-container {
        transform: rotate(180deg);
      }
    }
  }
}
.actions {
    display: flex;
    flex-direction: column;
    // //justify-content: center;
    align-items: center;
    padding: 0 5px;
    .actions-title{
      font-size: 12px;
      font-family: "Galyon Bold";
      padding-bottom: 4px;
    }
    .actions-content{
        font-family: "Galyon Regular";
        font-size: 11px;    
        flex: 1;       
        display: flex;
        align-items: center;
    }
    .action_button{
        opacity: 1;
        height: 12px;
        width: 12px;
        color: $color-white;
        background-color: $color-secondary;
        transition: $transition-default;
        border-radius: $rounded-primary;
        display: flex;
        align-items: center;
        justify-content: center;
        // margin-right: 8px;
        border: none;
        
        .icon-container {
            width: 12px;
            height: 12px;
            transform: scale(80%);
        }
        &.disabled {
            background-color: $color-grey-light !important;
            &:hover, *  {
                cursor: not-allowed !important;
                pointer-events: all !important;
            }
        }
        &:hover{
                cursor: pointer;
                opacity: 0.8;
            }
    }
}
.checklist {
    position: absolute;
    top: 32px;
    left: 0px;
    min-width: 250px;
    border-radius: $rounded-secondary;
    background-color: $color-white-light;
    padding: 20px 15px 20px 25px;
    z-index: 1000;
    max-height: 320px;
    border: 1px solid $color-grey-light;
    .filter-search-container {
      width: 95%;
      display: flex;
      align-items: center;
      margin-bottom: 12px;
      padding: 0px 0px 0px 15px;
      border: 0;
      border-radius: $rounded-secondary;
      background-color: $color-grey-light;
      .search-icon-container {
        flex: 0 0 18px;
        display: flex;
        align-items: center;
      }
      .search-icon {
        width: 12px;
      }
  input {
    font-size: 12px;
    padding: 4px 0px;
    margin-left: 12px;
    font-family: "Galyon Regular";
    border: none;
    outline: 0;
    background: transparent;
    flex: 1;
    }
  }
    .scroll-container {
        max-height: 220px !important;
        min-height: 50px;
        overflow-x: hidden;
        overflow-y: auto;
    }
    .selection-buttons {
        margin-top: 7px;
        display: flex;
        button.selection-button {
            background: none;
            border: none;
            color: $color-grey-dark;
            font-size: 12px;
            padding: 0 5px;
            text-decoration: underline;
            text-decoration-color: $color-grey;
            &:hover {
                text-decoration: none;
            }
            @media only screen and (max-width: $standard-laptop-screen){
              font-size: 12px;
            }
        }
    }
    // .option {
    //     font-family: "Galyon Regular";
    //     font-size: 14px;
    //     color: $color-grey-dark;
    //     margin-bottom: 5px;
    //     input {
    //         display: inline;
    //     }
    //     div {
    //         display: inline;
    //         margin-left: 10px;
    //     }
    // }
}
.dropdown {
    position: absolute;
    top: 32px;
    left: 0px;
    border-radius: $rounded-secondary;
    background-color: $color-white-light;
    padding: 20px 15px 20px 15px;
    z-index: 1000;
    border: 1px solid $color-grey-light;
    .range-number-input{
      display: flex;
      width: 100%;
      margin-top: 8px;
      justify-content: space-between;
      .input-container{
        display: flex;
        align-items: center;
        flex-basis: 40px;
      }
      .title{
        color: $color-grey-dark;
        font-family: "Galyon Regular";
        font-size: 12px;
        display: flex;
      }
      .number-input{
        display: flex;
        
        padding: 0px 0px 0px 8px;
        border: 0;
        color: $color-grey;
        font-size: 12px;
    font-family: "Galyon Regular";
    border: none;
    outline: 0;
    &:focus {
      border: 0;
      outline: none;
    }
      }
    }
}
/* Chrome, Safari, Edge, Opera */
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
/* Firefox */
input[type=number] {
  -moz-appearance: textfield;
}
.input-check{
  display: block;
  position: relative;
  padding-left: 25px;
  margin-bottom: 5px;
  white-space: nowrap;
  cursor: pointer;
  font-family: "Galyon Regular";
  font-size: 12px;
  color: $color-grey-dark;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
@media only screen and (max-width: $standard-laptop-screen){
  .input-check{
    font-size: 12px;
  }
}
.input-check input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
  height: 0;
  width: 0;
}
.checkmark {
  position: absolute;
  top: 0;
  left: 0;
  height: 12px;
  width: 12px;
  transition: $transition-default;
  background-color: $color-white-light;
  &.single{
    border-radius: 100%;
    width: 12px;
    height: 12px;
  }
}
        
/* On mouse-over, add a grey background color */
.checkmark {
  background-color: $color-grey-light;
}
/* When the checkbox is checked, add a blue background */
 .checkmark.enabled {
  background-color: $color-primary-light;
}
/* Create the checkmark/indicator (hidden when not checked) */
.checkmark:after {
  content: "";
  position: absolute;
  display: none;
}
/* Show the checkmark when checked */
.checkmark.enabled:after {
  display: block;
}
/* Style the checkmark/indicator */
.checkmark:after {
  left: 3px;
  top: 0.5px;
  width: 3px;
  height: 7px;
  border: solid white;
  border-width: 0 3px 3px 0;
  -webkit-transform: rotate(45deg);
  -ms-transform: rotate(45deg);
  transform: rotate(45deg) scaleY(0.8) scaleX(0.8);
}
.date-filter {
    position: absolute;
    top: 32px;
    left: 0px;
    border-radius: $rounded-secondary;
    background-color: $color-white-light;
    z-index: 1000;
    border: 1px solid $color-grey-light;
    min-width: 250px;
    padding: 5px 5px 0;
    max-height: unset;
    .vc-container {
        border: none;
    }
}
[v-cloak] {
    display: none;
}
</style>
