<template>
  <div>
    <h1>{{id ? "修改" : "新建"}}文章</h1>
    <el-form label-width="120px" @submit.native.prevent="save">
      <el-form-item label="所属分类">
        <el-select v-model="Article.categories" multiple>
          <el-option
            v-for="category in categories"
            :key="category._id"
            :label="category.name"
            :value="category._id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="文章名称">
        <el-input v-model="Article.title"></el-input>
      </el-form-item>
      <el-form-item label="文章正文">
        <vue-editor v-model="Article.body" useCustomImageHandler @image-added="addImage"></vue-editor>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" native-type="submit">保存</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import { VueEditor } from "vue2-editor";
export default {
  props: {
    id: {}
  },
  components: {
    VueEditor
  },
  data() {
    return {
      Article: {},
      categories: []
    };
  },
  methods: {
    async save() {
      if (this.id) {
        await this.$http.put(`rest/articles/${this.id}`, this.Article);
      } else {
        await this.$http.post("rest/articles", this.Article);
      }
      this.$router.push("/articles/list");
      this.$message({
        type: String,
        message: "保存成功"
      });
    },
    async fetch() {
      const res = await this.$http.get(`rest/articles/${this.id}`);
      this.Article = res.data;
    },
    async fetchArticles() {
      const res = await this.$http.get(`rest/categories`);
      this.categories = res.data;
    },
    async addImage(file, Editor, cursorLocation, resetUploader) {
      const formData = new FormData();
      formData.append("file", file);

      const res = await this.$http.post("upload", formData);
      Editor.insertEmbed(cursorLocation, "image", res.data.url);

      resetUploader();
    }
  },
  created() {
    this.fetchArticles();
    this.id && this.fetch();
  }
};
</script>