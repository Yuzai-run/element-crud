<template>
  <div id="crud">
    <div class="safeSpot">
      <div class="spot_wrapper">
        <div class="spot_add">
          <slot name="slot-button-front"></slot>
          <el-button
            type="primary"
            class="add_con"
            @click="handleAdd"
            v-if="tableOption.addBtn || tableOption.addBtn==null"
          >
            <i class="el-icon-plus"></i> 新增
          </el-button>
          <el-button
            class="add_con"
            @click="batchDel()"
            v-if="tableOption.delBtn || tableOption.delBtn==null"
          >
            <i class="el-icon-delete"></i> 批量删除
          </el-button>
          <el-button
            type="primary"
            class="add_con"
            @click="handleExport"
            v-if="tableOption.exportBtn"
          >
            <i class="el-icon-download"></i> 导出
          </el-button>
          <slot name="slot-button-behind"></slot>
        </div>
        <!--搜索-->
        <div style="height: 50px;">
          <search-form
            :optionForm="searchOption"
            :searchData="searchData"
            :isReset="isReset"
            @searchForm="goSearch"
            @reset="reset"
          ></search-form>
        </div>
        <slot name="preTable"></slot>
        <!--<div class="avue-crud__tip" v-if="tableOption.columns[0].type == 'selection'">
					<i class="el-icon-info"></i>
					<span class="avue-crud__tip-name">
       					 当前表格已选择
        			<span class="avue-crud__tip-count">{{selections.length}}</span>
       				 	项
      				</span>
      				<el-button type="text" size="small" @click="clearSelections">清空</el-button>
				</div>
				<div class="refresh_btn">
					<el-button icon="el-icon-refresh" circle size="small" @click="handleRefresh"></el-button>
					<el-button icon="el-icon-menu" circle size="small" @click="handleEditColumn"></el-button>
        </div>-->
        <el-table
          ref="multipleTable"
          :data="tableData"
          :stripe="tableOption.stripe"
          :border="tableOption.border"
          highlight-current-row
          @sort-change="handleSortChange"
          @cell-click="handleCellClick"
          @selection-change="handleSelectionChange"
          style="width: 100%"
        >
          <template v-for="(column, index) in tableOption.columns">
            <!--slot 添加自定义配置项-->
            <!--<slot :name="column.prop" v-if="column.slot"></slot>-->
            <el-table-column
              v-if="column.slot && (column.display==null || column.display)"
              :sortable="column.sortable"
              :key="index"
              :label="column.label"
              :width="column.width"
              :prop="column.prop"
            >
              <template slot-scope="scope">
                <slot :name="column.prop" :row="scope.row"></slot>
              </template>
            </el-table-column>
            <el-table-column
              v-else-if="column.type=='img'&& (column.display==null || column.display)"
              :sortable="column.sortable"
              :key="column.index"
              :label="column.label"
              :width="column.width"
              :type="column.type"
              :prop="column.prop"
              :formatter="column.formatter"
            >
              <template slot-scope="scope">
                <img :src="scope.row[column.prop]" v-preview="scope.row[column.prop]" height="50" />
              </template>
            </el-table-column>
            <el-table-column
              v-else-if="column.display==null || column.display"
              :sortable="column.sortable"
              :key="column.index"
              :label="column.label"
              :width="column.width"
              :type="column.type"
              :prop="column.prop"
              :formatter="column.formatter"
            ></el-table-column>
          </template>
          <slot></slot>
          <el-table-column
            label="操作"
            width="230"
            v-if="tableOption.isOperator==null || tableOption.isOperator"
          >
            <template slot-scope="scope">
              <div class>
                <el-button
                  @click="handleEdit(scope.row)"
                  type="text"
                  size="small"
                  v-if="tableOption.isEdit || tableOption.isEdit==null"
                >编辑</el-button>
                <el-button
                  @click="handleDel(scope.row.id,scope.row)"
                  type="text"
                  size="small"
                  v-if="tableOption.isDel || tableOption.isDel==null"
                >删除</el-button>
                <slot name="operateBtn" :row="scope.row"></slot>
              </div>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          layout="total, sizes, prev, pager, next, jumper"
          :total="totalSize"
          :page-sizes="pageSizes"
          :page-size="pageSize"
          @current-change="currentChange"
          @size-change="sizeChange"
          :current-page="currentPage"
        ></el-pagination>
      </div>
    </div>
    <!--打开弹窗表单-->
    <dialog-form
      ref="handleDialog"
      :isShow="isShow"
      :option="dialogOption"
      :formData="formData"
      @handleClose="handleClose"
      @handleSubmit="handleSubmit"
      @closed="closed"
    >
      <template slot="dialog-item">
        <slot name="dialog-item"></slot>
      </template>
    </dialog-form>

    <!--打开修改列弹窗-->
    <el-dialog :visible.sync="dialogVisible" title="列" v-dialogDrag>
      <el-checkbox-group id="column_checkbox" v-model="checkList" @change="handleCheckChange">
        <template v-for="(column, index) in tableOption.columns">
          <el-checkbox
		    :key="index"
            :label="column.label"
            :value="column.prop"
            v-if="column.label && column.type!='index'"
          ></el-checkbox>
        </template>
      </el-checkbox-group>
    </el-dialog>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import dialogForm from "@/views/components/dialog";
