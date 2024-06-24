<template>
  <el-form ref="postForm" :model="postForm" :rules="rules">
    <sticky :class-name="'sub-navbar'">
      <el-button v-if="!isEdit" @click="showGuide">显示帮助</el-button>
      <el-button v-loading="loading" type="success" style="margin-left: 10px;" @click="submitForm">{{ isEdit ? '编辑电子书' : '新增电子书' }}</el-button>
    </sticky>
    <div class="detail-container">
      <el-row>
        <warning />
        <el-col :span="24">
          <ebook-upload :file-list="fileList" :disabled="isEdit" @onSuccess="onUploadSuccess" @onRemove="onUploadRemove" />
        </el-col>
        <el-col :span="24">
          <el-form-item prop="title">
            <md-input v-model="postForm.title" :maxlength="100" name="name" required>书名</md-input>
          </el-form-item>
          <el-row>
            <el-col :span="12">
              <el-form-item label="作者：" prop="author" :label-width="labelWidth">
                <el-input v-model="postForm.author" placeholder="作者" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="出版社：" prop="publisher" :label-width="labelWidth">
                <el-input v-model="postForm.publisher" placeholder="出版社" />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item label="语言：" prop="language" :label-width="labelWidth">
                <el-input v-model="postForm.language" placeholder="语言" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="根文件：" prop="rootFile" :label-width="labelWidth">
                <el-input v-model="postForm.rootFile" placeholder="根文件" disabled />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item label="文件路径：" prop="filePath" :label-width="labelWidth">
                <el-input v-model="postForm.filePath" placeholder="文件路径" disabled />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="解压路径：" prop="unzipPath" :label-width="labelWidth">
                <el-input v-model="postForm.unzipPath" placeholder="解压路径" disabled />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item label="封面路径：" prop="coverPath" :label-width="labelWidth">
                <el-input v-model="postForm.coverPath" placeholder="封面路径" disabled />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="文件名称：" prop="originalName" :label-width="labelWidth">
                <el-input v-model="postForm.originalName" placeholder="文件名称" disabled />
              </el-form-item>
            </el-col>
          </el-row>
          <el-col :span="24">
            <el-form-item label="封面：" prop="cover" :label-width="labelWidth">
              <a v-if="postForm.cover" :href="postForm.cover" target="_blank">
                <img :src="postForm.cover" class="preview-image">
              </a>
              <span v-else>无</span>
            </el-form-item>
          </el-col>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="24">
          <el-form-item label="目录：" :label-width="labelWidth">
            <div v-if="contentsTree.length > 0" class="contents-wrapper">
              <el-tree :data="contentsTree" @node-click="onContentClick" />
            </div>
            <span v-else>无</span>
          </el-form-item>
        </el-col>
      </el-row>
    </div>
  </el-form>

</template>

<script>
import Sticky from '@/components/Sticky'
import EbookUpload from '@/components/EbookUpload'
import MdInput from '@/components/MDinput'
import Warning from './Warning'

import { createBook, updateBook, getBook } from '@/api/book'

const defaultForm = {
  title: '', author: '', publisher: '', language: '', rootFile: '', cover: '', url: '', originalName: '', fileName: '', coverPath: '', filePath: '', unzipPath: ''
}

const fileds = {
  title: '书名',
  author: '作者',
  publisher: '出版社',
  language: '语言'
}

export default {
  name: 'XmAdminDetail',
  components: { Sticky, EbookUpload, Warning, MdInput },
  props: {
    isEdit: Boolean
  },
  data() {
    const validateRequire = (rule, value, callback) => {
      if (value === '') {
        callback(new Error(fileds[rule.field] + '必须填写'))
      } else {
        callback()
      }
    }
    return {
      loading: false,
      postForm: Object.assign({}, defaultForm),
      fileList: [],
      labelWidth: '120px',
      contentsTree: [],
      rules: {
        title: [{ validator: validateRequire }],
        author: [{ validator: validateRequire }],
        publisher: [{ validator: validateRequire }],
        language: [{ validator: validateRequire }]
      }
    }
  },

  mounted() {

  },
  created() {
    if (this.isEdit) {
      const fileName = this.$route.params.fileName
      this.getBookData(fileName)
    }
  },

  methods: {
    getBookData(fileName) {
      getBook(fileName).then(response => {
        this.setData(response.data)
      })
    },
    showGuide() {},
    submitForm() {
      const onSuccess = (response) => {
        const { msg } = response
        this.$notify({
          title: '操作成功',
          message: msg,
          type: 'success',
          duration: 2000
        })
        this.loading = false
      }
      if (!this.loading) {
        this.loading = true
        this.$refs.postForm.validate((valid, fields) => {
          console.log(valid, fileds)
          if (valid) {
            const book = Object.assign({}, this.postForm)
            // delete book.contents
            delete book.contentsTree
            if (!this.isEdit) {
              createBook(book).then(response => {
                onSuccess(response)
                this.setDefault()
              }).catch(() => {
                this.loading = false
              })
            } else {
              updateBook(book).then(response => {
                onSuccess(response)
              }).catch(() => {
                this.loading = false
              })
            }
          } else {
            console.log(valid, fileds)
            this.$message({
              message: fileds[Object.keys(fields)[0]] + '为必传项',
              type: 'error'
            })
            this.loading = false
          }
        })
      }
    },
    setData(data) {
      const { title, author, publisher, language, rootFile, cover, url, originalName, contents, contentsTree, fileName, coverPath, filePath, unzipPath } = data
      this.postForm = {
        ...this.postForm,
        title, author, publisher, language, rootFile, cover, url, originalName, contents, contentsTree, fileName, coverPath, filePath, unzipPath
      }
      this.contentsTree = contentsTree
      this.fileList = [{ name: originalName || fileName, url }]
    },
    setDefault() {
      this.contentsTree = []
      this.fileList = []
      this.$refs.postForm.resetFields()
    },
    onUploadSuccess(data) {
      console.log('onUploadSuccess', data)
      this.setData(data)
    },
    onUploadRemove() {
      this.setDefault()
    },
    onContentClick(data) {
      if (data.text) {
        window.open(data.text)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.detail-container {
  padding: 40px 50px 20px;
  .preview-image {
    width: 200px;
    height: 270px;
  }
}
</style>
