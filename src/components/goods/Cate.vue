<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog"
            >添加分类</el-button
          >
        </el-col>
      </el-row>

      <!-- 表格 -->
      <tree-table
        class="treeTable"
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted == false"
            style="color: lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>

        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag
            size="mini"
            v-else-if="scope.row.cat_level === 1"
            type="success"
            >二级</el-tag
          >
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>

        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button
            size="mini"
            type="primary"
            icon="el-icon-edit"
            @click="showEditDialog(scope.row)"
            >编辑</el-button
          >
          <el-button
            size="mini"
            type="danger"
            icon="el-icon-delete"
            @click="cateDelete(scope.row.cat_id)"
            >删除</el-button
          >
        </template>
      </tree-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="40%"
      @close="addCateDialogClosed"
    >
      <!-- 添加分类的表单 -->
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>

        <!-- options指定数据源 -->
        <!-- props用来指定数据对象 -->
        <el-form-item label="父级分类:">
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="{ ...cascaderProps, checkStrictly: 'true' }"
            @change="parentCateChanged"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改 -->
    <el-dialog title="修改" :visible.sync="editDialogVisbile" width="50%">
      <!-- 内容主体区域 -->
      <el-form
        :model="editForm"
        :rules="addCateFormRules"
        ref="editFormRef"
        label-width="80px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisbile = false">取 消</el-button>
        <el-button type="primary" @click="cateEdit">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      /* 查询条件 */
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5,
      },
      /* 商品分类的数据列表,默认为空 */
      catelist: [],
      /* 总数据条数 */
      total: 0,
      /* 控制添加分类对话框的隐藏显示 */
      addCateDialogVisible: false,
      /* 控制修改对话框隐藏显示 */
      editDialogVisbile: false,
      /* 添加分类的表单数据对象 */
      addCateForm: {
        /* 将要添加的分类名称 */
        cat_name: "",
        /* 父级分类id */
        cat_pid: 0,
        /* 分类的等级,默认添加一级分类 */
        cat_level: 0,
      },
      /* 父级分类列表 */
      parentCateList: [],
      /* 指定级联选择器的配置对象 */
      cascaderProps: {
        value: "cat_id",
        label: "cat_name",
        children: "children",
      },
      /* 选中父级分类的ID数组 */
      selectedKeys: [],
      /* 修改数据 */
      editForm: [],
      editCateId: "",
      /* 添加表单的验证规则对象 */
      addCateFormRules: {
        cat_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" },
        ],
      },

      /* 为table指定列的定义 */
      columns: [
        {
          label: "分类名称",
          prop: "cat_name",
        },
        {
          label: "是否有效",
          /* 当前列是否定义模板 */
          type: "template",
          /* 表示当前模板名称为isok */
          template: "isok",
        },
        {
          label: "排序",
          /* 当前列是否定义模板 */
          type: "template",
          /* 表示当前模板名称为order */
          template: "order",
        },
        {
          label: "操作",
          /* 当前列是否定义模板 */
          type: "template",
          /* 表示当前模板名称为opt */
          template: "opt",
        },
      ],
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    /* 获取商品分类数据 */
    async getCateList() {
      const { data: res } = await this.$http.get(`categories`, {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("商品分类获取失败");
      }
      this.catelist = res.data.result;
      this.total = res.data.total;
    },
    /* 监听pagesize改变 */
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.queryInfo.pagenum = 1;
      this.getCateList();
    },
    /* 监听pagenum改变 */
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getCateList();
    },
    /* 点击按钮展示添加分类对话框 */
    showAddCateDialog() {
      /* 先获取父级分类数据列表 */
      this.getParentCateList();
      this.addCateDialogVisible = true;
    },
    /* 获取父级分类的数据列表 */
    async getParentCateList() {
      const { data: res } = await this.$http.get(`categories`, {
        params: { type: 2 },
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取父级分类数据失败");
      }
      this.parentCateList = res.data;
    },
    /* 选择项变化触发这个函数 */
    parentCateChanged() {
      /* 如果selectedKeys数组里的length大于0,证明选中的父级分类 */
      /* 反之就说明没有选中任何父级 */
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid =
          this.selectedKeys[this.selectedKeys.length - 1];
        /* 为当前分类的等级赋值  */
        this.addCateForm.cat_level = this.selectedKeys.length;
        return;
      } else {
        this.addCateForm.cat_pid = 0;
        this.addCateForm.cat_level = 0;
      }
    },
    /* 添加分类 */
    addCate() {
      this.$refs.addCateFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.post(
          `categories`,
          this.addCateForm
        );
        if (res.meta.status !== 201) {
          return this.$message.error("添加分类失败");
        }
        this.$message.success("添加分类成功");
        this.getCateList();
        this.addCateDialogVisible = false;
      });
    },
    /* 监听对话框关闭时间,重置表单数据 */
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields();
      this.selectedKeys = [];
      this.addCateForm.cat_pid = 0;
      this.addCateForm.cat_level = 0;
    },
    /* 双向绑定 */
    async showEditDialog(id) {
      this.editCateId = id.cat_id;
      const { data: res } = await this.$http.get("categories/" + id.cat_id);
      /* if (res.meta.status !== 200) {
        return this.$message.error("查询信息失败");
      } */
      this.editForm = res.data;
      this.editDialogVisbile = true;
    },
    /* 编辑 */
    async cateEdit() {
      const { data: res } = await this.$http.put(
        "categories/" + this.editForm.cat_id,
        { cat_name: this.editForm.cat_name }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("更新分类数据失败!");
      }
      this.$message.success("更新分类数据成功!");
      this.getCateList();
      this.editDialogVisbile = false;
    },
    /* 删除 */
    async cateDelete(id) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该分类, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      if (confirmResult !== "confirm") {
        return this.$message.info("已取消删除");
      }
      const { data: res } = await this.$http.delete("categories/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("删除分类失败");
      }
      this.$message.success("删除分类成功");
      this.getCateList();
    },
  },
};
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>