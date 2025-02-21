<script lang="ts" setup>
import { ref, onMounted, watch, computed } from "vue";

// Definición de interfaces
export interface Character {
  info: any;
  results: Result[];
}

export interface Result {
  id: number;
  name: string;
  status: string;
  species: string;
  type: string;
  gender: string;
  image: string;
  episode: string[];
  url: string;
  created: Date;
}

// Obtén los datos de la API (suponiendo que useFetch esté configurado)
const { data } = await useFetch<Character>(
  "https://rickandmortyapi.com/api/character"
);

// Variables reactivas para guardar las listas
const apiCharacters = ref<Result[]>([]);
const localCharacters = ref<Result[]>([]);

// Variable reactiva para el texto de búsqueda
const filterName = ref("");

// Computed para filtrar la copia local de la API
const filteredApiCharacters = computed(() =>
  apiCharacters.value.filter((character) =>
    character.name.toLowerCase().includes(filterName.value.toLowerCase())
  )
);

// Computed para filtrar los personajes creados localmente
const filteredLocalCharacters = computed(() =>
  localCharacters.value.filter((character) =>
    character.name.toLowerCase().includes(filterName.value.toLowerCase())
  )
);

// Al montar el componente, carga la data de la API y de los personajes creados (si existen) desde localStorage
onMounted(() => {
  const storedApi = localStorage.getItem("apiCharacters");
  if (storedApi) {
    apiCharacters.value = JSON.parse(storedApi);
  } else if (data.value && data.value.results) {
    apiCharacters.value = data.value.results;
    localStorage.setItem("apiCharacters", JSON.stringify(apiCharacters.value));
  }

  const storedLocal = localStorage.getItem("localCharacters");
  if (storedLocal) {
    localCharacters.value = JSON.parse(storedLocal);
  }
});

// Actualiza localStorage cada vez que cambie la copia local de la API
watch(
  apiCharacters,
  (newVal) => {
    localStorage.setItem("apiCharacters", JSON.stringify(newVal));
  },
  { deep: true }
);

// Actualiza localStorage para los personajes creados localmente
watch(
  localCharacters,
  (newVal) => {
    localStorage.setItem("localCharacters", JSON.stringify(newVal));
  },
  { deep: true }
);

// Función para eliminar un personaje de la copia local de la API
const removeApiCharacter = (id: number) => {
  apiCharacters.value = apiCharacters.value.filter(
    (character) => character.id !== id
  );
};

// Función para eliminar un personaje creado localmente
const removeLocalCharacter = (id: number) => {
  localCharacters.value = localCharacters.value.filter(
    (character) => character.id !== id
  );
};

// Objeto reactivo para almacenar los datos del formulario de creación
const newCharacterData = ref({
  name: "",
  status: "",
  species: "",
  type: "",
  gender: "",
  image: "",
});

const dialog = ref(false);

const createCharacter = () => {
  const newCharacter: Result = {
    id: Date.now(), // Genera un ID único basado en el timestamp
    name: newCharacterData.value.name,
    status: newCharacterData.value.status,
    species: newCharacterData.value.species,
    type: newCharacterData.value.type,
    gender: newCharacterData.value.gender,
    image: newCharacterData.value.image,
    episode: [],
    url: "",
    created: new Date(),
  };

  // Agrega el nuevo personaje a la lista de personajes creados localmente
  localCharacters.value.push(newCharacter);

  newCharacterData.value = {
    name: "",
    status: "",
    species: "",
    type: "",
    gender: "",
    image: "",
  };
  dialog.value = false;
};

const editDialog = ref(false);

const editingCharacterData = ref({
  id: 0,
  name: "",
  status: "",
  species: "",
  type: "",
  gender: "",
  image: "",
});

const editingId = ref<number | null>(null);

const openEditDialog = (character: Result) => {
  editingCharacterData.value = {
    id: character.id,
    name: character.name,
    status: character.status,
    species: character.species,
    type: character.type,
    gender: character.gender,
    image: character.image,
  };
  editingId.value = character.id;
  editDialog.value = true;
};

const updateCharacter = () => {
  const updatedCharacter: Result = {
    id: editingId.value!,
    name: editingCharacterData.value.name,
    status: editingCharacterData.value.status,
    species: editingCharacterData.value.species,
    gender: editingCharacterData.value.gender,
    image: editingCharacterData.value.image,
    episode: [],
    url: "",
    created: new Date(),
  };

  const indexLocal = localCharacters.value.findIndex(
    (c) => c.id === editingId.value
  );
  if (indexLocal !== -1) {
    localCharacters.value[indexLocal] = updatedCharacter;
  }
  const indexApi = apiCharacters.value.findIndex(
    (c) => c.id === editingId.value
  );
  if (indexApi !== -1) {
    apiCharacters.value[indexApi] = updatedCharacter;
  }

  editDialog.value = false;
};
</script>

