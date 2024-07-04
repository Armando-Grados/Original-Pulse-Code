<template>
  <div class="min-h-screen bg-gray-100 py-6 flex flex-col justify-center sm:py-12">
    <div class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-75 z-50" v-if="showOverlay">
      <div class="bg-white rounded p-5 text-center max-w-lg mx-auto">
        <h2 class="text-2xl font-bold mb-3">Thank You!</h2>
        <p class="text-xl">We're thrilled to hear that you had an awesome experience with us! If you're
            willing, we would greatly appreciate it if you could <a href="https://g.page/r/CVam69BGprYlEBM/review"
              target="_blank" class="text-blue-600 underline">leave us a Google Review</a>. Your positive feedback will
            help others learn about our services and the work we do.</p>
            <p class="text-xl mt-3 font-bold"><a href="https://g.page/r/CVam69BGprYlEBM/review"
              target="_blank" class="text-blue-500 underline">Google Review Link</a><br>or<br>
          <a href="https://evanced.net/" target="_blank" class="text-yellow-500 underline">Maybe Later</a></p>
      </div>
    </div>

    <div class="flex items-center justify-center"><img src="../img/evanced.png"></div>
    <h1 class="text-4xl font-bold mb-10 text-center">We value your feedback :-)</h1>
    <div class="relative py-3 sm:max-w-xl sm:mx-auto">
      <div
        class="absolute inset-0 bg-gradient-to-r from-green-500 to-green-50 shadow-lg transform -skew-y-6 sm:skew-y-0 sm:-rotate-6 sm:rounded-3xl">
      </div>
      <div class="relative px-4 py-10 bg-white shadow-lg sm:rounded-3xl sm:p-20">
        <h2 class="text-2xl font-bold">How are we doing?</h2>
        <div class="mt-5 space-y-3">
          <!-- Great Button -->
          <button :disabled="buttonsDisabled.great" @click="callZapierWebhook('Great, keep up the good work!')"
            class="w-full bg-green-500 hover:bg-green-600 text-white font-bold py-2 px-4 rounded h-20">Great, keep up the
            good work!</button>

          <!-- Okay Button -->
          <button :disabled="buttonsDisabled.ok"
            @click="callZapierWebhook('Okay, but there is room for improvement in some areas.')"
            class="w-full bg-yellow-500 hover:bg-yellow-600 text-white font-bold py-2 px-4 rounded h-20">Okay, but there's
            room for improvement in some areas.</button>

          <!-- Not Satisfied Button -->
          <button :disabled="buttonsDisabled.notok"
            @click="callZapierWebhook('Not satisfied, needs significant improvement.')"
            class="w-full bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded h-20">Not satisfied, needs
            significant improvement.</button>

        </div>
        <div v-if="showMessageNotOk" v-bind:class="{ 'animate__shakeX': showMessage }"
          class="mt-5 animate__animated bg-yellow-200 p-4 rounded border border-yellow-300">
          <p class="text-sm font-semibold">Thank you for your feedback. If you would like to share more about how we can
            improve, please <a href="https://calendly.com/evanced-net/evanced-customer-pulse" target="_blank"
              class="text-blue-600 underline">schedule a call with us</a>. We look forward to hearing from you!</p>
        </div>
        <div v-if="showMessageOk" v-bind:class="{ 'animate__shakeX': showMessage }"
          class="mt-5 animate__animated bg-yellow-200 p-4 rounded border border-yellow-300">
          <p class="text-sm font-semibold">Thank you for sharing your feedback.</p>
        </div>
        <div v-if="showFeedbackForm" class="mt-5">
          <form @submit.prevent="submitFeedback">
            <label class="block font-bold mb-2">Please leave your feedback below:</label>
            <textarea class="border rounded w-full py-2 px-3" rows="5" v-model="feedback"></textarea>
            <button class="bg-blue-500 text-white font-bold py-2 px-4 rounded mt-2" type="submit">Submit Feedback</button>
          </form>
        </div>
      </div>
    </div>
  </div>
  <img id="rocket" src="../img/rocket.png"
    class="hidden w-25 h-auto absolute left-1/2 bottom-10 transform -translate-x-1/2" />
</template>
<style>
@keyframes rocketAnimation {
  0% {
    transform: translateY(0) translateX(-50%);
    opacity: 0;
  }

  100% {
    transform: translateY(-1000px) translateX(400%);
    opacity: 1;
  }
}

</style>
<script>
import confetti from 'canvas-confetti';
import 'animate.css';

export default {
  data() {
    return {
      tite: 'great title',
      showMessageNotOk: false,
      showMessageOk: false,
      buttonsDisabled: {
        great: false,
        ok: false,
        notok: false,
      },
      showFeedbackForm: false,
      feedback: '',
      showOverlay: false,
    };
  },
  methods: {

    async callZapierWebhook(buttonText) {
      const webhookURL = 'https://hooks.zapier.com/hooks/catch/2829215/3uhamgs/';
      const data = {
        id: this.$route.params.id,
        customer_acronym: this.$route.params.customerAcronym,
        button_text: buttonText,
      };

      // Convert the data object to query string parameters
      const queryString = new URLSearchParams(data).toString();

      await fetch(`${webhookURL}?${queryString}`, {
        method: 'GET',
        headers: {
          'Accept': 'application/json',
        },
      });

      if (buttonText === 'Not satisfied, needs significant improvement.') {
        this.showMessageNotOk = true;
      }
      else if (buttonText == 'Great, keep up the good work!') {
        // confetti
        confetti({
          particleCount: 100,
          spread: 70,
          origin: { y: 0.6 },
        });
        // Add rocket animation
        const rocket = document.getElementById('rocket');
        rocket.classList.remove('hidden');
        rocket.style.animation = 'rocketAnimation 2s linear';
        setTimeout(() => {
          rocket.style.animation = '';
          rocket.classList.add('hidden');
        }, 2000);
        // show overlay
        await new Promise(resolve => setTimeout(resolve, 3000));
        this.showOverlay = true;
      }
      else if (buttonText == 'Okay, but there is room for improvement in some areas.') {
        this.showFeedbackForm = true;
      }

      // Disable buttons
      this.buttonsDisabled.ok = true;
      this.buttonsDisabled.notok = true;
      this.buttonsDisabled.great = true;
    },

    submitFeedback() {
      const webhookURL = 'https://hooks.zapier.com/hooks/catch/2829215/3uhnc61/';
      const data = {
        id: this.$route.params.id,
        customer_acronym: this.$route.params.customerAcronym,
        feedback_text: this.feedback,
      };

      // Convert the data object to query string parameters
      const queryString = new URLSearchParams(data).toString();

      fetch(`${webhookURL}?${queryString}`, {
        method: 'GET',
        headers: {
          'Accept': 'application/json',
        },
      })
        .then(() => {
          this.showFeedbackForm = false;
          this.showMessageOk = true;
          this.feedback = '';
          confetti({
            particleCount: 100,
            spread: 70,
            origin: { y: 0.6 },
          });
        })
        .catch((error) => {
          console.error('Error submitting feedback:', error);
        });
    },
  },
};
</script>
