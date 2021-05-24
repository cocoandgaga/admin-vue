<template>
  <div>
    <el-menu
      :default-active="activeIndex"
      :default-openeds="activeArr"
      class="el-menu-demo"
      mode="horizontal"
      @select="handleSelect"
      background-color="#545c64"
      text-color="#fff"
      active-text-color="#ffd04b"
    >
      <el-submenu index="task">
        <template slot="title">任务</template>
        <el-menu-item index="/front/task">任务看板</el-menu-item>
        <el-menu-item index="/front/attachment">附件资料 </el-menu-item>
         <el-menu-item index="/front/score">课程平时分</el-menu-item>
      </el-submenu>

      <el-submenu index="school">
        <template slot="title">系内消息</template>
        <el-menu-item index="/front/articles">系级公示</el-menu-item>
        <el-menu-item index="/front/schoolMap">校园地图</el-menu-item>
        <el-menu-item index="/front/examTime">考试时间</el-menu-item>
      </el-submenu>
      <el-submenu index="question">
        <template slot="title">问答</template>
        <el-menu-item index="/front/questions/0">考试相关</el-menu-item>
        <el-menu-item index="/front/questions/1">学习相关</el-menu-item>
        <el-menu-item index="/front/questions/2">其他问题</el-menu-item>
      </el-submenu>
      <el-submenu index="notification">
        <template slot="title">个人中心</template>
        <el-menu-item index="/front/notification">
          <!-- <icon-svg name="message"></icon-svg> -->
          <i class="el-icon-s-comment"></i>
          <span>回复评论</span>
        </el-menu-item>
        <el-menu-item index="/front/myprofile">
          <i class="el-icon-user"></i>
          <span>个人信息</span>
        </el-menu-item>
        <el-menu-item index="/front/publish">
          <i class="el-icon-question"></i>
          <span slot="title">发布问题</span>
        </el-menu-item>
      </el-submenu>
    </el-menu>
  </div>
</template>

<script>
export default {
  name: "fHeader",
  data() {
    return {
      activeIndex: "task",
      activeArr: ["/front/task"],
      isRead: false,
    };
  },
  // created() {
  //   let arr = ["task", "/front/task"];
  //   this.handleSelect("/front/task", arr);
  // },
  methods: {
    handleSelect(key, keyPath) {
      console.log(key);
      console.log(keyPath);
      var path = keyPath[1];
      var splitArr = keyPath[1].split("/");
      if (splitArr.length == 4) {
        let type = splitArr[splitArr.length - 1];
        path = keyPath[1].substring(0, keyPath[1].lastIndexOf("/"));
        this.$router.push({
          path: path,
          query: { type: type },
        });
      } else {
        this.$router.push({
          path: path,
        });
      }
    },
  },
};
</script>

<style>
.item {
  margin-top: 0px;
  margin-right: 40px;
}
</style>