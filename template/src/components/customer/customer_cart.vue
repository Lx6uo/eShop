<template>
    <div class="cart">
        <div class="cart-header">
            <div class="cart-header-title">Shopping List</div>
            <div class="cart-header-main">
                <div class="cart-info">Product Information</div>
                <div class="cart-price">Unit Price</div>
                <div class="cart-count">Quantity</div>
                <div class="cart-cost">Subtotal</div>
                <div class="cart-delete">Delete</div>
            </div>
        </div>
        <div class="cart-content">
            <!-- 列表显示购物清单 -->
            <div class="cart-content-main" v-for="(item, index)  in userCartList" :key="item.index">
                <div class="cart-info">
                    <img :src="`http://127.0.0.1:8001/media/${productDictList[item.fields.product_id].fields.product_image}`" alt=""
                    height="150px" width="150px">
                    <span>{{productDictList[item.fields.product_id].fields.product_name}}</span>
                </div>
                <div class="cart-price">
                    ￥ {{productDictList[item.fields.product_id].fields.product_cost}}
                </div>
                <div class="cart-count">
                    <span class="cart-control-minus"
                          @click="handleCount(index, -1)">-</span>
                    {{item.fields.num}}
                    <span class="cart-control-add"
                          @click="handleCount(index, 1)">+</span>
                </div>
                <div class="cart-cost">
                    ￥ {{productDictList[item.fields.product_id].fields.product_cost * (item.fields.num-0.0)}}
                </div>
                <div class="cart-delete">
                    <span class="cart-control-delete"
                          @click="handleDelete(index)">Delete</span>
                </div>
            </div>
            <div class="cart-empty" v-if="!userCartList.length">Shopping Cart is Empty</div>
        </div>
        <div class="cart-promotion" v-show="userCartList.length">
            <span>Use Coupon</span>
            <input type="text" v-model="promotionCode">
            <span class="cart-control-promotion"
                  @click="handleCheckCode">Verify</span>
        </div>
        <div class="cart-footer" v-show="userCartList.length">
            <div class="cart-footer-desc">
                Total Items: <span>{{countAll}}</span>
            </div>
            <div class="cart-footer-desc">
                Total Amount: <span>{{costAll - promotion}}</span>
                <br>
                <template v-if="promotion">
                    (Discount: <span>￥ {{promotion}} </span>)
                </template>
            </div>
            <div class="cart-footer-desc">
                <div class="cart-control-order"
                     @click="handleOrder">Checkout Now</div>
            </div>
        </div>
    </div>
</template>

