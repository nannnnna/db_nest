<template>
  <div v-if="load">
    <el-input
        v-model="searchQuery"
        placeholder="Search..."
        style="margin-bottom: 10px;">
    </el-input>
    <el-table :data="filteredData" style="width: 100%">
      <el-table-column type="expand">
        <template v-slot="props">
          <p>ID: {{ props.row.id }}</p>
          <p>Title: {{ props.row.title }}</p>
        </template>
      </el-table-column>
      <el-table-column prop="id" label="id" sortable></el-table-column>
      <el-table-column prop="title" label="Title" sortable></el-table-column>
      <el-table-column prop="price" label="Price" sortable></el-table-column>
      <el-table-column prop="author" label="Author" sortable></el-table-column>
      <el-table-column prop="link" label="Link" sortable></el-table-column>
      <el-table-column prop="description" label="Description" sortable></el-table-column>
      <el-table-column type="expand">
        <template v-slot="props">
          <el-input placeholder="Title" v-model="props.row.editingTitle" @keydown.stop
                    @keyup.stop></el-input>
          <el-input placeholder="Price" v-model="props.row.editingPrice" @keydown.stop
                    @keyup.stop></el-input>
          <el-input placeholder="Author" v-model="props.row.editingAuthor" @keydown.stop
                    @keyup.stop></el-input>
          <el-input placeholder="Link" v-model="props.row.editingLink" @keydown.stop
                    @keyup.stop></el-input>
          <el-input
              type="textarea"
              :rows="2"
              placeholder="Введите описание"
              v-model="props.row.editingDescription"
              @keydown.stop
              @keyup.stop>
          </el-input>
          <el-button
              type="primary"
              @click="saveDescription(props.row)">
            Сохранить
          </el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      testData: [],
      load: false,
      searchQuery: '',
    };
  },
  computed: {
    filteredData() {
      if (!this.searchQuery) {
        return this.testData;
      }
      // Фильтрация данных на основе строки поиска
      return this.testData.filter(item => {
        const searchString = this.searchQuery.toLowerCase();
        return Object.values(item)
            .join(' ')
            .toLowerCase()
            .includes(searchString);
      });
    },
  },
  created() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      axios.get('http://localhost:3027/books')
          .then(response => {
            this.testData = response.data.map(book => ({
              ...book,
              editingTitle: book.title,
              editingPrice: book.price,
              editingAuthor: book.author,
              editingLink: book.link,
              editingDescription: book.description,
            }));
            this.load = true;
          })
          .catch(error => {
            console.error(error);
          });
    },
    saveDescription(row) {
      const updatedBook = {
        id: row.id,
        title: row.editingTitle,
        price: row.editingPrice,
        author: row.editingAuthor,
        link: row.editingLink,
        description: row.editingDescription
      };
      axios.patch(`http://localhost:3027/books/${row.id}`, updatedBook)
          .then(() => {
            const index = this.testData.findIndex(book => book.id === row.id);
            if (index !== -1) {
              // Обновляем книгу в массиве
              this.testData[index] = {
                ...this.testData[index],
                ...updatedBook
              };
              // Важно: чтобы изменения были реактивными, можно использовать следующий трюк:
              this.testData = [...this.testData];
            }
            this.$message({
              type: 'success',
              message: 'Success!'
            });
          })
          .catch(error => {
            console.error(error);
          });
    }
  }
};
</script>

