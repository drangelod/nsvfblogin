<template>
	<Page>
    <ScrollView>
		<FlexboxLayout class="page">
			<StackLayout class="form">
				<Image class="logo" src="~/assets/images/NativeScript-Vue.png" />
				<Label class="header" text="APP NAME" />

				<StackLayout class="input-field" marginBottom="25">
					<TextField class="input" hint="Email" keyboardType="email" autocorrect="false" autocapitalizationType="none" v-model="user.email"
					 returnKeyType="next" @returnPress="focusPassword" fontSize="18" />
					<StackLayout class="hr-light" />
				</StackLayout>

				<StackLayout class="input-field" marginBottom="25">
					<TextField ref="password" class="input" hint="Password" secure="true" v-model="user.password" :returnKeyType="isLoggingIn ? 'done' : 'next'"
					 @returnPress="focusConfirmPassword" fontSize="18" />
					<StackLayout class="hr-light" />
				</StackLayout>

				<StackLayout v-show="!isLoggingIn" class="input-field">
					<TextField ref="confirmPassword" class="input" hint="Confirm password" secure="true" v-model="user.confirmPassword" returnKeyType="done"
					 fontSize="18" />
					<StackLayout class="hr-light" />
				</StackLayout>

				<Button :text="isLoggingIn ? 'Log In' : 'Sign Up'" @tap="submit" class="btn btn-primary m-t-20" />
        <Button v-show="isLoggingIn" :text="'\uf09a' +' Facebook login'" @tap="loginFacebook" class="fab btn btn-active" />
        <Button v-show="isLoggingIn" :text="'\uf1a0' +' Google login' " @tap="loginGoogle" class="fab btn btn-active" />
				<Label v-show="isLoggingIn" text="Forgot your password?" class="login-label" @tap="forgotPassword" />
			</StackLayout>

			<Label class="login-label sign-up-label" @tap="toggleForm">
	          <FormattedString>
	            <Span :text="isLoggingIn ? 'Don’t have an account? ' : 'Back to Login'" />
	            <Span :text="isLoggingIn ? 'Sign up' : ''" class="bold" />
	          </FormattedString>
	        </Label>
		</FlexboxLayout>
    </ScrollView>
	</Page>
</template>
<script>
// A stub for a service that authenticates users.
import firebase from "nativescript-plugin-firebase";
const userService = {
  async register(user) {
    return await firebase.createUser({
      email: user.email,
      password: user.password
    });
  },
  async login(user) {
    return await firebase.login({
      type: firebase.LoginType.PASSWORD,
      passwordOptions: {
        email: user.email,
        password: user.password
      }
    });
  },
  async loginFacebook(user) {
    console.log("facebook login")
    await firebase
      .login({
        type: firebase.LoginType.FACEBOOK,
        facebookOptions: {
          // full list: https://developers.facebook.com/docs/facebook-login/permissions/
          scope: ["public_profile", "email"] // optional: defaults to ['public_profile', 'email']
        }
      })
      .then(result => {
        console.dir(result)
        return Promise.resolve(JSON.stringify(result));
      })
      .catch(error => {
        console.log("Error logging in with Facebook")
        console.error(error);
        return Promise.reject(error);
      });
  },
  async loginGoogle(user) {
     await firebase
      .login({
        type: firebase.LoginType.GOOGLE       
      })
      .then(result => {
        return Promise.resolve(JSON.stringify(result));
      })
      .catch(error => {
        console.error(error);
        return Promise.reject(error);
      });
  },
  async resetPassword(email) {
    return await firebase.resetPassword({
      email: email
    });
  }
};
// A stub for the main page of your app. In a real app you’d put this page in its own .vue file.
const HomePage = {
  template: `
	<Page>
        <Label class="m-20" textWrap="true" text="You have successfully authenticated. This is where you build your core application functionality."></Label>
	</Page>
	`
};
var LoadingIndicator = require("nativescript-loading-indicator")
  .LoadingIndicator;