<template>
  <v-row class="align-center mr-4" no-gutters>
    <v-col cols="7">
      <v-text-field
        v-model="filterName"
        :loading="loading"
        class="ma-4"
        append-inner-icon="mdi-magnify"
        density="compact"
        label="Search templates"
        variant="solo"
        hide-details
        single-line
        @click:append-inner="onClick"
      />
    </v-col>
    <v-col cols="5" class="d-flex justify-end">
      <v-btn color="green" prepend-icon="mdi-plus" @click="dialog = true">
        Crear
      </v-btn>
    </v-col>
  </v-row>

  <v-container fluid>
    <v-row>
      <v-col v-for="item in filteredApiCharacters" :key="item.id">
        <v-card class="mx-auto" min-width="200" max-width="344">
          <v-img height="200px" :src="item.image" cover></v-img>
          <v-card-title>{{ item.name }}</v-card-title>
          <v-card-subtitle
            >{{ item.species }} - {{ item.status }}</v-card-subtitle
          >
          <v-card-actions>
            <v-btn
              color="blue"
              @click="openEditDialog(item)"
              style="
                background-color: transparent;
                border: 1px solid blue;
                border-radius: 4px;
                padding: 8px 16px;
              "
            >
              <v-icon left>mdi-pencil</v-icon>
              Editar
            </v-btn>

            <v-btn
              icon
              color="red"
              @click="removeApiCharacter(item.id)"
              style="
                background-color: transparent;
                border: 1px solid red;
                border-radius: 50%;
                padding: 8px;
              "
            >
              <v-icon>mdi-delete</v-icon>
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>

  <v-container fluid>
    <v-row>
      <v-col v-for="item in filteredLocalCharacters" :key="item.id">
        <v-card class="mx-auto" max-width="344">
          <v-img height="200px" :src="item.image" cover></v-img>
          <v-card-title>{{ item.name }}</v-card-title>
          <v-card-subtitle
            >{{ item.species }} - {{ item.status }}</v-card-subtitle
          >
          <v-card-actions>
            <v-btn
              color="blue"
              @click="openEditDialog(item)"
              style="
                background-color: transparent;
                border: 1px solid blue;
                border-radius: 4px;
                padding: 8px 16px;
              "
            >
              <v-icon left>mdi-pencil</v-icon>
              Editar
            </v-btn>
            <v-btn
              icon
              color="red"
              @click="removeLocalCharacter(item.id)"
              style="
                background-color: transparent;
                border: 1px solid red;
                border-radius: 50%;
                padding: 8px;
              "
            >
              <v-icon>mdi-delete</v-icon>
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>

  <v-dialog v-model="dialog" max-width="500px">
    <v-card>
      <v-card-title>
        <span class="text-h5">Crear Personaje</span>
      </v-card-title>
      <v-card-text>
        <v-form>
          <v-text-field
            label="Nombre"
            v-model="newCharacterData.name"
            required
          ></v-text-field>
          <v-text-field
            label="Estado"
            v-model="newCharacterData.status"
          ></v-text-field>
          <v-text-field
            label="Especie"
            v-model="newCharacterData.species"
          ></v-text-field>
          <v-text-field
            label="Género"
            v-model="newCharacterData.gender"
          ></v-text-field>
          <v-text-field
            label="URL de la imagen"
            v-model="newCharacterData.image"
          ></v-text-field>
        </v-form>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn text @click="dialog = false">Cancelar</v-btn>
        <v-btn color="primary" @click="createCharacter">Guardar</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>

  <v-dialog v-model="editDialog" max-width="500px">
    <v-card>
      <v-card-title>
        <span class="text-h5">Editar Personaje</span>
      </v-card-title>
      <v-card-text>
        <v-form>
          <v-text-field
            label="Nombre"
            v-model="editingCharacterData.name"
            required
          ></v-text-field>
          <v-text-field
            label="Estado"
            v-model="editingCharacterData.status"
          ></v-text-field>
          <v-text-field
            label="Especie"
            v-model="editingCharacterData.species"
          ></v-text-field>
          <v-text-field
            label="Género"
            v-model="editingCharacterData.gender"
          ></v-text-field>
          <v-text-field
            label="URL de la imagen"
            v-model="editingCharacterData.image"
          ></v-text-field>
        </v-form>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn text @click="editDialog = false">Cancelar</v-btn>
        <v-btn color="primary" @click="updateCharacter">Guardar Cambios</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>
