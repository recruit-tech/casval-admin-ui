<template>
  <div id="modal-authenticate" class="modal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">管理者の認証</h5>
        </div>
        <div class="modal-body">
          <p>
            管理者パスワードを入力してください。
            <small class="form-text text-danger">{{ errorMessage }}</small>
          </p>
          <div class="container mt-4">
            <div class="row">
              <div class="col"></div>
              <div class="col-7">
                <input type="password" class="form-control" v-model="password">
              </div>
              <div class="col"></div>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-primary" @click="authenticate">認証</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'ModalAuthenticate',
  data() {
    return {
      password: '',
      errorMessage: '',
    };
  },
  computed: {
    authApiClient: function createAuditApiClient() {
      return axios.create({
        baseURL: `${process.env.VUE_APP_AUTH_API_ENDPOINT}`,
        timeout: process.env.VUE_APP_API_TIMEOUT,
        validateStatus: () => true,
      });
    },
  },
  methods: {
    authenticate: async function applyRestriction() {
      try {
        const res = await this.authApiClient.post(null, {
          password: this.password,
        });
        switch (res.status) {
          case 200:
            window.localStorage.setItem('token', res.data.token);
            window.location.reload(true);
            break;
          default:
            this.errorMessage = '認証失敗';
        }
      } catch (e) {
        this.errorMessage = '認証失敗';
      }
    },
  },
};
</script>
