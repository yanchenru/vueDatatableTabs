<template>
  <div data-app>
    <v-toolbar flat color="white">
      <v-toolbar-title>CHANNELS</v-toolbar-title>
      <v-divider class="mx-2" inset vertical></v-divider>
      <v-spacer></v-spacer>
      <v-dialog v-model="dialog" max-width="600px" max-height="500px">
        <template v-slot:activator="{ on }">
          <v-btn color="primary" dark class="mb-2" v-on="on">New Channel</v-btn>
        </template>
        <v-card>
          <v-card-title>
            <span class="headline">{{ formTitle }}</span>
          </v-card-title>
          <v-card-text>
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedChannel.ch_id" label="ch_id"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedChannel.internal_name" label="internal_name"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedChannel.display_name" label="display_name"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedChannel.precision" label="precision"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedChannel.unit" label="unit"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedChannel.source_id" label="source_id"></v-text-field>
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
      v-on:saveScaling="saveScaling"
    />

    <v-data-table :headers="headers" :items="channelsArray" class="elevation-1">
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
          <v-icon small class="mr-2" @click="editChannel(props.item)">edit</v-icon>
          <v-icon small @click="deleteChannel(props.item)">delete</v-icon>
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
    editedItem: {},
    editedChannel: {
      ch_id: 0,
      internal_name: "Slot-",
      display_name: "Slot-",
      precision: 2,
      unit: "",
      source_id: 0
    },
    defaultChannel: {
      ch_id: 0,
      internal_name: "Slot-",
      display_name: "Slot-",
      precision: 2,
      unit: "",
      source_id: 0
    },
    scalingDialog: false,
    //editedScalingIndex: -1,
    editedScaling: [],
    channelsArray: []
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Channel" : "Edit Channel";
    },

    newChannelIndex() {
      return this.channelsArray.length;
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
      this.channelsArray = cfgJson.data.channels;
      this.editedChannel.ch_id = this.newChannelIndex;
      this.editedChannel.source_id = this.newChannelIndex;
    },

    editChannel(item) {
      this.editedIndex = this.channelsArray.indexOf(item);
      this.editedChannel = Object.assign({}, item);
      this.dialog = true;
    },

    deleteChannel(item) {
      const index = this.channelsArray.indexOf(item);
      confirm("Are you sure you want to delete this item?") &&
        this.channelsArray.splice(index, 1);
    },

    close() {
      this.dialog = false;
      this.defaultChannel.ch_id = this.newChannelIndex;
      this.defaultChannel.source_id = this.newChannelIndex;
      setTimeout(() => {
        this.editedChannel = Object.assign({}, this.defaultChannel);
        this.editedIndex = -1;
      }, 300);
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.channelsArray[this.editedIndex], this.editedChannel);
      } else {
        this.channelsArray.push(this.editedChannel);
      }
      this.close();
    },

    editScaling(scalings) {
      //this.editedScalingIndex = this.channelsArray.indexOf(scaling);
      this.editedScaling = scalings;
      this.scalingDialog = true;
    },

    closeScaling() {
      this.scalingDialog = false;
    },

    saveScaling(scalings) {
      //this.editedScalingIndex = this.channelsArray.indexOf(scaling);
      this.editedScaling = scalings;
    }
  }
};
</script>