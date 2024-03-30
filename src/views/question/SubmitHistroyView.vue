<template>
  <div id="questionSubmitView">
    <a-form :model="searchParams" layout="inline">
      <!--      <a-form-item field="questionId" label="题号" style="min-width: 240px">-->
      <!--        <a-input v-model="searchParams.questionId" placeholder="请输入" />-->
      <!--      </a-form-item>-->
      <a-form-item field="language" label="编程语言" style="min-width: 240px">
        <a-select
          v-model="searchParams.language"
          :style="{ width: '320px' }"
          placeholder="选择编程语言"
        >
          <a-option>java</a-option>
          <!--          <a-option>cpp</a-option>-->
          <!--          <a-option>go</a-option>-->
          <!--          <a-option>html</a-option>-->
        </a-select>
      </a-form-item>
      <a-form-item>
        <a-button @click="reloadHistorySubmit">
          <icon-refresh />
        </a-button>
      </a-form-item>
    </a-form>
    <a-divider size="0" />
    <a-table
      :ref="tableRef"
      :columns="columns"
      :data="dataList"
      :pagination="{
        showTotal: true,
        pageSize: searchParams.pageSize,
        current: searchParams.current,
        total,
      }"
      @page-change="onPageChange"
    >
      <template #judgeInfo="{ record }">
        {{ JSON.stringify(record.judgeInfo.message).replace(/^"(.*)"$/, "$1") }}
      </template>
      <template #status="{ record }">
        <a-space wrap>
          <a-tag v-if="record.status == 0" color="gray">
            {{ "待判题" }}
          </a-tag>
          <a-tag v-else-if="record.status == 1" color="brown">
            {{ "判题中" }}
          </a-tag>
          <a-tag v-else-if="record.status == 2" color="green">
            {{ "成功" }}
          </a-tag>
          <a-tag v-else-if="record.status == 3" color="red">
            {{ "失败" }}
          </a-tag>
        </a-space>
      </template>
      <template #createTime="{ record }">
        {{ formatTime(record.createTime) }}
      </template>
      <template #optional="{ record }">
        <a-space>
          <a-button type="primary" @click="lookCode(record)">
            查看代码
          </a-button>
        </a-space>
      </template>
    </a-table>
    <a-modal
      v-model:visible="visible"
      @ok="handleOk"
      @cancel="handleCancel"
      width="auto"
    >
      <template #title> 代码</template>
      <pre>{{ showCode }}</pre>
    </a-modal>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, watchEffect } from "vue";
import {
  QuestionControllerService,
  QuestionSubmitQueryRequest,
  QuestionSubmitVO,
} from "../../../generated";
import message from "@arco-design/web-vue/es/message";
import { useRouter } from "vue-router";
import { useStore } from "vuex";
import { defineProps, withDefaults } from "vue/dist/vue";

interface Props {
  value: number;
}

/**
 * 给组件指定初始值
 */
const props = withDefaults(defineProps<Props>(), {
  value: () => 0,
});
const tableRef = ref();
const store = useStore();
const dataList = ref([]);
const total = ref(0);
const searchParams = ref<QuestionSubmitQueryRequest>({
  questionId: undefined,
  language: undefined,
  pageSize: 10,
  current: 1,
});

const loadData = async () => {
  const res = await QuestionControllerService.listQuestionSubmitByPageUsingPost(
    {
      ...searchParams.value,
      sortField: "createTime",
      sortOrder: "descend",
      questionId: props.value,
      userId: store.state.user?.loginUser?.id,
    }
  );
  if (res.code === 0) {
    dataList.value = res.data.records;
    total.value = res.data.total;
  } else {
    message.error("加载失败，" + res.message);
  }
};

/**
 * 监听 searchParams 变量，改变时触发页面的重新加载
 */
watchEffect(() => {
  loadData();
});

/**
 * 页面加载时，请求数据
 */
onMounted(() => {
  loadData();
});
const formatTime = (time) => {
  // 创建一个Date对象
  const date = new Date(time);
  // 获取小时、分钟和秒
  const hours = date.getHours().toString().padStart(2, "0");
  const minutes = date.getMinutes().toString().padStart(2, "0");
  const seconds = date.getSeconds().toString().padStart(2, "0");
  // 格式化时间为24小时制
  return `${date.getFullYear()}-${(date.getMonth() + 1)
    .toString()
    .padStart(2, "0")}-${date
    .getDate()
    .toString()
    .padStart(2, "0")} ${hours}:${minutes}:${seconds}`;
};
const columns = [
  {
    title: "编程语言",
    dataIndex: "language",
  },
  {
    title: "判题信息",
    slotName: "judgeInfo",
  },
  {
    title: "判题状态",
    slotName: "status",
  },
  {
    title: "创建时间",
    slotName: "createTime",
  },
  {
    slotName: "optional",
  },
];

const onPageChange = (page: number) => {
  searchParams.value = {
    ...searchParams.value,
    current: page,
  };
};
const reloadHistorySubmit = () => {
  loadData();
};
const router = useRouter();

/**
 * 确认搜索，重新加载数据
 */
const doSubmit = () => {
  // 这里需要重置搜索页号
  searchParams.value = {
    ...searchParams.value,
    current: 1,
  };
};
const visible = ref(false);
const showCode = ref("");
const handleOk = () => {
  visible.value = false;
};
const handleCancel = () => {
  visible.value = false;
};
const lookCode = (questionSubmitVO: QuestionSubmitVO) => {
  showCode.value = questionSubmitVO.code;
  visible.value = true;
  // alert(questionSubmitVO.code);
};
</script>

<style scoped>
#questionSubmitView {
  max-width: 1280px;
  margin: 0 auto;
}
</style>
