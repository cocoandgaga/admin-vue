<template>
  <div>
    <el-form ref="searchForm" :model="searchForm" style="overflow: hidden">
      <el-form-item>
        <el-select
          v-model="searchForm.subjectName"
          placeholder="请选择要搜索任务的科目"
          filterable
          clearable
          @change="getBoard"
        >
          <el-option
            v-for="item in searchForm.deptNames"
            :key="item.id"
            :label="item.name"
            :value="item.name"
          >
          </el-option>
        </el-select>
      </el-form-item>
    </el-form>
    <div class="board" v-loading="loading.board">
      <div class="board__header" ref="boardHeader">
        <div class="board-column__wrapper">
          <div
            v-for="(column, index) in columnList"
            :key="index"
            class="board-column"
            :class="'flex-' + column.children.length"
          >
            {{ column.columnName }}
          </div>
        </div>
        <div class="board-status__wrapper">
          <div
            v-for="(statusColumn, index) in statusColumnList"
            :key="index"
            class="board-status"
            :class="statusColumn.typeId == 1 ? 'story' : 'task'"
          >
            {{ statusColumn.statusColumnName }}
            <span>({{ statusColumn.cardNum || 0 }})</span>
          </div>
        </div>
      </div>

      <div class="board__body" ref="boardBody">
        <div v-if="laneList.length == 0" style="margin-left: 10px">
          <el-divider>无任务发布</el-divider>
        </div>
        <template v-else>
          <div
            v-for="(lane, laneIndex) in laneList"
            :key="lane.id"
            class="lane"
          >
            <draggable
              v-for="(column, columnIndex) in lane.columnList"
              :key="column.id"
              class="card-container"
              draggable=".card__wrapper"
              filter=".story"
              animation="150"
              scroll-speed="30"
              :list="column"
              :group="statusColumnList[columnIndex].typeId + '_' + laneIndex"
              @add="(evt) => handleDragAdd(evt, columnIndex)"
              @end="(evt) => handleDrag(evt, laneIndex, columnIndex)"
            >
              <div
                v-for="(card, cardIndex) in column"
                :key="card.stuTaskId"
                class="card__wrapper"
                :class="
                  statusColumnList[columnIndex].typeId == 1 ? 'story' : 'task'
                "
                @click="
                  toEditCard(
                    card,
                    statusColumnList[columnIndex].typeId,
                    lane.userTaskId,
                    laneIndex
                  )
                "
              >
                <div class="card__tool">
                  <i
                    class="el-icon-document-add"
                    title="留言"
                    v-if="card.stuTaskId == null"
                    @click.stop="raiseProblem(card)"
                  ></i>
                  <i
                    class="el-icon-document"
                    title="详情"
                    v-if="card.stuTaskId == null"
                    @click.stop="getCardDetail(card)"
                  ></i>
                  <i
                    v-if="statusColumnList[columnIndex].typeId == 2"
                    class="el-icon-delete"
                    title="删除"
                    @click.stop="deleteCard(column, cardIndex)"
                  ></i>
                  <i
                    class="el-icon-check"
                    title="任务确认汇报"
                    v-if="card.stuTaskId == null"
                    @click.stop="getUserTaskStatus(card, 1)"
                  ></i>
                  <i
                    class="el-icon-close"
                    title="取消任务确认汇报"
                    v-if="card.stuTaskId == null"
                    @click.stop="getUserTaskStatus(card, 0)"
                  ></i>
                </div>
                <div class="card">
                  <div class="card__header">
                    <template v-if="statusColumnList[columnIndex].typeId == 1">
                      <span>{{ card.userName }}</span>
                      <span> #{{ card.subjectName }}</span>
                      <span>分数:{{ card.score }}</span>
                    </template>
                    <template
                      v-else-if="statusColumnList[columnIndex].typeId == 2"
                    >
                      <span>{{ card.stuName }}</span>
                      <span> #{{ card.subjectName }}</span>
                    </template>
                  </div>
                  <div class="card__body">
                    {{
                      statusColumnList[columnIndex].typeId == 1
                        ? card.userTaskName
                        : card.stuTaskName
                    }}
                  </div>
                  <div
                    v-if="statusColumnList[columnIndex].typeId == 2"
                    class="card__footer"
                  >
                    <el-button
                      type="text"
                      icon="el-icon-chat-dot-round"
                      @click.stop="toMessage(card)"
                    >
                      <template v-if="card.messageCount">{{
                        card.messageCount
                      }}</template>
                    </el-button>
                  </div>
                </div>
              </div>
              <el-link
                slot="footer"
                v-if="columnIndex == 2"
                icon="el-icon-plus"
                type="primary"
                class="full-width"
                :underline="false"
                @click="toAddCard(column, lane.userTaskId, lane.subjectName)"
              >
                添加任务
              </el-link>
            </draggable>
          </div>
        </template>
      </div>
    </div>
    <!-- 故事 -->
    <el-dialog
      title="任务"
      :visible.sync="storyDialogVisible"
      :close-on-click-modal="false"
      width="600px"
      @close="restStoryDialog"
    >
      <el-form
        :model="userTaskForm"
        ref="userTaskForm"
        disabled
        label-width="80px"
      >
        <el-form-item label="任务名称">
          <el-input
            v-model="userTaskForm.userTaskName"
            maxlength="60"
            show-word-limit
          ></el-input>
        </el-form-item>
        <el-form-item label="任务描述">
          <el-input
            v-model="userTaskForm.userTaskDescription"
            type="textarea"
            maxlength="200"
            show-word-limit
            :autosize="{ minRows: 2, maxRows: 6 }"
          ></el-input>
        </el-form-item>
        <el-form-item label="开始时间">
          <el-date-picker
            format="yyyy-MM-dd HH:mm:ss"
            v-model="userTaskForm.startTime"
            :disabled="true"
            type="date"
          >
          </el-date-picker>
        </el-form-item>
        <el-form-item label="结束时间">
          <el-date-picker
            format="yyyy-MM-dd HH:mm:ss"
            v-model="userTaskForm.endTime"
            :disabled="true"
            type="date"
          >
          </el-date-picker>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="storyDialogVisible = false">关闭</el-button>
      </span>
    </el-dialog>

    <el-dialog
      :title="taskOptName + '任务'"
      :visible.sync="taskDialogVisible"
      :close-on-click-modal="false"
      width="600px"
      @close="restTaskDialog"
    >
      <el-form
        :model="taskForm"
        ref="taskForm"
        :rules="taskFormRules"
        label-width="80px"
      >
        <el-form-item label="任务名称" prop="stuTaskName">
          <el-input
            v-model="taskForm.stuTaskName"
            placeholder="请输入任务名称"
            maxlength="60"
            show-word-limit
          ></el-input>
        </el-form-item>
        <el-form-item label="任务描述" prop="stuTaskDescription">
          <el-input
            v-model="taskForm.stuTaskDescription"
            type="textarea"
            placeholder="请输入任务描述"
            maxlength="200"
            show-word-limit
            :autosize="{ minRows: 2, maxRows: 6 }"
          ></el-input>
        </el-form-item>
        <el-form-item label="开始时间">
          <el-date-picker
            format="yyyy-MM-dd HH:mm:ss"
            v-model="taskForm.startTime"
            :disabled="true"
            type="date"
          >
          </el-date-picker>
        </el-form-item>
        <el-form-item label="结束时间">
          <el-date-picker
            format="yyyy-MM-dd HH:mm:ss"
            v-model="taskForm.endTime"
            :disabled="true"
            type="date"
          >
          </el-date-picker>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="taskDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitTask">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 留言 -->
    <el-dialog
      :title="currentCard.stuTaskName"
      :visible.sync="messageDialog.visible"
      :close-on-click-modal="false"
      width="600px"
      @close="handleMessageDialogClose"
    >
      <div
        v-loading="loading.messageloading"
        element-loading-text="拼命加载中"
        element-loading-spinner="el-icon-loading"
      >
        <el-timeline
          v-show="messageList.length > 0"
          ref="messageWrapper"
          class="message-wrapper"
          v-loading="messageDialog.contentLoading"
        >
          <el-timeline-item
            v-for="message in messageList"
            :key="message.id"
            :timestamp="
              message.userName + ' ' + mydateformat(message.createTime)
            "
            placement="top"
          >
            {{ message.content }}
          </el-timeline-item>
        </el-timeline>
        <el-divider v-if="messageList.length == 0">暂无留言</el-divider>
      </div>
      <el-form :model="messageForm" ref="messageForm">
        <el-form-item
          prop="content"
          :rules="[
            { required: true, message: '请输入留言', trigger: 'change' },
          ]"
        >
          <el-input
            type="textarea"
            v-model="messageForm.content"
            placeholder="请输入留言"
            maxlength="255"
            @input="messageContentChange"
            :autosize="{ minRows: 2, maxRows: 4 }"
          ></el-input>
        </el-form-item>
        <el-form-item class="text-right">
          <el-button
            type="primary"
            :loading="messageDialog.submitLoading"
            @click="submitMessage"
          >
            发 送
          </el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>
