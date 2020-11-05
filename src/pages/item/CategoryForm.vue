<template>
  <v-form v-model="valid" ref="categoryForm">
    <v-text-field
      label="品牌名称"
      v-model="category.name"
      :rules="[v => !!v || '品牌名称不能为空']"
      :counter="10"
      required
    />
    <v-layout class="my-4">
      <v-btn color="blue darken-1" text @click="closeWindow">Close</v-btn>
      <v-btn color="blue darken-1" text @click="submit">Save</v-btn>
    </v-layout>
  </v-form>
</template>

<script>
  export default {
    name: "CategoryForm",
    props: {
      oldCategory: Object,
      isEdit2: {
        type: Boolean,
        default: false
      },
      show: {
        type: Boolean,
        default: true
      }
    },
    data() {
      return {
        valid:false,
        category: {
          name: this.oldCategory.name
        }
      }
    },
    watch: {
      oldCategory:{
        deep:true,
        handler(val){
          Object.deepCopy(val,this.category);
        }
      }
    },
    methods: {
      submit() {
        this.oldCategory.name = this.category.name;
        // console.log(this)
        // 表单校验
        if (this.$refs.categoryForm.validate()) {
          // 将数据提交到后台
          this.$http({
            method: this.isEdit2 ? 'put' : 'post',
            url: '/item/category/save',
            data: this.oldCategory
          }).then(() => {
            // 关闭窗口
            this.$message.success("保存成功！");
            this.closeWindow();

          }).catch(() => {
            this.$message.error("保存失败！");
          });
        }
      },
      closeWindow(){
        this.$emit("close");
      }
    }
  }
</script>

<style scoped>

</style>
