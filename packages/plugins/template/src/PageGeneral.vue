<template>
  <div class="general-config">
    <tiny-form
      ref="generalForm"
      :model="pageSettingState.currentPageData"
      :rules="templateRules"
      label-width="120px"
      validate-type="text"
      :inline-message="true"
      :label-align="true"
      label-position="left"
      class="general-config-form"
    >
      <tiny-form-item prop="group" label="新增" class="form-item-page-type">
        <tiny-radio v-model="pageSettingState.currentPageData.group" class="page-type-radio" label="staticPages">
          模板类别
        </tiny-radio>
        <tiny-radio v-model="pageSettingState.currentPageData.group" class="page-type-radio" label="publicPages">
          模板实例
        </tiny-radio>
      </tiny-form-item>
      <tiny-form-item prop="name" label="名称">
        <tiny-input v-model="pageSettingState.currentPageData.name" placeholder="请输入名称"></tiny-input>
      </tiny-form-item>

      <tiny-form-item label="选择父模板类别" prop="parentId">
        <tiny-select
          v-model="pageSettingState.currentPageData.parentId"
          value-field="id"
          render-type="tree"
          :tree-op="treeFolderOp"
          text-field="name"
          placeholder="请选择父模板类型"
          popper-class="parent-fold-select-dropdown"
          @change="changeParentForderId"
        ></tiny-select>
      </tiny-form-item>
    </tiny-form>
  </div>
</template>

<script lang="jsx">
import { ref, computed, watchEffect } from 'vue'
import { Form, FormItem, Input, Select, Radio } from '@opentiny/vue'
import { usePage } from '@opentiny/tiny-engine-controller'
// import { REGEXP_PAGE_NAME, REGEXP_FOLDER_NAME, REGEXP_ROUTE } from '@opentiny/tiny-engine-controller/js/verification'
import { SvgButton } from '@opentiny/tiny-engine-common'
import PageHome from './PageHome.vue'

export default {
  components: {
    // SvgButton,
    TinyForm: Form,
    TinyFormItem: FormItem,
    TinyInput: Input,
    TinySelect: Select,
    // PageHome,
    TinyRadio: Radio
  },
  props: {
    modelValue: {
      type: Object,
      default: () => ({})
    },
    isFolder: {
      type: Boolean,
      default: false
    }
  },
  setup() {
    const { pageSettingState, changeTreeData, STATIC_PAGE_GROUP_ID } = usePage()
    const ROOT_ID = pageSettingState.ROOT_ID
    const oldParentId = ref(pageSettingState.currentPageData.parentId)

    watchEffect(() => {
      oldParentId.value = pageSettingState.oldParentId
    })

    const currentRoute = computed(() => {
      let route = pageSettingState.currentPageData.route || ''
      let parentId = pageSettingState.currentPageData.parentId

      while (parentId !== ROOT_ID) {
        const parent = pageSettingState.treeDataMapping[parentId]
        if (!parent) {
          break
        }
        route = `${parent.route}/${route}`
        parentId = parent.parentId
      }
      if (route.startsWith('/')) {
        route = route.slice(1)
      }

      return route
    })

    // 新建模板类别/实例校验规则
    const templateRules = {
      name: [
        { required: true, message: '请输入名称' },
        {
          pattern: REGEXP_FOLDER_NAME,
          message: '只允许包含英文字母、数字、下横线_、中横线-, 且以英文字母开头'
        },
        { min: 3, max: 25, message: '长度在 3 到 25 个字符' }
      ],

      group: [{ required: true, message: '必须选择新增类型' }]
    }

    const getFolders = (pages) => {
      const list = []

      pages.forEach((page) => {
        if (!page.isPage && page.id !== pageSettingState.currentPageData.id) {
          list.push(page)
          if (!page.children) {
            page.children = []
          }
          page.children = getFolders(page.children)
        }
      })

      return list
    }

    const treeFolderOp = computed(() => {
      const expandIcon = <SvgIcon name="text-page-folder-closed" class="folder-icon"></SvgIcon>
      const shrinkIcon = <SvgIcon name="text-page-folder" class="folder-icon"></SvgIcon>
      const staticPages = pageSettingState.pages[STATIC_PAGE_GROUP_ID]?.data || []
      const data = [{ name: '无', id: ROOT_ID }, ...getFolders(JSON.parse(JSON.stringify(staticPages)))]
      const options = {
        data: data,
        shrinkIcon: shrinkIcon,
        expandIcon: expandIcon,
        renderContent: (_h, { node, data }) => {
          return (
            <span style="display:flex">
              {node.isLeaf && data.id !== ROOT_ID ? (
                <svg-icon name="text-page-folder-closed" style="margin-right:14px"></svg-icon>
              ) : null}
              <span class="label">{node.label}</span>
            </span>
          )
        }
      }

      return options
    })

    const generalForm = ref(null)

    const validGeneralForm = () => {
      return new Promise((resolve, reject) => {
        generalForm.value.validate((valid) => {
          if (valid) {
            resolve(valid)
          } else {
            reject(valid)
          }
        })
      })
    }

    const changeParentForderId = (value) => {
      changeTreeData(value.id, oldParentId.value)
      oldParentId.value = value.id
    }

    return {
      templateRules,
      pageSettingState,
      generalForm,
      validGeneralForm,
      treeFolderOp,
      currentRoute,
      changeParentForderId
    }
  }
}
</script>

<style lang="less" scoped>
.general-config {
  padding: 20px;

  .general-config-form {
    .input-head {
      color: var(--ti-lowcode-page-manage-input-head-text-color);
    }
    :deep(.tiny-form-item) {
      padding: 0 12px;
      .tiny-input-group__prepend {
        border: 1px solid var(--ti-lowcode-page-manage-input-group-border-color);
        background: var(--ti-lowcode-page-manage-input-group-color);
        border-right: 1px solid var(--ti-lowcode-page-manage-input-group-border-right-color);
      }
      .page-type-radio {
        color: var(--ti-lowcode-page-manage-title-background-text-color);
      }
      .tiny-form-item__label {
        font-size: 14px;
        color: var(--ti-lowcode-page-manage-text-color);
      }
      .tiny-form-item__error {
        font-size: 14px;
        color: var(--ti-lowcode-page-manage-error-color);
      }
    }
  }
  .tip {
    color: var(--ti-lowcode-page-manage-tip-border-color);
    font-size: 14px;
    border-radius: 3px;
    display: flex;
    align-items: center;
    .icon {
      color: var(--ti-lowcode-page-manage-icon-text-color);
    }
    .text {
      color: var(--ti-lowcode-page-manage-btn-text-color);
    }
    .text-dim {
      color: var(--ti-lowcode-page-manage-btn-text-color);
    }
  }
}
</style>
<style lang="less">
.tiny-select-dropdown.parent-fold-select-dropdown {
  .tiny-tree {
    background-color: var(--ti-lowcode-page-manage-tree-text-background-color);

    .tiny-tree-node {
      &.is-current {
        .tiny-tree-node__content {
          background-color: var(--ti-lowcode-page-manage-tree-node-background-color);
        }
      }
    }
  }

  .tiny-tree-node__content {
    &:hover {
      --ti-lowcode-tree-node-content-hover-bg-color: var(--ti-lowcode-page-manage-tree-node-background-color);
    }
  }
}
</style>
