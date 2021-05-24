<template>
  <div class="quetion">
    <el-dialog
      :close-on-click-modal="false"
      :visible.sync="visible"
      :fullscreen="true"
      :show-close="true"
    >
      <div v-loading="loading">
        <el-row type="flex" justify="center">
          <el-col :span="13">
            <div id="artcle-info">
              <h2 class="text-center">
                <strong>{{ quesionForm.title }}</strong>
              </h2>
              <!-- 描述：文章信息 -->
              <div class="text-center timeAndView backTotop">
                <span class="article-time">
                  <i class="el-icon-time"></i>
                  <span
                    v-if="
                      quesionForm.gmtModified != null &&
                      quesionForm.gmtModified != ''
                    "
                  >
                    修改于： {{ mydateformat(quesionForm.gmtModified) }}
                  </span>
                  <span v-else>
                    创建于： {{ mydateformat(quesionForm.gmtCreate) }}
                  </span>
                </span>
              </div>
              <div>
                <div class="abstract">
                  简短描述：{{ quesionForm.description }}
                </div>
                <div class="artcle-content">
                  <el-card shadow="never" 
                    class="markdown-body info"
                    v-html="quesionForm.content"
                  >
                    {{ quesionForm.content }}
                  </el-card>
                </div>
              </div>
            </div>
            <div>
              <el-divider content-position="center">
                <el-button circle @click="addQuestionlikeCnt(quesionForm.id)">
                  <icon-svg name="like"></icon-svg
                  >{{
                    quesionForm.likeCnt == null ? 0 : quesionForm.likeCnt
                  }}</el-button
                >
              </el-divider>

              <div class="main">
                <div class="top">
                  <el-badge
                    :value="quesionForm.commentCnt"
                    style="margin: 10px"
                    class="item2"
                  >
                    <div>
                      共有评论
                      <i class="el-icon-s-comment"></i>
                    </div>
                  </el-badge>
                </div>
                <div class="bottom">
                  <div>
                    <el-input
                      style="margin: 10px; width: 95%"
                      v-model="commentForm.content1"
                      placeholder="评论问题"
                    ></el-input>
                    <div style="margin: 10px">
                      <el-button
                        type="primary"
                        size="mini"
                        @click="
                          commitComment(
                            quesionForm.id,
                            1,
                            quesionForm.creator,
                            quesionForm.creatorName,
                            0
                          )
                        "
                        >发表</el-button
                      >
                    </div>
                  </div>

                  <hr />
                  <div style="width: 90%">
                    <div style="width: 95%">
                      <div
                        style="text-align: center"
                        v-if="questionComments.length == null"
                      >
                        <p>暂无备注，我来发表第一条备注！</p>
                      </div>
                      <div v-else>
                        <div
                          class="comment"
                          v-for="(questionComment, index1) in questionComments"
                          :key="questionComment.id"
                        >
                          <div
                            @mouseleave.stop="questionComment.show = false"
                            style="online: none"
                          >
                            <el-link
                              class="author"
                              href="https://element.eleme.io"
                              target="_blank"
                              >{{ questionComment.creatorName }}</el-link
                            >
                            <span
                              >{{
                                mydateformat(questionComment.gmtCreate)
                              }}：</span
                            >
                            <div>
                              {{ questionComment.content }}
                            </div>
                            <!-- 父评论的点赞和回复 -->
                            <div
                              class="noSelect"
                              @click="
                                addlikeCnt(questionComment.id, 1, index1, null)
                              "
                            >
                              <icon-svg name="like"></icon-svg>
                              <span style="margin-right: 20px">{{
                                questionComment.likeCnt == null
                                  ? 0
                                  : questionComment.likeCnt
                              }}</span>
                            </div>
                            <div
                              class="noSelect"
                              @click.stop="
                                () => {
                                  questionComment.show = true;
                                  curComment = questionComment;
                                  commentForm.content2 = '';
                                  commentForm.curPid = questionComment.id;
                                }
                              "
                            >
                              <span
                                >回复
                                {{ questionComment.commentCnt }}
                              </span>
                            </div>
                            <div
                              class="noSelect"
                              @click="
                                deleteComment(
                                  questionComment.id,
                                  1,
                                  index1,
                                  null,
                                  quesionForm.id,
                                  -1
                                )
                              "
                            >
                              <i class="el-icon-delete"></i>
                            </div>
                            <template
                              v-if="
                                questionComment.childComments.length > 0 &&
                                questionComment.show
                              "
                            >
                              <!-- 所有子回复 -->
                              <div
                                class="reply"
                                v-for="(
                                  subComment, index2
                                ) in questionComment.childComments"
                                :key="subComment.id"
                              >
                                <!-- 一条子回复 -->
                                <div class="top">
                                  <p>
                                    <span>{{ subComment.creatorName }}</span>
                                    <span
                                      >{{
                                        mydateformat(subComment.gmtCreate)
                                      }}回复</span
                                    >
                                    <span>@{{ subComment.replierName }}：</span>
                                  </p>
                                  <div>
                                    {{ subComment.content }}
                                  </div>
                                  <div
                                    class="noSelect"
                                    @click="
                                      addlikeCnt(
                                        subComment.id,
                                        2,
                                        index1,
                                        index2
                                      )
                                    "
                                  >
                                    <icon-svg name="like"></icon-svg>
                                    <span style="margin-right: 20px">{{
                                      subComment.likeCnt == null
                                        ? 0
                                        : subComment.likeCnt
                                    }}</span>
                                  </div>

                                  <div
                                    class="noSelect"
                                    @click.stop="
                                      () => {
                                        subComment.show = !subComment.show;
                                        curComment = subComment;
                                        commentForm.content2 = '';
                                        commentForm.curPid =
                                          subComment.parentId;
                                      }
                                    "
                                  >
                                    <span>回复</span>
                                  </div>
                                  <div
                                    class="noSelect"
                                    @click="
                                      deleteComment(
                                        subComment.id,
                                        2,
                                        index1,
                                        index2,
                                        quesionForm.id,
                                        questionComment.id
                                      )
                                    "
                                  >
                                    <i class="el-icon-delete"></i>
                                  </div>
                                </div>
                              </div>
                            </template>

                            <div v-if="questionComment.show" class="input">
                              <el-input
                                v-model="commentForm.content2"
                                :placeholder="`回复@${curComment.creatorName}`"
                              ></el-input>
                              <el-button
                                type="primary"
                                size="mini"
                                @click="
                                  commitComment(
                                    questionComment.id,
                                    2,
                                    curComment.commentator,
                                    curComment.creatorName,
                                    index1
                                  )
                                "
                                >发表</el-button
                              >
                            </div>
                          </div>
                        </div>

                        <div class="block pagination">
                          <el-pagination
                            @size-change="sizeChangeHandle"
                            @current-change="currentChangeHandle"
                            :current-page="pagination.pageIndex"
                            :page-sizes="[10, 20, 50, 100]"
                            :page-size="pagination.pageSize"
                            :total="pagination.totalPage"
                            layout="total, sizes, prev, pager, next, jumper"
                          >
                          </el-pagination>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </el-col>
        </el-row>
      </div>
    </el-dialog>
  </div>
