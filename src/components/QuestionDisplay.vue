<template>
  <div class="video-container">
    <video autoplay loop muted playsinline class="background-video">
      <source src="@/assets/snowflakes.mp4" type="video/mp4" />
      Your browser does not support the video tag.
    </video>
    <div :style="gradientStyle" class="app-container">
      <div class="navigation-bar">
        <button v-for="(question, index) in questions" :key="index"
          :class="['nav-button', { active: index === currentQuestionIndex }]" @click="navigateToQuestion(index)">
          {{ index + 1 }}
        </button>
        <button class="nav-button" @click="toggleHowToPage">
          How to Play
        </button>
      </div>

      <div v-if="showHowToPage" class="how-to-screen">
        <h1 class="how-to-title">How to Play</h1>
        <ol class="how-to-list">
          <li>Each round you will be given a multiple-choice question.</li>
          <li>You have to bet all your 6 shots on each question but must leave one answer clear.</li>
          <li>The shots placed on the correct answer can be given out to others in the room.</li>
          <li>Any incorrectly allocated shots must be taken by your team.</li>
        </ol>
        <button class="close-button" @click="toggleHowToPage">Close</button>
      </div>


      <div v-else-if="showQuestionNumber" class="screen">
        <h1>Question {{ currentQuestionIndex + 1 }}</h1>
        <button @click="showNextScreen">Start</button>
      </div>

      <div v-else class="screen">
        <h1>{{ currentQuestion.question }}</h1>
        <div class="answers-container">
          <div v-for="(answer, index) in currentQuestion.answers" :key="index"
            :class="['answer-box', { correct: answer.label === currentQuestion.correctAnswer && answerRevealed }]">
            <h2>{{ answer.label }}</h2>
            <p>{{ answer.text }}</p>
          </div>
        </div>

        <div class="progress-bar-container">
          <div class="progress-bar" :style="{ width: timeLeftPercentage + '%', backgroundColor: barColor }"></div>
          <div class="time-remaining" :style="{ color: barColor }">
            {{ timeLeft }}s
          </div>
        </div>

        <div v-if="timeUp && !answerRevealed">
          <h2>Time's Up!</h2>
          <button @click="revealAnswer">Reveal Answer</button>
        </div>

        <div v-else-if="!answerLocked && !timeUp">
          <button @click="lockInAnswer">Lock in Answer</button>
        </div>

        <div v-else-if="answerRevealed">
          <h2>Correct Answer: {{ currentQuestion.correctAnswer }}</h2>
          <button @click="nextQuestion">Next Question</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      questions: [],
      currentQuestion: {},
      currentQuestionIndex: 0,
      showQuestionNumber: true,
      timeLeft: 45,
      timeUp: false,
      answerRevealed: false,
      answerLocked: false,
      timer: null,
      showHowToPage: false,
    };
  },
  computed: {
    timeLeftPercentage() {
      return (this.timeLeft / 45) * 100;
    },
    barColor() {
      if (this.timeLeft > 20) return "#32cd32"; // Green
      if (this.timeLeft > 10) return "#ffcc00"; // Yellow
      return "#ff6f61"; // Red
    },
    gradientStyle() {
      return {
        background: `linear-gradient(to bottom, #1c1c1c, #292929)`,
      };
    },
  },
  methods: {
    async loadQuestions() {
      try {
        const response = await fetch("questions.json");
        const data = await response.json();
        this.questions = data;
        this.currentQuestion = this.questions[0];
      } catch (error) {
        console.error("Failed to load questions:", error);
      }
    },
    startTimer() {
      this.timeLeft = 45;
      this.timeUp = false;
      this.answerRevealed = false;
      clearInterval(this.timer);
      this.answerLocked = false;
      this.timer = setInterval(() => {
        if (this.timeLeft > 0 && !this.answerLocked) {
          this.timeLeft--;
        } else {
          clearInterval(this.timer);
          if (!this.answerLocked) {
            this.timeUp = true; // Show time's up
          }
        }
      }, 1000);
    },
    revealAnswer() {
      this.answerRevealed = true;
    },
    nextQuestion() {
      if (this.currentQuestionIndex < this.questions.length - 1) {
        this.currentQuestionIndex++;
        this.currentQuestion = this.questions[this.currentQuestionIndex];
        this.showQuestionNumber = true;
      } else {
        alert("Game Over!");
        this.resetGame();
      }
    },
    resetGame() {
      this.currentQuestionIndex = 0;
      this.currentQuestion = this.questions[0];
      this.showQuestionNumber = true;
    },
    showNextScreen() {
      this.showQuestionNumber = false;
      this.startTimer();
    },
    navigateToQuestion(index) {
      this.currentQuestionIndex = index;
      this.currentQuestion = this.questions[index];
      this.showQuestionNumber = true;
      clearInterval(this.timer);
      this.timeLeft = 45;
      this.timeUp = false;
      this.answerRevealed = false;
    },
    toggleHowToPage() {
      this.showHowToPage = !this.showHowToPage;
    },
    lockInAnswer() {
      this.answerLocked = true; // Lock in the answer
      this.timeUp = true; // Show "Time's Up" immediately
      clearInterval(this.timer); // Stop the timer
    },
  },
  mounted() {
    this.loadQuestions();
  },
};
</script>

