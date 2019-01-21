<template>
  <div id="app">
    <section class="container dark">
      <header class="title accent-text" :ref="'title'">
        <h1 class="anaglyph">События</h1>
      </header>
      <Filters v-if="lists" :lists="lists" @change-filter="filterTable" @change-year="changeYear"/>
      <EventTable :table="table" :transleitor="transleitor"/>
      <div class="preloader dark" v-if="preloader">
         <h3>wait ...</h3>
      </div>
    </section>

  </div>
</template>

<script>
import EventTable from './components/EventTable/EventTable.vue'
import Filters from './components/Filters'
import qs from 'qs'
import { filter as rFilter, 
          sort as rSort,
          comparator, 
          gt, 
          prop,
          has,
          curry
        } from 'ramda'
import { isFuture, isWithinRange } from 'date-fns'

export default {
  name: 'app',
  components: {
    EventTable,
    Filters
  },
  data () {
    return  {
      table: null,
      year: null,
      currentDate: new Date(),
      transleitor: null,
      lists: null,
      fullTable: null,
      clearTable: null,
      preloader: true
    }
  },
  created () {
    let getDatas = [this.getData(),  this.getTransleitor()]
    Promise.all(getDatas).then((result) => {
      let year = ''
      if(window.location.search) {
        let params = qs.parse(window.location.search.slice(1))
        if(params.year) {
           year = Number(params.year)
        }
      }
      let table = result[0].map(el => {
        if(el.date) {
          el.date = this.convertToDate(el.date)
        }
        return el
      })
      this.fullTable = table
      this.table = this.tableReady(year, table)
      this.clearTable = [...this.table]
      this.transleitor = result[1]
      this.lists = this.collectLists(this.table, this.transleitor)
      this.preloader = false
    }).catch((e) => {
      console.error(e)
    })
  },
  methods: {
    getData() {
      return fetch('events.json').then(response => {
        return response.json()
      })
    },
    getTransleitor() {
      return fetch('russian.json').then(response => {
        return response.json()
      })
    },
    collectLists(table, transleitor) {
      let obj = {
            type: [],
            state:[],
            series: []
          }
          let lists = table.reduce((acc, el) => {
            Object.keys(acc).forEach(key => {
              if (has(key, el)) {
                acc[key].push(el[key])
              }
            })
            return acc
          }, obj )
          Object.keys(lists).forEach(key => {
             const unicArray = () => {
               let set = new Set(lists[key])
               return [...set]
             }
             let options = unicArray().reduce((acc, el) => {
               acc.push({id: el, title: transleitor[el] || `${el} не переведено`})
               return acc
             }, [])
             lists[key] = options
          })
          return lists
    },
    matchYear(year, table) {
      const isInFuture = (el) => {
        return isFuture(new Date(el.date))
      }
      const isInDate = curry((year, el) => {
          return isWithinRange(
            el.date, 
            new Date(year, 0, 1),
            new Date(year, 11, 31)
          )
      })
      const dateComparator = comparator(
        (a, b) => gt(
          prop('date', a), 
          prop('date', b)
        )
      );
      let filtered
      if(year && year > 0) {
        filtered = rFilter(isInDate(year), table)
      } else {
        filtered = rFilter(isInFuture, table)
      }
      let sorted = rSort(dateComparator, filtered)
      return sorted
    },
    tableReady(year, table) {
      return this.matchYear(year, table)
    },
    changeYear(val) {
      let table =  [...this.fullTable]
      table = this.matchYear(val.filters.year, table)
      this.clearTable = [...table]
      this.table = [...table]
      this.lists = this.collectLists(this.table, this.transleitor)
    },
    filterTable (val) {
      const haveTag = curry((payload, el) => {
        let {arr, name} = payload
        if(el[name]) {
          return arr.reduce((acc, item) => {
          if(item.id === el[name]) {
            acc = true
          }
          return acc
          }, false)
        } else {
          return false
        }
      })
      const matchText = curry((payload, el) => {
        let {text, name} = payload
        if (el[name]) {
          let full = el[name].toLowerCase()
          let str = text.toLowerCase()
          return full.indexOf(str) !== -1
        } else {
          return false
        }
      })
      const matchDateRange = curry((payload, el) => {
         let {dates, name} = payload
          if (el[name]) {
             return isWithinRange(el[name], dates[0], dates[1])
          } else {
            return false
          }
      })
      if(!val.filters.year) {
        let table = [...this.clearTable]
        Object.keys(val.filters).forEach(key => {
          if(key === 'type'|| key === 'state' || key === 'series') {
            table = rFilter(haveTag({arr: val.filters[key], name: key}), table)
          } else if (key === 'place'|| key === 'title' || key === 'desc' ) {
            table = rFilter(matchText({text:val.filters[key], name: key}), table)
          } else if (key === 'date') {
            if(val.filters[key].length === 2) {
              table = rFilter(matchDateRange({dates:val.filters[key], name: key}), table)
            }
          } 
        })
        this.table = [...table]
      }
    },
    convertToDate(el) {
      let str = el.split(' ')
      let date = `${str[0].split('.').reverse().join('-')} ${str[1]}`
      return new Date(date)
    }
  }
}
</script>

<style>
@import './assets/UI.css';
:root {
  --dark: #1A1A1D;
  --grey: #4E4E50;
  --accent: #C3073F;
  --compliment: #950740;
  --complimentDark: #6F2232;
}
* {
  box-sizing: border-box;
}
body{
  margin: 0;
}
#app {
  font-family: 'Arsenal', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.dark {
  background: var(--dark);
}
.container {
  width: 100%;
  min-height: 100vh;

  padding: 30px;
}
h1 {
  font-size: 40px;
}
.title {
  grid-column: 1 / 3;
}
.accent-text {
  font-weight: bold;
  color: var(--accent);
}
.anaglyph {
  border: none;
  color: var(--accent);
  text-align: center;
  text-overflow: clip;
  letter-spacing: 3px;
  text-shadow: -3px 0 1px var(--grey) , 3px 0 1px var(--complimentDark);
  animation: anaglyph 1.5s infinite alternate;
}
.preloader {
  width: 100%;
  position: fixed;
  top: 0;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10;
  color:  var(--accent);
}
@keyframes anaglyph {
   from {
     text-shadow: -3px 0 1px var(--grey) , 3px 0 1px var(--complimentDark);
   }
   to {
     text-shadow: 0px 0 0px var(--grey) , 0px 0 0px var(--complimentDark);
   }
}
</style>
