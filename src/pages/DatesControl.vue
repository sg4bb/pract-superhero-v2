<script setup>
import { ref, onMounted } from "vue";
import { db } from "../firebase";
import {
  collection,
  addDoc,
  doc,
  getDocs,
  serverTimestamp,
  query,
  Timestamp,
} from "firebase/firestore";
import { useQuasar } from "quasar";

const loading = ref(true);

const columns = [
  {
    name: "id",
    required: true,
    label: "Identificacion",
    align: "left",
    field: "id",
    sortable: true,
  },
  {
    name: "nombre",
    align: "center",
    label: "Nombre",
    field: "nombre",
    sortable: true,
  },
  {
    name: "color",
    align: "center",
    label: "Color",
    field: "color",
    sortable: true,
  },
  {
    name: "fecha",
    align: "center",
    label: "Fecha agregada",
    field: "fecha",
    sortable: true,
  },
];

const rows = ref([]);

const getItem = async () => {
  try {
    const q = query(collection(db, "tienda"));

    const querySnapshot = await getDocs(q);
    querySnapshot.forEach((doc) => {
      // console.log(doc.id, " => ", doc.data());
      // console.log(doc.data().fecha.toDate());

      const Item = {
        id: doc.id,
        nombre: doc.data().nombre,
        color: doc.data().color,
        fecha: doc.data().fecha.toDate("2021-01-01").toLocaleDateString(),
      };

      rows.value.push(Item);
    });
  } catch (error) {
    console.log(error);
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  getItem();
});

const $q = useQuasar();

const nombreItem = ref("");
const colorItem = ref("");

const addItem = async () => {
  if (nombreItem.value.trim() !== "" && colorItem.value.trim() !== "") {
    try {
      const docRef = await addDoc(collection(db, "tienda"), {
        nombre: nombreItem.value,
        color: colorItem.value,
        fecha: Timestamp.fromDate(new Date()),
      });
      console.log("Document written with ID: ", docRef.id);

      $q.notify({
        message: "Item agregado correctamente con el ID " + docRef.id,
        color: "positive",
        icon: "cloud_done",
      });

      nombreItem.value = "";
      colorItem.value = "";

      rows.value = [];
      getItem();
    } catch (error) {
      console.log(error);
    }
  } else {
    $q.notify({
      message: "Agrega datos validos",
      color: "negative",
      icon: "error",
    });
  }
};

const desdeFecha = ref("");
const hastaFecha = ref("");

const filtItem = async () => {
  console.log(Date.parse(desdeFecha.value));
  console.log(Date.parse(hastaFecha.value));

  try {
    rows.value = [];

    loading.value = true;

    const q = query(collection(db, "tienda"));

    const querySnapshot = await getDocs(q);
    querySnapshot.forEach((doc) => {
      // console.log(doc.id, " => ", doc.data());
      // console.log(doc.data().fecha.toDate());

      var ItemTimestamp = doc.data().fecha.toDate("2021-01-01").getTime();

      const Item = {
        id: doc.id,
        nombre: doc.data().nombre,
        color: doc.data().color,
        fecha: doc.data().fecha.toDate("2021-01-01").toLocaleDateString(),
      };


      if (
        ItemTimestamp >= Date.parse(desdeFecha.value) &&
        ItemTimestamp <= Date.parse(hastaFecha.value)
      ) {
        rows.value.push(Item)
      }
    });
  } catch (error) {
    console.log(error);
  } finally {
    loading.value = false;
  }
};
</script>

