<template>
  <div>
    <v-card class="ma-5 mb-12 pa-5">
      <v-tabs v-model="tabs" right color="deep-purple accent-4">
        <v-tab>Ingredients</v-tab>
        <v-tab>Ingredients' Log</v-tab>
      </v-tabs>
      <v-tabs-items v-model="tabs">
        <v-tab-item>
          <v-card flat class="ma-5 mb-12 pa-5">
            <v-card-title>Ube Halaya List of Ingredients</v-card-title>
            <v-row>
              <v-flex>
                <v-layout wrap>
                  <v-flex md4 v-for="item in halayaIngredients" :key="item.id">
                    <v-card id="cards" class="card-container ma-5">
                      <v-card-title class="deep-purple lighten-5">{{item.ingredients_name}}</v-card-title>
                      <hr>
                      <v-row class="mx-auto text-center">
                        <v-list-item>
                          Available Quantity :
                          <b>{{ item.ingredients_remaining}} {{item.ingredients_unit}}(s)</b>
                        </v-list-item>
                        <v-list-item>
                          Used Quantity :
                          <b>{{ item.total}} {{item.ingredients_unit}}(s)</b>
                        </v-list-item>
                        <v-list-item>
                          <v-chip
                            text-color="white"
                            :color="getColor(item.ingredients_status)"
                          >{{item.ingredients_status}}</v-chip>
                        </v-list-item>
                      </v-row>
                      <v-row class="d-flex justify-end mb-6">
                        <template>
                          <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn icon>
                              <v-icon
                                title="Add Quantity"
                                @click="editDialog = !editDialog, editIngredients(item) "
                              >mdi-plus</v-icon>
                            </v-btn>
                            <v-btn icon>
                              <v-icon
                                @click=" addUsedStockDialog=!addUsedStockDialog ,editEstimatedValue(item) "
                                title="Used Quantity"
                              >mdi-minus</v-icon>
                            </v-btn>
                          </v-card-actions>
                        </template>
                      </v-row>
                    </v-card>
                  </v-flex>
                </v-layout>
              </v-flex>
            </v-row>
          </v-card>
          <template>
            <v-dialog v-model="addUsedStockDialog" style="height:auto;" width="400px">
              <v-card class="ma-0 pa-0">
                <v-spacer></v-spacer>
                <v-card-title class="align-center">
                  <v-list-item-title
                    class="deep-purple--text d-flex align-center justify-center mx-auto headline"
                  >USED QUANTITY</v-list-item-title>
                </v-card-title>
                <v-divider></v-divider>
                <v-container>
                  <v-row class="mx-2">
                    <v-col cols="12">
                      <v-text-field
                        color="purple"
                        label="Ingredient's Name"
                        v-model="editValue.ingredients_name"
                        v-bind:disabled="disabled"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="8">
                      <v-text-field
                        color="purple"
                        min="1"
                        type="number"
                        label="Quantity"
                        v-model="usedIngredientsAmount"
                        :error-messages="ingredientsUsedAmountErrors"
                        @input="$v.usedIngredientsAmount.$touch()"
                        @blur="$v.usedIngredientsAmount.$touch()"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="4">
                      <v-text-field
                        color="purple"
                        label="Unit"
                        v-model="editValue.ingredients_unit"
                        v-bind:disabled="disabled"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn small outlined color="orange" @click="addUsedStockDialog = false">CANCEL</v-btn>
                  <v-btn small outlined color="purple darken-2" @click="addIngredientsAmount()">SAVE</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </template>
          <template>
            <v-dialog v-model="editDialog" style="height:auto;" width="400px">
              <v-card class="ma-0 pa-0">
                <v-spacer></v-spacer>
                <v-card-title class="deep-purple--text align-center">
                  <v-list-item-title
                    class="deep-purple--text d-flex align-center justify-center mx-auto headline"
                  >Purchased Quantity</v-list-item-title>
                </v-card-title>
                <v-divider></v-divider>
                <v-container>
                  <v-row class="mx-2">
                    <v-col cols="12">
                      <v-text-field
                        type="number"
                        color="purple"
                        v-model="editStockItem.ingredients_remaining"
                        label="Available Quantity"
                      >{{editStockItem.ingredients_remaining}}</v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn
                    small
                    outlined
                    color="orange"
                    @click="editDialog = false,reloadDataAddStock()"
                  >CANCEL</v-btn>
                  <v-btn small outlined color="purple darken-2" @click=" updateIngredients()">SAVE</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </template>
        </v-tab-item>
        <v-tab-item>
          <v-card class="ma-5 mb-12 pa-5">
            <div>
              <v-card flat>
                <h4>Filter</h4>
                <v-row>
                  <v-col cols="3">
                    <v-select v-model="month" label="Month" :items="months"></v-select>
                  </v-col>
                  <v-col cols="3">
                    <v-text-field v-model="year" label="Year"></v-text-field>
                  </v-col>
                  <v-col cols="2">
                    <v-btn @click="filter(month,year)" outlined medium color="purple">Apply</v-btn>
                  </v-col>
                  <v-spacer></v-spacer>
                  <v-col cols="2">
                    <div>
                      <v-menu offset-y>
                        <template v-slot:activator="{ on, attrs }">
                          <div>
                            <v-btn
                              @click="isEmpty(displayIngredientsRecords)"
                              class="float-right"
                              outlined
                              color="purple"
                              v-bind="attrs"
                              v-on="on"
                            >
                              <v-icon>mdi-download</v-icon>Export
                            </v-btn>
                          </div>
                        </template>
                        <v-list v-show="is_empty === false">
                          <v-col>
                            <IngredientsPdf
                              :headersIngredients="headersIngredients"
                              :displayIngredientsRecords="displayIngredientsRecords"
                            ></IngredientsPdf>
                            <div>
                              <download-csv
                                :data="displayIngredientsRecords"
                                name="IngredientsLog.csv"
                              >
                                <v-btn text small>Export as CSV</v-btn>
                              </download-csv>
                            </div>
                          </v-col>
                        </v-list>
                      </v-menu>
                    </div>
                  </v-col>
                </v-row>
              </v-card>
            </div>
            <v-card-title>
              Ingredients' Log
              <v-spacer></v-spacer>
              <v-text-field
                v-model="search"
                append-icon="mdi-magnify"
                label="Search"
                single-line
                hide-details
              ></v-text-field>
            </v-card-title>
            <v-spacer></v-spacer>
            <v-data-table
              :headers="headersIngredients"
              :items="displayIngredientsRecords"
              :search="search"
            >
              <template
                #item.used_ingredients_amount="{ item }"
              >{{ item.used_ingredients_amount }} {{ item.ingredients_unit }}</template>
              <template v-slot:item.updated_at="{ item }">
                <span>{{new Date(item.updated_at).toISOString().substring(0,10)}}</span>
              </template>
              <template v-slot:item.action="{ item }">
                <v-icon normal title="Edit">mdi-table-edit</v-icon>
                <v-icon class="mr-3" color="red" normal title="Delete Ingredient">mdi-delete</v-icon>
              </template>
            </v-data-table>
          </v-card>
        </v-tab-item>
      </v-tabs-items>
    </v-card>
  </div>
