<template>
  <v-card>
    <v-card-title>
      <v-btn @click="addBrand" color="primary">新增品牌</v-btn>
      <v-spacer/>
      <v-text-field
        append-icon="search"
        label="搜索"
        single-line
        hide-details
        v-model="search"
      />
    </v-card-title>
    <v-divider/>
    <v-data-table
      :headers="headers"
      :items="items"
      :pagination.sync="pagination"
      :total-items="totalItems"
      :loading="loading"
      class="elevation-1"
    >
      <!--循环体代码-->
      <template slot="items" slot-scope="props">
        <td class="text-xs-center">{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.name }}</td>
        <td class="text-xs-center"><img v-if="!!props.item.image" width="102" height="36" :src="props.item.image"/></td>
        <td class="text-xs-center">{{ props.item.letter }}</td>
        <td class="justify-center layout px-0">
          <v-btn icon @click="editBrand(props.item)">
            <i class="el-icon-edit"/>
          </v-btn>
          <v-btn icon @click="deleteBrand(props.item)">
            <i class="el-icon-delete"/>
          </v-btn>
        </td>
      </template>

      <template slot="expand" slot-scope="props">
        <v-card flat>
          <v-card-text>Peek-a-boo!</v-card-text>
        </v-card>
      </template>
      <template slot="no-data">
        <v-alert :value="true" color="error" icon="warning">
          对不起，没有查询到任何数据 :(
        </v-alert>
      </template>
      <template slot="pageText" slot-scope="props">
        共{{props.itemsLength}}条,当前:{{ props.pageStart }} - {{ props.pageStop }},
        共{{Math.ceil(props.itemsLength/props.pageStop)}}页
      </template>
    </v-data-table>

    <v-dialog v-model="show" max-width="600" scrollable v-if="show">
      <v-card>
        <v-toolbar dark dense color="primary">
          <v-toolbar-title>{{isEdit ? '修改品牌' : '新增品牌'}}</v-toolbar-title>
          <v-spacer/>
          <v-btn icon @click="show = false">
            <v-icon>close</v-icon>
          </v-btn>
        </v-toolbar>
        <v-card-text class="px-5 py-2">
          <!-- 表单组件;子组件 -->
          <brand-form :oldBrand="brand" :isEdit="isEdit" @close="show = false" :reload="getDataFromApi"/>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-card>

</template>

<script>
  import BrandForm from './BrandForm'
  import {brandData} from '../../mockDB'

  export default {
    name: "brand",
    components: {
      BrandForm
    },
    data() {
      return {
        search: '',// 搜索条件
        totalItems: 0,// 总条数
        items: [],// 表格数据
        loading: true,  //加载的进度条
        pagination: {},// 分页信息
        headers: [// 表头
          {text: 'id', align: 'center', value: 'id'},
          {text: '名称', align: 'center', sortable: false, value: 'name'},
          {text: 'LOGO', align: 'center', sortable: false, value: 'image'},
          {text: '首字母', align: 'center', value: 'letter', sortable: true,},
          {text: '操作', align: 'center', value: 'id', sortable: false}
        ],
        show: false,// 是否弹出窗口
        brand: {}, // 品牌信息
        isEdit: false // 判断是编辑还是新增
      }
    },
    watch: {
      pagination: {
        handler() {
          this.getDataFromApi();
        },
        deep: true
      },
      search: {
        handler() {
          this.getDataFromApi();
        }
      },
      show(val, oldVal) {
        // 表单关闭后重新加载数据
        if (oldVal) {
          this.getDataFromApi();
        }
      }
    },
    mounted() {//生命周期函数调用查询方法
      this.getDataFromApi();
    },
    methods: {
      addBrand() {
        this.brand = {};
        this.isEdit = false;
        this.show = true;
      },
      editBrand(item) {
        this.brand = item;
        this.isEdit = true;
        this.show = true;
        // 查询商品分类信息，进行回显
        this.$http.get("/item/category/brand/add" + item.id)
          .then(resp => {
            this.brand.categories = resp.data;
          })

      },
      deleteBrand(item) {
        this.$message.confirm('此操作将永久删除该品牌, 是否继续?').then(() => {
          // 发起删除请求
          this.$http.delete("/item/brand/delete?id=" + item.id,)
            .then(() => {
              // 删除成功，重新加载数据
              this.$message.success("删除成功！");
              this.getDataFromApi();
            })
        }).catch(() => {
          this.$message.info("删除已取消！");
        });

      },
      /**
         * pagination:Object
             descending:false
             page:1
             rowsPerPage:5
             sortBy:"id"
             totalItems:0
           search:""
       */
      getDataFromApi() {//查询
        //this.loading = true;//是否开启进度条
        this.$http.get('/item/brand/page',
          {
            params:{
              page:this.pagination.page,//分页
              row:this.pagination.rowsPerPage,//一页显示多少条
              searchKey:this.search,//搜索条件
              sortBy:this.pagination.sortBy,//根据谁降序或升序
              desc:this.pagination.descending//降序或升序
            }
          }
        ).then(response => {
          console.log(response)
          //成功后,将值赋给前段页面显示
          this.items = response.data.items;//设置当前页数据,或者一页的数据
          this.totalItems = response.data.total;//设置总条数,表格里数据显示的总条数 等于数据库的数据条数; 分页

          this.loading = false;//是否开启进度条;数据加载完毕关闭进度条
        })

        /*getDataFromApi() {
        this.loading = true;
        // 200ms后返回假数据
        window.setTimeout(() => {
          this.items = brandData.slice(0,4);//slice():选取元素,选取从 start(开始) 到数组结尾的所有元素。
          this.totalItems = 100 //表格里数据显示的总条数
          this.loading = false;
        }, 200)*/
      }
    }
  }
</script>

<style scoped>

</style>
