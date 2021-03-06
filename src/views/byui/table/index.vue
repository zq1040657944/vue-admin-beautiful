<template>
  <div class="table-container">
    <byui-query-form>
      <byui-query-form-left-panel>
        <el-button icon="el-icon-plus" type="primary" @click="handleAdd"
          >添加
        </el-button>
        <el-button icon="el-icon-delete" type="danger" @click="handleDelete"
          >删除
        </el-button>
        <el-button type="primary" @click="testMessage">baseMessage</el-button>
        <el-button type="primary" @click="testALert">baseAlert</el-button>
        <el-button type="primary" @click="testConfirm">baseConfirm</el-button>
        <el-button type="primary" @click="testNotify">baseNotify</el-button>
      </byui-query-form-left-panel>
      <byui-query-form-right-panel>
        <el-form
          ref="form"
          :model="queryForm"
          :inline="true"
          @submit.native.prevent
        >
          <el-form-item>
            <el-input v-model="queryForm.title" placeholder="标题" />
          </el-form-item>
          <el-form-item>
            <el-button
              icon="el-icon-search"
              type="primary"
              native-type="submit"
              @click="handleQuery"
              >查询
            </el-button>
          </el-form-item>
        </el-form>
      </byui-query-form-right-panel>
    </byui-query-form>

    <el-table
      ref="tableSort"
      v-loading="listLoading"
      :data="list"
      :element-loading-text="elementLoadingText"
      @selection-change="setSelectRows"
      @sort-change="tableSortChange"
    >
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column label="序号" width="95">
        <template slot-scope="scope">
          {{ scope.$index + 1 }}
        </template>
      </el-table-column>
      <el-table-column prop="title" label="标题"></el-table-column>
      <el-table-column label="点击量" prop="pageViews" sortable>
      </el-table-column>
      <el-table-column label="作者" prop="author"></el-table-column>
      <el-table-column label="头像">
        <template slot-scope="scope">
          <el-image
            v-if="imgShow"
            :preview-src-list="imageList"
            :src="scope.row.img"
          ></el-image>
        </template>
      </el-table-column>
      <el-table-column
        label="点击量"
        prop="pageViews"
        sortable
      ></el-table-column>
      <el-table-column class-name="status-col">
        <template slot-scope="scope">
          <el-tooltip
            :content="scope.row.status"
            class="item"
            effect="dark"
            placement="top-start"
          >
            <el-tag :type="scope.row.status | statusFilter"
              >{{ scope.row.status }}
            </el-tag>
          </el-tooltip>
        </template>
      </el-table-column>
      <el-table-column
        label="时间"
        prop="datetime"
        width="200"
      ></el-table-column>
      <el-table-column label="操作" width="180px" fixed="right">
        <template slot-scope="scope">
          <el-button type="text" @click="handleEdit(scope.row)"
            >编辑
          </el-button>
          <el-button type="text" @click="handleDelete(scope.row)"
            >删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      :background="background"
      :current-page="queryForm.pageNo"
      :layout="layout"
      :page-size="queryForm.pageSize"
      :total="total"
      @current-change="handleCurrentChange"
      @size-change="handleSizeChange"
    ></el-pagination>
    <edit ref="edit"></edit>
  </div>
</template>

<script>
import { getList, doDelete } from "@/api/table";
import Edit from "./components/edit";
export default {
  name: "Table",
  components: {
    Edit,
  },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: "success",
        draft: "gray",
        deleted: "danger",
      };
      return statusMap[status];
    },
  },
  data() {
    return {
      imgShow: true,
      list: [],
      imageList: [],
      listLoading: true,
      layout: "total, sizes, prev, pager, next, jumper",
      total: 0,
      background: true,
      selectRows: "",
      elementLoadingText: "正在加载...",
      queryForm: {
        pageNo: 1,
        pageSize: 20,
        title: "",
      },
    };
  },
  created() {
    this.fetchData();
  },
  beforeDestroy() {},
  mounted() {},
  methods: {
    tableSortChange() {
      const imageList = [];
      this.$refs.tableSort.tableData.forEach((item, index) => {
        imageList.push(item.img);
      });
      this.imageList = imageList;
    },
    setSelectRows(val) {
      this.selectRows = val;
    },
    handleAdd() {
      this.$refs["edit"].showEdit();
    },
    handleEdit(row) {
      this.$refs["edit"].showEdit(row);
    },
    handleDelete(row) {
      if (row.id) {
        this.$baseConfirm("你确定要删除当前项吗", null, () => {
          doDelete({ ids: row.id }).then((res) => {
            this.$baseMessage(res.msg, "success");
            this.fetchData();
          });
        });
      } else {
        if (this.selectRows.length > 0) {
          const ids = this.selectRows.map((item) => item.id).join();
          this.$baseConfirm("你确定要删除选中项吗", null, () => {
            doDelete({ ids: ids }).then((res) => {
              this.$baseMessage(res.msg, "success");
              this.fetchData();
            });
          });
        } else {
          this.$baseMessage("未选中任何行", "error");
          return false;
        }
      }
    },
    handleSizeChange(val) {
      this.queryForm.pageSize = val;
      this.fetchData();
    },
    handleCurrentChange(val) {
      this.queryForm.pageNo = val;
      this.fetchData();
    },
    handleQuery() {
      this.queryForm.pageNo = 1;
      this.fetchData();
    },
    fetchData() {
      this.listLoading = true;
      getList(this.queryForm).then((res) => {
        this.list = res.data;
        const imageList = [];
        res.data.forEach((item, index) => {
          imageList.push(item.img);
        });
        this.imageList = imageList;
        this.total = res.totalCount;
        setTimeout(() => {
          this.listLoading = false;
        }, 500);
      });
    },
    testMessage() {
      this.$baseMessage("test1", "success");
    },
    testALert() {
      this.$baseAlert("11");
      this.$baseAlert("11", "自定义标题", () => {
        /* 可以写回调; */
      });
      this.$baseAlert("11", null, () => {
        /* 可以写回调; */
      });
    },
    testConfirm() {
      this.$baseConfirm(
        "你确定要执行该操作?",
        null,
        () => {
          /* 可以写回调; */
        },
        () => {
          /* 可以写回调; */
        }
      );
    },
    testNotify() {
      this.$baseNotify("测试消息提示", "test", "success", "bottom-right");
    },
  },
};
</script>
