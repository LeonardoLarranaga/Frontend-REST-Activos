<template>
  <v-data-table 
    :headers="headers" 
    :items="activos" 
    :items-per-page="-1" 
    :sort-by="[{ key: 'id', order: 'asc' }]"
    fixed-header
    height="100vh"
    >

    <template v-slot:top>
      <v-toolbar color="indigo-darken-2">
        <v-toolbar-title color="indigo-darken-2">Activos</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>

        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ props }">
            <v-btn prepend-icon="mdi-plus-circle-multiple" variant="tonal" elevation="10" color="white" v-bind="props">
              Nuevo Activo
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <v-icon class="text-h5 margin" style="margin-top: -5px; margin-right: 8px;">{{ this.editedIndex === -1 ? "mdi-plus-circle-multiple" : "mdi-pencil-circle"}} </v-icon>
              <span class="text-h5 font-weight-bold">{{this.editedIndex === -1 ? "Nuevo Activo" : "Editar Activo"}}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" sm="6">
                    <v-text-field v-model="editedItem.id" label="Id"/>
                  </v-col>
                  <v-col cols="12" sm="6">
                    <v-text-field v-model="editedItem.serie" label="Serie"/>
                  </v-col>
                  <v-col cols="12" sm="6">
                    <v-text-field v-model="editedItem.serieUABC" label="Serie UABC"/>
                  </v-col>
                  <v-col cols="12" sm="6">
                    <v-text-field v-model="editedItem.tipo" label="Tipo"/>
                  </v-col>
                  <v-col cols="12">
                    <v-textarea v-model="editedItem.descripcion" label="Descripción" clearable/>
                  </v-col>
                  <v-col cols="12" sm="6">
                    <v-text-field v-model="editedItem.ubicacion" label="Ubicación"/>
                  </v-col>
                  <v-col cols="12" sm="6">
                    <v-text-field v-model="editedItem.responsable" label="Responsable"/>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer/>
              <v-btn 
                color="indigo-darken-2" 
                variant="text" 
                @click="close"
              >
                Cancelar
              </v-btn>
              <v-btn 
                color="indigo-darken-2" 
                variant="tonal" 
                @click="save"
              >
                Guardar
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="350px">
          <v-card>
            <v-card-title>
              <p class="text-h5 font-weight-bold" style="white-space: pre-wrap; text-align: center;">¿Quieres eliminar este activo?</p>
            </v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue-darken-1" variant="text" @click="closeDelete">Cancel</v-btn>
              <v-btn color="blue-darken-1" variant="tonal" @click="deleteItemConfirm">OK</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>

    <template v-slot:item.actions="{ item }">
      <v-icon 
        size="small" 
        @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon size="small" @click="deleteItem(item)"> mdi-delete </v-icon>
    </template>
  </v-data-table>
</template>

<script>
import axios from "axios"

export default {
    data: () => ({
      dialog: false,
      dialogDelete: false,
      headers: [
        {
          title: "Id",
          key: "id",
          sortable: true
        },
        { title: 'Serie', 
          key: 'serie', 
          sortable: false 
        },
        { 
          title: 'Serie UABC', 
          key: 'serieUABC', 
          sortable: false 
        },
        { 
          title: 'Tipo', 
          key: 'tipo', 
          sortable: false 
        },
        {
          title: 'Descripción',
          key: 'descripcion',
          sortable: false
        },
        {
          title: 'Ubicación',
          key: 'ubicacion',
          sortable: true
        },
        {
          title: 'Responsable',
          key: 'responsable',
          sortable: true
        },
        {
          title: "Acciones",
          key: "actions",
          sortable: false
        }
      ],
      activos: [],
      editedIndex: -1,
      editedItem: {
        id: 0,
        serie: "",
        serieUABC: "",
        tipo: "",
        descripcion: "",
        ubicacion: 0,
        responsable: 0,
        imagen: ""
      },
      defaultItem: {
        id: 0,
        serie: "",
        serieUABC: "",
        tipo: "",
        descripcion: "",
        ubicacion: 0,
        responsable: 0,
        imagen: ""
      },
    }),

    watch: {
      dialog(val) {
        val || this.close()
      },
      dialogDelete(val) {
        val || this.closeDelete()
      },
    },

    mounted() {
      this.fetchActivos()
    },

    methods: {
      async fetchActivos() {
        try {
          const response = await axios.get("https://localhost:4000/activo/")
          this.activos = response.data
        } catch (error) {
          console.log(error)
        }
      },

      async createActivo() {
        try {
          const response = await axios.post("https://localhost:4000/activo/", this.editedItem)          
          this.activos.push(response.data.activo)
        } catch (error) {
          console.log(error)
        }
      },

      async editActivo() {
        try {
          const response = await axios.put(`https://localhost:4000/activo/${this.editedItem.id}`, this.editedItem)
          Object.assign(this.activos[this.editedIndex], response.data.activo)
          this.close()
        } catch (error) {
          console.log(error)
        }
      },

      editItem(item) {
        this.editedIndex = this.activos.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem(item) {
        this.editedIndex = this.activos.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialogDelete = true
      },

      async deleteItemConfirm() {
        try {
          const response = await axios.delete(`https://localhost:4000/activo/${this.editedItem.id}`)
          if (response.status == 200) this.activos.splice(this.editedIndex, 1)
        } catch (error) {
          console.log(error)
        }

        this.closeDelete()
      },

      close() {
        this.dialog = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      closeDelete() {
        this.dialogDelete = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      save() {
        if (this.editedIndex > -1) {
          this.editActivo()
        } else {
          this.createActivo()
          this.close()
        }
      },
    },
  }
</script>