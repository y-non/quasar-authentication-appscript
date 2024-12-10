<script setup>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";
import { Dialog, Loading, Notify } from "quasar";

const router = useRouter();
const username = localStorage.getItem("username");
const password = localStorage.getItem("password");
const loadingTable = ref(false);
const filterTable = ref("");

if (!username || !password) {
  router.push("/login");
}

const userData = ref([]);
const newData = ref({ umsatz: "", notizen: "" });

const columns = [
  {
    name: "umsatz",
    label: "Umsatz",
    field: "umsatz",
    sortable: true,
    align: "center",
  },
  {
    name: "notizen",
    label: "notizen",
    field: "notizen",
    sortable: true,
    align: "center",
  },
  {
    name: "datum",
    label: "Datum",
    field: "datum",
    sortable: true,
    align: "center",
  },
  {
    name: "action",
    label: "Action",
    field: "action",
    sortable: true,
    align: "center",
  },
];

const fetchData = async () => {
  try {
    loadingTable.value = true;

    const response = await axios.post(
      "https://script.google.com/macros/s/AKfycbwLnFuwFg0TwlFxD7z8x6Fx2dYsh-IWoFQeYkzdAwvHhrddhNi5TuEgMRwj1TkAc-Ek/exec",
      {
        action: "fetchData",
        username,
        password,
      }
    );

    if (response.data.success) {
      let dataResponse = response.data.data || [];
      userData.value = dataResponse.map((item) => ({
        id: item[0],
        benutzername: item[1],
        umsatz: item[2],
        notizen: item[3],
        datum: new Date(item[4]).toLocaleDateString("vi-VN"),
      }));
    } else {
      alert("Failed to fetch data");
    }

    loadingTable.value = false;
  } catch (error) {
    console.error("Error fetching data:", error);
  }
};

const addData = async () => {
  try {
    Loading.show({
      message: "Đang thêm mới dữ liệu...",
    });

    axios.defaults.headers.common['Access-Control-Allow-Origin'] = '*';

    const response = await axios.post(
      "https://script.google.com/macros/s/AKfycbwLnFuwFg0TwlFxD7z8x6Fx2dYsh-IWoFQeYkzdAwvHhrddhNi5TuEgMRwj1TkAc-Ek/exec",
      {
        action: "addData",
        username,
        password,
        data: newData.value,
      }
    );

    if (response.data.success) {
      Notify.create({
        type: "positive",
        message: "Thêm mới thành công!",
        position: "top",
      });
      newData.value = { umsatz: "", notizen: "" };
      fetchData();
    } else {
      alert("Failed to add data");
    }
    Loading.hide();
  } catch (error) {
    console.error("Error adding data:", error);
  }
};

const deleteData = async (rowId) => {
  try {
    Dialog.create({
      title: "Xác nhận",
      message: "Bạn có chắc chắn muốn xóa hàng này không ?",
      ok: true,
      cancel: true,
    }).onOk(async () => {
      Loading.show({
        message: "Đang xóa dữ liệu...",
      });

      axios.defaults.headers.common['Access-Control-Allow-Origin'] = '*';

      const response = await axios.post(
        "https://script.google.com/macros/s/AKfycbwLnFuwFg0TwlFxD7z8x6Fx2dYsh-IWoFQeYkzdAwvHhrddhNi5TuEgMRwj1TkAc-Ek/exec",
        {
          action: "deleteData",
          username,
          password,
          id: rowId,
        }
      );

      if (response.data.success) {
        Notify.create({
          type: "positive",
          message: "Xóa thành công!",
          position: "top",
        });
        fetchData();
      } else {
        alert("Failed to delete data");
      }
      Loading.hide();
    });
  } catch (error) {
    console.error("Error deleting data:", error);
  }
};

const updateData = async (rowId, columnName, value) => {
  try {
    axios.defaults.headers.common['Access-Control-Allow-Origin'] = '*';

    const response = await axios.post(
      "https://script.google.com/macros/s/AKfycbwLnFuwFg0TwlFxD7z8x6Fx2dYsh-IWoFQeYkzdAwvHhrddhNi5TuEgMRwj1TkAc-Ek/exec",
      {
        action: "updateData",
        username,
        password,
        id: rowId,
        columnChangeName: columnName,
        valueChange: value,
      }
    );

    console.log(response);

    if (response.data.success) {
    } else {
      alert("Error when update data");
    }
  } catch (error) {
    console.error("Error updating data:", error);
  }
};

