<template>
  <v-dialog v-model="dialog" max-width="800px" max-height="500px">
    <v-card>
      <v-card-title>
        <span class="headline">Scalings</span>
      </v-card-title>
      <v-card-text>
        <b-tabs card>
          <b-tab v-for="i in tabs" :key="i" :title="`Scaling ${i}`">
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedScalingArray[i].type" label="type"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedScalingArray[i]['avg.samples']" label="avg.samples"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedScalingArray[i]['itp.raw_low']" label="itp.raw_low"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedScalingArray[i]['itp.raw_high']" label="itp.raw_high"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedScalingArray[i]['itp.scaled_low']" label="itp.scaled_low"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedScalingArray[i]['itp.scaled_high']" label="itp.scaled_high"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedScalingArray[i]['itp.offset']" label="itp.offset"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedScalingArray[i]['kf.factor']" label="kf.factor"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedScalingArray[i]['kf.multiple']" label="kf.multiple"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedScalingArray[i]['ttl.samples_per_sec']" label="ttl.samples_per_sec"></v-text-field>
                </v-flex>
              </v-layout>
            </v-container>

            <b-button
              size="sm"
              variant="danger"
              class="float-right"
              @click="() => closeTab(i)"
            >Close tab</b-button>
          </b-tab>

          <!-- New Tab Button (Using tabs slot) -->
          <template slot="tabs">
            <b-nav-item @click.prevent="newTab" href="#">
              <b>+</b>
            </b-nav-item>
          </template>

          <!-- Render this if no tabs -->
          <div slot="empty" class="text-center text-muted">
            There are no open tabs
            <br />Open a new tab using the
            <b>+</b> button above.
          </div>
        </b-tabs>
      </v-card-text>

      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="blue darken-1" flat @click="close">Cancel</v-btn>
        <!-- <v-btn color="blue darken-1" flat @click="removeTab">Remove</v-btn>
        <v-btn color="blue darken-1" flat @click="newTab">Add</v-btn>-->
        <v-btn color="blue darken-1" flat @click="save">Save</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  props: ["scalingDialog", "editedScalingArray"],

  data() {
    return {
      dialog: this.scalingDialog,
      //length: this.editedScaling.length,
      tabs: [],
      //tab: null,
      tabCounter: 0,
      scalings: [],
      editedScaling: {
        name: "",
        calories: 0,
        fat: 0,
        carbs: 0,
        protein: 0
      },
      defaultScaling: {
        type: "INTERPOLATION",
        calories: 0,
        fat: 0,
        carbs: 0,
        protein: 0
      },
      editedIndex: -1
    };
  },

  computed: {
  },

  watch: {
    scalingDialog(val) {
      this.dialog = val;

      if (val === true) {
        this.scalings = this.editedScalingArray;

        for (let i = 0; i < this.editedScalingArray.length; i++) {
          this.newTab();
        }
      }
    },
    dialog(val) {
      val || this.close();
    }
    // editedScaling(val) {
    //   this.length = val.length;
    //   this.scalings = val;

    //   for (let i = 0; i < val.length; i++) {
    //     this.tabs.push(this.tabCounter++);
    //   }
    // }
  },

  methods: {
    close() {
      //this.dialog = false;
      setTimeout(() => {
        this.tabs = [];
        this.tabCounter = 0;
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      }, 300);

      this.$emit("closeDialog", this.dialog);

      //this.editedScaling = []
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.cfgJsonData[this.editedIndex], this.editedItem);
      } else {
        this.cfgJsonData.push(this.editedItem);
      }
      this.close();
    },

    closeTab(x) {
      for (let i = 0; i < this.tabs.length; i++) {
        if (this.tabs[i] === x) {
          this.tabs.splice(i, 1);
        }
      }
    },

    newTab() {
      this.tabs.push(this.tabCounter++);
    }
  }
};
</script>
