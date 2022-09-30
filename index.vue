<template>
    <div class="select-filters">
        <div class="filters" v-if="widget.selection.settings.enable_filters">
            <div class="sub-title">Configure Filters</div>
            <Draggable 
                handle=".filter-drag"
                v-bind="{ animation: 200 }"
                v-model="MapFilterBar.selection.filters">
            <div v-for="(filter, index) in MapFilterBar.selection.filters" :key="filter.id" class="filter-selection" @dragstart="dragstart" @dragend="dragend">
                <div class="filter-header">
                    <div class="filter-container">
                        <label class="name">Name: </label><input type="text" v-model="filter.name" maxlength="15" placeholder="max 15 characters"/>
                    </div>
                    <div class="filter-container type">
                        <label class="type">Type: </label>
                        <div class="type-select">
                            <v-select :options="['Range Histogram', 'Agenda', 'Single-Select', 'Multi-Select']" v-model="filter.type">
                            </v-select>
                        </div>
                    </div>
                    <div class="filter-container type" v-if="filter.type === 'Agenda'">
                        <label class="type">Time: </label>
                        <div class="toggle-switch">
                            <div class="switch" @click="filter.time = !filter.time">
                                <input type="checkbox" v-model="filter.time">
                                <span class="slider round"></span>
                            </div>
                        </div>
                    </div>
                    <div class="filter-container type" v-if="filter.type === 'Range Histogram'">
                        <label class="type">Color: </label>
                        <v-swatches v-model="filter.color"
                            show-fallback
                            fallback-input-type="color"
                            fallback-ok-text="Select"
                            popover-x="left"
                            popover-y="top"
                            shapes="circles"
                            :swatches="swatches"               
                        >
                        </v-swatches>
                    </div>
                    <div class="filter-container type">
                        <label class="type">Count: </label>
                        <button class="show-filter" @click="hideCount(index)">
                            <div class="icon-container">
                                <eye-icon color="grey" v-if="filter.showCount"></eye-icon>
                                <eye-blocked-icon color="grey" v-else></eye-blocked-icon>
                            </div>
                        </button>
                    </div>
                    <div class="filter-container type">
                        <label class="type">Related Filters: </label>
                        <div class="type-select">
                            <v-select 
                                :options="optionsRelatedFilters" 
                                v-model="filter.related_filters"
                                :reduce="optionsRelatedFilters => optionsRelatedFilters.value"
                                :multiple="true"
                            ></v-select>
                        </div>
                    </div>
                    <div class="filter-container">
                        <button class="filter-drag">
                            <div class="icon-container">
                                <drag-vertical-icon color="grey"></drag-vertical-icon>
                            </div>
                        </button>
                        <button class="delete-filter" @click="removeFilter(filter.id)">
                            <div class="icon-container">
                                <cross-icon color="grey"></cross-icon>
                            </div>
                        </button>
                    </div>
                </div>
                <h4>Applies to</h4>
                <div v-for="filter_on in filter.filters_on" :key="filter_on.id" class="applies-to">
                    <div class="applies-container">
                        <div class="applies-property">
                            <label>DataSet:</label>
                            <div class="query-select">
                                <v-select :options="queries" v-model="filter_on.query" @input="getKeys(filter_on.query.name); filter_on.key = ''" label="name">
                                </v-select>
                            </div>
                        </div>
                        <div class="applies-property" v-if="getQuery(filter_on.query.name).keys">
                            <label>Key:</label>
                            <div class="query-select"><v-select :options="getQuery(filter_on.query.name).keys" v-model="filter_on.key"></v-select></div>
                        </div>
                    </div>
                    <div class="applies-container">
                        <button class="delete-filter-on" @click="removeFiltersOn(filter.id, filter_on.id)">
                            <div class="icon-container">
                                <cross-icon color="grey"></cross-icon>
                            </div>
                        </button>
                    </div>
                </div>
                <button @click="addFiltersOn(index)" class="add-button">
                    <div class="icon-container">  
                        <add-icon color="grey"></add-icon>
                    </div>
                </button>
            </div>
            </Draggable>
            <button @click="addFilter" class="add-button">
                <div class="icon-container">  
                    <add-icon color="grey"></add-icon>
                </div>
            </button>
        </div>

