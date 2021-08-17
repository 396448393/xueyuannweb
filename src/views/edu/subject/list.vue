<template>
  <div class="app-container">
    <el-input v-model="filterText" placeholder="搜索" style="margin-bottom:30px;" />

    <el-button type="text" @click="dialogFormVisible = true">添加一级分类</el-button>
    <el-tree
      ref="subjectTree"
      v-loading="loading"
      :data="subjectList"
      :props="defaultProps"
      :filter-node-method="filterNode"
      :expand-on-click-node="false"
      class="filter-tree"
      default-expand-all
      node-key="id"
    >
      <span slot-scope="{ node, data}" class="custom-tree-node">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level == 1"
            type="text"
            size="mini"
            @click="() => {dialogFormVisible = true; subject.parentId =data.id}"
          >添加二级分类</el-button>
          <el-button
            v-if="node.level == 2 || (node.level == 1 && node.childNodes.length == 0)"
            type="text"
            size="mini"
            @click="() => remove(node,data)"
          >删除</el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :visible.sync="dialogFormVisible" title="添加分类">
      <el-form :model="subject" label-width="120px">
        <el-form-item label="分类标题">
          <el-input v-model="subject.title"/>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="append()">确 定</el-button>
      </div>
    </el-dialog>

  </div>
</template>
<script>
import subject from '@/api/edu/subject'
export default {
  data() {
    return {
      filterText: '',
      subjectList: [],
      defaultProps: {
        children: 'children',
        label: 'title'
      },
      loading: false,
      dialogFormVisible: false,
      subject: {
        title: '',
        parantId: ''
      }
    }
  },
  watch: {
    filterText(val) {
      this.$refs.subjectTree.filter(val)
    }
  },
  created() {
    this.fetchNodeList()
  },
  methods: {
    fetchNodeList() {
      this.loading = true
      subject.getNestedTreeList().then(response => {
        if (response.success === true) {
          this.subjectList = response.data.items
        }
        this.loading = false
      })
    },
    filterNode(value, data) {
      if (!value) return true
      return data.title.toLowerCase().indexOf(value.toLowerCase()) !== -1
    },
    remove(node, data) {
      console.log(node)
      console.log(data)
      this.$confirm('此操作将永久删除该记录，是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        return subject.removeById(data.id)
      }).then(() => {
        this.fetchNodeList()// 刷新列表
        // this.$refs.subjectTree.remove(node)
        this.$message({
          type: 'success',
          message: '删除成功！'
        })
      }).catch((response) => {
        if (response === 'cancel') {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        } else {
          this.$message({
            type: 'error',
            message: '删除失败'
          })
        }
      })
    },
    appendLevelOne() {
      subject.saveLevelOne(this.subject).then(response => {
        this.$message({
          type: 'success',
          message: '保存成功！'
        })
        this.dialogFormVisible = false // 关闭对话框
        this.fetchNodeList() // 刷新列表
      }).catch((response) => {
        this.$message({
          type: 'error',
          message: '保存失败'
        })
      })
    },
    append(data) {
      if (!this.subject.parantId){
        this.appendLevelOne()
      } else {
        this.appendLevelTwo()
      }
    },
    appendLevelTwo() {
      subject.saveLevelTwo(this.subject).then(response => {
        this.$message({
          type: 'success',
          message: '保存成功'
        })
        this.dialogFormVisible = false
        this.fetchNodeList()
        this.subject.title = ''
        this.subject.parantId = ''
      }).catch((response => {
        this.$message({
          type: 'error',
          message: response.data.message
        })
      }))
    }
  }
}
</script>
