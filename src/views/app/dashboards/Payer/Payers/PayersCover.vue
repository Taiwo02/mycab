<template>
  <div v-if="isLoading && !isFetched" style="h-100">

    <div class="align-middle">
      <div class="d-flex justify-content-center">
        <b-spinner variant="primary" />
      </div>
    </div>
  </div>

  <div v-else-if=" !isLoading && !isFetched">

    Error template here
  </div>
  <div v-else>
    <!--<datatable-heading
      :title="$t('menu.divided-table')"
      :selectAll="selectAll"
      :isSelectedAll="isSelectedAll"
      :isAnyItemSelected="isAnyItemSelected"
      :keymap="keymap"
      :changePageSize="changePageSize"
      :searchChange="searchChange"
      :from="from"
      :to="to"
      :total="total"
      :perPage="perPage"
    ></datatable-heading>-->
    <b-row>
      <b-colxx xxs="12">
        <h2 class="text-center text-muted mb-2">LIST OF PAYERS</h2>
        <vuetable
          ref="vuetable"
          class="table-divided order-with-arrow"
          :query-params="makeQueryParams"
          :per-page="perPage"
          :api-mode="false"
          :data="localData"
          :fields="fields"
          pagination-path
          :row-class="onRowClass"
          @vuetable:pagination-data="onPaginationData"
          @vuetable:cell-rightclicked="rightClicked"
        >
          <template slot="actions" slot-scope="props">
            <b-button ref="vehic" id="vehic"
              @click="cellClick(props.rowData)"
              :disabled="!props.rowData.account_business_detail"
              variant="primary"
            >
            <!-- payload.account_business_detail.account_id-->
              <i class="simple-icon-login"></i>
              <!-- <i v-else class="simple-icon-exclamation" />  v-if="props.rowData.account_business_detail" -->
            </b-button>
          </template>
          <template slot="account_business_detail" slot-scope="props">
            <b-button :disabled="!props.rowData.account_business_detail"
              ref="view" id="view" variant="primary"
             @click="cellModal(props.rowData)"
            >
              <i class="simple-icon-magnifier"/>
              <!-- <i v-else class="simple-icon-exclamation" />   v-if="props.rowData.account_business_detail"  -->
            </b-button>

          </template>
        </vuetable>
        <vuetable-pagination-bootstrap
          class="mt-4"
          ref="pagination"
          @vuetable-pagination:change-page="onChangePage"
        />
      </b-colxx>
    </b-row>

    <!--<v-contextmenu ref="contextmenu">
      <v-contextmenu-item @click="onContextMenuAction('copy')">
        <i class="simple-icon-docs" />
        <span>Copy</span>
      </v-contextmenu-item>
      <v-contextmenu-item @click="onContextMenuAction('move-to-archive')">
        <i class="simple-icon-drawer" />
        <span>Move to archive</span>
      </v-contextmenu-item>
      <v-contextmenu-item @click="onContextMenuAction('delete')">
        <i class="simple-icon-trash" />
        <span>Delete</span>
      </v-contextmenu-item>
    </v-contextmenu>-->
    <div>
      <b-modal id="modallg" ref="modallg" size="xl" title="Payer vehicles" hide-footer>
          <div>
            <!-- <PayerVehicles :localData="selectedItemVehicles " />-->
          </div>
      </b-modal>
    </div>
    <div>
      <b-modal id="payerModal" ref="payerModal" modal-class="modal-right" title="Details" hide-footer>
          <payer-side-details v-if="selectedPayload" :selectedPayload="selectedPayload" />
      </b-modal>
    </div>
  </div>
</template>
<script>// @ts-nocheck
import Vuetable from "vuetable-2/src/components/Vuetable";
import VuetablePaginationBootstrap from "../../../../../components/Common/VuetablePaginationBootstrap.vue";
import { PROXY } from '../../../../../constants/config';
import { hToken,/*, loadash*/
PAYERS} from '../../../../../constants/formKey';
import PayerSideDetails from './PayerSideDetails.vue';
// import PayerVehicles from './PayerVehicles.vue';
// import DatatableHeading from "../../../../containers/datatable/DatatableHeading";

