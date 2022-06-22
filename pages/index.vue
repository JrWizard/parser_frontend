<template>
  <v-app>
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
    <v-dialog
      v-model="detailDialog"
      width="500"
    >
      <v-card>
        <v-card-title class="text-h5">
          CRON Job Detail
        </v-card-title>

        <v-divider></v-divider>
        <v-card-text>
          <p>ID: {{ cronJob.id }}</p>
          <p>Name: {{ cronJob.name }}</p>
          <p>Expression: {{ cronJob.expression }}</p>
          <p>URL: {{ cronJob.url }}</p>
          <p>Email: {{ cronJob.email_me }}</p>
          <p>Log: {{ cronJob.log }}</p>
          <p>Post: {{ cronJob.post }}</p>
          <p>Status: {{ cronJob.status }}</p>
          <p>Average executable time: {{ cronJob.executable_time }}</p>
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

import _ from 'lodash'
import { GetCronJobs } from "~/gql/queries";

export default {
  name: 'CronJobsList',
  apollo: {
    getCronJobs: {
      query: GetCronJobs,
      variables () {
        return {
          first: this.tableOptions.itemsPerPage,
          page: this.tableOptions.page,
          orderBy: this.sortingArray
        }
      }
    }
  },
  data () {
    return {
      page: 1,
      tableOptions: {
        page: 1,
        itemsPerPage: 10
      },
      rowsPerPageItems: [5,10,20,30],
      tableHeaders: [
        {
          text: 'ID',
          value: 'id'
        },
        {
          text: 'Name',
          value: 'name'
        }
      ],
      detailDialog: false,
      cronJob: {
        id: 0,
        name: '',
        expression: '',
        url: '',
        email_me: '',
        log: '',
        post: '',
        status: '',
        executable_time: 0
      }
    }
  },
  computed: {
    cronJobs () {
      return this.getCronJobs?.data
    },
    totalCronJobs () {
      return this.getCronJobs?.paginatorInfo.total
    },
    isLoading () {
      return this.$apollo.queries.getCronJobs?.loading
    },
    sortingArray () {
      const clonedTableOptions = _.cloneDeep(this.tableOptions)
      return _.map(clonedTableOptions.sortBy, function (sort, index) {
        return ({
          column: sort,
          order: clonedTableOptions.sortDesc[index] === true ? 'DESC' : 'ASC'
        })
      })
    }
  },
  methods: {
    showDetail () {
      this.openDetailDialog()
    },
    openDetailDialog () {
      this.detailDialog = true
    },
    closeDetailDialog () {
      this.detailDialog = false
    }
  }
}
</script>
