# tinymce-vue
vue 富文本编辑器
## 准备
1.下载tinymce js文件，放入到static目录下
2.在index.html页面引入<script src="<%=BASE_URL%>/plugins/tinymce/4.7.5/tinymce.min.js"></script>

##使用
<Tinymce v-model="form.data.content"/>
