<template>
  <section class="login">
    <topbar></topbar>
    <img :src="imageURL" alt="" class="welcomeImage">
    <p class="welcomeMessage">
      {{content}}
    </p>
    <h2>Sign In</h2>
    <form @submit.prevent="signIn">

      <!--Replace this with a material component.  Pull names from Airtable Participants-->
      <label >Name
      <v-select :debounce="250" :on-search="getParticipants" v-model="participant" :options="participants" placeholder="Select Participant"></v-select>
      </label>
      <label>
        4 Digit Pin
        <!--Again replace this with a material style component.  The pin number needs to match the pin number of the selected name.  I'm using a number input here because it will always be a four digit number and the numerical keyboard on a phone is easier to use.-->
        <input type="number" v-model="pin" id="pin">
      </label>
    </form>
    <div class="submitButton">
      <button type="button" @click="signIn" :disabled="loading">Sign In</button>
    </div>
  </section>
</template>

<script>
import Airtable from 'airtable'
import { AIRTABLE_APP_ID,AIRTABLE_APP_KEY } from '../../config'
import Auth from '../../Auth'
import VueRouter from 'vue-router'
import Topbar from './common/Topbar'
import vSelect from "vue-select"
import { IronmanMixin } from './mixins'

export default {
  name: 'login',
  mixins: [IronmanMixin],
  components:{
    'topbar': Topbar,
    vSelect
  },
  watch: {
    participant: function(val, oldVal) {
      // Focus to pin when participant is selected
      document.getElementById('pin').focus();
    }
  },
  created: function(){
  },
  mounted: function(){
    // If user is loggedin then redirect it to home page
    if(Auth.userLoggedIn()){
      //const router = new VueRouter();
      this.$router.push('/ironman/home');
    }

    // Configure Airtable
    Airtable.configure({ apiKey: AIRTABLE_APP_KEY });
    this.base = Airtable.base(AIRTABLE_APP_ID);

    // this.getParticipants();
    this.getInfo();
  },
  computed:{
      imageURL: function(){
          if(this.info != null && this.info['fields']['Image'][0]['url']){
              return this.info['fields']['Image'][0]['url'];
          }
          return "";
      },
      content: function(){
          if(this.info != null && this.info['fields']['Text']){
              return this.info['fields']['Text'];
          }
          return "";
      }
  },
  data () {
    return {
      participants: [],
      participant: "",
      pin: "",
      base: null,
      loading: false,
      info: null
    }
  },
  methods: {
    getInfo: function(){
      var currentApp = localStorage.getItem('currentApp')
      currentApp = JSON.parse(currentApp)
      this.info = currentApp;
    },
    signIn: function(){
      var self = this;
      // Validate
      console.log(this.participant, this.pin)
      if(this.participant == "" || this.participant == null || this.pin == ""){
        alert("Participant field and Pin Field can not be empty")
        return false;
      }

      this.loading = true;
      Auth.login(this.participant.value,this.pin).then(function(response){
        if(response == true){
          const router = new VueRouter();
          router.push('/ironman/home');
        }else{
          alert("Unable to login");
        }
        self.loading = false;
      }).catch(function(error){
        console.log(error)
        alert("Unable to login");
        self.loading = false;
      })

    },
    getParticipants: function(search,loading){
      var self = this;
      loading(true)

      var args = {
        view: 'Grid view',
        fields: ['Name','First Name','Last Name'],
        filterByFormula: "SEARCH(LOWER(TRIM('"+search+"')),LOWER(TRIM({Name})))",
      };
      this.base('Participants').select(args).eachPage(function page(records, fetchNextPage) {
        self.participants = []
        records.forEach(function(item,key){
          self.participants.push({
            label: item.fields['Name'],
            value: item.id
          });
        });
        loading(false)
      },function(err){
        loading(false)
      });
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
img.welcomeImage {
  height: 275px;
  width: 357px;
  margin-left: auto;
  margin-right: auto;
}
.login h2 {
  color: #0089d0;
}
.login .submitButton button {
  background-color: #0089d0;
}
</style>
