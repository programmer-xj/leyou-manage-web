<template>

  <div>
    <v-card-text class="px-5">
      <v-dialog max-width="500" v-model="show" persistent>
        <v-card>
          <!--对话框的标题-->
          <v-toolbar dense dark color="primary">
            <v-toolbar-title>{{isEdit?"修改":"新增"}}品牌</v-toolbar-title>
            <v-spacer/>
            <!--关闭窗口的按钮-->
            <v-btn icon @click="closeWindow">
              <v-icon>close</v-icon>
            </v-btn>
          </v-toolbar>
          <my-brand-form @close="closeWindow" :oldBrand="oldBrand" :isEdit="isEdit"></my-brand-form>
        </v-card>

      </v-dialog>
    </v-card-text>
    <v-card>
      <v-card-title>
        <v-btn color="primary" @click="addBrand">新增</v-btn>
        <!--空间隔离组件-->
        <v-spacer/>
        <!--搜索框，与search属性关联-->
        <v-text-field label="输入关键字搜索" v-model="search" append-icon="search" hide-details/>
      </v-card-title>
      <!-- 分割线 -->
      <v-divider/>
      <v-data-table
        :headers="headers"
        :items="brands"
        :search="search"
        :total-items="totalBrands"
        :pagination.sync="pagination"
        :loading="loading"
        class="elevation-1"
      >
        <template slot="items" slot-scope="props">
          <td>{{ props.item.id }}</td>
          <td class="text-xs-right">{{ props.item.name }}</td>
          <td class="text-xs-right"><img :src="props.item.image"/></td>
          <td class="text-xs-right">{{ props.item.letter }}</td>
          <td class="justify-center layout">
            <v-btn color="info" @click="editBrand(props.item)">编辑</v-btn>
            <v-btn color="warning" @click="deleteBrand(props.item)">删除</v-btn>
          </td>
        </template>
      </v-data-table>
    </v-card>
  </div>
</template>

<script>

  import MyBrandForm from "./MyBrandForm"

  export default {
    name: "MyBrand",
    data() {
      return {
        search: '', // 搜索过滤字段
        totalBrands: 0, // 总条数
        brands: [], // 当前页品牌数据
        loading: true, // 是否在加载中
        pagination: {}, // 分页信息
        headers: [
          {text: 'id', align: 'center', value: 'id'},
          {text: '名称', align: 'center', sortable: false, value: 'name'},
          {text: 'LOGO', align: 'center', sortable: false, value: 'image'},
          {text: '首字母', align: 'center', value: 'letter', sortable: true,},
          {text: '操作', align: 'center', value: 'id', sortable: false,}
        ],
        show: false,
        isEdit: false,
        oldBrand: {}
      }

    },
    components: {
      MyBrandForm
    },
    watch: {
      pagination: {
        handler() {
          this.getDataFromServer()
        },
        deep: true
      },
      search() {
        this.getDataFromServer()
      }
    },
    mounted() {
      this.getDataFromServer() // 从服务的加载数的方法。

    },
    methods: {
      getDataFromServer() { // 从服务的加载数的方法。
        this.$http.get("/item/brand/page", {
          params: {
            key: this.search,
            page: this.pagination.page,
            rows: this.pagination.rowsPerPage,
            pageBy: this.pagination.sortBy,
            desc: this.pagination.descending

          }
        }).then(res => {
          this.brands = res.data.items;
          this.totalBrands = res.data.total;
          // 完成赋值后，把加载状态赋值为false
          this.loading = false;
        })

      },
      addBrand() {
        //清空数据
        this.oldBrand = null;
        this.isEdit = false;
        this.show = true;

      },
      editBrand(oldBrand) {
        this.$http.get("/item/category/bid/" + oldBrand.id)
          .then(({data}) => {
            this.isEdit = true;
            this.show = true;
            this.oldBrand = oldBrand;
            this.oldBrand.categories = data;
            console.log(data)
          })

      },
      deleteBrand() {
      },
      closeWindow() {
        this.show = false;
        // 重新加载数据
        this.getDataFromServer();
      }
    }
  }
</script>

<style scoped>

</style>
