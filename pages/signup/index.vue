<template>
  <div class="row">
    <div id="col1" class="bg-image d-none d-sm-block">
      <MswaliExplained />
    </div>
    <div id="col2">
      <div style="margin: 0 auto; min-height: 100vh">
        <div class="centered-container">
          <div style="width: 240px">
            <LogoPurple class="d-block d-sm-none" />
            <br />
            <div class="player-container"></div>
            <br />
            <div class="heading2">Let's get to knw you</div>
            <div class="text1">Enter user name for {{ this.phoneNumber }}</div>
            <br />
            <input
              class="rounded-border-input"
              type="text"
              v-model="userName"
              placeholder="Username"
              style="margin-block: 20px"
              v-on:keyup.enter="signUpuser"
            />
            <RoundedCyanLoadingButton
              buttonText="Sign Up"
              showIcon="true"
              @click="signUpuser()"
            />

            <div class="subheading5" style="color: #bbb; padding-top: 10px">
              Have an account already?
              <a href="/login" class="subheading5">LOGIN</a>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import MswaliExplained from "../../components/MswaliExplained.vue";
import RoundedCyanLoadingButton from "../../components/Buttons/RoundedCyanLoadingButton.vue";
export default {
  head: {
    title: "Sign Up",
  },
  data() {
    return {
      status: "not_accepted",
      show_error: false,
      loading: false,
      phoneNumber: this.$store.state.newUserPhone,
      userName: "",
    };
  },
  mounted() {
    if (this.$store.state.isAuthenticated) {
      this.$router.push("/home");
    } else {
      return;
    }
  },
  methods: {
    showMissingFieldsToast(toaster, variant = "danger") {
      this.$bvToast.toast("Make sure you have filled up all fields", {
        title: `All fields are required!`,
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

    validatePhoneNumber(input_str) {
      var re = /^[\+]?[(]?[0-9]{3}[)]?[-\s\.]?[0-9]{3}[-\s\.]?[0-9]{4,6}$/im;

      return re.test(input_str);
    },
    async signUpuser() {
      const config = {
        headers: {
          Accept: "application/json",
        },
      };
      if (
        !!this.phoneNumber &&
        !!this.userName &&
        this.validatePhoneNumber(this.phoneNumber)
      ) {
        try {
          const registerUser = await axios.post(
            `/apiproxy/api/create-user&account_number=${this.phoneNumber}&username=${this.userName}`,
            config,
          );
          // send user an OTP and direct them to verify
          const result = await axios.get(
            `/apiproxy/api/generate-otp&msisdn=${this.phoneNumber}`,
            config,
          );
          await this.$store.commit("updateSignUpPhone", this.phoneNumber);
          await this.$store.commit("updateSignUpName", this.userName);
          await this.$store.commit("updateSignUpOTP", result);
          await this.$router.push("/otp");
        } catch (err) {
          this.sendOTPErrorToast();
        }
      } else {
        this.showMissingFieldsToast();
      }
    },
  },
  components: { MswaliExplained, RoundedCyanLoadingButton },
};
</script>
<style>
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
</style>
