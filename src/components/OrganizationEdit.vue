<template>
  <div>
    <el-button type="primary" @click="createContact">Create Contacts</el-button>
    <el-button type="primary" @click="createLink">Create Link</el-button>
    <el-card class="form">
      <form-schema ref="formSchema" :schema="schema" v-model="model">
        <el-button type="primary" @click="submit">Edit</el-button>
      </form-schema>
    </el-card>
  </div>
</template>
<script>
import FormSchema from 'vue-json-schema'
import schema from '../schema/organization_form.json'
import { HTTP } from '../http-common.js'
import objectDiff from '../objDiff.js'
import { getLanguage } from '../utils.js'

FormSchema.setComponent('form', 'el-form', ({ vm }) => {
  const labelPosition = 'top'
  const labelWidth = '120px'
  const model = vm.data
  const rule = {}

  vm.fields.forEach((field) => {
    const type = field.schemaType
    const required = field.required
    const message = field.title
    const trigger = 'blur'

    rule[field.name] = { type, required, message, trigger }
  })

  return { labelWidth, rule, model, labelPosition }
})

FormSchema.setComponent('label', 'el-form-item', ({ field }) => ({
  prop: field.name
}))

FormSchema.setComponent('text', 'el-input')

export default {
  data () {
    return {
      schema: schema,
      model: {}
    }
  },
  watch: {
    '$route': function (to, from) {
      this.fetch_entity(this.$route.params.entity_id)
    }
  },
  created () {
    console.log(this.$store.state.loggedIn)
    this.fetch_entity(this.$route.params.entity_id)
  },
  methods: {
    submit (e) {
      var entityId = this.$route.params.entity_id
      var language = getLanguage()
      var url = '/' + language + '/organizations/' + entityId
      var loggedIn = this.$store.state.loggedIn
      console.log(loggedIn)
      if (loggedIn) {
        this.$refs.formSchema.form().validate((valid) => {
          if (valid) {
            console.log(JSON.stringify(this.model))
            var newObj = objectDiff(this.model)
            console.log(JSON.stringify(newObj))
            this.$refs.formSchema.clearErrorMessage()
            HTTP.put(url, newObj)
              .then(response => {
                console.log(response)
              })
              .catch(e => {
                console.log(e)
              })
          } else {
            this.$refs.formSchema.setErrorMessage('Please Fill in the form')
            return false
          }
        })
      } else {
        this.$router.push('/login')
      }
    },
    fetch_entity (entityId) {
      var language = getLanguage()
      var url = '/' + language + '/organizations/' + entityId
      HTTP.get(url)
        .then(response => {
          this.model = response.data.result
          console.log(this.model)
        })
        .catch(e => {
          console.log(e)
        })
    },
    createContact () {
      var entityId = this.$route.params.entity_id
      var url = '/organizations/' + entityId + '/contact_details/create'
      this.$router.push(url)
    },
    createLink () {
      var entityId = this.$route.params.entity_id
      var url = '/organizations/' + entityId + '/links/create'
      this.$router.push(url)
    }
  },
  components: {
    FormSchema
  }
}
</script>
<style scoped>
  .form {
    text-align: left;
    width: 600px;
    margin: auto;
  }
  .el-form {
    text-align: left;
  }
</style>
