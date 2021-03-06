<template>
  <div class="request-form">
    <component @runDebug="runDebug" :is="component" :is-read-only="isReadOnly" :request="request"/>
    <ms-scenario-results v-loading="debugReportLoading" v-if="isCompleted" :scenarios="isCompleted ? request.debugReport.scenarios : []"/>
  </div>
</template>

<script>
  import {Request, RequestFactory} from "../../model/ScenarioModel";
  import MsApiHttpRequestForm from "./ApiHttpRequestForm";
  import MsApiDubboRequestForm from "./ApiDubboRequestForm";
  import MsScenarioResults from "../../../report/components/ScenarioResults";

  export default {
    name: "MsApiRequestForm",
    components: {MsScenarioResults, MsApiDubboRequestForm, MsApiHttpRequestForm},
    props: {
      request: Request,
      isReadOnly: {
        type: Boolean,
        default: false
      },
      debugReportId: String
    },
    data() {
      return {
        reportId: "",
        content: {scenarios:[]},
        debugReportLoading: false,
        showDebugReport: false
      }
    },
    computed: {
      component({request: {type}}) {
        let name;
        switch (type) {
          case RequestFactory.TYPES.DUBBO:
            name = "MsApiDubboRequestForm";
            break;
          default:
            name = "MsApiHttpRequestForm";
        }
        return name;
      },
      isCompleted() {
        return !!this.request.debugReport;
      }
    },
    watch: {
      debugReportId() {
        this.getReport();
      }
    },
    methods: {
      getReport() {
        if (this.debugReportId) {
          this.debugReportLoading = true;
          this.showDebugReport = true;
          this.request.debugReport = {};
          let url = "/api/report/get/" + this.debugReportId;
          this.$get(url, response => {
            let report = response.data || {};
            let res = {};
            if (response.data) {
              try {
                res = JSON.parse(report.content);
              } catch (e) {
                console.log(report.content)
                throw e;
              }
              if (res) {
                this.debugReportLoading = false;
                this.request.debugReport = res;
                this.deleteReport(this.debugReportId)
              } else {
                setTimeout(this.getReport, 2000)
              }
            } else {
              this.debugReportLoading = false;
            }
          });
        }
      },
      deleteReport(reportId) {
        this.$post('/api/report/delete', {id: reportId});
      },
      runDebug() {
        this.$emit('runDebug', this.request);
      }
    }
  }
</script>

<style scoped>

  .scenario-results {
    margin-top: 20px;
  }

  .request-form >>> .debug-button {
    margin-left: auto;
    display: block;
    margin-right: 10px;
  }

</style>
