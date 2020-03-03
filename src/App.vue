<template>
  <div id="app">

    <vh-datatables
      :config="datatablesConfig"
      @event:onClickView="view"
      @event:onClickEdit="edit"
      @event:onClickDelete="del"
      @event:onClickRefresh="refresh"
      @event:onClickAdd="add"
      @event:onAjaxDataResponse="response"
      @event:onError="error"
    >
      <template v-slot:table>
        <table class="vs-table vs-table--tbody-table" cellspacing="0" width="100%">
          <thead class="vs-table--thead">
            <tr>
              <th>Id</th>
              <th>User</th>
              <th>Email</th>
              <th>Activated</th>
              <th>Roles</th>
              <th></th>
              <th></th>
              <th></th>
            </tr>
          </thead>
          <tfoot>
            <tr>
              <th>&nbsp;</th>
              <th>&nbsp;</th>
              <th>&nbsp;</th>
              <th>&nbsp;</th>
              <th>&nbsp;</th>
              <th></th>
              <th></th>
              <th></th>
            </tr>
          </tfoot>
        </table>
      </template>
    </vh-datatables>
  </div>
</template>


<script>
import jQuery from "jquery";
const $ = jQuery;
window.$ = window.jQuery = jQuery;
import '@/components/volho/tables/style.scss'
import VhDatatables from "@/components/volho/tables/Datatables.vue";

export default {
  components: {
    VhDatatables
  },
  data() {
    return {
      datatablesConfig: {
        columns: [
          { data: "id", visible: false },
          { data: "displayName" },
          { data: "email" },
          {
            data: "emailVerified",
            searchable: false,
            orderable: false
          },
          {
            data: "roles",
            render: function(data, type, row) {
              var ret = "";
              $.each(data, function(index, value) {
                ret += (index === 0 ? "" : ", ") + value.name;
              });
              return ret;
            },
            orderable: false
          }
        ],
        dataUrl: "/users.json"
      }
    };
  },
  mounted() {},
  methods: {
    view(data, ev) {
      console.log("v", data, ev);
    },
    edit(data, ev) {
      console.log("e", data, ev);
    },
    del(data, ev) {
      console.log("d", data, ev);
    },
    refresh(data, ev) {
      console.log("r", data, ev);
    },
    add(data, ev) {
      console.log("a", data, ev);
    },
    response(data) {
      console.log("response", data);
    },
    error(data) {
      console.log("error", data);
    }
  }
};
</script>
