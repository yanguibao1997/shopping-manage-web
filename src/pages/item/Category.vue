<template>
  <v-card>
      <v-flex xs12 sm12>
        <v-tree url="/item/category/list"
                :isEdit="isEdit"
                @handleAdd="handleAdd"
                @handleEdit="handleEdit"
                @handleDelete="handleDelete"
                @handleClick="handleClick"
        />
      </v-flex>
  </v-card>
</template>

<script>
  export default {
    name: "category",
    data() {
      return {
        isEdit:true
      }
    },
    methods: {
      handleAdd(node) {
        // cab("123")
        // console.log("12122");
        // 后台添加
        this.$http.post("/item/category/addcategory",this.$qs.stringify(node)).then(res => {
/*          cab(res.data.id);*/
          this.$message.success("添加成功");
        }).catch( error => {
          this.$message.error("添加失败");
        })
      },
      handleClick(node) {
        console.log(node)
      },
      handleEdit(id,name){
        this.$http.put("/item/category/updatecategory?id="+id+"&name="+name).then(res => {
          this.$message.success(res.data);
        }).catch(error => {
          this.$message.error(error.data);
        });
      },
      handleDelete(id,pid){
        this.$http.delete("/item/category/deleteCategory?id="+id+"&parentId="+pid).then(res => {
          this.$message.success(res.data);
        }).catch(error => {
          this.$message.error(error.data);
        });
      }
    }
  };
</script>

<style scoped>

</style>
