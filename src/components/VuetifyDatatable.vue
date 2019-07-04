<template>
  <div data-app>
    <v-toolbar flat color="white">
      <v-toolbar-title>My CRUD</v-toolbar-title>
      <v-divider class="mx-2" inset vertical></v-divider>
      <v-spacer></v-spacer>
      <v-dialog v-model="dialog" max-width="500px" max-height="500px">
        <template v-slot:activator="{ on }">
          <v-btn color="primary" dark class="mb-2" v-on="on">New Item</v-btn>
        </template>
        <v-card>
          <v-card-title>
            <span class="headline">{{ formTitle }}</span>
          </v-card-title>
          <v-card-text>
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedItem.internal_name" label="internal_name"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedItem.display_name" label="display_name"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedItem.precision" label="precision"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedItem.carbs" label="unit"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedItem.protein" label="source_id"></v-text-field>
                </v-flex>
              </v-layout>
            </v-container>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" flat @click="close">Cancel</v-btn>
            <v-btn color="blue darken-1" flat @click="save">Save</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-toolbar>

    <Scaling
      :scalingDialog="scalingDialog"
      :editedScalingArray="editedScaling"
      v-on:closeDialog="closeScaling"
    />

    <v-data-table :headers="headers" :items="cfgJsonData" class="elevation-1">
      <template v-slot:items="props">
        <td class="text-xs-left">{{ props.item.ch_id }}</td>
        <td class="text-xs-left">{{ props.item.internal_name }}</td>
        <td class="text-xs-left">{{ props.item.display_name }}</td>
        <td class="text-xs-left">{{ props.item.precision }}</td>
        <td class="text-xs-left">{{ props.item.unit }}</td>
        <td class="text-xs-left">{{ props.item.source_id }}</td>
        <td class="text-xs-left">
          <v-icon small class="mr-2" @click="editScaling(props.item.scalings)">list</v-icon>
        </td>
        <td class="justify-center layout px-0">
          <v-icon small class="mr-2" @click="editItem(props.item)">edit</v-icon>
          <v-icon small @click="deleteItem(props.item)">delete</v-icon>
        </td>
      </template>
      <template v-slot:no-data>
        <v-btn color="primary" @click="initialize">Reset</v-btn>
      </template>
    </v-data-table>
  </div>
</template>

<script>
import cfgJson from "./cfg.json";
import Scaling from "./Scaling.vue";
// import axios from 'axios';

// export default {
//   name: "VuetifyDatatable",
//   data() {
//     return {
//       info: null
//     };
//   },
//   mounted() {
//     axios
//       .get("https://api.coindesk.com/v1/bpi/currentprice.json")
//       .then(response => (this.info = response));
//   }
// };
export default {
  name: "VuetifyDatatable",
  components: {
    Scaling
  },
  data: () => ({
    dialog: false,
    scalingDialog: false,
    headers: [
      {
        text: "ch_id",
        //align: "left",
        value: "name"
      },
      { text: "internal_name", value: "internal_name" },
      { text: "display_name", value: "display_name" },
      { text: "precision", value: "precision" },
      { text: "unit", value: "unit" },
      { text: "source_id", value: "source_id" },
      { text: "scalings", value: "scalings" },
      { text: "Actions", align: "center", value: "name", sortable: false }
    ],
    editedIndex: -1,
    editedItem: {
      name: "",
      calories: 0,
      fat: 0,
      carbs: 0,
      protein: 0
    },
    defaultItem: {
      name: "",
      calories: 0,
      fat: 0,
      carbs: 0,
      protein: 0
    },
    editedScalingIndex: -1,
    editedScaling: [],
    cfgJsonData: []
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    }
  },

  watch: {
    dialog(val) {
      val || this.close();
    }
  },

  created() {
    this.initialize();
  },

  methods: {
    initialize() {
      this.cfgJsonData = cfgJson.data.channels;
      //this.cfgJsonData = cfgJson;
    },

    editItem(item) {
      this.editedIndex = this.cfgJsonData.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      const index = this.cfgJsonData.indexOf(item);
      confirm("Are you sure you want to delete this item?") &&
        this.cfgJsonData.splice(index, 1);
    },

    editScaling(scalings) {
      //this.editedScalingIndex = this.cfgJsonData.indexOf(scaling);
      this.editedScaling = scalings;
      this.scalingDialog = true;
    },

    close() {
      this.dialog = false;
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      }, 300);
    },

    closeScaling() {
      this.scalingDialog = false;
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.cfgJsonData[this.editedIndex], this.editedItem);
      } else {
        this.cfgJsonData.push(this.editedItem);
      }
      this.close();
    }
  }
};
</script>