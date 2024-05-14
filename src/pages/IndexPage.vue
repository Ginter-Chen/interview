<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" type="number" min="0" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="add">{{
          returnButtonName()
        }}</q-btn>
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
      <div
        v-if="tempDataDeleteId !== ''"
        style="
          padding-left: 30px;
          padding-right: 30px;
          position: absolute;
          background-color: white;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
          border: 1px solid #000;
        "
      >
        <div style="font-size: 36px">提示</div>
        <div style="font-size: 25px">是否確定刪除資料</div>
        <div style="display: flex; justify-content: right; width: 300px">
          <button style="margin-right: 20px" @click="resetTempDataDeleteId">
            取消
          </button>
          <button @click="confirm">確定</button>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref, onMounted } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
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

const tempDataDeleteId = ref('');
const tempDataId = ref('');
const tempData = ref({
  name: '',
  age: '',
});
onMounted(() => {
  getData();
});
const getData = () => {
  axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a').then((response) => {
    blockData.value = response.data;
  });
};
const resetTempData = () => {
  tempDataId.value = '';
  tempData.value.name = '';
  tempData.value.age = '';
};
const add = () => {
  if (tempData.value.name === '' || tempData.value.age === '') return;
  if (tempDataId.value === '') {
    axios
      .post('https://dahua.metcfire.com.tw/api/CRUDTest', tempData.value)
      .then((response) => {
        console.log(response);
        resetTempData();
        getData();
      });
  } else {
    let patchData = {
      id: tempDataId.value,
      name: tempData.value.name,
      age: parseInt(tempData.value.age),
    };
    axios
      .patch('https://dahua.metcfire.com.tw/api/CRUDTest', patchData)
      .then((response) => {
        console.log(response.data);
        resetTempData();
        getData();
      });
  }
};
const returnButtonName = () => {
  if (tempDataId.value === '') {
    return '新增';
  } else {
    return '更新';
  }
};
const resetTempDataDeleteId = () => {
  tempDataDeleteId.value = '';
};
const confirm = () => {
  axios
    .delete(
      `https://dahua.metcfire.com.tw/api/CRUDTest/${tempDataDeleteId.value}`
    )
    .then((response) => {
      console.log(response.data);
      resetTempDataDeleteId();
      getData();
    });
};
function handleClickOption(btn, data) {
  if (btn === tableButtons.value[1]) {
    tempDataDeleteId.value = data.id;
  } else if (btn === tableButtons.value[0]) {
    tempDataId.value = data.id;
    tempData.value.name = data.name;
    tempData.value.age = data.age;
  }
}
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
