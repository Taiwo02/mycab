<template>
  <b-row>
    <b-colxx class="disable-text-selection">
      <list-page-heading
        :title="$t('menu.agents')"
        :selectAll="selectAll"
        :isSelectedAll="isSelectedAll"
        :isAnyItemSelected="isAnyItemSelected"
        :keymap="keymap"
        :displayMode="displayMode"
        :changeDisplayMode="changeDisplayMode"
        :changeOrderBy="changeOrderBy"
        :changePageSize="changePageSize"
        :sort="sort"
        :searchChange="searchChange"
        :from="from"
        :to="to"
        :total="total"
        :perPage="perPage"
        :sortOptions="sortOptions"
        :formKey="ADD_AGENT"
      ></list-page-heading>
      <template v-if="isLoad">
        <agent-page-listing
          :displayMode="displayMode"
          :items="agents"
          :selectedItems="selectedItems"
          :toggleItem="toggleItem"
          :lastPage="lastPage"
          :perPage="perPage"
          :page="page"
          :changePage="changePage"
          :handleContextMenu="handleContextMenu"
          :onContextMenuAction="onContextMenuAction"
        ></agent-page-listing>
      </template>
      <template v-else>
        <div class="loading"></div>
      </template>
    </b-colxx>
  </b-row>
</template>

<script>
import { PROXY } from "./../../../../constants/config";
import {hToken} from './../../../../constants/formKey'
import Axios from 'axios';
import ListPageHeading from "./../ListsHeader/ListPageHeading.vue";
import AgentListing from "./AgentListing";
// import ConversionRatesChartCard from "../../../containers/dashboards/ConversionRatesChartCard";
// import OrderStockRadarChart from "../../../containers/dashboards/OrderStockRadarChart";
// import ProductCategoriesDoughnut from "../../../containers/dashboards/ProductCategoriesDoughnut";
// import ProductCategoriesPolarArea from "../../../containers/dashboards/ProductCategoriesPolarArea";
// import ProfileStatuses from "../../../containers/dashboards/ProfileStatuses";
// import SalesChartCard from "../../../containers/dashboards/SalesChartCard";
// import SmallLineCharts from "../../../containers/dashboards/SmallLineCharts";
// import SortableStaticticsRow from "../../../containers/dashboards/SortableStaticticsRow";
// import AgentsCard from "../../../containers/dashboards/AgentsCard";
import {ADD_AGENT} from './../../../../constants/formKey';


