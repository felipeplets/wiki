<template lang="pug">
  .modal(v-bind:class='{ "is-active": isShown }')
    .modal-background
    .modal-container
      .modal-content
        header.is-blue
          span Create / Authorize User
          p.modal-notify(v-bind:class='{ "is-active": loading }'): i
        section
          label.label Email address:
          p.control.is-fullwidth
            input.input(type='text', placeholder='e.g. john.doe@company.com', v-model='email', autofocus)
        section
          label.label Provider:
          p.control.is-fullwidth
            select(v-model='provider')
              option(value='local') Local Database
              option(value='windowslive') Microsoft Account
              option(value='google') Google ID
              option(value='facebook') Facebook
              option(value='github') GitHub
              option(value='slack') Slack
        section(v-if='provider=="local"')
          label.label Password:
          p.control.is-fullwidth
            input.input(type='password', placeholder='', v-model='password')
        section(v-if='provider=="local"')
          label.label Full Name:
          p.control.is-fullwidth
            input.input(type='text', placeholder='e.g. John Doe', v-model='name')
        footer
          a.button.is-grey.is-outlined(v-on:click='cancel') Discard
          a.button(v-on:click='create', v-if='provider=="local"', v-bind:disabled='loading', v-bind:class='{ "is-disabled": loading, "is-blue": !loading }') Create User
          a.button(v-on:click='create', v-if='provider!="local"', v-bind:disabled='loading', v-bind:class='{ "is-disabled": loading, "is-blue": !loading }') Authorize User
</template>

<script>
  export default {
    name: 'admin-users-create',
    data () {
      return {
        email: '',
        provider: 'local',
        password: '',
        name: '',
        loading: false
      }
    },
    computed: {
      isShown () {
        return this.$store.state.adminUsersCreate.shown
      }
    },
    methods: {
      cancel () {
        this.$store.dispatch('adminUsersCreateClose')
        this.email = ''
        this.provider = 'local'
      },
      create () {
        let self = this
        this.loading = true
        this.$http.post('/admin/users/create', {
          email: this.email,
          provider: this.provider,
          password: this.password,
          name: this.name
        }).then(resp => {
          return resp.json()
        }).then(resp => {
          this.loading = false
          if (resp.ok) {
            this.cancel()
            window.location.reload(true)
          } else {
            self.$store.dispatch('alert', {
              style: 'red',
              icon: 'square-cross',
              msg: resp.msg
            })
          }
        }).catch(err => {
          this.loading = false
          self.$store.dispatch('alert', {
            style: 'red',
            icon: 'square-cross',
            msg: 'Error: ' + err.body.msg
          })
        })
      }
    }
  }
</script>
