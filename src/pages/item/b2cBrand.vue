<template>
    <v-card>
      <v-card-title>
        <v-btn @click="addBrand" color="primary">新增品牌</v-btn>
        <v-spacer></v-spacer>
        <v-text-field
          label="搜索"
          single-line
          hide-details
          append-icon="search"
          v-model="search"
        ></v-text-field>
      </v-card-title>

      <v-data-table
        :headers="headers"
        :items="desserts"
        :pagination.sync="options"
        :total-items="totalDesserts"
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
        <!--没有数据提示信息-->
        <template slot="no-data">
          <v-alert :value="true" color="error" icon="warning">
            对不起，没有查询到任何数据 :(
          </v-alert>
        </template>
        <!--分页显示总条数,一页几条数据,共分多少页-->
        <template slot="pageText" slot-scope="props">
          共{{props.itemsLength}}条,当前:{{ props.pageStart }} - {{ props.pageStop }},
          共{{Math.ceil(props.itemsLength/props.pageStop)}}页
        </template>
      </v-data-table>

      <!--弹出的对话框  persistent :反复出现 -->
      <v-dialog
        persistent
        v-model="show"
        max-width="500"
      >
        <!--标题,按钮,form表单-->
        <v-card>
          <!--对话框的标题-->
          <v-toolbar dense dark color="primary">
            <v-toolbar-title>
              {{isEdit ? '修改品牌' : '新增品牌'}}
            </v-toolbar-title>

              <v-spacer></v-spacer><!--spacer 左右分割,  icon:图标-->
            <!--关闭表单按钮 两种方法-->
            <!--1. @click="show = false" 2.调用方法-->
            <v-btn icon @click="colseBrand">
              <v-icon>close</v-icon>
            </v-btn>
          </v-toolbar>

            <!--调用表单组件;或子组件-->
            <v-card-text>
                <b2cBrandForm :isEdit="isEdit" :oldBrand="brand" @close="addBrandSuccess"></b2cBrandForm>
            </v-card-text>
        </v-card>
      </v-dialog>
    </v-card>
</template>

<script>
  import b2cBrandForm from "./b2cBrandForm"
    export default {
        name: "b2cBrand",
      data () {
        return {
          search: '',// 搜索条件
          totalDesserts: 0,//查询总条数
          desserts: [],//表格一页的数据
          loading: true,//进度条是否执行
          options: {},//分页
          headers: [
            {
              text: 'id',//表格字段名
              align: 'start',//对齐方式
              sortable: false,//是否开启排序
              value: 'id',//后台实体字段名
            },
            { text: '名称',align: 'start',sortable: false, value: 'name' },
            { text: 'LOGO',align: 'start', sortable: false,value: 'image' },
            { text: '首字母',align: 'start', value: 'letter' },
            { text: '操作',align: 'start',sortable: false, value: 'id' },
          ],
          show: false,//模态框是否弹出
          brand:{}, // 品牌信息修改回显,传递到子组件的参数
          isEdit:false//false:新增;true:修改
        }
      },
      watch: { //监控 属性值发生变化就触发
        options: {//监控options中的属性;查询传的参数
          deep: true,//是否生效
          handler() {//处理
            //当options中的属性发生变化,向后台发送请求,进行查询
            //当分页发生变化,排序发生变化,向后台发送请求,进行查询
            this.getDataFromApi();
          }
        },
        search: {//监控 搜索值 发生变化就触发;监控 搜索框
          deep: true,//是否生效
          handler() {
            //当搜索值发生变化,向后台发送请求,进行like模糊查询
            //console.log("当搜索值发生变化,向后台发送请求,进行like模糊查询")
            //调用查询的方法
            this.getDataFromApi();
          }
        }

      },
      components:{//局部组件,form表单
        b2cBrandForm
      },
      mounted () {//挂载后,钩子函数,生命周期函数最后
        //生命周期函数调用查询方法
        this.getDataFromApi();
      },
      methods: {
        colseBrand(){//表单 x号关闭表单
           this.show = false;//关闭窗口
        },
        addBrand(){//点击新增按钮 打开模态框
          this.show = true;//弹出窗口
          this.isEdit = false;//false:新增操作
          //清空表单 如果修改form的数据,中途放弃,又改为增加,此时在增加前清空form表单
          this.brand={};
          //2.自定义标签要手动清除
          this.brand.categories=[];//清空分类
          this.brand.image= "";
        },
        editBrand(oldBrand){//点击修改按钮,打开修改表单
          this.show = true;//弹出窗口
          this.isEdit = true;//true:修改操作
          //2.(三级联动或分类) 通过品牌id 获取分类的数据,数据需要从后台查询
          this.$http.get("/item/category/queryCategoryBrand/"+oldBrand.id)
            .then(response=>{
              //成功回调函数
              console.log(response)

              //1.将父组件的值传递到子组件或子表单;用于回显
              this.brand = oldBrand;
              //成功后,赋值,使分类回显
              this.brand.categories = response.data;
          })
        },
        deleteBrand(oldBrand){
          this.$message.confirm("此操作将永久删除该品牌, 是否继续?").then(() =>{
              // 发起删除请求
            this.$http.delete('/item/brand/delete?id='+oldBrand.id).then(()=>{
                 // console.log(response);
              this.getDataFromApi();
              // 删除成功，重新加载数据
              this.$message.success("删除成功！");
              //清空搜索
              this.search="";
            }).catch(()=>{
              this.$message.info("删除已取消！");
            })
          })
        }
        ,
        addBrandSuccess(){//后台增加成功后,关闭表单
          this.show = false;//关闭窗口
          //如果页面在搜索内容,那么后台增加成功后 清空搜索,才能进行下一步刷新
          this.search="";
          //后台增加成功后,调用查询方法,刷新分页
          this.getDataFromApi();
        },
        getDataFromApi () {//访问后台项目查询
          //写前端与后台交互,
          /**
           * options:Object
             * descending:false //降序或升序
               page:1  //分页
               rowsPerPage:5 //一页显示多少条
               sortBy:"letter" //根据谁降序或升序
               totalItems:6  //统计总数据
           **/
         this.$http.get('/item/brand/page',
              {
                params:{
                    page:this.options.page,//分页
                    row:this.options.rowsPerPage,//一页显示多少条
                    searchKey:this.search.trim(),//搜索条件
                    sortBy:this.options.sortBy,//根据谁降序或升序
                    desc:this.options.descending//降序或升序
                }
              }
          ).then(response =>{
           //成功后,将值赋给前段页面显示
          // console.log(response)
           this.desserts = response.data.items;//设置当前页数据,或者一页的数据
           this.totalDesserts = response.data.total;//设置总条数
           this.loading = false//进度条是否执行
         })
        },
        /*//有后端之后,删除此代码
        getDesserts () {
          return [
            {id:1, name: '华为', image: '', letter: 'H'},
            {id:2, name: '小米', image: 'xxx.png', letter: 'X'},
            {id:3, name: 'vivo', image: 'xxx.png', letter: 'V'},
            {id:4, name: '联想', image: 'xxx.png', letter: 'L'},
            {id:5, name: '三星', image: 'xxx.png', letter: 'S'},
            {id:6, name: 'oppo', image: 'xxx.png', letter: 'O'},
          ]
        },*/
      },
    }
</script>

<style scoped>

</style>