<script>
import draggable from "vuedraggable";
export default {
  components: {
    draggable,
  },
  computed: {
    taskOptName() {
      let taskOptName = "";
      if (!this.taskForm.stuTaskId) {
        taskOptName = "添加";
      } else {
        taskOptName = "修改";
      }
      return taskOptName;
    },
    taskOptType() {
      let taskOptType = "";
      if (!this.taskForm.stuTaskId) {
        taskOptType = "add";
      } else {
        taskOptType = "edit";
      }
      return taskOptType;
    },
  },
  data() {
    return {
      loading: {
        board: false,
        submitTask: false,
        messageloading: false,
      },
      value: "未完成",
      currentCard: {},
      messageList: [],
      laneList: [],
      laneIndex: "",
      taskDialogVisible: false,
      storyDialogVisible: false,
      userTaskForm: {
        userTaskName: "",
        userTaskDescription: "",
        startTime: "",
        endTime: "",
      },
      taskForm: {
        stuTaskId: "",
        stuTaskName: "",
        stuTaskDescription: "",
        stuId: "",
        stuName: "",
        userTaskId: "",
        deptId: "",
        startTime: "",
        endTime: "",
        subjectName: "",
      },
      messageForm: {
        content: "",
      },
      taskFormRules: {
        stuTaskName: [{ required: true, message: "请输入任务名称" }],
        stuTaskDescription: [{ required: true, message: "请输入任务描述" }],
      },
      searchForm: {
        deptNames: [],
        subjectName: "",
      },
      currentColumn: "",
      dragToColumnIndex: "",
      dragToColumnIndex: "",
      messageDialog: {
        title: "",
        visible: false,
        contentLoading: false,
        submitLoading: false,
      },
      columnList: [
        {
          columnId: 1,
          columnName: "老师发布任务",
          children: ["发布中", "已截止"],
          cardNum: "",
        },
        {
          columnId: 2,
          columnName: "待办列表",
          children: ["待办"],
          cardNum: "",
        },
        {
          columnId: 3,
          columnName: "任务完成情况",
          children: ["进行中", "已完成"],
          cardNum: "",
        },
      ],
      statusColumnList: [
        {
          columnId: 1,
          statusColumnId: 1,
          typeId: 1,
          statusColumnName: "发布中",
          sort: 1,
          cardNum: "",
        },
        {
          columnId: 1,
          statusColumnId: 2,
          typeId: 1,
          statusColumnName: "已截止",
          sort: 1,
          cardNum: "",
        },
        {
          columnId: 2,
          statusColumnId: 3,
          typeId: 2,
          statusColumnName: "待办",
          sort: 2,
          cardNum: "",
        },
        {
          columnId: 3,
          statusColumnId: 4,
          typeId: 2,
          statusColumnName: "进行中",
          sort: 3,
          cardNum: "",
        },
        {
          columnId: 3,
          statusColumnId: 5,
          typeId: 2,
          statusColumnName: "已完成",
          sort: 3,
          cardNum: "",
        },
      ],
    };
  },
  mounted() {
    this.getDeptNames();
    this.getBoard();
  },
  methods: {
    getDeptNames() {
      this.$http({
        url: this.$http.adornUrl("/finalexam/category/level/subjects", false),
        method: "get",
      }).then(({ data }) => {
        if (data.code == 0) {
          this.searchForm.deptNames = data.subjects;
        }
      });
    },
    getBoard() {
      this.loading.board = true;
      this.$http({
        url: this.$http.adornUrl("/finalexam/board/queryBoardData", false),
        method: "post",
        data: this.$http.adornData(
          {
            subjectName: this.searchForm.subjectName,
            deptId: null,
            stuId: null,
          },
          false
        ),
      }).then((data) => {
        let userStoryCardList = data.data.userStoryCardList;
        let laneList = [];
        let statuslen = this.statusColumnList.length;
        for (let i = 0; i < userStoryCardList.length; i++) {
          let {
            userTaskId,
            subjectName,
            taskCardList: cardList,
          } = userStoryCardList[i];
          //taskCardList是匹配的模式，cardList才是变量。真正被赋值的是变量cardList，而不是模式taskCardList
          let lane = {
            userTaskId,
            subjectName,
            columnList: this.statusColumnList.map((item) => []),
          };
          let storyCard = { ...userStoryCardList[i] };
          delete storyCard.taskCardList;
          cardList.unshift(storyCard); //unshift() 方法可向数组的开头添加一个或更多元素，并返回新的长度。
          for (let i = 0; i < cardList.length; i++) {
            let card = cardList[i];
            for (let i = 0; i < statuslen; i++) {
              let { columnId, statusColumnId } = this.statusColumnList[i];
              if (
                card.position != null &&
                card.position.columnId == columnId &&
                card.position.statusColumnId == statusColumnId
              ) {
                lane.columnList[i].push(card);
              }
            }
          }
          laneList.push(lane);
        }
        this.laneList = laneList;
        console.log("laneList:", laneList);
        this.countCard();
        this.loading.board = false;
      });
    },
    getUserTaskStatus(card, status) {
      this.$http({
        url: this.$http.adornUrl(
          `/finalexam/usertask/updateCompleteStatus/${card.userTaskId}/${status}`,
          false
        ),
        method: "get",
      }).then(({ data }) => {
        if (data.code == 0) {
          if (status == 0) this.$message.success("取消汇报确认任务成功");
          else this.$message.success("任务确认汇报成功");
        }
      });
    },
    handleDragAdd(evt, columnIndex) {
      this.dragToColumnIndex = columnIndex;
    },
    //move check
    handleDrag(evt, laneIndex, columnIndex) {
      console.log(evt);
      const { newDraggableIndex, oldDraggableIndex } = evt;
      if (this.dragToColumnIndex == null) {
        if (newDraggableIndex == oldDraggableIndex) {
          return false;
        }
      }
      const cardColumnIndex =
        this.dragToColumnIndex != null ? this.dragToColumnIndex : columnIndex;
      if (this.statusColumnList[cardColumnIndex].sort === 2) {
        // 拖回待办区，提示是否清空任务
        this.$confirm("是否清空该卡片下留言？", "提示", {
          type: "warning",
          confirmButtonText: "是",
          cancelButtonText: "否",
          closeOnClickModal: false,
        })
          .then(() => {
            this.handleDragEnd(evt, laneIndex, columnIndex, true);
          })
          .catch((e) => {
            this.handleDragEnd(evt, laneIndex, columnIndex, false);
          });
      } else {
        this.handleDragEnd(evt, laneIndex, columnIndex, false);
      }
    },
    // move
    handleDragEnd(evt, laneIndex, columnIndex, clearFlag) {
      const { newDraggableIndex, oldDraggableIndex } = evt;
      const cardColumnIndex =
        this.dragToColumnIndex != null ? this.dragToColumnIndex : columnIndex;
      this.dragToColumnIndex = null;
      let columnList = this.laneList[laneIndex].columnList;
      let card = columnList[cardColumnIndex][newDraggableIndex];
      let params = {
        sort: newDraggableIndex + 1, //单列在这一个的上下排序
        stuTaskId: card.stuTaskId,
        userTaskId: this.laneList[laneIndex].userTaskId,
        type: this.statusColumnList[cardColumnIndex].typeId,
        columnId: this.statusColumnList[cardColumnIndex].columnId,
        statusColumnId: this.statusColumnList[cardColumnIndex].statusColumnId,
        columnTypeSort: this.statusColumnList[cardColumnIndex].sort, //大列排序
        clearTask: clearFlag,
      };
      const loading = this.$loading({ target: evt.to.parentNode });
      this.$http({
        url: this.$http.adornUrl("/finalexam/board/moveCardItem", false),
        method: "post",
        data: this.$http.adornData(params, false),
      })
        .then((res) => {
          if (res.data != null && res.data.code == 0) {
            let taskcard = res.data.taskCard;
            card.position.statusColumnId = taskcard.position.statusColumnId;
            card.position.columnId = taskcard.position.columnId;
            card.startTime = taskcard.startTime;
            card.endTime = taskcard.endTime;
            this.countCard();
            loading.close();
          } else {
            this.$message.error(res.data.msg);
          }
        })
        .catch((e) => {
          columnList[cardColumnIndex].splice(newDraggableIndex, 1);
          columnList[columnIndex].splice(oldDraggableIndex, 0, card);
          loading.close();
          this.$message.error("任务已截止或未开始,不可拖动");
        });
    },
    handleMessageDialogClose() {
      this.$refs.messageForm.resetFields();
      this.messageList = [];
      this.currentCard = {};
    },

    //卡片留言去除开头空格
    messageContentChange(val) {
      this.messageForm.content = val.replace(/(^\s*)/g, "");
    },
    getMessageList() {
      this.messageDialog.contentLoading = true;
      this.loading.messageloading = true;
      this.$http({
        url: this.$http.adornUrl(
          "/finalexam/cardMessage/queryCardMessage",
          false
        ),
        method: "post",
        data: this.$http.adornData(
          {
            stuTaskId: this.currentCard.stuTaskId,
            type: this.currentCard.position.type, //1 老师 2学生
            userTaskId: this.currentCard.userTaskId,
          },
          false
        ),
      })
        .then((res) => {
          if (res.data != null && res.data.code == 0) {
            this.messageList = res.data.cardMessages;
            this.$nextTick(() => {
              this.$refs.messageWrapper.$el.scrollTop = this.$refs.messageWrapper.$el.scrollHeight;
            });
          } else {
            this.$message.error("无法获取留言信息");
          }
        })
        .finally((err) => {
          this.messageDialog.contentLoading = false;
          this.loading.messageloading = false;
        });
    },

    submitMessage() {
      this.$refs.messageForm.validate((valid) => {
        if (valid) {
          const dto = {};
          if (this.currentCard.position.type == 1) {
            dto = {
              stuTaskId: this.currentCard.stuTaskId,
              content: this.messageForm.content,
              type: this.currentCard.position.type,
              userTaskId: this.currentCard.userTaskId,
              score: this.currentCard.score,
              subjectName: this.currentCard.subjectName,
              taskName: this.currentCard.userTaskName,
            };
          } else if (this.currentCard.position.type == 2) {
            dto = {
              stuTaskId: this.currentCard.stuTaskId,
              content: this.messageForm.content,
              type: this.currentCard.position.type,
              userTaskId: this.currentCard.userTaskId,
            };
          } else {
            this.$message.error("卡片类型未知错误");
          }
          this.messageDialog.submitLoading = true;
          this.$http({
            url: this.$http.adornUrl(
              "/finalexam/cardMessage/saveCardMessage",
              false
            ),
            method: "post",
            data: this.$http.adornData(dto, false),
          })
            .then((res) => {
              if (res.data != null && res.data.code == 0) {
                this.$message.success("发送成功");
                this.$refs.messageForm.resetFields();
                this.currentCard.messageCount = this.messageList.length + 1;
                this.getMessageList();
              } else {
                this.$message.error(res.data.msg);
              }
            })
            .finally(() => {
              this.messageDialog.submitLoading = false;
            });
        } else {
          return false;
        }
      });
    },
    toMessage(card) {
      this.currentCard = card;
      this.messageDialog.visible = true;
      this.getMessageList();
    },
    restTaskDialog() {
      this.$refs.taskForm.resetFields();
    },
    restStoryDialog() {
      this.$refs.userTaskForm.resetFields();
    },
    raiseProblem(card) {
      this.currentCard = card;
      this.messageDialog.visible = true;
      this.getMessageList();
    },

    deleteCard(column, index) {
      const { stuTaskId, userTaskId, stuId } = column[index];
      let target = event.target ? event.target : event.srcElement;
      target = target.parentNode.parentNode;

      this.$confirm("确认删除该卡片?", "提示", {
        type: "warning",
      }).then(() => {
        const loading = this.$loading({ target });
        this.$http({
          url: this.$http.adornUrl(
            `/finalexam/stutask/deleteTaskCard/${userTaskId}/${stuTaskId}`,
            false
          ),
          method: "get",
        })
          .then((res) => {
            if (res.data != null && res.data.code == 0) {
              column.splice(index, 1);
              this.$message.success("删除成功");
            } else {
              this.$message.err(res.data.msg);
            }
          })
          .finally(() => {
            this.getBoard();
            loading.close();
          });
      });
    },
    toAddCard(column, userTaskId, subjectName) {
      this.currentColumn = column;
      this.taskDialogVisible = true;
      this.taskForm.userTaskId = userTaskId;
      this.taskForm.stuTaskName = "";
      this.taskForm.stuTaskId = "";
      this.taskForm.stuTaskDescription = "";
      this.taskForm.subjectName = subjectName;
    },
    submitTask() {
      this.$refs.taskForm.validate((valid) => {
        if (valid) {
          this.loading.submitTask = true;
          this.$http({
            url: this.$http.adornUrl(
              `/finalexam/stutask/${this.taskOptType}TaskCard`,
              false
            ),
            method: "post",
            data: this.$http.adornData(this.taskForm, false),
          }).then((res) => {
            console.log("res", res.data);
            if (res.data != null && res.data.code == 0) {
              let data = res.data.stuTask;
              this.$message.success(this.taskOptName + "成功");
              if (this.taskOptType == "add") {
                this.currentColumn.push(data);
                this.currentColumn = null;
              }
              if (this.taskOptType == "edit") {
                Object.keys(data).forEach((key) => {
                  this.currentCard[key] = data[key];
                });
                this.currentCard = {};
              }
              this.taskDialogVisible = false;
              this.loading.submitTask = false;
              this.countCard();
            } else {
              this.$message.error(res.data.msg);
            }
          });
        } else {
          return false;
        }
      });
    },
    getCardDetail(card) {
      //故事卡片
      this.storyDialogVisible = true;

      this.$nextTick(() => {
        this.userTaskForm = {
          userTaskName: card.userTaskName,
          userTaskDescription: card.userTaskDescription,
          startTime: card.startTime,
          endTime: card.endTime,
        };
      });
    },
    toEditCard(card, typeId, userTaskId, laneIndex) {
      this.currentCard = card;
      this.laneIndex = laneIndex;
      if (typeId == 2) {
        //任务卡片
        this.taskDialogVisible = true;
        this.$nextTick(() => {
          this.taskForm = {
            stuTaskId: card.stuTaskId,
            stuTaskName: card.stuTaskName,
            stuTaskDescription: card.stuTaskDescription,
            stuId: card.stuId,
            userTaskId: userTaskId,
            startTime: card.startTime,
            endTime: card.endTime,
          };
          this.$nextTick(() => {
            this.$refs.taskForm.clearValidate();
          });
        });
      } else if (typeId == 1) {
        //故事卡片
        //编辑用户故事
        this.$message.success("不可编辑老师发布的卡片");
      }
    },
    countCard() {
      this.statusColumnList.forEach((statusColumn, index) => {
        let cardNum = 0;
        this.laneList.forEach((lane) => {
          cardNum += lane.columnList[index].length;
        });
        this.$set(statusColumn, "cardNum", cardNum);
      });
    },
  },
};
</script>

