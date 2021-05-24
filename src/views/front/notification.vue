<template>
  <div>
    <el-button-group>
      <el-button type="primary" @click="getAllNotifications(1,null)">已读评论</el-button>
      <el-button type="primary" @click="getAllNotifications(0,null)">未读评论</el-button>
       <el-button type="primary" @click="getAllNotifications(null,3)">驳回留言</el-button>
    </el-button-group>
    <div>
      <div v-if="notifications.length >= 1">
        <div class="block">
          <el-timeline
            v-for="(notification, index) in notifications"
            :key="index"
          >
            <el-timeline-item
              :timestamp="mydateformat(notification.gmtCreate)"
              placement="top"
            >
              <el-card>
                <p>
                  {{ notification.notifierName }} 
                  <span v-if="notification.type==3">老师退回了你的任务留言</span>
                  <span
                    v-else
                    class="question"
                    @click="readMoreHandle(notification.questionId)"
                  >
                    <span v-if="notification.type == 1">评论了你的提问:</span>
                    <span v-if="notification.type == 2">回复了你的评论:</span>
                  </span> 
                </p>
                <h4 style="padding-left: 10px">{{ notification.content }}</h4>
                <el-button type="text" v-if="notification.type!=3"  @click="dialogReplyVisible = true"
                  >回复TA</el-button
                >
              </el-card>
              <el-dialog title="回复" :visible.sync="dialogReplyVisible">
                <el-input v-model="replyConent"></el-input>
                <div slot="footer" class="dialog-footer">
                  <el-button @click="dialogReplyVisible = false"
                    >取 消</el-button
                  >
                  <el-button
                    type="primary"
                    @click="
                      replyComment(
                        notification.questionId,
                        notification.notifierName,
                        notification.notifier,
                        notification.commentPid
                      )
                    "
                    >确 定</el-button
                  >
                </div>
              </el-dialog>
            </el-timeline-item>
          </el-timeline>
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
        <question v-if="readMoreIsVisible" ref="readMoreRef"></question>
      </div>
      <div v-else>
        <el-divider content-position="center">暂无</el-divider>
      </div>
    </div>
  </div>
</template>
<script>
import question from "./question";
export default {
  components: {
    question,
  },
  data() {
    return {
      replyConent: "",
      dialogReplyVisible: false,
      readMoreIsVisible: false,
      pagination: {
        pageIndex: 1,
        pageSize: 10,
        totalPage: 1,
      },
      notifications: [],
    };
  },
  created() {
    this.getAllNotifications(1);
  },
  methods: { 
    replyComment(questionId, receiverName, receiver, commentPid) {
      this.$http({
        url: this.$http.adornUrl("/finalexam/question/replyComment", false),
        method: "post",
        data: this.$http.adornData(
          {
            type: 2,
            content: this.replyConent,
            replierName: receiverName,
            receiver: receiver,
            questionId: questionId,
            commentPid: commentPid,
          },
          false
        ),
      }).then(({ data }) => {
        if (data.code == 0) {
          this.dialogReplyVisible = false;
         this.$message.success("回复成功")
        }
      });
    },
    readMoreHandle(id) {
      this.readMoreIsVisible = true;
      console.log("xxxx");
      this.$nextTick(() => {
        this.$refs.readMoreRef.init(id);
      });
    },
    getAllNotifications(status,type) {
      this.$http({
        url: this.$http.adornUrl(
          "/finalexam/question/get/notifications",
          false
        ),
        method: "get",
        params: this.$http.adornParams({
          page: this.pagination.pageIndex,
          limit: this.pagination.pageSize,
          status: status,
          type:type
        }),
      }).then(({ data }) => { 
        this.notifications = data.page.list;
        this.pagination.totalPage = data.page.totalCount;
      });
    },
    getQuestion(questionId) {
      this.$http({
        url: this.$http.adornUrl(
          `/finalexam/question/info/${questionId}`,
          false
        ),
        method: "get",
      }).then(({ data }) => {});
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pagination.pageSize = val;
      this.pagination.pageIndex = 1;
      this.getAllNotifications(1);
    },
    // 当前页
    currentChangeHandle(val) {
      this.pagination.pageIndex = val;
      this.getAllNotifications(1);
    },
  },
};
</script>

<style scoped>
.author {
  font-weight: 900;
  color: rgb(144, 219, 238);
}
.question {
  display: inline;
  cursor: pointer;
}
.question:hover {
  text-decoration: underline;
  color: rgb(144, 219, 238);
}
</style>
