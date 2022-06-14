<template>
  <div id="healers">
    <div class="content">
      <div class="head">Новая заявка на помощников</div>
    </div>
    <table class="form">
      <tr>
        <td class="penta" colspan="3">
          <SwitchVi label="Помощь с пентаграммой" checked v-on:change="triggerPenta"/>
        </td></tr>
      <tr v-for="ingred in dataInfo"
          class="ingred" :key="ingred.code">
        <td class="ingrNam">Необходимое количество: {{ingred.name}}</td>
        <td class="ingr">
          <Slider v-model="ingred.amount" :max="ingred.maxAmount" />
        </td>
        <td class="ingr"><input class="inpVal" v-model="ingred.amount" placeholder="0" v-on:change="ingred.amount=Math.min(ingred.amount,ingred.maxAmount)"></td>
      </tr>
      <!--сделать цикл-->
      <tr><td class="dop" colspan="2">
        <textarea class="inpVal" v-model="addInfo" placeholder="Дополнительно:"></textarea></td>
        <td></td>
      </tr>
      <tr><td class="btnLayer" colspan="3"><button class="btn1" v-on:click="send">Отправить</button>
      </td></tr>
      <tr class="fter"><td class="notification" colspan="3"><div v-show="notificationIsShow">
        Отправлено успешно!
      </div></td></tr>
    </table>
  </div>
</template>

<script>

import axios from "axios";

import SwitchVi from "@/components/Switch";
import Slider from '@vueform/slider'

export default {
  name: "HealersVi",
  components: {
    SwitchVi,
    Slider
  },
  el: '#healers',
  mounted: function () {
    // Attach event listener to the root vue element
    this.getIngr('http://localhost:8080/api/v1/healer/resources');
  },
  data() {
    return {
      needPenta: true,
      value: 0,
      maxVal:20,
      costil: 1,
      addInfo: "",
      notificationIsShow: false,
      dataInfo:null
    }
  },
  methods: {
    triggerPenta() {
      this.costil *= -1;
      if(this.costil<0){
      console.log("got trigger");
      this.needPenta = !this.needPenta;
    }},
    makeIngrList(){
      let index;
      for (index = 0; index < this.dataInfo.length; ++index) {
        //console.log(this.dataInfo[index]);
        this.dataInfo[index]["amount"]=0;
        console.log(this.dataInfo[index]);
      }
    },
    getIngr(url){
      return axios
          .get(url, {
            headers: {
              "Content-Type": "application/json"
            },
            auth: {
              username: 'healer',
              password: 'healer_pass'
            }} )
          .then(response => {
            this.dataInfo = response.data.resources;
            console.log("in get");
            console.log(this.dataInfo);
            this.makeIngrList();
          })
          .catch(error => {
            if (error.response) {
              // Request made and server responded
              console.log(error.response);
              console.log(error.response.status);
              console.log(error.response.headers);
            } else if (error.request) {
              // The request was made but no response was received
              console.log(error.request);
            } else {
              // Something happened in setting up the request that triggered an Error
              console.log('Error', error.message);
            }
          });
    },
    send(){
      let url="http://localhost:8080/api/v1/healer/request";
      //TODO
      console.log(this.dataInfo);
      const body = {
        "helperId": 0,
        "healerId": 1,
        "requiredPentaHelp": this.needPenta,
        "requestedResources": [],
        "description": this.addInfo
      }
      let index;
      for (index = 0; index < this.dataInfo.length; ++index) {
        //console.log(this.dataInfo[index]);
        if(this.dataInfo[index].amount){
          const res = {
            "resourceCode": this.dataInfo[index].code,
            "amount":this.dataInfo[index].amount
          }
          body.requestedResources.push(res);
        }
      }
      console.log(body);
      return axios
          .post(url, body,{
            headers: {
              "Content-Type": "application/json"
            },
            auth: {
              username: 'healer',
              password: 'healer_pass'
            }
          }).then(()=>{
            this.notificationIsShow = true;
            setTimeout(() => {
              this.notificationIsShow = false;
            }, 3000);
      });
    },
    validate(code, maxA){
      if(this.value[code]>maxA){
        this.value[code]=maxA;
      }
      this.addVal(code,this.value);
    }
  }
}
</script>

<style src="@vueform/slider/themes/default.css"></style>
<style scoped>
  @import '../assets/styles/healers/index.css';
</style>