<template>
  <div>
    <b-row v-if="displayMode==='image'" key="image">
      <b-colxx
        sm="6"
        lg="4"
        xl="3"
        class="mb-3"
        v-for="(item,index) in items"
        :key="index"
        :id="item.id"
      >
        <tags-list-item
          :key="item.id"
          :data="item"
          :selected-items="selectedItems"
          @toggle-item="toggleItem"
          v-contextmenu:contextmenu
        />
      </b-colxx>
    </b-row>
    <b-row v-else-if="displayMode==='thumb'" key="thumb">
      <b-colxx xxs="12" class="mb-3" v-for="(item,index) in items" :key="index" :id="item.id">
        <tags-thumb-list-item
          :key="item.id"
          :data="item"
          :selected-items="selectedItems"
          @toggle-item="toggleItem"
          v-contextmenu:contextmenu
        />
      </b-colxx>
    </b-row>
    <b-row v-else-if="displayMode==='list'" key="list">
      <b-colxx xxs="12" class="mb-3" v-for="(item,index) in items" :key="index" :id="item.id">
        <tag-list-item
          :key="item.id"
          :data="item"
          :selected-items="selectedItems"
          @toggle-item="toggleItem"
          v-contextmenu:contextmenu
        />
      </b-colxx>
    </b-row>
    <b-row v-if="lastPage>1">
      <b-colxx xxs="12">
        <b-pagination-nav
          :number-of-pages="lastPage"
          :link-gen="linkGen"
          :value="page"
          @change="(a)=>changePage(a)"
          :per-page="perPage"
          align="center"
        >
          <template v-slot:next-text>
            <i class="simple-icon-arrow-right" />
          </template>
          <template v-slot:prev-text>
            <i class="simple-icon-arrow-left" />
          </template>
          <template v-slot:first-text>
            <i class="simple-icon-control-start" />
          </template>
          <template v-slot:last-text>
            <i class="simple-icon-control-end" />
          </template>
        </b-pagination-nav>
      </b-colxx>
    </b-row>

    <v-contextmenu @contextmenu="handleContextMenu" ref="contextmenu">
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
    </v-contextmenu>
  </div>
</template>
<script>
import TagsImage from "./TagsImage.vue";
import TagsThumb from "./TagsThumb.vue";
import TagsListItem from "./TagsListItem.vue";
export default {
  components: {
    "tags-list-item": TagsImage,
    "tags-thumb-list-item": TagsThumb,
    "tag-list-item": TagsListItem
  },
  props: [
    "displayMode",
    "items",
    "selectedItems",
    "toggleItem",
    "lastPage",
    "perPage",
    "page",
    "changePage",
    "handleContextMenu",
    "onContextMenuAction"
  ],
  methods: {
    linkGen(pageNum) {
      return "#page-" + pageNum;
    }
  }
};
</script>
