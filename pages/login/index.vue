<template>
  <div>
    <div class="overlay" v-if="loading">
      <div style="margin: 20px">
        <b-spinner variant="primary" label="Spinning"></b-spinner>
        <div>Loading...</div>
      </div>
    </div>
    <div id="row">
      <div id="col1" class="bg-image d-none d-lg-block">
        <MswaliExplained />
      </div>
      <div id="col2">
        <div style="margin: 0 auto; min-height: 100vh">
          <div class="centered-container">
            <div>
              <div>
                <LogoPurple class="d-block d-sm-none" />
                <br />
                <div class="player-container"></div>
              </div>
              <br />
              <div class="heading2">Welcome to mSwali</div>
              <div class="text1">Enter your number to continue</div>
              <br />
              <div>
                <input
                  class="rounded-border-input"
                  type="number"
                  placeholder="0712 345 678"
                  v-model="phoneNumber"
                  required
                  style="margin-bottom: 10px"
                  v-on:keyup.enter="checkUserExists"
                />
              </div>
              <div>
                <RoundedCyanLoadingButton
                  buttonText="Proceed"
                  showIcon="true"
                  id="loginbtn"
                  @click="checkUserExists()"
                />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { mapGetters } from "vuex";
import { mapState, mapActions } from "vuex";
import MswaliExplained from "../../components/MswaliExplained.vue";
import RoundedCyanArrowButton from "../../components/Buttons/RoundedCyanArrowButton.vue";
import RoundedCyanLoadingButton from "../../components/Buttons/RoundedCyanLoadingButton.vue";
export default {
  head: {
    title: "Login",
  },
  data() {
    return {
      phoneNumber: this.phoneNumber,
      userName: this.userName,
      loading: false,
    };
  },
  mounted() {
    if (this.$store.state.isAuthenticated) {
      this.$router.push("/home");
    } else {
      this.navigateToLogin();
    }
  },
  computed: {
    ...mapGetters("products", ["maskedPhone"]),
    ...mapState({
      isExistingUser: "isExistingUser",
      loggedinUserPhone: "loggedinUserPhone",
      newUserPhone: "newUserPhone",
    }),
  },
  methods: {
    ...mapActions({
      peristIsExistingUSer: "peristIsExistingUSer",
      peristUserPhone: "peristUserPhone",
      persistupdateNewUserPhone: "persistupdateNewUserPhone",
    }),
    showMissingFieldsToast(toaster, variant = "danger") {
      this.$bvToast.toast("Enter a valid phone number to proceed", {
        title: `Phone number required`,
        variant: variant,
        toaster: toaster,
        solid: true,
      });
    },
    sendOTPErrorToast(toaster, variant = "danger") {
      this.$bvToast.toast("Error while sending OTP", {
        title: `Could not send OTP`,
        variant: variant,
        toaster: toaster,
        solid: true,
      });
    },
    goToSignup() {
      return this.$router.push("/signup");
    },
    navigateToLogin() {
      return this.$router.push("/login");
    },
    validatePhoneNumber(input_str) {
      var re = /^[\+]?[(]?[0-9]{3}[)]?[-\s\.]?[0-9]{3}[-\s\.]?[0-9]{4,6}$/im;

      return re.test(input_str);
    },
    async checkUserExists() {
      const config = {
        headers: {
          Accept: "application/json",
        },
      };
      if (!!this.phoneNumber && this.validatePhoneNumber(this.phoneNumber)) {
        try {
          // start loading
          this.$nuxt.$loading.start();
          const res = await axios.get(
            `/apiproxy/api/get-user&username=mast&account_number=${this.phoneNumber}`,
            config,
          );
          // check if user already exists
          if (!res.data.status) {
            // stop loading
            await this.peristUserPhone(this.phoneNumber);
            await this.$store.commit("updateSignUpPhone", this.phoneNumber);
            this.$router.push("/accept-signup");
            await this.persistupdateNewUserPhone(this.phoneNumber);
          } else {
            // update state that the user is an existing user and not need to show modal
            await this.peristIsExistingUSer(true);
            // send user an OTP and direct them to verify
            const result = await axios.get(
              `/apiproxy/api/generate-otp&msisdn=${this.phoneNumber}`,
              config,
            );
            // stop loading
            await this.peristUserPhone(this.phoneNumber);
            await this.$store.commit("updateSignUpPhone", this.phoneNumber);
            await this.$store.commit("updateSignUpOTP", res);
            this.$router.push("/otp");
          }
        } catch (err) {
          this.sendOTPErrorToast();
          await this.$store.dispatch("delayTwoSeconds");
          window.location.reload();
        }
      } else {
        this.showMissingFieldsToast();
        await this.$store.dispatch("delayTwoSeconds");
        window.location.reload();
      }
    },
  },
  components: {
    MswaliExplained,
    RoundedCyanArrowButton,
    RoundedCyanLoadingButton,
  },
};
</script>

<style>
.bg-image-overlay {
  background-image: url("~/assets/overlay.png");
  background-repeat: no-repeat;
  background-size: cover;
  width: 100%;
  min-height: 100vh;
}
.overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #160d3d;
  opacity: 68%;
  background: url(data:;base64,iVBORw0KGgoAAAANSUhEUgAAAAIAAAACCAYAAABytg0kAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAABl0RVh0U29mdHdhcmUAUGFpbnQuTkVUIHYzLjUuNUmK/OAAAAATSURBVBhXY2RgYNgHxGAAYuwDAA78AjwwRoQYAAAAAElFTkSuQmCC)
    repeat scroll transparent\9; /* ie fallback png background image */
  z-index: 9999;
  /*background-image: url("~/assets/overlay.png");*/
  background-repeat: no-repeat;
  background-size: cover;
  color: white;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}
.row {
  width: 100%;
  display: flex;
  flex-direction: row;
  justify-content: center;
}
#col1 {
  width: 50%;
}
#col2 {
  width: 50%;
}
</style>
