<template>
  <div class="f-footer">
    <el-main class="main">
      <el-row>
        <el-col :span="12" class="item-col"> 
		  <div class="title">捐赠</div>
          <div class="item">
            <el-row> 
              <el-col :span="8"> 
                <h5>支付宝</h5>
                <img src="" style="height: 100px" />
              </el-col>
              <el-col :span="8"><h5>微信</h5>
                <img src="" style="height: 100px"/></el-col>
            </el-row>
          </div>
        </el-col>
        <el-col :span="12" class="item-col">
          <div class="item">
            <i class="el-icon-caret-right"></i>
            本站已稳定运行123天4时36分22秒
          </div>
          <div class="item">
            <i class="el-icon-caret-right"></i>
            QQ邮箱:1004027030@qq.com
          </div>
          <div class="item">
            <i class="el-icon-caret-right"></i>
            © zxj.cn |
            <a class="out-link" href="http://www.beian.miit.gov.cn"
              >闽ICP备17015355号</a
            >
          </div>
          <div class="item">
            <i class="el-icon-caret-right"></i>
            <a href="https://gitee.com/salty_orange" class="out-link"
              >gitee地址</a
            >
          </div>
        </el-col>
      </el-row>
    </el-main>
  </div>
</template>

<script>
export default {
  name: "fFooter",
};
</script>

<style scoped>
.f-footer {
  padding: 20px;
  text-align: center;
  color: #fff;
  background-color: rgb(32 41 49);
}
.main {
  width: 100%;
  margin: 0 auto;
  color: #ccc;
  height: 5%;
}
.out-link {
  text-decoration: none;
  color: #ccc;
}
.out-link:hover {
  color: #ffffff;
}
.item-col {
  text-align: left;
  line-height: 50px;
}
.item-col.weside {
  font-size: 14px;
}
.item-col .title {
  color: #ddd;
  margin-bottom: 10px;
}
.item span {
  margin-right: 5px;
}
</style>