<script setup lang="ts">
import { ref, computed, reactive } from 'vue';
import { useRouter, useRoute } from 'vue-router';

const router = useRouter();
const route = useRoute();

// 表单状态
const formState = reactive({
  submitting: false,
  success: false,
  error: false,
  errorMessage: '',
  successMessage: '',
});

// 表单数据
const formData = reactive({
  name: '',
  email: '',
  password: '',
  confirmPassword: '',
});

// 备份数据（用于取消时恢复）
const initialFormData = { ...formData };

// 表单引用
const formRef = ref();

// 表单验证规则
const rules = {
  name: [
    (v: string) => !!v || '姓名不能为空',
    (v: string) => v.length >= 2 || '姓名至少2个字符',
    (v: string) => v.length <= 20 || '姓名不能超过20个字符',
  ],
  email: [
    (v: string) => !!v || '邮箱不能为空',
    (v: string) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v) || '请输入有效的邮箱地址',
  ],
  password: [
    (v: string) => !!v || '密码不能为空',
    (v: string) => v.length >= 8 || '密码至少8个字符',
    (v: string) => v.length <= 32 || '密码不能超过32个字符',
    (v: string) => /(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(v) || '密码必须包含大小写字母和数字',
  ],
  confirmPassword: [
    (v: string) => !!v || '请确认密码',
    (v: string) => v === formData.password || '两次输入的密码不一致',
  ],
};

// 密码可见性
const showPassword = ref(false);
const showConfirmPassword = ref(false);

// 模拟API调用
const simulateApiCall = () => {
  return new Promise<void>((resolve, reject) => {
    setTimeout(() => {
      if (Math.random() > 0.2) {
        resolve();
      } else {
        reject(new Error('服务器暂时不可用，请稍后重试'));
      }
    }, 2000);
  });
};

// 提交表单
const handleSubmit = async () => {
  formState.success = false;
  formState.error = false;

  const isValid = await formRef.value?.validate();
  if (!isValid) {
    formState.error = true;
    formState.errorMessage = '请检查表单中的错误信息';
    return;
  }

  formState.submitting = true;

  try {
    await simulateApiCall();
    formState.success = true;
    formState.successMessage = '注册成功！您的账号已创建。';
    formState.error = false;
  } catch (error: any) {
    formState.error = true;
    formState.errorMessage = error.message || '注册失败，请稍后重试';
    formState.success = false;
  } finally {
    formState.submitting = false;
  }
};

// 重置表单
const handleReset = () => {
  formRef.value?.reset();
  Object.assign(formData, initialFormData);
  formState.success = false;
  formState.error = false;
};

// 取消操作
const handleCancel = () => {
  const hasChanges = Object.keys(initialFormData).some(
    (key) => formData[key as keyof typeof formData] !== initialFormData[key as keyof typeof initialFormData]
  );

  if (hasChanges) {
    const confirmed = window.confirm('您有未保存的更改，确定要离开吗？');
    if (confirmed) {
      handleReset();
      router.push('/auth/login');
    }
  } else {
    router.push('/auth/login');
  }
};

// 返回登录页
const handleBack = () => {
  router.push('/auth/login');
};

// 关闭提示
const closeAlert = () => {
  formState.success = false;
  formState.error = false;
};
</script>

<template>
  <v-row class="d-flex mb-3">
    <!-- 成功提示 -->
    <v-col cols="12">
      <v-alert
        v-if="formState.success"
        color="success"
        variant="tonal"
        class="mb-3"
        dismissible
        @dismiss="closeAlert"
      >
        <template v-slot:prepend>
          <v-icon>mdi-check-circle</v-icon>
        </template>
        {{ formState.successMessage }}
      </v-alert>
    </v-col>

    <!-- 错误提示 -->
    <v-col cols="12">
      <v-alert
        v-if="formState.error"
        color="error"
        variant="tonal"
        class="mb-3"
        dismissible
        @dismiss="closeAlert"
      >
        <template v-slot:prepend>
          <v-icon>mdi-alert-circle</v-icon>
        </template>
        {{ formState.errorMessage }}
      </v-alert>
    </v-col>

    <!-- 表单区域 -->
    <v-form
      ref="formRef"
      v-model="formData"
      :rules="rules"
      @submit.prevent="handleSubmit"
    >
      <!-- 姓名 -->
      <v-col cols="12">
        <v-text-field
          v-model="formData.name"
          label="姓名"
          placeholder="请输入您的姓名"
          variant="outlined"
          color="primary"
          :rules="rules.name"
          required
          :disabled="formState.submitting"
          prepend-inner-icon="mdi-account"
        />
      </v-col>

      <!-- 邮箱地址 -->
      <v-col cols="12">
        <v-text-field
          v-model="formData.email"
          label="邮箱地址"
          placeholder="请输入您的邮箱地址"
          variant="outlined"
          color="primary"
          type="email"
          :rules="rules.email"
          required
          :disabled="formState.submitting"
          prepend-inner-icon="mdi-email"
        />
      </v-col>

      <!-- 密码 -->
      <v-col cols="12">
        <v-text-field
          v-model="formData.password"
          label="密码"
          placeholder="请输入密码"
          variant="outlined"
          color="primary"
          :type="showPassword ? 'text' : 'password'"
          :rules="rules.password"
          required
          :disabled="formState.submitting"
          prepend-inner-icon="mdi-lock"
          :append-inner-icon="showPassword ? 'mdi-eye-off' : 'mdi-eye'"
          @click:append-inner="showPassword = !showPassword"
        >
          <template v-slot:details>
            <div class="text-caption text-muted">
              密码需包含大小写字母和数字，长度8-32位
            </div>
          </template>
        </v-text-field>
      </v-col>

      <!-- 确认密码 -->
      <v-col cols="12">
        <v-text-field
          v-model="formData.confirmPassword"
          label="确认密码"
          placeholder="请再次输入密码"
          variant="outlined"
          color="primary"
          :type="showConfirmPassword ? 'text' : 'password'"
          :rules="rules.confirmPassword"
          required
          :disabled="formState.submitting"
          prepend-inner-icon="mdi-lock-check"
          :append-inner-icon="showConfirmPassword ? 'mdi-eye-off' : 'mdi-eye'"
          @click:append-inner="showConfirmPassword = !showConfirmPassword"
        />
      </v-col>

      <!-- 按钮区域 -->
      <v-col cols="12" class="pt-3">
        <div class="d-flex flex-wrap justify-space-between">
          <!-- 左侧：返回/取消 -->
          <div>
            <v-btn
              variant="text"
              color="secondary"
              @click="handleBack"
              :disabled="formState.submitting"
              prepend-icon="mdi-arrow-left"
            >
              返回登录
            </v-btn>
            <v-btn
              variant="text"
              color="secondary"
              @click="handleCancel"
              :disabled="formState.submitting"
              prepend-icon="mdi-close"
              class="ml-2"
            >
              取消
            </v-btn>
          </div>

          <!-- 右侧：重置/提交 -->
          <div>
            <v-btn
              variant="outlined"
              color="secondary"
              @click="handleReset"
              :disabled="formState.submitting"
              prepend-icon="mdi-refresh"
            >
              重置
            </v-btn>
            <v-btn
              type="submit"
              color="primary"
              size="large"
              block
              flat
              :loading="formState.submitting"
              :disabled="formState.submitting"
              prepend-icon="mdi-check"
              class="ml-2"
            >
              {{ formState.submitting ? '注册中...' : 'Sign up' }}
            </v-btn>
          </div>
        </div>
      </v-col>
    </v-form>
  </v-row>
</template>