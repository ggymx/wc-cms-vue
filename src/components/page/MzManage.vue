<!--新添页面：媒资管理-->
<template>
  <div class="mzContainer">
    <header>
      <section>
        <label>省份：</label>
        <el-select v-model="ele.selPro" multiple filterable placeholder="最多选31个">
          <el-option
            v-for="(item , index) in ele.provinces"
            :key="index"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </section>
      <section>
        <ul>
          <li>
            <label>视频ID：</label>
            <el-input v-model="ele.vID" placeholder="请输入视频ID" clearable></el-input>
          </li>
          <li>
            <label>视频名称：</label>
            <el-input v-model="ele.vName" placeholder="请输入视频名称" clearable></el-input>
          </li>
          <li>
            <label>公司名称：</label>
            <el-input v-model="ele.cName" placeholder="请输入公司名称" clearable></el-input>
          </li>
        </ul>
      </section>
      <section>
        <ul>
          <li>
            <label>注入状态：</label>
            <el-select v-model="ele.selInj" filterable placeholder="选择注入状态">
              <el-option
                v-for="(item , index) in ele.injStatus"
                :key="index"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </li>
          <li>
            <label>储存状态：</label>
            <el-select v-model="ele.selStr" filterable placeholder="选择储存状态">
              <el-option
                v-for="(item , index) in ele.strStatus"
                :key="index"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </li>
          <li>
            <label>创建时间：</label>
            <el-date-picker v-model="ele.selDate" align="right" type="date" placeholder="选择日期"></el-date-picker>
          </li>
        </ul>
      </section>
    </header>
    <article class="table">
      <div>
        <el-button type="primary">搜索</el-button>
        <el-button type="info">重置</el-button>
        <!--子组件修改值，通过emit通知修改，sync捕获修改值并更改父组件中的属性值-->
        <!--表头按钮组-->
        <v-tbCommonBtn :filter.sync="isFilterShow" :ep-file.sync="isEpFlieShow"></v-tbCommonBtn>
      </div>
      <el-table
        :data="mzArr"
        border
        style="width: 100%;font-size:0.8rem"
        stripe
        v-loading="loading"
        id="mzTable"
      >
        <!-- <el-table-column fixed prop="ID" label="ID" width="80"></el-table-column>-->
        <!--for循环尽量带上item和index-->
        <blockquote v-for="(item , index) in mzTreeTmp" :key="index">
          <el-table-column :prop="item.prop" :label="item.label" width="120"></el-table-column>
        </blockquote>

        <el-table-column label="操作" width="200" v-if="noErr">
          <template slot-scope="scope">
            <el-button
              @click="handleClick(scope.row)"
              type="text"
              size="small"
              style="color: #909399"
            >注入</el-button>
            <el-button type="text" size="small">更新</el-button>
            <el-button
              @click="handleClick(scope.row)"
              type="text"
              size="small"
              style="color: #f56c6c"
            >删除</el-button>
            <el-button type="text" size="small" style="color: #e6a23c">重新操作</el-button>
          </template>
        </el-table-column>
      </el-table>
    </article>
    <footer>
      <!--分页-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="pagination.curPage"
        :page-sizes="[10, 20, 50, 100, 200, 500]"
        :page-size="pagination.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="pagination.dataCount"
        style="display:inline-block;margin-right:20px;"
      ></el-pagination>
      <el-button type="primary">新建</el-button>
      <el-button type="info">导出</el-button>
      <el-button type="info">批量新建</el-button>
    </footer>
    <!--引入animate动画-->
    <transition
      name="fade"
      enter-active-class="animated fadeIn"
      leave-active-class="animated fadeOutLeft"
      :duration="200"
    >
      <v-columnFilter
        :th-label="mzfilter"
        :tb-data="mzTree"
        :check-mz.sync="checkMz"
        :position="position.filterBox"
        @updateTree="updateMzTree"
        v-if="isFilterShow"
      ></v-columnFilter>
    </transition>
    <!--上传文件弹出框，待封装-->
    <transition
      name="fade"
      enter-active-class="animated fadeIn"
      leave-active-class="animated fadeOutRight"
      :duration="200"
    >
      <!--驼峰命名转换-->
      <v-exportb :tid="outTable.id" :tname="outTable.name" :position="position.fileBox" v-if="isEpFlieShow"></v-exportb>
    </transition>
  </div>
</template>
<script>
import axios from "../../utils/request";
import { setTimeout } from "timers";
import vTbCommonBtn from "../common/TbCommonBtn";
import vColumnFilter from "../common/ColumnFilter";
import vExportb from "../common/ExportTb";

