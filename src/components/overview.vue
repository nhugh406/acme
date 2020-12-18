<template>
  <v-card>
    <v-card-title>Overview</v-card-title>
    <v-card-text>
      <template v-if="data.length == 0">
        <v-row justify="center" align="center" class="fill-height">
          <v-col cols="12" class="text-center">
            <h6 class="text-h6">No Data Yet</h6>
            <p class="text-body-1 mt-2">
              Start by choosing a Customer and adding some products.
            </p>
          </v-col>
        </v-row>
      </template>
      <template v-else>
        <v-expansion-panels>
          <v-expansion-panel
            v-for="(item, name, index) in grouped"
            :key="index"
          >
            <v-expansion-panel-header>
              {{ name }}
            </v-expansion-panel-header>
            <v-expansion-panel-content>
              <v-simple-table>
                <template v-slot:default>
                  <thead>
                    <tr>
                      <th>Product Name</th>
                      <th>Domain</th>
                      <th>Start Date</th>
                      <th>Duration</th>
                      <th>Expiration Notification</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(rec, index) in item" :key="index">
                      <td>{{ rec.ProductName }}</td>
                      <td>{{ rec.Domain }}</td>
                      <td>{{ rec.StartDate }}</td>
                      <td>{{ rec.Duration }}</td>
                      <td>{{ rec.Expiration }}</td>
                    </tr>
                  </tbody>
                </template>
              </v-simple-table>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>
      </template>
    </v-card-text>
    <v-card-actions>
      <v-spacer></v-spacer>
      <v-btn text @click="closeDialog">Ok</v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
import _ from "lodash";
export default {
  name: "overview",
  props: ["data"],
  methods: {
    closeDialog() {
      var self = this;
      self.$emit("overview-close");
    },
  },
  computed: {
    grouped() {
      var self = this;
      return _.groupBy(self.data, "CustomerName");
    },
  },
};
</script>

<style>
</style>