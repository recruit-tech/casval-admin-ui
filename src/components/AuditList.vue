<template>
  <div class="pb-5">
    <table class="table table-hover mb-0">
      <thead>
        <tr>
          <th scope="col">案件名</th>
          <th scope="col">連絡先</th>
          <th scope="col">登録日</th>
          <th scope="col">最終更新日時</th>
          <th scope="col">アクション</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(audit, index) in audits" :key="index">
          <td><a :href="`${origin}#/${audit.id}`" target="_blank">{{audit.name}}</a></td>
          <td><span v-for="(contact, index) in audit.contacts" :key="index">{{contact.name}}<br></span></td>
          <td>{{displayDateTime(audit.created_at)}}</td>
          <td>{{displayDateTime(audit.updated_at)}}</td>
          <td>
            <a :href="generateMailURL(audit)">担当者に連絡</a><br>
            <a href="#" @click.prevent="remandAudit(index, audit.id)" v-if="audit.submitted">
              提出を差し戻す
            </a>
          </td>
        </tr>
      </tbody>
    </table>
    <hr class="mt-0">
    <button class="btn btn-sm btn-outline-secondary float-right" @click="loadAuditIndex()">さらに見る</button>
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
      return m.format('YYYY年MM月DD日 (ddd) HH:mm:ss');
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
            this.errorMessage = '検査案件の差し戻しに失敗しました';
          }
        }
      } catch (e) {
        this.errorMessage = '検査案件の差し戻しに失敗しました';
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
            this.errorMessage = '検査結果の読み込みに失敗しました';
        }
      } catch (e) {
        this.errorMessage = '検査結果の読み込みに失敗しました';
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
