<template>
  <v-app>
    <v-app-bar app color="blue darken-3" class="white--text">
      <v-toolbar-title>ACME</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-toolbar-items>
        <v-btn @click="overviewPop = true" text dark> All Customers </v-btn>
      </v-toolbar-items>
    </v-app-bar>

    <v-main>
      <v-container>
        <v-row>
          <v-col cols="12" md="4">
            <v-card>
              <v-card-title> Customer Selection </v-card-title>
              <v-card-text>
                <v-select
                  outlined
                  label="Select a Customer"
                  v-model="selCust"
                  :items="custList"
                  item-text="name"
                  item-value="id"
                  hint="Select a Customer"
                ></v-select>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
        <v-slide-y-transition mode="out-in">
          <template v-if="selCust != ''">
            <v-row key="custSelected">
              <v-col cols="12" md="6">
                <v-card>
                  <v-card-title>
                    <span>Products</span>
                    <v-spacer></v-spacer>
                    <v-btn @click="addProduct" depressed color="primary">
                      <v-icon>mdi-plus</v-icon> Add Product
                    </v-btn>
                  </v-card-title>
                  <v-card-text>
                    <v-data-table
                      :headers="prodHeaders"
                      :items="filteredCustProd"
                    >
                    </v-data-table>
                  </v-card-text>
                </v-card>
              </v-col>
              <v-col cols="12" md="6">
                <v-card>
                  <v-card-title>Email Schedule</v-card-title>
                  <v-card-text>
                    <v-list two-line>
                      <v-list-item
                        :key="index"
                        v-for="(item, index) in filteredCustProd"
                      >
                        <v-list-item-content>
                          <v-list-item-title>
                            {{ item.ProductName }} -
                            {{ item.Domain }} Expiration Email
                          </v-list-item-title>

                          <v-list-item-subtitle>
                            {{ item.Expiration }}
                          </v-list-item-subtitle>
                        </v-list-item-content>
                      </v-list-item>
                    </v-list>
                  </v-card-text>
                </v-card>
              </v-col>
            </v-row>
          </template>
        </v-slide-y-transition>
      </v-container>
    </v-main>
    <v-dialog v-model="addProductPop" max-width="450">
      <v-card>
        <v-card-title>Add New Product</v-card-title>
        <v-card-text>
          <v-fade-transition mode="out-in">
            <template v-if="addNewProduct">
              <v-form v-model="intakeValid" ref="newIntakeForm">
                <v-row>
                  <v-col cols="12">
                    <v-select
                      :rules="[rules.required]"
                      outlined
                      v-model="intakeForm.ProductName"
                      prepend-inner-icon="mdi-format-list-checks"
                      :items="prodList"
                      item-text="ProductName"
                      item-value="ProductName"
                      label="Product"
                    ></v-select>
                  </v-col>
                  <v-col cols="12">
                    <v-text-field
                      :rules="[rules.required, rulesDomain]"
                      type="text"
                      outlined
                      prepend-inner-icon="mdi-earth"
                      label="Domain"
                      v-model="intakeForm.Domain"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" md="6">
                    <v-menu
                      v-model="menu"
                      :close-on-content-click="false"
                      :nudge-right="40"
                      transition="scale-transition"
                      offset-y
                      min-width="290px"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                          outlined
                          v-model="intakeForm.StartDate"
                          label="Start Date"
                          prepend-inner-icon="mdi-calendar"
                          readonly
                          v-bind="attrs"
                          v-on="on"
                        ></v-text-field>
                      </template>
                      <v-date-picker
                        v-model="intakeForm.StartDate"
                        @input="menu = false"
                      ></v-date-picker>
                    </v-menu>
                  </v-col>
                  <v-col cols="12" md="6">
                    <v-text-field
                      :rules="[rules.required]"
                      type="number"
                      v-model="intakeForm.Duration"
                      outlined
                      label="Duration"
                      :hint="
                        intakeForm.Duration != undefined
                          ? intakeForm.Duration + ' ' + durationTxt
                          : ''
                      "
                      prepend-inner-icon="mdi-pound"
                    >
                      <template v-slot:label>
                        <span>Duration</span>
                        <template
                          v-if="
                            durationTxt != '' &&
                            durationTxt != undefined &&
                            intakeForm.ProductName != '' &&
                            intakeForm.ProductName != undefined
                          "
                        >
                          {{ durationTxt }}
                        </template>
                      </template>
                    </v-text-field>
                  </v-col>
                </v-row>
              </v-form>
            </template>
          </v-fade-transition>
          <v-fade-transition>
            <template v-if="intakeValid">
              <h6 class="text-h6">
                Expected Expiration: <b>{{ expectedExpiration }}</b>
              </h6>
            </template>
          </v-fade-transition>
        </v-card-text>
        <v-divider></v-divider>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn @click="cancelAddProduct" depressed> Cancel </v-btn>
          <v-btn
            @click="addNewProduct"
            :disabled="!intakeValid"
            text
            color="primary"
          >
            Submit
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="overviewPop" max-width="650">
      <overview @overview-close="overviewPop = false" :data="custProducts" />
    </v-dialog>
  </v-app>
</template>

