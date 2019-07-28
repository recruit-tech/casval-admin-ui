<template>
  <div>
    <div class="container py-5">
      <div class="row">
        <div class="col">
          <div class="card">
            <div class="card-body">
              <div class="h5">{{ $t('audit.register-new-audit') }}</div>
              <hr class="mb-2" />
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
              <div class="h5 mb-3">{{ $t('audit.list') }}</div>
              <audit-list :audit-api-client="auditApiClient"></audit-list>
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
      required: true
    }
  },
  components: {
    AuditList,
    AuditRegistrationForm
  }
};
</script>
