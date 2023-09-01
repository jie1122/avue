<template>
  <div :class="b()">
    <el-button 
      type="primary"
      :size="size"
      @click="handleShow"
      :disabled="disabled"
    >
      <slot>数据加载</slot>
    </el-button>
    <div v-if="box">
      <el-dialog class="avue-dialog"
                 :width="dialogWidth"
                 :append-to-body="$AVUE.appendToBody"
                 lock-scroll
                 title="数据加载"
                 v-model="box">
        <avue-crud :class="b('crud')"
                   ref="crud"
                   v-if="box&&cProps.needPage"
                   :option="option"
                   :data="data"
                   :table-loading="loading"
                   @on-load="onList"
                   @search-change="handleSearchChange"
                   @search-reset="handleSearchChange"
                   @current-row-change="handleCurrentChange"
                   v-model:page="page"
                   v-model:search="search"
                   ></avue-crud>
        <avue-crud :class="b('crud')"
                   ref="crud"
                   v-if="box&&!cProps.needPage"
                   :option="option"
                   :data="data"
                   :table-loading="loading"
                   @on-load="onList"
                   @search-change="handleSearchChange"
                   @search-reset="handleSearchChange"
                   @current-row-change="handleCurrentChange"
                   v-model:search="search"
                   ></avue-crud>
                   
        <template #footer>
          <span class="dialog-footer">
            <el-button type="primary"
                       :size="size"
                       icon="el-icon-check"
                       @click="setVal">{{t("common.submitBtn")}}</el-button>
          </span>
        </template>

      </el-dialog>
    </div>

  </div>
</template>

<script>
import create from "core/create";
import props from "common/common/props.js";
import event from "common/common/event.js";
import locale from "core/locale";
import { getAsVal } from "utils/util";

export default create({
  name: "data-load",
  mixins: [props(), event(), locale],
  inject: ["formSafe"],

  data () {
    return {
      active: {},
      page: {},
      search:{},
      loading: false,
      box: false,

      data: []
    };
  },
  props: {

    dialogWidth: {
      type: String,
      default: '80%'
    },
  },
  computed: {

    option () {
      return Object.assign({
        menu: false,
        header: false,
        size: this.size,
        headerAlign: 'center',
        align: 'center',
        highlightCurrentRow: true,
      }, this.column.children)
    },
    cProps(){
      return this.column.children.props || {}
    },
    cColumn(){
      return this.column.children.column
    }
  },
  methods: {

    handleShow () {
      // this.$refs.main.blur();
      if (this.disabled || this.readonly) return;
      this.page = {
        currentPage: 1,
        total: 0
      }
      // 将表单数据根据映射关系赋值到搜索框作为默认搜索条件
      const formData = this.formSafe.form
      this.cColumn.forEach(e => {
        if (e.target && e.search && Object.hasOwnProperty.call(formData, e.target)) {
          this.search[e.prop] =formData[e.target]
        }
      })
      this.box = true;
    },
    setVal () {

      const formData = this.formSafe.form
      this.cColumn.forEach(e => {
        if (e.target && Object.hasOwnProperty.call(formData, e.target)) {
          formData[e.target] = this.active[e.prop]
        }
      })
      this.box = false
    },
    // 表格选中行变化
    handleCurrentChange (val) {
      this.active = val;
    },
    handleSearchChange (form, done) {
      done()
      this.loadData(done)
    },
    onList (callback) {
      this.loadData()
    },
    loadData(done){
      if (!this.cProps) return
      const { url,tableName, method, needPage, currentPageKey, pageSizeKey, totalKey, recordsKey, resKey, auto } = this.cProps

      if (!auto || !url) return 
      let page = this.page
      let data = this.search
      // 分页相关
      if (!page) page = { currentPage: 1, pageSize: 10 }
      const { currentPage, pageSize } = page
      const params = {
        [currentPageKey]: currentPage,
        [pageSizeKey]: pageSize 
      }

      // 构造查询条件
      const queryColumns = [] // 要查询的列
      const whereColumnList = [] // 查询条件
      this.cColumn.forEach(col => {
        queryColumns.push(col.prop)
        if (data && data[col.prop] !== null && data[col.prop] !== undefined  && data[col.prop] !== '') {
          whereColumnList.push({
            columnName:col.prop ,
            columnValue: data[col.prop],
            whereType: col.whereType || 'key_10', //默认为相等
          })
        }
      })
      // 查询参数
      const queryData = {
        tableName:tableName, //表名
        queryColumns  ,
        whereColumnList   
      }

      this.loading = true;
      this.$axios({
        method: method || 'post',
        url,
        params,
        data:queryData
      }).then(res => {
        const response = getAsVal(res, resKey || 'data')
        if (!response) this.$message.error('未查询到数据或者返回层级配置错误')

        let result = {}
        if (needPage) {
          const total = getAsVal(response, totalKey || 'total')
          if (total || total == 0) {
            result.total = total
          } else {
            this.$message.error('总条数配置错误')
            return
          }
          const records = getAsVal(response, recordsKey || 'records')
          if (records) {
            result.data = records
          } else {
            // this.$message.error('列表配置错误')
            result.data = []
          }
        } else{
          result = { total: response.length, data: response }
        }
        this.page.total = result.total;
        this.data = result.data
      }).finally( () => {
        this.loading = false;
        done && done()
      })

    }
  }
});
</script>
