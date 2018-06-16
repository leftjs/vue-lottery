<template>
  <div class="home">
    <img :src="logo" id="logo" alt="">
    <p style="font-size: 5rem; margin: 1rem;">
      {{currentTitle}}
    </p>
    <el-input id="current" v-model="current"></el-input>
    <div id="btn_group">
      <el-button class="btn" plain round type="danger" id="start" @click="trigger" :disabled="currentTitle == '抽奖完毕'">{{ current_interval ? "暂停" : "开始"}}</el-button>
      <el-button class="btn" plain round type="info" @click="reset">重置</el-button>
      <el-button class="btn" plain round type="warning" @click="openConfig=!openConfig" :disabled="startLottery == true">设置</el-button>
      <el-button class="btn" plain round type="success" @click="openAward=!openAward">获奖</el-button>
    </div>
    <el-dialog title="抽奖设置" :visible.sync="openConfig" :before-close="beforeConfigClose">
      <el-form label-width="100px" style="width: 50%; margin: 0 auto;" :model="config" ref="configForm" label-position="right">
        <el-form-item label="抽奖人数" :rules="rules.number" prop="number">
          <el-input type="number" v-model.number="config.number"></el-input>
        </el-form-item>
        <p style="border-bottom: 1px dashed lightgrey; padding-bottom: 10px; color: grey;">奖项设置</p>
        <el-form-item v-for="(reward,index) in config.rewards" :key="reward.key" :label="reward.name" :rules="rules.reward" :prop="'rewards.' + index + '.value'">
          <div style="display: flex;">
            <el-input v-model.number="reward.value" placeholder="奖项名称/获奖人数"></el-input>
            <el-button style="margin-left: 20px;" @click.prevent="removeReward(reward)" type="danger">删除</el-button>
          </div>

        </el-form-item>
        <el-form-item label-width="0">
          <div style="width: 100%; display:flex; flex-direction: row; justify-content: center;">
            <el-button @click="addReward">添加</el-button>
            <el-button @click="init">默认</el-button>
          </div>
        </el-form-item>
      </el-form>
    </el-dialog>

    <el-dialog title="获奖名单" :visible.sync="openAward">
      <div v-for="reward in config.rewards" :key="reward.key" style="border-bottom: 1px dashed lightgrey">
        <p style="font-size: 2.4rem;">{{reward.title}} ({{reward.count-reward.left}} / {{reward.count}})</p>
        <p style="font-size: 1.2rem;">{{JSON.stringify(reward.awards)}}</p>
      </div>
    </el-dialog>
  </div>

</template>

<script>
// @ is an alias to /src
import HelloWorld from "@/components/HelloWorld.vue";
import _ from "lodash";