<!-- het configureren van de map -->
        <div class="map-configuration">
            <div class="sub-title">Configure Map</div>
            <div class="general-settings">
                <label for="widgetName">Title</label>
                <input type="text" id="widgetName" v-model="MapFilterBar.selection.name" placeholder="Provide title" />

                <v-select 
                        :options="['Custom', 'Median', 'Modus']"
                        v-model="MapFilterBar.selection.locationStart">
                        </v-select>

                <div v-if="MapFilterBar.selection.locationStart === 'Custom'">
                    <label for="longstart">longitude start</label>
                    <input type="number" id="longstart" v-model="MapFilterBar.selection.longitude" placeholder="Provide Long start" />

                    <label for="latstart">Latitude Start</label>
                    <input type="number" id="latstart" v-model="MapFilterBar.selection.latitude" placeholder="Provide lat start" />
                </div>

                
                <div v-if="MapFilterBar.selection.locationStart === 'Median' || MapFilterBar.selection.locationStart === 'Modus'">
                    Which datasets are used to determine starting location
                    <v-select 
                        :options="queries"
                        label="name"
                        v-model="MapFilterBar.selection.locationStartDatasets"
                        :multiple="true"
                    ></v-select>
                </div>


                <label for="zoom">Zoom Start</label>
                <input type="number" id="zoom" v-model="MapFilterBar.selection.zoom" placeholder="Provide zoom start" />
                
                <label>Map Style</label>
                <v-select 
                    :options="getMapboxStyles"
                    :reduce="getMapboxStyles => getMapboxStyles.value"
                    v-model="MapFilterBar.selection.mapStyle">
                </v-select>
                

                <label for="widgetName">Information</label>
                <!-- <input id="widgetName" v-model="MapFilterBar.selection.information" placeholder="Information tooltip text" /> -->
                <DashboardVisualizationsWysiwyg
                    @Update="UpdateInformation"
                    :selectionContent="MapFilterBar.selection.information"
                >
                </DashboardVisualizationsWysiwyg>
            </div>
            <div class="dataset" v-for="(dataset, index) in MapFilterBar.selection.datasets" :key="dataset.id">
                <div class="map-selection">
                    <div class="selection">
                        <label>Dataset:&nbsp;</label>
                        <v-select :options="queries" v-model="dataset.datasetID" @input="getKeysDataSet(dataset.datasetID)" label="name">
                        </v-select>
                    </div>
                    <div class="selection" v-if="dataset.datasetID !== null">
                        <label>Long:&nbsp;</label>
                            <v-select 
                            :options="getColumnsData('number', dataset.keys)" 
                            v-model="dataset.longitude">
                            </v-select>
                            
                    </div>
                    <div class="selection" v-if="dataset.datasetID !== null">
                        <label>Lat:&nbsp;</label>
                        <v-select 
                        :options="getColumnsData('number', dataset.keys)"
                        v-model="dataset.latitude">
                        </v-select>
                    </div>
                    <div class="button-container">
                        <button class="delete dataset-button" @click="deleteDataset(index, dataset.id)" >
                            <div class="icon-container">
                                <cross-icon color="grey"></cross-icon>
                            </div>
                        </button>
                    </div>
                </div>
                <div class="map-pinpoint">
                    <div class="selection" v-if="dataset.datasetID !== null">
                        <label>Items:&nbsp;</label>
                        <div class="input-group-number">
                            <input type="number" placeholder="Amount pinpoints" v-model="dataset.limit" max="1000"/>
                        </div>
                        <!-- <div class="button-container">
                            <button class="crease" @click="increase()">
                            <add-icon color="grey-dark" class="add-icon"></add-icon>
                            </button>
                            <button class="crease" @click="decrease()">
                            <substraction-icon color="grey-dark" class="add-icon de"></substraction-icon>
                            </button>
                        </div> -->
                    </div>
                    <div class="selection" v-if="dataset.datasetID !== null">
                        <label>Pinpoint:&nbsp;</label>
                        <v-select 
                        :options="['Color', 'Picture', 'Icon', 'SVG']"
                        :value="dataset.source"
                        @input="setPinPoint(dataset.id, $event)"
                        >
                        </v-select>
                    </div>
                    <div class="selection" v-if="dataset.source === 'Color'">
                        <label>Color:</label>
                        <div class="custom-color-picker">
                            <v-swatches v-model="dataset.color"
                                show-fallback
                                fallback-input-type="color"
                                fallback-ok-text="Select"
                                popover-x="left"
                                popover-y="top"
                                shapes="circles"
                                :swatches="swatches"               
                            >
                            </v-swatches>
                        </div>
                    </div>
                    <div class="selection" v-if="dataset.source === 'Icon'">
                        <label>Icon:</label>
                        <transition type="transition">                            
                            <GeneralIconPicker id="icon" 
                                :actionsSettings.sync="dataset.icon" 
                                :actionKey="dataset.id" 
                                :identifier="'pinpoint'"
                                @setIcon="setIcon">
                                <div class="picker"></div>
                            </GeneralIconPicker>
                        </transition>
                    </div>
                    <div class="selection-toggle" v-if="dataset.source === 'Picture' || dataset.source === 'SVG'">
                        <div class="toggle-container">
                            <label>Upload:&nbsp;</label>
                            <div class="toggle-switch">
                                <div class="switch left" @click="dataset.picture.enableUpload=!dataset.picture.enableUpload">
                                    <input type="checkbox" v-model="dataset.picture.enableUpload">
                                    <span class="slider round"></span>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="selection-name" v-if="(dataset.source === 'Picture' || dataset.source === 'SVG') && dataset.picture.enableUpload === false">
                        <label>Picture&nbsp;Url:&nbsp;</label>
                        <input type="text" v-model="dataset.picture.url" placeholder="Provide Picture Url" />
                    </div>
                    <div class="selection-name upload" v-if="(dataset.source === 'Picture' || dataset.source === 'SVG') && dataset.picture.enableUpload === true">
                        <input type="file" :accept="''" @change="readFile" ref="input" id="fileUpload" hidden/>
                        <div class="icon-upload">
                            <div class="icon-container" @click="targetUpload(index)">
                                <upload-icon color="grey" class="input-icon"></upload-icon>
                            </div>
                        </div>
                        <div v-if="dataset.picture.uploaded">
                            <span>{{dataset.picture.name}}</span>
                        </div>
                        
                    </div>
                </div>
                <div class="toggle-container">
                    <div class="toggle-name">Pop up</div>
                    <div class="toggle-switch">
                        <div class="switch left" @click="dataset.togglePopUp=!dataset.togglePopUp">
                            <input type="checkbox" v-model="dataset.togglePopUp">
                            <span class="slider round"></span>
                        </div>
                    </div>
                </div>
                <div v-if="dataset.togglePopUp">
                    <div class="map-pipoint-details" v-for="(details, details_index) in dataset.details" :key="details.id">
                        <div class="selection-name">
                            <label>Name:&nbsp;</label>
                            <input type="text"  v-model="details.name" placeholder="Provide title" />
                        </div>
                        <div class="selection">
                            <label>Key:&nbsp;</label>
                            <v-select 
                            :options="getColumns()"
                            v-model="details.key" 
                            >
                            </v-select>
                        </div>
                        <div class="button-container">
                            <button class="delete dataset-button" @click="deletePopUpDetails(index, details_index)" >
                                <div class="icon-container">
                                    <cross-icon color="grey"></cross-icon>
                                </div>
                            </button>
                        </div>
                    </div>
                </div>
                <div class="button-container" v-if="dataset.togglePopUp">
                    <button class="dataset-button" @click="addPopUpDetails(index)" >
                        <div class="icon-container">
                            <add-icon color="grey"></add-icon>
                        </div>
                    </button>
                </div>
                <div class="toggle-container">
                    <div class="toggle-name">Actions</div>
                    <div class="toggle-switch">
                        <div class="switch left" @click="dataset.enableActions=!dataset.enableActions">
                            <input type="checkbox" v-model="dataset.enableActions">
                            <span class="slider round"></span>
                        </div>
                    </div>
                </div>
                <div class="data-settings-container" v-if="dataset.enableActions">
                    <div class="dataset row-container" v-for="actions in dataset.actionButtons" :key="actions.buttonKey">
                        <div class="row-single">
                            <button class="row-drag">
                                <div class="icon-container">
                                    <draggable-handle-icon color="grey"></draggable-handle-icon>
                                </div>
                            </button>
                            <div class="row-item">
                                <label for="widgetIcon">Icon:&nbsp;</label>
                                <transition type="transition">
                                    <GeneralIconPicker id="icon" 
                                        :actionsSettings.sync="actions.icon" 
                                        :actionKey="actions.buttonKey" 
                                        :dataset="dataset.id"
                                        :identifier="'actionButton'"
                                        @setIcon="setIcon">
                                        <div class="picker"></div>
                                    </GeneralIconPicker>
                                </transition>
                            </div>
                            <div class="color">
                                <label>Color:&nbsp;</label>
                                <div class="custom-color-picker">
                                    <v-swatches v-model="actions.color"
                                        show-fallback
                                        fallback-input-type="color"
                                        fallback-ok-text="Select"
                                        popover-x="left"
                                        popover-y="bottom"
                                        shapes="circles"
                                        :swatches="swatches"               
                                    >
                                    </v-swatches>
                                </div>      
                            </div>
                            <div class="button-container">
                                <button class="dataset-button delete" @click="deleteActionButton(index,actions.buttonKey)" >
                                    <div class="icon-container">
                                        <cross-icon color="grey"></cross-icon>
                                    </div>
                                </button>
                            </div>
                        </div>
                        <div class="dataset row-container" v-for="action  in actions.actions" :key="action.actionKey">
                            <div class="row-single">
                                <div class="row-select">
                                    <label>Type:&nbsp;</label>
                                    <v-select placeholder="Select Type"
                                        v-model="action.type"
                                        @input="updateAction($event, actions.buttonKey, action.actionKey, index )"
                                        :options="options"
                                        :reduce="options => options.type"
                                        label="type">
                                    </v-select>
                                </div>
                                <div class="row-select" v-if="action.type == 'Link'">
                                    <label>Page:&nbsp;</label>
                                    <v-select placeholder="Select Key"
                                        v-model="action.link.page"
                                        :options="Pages"
                                        label="name"
                                        :reduce="Pages => Pages.slug">
                                    </v-select>
                                </div>
                                <div class="row-select" v-if="action.type == 'Link'">
                                    <label>Key:&nbsp;</label>
                                    <v-select placeholder="Select Column" 
                                        v-model="action.link.key"
                                        :options="getColumns()"
                                    ></v-select>
                                </div>
                                
                                <div class="row-name" v-if="action.type == 'External Link'">
                                    <label>To:&nbsp;</label>
                                    <input type="text" v-model="action.to" placeholder="Provide to"/>
                                </div>
                                
                                <div class="row-name" v-if="action.type == 'Link' || action.type == 'External Link'">
                                    <label>External tab:&nbsp;</label>
                                     <div class="toggle-container">
                                    <div class="toggle-switch">
                                        <div class="switch right"  @click="action.linkExternalTab =! action.linkExternalTab">
                                            <input type="checkbox" v-model="action.linkExternalTab">
                                            <span class="slider round"></span>
                                        </div>
                                    </div>
                                    </div>
                                </div>
                                <div class="button-container">
                                    <button class="dataset-button delete" @click="deleteAction(index,actions.buttonKey, action.actionKey)" >
                                        <div class="icon-container">
                                            <cross-icon color="grey"></cross-icon>
                                        </div>
                                    </button>
                                </div>   
                            </div>
                        </div>
                        <div class="button-container">
                            <button class="dataset-button" @click="addAction(index, actions.buttonKey)">
                                <div class="icon-container">
                                    <add-icon color="grey"></add-icon>
                                </div>
                            </button>
                        </div>
                    </div>
                    <div class="button-container">
                        <button class="dataset-button" @click="addButton(index)">
                            <div class="icon-container">
                                <add-icon color="grey"></add-icon>
                            </div>
                        </button>
                    </div>
                </div>
                <!-- <div class="applies-container">
                    <button class="delete-filter-on" @click="removeFiltersOn(filter.id, filter_on.id)">
                        <div class="icon-container">
                            <cross-icon color="grey"></cross-icon>
                        </div>
                    </button>
                </div> -->
            </div>
            <button class="add-button" @click="addDataset">
                <div class="icon-container">
                    <add-icon color="grey"></add-icon>
                </div>
            </button>
        </div>
    </div>
