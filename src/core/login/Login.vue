<template>
  <farm-main :paddingTop="['xl', 'xxl']" :paddingX="['m', 'xl', 'xxl']">
    <div class="input-group">
      <div class="input-group-prepend">
        <span class="input-group-text">https://</span>
      </div>
      <input
        v-model="farmosUrl"
        :placeholder="$t('Enter your farmOS URL')"
        autofocus
        type="url"
        autocomplete="url"
        class="form-control"
        v-on:input="checkValues"
      >
    </div>
    <br>
    <div class="input-group">
      <input
        v-model="username"
        :placeholder="$t('Enter your username')"
        type="text"
        autocomplete="username"
        class="form-control"
        v-on:input="checkValues"
      >
    </div>
    <br>
    <div class="input-group">
      <input
        v-model="password"
        :placeholder="$t('Enter your password')"
        type="password"
        autocomplete="current-password"
        class="form-control"
        v-on:input="checkValues"
      >
    </div>
    <br>
    <div class="input-group login-submit">
      <div v-if="authPending">
        <icon-spinner/>
      </div>
      <button
        v-else
        :disabled="!this.valuesEntered"
        title="Submit credentials"
        @click="submitCredentials"
        class="btn btn-success btn-navbar"
        type="button"
      >
        {{ $t('Submit credentials')}}
      </button>
    </div>
    <br>
    <p style="textAlign: center">
      {{ $t('Need a server? Check out')}}
      <a href="https://farmos.org/hosting/">{{ $t('hosting options')}}</a>.
    </p>
  </farm-main>
</template>

<script>
export default {
  name: 'Login',
  data() {
    return {
      valuesEntered: false,
      authPending: false,
      username: '',
      password: '',
      farmosUrl: '',
    };
  },

  methods: {
    checkValues() {
      const urlIsValid = process.env.NODE_ENV === 'development' || this.username !== '';
      const usernameIsValid = this.username !== '';
      const passwordIsValid = this.password !== '';
      if (urlIsValid && usernameIsValid && passwordIsValid) {
        this.valuesEntered = true;
      }
    },
    submitCredentials() {
      const payload = {
        farmosUrl: this.farmosUrl,
        username: this.username,
        password: this.password,
        router: this.$router,
      };

      this.authPending = true;

      this.$store.dispatch('authorize', payload)
        .then(() => {
          this.authPending = false;
          this.$store.commit('setLoginStatus', true);
          return this.$store.dispatch('updateFieldModules', this.$router);
        })
        .then(res => this.$store.dispatch('updateUserAndSiteInfo', res))
        .then(res => this.$store.dispatch('updateFarmResources', res));
    },

  },
  created() {
    this.farmosUrl = localStorage.getItem('host')?.replace(/(^\w+:|^)\/\//, '') || '';
  },
};

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

 .login-submit {
   justify-content: center;
 }

</style>
