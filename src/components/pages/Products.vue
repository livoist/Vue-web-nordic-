<template>
  <div class="font-weight-bold">
    <loading :active.sync="isLoading"></loading>
    <div class="d-flex justify-content-end mt-2">
      <div class>
        <button class="btn btn-outline-primary mr-3" @click="openModal(true)"><i class="fas fa-plus-square mr-2"></i>Build a new product</button>
      </div>
    </div>
      <div class="form-row">
        <div class="col-12">
          <table class="table mt-4">
      <thead class="bg-dark text-white">
        <th width="120">Types</th>
        <th width="180">Name</th>
        <th width="auto" class="text-right">Original price</th>
        <th width="auto" class="text-right">Price</th>
        <th width="auto">Enable</th>
        <th>edit</th>
      </thead>
      <tbody>
        <tr v-for="(item) in products" :key="item.id">
          <td>{{ item.category }}</td>
          <td>{{ item.title }}</td>
          <td class="text-right">{{item.origin_price | currency}}</td>
          <td class="text-right">{{item.price | currency}}</td>
          <td>
            <span v-if="item.is_enabled" class="text-success"><i class="fas fa-lock-open"></i></span>
            <span v-else class="text-danger"><i class="fas fa-lock"></i></span>
          </td>
          <td>
            <div class="btn btn-outline-primary btn-sm my-1" @click="openModal(false ,item)"><i class="fas fa-pencil-alt"></i></div>
            <div class="btn btn-outline-primary btn-sm my-1" @click="deleteProduct(item)"><i class="fas fa-trash"></i></div>
          </td>
        </tr>
      </tbody>
    </table>
        </div>
      </div>
    <!-- 切換頁面Bar -->
    <nav aria-label="Page navigation example">
      <ul class="pagination">
        <li class="page-item" :class="{'disabled': !pagination.has_pre}">
          <a
            class="page-link"
            href="#"
            aria-label="Previous"
            @click.prevent="getProducts(pagination.current_page - 1)"
          >
            <span aria-hidden="true">&laquo;</span>
            <span class="sr-only">Previous</span>
          </a>
          <!-- 點擊事件(取消預設行為)，取得產品頁面的所在頁數-1(回上頁) -->
        </li>
        <li
          class="page-item"
          v-for="page in pagination.total_pages"
          :key="page"
          :class="{'active':pagination.current_page === page}"
        >
          <a class="page-link" href="#" @click.prevent="getProducts(page)">{{ page }}</a>
        </li>
        <!-- 點擊事件(取消預設行為)，取得產品頁面的所在頁數，並且顯示出來 -->
        <!-- 條件切換(active)，若現在的頁面等於取得的頁面則顯示active -->
        <li class="page-item" :class="{'disabled': !pagination.has_next}">
          <a
            class="page-link"
            href="#"
            aria-label="Next"
            @click.prevent="getProducts(pagination.current_page + 1)"
          >
            <span aria-hidden="true">&raquo;</span>
            <span class="sr-only">Next</span>
          </a>
          <!-- 點擊事件(取消預設行為)，取得產品頁面的所在頁數+1(往下頁) -->
        </li>
      </ul>
    </nav>

    <!-- productModal -->
    <div
      class="modal fade"
      id="productModal"
      tabindex="-1"
      role="dialog"
      aria-labelledby="exampleModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog modal-lg" role="document">
        <div class="modal-content border-0">
          <div class="modal-header bg-dark text-white">
            <h5 class="modal-title" id="exampleModalLabel">
              <span>New product</span>
            </h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <div class="row">
              <div class="col-sm-4">
                <div class="form-group">
                  <label for="image">Image URL</label>
                  <input
                    v-model="tempProduct.imageUrl"
                    type="text"
                    class="form-control"
                    id="image"
                    placeholder="請輸入圖片連結"
                  >
                </div>
                <div class="form-group">
                  <label for="customFile">
                    or upload image
                    <i class="fas fa-sync fa-spin" v-if="status.fileUploading"></i>
                  </label>
                  <input
                    type="file"
                    id="customFile"
                    class="form-control"
                    ref="files"
                    @change="uploadFile"
                  >
                </div>
                <img
                  img="https://images.unsplash.com/photo-1483985988355-763728e1935b?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=828346ed697837ce808cae68d3ddc3cf&auto=format&fit=crop&w=1350&q=80"
                  class="img-fluid"
                  :src="tempProduct.imageUrl"
                  alt
                >
              </div>
              <div class="col-sm-8">
                <div class="form-group">
                  <label for="title">Title</label>
                  <input
                    v-model="tempProduct.title"
                    type="text"
                    class="form-control"
                    id="title"
                    placeholder="請輸入標題"
                  >
                </div>

                <div class="form-row">
                  <div class="form-group col-md-6">
                    <label for="category">Type</label>
                    <input
                      v-model="tempProduct.category"
                      type="text"
                      class="form-control"
                      id="category"
                      placeholder="請輸入分類"
                    >
                  </div>
                  <div class="form-group col-md-6">
                    <label for="num">unit</label>
                    <input
                      v-model="tempProduct.num"
                      type="text"
                      class="form-control"
                      id="num"
                      placeholder="請輸入數量"
                    >
                  </div>
                </div>

                <div class="form-row">
                  <div class="form-group col-md-6">
                    <label for="origin_price">OriginPrice</label>
                    <input
                      v-model="tempProduct.origin_price"
                      type="number"
                      class="form-control"
                      id="origin_price"
                      placeholder="請輸入原價"
                    >
                  </div>
                  <div class="form-group col-md-6">
                    <label for="price">Price</label>
                    <input
                      v-model="tempProduct.price"
                      type="number"
                      class="form-control"
                      id="price"
                      placeholder="請輸入售價"
                    >
                  </div>
                </div>
                <hr>

                <div class="form-group">
                  <label for="description">dascription</label>
                  <textarea
                    v-model="tempProduct.description"
                    type="text"
                    class="form-control"
                    id="description"
                    placeholder="請輸入產品描述"
                  ></textarea>
                </div>
                <div class="form-group">
                  <label for="content">content</label>
                  <textarea
                    v-model="tempProduct.content"
                    type="text"
                    class="form-control"
                    id="content"
                    placeholder="請輸入產品說明內容"
                  ></textarea>
                </div>
                <div class="form-group">
                  <div class="form-check">
                    <input
                      v-model="tempProduct.is_enabled"
                      :true-value="1"
                      :false-value="0"
                      class="form-check-input"
                      type="checkbox"
                      id="is_enabled"
                    >
                    <label class="form-check-label" for="is_enabled">enable</label>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-outline-secondary" data-dismiss="modal">cancel</button>
            <button type="button" class="btn btn-primary" @click="updateProduct">comfirm</button>
          </div>
        </div>
      </div>
    </div>
    <div
      class="modal fade"
      id="delProductModal"
      tabindex="-1"
      role="dialog"
      aria-labelledby="exampleModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog" role="document">
        <div class="modal-content border-0">
          <div class="modal-header bg-danger text-white">
            <h5 class="modal-title" id="exampleModalLabel">
              <span>delete</span>
            </h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            comfirm delete
            <strong class="text-danger">{{ tempProduct.title }}</strong> Merchandise (cannot be recovered after deletion)。
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-outline-secondary" data-dismiss="modal">cancel</button>
            <button type="button" class="btn btn-danger">comfirm delete</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import $ from "jquery";