</template>

<script>
import API from '~/plugins/API';
import { mapGetters } from "vuex";
import VSwatches from 'vue-swatches';
import Draggable from 'vuedraggable';
import SwatchesMixin from '~/mixins/components/swatchColors';


export default {
    mixins:[SwatchesMixin],
    props:{
        widget:{
            type : Object,
            required : true,
        },
    },
    components: {
        Draggable,
        VSwatches
    },
    data: function(){
        return{
            MapFilterBar: {},
            MapboxStyles: [],
            options: [],
            publicBool: false,
            datasetIndex: 0,
            optionsTypes: ['Link','External Link'],
        }
    },
    computed:{
        ...mapGetters("app", ['getPages']),
        Pages: function(){
            return this.getPages
        },
        queries: function () {
            let queries = [];

            this.MapFilterBar.transformations.forEach(function (item) {
                queries.push({name: item.name, id: item.id});
            });

            return queries;
        },
        isValid() {
            let valid = false;
            let errors = 0;
            let sel = this.MapFilterBar.selection;
            let transformations = this.MapFilterBar.transformations;
            if(transformations.length == 0 || sel.datasets == 0){
                errors++;
            }
            if(sel.datasets){
                for(let i = 0; i < sel.datasets.length; i++){
                    if(sel.datasets[i].latitude && sel.datasets[i].latitude !== "" && 
                        sel.datasets[i].longitude !== "" && sel.datasets[i].longitude !== "" &&
                        sel.datasets[i].datasetID.name && sel.datasets[i].datasetID.name !== "" ){
                        if(sel.datasets[i].source == "Color"){
                            if(sel.datasets[i].color && sel.datasets[i].color !== ""){
                            } else { errors++; }
                        }else if(sel.datasets[i].source == "Picture"){
                             if((sel.datasets[i].picture.slug && sel.datasets[i].picture.slug !== "") ||
                                (sel.datasets[i].picture.url && sel.datasets[i].picture.url !== "")){
                            } else { errors++; }
                        } else if(sel.datasets[i].source == "Icon"){
                            if((sel.datasets[i].icon && sel.datasets[i].icon !== "")){
                            } else { errors++; }
                        } else { errors++; }
                    } else { errors++; }
                };
            }else{
                errors ++;
            }
            if(sel.name !== "" && sel.name){ 
            }else{
                errors ++;
            }
            if (errors == 0) {
                valid = true;
            }else{
                valid = true;

            }
            
            return true
        },
        checkColumns() {
            let columns = {};
            let urlCheck = new RegExp("([a-zA-Z0-9]+://)?([a-zA-Z0-9_]+:[a-zA-Z0-9_]+@)?([a-zA-Z0-9.-]+\\.[A-Za-z]{2,4})(:[0-9]+)?(/.*)?");
            for(let i = 0, l = this.MapFilterBar.temp_data.selectedData.length; i < l; i++){
                for (const [key, value] of Object.entries(this.MapFilterBar.temp_data.selectedData[i])) {
                    if (typeof value === "string"){
                        if(value.match(urlCheck)){
                            columns[key] = "url"
                        }else{
                            columns[key] = "string";
                        }
                    }else if(typeof value === "number"){
                        columns[key] = "number";
                    }
                }
            }
            return columns;         
        },
        optionsRelatedFilters(){
            let filters = this.MapFilterBar.selection.filters;
            
            let options = [];
            for(let i = 0; i < filters.length; ++i){
                let item = {"label": filters[i].name, "value": {"id":filters[i].id, "key": filters[i].name}};
                options.push(item);
            }
            
            return options;
        },
        getMapboxStyles(){
            let options = [];

            let styles = this.MapboxStyles;

            for(let i = 0; i< styles.length; ++i){
               let item = {"label": styles[i].name, "value": styles[i].id};  
               options.push(item);       
            }

            return options

        }
        // columns: function(){
        //     let columns = [];
        //     for (let i = 0; i < this.MapFilterBar.selection.columns.length; i++){
        //         let col = {"label": this.MapFilterBar.selection.columns[i].name, "value": this.selection.columns[i].key};
        //         columns.push(col)
        //     }
        //     return columns;
        // }
    },
    methods:{
        addFilter: function () {
            let id = Math.round(Math.random() * (100000 - 0) + 0);
            this.MapFilterBar.selection.filters.push({
                id: id,
                name: "",
                type: "",
                time: false,
                color:  "",
                filters_on: [
                    {
                        id: Math.round(Math.random() * (100000 - 0) + 0),
                        query: {},
                        key: "",
                    }
                ],
                related_filters: [],
                showCount: true
            });
        },
        removeFilter: function(filterID){
            for(let i=0; i < this.MapFilterBar.selection.filters.length; i++){
                if(this.MapFilterBar.selection.filters[i].id === filterID){
                    this.MapFilterBar.selection.filters.splice(i,1)
                }
            }
        },
        addFiltersOn: function (filterIndex) {
            this.MapFilterBar.selection.filters[filterIndex].filters_on.push({
                id: Math.round(Math.random() * (100000 - 0) + 0),
                query: {},
                key: "",
                keys: []
            });
        },
        removeFiltersOn: function(filterID, filter_onID){
            for(let i=0; i < this.MapFilterBar.selection.filters.length; i++){
                if(this.MapFilterBar.selection.filters[i].id === filterID){
                    for(let j=0; j < this.MapFilterBar.selection.filters[i].filters_on.length; j++){
                        if(this.MapFilterBar.selection.filters[i].filters_on[j].id === filter_onID){
                            this.MapFilterBar.selection.filters[i].filters_on.splice(j,1)
                        }
                    }
                }
            }
        },
        getQuery: function (queryName) {
            let query = false;

            for (let i = 0; i < this.MapFilterBar.transformations.length; i++) {
                if (this.MapFilterBar.transformations[i].name === queryName) {
                    query = this.MapFilterBar.transformations[i];
                    break;
                }
            }
            return query;
        },
        getKeys: async function (queryName) {
            let query = false;

            for (let i = 0; i < this.MapFilterBar.transformations.length; i++) {
                if (this.MapFilterBar.transformations[i].name === queryName) {
                    query = this.MapFilterBar.transformations[i];
                    break;
                }
            }
            if (query) {
                let offset = 0;
                let limit = 1;
                let sorting = "id";
                let order = "ASC";
                let search = "";

                let response = await API.Collection.Aggregate(this.$store.state.general.auth.token, query.app, query.collection, query.transformations, this.MapFilterBar.selection, this.MapFilterBar.type.type, offset, limit, sorting, order, search, this.itemId);

                if (response.data.data && response.data.data.length) {
                    this.$set(query, 'keys', Object.keys(response.data.data[0]));
                } else {
                    this.$set(query, 'keys', []);
                }
            } 
        },
        getKeysDataSet: async function(datasetID){
            let dataset = false;
            let oldDataSet = this.MapFilterBar.selection.datasets.find(item => item.datasetID.id === datasetID.id);

            for (let i = 0; i < this.MapFilterBar.transformations.length; i++) {
                if (this.MapFilterBar.transformations[i].name === datasetID.name) {
                    dataset = this.MapFilterBar.transformations[i];
                    break;
                }
            }

            if (dataset) {
                let offset = 0;
                let limit = 1;
                let sorting = "id";
                let order = "ASC";
                let search = "";

                let response = await API.Collection.Aggregate(this.$store.state.general.auth.token, dataset.app, dataset.collection, dataset.transformations, this.MapFilterBar.selection, this.MapFilterBar.type.type, offset, limit, sorting, order, search, this.itemId);

                if (response.data.data && response.data.data.length) {
                    // oldDataSet[keys] = Object.keys(response.data.data[0])
                    this.$set(oldDataSet, 'keys', Object.keys(response.data.data[0]));
                } else {

                    this.$set(oldDataSet, 'keys', []);
                }
            } 
        },
        addDataset: function(){
            this.MapFilterBar.selection.datasets.push({
                id: Math.round(Math.random() * (100000 - 0) + 0),
                datasetID: null,
                locationStart: "",
                mapStyle: "",
                locationStartDatasets: [],
                longitude: null,
                latitude: null,
                zoom: null,
                togglePopUp: false,
                enableActions: false,
                actionButtons:[],
                keys: [],
                icon: "",
                source: "",
                color: "",
                picture: {
                    enableUpload: false,
                    url: "",
                    uploaded:false,
                    timestamp: "",
                    full_name: "",
                    name: "",
                    slug: "",
                },
                svg: "",
                limit: 100,
                details:[],
            });
        },
        addButton: function(index) {
            this.MapFilterBar.selection.datasets[index].actionButtons.push({
                buttonKey: this.generateKey(),
                color: "",
                icon: "",
                actions: []
            });
        },
        addAction: function(index, buttonKey){
            for(let i=0; i < this.MapFilterBar.selection.datasets[index].actionButtons.length; i++){
                if(this.MapFilterBar.selection.datasets[index].actionButtons[i].buttonKey === buttonKey){
                    this.MapFilterBar.selection.datasets[index].actionButtons[i].actions.push({ 
                        actionKey: this.generateKey(),  
                        type: "",
                        to: "",
                        link: {
                            page: "",
                            key: ""
                        },
                        linkExternalTab: false,
                        extern: false
                    });
                }
            }
        },
        addPopUpDetails: function(index){
            this.MapFilterBar.selection.datasets[index].details.push({
                id: Math.floor(Math.random() * 1000000),
                name: "",
                key: "",
            })
        },
        updateAction: function(event, buttonKey, actionKey, index){
            let type = event;
            // check if old type exists in the action
            // let exists = this.selectionSettings.optionsTypes.includes(this.selectionSettings.action.type);

            // if(exists){
            // let oldType = this.selectionSettings.action.type;
            
            let actionSettings = this.MapFilterBar.selection.datasets[index];
            for(let i=0; i < actionSettings.actionButtons.length; i++){
                if (actionSettings.actionButtons[i].buttonKey === buttonKey){
                    for(let j=0; j < actionSettings.actionButtons[i].actions.length; j++){
                        if(actionSettings.actionButtons[i].actions[j].actionKey === actionKey){
                            if(actionSettings.actionButtons[i].actions[j].type === "External Link"){
                                actionSettings.actionButtons[i].actions[j].extern = true;
                                actionSettings.actionButtons[i].actions[j].linkExternalTab = false;
                                actionSettings.actionButtons[i].actions[j].to = "";

                            } else if (actionSettings.actionButtons[i].actions[j].type === "Link"){
                                actionSettings.actionButtons[i].actions[j].extern = false;
                                actionSettings.actionButtons[i].actions[j].linkExternalTab = false;
                                actionSettings.actionButtons[i].actions[j].to = "";
                            }
                        }
                    }
                }
            }
        },
        deleteDataset: function(index, id){
            this.MapFilterBar.selection.datasets.splice(index,1);     
        },
        deletePopUpDetails: function(index, details_index){
            this.MapFilterBar.selection.datasets[index].details.splice(details_index,1)
        },
        deleteActionButton: function(index, buttonKey){
            for(let i=0; i < this.MapFilterBar.selection.datasets[index].actionButtons.length; i++){
                if (this.MapFilterBar.selection.datasets[index].actionButtons[i].buttonKey === buttonKey){
                    this.MapFilterBar.selection.datasets[index].actionButtons.splice(i,1);    
                }
            }
        },
        deleteAction: function(index, buttonKey, actionKey){
            let actionSettings = this.MapFilterBar.selection.datasets[index];
            for(let i=0; i < actionSettings.actionButtons.length; i++){
                if (actionSettings.actionButtons[i].buttonKey === buttonKey){
                    for(let j=0; j <actionSettings.actionButtons[i].actions.length; j++){
                        if(actionSettings.actionButtons[i].actions[j].actionKey === actionKey){
                            actionSettings.actionButtons[i].actions.splice(j ,1)
                        }
                    }
                }
            }
        },
        getColumns: function(){
            let columns = [];
            for(const [key] of Object.entries(this.checkColumns)) {
                columns.push(key);
            }
            return columns;
        },
        getColumnsData: function(type){
            let columns = [];
            for(let i = 0; i < this.MapFilterBar.temp_data.selectedData.length; i++){
                for(const [key, value] of Object.entries(this.MapFilterBar.temp_data.selectedData[i])) {
                    if (typeof value === type) {
                        columns.push(key);
                    }
                }
                
            }
            columns = [...new Set(columns)];
            return columns;
        },
        dragstart: function(e) {
            this.img = document.createElement("img");
            this.img.style.display = "none"; /* or visibility: hidden, or any of the above */
            e.dataTransfer.setDragImage(this.img, 0,0);
        },
        dragend: function (e) {
            this.img.remove();
        },
        setPinPoint: function(id, event){
            for(let i = 0; i < this.MapFilterBar.selection.datasets.length; i++){
                if(this.MapFilterBar.selection.datasets[i].id === id){
                    if(event === "Color"){
                        this.MapFilterBar.selection.datasets[i].picture = {
                            url: "",
                            slug: "",
                        }
                    }else{
                        this.MapFilterBar.selection.datasets[i].color = ""; 
                    }
                    this.MapFilterBar.selection.datasets[i].source = event;
                }
            }
        },
        setIcon(icon, actionKey, identifier, dataset){

            if(identifier === 'actionButton'){
                for(let i=0; i< this.MapFilterBar.selection.datasets.length; i++){
                    if(this.MapFilterBar.selection.datasets[i].id === dataset){
                        for(let j =0; j < this.MapFilterBar.selection.datasets[i].actionButtons.length; j++){
                            if(this.MapFilterBar.selection.datasets[i].actionButtons[j].buttonKey === actionKey){
                                this.MapFilterBar.selection.datasets[i].actionButtons[j].icon = icon.component
                            }
                        }
                    }
                }
            }else if(identifier === 'pinpoint'){
                for(let i=0; i < this.MapFilterBar.selection.datasets.length; i++){
                    if(this.MapFilterBar.selection.datasets[i].id === actionKey){
                        this.MapFilterBar.selection.datasets[i].icon = icon.component
                    }
                }
            }



            
            // for(let i=0; i < this.MapFilterBar.selection.datasets[index].actionButtons.length; i++){
            //     if(this.MapFilterBar.selection.datasets[index].actionButtons[i].buttonKey === actionKey){
            //         this.MapFilterBar.selection.datasets[index].actionButtons[i].icon = icon.component
            //     }
            // }
            // for(let i=0; i < this.MapFilterBar.selection.datasets.length; i++){
            //     if(this.MapFilterBar.selection.datasets[i].id === actionKey){
            //         this.MapFilterBar.selection.datasets[i].icon = icon.component
            //     }
            // }
        },
        UpdateInformation: function(val){
            this.MapFilterBar.selection.information = val;
        },
        hideCount: function (index){
            this.MapFilterBar.selection.filters[index].showCount = !this.MapFilterBar.selection.filters[index].showCount;
        },
        targetUpload: function (value) {
            let fileUpload = document.getElementById('fileUpload')
            if (fileUpload != null) {
                fileUpload.click()
            }
            this.datasetIndex = value
            // this.$refs.input.click();
        },
        readFile: function (e) {
            let files = e.target.files || e.dataTransfer.files;
            if (!files.length)
                return;
            let file = files[0];
            let fileName = encodeURI(file.name);
            // let fileType = fileName.split('.').pop();

            let reader = new FileReader();


            reader.onload = this.uploadFile(fileName, reader);
            reader.readAsDataURL(file);
        },
        uploadFile: async function (fileName, reader) {
            let folderPath = ["file_upload", "pinpoints"]
            let response = await API.File.ReadOne(this.$store.state.general.auth.token, folderPath, fileName);
        
            // get the content after the mime type
            let content = reader.result.split(',')[1];
            this.publicBool = true;

            let uploaded = await API.File.CreateOne(this.$store.state.general.auth.token, folderPath, fileName, content, this.publicBool);
            if (uploaded.status == 200) {
                
                this.MapFilterBar.selection.datasets[this.datasetIndex].picture.uploaded = true;
                this.MapFilterBar.selection.datasets[this.datasetIndex].picture.timestamp = Math.floor(Date.now() / 1000);
                this.MapFilterBar.selection.datasets[this.datasetIndex].picture.full_name = uploaded.data.data.file_name;
                this.MapFilterBar.selection.datasets[this.datasetIndex].picture.name = fileName;
            }
                
        },
    },
    watch:{
        widget: {
            immediate: true,
            handler: function(val){
                if(val !== ""){
                    let mapbox = val;
                    for(let i = 0; i < mapbox.selection.datasets.length; i++){
                        if(mapbox.selection.datasets[i].picture.hasOwnProperty("enableUpload") === false){
                            mapbox.selection.datasets[i].picture.enableUpload = false;
                        } 
                        if(mapbox.selection.datasets[i].picture.hasOwnProperty("uploaded") === false){
                            mapbox.selection.datasets[i].picture.uploaded = false;
                        } 
                        if(mapbox.selection.datasets[i].picture.hasOwnProperty("full_name") === false){
                            mapbox.selection.datasets[i].picture.full_name = "";
                        } 
                        if(mapbox.selection.datasets[i].picture.hasOwnProperty("name") === false){
                            mapbox.selection.datasets[i].picture.name = "";
                        } 
                        if(mapbox.selection.datasets[i].picture.hasOwnProperty("timestamp") === false){
                            mapbox.selection.datasets[i].picture.timestamp = "";
                        } 
                    
                        for(let j = 0; j < mapbox.selection.datasets[i].actionButtons.length; j++){
                            for(let k = 0; k < mapbox.selection.datasets[i].actionButtons[j].actions.length; k++){
                                if(mapbox.selection.datasets[i].actionButtons[j].actions[k].hasOwnProperty("link") === false){         
                                    mapbox.selection.datasets[i].actionButtons[j].actions[k].link = {};
                                    mapbox.selection.datasets[i].actionButtons[j].actions[k].link.page = "";
                                    mapbox.selection.datasets[i].actionButtons[j].actions[k].link.key = "";
                                }
                            }
                        }
                    }
                    this.MapFilterBar = mapbox;
                }
            }
        },
        "MapFilterBar.selection": {
            handler: function(val){
                this.$emit("updateSelection", val, this.MapFilterBar.key, this.isValid);
            },
            deep: true
        },
        isValid: {
            immediate: true,
            handler: function(val) {
                let widget = JSON.parse(JSON.stringify({...this.widget}))
                widget.valid = val;

                this.$emit("updateWidget", widget);
            }
        },
        
    },
    async created(){
        if(this.MapFilterBar.selection.hasOwnProperty("enableActions") === false){
            this.MapFilterBar.selection.enableActions = false;
        } 
        if(this.MapFilterBar.selection.hasOwnProperty("locationStart") === false){
            this.MapFilterBar.selection.locationStart = "";
        } 
        if(this.MapFilterBar.selection.hasOwnProperty("locationStartDatasets") === false){
            this.MapFilterBar.selection.locationStartDatasets = [];
        } 
       
        for(let i = 0; i < this.MapFilterBar.selection.datasets.length; i++){
            if(this.MapFilterBar.selection.datasets[i].hasOwnProperty("icon") === false){
                this.MapFilterBar.selection.datasets[i].icon = "";
            } 
            if(this.MapFilterBar.selection.datasets[i].picture.hasOwnProperty("enableUpload") === false){
                this.MapFilterBar.selection.datasets[i].picture.enableUpload = false;
            } 
            if(this.MapFilterBar.selection.datasets[i].picture.hasOwnProperty("uploaded") === false){
                this.MapFilterBar.selection.datasets[i].picture.uploaded = false;
            } 
            if(this.MapFilterBar.selection.datasets[i].picture.hasOwnProperty("full_name") === false){
                this.MapFilterBar.selection.datasets[i].picture.full_name = "";
            } 
            if(this.MapFilterBar.selection.datasets[i].picture.hasOwnProperty("name") === false){
                this.MapFilterBar.selection.datasets[i].picture.name = "";
            } 
            if(this.MapFilterBar.selection.datasets[i].picture.hasOwnProperty("timestamp") === false){
                this.MapFilterBar.selection.datasets[i].picture.timestamp = "";
            } 
           
            for(let j = 0; j < this.MapFilterBar.selection.datasets[i].actionButtons.length; j++){
                for(let k = 0; k < this.MapFilterBar.selection.datasets[i].actionButtons[j].actions.length; k++){
                    if(this.MapFilterBar.selection.datasets[i].actionButtons[j].actions[k].hasOwnProperty("link") === false){         
                        this.MapFilterBar.selection.datasets[i].actionButtons[j].actions[k].link = {};
                        this.MapFilterBar.selection.datasets[i].actionButtons[j].actions[k].link.page = "";
                        this.MapFilterBar.selection.datasets[i].actionButtons[j].actions[k].link.key = "";
                    }
                }
            }
        }

        for(let i = 0; i < this.MapFilterBar.selection.filters.length; ++i){
            if(this.MapFilterBar.selection.filters[i].hasOwnProperty("related_filters") === false){
                this.MapFilterBar.selection.filters[i].related_filters = []
            }

            if(this.MapFilterBar.selection.filters[i].hasOwnProperty("showCount") === false){
                this.MapFilterBar.selection.filters[i].showCount = true;
            }
        }

        let optionsTypes = this.optionsTypes;
        let options = [];
        let option;

        for(let i=0; i < optionsTypes.length; i++){
            option = {
                    type: optionsTypes[i],
                    enabled: true
                }
            options.push(option);
        }

        let response = await API.Mapbox.ReadAllStyles(this.$store.state.general.auth.token);
        if(response.data.code === 200){
            this.MapboxStyles = response.data.data
        }
        

        this.options = options;
    }
};
</script>