export default {
  props: ["title"],
  components: {
    vuetable: Vuetable,
    "vuetable-pagination-bootstrap": VuetablePaginationBootstrap,
    // PayerVehicles,
    PayerSideDetails,
    // "datatable-heading": DatatableHeading
  },
  data() {
    return {
      head: {headers: hToken()},
      isLoad: false,
      // apiBase: `${PROXY}admin/payer/details`,//apiUrl + "/cakes/fordatatable",
      sort: "",
      page: 1,
      perPage: 8,
      search: "",
      from: 0,
      to: 0,
      total: 0,
      lastPage: 0,
      items: [],
      selectedItems: [],
      selectedPayload: null,
      selectedItemVehicles: [],

      isFetched: false,
      isLoading: true,

      fields: [,
        {
        name: "first_name",
        sortField: "first_name",
        title: "First Name",
        titleClass: "",
        dataClass: "list-item-heading",
        width: "10%"
        },
        {
          name:"last_name",
          sortField: "last_name",
          title: "Last Name",
          titleClass: "",
          dataClass: "",
          width: "10%"
        },
        {
          name: "account_no",
          sortField: "account_no",
          title: "Account no.",
          titleClass: "",
          dataClass: "",
          width: "10%"
        },
        {
          name: "is_business",
          sortField: "is_business",
          title: "Type",
          titleClass: "",
          dataClass: "",
          width: "10%",
          callback(val){
            return val? "Business":"Individual";
          }
        },
        {
          name: "phone",
          sortField: "phone",
          title: "Phone",
          titleClass: "",
          dataClass: "",
          width: "10%"
        },
        {
          name: "__slot:actions",
          title: "Full Details",
          titleClass: "center aligned text-right",
          dataClass: "center aligned text-right",
          width: "10%"
        },
        {
          name: "__slot:account_business_detail",
          title: "Preview",
          titleClass: "center aligned text-right",
          dataClass: "center aligned text-right",
          width: "10%"
        },
        // {
        //   name: "account_vehicles",
        //   title: "Vehicle",
        //   titleClass: "center aligned text-right",
        //   dataClass: "center aligned text-right",
        //   width: "5%"
        // }
      ]
    };
  },
  methods: {
    makeQueryParams(sortOrder, currentPage, perPage) {
      this.selectedItems = [];
      return sortOrder[0]
        ? {
            sort: sortOrder[0]
              ? sortOrder[0].field + "|" + sortOrder[0].direction
              : "",
            page: currentPage,
            per_page: this.perPage,
            search: this.search
          }
        : {
            page: currentPage,
            per_page: this.perPage,
            search: this.search
          };
    },
    onRowClass(dataItem, index) {
      if (this.selectedItems.includes(dataItem.id)) {
        return "selected";
      }
      return "";
    },

    cellClicked(item, field, event){
      // console.log(field, 'feild');
      // console.log(item, 'item');
      // console.log(event,'eve');
    },

    rowClicked(dataItem, event) {
      // const itemId = dataItem.id;
      // console.log(dataItem)
      // this.selectedItemVehicles = dataItem.account_vehicles;
      // // this.$refs.modallg.show()
      // return;
      // if (event.shiftKey && this.selectedItems.length > 0) {
      //   let itemsForToggle = this.items;
      //   var start = this.getIndex(itemId, itemsForToggle, "id");
      //   var end = this.getIndex(
      //     this.selectedItems[this.selectedItems.length - 1],
      //     itemsForToggle,
      //     "id"
      //   );
      //   itemsForToggle = itemsForToggle.slice(
      //     Math.min(start, end),
      //     Math.max(start, end) + 1
      //   );
      //   this.selectedItems.push(
      //     ...itemsForToggle.map(item => {
      //       return item.id;
      //     })
      //   );
      //   this.selectedItems = [...new Set(this.selectedItems)];
      // } else {
      //   if (this.selectedItems.includes(itemId)) {
      //     this.selectedItems = this.selectedItems.filter(x => x !== itemId);
      //   } else this.selectedItems.push(itemId);
      // }
    },
    rightClicked(dataItem, field, event) {
      event.preventDefault();
      if (!this.selectedItems.includes(dataItem.id)) {
        this.selectedItems = [dataItem.id];
      }
      // this.$refs.contextmenu.show({ top: event.pageY, left: event.pageX });
    },
    onPaginationData(paginationData) {
      console.log(paginationData);
      this.from = paginationData.from;
      this.to = paginationData.to;
      this.total = paginationData.total;
      this.lastPage = paginationData.last_page;
      this.items = paginationData.data;
      this.$refs.pagination.setPaginationData(paginationData);
    },
    onChangePage(page) {
      this.$refs.vuetable.changePage(page);
    },

    changePageSize(perPage) {
      this.perPage = perPage;
      this.$refs.vuetable.refresh();
    },

    searchChange(val) {
      this.search = val;
      this.$refs.vuetable.refresh();
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

    onContextMenuAction(action) {
      console.log(
        "context menu item clicked - " + action + ": ",
        this.selectedItems
      );
    },
    cellClick(payload){
        if(!payload.account_business_detail || !payload.account_business_detail.account_id) return;

        this.$router.push(`payers/${payload.account_business_detail.account_id}`);
    },

    cellModal(payload){
      if(!payload.account_business_detail) return;
      let payLoad = {...payload};
      delete payLoad.account_vehicles;
      let copy = {...payLoad};
      let account = copy.account_business_detail;
      delete payLoad.account_business_detail;
      let isBusiness = copy.is_business;
      delete payLoad.is_business;
      this.selectedPayload = [payLoad, account, isBusiness];
      // console.log(this.selectedPayload);
      this.$refs.payerModal.show();
    },

    getPayers(){
      this.$store.dispatch(PAYERS);
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
    localData(){
      return this.$store.getters.payers
    },
    resKey(){
      return this.$store.getters.resKey
    }
  },
  watch: {
    resKey(){
      if(this.resKey && this.resKey.owner && this.resKey.owner == PAYERS){
        if(!this.resKey.status){
          this.isFetched = true;
        }else{
          this.isFetched = false;
        }
        this.isLoading = false;
      }
    }
  },
  created(){
    this.getPayers()
    // console.log(this.$route.path);
    // console.log(this.head);
    // console.log( loadash*/.sortBy([{a:1,b:2,c:{a:1,b:2}},{a:1,b:2,c:{a:5,b:2}},{a:5,b:2,c:{a:2,b:2}},{a:3,b:2,c:{a:1,b:2}}], ['c.a','c.b']));
  }
};
</script>
