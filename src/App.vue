<template>
  <div id="app" class="container-fluid">
    <Header/>
    <div class="container">
      <tl-chart-form :title="formTitle" @submitForm="onFormSubmit"/>
      <div class="time-line-container">
        <!-- time line chart component -->
        <time-line-chart v-if="timeline.length > 0" :data="timeline" :title="timelineTitle"></time-line-chart>
      </div>
    </div>
    <Footer/>
  </div>
</template>

<script>
import Header from './components/Header.vue';
import Footer from './components/Footer.vue';
import Timelinechart from './components/Timelinechart.vue'
import Form from './components/Form.vue';
export default {
  name: 'app',
  components: {
    Header,
    Footer,
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
    }
  },
  mounted(){
  },
  data(){
    return{
      timelineTitle:"time line chart",
      formTitle: "Event Form",
      timeline:[
        {
          label: "Event-1",
          data: [
            {
              type: "POINT",
              at: new Date([2018, 1, 1]),
              msg:'Tooltip 1'
            },
            {
              type: "POINT",
              at: new Date([2018, 5, 15]),
              msg:'Tooltip 2'
            }
          ]
        },
        {
          label: "Event-2",
          data: [
            {
              type: "POINT",
              at: new Date([2018, 1, 11]),
              msg:'Tooltip 3'
            },
            {
              type: "POINT",
              at: new Date([2018, 8, 25]),
              msg:'Tooltip 4'
            }
          ]
        }
      ]
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
