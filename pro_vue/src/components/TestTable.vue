<template>
  <div v-if="load">
    <el-input
        v-model="searchQuery"
        placeholder="Search..."
        style="margin-bottom: 10px;">
    </el-input>
    <el-table :data="pagedData" style="width: 100%">
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
      <el-table-column label="Actions">
        <template v-slot="props">
          <el-button
              type="danger"
              size="mini"
              @click="deleteBook(props.row)">Delete</el-button>
        </template>
      </el-table-column>
      <el-table-column type="expand">
        <template v-slot="props">
          <el-form :model="props.row" label-width="120px">
            <el-form-item label="Title">
              <el-input v-model="props.row.editingTitle"></el-input>
            </el-form-item>
            <el-form-item label="Price">
              <el-input v-model="props.row.editingPrice"></el-input>
            </el-form-item>
            <el-form-item label="Author">
              <el-input v-model="props.row.editingAuthor"></el-input>
            </el-form-item>
            <el-form-item label="Link">
              <el-input v-model="props.row.editingLink"></el-input>
            </el-form-item>
            <el-form-item label="Description">
              <el-input type="textarea" v-model="props.row.editingDescription"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="saveDescription(props.row)">Save</el-button>
            </el-form-item>
          </el-form>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[20, 50, 100]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="filteredData.length">
    </el-pagination>
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
      currentPage: 1,
      pageSize: 20,
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
    pagedData() {
      const start = (this.currentPage - 1) * this.pageSize;
      const end = start + this.pageSize;
      return this.filteredData.slice(start, end);
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
    handleSizeChange(newSize) {
      this.pageSize = newSize;
      this.currentPage = 1; // Сбросить на первую страницу при изменении размера страницы
    },
    handleCurrentChange(newPage) {
      this.currentPage = newPage;
    },
    deleteBook(row) {
      axios.delete(`http://localhost:3027/books/${row.id}`)
          .then(() => {
            this.testData = this.testData.filter(item => item.id !== row.id);
            this.$message({
              type: 'success',
              message: 'Deleted!'
            });
          })
          .catch(error => {
            console.error(error);
            this.$message.error('Error with deleting');
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

