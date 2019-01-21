<template>
<div>
  <TimeLine v-model="filters.year" @input="changeYear"/>
  <div class="filters">
    <div>
      <p>Дата</p>
      <div class="row-inp">
        <span>от</span>
        <input type="date" id="start" name="event-start"
          v-model="filters.date[0]" @input="passFilter">
      </div>
      <div class="row-inp">
        <span>до</span>  
        <input type="date" id="end" name="event-end"
          v-model="filters.date[1]"  @input="passFilter">
      </div>
    </div>
    <div>
        <p>Название</p>
        <input type="text" v-model="filters.title"  @change="passFilter"/>
    </div>
    <div v-if="lists.type.length > 0">
        <p>Тип</p>
        <multiselect v-model="filters.type" 
        :options="lists.type" 
        placeholder="Выберите тип"
        :show-labels="false"
         track-by="id"
         label="title"
         :multiple="true"
         @input="passFilter"
        ></multiselect>
    </div>
    <div v-if="lists.state.length > 0">
        <p>Статус</p>
         <multiselect v-model="filters.state" 
         :options="lists.state" 
         placeholder="Выберите состояние"
         :show-labels="false"
         track-by="id"
         label="title"
         :multiple="true"
          @input="passFilter"
         ></multiselect>
    </div>
    <div v-if="lists.series.length > 0">
        <p>Серия</p>
        <multiselect v-model="filters.series" 
        :options="lists.series"
        :show-labels="false"
        placeholder="Выберите серию"
        track-by="id"
        label="title"
        :multiple="true"
        @input="passFilter"
        >
        </multiselect>
    </div>
    <div>
       <p>Место</p>
       <input type="text"  @change="passFilter('place')"  v-model="filters.place"  placeholder="поиск по месту"/>
    </div>
    <div>
      <p>Описание</p>
      <input type="text"  v-model="filters.desc" @change="passFilter('desc')" placeholder="поиск по описанию"/>
    </div>
  </div>
</div>
  
</template>

<script>
import Multiselect from 'vue-multiselect'
import TimeLine from '../components/TimeLine.vue'
import qs from 'qs'
export default {
  props: {
    lists: {
      type: Object
    }
  },
  components: {
    Multiselect,
    TimeLine
  },
  data () {
    return {
      filters: {
        type: [],
        state: [],
        series: [],
        date:[],
        title: '',
        desc: '',
        place: '',
        year: ''
      }
    }
  },
  created ( ) {
    this.getUrlParams()

  },
  methods: {
    clearFilters (year) {
      Object.keys(this.filters).forEach(key => {
          if(key === 'year' && year) {
            this.filters[key] = this.filters[key]
          } else {
            if(this.filters[key] && this.filters[key].isArray) {
              this.filters[key] = []
            } else {
              this.filters[key] = ''
            }
          }
      })
    },
    changeYear (val) {
      this.$emit('change-year', {
         filters: {year: val}
      })
      this.clearFilters(val)
      this.setUrl({year: val})
    },
    getNotEmpty () {
      return Object.keys(this.filters).reduce((acc,key) => {
          if(this.filters[key].length > 0 && key !== 'year') {
             acc[key] = this.filters[key]
          }
          return acc
       }, {})
    },
    passFilter() {
       let fullFilters = this.getNotEmpty()
       this.$emit('change-filter', {
         filters: fullFilters
       })
       this.setUrl()
    },
    getUrlParams () {
      let params = qs.parse(window.location.search.slice(1))
      this.filters = Object.assign(this.filters, params)
      let fullFilters = this.getNotEmpty()
      this.$emit('change-filter', {
         filters: fullFilters
      })
    },
    setUrl(val) {
      let search =  qs.parse(window.location.search.slice(1))
      let params
      if(!val) {
        params = Object.assign(search, this.filters)
        params = Object.keys(params).reduce((acc,key) => {
          if(params[key].length > 0 || params[key] > 0 ) {
             acc[key] = params[key]
          }
          return acc
        }, {})
      } else {
        params = val
      }
      params = qs.stringify(params)
      let url = window.location.origin + window.location.pathname + `?${params}`
      history.replaceState({ state: url }, '', url)
    }
  }
}
</script>

<style scoped>
  .filters {
    color: #fff;
    padding-bottom: 40px;
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
    grid-gap: 30px;
  }
  input {
    font-family: inherit;
    margin-left: 10px;
    margin-right: 20px;
  }
  .row-inp {
    display: flex;
    margin-bottom: 10px;
    align-items: center;
  }
</style>
