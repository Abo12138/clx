<template>
  <div class="customer">
    <!-- 查询 -->
    <el-card class="box-card">
      <el-form :model="customerform" ref="customerform" label-width="100px">
        <el-form-item prop="name" class="name" :rules="rule1">
          <el-input v-model="customerform.name" class="nameBtn" placeholder="姓名/客户名/手机号/邮箱"></el-input>
        </el-form-item>
      </el-form>
      <el-button type="primary" @click="search()" class="addBtn">查询</el-button>
      <el-button @click="resetForm('customerform')" class="resetBtn">重置</el-button>
    </el-card>
    <!-- 客户列表 -->
    <el-card>
      <el-button type="primary" @click="addData()">新增</el-button>
      <el-button type="primary" @click="editData()">编辑</el-button>
      <el-button type="primary" @click="start()">启用</el-button>
      <el-button type="primary" @click="forbid()">禁用</el-button>
      <el-button type="primary" @click="Delete()">删除</el-button>
      <el-button type="primary" @click="resetPwd()">重置密码</el-button>
      <siri-table
        :data="customerData"
        :page="page"
        @handleSizeChange="handleSizeChange"
        @handleSelectionChange="handleSelectionChange"
        @handleCurrentChange="handleCurrentChange"
        :tableHeight="tableHeight"
      >
        <el-table-column type="selection" width="55"></el-table-column>
        <el-table-column
          class="customer-table"
          v-for="item in columnList"
          :key="item.num"
          :label="item.lable"
          :width="item.width"
          :prop="item.prop"
          :show-overflow-tooltip="true"
        ></el-table-column>
      </siri-table>
    </el-card>
    <customer-add ref="addData"></customer-add>
    <customer-edit ref="editData"></customer-edit>
  </div>
