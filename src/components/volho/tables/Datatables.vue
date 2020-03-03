<template>
  <div>
      <div class="vx-row mb-6 ">
        <div class="vx-col w-15">
        <vs-select id="lengthChange" v-model="tableLength" name="lengthChange" label="Items per page" class="w-full">
            <vs-select-item key="5" value="5" text="5" />
            <vs-select-item key="10" value="10" text="10" />
            <vs-select-item key="20" value="20" text="20" />
            <vs-select-item key="-1" value="-1" text="All" />
        </vs-select>
        </div>
        <div class="vx-col w-20">
        <vs-input v-model="tableSearch" placeholder="Keyword" label="Search"/>
        </div>
    </div>

    <slot name="table"></slot>
  </div>
</template>

<script>
import jQuery from 'jquery'
const $ = jQuery
window.$ = window.jQuery = jQuery
import "datatables.net";

export default {
  name: "VhDatatables",
  props: {
    tableClass: String,
    config: {
      required: true,
      type: Object
    }
  },
  data() {
    return {
        tableSearch: "",
        tableLength: 5
    };
  },
  computed: {
    isLoaded() {
      return $.fn.dataTable.isDataTable($(this.$el).find("table"));
    }
  },
  watch: {
      tableLength(newVal, oldVal) {
          console.log(newVal, oldVal);
          var vm = this;
          $(vm.$el)
            .find("table")
            .DataTable().page.len(newVal).draw();
      },
      tableSearch(newVal, oldVal) {
          console.log(newVal, oldVal);
          var vm = this;
          $(vm.$el)
            .find("table")
            .DataTable().search(newVal).draw() ;
      }
  },
  created() {

  },
  mounted() {
    this.init();
    this.bindClick();
    this.bindError();
  },
  methods: {
    init() {
      var vm = this;
      var settings = $.extend({}, {}, this.config);

      var nTable = $(this.$el).find("table");
      var columns = settings.columns;
      var dataUrl = settings.dataUrl;
      var colIndex = settings.columns.length;

      var table = nTable.DataTable({
        pageLength: 5,
        searching: true,
        lengthChange: false,
        ordering: true,
        order: [[0, "desc"]],
        processing: true,
        serverSide: false,
        lengthMenu: [[5, 10, 25, 50, -1], [5, 10, 25, 50, "All"]],
        bStateSave: false,
        fnStateSave: function(oSettings, oData) {
          localStorage.setItem(
            "DataTables_" + window.location.pathname,
            JSON.stringify(oData)
          );
        },
        fnStateLoad: function(oSettings) {
          return JSON.parse(
            localStorage.getItem("DataTables_" + window.location.pathname)
          );
        },
        ajax: {
          type: "GET",
          dataType: "json",
          url: dataUrl,
          data: function(d) {
            vm.$emit(`event:onAjaxDataResponse`, d);
          }
        },
        columns: columns,
        pagingType: "full_numbers",
        rowGroup: {
          className: "tr-values vs-table--tr tr-table-state-null"
        },
        createdRow: function(row, data, dataIndex) {
          $(row).addClass("tr-values vs-table--tr tr-table-state-null");
        },
        drawCallback: function() {
        },
        columnDefs: [
          {
            targets: 0,
            className: "td-check"
          },
          {
            targets: colIndex + 0,
            className: "td vs-table--td",
            data: "id",
            render: function(data, type, row) {
              return '<a href="#" data-action="onClickView" class="vh-dt-view"> View </a>';
            }
          },
          {
            targets: colIndex + 1,
            data: "id",
            render: function(data, type, row) {
              return '<a href="#" data-action="onClickEdit" class="vh-dt-edit"> Edit </a>';
            }
          },
          {
            targets: colIndex + 2,
            data: "id",
            render: function(data, type, row) {
              return '<a href="#" data-action="onClickDelete" class="vh-dt-remove"> Delete </a>';
            }
          }
        ]
      });

        $("div.dataTables_filter").hide();
    },
    fetchData(from) {
      this.isLoaded &&
        $(this.$el)
          .find("table")
          .DataTable()
          .ajax.url(from)
          .load();
    },
    refresh(resetPaging) {
      this.isLoaded &&
        $(this.$el)
          .find("table")
          .DataTable()
          .ajax.reload(null, !!resetPaging);
    },
    bindClick() {
      let vm = this;
      $(vm.$el).on("click", "a, button", function(ev) {
        ev.preventDefault();

        let $btn = $(this);
        if ($btn.data("action")) {
          let rowData = $(vm.$el)
            .find("table")
            .DataTable()
            .row($btn.parents("tr"))
            .data();
          vm.$emit(`event:${$btn.data("action")}`, rowData, ev);
        }
      });
    },
    bindError() {
      let vm = this;
      // Disable default error mode (alert) from DataTables
      $.fn.dataTable.ext.errMode = "none";
      $(vm.$el)
        .find("table")
        .on("error.dt", (e, settings, techNote, message) => {
          vm.$emit("event:onError", { settings, techNote, message });
        });
    }
  },
};
</script>

<style scoped language="scss">

.vs-table--tbody-table:nth-child(2n) {
  background: transparent;
}
   
</style>
