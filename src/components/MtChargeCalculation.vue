<template>
  <div>
    <el-card class="box-card">
      <el-row>
        <el-col :span="8">
          客单价:
          <input type="number" v-model="state.unitprice">
        </el-col>
        <el-col :span="8">
          公里数:
          <input type="number" v-model="state.mileage">
        </el-col>
        <el-col :span="8">
          时间段:
          <select v-model="state.time">
            <option :value="1">21：00前</option>
            <option :value="2">21：00后</option>
          </select>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="8">新式服务费:{{feilv.new.toFixed(2)}}元</el-col>
        <el-col :span="8">费率:{{feilv.newbfh.toFixed(2)}}%</el-col>
        <el-col :span="8">实收:{{feilv.newsj.toFixed(2)}}</el-col>
      </el-row>
      <el-row>
        <el-col :span="8">旧式服务费:{{feilv.old.toFixed(2)}}元</el-col>
        <el-col :span="8">费率:{{feilv.oldbfh}}%</el-col>
        <el-col :span="8">实收:{{feilv.oldsj.toFixed(2)}}元</el-col>
      </el-row>
      <el-row>
        <el-col :span="8">服务费相差:{{feilv.sum1.toFixed(2)}}元</el-col>
        <el-col :span="8">费率相差:{{feilv.sum2}}%</el-col>
        <el-col :span="8">实收相差:{{feilv.sum3.toFixed(2)}}元</el-col>
      </el-row>
      <el-row>
        <el-col :span="24">
          <el-button type="primary" style="width:100%;" round @click="save">保存</el-button>
        </el-col>
      </el-row>
    </el-card>

    <el-scrollbar height="500px" style="margin-top:10px 20px;">
      <el-card class="box1-card" v-for="(item,index) in local.data" :key="index">
        <template #header>
          <div class="card-header">
            <el-button class="button" type="text" @click="remove(item,index)">删除</el-button>
          </div>
        </template>
        <el-row>
          <el-col :span="8">客单价:{{item.unitprice}}</el-col>
          <el-col :span="8">公里数:{{item.mileage}}</el-col>
          <el-col :span="8">时间段:{{item.time==1?"21：00前":"21：00后"}}</el-col>
        </el-row>
        <el-row>
          <el-col :span="8">新式服务费:{{item.new.toFixed(2)}}元</el-col>
          <el-col :span="8">费率:{{item.newbfh.toFixed(2)}}%</el-col>
          <el-col :span="8">实收:{{item.newsj.toFixed(2)}}</el-col>
        </el-row>
        <el-row>
          <el-col :span="8">旧式服务费:{{item.old.toFixed(2)}}元</el-col>
          <el-col :span="8">费率:{{item.oldbfh.toFixed(2)}}%</el-col>
          <el-col :span="8">实收:{{item.oldsj.toFixed(2)}}元</el-col>
        </el-row>
        <el-row>
          <el-col :span="8">服务费相差:{{item.sum1.toFixed(2)}}元</el-col>
          <el-col :span="8">费率相差:{{item.sum2.toFixed(2)}}%</el-col>
          <el-col :span="8">实收相差:{{item.sum3.toFixed(2)}}元</el-col>
        </el-row>
      </el-card>
    </el-scrollbar>
  </div>
</template>

<script>
import { reactive, watch, watchEffect, onMounted } from "vue";
import { computed, ref } from "@vue/reactivity";
import Cookies from "js-cookie";

export default {
  name: "MtChargeCalculation",
  setup() {
    const local = reactive({
      data: []
    });

    const state = reactive({
      unitprice: 0,
      mileage: 0,
      time: 1
    });

    const feilv = reactive({
      new: computed(() => {
        let js = state.unitprice * 0.07 <= 1.08 ? 1.08 : state.unitprice * 0.07;
        const jl = getjl(state.mileage);
        const jg = getjg(state.unitprice);
        const sj = getsj(state.time);
        return js + (jl + jg + sj);
      }),
      newsj: computed(() => {
        return state.unitprice - feilv.new;
      }),
      newbfh: computed(() => {
        let a = ((feilv.new.toFixed(2) / state.unitprice) * 100).toFixed(2);
        if (a && a > 0 && a != Infinity) {
          return a;
        }
        return 0;
      }),
      old: computed(() => {
        return state.unitprice * 0.21 < 3.5 ? 3.5 : state.unitprice * 0.21;
      }),
      oldsj: computed(() => {
        return state.unitprice - feilv.old;
      }),
      oldbfh: computed(() => {
        let a = ((feilv.old.toFixed(2) / state.unitprice) * 100).toFixed(2);

        if (a && a > 0 && a != Infinity) {
          console.log(a);
          return a;
        }
        return 0;
      }),
      sum1: computed(() => {
        return feilv.new - feilv.old;
      }),
      sum2: computed(() => {
        let a = (feilv.newbfh - feilv.oldbfh).toFixed(2);

        if (a && a > 0 && a != Infinity) {
          return a;
        }
        return 0;
      }),
      sum3: computed(() => {
        return feilv.newsj - feilv.oldsj;
      })
    });

    onMounted(() => {
      refsh();
    });

    function getsj(val) {
      switch (val) {
        case 1:
          return 0;
        case 2:
          return 0.5;
      }
    }

    function getjg(val) {
      if (val >= 0 && val <= 20) {
        return 0;
      }
      if (val > 20 && val <= 30) {
        console.log(Math.ceil(val - 20));
        return Math.ceil(val - 20) * 0.13;
      }
      if (val > 30) {
        return 1.3 + Math.ceil(val - 30) * 0.12;
      }
    }

    function getjl(val) {
      if (val >= 0 && val <= 3) {
        return 2.5;
      } else {
        return 2.5 + Math.ceil((val - 3) * 10) * 0.2;
      }
    }

    watchEffect(() => {
      console.log(state.unitprice, state.mileage, state.time);
    });

    function save() {
      console.log(feilv);
      let data = getToken();
      if (data == null) {
        data = [];
      }
      data.push({
        ...state,
        ...feilv
      });
      setToken(data);
      refsh();
    }

    function refsh() {
      let data = getToken();
      if (data == null) {
        data = [];
      }
      local.data = data.reverse();
    }

    function remove(item, index) {
      let data = getToken();
      if (data == null) {
        data = [];
      }
      data.splice(data.length - 1 - index, 1);
      setToken(data);
      refsh();
    }

    return {
      state,
      feilv,
      save,
      local,
      remove
    };
  }
};

const TokenKey = "savecomputed";

function getToken() {
  return JSON.parse(localStorage.getItem(TokenKey));
}

function setToken(token) {
  return localStorage.setItem(TokenKey, JSON.stringify(token));
}

function removeToken() {
  return Cookies.remove(TokenKey);
}
</script>

<style>
.box-card {
  width: 100%;
  line-height: 50px;
  text-align: left;
}
.box1-card {
  width: 100%;
  line-height: 24px;
  text-align: left;
  margin: 10px 0px;
}
.card-header {
  height: 18px;
  text-align: right;
}
.el-button {
  padding: 0px;
  min-height: 0px;
}
</style>

