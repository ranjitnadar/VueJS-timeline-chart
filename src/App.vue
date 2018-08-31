<template>
  <div id="app" class="container-fluid">
    <app-header/>
    <div class="container">
      <tl-chart-form :title="formTitle" @submitForm="onFormSubmit"/>
      <div class="time-line-container">
        <!-- time line chart component -->
        <time-line-chart v-if="timeline.length > 0" :data="timeline" :title="timelineTitle"></time-line-chart>
      </div>
    </div>
    <app-footer/>
  </div>
</template>

<script>
import Header from './components/Header.vue';
import Footer from './components/Footer.vue';
import Timelinechart from './components/Timelinechart.vue'
import Form from './components/Form.vue';
import axios from 'axios';
export default {
  name: 'app',
  components: {
    "app-header":Header,
    "app-footer":Footer,
    "time-line-chart":Timelinechart,
    "tl-chart-form":Form
  },
  methods:{
    /**
     * On Emit take value from child component
     * check label exit in array if no push it to parent array
     * if exist push it to respective label data
    */
    onFormSubmit(value){
      let newIndex = this.timeline
          .map(el => el.label)
          .indexOf(value.type);
        if (newIndex == -1) {
          this.$set(this.timeline, this.timeline.length, {
            label: value.type,
            data: [
              {
                label: value.tooltip,
                type: "POINT",
                at: value.date,
                msg: value.tooltip
              }
            ]
          });
        } else {
          this.$set(
            this.timeline[newIndex].data,
            this.timeline[newIndex].data.length,
            {
              label: value.tooltip,
              type: "POINT",
              at: value.date,
              msg:  value.tooltip
            }
          );
        }

    },
    getData(){
      axios.get('http://api.jsonbin.io/b/5b88ec4eab9a186eafe32547').then(res=>{
        let newarr = res.data.map(d =>{
          let newD =  d.data.map(r=>{
            r.at = new Date(r.at);
            return r;
          })
          d.data = newD;
          return d;
        });
        this.timeline = newarr;
      }).catch(e =>{
        console.log(e);
      })
    }
  },
  mounted(){
    this.getData();
  },
  data(){
    return{
      timelineTitle:"time line chart",
      formTitle: "Event Form",
      timeline:[]
    }
  }
}
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
    width:100%;
  }
  #app .row{
    margin: 0 auto;
  }
  .form-group {
    text-align: left;
  }
  .vdp-datepicker * {
    border: 0px;
  }
  .time-line-container{
    border-top:1px solid #f0f0f0;
    padding-top:20px;
  }
</style>
