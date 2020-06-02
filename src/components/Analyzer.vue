<template>
  <div class="vue-analyzer" @keydown.enter="analyseData">
    <el-input
      class="input-area"
      type="textarea"
      :autosize="{minRows: 6, maxRows: 6}"
      :placeholder="placeholder"
      v-model="textarea"
    ></el-input>
    <div>
      <el-button class="confirm-button" @click="analyseData" size="small">导入</el-button>
      <el-checkbox v-model="showOriginData">显示原始数据</el-checkbox>
    </div>
    <div class="analysis-area">
      <div class="origin-table" v-show="showOriginData">
        <el-table :data="originData" height="500" border style="width: 100%">
          <el-table-column prop="id" label="编号"></el-table-column>
          <el-table-column prop="name" label="成员"></el-table-column>
          <el-table-column prop="boss" label="BOSS"></el-table-column>
          <el-table-column prop="circle" label="周目"></el-table-column>
          <el-table-column prop="damage" label="伤害"></el-table-column>
          <el-table-column prop="type" label="标记"></el-table-column>
        </el-table>
      </div>

      <div class="handled-table" v-show="!showOriginData">
        <div class="handled-filter-wrapper">
          <span class="filter">
            <el-select
              @change="buildHandledData"
              v-model="filters.round"
              multiple
              placeholder="筛选出刀轮次"
              size="small"
            >
              <el-option
                v-for="item in roundOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </span>

          <span class="filter">
            <el-select
              @change="buildHandledData"
              v-model="filters.circle"
              multiple
              placeholder="筛选周目"
              size="small"
            >
              <el-option
                v-for="item in circleOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </span>

          <span class="filter">
            <el-select
              @change="buildHandledData"
              v-model="filters.boss"
              multiple
              placeholder="筛选BOSS"
              size="small"
            >
              <el-option
                v-for="item in bossOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </span>
        </div>

        <el-table :data="handledData" height="500" border style="width: 100%">
          <el-table-column type="index" width="50"></el-table-column>
          <el-table-column prop="name" label="成员"></el-table-column>
          <el-table-column prop="time" label="出刀次数"></el-table-column>
          <el-table-column prop="damage" label="伤害" sortable></el-table-column>
          <el-table-column prop="percentage" label="伤害比例">
            <template slot-scope="scope">
              <el-progress :percentage="scope.row.percentage"></el-progress>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      textarea: "",
      placeholder: `示例：
编号|出刀者|周目|Boss|伤害|标记
E001|aaa|r1|b1|825,469|通常
E002|bbbbbb|r1|b1|690,811|尾刀`,
      showOriginData: false,
      originData: [],
      handledData: [],
      dataArray: [],
      filters: {
        round: [],
        boss: [],
        circle: []
      },
      roundOptions: [],
      circleOptions: [],
      bossOptions: [
        {
          value: 1,
          label: "一王"
        },
        {
          value: 2,
          label: "二王"
        },
        {
          value: 3,
          label: "三王"
        },
        {
          value: 4,
          label: "四王"
        },
        {
          value: 5,
          label: "五王"
        }
      ]
    };
  },
  components: {},
  methods: {
    analyseData() {
      const data = this.textarea;
      const resData = this.checkAndBuildData(data);
      if (!resData) {
        this.$message.error("数据格式有误");
        return;
      }
    },
    checkAndBuildData(data) {
      const dataArray = data.match(/[^\r\n]+/g);
      const affectiveData = dataArray.filter(item => {
        return this.isData(item);
      });
      if (dataArray.length - affectiveData.length > 1) return false;
      if (dataArray.length === 0) return true;

      let resData = affectiveData.map(item => {
        let res = item.split("|");
        while (res.length > 6) {
          console.log(res.length);
          res[1] += `|${res.splice(item[2], 1)}`;
        }
        return res;
      });

      this.dataArray = resData;

      this.buildOptions();
      this.buildOriginData();
      this.buildHandledData();
      return true;
    },
    buildOptions() {
      const dataArray = this.dataArray;

      const map = {};
      const roundOptions = [];

      dataArray.forEach(item => {
        const name = item[1];
        if (this.isMainFight(item[5]))
          map[name] ? map[name]++ : (map[name] = 1);
      });
      const arrFromMap = Object.values(map);
      const maxRound = Math.ceil(Math.max(...arrFromMap) / 3);

      for (let i = 1; i <= maxRound; i++) {
        roundOptions.push({
          value: i,
          label: `第 ${i} 轮`
        });
      }

      this.roundOptions = roundOptions;

      const circleOptions = [];
      const maxCircleStr = dataArray[dataArray.length - 1][2];
      const maxCircle = parseInt(maxCircleStr.match(/\d+/)[0]);

      for (let i = 1; i <= maxCircle; i++) {
        circleOptions.push({
          value: i,
          label: `第 ${i} 周目`
        });
      }

      this.circleOptions = circleOptions;
    },
    buildOriginData() {
      const dataArray = this.dataArray;
      const originData = dataArray.map(item => {
        return {
          id: item[0],
          name: item[1],
          circle: this.getCircle(item[2]),
          boss: this.getBoss(item[3]),
          damage: this.getDamage(item[4]),
          type: item[5]
        };
      });

      this.originData = originData;
    },
    buildHandledData() {
      const dataArray = this.dataArray;
      const handledCount = {};
      const handledMap = {};

      dataArray.forEach(item => {
        const targetRound = this.filters.round;
        const targetBoss = this.filters.boss;
        const targetCircle = this.filters.circle;

        const name = item[1];

        // get current round

        if (this.isMainFight(item[5]))
          handledCount[name] ? handledCount[name]++ : (handledCount[name] = 1);

        const count = handledCount[name];
        const curRound = Math.ceil(count / 3);

        // get current boss
        const curBoss = parseInt(item[3].match(/\d+/)[0]);

        // get current circle
        const curCircle = parseInt(item[2].match(/\d+/)[0]);

        if (
          (targetRound.length === 0 || targetRound.includes(curRound)) &&
          (targetBoss.length === 0 || targetBoss.includes(curBoss)) &&
          (targetCircle.length === 0 || targetCircle.includes(curCircle))
        ) {
          if (handledMap[name]) {
            handledMap[name].damage += this.getDamage(item[4]);
            handledMap[name].time++;
          } else {
            handledMap[name] = {
              name,
              time: 1,
              damage: this.getDamage(item[4])
            };
          }
        }
      });

      const handledData = Object.values(handledMap);
      const damageArray = handledData.map(item => item.damage);
      const maxDamage = Math.max(...damageArray);
      handledData.forEach(item => {
        item.percentage = parseInt(item.damage / maxDamage * 100);
      });

      this.handledData = handledData;
    },
    isData(string) {
      const re = /E\d+\|.+(\|\w\d+\|\w\d+\|[\d,]+\|[^|]+)$/;
      return re.test(string);
    },
    isMainFight(str) {
      return ["通常", "尾刀"].includes(str);
    },
    getCircle(str) {
      const num = str.match(/\d+/)[0];
      return String(num) + " 周目";
    },
    getBoss(str) {
      const num = str.match(/\d+/)[0];
      return String(num) + " 王";
    },
    getDamage(str) {
      const num = str.match(/\d+/g).reduce((prev, cur) => {
        return prev + cur;
      }, "");
      return parseInt(num);
    }
  },
  computed: {
    tableData() {
      return this.showOriginData ? this.originData : this.handledData;
    }
  },
  mounted() {}
};
</script>

<style>
.vue-analyzer {
  width: 100%;
  height: 100%;
}

.vue-analyzer .input-area {
  max-width: 50%;
  padding-top: 60px;
}

.vue-analyzer .confirm-button {
  margin: 20px 20px 20px 0;
}

.analysis-area {
  width: 80%;
  margin: auto;
  padding-bottom: 30px;
}

.handled-table .handled-filter-wrapper {
  margin-bottom: 20px;
  float: left;
}

.handled-table .handled-filter-wrapper .filter {
  margin-right: 20px;
}
</style>