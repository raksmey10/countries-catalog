<template>
  <h1 align="center">Countries Catalog</h1>
  <el-row :gutter="20">
    <el-col :span="3"></el-col>
    <el-col :span="18">
      <br />
      <el-form :inline="true" align="center">
        <el-form-item>
          <el-input v-model="countryName" clearable @clear="handleClear" placeholder="Country Name..." />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" :icon="Search" @click="onSubmit" :loading="loadingBtn" :disabled="disabled">Search</el-button>
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
        <el-table-column
            type="index"
            label="&numero;"
        />

        <el-table-column prop="name.official" label="Country Name" min-width="200" sortable>
          <template #default="scope">
            <span>
              <img :src="scope.row.flags.png" alt="country flags" style="width:25px">
            </span>
            &nbsp;
            <el-tooltip
                effect="light"
                :content="scope.row.name.official"
                placement="right"
            >
              <span class="link-type ellipsis-wrapper" @click="info(scope.row)">
                {{ scope.row.name.official }}
              </span>
            </el-tooltip>
          </template>
        </el-table-column>

        <el-table-column prop="cca2" label="CCA2" />
        <el-table-column prop="cca3" label="CCA3" />
        <el-table-column prop="name.nativeName.eng.official" label="Native Name" min-width="150" />

        <el-table-column prop="altSpellings.0" label="Alternative">
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

        <el-table-column prop="idd.root" label="Calling Codes">
          <template #default="scope">
            <el-tag>{{ scope.row.idd.root }}</el-tag>
          </template>
        </el-table-column>
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
    </el-col>
    <el-col :span="4"></el-col>
  </el-row>

  <el-dialog
      v-model="dialogVisible"
      title="Other Information"
      width="30%"
      :show-cancel="false"
  >
    <CountriesInfo :countryInfo="countryInfo" />
  </el-dialog>
</template>

<script setup lang="ts">
  import { ref, reactive, computed } from 'vue';
  import { Search } from '@element-plus/icons-vue'
  import { ElMessage } from 'element-plus'
  import CountriesInfo from './CountriesInfo.vue'

  // define ref
  const countryInfo = ref({})
  const dialogVisible = ref(false)
  const endPoint = ref('')
  const keywords = ref('')
  const countryName = ref('')
  const loadingList = ref(false)
  const loadingBtn = ref(false)
  const list = ref([])
  const currentPageList = ref([])
  const PagePagination = reactive({
    pageSize: 25, // numbers of items per page
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
      // console.log(data)
      if (data.status == 404) {
        ElMessage({
          showClose: true,
          message: 'Data not found',
          type: 'warning',
        })
        loadingList.value = false
        loadingBtn.value = false
        PagePagination.total = 0
        list.value = []
      }
      loadingList.value = false
      loadingBtn.value = false
      PagePagination.pageNum = 1
      list.value = data
      getCurrentPageList()
    })
  }
  // initiate countries list
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

  // handle search
  const onSubmit = () => {
    loadingBtn.value = true
    keywords.value = countryName.value
    getList(keywords.value)
  }

  // disable search button when search box is empty
  const disabled = computed(() => !countryName.value);

  // clear value from search box
  const handleClear = () => {
    getList()
  }

  // get country info
  const info = (item: object) => {
    countryInfo.value = item
    dialogVisible.value = true
  }
</script>

<style lang="scss" scoped>
  .ellipsis-wrapper {
    position: absolute;
    max-width: calc(100% - 5rem);
    line-height: calc(2.5rem - 1px);
    top: 0;
    left: 3rem;
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
  }

  .link-type {
    color: #333333;
    cursor: pointer;

    &:hover {
      color: rgb(32, 160, 255);
    }
  }
</style>
