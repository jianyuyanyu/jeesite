<!--
 * Copyright (c) 2013-Now https://jeesite.com All rights reserved.
 * No deletion without permission, or be held responsible to law.
 * @author ThinkGem
-->
<template>
  <CollapseForm
    :config="formConfig"
    :loading="loadingRef"
    :okLoading="okLoadingRef"
    :okAuth="'cms:article:edit'"
    @ok="handleSubmit"
    @close="close"
  >
    <template #title>
      <Icon :icon="getTitle.icon" class="m-1 pr-1" />
      <span> {{ getTitle.value }} </span>
    </template>
    <template #main>
      <FormBasic ref="formBasicRef" />
    </template>
    <template #detail>
      <FormDetail ref="formDetailRef" />
    </template>
    <template #other>
      <FormOther ref="formOtherRef" />
    </template>
    <template #view>
      <FormView ref="formViewRef" />
    </template>
  </CollapseForm>
</template>
<script lang="ts" setup name="ViewsCmsArticleForm">
  import { ref, unref, computed, onMounted } from 'vue';
  import { useEmitter } from '@jeesite/core/store/modules/user';
  import { useI18n } from '@jeesite/core/hooks/web/useI18n';
  import { useMessage } from '@jeesite/core/hooks/web/useMessage';
  import { router } from '@jeesite/core/router';
  import { Icon } from '@jeesite/core/components/Icon';
  import { CollapseForm } from '@jeesite/core/components/CollapseForm';
  import { Article, articleSave, articleForm } from '@jeesite/cms/api/cms/article';
  import { useQuery } from '@jeesite/core/hooks/web/usePage';
  import { useTabs } from '@jeesite/core/hooks/web/useTabs';

  import FormBasic from './formBasic.vue';
  import FormDetail from './formDetail.vue';
  import FormView from './formView.vue';
  import FormOther from './formOther.vue';

  const formConfig = ref<any[]>([
    {
      label: '基本信息',
      value: 'main',
      open: true,
    },
    {
      label: '详细信息',
      value: 'detail',
      open: true,
    },
    {
      label: '其他信息',
      value: 'other',
      open: true,
    },
    {
      label: '视图配置',
      value: 'view',
      open: true,
    },
  ]);

  const emitter = useEmitter();

  const { t } = useI18n('cms.article');
  const { showMessage } = useMessage();
  //const { meta } = unref(router.currentRoute);
  const { setTitle, close } = useTabs(router);
  const record = ref<Article>({} as Article);
  const loadingRef = ref<boolean>(false);
  const okLoadingRef = ref<boolean>(false);
  const query = useQuery();

  const getTitle = computed(() => ({
    icon: /*meta.icon || */ 'i-ant-design:book-outlined',
    value: record.value.isNewRecord ? t('新增文章') : t('编辑文章'),
  }));

  const formBasicRef = ref<InstanceType<typeof FormBasic>>();
  const formDetailRef = ref<InstanceType<typeof FormDetail>>();
  const formOtherRef = ref<InstanceType<typeof FormOther>>();
  const formViewRef = ref<InstanceType<typeof FormView>>();

  async function resetFields() {
    await formBasicRef.value?.resetFields();
    await formDetailRef.value?.resetFields();
    await formOtherRef.value?.resetFields();
    await formViewRef.value?.resetFields();
  }

  async function setFieldsValue(values: Recordable, res: any) {
    await formBasicRef.value?.setFieldsValue(values);
    await formDetailRef.value?.setFieldsValue(values);
    await formOtherRef.value?.setFieldsValue(values);
    await formViewRef.value?.setFieldsValue(values, res);
  }

  async function validate(): Promise<Recordable<Article>> {
    return Object.assign(
      await formBasicRef.value?.validate(),
      await formDetailRef.value?.validate(),
      await formOtherRef.value?.validate(),
      await formViewRef.value?.validate(),
    );
  }

  onMounted(async () => {
    loadingRef.value = true;
    await resetFields();
    const res = await articleForm(unref(query));
    record.value = (res.article || {}) as Article;
    record.value.__t = new Date().getTime();
    await setFieldsValue(record.value, res);
    await setTitle(unref(getTitle).value);
    loadingRef.value = false;
  });

  function handleClose() {}

  async function handleSubmit() {
    try {
      okLoadingRef.value = true;
      const data = await validate();
      const params: any = {
        isNewRecord: record.value.isNewRecord,
        id: record.value.id,
      };
      // console.log('submit', params, data, record);
      const res = await articleSave(params, data);
      showMessage(res.message);
      emitter.emit('cms-article-reload');
      setTimeout(close);
    } catch (error: any) {
      if (error && error.errorFields) {
        showMessage(error.message || t('common.validateError'));
      }
      console.log('error', error);
    } finally {
      okLoadingRef.value = false;
    }
  }
</script>
