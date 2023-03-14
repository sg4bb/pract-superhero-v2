<script setup>
import { ref, onMounted } from "vue";
import { db } from "../firebase";
import {
  collection,
  query,
  getDocs,
  getDoc,
  doc,
  addDoc,
  deleteDoc,
  updateDoc
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

const IdHero = ref("");
const nomHero = ref("");
const descHero = ref("");
const poderHero = ref("");

const dialogaddhero = ref(false);

const AddHero = async () => {
  if (
    nomHero.value.trim() !== "" &&
    descHero.value.trim() !== "" &&
    poderHero.value.trim() !== ""
  ) {
    try {
      const docRef = await addDoc(collection(db, "superheroes"), {
        nombre: nomHero.value,
        poder: poderHero.value,
        descripcion: descHero.value,
      });

      console.log("Document written with ID: ", docRef.id);

      $q.notify({
        message: "Superhéroe agregado correctamente",
        color: "positive",
        icon: "cloud_done",
      });

      dialogaddhero.value = false;

      nomHero.value = "";
      poderHero.value = "";
      descHero.value = "";

      rows.value = [];
      getHero();
    } catch (error) {
      console.log(error);
    }
  } else {
    $q.notify({
      message: "Digite una busqueda valida",
      color: "negative",
      icon: "error",
    });
  }
};

const deleteHero = async (Id) => {
  try {
    await deleteDoc(doc(db, "superheroes", Id));

      $q.notify({
        message: "Superhéroe borrado correctamente",
        color: "positive",
        icon: "cloud_done",
      });

      rows.value = [];
      getHero();
  } catch (error) {
    console.log(error);
  }
};

const acteditdialog = async (Id) => {
  dialogedithero.value = true;

  try {

    const docRef = doc(db, "superheroes", Id);
    const docSnap = await getDoc(docRef);

    if (docSnap.exists()) {
      // console.log("Document data:", docSnap.data());

      IdHero.value = docSnap.id;
      nomHero.value = docSnap.data().nombre;
      poderHero.value = docSnap.data().poder;
      descHero.value = docSnap.data().descripcion;

    } else {
      // doc.data() will be undefined in this case
      console.log("No such document!");
    }

  } catch (error) {
    console.log(error);
  }
}

const dialogedithero = ref(false);

const EditHero = async () => {
  if (
    nomHero.value.trim() !== "" &&
    descHero.value.trim() !== "" &&
    poderHero.value.trim() !== ""
  ) {

    try {

      const docRef = doc(db, "superheroes", IdHero.value);
      // Set the "capital" field of the city 'DC'
      await updateDoc(docRef, {
        nombre: nomHero.value.trim(),
        poder: poderHero.value.trim(),
        descripcion: descHero.value.trim()
      });

      dialogedithero.value = false;

      $q.notify({
        message: "Superhéroe actualizado correctamente",
        color: "positive",
        icon: "cloud_done",
      });

      nomHero.value = "";
      poderHero.value = "";
      descHero.value = "";

      rows.value = [];
      getHero();

    } catch (error) {
      console.log(error);
    }
  } else {
    $q.notify({
      message: "Digite una busqueda valida",
      color: "negative",
      icon: "error",
    });
  }
}

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
            label="Agregar"
            @click="dialogaddhero = true"
          />
        </template>

        <q-space />

        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th auto-width />
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td auto-width>
              <q-btn
                size="sm"
                square
                dense
                class="q-mb-sm"
                color="primary"
                @click="deleteHero(props.row.IdHero)"
                icon="delete"
              />
              <q-btn
                size="sm"
                square
                dense
                class="q-ml-sm q-mb-sm"
                color="primary"
                @click="acteditdialog(props.row.IdHero)"
                icon="edit"
              />
            </q-td>
            <q-td v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.value }}
            </q-td>
          </q-tr>
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

    <q-dialog v-model="dialogaddhero">
      <q-card style="width: 400px; max-width: 80vw">
        <q-toolbar>
          <q-toolbar-title class="text-overline">
            <span class="text-weight-bold text-primary">Agregar </span>
            <span class="text-weight-bold">Super-Hero</span>
          </q-toolbar-title>
          <q-icon name="search" color="primary" size="sm"></q-icon>

          <!-- <q-btn flat round dense icon="close" v-close-popup /> -->
        </q-toolbar>

        <q-card-section>
          <q-form @submit.prevent="AddHero">
            <q-input
              dense
              outlined
              v-model="nomHero"
              label="Nombre del superhéroe"
            />

            <q-input
              dense
              outlined
              v-model="poderHero"
              class="q-mt-md"
              label="Poder del superhéroe"
            />

            <q-input
              dense
              outlined
              v-model="descHero"
              class="q-mt-md"
              type="textarea"
              label="Descripción del superhéroe"
            />

            <q-btn
              type="submit"
              class="q-mt-md full-width"
              no-caps
              color="primary"
              label="Ingresar"
              dense
            ></q-btn>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>


    <q-dialog v-model="dialogedithero">
      <q-card style="width: 400px; max-width: 80vw">
        <q-toolbar>
          <q-toolbar-title class="text-overline">
            <span class="text-weight-bold text-primary">Editar </span>
            <span class="text-weight-bold">Super-Hero</span>
          </q-toolbar-title>
          <q-icon name="search" color="primary" size="sm"></q-icon>

          <!-- <q-btn flat round dense icon="close" v-close-popup /> -->
        </q-toolbar>

        <q-card-section>
          <q-form @submit.prevent="EditHero">
            <q-input
              dense
              outlined
              v-model="nomHero"
              label="Nombre del superhéroe"
            />

            <q-input
              dense
              outlined
              v-model="poderHero"
              class="q-mt-md"
              label="Poder del superhéroe"
            />

            <q-input
              dense
              outlined
              v-model="descHero"
              class="q-mt-md"
              type="textarea"
              label="Descripción del superhéroe"
            />

            <q-btn
              type="submit"
              class="q-mt-md full-width"
              no-caps
              color="primary"
              label="Ingresar"
              dense
            ></q-btn>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>

