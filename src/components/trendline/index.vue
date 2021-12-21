<template>
  <div class="trendlineOutbox">
    <div class="trendline" ref="trendlineRef"></div>
  </div>
</template>
<script>
import * as d3 from "d3";

export default {
  name: "trendline",
  data() {
    return {};
  },

  methods: {
    d3draw(val) {
      let PathArray = [];
      let TimeArray = [];
      let IndexArray = [];
      let NameArray = [];
      let NameIndexArray = [];
      let BarArray = [];

      val.map((ele, index) => {
        let PathData = [];
        NameArray.push(ele.Name);
        NameIndexArray.push(index);
        ele.Data.map((el, i) => {
          PathData.push(el.Data);
          if (index === 0) {
            TimeArray.push(el.Time + "test");
            IndexArray.push(i);
            BarArray.push({
              key: i,
              value: "360",
            });
          }
        });
        PathArray.push(PathData);
      });
      this.d3Fun(
        PathArray,
        TimeArray,
        IndexArray,
        NameArray,
        NameIndexArray,
        BarArray
      );
    },
    CanD3SvgFun() {
      //画动态图之前清除之前图形
      d3.selectAll(".My_svg").selectAll("g").remove();
    },
    d3Fun(
      PathArray,
      TimeArray,
      IndexArray,
      NameArray,
      NameIndexArray,
      BarArray
    ) {
      console.log(
        this.$refs.trendlineRef.offsetWidth,
        this.$refs.trendlineRef.offsetHeight,
        PathArray,
        TimeArray,
        IndexArray,
        NameArray,
        NameIndexArray,
        BarArray
      );
      //设置宽高
      let width = this.$refs.trendlineRef.offsetWidth;
      let height = this.$refs.trendlineRef.offsetHeight;
      let top_margin = ~~(height / 13);
      let SvgRate = 0.8;
      let translateRateX = 0.1;
      let translateRateY = 0.1;
      //d3初始化svg容器
      let svg = d3
        .select(this.$refs.trendlineRef)
        .append("svg")
        .attr("class", "My_svg")
        .attr("width", width)
        .attr("height", height)
        .attr("viewBox", [0, 0, width * 1, height * 1])
        .append("g")
        .attr("class", "My_Root_G")
        .attr("transform", `translate(${width * 0},${height * 0})`);
      //画X,Y坐标轴
      var xScale = d3
        .scaleLinear()
        .domain([0, d3.max(IndexArray)])
        .range([0, width * SvgRate])
        .nice();
      var axisX = d3
        .axisBottom()
        .scale(xScale) //指定比例尺
        .tickFormat((d) => {
          console.log("d.substring(0,2)", d);
          return d === IndexArray.length
            ? TimeArray[IndexArray.length - 1]
            : TimeArray[d];
        })
        .ticks(8); //指定刻度的数量
      svg
        .append("g")
        .attr("class", "axisX")
        .attr("class", "DongTaiDrawClass DongTaiDrawPipe")
        .attr(
          "transform",
          `translate(${width * translateRateX}, ${height * SvgRate})`
        )

        .call(axisX)
        .call((g) =>
          g
            .select(".domain")
            .attr("class", "axisX_MydomainPathStyle")
            .attr("stroke-width", 0.5)
            .attr("stroke", "#444D")
        )
        .call((g) =>
          g
            .selectAll(".tick")
            .selectAll("line")

            .attr("class", "axisX_MyTickLineStyle")
            .attr("stroke-width", 0.5)
            .attr("stroke", "#444D")
        )
        .call((g) =>
          g
            .selectAll(".tick")
            .selectAll("text")
            .attr("class", "axisX_MyTextStyle")
            .attr("stroke", "#E1E1E1")
            .attr("stroke-width", 0.5)
        )
        .on("mousemove", function (d, i) {
          console.log("over--axisX", d, i);
        });
      //y轴
      var yScale = d3
        .scaleLinear()
        .domain([0, d3.max(NameIndexArray)])
        .range([0, -height * SvgRate])
        .nice();
      var axisY = d3
        .axisLeft()
        .scale(yScale) //指定比例尺
        .tickFormat((d) => {
          return d === NameIndexArray.length
            ? NameArray[NameIndexArray.length - 1]
            : NameArray[d];
        })
        .ticks(8); //指定刻度的数量
      svg
        .append("g")
        .attr("class", "axisY")

        .attr(
          "transform",
          `translate(${width * translateRateX}, ${height * SvgRate})`
        )
        .call(axisY)
        .call((g) =>
          g
            .attr("class", "axisX_MydomainPathStyle")
            .attr("stroke-width", 0.5)
            .attr("stroke", "#444D")
        )
        .call((g) =>
          g
            .selectAll(".tick")
            .selectAll("line")
            .attr("class", "axisX_MyTickLineStyle")
            .attr("stroke-width", 0.5)
            .attr("stroke", "#444D")
        )
        .call((g) =>
          g
            .selectAll(".tick")
            .selectAll("text")
            .attr("class", "axisX_MyTextStyle")
            .attr("stroke", "#E1E1E1")
            .attr("stroke-width", 0.5)
        );
      ///绘制path
      var linearX = d3
        .scaleLinear()
        .domain([0, PathArray[0].length - 1])
        .range([0, width * SvgRate]);
      var linearY = d3
        .scaleLinear()
        .domain([0, PathArray.length - 1])
        .range([top_margin, height * SvgRate]);
      var line = d3
        .line()
        .x((d, i) => linearX(i))
        .y((d) => -d * SvgRate);
      d3.select(".My_Root_G")
        .append("g")
        .selectAll(".wave")
        .data(PathArray)
        .enter()
        .append("path")
        .attr(
          "transform",
          (d, i) => `translate(${width * translateRateX},${linearY(i * 0.6)})`
        )
        .attr("class", "wave")
        .attr("d", line)
        .attr("fill", "transparent")
        .attr("stroke", "#444")
        .attr("stroke-width", 1);
      ///////

      var tooltip = d3
        .select(".trendline")
        .append("div")
        .attr("id", "tooltip") //用于css设置类样式
        .attr("class", "d3-tip")
        //.attr("opacity", 1.0)
        .style("display", "none")
        .style("position", "absolute");
      svg
        .append("g")
        .attr("class", "MytooltipLineG")
        .attr(
          "transform",
          `translate(${width * translateRateX}, ${height * SvgRate})`
        )
        .selectAll("line")
        .data(BarArray)
        .enter()
        .append("line")
        .attr("class", "MytooltipLine")

        .attr("x1", (d) => {
          console.log("x1", d);
          return xScale(d.key);
        })
        .attr("y1", 0)
        .attr("x2", (d) => {
          console.log("x2", d);
          return xScale(d.key);
        })
        .attr("y2", yScale(height * SvgRate))
        .attr("stroke", "transparent")
        .attr("stroke-width", "1px")
        .on("mousemove", function (event, d) {
          console.log("over", event, d.key);
          var x = event.offsetX * 1 > 432 ? 432 : event.offsetX * 1;
          var y = event.offsetY * SvgRate;
          // tooltip.style("opacity", 0.0);
          // tooltip.style("display", "none");
          let InitHtml = `<div class="MytooltipOutbox">`;
          let InitTitleHtml = "";
          NameArray.map((item, index) => {
            if (index === 0) {
              InitTitleHtml = ` <div class="MytooltipTitlebox">时间:${
                d.key === PathArray.length ? PathArray.length - 1 : d.key
              }</div>`;
            }
            InitHtml += `
             <div class="MytooltipItembox">${item}:${
              PathArray[index][
                d.key === PathArray.length ? PathArray.length - 1 : d.key
              ]
            }</div>
           
            `;
          });
          let NewInitTitleHtml = `</div>`;
          tooltip
            .html(InitTitleHtml + InitHtml + NewInitTitleHtml)
            .style("max-width", 120 + "px")
            .style("left", x + "px")
            .style("top", y + "px")
            .style("display", "block")
            .style("opacity", 1.0);
        })
        .on("mouseout", function (d, i) {
          // tooltip.style("opacity", 0.0);
          // tooltip.style("display", "none");
        });
    },
  },
};
</script>
<style lang="scss" scoped>
.trendlineOutbox {
  .trendline {
    width: 560px;
    height: 360px;
    // background-color: rgb(230, 224, 224);
    margin-top: 60px;
    position: relative;
    svg {
      // .wave:hover {
      //   fill: rgb(207, 38, 38);
      //   stroke: rgb(207, 38, 38);
      //   stroke-width: 1;
      // }
    }
  }
}
</style>
<style lang="scss">
.d3-tip {
  /*提示框的样式*/
  box-shadow: 0 0 9px 3px #999;
  position: absolute !important; /*不可缺少*/
  line-height: 1;
  font-weight: bold;

  background-color: rgb(0, 0, 0);
  opacity: 0.8;
  color: #fff;
  border-radius: 2px;
  z-index: 9999999;
}
.trendline .MytooltipLine:hover {
        fill: rgb(207, 38, 38);
        stroke: rgb(207, 38, 38);
        stroke-width: 1;
      }
.MytooltipTitlebox {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  width: 100%;
  height: 36px;
  text-indent: 10px;
  pointer-events: none;
}
.MytooltipOutbox {
  max-width: 260px;
  max-height: 130px;
  overflow: auto;
   pointer-events: none;
  &::-webkit-scrollbar {
    width: 5px;
    height: 5px;
    background-color: #f5f5f5;
    background-color: transparent;
  }
  /*定义滚动条轨道 内阴影+圆角*/
  &::-webkit-scrollbar-track {
    //  -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
    border-radius: 10px;
    background-color: #f5f5f5;
    background-color: transparent;
  }
  /*定义滑块 内阴影+圆角*/
  &::-webkit-scrollbar-thumb {
    border-radius: 10px;
    // -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, .3);
    background-color: #cfd6db;
    //background-color: transparent;
  }
  .MytooltipItembox {
     text-indent: 10px;
    margin-bottom: 10px;
    &:last-child {
      margin-bottom: 0;
    }
  }
}
</style>