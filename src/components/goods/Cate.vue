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
                <el-button type="primary" @click="showCatDialog">添加分类</el-button>
            </el-col>
        </el-row>
        <tree-table class="treeTable" :data="catalist" :columns="columns" :selection-type="false"
        :expand-type="false" show-index index-text="#" border>
        <template slot="isok" slot-scope="scope">
           <i class="el-icon-success" v-if="scope.row.cat_deleted === false"
           style="color: lightgreen;"></i>
           <i class="el-icon-error" v-else style="color: red;"></i>
        </template>
        <template slot="order" slot-scope="scope">
           <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
           <el-tag size="mini" v-else-if="scope.row.cat_level === 1" type="success">二级</el-tag>
           <el-tag size="mini" v-else type="warning">三级</el-tag>
        </template>
        <template slot="opt" slot-scope="">
           <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
           <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
        </tree-table>
         <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="querInfo.pagenum"
      :page-sizes="[3, 5, 8, 10]"
      :page-size="querInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>
    <el-dialog
  title="添加分类"
  :visible.sync="catDialogVisible"
  width="50%" @close="cateClosed">
  <el-form :model="cateForm" :rules="cateFormRules" ref="cateFormRef" label-width="100px">
  <el-form-item label="分类名称" prop="cat_name">
    <el-input v-model="cateForm.cat_name"></el-input>
  </el-form-item>
  <el-form-item label="父级分类">
      <el-cascader
    v-model="selectedKeys" expandTrigger="hover"
    :options="parentCateList"
    :props="cascaderProps"
    @change="handleChange" clearable change-on-select size="mini"></el-cascader>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="catDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCate">确 定</el-button>
  </span>
</el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    return {
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      catalist: [],
      total: 0,
      columns: [{
        label: '分类名称',
        prop: 'cat_name'
      }, {
        label: '是否有效',
        type: 'template',
        template: 'isok'
      }, {
        label: '排序',
        type: 'template',
        template: 'order'
      }, {
        label: '操作',
        type: 'template',
        template: 'opt'
      }],
      catDialogVisible: false,
      cateForm: {
        cat_name: '',
        cat_id: 0,
        cat_level: 0
      },
      cateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      parentCateList: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        chilldren: 'chilldren'
      },
      selectedKeys: []
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.querInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      this.catalist = res.data.result
      this.total = res.data.total
    },
    handleSizeChange (newSize) {
      this.querInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange (newPage) {
      this.querInfo.pagenum = newPage
      this.getCateList()
    },
    showCatDialog () {
      this.getParentCateList()
      this.catDialogVisible = true
    },
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类失败')
      }
      this.parentCateList = res.data
    },
    handleChange () {
      console.log(this.selectedKeys)
      if (this.selectedKeys > 0) {
        this.cateForm.cat_pid = this.selectedKeys[ this.selectedKeys.length - 1 ]
        this.cateForm.cat_level = this.selectedKeys.length
      } else {
        this.cateForm.cat_pid = 0
        this.cateForm.cat_level = 0
      }
    },
    addCate () {
      this.$refs.cateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.cateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功')
        this.getCateList()
        this.catDialogVisible = false
      })
    },
    cateClosed () {
      this.$refs.cateFormRef.resetFields()
      this.selectedKeys = []
      this.cateForm.cat_level = 0
      this.cateForm.cat_pid = 0
    }
  }
}
</script>
<style lang="less" scoped>
  .treeTable {
      margin-top: 15px;
  }
  .el-cascader {
      width: 100%;
      height: 50%
  }
</style>