</template>
   

<script>
import marked from "marked";
import { mavonEditor } from "mavon-editor";
import "mavon-editor/dist/css/index.css";
export default {
  components: { mavonEditor, marked },
  data() {
    return {
      questionlikeIds: [],
      likeListId1: [],
      likeListId2: [],
      type: 1,
      loading: false,
      pagination: {
        pageIndex: 1,
        pageSize: 10,
        totalPage: 1,
      },
      commentForm: {
        type: 1,
        content1: "",
        content2: "",
        curPid: "",
      },
      curComment: {},
      questionComments: [],
      visible: true,
      quesionForm: {
        id: 0,
        creatorName: "",
        creator: 0,
        title: "",
        description: "",
        tags: "",
        commentCnt: 0,
        likeCnt: 0,
        gmtCreate: "",
        gmtModified: "",
        content: "",
      },
    };
  },
  methods: {
    addlikeCnt(commentId, type, index1, index2) {
      let list1 = this.likeListId1;
      let list2 = this.likeListId2;
      let isLiked = false;
      let id = this.quesionForm.id;
      let id1 = id + "-" + index1;
      let id2 = id + "-" + index1 + "-" + index2;
      if (index2 == null) {
        if (list1.indexOf(id1) != -1) isLiked = true;
      }
      if (index2 != null) {
        if (list2.indexOf(id2) != -1) isLiked = true;
      }

      if (isLiked) {
        this.$http({
          url: this.$http.adornUrl(
            `/finalexam/question/unlikeComment/${commentId}`,
            false
          ),
          method: "get",
        }).then(({ data }) => {
          if (data.code == 0) {
            if (index2 == null) {
              this.likeListId1.splice(id1, 1);
            } else {
              this.likeListId2.splice(id2, 1);
            }

            if (type == 1) {
              this.questionComments[index1].likeCnt =
                this.questionComments[index1].likeCnt - 1;
            }
            if (type == 2) {
              this.questionComments[index1].childComments[index2].likeCnt =
                this.questionComments[index1].childComments[index2].likeCnt - 1;
            }
          }
        });
      } else {
        this.$http({
          url: this.$http.adornUrl(
            `/finalexam/question/likeComment/${commentId}`,
            false
          ),
          method: "get",
        }).then(({ data }) => {
          if (data.code == 0) {
            if (index2 == null) {
              this.likeListId1.push(id1);
            } else {
              this.likeListId2.push(id2);
            }
            if (type == 1) {
              this.questionComments[index1].likeCnt =
                this.questionComments[index1].likeCnt + 1;
            }
            if (type == 2) {
              this.questionComments[index1].childComments[index2].likeCnt =
                this.questionComments[index1].childComments[index2].likeCnt + 1;
            }
          }
        });
      }
    },
    deleteComment(commentId, type, index1, index2, questionId, pid) {
      this.$http({
        url: this.$http.adornUrl(
          `/finalexam/question/deleteComment/${commentId}/${questionId}/${pid}`,
          false
        ),
        method: "get",
      }).then(({ data }) => {
        if (data.code == 0) {
          if (type == 1) {
            this.questionComments.splice(index1, 1);
          }
          if (type == 2) {
            this.questionComments[index1].commentCnt =
              this.questionComments[index1].commentCnt - 1;
            this.questionComments[index1].childComments.splice(index2, 1);
          }

          this.quesionForm.commentCnt = this.quesionForm.commentCnt - 1;
        }
      });
    },
    addQuestionlikeCnt(id) {
      let likelist = this.questionlikeIds;
      let isLiked = false;
      if (likelist.indexOf(id) != -1) isLiked = true;
      if (isLiked) {
        this.$http({
          url: this.$http.adornUrl(
            "/finalexam/question/subQuestionLikeCnt",
            false
          ),
          method: "get",
          params: { id: id },
        }).then(({ data }) => {
          if (data.code == 0) {
            this.questionlikeIds.splice(likelist.indexOf(id), 1);
            this.quesionForm.likeCnt = data.question.likeCnt;
          }
        });
      } else {
        this.$http({
          url: this.$http.adornUrl(
            "/finalexam/question/addQuestionLikeCnt",
            false
          ),
          method: "get",
          params: { id: id },
        }).then(({ data }) => {
          if (data.code == 0) {
            this.questionlikeIds.push(id);
            this.quesionForm.likeCnt = data.question.likeCnt;
          }
        });
      }
    },
    commitComment(parentId, type, receiver, replierName, index) {
      let content = "";
      let commentPid = "";
      if (type == 1) {
        content = this.commentForm.content1;
      } else if (type == 2) {
        content = this.commentForm.content2;
        commentPid = this.commentForm.curPid;
        console.log("cpid:", commentPid);
      }
      this.$http({
        url: this.$http.adornUrl("/finalexam/question/saveComment", false),
        method: "post",
        data: this.$http.adornData(
          {
            commentPid: commentPid,
            content: content,
            parentId: parentId,
            type: type,
            receiver: receiver,
            replierName: replierName,
            questionId: this.quesionForm.id,
          },
          false
        ),
      }).then(({ data }) => {
        if (data.code == 0) {
          if (type == 1) {
            this.questionComments.unshift(data.comment);
            this.commentForm.content1 = "";
          }
          if (type == 2) {
            this.questionComments[index].commentCnt =
              this.questionComments[index].commentCnt + 1;
            this.questionComments[index].childComments.push(data.comment);
            this.commentForm.content2 = "";
          }
          this.quesionForm.commentCnt = this.quesionForm.commentCnt + 1;
        } else {
          this.$message.error(data.msg);
        }
      });
    },
    getQuestionComments(questionId) {
      this.$http({
        url: this.$http.adornUrl("/finalexam/question/queryComments", false),
        params: this.$http.adornParams({
          pageIndex: (this.pagination.pageIndex - 1) * this.pagination.pageSize,
          pageSize: this.pagination.pageSize,
          questionId: questionId,
        }),
        method: "get",
      }).then(({ data }) => {
        if (data.code == 0) {
          if (data.questionComments != null) {
            this.questionComments = data.questionComments;
            this.pagination.totalPage = this.quesionForm.commentCnt;
          } else {
            this.questionComments = [];
            this.pagination.totalPage = 0;
          }
        } else {
          this.$message.error(res.data.msg);
        }
      });
    },

    init(id) {
      this.visible = true;
      this.quesionForm.id = id || 0;
      this.$nextTick(() => {
        this.quesionForm.title = "";
        this.quesionForm.description = "";
        this.quesionForm.tags = "";
        this.quesionForm.gmtCreate = "";
        this.quesionForm.gmtModified = "";
        this.quesionForm.content = "";
        if (this.quesionForm.id) {
          this.loading = true;
          this.$http({
            url: this.$http.adornUrl(
              `/finalexam/question/info/${this.quesionForm.id}`
            ),
            method: "get", 
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.quesionForm.commentCnt = data.question.commentCnt;
              this.quesionForm.title = data.question.title;
              this.quesionForm.description = data.question.description;
              this.quesionForm.creator = data.question.creator;
              this.quesionForm.tags = data.question.tags;
              this.quesionForm.gmtCreate = data.question.gmtCreate;
              this.quesionForm.gmtModified = data.question.gmtModified;
              this.quesionForm.content = marked(data.question.content);
              this.quesionForm.creatorName = data.question.creatorName;
              this.quesionForm.likeCnt = data.question.likeCnt;
              this.getQuestionComments(data.question.id);
              console.log("ddd:", this.quesionForm);
            }
            this.loading = false;
          });
        }
      });
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pagination.pageSize = val;
      this.pagination.pageIndex = 1;
      let id = this.quesionForm.id;
      this.getQuestionComments(id);
    },
    // 当前页
    currentChangeHandle(val) {
      this.pagination.pageIndex = val;
      let id = this.quesionForm.id;
      this.getQuestionComments(id);
    },
  },
};
</script>

