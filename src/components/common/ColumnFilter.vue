<!--表格列过滤-->
<template>
  <div class="filter" :style="{top:position.top,right:position.right}">
    <el-checkbox
      :indeterminate="isIndeterminate"
      v-model="checkAll"
      @change="setCheckMzAll"
    >全选（勾选过滤）</el-checkbox>
    <div style="margin: 8px 0;"></div>
    <el-checkbox-group v-model="tmp_checkMz" @change="setCheckMz">
      <el-checkbox
        v-for="(item, index) in thLabel"
        :label="item"
        :key="index"
        style="display:block;"
      >{{item}}</el-checkbox>
    </el-checkbox-group>
  </div>
</template>
<script>
export default {
  data() {
    return {
      checkAll: false,
      isIndeterminate: true,
      tmp_checkMz: []
    };
  },
  props: {
    //表头标签
    thLabel: {
      type: Array,
      // 不能写成：default:[]，非基本类型都使用工厂模式
      default: ()=>["未加载"]
    },
    //表格数据
    tbData:{
        type: Array,
        default:()=>[]
    },
    checkMz:{
        type:Array,
        default:()=>[]
    },
    position:{
        type:Object,
        default: ()=>{}
    }
  },
  watch:{
      tmp_checkMz(val,oldVal){
         console.log('监听事件------');
         this.$emit('update:check-mz',val);
      }
  },
  methods: {
    //筛选列（全选）
    setCheckMzAll(val) {
      //重新初始化
      let mzTreeTmp = this.tbData;
      console.log("触发全选事件", val);
      this.tmp_checkMz = val ? this.thLabel : [];
      console.log("this.checkMz=-----", this.tmp_checkMz);
      this.isIndeterminate = false;
      //操作mzArrTmp，mzTreeTmp
      this.tmp_checkMz.forEach(element => {
        mzTreeTmp = mzTreeTmp.filter(item => element !== item.label);
      });
      this.$emit('updateTree',mzTreeTmp);
      console.log("mzArrTmp改变------", mzTreeTmp);
    },
    //筛选列
    setCheckMz(checkMz) {
      //重新初始化
      let mzTreeTmp = this.tbData;
      console.log("勾选的标签页-----", checkMz);
      let checkedCount = checkMz.length;
      this.checkAll = checkedCount === this.thLabel.length;
      this.isIndeterminate =
        checkedCount > 0 && checkedCount < this.thLabel.length;
      //操作mzArrTmp，mzTreeTmp
      checkMz.forEach(element => {
        mzTreeTmp = mzTreeTmp.filter(item => element !== item.label);
      });
      this.tmp_checkMz=this.checkMz;
      console.log("mzArrTmp改变------", mzTreeTmp);
      //通过冒泡将子组件的值传给父组件
      this.$emit('updateTree',mzTreeTmp);
    }
  },
  created() {
    this.tmp_checkMz=this.checkMz;
    console.log('this.tmp_checkMz---',this.tmp_checkMz);
  },
  destroyed(){
    console.log('columnFilter---组件销毁');
  }
};
</script>
<style lang="less" scoped>
.filter {
  position: absolute;
  top: 29px;
  // min-width: 150px;
  min-height: 40px;
  text-align: left;
  overflow-y: auto;
  background-color: rgb(255, 255, 255);
  box-shadow: rgba(0, 0, 0, 0.12) 0px 2px 4px;
  padding: 5px 0px;
  border-width: 1px;
  border-style: solid;
  border-color: rgb(210, 210, 210);
  border-image: initial;
  height: auto;
  padding: 10px;
  border-radius: 5px;
  // top: 212px;
  // right: 192px;
  z-index: 5;
  box-shadow: 0 0 20px #99ccff;
}
</style>
