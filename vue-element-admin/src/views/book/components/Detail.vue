<template>
  <el-form ref="postForm" :model="postForm" :rules="rules">
    <sticky :className="'sub-navbar'">
      <el-button v-if="!isEdit" @click="showGuide">显示帮助</el-button>
      <el-button
        v-loading="loading"
        @click="submitForm"
        style="margin-left: 10px"
        type="success"
        >{{ isEdit ? "编辑电子书" : "新增电子书" }}</el-button
      >
    </sticky>
    <div class="detail-container">
      <el-row>
        <Warning></Warning>
        <el-col :span="24">
          <EbookUpload
            :fileList="fileList"
            :disabled="isEdit"
            @onSuccess="onUploadSuccess"
            @onRemove="onUploadRemove"
          ></EbookUpload>
        </el-col>
        <el-col :span="24">
          <el-form-item prop="title">
            <MDInput
              v-model="postForm.title"
              :maxlength="100"
              name="name"
              required
              >书名</MDInput
            >
          </el-form-item>
          <el-row>
            <el-col :span="12">
              <el-form-item
                prop="author"
                label="作者："
                :label-width="labelWidth"
              >
                <el-input
                  v-model="postForm.author"
                  placeholder="作者"
                ></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item
                prop="publisher"
                label="出版社："
                :label-width="labelWidth"
              >
                <el-input
                  v-model="postForm.publisher"
                  placeholder="出版社"
                ></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item
                prop="language"
                label="语言："
                :label-width="labelWidth"
              >
                <el-input
                  v-model="postForm.language"
                  placeholder="语言"
                ></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item
                prop="rootFile"
                label="根文件："
                :label-width="labelWidth"
              >
                <el-input
                  v-model="postForm.rootFile"
                  placeholder="根文件"
                ></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item
                prop="filePath"
                label="文件路径："
                :label-width="labelWidth"
              >
                <el-input
                  v-model="postForm.filePath"
                  placeholder="文件路径"
                  disabled
                ></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item
                prop="unzipPath"
                label="解压路径："
                :label-width="labelWidth"
              >
                <el-input
                  v-model="postForm.unzipPath"
                  placeholder="解压路径"
                  disabled
                ></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item
                prop="coverPath"
                label="封面路径："
                :label-width="labelWidth"
              >
                <el-input
                  v-model="postForm.filePath"
                  placeholder="封面路径"
                  disabled
                ></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item
                prop="originalName"
                label="文件名称："
                :label-width="labelWidth"
              >
                <el-input
                  v-model="postForm.originalName"
                  placeholder="文件名称"
                  disabled
                ></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24">
              <el-form-item
                prop="cover"
                label="封面："
                :label-width="labelWidth"
              >
                <a v-if="postForm.cover" :href="postForm.cover" target="_blank">
                  <img :src="postForm.cover" class="preview-img" />
                </a>
                <span v-else>无</span>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24">
              <el-form-item label="目录：" :label-width="labelWidth">
                <div
                  v-if="contentsTree && contentsTree.length > 0"
                  class="contents-wrapper"
                >
                  <el-tree
                    :data="contentsTree"
                    @node-click="onContentClick"
                  ></el-tree>
                </div>
                <span v-else>无</span>
              </el-form-item>
            </el-col>
          </el-row>
        </el-col>
      </el-row>
    </div>
  </el-form>
</template>

<script>
import Sticky from "../../../components/Sticky/index.vue";
import MDInput from "../../../components/MDinput/index.vue";
import Warning from "./Warning.vue";
import EbookUpload from "../../../components/EbookUpload/index.vue";
import { createBook, getBook, updateBook } from "../../../api/book";

const defaultForm = {
  title: "",
  author: "",
  publisher: "",
  language: "",
  rootFile: "",
  cover: "",
  url: "",
  originalName: "",
  fileName: "",
  coverPath: "",
  filePath: "",
  unzipPath: "",
};

const fields = {
  title: "书名",
  author: "作者",
  publisher: "出版社",
  language: "语言",
};

export default {
  components: { Sticky, Warning, EbookUpload, MDInput },
  props: {
    isEdit: Boolean,
  },
  data() {
    const validateRequire = (rule, value, callback) => {
      if (value?.length === 0) {
        callback(new Error(fields[rule.field] + "必须填写"));
      } else {
        callback();
      }
    };
    return {
      loading: false,
      postForm: {
        status: "",
      },
      fileList: [],
      labelWidth: "120px",
      contentsTree: [],
      rules: {
        title: [{ validator: validateRequire }],
        author: [{ validator: validateRequire }],
        language: [{ validator: validateRequire }],
        publisher: [{ validator: validateRequire }],
      },
    };
  },
  created() {
    if (this.isEdit) {
      const fileName = this.$route.params.fileName;
      this.getBookData(fileName);
    }
  },
  methods: {
    getBookData(fileName) {
      getBook(fileName).then((response) => {
        this.setData(response.data);
      });
    },
    onContentClick(data) {
      if (data.text) {
        window.open(data.text);
      }
    },
    setData(data) {
      const {
        title,
        author,
        publisher,
        language,
        rootFile,
        cover,
        url,
        originalName,
        contents,
        contentsTree,
        fileName,
        coverPath,
        filePath,
        unzipPath,
      } = data;
      this.postForm = {
        ...this.postForm,
        title,
        author,
        publisher,
        language,
        rootFile,
        cover,
        url,
        originalName,
        contents,
        contentsTree,
        fileName,
        coverPath,
        filePath,
        unzipPath,
      };
      this.contentsTree = contentsTree;
      this.fileList = [{ name: originalName || fileName, url }];
    },
    onUploadSuccess(data) {
      this.setData(data);
    },
    setDefault() {
      // this.postForm = Object.assign({}, de faultForm);
      this.contentsTree = [];
      this.fileList = [];
      this.$refs.postForm.resetFields();
    },
    onUploadRemove() {
      this.setDefault();
    },
    submitForm() {
      const onSuccess = (response) => {
        const { msg } = response;
        this.$notify({
          title: "成功",
          message: msg,
          type: "success",
          duration: 2000,
        });
        this.loading = false;
      };
      this.$refs.postForm.validate((valid) => {
        if (valid) {
          this.loading = true;
          const book = Object.assign({}, this.postForm);
          delete book.contentsTree;
          if (!this.isEdit) {
            createBook(book)
              .then((response) => {
                onSuccess(response);
                this.setDefault();
              })
              .catch(() => {
                this.loading = false;
              });
          } else {
            updateBook(book)
              .then((response) => {
                onSuccess(response);
              })
              .catch(() => {
                this.loading = false;
              });
          }
        } else {
          return false;
        }
      });
    },
    showGuide() {},
  },
};
</script>

<style lang="scss" scoped>
.detail-container {
  padding: 40px 50px 20px;
  .preview-img {
    width: 200px;
    height: 270px;
  }
}
</style>
