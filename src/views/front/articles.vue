<template>
  <div class="home">
    <div
      v-loading.fullscreen.lock="dataListLoading"
      element-loading-text="拼命加载中"
    >
      <el-row id="artList" type="flex" justify="space-around">
        <el-col :span="16">
          <div v-if="dataList != null && dataList.length > 0">
            <ul style="list-style-type: none">
              <li v-for="(item, index) in dataList" :key="index">
                <el-row class="art-item">
                  <el-card shadow="hover">
                    <h5>
                      <router-link
                        :to="{ path: '/front/article', query: { id: item.id } }"
                        tag="span"
                        class="art-title"
                        >{{ item.title }}</router-link
                      >
                    </h5>
                    <el-row
                      class="art-info d-flex align-items-center justify-content-start"
                    >
                      <div class="art-time">
                        <i class="el-icon-time"></i>：{{
                          mydateformat(item.gmtCreate)
                        }}
                      </div>
                      <div class="d-flex align-items-center">
                        <img class="tag" src="~@/assets/img/tag.png" />：
                        <el-tag size="mini">{{ item.tag }}</el-tag>
                      </div>
                    </el-row>
                    <el-row class="art-body">
                      <div class="side-img hidden-sm-and-down">
                        <img class="art-banner" :src="item.imageUrl" />
                      </div>
                      <div class="side-abstract">
                        <div class="art-abstract">
                          {{ item.description }}
                        </div>
                        <div class="art-more">
                          <router-link
                            :to="{
                              path: '/front/article',
                              query: { id: item.id },
                            }"
                            tag="span"
                            key="key"
                          >
                            <el-button plain>阅读更多</el-button>
                          </router-link>
                          <div class="view">
                            <i class="el-icon-view"></i>{{ item.viewCnt }}
                          </div>
                        </div>
                      </div>
                    </el-row>
                  </el-card>
                  <span
                    v-if="
                      item.likeCnt >= 50 ||
                      item.viewCnt >= 1000 ||
                      item.commentCnt >= 20
                    "
                  >
                    <icon-svg name="hot" class="star"></icon-svg>
                  </span>
                </el-row>
              </li>
            </ul>
          </div>
          <div class="block pagination">
            <el-pagination
              @size-change="sizeChangeHandle"
              @current-change="currentChangeHandle"
              :current-page="pageIndex"
              :page-sizes="[10, 20, 50, 100]"
              :page-size="pageSize"
              :total="totalPage"
              layout="total, sizes, prev, pager, next, jumper"
            >
            </el-pagination>
          </div>
        </el-col>
        <el-col :span="6" class="hidden-sm-and-down" id="side">
          <div class="item"></div>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
export default {
  name: "articles",
  data() {
    return {
      dataListLoading: true,
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 1,
    };
  },
  created() {
    this.geMaterialsDataList();
  },
  methods: {
    geMaterialsDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/finalexam/articles/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
        }),
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.dataList = data.page.list;
          this.totalPage = data.page.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },

    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.geMaterialsDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.geMaterialsDataList();
    },
  },
};
</script>


<style lang="scss" scoped>
#side .item {
  margin-bottom: 30px;
}

.art-item {
  margin-bottom: 30px;
  position: relative;
}

.art-item .star {
  width: 60px;
  height: 60px;
  position: absolute;
  top: 0;
  right: 0;
}

img.tag {
  width: 16px;
  height: 16px;
}

.art-title {
  border-left: 3px solid #f56c6c;
  padding-left: 5px;
  cursor: pointer;
}

.art-title:hover {
  padding-left: 10px;
  color: #409eff;
}

.art-time {
  margin-right: 20px;
}

.art-body {
  display: flex;
  padding: 10px 0;
}

.side-img {
  height: 150px;
  width: 270px;
  overflow: hidden;
  margin-right: 10px;
}

img.art-banner {
  width: 100%;
  height: 100%;
  transition: all 0.6s;
}

img.art-banner:hover {
  transform: scale(1.4);
}

.side-abstract {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.art-abstract {
  flex: 1;
  color: #aaa;
}

.art-more {
  height: 40px;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
}

.art-more .view {
  color: #aaa;
}
h5 {
  font-size: 18px;
}
.pagination {
  background-color: #f9f9f9;
}
</style>