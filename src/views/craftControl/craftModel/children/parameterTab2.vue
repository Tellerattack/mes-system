/*
 * @Author: xgw 
 * @Date: 2019-05-15 11:51:15 
 * @Last Modified by: xgw
 * @Last Modified time: 2019-05-15 11:53:08
 */
<!--投料过程参数配置-->
<template>
  <div class="parameterTab2">
    <div class="mt-3 el-card box-card is-always-shadow">
      <div class="el-card__body">
        <div class="fl">编辑生产步骤：</div>
        <el-switch v-model="order" active-text="是" inactive-text="否" active-color="#13ce66" inactive-color="#ff4949" @change="chageModel">
        </el-switch>
      </div>
    </div>
    <template v-if="order">
      <div class="mt-3 el-card box-card is-always-shadow stepContent" v-for="(item ,index) in stepData" :key="index">
        <div class="el-card__header fz16">
          <div v-text="item.stepName" class="stepName"></div>
          <div class="operation">
            <div class="fl" style="width:80px;" @click.stop="showElse(item)">
              <i v-text="item.open?'折叠':'展开'" class="fz16"></i>
              <i :class="item.open?'el-icon-caret-bottom':'el-icon-caret-right'" style="font-size: 16px;"></i>
            </div>
          </div>
        </div>
        <div class="el-card__body" v-show="item.open">
          <div class="el-card box-card" style="width:600px">
            <div class="el-card__header fz16">#参数名称#</div>
            <div class="el-card__body">
              <el-form :model="parameterForm" label-width="120px" label-position="left">
                <el-form-item label="参数值种类">
                  <el-select v-model="parameterForm.parameterType">
                    <el-option v-for="item in parameterTypeData" :key="item.value" :label="item.label" :value="item.value">
                    </el-option>
                  </el-select>
                </el-form-item>
                <el-form-item label="参数值是否分档">
                  <el-switch v-model="parameterForm.classify" active-text="是" inactive-text="否" active-color="#13ce66" inactive-color="#ff4949">
                  </el-switch>
                  <el-button type="primary" round size="small" class="ml-5" :disabled="!parameterForm.classify">增加分档</el-button>
                </el-form-item>
              </el-form>
              <div class="el-card box-card">
                <div class="el-card__header fz16">分档1</div>
                <div class="el-card__body">
                  <el-form :model="parameterForm" label-width="120px" label-position="left">
                    <el-form-item label="分档名称">
                      <el-input v-model="parameterForm.a"></el-input>
                    </el-form-item>
                    <el-form-item label="参数值">
                      <el-input v-model="parameterForm.b"></el-input>
                    </el-form-item>
                  </el-form>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </template>
    <template v-else>
      <div class="mt-3 el-card box-card is-always-shadow" style="width:600px">
        <div class="el-card__header fz16">#参数名称#</div>
        <div class="el-card__body">
          <el-form :model="parameterForm" label-width="120px" label-position="left">
            <el-form-item label="参数值种类">
              <el-select v-model="parameterForm.parameterType">
                <el-option v-for="item in parameterTypeData" :key="item.value" :label="item.label" :value="item.value">
                </el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="参数值是否分档">
              <el-switch v-model="parameterForm.classify" active-text="是" inactive-text="否" active-color="#13ce66" inactive-color="#ff4949">
              </el-switch>
              <el-button type="primary" round size="small" class="ml-5" :disabled="!parameterForm.classify">增加分档</el-button>
            </el-form-item>
          </el-form>
          <div class="el-card box-card">
            <div class="el-card__header fz16">分档1</div>
            <div class="el-card__body">
              <el-form :model="parameterForm" label-width="120px" label-position="left">
                <el-form-item label="分档名称">
                  <el-input v-model="parameterForm.a"></el-input>
                </el-form-item>
                <el-form-item label="参数值">
                  <el-input v-model="parameterForm.b"></el-input>
                </el-form-item>
              </el-form>
            </div>
          </div>
        </div>
      </div>
    </template>
    <el-dialog title="编辑生产步骤" :visible.sync="visibleDialog" width="900px" :close-on-click-modal='false'>
      <el-button type="primary" class="mr-3 " @click="addStep">添加步骤</el-button>
      <el-button type="primary">保 存</el-button>
      <div class="mt-3 el-card box-card is-always-shadow stepContent" v-for="(item ,index) in stepData" :key="index">
        <div class="el-card__header fz16">
          <el-input v-model="item.stepName" size="small" placeholder="请输入内容" style="width:300px"></el-input>
          <div class="operation fr">
            <el-button type="danger" plain class="mr-3 fl" size="samll">删除步骤</el-button>
            <div class="fz16 fl" style="width:80px;" @click.stop="showElse(item)">
              <i v-text="item.open?'折叠':'展开'" class="fz16"></i>
              <i :class="item.open?'el-icon-caret-bottom':'el-icon-caret-right'" style="font-size: 16px;"></i>
            </div>
          </div>
        </div>
        <div class="el-card__body" v-show="item.open">
          <el-button type="primary" round size="small" class="mb-3">绑定参数</el-button>
          <el-table :data="tableDate" border>
            <el-table-column type="index" label="序号" align="center" width="50">
            </el-table-column>
            <el-table-column prop="material_name" label="参数名称" align="center">
            </el-table-column>
            <el-table-column prop="material_model" label="参数单位" align="center">
            </el-table-column>
            <el-table-column prop="material_unit" label="参数标签" align="center">
            </el-table-column>
            <el-table-column prop="material_unit" label="参数描述" align="center">
            </el-table-column>
            <el-table-column align="center" label="操作">
              <template slot-scope="scope">
                <el-button type="text" @click="delRow(scope.row.material_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </div>
      </div>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: "parameterTab2",
  data() {
    return {
      order: true,
      visibleDialog: false,

      open: true,
      stepData: [
        {
          stepName: "步骤一",
          open: true
        },
        {
          stepName: "步骤二",
          open: false
        }
      ],
      tableDate: [
        {
          material_id: "material_id",
          material_name: "material_name",
          material_model: "material_model",
          material_unit: "material_unit"
        }
      ],
      parameterTypeData: [
        {
          value: "0",
          label: "固定值"
        },
        {
          value: "1",
          label: "范围值"
        },
        {
          value: "2",
          label: "浮动值"
        },
        {
          value: "3",
          label: "计算值"
        },
        {
          value: "4",
          label: "文本描述"
        }
      ],
      parameterForm: {
        classify: true, //参数值是否分档
        parameterType: "0", //参数值种类
        a: "",
        b: ""
      }
    };
  },
  mounted() {},
  methods: {
    //切换生产状态
    chageModel(val) {
      if (val) {
        this.visibleDialog = true;
      } else {
        this.visibleDialog1 = true;
      }
    },
    //添加步骤
    addStep() {},
    showElse(item) {
      item.open = !item.open;
    },

    delRow(){}
  }
};
</script>
<style lang="scss" scoped>
.stepContent {
  position: relative;
  .stepName {
    height: 32px;
    line-height: 32px;
  }
  .operation {
    position: absolute;
    top: 15px;
    right: 20px;
    height: 40px;
    line-height: 40px;
  }
}
</style>