import searchForm from "@/views/components/searchForm";
//	import tablePagination from '@/views/components/table'
import common from "@/common/common";
export default {
  components: {
    dialogForm,
    //			tablePagination,
    searchForm
  },
  data() {
    return {
      formData: {},
      selections: [],
      isShow: false,
      isAdd: false,
      dialogVisible: false,
      checkList: [],
      defaultTableOption: {},
      defaultFormData: {}
    };
  },
  props: {
    searchOption: {
      type: Object,
      default: () => {}
    },
    searchData: {
      type: Object,
      default: () => {}
    },
    isReset: {
      type: Boolean,
      default: true
    },
    tableOption: {
      type: Object,
      required: true
    },
    tableData: {
      type: Array,
      default: () => []
    },
    currentPage: {
      type: Number,
      default: 1
    },
    totalSize: {
      type: Number,
      default: 0
    },
    pageSizes: {
      type: Array
    },
    pageSize: {
      type: Number
    },
    dialogOption: {
      type: Object,
      defalut: () => {}
    },
    dialogData: {
      type: Object,
      default: () => {}
    }
  },
  created() {
    this.defaultFormData = common.deepCopy(this.dialogData);
    this.formData = common.deepCopy(this.dialogData);
    this.defaultTableOption = common.deepCopy(this.tableOption.columns);
    let columns = common.deepCopy(this.tableOption.columns);
    this.checkList = columns
      .filter(ele => ele.label && ele.type != "index")
      .map(el => el.label);
  },
  watch: {
    dialogData: {
      handler(data) {
        this.defaultFormData = common.deepCopy(data);
        this.formData = common.deepCopy(data);
      },
      deep: true
    }
  },
  computed: {
    ...mapGetters(["buttons"]),
    title() {
      return this.dialogOption.name;
    }
  },
  methods: {
    //点击新增
    handleAdd() {
      this.isShow = true;
      this.isAdd = true;
      this.dialogOption.title = `新增${this.title}`;
      this.$emit("handleClickAdd", row);
    },

    //点击编辑，回显数据
    handleEdit(row) {
      this.isShow = true;
      this.isAdd = false;
      this.dialogOption.title = `编辑${this.title}`;
      this.formData = common.deepCopy(row);
      this.$emit("handleClickEdit", row);
    },

    //点击删除
    handleDel(id, row) {
      let self = this;
      this.$confirm(`此操作将永久删除该${this.title}, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$emit("handleDel", id, row);
        })
        .catch(() => {
          return false;
        });
    },

    //点击批量删除
    batchDel() {
      //验证是否勾选了列表
      if (!this.selections.length) {
        this.$message.warning("请勾选需要删除的列表");
        return false;
      }
      this.handleDel(
        this.selections.map(el => el.id).join(","),
        this.selections
      );
    },
    //点击关闭弹窗
    handleClose() {
      this.isShow = false;
      this.formData = common.deepCopy(this.defaultFormData);
      this.$emit("handleClose");
    },
    //弹窗关闭动画结束时
    closed() {
      this.$emit("closed");
    },
    //关闭弹窗方法
    closeDialog() {
      this.isShow = false;
    },
    //勾选列表
    handleSelectionChange(data) {
      this.selections = data;
      this.$emit("handleSelectionChange", data);
    },
    //清空所选的列表
    clearSelections() {
      this.$refs.multipleTable.clearSelection();
    },
    //点击确定，获取弹窗数据，进行提交
    handleSubmit(data) {
      if (this.isAdd) {
        //新增
        this.$emit("handleAdd", data);
      } else {
        //编辑
        this.$emit("handleEdit", data);
      }
    },

    //搜索
    goSearch(data) {
      this.$emit("goSearch", data);
    },
    //重置
    reset() {
      this.$emit("reset");
    },
    //点击某个单元格
    handleCellClick(row, column, cell, event) {
      this.$emit("handleCellClick", row, column, cell, event);
    },
    //排序
    handleSortChange(column, prop, order) {
      this.$emit("handleSortChange", column, prop, order);
    },
    //修改显示列
    handleEditColumn() {
      this.dialogVisible = true;
      this.$emit("handleEditColumn");
    },
    //修改列
    handleCheckChange() {
      this.tableOption.columns.forEach((ele, ind) => {
        if (
          this.checkList.includes(ele.label) ||
          ele.label == undefined ||
          ele.type == "index"
        ) {
          this.$set(ele, "display", true);
        } else {
          this.$set(ele, "display", false);
        }
      });
    },
    //刷新恢复列
    handleRefresh() {
      this.tableOption.columns = common.deepCopy(this.defaultTableOption);
      this.$emit("handleRefresh");
    },
    //导出
    handleExport() {
      this.$emit("handleExport");
    },
    //翻页
    currentChange(val) {
      this.$emit("handleCurrent", val);
    },
    //改变条数
    sizeChange(val) {
      this.$emit("handleSize", val);
    }
  }
};
</script>

<style>
#crud .safeSpot {
  background: #fff;
  margin: 20px;
}

#crud .spot_wrapper {
  padding: 20px;
}

#crud .spot_wrapper .spot_add {
  float: right;
}

#crud .safeSpot .el-table {
  padding-top: 10px;
}

#crud .safeSpot .el-table .cell {
  text-align: center;
}

#crud .safeSpot .el-table th {
  color: #000;
  font-weight: 900;
  background: #fafafa;
}

/*打开新增表单*/

#crud .el-dialog {
  width: 35%;
}

#crud .el-dialog .name_input {
  width: 90%;
}
/*分页*/

#crud .el-pagination {
  margin-top: 10px;
  text-align: right;
}
#column_checkbox .el-checkbox:first-child {
  margin-left: 30px;
}
#crud .avue-crud__tip {
  float: left;
  font-size: 14px;
  text-align: center;
}
#crud .avue-crud__tip-name {
  margin-right: 10px;
}
#crud .avue-crud__tip-count {
  font-size: 16px;
  font-weight: 600;
}
#crud .refresh_btn {
  float: right;
}
#crud .avue-crud__tip .el-button {
  margin-bottom: 0;
}
</style>