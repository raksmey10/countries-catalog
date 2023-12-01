<template>
  <div>
    <br />
    <el-form :inline="true">
      <el-form-item>
        <el-input v-model="countryName" clearable @clear="handleClear" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" :icon="Search" @click="onSubmit" :loading="loadingBtn">Search</el-button>
      </el-form-item>
    </el-form>
    <br />
    <el-table
        highlight-current-row
        v-loading="loadingList"
        :data="currentPageList"
        :default-sort="{ prop: 'name.official', order: 'ascending' }"
        style="width: 100%"
    >

      <el-table-column prop="name.official" label="Name" width="350" sortable>
        <template #default="scope">
          <span>
            <img :src="scope.row.flags.png" alt="country flags" style="width:25px">
          </span>
          &nbsp;
          <span>
            <el-link type="primary">
              {{ scope.row.name.official }}
            </el-link>
          </span>
        </template>
      </el-table-column>

      <el-table-column prop="cca2" label="2 Character Country Code" />
      <el-table-column prop="cca3" label="3 Character Country Code" />
      <el-table-column prop="name.nativeName.eng.official" label="Native Country Name" />

      <el-table-column prop="altSpellings.0" label="Alternative Country Name">
        <template #default="scope">

          <el-popover effect="dark" trigger="hover" placement="left" width="auto">
            <template #default>
              <div v-for="item in scope.row.altSpellings" :key="item[0]">
                {{ item }}
              </div>
            </template>
            <template #reference>
              <el-link type="primary">
                {{ scope.row.altSpellings[0] }}
              </el-link>
            </template>
          </el-popover>
        </template>
      </el-table-column>

      <el-table-column prop="idd.root" label="Country Calling Codes" />
    </el-table>
    <br />
    <br />
    <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="PagePagination.pageNum"
        :page-sizes="[10, 25, 50, 100]"
        :page-size="PagePagination.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="PagePagination.total"
        style="display: flex;justify-content: center  "
    />
  </div>
</template>

<script setup lang="ts">
  import { ref, reactive } from 'vue';
  import { Search } from '@element-plus/icons-vue'

  const endPoint = ref('')
  const keywords = ref('')
  const countryName = ref('')
  const loadingList = ref(false)
  const loadingBtn = ref(false)
  const list = ref([])
  const currentPageList = ref([])
  const PagePagination = reactive({
    pageSize: 10, // numbers of items per page
    pageNum: 1, // current page number
    total: 0 // total number of items
  })

  // get list of items
  const getList = (keywords = '') => {
    loadingList.value = true
    loadingBtn.value = true
    if (keywords != '') {
      endPoint.value = `name/${keywords}`
    } else {
      endPoint.value = `all`
    }
    fetch(`https://restcountries.com/v3.1/${endPoint.value}`)
    .then(res => res.json())
    .then(data => {
      console.log(data)
      if (data == null) {
        loadingList.value = false
        loadingBtn.value = false
        PagePagination.total = 0
        list.value = []
      } else {
        loadingList.value = false
        loadingBtn.value = false
        PagePagination.pageNum = 1
        list.value = data
        // console.log(list.value)
        getCurrentPageList()
      }
    })
  }
  getList()

  // handle data pagination fetched from api
  const getCurrentPageList = () => {
    PagePagination.total = list.value.length
    const temporaryList = ref([])
    const beginIndex = (PagePagination.pageNum - 1) * PagePagination.pageSize
    const endIndex = PagePagination.pageNum * PagePagination.pageSize
    for (let index = beginIndex; index < endIndex; index++) {
      if (index < list.value.length) {
        const element = list.value[index]
        temporaryList.value.push(element)
      }
    }
    currentPageList.value = temporaryList.value
  }

  // Triggered when page Size (number of items displayed on one page) changes
  const handleSizeChange = (val: number) => {
    PagePagination.pageNum = 1
    PagePagination.pageSize = val
    getCurrentPageList()
  }

  // Triggered when changed
  const handleCurrentChange = (val: number) => {
    PagePagination.pageNum = val
    getCurrentPageList()
  }

  const onSubmit = () => {
    loadingBtn.value = true
    keywords.value = countryName.value
    getList(keywords.value)
  }

  const handleClear = () => {
    getList()
  }
</script>

<style lang="scss">
//@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@300&display=swap');
body{
  //font-family: 'Nunito', sans-serif;
  font-family: 'Helvetica Neue', Helvetica, 'PingFang SC', 'Hiragino Sans GB',
  'Microsoft YaHei', '微软雅黑', Arial, sans-serif;
}
</style>
