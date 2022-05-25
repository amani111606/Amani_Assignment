<template>
  <div>
    <h1>Products</h1>
    <div>
      <span
        @click="showProducts()"
        style="float: left; font-size: 20px; font-weigt: bold"
        ><u>View Cart</u></span
      >
      <div style="padding: 30px" v-if="showpro">
        <table class="show-table" border="2" v-if="cartproductsInfo.length > 0">
          <thead>
            <tr>
              <th>Sno</th>
              <th>Title</th>
              <th>Price</th>
              <th>Stock</th>
              <th>Category</th>
            </tr>
          </thead>
          <tr v-for="(item, index) in cartproductsInfo" :key="item.id">
            <td>{{ index + 1 }}</td>
            <td>{{ item.title }}</td>
            <td>{{ item.price }}</td>
            <td>{{ item.stock }}</td>
            <td>{{ item.category }}</td>
          </tr>
        </table>
        <div v-else>
          <span
            style="
              color: red;
              font-size: 18px;
              float: left;
              padding: 15px;
              font-weight: bold;
            "
            >Your Cart is Empty</span
          >
        </div>
      </div>
    </div>
    <div class="table-div">
      <table border="1px" class="prod-table">
        <thead>
          <tr>
            <th>Id</th>
            <th>Title</th>
            <th>Description</th>
            <th @click="sortOnPrice('price')">
              Price <a href="#" class="sort-by"></a>
            </th>
            <th @click="sortOnPrice('discount')">
              Discount % <a href="#" class="sort-by"></a>
            </th>
            <th @click="sortOnPrice('rating')">
              Rating <a href="#" class="sort-by"></a>
            </th>
            <th>Stock</th>
            <th>Buy</th>
            <th>Category</th>
            <th>Brand</th>
            <th>Thumbnail</th>
            <th>Images</th>
          </tr>
        </thead>
        <tr>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
          <td>
            <select
              name="category"
              @change="filteredProducts($event)"
              v-model="searchTerm"
            >
              <option value="">select</option>
              <option
                v-for="(cat, index) in categoryList"
                :key="index"
                :value="cat"
              >
                {{ cat }}
              </option>
            </select>
          </td>
          <td>
            <select
              name="Brand"
              @change="filteredProducts($event)"
              v-model="searchBrand"
            >
              <option value="">select</option>
              <option
                v-for="(brand, index) in brandList"
                :key="index"
                :value="brand"
              >
                {{ brand }}
              </option>
            </select>
          </td>
          <td></td>
          <td></td>
        </tr>
        <tr v-for="item in products" :key="item.id">
          <td>{{ item.id }}</td>
          <td>{{ item.title }}</td>
          <td>{{ item.description }}</td>
          <td>{{ item.price }}</td>
          <td>{{ item.discountPercentage }}</td>
          <td>{{ item.rating }}</td>
          <td>{{ item.stock }}</td>
          <td>
            <button @click="buyProducts(item.stock, item.id)">
              Add to Cart
            </button>
          </td>
          <td>{{ item.category }}</td>
          <td>{{ item.brand }}</td>
          <td>{{ item.thumbnail }}</td>
          <td><img :src="item.images[0]" /></td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import axios from "axios";
import VueAxios from "vue-axios";
Vue.use(VueAxios, axios);

