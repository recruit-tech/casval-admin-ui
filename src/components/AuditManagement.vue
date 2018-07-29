<template>
  <div>
    <div class="container py-5">
      <div class="row">
        <div class="col">
          <div class="card">
            <div class="card-body">
              <div class="h5">新規検査の登録</div>
              <audit-registration-form :audit-api-client="auditApiClient"></audit-registration-form>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="container pb-5">
      <div class="row">
        <div class="col">
          <div class="card">
            <div class="card-body">
              <div class="h5 mb-3">検査中</div>
              <audit-list :audit-api-client="auditApiClient" :mode="'unsubmitted'"></audit-list>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="container pb-5">
      <div class="row">
        <div class="col">
          <div class="card">
            <div class="card-body">
              <div class="h5 mb-3">検査完了</div>
              <audit-list :audit-api-client="auditApiClient" :mode="'submitted'"></audit-list>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import AuditList from './AuditList.vue';
import AuditRegistrationForm from './AuditRegistrationForm.vue';

export default {
  name: 'AuditManagement',
  props: {
    auditApiClient: {
      type: Function,
      required: true,
    },
  },
  components: {
    AuditList,
    AuditRegistrationForm,
  },
};
</script>
