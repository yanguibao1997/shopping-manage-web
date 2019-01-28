<template>
  <div>
    <quilleditor v-model="content"
                 ref="myTextEditor"
                 :options="editorOption"
                 @change="onChange">
      <div id="toolbar" slot="toolbar">
        <select class="ql-size">
          <option value="small"></option>
          <!-- Note a missing, thus falsy value, is used to reset to default -->
          <option selected></option>
          <option value="large"></option>
          <option value="huge"></option>
        </select>
        <!-- Add subscript and superscript buttons -->
        <span class="ql-formats"><button class="ql-script" value="sub"></button></span>
        <span class="ql-formats"><button class="ql-script" value="super"></button></span>
        <span class="ql-formats"><button type="button" class="ql-bold"></button></span>
        <span class="ql-formats"><button type="button" class="ql-italic"></button></span>
        <span class="ql-formats"><button type="button" class="ql-blockquote"></button></span>
        <span class="ql-formats"><button type="button" class="ql-list" value="ordered"></button></span>
        <span class="ql-formats"><button type="button" class="ql-list" value="bullet"></button></span>
        <span class="ql-formats"><button type="button" class="ql-link"></button></span>
        <span class="ql-formats">
          <button type="button" @click="imgClick" style="outline:none">
            <svg viewBox="0 0 18 18">
              <rect class="ql-stroke" height="10" width="12" x="3" y="4"></rect>
              <circle class="ql-fill" cx="6" cy="7" r="1"></circle>
              <polyline class="ql-even ql-fill" points="5 12 5 11 7 9 8 10 11 7 13 9 13 12 5 12"></polyline>
            </svg>
          </button>
        </span>
        <span class="ql-formats"><button type="button" class="ql-video"></button></span>
      </div>
    </quilleditor>
  </div>
</template>
<script>
  import 'quill/dist/quill.core.css'
  import 'quill/dist/quill.snow.css'
  import 'quill/dist/quill.bubble.css'

  import {quillEditor} from 'vue-quill-editor'

  export default {
    name: "v-editor",
    props: {
      value: {
        type: String
      },
      /*上传图片的地址*/
      uploadUrl: {
        type: String,
        default: '/'
      },
      /*上传图片的file控件name*/
      fileName: {
        type: String,
        default: 'file'
      },
      maxUploadSize:{
        type:Number,
        default: 1024*1024*5
      }
    },
    data() {
      return {
        content: '',
        editorOption: {
          modules: {
            toolbar: '#toolbar'
          }
        },
      }
    },
    methods: {
      onChange() {
        this.$emit('range', this.content)
      },
      /*选择上传图片切换*/
      onFileChange(e) {
        //target 事件属性可返回事件的目标节点（触发该事件的节点），如生成事件的元素、文档或窗口
        var fileInput = e.target;

        if (fileInput.files.length === 0) {
          return
        }
        this.editor.focus();
        // console.log("图片大小"+fileInput.files[0].size);

        // console.log("设置大小"+this.maxUploadSize);

        if (fileInput.files[0].size > this.maxUploadSize) {
          /*this.$alert('图片不能大于5M', {
            confirmButtonText: '确定',
            type: 'warning',
          });*/

          this.$message.error("图片不能大于5M");
        }

        var data = new FormData;
        data.append(this.fileName, fileInput.files[0]);

        this.$http.post(this.uploadUrl, data).then( res => {
            if (res.data) {
              // console.log(res.data);
              // insertEmbed    当点击quill中上传文件的button后调用
              // 这里url是先上传文件后从服务器返回的文件资源地址
              // this.quill.getSelection()
              var range = this.$refs.myTextEditor.quill.getSelection();
              this.editor.insertEmbed(range.index, 'image', res.data)
            }
        });
      },
      /*点击上传图片按钮*/
      imgClick() {
        if (!this.uploadUrl) {
          console.log('no editor uploadUrl');
          return;
        }
        /*内存创建input file*/

        var input = document.createElement('input');
        input.type = 'file';

        //默认的名字为  file
        input.name = this.fileName;

        //在文件上传中使用 accept 属性
        input.accept = 'image/jpeg,image/png,image/jpg,image/gif';
        // input.multiple=true;
        input.onchange = this.onFileChange;
        input.click()
      }
    },
    computed: {
      editor() {
        return this.$refs.myTextEditor.quill
      }
    },
    components: {
      'quilleditor': quillEditor
    },
    mounted() {
      this.content = this.value
    },
    watch: {
      'value'(newVal, oldVal) {
        if (this.editor) {
          if (newVal !== this.content) {
            this.content = newVal
          }
        }
      },
    }
  }

</script>
