<template>
  <div class="py-2 px-2 height-100pc user-center" elevation="1">
    <v-row no-gutters justify="space-between">
      <v-col class="ml-3">
        <v-toolbar
            dark
            color="blue darken-3"
            class="mb-1"
        >
          <v-spacer></v-spacer>
          <v-toolbar-title>
            <h4>Add Product</h4>
          </v-toolbar-title>
          <v-spacer></v-spacer>
        </v-toolbar>
        <v-card>
          <v-card flat class="pa-3">
            <v-row>
              <v-col md="6">
                <v-form ref="form">
                  <!-- 字段渲染 -->
                  <v-container>
                    <v-row>
                      <v-col md="12">
                        <v-text-field
                            label="Product Name"
                            :rules="formEmptyRule"
                            v-model="productForm.product_name"
                            clearable
                        />
                      </v-col>
                    </v-row>
                    <v-row>
                      <v-col md="12">
                        <v-text-field
                            label="Product Brand"
                            :rules="formEmptyRule"
                            v-model="productForm.product_brand"
                            clearable
                        />
                      </v-col>
                    </v-row>
                    <v-row>
                      <v-col md="12">
                        <v-text-field
                            label="Product Color"
                            :rules="formEmptyRule"
                            v-model="productForm.product_color"
                            clearable
                        />
                      </v-col>
                    </v-row>

                    <v-row>
                      <v-col md="12">
                        <v-text-field
                            label="Product Price"
                            :rules="costRule"
                            v-model="productForm.product_cost"
                            clearable
                        />
                      </v-col>
                    </v-row>
                    <!--
                    <v-row>
                      <v-col md="12">
                        <v-text-field
                            label="商品库存"
                            :rules="formEmptyRule"
                            clearable
                        />
                      </v-col>
                    </v-row>
                    !-->
                  </v-container>
                </v-form>
              </v-col>
              <v-col md="6">
                <el-form ref="form" :model="productForm" label-width="80px">
                  <el-form-item label="Main Image">
                    <input type="file" @change="getImageFile1" id="img">
                    <img :src="showPic1" height="150px" width="150px"/>
                  </el-form-item>

                </el-form>
                <el-form ref="form" :model="productForm" label-width="80px">
                  <el-form-item label="Detail Image">
                    <input type="file" @change="getImageFile2" id="img">
                    <img :src="showPic2" height="150px" width="150px"/>
                  </el-form-item>

                </el-form>
              </v-col>
            </v-row>
            <v-divider/>
            <v-row justify="end">
              <v-col md="4" align="end">
                <v-btn
                    small
                    tile
                    class="mr-2"
                    color="primary"
                    @click="saveProduct"
                >
                  Add
                </v-btn>
              </v-col>
            </v-row>
          </v-card>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>


<script>
import {required} from "@/utils/widget";

export default {
  name: "business_add_commodity",
  data() {
    return {
      productForm: {
        product_name: "", // Product Name
        product_color: "", // Product Color
        // TODO: Product Cost/Product Price
        product_brand: "", // Product Brand
        product_cost: "", // Product Price
        product_image1: 'http://127.0.0.1:8001/media/avatar/default_pic.jpg', // 主页图片
        product_image2: 'http://127.0.0.1:8001/media/avatar/default_pic.jpg'
        // TODO：细节图片
      },
      showPic1:'http://127.0.0.1:8001/media/avatar/default_pic.jpg',
      showPic2:'http://127.0.0.1:8001/media/avatar/default_pic.jpg',
      formEmptyRule: [required("This field")],
      costRule:[required("This field"),
          function (v) {
            return  /^(([1-9]{1}\d*)|(0{1}))(\.\d{1,2})?$/.test(v) || `Amount can have at most two decimal places`;
          }
      ]

    };
  },

  methods: {
    // TODO：保存商品信息
    async saveProduct() {
      let formData = new FormData();
        formData.append('product_image1', this.productForm.product_image1);
        formData.append('product_image2', this.productForm.product_image2);

        const json = JSON.stringify(this.productForm);
        const blob = new Blob([json], {
          type: 'application/json'
        });

        formData.append('forms',blob)

        const json2 = JSON.stringify((this.$store.state.userId));
        const blob2 = new Blob([json2], {
          type: 'application/json'
        });

        formData.append('user_id',blob2)
        console.log("blob2")
        console.log(blob2)
        console.log(JSON.stringify((this.$store.state.user_id)))

        var configs = {
          headers:{'Content-Type':'multipart/form-data'}
        }

        await this.axios.post('add_product/',formData, configs).then(response => {
          console.log(formData.get('forms'))
        if(response.data.msg==="success"){
          //this.showPic1 = 'http://127.0.0.1:8000/media/'+response.data.image1
          //this.showPic2 = 'http://127.0.0.1:8000/media/'+response.data.image2
          console.log(response)




          window.location.href = '/#/business/commodity';
          window.alert("Product successfully listed")

          }
        console.log(response)
        })
        .catch(function (error) {
            console.log(error);
        });

    },
    getImageFile1:function(e) {
       this.productForm.product_image1 = e.target.files[0];
      let img = new FileReader();
      img.readAsDataURL(this.productForm.product_image1);
      console.log("img:",img)
      img.onload = ({ target }) => {
        this.showPic1 = target.result; //将img转化为二进制数据
        console.log("target:",target)
      };
    },
    getImageFile2:function(e) {
       this.productForm.product_image2 = e.target.files[0];
      let img = new FileReader();
      img.readAsDataURL(this.productForm.product_image2);
      console.log("img:",img)
      img.onload = ({ target }) => {
        this.showPic2 = target.result; //将img转化为二进制数据
        console.log("target:",target)
      };
    },
    async onSubmit(){
      console.log("good");
    },
    clear(){
      this.productForm={
        product_name: "", // Product Name
        product_color: "", // Product Color
        // TODO: Product Cost/Product Price
        product_brand: "", // Product Brand
        product_cost: "", // Product Price
        product_image1: "", // 主页图片
        product_image2: ""
        // TODO：细节图片
      },
      this.showPic1="",
      this.showPic2=""
    }
  },
};
</script>

<style>

</style>
