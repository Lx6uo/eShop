<template>
  <div v-if="product">
    <div class="product">
      <!-- Product image, name, price -->
      <div class="product-image">
        <img :src="`/media/${product.fields.product_image}`" alt="">
      </div>
      <div class="product-info">
        <h1 class="product-name">Product ID: {{ product.pk }}</h1>
        <h1 class="product-name">{{ product.fields.product_name }}</h1>
        <div class="product-cost">Product Stock: {{ product.fields.product_stock }}</div>
        <div class="product-cost">Product Sales: {{ product.fields.product_sales }}</div>
        <div class="product-cost">Product Price: {{ product.fields.product_cost }}</div>
        <div class="product-cost">Product Brand: {{ product.fields.product_brand }}</div>
        <div class="product-cost">Product Color: {{ product.fields.product_color }}</div>
        <div class="product-cost">Product Status: {{ product.fields.product_status }}</div>

        <div class="product-add-cart" v-if="isOn" @click="handleAddCart">Edit Product</div>
        <div class="product-add-cart" v-if="isOn" @click="removePro">Remove Product</div>
        <div class="product-add-cart" v-if="!isOn" @click="handleAddCart">Edit and List Product</div>
      </div>



    </div>



    <div class="product-desc">
      <h2>Product Description</h2>
      <img :src="`/media/${product.fields.product_imageDetail}`" alt="">
    </div>
  </div>
</template>

<script>
//导入本地数据
//import product_data from './product.js';

export default {
  data() {
    return {
      //获取路由中的参数
      id: parseInt(this.$route.params.id),
      product: null,
      product_goods: 0,
      begood: true,
      bestar: true,
      shop_stars: 0,
      isOn: true,
    }
  },
  methods: {
    async removePro() {
      await this.axios.get('delete_product/',
        { params: { product_id: this.id } })
        .then((response) => {
          console.log(response);

        })
        .catch(function (error) {
          console.log(error);
        });
    },

    async getProductLikes() {
      await this.axios.get('get_product_likes/',
        { params: { product_id: this.id } })
        .then((response) => {
          console.log(response);
          this.product_goods = response.data.goods

        })
        .catch(function (error) {
          console.log(error);
        });

      await this.axios.get('get_shop_stars/',
        { params: { business_id: this.product.fields.product_business } })
        .then((response) => {
          console.log(response);
          this.shop_stars = response.data.stars

        })
        .catch(function (error) {
          console.log(error);
        });
    },
    async changeGood() {
      if (this.begood) {
        this.product_goods -= 1
      } else {
        this.product_goods += 1
      }
      this.begood = !this.begood

      await this.axios.get('toggle_user_like_to_product/',
        { params: { user_id: this.$store.state.userId, product_id: this.id } })
        .then((response) => {
          console.log(response);

        })
        .catch(function (error) {
          console.log(error);
        });
    },
    async changeStar() {
      if (this.bestar) {
        this.shop_stars -= 1
      } else {
        this.shop_stars += 1
      }
      this.bestar = !this.bestar

      await this.axios.get('toggle_user_star_to_shop/',
        { params: { user_id: this.$store.state.userId, business_id: this.product.fields.product_business } })
        .then((response) => {
          console.log(response);

        })
        .catch(function (error) {
          console.log(error);
        });
    },
    async initUserToProduct() {
      await this.axios.get('get_user_like_to_product/',
        { params: { user_id: this.$store.state.userId, product_id: this.id } })
        .then((response) => {
          console.log(response);
          //this.list = response.data.list
          if (response.data.good === 'good') {
            this.begood = true
          } else {
            this.begood = false
          }

        })
        .catch(function (error) {
          console.log(error);
        });

      await this.axios.get('get_user_star_to_shop/',
        { params: { user_id: this.$store.state.userId, business_id: this.product.fields.product_business } })
        .then((response) => {
          console.log(response);
          //this.list = response.data.list
          if (response.data.star === 'star') {
            this.bestar = true
          } else {
            this.bestar = false
          }

        })
        .catch(function (error) {
          console.log(error);
        });
    },


    async getProduct(x1) {
      await this.axios.get('fetch_product/', { params: { product_id: x1 } })
        .then((response) => {
          console.log(response);
          this.product = response.data.product
        })
        .catch(function (error) {
          console.log(error);

        });
    },
    async handleAddCart() {
      this.$router.push({ path: '/business/business_edit_commodity/' + this.id })

    }
  },
  async created() {
    //初始化数据

    await this.getProduct(this.id)

    if (this.product.fields.product_status == '上架') {
      this.isOn = true
    } else {
      this.isOn = false
    }

  },

  async mounted() {
    //初始化数据

    await this.getProduct(this.id)


  }
}
</script>
<!-- scoped属性表示只对当前组件有效，不影响其他组件 -->
<style scoped>
.word {
  font-size: 32px;
}

.fillheart {
  color: red;

  font-size: 32px;
}

.heart {
  color: gray;

  font-size: 32px;
}

.fillstar {
  color: yellow;

  font-size: 32px;
}

.star {
  color: gray;

  font-size: 32px;
}

.product {
  margin: 32px;
  padding: 32px;
  background: #fff;
  border: 1px solid #dddee1;
  border-radius: 10px;
  overflow: hidden;
}

.product-image {
  width: 50%;
  height: 550px;
  float: left;
  text-align: center;
}

.product-image img {
  height: 100%;
}

.product-info {
  width: 50%;
  padding: 150px 0 250px;
  height: 150px;
  float: left;
  text-align: center;
}

.product-cost {
  color: #f2352e;
  margin: 8px 0;
}

.product-add-cart {
  display: inline-block;
  padding: 8px 64px;
  margin: 8px 0;
  background: #2d8cf0;
  color: #fff;
  border-radius: 4px;
  cursor: pointer;
}

.product-desc {
  background: #fff;
  margin: 32px;
  padding: 32px;
  border: 1px solid #dddee1;
  border-radius: 10px;
  text-align: center;
}

.product-desc img {
  display: block;
  width: 50%;
  margin: 32px auto;
  padding: 32px;
  border-bottom: 1px solid #dddee1;
}
</style>