<style lang="scss" scoped>
$--column-min-width: 138px;
$--border-color-light: #74cef8ea;
$--color-danger: #df3434ea;
$--color-primary: #74cef8ea;
$--color-text-primary: rgba(230, 46, 46, 0);
$--color-text-default: rgba(230, 46, 46, 0);
// board
.board {
  margin: -16px;
  margin-top: 0px;
  background: rgba(0, 0, 0, 0) url("~@/assets/img/whitebg.jpg") no-repeat scroll
    100% 100%;
  background-size: auto;
  background-size: cover;
}
.board__header {
  // overflow-y: scroll;
  overflow-x: hidden;
  font-size: 12px;
  line-height: 24px;
  .board-column__wrapper,
  .board-status__wrapper {
    display: flex;
    text-align: center;
  }
  .board-column,
  .board-status {
    box-sizing: border-box;
    padding: 0;
    border-top: 1px solid $--border-color-light;
    border-right: 1px solid $--border-color-light;
    &:first-child {
      border-left: 1px solid $--border-color-light;
    }
  }
  .board-status {
    border-bottom-width: 2px;
    border-bottom-style: solid;
    &.story {
      border-bottom-color: #409eff;
    }
    &.task {
      border-bottom-color: #b3d8ff;
    }
  }
  .board-column {
    &.flex-1 {
      flex: 1;
      min-width: $--column-min-width;
    }
    &.flex-2 {
      flex: 2;
      min-width: $--column-min-width * 2;
    }
    &.flex-3 {
      flex: 3;
      min-width: $--column-min-width * 3;
    }
  }
  .board-status {
    flex: 1;
    min-width: $--column-min-width;
  }
}
.board__body {
  font-size: 12px;
  // overflow-y: scroll;
  height: calc(100vh - 256px);
}
.fullscreen .board__body {
  height: calc(100vh - 157px);
}
.lane {
  display: flex;
}

