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
            <v-form ref="form" v-model="valid">
              <v-container grid-list-md>
                <v-layout wrap>
                  <v-flex xs12 sm6 md4>
                    <v-text-field
                      v-model="editedChannel.ch_id"
                      label="Channel Id"
                      :rules="[rules.required, rules.positiveInteger]"
                      type="number"
                      min="0"
                      :readonly="editReadonly"
                    ></v-text-field>
                  </v-flex>
                  <v-flex xs12 sm6 md4>
                    <v-text-field
                      v-model="editedChannel.internal_name"
                      label="Internal Name"
                      :rules="[rules.required]"
                      :readonly="editReadonly"
                    ></v-text-field>
                  </v-flex>
                  <v-flex xs12 sm6 md4>
                    <v-text-field
                      v-model="editedChannel.display_name"
                      label="Display Name"
                      :rules="[rules.required]"
                    ></v-text-field>
                  </v-flex>
                  <v-flex xs12 sm6 md4>
                    <v-text-field
                      v-model="editedChannel.precision"
                      label="Precision"
                      :rules="[rules.required, rules.positiveInteger]"
                      type="number"
                      min="0"
                    ></v-text-field>
                  </v-flex>
                  <v-flex xs12 sm6 md4>
                    <v-text-field v-model="editedChannel.unit" label="Unit"></v-text-field>
                  </v-flex>
                  <v-flex xs12 sm6 md4>
                    <v-text-field
                      v-model="editedChannel.source_id"
                      label="Source Id"
                      :rules="[rules.required, rules.positiveInteger]"
                      type="number"
                      min="0"
                    ></v-text-field>
                  </v-flex>
                </v-layout>
              </v-container>
            </v-form>
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
          <v-icon small class="mr-2" @click="editScaling(props.item)">list</v-icon>
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
  data() {
    return {
      valid: true,
      rules: {
        required: value => value===0 || !!value || "Required.",
        positiveInteger: value => /^\d+$/.test(value) || "Non-negative integer only"
      },
      dialog: false,
      headers: [
        {
          text: "Channel Id",
          value: "name"
        },
        { text: "Internal Name", value: "internal_name" },
        { text: "Display Name", value: "display_name" },
        { text: "Precision", value: "precision" },
        { text: "Unit", value: "unit" },
        { text: "Source Id", value: "source_id" },
        { text: "Scalings", value: "scalings" },
        { text: "Actions", align: "center", value: "name", sortable: false }
      ],
      editedChannelIndex: -1,
      editedChannel: {
        ch_id: 0,
        internal_name: "Slot-",
        display_name: "Slot-",
        precision: 2,
        unit: "",
        source_id: 0,
        scalings: []
      },
      defaultChannel: {
        ch_id: 0,
        internal_name: "Slot-",
        display_name: "Slot-",
        precision: 2,
        unit: "",
        source_id: 0,
        scalings: []
      },
      scalingDialog: false,
      editedScaling: [],
      channelsArray: [],
      editReadonly: false
    };
  },

  computed: {
    formTitle() {
      return this.editedChannelIndex === -1 ? "New Channel" : "Edit Channel";
    },

    newChannelIndex() {
      return (
        this.channelsArray.reduce(
          (max, p) => (p.ch_id > max ? p.ch_id : max),
          this.channelsArray[0].ch_id
        ) + 1
      );
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
      this.editReadonly = true;
      this.editedChannelIndex = this.channelsArray.indexOf(item);
      this.editedChannel = Object.assign({}, item);
      this.dialog = true;
    },

    deleteChannel(item) {
      const index = this.channelsArray.indexOf(item);
      confirm("Are you sure you want to delete this item?") &&
        this.channelsArray.splice(index, 1);
    },

    close() {
      this.editReadonly = false;
      this.dialog = false;
      this.defaultChannel.ch_id = this.newChannelIndex;
      this.defaultChannel.source_id = this.newChannelIndex;
      setTimeout(() => {
        this.editedChannel = Object.assign({}, this.defaultChannel);
        this.editedChannelIndex = -1;
      }, 300);
    },

    save() {
      if (this.$refs.form.validate()) {
        if (this.editedChannelIndex > -1) {
          Object.assign(
            this.channelsArray[this.editedChannelIndex],
            this.editedChannel
          );
        } else {
          this.channelsArray.push(this.editedChannel);
        }
        this.close();
      }
    },

    editScaling(item) {
      this.editedChannelIndex = this.channelsArray.indexOf(item);
      this.editedScaling = item.scalings;
      this.scalingDialog = true;
    },

    closeScaling() {
      this.scalingDialog = false;
      this.editedScaling = [];
    },

    saveScaling(scalings) {
      this.channelsArray[this.editedChannelIndex].scalings = scalings;
      this.editedScaling = scalings;
    }
  }
};
</script>