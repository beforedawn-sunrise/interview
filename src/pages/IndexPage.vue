<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="addData">新增</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div v-if="editingIndex !== props.row.id">
                {{ col.value }}
              </div>
              <q-input
                v-if="editingIndex == props.row.id"
                filled
                :model-value="col.value"
                @update:model-value="(value) => emit('update:text', value)"
                label="Filled"
              />
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref, onMounted, watch } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}

interface blockDataType {
  name: string;
  age: number;
  id: string;
}
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
] as blockDataType[]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tempData = ref({
  name: '',
  age: '',
});

const editingIndex = ref('aaa');

const deleteIndex = ref('bbb');

const countEditBtnClicks = ref(0);

async function handleClickOption(btn, data) {
  console.log(btn);
  if (btn.status == 'edit') {
    countEditBtnClicks.value++;
    console.log(countEditBtnClicks.value);
    editingIndex.value = data.id;

    if (countEditBtnClicks.value % 2 == 0) {
      await editData();
    }
  } else {
    deleteIndex.value = data.id;
    await deleteData();
  }
}

async function deleteData() {
  try {
    const response = await axios.delete(
      `https://dahua.metcfire.com.tw/api/CRUDTest/${deleteIndex.value}`
    );
    if (response.status === 200) {
      await getData();
    }
    return response.data;
  } catch (error) {
    return error;
  }
}
const addData = async () => {
  try {
    const response = await axios.post(
      'https://dahua.metcfire.com.tw/api/CRUDTest',
      tempData.value
    );
    if (response.status === 200) {
      await getData();
    }
    return response.data;
  } catch (error) {
    return error;
  }
};

const getData = async () => {
  try {
    const response = await axios.get(
      'https://dahua.metcfire.com.tw/api/CRUDTest/a'
    );
    console.log(response.data);

    blockData.value = response.data;
  } catch (error) {
    return error;
  }
};

const editData = async () => {
  console.log(blockData.value);
  const editObject = blockData.value.filter(
    (item) => item.id == editingIndex.value
  )[0];
  try {
    const response = await axios.patch(
      'https://dahua.metcfire.com.tw/api/CRUDTest',
      editObject
    );
    if (response.status === 200) {
      editingIndex.value = 'aaa';
      await getData();
    }
    return response.data;
  } catch (error) {
    return error;
  }
};

onMounted(async () => {
  await getData();
});

const emit = defineEmits(['update:text']);

watch(blockData, (newVal) => {
  // 當值變化時，發出 input 事件
  emit('update:text', newVal);
});
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