<style lang="scss" scoped>
.select-filters {
    //padding: 15px 25px;
    flex:1;
    margin-right: 20px;

    .map-configuration{
        .sub-title{
            font-family: "Galyon Bold";
            color: $color-black;
            font-size: 16px;
            padding: 0 0 10px 0;
        }

        .general-settings{
            .picker{
            position: absolute !important;
            border-radius: 5px;
            border-color: $color-grey-light;
            border-style: solid;
            border-width: thin;
            }
            label {
            @include general-text($font-size: $label-size-medium);
            }
            input {
                @include general-text($text-size-medium);
                width: 100%;
                border-radius: 5px;
                outline: 0;
                border: none;
                margin-top: 7px;
                margin-bottom: 7px;
                height: 34px;
                text-indent: 16px;
                background-color: $color-white-dark;

                &.vs__search{
                height: 19px;
                border: 1px solid transparent;
                border-left: none;
                outline: none;
                margin: 4px 0 0 0;
                background: none;
                width: 0;            
                }
            }
            input::placeholder{
                text-indent: 16px;  
                color: $color-grey; 
                font-size: 14px;
                vertical-align: middle;
            }
        }

        .add-button {
            @include round-container($padding: 7px, $width: auto, $height: auto);
            @include icon-background-color($background: $color-white-dark, $hover: $color-grey-light, $active: $color-grey-twilight);
            @include icon-size($width: 16px, $height: 16px);
        }

        .dataset{
            display:flex;
            border: thin solid $color-grey-light;
            border-radius: 6px;
            padding: 10px;
            margin-bottom: 12px;
            position: relative;
            justify-content: space-between;
            background-color: $color-white;
            overflow: none;
            max-height: 100%;
            //align-items: center;
            flex-direction: column;
            label {
                @include general-text($font-size: $label-size-small);
            }
            .map-selection{
                display: flex;
                flex: 1;
                padding-right: 16px;
                padding-bottom: 10px;
            }
            .map-pinpoint{
                display: flex;
                flex: 1;
                
            }
            .toggle-container{
                justify-content: flex-start;
                label {
                    display: flex;
                    align-items: center;
                }
            }
            .map-pipoint-details{
                display:flex;
                border: thin solid $color-grey-light;
                border-radius: 6px;
                padding: 10px;
                margin-bottom: 12px;
                position: relative;
                justify-content: space-between;
                background-color: $color-white;
                overflow: none;
                max-height: 100%;                
            }
            .v-select{
                width: 100%;
                min-width: 0;
                .vs__dropdown-toggle{
                    .vs__selected-options{
                    overflow: hidden;
                    flex-wrap: nowrap;
                    .vs__selected{
                        overflow: hidden;
                        text-overflow: ellipsis; 
                        white-space: nowrap;
                        //content: "";
                        display: inline-block;
                        width: 100%;
                    }
                    }
                }
                .vs__dropdown-menu{
                    min-width: 100%;
                    width: fit-content;
                    width: -moz-fit-content;
                    overflow-x: hidden;
                }
            }
            .selection-drag{
                position: absolute;
                display: flex;
                width: 25px;
                height: 25px;
                border: none;
                left: 0px;
                outline: 0;
                background-color: transparent;
                border-radius: 100%;
                transition: all 0.3s ease-in-out;
                .icon-container {
                    width: 25px;
                    height: 25px;
                }
            }
            .selection{
                display: flex;
                justify-content: space-between;
                align-items: center;
                width: 30%;
                padding-left:5px;       
                padding-right: 5px;
                &.left{
                margin-left:10px;
                }
            }
            .selection-toggle{
                display: flex;
                justify-content: space-between;
                align-items: center;
                width: 10%;
                padding-left:5px;       
                padding-right: 5px;
                &.left{
                margin-left:10px;
                }
            }
            .selection-name{
                display: flex;
                justify-content: space-between;
                align-items: center;
                width: 30%;
                min-width:170px;
                padding-left:5px;       
                padding-right: 5px;

                &.upload {
                    width: 10%;
                }
                span{
                    font-family: "Galyon Regular";
                    font-size: 14px;
                    color: $color-black-light;
                }
                
                &.left{
                    margin-left:10px;
                }

                input {
                    @include general-text($text-size-medium);
                    width: 100%;
                    border-radius: 5px;
                    outline: 0;
                    border-color: $color-grey-light;
                    border-style: solid;
                    border-width: thin;
                    height: 32px;
                    text-indent: 16px;
                }
                input::placeholder{
                    text-indent: 16px;  
                    color: $color-grey; 
                    font-size: 14px;
                    vertical-align: middle;
                }

                .input-icon {
                    width: 18px;
                }
                .icon-upload{
                    border-radius: 5px;
                    .icon-container{
                        flex: 0 0 18px;
                        margin-right: 12px;
                        display: flex;
                        align-items: center;

                        &:hover {
                            background-color: $color-grey-light;
                        }
                    }
                }
            }
            .color{
                width: 80px;
                display: flex;
                justify-content: space-between;
                align-items: center;
                margin-right: 18px;
                padding-left: 5px;
                .custom-color-picker{
                    .vue-swatches{
                    display: flex;
                    align-items: center;
                    .vue-swatches__fallback__input{
                        padding: 0;
                    }
                    .vue-swatches__container{
                        margin-bottom: 50px !important;
                    }
                    .vue-swatches__trigger{
                        width: 26px !important;
                        height: 26px !important;
                    }
                    .vue-swatches__fallback__button{
                        @include general-text($text-size-small, false, $color-secondary-dark);
                        font-weight: normal;
                    }

                    }
                }
            }
            .button-container{
                display:flex;
                justify-content: center;
            }
        
            .dataset-button{
            @include round-container($padding: 8px, $width: 30px, $height: 30px);
            @include icon-background-color($background: $color-grey-light, $hover: $color-white-dark, $active: transparent);
            @include icon-size($width: 15px, $height: 15px);

            display: flex;
            margin-bottom:10px;
            &.delete{
                background-color: $color-white;
                margin: 0;
                position: absolute;
                right: 0px;
                top: 0px;
                
                &:hover {
                    background-color: $color-white-dark;
                }
            }
            }
        }
    }
    
    .filter-selection {
        background-color: $color-white-dark;
        padding: 15px 25px;
        margin-bottom: 10px;
        margin-top: 10px;
        border-radius: 6px;

        .filter-header{
            display: flex;

            .filter-container{
                display: flex;
                align-items: center;
                justify-content: center;
                
                .show-filter{
                    @include round-container($padding: 7px, $width: auto, $height: auto);
                    @include icon-size($width: 16px, $height: 16px);
                    @include icon-background-color($background: transparent, $hover: $color-grey-light, $active: $color-grey-light);
                }
                &:last-child{
                    margin-left: auto;

                    .filter-drag {
                        //position: relative;
                        //display: flex;
                        margin-right: 7px;
                        @include round-container($padding: 7px, $width: auto, $height: auto);
                        @include icon-size($width: 18px, $height: 18px);
                        @include icon-background-color($background: transparent, $hover: $color-grey-light, $active: $color-grey-light);
                    }

                    .delete-filter{
                        @include round-container($padding: 7px, $width: auto, $height: auto);
                        @include icon-size($width: 16px, $height: 16px);
                        @include icon-background-color($background: transparent, $hover: $color-grey-light, $active: $color-grey-light);
                    }
                }

                .name {
                    font-family: "Galyon Regular";
                    margin-right: 5px;
                    font-size: 14px;
                }

                input {
                    font-family: "Galyon Regular";
                    outline: 0;
                    border: none;
                    background-color: $color-grey-light;
                    color: $color-black-light;
                    padding: 7px 15px;
                    border-radius: 3px;
                }

                &.type{
                    .type {
                        font-family: "Galyon Regular";
                        margin-right: 5px;
                        margin-left: 15px;
                        font-size: 14px;
                    }

                    .type-select {
                        min-width: 200px;
                        display: inline-block;

                        .v-select {
                            background-color: $color-grey-light;
                        }
                    }
                }
            }
        }

        h4 {
            font-family: "Galyon Bold";
            font-size: 12px;
            margin-bottom: 5px;
            margin-top: 8px;
        }

        .applies-to {
            background-color: $color-grey-light;
            margin-bottom: 8px;
            padding: 8px 10px;
            border-radius: 6px;
            display: flex;
            justify-content: space-between;

            label {
                margin-right: 5px;
            }

            .applies-container{
                display: flex;

                .applies-property{
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    margin-right: 20px;

                    label{
                        font-family: "Galyon Regular";
                        font-size: 14px;
                        margin-right: 12px;
                    }

                    .query-select{
                        min-width: 300px;
                    }
                }

                .delete-filter-on{
                    @include round-container($padding: 7px, $width: auto, $height: auto);
                    @include icon-size($width: 16px, $height: 16px);
                    @include icon-background-color($background: transparent, $hover: $color-white-dark, $active: $color-white);
                }
            }
        }
    }

    .add-button {
        @include round-container($padding: 7px, $width: auto, $height: auto);
        @include icon-background-color($background: $color-white-dark, $hover: $color-grey-light, $active: $color-grey-twilight);
        @include icon-size($width: 16px, $height: 16px);
    }
}
.filter-queries {
    padding: 0 25px;

    .filter-query-container{
        display: flex;

        .queries{
            flex: auto;
        }

        .filter-bar-settings{
            flex: 0 0 400px;
        }
    }

    .single-query {
        margin: 10px 0;
        padding: 10px 15px;
        background: $color-grey-light;
        border-radius: 3px;
        display: flex;
        max-width: 265px;

        input {
            font-family: "Galyon Regular";
            font-size: 14px;
            color: $color-black-light;
            border: none;
            background: none;
            outline: 0;
        }

        button {
            @include round-container($padding: 7px, $width: 21px, $height: 21px);
            @include icon-background-color($background: $color-white-dark, $hover: $color-white, $active: $color-grey-light);
            @include icon-size($width: 16px, $height: 16px);
        }
    }
}
  


