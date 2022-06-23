<template>
  <v-app>
    <v-card>
      <v-card-title>
        <v-row>
          <v-col md="4" lg="8" align-self="center">
            CRON Jobs
          </v-col>
          <v-col md="8" lg="4">
            <v-text-field
              v-debounce:300ms="changeSearch"
              append-icon="mdi-magnify"
              label="Search"
              single-line
            ></v-text-field>
          </v-col>
        </v-row>
      </v-card-title>
      <v-skeleton-loader
        v-if="!cronJobs"
        class="elevation-2"
        boilerplate
        :type="'table-thead, table-row-divider@'+tableOptions.itemsPerPage+', table-tfoot'"
      />
      <v-data-table
        v-else
        :headers="tableHeaders"
        :items="cronJobs"
        :options.sync="tableOptions"
        :items-per-page="10"
        class="elevation-1"
        :server-items-length="totalCronJobs"
        :loading="isLoading"
        :footer-props="{
        'items-per-page-options': [5,10,20,30]
      }"
        @click:row="showDetail"
      ></v-data-table>
    </v-card>
    <v-dialog
      v-model="detailDialog"
      width="500"
    >
      <v-card>
        <v-card-title class="text-h5">
          CRON Job Detail
        </v-card-title>

        <v-divider></v-divider>

        <v-card-text class="pb-0 pt-4">
          <p class="justify-space-between d-flex"><span>ID:</span> <span>{{ cronJob.id }}</span></p>
          <p class="justify-space-between d-flex"><span>Name:</span> <span>{{ cronJob.name }}</span></p>
          <p class="justify-space-between d-flex"><span>Expression:</span> <span>{{ cronJob.expression }}</span></p>
          <p class="justify-space-between d-flex"><span>URL:</span> <span>{{ cronJob.url }}</span></p>
          <p class="justify-space-between d-flex"><span>Email:</span> <span>{{ cronJob.email_me }}</span></p>
          <p class="justify-space-between d-flex"><span>Log:</span> <span>{{ cronJob.log }}</span></p>
          <p class="justify-space-between d-flex"><span>Post:</span> <span>{{ cronJob.post }}</span></p>
          <p class="justify-space-between d-flex"><span>Status:</span> <span>{{ cronJob.status }}</span></p>
          <p class="justify-space-between d-flex"><span>Average executable time:</span> <span>{{ cronJob.execution_time
            }}</span></p>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="primary"
            text
            @click="closeDetailDialog"
          >
            Close
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script>

import _ from "lodash";
import { GetCronJobDetail, GetCronJobs } from "~/gql/queries";

export default {
  name: "CronJobsList",
  apollo: {
    getCronJobs: {
      query: GetCronJobs,
      variables() {
        return this.queryVariables
      }
    }
  },
  data() {
    return {
      page: 1,
      tableOptions: {
        page: 1,
        itemsPerPage: 10
      },
      tableSearch: '',
      rowsPerPageItems: [5, 10, 20, 30],
      tableHeaders: [
        {
          text: "ID",
          value: "id"
        },
        {
          text: "Name",
          value: "name"
        }
      ],
      detailDialog: false,
      cronJob: {
        id: 0,
        name: "",
        expression: "",
        url: "",
        email_me: "",
        log: "",
        post: "",
        status: "",
        execution_time: 0
      }
    };
  },
  computed: {
    cronJobs() {
      return this.getCronJobs?.data;
    },
    totalCronJobs() {
      return this.getCronJobs?.paginatorInfo.total;
    },
    isLoading() {
      return this.$apollo.queries.getCronJobs?.loading;
    },
    queryVariables () {
      const variables = {
        first: this.tableOptions.itemsPerPage,
        page: this.tableOptions.page,
        orderBy: this.sortingArray
      }

      if (this.tableSearch) {
          variables.search = "%" + this.tableSearch + "%"
      }

      return variables
    },
    sortingArray() {
      const clonedTableOptions = _.cloneDeep(this.tableOptions);
      return _.map(clonedTableOptions.sortBy, function(sort, index) {
        return ({
          column: sort,
          order: clonedTableOptions.sortDesc[index] === true ? "DESC" : "ASC"
        });
      });
    }
  },
  methods: {
    changeSearch (value) {
      this.tableSearch = value
    },
    showDetail(value) {
      this.loadDetailData(value.id).then((job) => {
        this.cronJob = job.data?.getCronJobDetail;
        this.openDetailDialog();
      });
    },
    async loadDetailData(id) {
      return await this.$apollo.query({
        query: GetCronJobDetail,
        variables: {
          id
        }
      });
    },
    openDetailDialog() {
      this.detailDialog = true;
    },
    closeDetailDialog() {
      this.detailDialog = false;
    }
  }
};
</script>