<script>
import overview from "@/components/overview";
export default {
  components: {
    overview,
  },
  computed: {
    expectedExpiration() {
      var self = this;
      var dt = new Date(self.intakeForm.StartDate);
      var yearDt = 365 * Number(self.intakeForm.Duration);
      var monthDt = 30 * Number(self.intakeForm.Duration);
      if (self.durationTxt == "Years") {
        dt.setDate(dt.getDate() + yearDt);
      } else {
        dt.setDate(dt.getDate() + monthDt);
      }
      return dt.toLocaleDateString();
    },
    durationTxt() {
      var self = this;
      switch (self.intakeForm.ProductName) {
        case "Hosting":
          return "Months";
        case "Email":
          return "Months";
        default:
          return "Years";
      }
    },
    filteredCustProd() {
      var self = this;
      return self.custProducts.filter((r) => {
        return r.CustomerID == self.selCust;
      });
    },
  },
  data: () => ({
    overviewPop: false,
    selCust: "",
    addProductPop: false,
    menu: "",
    intakeValid: "",
    intakeForm: {
      CustID: "",
      Domain: "",
      ProductName: "",
      StartDate: new Date().toISOString().substr(0, 10),
      Duration: "",
    },
    rules: {
      required: (value) => !!value || "Required.",
    },
    custList: [
      {
        name: "Customer 123",
        id: "Cust123",
      },
      {
        name: "Customer 234",
        id: "Cust234",
      },
      {
        name: "Customer 345",
        id: "Cust345",
      },
      {
        name: "Customer 456",
        id: "Cust456",
      },
    ],
    prodList: [
      {
        ProductName: "Domain",
        AllowedString: [".com", ".org"],
        DurationType: "Years",
        DurationLimit: 1,
        ExpirationEmail: 2,
        ActivationEmail: 0,
      },
      {
        ProductName: "Hosting",
        AllowedString: [".com", ".org"],
        DurationType: "Months",
        DurationLimit: 1,
        ExpirationEmail: 3,
        ActivationEmail: 1,
      },
      {
        ProductName: "Email",
        AllowedString: [".com", ".org"],
        DurationType: "Years",
        DurationLimit: 0,
        ExpirationEmail: 2,
        ActivationEmail: 0,
      },
      {
        ProductName: "P-Domain",
        AllowedString: [".com", ".org"],
        DurationType: "Years",
        DurationLimit: 0,
        ExpirationEmail: 2,
        ActivationEmail: 0,
      },
      {
        ProductName: "E-Domain",
        AllowedString: [".com", ".org"],
        DurationType: "Years",
        DurationLimit: 0,
        ExpirationEmail: 2,
        ActivationEmail: 0,
      },
    ],
    custProducts: [],
    prodHeaders: [
      {
        text: "Product",
        align: "start",
        sortable: true,
        value: "ProductName",
      },
      {
        text: "Domain",
        align: "center",
        sortable: true,
        value: "Domain",
      },
      {
        text: "Start Date",
        align: "start",
        sortable: true,
        value: "StartDate",
      },
      {
        text: "Duration",
        align: "start",
        sortable: true,
        value: "Duration",
      },
    ],
  }),
  methods: {
    rulesProd() {
      var self = this;
      if (self.intakeForm.ProductName == "Domain") {
        return;
      }
    },
    rulesDomain() {
      var self = this;
      if (self.intakeForm.ProductName == "Domain") {
        if (
          self.intakeForm.Domain.toLowerCase().indexOf(".com") < 0 &&
          self.intakeForm.Domain.toLowerCase().indexOf(".org") < 0
        ) {
          return "Domains must be either '.com' or '.org'.";
        } else {
          return true;
        }
      } else if (self.intakeForm.ProductName == "E-Domain") {
        if (self.intakeForm.Domain.toLowerCase().indexOf(".edu") < 0) {
          return "Domains must be '.edu'";
        } else {
          return true;
        }
      } else {
        return true;
      }
    },
    addProduct() {
      var self = this;
      self.addProductPop = true;
    },
    addNewProduct() {
      var self = this;
      var dt = new Date(self.intakeForm.StartDate);
      var yearDt = 365 * Number(self.intakeForm.Duration);
      var monthDt = 30 * Number(self.intakeForm.Duration);
      if (self.durationTxt == "Years") {
        dt.setDate(dt.getDate() + yearDt);
      } else {
        dt.setDate(dt.getDate() + monthDt);
      }
      if (
        self.custProducts.find((r) => {
          return r.Domain.toLowerCase() == self.intakeForm.Domain.toLowerCase();
        })
      ) {
        alert("Domain already exist. Cannot add again");
      } else {
        self.custProducts.push({
          CustomerID: self.selCust,
          CustomerName: self.custList.find((r) => {
            return r.id == self.selCust;
          }).name,
          ProductName: self.intakeForm.ProductName,
          Domain: self.intakeForm.Domain,
          StartDate: self.intakeForm.StartDate,
          Duration:
            self.intakeForm.Duration +
            " " +
            self.prodList.find((r) => {
              return r.ProductName == self.intakeForm.ProductName;
            }).DurationType,
          Expiration: dt.toLocaleDateString(),
        });
        self.addProductPop = false;
        self.clearIntake();
      }
    },
    cancelAddProduct() {
      var self = this;
      self.addProductPop = false;
      self.clearIntake();
    },
    clearIntake() {
      var self = this;
      self.$refs.newIntakeForm.reset();
      self.intakeForm.StartDate = new Date().toISOString().substr(0, 10);
    },
  },
};
</script>