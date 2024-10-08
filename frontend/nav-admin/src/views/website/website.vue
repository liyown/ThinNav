<script setup lang="ts">
import { ref, onMounted, watch } from "vue";
import { debounce } from "lodash-es"; // 引入 debounce 函数
import { getWebs } from "@/api/website";
import { useRenderIcon } from "@/components/ReIcon/src/hooks";
import AddFill from "@iconify-icons/ri/add-circle-line";
import SearchIcon from "@iconify-icons/ri/search-line"; // 引入搜索图标
import {
  handleEdit,
  handleDelete,
  handleCreat,
  setFetchWebsMethod
} from "./webOperate";

defineOptions({
  name: "Website"
});

const webData = ref([]);
const pagination = ref({ current: 1, pageSize: 10, total: 0 });
const searchKeyword = ref("");

// 使用 debounce 优化搜索，延迟触发 API 请求
const fetchWebs = debounce(async (keyword = "") => {
  const { data, total } = await getWebs({
    page: pagination.value.current || 1,
    limit: pagination.value.pageSize || 10,
    search: keyword
  });
  webData.value = data;
  pagination.value.total = total;
}, 300);

onMounted(() => {
  fetchWebs();
  setFetchWebsMethod(fetchWebs);
});

watch(
  [
    () => pagination.value.current,
    () => pagination.value.pageSize,
    searchKeyword
  ],
  ([, , keyword]) => fetchWebs(keyword)
);

const onPageSizeChange = (size: number) => {
  pagination.value.pageSize = size || 10;
  pagination.value.current = 1;
};

const onCurrentChange = (current: number) => {
  pagination.value.current = current || 1;
};

const columns: TableColumnList = [
  {
    label: "名称",
    prop: "name"
  },
  {
    label: "排序",
    prop: "order"
  },
  {
    label: "描述",
    prop: "description",
    showOverflowTooltip: true // 启用文字溢出省略号和 tooltip 显示
  },
  {
    label: "分类",
    prop: "category_name"
  },
  {
    label: "操作",
    slot: "operation"
  }
];
</script>

<template>
  <el-card shadow="never">
    <div
      style="display: flex; align-items: center; justify-content: space-between"
    >
      <el-button :icon="useRenderIcon(AddFill)" @click="handleCreat()">
        新建网址
      </el-button>
      <el-input
        v-model="searchKeyword"
        placeholder="搜索网址"
        style="width: 300px"
        clearable
        :prefix-icon="useRenderIcon(SearchIcon)"
      />
    </div>
    <pure-table :data="webData" :columns="columns" class="mt-4">
      <!-- 添加上间距 -->
      <template #operation="{ row }">
        <el-button size="small" @click="handleEdit(row)"> 编辑 </el-button>
        <el-button size="small" type="danger" @click="handleDelete(row)">
          删除
        </el-button>
      </template>
    </pure-table>
    <el-pagination
      v-model:currentPage="pagination.current"
      class="float-right mt-4 mb-4"
      :page-size="pagination.pageSize"
      :total="pagination.total"
      :page-sizes="[10, 20, 50]"
      :background="true"
      layout="total, sizes, prev, pager, next, jumper"
      @size-change="onPageSizeChange"
      @current-change="onCurrentChange"
    />
  </el-card>
</template>
