<template>
  <div class="app-container">
    <filter-bar
      :refresh="true"
      @handleRefresh="getList"
    >
      <template #moreButton>
        <el-form-item>
          <el-button class="filter-item" type="warning" icon="el-icon-upload" @click="handleAdd">上传</el-button>
        </el-form-item>
      </template>
    </filter-bar>

    <el-alert :closable="false" style="margin-bottom: 10px;">
      <i class="el-icon-s-opportunity" />
      {{ apiInfo.info }} | {{ apiInfo.hash }} | {{ apiInfo.api_class }}
    </el-alert>

    <data-table
      :data-source="list"
      :data-config="dataConfig"
      :btn-width="160"
      :type="true"
      :edit="true"
      :del="true"
      :list-loading="listLoading"
      @handleEdit="handleEdit"
      @handleDel="handleDel"
    >
      <template #data_type="{scope}">
        <el-tag>{{ dataType[scope.row.data_type] }}</el-tag>
      </template>
      <template #is_must="{scope}">
        <el-tag :type="scope.row.is_must === 1 ? 'danger' : ''">{{ scope.row.is_must === 1 ? '必填' : '可选' }}</el-tag>
      </template>
    </data-table>

    <data-form
      :data-form="uploadForm"
      :data-form-rules="uploadFormRules"
      :form-visible.sync="uploadFormVisible"
      :form-config="uploadConfig"
      :submit-loading="uploadSubmitLoading"
      title="上传返回字段"
      label-width="80px"
      @formClose="uploadFormVisible = false"
      @formSubmit="uploadFormSubmit"
    />

    <data-form
      :data-form="dialogForm"
      :data-form-rules="dialogFormRules"
      :form-visible.sync="dialogFormVisible"
      :form-config="dialogConfig"
      :submit-loading="dialogSubmitLoading"
      :title="commonTitle('edit')"
      :menu-path="menuOptions"
      label-width="80px"
      @formClose="formClose"
      @formSubmit="dialogFormSubmit"
    >
      <template #defaults>
        <el-input
          v-model="dialogForm.defaults"
          :disabled="dialogForm.is_must === 1"
          maxlength="255"
          style="width: 50%;"
          clearable
        />
        <el-tag type="danger">仅在字段非必填的情况下生效</el-tag>
      </template>
    </data-form>

    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="pagination.page"
      :limit.sync="pagination.size"
      @pagination="getList"
    />

    <el-tooltip placement="top" content="Top">
      <back-to-top />
    </el-tooltip>
  </div>
</template>

<script>
import BackToTop from '@/components/BackToTop'
import FilterBar from '@/components/FilterBar'
import DataTable from '@/components/DataTable'
import DataForm from '@/components/DataForm'
import Pagination from '@/components/Pagination'
import mixin from '@/utils/mixin'
import { commonTitle } from '@/utils/i18n'
import { getResponse, upJson, edit, del } from '@/api/fields'
import { responseDataConfig as dataConfig } from './config'

export default {
  name: 'InterfaceResponse',
  components: {
    BackToTop,
    FilterBar,
    DataTable,
    DataForm,
    Pagination
  },
  mixins: [mixin],
  data() {
    return {
      dataConfig: dataConfig.fields,
      dialogForm: {},
      dialogFormRules: dataConfig.dialogFormRules,
      dialogConfig: dataConfig.dialogFields,
      uploadForm: {},
      uploadFormRules: dataConfig.uploadFormRules,
      uploadConfig: dataConfig.uploadFields,
      uploadFormVisible: false,
      uploadSubmitLoading: false,
      hash: '',
      dataType: {},
      apiInfo: {},
      tempRoute: {}
    }
  },
  computed: {
    lang() {
      return this.$store.getters.language
    },
    menuOptions() {
      const result = []
      Object.keys(this.dataType).forEach(v => {
        result.push({ value: v, label: this.dataType[v] })
      })
      return result
    }
  },
  created() {
    this.hash = this.$route.params && this.$route.params.hash
    this.tempRoute = Object.assign({}, this.$route)
    this.setTagsViewTitle()
    this.setPageTitle()
  },
  mounted() {
    this.getList()
  },
  methods: {
    setTagsViewTitle() {
      const title = this.lang === 'zh' ? '返回参数' : 'Response'
      const route = Object.assign({}, this.tempRoute, { title: `${title}-${this.hash}` })
      this.$store.dispatch('tagsView/updateVisitedView', route)
    },
    setPageTitle() {
      const title = this.lang === 'zh' ? '返回参数' : 'Response'
      document.title = `${title} - ${this.hash}`
    },
    getList() {
      this.listLoading = true
      const params = {
        ...this.pagination,
        hash: this.hash
      }
      getResponse(params).then(response => {
        this.list = response.data.list
        this.total = response.data.count
        this.dataType = response.data.dataType
        this.apiInfo = response.data.apiInfo
        this.listLoading = false
      }).catch(() => {
        this.listLoading = false
      })
    },
    handleAdd() {
      this.uploadForm = { ...dataConfig.uploadForm }
      this.uploadFormVisible = true
    },
    handleEdit(row) {
      this.dialogForm.id = row.id
      this.dialogForm.field_name = row.field_name
      this.dialogForm.data_type = row.data_type.toString()
      this.dialogForm.defaults = row.default
      this.dialogForm.range = row.range
      this.dialogForm.is_must = row.is_must
      this.dialogForm.info = row.info
      this.dialogFormVisible = true
    },
    dialogFormSubmit(params) {
      this.dialogSubmitLoading = true
      params.hash = this.hash
      edit(params).then(response => {
        this.dialogFormVisible = false
        this.dialogSubmitLoading = false
        this.$message.success(response.msg)
        this.getList()
      }).catch(() => {
        this.dialogSubmitLoading = false
      })
    },
    uploadFormSubmit(params) {
      this.uploadSubmitLoading = true
      params.hash = this.hash
      upJson(params).then(response => {
        this.uploadFormVisible = false
        this.uploadSubmitLoading = false
        this.$message.success(response.msg)
        this.getList()
      }).catch(() => {
        this.uploadSubmitLoading = false
      })
    },
    handleDel(index, row) {
      del(row.id).then(response => {
        this.list.splice(index, 1)
        this.$message.success(response.msg)
      })
    },
    commonTitle
  }
}
</script>
