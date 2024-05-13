<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model.number="tempData.age" label="年龄" type="number" />
        <q-btn color="primary" class="q-mt-md" @click="addData()">新增</q-btn>
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
              <div>{{ col.value }}</div>
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

    <!-- 修改 -->
    <q-dialog v-model="isDialogOpen" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <q-input
            filled
            v-model="editData.name"
            label="姓名"
            class="q-ma-sm"
            :rules="[(val) => !!val || '姓名不得空白']"
          />
          <q-input
            filled
            v-model="editData.age"
            label="年齡"
            type="number"
            class="q-ma-sm"
            :rules="[
              (val) => !!val || '年齡不得空白',
              (val) => (val > 0 && Number.isInteger(val)) || '年齡必須為正整數',
            ]"
          />
        </q-card-section>
        <q-card-section align="right">
          <q-btn flat label="取消" color="negative" @click="closeDialog" />
          <q-btn flat label="確認修改" color="positive" @click="updateData" />
        </q-card-section>
      </q-card>
    </q-dialog>

    <q-dialog v-model="isDeleteDialogOpen">
      <q-card>
        <q-card-title>
          <p>提示</p>
        </q-card-title>
        <q-card-section>
          <div class="text-h6">是否刪除該筆資料？</div>
        </q-card-section>

        <q-card-section align="right">
          <q-btn
            flat
            label="取消"
            color="negative"
            @click="closeDeleteDialog"
          />
          <q-btn flat label="確認" color="positive" @click="confirmDelete" />
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup lang="ts">
import { QTableProps } from 'quasar';
import { ref, onMounted } from 'vue';
import { api } from 'src/boot/axios';

interface BtnType {
  label: string;
  icon: string;
  status: string;
}

const blockData = ref([]);
const isDialogOpen = ref(false);
const editData = ref({ id: '', name: '', age: '' });

const isDeleteDialogOpen = ref(false);
const deleteCandidate = ref(null);

function openEditDialog(data) {
  editData.value = { ...data };
  isDialogOpen.value = true;
}

function closeDialog() {
  isDialogOpen.value = false;
}

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

// 抓取
async function fetchBlockData() {
  try {
    const response = await api.get('/CRUDTest/a');
    blockData.value = response.data;
  } catch (error) {
    console.error('Failed to fetch data:', error);
  }
}

// 新增
async function addData() {
  try {
    await api.post('/CRUDTest', tempData.value);
    fetchBlockData();
    tempData.value = { name: '', age: '' };
  } catch (error) {
    console.error('新增失敗:', error);
  }
}

// 刪除數據
async function deleteData(id) {
  try {
    await api.delete(`/CRUDTest/${id}`);
    fetchBlockData();
  } catch (error) {
    console.error('無法刪除:', error);
  }
}
//更新
async function updateData() {
  try {
    await api.patch('/CRUDTest/', {
      ID: editData.value.id,
      name: editData.value.name,
      age: editData.value.age,
    });
    fetchBlockData();
    closeDialog();
  } catch (error) {
    console.error('更新失敗:', error);
  }
}

function confirmDelete() {
  deleteData(deleteCandidate.value);
  closeDeleteDialog();
}

function openDeleteDialog(id) {
  deleteCandidate.value = id;
  isDeleteDialogOpen.value = true;
}

function closeDeleteDialog() {
  isDeleteDialogOpen.value = false;
}

function handleClickOption(btn, data) {
  if (btn.status === 'edit') {
    openEditDialog(data);
  } else if (btn.status === 'delete') {
    openDeleteDialog(data.id);
  }
}

onMounted(fetchBlockData);
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