</template>
<script>
import siriTable from "../../components/SiruiTable/index";
import customerAdd from "../customerManages/CustomerAdd";
import customerEdit from "../customerManages/CustomerEdit";
export default {
  components: { siriTable, customerAdd, customerEdit },
  data() {
    var checkName = (rule, value, callback) => {
      var userName = /^[a-zA-Z][a-zA-Z0-9]{3,15}$/;
      var email = /^([a-zA-Z]|[0-9])(\w|\-)+@[a-zA-Z0-9]+\.([a-zA-Z]{2,4})$/;
      var phone = /^1[3456789]\d{9}$/;
      var name = /([\*\"\'<>\/])+/;
      setTimeout(() => {
        if (
          !email.test(value) &&
          !userName.test(value) &&
          !phone.test(value) &&
          name.test(value)
        ) {
          callback(new Error("请输入正确的用户名/客户名/手机号/邮箱"));
        } else {
          callback();
        }
      }, 100);
    };

    return {
      tableHeight: 400,
      customerform: { name: "" },
      page: {
        pages: 0,
        total: 0,
        pageSize: 10,
        pageNum: 1
      },
      rule1: {
        name: {
          validator: checkName
        }
      },
      columnList: [
        {
          prop: "userName",
          lable: "客户名",
          width: 200,
          num: 1
        },
        { prop: "nickName", lable: "用户姓名", width: 200, num: 2 },
        { prop: "phonenumber", lable: "手机号", width: 200, num: 3 },
        { prop: "email", lable: "邮箱", width: 300, num: 4 },

        { prop: "status", lable: "状态", width: 100, num: 5 },
        { prop: "currentPoints", lable: "积分", width: 100, num: 6 }
      ],
      customerData: [],
      multipleSelection: [],
      Ostatus: ""
    };
  },
  mounted() {
    this.getData();
  },
  methods: {
    resetForm(formName) {
      this.$refs[formName].resetFields();
      this.handleCurrentChange(1);
      this.getData();
    },

    // 查询用户
    search() {
      this.handleCurrentChange(1);
      this.getData();
    },
    // 转换汉字
    Words() {
      for (let i = 0; i < this.customerData.length; i++) {
        if (this.customerData[i].status == 1) {
          this.customerData[i].status = "停用";
        } else if (this.customerData[i].status == 0) {
          this.customerData[i].status = "启用";
        }
      }
    },
    //查询数据列表

    getData() {
      this.$axios
        .get(
          "prod-api/backend/customer/list?" +
            "pageNum=" +
            this.page.pageNum +
            "&pageSize=" +
            this.page.pageSize +
            "&searchValue=" +
            this.customerform.name
        )
        .then(res => {
          if (200 == res.data.code) {
            this.customerData = res.data.rows;
            this.page.total = res.data.total;
            this.Words();
            this.$message({
              showClose: true,
              message: "查询成功",
              type: "success"
            });
          } else {
            this.$message({
              showClose: true,
              message: res.data.msg,
              type: "error"
            });
          }
        });
    },

    // 每页条数
    handleSizeChange(val) {
      this.page.pageSize = val;
      this.getData();
    },
    // 选中触发事件
    handleSelectionChange(row) {
      this.multipleSelection = row;
      // alert(JSON.stringify(this.multipleSelection));
    },
    //第几页
    handleCurrentChange(val) {
      this.page.pageNum = val;
      this.getData();
    },
    //显示新增弹出框
    addData() {
      this.$refs.addData.openDialog(
        true,
        JSON.parse(JSON.stringify(this.multipleSelection))
      );
    },
    //显示修改弹出框
    editData() {
      if (1 == this.multipleSelection.length) {
        this.$refs.editData.openDialog(
          true,
          JSON.parse(JSON.stringify(this.multipleSelection))
        );
      } else if (this.multipleSelection.length > 1) {
        this.$message({
          type: "info",
          message: "请选择一条数据进行操作"
        });
      } else {
        this.$message({
          type: "info",
          message: "请选择数据进行操作"
        });
      }
    },
    //启用
    start() {
      if (this.multipleSelection.length >= 1) {
        this.$confirm("是否启用", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            let params = {
              userIds: [],
              operate: "enable"
            };

            for (var i = 0; i < this.multipleSelection.length; i++) {
              params.userIds.push(this.multipleSelection[i].userId);

              this.Ostatus = this.multipleSelection[i].status;
            }
            params.userIds = params.userIds.join(",");
            if (this.Ostatus == "启用") {
              this.$message({
                type: "error",
                message: "状态为启用的用户，不能启用"
              });
            } else {
              this.$axios
                .put(
                  "prod-api/backend/customer/" +
                    params.operate +
                    "/" +
                    params.userIds
                )
                .then(res => {
                  if (res.data.code == 200) {
                    this.handleCurrentChange(1);
                    this.getData();

                    this.Words();

                    this.$message({
                      type: "success",
                      message: res.data.msg
                    });
                  } else {
                    this.$message({
                      type: "error",
                      message: res.data.msg
                    });
                  }
                });
            }
          })
          .catch(() => {
            this.$message({
              type: "info",
              message: "已取消启用"
            });
          });
      } else {
        this.$message({
          type: "info",
          message: "请选择数据进行操作"
        });
      }
    },
    //禁用
    forbid() {
      if (this.multipleSelection.length >= 1) {
        this.$confirm("是否禁用", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            let params = {
              userIds: [],
              operate: "disable"
            };

            for (var i = 0; i < this.multipleSelection.length; i++) {
              params.userIds.push(this.multipleSelection[i].userId);
            }
            params.userIds = params.userIds.join(",");

            this.$axios
              .put(
                "prod-api/backend/customer/" +
                  params.operate +
                  "/" +
                  params.userIds
              )
              .then(res => {
                if (res.data.code == 200) {
                  this.handleCurrentChange(1);
                  this.getData();
                  this.Words();

                  this.$message({
                    type: "success",
                    message: res.data.msg
                  });
                } else {
                  this.$message({
                    type: "error",
                    message: res.data.msg
                  });
                }
              });
          })
          .catch(() => {
            this.$message({
              type: "info",
              message: "已取消禁用"
            });
          });
      } else {
        this.$message({
          type: "info",
          message: "请选择数据进行操作"
        });
      }
    },
    //删除
    Delete() {
      if (this.multipleSelection.length >= 1) {
        this.$confirm("此操作将会将数据永久删除，是否继续？", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            let params = {
              userIds: []
            };
            for (var i = 0; i < this.multipleSelection.length; i++) {
              params.userIds.push(this.multipleSelection[i].userId);
            }
            params.userIds = params.userIds.join(",");

            this.$axios
              .delete("prod-api/backend/customer/delete/" + params.userIds)
              .then(res => {
                if (res.data.code == 200) {
                  this.$message({
                    type: "success",
                    message: res.data.msg
                  });
                  this.getData();
                } else {
                  this.$message({
                    type: "error",
                    message: res.data.msg
                  });
                }
              });
          })
          .catch(() => {
            this.$message({
              type: "info",
              message: "已取消删除"
            });
          });
      } else {
        this.$message({
          type: "info",
          message: "请选择数据进行操作"
        });
      }
    },

    resetPwd() {
      //  重置密码 调接口
      if (this.multipleSelection.length >= 1) {
        this.$confirm("是否重置密码", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            let params = {
              userIds: []
            };

            for (var i = 0; i < this.multipleSelection.length; i++) {
              params.userIds.push(this.multipleSelection[i].userId);
            }
            params.userIds = params.userIds.join(",");

            this.$axios
              .put("prod-api/backend/customer/reset/" + params.userIds)
              .then(res => {
                if (res.data.code == 200) {
                  this.$message({
                    type: "success",
                    message: res.data.msg
                  });
                } else {
                  this.$message({
                    type: "error",
                    message: res.data.msg
                  });
                }
              });
          })
          .catch(() => {
            this.$message({
              type: "info",
              message: "已取消重置密码"
            });
          });
      } else {
        this.$message({
          type: "info",
          message: "请选择数据进行操作"
        });
      }
    }
  }
};
</script>
<style lang="scss" scoped>
.customer {
  margin: 10px;
  .box-card {
    height: 120px;
    margin-bottom: 10px;
    .name,
    .phone {
      width: 200px;
      float: left;
      position: relative;
      top: 15px;
      left: -40px;
      margin-right: 80px;
      .nameBtn,
      .phoneBtn {
        width: 200px;
      }
    }
    .addBtn,
    .resetBtn {
      position: relative;
      left: 50px;
      top: 15px;
      margin-right: 50px;
    }
  }
  .customer-table {
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
  }
}
</style>