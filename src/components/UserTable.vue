<template>
  <div>
      <div v-if="users.length">
      <div v-if="search" class="searchBlock mt-4 mb-4 d-flex justify-content-center align-items-center">
        <div class="input-search w-25">
            <input type="text" v-model="filter" class="form-control" placeholder="search phrase" aria-label="search"> 
        </div>
      </div>
      <table id="tableComponent" class="table table-bordered table-striped">
          <thead>
            <tr>
              <th v-for="oneColumn in this.column" :key="oneColumn"> 
                <h4 :key="itemSortUpdate" @click="sortTable(oneColumn)">
                  {{formatHeader(oneColumn)}}
                </h4>
              </th>
            </tr>
          </thead>
          <tbody>
            <tr 
            v-for="(user, index) in filterList" :key='index' 
            v-show="whatsShow(index)"
            >
              <td v-for="oneColumn in this.column" :key='oneColumn'>
                <h6 v-if="chackEmail(getDotNotation(user, oneColumn))">
                  <a v-bind:href="`mailto:${getDotNotation(user, oneColumn)}`"> {{ getDotNotation(user, oneColumn) }} </a>
                </h6>
                <h6 v-else>
                  {{ getDotNotation(user, oneColumn) }}
                </h6>
              </td>
            </tr>
        </tbody>
      </table>
        <div v-if="pagination" id="page-navigation" class="mt-2 mb-4 p-0">
          <button type="button" class="btn btn-outline-primary" @click="movePages(-1)">Back </button>
          <h6 class="p-0 m-0">{{startRow / rowsPerPage + 1}} out of {{Math.ceil(this.Actuallylength / rowsPerPage)}}</h6>
          <button type="button" class="btn btn-outline-primary" @click="movePages(1)">Next</button>
        </div>
      </div>
  </div>
</template>

<script>
var nestedProperty = require("nested-property");

export default {
  name: 'UserTable',
  props:{
    endpoint: String,
    search: Boolean,
    sorting: Boolean,
    pagination: Boolean,
    col: {
      type: String,
      default: "id,name,email,company.name,address.city,website"
    }
  },

  data(){
    return{
      users: [],
      column: this.col.split(','),
      itemSort:'id',
      itemSortDir:'asc',
      itemSortUpdate: 0,
      reg: /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,24}))$/, //eslint-disable-line
      filter:'',
      rowsPerPage: 3,
      startRow: 0,
      Actuallylength: 0
    }
  },

  mounted(){
    fetch(this.endpoint)
    .then(res => res.json())
    .then(data => this.users = data)
    .catch(err => console.log(err.message))
  },
  methods: {
    getDotNotation(arr, source) {
      return nestedProperty.get(arr, source);
    },
    sortTable(column){
      if(column === this.itemSort) {
        this.itemSortDir = this.itemSortDir==='asc'?'desc':'asc';
      }
      this.itemSort = column;
      this.itemSortUpdate++;
      this.itemSortUpdate--;
    },
    chackEmail(text){
      if(this.reg.test(text)){
        return true;
      }
      return false;
    },
    movePages(buttonCount) {
      let newStartRow = this.startRow + (buttonCount * this.rowsPerPage);
      if (newStartRow >= 0 && newStartRow < this.Actuallylength) {
        this.startRow = newStartRow;
      }
    },
    formatHeader(text){
      let title =  text.charAt(0).toUpperCase() + text.slice(1);
      return title.replace(".", " ");
    },
    whatsShow(index){
      if(this.pagination){
        if(index >= this.startRow && index < this.startRow + this.rowsPerPage){
          return true
        }
        return false
      }else{
        return true
      }
    },
    setLength(index){
      this.Actuallylength = index;
    }
  },
  computed: {
    sortUsers: function(){
      if(this.sorting){
      let tab = this.users;
      let item = this.itemSort;
      let direction = this.itemSortDir;

      return tab.sort((a,b) => {
        let mod = 1;
        if(direction === 'desc') {mod = -1}
        if(this.getDotNotation(a, item) < this.getDotNotation(b, item)) {return -1 * mod}
        if(this.getDotNotation(a, item) > this.getDotNotation(b, item)) {return 1 * mod}
        return 0;
      });
      }
      return 0;
    },
    filterList: function(){
      let tab = this.users;
      if(this.search){
        let filter = this.filter;
        let column = this.column;

        let newTab =  tab.filter((item) => {
          for(let i = 0; i <= column.length; i++)
            {
              if(this.getDotNotation(item, column[i])
              .toString()
              .toLowerCase()
              .includes(filter.toLowerCase())){
                return true
              }
            }
          })
        this.setLength(newTab.length);
        return newTab;
      }else{
        this.setLength(tab.length);
        return tab;
      }
    }
  }
}
</script>

<style scoped lang="scss">
h4{
  cursor: pointer;
}
#page-navigation{
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;

  h6{
    font-weight: 600;
    padding: 0 15px 0 15px !important;
  }
}
</style>
