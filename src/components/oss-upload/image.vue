<template>
  <div>
    <el-upload
      class="pic-uploader-component"
      :action="'/api/oss/ossUploadImage'"
      :headers="headers"
      :show-file-list="false"
      :name="'upfile'"
      :on-success="handleUploadSuccess"
      :before-upload="beforeAvatarUpload"
      :on-error="onerror"
    >
      <img v-if="value" :src="resourcesUrl + value" class="pic">
      <i v-else class="el-icon-plus pic-uploader-icon" />
    </el-upload>
  </div>
</template>

<script>
  import {
    Notification
  } from 'element-ui'
import { getToken } from '@/utils/auth'
export default {
  props: {
    value: {
      default: '',
      type: String
    }
  },
  data() {
    return {
      resourcesUrl: '',
      headers: {
        'Authorization': getToken()
      }
    }
  },
  computed: {

  },
  methods: {
    // 图片上传
    handleUploadSuccess(response, file, ) {

      this.$emit('input', file.response)
    },
    // 限制图片上传大小
    beforeAvatarUpload(file) {
      const isLt2M = file.size / 1024 / 1024 < 100
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 100MB!')
      }
      return isLt2M
    },
    onerror(response, file, fileList){
      var resut=JSON.parse(response.message);
      Notification.error({
        title: resut.message,
        duration: 5000
      })

    },
  }
}
</script>
<style lang="scss">
  .pic-uploader-component .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
    .pic-uploader-icon {
      font-size: 28px;
      color: #8c939d;
      width: 150px;
      height: 150px;
      line-height: 150px;
      text-align: center;
    }
    .pic {
      width: 178px;
      height: 178px;
      display: block;
    }
  }
  .pic-uploader-component .el-upload:hover {
    border-color: #409EFF;
  }

</style>
