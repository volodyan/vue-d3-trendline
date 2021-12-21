<template>
  <div class="hello">
    <component :is='"trendline"' ref="trendlineComponentRef"></component>
  </div>
</template>

<script>
import trendline from './trendline/index.vue'
export default {
  name: "HelloWorld",
  components:{
    trendline
  },
  mounted(){
    this.GetResdata()
  },
  methods:{
    GetResdata(){
      this.$axios.get("data/trendline.json")
      .then((res) => {
        console.log("res",res)
        if(!!res.data.length){
          this.$refs.trendlineComponentRef.d3draw(res.data)
        }else{

        }
        
      })
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.hello {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-flow: row nowrap;
  justify-content: center;
  background: transparent;
}
</style>
