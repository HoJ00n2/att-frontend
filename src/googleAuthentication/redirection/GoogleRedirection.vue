<template>
  <div></div>
</template>

<script>
import { mapActions } from 'vuex'

const googleAuthenticationModule = 'googleAuthenticationModule'
const accountModule = 'accountModule'
const authenticationModule = 'authenticationModule'

export default {
  methods: {
    ...mapActions(googleAuthenticationModule, [
      'requestAccessTokenToDjangoRedirection', 'requestGoogleUserInfoToDjango']),
    ...mapActions(accountModule, ['requestEmailDuplicationCheckToDjango']),
    ...mapActions(authenticationModule, ['requestAddRedisAccessTokenToDjango']),

    async setRedirectData () {
        const code = this.$route.query.code
        await this.requestAccessTokenToDjangoRedirection({ code })

        const userInfo = await this.requestGoogleUserInfoToDjango()
        const email = userInfo.email
        console.log('userInfo:', userInfo)
        console.log('email:', userInfo.email)

        const isEmailDuplication = await this.requestEmailDuplicationCheckToDjango(email)
        
        if (isEmailDuplication === true) {
          console.log('기존 가입 고객입니다.')
          const accessToken = localStorage.getItem("accessToken")

          if (accessToken) {
              await this.requestAddRedisAccessTokenToDjango({ email, accessToken })
          } else {
              console.error('AccessToken is missing');
          }
          this.$router.push('/')
        } else {
                console.log('신규 가입 고객입니다.')
                this.$router.push('/account/google-register')
        }
      }
  },
  async created () {
      await this.setRedirectData()
  }
}
</script>