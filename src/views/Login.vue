<template>
  <v-app>
    <div class="c-target c-target-bg">
      <v-card id="auth-lock-panel" class="auth-lock-panel auth-lock-panel--size shadow-smallest">
        <v-progress-linear :active="loading" :indeterminate="loading" absolute top color="primary"></v-progress-linear>
        <v-container class="scroller">
          <v-card class="noshadow card-invoice-aditional">
            <figure class="disflex">
              <img width="80" src="../assets/img/logo.svg" alt="Parcel logo" />
            </figure>
          </v-card>
          <form ref="form" lazy-validation scrollable class="auth-lock-form" @submit.prevent="postLogin">
            <div data-input-type="text-field" data-layout="layout-item-input">
              <v-col cols="12" class="wrapper-input-5 pt-5 pb-5">
                <div class="wrapper">
                  <div class="diana-input-data">
                    <v-text-field id="" class="diana-input" ref="" solo dense persistent-hint
                      placeholder="jennifer@fake.com" type="email" v-model="user.email" value="jennifer@fake.com"
                      @focus.prevent="focusedPassword = true" @blur.prevent="focusedPassword = false"
                      :rules="[rules.required, rules.email]"></v-text-field>
                  </div>
                </div>
              </v-col>
              <v-col cols="12" class="wrapper-input-5 pb-8">
                <div class="wrapper">
                  <div class="diana-input-data">
                    <v-text-field id="" class="diana-input" ref="" solo dense persistent-hint placeholder="123456789"
                      v-model="user.password" value="123456789" :append-icon="showEye ? 'mdi-eye' : 'mdi-eye-off'"
                      :type="showEye ? 'text' : 'password'" @click:append="showEye = !showEye"
                      @focus.prevent="focusedPassword = true" @blur.prevent="focusedPassword = false"
                      :rules="[rules.password, rules.counter]"></v-text-field>
                  </div>
                </div>
              </v-col>
            </div>
            <div class="wrapper-input-5">
              <div class="wrapper">
                <v-btn type="submit" class="diana-btn diana-btn--bg w-full" elevation="1" :loading="loading"
                  :disabled="loading">
                  <span class="diana-btn--title">Next</span>
                </v-btn>
              </div>
            </div>
          </form>

          <div class="wrapper-input-5">
            <span class="recaptcha">This site is protected by reCAPTCHA and the Google
              <a href="https://policies.google.com/privacy?hl=en" target="_blank">Privacy Policy</a>
              and
              <a href="https://policies.google.com/terms?hl=en" target="_blank">Terms of Service</a>
              apply</span>
          </div>
        </v-container>
      </v-card>
    </div>
  </v-app>
</template>

<script>
import { validationMixin } from "vuelidate";
import { required, email } from "vuelidate/lib/validators";
// import { isEmpty, isBlank } from "@/services/srv_utils.js";
// import srvToasted from "@/services/srv_toasted.js";
// import serviceAxiosGet from "../services/srv_axios";
import ParcelUtils from "@/assets/js/parcel_utils.js";
// import jsCrypto from "crypto-js";
import jsCookie from "js-cookie";
import moment from "moment";

export default {
  mixins: [validationMixin],
  name: "Login",
  metaInfo: {
    title: "SignIn",
  },
  validations: {
    user: {
      email: { required, email },
      password: {
        required,
        containsUppercas: function (value) {
          return /[A-Z]/.test(value);
        },
        containsLowercase: function (value) {
          return /[a-z]/.test(value);
        },
        containsNumber: function (value) {
          return /[0-9]/.test(value);
        },
        containsSpecial: function (value) {
          return /[#?!@$%^*&-]/.test(value);
        },
      },
    },
  },
  middleware: "",
  props: {},
  components: {},
  data: () => ({
    parcelCls: new ParcelUtils(),
    error: null,
    toasted: {
      CUSTOM: "custom",
      DEFAULT: "default",
      INFO: "info",
      ERROR: "error",
      SUCCESS: "success",
      WARNING: "warning",
    },
    loader: null,
    loading: false,
    submitted: false,
    rules: {
      required: (value) => !!value || "Required",
      min: (v) => v.length >= 8 || "Almost 8 characters",
      counter: (value) => value.length <= 15 || "Maximun 15 characters",
      email: (value) => {
        const pattern =
          /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
        return pattern.test(value) || "E-mail is invalid";
      },
      password: (value) => {
        const pattern = /(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).{8,}/;
        return pattern.test(value) || "Required";
      },
    },
    customerProfile: {},
    user: {
      email: "",
      password: "",
    },
    focusedPassword: false,
    focusedEmail: false,
    //
    submitStatus: null,
    submitError: false,
    submitWarning: false,
    submitSuccess: false,
    showEye: false,
    showProgress: false,
    recaptchaToken: null,
    employeesFromJson: [],
    customersFromJson: [],
    rolesFromJson: [],
    employeesEntries: {},
    customersEntries: {},
    rolesEntries: {},
    appStoreEmail: "",
    appStorePassword: "",
    appStoreUserName: "",
    appStoreUserId: "",
    startDate: "",
    lastDate: "",
  }),
  computed: {},
  watch: {
    loading(val) {
      if (!val) return;

      setTimeout(() => (this.loading = false), 3000);
    },
  },
  // Hooks
  beforeCreate() { },
  created() { },
  beforeMount() { },
  async mounted() {

    this.loadRequest();
    this.signOut();

  },
  beforeUpdate() { },
  updated() { },
  beforeDestroy() { },
  destroyed() { },
  methods: {
    loadRequest() { },
    async recaptcha() {
      try {
        /**  load the recaptcha */
        await this.$recaptchaLoaded();
        /** run recaptcha with the action login  */
        this.recaptchaToken = await this.$recaptcha("login");
        /** validate the token */
        console.log(
          `parcel@: The recaptcha/Token is ${this.recaptchaToken ? "TRUE" : "FALSE"
          }`
        );
      } catch (e) {
        this.error = e.message;
      }
    },
    signOut() {
      try {
        this.$store.dispatch("axnAuthenticated", false);
        this.$store.dispatch("axnAuth", null);
        jsCookie.remove("logged_in");
        jsCookie.remove("access_token");
        jsCookie.remove("vuex");
        localStorage.removeItem("logged_in");
        localStorage.removeItem("access_token");
        localStorage.removeItem("_grecaptcha");
        this.error = null;
      } catch (err) {
        this.error = err.message;
      }
    },
    getLastTime() {
      this.lastDate = new Date().getTime();
      this.lastDate = new Date().getTime - this.lastDate;
      moment.defaultFormat = "YYYY.DD.MM HH:mm:ss.SSS";
      this.startDate = moment().format();
    },
    async postLogin() {
      this.loading = !this.loading;
      this.submitted = true;
      this.loading = true;
      this.$v.$touch();

      this.getLastTime();

      setTimeout(() => {
        this.$router.push("/");
      }, 2000);

    },
  },

  // end Hooks
};
</script>

