<template>
  <div class="container">
    <div class="row">
      <div class="col-3">
        <div class="form-group">
          <button class="btn btn-primary" @click="clearFilter">Очистить фильтры</button>
        </div>
        <div class="form-group">
          <p>Страна</p>
          <v-select :options="countryOptions" v-model="countryValue" />
        </div>
        <div class="form-group">  
          <p>Тип</p>
          <v-select multiple :options="typeOptions" v-model="typeValue"/>
        </div>
        <div class="form-group">
          <b-button v-b-toggle.collapse-1 variant="success btn-block">Звезды</b-button>
          <b-collapse id="collapse-1" class="mt-2">
            <div>
              <input type="checkbox" class="mr-2" value="1" v-model="stars">
              <label for="">1 звезда</label>
            </div>
            <div>
              <input type="checkbox" class="mr-2" value="2" v-model="stars">
              <label for="">2 звезды</label>
            </div>
            <div>
              <input type="checkbox" class="mr-2" value="3" v-model="stars">
              <label for="">3 звезды</label>
            </div>
            <div>
              <input type="checkbox" class="mr-2" value="4" v-model="stars">
              <label for="">4 звезды</label>
            </div>
            <div>
              <input type="checkbox" class="mr-2" value="5" v-model="stars">
              <label for="">5 звезд</label>
            </div>
          </b-collapse>
        </div>
        <div class="form-group">
          <p>Количество отзывов от</p>
          <input type="text" @input="validateReviews($event)" class="form-control" v-model="cntReviews">
        </div>
        <div class="form-group">
          <p>Цена до</p>
          <div class="price-container">
            <div>0</div>
            <range-slider :max="10000" :value="price" :getPrice="getPrice"/>
            <div>100</div>
          </div>  
        </div>
        <div class="form-group">
          <button class="btn btn-primary" @click="filter">Применить фильтры</button>
        </div>
      </div>
      <div class="col-9">
        <div class="hotels" v-if="hotels.length > 0">
          <div class="hotels-header">
            <div class="row">
              <div class="col-3">
                <strong>Название</strong>
              </div>
              <div class="col-9">
                <strong>Описание</strong>
              </div>
            </div>
          </div>
          <div class="hotels-body">
            <div class="hotel-item" v-for="hotel in hotels" :key="hotel.name">
              <div class="row">
                <div class="col-3">{{ hotel.name }}</div>
                <div class="col-6">{{ hotel.description }}</div>
                <div class="col-3"><button class="btn btn-primary">Забронировать</button></div>
              </div>
            </div>
          </div>
        </div>
        <div v-else>
          Записей не найдено
        </div>
        <nav aria-label="Page navigation example" v-if="hotels.length > 0">
          <ul class="pagination">
            <li class="page-item">
              <button class="page-link" :disabled="isDisabledPrev" @click="getPrevPage()">Previous</button>
              </li>
            <li class="page-item" v-for="i in cntPage" :key="i">
              <button v-if="filterHotels.length > 0" class="page-link" @click="getHotels(filterHotels, i)" :class="{'active': page == i}">{{i}}</button>
              <button v-else class="page-link" @click="getHotels(allHotels, i)" :class="{'active': page == i}">{{i}}</button>
            </li>
            <li class="page-item">
              <button class="page-link" :disabled="isDisabledNext" @click="getNextPage()">Next</button>
            </li>
          </ul>
        </nav>
      </div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src

import vSelect from 'vue-select';
import RangeSlider from '@/components/RangeSlider'
import data from '@/data/hotels.json'

export default {
  name: 'Home',
  data() {
    return {
      typeOptions: [],
      countryOptions: [],
      typeValue: '',
      countryValue: '',
      cntReviews : '',
      prevCntReviews: '',
      stars: [],
      price: 0,
      allHotels: [],
      hotels: [],
      filterHotels: [],
      page: 1,
      hotelsPerPage: 3,
      cntPage: 1,
      isDisabledPrev: true,
      isDisabledNext: false,
    }
  },
  components: {
    vSelect,
    RangeSlider
  },
  methods: {
    getPrice: function(price) {
      this.price = price
    },

    getCntPage: function(hotels) {
      this.cntPage = Math.ceil(hotels.length / 3);
    },

    getHotels: function(hotels, page) {
      this.getCntPage(hotels);
      this.page = page;
      let tmp = [];
      let endPos = this.hotelsPerPage * page;
      let startPos = endPos - this.hotelsPerPage;
      for(let i = startPos; i < endPos; i++ ) {
        if(hotels[i]) {
          tmp.push(hotels[i])
        }
      }
      this.hotels = tmp;
      this.isDisabledPrev = page == 1 ? true : false;
      this.isDisabledNext = page == Math.ceil(this.allHotels.length / 3) ? true : false;
      if(this.typeValue || this.countryValue || this.cntReviews || this.stars.length || this.price) {
        this.isDisabledNext = page == Math.ceil(this.filterHotels.length / 3) ? true : false;
      }  
    },
    getPrevPage: function() {
      this.page--;
      if(this.typeValue || this.countryValue || this.cntReviews || this.stars.length || this.price) {
        this.getHotels(this.filterHotels, this.page)
      } else {
        this.getHotels(this.allHotels, this.page)
      }
    },

    getNextPage: function(page) {
      this.page++;
      if(this.typeValue || this.countryValue || this.cntReviews || this.stars.length || this.price) {
        this.getHotels(this.filterHotels, this.page)
      } else {
        this.getHotels(this.allHotels, this.page)
      }
    },

    validateReviews: function(event) {
      if(!parseInt(event.data) && event.inputType != 'deleteContentBackward') {
        this.cntReviews = this.prevCntReviews;
      } else {
        this.prevCntReviews = this.cntReviews;
      }
    },

    filter: function() {
      let result = this.allHotels;
      if(this.countryValue) {
        result = result.filter((item) => {
          if(item.country == this.countryValue) {
            return item;
          }
        })
      }  
      if(this.typeValue) {
        result = result.filter((item) => {
          if(this.typeValue.indexOf(item.type) != -1) {
            return item;
          }
        })
      }

      if(this.stars.length) {
        result = result.filter((item) => {
          if(this.stars.indexOf(item.stars.toString()) != -1) {
            return item;
          }
        })
      }

      if(this.cntReviews) {
        result = result.filter((item) => {
          if(item.reviews_amount >= this.cntReviews) {
            return item;
          }
        })
      }

      if(this.price) {
        result = result.filter((item) => {
          if(item.min_price <= this.price) {
              return item;
          }
        })
      }
      
      this.page = 1;
      this.filterHotels = result;
      this.getHotels(this.filterHotels, this.page);
    },

    clearFilter: function() {
      this.typeValue = '';
      this.countryValue = '';
      this.cntReviews = '';
      this.stars = [];
      this.price = 0,
      this.filterHotels = [];
      this.page = 1;
      this.getPrice(0)
      this.getHotels(this.allHotels, this.page);
    }
  },
  mounted: function() {
    let countryOptions = [];
    data.hotels.map((item) => {
      countryOptions.push(item.country)
    })
    countryOptions = new Set(countryOptions);
    this.countryOptions = [...countryOptions];

    let typeOptions = [];
    data.hotels.map((item) => {
      typeOptions.push(item.type)
    })
    typeOptions = new Set(typeOptions);
    this.typeOptions = [...typeOptions];

    this.allHotels = data.hotels
    this.getCntPage(this.allHotels);
    this.getHotels(this.allHotels, this.page);
  },
  updated: function () {
    
  },
}
</script>
