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
          <el-input
              type="textarea"
              :rows="2"
              placeholder="Введите описание"
              v-model="props.row.editingDescription" >
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
        title: row.title,
        price: row.price,
        author: row.author,
        link: row.link,
        description: row.editingDescription
      };
      axios.patch(`http://localhost:3027/books/${row.id}`, updatedBook)
          .then(() => {
            const index = this.testData.findIndex(book => book.id === row.id);
            if (index !== -1) {
              // Обновляем книгу в массиве
              this.testData.splice(index, 1, {
                ...this.testData[index],
                ...updatedBook
              });
            }
            this.$message({
              type: 'success',
              message: 'Описание успешно сохранено!'
            });
          })
          .catch(error => {
            console.error(error);
          });
    }
  }
};
</script>