export default {
  name: "hello-world",
  data() {
    return {
      products: [],
      filteredproducts: [],
      searchTerm: "",
      searchBrand: "",
      searchorderkey: 0,
      cartproducts: [],
      dprod: [],
      dprod1: [],
      cartproductsInfo: [],
      showpro: null,
      categoryList: [],
      brandList: [],
      duplicateList: [],
      duplicateList1: [],
    };
  },
  computed: {
    filteredProducts() {
      if (
        this.searchTerm === "" &&
        this.products.length !== this.filteredproducts.length
      ) {
        this.getData("");
      } else {
        this.filteredproducts.forEach((prod, index) => {
          if (index === 0) {
            this.products = [];
          }
          if (
            prod.category
              .toLowerCase()
              .includes(this.searchTerm.toLowerCase()) &&
            prod.brand.toLowerCase().includes(this.searchBrand.toLowerCase())
          ) {
            this.products.push(prod);
          }
        });
      }
      this.barndFilter();

      return "";
    },
  },
  methods: {
    barndFilter() {
      this.duplicateList1 = [];
      this.filteredproducts.forEach((prod, index) => {
        if (prod.category === this.searchTerm) {
          this.duplicateList1.push(prod.brand);
        }
      });
      this.brandList = new Set(this.duplicateList1);
    },
    getData(col) {
      this.searchBrand = "";
      Vue.axios
        .get("https://dummyjson.com/products?limit=100")
        .then((response) => {
          this.products = response.data.products;
          //categoryList
          if (col) {
            if (this.searchorderkey === 0) {
              if (col === "price") {
                this.products.sort((t1, t2) => (t1.price < t2.price ? -1 : 1));
              } else if (col === "discount") {
                this.products.sort((t1, t2) =>
                  t1.discountPercentage < t2.discountPercentage ? -1 : 1
                );
              } else {
                this.products.sort((t1, t2) =>
                  t1.rating < t2.rating ? -1 : 1
                );
              }
              this.searchorderkey = 1;
            } else {
              if (col === "price") {
                this.products.sort((t1, t2) => (t1.price < t2.price ? 1 : -1));
              } else if (col === "discount") {
                this.products.sort((t1, t2) =>
                  t1.discountPercentage < t2.discountPercentage ? 1 : -1
                );
              } else {
                this.products.sort((t1, t2) =>
                  t1.rating < t2.rating ? 1 : -1
                );
              }
              this.searchorderkey = 0;
            }
          }
          this.filteredproducts = response.data.products;
          this.filteredproducts.forEach((prod, index) => {
            this.duplicateList.push(prod.category);
          });
          this.categoryList = new Set(this.duplicateList);
        });
    },
    sortOnPrice(col) {
      this.getData(col);
    },
    buyProducts(productscount, itemid) {
      if (productscount < 50) {
        alert("hurry! only a few items left");
      }
      this.cartproducts.push(itemid);
      /*for (var i = 0; i < this.cartproducts.length; i++) {
        console.log(this.filteredproducts.this.cartproducts[i]);
      }*/
    },
    showProducts() {
      this.cartproductsInfo = [];
      this.showpro = !this.showpro;
      //this.dprod = new Set(this.cartproducts);
      console.log("----------------");
      /* for (var c = 0; c < this.cartproducts.length; c++) {
        console.log("@@@@@@@@@@@@@@@@@@@@@@@@@");
        console.log(this.dprod.id);
      }*/

      this.filteredproducts.forEach((prod, index) => {
        for (var j = 0; j < this.cartproducts.length; j++) {
          if (prod.id === this.cartproducts[j]) {
            this.dprod.push(prod);
          }
        }
      });
      this.dprod1 = new Set(this.dprod);
      this.dprod1.forEach((prod, index) => {
        this.cartproductsInfo.push(prod);
      });
      /*for (var i = 0; i < this.cartproducts.length; i++) {
        console.log(this.filteredproducts.this.cartproducts[i]);
      }*/
    },
  },
  mounted() {
    this.getData("");
    this.cartproducts = [];
    this.cartproductsInfo = [];
  },
};
</script>
<style scoped>
h1 {
  background-color: green;
  color: white;
}
.show-table {
  border-collapse: collapse;
  width: 50%;
  text-align: left;
  background: #f2f2f2;
  border: 1px solid black;
}
.table-div {
  width: 100%;
  overflow: auto;
}
.prod-table {
  width: 100%;
  border-collapse: collapse;
  border: 1px solid black;
  text-align: left;
}
.prod-table td {
  padding-left: 10px;
}
.prod-table th {
  background-color: #5e00ff;
  color: white;
  font-weight: bold;
  font-size: 18px;
  min-width: 150px;
  text-align: center;
  height: 40px;
}
.prod-table tr:nth-child(even) {
  background-color: #f2f2f2;
}
img {
  width: 50px;
  height: 50px;
}
select {
  width: 100%;
  height: 30px;
  font-size: 20px;
}
.arrow {
  box-sizing: border-box;
  height: 5vw;
  width: 5vw;
  border-style: solid;
  border-color: white;
  border-width: 0px 1px 1px 0px;
  transform: rotate(45deg);
  transition: border-width 150ms ease-in-out;
}
.prod-table th a.sort-by {
  padding-right: 30px;
  position: relative;
}
a.sort-by:before,
a.sort-by:after {
  border: 4px solid transparent;
  content: "";
  display: block;
  height: 0;
  right: 5px;
  top: 50%;
  position: absolute;
  width: 0;
}
a.sort-by:before {
  border-bottom-color: white;
  margin-top: -9px;
}
a.sort-by:after {
  border-top-color: white;
  margin-top: 1px;
}
</style>