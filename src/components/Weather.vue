<template>
<div class="container">
    
    <div class="content">

        <v-btn
        v-if="!show"
            class="btn1"
            color="primary"
            elevation="2"
            @click="openKeyInBox()"    
            >
           Key In
        </v-btn>

        <v-btn
        v-if="!show"
            color="primary"
            elevation="2"
            @click="openSelectBox()"    
            >
           Select
        </v-btn>

        <v-btn
        v-if="show"
            color="red"
            elevation="2"
            class="btn3"
            @click="closeBox()"    
            >
           Cancel
        </v-btn>

        <v-btn
        v-if="show"
            color="green"
            elevation="2"
            @click="getAPI()"    
            >
           Query
        </v-btn>

        
       <div class="input-city">
        <v-text-field 
            v-if="inputingKey"
            ref="weatherInput"
            color="teal lighten-2"
            shaped
            outlined
            v-model="inputText"
            class="custom-label-color"
            label="请输入城市名称，仅支持单关键词查询"
            placeholder=如“广东省”、“广州市”、“天河区”等单词，不支持输入“广东省广州市”等
            append-icon="X"
            @click:append="clearText()"
            @keyup.enter="getAPI()"
            >
        </v-text-field>

        <v-treeview
            v-if="inputingSelect"
            activatable
            color="primary"
            dense
            :open-all="false"
            :items="districts"
            @update:active="clickNode($event)"
        ></v-treeview>
       </div>

        <WeatherInfo :dataAPI="dataAPI" :query-type="queryType" :inputText="inputText" />
    </div>
</div>


</template>

