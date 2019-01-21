<template>
  <div class="time-line">
     <div class="year" v-for="(year, i) in years" 
     :key="year"
     :class="{active: i === active}"
      @click="passYear({year: year, index: i })">
        <span v-text="year"></span>
     </div>
     <div class="year" 
      :class="{active: active === ''}"
       @click="passYear({year: '', index: '' })"
      >
      <span>Предстоящие</span>
     </div>
  </div>
</template>

<script>
export default {
  props: {
    value: {
      type: [Number, String]
    }
  },
  data () {
    return  {
      years: [2016, 2017, 2018, 2019],
      active: ''
    }
  },
  created() {
    if(this.value !== '') {
      this.active = this.years.indexOf(Number(this.value))
    }
  },
  methods: {
    passYear(val) {
      this.active = val.index
      this.$emit('input', val.year)
    }
  }
}
</script>

<style scoped>
  .time-line {
    display: flex;
  }
  .year {
    color: #fff;
    width:100px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 1px solid var(--accent);
    border-radius: 10px;
    margin-right: 20px;
    transition: background 1s ease;
    cursor: pointer;
  }
  .year.active, .year:hover {
    background: var(--accent);
  }
</style>
