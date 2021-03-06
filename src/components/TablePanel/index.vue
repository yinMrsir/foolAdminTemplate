<template>
  <div :class="['padding', style]">
    <a-table
        :columns="columns"
        :row-key="record => record.id"
        :data-source="dataSource"
        :pagination="pagination"
        :loading="loading"
        @change="handleTableChange"
    >
      <template #[item]="data" v-for="item in Object.keys($slots)">
        <slot :name="item" v-bind="data"></slot>
      </template>
    </a-table>
  </div>
</template>

<script>
import { useRequest } from 'vue-request';
import { defineComponent, reactive, ref } from 'vue';
import fetch from '@/utils/fetch';

export default defineComponent({
  props: {
    request: String,
    columns: {
      type: Array,
      default: () => []
    },
    style: String
  },
  setup(props) {
    /**
     * 定义变量
     */
    const columns = props.columns;
    const dataSource = ref([])
    const pagination = reactive({
      total: 0,
      current: 1
    });
    /**
     * table 数据请求
     * @param params
     * @return {Promise<unknown>}
     */
    const queryData = params => fetch.get(props.request, params);
    let { run, loading } = useRequest(queryData, {
      defaultParams: [
        {
          per_page: 10,
        },
      ],
      onError: () => {
        loading = false
      },
      onSuccess: (res) => {
        /**
         * 此处可通过不同项目接口请求返回的列表数据进行修改
         */
        dataSource.value = res.data
        pagination.total = res.meta.total
      }
    });
    /**
     * 点击切换分页
     * @param pag
     * @param filters
     * @param sorter
     */
    const handleTableChange = (pag, filters, sorter) => {
      Object.assign(pagination, pag);
      run({
        per_page: pagination.pageSize,
        page: pagination.current,
        sortField: sorter.field,
        sortOrder: sorter.order,
        ...filters,
      });
    };

    /**
     * 请求table数据
     * @param params
     */
    const featTable = params => {
      if (params.hasOwnProperty('page')) {
        pagination.current = params.page
      }
      run(params)
    }

    return {
      dataSource,
      pagination,
      loading,
      columns,
      handleTableChange,
      featTable
    };
  },
});
</script>
