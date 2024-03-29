<template>
  <div class="page-content">
    <JKTable
      :dataList="dataList"
      :dataCount="dataCount"
      v-bind="contentTableConfig"
      v-model:page="pageInfo"
    >
      <!-- header中的插槽 -->
      <template #headerHandler>
        <el-button
          v-if="isCreate"
          type="primary"
          size="medium"
          @click="handleNewClick"
          >新建用户</el-button
        >
        <el-button icon="el-icon-refresh"></el-button>
      </template>
      <!-- 列表中的插槽 -->
      <template #status="scope">
        <el-button
          size="mini"
          :type="scope.row.enable ? 'success' : 'danger'"
          >{{ scope.row.enable ? '启用' : '禁用' }}</el-button
        >
      </template>
      <template #createAt="scope">
        <span>{{ $filters.formatTime(scope.row.createAt) }}</span>
      </template>
      <template #updateAt="scope">
        <span>{{ $filters.formatTime(scope.row.updateAt) }}</span>
      </template>
      <template #handler="scope">
        <div class="handle-btns">
          <el-button
            v-if="isUpdate"
            icon="el-icon-edit"
            size="mini"
            type="primary"
            @click="handleEditClick(scope.row)"
            >编辑</el-button
          >
          <el-button
            v-if="isDelete"
            icon="el-icon-delete"
            size="mini"
            type="danger"
            @click="handleDeleteClick(scope.row)"
            >删除</el-button
          >
        </div>
      </template>
      <!--<template #image="scope">
        <slot name="image" :row="scope.row"></slot>
      </template> -->
      <!-- 使用动态的绑定,跨组件传插槽 -->
      <template
        v-for="item in otherPropSlots"
        :key="item.prop"
        #[item.slotName!]="scope"
      >
        <template v-if="item.slotName">
          <slot :name="item.slotName" :row="scope.row"></slot>
        </template>
      </template>
    </JKTable>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, PropType, ref, watch } from 'vue'
import { useStore } from '@/store'
import JKTable from '@/base-ui/table'
import { ITableConfig } from '../type'
import { usePermission } from '../../../hooks/use-permission'

export default defineComponent({
  props: {
    contentTableConfig: {
      type: Object as PropType<ITableConfig>,
      required: true
    },
    pageName: {
      type: String,
      required: true
    }
  },
  components: {
    JKTable
  },
  emit: ['newBtnClcik', 'editBtnClcik'],
  setup(props, { emit }) {
    //获取操作的权限
    const isCreate = usePermission(props.pageName, 'create')
    const isUpdate = usePermission(props.pageName, 'update')
    const isDelete = usePermission(props.pageName, 'delete')
    const isQuery = usePermission(props.pageName, 'query')

    //监听页脚pageInfo数据改变
    const pageInfo = ref({ currentPage: 0, pageSize: 10 })
    watch(pageInfo, () => getPageData())

    //发送网络请求
    const store = useStore()
    const getPageData = (queryInfo: any = {}) => {
      if (!isQuery) return
      store.dispatch('system/getPageListAction', {
        pageUrl: '/users/list',
        pageName: props.pageName,
        queryInfo: {
          offset: pageInfo.value.currentPage * pageInfo.value.pageSize,
          size: pageInfo.value.pageSize,
          ...queryInfo
        }
      })
      //console.log({ ...queryInfo })
    }
    getPageData()

    //从vuex中获取数据
    const dataList = computed(() =>
      store.getters['system/pageListData'](props.pageName)
    )
    const dataCount = computed(() =>
      store.getters['system/pageListCount'](props.pageName)
    )

    //获取其它动态插槽名称
    const otherPropSlots = props.contentTableConfig.propList.filter((item) => {
      if (item.slotName === 'status') return false
      if (item.slotName === 'createAt') return false
      if (item.slotName === 'updateAt') return false
      if (item.slotName === 'handler') return false
      return true
    })

    //删除/编辑/新建的操作
    const handleDeleteClick = (item: any) => {
      //console.log(item)
      store.dispatch('system/deletePageDataAction', {
        pageName: props.pageName,
        id: item.id
      })
    }

    const handleNewClick = () => {
      emit('newBtnClcik')
    }

    const handleEditClick = (item: any) => {
      emit('editBtnClcik', item)
      //console.log(item)
    }
    return {
      dataList,
      getPageData,
      dataCount,
      pageInfo,
      otherPropSlots,
      isUpdate,
      isDelete,
      isCreate,
      handleDeleteClick,
      handleNewClick,
      handleEditClick
    }
  }
})
</script>

<style scoped lang="less">
.handle-btns {
  display: flex;
}
</style>