export default {
  name: "home",
  components: {
    HelloWorld
  },
  data() {
    let checkNum = (rule, value, callback) => {
      if (!value) {
        return new Error("必填信息");
      } else {
        if (!Number.isInteger(value)) {
          callback(new Error("请输入数字值"));
        } else {
          if (value < 0) {
            callback(new Error("不能小于0"));
          } else {
            callback();
          }
        }
      }
    };

    let checkRewardValue = (rule, value, callback) => {
      if (!value) {
        return new Error("必填信息");
      } else {
        if (value.split("/").length < 2) {
          callback(new Error("输入格式不对(奖项/数量)"));
        } else {
          let valueArr = value.split("/");
          if (valueArr[0].toString() === "") {
            callback(new Error("请输入奖项名称"));
          }
          if (Number.isNaN(parseInt(valueArr[1]))) {
            callback(new Error("请输入奖项人数"));
          }

          if (parseInt(valueArr[1]) < 1) {
            callback(new Error("奖项人数必须大于0"));
          }
          callback();
        }
      }
    };
    const requiredRule = { required: true, message: "必填信息", trigger: ["blur", "change"] };
    return {
      logo: require("../assets/logo.png"),
      current: "GO",
      current_interval: null,
      currentTitle: "",
      openConfig: true,
      config: {
        nubmer: 0,
        rewards: []
      },
      startLottery: false,
      openAward: false,
      numbers: [],
      rules: {
        number: [requiredRule, { validator: checkNum, trigger: ["blur", "change"] }],
        reward: [requiredRule, { validator: checkRewardValue, trigger: ["blur", "change"] }]
      }
    };
  },
  methods: {
    trigger() {
      let that = this;
      if (this.startLottery === false) {
        this.startLottery = true;
      }
      if (this.current_interval) {
        clearInterval(this.current_interval);
        this.win(this.current);
        this.current_interval = null;
        return;
      }
      this.current_interval = setInterval(() => {
        let count = this.numbers.length;
        let random = this.numbers[_.random(count - 1)];
        that.current = random;
      }, 10);
    },
    win(current) {
      let index = this.numbers.indexOf(current);
      this.numbers.splice(index, 1);

      _.forEach(this.config.rewards, (reward, index) => {
        if (reward.left > 0) {
          let awards = reward.awards || [];
          awards.push(current);
          this.$message.success(`恭喜 ${this.currentTitle} 获得者: ${current}`);
          this.config.rewards[index].left--;
          return false;
        }
      });
      this.setCurrentTitle();
    },
    addReward() {
      let rewards = this.config.rewards;
      let key;
      if (rewards.length > 0) {
        key = rewards[rewards.length - 1].key + 1;
      } else {
        key = 0;
      }
      this.config.rewards.push({
        name: `奖项${key}`,
        value: "",
        key
      });
    },
    removeReward(item) {
      let index = this.config.rewards.indexOf(item);
      if (index !== -1) {
        this.config.rewards.splice(index, 1);
      }
    },
    init() {
      const defaultConfig = {
        rewards: [
          {
            key: 0,
            name: "奖项0",
            value: "三等奖/10"
          },
          {
            key: 1,
            name: "奖项1",
            value: "二等奖/5"
          },
          {
            key: 2,
            name: "奖项2",
            value: "一等奖/3"
          },
          {
            key: 3,
            name: "奖项3",
            value: "特等奖/1"
          }
        ],
        number: 100
      };
      this.config = defaultConfig;
    },

    reset() {
      this.init();
      this.clear();
    },
    clear() {
      this.current = "GO";
      this.startLottery = false;
      if (this.current_interval) {
        clearInterval(this.current_interval);
        this.current_interval = null;
      }

      this.processRewards();
    },
    processRewards() {
      let rewards = this.config.rewards;
      this.config.rewards = _.map(rewards, item => {
        let arr = item.value.split("/");
        return {
          ...item,
          title: arr[0].toString(),
          count: parseInt(arr[1]),
          left: parseInt(arr[1]),
          awards: []
        };
      });
      this.setCurrentTitle();
    },

    setCurrentTitle() {
      _.forEach(this.config.rewards, (reward, index) => {
        if (reward.left > 0) {
          this.currentTitle = `${reward.title} (${reward.left} / ${reward.count})`;
          return false;
        }
        if (index === this.config.rewards.length - 1) {
          this.currentTitle = "抽奖完毕";
        }
      });
    },
    beforeConfigClose(done) {
      this.$refs["configForm"].validate(valid => {
        if (valid) {
          let arrs = [];
          let count = this.config.number;
          for (let i = 0; i < count; i++) {
            arrs.push(i + 1);
          }
          this.numbers = arrs;
          this.clear();

          this.openConfig = !this.openConfig;
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    }
  },
  created() {
    this.init();
  }
};
</script>



<style lang="scss">
html,
body,
#app,
.home {
  height: 100%;
}

.home {
  background: url("../assets/background.png");
  background-size: 100% 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

#logo {
  width: 500px;
  position: absolute;
  left: 20px;
  top: 20px;
}
#start {
  margin-top: 20px;
}
#current {
  width: 300px;
  height: 200px;
  border-radius: 20px;
  text-align: center;
  font-size: 10rem;
  background: transparent;
  border: 1px solid #b20af0;
}
#result {
  width: 100px;
  height: 200px;
  background: transparent;
}

.btn {
  width: 150px;
  height: 80px;
  font-size: 2rem;
}
</style>