.filter-container {
    .toggle-switch{
        // width: 50%;
        height: 35px;
        position: relative;
        .switch {
            position: absolute;
            right: -70px; //0px still leaves a bit of room.
            width: 80px;
            height: 34px;
            transform: scaleY(0.5) scaleX(0.5);
        
            input {
                opacity: 0;
                width: 0;
                height: 0;
            }

            /* The slider */
            .slider {
                position: absolute;
            cursor: pointer;
                top: 0;
                left: 0;
                right: 0;
                bottom: 0;
                background-color: $color-grey-light;
                -webkit-transition: .4s;
                transition: .4s;
            }

            .slider:before {
                position: absolute;
                content: "";
                height: 26px;
                width: 26px;
                left: 4px;
                bottom: 4px;
                background-color: $color-white-light;
                -webkit-transition: .4s;
                transition: .4s;
            }

            input:checked + .slider {
                background-color: $color-primary;
            }

            input:focus + .slider {
                box-shadow: 0 0 1px $color-primary;
            }

            input:checked + .slider:before {
                -webkit-transform: translateX(26px);
                -ms-transform: translateX(26px);
                transform: translateX(46px);
            }

            /* Rounded sliders */
            .slider.round {
                border-radius: 34px;
            }

            .slider.round:before {
                border-radius: 50%;
            }
        }
    }
    .vue-swatches{
        display: flex;
        align-items: center;
        .vue-swatches__fallback__input{
            padding: 0;
        }
        .vue-swatches__container{
            margin-bottom: 50px !important;
        }
        .vue-swatches__trigger{
            width: 22px !important;
            height: 22px !important;
        }
        .vue-swatches__fallback__button{
          @include general-text($text-size-small, false, $color-secondary-dark);
          font-weight: normal;
        }
    }
}
.row-container{
    display: flex;
    position: relative;
    justify-content: flex-start;
    width: 100%;
    border: thin solid $color-grey-light;
    border-radius: 5px;
    padding: 10px;
    margin-top: 10px;
    background-color: $color-white;
    flex-direction: column;
    
    &.addButton {
        max-width: 1000px;
    }
    .dataset {
        flex-direction: column;
        width: 100%;
    }
    .row-single {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: flex-start;
        width: 100%;

        &.insert {
            margin-top: 10px;
            align-items: flex-start;
        }
        &.addButton {
            flex-direction: column;
            align-items: flex-start;
            padding-left: 5px 5px 5px 10px;
        }
        .data-item {
            width: 25%;
            &.questionnaire {
                width: 50%;
            }
        }
        .item-settings {
            width: 100%;
            padding-left: 20px;
            display: flex;
            justify-content: flex-start;
            flex-direction: column;
            &.columns {
                justify-content: flex-end;
            }
        }
        .row-drag{
            position: relative;
            display: flex;
            width: 25px;
            height: 25px;
            border: none;
            left: 0px;
            outline: 0;
            background-color: transparent;
            border-radius: 100%;
            transition: all 0.3s ease-in-out;

            .icon-container {
                width: 20px;
                height: 20px;
            }
        }
        .row-name{
            display:flex;
            //flex: 0 0 40%;
            width: 18%;
            margin-left: 10px;
            justify-content: space-between;
            align-items: center;
            padding-left:5px;       
            padding-right: 5px;
            input {
                width: 100%;
                border-radius: 5px;
                outline: 0;
                border-color: $color-grey-light;
                border-style: solid;
                border-width: thin;
                height: 34px;
                text-indent: 16px;
                font-size: 14px;
                font-family: "Galyon Regular";
                color: $color-black-light;
            }
            input::placeholder{
                text-indent: 16px;  
                color: $color-grey; 
                font-size: 14px;
                vertical-align: middle;
            }
            &.action {
                width: 10%;
            }
            &.addButton {
                width: 100%;
                margin-left: 0;
                
                label {
                    width: 100%;
                    }

            }
            &.conditions {
                width: 85%;
            }
            &.columns {
                width: 100%;
                flex-direction: column;
                align-items: flex-start;
            }
        }
        .row-json {
            display:flex;
            //flex: 0 0 40%;
            width: 25%;
            margin-left: 20px;
            justify-content: space-between;
            align-items: flex-start;
            padding:5px;
        }
        .row-item {
            display:flex;
            //flex: 0 0 40%;
            width: 12%;
            margin-right: 25px;
            justify-content: space-around;
            align-items: center;
            padding-left:5px;       
            padding-right: 5px;
            animation-duration: 0.8s;
            margin-left: 10px;

            
            
        }
        .row-select{
            display:flex;
            //flex: 0 0 40%;
            width: 18%;
            margin-right: 25px;
            justify-content: space-between;
            align-items: center;
            padding-left:5px;       
            padding-right: 5px;
            animation-duration: 0.8s;

            &.action {
                width: 100%;
                padding: 5px;

                label {
                    width: 40%;
                }
            }

                &.addButton {
                width: 100%;
                padding: 5px;
                
                label {
                    width: 100%;
                    }
                .v-select {
                    width: 100%
                    }

            }

        }
        .row-button-delete{
            display: flex;
            position: absolute;
            right: 8px;
            width: 26px;
            height: 26px;
            margin: 0;
            padding: 6px;
            border: none;
            outline: 0;
            background-color: transparent;
            border-radius: 100%;
            .icon-container {
                width: 20px;
                height: 20px;
            }
            &:hover {
                background-color: $color-white-dark;
            }
        }
        .button-collapse {
            position: relative;
            display: flex;
            padding: 5px;
            border: none;
            left: 0px;
            outline: 0;
            background-color: transparent;
            border-radius: 100%;
            transition: all 0.3s ease-in-out;

            .icon-container {
                width: 15px;
                height: 15px;
            }
        }
    }
}



.input-group-number {
  width: 100%;
  display: flex;
  align-items: center;
  padding: 0px 0px 0px 15px;
  border: 0;
  border-radius: 8px;
  background-color: $color-white-dark;

  .icon-container{
    flex: 0 0 18px;
    display: flex;
    align-items: center;

    &.valid{
      padding: 0px 8px;
      flex: 0 0 30px;
    }
  }

  .input-icon {
    width: 18px;
  }

  input {
    font-size: 14px;
    padding: 8px 0px;
    margin-left: 12px;
    font-family: "Galyon Regular";
    border: none;
    outline: 0;
    background: transparent;
    flex: 1;


    &:focus {
      background-color: $color-white-dark;
      border: 0;
      outline: none;
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


.button-container{
  display: flex;
  align-items: center;
  flex: 0 0 40px;

  .crease{
    border: 0;
    outline: none;
    background-color: transparent;
    margin-right: 5px;
  }
}


.add-icon {
  width: 14px;
}
</style>
