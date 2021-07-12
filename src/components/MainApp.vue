<template>
  <div class="container">
    <form>
      <table>
        <thead>
          <tr>
            <th>Product Code</th>
            <th>Description</th>
            <th>Quantity</th>
            <th>Total Cost Price</th>
            <th>Total Sell Price</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, index) in table" :key="index">
            <td>
              {{ row["Product Code"] }}
            </td>
            <td>
              {{ row["Description"] }}
            </td>

            <td
              @click.prevent="iconClicked"
              contenteditable="true"
              @keypress="modifyQuantity($event, index)"
              :v-bind="row['Quantity']"
              @keydown.enter.exact.prevent="modifyQuantity($event, index)"
            >
              {{ row["Quantity"] }}
              <p class="fa fa-edit" v-if="clicked"></p>
            </td>
            <td>
              {{ row["Total Cost Price"] }}
            </td>
            <td>
              {{ row["Total Sell Price"] }}
            </td>
            <td style="background-color: transparent">
              <button class="btn" @click.prevent="removeRow($event, index)">
                <i class="fa fa-trash"></i>
              </button>
            </td>
          </tr>

          <tr v-if="!show">
            <td>
              <input type="text" ref="code" />
            </td>
            <td>
              <input type="text" ref="description" />
            </td>

            <td>
              <input type="text" ref="quantity" />
            </td>
            <td></td>
            <td></td>
          </tr>
        </tbody>
      </table>
    </form>
    <div>
      <button class="addRow" v-if="show" @click="showRow">+</button>
      <div v-if="errors.length">
        <p v-for="(error, index) in errors" :key="index">{{ error }}</p>
      </div>
    </div>
    <button class="save" v-if="!show" @click.prevent="addRow">Save</button>
    <button class="save" v-if="!show" @click.prevent="cancel">Cancel</button>
  </div>
</template>

<script>
// Get the item list
import itemList from "../data/itemlist.json";
// get the csv rates as json file
import rates from "../data/rates_lookup.csv";
export default {
  data() {
    return {
      itemList,
      rates,
      table: [],
      show: true,
      errors: [],
      state: true,
      clicked: true,
    };
  },

  mounted: function () {
    // Call the init funtion when the component is mounted
    this.initial();
  },

  methods: {
    // this function gets the data from the json files and sets the table values
    initial: function () {
      // temp aray to hold the values
      let enmptyArray = [];
      let cost = 0;
      let sell = 0;
      // loop through the two json files and calculate the cost and the selling price
      try {
        itemList.forEach((item) => {
          rates.forEach((rate) => {
            if (item.code == rate["Product Code"]) {
              cost = parseFloat(
                item.quantity * rate["Unit Cost Price"]
              ).toFixed(2);

              sell = parseFloat(
                item.quantity * rate["Unit Sell Price"]
              ).toFixed(2);

              // Create an object that holds all the desired values
              enmptyArray.push({
                "Product Code": item.code,
                Description: item.description,
                Quantity: item.quantity,
                "Total Cost Price": cost,
                "Total Sell Price": sell,
              });
            }
          });
        });
      } catch (e) {
        console.log(e);
        enmptyArray = [];
      }

      // Push the newly created object to the main table that is going to be used to populate the form table

      this.table = enmptyArray;
    },

    showRow: function () {
      this.show = !this.show;
    },

    addRow: function () {
      // Initialise the errors back to zero
      this.errors.length = 0;
      // Get the product code entered
      let code = this.$refs.code.value.trim();
      // Get the description
      let description = this.$refs.description.value.toLowerCase().trim();
      // Get the quantity
      let quantity = parseInt(this.$refs.quantity.value.trim());
      let codeList = [];
      let itemName = [];
      // Strore the values inside an Array to be checked
      this.itemList.forEach((item) => codeList.push(item.code));
      this.itemList.forEach((item) => itemName.push(item.description.toLowerCase()));

      // Display error if no match
      if (!codeList.includes(code)) {
        this.errors.push("Please a enter a correct Product Code");

        return;
      }
      if (!description |!itemName.includes(description)) {
        this.errors.push("Please a enter a correct Description");

        return;
      }
      if ( !quantity) {
        this.errors.push("Please provide a correct Quantity");

        return;
      }

      this.show = !this.show;

      // Construct a new object with the entered values
      let obj = {
        code: code,
        job: "",
        description: description,
        quantity: quantity,
      };
      // Add the new object to the item list
      this.itemList.push(obj);

      // render the table again
      this.initial();
    },

    // The cancel button to hide the button
    cancel: function () {
      this.show = !this.show;
      this.errors.length = 0;
    },
    removeRow: function (e, index) {
      // check if the enter key is pressed
      if (e.clientX == "0") return;
      // else delete the row
      this.table.splice(index, 1);
    },

    // Modify the Quantity
    modifyQuantity: function (e, index) {
      // Wait 10 milliseconds for the value
      setTimeout(() => {
        // Get the entered value and update the Quantity
        let newValue = parseInt(e.target.innerText);
        this.itemList[index].quantity = newValue;
        // Call the initial function to recalculate the sellin and the cost prices
        this.initial();
      }, 10);
    },
    // Hide the modify icon
    iconClicked: function () {
      this.clicked = false;
    },
  },
};
</script>

<style scoped>
table {
  width: 70%;
  margin: auto;
  table-layout: fixed;
}
body {
  font-family: "lato", sans-serif;
  color: #444a50;
}
td {
  border-radius: 3px;
  padding: 5px 10px;
  justify-content: space-between;
  margin-bottom: 5px;
  background-color: rgba(0, 0, 0, 0.05);
}
thead th {
  padding: 10px;
}
thead {
  background-color: #b9d342;
  font-size: 14px;
  letter-spacing: 0.03em;
}
td input {
  box-sizing: border-box;
}
.btn {
  border: none;
  color: #444a50;
  background-color: transparent;
  padding: 12px 16px;
  font-size: 16px;
  cursor: pointer;
  margin-left: -70%;
}

.btn:hover {
  background-color: rgb(228, 22, 22);
}
.addRow {
  margin-right: 68%;
  margin-top: 7px;
}
.save {
  background-color: #b9d342;
  border-radius: 28px;
  border: 1px solid #b9d342;
  display: inline-block;
  cursor: pointer;
  color: #444a50;
  font-weight: bold;
  font-family: Arial;
  font-size: 17px;
  padding: 7px 16px;
}
.save:hover {
  background-color: #444a50;
  color: #ffffff;
}
</style>

