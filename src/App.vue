<template>
  <div id="app">
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
      <img src="./assets/logo-white.svg" class="logo mr-3">
      <div class="collapse navbar-collapse" id="navbarNavDropdown">
        <ul class="navbar-nav">
          <li class="nav-item" :class="{ 'active': currentTab==='audit' }" @click="changeTab('audit')">
            <a class="nav-link" href="#">検査管理</a>
          </li>
          <li class="nav-item" :class="{ 'active': currentTab==='vulnerability' }" @click="changeTab('vulnerability')">
            <a class="nav-link" href="#">脆弱性管理</a>
          </li>
        </ul>
      </div>
    </nav>
    <audit-management v-if="currentTab==='audit'"></audit-management>
    <vulnerability-management v-else-if="currentTab==='vulnerability'"></vulnerability-management>
  </div>
</template>

<script>
import axios from 'axios';
import AuditManagement from './components/AuditManagement.vue';
import VulnerabilityManagement from './components/VulnerabilityManagement.vue';

export default {
  name: 'app',
  data() {
    return {
      currentTab: 'audit',
    };
  },  
  components: {
    AuditManagement,
    VulnerabilityManagement,
  },
  methods: {
    changeTab: function changeTab(tab) {
      this.currentTab = tab;
    },
  },
  computed: {
    auditApiClient: function createAuditApiClient() {
      return axios.create({
        baseURL: `${process.env.VUE_APP_AUDIT_API_ENDPOINT}/${this.auditId}`,
        timeout: process.env.VUE_APP_API_TIMEOUT,
        headers: { Authorization: `Bearer ${this.token}` },
        validateStatus: () => true,
      });
    },
  },
};
</script>

<style>
html {
  height: 100%;
}
body {
  height:100%;
  background-color: #f0f0f0;
}
</style>

<style scoped>
img.logo {
  height: 1.0rem;
}
</style>