<template>
  <q-page>
    <div class="row q-col-gutter-md q-pa-xl justify-center">
      <div class="col-12 col-md-6">
        <q-card>
          <q-card-section class="bg-primary text-white row">
            <div>
              <div class="text-h6 text-weight-bolder">Registra un Item</div>
              <div class="text-subtitle2">Completa los campos</div>
            </div>

            <div class="absolute-right">
              <q-icon
                name="add_circle"
                class="q-pa-lg"
                on-right
                size="md"
              ></q-icon>
            </div>
          </q-card-section>

          <q-separator />

          <q-card-section class="q-pb-lg">
            <q-form class="q-gutter-lg" @submit.prevent="addItem">
              <q-input dense type="text" v-model="nombreItem" label="Nombre" />

              <q-input type="text" dense v-model="colorItem" label="Color" />

              <q-card-actions align="right">
                <q-btn flat type="submit" color="primary"> Ingresar </q-btn>
              </q-card-actions>
            </q-form>
          </q-card-section>
        </q-card>
      </div>

      <div class="col-12 col-md-6">
        <q-card class="dates-card">
          <q-card-section class="text-primary text-center">
            <div class="text-h6 text-weight-bolder">Fechas</div>
            <div class="text-subtitle2"></div>
          </q-card-section>

          <q-separator inset />

          <q-card-section>
            <q-form @submit.prevent="filtItem">
              <q-input
                filled
                v-model="desdeFecha"
                label="Desde"
                :rules="['YYYY-MM-DD HH:mm']"
              >
                <template v-slot:prepend>
                  <q-icon name="event" class="cursor-pointer q-mr-sm">
                    <q-popup-proxy
                      cover
                      transition-show="scale"
                      transition-hide="scale"
                    >
                      <q-date v-model="desdeFecha" mask="YYYY-MM-DD HH:mm">
                        <div class="row items-center justify-end">
                          <q-btn
                            v-close-popup
                            label="Close"
                            color="primary"
                            flat
                          />
                        </div>
                      </q-date>
                    </q-popup-proxy>
                  </q-icon>

                  <q-icon name="timer" class="cursor-pointer">
                    <q-popup-proxy
                      cover
                      transition-show="scale"
                      transition-hide="scale"
                    >
                      <q-time v-model="desdeFecha" mask="YYYY-MM-DD HH:mm">
                        <div class="row items-center justify-end">
                          <q-btn
                            v-close-popup
                            label="Close"
                            color="primary"
                            flat
                          />
                        </div>
                      </q-time>
                    </q-popup-proxy>
                  </q-icon>
                </template>
              </q-input>

              <q-input
                filled
                v-model="hastaFecha"
                label="Hasta"
                :rules="['YYYY-MM-DD HH:mm']"
              >
              <template v-slot:prepend>
                  <q-icon name="event" class="cursor-pointer q-mr-sm">
                    <q-popup-proxy
                      cover
                      transition-show="scale"
                      transition-hide="scale"
                    >
                      <q-date v-model="hastaFecha" mask="YYYY-MM-DD HH:mm">
                        <div class="row items-center justify-end">
                          <q-btn
                            v-close-popup
                            label="Close"
                            color="primary"
                            flat
                          />
                        </div>
                      </q-date>
                    </q-popup-proxy>
                  </q-icon>

                  <q-icon name="timer" class="cursor-pointer">
                    <q-popup-proxy
                      cover
                      transition-show="scale"
                      transition-hide="scale"
                    >
                      <q-time v-model="hastaFecha" mask="YYYY-MM-DD HH:mm">
                        <div class="row items-center justify-end">
                          <q-btn
                            v-close-popup
                            label="Close"
                            color="primary"
                            flat
                          />
                        </div>
                      </q-time>
                    </q-popup-proxy>
                  </q-icon>
                </template>
              </q-input>

              <q-card-actions align="right">
                <q-btn flat type="submit" color="primary"> Buscar </q-btn>
              </q-card-actions>
            </q-form>
          </q-card-section>

          <q-separator />
        </q-card>
      </div>

      <div class="col-12 col-md-12 q-py-md">
        <q-table
          class=""
          title="Tienda"
          :loading="loading"
          color="primary"
          :rows="rows"
          :columns="columns"
          :row-key="rows.id"
        />
      </div>
    </div>
  </q-page>
</template>

<style scoped>
.dates-card {
  height: calc(100% + 1px);
}
</style>
