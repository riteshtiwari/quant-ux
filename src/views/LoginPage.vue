<template>
    <div class="MatcLoginPage">
        <!-- Animated background with gradient -->
        <div class="MatcLoginBackground">
            <div class="MatcLoginBackgroundGradient"></div>
            <div class="MatcLoginBackgroundShapes">
                <div class="MatcLoginShape MatcLoginShape1"></div>
                <div class="MatcLoginShape MatcLoginShape2"></div>
                <div class="MatcLoginShape MatcLoginShape3"></div>
            </div>
        </div>

        <div class="MatcLoginPageDialog" v-if="isQuxAuth">
            <div class="MatcLoginPageContainer">
                <!-- Logo/Brand -->
                <div class="MatcLoginHeader">
                    <div class="MatcLoginLogo">
                        <div class="MatcLoginLogoIcon">
                            <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                                <path d="M12 2L13.09 8.26L20 9L13.09 9.74L12 16L10.91 9.74L4 9L10.91 8.26L12 2Z" fill="url(#gradient)"/>
                                <defs>
                                    <linearGradient id="gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                                        <stop offset="0%" style="stop-color:#ff0080"/>
                                        <stop offset="50%" style="stop-color:#8b5cf6"/>
                                        <stop offset="100%" style="stop-color:#00d4ff"/>
                                    </linearGradient>
                                </defs>
                            </svg>
                        </div>
                        <h1 class="MatcLoginTitle">Quant-UX</h1>
                        <p class="MatcLoginSubtitle">Prototype, Test and Learn</p>
                    </div>
                </div>

                <!-- Tab Navigation -->
                <div class="MatcLoginTabs" v-if="!resetToken">
                    <button 
                        :class="['MatcLoginTab', {'MatcLoginTabActive': tab === 'login'}]" 
                        @click="setTab('login')"
                    >
                        LOGIN
                    </button>
                    <button 
                        :class="['MatcLoginTab', {'MatcLoginTabActive': tab === 'signup'}]" 
                        @click="setTab('signup')" 
                        v-if="allowSignUp"
                    >
                        SIGN UP
                    </button>
                </div>

                <!-- Form Content -->
                <div class="MatcLoginContent">
                    <!-- Login Form -->
                    <div v-if="tab === 'login'" class="MatcLoginForm">
                        <div class="MatcFormGroup">
                            <label class="MatcFormLabel">Email</label>
                            <input 
                                class="MatcFormInput" 
                                placeholder="Enter your email" 
                                type="email" 
                                v-model="email"
                            >
                        </div>

                        <div class="MatcFormGroup">
                            <label class="MatcFormLabel">Password</label>
                            <input 
                                class="MatcFormInput" 
                                placeholder="Enter your password" 
                                type="password" 
                                v-model="password" 
                                @keyup.enter="login"
                            >
                        </div>

                        <button class="MatcLoginButton" @click="login">
                            <span>LOGIN</span>
                            <div class="MatcLoginButtonShine"></div>
                        </button>

                        <button 
                            class="MatcLoginLink" 
                            @click="requestPasswordReset" 
                            v-if="hasLoginError"
                        >
                            Forgot Password?
                        </button>

                        <div class="MatcErrorMessage" v-show="errorMessage">{{errorMessage}}</div>
                    </div>

                    <!-- Signup Form -->
                    <div v-if="tab === 'signup'" class="MatcLoginForm">
                        <div class="MatcFormGroup">
                            <label class="MatcFormLabel">Email</label>
                            <input 
                                class="MatcFormInput" 
                                placeholder="Enter your email" 
                                type="email" 
                                v-model="email"
                            >
                        </div>

                        <div class="MatcFormGroup">
                            <label class="MatcFormLabel">Password</label>
                            <input 
                                class="MatcFormInput" 
                                placeholder="Create a password" 
                                type="password" 
                                v-model="password" 
                                @keyup.enter="signup"
                            >
                        </div>

                        <div class="MatcFormGroup MatcFormGroupCheckbox">
                            <label class="MatcCheckboxLabel">
                                <CheckBox v-model="tos" label=""/>
                                <span class="MatcCheckboxText">
                                    I accept the <a href="#/tos.html" target="_blank" class="MatcCheckboxLink">terms of service</a>
                                </span>
                            </label>
                        </div>

                        <button class="MatcLoginButton" @click="signup">
                            <span>SIGN UP</span>
                            <div class="MatcLoginButtonShine"></div>
                        </button>

                        <div class="MatcErrorMessage" v-show="errorMessage">{{errorMessage}}</div>
                    </div>

                    <!-- Reset Password Form -->
                    <div v-if="resetToken" class="MatcLoginForm">
                        <div class="MatcFormGroup">
                            <label class="MatcFormLabel">Email</label>
                            <input 
                                class="MatcFormInput" 
                                placeholder="Enter your email" 
                                type="email" 
                                v-model="email"
                            >
                        </div>

                        <div class="MatcFormGroup">
                            <label class="MatcFormLabel">New Password</label>
                            <input 
                                class="MatcFormInput" 
                                placeholder="Enter new password" 
                                type="password" 
                                v-model="password"
                            >
                        </div>

                        <button class="MatcLoginButton MatcLoginButtonDanger" @click="resetPassword">
                            <span>Set New Password</span>
                            <div class="MatcLoginButtonShine"></div>
                        </button>

                        <div class="MatcErrorMessage" v-show="errorMessage">{{errorMessage}}</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>


