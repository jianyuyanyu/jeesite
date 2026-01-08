<!--
 * Copyright (c) 2013-Now https://jeesite.com All rights reserved.
 * No deletion without permission, or be held responsible to law.
 * @author ThinkGem
-->
<template>
  <div class="jeesite-collapse-form-page">
    <div class="jeesite-collapse-form-header" v-if="$slots.title">
      <div class="jeesite-collapse-form-header-title">
        <slot name="title"></slot>
      </div>
    </div>
    <ScrollContainer ref="contentRef" :style="{ height: contentHeight + 'px' }" v-loading="props.loading">
      <div v-for="item in configList" :key="item.value">
        <Collapse :class="item.value" :default-active-key="item.open ? [item.value] : []">
          <Collapse.Panel :key="item.value" :header="item.label" :forceRender="true">
            <slot :name="item.value"></slot>
          </Collapse.Panel>
        </Collapse>
      </div>
    </ScrollContainer>
    <div class="jeesite-collapse-form-actions">
      <slot v-if="$slots.actions" name="actions"></slot>
      <template v-else>
        <a-button type="default" @click="handleClose" v-auth="props.okAuth">
          <Icon icon="i-ant-design:close-outlined" /> {{ t('common.closeText') }}
        </a-button>
        <a-button type="primary" @click="handleSubmit" :loading="props.loading || props.okLoading">
          <Icon icon="i-ant-design:check-outlined" /> {{ t('common.okText') }}
        </a-button>
      </template>
    </div>
  </div>
</template>
<script lang="ts" setup name="CollapseForm">
  import { nextTick, ref, watch } from 'vue';
  import { Collapse } from 'ant-design-vue';
  import { Icon } from '@jeesite/core/components/Icon';
  import { useI18n } from '@jeesite/core/hooks/web/useI18n';
  import { propTypes } from '@jeesite/core/utils/propTypes';
  import { ScrollContainer } from '@jeesite/core/components/Container';
  import { useWindowSizeFn } from '@jeesite/core/hooks/event/useWindowSizeFn';
  import { onMountedOrActivated } from '@jeesite/core/hooks/core/onMountedOrActivated';
  import { useLayoutHeight } from '@jeesite/core/layouts/default/content/useContentViewHeight';

  const props = defineProps({
    config: propTypes.array.def([]),
    storeKey: propTypes.string.def('path'),
    okAuth: propTypes.string,
    loading: propTypes.bool,
    okLoading: propTypes.bool,
  });

  const emit = defineEmits(['update:checked', 'close', 'ok']);

  const { t } = useI18n();
  const configList = ref<any[]>(props.config);
  const checkedList = ref<any[]>(props.config);
  const contentRef = ref<ComponentRef>();
  const contentHeight = ref<number>(200);
  const { headerHeightRef } = useLayoutHeight();

  function calcContentHeight() {
    const parentElement = contentRef.value?.$el.parentElement;
    const tabsElement = parentElement?.querySelector('.jeesite-form-settings-tabs');
    const headerElement = parentElement?.querySelector('.jeesite-collapse-form-header');
    const actionsElement = parentElement?.querySelector('.jeesite-collapse-form-actions');
    if (parentElement && actionsElement) {
      let height =
        document.body.clientHeight -
        headerHeightRef.value -
        (tabsElement?.scrollHeight || 0) -
        actionsElement.scrollHeight -
        32;
      if (headerElement) {
        height -= headerElement.scrollHeight;
      }
      contentHeight.value = height;
    }
  }

  onMountedOrActivated(() => {
    nextTick(() => {
      calcContentHeight();
    });
  });

  useWindowSizeFn(
    () => {
      calcContentHeight();
    },
    50,
    { immediate: true },
  );

  watch(
    () => checkedList.value,
    (v) => {
      emit('update:checked', v);
    },
    {
      immediate: true,
    },
  );

  function handleClose() {
    emit('close');
  }

  function handleSubmit() {
    emit('ok');
  }
</script>
<style lang="less">
  .jeesite-collapse-form {
    &-header {
      background-color: @component-background;
      //margin-bottom: 5px;
      padding: 10px 12px;
      border-bottom: 1px solid @table-border-color;
      border-radius: 10px 10px 0 0;

      &-title {
        font-size: 16px;
      }

      .anticon {
        color: @primary-color;
      }
    }

    &-page {
      .scrollbar {
        //border-radius: 4px !important;

        &__view > div {
          //margin-bottom: 5px;

          &:last-child {
            margin-bottom: 0;
          }
        }
      }

      .ant-collapse {
        border: 0 !important;

        &-item {
          border: 0 !important;
          //border-radius: 4px !important;
        }

        &-header {
          font-size: 15px !important;
          color: fade(@primary-color, 90%) !important;
          padding: 8px 16px !important;
          border: 0 !important;
          //border-radius: 4px !important;
          border-radius: 0 !important;
          background-color: @component-background;

          .ant-collapse-expand-icon {
            padding-top: 3px;
          }
        }

        &-content {
          border: 0 !important;
          border-radius: 0 !important;
          border-bottom: 1px solid @table-border-color !important;
          //padding-top: 5px !important;
          //border-radius: 0 0 4px 4px !important;
        }

        //&-item-active {
        //  .ant-collapse-header {
        //    border-radius: 4px 4px 0 0 !important;
        //  }
        //}
      }
    }

    &-actions {
      padding: 10px;
      //margin-top: 5px;
      margin-bottom: 0;
      text-align: center;
      border-radius: 0 0 10px 10px !important;
      background-color: @component-background;

      .ant-btn {
        margin-left: 8px;
      }
    }
  }
</style>