<script>
import axios from 'axios'
import WeatherInfo from './WeatherInfo.vue'
export default {
    name: "Weather",
    components: { WeatherInfo },
    data() {
        return {
            show: false,
            alertBox: 'alertBox',
            inputingKey: false,
            inputingSelect: false,
            inputText: '',
            dataAPI: null,
            rules: {
                name: [val => (val || '').length > 0 || 'This field is required']
            },
            districts: [],
            selectedAdcode: "",
            queryType: 1,
        };
    },
    mounted(){
        let options = {
            enableHighAccuracy: false,
            timeout: 5000,
            maximumAge: 0
        };
        if(navigator.geolocation){
            navigator.geolocation.getCurrentPosition(this.onGeoSuccess, this.onGeoError, options);
        }
    },
    methods: {
        onGeoSuccess(ops) {
            let latitude = this.limitDotStr6(ops.coords.latitude)
            let longitude = this.limitDotStr6(ops.coords.longitude)
            const location = `${longitude},${latitude}`
            axios.get(`https://restapi.amap.com/v3/geocode/regeo?key=${process.env.VUE_APP_KEY_WEATHER}&location=${location}`)
            .then(res => {
                this.inputText = res.data?.regeocode?.addressComponent?.adcode
                this.queryType = 1
                this.getAPI()
            })
        },

        onGeoError(err) {
            console.log(err);
        },

        limitDotStr6(str) {
            str = str.toString()
            const parts = str.split(".")
            if(parts.length > 1 && parts[1].length > 6){
                return `${parts[0]}.${parts[1].substring(0,6)}`
            }
            return str
        },

        clearText() {
            this.inputText = ''               
            this.$refs.weatherInput.focus()         
        },

        showToastMessage() {
            const app = document.getElementById('app')
                const toast = document.createElement('div')
                toast.classList.add('toast')
                toast.style.animation = ''
                toast.innerHTML =  `
                    <img class="toast-icon" src="https://img.icons8.com/external-sbts2018-outline-color-sbts2018/58/000000/external-alert-emergencies-set-sbts2018-outline-color-sbts2018.png"/>
                    
                    <div class="toast-message">
                            <h3>Danger</h3>
                            <h5>There's not have any city</h5>
                    </div>
                `
                app.appendChild(toast)

                setTimeout(() => {
                    app.removeChild(toast)
                },3000)
        },

        closeBox(){
            this.dataAPI = ""
            this.inputingKey = false
            this.inputingSelect = false
            this.show = !this.show
        },

        openKeyInBox(){
            this.queryType = 1
            this.inputingKey = true
            this.show = !this.show
        },

        openSelectBox(){
            this.queryType = 2
            axios.get(`https://restapi.amap.com/v3/config/district?key=${process.env.VUE_APP_KEY_WEATHER}&keywords=中国&subdistrict=3`)
            .then(res => {
                const data = res.data.districts[0].districts
                this.districts = this.recursion(data)
            })
            this.inputingSelect = true
            this.show = !this.show
        },
        
        recursion(arr){
            const data = []
            arr.map((item,index) => {
                let obj = {}
                obj.name = item.name
                obj.adcode = item.adcode
                obj.id = item.adcode
                if(item.districts.length > 0){
                    obj.children = this.recursion(item.districts)
                }else{
                    obj.children = []
                }
                data.push(obj)
            })
            return data
        },

        getAPI() {
            if(this.inputText == "" && this.selectedAdcode == "") {
                this.showToastMessage()
                this.$refs.weatherInput.focus();           
            }
            else {
                try {
                    if(this.queryType == 1){
                        let adcode = ""
                        axios.get(`https://restapi.amap.com/v3/config/district?key=${process.env.VUE_APP_KEY_WEATHER}&keywords=${this.inputText}`)
                        .then(response => {
                            if(response.data ?. districts.length == 0 || response.data ?. districts[0] ?. adcode == 100000){
                                this.showToastMessage()
                                this.inputText = ''
                                return
                            }
                            adcode = response.data ?. districts[0] ?. adcode

                            this.getWeatherInfo(adcode)
                        })
                    }else{
                        this.getWeatherInfo(this.selectedAdcode)
                    }
                }
                catch (error) {
                   if (error.response) {
                        console.log(error.response.data);
                        console.log(error.response.status);
                        console.log(error.response.headers);
                    } else if (error.request) {
                        console.log(error.request);
                    }
                    else {
                        console.log('Error', error.message);
                    }
                }             
            }     
        },

        getWeatherInfo(adcode){
            axios.get(`https://restapi.amap.com/v3/weather/weatherInfo?key=${process.env.VUE_APP_KEY_WEATHER}&city=${adcode}`)
            .then(response => {
                this.dataAPI = response.data;
                this.inputText = ''            
                this.selectedAdcode = ''            
            })  
            .catch(() => {
                this.showToastMessage()
                this.inputText = '' 
                this.selectedAdcode = ''            
            });
        },

        clickNode(e){
            this.selectedAdcode = e[0]
        }
    }
}
</script>

<style>
.down-arrow{
    fill: var(--white-color);
}

.custom-label-color input{
  color: var(--white-color)!important;
}
.input-city {
  margin-top: 20px;
}
.container{
    margin: 0 auto;
    position: relative;
}

#container{
    position: fixed;
    top: 50%;
    left: 65%;
    transform: translate(-50%,-50%);
}

.content{
    margin: 0 auto;
    width: 700px;
}

.content .btn1, .content .btn3{
    margin-right: 30px;
}

.v-treeview{
    height: 500px;
    overflow: scroll;
    width: 200px;
}

/* toast  */
.toast{
    background-color: #FF4949;
    position: absolute;
    top: 20px;
    right: 10px;
    border-radius: 15px;
    height: 60px;
    width: 250px;
    display: flex;
    align-items: center;
    animation: slideIn  ease 1.5s, fadeOut linear 1s 3s;
    box-shadow: -1px 2px 10px 0px rgba(0,0,0,0.75);
}

.toast-message{
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    color: var(--white-color);
}

.toast-icon{
    height: 30px;
    padding: 0 10px;
}

@keyframes slideIn{
    from {
        right: -40px;       
        opacity: 0;
    }
    to {
        right: 10px;
        opacity: 1;
    }
}

@keyframes fadeOut{
    to{
        opacity: 0;
    }
}

</style>