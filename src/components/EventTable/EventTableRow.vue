<template>
  <div>
    <div v-text="row.date && row.date !== 'Invalid Date'  ? format(row.date, ): nofieldText">
      
    </div>
    <div v-text="row.title ? row.title: nofieldText">
      
    </div>
    <div v-text="transleitor[row.type] ? transleitor[row.type]: nofieldText">
        
    </div>
     <div v-text="row.place ? row.place: nofieldText">
      
    </div>
     <div v-text="row.desc ? row.desc: nofieldText">
      
    </div>
    <div v-text="transleitor[row.state] ? transleitor[row.state]: nofieldText">
      
    </div>
    <div v-text="transleitor[row.series] ? transleitor[row.series]: nofieldText">
      
    </div>
    <div>
      <span v-text="row.price ? row.price: nofieldText"></span>
    </div>
     <div >
        <a v-if="row.link" :href="row.link">
          Ссылка
        </a>
        <span v-else>{{nofieldText}}</span>
    </div>
  </div>
</template>

<script>
import {format} from 'date-fns'
export default {
  props: {
    row: {
      type: Object
    },
    transleitor: {
      type: Object
    }
  },
  data() {
    return  {
      nofieldText: '--'
    }
  },
  methods: {
    format(el) {
      return format(el, 'DD.MM.YYYY HH:ss');
    }
  }
}
</script>

<style scoped>
  div {
    word-wrap: break-word;
    padding: 10px 0;
  }
</style>