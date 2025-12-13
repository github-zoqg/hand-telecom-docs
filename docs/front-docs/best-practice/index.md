---
sidebar_position: 1
sidebar_label: "a-table表格实现行内编辑"
---

# ant design vue 表格中实现行内新增编辑与必输校验

## 具体功能概述
- 表格支持单行编辑模式（点击 "编辑" 按钮进入编辑状态）
- 支持新增空白行并直接进入编辑状态
- 编辑状态下显示输入框等表单控件
- 保存时对当前编辑行的必输项进行校验
- 校验不通过时显示错误提示
- 支持取消编辑功能（恢复原始数据）

## 实现思路
核心解决方案是**将表格编辑行与 Form 表单校验机制结合**：

1. 定义一个表单 fromData 对象用于存储表格列所有编辑字段
2. 为表格数据添加编辑状态标识（`editable` 字段）
3. 编辑状态下的单元格使用 Form 控件（Input、Select 等）
4. 通过 Form 组件的校验规则定义必输项
5. 保存时触发当前编辑行的局部校验
6. 校验通过后再执行保存逻辑

## 具体实现
![](https://cdn.nlark.com/yuque/0/2025/png/25606879/1761811738602-15edface-7068-4cf0-bce7-79f6e5598834.png)

### 页面结构
```vue

<a-form :model="formData" ref="formRef">
  <a-table
    :columns="tableColumns"
    :data-source="dataSource"
    :loading="loading"
    :pagination="pagination"
    :scroll="{ x: 1400 }"
    row-key="id"
    @change="handleTableChange"
    >
    <template #bodyCell="{ column, index, record }">
      <!-- 行号列 -->
      <template v-if="column.key === 'lineNumber'">
        {{ padLineNumber(index) }}
      </template>
      <!-- 当editable为true时出现编辑框，过滤不需要编辑的列 -->
      <template
        v-else-if="record.editable && !['lineNumber'].includes(column.key)"
        >
        <!-- 绑定到 Form 表单，字段名使用行 ID 区分不同行 -->
        <a-form-item
          :name="`${column.key}_${record._id}`"
          :rules="commonRules.dynamicRule()"
          >
          <a-input
            v-model:value="formData[`${column.key}_${record._id}`]"
            />
        </a-form-item>
      </template>
    </template>
  </a-table>
</a-form>
```

### 核心代码
```javascript
// 定义表单数据
const formData = reactive({});
const formRef = ref(null);

// 通用表单验证规则 - 所有字段统一使用相同的提示文本
const commonRules = {
  // 这个函数会为每个字段动态生成规则
  dynamicRule: () => [
    {
      required: true,
      message: "内容不能为空",
      whitespace: true, // 同时验证空白字符串
      trigger: "blur", // 在输入框失去焦点时触发验证
    },
  ],
};
```

### 实现单行的编辑、取消、保存校验
```javascript
// 新增行
const handleAddRow = () => {
  console.log("新增行");
  // 新增行逻辑
  const newRow = {
    _id: Date.now(), // 简单的 ID 生成方式
    _status: "create", // 新增行默认状态为create
    editable: true,
  };
  dataSource.unshift(newRow);
};

// 编辑行
const handleEdit = (record) => {
  console.log("编辑行", record);
  // 拿到当前行数据赋值给表单
  Object.keys(record).forEach((key) => {
    // 注意：这块_id必须为单行的唯一值，是为了区分不同行的编辑框，避免覆盖
    formData[`${key}_${record._id}`] = record[key];
    // 特殊的赋值逻辑。。。。
  });
  // 标记为编辑状态
  record._status = "edit";
  record.editable = true;
};

// 取消编辑
const handleCancel = (record) => {
  console.log("取消编辑", record);
  // 如果他是新增则删除
  if (record._status === "create") {
    dataSource.splice(dataSource.indexOf(record), 1);
    return;
  }
  // 取消编辑逻辑
  record._status = "";
  record.editable = false;
};

// 保存编辑 - 行级别验证
const handleLineSave = async (record) => {
  console.log("保存编辑", record);
  try {
    // 获取当前行所有需要验证的字段名
    const fieldsToValidate = tableColumns
      .filter((col) => !["lineNumber", "operation"].includes(col.key)) // 排除不需要验证的列
      .map((col) => `${col.key}_${record._id}`);

    // 对当前行的所有字段进行验证
    await formRef.value.validateFields(fieldsToValidate);

    // 验证通过，将表单数据更新到数据源
    updateRowData(record);

    // 标记为已保存状态
    record._status = "saved";
    record.editable = false;
  } catch (error) {
    console.log("行验证失败:", error);
    // 验证失败，不做状态更新
  }
};

// 将表单数据更新到行数据
const updateRowData = (record) => {
  // 过滤表单中包含当前行ID的key
  const formKeys = Object.keys(formData);
  const filteredKeys = formKeys.filter((key) => key.includes(`_${record._id}`));
  if (filteredKeys.length > 0) {
    filteredKeys.forEach((key) => {
      record[key.replace(`_${record._id}`, "")] = formData[key];
    });
  }
};
```

### 整体表格保存提交
```javascript
// 保存逻辑
const handleSave = async () => {
  console.log("保存数据");
  // 保存数据逻辑
  try {
    await formRef.value.validate();
    console.log("表单验证通过，提交数据:", formData);
    // 获取表单的key值
    const formKeys = Object.keys(formData);
    // 提取表单数据中的核算属性定义
    const calculationAttributeDefinitions = dataSource.map((row) => {
      const newRow = {
        ...row,
      };
      // 过滤一下表单中包含当前行ID的key
      const filteredKeys = formKeys.filter((key) =>
        key.includes(`_${row._id}`)
      );
      if (filteredKeys.length > 0) {
        filteredKeys.forEach((key) => {
          newRow[key.replace(`_${row._id}`, "")] = formData[key];
        });
      }
      return newRow;
    });
    console.log("最终表格数据：", calculationAttributeDefinitions);
    // 提交数据到后端
    // ...
  } catch (error) {
    console.log("表单验证失败:", error);
  }
};
```

## 完整主页面代码
页面中引入的一些表格组合式 API、表格配置和默认参数这块就不在下方展示了；大家根据业务需求调整修改即可。

```vue
<template>
  <a-card title="核算属性定义列表" class="result-card">
    <template #extra>
      <div class="toolbar">
        <a-space>
          <a-button @click="handleAddRow"> 新增 </a-button>
          <a-button @click="handleSave"> 保存 </a-button>
        </a-space>
      </div>
    </template>
    <a-form :model="formData" ref="formRef">
      <a-table
        :columns="tableColumns"
        :data-source="dataSource"
        :loading="loading"
        :pagination="pagination"
        :scroll="{ x: 1400 }"
        row-key="id"
        @change="handleTableChange"
      >
        <template #bodyCell="{ column, index, record }">
          <!-- 行号列 -->
          <template v-if="column.key === 'lineNumber'">
            {{ padLineNumber(index) }}
          </template>
          <!-- 操作列 -->
          <template v-else-if="column.key === 'operation'">
            <a-space>
              <a-button
                type="primary"
                v-if="!record.editable"
                @click="handleEdit(record)"
                >编辑</a-button
              >
              <a-button
                type="danger"
                v-if="record.editable"
                @click="handleCancel(record)"
                >取消</a-button
              >
              <a-button
                type="danger"
                v-if="record.editable"
                @click="handleLineSave(record)"
                >保存</a-button
              >
            </a-space>
          </template>
          <!-- 当editable为true时出现编辑框，过滤不需要编辑的列 -->
          <template
            v-else-if="record.editable && !['lineNumber'].includes(column.key)"
          >
            <!-- 绑定到 Form 表单，字段名使用行 ID 区分不同行 -->
            <a-form-item
              :name="`${column.key}_${record._id}`"
              :rules="commonRules.dynamicRule()"
            >
              <a-input
                v-model:value="formData[`${column.key}_${record._id}`]"
              />
            </a-form-item>
          </template>
        </template>
      </a-table>
    </a-form>
  </a-card>
</template>

<script setup>
import { tableColumns, mockDataSource } from "../utils/fieldMapping";
import { useTableData } from "../utils/useTableData";

// 获取到父组件传递的方法
const props = defineProps({
  queryFormData: {
    type: Object,
    default: () => {},
  },
});

// 表单数据
const formData = reactive({});
const formRef = ref(null);

// 通用表单验证规则 - 所有字段统一使用相同的提示文本
const commonRules = {
  // 这个函数会为每个字段动态生成规则
  dynamicRule: () => [
    {
      required: true,
      message: "内容不能为空",
      whitespace: true, // 同时验证空白字符串
      trigger: "blur", // 在输入框失去焦点时触发验证
    },
  ],
};

// 使用表格数据组合式函数
const {
  dataSource,
  loading,
  pagination,
  handleTableChange,
  loadData,
  refreshData,
} = useTableData({
  queryOptions: {
    preUrl: "/v1/0/apc-payment-docs/queryForPage",
  },
  queryFormData: props.queryFormData,
  onDataLoaded: (data) => {
    console.log("数据加载完成:", data);
    // 可以在这里添加数据加载后的额外处理逻辑
  },
});

// 行号小于10自动补0
const padLineNumber = computed(() => (index) => {
  return (index + 1).toString().padStart(2, "0");
});

// 编辑行
const handleEdit = (record) => {
  console.log("编辑行", record);
  // 拿到当前行数据赋值给表单
  Object.keys(record).forEach((key) => {
    // 注意：这块_id必须为单行的唯一值，是为了区分不同行的编辑框，避免覆盖
    formData[`${key}_${record._id}`] = record[key];
    // 特殊的赋值逻辑。。。。
  });
  // 标记为编辑状态
  record._status = "edit";
  record.editable = true;
};
// 取消编辑
const handleCancel = (record) => {
  console.log("取消编辑", record);
  // 如果他是新增则删除
  if (record._status === "create") {
    dataSource.splice(dataSource.indexOf(record), 1);
    return;
  }
  // 取消编辑逻辑
  record._status = "";
  record.editable = false;
};
// 保存编辑 - 行级别验证
const handleLineSave = async (record) => {
  console.log("保存编辑", record);
  try {
    // 获取当前行所有需要验证的字段名
    const fieldsToValidate = tableColumns
      .filter((col) => !["lineNumber", "operation"].includes(col.key)) // 排除不需要验证的列
      .map((col) => `${col.key}_${record._id}`);

    // 对当前行的所有字段进行验证
    await formRef.value.validateFields(fieldsToValidate);

    // 验证通过，将表单数据更新到数据源
    updateRowData(record);

    // 标记为已保存状态
    record._status = "saved";
    record.editable = false;
  } catch (error) {
    console.log("行验证失败:", error);
    // 验证失败，不做状态更新
  }
};

// 将表单数据更新到行数据
const updateRowData = (record) => {
  // 过滤表单中包含当前行ID的key
  const formKeys = Object.keys(formData);
  const filteredKeys = formKeys.filter((key) => key.includes(`_${record._id}`));
  if (filteredKeys.length > 0) {
    filteredKeys.forEach((key) => {
      record[key.replace(`_${record._id}`, "")] = formData[key];
    });
  }
};

// 新增行
const handleAddRow = () => {
  console.log("新增行");
  // 新增行逻辑
  const newRow = {
    _id: Date.now(), // 简单的 ID 生成方式
    _status: "create", // 新增行默认状态为create
    editable: true,
  };
  dataSource.unshift(newRow);
};

// 保存逻辑
const handleSave = async () => {
  console.log("保存数据");
  // 保存数据逻辑
  try {
    await formRef.value.validate();
    console.log("表单验证通过，提交数据:", formData);
    // 获取表单的key值
    const formKeys = Object.keys(formData);
    // 提取表单数据中的核算属性定义
    const calculationAttributeDefinitions = dataSource.map((row) => {
      const newRow = {
        ...row,
      };
      // 过滤一下表单中包含当前行ID的key
      const filteredKeys = formKeys.filter((key) =>
        key.includes(`_${row._id}`)
      );
      if (filteredKeys.length > 0) {
        filteredKeys.forEach((key) => {
          newRow[key.replace(`_${row._id}`, "")] = formData[key];
        });
      }
      return newRow;
    });
    console.log("最终表格数据：", calculationAttributeDefinitions);
    // 提交数据到后端
    // ...
  } catch (error) {
    console.log("表单验证失败:", error);
  }
};

// 导出逻辑
const handleExport = () => {
  console.log("导出数据");
};

</script>
```

