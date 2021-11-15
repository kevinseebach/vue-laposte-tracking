<template>
    <div class="container-stepper">
      <div class="container">
        <h2 class="mainTitleTracking">{{carrier}} N°{{ tracking }}</h2>
        <div v-if="returnCodeOkapi == '200'">
          <div class="timeline-progress-bar">
            <div class="dots">
              <div class="dots__wrapper" v-bind:class="{ active: ev.status || isFinal}" v-for="(ev,index) in timeline" :key="ev.id">
                <div class="dots__item" v-bind:class="{ active: ev.status || isFinal}"></div>
                <span class="dots__line" v-if="index != Object.keys(timeline).length - 1"></span>
              </div>
            </div>
          </div>
          <ul class="progressbar">
            <li v-for="ev in timeline" :key="ev.id">
              <p class="labelShort" v-if="ev.status">{{ev.shortLabel}}</p>
              <p class="date"  v-if="ev.status">{{ev.date|frenchDate}}</p>
            </li>
          </ul>
        <div class="container-detail-etapes">
          <h3 class="text-center">Étapes d’acheminement</h3>
          <div class="details ">
            <table class="table">
              <thead>
                <tr>
                  <th>DATES</th>
                  <th>ÉTAPES</th>
                </tr>
              </thead>
              <tbody>
                <tr class="" v-for="item in steps" :key="item.id">
                  <td class="">{{ item.date | frenchDate}}</td>
                  <td>
                    {{ item.label }}
                    <span v-if="item.infoPlus"><br/>{{item.infoPlus}}</span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
      <div v-else-if="returnCodeOkapi == '400'" class="warningTracking">
        Numéro invalide
      </div>
      <div v-else class="errorTracking">
        Erreur
      </div>
    </div>


  </div>
</template>

<script>
import axios from "axios"
import moment from 'moment'
export default {
  name: "LaposteTracking",
  data(){
    return {
      returnCodeOkapi:"",
      steps: [],
      isFinal : false,
      timeline : [],
      carrier: ""
    }
  },
  props: {
      tracking : {
        type :String,
        required : true
      },
  },
  filters: {
    frenchDate: function (date) {
      return (date) ? moment(date).format('DD/MM/YYYY') : "N.C.";
    }
  },
  mounted: function() {
    axios
      .get('https://api.laposte.fr/suivi/v2/idships/'+this.tracking,{
        'headers': {
          'X-Okapi-Key': 'hNGX6CyWPEg512pUgcnyl3z2TwFMgFM38d4qGVWW6cnErRlmQZC1XacRgUpENmu4'
        }
      })
      .then(res => {
        this.steps = [];
        this.returnCodeOkapi = res.data.returnCode;
        this.timeline = res.data.shipment.timeline;
        this.isFinal = res.data.isFinal;
        this.carrier = res.data.shipment.product;
        res.data.shipment.event.forEach((item,i) => {
          item.id = i;
          if(item.code == "AG1")item.infoPlus = res.data.shipment.contextData.removalPoint.name;
          this.steps.push(item);
        });
      })


  }
}
</script>

<style media="screen">
.container-stepper{
  max-width: 90%;
  margin: auto;
}
.mainTitleTracking{
  text-transform: capitalize;
}
.progressbar {
    display: flex;
    font-size: 100%;
    vertical-align: baseline;
    background: transparent;
    list-style-type: none;
    margin-top:0px;
    box-sizing: border-box;
    line-height: 1.4;
    margin-bottom: 30px;
    padding: 0;
}
.progressbar li {
    color: #ffc928;
    font-size: 12px;
    flex-grow: 1;
    padding: 0 10px;
    position: relative;
    text-align: center;
}

.progressbar li{
    list-style-type: none;
    width: 20%;
}
.progressbar li .labelShort {
    color: #003da5;
    font-size: 12px;
    margin-top: 5px;
    margin-bottom: 5px;
}
.progressbar li .date {
    color: #20458f;
    font-size: 15px;
    font-weight: 700;
    padding-bottom: 12px;
    margin-top:0;
}
.dots {
    width: 100%;
}

.dots, .dots__wrapper {
    display: flex;
    position: relative;
}

.dots__wrapper {
    align-items: center;
    width: 20%;
    justify-content: center;
}


.dots__item {
    background-color: #ccc;
    border-radius: 50%;
    height: 25px;
    width: 25px;
    z-index: 5;
}
.dots__line {
    height: 4px;
    left: 50%;
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    width: 100%;
    z-index: 3;
    background: #CCC;
}

.timeline-progress-bar .active .dots__line {
    background-color: #ffc928;
}
.dots__wrapper.active {
    background-color: transparent;
}
.timeline-progress-bar .dots__wrapper.active .dots__item {
    background-color: #ffc928;
}

.container-detail-etapes table{
  width: 90%;
  margin: auto;
  border-collapse: collapse;
border-spacing: 0;
}
.container-detail-etapes thead th{
  background: #ccc;
  padding: 10px;
}
.container-detail-etapes tr td{
  padding: 5px 10px;
}
.container-detail-etapes tr td:nth-child(2){
  text-align: left;
}
.container-detail-etapes td,
.container-detail-etapes th{
  border:2px solid #CCC;
  margin: 0;
}
.container-detail-etapes tr{
  margin: 0;
  padding: 0;
}
.errorTracking{
  background: #e39090;
color: #720c0c;
padding: 20px;
border-radius: 20px;
}
.warningTracking{
  background: #e3b790;
color: #946815;
padding: 20px;
border-radius: 20px;
}
</style>
