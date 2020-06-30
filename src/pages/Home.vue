<template>
  <div>
    <h1>VueJS And Firebase Authentication</h1>
    <div v-if="authUser">
      <h2><img :src="photoUrl" :alt="displayName" width="50px">
        Signed in as {{authUser.email}}
        <img v-if="linkedGithub" src="https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Ftse1.mm.bing.net%2Fth%3Fid%3DOIP.F6ZlLzRbxeotstT7oMlahAHaHl%26pid%3DApi&f=1" width="30px" />
        <img v-if="linkedPassword" src="https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2Fcdn.onlinewebfonts.com%2Fsvg%2Fdownload_296750.png&f=1&nofb=1" width="30px" />
      </h2>

      <p>🦊 Hello, {{authUser.displayName || 'Friend'}}, we know you like {{favoriteButt || ''}} butts ^3</p>

      <button @click="signOut">Sign Out</button>
      <button v-if="!linkedGithub" @click="linkGithub">Link Github</button>
      <button v-else @click="unlinkGithub">Unlink Github</button>

      <form @submit.prevent="updateCustomDetails">
        <h2>Update Favorite Butt</h2>
        <input v-model="favoriteButt" placeholder="Your favorite but, round, white, blacc,. etc...">
        <button>Update</button>
      </form>
      
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
      favoriteButt: null,
      authUser: null
    };
  },

  computed: {
    linkedGithub() {
      return this.authUser.providerData.find(provider => provider.providerId === 'github.com')
    },

    linkedPassword() {
      return this.authUser.providerData.find(provider => provider.providerId === 'password')
    }
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

    linkGithub() {
      const provider = new window.firebase.auth.GithubAuthProvider();
      this.authUser
        .linkWithPopup(provider)
        .then(data => console.log(data.user, data.credential.accessToken))
        .catch(error => alert(":( " + error.message));
    },

    unlinkGithub() {
      this.authUser.unlink('github.com')
    },

    updateProfile() {
      this.authUser
        .updateProfile({
          displayName: this.displayName,
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
    },

    updateCustomDetails() {
      window.firebase.database().ref('users').child(this.authUser.uid)
        .update({ favoriteButt: this.favoriteButt })
    }
  },

  created() {
    window.firebase.auth().onAuthStateChanged(user => {
      this.authUser = user;
      if (user) {
        this.displayName = user.displayName;
        this.photoUrl = user.photoURL;
        this.email = user.email;
        window.firebase.database().ref('users').child(user.uid).once('value', snap => {
          if (snap.val()) this.favoriteButt = snap.val().favoriteButt;
          // Vue.set(this.authUser, 'favoriteButt', this.favoriteButt)
        })
        .catch(error => alert(":( " + error.message));
      }
    });
  }
};
</script>

<style>
</style>
