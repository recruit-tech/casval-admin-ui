<template>
  <div class="pb-5">
    <table class="table table-hover mb-0">
      <thead>
        <tr>
          <th scope="col">{{ $t('audit.name') }}</th>
          <th scope="col">{{ $t('audit.contact') }}</th>
          <th scope="col">{{ $t('audit.created-at') }}</th>
          <th scope="col">{{ $t('audit.updated-at') }}</th>
          <th scope="col">{{ $t('audit.action') }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(audit, index) in audits" :key="index">
          <td><a :href="`${origin}#/${audit.id}`" target="_blank">{{audit.name}}</a></td>
          <td><span v-for="(contact, index) in audit.contacts" :key="index">{{contact.name}}<br></span></td>
          <td>{{displayDateTime(audit.created_at)}}</td>
          <td>{{displayDateTime(audit.updated_at)}}</td>
          <td>
            <a :href="generateMailURL(audit)">{{ $t('audit.send-email') }}</a><br>
            <a href="#" @click.prevent="remandAudit(index, audit.id)" v-if="audit.submitted">
              {{ $t('audit.withdraw') }}
            </a>
          </td>
        </tr>
      </tbody>
    </table>
    <hr class="mt-0">
    <button class="btn btn-sm btn-outline-secondary float-right" @click="loadAuditIndex()">{{ $t('audit.read-more') }}</button>
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
      oldestAuditId: 0,
      origin: process.env.VUE_APP_USER_ORIGIN,
      utcOffset: 0,
    };
  },
  methods: {
    generateMailURL: function generateMailURL(audit) {
      let url = 'mailto:';
      audit.contacts.forEach((contact) => {
        url = `${url}${contact.email};`;
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
    remandAudit: async function remandAudit(index, auditId) {
      try {
        const res = await this.auditApiClient.delete(`${auditId}/submit`);
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
    loadAuditIndex: async function loadAuditIndex() {
      try {
        const res = await this.auditApiClient.get(`?mode=${this.mode}&before=${this.oldestAuditId}`);
        switch (res.status) {
          case 200:
          case 304:
            if (res.data.length > 0) {
              this.audits = this.audits.concat(res.data);
              this.oldestAuditId = this.audits[this.audits.length - 1].audit_id;
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
    this.loadAuditIndex(0);
    moment.locale(this.$i18n.locale);
    this.utcOffset = moment().utcOffset();
  },
};
</script>
