<template>
  <q-page class="row q-pt-xl page-background">
    <div class="form-container full-width q-px-xl q-pb-xl">
      <div class="logo-container">
        <img
          src="https://source.unsplash.com/random/150x150"
          alt="Logo"
          class="logo"
        />
      </div>
      <div class="q-mb-xl">
        <q-input
          v-model="tempData.name"
          label="姓名"
          :error="!tempData.name && errorVisible"
          error-message="姓名為必填"
          dense
          outlined
          clearable
          class="input-3d"
        />
        <q-input
          v-model="tempData.age"
          label="年齡"
          type="number"
          :error="!tempData.age && errorVisible"
          error-message="年齡為必填"
          dense
          outlined
          clearable
          class="q-mt-md input-3d"
        />
        <q-btn
          color="primary"
          class="q-mt-md full-width btn-3d"
          @click="addOrUpdateEntry"
        >
          {{ editMode ? '更新' : '新增' }}
        </q-btn>
      </div>

      <q-table
        flat
        bordered
        :rows="blockData"
        :columns="tableConfig"
        row-key="id"
        hide-pagination
        separator="cell"
        class="bg-white table-3d"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th>操作</q-th>
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
              {{ props.row[col.field] }}
            </q-td>
            <q-td class="text-right" auto-width>
              <q-btn
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md btn-3d"
                @click="handleClickOption(btn, props.row)"
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
    <q-notify position="top-right" />
  </q-page>
</template>

<script setup lang="ts">
import { uid, useQuasar } from 'quasar';
import { ref } from 'vue';

// Define types for button and data entries
interface btnType {
  label: string;
  icon: string;
  status: string;
}

interface DataType {
  id: string;
  name: string;
  age: number;
}

// Initialize the Quasar instance for notifications
const $q = useQuasar();

// Sample data
const blockData = ref<DataType[]>([
  {
    id: uid(),
    name: 'test',
    age: 25,
  },
]);

// Table configuration
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

// Table action buttons
const tableButtons = ref<btnType[]>([
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

// Temporary data for form
const tempData = ref<DataType>({
  id: '',
  name: '',
  age: 0,
});
const errorVisible = ref(false);
const editMode = ref(false);

// Add or update entry in the table
function addOrUpdateEntry() {
  errorVisible.value = !tempData.value.name || !tempData.value.age;

  if (!errorVisible.value) {
    if (editMode.value) {
      const index = blockData.value.findIndex(
        (item) => item.id === tempData.value.id
      );
      if (index !== -1) {
        blockData.value[index] = { ...tempData.value };
        $q.notify({ message: '資料已更新', color: 'positive' });
      }
    } else {
      blockData.value.push({ ...tempData.value, id: uid() });
      $q.notify({ message: '資料已新增', color: 'positive' });
    }
    resetForm();
  }
}

// Handle edit and delete button clicks
function handleClickOption(btn, row) {
  if (btn.status === 'edit') {
    editMode.value = true;
    tempData.value = { ...row };
  } else if (btn.status === 'delete') {
    const index = blockData.value.findIndex((item) => item.id === row.id);
    if (index !== -1) {
      blockData.value.splice(index, 1);
      $q.notify({ message: '資料已刪除', color: 'negative' });
    }
  }
}

// Reset the form fields and switch off edit mode
function resetForm() {
  tempData.value = { id: '', name: '', age: 0 };
  editMode.value = false;
}
</script>

<style lang="scss" scoped>
.page-background {
  background: url('https://source.unsplash.com/random/1600x900') no-repeat
    center center fixed;
  background-size: cover;
  color: #ffffff; /* Set font color to white */
}

.form-container {
  background-color: rgba(234, 238, 230, 0.7);
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  transform: perspective(1000px) rotateX(5deg) translateZ(10px);
}

.logo-container {
  text-align: center;
  margin-bottom: 20px;
}

.logo {
  width: 150px;
  height: auto;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  transform: perspective(1000px) translateZ(10px);
}

.input-3d {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  transform: perspective(1000px) translateZ(10px);
  color: #333; /* Set font color to a dark gray */
}

.btn-3d {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  transform: perspective(1000px) translateZ(10px);
  color: #fff; /* Set font color to white */
}

.table-3d {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  transform: perspective(1000px) translateZ(10px);
  color: #333; /* Set font color to a dark gray */
}

.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
  color: #020202;
}
</style>