const logout = () => {
  const result = Dialog.create({
    title: "Xác nhận",
    message: "Bạn có chắc chắn muốn đăng xuẩt ?",
    ok: true,
    cancel: true,
  }).onOk(() => {
    Notify.create({
      type: "positive",
      message: "Đăng xuất thành công!",
      position: "top",
    });
    localStorage.removeItem("username");
    localStorage.removeItem("password");
    router.push("/");
  });
};

onMounted(async () => {
  await fetchData();
});
</script>

<template>
  <q-page class="q-pa-md">
    <div class="flex justify-end">
      <q-btn label="Logout" color="red-9" icon="logout" @click="logout" />
    </div>
    <q-form class="q-gutter-md q-py-lg flex column" @submit="addData">
      <q-input
        v-model="newData.umsatz"
        label="Umsatz"
        :rules="[
          (val) => !!val || 'Field is required',
          (val) => val > 0 && val <= 10000 || 'Value must lower or equal 10000',
        ]"
        outlined
        type="number"
      />
      <q-input
        v-model="newData.notizen"
        label="Notizen"
        :rules="[(val) => !!val || 'Field is required']"
        outlined
      />
      <q-btn
        label="Add Data"
        type="submit"
        color="green"
        icon="add"
        class="q-py-sm"
      />
    </q-form>
    <q-table
      :rows="userData"
      :columns="columns"
      row-key="id"
      :loading="loadingTable"
      :filter="filterTable"
      bordered
      :rows-per-page-options="[20]"
    >
      <template v-slot:top-right>
        <div class="flex flex-center">
          <q-icon
            name="refresh"
            size="md"
            class="q-mr-lg cursor-pointer text-grey-8"
            @click="fetchData"
          />

          <q-input
            borderless
            dense
            debounce="200"
            v-model="filterTable"
            placeholder="Search"
            style="width: 300px"
          >
            <template v-slot:append>
              <q-icon name="search" />
            </template>
          </q-input>
        </div>
      </template>

      <template v-slot:body="props">
        <tr style="width: 100%" :props="props">
          <td class="text-center" key="stt" :props="props" style="width: 15%">
            <span style="font-size: 1.1em"
              ><span style="white-space: pre-wrap">{{
                props.row.datum
              }}</span></span
            >
          </td>

          <td class="text-center" key="stt" :props="props" style="width: 15%">
            <q-badge class="q-pa-sm q-px-lg">
              {{ props.row.umsatz }}
              <q-popup-edit
                title="Umsatz"
                v-model="props.row.umsatz"
                :validate="(value) => value <= 10000 && value >= 0"
                v-slot="scope"
                class="text-blue"
                buttons
                label-set="OK"
                label-cancel="Cancel"
                dense
                @update:model-value="
                  updateData(props.row.id, 'umsatz', props.row.umsatz)
                "
              >
                <q-input
                  dense
                  type="number"
                  v-model="scope.value"
                  autofocus
                  @keyup.enter="scope.set"
                  :rule="(value) => value < 10000 && value >= 0"
                  hint="0 >= value <= 10000 "
                />
              </q-popup-edit>
            </q-badge>
          </td>

          <td class="text-center" key="stt" :props="props" style="width: 15%">
            <q-badge class="q-pa-sm q-px-lg">
              {{ props.row.notizen }}
              <q-popup-edit
                title="notizen"
                v-model="props.row.notizen"
                v-slot="scope"
                class="text-blue"
                buttons
                label-set="OK"
                label-cancel="Cancel"
                dense
                @update:model-value="
                  updateData(props.row.id, 'notizen', props.row.notizen)
                "
              >
                <q-input
                  dense
                  type="text"
                  v-model="scope.value"
                  autofocus
                  @keyup.enter="scope.set"
                  :rule="(value) => value < 200 && value >= 0"
                />
              </q-popup-edit>
            </q-badge>
          </td>

          <td class="text-center" key="stt" :props="props" style="width: 15%">
            <span style="font-size: 1.1em"
              ><span style="white-space: pre-wrap"
                ><q-icon
                  @click="deleteData(props.row.id)"
                  name="delete"
                  size="md"
                  color="red-8" /></span
            ></span>
          </td></tr
      ></template>
    </q-table>
  </q-page>
</template>
