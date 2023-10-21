<template>
    <v-container fluid>
        <v-row no-gutters 
        style="border: 1px solid green; border-radius: 5px" 
        class="pa-3 mb-3">
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
        <v-textarea
          name="input-7-1"
          outlined
          placeholder="Input your content..."
          rows="12"
          v-model="curContent"
          hide-details
        ></v-textarea>
        <v-row class="ma-0 pa-0 mt-3">
            <v-spacer></v-spacer>
            <v-btn :disabled="curContent=='' || isExtracting" color="primary" @click="extractNER">Extract</v-btn>
        </v-row>

        <div v-if="!isExtracting && resContent!=''" class="mt-5">Result here</div>
        <div v-html="resContent"></div>
    </v-container>
    

</template>
<script>
import axios from "axios"
export default ({
    data() {
        return {
            mask: '!#XXXXXX',
            isExtracting: false,
            curContent: "",
            resContent: "",
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
            this.isExtracting = true;
            const formData = new FormData();
            formData.append('judgement', this.curContent);
            formData.append('colors', JSON.stringify(this.colors));
            console.log(this.colors);

            axios.post("http://655a-34-90-88-14.ngrok.io/ner", formData)
            .then((response)=>{
                if(response.status==200){
                    console.log("here ")
                    this.resContent = response.data.content;
                }
            })
            .catch(error => {
            console.error("There was an error!", error);
            })
            .finally(()=>{
                this.isExtracting = false;
            });
        },
        // swatchStyle(name) {
        // return {
        //     backgroundColor: this.colors[name],
        //     cursor: 'pointer',
        //     height: '30px',
        //     width: '30px',
        //     borderRadius: this.listMenu[name] ? '50%' : '4px',
        //     transition: 'border-radius 200ms ease-in-out'
        // }
        // }
    }
})
</script>

<style scoped>
.cus-padding{
    width: fit-content
}
</style>