<style scoped>
/* How to Play Screen */
.how-to-screen {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 30px;
  background: rgba(0, 0, 0, 0.8);
  color: #fff;
  text-align: left;
  border-radius: 15px;
  max-width: 600px;
  margin: auto;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.6);
}

.how-to-title {
  font-size: 2.5rem;
  margin-bottom: 20px;
  text-align: center;
  text-shadow: 0 4px 8px rgba(0, 0, 0, 0.7);
}

.how-to-list {
  list-style-type: decimal;
  margin: 0;
  padding: 0 20px;
  font-size: 1.2rem;
  line-height: 1.8;
}

.how-to-list li {
  margin-bottom: 10px;
}

.close-button {
  margin-top: 20px;
  padding: 10px 20px;
  font-size: 1.2rem;
  background: rgba(85, 85, 85, 0.9);
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: 0.3s ease-in-out;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.6);
}

.close-button:hover {
  background: rgba(102, 102, 102, 1);
}

.close-button:active {
  background: rgba(50, 50, 50, 0.9);
}


/* Video Background */
.video-container {
  position: relative;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

.background-video {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: -1;
}

/* App Container */
.app-container {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 20px;
  text-align: center;
  color: #fff;
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
  z-index: 1;
  background: rgba(0, 0, 0, 0.6);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.8);
  border-radius: 15px;
}

/* Navigation Bar */
.navigation-bar {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}

.nav-button {
  padding: 10px 15px;
  border: none;
  border-radius: 5px;
  background: rgba(68, 68, 68, 0.8);
  color: #fff;
  cursor: pointer;
  font-size: 1rem;
  transition: 0.3s ease-in-out;
}

.nav-button:hover {
  background: #555;
}

.nav-button.active {
  background: #32cd32;
  font-weight: bold;
  box-shadow: 0 0 10px #32cd32;
}

/* Screen Titles */
.screen h1 {
  font-size: 3rem;
  margin-bottom: 20px;
  text-shadow: 0 4px 8px rgba(0, 0, 0, 0.7);
}

/* Answers Container */
.answers-container {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
  margin: 20px 0;
}

.answer-box {
  background: rgba(68, 68, 68, 0.8);
  border: 2px solid #fff;
  border-radius: 10px;
  padding: 20px;
  text-align: center;
  font-size: 1.5rem;
  transition: 0.3s ease-in-out;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.6);
}

.answer-box:hover {
  transform: scale(1.05);
  background: rgba(255, 255, 255, 0.2);
}

.answer-box.correct {
  background: rgba(50, 205, 50, 0.9);
  transform: scale(1.1);
  box-shadow: 0 4px 15px rgba(50, 205, 50, 0.9);
}

/* Progress Bar */
.progress-bar-container {
  margin-top: 20px;
  background: rgba(51, 51, 51, 0.8);
  height: 20px;
  border-radius: 10px;
  overflow: hidden;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 5px;
}

.progress-bar {
  height: 100%;
  transition: width 0.5s ease, background-color 0.5s ease;
  border-radius: 10px;
}

.time-remaining {
  font-size: 1.2rem;
  margin-left: 10px;
  font-weight: bold;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.6);
}

/* Buttons */
button {
  margin-top: 20px;
  padding: 10px 20px;
  font-size: 1.2rem;
  background: rgba(85, 85, 85, 0.8);
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: 0.3s ease-in-out;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.6);
}

button:hover {
  background: rgba(102, 102, 102, 1);
}

button:active {
  background: rgba(50, 50, 50, 0.9);
}
</style>