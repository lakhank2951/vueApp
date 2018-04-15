<template>
  <div class="hello">
      <!-- loader initilize app -->
      <div v-if="spinner">
        <h1 v-if="auth">Authenticating</h1>
        <h1 v-if="fetch">Fetching</h1>
        <md-progress-spinner :md-diameter="100" :md-stroke="10" md-mode="indeterminate"></md-progress-spinner>
      </div>

    <!-- display data  -->
      <div v-if="questionData">
        <p v-html="tempcasedata"></p>
        <md-button class="md-raised md-accent" @click="previous" v-if="disableMePrev">previous</md-button>
        <md-button class="md-raised md-primary" @click="next" v-if="disableMeNext">next</md-button>
      </div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      api: 'https://simulationapi.ur-nl.com', //global api URL
      spinner: false,
      auth: false,
      fetch: false,
      disableMeNext : false,
      disableMePrev: false,
      questionData : false ,
      index: 0,
      tempcasedata: [],
      questionsArray:[]   
    }
  },
  methods:{
    // post call & get access token
    getToken(){
      this.$http.post(this.api + '/oauth/token.json',{
         'headers': {
          'content-type': 'application/x-www-form-urlencoded',
        }, 'grant_type': 'password',
        'scope': 'user',
        'client_id': '4874eafd0f7a240625e59b2b123a142a669923d5b0d31ae8743f6780a95187f5',
        'client_secret': '908f6aee4d4cb27782ba55ae0c814bf43419f3220d696206212a29fe3a05cd88',
        'auth_token': 'azd4jXWWLagyb9KzgfDJ'
      })
      .then(response =>{
        // store access token in localStorage
        window.localStorage.setItem('access_token', response.data.access_token);
        this.spinner = true;
        this.auth = true;
        this.fetch = false;
        // call authenticate user function on sucess response 
        this.authUser();
        
      }, error =>{
        this.handleError();
      });
    },

    // authenticate user
    authUser(){
      this.$http.get(this.api + '/users/current.json',{
        headers:{
        'Authorization': 'Bearer '+localStorage.getItem('access_token'),
        'content-type': 'application/json'
        }
      })
      .then(response =>{
        this.spinner = true;
        this.auth = false;
        this.fetch = true;

        // call get study function on success response
        this.getCaseStudy();
      },error =>{
        this.handleError();
      })
    },

    // get case_study
    getCaseStudy(){
      this.$http.get(this.api + '/case_study/companies/58cba141ba169e0eab2657c9/company_case_studies/595c859eba169ec47e4f20d4/user_company_case_studies/595ce021ba169edb9c733e90?include[company_case_study][include]=questions',{
        headers:{
        'Authorization': 'Bearer '+localStorage.getItem('access_token'),
        'content-type': 'application/json'
        }
      })
      .then(response =>{
        this.questionData = true

        this.spinner = false;
        this.fetch = false;
        this.auth = false;

        // store questions array from case study object in questionsArray (temp array)  
        this.questionsArray = response.data.user_company_case_study.company_case_study.questions;
        this.disableMeNext = true;
  
        this.tempcasedata = this.questionsArray[this.index].body
      },error =>{
        this.handleError();
      })
    },

    // get next question
    next() {
      this.index++;
      if (this.index <= this.questionsArray.length - 1) {
        this.tempcasedata = this.questionsArray[this.index].body

        if (this.index === this.questionsArray.length - 1) {
          this.disableMeNext = false;
        }
        else {
          this.disableMePrev = true;
        }
      }
    },

    //get previous question
    previous() {
      this.index--
      if (this.index <= this.questionsArray.length && this.index >= 0) {
        this.tempcasedata = this.questionsArray[this.index].body
        this.disableMeNext = true;
        if (this.index === 0) {
          this.disableMePrev = false;
        }
      } else {
        this.disableMePrev = false;
      }
    },

    // global error handle
    handleError() {
        alert('Error in Fetching Data')   
    }
  },
  mounted(){
    // call function when page fully loaded
    this.getToken();
  }
}
</script>

<!-- Add 'scoped' attribute to limit CSS to this component only -->
<style scoped>

.hello {
    text-align: center;
    margin-top: 10%;
    padding-left: 15%;
    padding-right: 15%;
    color: #58595b;
}

p{
  font-size: 1rem;
}

</style>
