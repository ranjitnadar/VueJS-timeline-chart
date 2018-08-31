<template>
	<div>
    <h3 class="text-left" v-if="title">{{title}}</h3>
		<div id="timechart"></div>
	</div>
</template>
<script>
import * as d3 from "d3";
import d3Tip from "d3-tip"; // d3 tool tip
d3.tip = d3Tip;
export default {
  name: "time-line-chart",
  props: {
    data: Array,
    title: String
  },
  watch: {
    data: {
      /**
       * watch for newval and trigger initcall method
       *  this.initCall() function trigger if props value change, looking for best practise for how to call methods if props change. i will update this part if i find any.
       * deep true to check nested array
       */
      handler: function(newVal, oldVal) {
        this.timeline = newVal;
        this.initCall();
      },
      deep: true
    }
  },
  data() {
    /**
     * Init Data
     */
    return {
      width: 420,
      barHeight: 20,
      tlconfig: {},
      timeline: []
    };
  },
  beforeCreate() {
    //console.log("before create");
  },
  created() {
    //console.log("create");
  },
  beforeMount() {
    //console.log("beforeMount");
  },
  beforeUpdate() {
    //console.log("before updated");
  },
  updated() {
    //console.log("updated");
  },
  mounted() {
    this.timeline = this.data;
    this.initCall();
    //console.log("mounted");
  },
  methods: {
    initCall() {
      document.getElementById("timechart").innerHTML = "";
      let element = document.getElementById("timechart");
      this.chart(element, this.timeline, {
        tip: (d) => {
          let mlist = [ "Jan", "Feb", "March", "Apr", "May", "June", "July", "Aug", "Sep", "Oct", "Nov", "Dec" ];
          return `<span class="tooltiptext">${d.msg} <em> ${this.getDatesuffix(new Date(d.at).getDate())} ${mlist[new Date(d.at).getMonth()]}  ${new Date(d.at).getFullYear()} </em></span>`;
        }
      });
    },
    /**
     * el :selector can be class or id
     * data: structured format json object
     * opts: options
     */
    chart(el, data, opts) {
      el.classList.add("d3-time-line-chart");
      this.tlconfig.options = this.extendOptions(opts);
      //allEl: all element
      this.tlconfig.allEl = data.reduce((agg, e) => agg.concat(e.data),[]);
      this.tlconfig.minDt = d3.min(this.tlconfig.allEl, this.getPointMinDt);
      this.tlconfig.maxDt = d3.max(this.tlconfig.allEl, this.getPointMaxDt);
      this.tlconfig.elWidth = this.tlconfig.options.width || el.clientWidth;
      this.tlconfig.elHeight = this.tlconfig.options.height || el.clientHeight;
      this.tlconfig.margin = {
        top: 0,
        right: 0,
        bottom: 20,
        left: 0
      };
      this.tlconfig.width =
        this.tlconfig.elWidth -
        this.tlconfig.margin.left -
        this.tlconfig.margin.right;
      this.tlconfig.height =
        this.tlconfig.elHeight -
        this.tlconfig.margin.top -
        this.tlconfig.margin.bottom;
      this.tlconfig.groupWidth = this.tlconfig.options.hideGroupLabels
        ? 0
        : 200;

      this.tlconfig.x = d3.time
        .scale()
        .domain([this.tlconfig.minDt, this.tlconfig.maxDt])
        .range([this.tlconfig.groupWidth, this.tlconfig.width]);

      this.tlconfig.xAxis = d3.svg
        .axis()
        .scale(this.tlconfig.x)
        .orient("bottom")
        .tickSize(-this.tlconfig.height);

      let zoom = d3.behavior
        .zoom()
        .x(this.tlconfig.x)
        .on("zoom", this.zoomed);

      let svg = d3
        .select(el)
        .append("svg")
        .attr(
          "width",
          this.tlconfig.width +
            this.tlconfig.margin.left +
            this.tlconfig.margin.right
        )
        .attr(
          "height",
          this.tlconfig.height +
            this.tlconfig.margin.top +
            this.tlconfig.margin.bottom
        )
        .append("g")
        .attr(
          "transform",
          "translate(" +
            this.tlconfig.margin.left +
            "," +
            this.tlconfig.margin.top +
            ")"
        )
        .call(zoom);
      this.tlconfig.svg = svg;
      svg
        .append("defs")
        .append("clipPath")
        .attr("id", "chart-content")
        .append("rect")
        .attr("x", this.tlconfig.groupWidth)
        .attr("y", 0)
        .attr("height", this.tlconfig.height)
        .attr("width", this.tlconfig.width - this.tlconfig.groupWidth);

      svg
        .append("rect")
        .attr("class", "chart-bounds")
        .attr("x", this.tlconfig.groupWidth)
        .attr("y", 0)
        .attr("height", this.tlconfig.height)
        .attr("width", this.tlconfig.width - this.tlconfig.groupWidth);

      svg
        .append("g")
        .attr("class", "x axis")
        .attr("transform", "translate(0," + this.tlconfig.height + ")")
        .call(this.tlconfig.xAxis);

      let groupHeight = this.tlconfig.height / data.length;
      this.tlconfig.groupSection = svg
        .selectAll(".group-section")
        .data(data)
        .enter()
        .append("line")
        .attr("class", "group-section")
        .attr("x1", 0)
        .attr("x2", this.tlconfig.width)
        .attr("y1", (d, i) => {
          return groupHeight * (i + 1);
        })
        .attr("y2", (d, i) => {
          return groupHeight * (i + 1);
        });

      if (!this.tlconfig.options.hideGroupLabels) {
        svg
          .selectAll(".group-label")
          .data(data)
          .enter()
          .append("text")
          .attr("class", "group-label")
          .attr("x", 0)
          .attr("y", (d, i) => {
            return groupHeight * i + groupHeight / 2 + 5.5;
          })
          .attr("dx", "0.5em")
          .text(d => d.label);

        svg
          .append("line")
          .attr("x1", this.tlconfig.groupWidth)
          .attr("x2", this.tlconfig.groupWidth)
          .attr("y1", 0)
          .attr("y2", this.tlconfig.height)
          .attr("stroke", "white");
      }

      let groupDotItems = svg
        .selectAll(".group-dot-item")
        .data(data)
        .enter()
        .append("g")
        .attr("clip-path", "url(#chart-content)")
        .attr("class", "item")
        .attr("transform", (d, i) => `translate(0, ${groupHeight * i})`)
        .selectAll(".dot")
        .data(d => {
          return d.data.filter(_ => _.type === "POINT");
        })
        .enter();

      let dots = groupDotItems
        .append("circle")
        .attr("class", this.withCustom("dot"))
        .attr("cx", d => {
          return isNaN(this.tlconfig.x(d.msg)) ? 0 : this.tlconfig.x(d.msg);
        })
        .attr("cy", groupHeight / 2)
        .attr("r", 5);

      if (this.tlconfig.options.tip) {
        if (d3.tip) {
          let tip = d3
            .tip()
            .attr("class", "d3-tip")
            .html(this.tlconfig.options.tip);
          svg.call(tip);
          dots.on("mouseover", tip.show).on("mouseout", tip.hide);
        } else {
          console.error("Please make sure you have d3.tip included as dependency (https://github.com/Caged/d3-tip)");
        }
      }

      this.zoomed();
    },
    zoomed() {
      this.tlconfig.svg.select(".x.axis").call(this.tlconfig.xAxis);
      this.tlconfig.svg.selectAll("circle.dot").attr("cx", d => {
        return isNaN(this.tlconfig.x(d.at)) ? 0 : this.tlconfig.x(d.at);
      });
    },
    withCustom(defaultClass) {
      return d =>
        d.customClass ? [d.customClass, defaultClass].join(" ") : defaultClass;
    },

    extendOptions(opts = {}) {
      let defaultOptions = {
        intervalMinWidth: 8, // px
        tip: undefined,
        textTruncateThreshold: 30,
        timerTickInterval: 1000,
        hideGroupLabels: false
      };
      Object.keys(opts).map(k => (defaultOptions[k] = opts[k]));
      return defaultOptions;
    },
    /**
     * Get Point Min
     */
    getPointMinDt(p) {
      return p.type === "POINT" ? p.at : p.from;
    },
    /**
     * Get Point Max
     */
    getPointMaxDt(p) {
      return p.type === "POINT" ? p.at : p.to;
    },
    
    /**
     * param date:number
     * result added with suffix 
     * eg: 1st, 2nd, 3rd so on.
    */
    getDatesuffix(n) {
        let s=["th","st","nd","rd"],
        v=n%100;
        return n+(s[(v-20)%10]||s[v]||s[0]);
    }
  }
};
</script>
<style>

  *,
  *:after,
  *:before {
    box-sizing: border-box;
  }
  .d3-time-line-chart .axis path {
    fill: none;
    stroke: none;
  }

  .d3-time-line-chart line {
    stroke: black;
  }

  .d3-time-line-chart rect.interval {
    ry: 5;
    rx: 5;
    fill: black;
    stroke: #2b2b2b;
  }
  .d3-time-line-chart rect,
  .d3-time-line-chart rect.chart-bounds {
    fill: transparent;
  }
  #timechart {
    width: 100%;
    height: 15em;
    background: rgba(218, 218, 218, 0.18);
    opacity: 0.9;
  }

  #timechart line {
    stroke: #f0f0f0;
  }

  .d3-tip {
    position: relative;
    display: inline-block;
    width:120px;
    height: 10px;
  }

  .d3-tip .tooltiptext {
    width:120px;
    background-color: #fff;
    color: #000;
    text-align: center;
    border-radius: 5px;
    padding: 5px 0;
    position: absolute;
    z-index: 1;
    bottom: 100%;
    left: 50%;
    margin-left: -60px;
    border:1px solid #000;
  }

  .d3-tip .tooltiptext::after {
    content: " ";
    position: absolute;
    top: 100%;
    left: 50%;
    margin-left: -5px;
    border-width: 5px;
    border-style: solid;
    border-color: #000 transparent transparent transparent;
  }
  .tooltiptext em {
      font-size: 0.78em;
      display: block;
      color: rgb(100, 100, 100);
  }

  [flex] {
    display: flex;
  }
  [flex-fill] {
    flex: 1;
  }
  [flex-full-center] {
    align-items: center;
    justify-content: center;
  }
  [flex-direction="column"] {
    flex-direction: column;
  }
  [flex-direction="row"] {
    flex-direction: row;
  }
  [flex-size="40"] {
    flex: 40;
  }
  [flex-size="60"] {
    flex: 60;
  }
</style>
