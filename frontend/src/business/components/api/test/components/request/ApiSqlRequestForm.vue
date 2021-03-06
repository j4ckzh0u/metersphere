<template>
  <el-form :model="request" :rules="rules" ref="request" label-width="100px" :disabled="isReadOnly">

    <el-form-item :label="$t('api_test.request.name')" prop="name">
      <el-input v-model="request.name" maxlength="300" show-word-limit/>
    </el-form-item>

    <el-form-item :label="'连接池'" prop="dataSource">
      <el-select v-model="request.dataSource">
        <el-option v-for="(item, index) in scenario.databaseConfigs" :key="index" :value="item.name" :label="item.name"/>
      </el-select>
    </el-form-item>

    <!--<el-form-item :label="'查询类型'" prop="protocol">-->
      <!--<el-select v-model="request.queryType">-->
        <!--<el-option label="dubbo://" :value="protocols.DUBBO"/>-->
      <!--</el-select>-->
    <!--</el-form-item>-->

    <el-form-item :label="'超时时间'" prop="queryTimeout">
     <el-input-number :disabled="isReadOnly" size="mini" v-model="request.queryTimeout" :placeholder="$t('commons.millisecond')" :max="1000*10000000" :min="0"/>
    </el-form-item>

    <el-button :disabled="!request.enable || !scenario.enable || isReadOnly" class="debug-button" size="small" type="primary" @click="runDebug">{{$t('api_test.request.debug')}}</el-button>

    <el-tabs v-model="activeName">
      <el-tab-pane :label="'sql脚本'" name="sql">
        <div class="sql-content" >
          <ms-code-edit mode="sql" :read-only="isReadOnly" :modes="['sql']" :data.sync="request.query" theme="eclipse" ref="codeEdit"/>
        </div>
      </el-tab-pane>
      <el-tab-pane :label="$t('api_test.request.assertions.label')" name="assertions">
        <ms-api-assertions :is-read-only="isReadOnly" :assertions="request.assertions"/>
      </el-tab-pane>
      <el-tab-pane :label="$t('api_test.request.extract.label')" name="extract">
        <ms-api-extract :is-read-only="isReadOnly" :extract="request.extract"/>
      </el-tab-pane>
      <el-tab-pane :label="$t('api_test.request.processor.pre_exec_script')" name="beanShellPreProcessor">
        <ms-jsr233-processor :is-read-only="isReadOnly" :jsr223-processor="request.jsr223PreProcessor"/>
      </el-tab-pane>
      <el-tab-pane :label="$t('api_test.request.processor.post_exec_script')" name="beanShellPostProcessor">
        <ms-jsr233-processor :is-read-only="isReadOnly" :jsr223-processor="request.jsr223PostProcessor"/>
      </el-tab-pane>
    </el-tabs>
  </el-form>
</template>

<script>
  import MsApiKeyValue from "../ApiKeyValue";
  import MsApiAssertions from "../assertion/ApiAssertions";
  import {DubboRequest, Scenario, SqlRequest} from "../../model/ScenarioModel";
  import MsApiExtract from "../extract/ApiExtract";
  import ApiRequestMethodSelect from "../collapse/ApiRequestMethodSelect";
  import MsDubboInterface from "@/business/components/api/test/components/request/dubbo/Interface";
  import MsDubboRegistryCenter from "@/business/components/api/test/components/request/dubbo/RegistryCenter";
  import MsDubboConfigCenter from "@/business/components/api/test/components/request/dubbo/ConfigCenter";
  import MsDubboConsumerService from "@/business/components/api/test/components/request/dubbo/ConsumerAndService";
  import MsJsr233Processor from "../processor/Jsr233Processor";
  import MsCodeEdit from "../../../../common/components/MsCodeEdit";

  export default {
    name: "MsApiSqlRequestForm",
    components: {
      MsCodeEdit,
      MsJsr233Processor,
      MsDubboConsumerService,
      MsDubboConfigCenter,
      MsDubboRegistryCenter,
      MsDubboInterface, ApiRequestMethodSelect, MsApiExtract, MsApiAssertions, MsApiKeyValue
    },
    props: {
      request: SqlRequest,
      scenario: Scenario,
      isReadOnly: {
        type: Boolean,
        default: false
      }
    },

    data() {
      return {
        activeName: "sql",
        rules: {
          name: [
            {required: true, message: this.$t('commons.input_name'), trigger: 'blur'},
            {max: 300, message: this.$t('commons.input_limit', [1, 300]), trigger: 'blur'},
          ],
          dataSource: [
            {required: true, message: this.$t('commons.input_name'), trigger: 'blur'},
          ],
        }
      }
    },

    methods: {
      useEnvironmentChange(value) {
        if (value && !this.request.environment) {
          this.$error(this.$t('api_test.request.please_add_environment_to_scenario'), 2000);
          this.request.useEnvironment = false;
        }
        this.$refs["request"].clearValidate();
      },
      runDebug() {
        this.$emit('runDebug');
      }
    },

    computed: {}
  }
</script>

<style scoped>

  .sql-content {
    height: calc(100vh - 570px);
  }

</style>
