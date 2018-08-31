<template>
    <div class="form-container">
        <!-- form start -->
        <h3 class="text-left" v-if="title">{{title}}</h3>
        <div class="row">
          <form action=""  class="form-inline" v-on:submit.prevent="onSubmit" >
            <div class="form-group  mb-sm-6 mb-3">
              <label for="timelineType"  class="sr-only">Type</label>
              <select  v-model="timelinenew.type"  class="form-control" id="timelineType" >
                <option value="" :selected="true">Select Event</option>
                <option v-for="(option, key) in eventOptions" 
                :value="option.title" v-bind:key="key"
                >{{ option.title }}</option>
              </select>
            </div>
            <div class="form-group  mx-sm-3 mb-3">
              <label for="timelineDate"  class="sr-only">Date</label>
              <datepicker v-model="timelinenew.date"  placeholder="Select Date" class="form-control" name="timelineDate"></datepicker>
            </div>
            <div class="form-group  mx-sm-3 mb-3">
              <label for="timelineDescription"  class="sr-only">Tooltip</label>
              <input type="text" v-model="timelinenew.tooltip" class="form-control" id="timelineDescription" placeholder="Description">
            </div>
            <div class="form-group  mx-sm-3 mb-3">
                  <input type="submit" class="btn btn-primary" value="Add Item"/>
              </div>
          </form>
        </div>
        <!-- form end -->
      </div>
</template>
<script>
import Datepicker from "vuejs-datepicker";
export default {
  name: "tl-chart-form",
  props: {
    title: String
  },
  components: {
    Datepicker
  },
  data() {
    return {
      eventOptions: [
        {
          title: "Event-1"
        },
        {
          title: "Event-2"
        },
        {
          title: "Event-3"
        },{
          title: "Event-4"
        }
      ],
      timelinenew: {
        type: "",
        date: "",
        tooltip: ""
      }
    };
  },
  mounted() {},
  methods: {
    /**
     * Form data submit to parent component
     * Form reset to empty
    */
    onSubmit() {
      if (
        this.timelinenew.type &&
        this.timelinenew.date &&
        this.timelinenew.tooltip
      ) {
        this.$emit("submitForm", this.timelinenew);
        this.timelinenew = {};
      }
    }
  }
};
</script>
<style>
  h3 {
    text-transform: capitalize;
  }
  
  .form-container{
    padding-top:10px;
  }
</style>
