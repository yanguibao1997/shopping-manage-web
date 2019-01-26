<template>
    <div>
        <v-data-table
        :headers="headers"
        :items="groups"
        hide-actions
        select-all
        v-model="selected"
        class="elevation-0"
        >
            <template slot="items" slot-scope="props">
                <tr @click="selectGroup(props.item)">
                    <td>
                      <v-checkbox
                        v-model="props.selected"
                        primary
                        hide-details
                      ></v-checkbox>
                    </td>
                    <td class="text-xs-center">{{ props.item.id }}</td>
                    <td class="text-xs-center">{{ props.item.name }}</td>
                    <td class="justify-center layout px-0">
                    <v-btn icon @click.stop="editGroup(props.item)">
                        <i class="el-icon-edit"/>
                    </v-btn>
                    <v-btn icon @click.stop="deleteGroup(props.item.id)">
                        <i class="el-icon-delete"/>
                    </v-btn>
                    </td>
                </tr>
            </template>
            <template slot="no-data">
                该分类下暂无规格组或尚未选择分类
            </template>
        </v-data-table>

        <v-btn color='primary' @click="addGroup" outline round>新增分组</v-btn>
        <v-btn color="primary" @click="deleteSomeGroup" outline round>删除分组</v-btn>
        <v-dialog v-model="show" width="300" height="200" persistent>
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>{{isEdit ? '修改' : '新增'}}分组</v-toolbar-title>
          <v-spacer/>
          <!--关闭窗口的按钮-->
          <v-btn icon @click="show=false"><v-icon>close</v-icon></v-btn>
        </v-toolbar>
        <v-card>
            <v-card-text>
                <v-text-field label="分组名称：" v-model="group.name"  />
            </v-card-text>
            <v-card-actions>
                <v-layout class="my-3" row>
                  <v-btn class="ml-5" @click.native="show=false" color="primary"  round>取消</v-btn>
                  <v-spacer class="px-1"/>
                  <v-btn class="mr-5" @click.native="save" round>保存</v-btn>
                </v-layout>
            </v-card-actions>
        </v-card>
        </v-dialog>
    </div>
</template>

<script>
export default {
  name: "spec-group",
  props: {
    cid: {
      type: Number,
      default:0,
    },
  },
  data() {
    return {
      groups:[],
      headers: [
        { text: "id", value: "id", align: "center", sortable: false },
        { text: "组名", value: "name", align: "center", sortable: false },
        { text: "操作", align: "center", sortable: false }
      ],
      show: false, // 是否打开编辑窗口
      group:{name:''},
      isEdit: false, // 是否是编辑
      selected:[]
    };
  },
  watch:{
      cid(){
          this.loadData();
      }
  },
  methods:{
      loadData(){
          this.$http.get("/item/specification/querySpecGroupByCid/" + this.cid).then(({data}) => {
              this.groups = data;
          }).catch(() => {
              this.groups = [];
          })
      },
      editGroup(group){
          Object.assign(this.group, group);
          this.show = true;
          this.isEdit = true;
      },
      addGroup(){
          this.group = {cid:this.cid};
          this.show = true;
          this.isEdit = false;
      },
      save(){
           this.$http({
            method: this.isEdit ? 'put' : 'post',
            url: '/item/specification/addOrUpdateSpecGroup',
            data: this.$qs.stringify(this.group)
          }).then(() => {
            // 关闭窗口
            this.show = false;
            this.$message.success("保存成功！");
            this.loadData();
          }).catch(() => {
              this.$message.error("保存失败！");
          });
      },
      deleteSomeGroup(){
        if(this.selected.length>0) {
          this.$message.confirm('此操作将永久删除数据，是否继续?', '提示', {
            confirmButtonText: '确定删除',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            /*for(const i in this.selected){
              // console.log(i+"========================="+this.selected[i].id);

            ids.join(",")
          }*/
            const ids = this.selected.map(s => s.id).join(",");

            this.$http.delete("/item/specification/deleteSpecGroup?ids=" + ids).then(() => {
              this.selected=[];
              this.loadData();
              this.$message.success("删除成功");
            }).catch(() => {
              this.$message.error("删除失败");
            });
          }).catch(() => {
            this.$message.info('已取消删除');
          })
        }else{
          this.$message.info("至少选择一个");
        }
      },
      deleteGroup(id){
          this.$message.confirm("确认要删除分组吗？").then(() => {
            this.$http.delete("/item/specification/deleteSpecGroup?ids=" + id)
                .then(() => {
                    this.$message.success("删除成功");
                    this.loadData();
                })
          })
      },
      selectGroup(group){
          this.$emit("select", group);
      }
  }
};
</script>
