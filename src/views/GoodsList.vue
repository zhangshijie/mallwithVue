<template>
  <div>
      <nav-header></nav-header>
      <nav-bread>
        <span slot="bread">Goods</span>
      </nav-bread>
      <div class="accessory-result-page accessory-page">
        <div class="container">
          <div class="filter-nav">
            <span class="sortby">Sort by:</span>
            <a href="javascript:void(0)" class="default cur">Default</a>
            <a href="javascript:void(0)" @click="sortGoods"  class="price">Price 
              <svg class="icon icon-arrow-short" v-bind:class="{'sort-up':sortFlag}">
                <use xlink:href="./../static/svg.svg#icon-arrow-short"></use>
              </svg>
            </a>
            <a href="javascript:void(0)" class="filterby stopPop" @click="showFilterPopShow">Filter by</a>
          </div>
          <div class="accessory-result">
            <!-- filter -->
            <div class="filter stopPop" id="filter" v-bind:class="{'filterby-show': filterBy }">
              <dl class="filter-price">
                <dt>Price:</dt>
                <dd><a href="javascript:void(0)" v-bind:class="{ 'cur': priceChecked == 'all' }"  @click="priceChecked = 'all' ">All</a></dd>
                <dd v-for="(price,index) in priceFilter">
                  <a href="javascript:void(0)" v-bind:class=" {'cur':priceChecked == index}" @click="setPriceFilter(index)">{{ price.startPrice }} -  {{ price.endPrice }} </a>
                </dd>
              </dl>
            </div>

            <!-- search result accessories list -->
            <div class="accessory-list-wrap">
              <div class="accessory-list col-4">
                <ul>
                  <li v-for="(item, index) in goodsList">
                    <div class="pic">
                      <a href="#"><img v-lazy="'/static/'+item.productImage" alt=""></a>
                    </div>
                    <div class="main">
                      <div class="name">{{ item.prodcutName }}</div>
                      <div class="price">{{ item.salePrice }}</div>
                      <div class="btn-area">
                        <a href="javascript:;" class="btn btn-cart" @click= "addCart(item.productId)">加入购物车</a>
                      </div>
                    </div>
                  </li>
                </ul>
                <div v-infinite-scroll="loadMore" infinite-scroll-disabled="busy" infinite-scroll-distance="10">
                  <img src = "@/static/loading-svg/loading-spinning-bubbles" v-show="loading">
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="md-overlay" v-show="overLayFlag" @click="closePop"></div>
      <Modal v-bind:mdShow="mdShow" v-on:close="closeModal">
        <p slot="message">
          请先登录，否则不能加入购物车
        </p>
        <div slot="btnGroup">
          <a class="btn btn--m" href="javascript:void(0)" @click="closeModal">关闭</a>
        </div>
      </Modal>
        <Modal v-bind:mdShow="mdShowCart" v-on:close="closeModal">
        <p slot="message">
          加入购物车成功
        </p>
        <div slot="btnGroup">
          <a class="btn btn--m" href="javascript:void(0)" @click="mdShowCart = false">继续购物</a>
          <router-link class="btn btn--m" href="javascript:void(0)" to="/cart">查看购物车</router-link>
        </div>
      </Modal>
      
      <nav-footer></nav-footer>
  </div>
</template>

<script>
  import '@/assets/css/base.css'
  import '@/assets/css/product.css'
  import NavHeader from '@/components/Header.vue'
  import NavFooter from '@/components/NavFooter.vue'
  import NavBread from '@/components/NavBread.vue'
  import Modal from '@/components/Modal.vue'
  import axios from 'axios'

  export default {
    data() {
      return {
        goodsList:[],
        sortFlag: true,
        page: 1,
        pageSize:8,
        busy:false,
        loading:false,
        mdShow:false,
        mdShowCart:false,
        priceFilter: [
          {
            startPrice: '0.00',
            endPrice: '500.00'
          },
          {
            startPrice: '500.00',
            endPrice: '1000.00'
          },
          {
            startPrice: '1000.00',
            endPrice: '2000.00'
          },
        ],
        priceChecked:'all',
        filterBy:false,
        overLayFlag:false
      }
    },
    components: {
      NavHeader,
      NavFooter,
      NavBread,
      Modal
    },
    mounted() {
      axios.defaults.baseURL =  'http://localhost:3000';
      axios.defaults.withCredentials = true;
      this.getGoodslist(false);
    },
    methods: {
      getGoodslist(flag = false){
        let param = {
          page:this.page,
          pageSize:this.pageSize,
          sort:this.sortFlag?1:-1,
          priceLevel:this.priceChecked
        } 
        this.loading = true;
          axios({
            method:'get',
            url:'/goods/list',
            headers: {
              'Content-Type': 'application/x-www-form-urlencoded'
            },
            params:param
          }).then((result) => {
            let res = result.data;
            if(res.status == '0'){
              if (flag) {
                this.goodsList = this.goodsList.concat(res.result.list);
                if (res.result.count == 0) {
                  this.busy = true;
                }else{
                  this.busy = false;
                }
              }else{
                this.goodsList = res.result.list;
                this.busy = false;
              }
            }else{
              this.goodsList = [];
            }
            this.loading = false;
          })
      },
      showFilterPopShow(){
        this.filterBy = true;
        this.overLayFlag = true;
      },
      closePop(){
        this.filterBy = false;
        this.overLayFlag = false;
      },
      setPriceFilter(index){
        this.priceChecked = index;
        this.page = 1;
        this.getGoodslist(false);
        this.closePop();
      },
      sortGoods(){
        this.sortFlag = !this.sortFlag;
        this.page = 1;
        this.getGoodslist(false);
      },
      loadMore(){
        this.busy = true;
        setTimeout(() => {
          this.page++;
          this.getGoodslist(true);
        }, 500);
      },
      addCart(productId){
       axios({
            method:'get',
            url:'/goods/addCart',
            headers: {
              'Content-Type': 'application/x-www-form-urlencoded'
            },
            params:{productId:productId}
          }).then((result) => { 
            if (result.status == 0){
              // alert("加入成功");
              this.mdShowCart = true;
            }else{
              this.mdShowCart = true;
              // alert(result.msg);
            }
          });
      },
      closeModal(){
        this.mdShow = false;
      }
    }
  }
</script>