export default {
  name:'mzManage',
  data() {
    return {
      //元素绑定值
      ele: {
        selPro: "",
        vID: "",
        vName: "",
        cName: "",
        selStr: "",
        selDate: "",
        selInj: "",
        strStatus: [
          { value: "1", label: "全部" },
          { value: "2", label: "未储存" },
          { value: "3", label: "储存中" },
          { value: "4", label: "储存成功" },
          { value: "5", label: "储存失败" },
          { value: "6", label: "已移除" }
        ],
        injStatus: [
          { value: "1", label: "全部" },
          { value: "2", label: "未注入" },
          { value: "3", label: "注入成功" },
          { value: "4", label: "注入中" },
          { value: "5", label: "注入失败" }
        ],
        provinces: [
          {
            value: "sh",
            label: "上海市"
          },
          {
            value: "bj",
            label: "北京市"
          },
          {
            value: "fj",
            label: "福建省"
          },
          {
            value: "zj",
            label: "浙江省"
          }
        ]
      },

      //后台接收到的表格数据
      mzArr: null,
      //后台接收的控制表头的数据
      mzTree: null,
      mzTreeTmp: null,
      mzfilter: null,
      //分页控件
      pagination: {
        curPage: 1,
        dataCount: 0,
        pageSize: 20
      },
      //筛选列
      checkAll: false,
      checkMz: [],
      //控制筛选列弹出框的显示
      isFilterShow: false,
      isEpFlieShow: false,
      loading: true,
      //控制表格操作按钮是否显示
      noErr: true,
      //导出的表格信息
      outTable: {
        id: "mzTable",
        name: "媒资管理"
      },
      //控制弹出框的位置
      position: {
        filterBox: {
          top: "212px",
          right: "192px"
        },
        fileBox:{
          top: "212px",
          right: "80px"
        }
      }
    };
  },
  //注册组件
  components: {
    vTbCommonBtn,
    vColumnFilter,
    vExportb
  },
  created() {
    this.getData(1, this.pagination.pageSize,'id','asc');
    console.log('首次初始化');
  },
  /*配有keep-alive时的特有方法 */
  activated(){
    console.log('激活keep-alive中的缓存mzManage');
  },
  methods: {
    getData(pageIndex=1, pageSize,sortField,sort) {
      this.loading=true;
      axios
        .get("http://172.26.58.48/vda/getData", {
          params: {
            pageSize,
            pageIndex,
            sortField,
            sort:'asc'
          }
        })
        .then(res => {
          console.log('测试------',res);
          // this.mzArr = res.mzArr;
          // this.pagination.dataCount = this.mzArr.length;
          // this.mzTree = res.mzTree;
          // this.mzfilter = res.mzTree.map(item => item.label);
          // this.mzTreeTmp = this.mzTree.slice();
          this.mzArr = res.pageData.data.map(item=>{
            for(let i in item){
              // console.log('循环的i的值：',i);
              item[i]=item[i]||'暂无数据'
            }
            return item;
            });
          this.pagination.dataCount = res.count;
          this.mzTree = res.pageData.cols;
          this.mzfilter = this.mzTree.map(item => item.label);
          this.mzTreeTmp = this.mzTree.slice();
          console.log("mzArrTmp------", this.mzArrTmp, this.mzTreeTmp);
          setTimeout(() => {
            this.loading = false;
          }, 500);
          // console.log('mzfilter------',this.mzfilter);
        })
        .catch(err => {
          console.log('mz链接错误----');
           this.$message.error(`${err.message}`);
           this.loading = false;
           this.noErr = false;
        });
    },
    handleClick(row) {
      console.log("传入row");
    },
    //分页逻辑
    handleCurrentChange(pageIndex) {
      this.pagination.curPage=pageIndex;
      console.log("当前页面" ,this.pagination.curPage==pageIndex);
      this.getData(this.pagination.curPage,this.pagination.pageSize,'id','asc');
    },
    handleSizeChange(pageSize) {
      //limit  控制每页多少
      this.pagination.pageSize=pageSize;
      console.log("每个页面的条数----", this.pagination.pageSize , pageSize);
      this.getData(this.pagination.curPage,this.pagination.pageSize,'id','asc')
    },
    updateMzTree(data) {
      console.log("冒泡所传过来的值：", data);
      this.mzTreeTmp = data;
    }
  }
};
</script>
<style lang="less" scoped>
.mzContainer {
  width: 100%;
  min-height: 780px;
  padding-left: 30px;
  padding-top: 30px;
  position: relative;
}
ul {
  list-style-type: none;
  margin-top: 20px;
  & > li {
    display: inline-block;
    width: 33.3%;
    & > .el-input--small {
      width: 70%;
    }
  }
}

article.table {
  margin-top: 20px;
  & > div {
    margin-bottom: 10px;
    position: relative;
    & > div {
      position: absolute;
      right: 50px;
      display: inline-block;
    }
  }
}
label {
  color: #909399;
}
footer {
  margin-top: 20px;
}
.el-button--text {
  font-size: 0.8rem;
}
.el-select {
  width: 60%;
}

.el-table,
.el-table__expanded-cell {
  background-color: transparent;
}
</style>

