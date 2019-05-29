<template>
    <div class="upload-container">
        <el-upload :multiple="true"
                   :file-list="fileList"
                   :show-file-list="true"
                   :on-remove="handleRemove"
                   :http-request="upload"
                   :before-upload="beforeAvatarUpload"
                   class="editor-slide-upload"
                   action=""
                   list-type="picture-card">
            <el-button size="small"
                       type="primary">点击上传</el-button>
        </el-upload>
    </div>
</template>
<script>
import ResourceRepo from "@/assets/repos/admin/resource";
export default {
    name: "EditorImage",
    props: {},
    data()
    {
        return {
            listObj: {},
            fileList: []
        };
    },
    methods: {
        // 检查是否全部上传成功
        checkAllSuccess()
        {
            // Object.keys返回所有的key值
            // every 检测，有一个不满足就返回false
            return Object.keys(this.listObj).every(item => this.listObj[item].hasSuccess);
        },
        // 插入到富文本
        handleSubmit()
        {
            // 根据key拿出所有的数据，并且返回一个新的数组
            const arr = Object.keys(this.listObj).map(v => this.listObj[v]);
            if (!this.checkAllSuccess())
            {
                this.$message("请等待所有图片上传成功");
                return false;
            }
            this.$emit("successCBK", arr);
            this.listObj = {};
            this.fileList = [];
            return true;
        },
        handleRemove(file)
        {
            const uid = file.uid;
            const objKeyArr = Object.keys(this.listObj);
            for (let i = 0, len = objKeyArr.length; i < len; i++)
            {
                if (this.listObj[objKeyArr[i]].uid === uid)
                {
                    delete this.listObj[objKeyArr[i]];
                    return;
                }
            }
        },
        // 文件上传前操作
        beforeAvatarUpload(file)
        {
            const isJPG = file.type === "image/jpeg";
            const isPNG = file.type === "image/png";
            const isLt2M = file.size / 1024 / 1024 < 2;
            const _self = this;
            if (!isJPG && !isPNG)
            {
                this.$message.error("上传头像图片只能是 JPG/PNG 格式!");
            }
            if (!isLt2M)
            {
                this.$message.error("上传头像图片大小不能超过 2MB!");
            }
            _self.listObj[file.name] = { hasSuccess: false, uid: file.uid, width: this.width, height: this.height };
            return (isJPG || isPNG) && isLt2M;
        },
        // 上传到服务器
        upload(data)
        {
            const _self = this;
            let uploadData = new FormData();
            uploadData.append("file", data.file);
            ResourceRepo.upload(uploadData).then(response =>
            {
                if (response !== null)
                {
                    this.listObj[data.file.name].url = process.env.BASE_API + "/service-admin/" + response.path;
                    _self.listObj[data.file.name].hasSuccess = true;
                }
            });
        }
    }
};
</script>
<style rel="stylesheet/scss" lang="scss" scoped>
.upload-container{
    width: 100%;
}
.editor-slide-upload {
    margin-bottom: 20px;
  /deep/ .el-upload--picture-card {
    width: 100%;
  }
}
</style>
