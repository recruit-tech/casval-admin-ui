<template>
  <div class="pb-5">
    <table class="table table-hover mb-0">
      <thead>
        <tr>
          <th scope="col">{{ $t('audit.name') }}</th>
          <th scope="col">{{ $t('audit.contact') }}</th>
          <th scope="col">{{ $t('audit.created-at') }}</th>
          <th scope="col">{{ $t('audit.updated-at') }}</th>
          <th scope="col" v-if="mode === 'submitted'">{{ $t('audit.action') }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(audit, index) in audits" :key="index">
          <td><a :href="`${origin}#/${audit.uuid}`" target="_blank">{{audit.name}}</a></td>
          <td><a :href="generateMailURL(audit)"><span v-for="(contact_name, index) in audit.contact_names" :key="index">{{contact_name}}<br></span></a></td>
          <td>{{displayDateTime(audit.created_at)}}</td>
          <td>{{displayDateTime(audit.updated_at)}}</td>
          <td v-if="mode === 'submitted'">
            <button type="button" class="btn btn-sm btn-danger float-right mr-2" @click.prevent="remandAudit(index, audit.uuid)" v-if="audit.submitted">
              {{ $t('audit.withdraw') }}
            </button>
          </td>
        </tr>
      </tbody>
    </table>
    <hr class="mt-0">
    <div class="d-flex flex-row justify-content-end my-0">
      <div class="pr-1"><button type="button" class="btn btn-sm btn-outline-secondary float-right mr-2" :disabled="currentPage==1" @click="loadAuditIndex(currentPage - 1)">{{ $t('audit.previous') }}</button></div>
      <div><button type="button" class="btn btn-sm btn-outline-secondary float-right" :disabled="loadedCount < displayCount" @click="loadAuditIndex(currentPage + 1)">{{ $t('audit.next') }}</button></div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
import moment from 'moment';

export default {
  name: 'AuditList',
  props: {
    auditApiClient: {
      type: Function,
      required: true,
    },
    mode: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      audits: [],
      errorMessage: '',
      currentPage: 1,
      origin: process.env.VUE_APP_USER_ORIGIN,
      utcOffset: 0,
      displayCount: 10,
      loadedCount: 0,
    };
  },
  methods: {
    generateMailURL: function generateMailURL(audit) {
      let url = 'mailto:';
      audit.contact_emails.forEach((email) => {
        url = `${url}${email};`;
      });
      return url;
    },
    displayDateTime: function displayDateTime(datetime) {
      if (datetime.length === 0) {
        return '';
      }
      const m = moment(datetime, 'YYYY-MM-DD hh:mm:ss').add(this.utcOffset, 'minutes');
      return m.format(this.$i18n.t('audit.datetime-format'));
    },
    remandAudit: async function remandAudit(index, auditUUID) {
      if (window.confirm(this.$i18n.t('audit.withdraw-confirmation')) === false) {
        return;
      }
      try {
        const res = await this.auditApiClient.delete(`${auditUUID}/submit`);
        switch (res.status) {
          case 200: {
            this.errorMessage = '';
            Vue.delete(this.audits, index);
            break;
          }
          default: {
            this.errorMessage = this.$i18n.t('audit.error-withdraw');
          }
        }
      } catch (e) {
        this.errorMessage = this.$i18n.t('audit.error-withdraw');
      }
    },
    loadAuditIndex: async function loadAuditIndex(page) {
      try {
        const res = await this.auditApiClient.get(`?mode=${this.mode}&page=${page}&count=${this.displayCount}`);
        switch (res.status) {
          case 200:
          case 304:
            this.loadedCount = res.data.length;
            if (this.loadedCount > 0) {
              this.audits = res.data;
              this.currentPage = page;
            }
            break;
          default:
            this.errorMessage = this.$i18n.t('audit.error-loading');
        }
      } catch (e) {
        this.errorMessage = this.$i18n.t('audit.error-loading');
      }
    },
  },
  mounted: function mounted() {
    this.loadAuditIndex(this.currentPage);
    moment.locale(this.$i18n.locale);
    this.utcOffset = moment().utcOffset();
  },
};
</script>
