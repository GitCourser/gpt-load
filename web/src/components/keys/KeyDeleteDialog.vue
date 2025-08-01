<script setup lang="ts">
import { keysApi } from "@/api/keys";
import { Close } from "@vicons/ionicons5";
import { NButton, NCard, NInput, NModal, useMessage } from "naive-ui";
import { ref, watch } from "vue";

interface Props {
  show: boolean;
  groupId: number;
  groupName?: string;
}

interface Emits {
  (e: "update:show", value: boolean): void;
  (e: "success"): void;
}

const props = defineProps<Props>();

const emit = defineEmits<Emits>();

const loading = ref(false);
const keysText = ref("");
const message = useMessage();

// 监听弹窗显示状态
watch(
  () => props.show,
  show => {
    if (show) {
      resetForm();
    }
  }
);

// 重置表单
function resetForm() {
  keysText.value = "";
}

// 关闭弹窗
function handleClose() {
  emit("update:show", false);
}

// 提交表单
async function handleSubmit() {
  if (loading.value || !keysText.value.trim()) {
    return;
  }

  try {
    loading.value = true;

    const res = await keysApi.deleteKeys(props.groupId, keysText.value);
    const { deleted_count, ignored_count, total_in_group } = res || {};
    const msg = `成功删除 ${deleted_count} 个密钥，忽略 ${ignored_count} 个密钥。当前分组共有 ${total_in_group} 个密钥。`;
    message.info(msg, {
      closable: true,
      duration: 5000,
    });

    emit("success");
    handleClose();
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <n-modal :show="show" @update:show="handleClose" class="form-modal">
    <n-card
      style="width: 800px"
      :title="`删除 ${groupName || '当前分组'} 的密钥`"
      :bordered="false"
      size="huge"
      role="dialog"
      aria-modal="true"
    >
      <template #header-extra>
        <n-button quaternary circle @click="handleClose">
          <template #icon>
            <n-icon :component="Close" />
          </template>
        </n-button>
      </template>

      <n-input
        v-model:value="keysText"
        type="textarea"
        placeholder="输入要删除的密钥，每行一个"
        :rows="8"
        style="margin-top: 20px"
      />

      <template #footer>
        <div style="display: flex; justify-content: flex-end; gap: 12px">
          <n-button @click="handleClose">取消</n-button>
          <n-button type="error" @click="handleSubmit" :loading="loading" :disabled="!keysText">
            删除
          </n-button>
        </div>
      </template>
    </n-card>
  </n-modal>
</template>

<style scoped>
.form-modal {
  --n-color: rgba(255, 255, 255, 0.95);
}

:deep(.n-input) {
  --n-border-radius: 6px;
}

:deep(.n-card-header) {
  border-bottom: 1px solid rgba(239, 239, 245, 0.8);
  padding: 10px 20px;
}

:deep(.n-card__content) {
  max-height: calc(100vh - 68px - 61px - 50px);
  overflow-y: auto;
}

:deep(.n-card__footer) {
  border-top: 1px solid rgba(239, 239, 245, 0.8);
  padding: 10px 15px;
}
</style>