<script>
    //import product_data from './product';
    export default {
        name: "customer_cart",
        data(){
            return {
                promotion: 0,
                promotionCode: '',
                userCartList:[],  //从cart表取到的原数据
                cartProductList:[],  //cart表里包含的product表的原数据
                productList: [],
                productDictList:[]  //product_id -> product
            }
        },
        computed: {


            //Total number of items in shopping cart
            countAll(){
                let count = 0;
                this.userCartList.forEach(item => {
                    count += item.fields.num;
                });
                return count;
            },//Total cost of items in shopping cart
            costAll(){
                let cost = 0;
                this.userCartList.forEach(item => {
                    cost += this.productDictList[item.fields.product_id].fields.product_cost * (item.fields.num-0.0);
                    console.log(this.productDictList[item.fields.product_id].fields.product_cost * (item.fields.num-0.0));
                    console.log((item.fields.num-0.0));
                    console.log(this.productDictList[item.fields.product_id].fields.product_cost);
                });
                return cost;
            }

        },
        methods: {
            //Notify Vuex, complete order

            async handleOrder(){
              let m=[]
              await this.axios.get('fetch_userInfo',{params:{user_id: this.$store.state.userId}})
              .then((response) => {
                  console.log(response);
                  m=response


              })
              .catch(function (error) {
                  console.log(error);

              });
              console.log(m)

              if(m.data.user_mobile===""){
                console.log("null")
                window.alert("Please complete contact information")
                this.$router.push({path: '/customerPerson'})
                return
              }

              if((m.data.user_province==="")||(m.data.user_city==="")||(m.data.user_area==="")){
                window.alert("Please complete address information")
                this.$router.push({path: '/customerPerson'})
                return
              }


              if(m.data.user_address===""){
                window.alert("Please complete detailed address")
                this.$router.push({path: '/customerPerson'})
                return
              }

              console.log('full')

              await this.axios.get('add_order/',
                  {params:{user_id: this.$store.state.userId}})
              .then((response) => {
                  console.log(response);
                  //this.list = response.data.list

              })
              .catch(function (error) {
                  console.log(error);

              });
              console.log("this end");

              var i = this.userCartList.length
              for (; i>1; i--) {
                await this.axios.get('delete_item_in_cart/',
                  {params:{user_id: this.$store.state.userId, product_id: this.userCartList[i-1].fields.product_id}})
              .then((response) => {
                  console.log(response);
                  //this.list = response.data.list
                  //window.alert("Delete successful")
              })
              .catch(function (error) {
                  console.log(error);
              });
              }
              await this.handleDelete(0)



              window.alert('Purchase successful');


            },





            //Verify promotion code, use ccylovehs as the coupon string
            handleCheckCode(){
                if(this.promotionCode === ''){
                    window.alert('Please enter promotion code');
                    return;
                }
                if(this.promotionCode !== 'ccylovehs'){
                    window.alert('Promotion code is incorrect');
                    return;
                }
                this.promotion = 500;
            },

            async handleCount(index,count) {
              await this.axios.get('edit_num_in_cart/',
                  {
                    params: {
                      user_id: this.$store.state.userId,
                      product_id: this.userCartList[index].fields.product_id,
                      num: count
                    }
                  })
                  .then((response) => {
                    console.log("!!!!!!!!!!!!!!!!!----------");
                    console.log(response);
                    //this.list = response.data.list
                    console.log("!!!!!!!!!!!!!!!!!!!!!!!!!!!");
                  })
                  .catch(function (error) {
                    console.log(error);
                  });
              await this.cartList(this.$store.state.userId)
            },

            //Delete product by finding product id according to index
            async handleDelete(index){
              await this.axios.get('delete_item_in_cart/',
                  {params:{user_id: this.$store.state.userId, product_id: this.userCartList[index].fields.product_id}})
              .then((response) => {
                  console.log(response);
                  //this.list = response.data.list
                  //window.alert("Delete successful")
              })
              .catch(function (error) {
                  console.log(error);
              });

              await this.cartList(this.$store.state.userId)
              //this.$store.commit('deleteCart', this.cartList[index].id);
              //const index = state.cartList.findIndex(item => item.id === id);
              //state.cartList.splice(index, 1)
            },
            //Shopping cart data
            async cartList(x1){
              await this.axios.get('show_cart/',{params:{user_id: x1}})
              .then((response) => {
                  console.log(response);
                  this.userCartList = response.data.list
              })
              .catch(function (error) {
                  console.log(error);
              });

              await this.cartProductsList(this.$store.state.userId)

              this.productDictList = {};
              await this.cartProductList.forEach(item => {
                    this.productDictList[item.pk] = item;
                });
            },
          async cartProductsList(x1){
              await this.axios.get('cartProductsList/',{params:{user_id: x1}})
              .then((response) => {
                  console.log(response);
                  this.cartProductList = response.data.list
              })
              .catch(function (error) {
                  console.log(error);
              });

            }
        },

        async created(){
          console.log("cart-create");
          console.log(this.$store.state.userId)
          await this.cartList(this.$store.state.userId)
          console.log("cart-create-success");
          await this.cartProductsList(this.$store.state.userId)
          console.log(this.productDictList);
          console.log(this.productDictList[2].fields.product_cost);


        }
    }
</script>

<style scoped>
    .cart{
        margin: 32px;
        background: #fff;
        border: 1px solid #dddee1;
        border-radius: 10px;
    }
    .cart-header-title{
        padding: 16px 32px;
        border-bottom: 1px solid #dddee1;
        border-radius: 10px 10px 0 0;
        background: #f8f8f9;
    }
    .cart-header-main{
        padding: 8px 32px;
        overflow: hidden;
        border-bottom: 1px solid #dddee1;
        background: #eee;
        overflow: hidden;
    }
    .cart-empty{
        text-align: center;
        padding: 32px;
    }
    .cart-header-main div{
        text-align: center;
        float: left;
        font-size: 14px;
    }
    div.cart-info{
        width: 60%;
        text-align: left;
    }
    .cart-price{
        width: 10%;
    }
    .cart-count{
        width: 10%;
    }
    .cart-cost{
        width: 10%;
    }
    .cart-delete {
        width: 10%;
    }
    .cart-content-main{
        padding: 0 32px;
        height: 60px;
        line-height: 60px;
        text-align: center;
        border-bottom: 1px dashed #e9eaec;
        overflow: hidden;
    }
    .cart-content-main div{
        float: left;
    }
    .cart-content-main img{
        width: 40px;
        height: 40px;
        position: relative;
        top: 10px;
    }
    .cart-control-minus,
    .cart-control-add{
        display: inline-block;
        margin: 0 4px;
        width: 24px;
        height: 24px;
        line-height: 22px;
        text-align: center;
        background: #f8f8f9;
        border-radius: 50%;
        box-shadow: 0 1px 1px rgba(0,0,0,.2);
        cursor: pointer;
    }
    .cart-control-delete{
        cursor: pointer;
        color: #2d8cf0;
    }
    .cart-promotion{
        padding: 16px 32px;
    }
    .cart-control-promotion,
    .cart-control-order{
        display: inline-block;
        padding: 8px 32px;
        border-radius: 6px;
        background: #2d8cf0;
        color: #fff;
        cursor: pointer;
    }
    .cart-control-promotion{
        padding: 2px 6px;
        font-size: 12px;
        border-radius: 3px;
    }
    .cart-footer{
        padding: 32px;
        text-align: right;
    }
    .cart-footer-desc{
        display: inline-block;
        padding: 0 16px;
    }
    .cart-footer-desc span{
        color: #f2352e;
        font-size: 20px;
    }
</style>