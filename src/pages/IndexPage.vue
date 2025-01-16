<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" type="number" />
        <q-btn color="primary" class="q-mt-md" @click="createData">新增</q-btn>
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
              <div>{{ props.row[col.field] }}</div>
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
import { ref, onMounted } from 'vue';

const API_BASE_URL = 'https://dahua.metcfire.com.tw/api/CRUDTest';

const blockData = ref([]);
const tableConfig = [
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
];
const tableButtons = [
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
];
const tempData = ref({
  name: '',
  age: '',
});

async function fetchData() {
  try {
    const response = await axios.get(`${API_BASE_URL}/a`);
    blockData.value = response.data || [];
  } catch (error) {
    console.error('取得資料失敗');
  }
}

async function createData() {
  if (!tempData.value.name || !tempData.value.age) {
    alert('請填寫姓名與年齡');
    return;
  }

  try {
    await axios.post(
      API_BASE_URL,
      {
        name: tempData.value.name,
        age: tempData.value.age,
      },
      {
        headers: {
          'Content-Type': 'application/json',
        },
      }
    );
    tempData.value = {
      name: '',
      age: '',
    };
    await fetchData();
  } catch (error) {
    console.error('新增資料失敗');
  }
}

async function handleEdit(row) {
  const newName = prompt('請輸入新姓名', row.name);
  const newAge = prompt('請輸入新年齡', row.age);
  if (newName && newAge) {
    try {
      const response = await axios.patch(
        `${API_BASE_URL}`,
        {
          id: row.id,
          name: newName,
          age: newAge,
        },
        {
          headers: {
            'Content-Type': 'application/json',
          },
        }
      );

      const index = blockData.value.findIndex((item) => item.id === row.id);
      if (index !== -1) {
        blockData.value[index].name = newName;
        blockData.value[index].age = newAge;
      }
    } catch (error) {
      console.error('更新資料失敗');
    }
  }
}

async function handleDelete(row) {
  if (!confirm('確定刪除此筆資料?')) return;
  try {
    const response = await axios.delete(`${API_BASE_URL}/${row.id}`);

    fetchData();
  } catch (error) {
    console.error('刪除資料失敗');
  }
}

async function handleClickOption(btn, row) {
  if (btn.status === 'edit') {
    await handleEdit(row);
  } else if (btn.status === 'delete') {
    await handleDelete(row);
  }
}

onMounted(() => {
  fetchData();
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