var loader = new LoadingIndicator();
export default {
  data() {
    return {
      isLoggingIn: true,
      user: {
        email: "test@test.com",
        password: "tester",
        confirmPassword: "tester"
      }
    };
  },
  methods: {
    toggleForm() {
      this.isLoggingIn = !this.isLoggingIn;
    },
    submit() {
      if (!this.user.email || !this.user.password) {
        this.alert("Please provide both an email address and password.");
        return;
      }
      loader.show();
      if (this.isLoggingIn) {
        this.login();
      } else {
        this.register();
      }
    },
    login() {
      userService
        .login(this.user)
        .then(() => {
          loader.hide();
          this.$navigateTo(HomePage);
        })
        .catch(err => {
          console.error(err);
          loader.hide();          
          this.alert(err);
        });
    },
    loginFacebook() {
      //loader.show();//Don't use this for facebook logins, as the popup covers the UI on IOS
      userService
        .loginFacebook(this.user)
        .then(() => {
        //  loader.hide();
          this.$navigateTo(HomePage);
        })
        .catch((err) => {
          //loader.hide();
          console.error(err);
          this.alert(err)
        });
    },
    loginGoogle(){
        //loader.show();//Don't use this for google logins, as the popup covers the UI on IOS
        userService
        .loginGoogle(this.user)
        .then((result) => {
          //loader.hide();          
          this.$navigateTo(HomePage);          
        })
        .catch((error) => {
          //loader.hide();
          console.error(err);
          this.alert(error)
        });
    },
    register() {
      if (this.user.password != this.user.confirmPassword) {
        loader.hide();
        this.alert("Your passwords do not match.");
        return;
      }
      if (this.user.password.length < 6) {
        loader.hide();
        this.alert("Your password must at least 6 characters.");
        return;
      }
      var validator = require("email-validator");
      if (!validator.validate(this.user.email)) {
        loader.hide();
        this.alert("Please enter a valid email address.");
        return;
      }
      userService
        .register(this.user)
        .then(() => {
          loader.hide();
          this.alert("Your account was successfully created.");
          this.isLoggingIn = true;
        })
        .catch(err => {
          console.error(err);
          loader.hide();
          this.alert(err);
        });
    },
    forgotPassword() {
      prompt({
        title: "Forgot Password",
        message:
          "Enter the email address you used to register for APP NAME to reset your password.",
        inputType: "email",
        defaultText: "",
        okButtonText: "Ok",
        cancelButtonText: "Cancel"
      }).then(data => {
        if (data.result) {
          loader.show();
          userService
            .resetPassword(data.text.trim())
            .then(() => {
              loader.hide();
              this.alert(
                "Your password was successfully reset. Please check your email for instructions on choosing a new password."
              );
            })
            .catch(err => {
              loader.hide();
              this.alert(err);
            });
        }
      });
    },
    focusPassword() {
      this.$refs.password.nativeView.focus();
    },
    focusConfirmPassword() {
      if (!this.isLoggingIn) {
        this.$refs.confirmPassword.nativeView.focus();
      }
    },
    alert(message) {
      return alert({
        title: "APP NAME",
        okButtonText: "OK",
        message: message
      });
    }
  }
};
</script>
	
<style scoped>
.page {
  align-items: center;
  flex-direction: column;
}

.form {
  margin-left: 30;
  margin-right: 30;
  flex-grow: 2;
  vertical-align: middle;
}

.logo {
  margin-bottom: 12;
  height: 90;
  font-weight: bold;
}

.header {
  horizontal-align: center;
  font-size: 25;
  font-weight: 600;
  margin-bottom: 70;
  text-align: center;
  color: #d51a1a;
}

.input-field {
  margin-bottom: 25;
}

.input {
  font-size: 18;
  placeholder-color: #a8a8a8;
}

.input-field .input {
  font-size: 54;
}

.btn-primary {
  height: 50;
  margin: 30 5 15 5;
  background-color: #d51a1a;
  border-radius: 5;
  font-size: 20;
  font-weight: 600;
}

.login-label {
  horizontal-align: center;
  color: #a8a8a8;
  font-size: 16;
}

.sign-up-label {
  margin-bottom: 20;
}

.bold {
  color: #000000;
}
</style>