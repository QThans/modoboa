<template>
<creation-form
  ref="form"
  :title="'New alias'|translate"
  :steps="steps"
  :form-getter="getForm"
  :form-observer-getter="getObserver"
  :validate-object="validateAlias"
  :summary-sections="summarySections"
  @close="close"
  @create="submit"
  >
  <template v-slot:form.general="{ step }">
    <alias-general-form :ref="`form_${step}`" v-model="alias" />
  </template>
  <template v-slot:form.recipients="{ step }">
    <alias-recipient-form :ref="`form_${step}`" v-model="alias.recipients" />
  </template>
</creation-form>
</template>

<script>
import { bus } from '@/main'
import aliases from '@/api/aliases'
import AliasGeneralForm from '@/components/identities/AliasGeneralForm'
import AliasRecipientForm from './AliasRecipientForm'
import CreationForm from '@/components/tools/CreationForm'

export default {
  components: {
    AliasGeneralForm,
    AliasRecipientForm,
    CreationForm
  },
  computed: {
    summarySections () {
      const result = [
        {
          title: this.$gettext('General'),
          items: [
            { key: this.$gettext('Random address'), value: this.alias.random, type: 'yesno' },
            { key: this.$gettext('Address'), value: this.alias.address }
          ]
        }
      ]
      if (this.alias.expire_at !== undefined) {
        result[0].items.push({
          key: this.$gettext('Expire at'), value: this.alias.expire_at
        })
      }
      if (this.alias.description !== undefined) {
        result[0].items.push({
          key: this.$gettext('Description'), value: this.alias.description
        })
      }
      result.push({
        title: this.$gettext('Recipients'),
        items: [
          { key: this.$gettext('Recipients'), value: this.alias.recipients.join(', ') }
        ]
      })
      return result
    }
  },
  data () {
    return {
      alias: this.getInitialForm(),
      steps: [
        {
          name: 'general',
          title: this.$gettext('General')
        },
        {
          name: 'recipients',
          title: this.$gettext('Recipients')
        }
      ]
    }
  },
  methods: {
    close () {
      this.alias = this.getInitialForm()
      this.$refs.form.resetForm()
      this.$emit('close')
    },
    getInitialForm () {
      return {
        random: false,
        enabled: true,
        recipients: []
      }
    },
    getForm (step) {
      return this.$refs[`form_${step}`]
    },
    getObserver (step) {
      return this.$refs[`form_${step}`].$refs.observer
    },
    validateAlias () {
      const data = { ...this.alias }
      if (data.recipients.length === 0) {
        delete data.recipients
      }
      const validation = aliases.validate(data)
      validation.catch(error => {
        if (error.response.status === 409 && error.response.data.id !== undefined) {
          bus.$emit('notification', { msg: this.$gettext('Alias already exists, redirecting to edit page'), type: 'warning' })
          this.$router.push({ name: 'AliasEdit', params: { id: error.response.data.id } })
        }
      })
      return validation
    },
    submit () {
      aliases.create(this.alias).then(resp => {
        bus.$emit('notification', { msg: this.$gettext('Alias created') })
        this.$emit('created')
        this.close()
      })
    }
  },
  mounted () {
    this.$store.dispatch('identities/fetchDomains')
  }
}
</script>
