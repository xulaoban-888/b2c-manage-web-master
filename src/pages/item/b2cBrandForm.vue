<template>
  <v-form v-model="valid" ref="brandForm">
    <v-text-field
      v-model="brand.name"
      label="请输入品牌名称"
      required
      :rules="nameRules"
    ></v-text-field>
    <v-text-field
      v-model="brand.letter"
      label="请输入品牌首字母"
      required
      :rules="letterRules"
    ></v-text-field>
    <!--展示分类信息,让品牌(选择三级联动) 使用自定义全局组件:cascader-->
    <!--分类商品的url: /item/category/list-->
    <!--- url：加载商品分类选项的接口路径
          - multiple：是否多选，这里设置为true，因为一个品牌可能有多个分类
          - requried：是否是必须的，这里为true，会在提示上加*，提醒用户
          - v-model：关联我们brand对象的categories属性
          - label：文字说明
      -->
    <v-cascader url="/item/category/list"
                multiple
                required
                v-model="brand.categories"
                label="商品分类"
    >
    </v-cascader>
    <!--使用自定义全局组件:upload  layout:布局; flex:分块; xs3:栅格属性,就是一个页面12分,占3份-->
    <v-layout>
        <v-flex xs3>
          <span style="font-size: 16px; color: #444">请选择logo:</span>
        </v-flex>
        <v-flex xs9>
            <v-upload   v-model="brand.image"
                        url="/upload/image"
                        required
                        :multiple="false"
                        :pic-width="250"
                        :pic-height="90"
            >
            </v-upload>
        </v-flex>
    </v-layout>

    <!--按钮-->
    <v-layout>
      <v-flex xs6></v-flex>

      <v-flex xs6>
        <v-btn @click="submit" small color="primary">提交</v-btn>
        <v-btn @click="clear" small color="warning">重置</v-btn>
      </v-flex>
    </v-layout>
  </v-form>
</template>

<script>
    export default {
        name: "b2cBrandForm",
        props:{//用props属性接收父组件的参数
          //获取父组件的参数就是要回显的品牌数据(这时要监控:修改不同的数据,回显不同的信息)
          oldBrand:{
            type:Object//参数类型
          },
          isEdit:{//修改状态,true:修改; false:增加
            type:Boolean,//参数类型
            default:false
          }
        },
        data(){
          return{
            valid: false,
            brand: {
              name: "",
              letter: "",
              image:"",
              categories: []
            },
            nameRules: [
              v => !!v || '品牌名称不能为空',
              v => /^[a-zA-Z0-9_\u4e00-\u9fa5()（）]{2,20}$/.test(v)  || '品牌名称长度为2-20位',
            ],
            letterRules: [
              v => !!v || '品牌首字母不能为空',
              v => /^[A-Z]{1}$/.test(v) || '品牌首字母只能是一位并大写',
            ]
          }
        },
      watch:{//监控属性值
          oldBrand:{//监控:修改不同的数据,回显不同的信息,值的变化
            deep:true,//是否生效
            handler(){
              //将oldBrand的中的数据回显到表单上
              /* 第一种回显
              this.brand.name = this.oldBrand.name;
              this.brand.letter = this.oldBrand.letter;
              this.brand.image = this.oldBrand.image;
              this.brand.id = this.oldBrand.id;
              //分类回显
              this.brand.categories = this.oldBrand.categories;
              */
              //第二种回显
              this.brand = this.oldBrand;
            }
          },
          "brand.letter":{
            deep:true,
            handler(){
              //console.log("brand,变化啦");
              //toLocaleUpperCase把首字母转换成大写
              //toUpperCase把首字母转换成大写
              if(this.brand.letter != null){
                this.brand.letter = this.brand.letter.toLocaleUpperCase();
              }

            }
          }
      },
      methods:{
          submit(){//数据校验 ,当数据都合格之后,提交到后台
            //验证表单都不为空,请求后台
            if(this.$refs.brandForm.validate()){
              //传参封装
              let param={};
              param.name = this.brand.name;
              param.letter = this.brand.letter;
              param.image = this.brand.image;
              let cids = this.brand.categories.map(c=> c.id).join(",");
              param.cids = cids;

              if(this.isEdit){//true:修改
                param.id = this.brand.id;
                //使用工具将json参数转换为 ? & = 的方式
                this.$http.put('/item/brand',this.$qs.stringify(param)).then(response=>{
                  //组件内的通讯,父向子  子向父
                  //子组件调用父组件的传过来的函数,关闭窗口,刷新页面数据
                  this.$emit("close");
                  //成功弹框提示
                  this.$message.success("修改成功");
                  this.clear();//清空表单
                }).catch(error=>{
                  //失败弹框提示
                  this.$message.error("提交表单失败");
                })
              }else{//新增
                  //使用工具将json参数(如:key:"华为")转换为 ?key=华为&page=1 的格式(json字符串转化json对象)
                  this.$http.post('/item/brand',this.$qs.stringify(param)).then(response=>{
                    //组件内的通讯,父向子  子向父
                    //子组件调用父组件的传过来的函数,关闭窗口,刷新页面数据
                    this.$emit("close");
                    //成功弹框提示
                    this.$message.success("增加成功");
                    this.clear();//清空表单
                  }).catch(error=>{
                    //失败弹框提示
                    this.$message.error("提交表单失败");
                  })
                }
              }
          },
          clear(){//重置 (增加或修改) 清空brandForm表单
            //1.只能清空brandForm表单中原有标签的值
            this.$refs.brandForm.reset();
            //2.自定义标签要手动清除
            this.brand.categories=[];//清空分类
            this.brand.image= "";
            this.brand.id="";
          }
      }

    }
</script>

<style scoped>

</style>
