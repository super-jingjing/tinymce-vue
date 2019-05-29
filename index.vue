<template>
    <div :class="{fullscreen:fullscreen}"
         class="tinymce-container editor-container">
        <textarea :id="tinymceId"
                  :disabled="disable"
                  class="tinymce-textarea"/>
        <div class="editor-custom-btn-container">
            <el-button style="background:#1890ff"
                       icon="el-icon-upload"
                       size="mini"
                       type="primary"
                       @click="dialogVisible=true">上传图片</el-button>
            <el-dialog :visible.sync="dialogVisible"
                       multiple="true"
                       append-to-body>
                <editorImage
                    ref="editorImage"
                    class="editor-upload-btn"
                    @successCBK="imageSuccessCBK"/>
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary"
                           @click="handleSubmit">确 定</el-button>
            </el-dialog>
        </div>
    </div>
</template>
<script>
import editorImage from "../editor-image";
import plugins from "./plugins";
import toolbar from "./toolbar";
export default {
    name: "Tinymce",
    components: { editorImage },
    props: {
        id: {
            type: String,
            default: function()
            {
                return "vue-tinymce-" + +new Date() + ((Math.random() * 1000).toFixed(0) + "");
            }
        },
        value: {
            type: String,
            default: ""
        },
        toolbar: {
            type: Array,
            required: false,
            default()
            {
                return [];
            }
        },
        menubar: {
            type: String,
            default: "file edit insert view format table"
        },
        height: {
            type: Number,
            required: false,
            default: 360
        },
        disable: {
            type: Boolean,
            default: false
        }
    },
    data()
    {
        return {
            hasChange: false,
            hasInit: false,
            tinymceId: this.id,
            fullscreen: false,
            languageTypeList: {
                "en": "en",
                "zh": "zh_CN"
            },
            dialogVisible: false
        };
    },
    // computed计算属性  当属性发生变化时触发方法，不需要手动触发
    computed: {
        language()
        {
            return this.languageTypeList[this.$store.getters.language];
        }
    },
    // 监测Vue实例上的数据变动
    watch: {
        value(val)
        {
            if (!this.hasChange && this.hasInit)
            {
            // 在dom更新后再执行
                this.$nextTick(() =>
                {
                    window.tinymce.get(this.tinymceId).setContent(val || "");
                });
            }
        },
        language()
        {
            this.destroyTinymce();
            this.$nextTick(() => this.initTinymce());
        }
    },
    mounted()
    {
        this.initTinymce();
    },
    // 当页面存在缓存的时候执行该函数。
    activated()
    {
        this.initTinymce();
    },
    // 在页面结束时触发该方法，可清除缓存。
    deactivated()
    {
        this.destroyTinymce();
    },
    destroyed()
    {
        this.destroyTinymce();
    },
    methods: {
        initTinymce()
        {
            const _this = this;
            window.tinymce.init({
                language: this.language,
                selector: `#${this.tinymceId}`,
                height: this.height,
                body_class: "panel-body",
                object_resizing: false,
                toolbar: this.toolbar.length > 0 ? this.toolbar : toolbar,
                menubar: this.menubar,
                plugins: plugins,
                end_container_on_empty_block: true,
                powerpaste_word_import: "clean",
                code_dialog_height: 450,
                code_dialog_width: 1000,
                advlist_bullet_styles: "square",
                advlist_number_styles: "default",
                imagetools_cors_hosts: ["www.tinymce.com", "codepen.io"],
                default_link_target: "_blank",
                link_title: false,
                nonbreaking_force_tab: true,
                init_instance_callback: editor =>
                {
                    if (_this.value)
                    {
                        editor.setContent(_this.value);
                    }
                    _this.hasInit = true;
                    editor.on("NodeChange", () =>
                    {
                        this.hasChange = true;
                        this.$emit("input", editor.getContent());
                    });
                    editor.on("Change", () =>
                    {
                        this.hasChange = true;
                        this.$emit("input", editor.getContent());
                    });
                    editor.on("Keyup", () =>
                    {
                        this.hasChange = true;
                        this.$emit("input", editor.getContent());
                    });
                    editor.on("SetContent", () =>
                    {
                        // this.hasChange = true;
                        this.$emit("input", editor.getContent());
                    });
                },
                setup(editor)
                {
                    editor.on("FullscreenStateChanged", (e) =>
                    {
                        _this.fullscreen = e.state;
                    });
                }
            });
        },
        destroyTinymce()
        {
            const tinymce = window.tinymce.get(this.tinymce);
            if (this.fullscreen)
            {
                tinymce.execCommand("mceFullScreen");
            }
            if (tinymce)
            {
                tinymce.destroyed();
            }
        },
        setContent(value)
        {
            window.tinymce.get(this.tinymceId).setContent(value);
        },
        getContent()
        {
            window.tinymce.get(this.tinymceId).getContent();
        },
        imageSuccessCBK(arr)
        {
            const _this = this;
            arr.forEach(v =>
            {
                window.tinymce.get(_this.tinymceId).insertContent(`<img class="wscnph" src="${v.url}" >`);
            });
        },
        handleSubmit()
        {
            if (this.$refs.editorImage.handleSubmit())
            {
                this.dialogVisible = false;
            }
        }
    }
};
</script>
<style scoped>
.tinymce-container {
  position: relative;
}
.tinymce-container>>>.mce-fullscreen {
  z-index: 10000;
}
.tinymce-textarea {
  visibility: hidden;
  z-index: -1;
}
.editor-custom-btn-container {
  position: absolute;
  right: 4px;
  top: 4px;
  /*z-index: 2005;*/
}
.fullscreen .editor-custom-btn-container {
  z-index: 10000;
  position: fixed;
}
.editor-upload-btn {
  display: inline-block;
}
</style>
