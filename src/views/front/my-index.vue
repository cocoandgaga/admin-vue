<template>
  <div class="app">
    <el-container>
      <el-header>
        <f-header :class="{ navBarWrap: navBarFixed }"></f-header>
      </el-header>
      <el-main class="bgimg">
        <el-row type="flex" justify="center" id="content">
          <el-col :xs="20" :md="20" :style="{ minHeight: minHeight + 'px' }">
            <router-view></router-view>
          </el-col>
        </el-row>
      </el-main>
      <el-footer>
        <f-footer></f-footer>
      </el-footer>
    </el-container>
  </div>
</template>

<script>
import fHeader from "@/components/front-common/f-header.vue";
import fFooter from "@/components/front-common/f-footer.vue";
export default {
  data() {
    return {
      minHeight: 0,
      navBarFixed: false,
    };
  },
  components: {
    fHeader,
    fFooter,
  },
  methods: {
    watchScroll() {
      var scrollTop =
        window.pageYOffset ||
        document.documentElement.scrollTop ||
        document.body.scrollTop;
      //  当滚动超过 50 时，实现吸顶效果
      if (scrollTop > 50) {
        this.navBarFixed = true;
      } else {
        this.navBarFixed = false;
      }
    },
  },
  mounted() {
    let that = this;
    that.minHeight = document.documentElement.clientHeight;
    window.addEventListener("scroll", that.watchScroll);
    window.onresize = function () {
      that.minHeight = document.documentElement.clientHeight;
    };
  },
};
</script>

<style scoped>
.app {
  font-family: "microsoft yahei";
}
#content {
  background-color: #f9f9f9;
  background: rgba(0, 0, 0, 0) url("~@/assets/img/profile.jpg") no-repeat scroll
    100% 100%;
  background-size: auto;
  background-size: cover;
}
.navBarWrap {
  position: fixed;
  top: 0;
  z-index: 999;
  width: 100%;
}
</style>