// card
.card-container {
  flex: 1;
  min-width: $--column-min-width;
  box-sizing: border-box;
  padding: 5px;
  background-color: #fff;
  border-right: 1px solid $--border-color-light;
  border-top: 1px solid $--border-color-light;
  border-bottom: 1px solid $--border-color-light;
  &:first-child {
    border-left: 1px solid $--border-color-light;
  }
  > .el-link {
    font-size: 12px;
    margin: 5px 0;
  }
}
.card__wrapper {
  position: relative;
  overflow: visible;
  &.sortable-ghost {
    .card {
      opacity: 0.8;
    }
    .el-badge__content,
    .card__header,
    .card__body,
    .card__footer {
      visibility: hidden;
    }
  }
  &.sortable-drag {
    .card {
      transform: rotateZ(5deg);
    }
  }
  &.story .card {
    background-color: #ecf5ff;
  }
  &:hover {
    .card__tool {
      display: block;
    }
  }
  .card__tool {
    cursor: pointer;
    display: none;
    position: absolute;
    z-index: 999;
    border-radius: 4px;
    top: 1px;
    right: 6px;
    color: $--color-primary;
    background-color: rgba(255, 255, 255, 0.8);
    > i {
      padding: 5px;
    }
  }
}
.card {
  cursor: move;
  margin: 5px;
  padding: 10px;
  border: 1px solid $--border-color-light;
  border-radius: 4px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  background-color: #fff;
  overflow: hidden;
  &:hover {
    border-color: $--border-color-light;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.2);
  }
  .card__header {
    overflow: hidden;
    margin-bottom: 6px;
  }
  .card__body {
    font-size: 15px;
  }
  .card__footer {
    margin-top: 6px;
    text-align: right;
    .el-button--text {
      padding: 0px;
    }
  }
  &.el-badge {
    display: block;
    ::v-deep .el-badge__content.is-fixed.is-dot {
      left: -8px;
      right: auto;
      top: 7px;
    }
  }
}

.message-wrapper {
  overflow: auto;
  margin-bottom: 20px;
  max-height: calc(55vh - 121px);
  .el-timeline-item:last-child {
    padding-bottom: 0px;
  }
  + .el-divider {
    margin-top: 8px;
  }
}

.text-danger {
  color: $--color-danger !important;
}
.is-error /deep/ .el-input__inner {
  border-color: $--color-danger !important;
}
</style>
