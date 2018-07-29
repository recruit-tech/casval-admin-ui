<template>
  <div>
    <div class="container-fluid py-5">
      <div class="container">
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
    </div>
    <div class="container pt-5">
      <div class="h5">検査中</div>
    </div>
    <div class="container">
      <div class="h5">検査完了</div>
    </div>
  </div>
</template>

<script>
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
    AuditRegistrationForm,
  },
};
</script>
