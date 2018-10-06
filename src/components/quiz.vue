<template>
  <div id="app">
  <h1 id="header">{{ quiz.title }}</h1>
  <!-- index is used to check with current question index -->
  <div v-for="(question, index) in quiz.questions" :key="index">
    <!-- Hide all questions, show only the one with index === to current question index -->
    <div v-show="index === questionIndex">
      <h2><b> {{index + 1}}) {{ question.text }}</b></h2>
      <ol>
        <li v-for="(response, i) in question.responses" :key="i">
          <label>
            <!-- v-model creates binding with userResponses -->
            <el-radio @change="descriptionOfAnswer(response)"
                   :label="response.text"
                   v-model="userResponses[index]"></el-radio>
          </label>
        </li>
      </ol>
      <div v-if="showDescription" id="answerDesc">
        <h3><b>Answer Description:</b></h3>{{description}}
      </div>
      <!-- The two navigation buttons -->
      <!-- Note: prev is hidden on first question -->
      <el-button v-if="questionIndex > 0" v-on:click="prev">Prev.</el-button>
      <el-button v-on:click="next" v-if="questionIndex + 1 !== quiz.questions.length">Next</el-button>
      <el-button type="primary" @click="dialogVisible = true">FINISH</el-button>
    </div>
  </div>
  <el-dialog
    title="Test Submit"
    :visible.sync="dialogVisible"
    width="30%"
    :before-close="handleClose">
    <span>Do you want to submit the test result?</span>
    <span slot="footer" class="dialog-footer">
      <el-button @click="dialogVisible = false">Cancel</el-button>
      <el-button type="primary" @click="userInput = true, dialogVisible = false, questionIndex = quiz.questions.length">Confirm</el-button>
    </span>
  </el-dialog>
  <div v-show="questionIndex === quiz.questions.length && userInput === true">
    <h2 id="finish">
    Quiz finished.
  </h2>
    <p>
      <b>Total score:</b> <span style="color:blue">{{ score() }}</span> / <span>{{ quiz.questions.length }}</span>
    </p>
  </div>
</div>
</template>

<script>
import axios from 'axios'
export default {
  data () {
    return {
      showDescription : false,
      description: '',
      dialogVisible: false,
      quiz: {},
      // Store current question index
      questionIndex: 0,
      userResponses: [],
      userInput: false
    }
  },
  methods: {
    // Go to next question
    next: function() {
      this.showDescription = false
      this.description = ''
      this.questionIndex++;
    },
    // Go to previous question
    prev: function() {
      this.showDescription = false
      this.description = ''
      this.questionIndex--;
    },
    // Return "true" count in userResponses
    score: function() {
      return this.userResponses.filter(function(val) { return val }).length;
    },
    // asking the final submission of the test
    handleClose(done) {
      this.$confirm('Are you sure to close this dialog?')
        .then(_ => {
          done();
        })
        .catch(_ => {});
    },
    descriptionOfAnswer (res) {
      if (res.hasOwnProperty('correct')) {
        this.showDescription = true
        this.description = res.description
      } else {
        this.showDescription = false
        this.description = ''
      }
    }
  },
  mounted () {
    axios.get('https://api.myjson.com/bins/bnhn8')
    .then((res)=>{
      this.quiz = res.data
      // An array initialized with "false" values for each question
      // It means: "did the user answered correctly to the question n?" "no".
      this.userResponses = Array(this.quiz.questions.length).fill(false)
    })
    .catch((err)=>{
      console.log(err)
    }) 
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

#app {
  border: 1px solid;
    padding: 10px;
    box-shadow: 5px 10px #888888;
}

#header {
  font-size: 35px;
  text-transform: uppercase;
  text-decoration: underline
}

#finish {
  color: green
}

#answerDesc{
  border: 1px solid #EEE;
  box-shadow: 2px 2px #888888;
  margin-left: 2%;
  margin-bottom: 20px
}
</style>
