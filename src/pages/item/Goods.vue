<template>
  <v-card>
    <v-toolbar class="elevation-0">
      <v-btn @click="addItem" color="primary">新增商品</v-btn>
      <v-spacer/>
      <v-flex xs3>
        状态：
        <v-btn-toggle mandatory v-model="search.saleable">
          <v-btn flat :value="2">
            全部
          </v-btn>
          <v-btn flat :value="1">
            上架
          </v-btn>
          <v-btn flat :value="0">
            下架
          </v-btn>
        </v-btn-toggle>
      </v-flex>
      <v-flex xs3>
        <v-text-field
          append-icon="search"
          label="搜索"
          single-line
          hide-details
          v-model="search.key"
        />
      </v-flex>
    </v-toolbar>
    <v-divider/>
    <v-data-table
      :headers="headers"
      :items="items"
      :pagination.sync="pagination"
      :total-items="totalItems"
      :loading="loading"
      class="elevation-1"
    >
      <template slot="items" slot-scope="props">
        <td class="text-xs-center">{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.title }}</td>
        <td class="text-xs-center">{{ props.item.categoryName}}</td>
        <td class="text-xs-center">{{ props.item.brandName }}</td>
        <td class="justify-center layout px-0">
          <v-btn icon small @click="editItem(props.item)">
            <i class="el-icon-edit"/>
          </v-btn>
          <v-btn icon small @click="deleteItem(props.item.id)">
            <i class="el-icon-delete"/>
          </v-btn>
          <v-btn icon small @click="editSaleable(props.item)" v-if="props.item.saleable">下架</v-btn>
          <v-btn icon @click="editSaleable(props.item)" v-else>上架</v-btn>
        </td>
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

    <v-dialog v-if="show" v-model="show" max-width="900" scrollable persistent>
      <v-card>
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>{{isEdit ? '修改商品' : '新增商品'}}</v-toolbar-title>
          <v-spacer/>
          <v-btn icon @click="closeForm">
            <v-icon>close</v-icon>
          </v-btn>
        </v-toolbar>
        <v-card-text style="height: 600px;">
          <!-- 表单 -->
          <goods-form :oldGoods="selectedGoods" :isEdit="isEdit" :step="step" ref="goodsForm" @closeForm="closeForm"/>
        </v-card-text>
        <v-card-actions>
          <v-layout row justify-center>
            <v-flex xs2>
              <v-btn :disabled="step === 1" @click="lastStep">上一步</v-btn>
            </v-flex>
            <v-flex xs2>
              <v-btn :disabled="step === 4" color="primary" @click="nextStep">下一步</v-btn>
            </v-flex>
          </v-layout>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
  import GoodsForm from './GoodsForm'
  import {goodsData} from "../../mockDB";

  export default {
    name: "item",
    components: {
      GoodsForm
    },
    data() {
      return {
        search: {
          key: '',
          saleable: 1,
        },// 过滤字段
        totalItems: 0,// 每页总条数
        items: [],// 表格数据
        loading: true,
        pagination: {},// 分页信息
        headers: [// 表头
          {text: 'id', align: 'center', value: 'id'},
          {text: '标题', align: 'center', sortable: false, value: 'name'},
          {text: '商品分类', align: 'center', sortable: false, value: 'image'},
          {text: '品牌', align: 'center', value: 'letter', sortable: true,},
          {text: '操作', align: 'center', value: 'id', sortable: false}
        ],
        show: false,// 是否弹出窗口
        selectedGoods: null, // 选中的商品信息
        isEdit: false, // 判断是编辑还是新增
        step: 1// 表单中的导航条
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
        },
        deep: true
      }
    },
    mounted() {//钩子函数,生命周期函数
      this.getDataFromApi();
    },
    methods: {
      closeForm() {
        this.isEdit = false;
        this.show = false;
        this.step = 1;
        this.selectedGoods = null;
        this.getDataFromApi();
      },
      lastStep() {
        if (this.step === 1) {
          return;
        }
        this.step--;
      },
      nextStep() {
        if (this.step === 4) {
          return;
        }
        if (this.$refs.goodsForm.$refs.basic.validate()) {
          this.step++;
        }
      },
      addItem() {
        this.selectedGoods = null;
        this.isEdit = false;
        this.show = true;
      },
      editItem(item) {
        //获得当前要修改的数据,赋值给selectedGoods
        this.selectedGoods = item;
        // 先得到分类名称
        const names = item.categoryName.split("/");
        // 组织商品分类数据
        this.selectedGoods.categories = [
          {id: item.cid1, name: names[0]},
          {id: item.cid2, name: names[1]},
          {id: item.cid3, name: names[2]}
        ];
        // 查询商品详情detail表回显的接口
        /* this.$http.get("/item/good/spu/detail/" + item.id)
          .then(resp => {
            this.selectedGoods.spuDetail = resp.data;
            this.isEdit = true;
            this.show = true;
          })
        //sku表回显

      },*/

        //多个axios同步请求后台  (第一种写法)
       let sendArr = [
          // 查询商品详情detail表回显的接口
          this.$http.get("/item/good/spu/detail/" + item.id),
          // 查询sku表回显的接口
          this.$http.get("/item/good/skuList/" + item.id)
        ]
        this.$http.all(sendArr).then(this.$http.spread((detail, sku) => {
            //多个函数同时请求,分别赋值 1.spuDetail商品详情赋值 2.sku集合赋值
            this.selectedGoods.spuDetail = detail.data;
            this.selectedGoods.skus = sku.data;
            //console.log(this.selectedGoods);
            //成功后点击修改,展示form表单
            this.isEdit = true;
            this.show = true;
          })
        )
        // 查询商品详情 (第二种写法)
       /* this.$http.get("/item/good/detail/" + item.id)
          .then(resp => {
            this.selectedGoods.spuDetail = resp.data.spuDetail;
            this.selectedGoods.skus = resp.data.skus;
            console.log(this.selectedGoods);
          })
        this.isEdit = true;
        this.show = true;*/
      },
      deleteItem(id) {
        this.$message.confirm('此操作将永久删除该商品, 是否继续?')
          .then(() => {
            // 发起删除请求
            this.$http.delete("/item/good?id=" + id)
              .then(() => {
                // 删除成功，重新加载数据
                this.getDataFromApi();
                //清空搜索
                this.search.key="";
                this.$message.info('删除成功!');
              })
          })
          .catch(() => {
            this.$message.info('已取消删除');
          });

      },
      //上下架  第一种写法,传实体
      editSaleable(item){
        this.$http.put("/item/good/saleable/", item).then(()=>{
          this.getDataFromApi();
        })
      },
      //上下架  第二种写法 传id
      /*editSaleable(id){
       this.$http.put("/item/good/saleable/"+id).then(()=>{
         this.getDataFromApi();
       })
     },*/
   getDataFromApi() {
     this.$http.get("/item/good/page",
       {
         params:{
           page:this.pagination.page,//分页
           rows:this.pagination.rowsPerPage,//一页显示多少条
           search:this.search.key.trim(),//搜索条件
           saleable: this.search.saleable,//上架:1或下架:0
         }
       }).then(response=>{
         //console.log(response)
         this.items = response.data.items;//设置当前页数据,或者一页的数据
         this.totalItems = response.data.total;//设置总条数
         this.loading = false;//进度条是否执行

     })
    /*setTimeout(() => {
       // 返回假数据
       this.items = goodsData.slice(0, 4);
       this.totalItems = 25;
       this.loading = false;
     }, 300)*/
      }
    }
  }
</script>

<style scoped>
  label {
    font-size: 14px;
  }
</style>
