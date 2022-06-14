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
          <Slider v-model="values[ingred.code]" :max="ingred.maxAmount"/>
        </td>
        <td class="ingr"><input class="inpVal" v-model="values[ingred.code]" placeholder="{{values[ingred.code]}}" v-on:change="validate(ingred.code,ingred.maxAmount)"></td>
      </tr>


      <tr><td class="dop" colspan="2">
        <textarea class="inpVal" v-model="addInfo" placeholder="Дополнительно:"></textarea></td></tr>
      <tr><td class="btnLayer" colspan="3"><button class="btn1" v-on:click="send">Отправить</button></td></tr>
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
      values: {},
      costil: 1,
      addInfo: null,
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
      //TODO
    },
    validate(code,maxV){
      if(this.values[code]>maxV){
        this.value[code]=maxV;
      }
    }
  }
}
</script>

<style src="@vueform/slider/themes/default.css"></style>
<style scoped>
  @import '../assets/styles/healers/index.css';
</style>