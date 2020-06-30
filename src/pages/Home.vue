<template>
  <div>
    <h1>VueJS And Firebase Authentication</h1>
    <div v-if="authUser">
      <h2>Signed in as {{authUser.email}}</h2>
      <button @click="signOut">Sign Out</button>

      <form @submit.prevent="updateProfile">
        <h2>Update Profile</h2>
        <input v-model="displayName" placeholder="Your name here...">
        <input v-model="photoUrl" placeholder="e.g https://picz.ge/213.png">
        <button>Update</button>
      </form>

      <form @submit.prevent="updateEmail">
        <h2>Update Email</h2>
        <input v-model="email" placeholder="Your email">
        <button>Update</button>
      </form>

      <form @submit.prevent="updatePassword">
        <h2>Update Password</h2>
        <input type="password" v-model="newPassword" placeholder="Your new password">
        <button>Update</button>
      </form>
    </div>

    <div v-else>
      <div>
        <h3>Sign In With GitHub</h3>
        <button @click="signInWithGitHub">@GitHub</button>
      </div>

      <form @submit.prevent="register">
        <h2>Register</h2>
        <input type="email" v-model="email" placeholder="Type your email">
        <input type="password" v-model="password" placeholder="Fill the password">
        <br>
        <button type="submit">Register</button>
      </form>

      <form @submit.prevent="signIn">
        <h2>Sign In</h2>
        <input type="email" v-model="email" placeholder="Type your email">
        <input type="password" v-model="password" placeholder="Tyoe your password">
        <br>
        <button type="submit">Log in</button>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
      email: "",
      password: "",
      displayName: null,
      photoUrl: null,
      newPassword: null,
      authUser: null
    };
  },

  methods: {
    register() {
      window.firebase
        .auth()
        .createUserWithEmailAndPassword(this.email, this.password)
        // .then((user) => this.authUser = user)
        .catch(error => alert(":( " + error.message));
    },

    signIn() {
      window.firebase
        .auth()
        .signInWithEmailAndPassword(this.email, this.password)
        // .then((user) => this.authUser = user)
        .catch(error => alert(":( " + error.message));
    },

    signOut() {
      window.firebase.auth().signOut();
      // .then(() => this.authUser = null);
    },

    signInWithGitHub() {
      const provider = new window.firebase.auth.GithubAuthProvider();
      window.firebase
        .auth()
        .signInWithPopup(provider)
        .then(data => console.log(data.user, data.credential.accessToken))
        .catch(error => alert(":( " + error.message));
    },

    updateProfile() {
      this.authUser
        .updateProfile({
          displayName: this.diplayName,
          photoURL: this.photoUrl
        })
        .then(data => console.log(data))
        .catch(error => alert(":( " + error.message));
    },

    updateEmail() {
      this.authUser
        .updateEmail(this.email)
        .catch(error => alert(":( " + error.message));
    },

    updatePassword() {
      this.authUser
        .updatePassword(this.newPassword)
        .then(()=> this.newPassword = null)
        .catch(error => alert(":( " + error.message));
    }
  },

  created() {
    window.firebase.auth().onAuthStateChanged(user => {
      this.authUser = user;
      if (user) {
        this.displayName = user.displayName;
        this.photoUrl = user.photoURL;
        this.email = user.email;
      }
    });
  }
};
</script>

<style>
</style>
