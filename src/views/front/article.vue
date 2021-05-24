<template>
  <div>
    <el-row class="main" type="flex" justify="center">
      <el-col :span="16">
        <div
          id="artcle-info"
          v-bind:style="{
            backgroundImage: 'url(' + article.imageUrl + ')',
          }"
        >
          <h2 class="text-center">
            <strong>{{ article.title }}</strong>
          </h2>
          <!-- 描述：文章信息 -->
          <div class="text-center timeAndView">
            <span class="article-time">
              <i class="el-icon-time"></i>
              发表于：<span>{{ mydateformat(article.gmtCreate) }}</span>
            </span>
            &nbsp;|&nbsp;
            <span class="article-views">
              <i class="el-icon-view"></i>
              阅读量：<span>{{ article.viewCnt }}</span>
            </span>
          </div>
          <p class="abstract">前言：{{ article.description }}</p>
        </div>
        <hr />
        <div id="artcle-content">
          <div v-html="article.context">
            {{ article.context }}
          </div>
        </div>
        <div id="statement">
          <div class="item" shadow="hover">
            <router-link title="进入作者主页"
              :to="{
              path: '/front/profile',
                query: { id: article.creator },
              }"
              tag="span"
              key="key"
            >
            作者:{{article.author}}
            </router-link>
          </div> 
          <div class="item" style="padding-top:10px">本博客所有文章除特别声明外,转载请注明出处!</div>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
export default {
  name: "article",
  data() {
    return {
      article_id: "",
      article: {},
    };
  },
  created() {
    this.article_id = this.$route.query.id;
    this.geArticle();
  },
  methods: {
    geArticle() {
      this.$http({
        url: this.$http.adornUrl(
          `/finalexam/articles/info/${this.article_id}`
        ),
        method: "get",
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.article = data.article;
        } else {
        }
      });
    },
  },
};
</script>

<style scoped>
.item :hover{ 
  padding-left: 10px;
  padding-bottom: 10px;
  color: #409eff; 
  cursor: pointer; 
}
#artcle-info {
  padding: 20px;
  margin-bottom: 40px;
}

#artcle-info .abstract {
  color: #ffffff;
  border-left: 3px solid #f56c6c;
  padding: 10px;
  background-color: rgba(126, 129, 135, 0.3);
}

#artcle-info .timeAndView {
  padding: 20px;
  line-height: 30px;
  font-size: 16px;
  color: black;
}

pre.has {
  color: #ffffff;
  background-color: rgba(0, 0, 0, 0.8);
}

img.has {
  width: 100%;
}

#statement {
  border-left: 3px solid #f56c6c;
  padding: 20px;
  background-color: #ebeef5;
}
</style>