<style lang="scss">
    @import "../style/components/login.scss";
    @import '../style/toolbar/tab.scss';
    
    // Force override any existing styles
    .MatcLoginPage {
        background: linear-gradient(135deg, #0a0a0a 0%, #1a0b2e 25%, #16213e 50%, #0f3460 75%, #0a0a0a 100%) !important;
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif !important;
    }
    
    .MatcLoginPageContainer {
        background: rgba(255, 255, 255, 0.05) !important;
        backdrop-filter: blur(20px) !important;
        -webkit-backdrop-filter: blur(20px) !important;
        border: 1px solid rgba(255, 255, 255, 0.1) !important;
        border-radius: 24px !important;
        color: #ffffff !important;
    }
</style>

<script>


import Services from 'services/Services'
import Logger from 'common/Logger'
import CheckBox from '../common/CheckBox.vue'

export default {
  name: "Header",
  mixins: [],
  props: ['user'],
  data: function() {
    return {
        hasLoginError: false,
        resetToken: false,
        email: '',
        password: '',
        tos: false,
        errorMessage: ' ',
        signupInProgress: false,
        tab: 'login',
        config: {}
    }
  },
  computed: {
    isQuxAuth () {
        return Services.getConfig().auth !== 'keycloak'
    },
    allowSignUp () {
        return this.config && this.config.user && this.config.user.allowSignUp === true
    }
  },
  watch: {
    'user' (v) {
      this.logger.log(6, 'watch', 'user >> ' + v.email)
      this.user = v
    }
  },
  components: {
    CheckBox
  },
  methods: {
      setTab (tab) {
        this.tab = tab
        this.errorMessage = ' '
      },
      async resetPassword () {
        this.logger.info('resetPassword', 'enter ', this.email)

        if (this.email.length < 2) {
            this.errorMessage = "Please enter your email"
            return;
        }

        if (this.password.length < 6) {
            this.errorMessage = "Password too short"
            return;
        }

        if (this.resetToken.length < 6) {
            this.errorMessage = "Token is wrong"
            return;
        }

        let result = await Services.getUserService().reset2(this.email, this.password, this.resetToken)
        if (result.type === 'error') {
            this.errorMessage = 'Someything is wrong'
        } else {
            this.errorMessage = ''
            this.resetToken = ''
            this.tab = 'login'
            this.$router.push('/')
        }
 
      },
      async requestPasswordReset () {
        this.logger.info('requestPasswordReset', 'enter ', this.email)
        await Services.getUserService().reset(this.email)
        this.errorMessage = 'Check you mail.'
      },
      async login () {
        this.logger.info('login', 'enter ', this.email)
        var result = await Services.getUserService().login({
            email:this.email,
            password: this.password
        })
        if (result.type == "error") {
            this.$root.$emit("Error", "Wrong login credentials")
            this.errorMessage = "Login is wrong"
            this.hasLoginError = true
        } else {
            this.$emit('login', result);
            this.$root.$emit('UserLogin', result)
            this.hasLoginError = false
        }
      },
      async signup() {
        this.logger.info('signup', 'enter ', this.email)

        if (this.signupInProgress) {
            this.logger.info('signup', 'already in progress')
            return;
        }

     
        if (this.password.length < 6) {
            this.errorMessage = "Password too short"
            return;
        }

        if (this.tos !== true) {
            this.errorMessage = "Please accept terms of service"
            return;
        }

        this.signupInProgress = true
        var result = await Services.getUserService().signup({
            email:this.email,
            password: this.password,
            tos: this.tos
        })
        this.signupInProgress = false
        if (result.type == "error") {
            if (result.errors.indexOf("user.create.domain") >= 0) {
                this.errorMessage = "Not the correct domain"
            } else if (result.errors.indexOf("user.create.nosignup") >=0 ) {
                this.errorMessage = "No sign-ups allowed."
            } else if (result.errors.indexOf("user.email.not.unique") >= 0) {
                this.errorMessage = "Email is taken"
            } else {
                this.errorMessage = "Password too short"
            }
        } else {
            let user = await Services.getUserService().login({
                email:this.email,
                password: this.password,
            })
            this.$emit('login', user);
            this.$root.$emit('UserLogin', user)
            this.logger.log(-1,'signup', 'exit with login', this.email)
        }
      }
  },
  async mounted() {
    this.logger = new Logger('LoginPage')
   	this.resetToken = this.$route.query.id
    if (this.resetToken && this.resetToken.length > 2) {
        this.logger.log(-1,'mounted', 'reset ')
        this.tab = 'reset'
    }

    this.config = Services.getConfig()
    this.logger.log(1,'mounted', 'exit > ')
  }
}
</script>