export default {
  data() {
    return {
      products: [],
      //起始資料值
      pagination: {},
      tempProduct: {},
      //更新的新物件儲存地方
      isNew: false,
      //判斷是否為新的物件或是編輯舊的物件
      isLoading: false,
      //loding icon判斷條件
      status: {
        fileUploading: false
      }
      //圖片載入時觸發條件
    };
  },
  methods: {
    getProducts(page = 1) {
      const api = `${process.env.APIPATH}/api/${
        process.env.CUSTOMPATH
      }/admin/products?page=${page}`;
      const vm = this;
      console.log(process.env.APIPATH, process.env.CUSTOMPATH);
      vm.isLoading = true;
      this.$http.get(api).then(response => {
        console.log(response.data);
        vm.isLoading = false;
        vm.products = response.data.products;
        vm.pagination = response.data.pagination;
      });
    },
    updateProduct() {
      let api = `${process.env.APIPATH}/api/${
        process.env.CUSTOMPATH
      }/admin/product`;
      let httpMethod = "post";
      const vm = this;
      if (!vm.isNew) {
        api = `${process.env.APIPATH}/api/${
          process.env.CUSTOMPATH
        }/admin/product/${vm.tempProduct.id}`;
        httpMethod = "put";
        //如果不是新的，進入編輯模式，改為put修改，並且綁上ID確認為哪個物件
        //更新API的方法，get為取得，post為建立，put為修改(後補上ID確認修改項目)
      }
      this.$http[httpMethod](api, { data: vm.tempProduct }).then(response => {
        console.log(response.data);
        if (response.data.success) {
          $("#productModal").modal("hide");
          vm.getProducts();
        } else {
          $("#productModal").modal("hide");
          vm.getProducts();
          console.log("新增失敗");
        }
      });
    },
    deleteProduct(item) {
      let api = `${process.env.APIPATH}/api/${
        process.env.CUSTOMPATH
      }/admin/product/${item.id}`;
      const vm = this;
      this.$http.delete(api).then(response => {
        console.log(response.data);
      });
      vm.getProducts();
    },
    openModal(isNew, item) {
      if (isNew) {
        this.tempProduct = {};
        this.isNew = true;
      } else {
        this.tempProduct = Object.assign({}, item);
        this.isNew = false;
      }
      $("#productModal").modal("show");
    },
    uploadFile() {
      const uploadedFile = this.$refs.files.files[0];
      // 取出檔案
      const vm = this;
      const formData = new FormData();
      //建立formData物件
      formData.append("file-to-upload", uploadedFile);
      // 把(上傳的位置,上傳的圖片)圖片上傳
      const url = `${process.env.APIPATH}/api/${
        process.env.CUSTOMPATH
      }/admin/upload`;
      // 判斷上傳圖片是否顯示(預設false，寫入中為true，)
      vm.status.fileUploading = true;
      this.$http
        .post(url, formData, {
          headers: {
            "Content-Type": "multipart/form-data"
          }
        })
        .then(response => {

          if (response.data.success) {
            // 判斷上傳icon是否顯示(預設false，寫入完成為false)
            vm.status.fileUploading = false;
            //vm.tempProduct.imageUrl = response.data.imageUrl
            //console.log(vm.tempProduct)
            vm.$set(vm.tempProduct, "imageUrl", response.data.imageUrl);
            // 因為沒有雙向綁定，所以用vue-set強制寫入(寫入的物件,寫入的屬性,寫入資料的路徑)
          } else {
            vm.status.fileUploading = false;
          }
        });
    },
    frontPage() {
      this.$router.push("/HomePage");
    }
  },
  created() {
    this.getProducts();
    
  }
};
</script>