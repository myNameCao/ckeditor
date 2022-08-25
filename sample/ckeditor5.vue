<template>
  <div :id="id" class="edite">
    <div class="editor__toolbar"></div>
    <div :ref="id" class="ckeditor5"></div>
  </div>
</template>
<script>
  export default {
    name: 'Ckeditor5',
    props: {
      value: {
        type: String,
        default: ''
      },
      item: {
        type: Object,
        default() {
          return {}
        }
      }
    },
    data() {
      return {
        id: `ck-${parseInt(Math.random() * 10000 + Date.now()).toString()}-ck`,
        editor: null,
        ischange: false
      }
    },
    watch: {
      value(newValue) {
        if (newValue !== this.editor.getData({ trim: 'none' })) {
          this.editor.setData(newValue)
        }
      }
    },
    mounted() {
      DecoupledDocumentEditor.create(this.$refs[this.id], {
        licenseKey: ''
      })
        .then(editor => {
          this.editor = editor
          // 初始化数据
          editor.setData(this.value)
          // 内容监听
          editor.model.document.on('change:data', () => {
            this.ischange = true
          })
          // 聚焦事件
          editor.ui.focusTracker.on(
            'change:isFocused',
            (evt, name, isFocused) => {
              if (!isFocused) {
                this.onEditorBlur(editor)
                if (this.ischange) {
                  this.onChange(editor)
                }
                this.ischange = false
              }
              if (isFocused) {
                this.onEditorFocus(editor)
              }
            }
          )
          // 隐藏 toobar
          editor.ui.view.toolbar.element.style.display = 'none'
          let el = document.querySelector(`#${this.id}`)
          el.querySelector('.editor__toolbar').appendChild(
            editor.ui.view.toolbar.element
          )
          el.querySelector('.ck-toolbar').classList.add('ck-reset_all')
        })
        .catch(error => {
          console.error(error)
        })
    },
    beforeDestroy() {
      this.editor.destroy()
    },
    methods: {
      onChange() {
        let val = this.editor.getData({ trim: 'none' })
        console.log(val)
        this.$emit('input', val)
      },
      onEditorBlur(editor) {
        editor.ui.view.toolbar.element.style.display = 'none'
      },
      onEditorFocus(editor) {
        editor.ui.view.toolbar.element.style.display = 'block'
      }
    }
  }
</script>
<style lang="less" scoped>
  .edite {
    width: 100%;
    height: 100%;
    position: relative;
    .ckeditor5 {
      position: absolute;
      top: 0;
      width: 100%;
      height: 100%;
      padding-top: 2mm;
    }
  }
  /deep/ .ck-toolbar {
    position: absolute;
    top: -42px;
    z-index: 99999;
  }
  /deep/ .ck-dropdown__panel_se {
    //fix 弹框 不在toobar 下面
    transform: translate3d(0, 0, 0) !important;
  }

  /deep/ .ckeditor5 {
    p {
      margin: 0 !important;
      font-family: 'Times New Roman';
      word-break: break-all;
      white-space: pre-line;
    }
  }
  /deep/ .ck-focused {
    p {
      position: relative;
      &::after {
        content: '↩';
        white-space: pre-wrap;
        color: #aaa;
        position: absolute;
        top: 0;
      }
    }
  }
  /deep/ .ckeditor5 div {
    margin: 0 !important;
    word-break: break-all;
    white-space: pre-line;
    font-family: 'Times New Roman';
  }
</style>
