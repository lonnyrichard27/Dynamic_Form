<template>
<div>
  <div v-if="wizardInProgress" v-show="asyncState !== 'pending'">
    <!-- keep-alive persists the state -->
    <keep-alive>
      <component
        :is="steps[currentStepNumber - 1]"
        ref="currentStep"
        @update="processStep" 
        :wizard-data="form">
      </component>
    </keep-alive>
    <div class="progress-bar">
      <div :style="`width: ${progress}%;`"></div>
    </div>

    <!-- Actions -->
    <div class="buttons">
      <button
        @click="goBack"
        v-if="currentStepNumber > 1"
        class="btn-outlined"
      >Back
      </button>
      <button
        @click="nextButtonAction"
        class="btn"
        :disabled="!canGoNext"
      >{{isLastStep ? 'Complete Order': 'Next' }}</button>
    </div>

    <pre><code>{{form}}</code></pre>
  </div>
  <div v-else>
    <h2 class="subtitle">We look forward to shipping your first bos!</h2>
  </div>
  <div class="loading-wrapper" v-if="asyncState === 'pending'">
    <div class="loader">
      <!-- <img src="./spinner.svg" alt=""> -->
      <p>Please wait we're hitting our servers</p>
    </div>
  </div>
</div>
</template>

<script>
import { postFormToDB } from "../api";
import FormPlanPicker from './FormPlanPicker'
import FormUserDetails from './FormUserDetails'
import FormReviewOrder from './FormReviewOrder'
import FormAddress from './FormAddress.vue'

export default {
  name: 'FormWizard',
  components: {
    FormPlanPicker,
    FormUserDetails,
    FormAddress,
    FormReviewOrder
  },
  data () {
    return {
      steps: [
        'FormPlanPicker',
        'FormUserDetails',
        'FormAddress',
        'FormReviewOrder'
      ],
      currentStepNumber: 1,
      canGoNext: false,
      form: {
        plan: null,
        email: null,
        name: null,
        password: null,
        address: null,
        recipient: null,
        chocolate: false,
        otherTreat: false,
        asyncState: null
      }
    }
  },
  computed: {
    isLastStep(){
      return this.currentStepNumber === this.length
    },
    wizardInProgress(){
      return this.currentStepNumber <= this.length
    },
    length(){
      return this.steps.length
    },
    progress () {
      return this.currentStepNumber/this.length * 100  
    }
  },
  methods: {
    submitOrder(){
      this.asyncState = 'pending'
      postFormToDB(this.form)
        .then(() => {
          console.log('form submitted', this.form);
          this.asyncState = 'success'
          this.currentStepNumber++
        })
    },
    nextButtonAction () {
      if(this.isLastStep) {
        this.submitOrder()
      } else {
        this.goNext()
      }
    },
    processStep(step){
      // object.assign copies the properties of a data and puts it in another object
      Object.assign(this.form, step.data)
      this.canGoNext = step.valid
    },
    goBack () {
      this.currentStepNumber--
      // the boolean below persists the state in the previous component
      this.canGoNext = true
    },
    goNext () {
      this.currentStepNumber++
      // this.canGoNext = false
      // the nexttick function is used to wait for vue to update the real DOM which accepts a callback
      this.$nextTick(() =>{

        // the below method is that when the user clicke next they can only go forward once but when they click back they can go back twice
        this.$refs.currentStep.submit()
      })
    }
  }
}
</script>
