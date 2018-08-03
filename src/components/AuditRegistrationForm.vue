<template>
  <div>
    <div class="row py-3">
      <div class="col-2 pt-1 mt-1 text-secondary">
        検査案件名
      </div>
      <div class="col">
        <input type="text" class="form-control " v-model="name" placeholder="案件名">
      </div>
    </div>
    <div class="row pb-3" v-for="(contact, index) in contacts" :key="index">
      <div class="col-2 pt-1 mt-1 text-secondary">
        担当者 {{ index + 1 }}
      </div>
      <div class="col-4">
        <input type="text" class="form-control" v-model="contact.name" placeholder="名前" @keydown="addInputForm(index)">
      </div>
      <div class="col">
        <input type="email" class="form-control" v-model="contact.email" placeholder="メールアドレス" @keydown="addInputForm(index)" required>
      </div>
    </div>
    <div class="row">
      <div class="col">
        <small class="text-danger">{{ errorMessage }}</small>
      </div>
    </div>
    <div class="row">
      <div class="col text-right">
        <button class="btn btn-primary" @click="addAudit">案件を登録</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'AuditRegistrationForm',
  props: {
    auditApiClient: {
      type: Function,
      required: true,
    },
  },
  data() {
    return {
      contacts: [{ name: '', email: '' }],
      errorMessage: '',
      name: '',
    };
  },
  methods: {
    addInputForm: function addInputForm(index) {
      if (index + 1 === this.contacts.length) {
        this.contacts.push({ name: '', email: '' });
      }
    },
    addAudit: async function addAudit() {
      this.errorMessage = '';

      if (this.name.length === 0) {
        this.errorMessage = '案件名を入力してください';
      }

      const filteredContacts = this.contacts.filter((item) => {
        const condition = item.name.length > 0 || item.email.length > 0;
        return condition;
      });

      filteredContacts.forEach((item) => {
        const emailValidation = /^[\w\.-]+@([\w-]+\.)+\w+$/; // eslint-disable-line
        if (item.email.length === 0) {
          this.errorMessage = `${item.name} さんのメールアドレスを入力してください`;
        } else if (!emailValidation.test(item.email)) {
          this.errorMessage = `${item.email} は正しいメールアドレスではありません`;
        }
      });

      if (filteredContacts.length === 0) {
        this.errorMessage = '担当者の情報を入力してください';
      }

      if (this.errorMessage.length > 0) {
        return;
      }

      try {
        const res = await this.auditApiClient.post('', {
          name: this.name,
          contacts: filteredContacts,
        });
        switch (res.status) {
          case 200: {
            this.contacts = [{ name: '', email: '' }];
            this.errorMessage = '';
            this.name = '';
            window.eventBus.$emit('AUDIT_REGISTERED', res.data);
            break;
          }
          default: {
            this.errorMessage = '検査の登録に失敗しました';
          }
        }
      } catch (e) {
        this.errorMessage = '検査の登録に失敗しました';
      }
    },
  },
};
</script>
