<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getGoodsList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getGoodsList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddpage">添加商品</el-button>
        </el-col>
      </el-row>

      <!-- table表格区域 -->
      <el-table :data="goodlist" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column
          label="商品价格(元)"
          prop="goods_price"
          width="95px"
        ></el-table-column>
        <el-table-column
          label="商品重量"
          prop="goods_weight"
          width="70px"
        ></el-table-column>
        <el-table-column label="创建时间" prop="add_time" width="140px">
          <template slot-scope="scope">
            {{ scope.row.add_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="editDialogVisibleShow(scope.row.goods_id)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeById(scope.row.goods_id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
      >
      </el-pagination>
    </el-card>

    <!-- 编辑区域 -->
    <el-dialog
      title="修改商品"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form
        :model="editForm"
        :rules="editFormRoles"
        ref="editFormRef"
        label-width="100px"
      >
        <el-form-item label="商品名称" prop="goods_name">
          <el-input v-model="editForm.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="商品价格(元)" prop="goods_price">
          <el-input v-model="editForm.goods_price"></el-input>
        </el-form-item>
        <el-form-item label="商品重量" prop="goods_weight">
          <el-input v-model="editForm.goods_weight"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editFormChange">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    var checkNumber = (rule, value, cb) => {
      const regNumber = /^([1-9][0-9]*)+(.[0-9]{1,2})?$/;
      if (regNumber.test(value)) {
        return cb();
      }
      cb(new Error("请输入数字(最多两位小数)"));
    };
    return {
      // 查询参数对象
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10,
      },
      // 商品列表
      goodlist: [],
      // 总数据条数
      total: 0,
      // 编辑菜单显示
      editDialogVisible: false,
      // 编辑信息
      editForm: {
        goods_name: "",
        goods_price: "",
        goods_weight: "",
        goods_cat: "",
        goods_number: "",
      },
      // 编辑菜单验证规则
      editFormRoles: {
        goods_name: [
          { required: true, message: "请输入参数名称", trigger: "blur" },
        ],
        goods_price: [
          { required: true, message: "请输入参数名称", trigger: "blur" },
          { validator: checkNumber, trigger: "blur" },
        ],
        goods_weight: [
          { required: true, message: "请输入参数名称", trigger: "blur" },
          { validator: checkNumber, trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getGoodsList();
  },
  methods: {
    // 根据分页获取对应的商品列表
    async getGoodsList() {
      const { data: res } = await this.$http.get("goods", {
        params: this.queryInfo,
      });

      if (res.meta.status !== 200) {
        return this.$message.error("获取数据失败!");
      }
      console.log(res.data);
      this.goodlist = res.data.goods;
      this.total = res.data.total;
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getGoodsList();
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getGoodsList();
    },
    async removeById(id) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该文件, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      if (confirmResult !== "confirm") {
        return this.$message.info("已经取消删除!");
      }
      const { data: res } = await this.$http.delete(`goods/${id}`);
      if (res.meta.status !== 200) {
        return this.$message.error("删除失败！");
      }
      this.$message.success("删除成功!");
      this.getGoodsList();
    },
    goAddpage() {
      this.$router.push("/goods/add");
    },
    // 编辑按钮点击事件
    async editDialogVisibleShow(id) {
      // 获得点击商品的信息
      const { data: res } = await this.$http.get(`goods/${id}`);
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品信息失败!");
      }
      this.editForm = res.data;
      this.editDialogVisible = true;
    },
    // 关闭编辑窗口数据清零
    editDialogClosed() {
      this.$refs.editFormRef.resetFields();
    },
    // 点击确定提交修改数据
    editFormChange() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.put(
          "goods/" + this.editForm.goods_id,
          {
            goods_name: this.editForm.goods_name,
            goods_price: this.editForm.goods_price,
            goods_weight: this.editForm.goods_weight,
            goods_cat: this.editForm.goods_cat,
            goods_number: this.editForm.goods_number,
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("更新商品信息失败!");
        }

        this.editDialogVisible = false;
        this.getGoodsList();
        this.$message.success("商品信息更新成功!");
      });
    },
  },
};
</script>

<style lang="less" scoped>
.el-table {
  margin: 15px 0;
}
</style>

