<script setup>
import { ref, onMounted } from "vue";
import { db } from "../firebase";
import {
  collection,
  query,
  getDocs,
  getDoc,
  doc
} from "firebase/firestore";
import { useQuasar } from "quasar";

const $q = useQuasar();

const filter = ref("");

const columns = [
  {
    name: "IdHero",
    required: true,
    label: "IdHero",
    align: "left",
    field: "IdHero",
    sortable: true,
  },
  {
    name: "NomHero",
    align: "center",
    label: "NomHero",
    field: "NomHero",
    sortable: true,
  },
  {
    name: "PoderHero",
    align: "center",
    label: "PoderHero",
    field: "PoderHero",
    sortable: true,
  },
  { name: "DescHero", align: "center", label: "DescHero", field: "DescHero" },
];

const rows = ref([]);

const loading = ref(true);

async function getHero() {
  try {
    const q = query(collection(db, "superheroes"));

    const querySnapshot = await getDocs(q);
    querySnapshot.forEach((doc) => {
      // console.log(doc.id, " => ", doc.data());

      const HeroData = {
        IdHero: doc.id,
        NomHero: doc.data().nombre,
        PoderHero: doc.data().poder,
        DescHero: doc.data().descripcion,
      };

      rows.value.push(HeroData);
    });
  } catch (error) {
    console.log(error);
  } finally {
    loading.value = false;
  }
}

onMounted(() => {
  getHero();
});

// Resetear tabla
function reset() {
  rows.value = [];

  getHero();
}

// Obtener heroe por id
const dialogfilt = ref(false);
const IdHero = ref("");

const HerobyId = async () => {
  if (IdHero.value.trim() !== "") {
    try {
      const docRef = doc(db, "superheroes", IdHero.value.trim());
      const docSnap = await getDoc(docRef);

      if (docSnap.exists()) {
        // console.log("Document data:", docSnap.data());
        loading.value = true;
        rows.value = [];


        const HeroData = {
          IdHero: docSnap.id,
          NomHero: docSnap.data().nombre,
          PoderHero: docSnap.data().poder,
          DescHero: docSnap.data().descripcion,
        };

        rows.value.push(HeroData);

      } else {
        // doc.data() will be undefined in this case
        $q.notify({
          message: "No se encontro ningún superhéroe",
          color: "negative",
          icon: "error",
        });

        IdHero.value = '';
      }

    } catch (error) {
      console.log(error);
    } finally {
      loading.value = false;
    }
  } else {
    $q.notify({
      message: "Digite una busqueda valida",
      color: "negative",
      icon: "error",
    });
  }
};

// Obtener heroe por parametro
const dialogparam = ref(false);
const HeroParam = ref("");

const HerobyParam = async () => {
  if (HeroParam.value.trim() !== "") {
    try {
      const rowBackup = [];
      const q = query(collection(db, "superheroes"));

      const querySnapshot = await getDocs(q);
      querySnapshot.forEach((doc) => {
        // console.log(doc.id, " => ", doc.data());

        if (doc.data().descripcion.toLowerCase().includes(HeroParam.value.trim().toLowerCase())){
          const HeroData = {
            IdHero: doc.id,
            NomHero: doc.data().nombre,
            PoderHero: doc.data().poder,
            DescHero: doc.data().descripcion,
          };

          rowBackup.push(HeroData);
        }
      });

      if (rowBackup.length !== 0){

        loading.value = true;
        rows.value = [];


        for (let i = 0; i <= rowBackup.length - 1; i++) {
          console.log(rowBackup[i]);
          rows.value.push(rowBackup[i]);
        }

        dialogparam.value = false;
        HeroParam.value = '';

      } else {
        $q.notify({
          message: "No se encontro ningún superhéroe",
          color: "negative",
          icon: "error",
        });

        HeroParam.value = '';
      }
    } catch (error) {
      console.log(error);
    } finally {
      loading.value = false;
    }
  } else {
    $q.notify({
      message: "Digite una busqueda valida",
      color: "negative",
      icon: "error",
    });
  }
};
</script>

<template>
  <q-page>
    <div class="q-pa-xl">
      <q-table
        title="Superhéroes"
        :loading="loading"
        :rows="rows"
        :columns="columns"
        :filter="filter"
        :row-key="rows.IdHero"
      >
        <template v-slot:top-left>
          <q-btn
            color="primary"
            :disable="loading"
            label="ID"
            @click="dialogfilt = true"
          />
          <q-btn
            class="q-ml-sm"
            color="primary"
            :disable="loading"
            label="Parámetro"
            @click="dialogparam = true"
          />
          <q-btn
            class="q-ml-sm q-mr-xl"
            color="primary"
            :disable="loading"
            label="Reset"
            @click="reset"
          />
        </template>

        <q-space />

        <template v-slot:top-right>
          <q-input
            borderless
            dense
            debounce="300"
            v-model="filter"
            placeholder="Buscar"
          >
            <template v-slot:append>
              <q-icon name="search" />
            </template>
          </q-input>
        </template>
      </q-table>
    </div>

    <q-dialog v-model="dialogfilt">
      <q-card style="width: 400px; max-width: 80vw">
        <q-toolbar>
          <q-toolbar-title class="text-overline">
            <span class="text-weight-bold text-primary">Buscar por ID </span>
            <span class="text-weight-bold">Super-Hero</span>
          </q-toolbar-title>
          <q-icon name="search" color="primary" size="sm"></q-icon>

          <!-- <q-btn flat round dense icon="close" v-close-popup /> -->
        </q-toolbar>

        <q-card-section>
          <q-form @submit.prevent="HerobyId">
            <q-input dense outlined v-model="IdHero" />

            <q-btn
              type="submit"
              class="q-mt-md full-width"
              no-caps
              color="primary"
              label="Buscar"
              dense
            ></q-btn>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>

    <q-dialog v-model="dialogparam">
      <q-card style="width: 400px; max-width: 80vw">
        <q-toolbar>
          <q-toolbar-title class="text-overline">
            <span class="text-weight-bold text-primary"
              >Buscar por Parametro
            </span>
            <span class="text-weight-bold">Super-Hero</span>
          </q-toolbar-title>
          <q-icon name="search" color="primary" size="sm"></q-icon>

          <!-- <q-btn flat round dense icon="close" v-close-popup /> -->
        </q-toolbar>

        <q-card-section>
          <q-form @submit.prevent="HerobyParam">
            <q-input dense outlined v-model="HeroParam" />

            <q-btn
              type="submit"
              class="q-mt-md full-width"
              no-caps
              color="primary"
              label="Buscar"
              dense
            ></q-btn>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>
