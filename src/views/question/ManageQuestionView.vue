<template>
  <div id="manageQuestionView">
    <a-button
      type="primary"
      @click="handleClick"
      style="margin-left: 10%; margin-bottom: 30px"
      >创建题目
    </a-button>
    <a-modal
      v-model:visible="visible"
      @ok="handleOk"
      @cancel="handleCancel"
      width="1000px"
    >
      <template #title> Title</template>
      <div>
        <AddQuestionView />
      </div>
    </a-modal>
    <a-modal
      v-model:visible="visible2"
      @ok="handleOk2"
      @cancel="handleCancel2"
      width="1000px"
    >
      <template #title> Title</template>
      <div>
        <MdViewer :value="MDValue" />
      </div>
    </a-modal>
    <a-card style="width: 90%; margin: auto">
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
        <template #id="{ record }">
          <div class="id-container">
            {{ record.id }}
          </div>
        </template>
        <template #tags="{ record }">
          <div v-for="item in record.tags" :key="item">
            <a-tag>
              {{ item }}
            </a-tag>
          </div>
        </template>
        <template #judgeConfig="{ record }">
          <a-descriptions title="" :column="{ xs: 1, md: 3, lg: 4 }">
            <a-descriptions-item label="timeLimit:">
              <a-tag>{{ record.judgeConfig.timeLimit }}</a-tag>
            </a-descriptions-item>
            <a-descriptions-item label="memoryLimit:">
              <a-tag>{{ record.judgeConfig.memoryLimit }}</a-tag>
            </a-descriptions-item>
            <a-descriptions-item label="stackLimit:">
              <a-tag>{{ record.judgeConfig.stackLimit }}</a-tag>
            </a-descriptions-item>
          </a-descriptions>
        </template>
        <template #createTime="{ record }">
          {{ moment(record.createTime).format("YYYY-MM-DD") }}
        </template>
        <template #optional="{ record }">
          <a-space>
            <a-button status="success" @click="handleClick2(record.content)"
              >查看题目内容
            </a-button>
            <a-button status="warning" @click="handleClick2(record.answer)"
              >查看题目答案
            </a-button>
            <a-button type="primary" @click="doUpdate(record)"> 修改</a-button>
            <a-button status="danger" @click="doDelete(record)">删除</a-button>
          </a-space>
        </template>
      </a-table>
    </a-card>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, watchEffect } from "vue";
import { Question, QuestionControllerService } from "../../../generated";
import message from "@arco-design/web-vue/es/message";
import { useRouter } from "vue-router";
import moment from "moment";
import AddQuestionView from "@/views/question/AddQuestionView.vue";
import MdViewer from "@/components/MdViewer.vue";

const visible = ref(false);
const visible2 = ref(false);
const MDValue = ref();
const handleClick = () => {
  visible.value = true;
};
const handleClick2 = (value) => {
  MDValue.value = value;
  visible2.value = true;
};
const handleOk = () => {
  visible.value = false;
};
const handleOk2 = () => {
  visible2.value = false;
};
const handleCancel = () => {
  visible.value = false;
};
const handleCancel2 = () => {
  visible2.value = false;
};
const tableRef = ref();

const dataList = ref([]);
const total = ref(0);
const searchParams = ref({
  pageSize: 10,
  current: 1,
});

const loadData = async () => {
  const res = await QuestionControllerService.listQuestionVoByPageUsingPost(
    searchParams.value
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

// {id: "1", title: "A+ D", content: "新的题目内容", tags: "["二叉树"]", answer: "新的答案", submitNum: 0,…}

const columns = [
  {
    title: "id",
    slotName: "id",
    width: 100,
  },
  {
    title: "标题",
    dataIndex: "title",
  },
  {
    title: "标签",
    slotName: "tags",
  },
  {
    title: "提交数",
    dataIndex: "submitNum",
  },
  {
    title: "通过数",
    dataIndex: "acceptedNum",
  },
  {
    title: "判题配置",
    slotName: "judgeConfig",
  },
  {
    title: "判题用例",
    dataIndex: "judgeCase",
  },
  {
    title: "用户id",
    dataIndex: "userId",
  },
  {
    title: "创建时间",
    slotName: "createTime",
    // width: "150",
  },
  {
    title: "操作",
    slotName: "optional",
  },
];

const onPageChange = (page: number) => {
  searchParams.value = {
    ...searchParams.value,
    current: page,
  };
};

const doDelete = async (question: Question) => {
  const res = await QuestionControllerService.deleteQuestionUsingPost({
    id: question.id,
  });
  if (res.code === 0) {
    message.success("删除成功");
    loadData();
  } else {
    message.error("删除失败");
  }
};

const router = useRouter();

const doUpdate = (question: Question) => {
  router.push({
    path: "/update/question",
    query: {
      id: question.id,
    },
  });
};
</script>

<style scoped>
#manageQuestionView {
}

.id-container {
  width: 100px;
  font-size: 14px;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  display: block;
}
</style>
