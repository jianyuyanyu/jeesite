<!--
 * Copyright (c) 2013-Now https://jeesite.com All rights reserved.
 * No deletion without permission, or be held responsible to law.
 * @author ThinkGem
-->
<template>
  <BasicForm @register="registerForm">
    <template #content="{ model, field }">
      <WangEditor v-model:value="model[field]" :bizKey="record.id" :bizType="'article_content'" :height="400" />
    </template>
  </BasicForm>
</template>
<script lang="ts" setup name="ViewsCmsArticleFormDetail">
  import { ref } from 'vue';
  import { useI18n } from '@jeesite/core/hooks/web/useI18n';
  import { BasicForm, FormSchema, useForm } from '@jeesite/core/components/Form';
  import { Article } from '@jeesite/cms/api/cms/article';
  import type { NamePath } from 'ant-design-vue/lib/form/interface';
  import { WangEditor } from '@jeesite/core/components/WangEditor';

  const { t } = useI18n('cms.article');
  const record = ref<Article>({} as Article);

  const inputFormSchemas: FormSchema<Article>[] = [
    {
      label: t('摘要'),
      field: 'description',
      component: 'InputTextArea',
      componentProps: {
        maxlength: 500,
      },
      colProps: { md: 24, lg: 24 },
    },
    {
      label: t('正文'),
      field: 'articleData.content',
      component: 'InputTextArea',
      componentProps: {
        maxlength: 1000,
      },
      required: true,
      slot: 'content',
      colProps: { md: 24, lg: 24 },
    },
  ];

  const [registerForm, { resetFields, setFieldsValue, validate }] = useForm<Article>({
    labelWidth: 70,
    schemas: inputFormSchemas,
    baseColProps: { md: 24, lg: 12 },
  });

  defineExpose({
    resetFields: async () => {
      await resetFields();
    },
    setFieldsValue: async (values: Recordable) => {
      record.value = values as Article;
      await setFieldsValue(values);
    },
    validate: async (nameList?: NamePath[]) => {
      return await validate(nameList);
    },
  });
</script>
