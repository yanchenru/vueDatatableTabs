<template>
  <v-dialog v-model="dialog" max-width="800px" max-height="500px">
    <v-card>
      <v-card-title>
        <span class="headline">Scalings</span>
      </v-card-title>
      <v-card-text>
        <b-tabs v-if="scalings.length>0" card>
          <b-tab v-for="i in tabs" :key="i" :title="`Scaling ${i}`">
            <b-button size="sm" class="float-right" @click="() => closeTab(i)">Close tab</b-button>
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="scalings[i].type" label="type"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="scalings[i]['avg.samples']" label="avg.samples"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="scalings[i]['itp.raw_low']" label="itp.raw_low"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="scalings[i]['itp.raw_high']" label="itp.raw_high"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="scalings[i]['itp.scaled_low']" label="itp.scaled_low"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="scalings[i]['itp.scaled_high']" label="itp.scaled_high"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="scalings[i]['itp.offset']" label="itp.offset"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="scalings[i]['kf.factor']" label="kf.factor"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="scalings[i]['kf.multiple']" label="kf.multiple"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field
                    v-model="scalings[i]['ttl.samples_per_sec']"
                    label="ttl.samples_per_sec"
                  ></v-text-field>
                </v-flex>
              </v-layout>
            </v-container>
          </b-tab>

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
      tabs: [],
      tabCounter: 0,
      defaultScaling: {
        type: "INTERPOLATION",
        "avg.samples": 0,
        "itp.raw_low": 0,
        "itp.raw_high": 1,
        "itp.scaled_low": 0,
        "itp.scaled_high": 1,
        "itp.offset": 0,
        "kf.factor": 0,
        "kf.multiple": 0,
        "ttl.samples_per_sec": 0
      },
      scalings: [],
      savedScalingArray: []
    };
  },

  computed: {},

  watch: {
    scalingDialog(val) {
      this.dialog = val;
    },

    dialog(val) {
      val || this.close();
    },

    editedScalingArray(val) {
      this.scalings = JSON.parse(JSON.stringify(val));

      for (let i = 0; i < val.length; i++) {
        this.tabs.push(this.tabCounter++);
      }
    }
  },

  methods: {
    close() {
      this.dialog = false;

      setTimeout(() => {
        this.tabs = [];
        this.tabCounter = 0;
        this.scalings = [];
        this.savedScalingArray = [];
      }, 300);

      this.$emit("closeDialog", this.dialog);
    },

    save() {
      for (let i = 0; i < this.scalings.length; i++) {
        if (this.tabs.includes(i)) {
          this.savedScalingArray.push(this.scalings[i]);
        }
      }

      this.$emit("saveScaling", this.savedScalingArray);

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
      this.scalings.push({ ...this.defaultScaling });
      this.tabs.push(this.tabCounter++);
    }
  }
};
</script>
