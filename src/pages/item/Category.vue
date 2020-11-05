<template>
  <v-card>
    <v-flex xs12 sm10>
      <v-tree url="/item/category/list"

              :isEdit="isEdit"
              @handleAdd="handleAdd"
              @handleEdit="handleEdit"
              @handleDelete="handleDelete"
              @handleClick="handleClick"

      />
    </v-flex>

    <!--模态框-->
    <v-dialog v-model="show" max-width="600" scrollable v-if="show">
      <v-card>
        <v-toolbar dark dense color="primary">
          <v-toolbar-title>{{isEdit2 ? '修改品牌' : '新增品牌'}}</v-toolbar-title>
          <v-spacer/>
          <v-btn icon @click="show = false">
            <v-icon>close</v-icon>
          </v-btn>
        </v-toolbar>
        <v-card-text class="px-5 py-2">
          <!--表单-->
          <category-form :oldCategory="category" :isEdit2="isEdit2" @close="show = false"/>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
  import {treeData} from '../../mockDB'
  import CategoryForm from './CategoryForm'
  export default {
    name: "category",
    components: {
      CategoryForm
    },
    data() {
      return {
        treeData: treeData,
        isEdit:true,
        show: false,// 是否弹出窗口
        category: {}, // 分类信息
        isEdit2: false // 判断是编辑还是新增
      }
    },
    methods: {
      handleAdd(node) {
        this.category = node;
        this.isEdit2 = false;
        this.show = true;

        // console.log("add .... ");
        // console.log(category);
      },
      handleEdit(id, name) {

        // 查询商品分类信息，进行回显
        this.$http.get('/item/category/findById?id=' + id)
          .then(resp => {
            this.category=resp.data;
            this.isEdit2 = true;
            this.show = true;
          })
        console.log("edit... id: " + id + ", name: " + name)
      },
      handleDelete(id) {
        console.log("delete ... " + id);
        this.$http.delete('/item/category/remove?id=' + id).then(response => {
          console.log(response)
          //判断状态码
          //刷新
        });
      },
      handleClick(node) {
        console.log(node)
      }
    }
  };
</script>

<style scoped>

</style>
