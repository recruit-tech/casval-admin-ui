<template>
  <div>
    <div class="row py-3">
      <div class="col-2 pt-1 mt-1 text-secondary">
        {{ $t('audit.name') }}
      </div>
      <div class="col">
        <input type="text" class="form-control " v-model="name" :placeholder="$t('audit.name')">
      </div>
    </div>
    <div class="row pb-3" v-for="(contact, index) in contacts" :key="index">
      <div class="col-2 pt-1 mt-1 text-secondary">
        {{ $t('audit.contact') }} {{ index + 1 }}
      </div>
      <div class="col-4">
        <input type="text" class="form-control" v-model="contact.name" :placeholder="$t('audit.contact-name')" @keydown="addInputForm(index)">
      </div>
      <div class="col">
        <input type="email" class="form-control" v-model="contact.email" :placeholder="$t('audit.contact-email')" @keydown="addInputForm(index)" required>
      </div>
    </div>
    <div class="row">
      <div class="col">
        <small class="text-danger">{{ errorMessage }}</small>
      </div>
    </div>
    <div class="row">
      <div class="col text-right">
        <button class="btn btn-primary" @click="addAudit">{{ $t('audit.register') }}</button>
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
        this.errorMessage = this.$i18n.t('audit.error-no-audit-name');
      }

      const filteredContacts = this.contacts.filter((item) => {
        const condition = item.name.length > 0 || item.email.length > 0;
        return condition;
      });

      filteredContacts.forEach((item) => {
        const emailValidation = /^[\w\.-]+@([\w-]+\.)+\w+$/; // eslint-disable-line
        if (item.email.length === 0) {
          this.errorMessage = this.$i18n.t('audit.error-invalid-name', { name: item.name });
        } else if (!emailValidation.test(item.email)) {
          this.errorMessage = this.$i18n.t('audit.error-invalid-email', { email: item.email });
        }
      });

      if (filteredContacts.length === 0) {
        this.errorMessage = this.$i18n.t('audit.error-no-contact');
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
            window.location.reload(true);
            break;
          }
          default: {
            this.errorMessage = this.$i18n.t('audit.error-general');
          }
        }
      } catch (e) {
        this.errorMessage = this.$i18n.t('audit.error-general');
      }
    },
  },
};
</script>
