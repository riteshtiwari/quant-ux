<template>
  <div class="MatcHeader" id="" style="background: rgba(255, 255, 255, 0.05) !important; backdrop-filter: blur(20px) !important; -webkit-backdrop-filter: blur(20px) !important; border: 1px solid rgba(255, 255, 255, 0.1) !important; border-radius: 16px !important; box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3) !important; color: #ffffff !important;">
 
      <div class="MatcHeaderLeft">
     
        <a href="#/" ref="myPrototype" style="color: rgba(255, 255, 255, 0.9) !important; text-decoration: none !important;">
          <img src="../style/img/QUXLogoBlack.svg" class="MatcHeaderLogo" ref="logo" style="filter: brightness(0) invert(1) !important;">
          Quant-UX
        </a>

      </div>
      <div class="container MatcHeaderCenter">
        <div class="MatcHeaderCenterLeft">
         
      
        </div>
        <div class="MatcHeaderCenterRight">
       
        </div>
      </div>
      <div class="MatcHeaderRight">
   
          <LanguagePicker @change="setLanguage" />
        <!--  
          <a class="" href="#/help.html">
            <QIcon icon="Book" :tooltip="$t('header.tooltip.documentation')"/>
          </a>
         <AccountButton :user="user"/> -->
        <!-- <a class="" href="#/my-account.html">
          <QIcon icon="Account"/>
          {{ $t('header.my-account') }}
        </a>
        <a class="" href="#/logout.html">{{ $t('header.logout') }}</a>
      -->
       
        <a class="" href="#/logout.html" style="color: rgba(255, 255, 255, 0.9) !important; text-decoration: none !important; padding: 8px 12px !important; border-radius: 8px !important; transition: all 0.2s ease !important;">
          <QIcon icon="Logout"/>
        </a>
       
      </div>

</div>
</template>

<style lang="scss">
@import "../style/components/menu.scss";
@import "../style/components/studio-glassmorphism.scss";
</style>


<script>

import Services from 'services/Services'
import Logger from 'common/Logger'
import hash from "dojo/hash";
import LanguagePicker from "page/LanguagePicker";
// import AccountButton from 'page/AccountButton'
import QIcon from 'page/QIcon'
import _Tooltip from "common/_Tooltip";

export default {
  name: "Header",
  mixins: [_Tooltip],
  props: ['user'],
  data: function () {
    return {
    }
  },
  watch: {
    'user'(v) {
      this.logger.log(6, 'watch', 'user >> ' + v.email)
      this.user = v
    }
  },
  components: {
    'LanguagePicker': LanguagePicker,
    'QIcon': QIcon,
    // 'AccountButton': AccountButton
  },
  methods: {
    setLanguage(language) {
      this.logger.log(-1, "setLanguage", "entry", language);
      Services.getUserService().setLanguage(language)
      this.$root.$i18n.locale = language
      this.$root.$emit('Success', this.$i18n.t('common.language-changed'))

    },

    logout() {
      this.logger.log(2, "logout", "entry");
      Services.getUserService().logout()
      this.$emit('logout', Services.getUserService().GUEST)
      hash("/", true);
    }
  },
  async mounted() {
    this.logger = new Logger('Header')
    this.logger.log(7, 'mounted', 'exit >> ' + this.user.email)

    this.addTooltip(this.$refs.logo, this.$t("header.tooltip.my-prototypes"))
    this.addTooltip(this.$refs.myPrototype, this.$t("header.tooltip.my-prototypes"))
  }
}
</script>

