<script setup>
import SignIn from "./components/SignIn.vue";
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
      <button v-on:click="githublogin()">Github SignIn</button>
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


      <input
        type="text"
        required
        name="langue"
        id="langue"
        placeholder="edit me"
      /><br />
      <label>Poem's language</label><br />
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
      <input type="checkbox" id="notpublic"/>
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
  import { ref } from 'vue';

  const currentPoemIndex = ref(0);

  var connected = false;
  var pocketbase_ip = "";
  //if (import.meta.env.MODE === "production")
  pocketbase_ip = "https://sharedpoesy.nino-da-silva.fr:443";
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
          document.getElementById("status").innerHTML = "You are now logged in with Google";
          connected = true;
          currentUser = pb.authStore.model;
          document.getElementById("signOut").style.visibility = "hidden";
          document.getElementById("addPoem").style.visibility = "visible";
        }
      },

      async githublogin() {
        await pb.collection("users").authWithOAuth2({ provider: "github" });
        if (pb.authStore.isValid) {
          document.getElementById("status").innerHTML = "You are now logged in with GitHub";
          connected = true;
          currentUser = pb.authStore.model;
          document.getElementById("signOut").style.visibility = "hidden";
          document.getElementById("addPoem").style.visibility = "visible";
        }
      },

      async createPoem() {
        var notpublic = false;
        if (document.getElementById("notpublic").checked){
          notpublic = true;
        }
        const record = await pb.collection("poems").create({
          title: document.getElementById("title").value,
          content: document.getElementById("content").value,
          private: notpublic,
          email: currentUser.email,
          illustration: document.getElementById("file").files[0],
          langue: document.getElementById("langue").value,
        });
      },

      async fetchPoems() {
        const response = await pb.collection('poems').getList(1, 50, { filter: 'email = "' + currentUser.email + '" || private = false' });
        
        // Check if the response contains items
        if (response && response.items && response.items.length > 0) {
          // Assuming that `response.items` is an array of poems
          const firstPoem = response.items[0];

          // Update your HTML elements with the data from the first poem
          document.getElementById('poemtitle').innerHTML = firstPoem.title;
          document.getElementById('poemlangue').innerHTML = firstPoem.langue;
          document.getElementById('poemcontent').value = firstPoem.content;
          
          // The "illustration" property appears to be a file path or URL.
          // You can set the "src" attribute of an image element to display it.
          // document.getElementById('poemillustration').src = firstPoem.illustration;

          document.getElementById('poemillustration').src = `https://sharedpoesy.nino-da-silva.fr/api/files/poems/${firstPoem.id}/${firstPoem.illustration}`;
          
          // You may want to store the index of the currently displayed poem here.
        }
        //store the indexof the currently displayed poem
        currentPoem = 0;
      },
      //this method allows the already registred user to log in the system.

      async nextPoem() {
        currentPoemIndex.value = currentPoemIndex.value + 1;
        console.log(currentPoemIndex.value);
        const response2 = await pb.collection('poems').getList(1, 50, { filter: 'email = "' + currentUser.email + '" || private = false' });
      
        // Vérifiez que this.response est défini et contient des éléments.
    
        // Vérifiez si l'index actuel est inférieur à la longueur du tableau d'éléments.
        if (response2 && response2.items && response2.items.length > 0) {
          const nextPoem = response2.items[currentPoemIndex.value];

          // Mettez à jour vos éléments HTML avec les données du poème suivant.
          document.getElementById('poemtitle').innerHTML = nextPoem.title;
          document.getElementById('poemlangue').innerHTML = nextPoem.langue;
          document.getElementById('poemcontent').value = nextPoem.content;
          document.getElementById('poemillustration').src = `https://sharedpoesy.nino-da-silva.fr/api/files/poems/${nextPoem.id}/${nextPoem.illustration}`;
        }
      },
    }
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