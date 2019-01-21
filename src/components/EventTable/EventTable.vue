<template>
  <div class="table">
    <div class="row head">
     <div>Дата</div>
     <div>Наименование</div>
     <div>Тип</div>
     <div>Место</div>
    <div>Описание</div>
    <div>Статус</div>
    <div>Серия</div>
    <div>Цена</div>
    <div>Ссылка</div>
    </div>
    <EventTableRow class="row" v-for="(row, i) in table" :key="i" :row="row" :transleitor="transleitor"/>
  </div>
</template>

<script>
import EventTableRow from './EventTableRow'
export default {
  props:{
    table: {
      type: Array
    },
    transleitor: {
      type: Object
    }
  },
  components: {
    EventTableRow
  }
}
</script>

<style scoped>
   .row {
     color: #ffff;
     display: grid;
     grid-template-columns: 
     100px 
     minmax(100px, 350px) 
     minmax(100px, 150px) 
     repeat(2, minmax(100px, 350px)) 
     minmax(100px, 150px)
     minmax(100px, 200px)
     200px
     minmax(100px, 200px);
     border-bottom: 1px solid #ffff;
     grid-gap: 10px;
   }
   .head {
     padding: 10px;
   }
</style>
