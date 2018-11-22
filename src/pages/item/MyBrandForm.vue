<template>
  <!--对话框的内容，表单-->

  <v-form v-model="valid" ref="myBrandForm">
    <v-text-field v-model="brand.name" label="请输入品牌名称" required :rules="nameRules"/>
    <v-text-field v-model="brand.letter" label="请输入品牌首字母" required :rules="letterRules"/>
    <!--三级下拉框-->
    <v-cascader
      url="/item/category/list"
      multiple
      required
      v-model="brand.categories"
      label="请选择商品分类"/>
    <!--图片上传-->
    <v-layout row>
      <v-flex xs3>
        <span style="font-size: 16px; color: #444">品牌LOGO：</span>
      </v-flex>
      <v-flex>
        <v-upload
          v-model="brand.image"
          url="/upload/image"
          :multiple="false"
          :pic-width="250"
          :pic-height="90"
        />
      </v-flex>
    </v-layout>
    <!--按钮-->
    <v-layout class="my-4" row>
      <v-spacer/>
      <v-btn @click="submit" color="primary">提交</v-btn>
      <v-btn @click="clear">重置</v-btn>
    </v-layout>
  </v-form>

</template>

<script>
  export default {
    name: "MyBrandForm",
    props:{
      oldBrand:{
        type:Object,
      },
      isEdit:{
        default:false
      }
    },
    data() {
      return {
        valid: false, // 表单校验结果标记
        brand: {
          name: '', // 品牌名称
          letter: '', // 品牌首字母
          image: '',// 品牌logo
          categories: [], // 品牌所属的商品分类数组
        },
        //校验规则
        nameRules: [
          v => !!v || "品牌名称不能为空",
          v => v.length > 1 || "品牌名称至少2位"
        ],
        letterRules: [
          v => !!v || "首字母不能为空",
          v => /^[A-Z]{1}$/.test(v) || "品牌字母只能是A~Z的大写字母,且只能为一位"
        ]
      }
    },
    methods: {
      submit() {
        if (this.$refs.myBrandForm.validate()) {
          const {categories, letter, ...params} = this.brand;
          params.cids = categories.map(c => c.id).join(",");
          params.letter = letter.toUpperCase();
          // 将数据提交到后台
          this.$http({
            method: this.isEdit ? 'put' : 'post', // 动态判断是POST还是PUT
            url: '/item/brand',
            data: this.$qs.stringify(this.brand)
          })
            .then(() => {
              this.$emit("close");
              this.$message.success("提交成功");
            })
            .catch(() => {
              this.$message.error("提交失败");
            })
        }
      },

      clear() {
        this.$refs.myBrandForm.reset();
        this.brand.categories = [];
      }
    },

    watch: {
      oldBrand: {// 监控oldBrand的变化
        handler(val) {
          console.log(val)
          if(val){
            // 注意不要直接复制，否则这边的修改会影响到父组件的数据，copy属性即可
            this.brand =  Object.deepCopy(val)
          }else{
            // 为空，初始化brand
            this.brand = {
              name: '',
              letter: '',
              image: '',
              categories: [],
            }
          }
        },
        deep: true
      }
    }
  }
</script>

<style scoped>

</style>