</template>

<script>
import JsonCSV from "vue-json-csv";
import IngredientsPdf from "./IngredientsPdf.vue";
import Swal from "sweetalert2";
import axios from "axios";
import { setInterval } from "timers";
import {
  required,
  minLength,
  maxLength,
  between
} from "vuelidate/lib/validators";
export default {
  name: "Ingredients",
  components: { IngredientsPdf },
  data() {
    return {
      loading: true,
      displayIngredientsRecords: [],
      month: "",
      year: " ",
      is_empty: false,
      months: [
        "All",
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December"
      ],
      dropdown: [{ title: "Download as PDF" }, { title: "Download as CSV" }],
      stockDialog: false,
      editDialog: false,
      addUsedStockDialog: false,
      ingredientsUnit: "",
      ingredientsName: "",
      stockStatus: "calculating...",
      search: "",
      ubeKilo: "",
      stocks: [],
      availableIngredients: "",
      usedIngredientsAmount: "",
      halayaIngredients: [],
      editStockItem: {},
      itemSelect: [],
      editValue: [],
      tabs: null,
      disabled: true,
      headersIngredients: [
        {
          text: "Ingredient's Name",
          align: "start",
          sortable: false,
          value: "ingredients_name"
        },
        // { text: "Available Quantity", value: "ingredients_remaining" },
        { text: "Used Quantity", value: "used_ingredients_amount" },
        { text: "Recorded Date", value: "updated_at" }
      ]
    };
  },
  validations: {
    ingredientsUnit: {
      required
    },
    ingredientsName: {
      required
    },
    availableIngredients: {
      required
    },
    usedIngredientsAmount: {
      required
    }
  },
  beforeCreate() {
    let config = {};
    config.headers = {
      Authorization: "Bearer " + localStorage.getItem("token"),
      "Access-Control-Allow-Origin": "*"
    };
    this.config = config;
  },
  created() {
    this.getHalayaIngredients();
    this.getAllIngredientsName();
    this.retrieveUsedIngredients();
  },
  props: {
    color: "primary"
  },
  computed: {
    ingredientsNameErrors() {
      const errors = [];
      if (!this.$v.ingredientsName.$dirty) return errors;
      !this.$v.ingredientsName.required && errors.push("name is required.");
      return errors;
    },
    ingredientsUnitErrors() {
      const errors = [];
      if (!this.$v.ingredientsUnit.$dirty) return errors;
      !this.$v.ingredientsUnit.required && errors.push("quantity is required.");
      return errors;
    },
    ingredientsAvailableErrors() {
      const errors = [];
      if (!this.$v.availableIngredients.$dirty) return errors;
      !this.$v.availableIngredients.required &&
        errors.push("name is required.");
      return errors;
    },
    ingredientsUsedAmountErrors() {
      const errors = [];
      if (!this.$v.usedIngredientsAmount.$dirty) return errors;
      !this.$v.usedIngredientsAmount.required &&
        errors.push("unit is required.");
      return errors;
    }
  },
  methods: {
    getColor(status) {
      if (status === "Alert! Very Low") return "red";
      else if (status === "Warning! Running Low") return "orange";
      else if (status === "Calculating...") return "blue";
      else return "green";
    },
    getHalayaIngredients() {
      this.loading = true;
      axios
        .get(this.url + "/api/getHalayaIngredients", this.config)
        .then(response => {
          this.loading = false;
          let results = [];
          for (var i = 0; i < response.data.length; i++) {
            if (this.containsObject(results, response.data[i].id)) {
            } else {
              results.push(response.data[i]);
              this.halayaIngredients = results;
            }
            continue;
          }
        });
    },
    editEstimatedValue(item) {
      axios
        .get(
          this.url + "/api/post/updateEstimatedValue/" + item.id,
          this.config
        )
        .then(response => {
          this.editValue = response.data;
        });
    },
    retrieveUsedIngredients() {
      this.$vloading.show();
      axios.get(this.url + "/api/fetchUsedIng", this.config).then(response => {
        setTimeout(() => {
          this.$vloading.hide();
        }, 1000);
        this.displayIngredientsRecords = response.data.data;
      });
    },
    getAllIngredientsName() {
      let nameArray = [];
      axios
        .get(this.url + "/api/fetch/ingredientsName", this.config)
        .then(response => {
          for (var i = 0; i < response.data.length; i++) {
            if (nameArray.includes(response.data[i].ingredients_name)) {
            } else {
              nameArray.push(response.data[i].ingredients_name);
              this.itemSelect = nameArray;
            }
            continue;
          }
        });
    },
    reloadDataAddStock() {
      (this.ingredientsName = ""),
        (this.ingredientsUnit = ""),
        this.getHalayaIngredients();
      this.retrieveUsedIngredients();
      this.$v.$reset();
    },
    reloadDataAddUsedAmount() {
      (this.availableIngredients = ""),
        (this.usedIngredientsAmount = ""),
        this.getHalayaIngredients();
      this.retrieveUsedIngredients();
      this.$v.$reset();
    },
    editIngredients(item) {
      axios
        .get(this.url + "/api/post/editStock/" + item.id, this.config)
        .then(response => {
          this.editStockItem = response.data;
        });
    },
    updateIngredients() {
      axios
        .post(
          this.url + "/api/post/updateStock",
          this.editStockItem,
          this.config
        )
        .then(response => {
          this.getHalayaIngredients();
          this.retrieveUsedIngredients();
          this.editDialog = false;
        });
    },
    containsObject(arr, id) {
      return arr.some(function(el) {
        return el.id === id;
      });
    },
    addIngredientsAmount() {
      this.$v.$touch();
      if (
        this.editValue.ingredients_name === "" ||
        this.usedIngredientsAmount === ""
      ) {
        this.addUsedStockDialog = true;
      } else {
        let newAddedAmount = {
          availableIngredients: this.editValue.ingredients_name,
          ingredientsUnit: this.editValue.ingredients_unit,
          usedIngredientsAmount: this.usedIngredientsAmount
        };
        axios
          .post(
            this.url + "/api/post/usedIngredients",
            newAddedAmount,
            this.config
          )
          .then(response => {
            this.reloadDataAddUsedAmount();
            this.retrieveUsedIngredients();
            this.addUsedStockDialog = false;
          });
      }
    },
    getMonthNumber(month) {
      switch (month) {
        case "January":
          return "01";
          break;
        case "February":
          return "02";
          break;
        case "March":
          return "03";
          break;
        case "April":
          return "04";
          break;
        case "May":
          return "05";
          break;
        case "June":
          return "06";
          break;
        case "July":
          return "07";
          break;
        case "August":
          return "08";
          break;
        case "September":
          return "09";
          break;
        case "October":
          return "10";
          break;
        case "November":
          return "11";
          break;
        case "December":
          return "12";
      }
    },
    filter(month, year) {
      if (month == "All") {
        this.year = " ";
        this.retrieveUsedIngredients();
      } else if (this.year == " ") {
        Swal.fire({
          position: "center",
          icon: "error",
          title: "Year cannot be empty",
          showConfirmButton: true
        });
      } else {
        let month_number = this.getMonthNumber(month);
        axios
          .post(
            this.url + `/api/filterIngredientsLog/${month_number}/${year}`,
            {},
            this.config
          )
          .then(response => {
            console.log("-->>", response.data);
            if (response.data.data.length == 0) {
              this.is_empty = true;
              this.displayIngredientsRecords = response.data.data;
            } else {
              this.is_empty = false;
              this.displayIngredientsRecords = response.data.data;
            }
          });
      }
    },
    isEmpty(displayIngredientsRecords) {
      if (displayIngredientsRecords.length == 0) {
        Swal.fire({
          position: "center",
          icon: "warning",
          title: "Cannot be Downloaded. No Data Available",
          showConfirmButton: true
        });
      } else {
        this.is_empty = false;
      }
    }
  }
};
</script>
<style scoped>
#cards {
  padding: 10px !important;
  justify-content: center !important;
  width: 270px;
}
.v-application .mb-6 {
  margin-bottom: 0px !important;
}
.row {
  display: flex;
  flex-wrap: wrap;
  flex: 1 1 auto;
  margin-right: 0px !important;
  margin-left: 0px !important;
}
</style>