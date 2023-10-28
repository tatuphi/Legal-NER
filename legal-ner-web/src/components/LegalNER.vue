<template>
    <v-container fluid>
        <v-row no-gutters 
        style="border: 1px solid green; border-radius: 5px" 
        class="pa-3">
            <v-col v-for="(value, name, index) in colors" :key="index+'_'+name" 
            class="mr-3"
            style="flex-grow: 0">
                <div>
                <span class="text-caption font-weight-light">{{name}} </span>
                <v-text-field 
                v-model="colors[name]" 
                v-mask="mask" 
                hide-details
                solo 
                outlined 
                dense 
                class="ma-0 pa-0"  
                style="width: 150px;" 
                >
                    <template v-slot:append>
                        <v-menu v-model="listMenu[name]" top nudge-bottom="105" nudge-left="16" :close-on-content-click="false">
                            <template v-slot:activator="{ on }">
                                <div :style="swatchStyle(name)" v-on="on" />
                            </template>
                            <v-card>
                                <v-card-text class="pa-0">
                                    <v-color-picker v-model="colors[name]" flat />
                                </v-card-text>
                            </v-card>
                        </v-menu>
                    </template>
                </v-text-field>
                </div>
            </v-col>
        </v-row>
        <v-row no-gutters class="my-2">
            <v-text-field
            dense
            v-model="curURL"
            outlined
            hide-details
            placeholder="Input file url..."/>
            <v-spacer></v-spacer>
            <v-btn class="mt-3" small color="primary" 
            :disabled="curURL=='' || isExtracting"
            @click="nerByUrl">
            <v-progress-circular
            v-if="curURL!='' && isExtracting"
            :size="12"
            :width="1"
            indeterminate
            class="mr-1"
            ></v-progress-circular>
            URL</v-btn>
            <v-btn 
            class="ml-2 mt-3"
            small color="primary"
            :disabled="curContent=='' || isExtracting" 
            @click="extractNER">
            <v-progress-circular
            v-if="curContent!='' && isExtracting"
            :size="12"
            :width="1"
            indeterminate
            class="mr-1"
            ></v-progress-circular>
            Extract</v-btn>
        </v-row>
        <v-textarea
          name="input-7-1"
          outlined
          placeholder="Input your content..."
          rows="12"
          v-model="curContent"
          hide-details
        ></v-textarea>
        
        <div v-if="resData.content" class="mt-5">
        <v-row class="ma-0 pa-0 mb-1">
            <v-btn @click="dialogInfo = true" small outlined class="mr-2">Entities</v-btn>
            <v-btn @click="onDownload" small outlined>Download</v-btn>
        </v-row>
        <div v-html="resData.content" 
        style="border: 1px solid green; height: 580px; overflow-y:scroll; border-radius: 5px" 
        class="pa-2"></div>
        <v-dialog
            v-model="dialogInfo"
            width="auto"
        >
            <v-card min-height="500" min-width="500">
            <v-card-actions>
                <v-row class="ma-0 pa-0 mt-3 mx-2">
                    <v-select
                    dense
                    outlined
                    hide-details
                    label="Select type"
                    v-model="selectType"
                    :items="entityItems"
                    style="width: 200px"
                    ></v-select>
                    <v-spacer></v-spacer>
                    <v-icon @click="dialogInfo = false">mdi-close</v-icon>
                </v-row>
            </v-card-actions>
            <v-card-text >
                <v-data-table
                    :headers="headers"
                    :items="freqNER[selectType.toUpperCase()]"
                >
                </v-data-table>
            </v-card-text>
            
            </v-card>
        </v-dialog>
        </div>
    </v-container>
    

</template>
<script>
import axios from "axios";

