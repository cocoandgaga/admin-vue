<template>
  <div>
    <div class="size">
      <el-input
        class="distance"
        placeholder="请输入搜索文件名"
        v-model="searchStr" 
      >
        <i slot="prefix" class="el-input__icon el-icon-search"></i>
        <el-button slot="append" icon="el-icon-search" @click="getAllAttachments"></el-button>
      </el-input>
      <div v-for="(attachment, index) in attachments" :key="index">
        <div class="item">
          <img
            src="~@/assets/img/pdf.png"
            v-if="attachment.fileName.indexOf('pdf') > 0"
          />
          <img
            src="~@/assets/img/doc.png"
            v-if="
              attachment.fileName.indexOf('doc') > 0 ||
              attachment.fileName.indexOf('docx') > 0
            "
          />
          <img
            src="~@/assets/img/zip.png"
            v-if="attachment.fileName.indexOf('zip') > 0"
          />
          <img
            src="~@/assets/img/video.png"
            v-if="attachment.fileName.indexOf('mp4') > 0"
          />
          <img
            src="~@/assets/img/ppt.png"
            v-if="attachment.fileName.indexOf('pptx') > 0"
          />
          <img
            src="~@/assets/img/jpg.png"
            v-if="attachment.fileName.indexOf('jpg') > 0"
          />
          <img
            src="~@/assets/img/png.png"
            v-if="attachment.fileName.indexOf('png') > 0"
          />
          <img
            src="~@/assets/img/txt.png"
            v-if="attachment.fileName.indexOf('txt') > 0"
          />
          <a :href="'https://view.officeapps.live.com/op/view.aspx?src='+attachment.url" target="_blank">
            <span>{{ attachment.fileName }}</span>
          </a>
        </div>
        <div style="width: 10%; display: inline-block">
          <el-button
            type="primary"
            @click="download(attachment.url)"
            icon="el-icon-download"
            size="small"
            circle
          ></el-button>
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
    <div></div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      searchStr: "",
      pagination: {
        pageIndex: 1,
        pageSize: 10,
        totalPage: 1,
      },
      attachments: [],
    };
  },
  created() {
    this.getAllAttachments();
  },
  methods: {
    download(url) {
      let uri = url.substring(url.indexOf("=") + 1, url.length);
      window.open(uri, "_blank");
    },
    getAllAttachments() {
      this.$http({
        url: this.$http.adornUrl("/finalexam/attachment/get", false),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          value: this.searchStr,
        }),
      }).then(({ data }) => {
        if (data.code == 0) {
          this.attachments = data.page.list;
          this.pagination.totalPage = data.page.totalCount;
        } else {
          this.$message.error(data.msg);
        }
      });
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pagination.pageSize = val;
      this.pagination.pageIndex = 1;
      this.getAllAttachments();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pagination.pageIndex = val;
      this.getAllAttachments();
    },
  },
};
</script>

<style scoped>
img {
  width: 40px;
  height: 40px;
}
.item {
  margin: 10px;
  display: inline-block;
  width: 80%;
  cursor: pointer;
  background: rgb(253, 253, 253);
}

.size {
  width: 50%;
  margin: 10px;
}
</style>