export default {

  components: {
    "list-page-heading": ListPageHeading,
    "agent-page-listing": AgentListing
  },
  data() {
    return {
      ADD_AGENT,
        sortOptions: [
        {
          column: "firstname",
          label: "firstname"
        },
         {
          column: "lastname",
          label: "Lastname"
        },
          {
          column: "id",
          label: "id"
        },
        {
          column: "agent_type",
          label: "Agent_type"
        },
        {
          column: "phone",
          label: "Phone"
        }
      ],

       sort: {
        column: "firstname",
        label: "Name"
      },
         agents: [
        {
          firstname: 'Stephanie',
          lastname: 'Sunday',
          agent_type: 'commercial',
          garage:"Sabo Ogbomoso",
          phone:"0908924664567",
          img:"/assets/img/agents/agent1.jfif"
        },
         {
          firstname: 'Bola',
          lastname: 'Sunday',
          agent_type: 'commercial',
          garage:"Sabo Ogbomoso",
          phone:"0908924664567",
          img:"/assets/img/agents/agent2.jfif"

        },
         {
           firstname: 'Victor',
          lastname: 'Ogunniran',
          agent_type: 'commercial',
          garage:"Taki Ogbomoso",
          phone:"080892656764567",
          img:"/assets/img/agents/agent3.jfif"

        },
         {
          firstname: 'Sola',
          lastname: 'Jonson',
          agent_type: 'commercial',
          garage:"Hojo Ibadan",
          phone:"0908924789889",
          img:"/assets/img/agents/agent4.jfif"
        },
      ],
      isLoad: true,
      apiBase: PROXY + "",
      displayMode: "list",
      sort: {
        column: "firstname",
        label: "firstname"
      },
      page: 1,
      perPage: 4,
      search: "",
      from: 0,
      to: 0,
      total: 0,
      lastPage: 0,
      items: [],
      paramName:"",
      selectedItems: []
    };
  },
  methods: {
    loadItems() {
      this.isLoad = false;
           let resp = this.sort.column
        this.items =  this.agents
        .sort(function(a, b){
          var x = a[resp]; var y = b[resp]
          return ((x > y) ? 1 : ((x < y) ? -1 : 0))
            });
          this.isLoad = true;
          console.log(this.items)
      // Axios
      //   .get(`${this.PROXY}`,{headers:hToken()})
      //   .then(response => {
      //     return response.data;
      //   })
      //   .then(res => {
      //     console.log(res)
      //     this.total = res.total;
      //     this.from = res.from;
      //     this.to = res.to;

      //   //   this.items = res.data.map(x => {
      //   //     return {
      //   //       ...x,
      //   //       img: x.img.replace("/img/", "/img/products/")
      //   //     };
      //   //   });
      //     this.perPage = res.per_page;
      //     this.selectedItems = [];
      //     this.lastPage = res.last_page;
      //     this.isLoad = true;
      //   });
    },

    changeDisplayMode(displayType) {
      this.displayMode = displayType;
    },
    changePageSize(perPage) {
      this.page = 1;
      this.perPage = perPage;
    },
    changeOrderBy(sort) {
      this.sort = sort;
      this.loadItems()
    },
    searchChange(val) {
      this.search = val;
      this.page = 1;
    },

    selectAll(isToggle) {
      if (this.selectedItems.length >= this.items.length) {
        if (isToggle) this.selectedItems = [];
      } else {
        this.selectedItems = this.items.map(x => x.id);
      }
    },
    keymap(event) {
      switch (event.srcKey) {
        case "select":
          this.selectAll(false);
          break;
        case "undo":
          this.selectedItems = [];
          break;
      }
    },
    getIndex(value, arr, prop) {
      for (var i = 0; i < arr.length; i++) {
        if (arr[i][prop] === value) {
          return i;
        }
      }
      return -1;
    },
    toggleItem(event, itemId) {
      if (event.shiftKey && this.selectedItems.length > 0) {
        let itemsForToggle = this.items;
        var start = this.getIndex(itemId, itemsForToggle, "id");
        var end = this.getIndex(
          this.selectedItems[this.selectedItems.length - 1],
          itemsForToggle,
          "id"
        );
        itemsForToggle = itemsForToggle.slice(
          Math.min(start, end),
          Math.max(start, end) + 1
        );
        this.selectedItems.push(
          ...itemsForToggle.map(item => {
            return item.id;
          })
        );
      } else {
        if (this.selectedItems.includes(itemId)) {
          this.selectedItems = this.selectedItems.filter(x => x !== itemId);
        } else this.selectedItems.push(itemId);
      }
    },
    handleContextMenu(vnode) {
      if (!this.selectedItems.includes(vnode.key)) {
        this.selectedItems = [vnode.key];
      }
    },
    onContextMenuAction(action) {
      console.log(
        "context menu item clicked - " + action + ": ",
        this.selectedItems
      );
    },
    changePage(pageNum) {
      this.page = pageNum;
    }
  },
  computed: {
    isSelectedAll() {
      return this.selectedItems.length >= this.items.length;
    },
    isAnyItemSelected() {
      return (
        this.selectedItems.length > 0 &&
        this.selectedItems.length < this.items.length
      );
    },
    apiUrl() {
      return `${this.apiBase}?sort=${this.sort.column}&page=${this.page}&per_page=${this.perPage}&search=${this.search}`;
    }

  },
  watch: {
    search() {
      this.page = 1;
    },
    apiUrl() {
      this.loadItems();
    }
  },
   mounted: function(){
    //  alert();
      this.paramName =this.$router.currentRoute.params.name
      console.log(this.paramName)
        this.loadItems();
  },
 watch: {
    $route(to, from) {
      this.paramName = to.params.name
      console.log(this.paramName)

    }
  }
};
</script>