export default ({
    data() {
        return {
            selectType: "PROVISIONS",
            dialogInfo: false,
            resData:{
                content: null,
                data: null,
            },
            entityItems:[
                'PETITIONER',
                'RESPONDENT',
                'JUDGE',
                'WITNESS',
                'LAWYER',
                'OTHER_PERSON',
                'PROVISION',
                'STATUTE',
                'GPE',
                'ORG',
                'COURT',
                'CASE_NUMBER',
                'DATE',
                'PRECEDENT',
            ],
            freqNER:{
                'PETITIONER': [],
                'RESPONDENT':[],
                'JUDGE': [],
                'WITNESS':[],
                'LAWYER':[],
                'OTHER_PERSON': [],
                'PROVISION': [],
                'STATUTE':[],
                'GPE':[],
                'ORG':[],
                'COURT': [],
                'CASE_NUMBER': [],
                'DATE':[],
                'PRECEDENT':[],
            },
            mask: '!#XXXXXX',
            isExtracting: false,
            curContent: "",
            curURL: "",
            colors: {"PETITIONER": "#FFEB3B", "RESPONDENT": "#4CAF50", "JUDGE": "#E91E63", "WITNESS": "#9C27B0", "LAWYER": "#F44336",
            "OTHER_PERSON": "#00BCD4",
            "PETITIONER_match": "#FFEB3B", "RESPONDENT_match": "#4CAF50", "JUDGE_match": "#E91E63",
            "WITNESS_match": "#9C27B0", "LAWYER_match": "#F44336",
            "PROVISION": "#33E9FF", "STATUTE": "#1C4D53", "GPE": "#A6A82F", "ORG": "#603255", "COURT": "#56A065",
            "DATE": "#804538", "CASE_NUMBER": "#71326E"},
            listMenu:{"PETITIONER": false, "RESPONDENT": false, "JUDGE": false, "WITNESS": false, "LAWYER": false,
            "OTHER_PERSON": false,
            "PETITIONER_match": false, "RESPONDENT_match": false, "JUDGE_match": false,
            "WITNESS_match": false, "LAWYER_match": false,
            "PROVISION": false, "STATUTE": false, "GPE": false, "ORG": false, "COURT": false,
            "DATE": false, "CASE_NUMBER": false},
             headers: [
                {
                text: 'Name',
                align: 'left',
                value: 'name',
                },
                { text: 'Count', value: 'count', align: 'right' },
            ],
        }
    },
    computed: {
        swatchStyle: function () {
            return function (name) {
                return {
                    backgroundColor: this.colors[name],
                    cursor: 'pointer',
                    height: '30px',
                    width: '30px',
                    borderRadius: this.listMenu[name] ? '50%' : '4px',
                    transition: 'border-radius 200ms ease-in-out'
                }
            };
        }
    },
    methods:{
        extractNER(){
            this.resetData();
            this.isExtracting = true;
            const formData = new FormData();
            formData.append('judgement', this.curContent);
            formData.append('colors', JSON.stringify(this.colors));

            axios.post("http://83ab-35-188-53-64.ngrok.io/ner", formData)
            .then((response)=>{
                if(response.status==200){
                    this.resData = response.data;
                    this.parseData();
                    this.handleData(this.resData.data);
                }
            })
            .catch(error => {
            console.error("There was an error!", error);
            })
            .finally(()=>{
                this.isExtracting = false;
            });
        },
        nerByUrl(){
            this.resetData();
            this.isExtracting = true;
            const formData = new FormData();
            formData.append('url', this.curURL);
            formData.append('colors', JSON.stringify(this.colors));

            axios.post("http://83ab-35-188-53-64.ngrok.io/ner/url", formData)
            .then((response)=>{
                if(response.status==200){
                    this.resData = response.data;
                    this.parseData();
                    this.handleData(this.resData.data);
                }
            })
            .catch(error => {
            console.error("There was an error!", error);
            })
            .finally(()=>{
                this.isExtracting = false;
            });
        },
        downloadJSON(content, fileName, contentType) {
            const a = document.createElement("a");
            const file = new Blob([content], { type: contentType });
            a.href = URL.createObjectURL(file);
            a.download = fileName;
            a.click();
		},
		onDownload(){
			this.downloadJSON(JSON.stringify(this.resData.data), new Date().toLocaleString('en-US') + ".json", "text/plain");
		},
        parseData(){
            let obj1 = this.resData.statutes;
            let arrObj1 = [];
            for( let key1 in obj1){
                arrObj1.push({
                    'name': key1,
                    'count': obj1[key1]
                })
            }
            this.resData.statutes = arrObj1;

            let obj2 = this.resData.provisions;
            let arrObj2 = [];
            for( let key2 in obj2){
                arrObj2.push({
                    'name': key2,
                    'count': obj2[key2]
                })
            }
            this.resData.provisions = arrObj2
        },
        handleData(dataObj){
            let fArr = {
                'PETITIONER': [],
                'RESPONDENT':[],
                'JUDGE': [],
                'WITNESS':[],
                'LAWYER':[],
                'OTHER_PERSON': [],
                'PROVISION': [],
                'STATUTE':[],
                'GPE':[],
                'ORG':[],
                'COURT': [],
                'CASE_NUMBER': [],
                'DATE':[],
                'PRECEDENT':[],
            };
            let origin_arr = dataObj.annotations[0].result;
           
            for(var i=0; i<origin_arr.length; i++){
                let iOBJ = origin_arr[i].value;
                let strLabel = iOBJ.labels[0];
                let idx = -1;
                if(fArr[strLabel].length>0){

                idx = fArr[strLabel].findIndex(object => {
                return object.name == iOBJ.text;
                });

                }
                if(idx!==-1){
                    fArr[strLabel][idx].count +=1;
                } else {
                    fArr[strLabel].push({
                        name: iOBJ.text,
                        count: 1
                    });
                }
            }
            this.freqNER = fArr;
        },
        resetData(){
            this.resData = {
                content: null,
                data: null,
                precedents: null,
                provisions: null,
                statutes: null,
            };
        },
    }
})
</script>
