<script setup>
import SignIn from "./components/SignIn.vue";
import FamousPoets from "./components/FamousPoets.vue";
import PocketBase from "pocketbase";
</script>

<template>
  <header>
    <img
      alt="Poetry"
      class="logo"
      src="./assets/logo.png"
      width="125"
      height="125"
    />
    <div class="wrapper" id="signOut">
      <div><SignIn msg="Poet ! Tell us who you are !" /></div>
      <label>email: </label><br />
      <input
        type="email"
        required
        id="email"
        placeholder="username@domain.tld"
      /><br />
      <label>password: </label><br />
      <input type="password" required id="passwd" /><br />
      <button v-on:click="register()">Email SignUp</button>
      <button v-on:click="login()">Email SignIn</button>
      <button v-on:click="googlelogin()">Google SignIn</button>
      <p><label id="status"> You are not yet connected </label><br /></p>
    </div>
    <div class="hidden" id="addPoem">
      <div><SignIn msg="Write your poem !" /></div>
      <h3>The poem remains private, until you make it public</h3>
      <label>Poem's title</label><br />
      <input
        type="text"
        required
        name="title"
        id="title"
        placeholder="edit me"
      /><br />
      <label>Poem's content</label><br />
      <textarea
        required
        name="content"
        id="content"
        placeholder="edit me"
        rows="10"
        cols="50"
      >
Your poem here ...
      </textarea>
      <br />
      <label>Illustration: </label>

      <input
        type="file"
        id="file"
        name="file"
        placeholder="my file"
        accept="image/png, image/jpeg"
      /><br />
      <!--<img id="illustration" src="./assets/null.png" alt="poem illustration" width="75" height="75"/><br>-->
      <input type="checkbox" id="notpublic" value="true" />
      <label>Private poem</label>
      <br /><button v-on:click="createPoem()">Add the poem</button>
      <button v-on:click="fetchPoems()">List of poems</button><br />
      <label
        for="poemtitle"
        id="poemtitle"
        style="color: teal; font-weight: 500"
      >
        ...
      </label>
      <img
        id="poemillustration"
        src="./assets/null.jpg"
        alt="poem illustration"
        width="75"
        height="75"
        style="background-color: gray"
      /><br />
      <textarea id="poemcontent" readonly rows="10" cols="50"> ... </textarea>
      <br />
      <button v-on:click="nextPoem()">Next poem</button><br />
    </div>
  </header>

  <main>
    <FamousPoets />
  </main>
</template>

<script>
var connected = false;
var pocketbase_ip = "";
//if (import.meta.env.MODE === "production")
pocketbase_ip = "https://pocketbase.hmabed.tech:443";
//else pocketbase_ip = "http://127.0.0.1:8090";
const pb = new PocketBase(pocketbase_ip);
var currentUser;
export default {
  methods: {
    //this method allows a new user to sign up the system. Once done, the user receives an email
    //asking for account validation. Once the validation made the user is added to the system
    async login() {
      await pb
        .collection("users")
        .authWithPassword(
          document.getElementById("email").value,
          document.getElementById("passwd").value
        );

      if (pb.authStore.isValid) {
        document.getElementById("status").innerHTML = "You are now logged in";
        connected = true;
        currentUser = pb.authStore.model;
        document.getElementById("signOut").style.visibility = "hidden";
        document.getElementById("addPoem").style.visibility = "visible";
      }
    },
    //this method allows the already registred user to log in the system.
    async register() {
      currentUser = await pb.collection("users").create({
        email: document.getElementById("email").value,
        password: document.getElementById("passwd").value,
        passwordConfirm: document.getElementById("passwd").value,
        name: "John Di",
      });
      if (currentUser) {
        document.getElementById("status").innerHTML =
          "Wainting for your email validation ...";
        await pb
          .collection("users")
          .requestVerification(document.getElementById("email").value);
      }
    },
    async googlelogin() {
      await pb.collection("users").authWithOAuth2({ provider: "google" });
      if (pb.authStore.isValid) {
        document.getElementById("status").innerHTML = "You are now logged in";
        connected = true;
        currentUser = pb.authStore.model;
        document.getElementById("signOut").style.visibility = "hidden";
        document.getElementById("addPoem").style.visibility = "visible";
      }
    },
    async createPoem() {
      const record = await pb.collection("poems").create({
        title: document.getElementById("title").value,
        content: document.getElementById("content").value,
        private: document.getElementById("notpublic").value,
        email: currentUser.email,
        illustration: document.getElementById("file").files[0],
      });
    },
    async fetchPoems() {
      //call a request filtering all accessible poems
      //extract the number of readable poems
      //if the number of readable poems is not null
      //then display the first one
      //      document.getElementById('poemtitle').innerHTML=
      //      document.getElementById('poemcontent').value=
      //      document.getElementById('poemillustration').src=
      //store the indexof the currently displayed poem
    },
    //this method allows the already registred user to log in the system.
  },
  nextPoem() {
    //if the current displayed poem is not the last
    //then
    //skip to the next poem
    //and display the new current poem
    //document.getElementById('poemtitle').innerHTML=
    //document.getElementById('poemcontent').value=
    //document.getElementById('poemillustration').src=
  },
};
</script>

<style>
@import "./assets/base.css";

header .hidden {
  visibility: hidden;
  overflow: hidden;
  display: flex;
  display: inline-block;
  place-items: flex-start;
  flex-wrap: wrap;
}

#app {
  max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;

  font-weight: normal;
}

header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

a,
.green {
  text-decoration: none;
  color: hsla(160, 100%, 37%, 1);
  transition: 0.4s;
}

@media (hover: hover) {
  a:hover {
    background-color: hsla(160, 100%, 37%, 0.2);
  }
}

@media (min-width: 1024px) {
  body {
    display: flex;
    place-items: center;
  }

  #app {
    display: grid;
    grid-template-columns: 1fr 1fr;
    padding: 0 2rem;
  }

  header {
    display: inline-block;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  header .wrapper {
    display: flex;
    display: inline-block;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  .logo {
    margin: 0 2rem 0 0;
  }
}
</style>