<style scoped>
.item :hover {
  padding-left: 10px;
  padding-bottom: 10px;
  color: #409eff;
  cursor: pointer;
}

.item2 {
  margin-top: 10px;
  margin-right: 40px;
}

.info {
  border-radius: 0px;
  border:1px solid rgb(226, 219, 219) ;
  widows: 200%;
  /* margin-left: -80px;
  margin-right: -80px; */
  margin-top: 40px;
  padding: 15px;
  width: 100%;
}
#artcle-info {
  padding: 20px;
  /* background-image: url(this.reviewMaterials.imageUrl);  */
  margin-bottom: 40px;
}
#artcle-content {
  padding: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);
}

#artcle-info .abstract {
  color: #ffffff;
  border-left: 3px solid #f56c6c;
  padding: 10px;
  background-color: rgba(126, 129, 135, 0.3);
}
/* .main .bottom {
  padding: 20px 0;
} */
.main .top {
  border-bottom: 1px solid #e9e9e9;
}
.main {
  margin-top: 50px;
  border: 1px solid #e9e9e9;

  background: #eaeff5;
  /* margin-left: 10px; */
}
#artcle-info .timeAndView {
  padding: 20px;
  line-height: 30px;
  font-size: 16px;
  color: black;
}
.comment {
  padding-bottom: 2%;
  padding-top: 1%;
  padding-left: 2%;
  margin-bottom: 1%;
  border-bottom: 1px solid #f7d4d4;
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

.author {
  font-weight: 900;
}
p {
  font-size: 12px;
  color: #060606;
  margin-top: 0.5%;
  margin-bottom: 0.5%;
}
.firstTextArea {
  padding-bottom: 0%;
  padding-top: 1%;
  padding-left: 5%;
}
.reply {
  padding-bottom: 1%;
  padding-top: 1%;
  padding-left: 10%;
}
div .noSelect {
  -moz-user-select: none; /*mozilar*/
  -webkit-user-select: none; /*webkit*/
  -ms-user-select: none; /*IE*/
  user-select: none;
  display: inline;
}
button {
  margin-left: 88%;
  margin-top: 1%;
}

.input {
  margin-left: 10%;
}
</style>