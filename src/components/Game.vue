<template>
  <div class="wrapper">

    <div>{{ levels[currentLevel].name }} level. Round: {{ currentRound }}</div>

    <h1>{{ timer / 1000 }} seconds</h1>

    <div
        class="button-list"
        @click="onButtonClick"
    >
      <button
          v-for="buttonIndex in levels[currentLevel].buttonCount"
          :key="buttonIndex"
          :data-index="buttonIndex"
          class="button"
          ref="buttons"
      >
        {{ buttonIndex }}
      </button>
    </div>

    <button
        :disabled="gameIsRunning"
        @click="startGame"
    >
      Start game
    </button>

    <audio class="audio" src="../assets/audio1.mp3"></audio>
    <audio class="audio" src="../assets/audio2.mp3"></audio>
    <audio class="audio" src="../assets/audio3.mp3"></audio>
  </div>
</template>

<script>
export default {
  name: 'Game',

  data() {
    return {
      currentLevel: 1,
      currentRound: 1,

      timer: 0,
      timerInterval: null,

      levels: {
        1: {
          name: 'Easy',
          timeout: 1500,
          buttonCount: 4
        },
        2: {
          name: 'Middle',
          timeout: 1000,
          buttonCount: 6
        },
        3: {
          name: 'Hard',
          timeout: 400,
          buttonCount: 8
        },
      },

      sequence: new Set(),

      gameIsRunning: false,
      buttonsAreDisabled: true,
      currentTimeout: null
    }
  },

  methods: {

    async startGame() {
      this.gameIsRunning = true
      this.setTimer()
      this.generateSequence()
      await this.highlightButtons()
      this.buttonsAreDisabled = false
      this.restartTimeout()
      this.startTimer()
    },

    setTimer() {
      this.timer = this.levels[this.currentLevel].timeout
    },

    startTimer() {
      const startDate = Date.now()
      const endDate = startDate + this.levels[this.currentLevel].timeout
      this.timerInterval = setInterval(() => {
        this.timer = endDate - Date.now()
      }, 10)
    },

    stopTimer() {
      this.timer = 0
      clearInterval(this.timerInterval)
    },

    generateSequence() {
      const maxRandomNumber = this.levels[this.currentLevel].buttonCount
      const sequenceLength = this.randomIntNumber(1, maxRandomNumber)
      const randomNumber = this.randomIntNumber(1, maxRandomNumber + 1)

      this.sequence.add(randomNumber)

      if(this.sequence.size >= sequenceLength) {
        return
      }

      this.generateSequence()
    },

    randomIntNumber(min, max) {
      return Math.floor(Math.random() * (max - min) + min)
    },

    highlightButtons() {
      return new Promise((resolve) => {
        const HIGHLIGHTING_TIMER = 1000
        const buttonIndexes = Array.from(this.sequence)
        const buttonClass = 'button--highlighted'

        let iterationCount = 0

        let interval = setInterval(() => {
          iterationCount++

          if(iterationCount === this.sequence.size) {
            clearInterval(interval)
          }

          const buttonIndex = buttonIndexes[iterationCount - 1]
          const button = this.$refs.buttons[buttonIndex - 1]
          button.classList.add(buttonClass)

          this.playAudio()

          const iterationCountFrozen = JSON.parse(JSON.stringify(iterationCount))
          setTimeout(() => {
            button.classList.remove(buttonClass)

            if(iterationCountFrozen === this.sequence.size) {
              console.log('resolved')
              resolve()
            }
          }, HIGHLIGHTING_TIMER)

        }, HIGHLIGHTING_TIMER)

      })
    },

    restartTimeout() {
      clearTimeout(this.currentTimeout)

      this.currentTimeout = setTimeout(() => {
        this.playerLost()
      }, this.levels[this.currentLevel].timeout)
    },

    resetSequence() {
      this.sequence = new Set()
    },

    resetProgress() {
      this.currentLevel = 1
      this.currentRound = 1
    },

    checkProgress() {
      if(this.currentRound === 0) {
        this.resetProgress()
        return
      }

      if(this.currentLevel === 3 && this.currentRound > 3) {
        this.resetProgress()
        alert('Game Over! My Congratulations!')
        return
      }

      if(this.currentRound > 3) {
        this.currentLevel += 1
        this.currentRound = 1
        return
      }

      console.log('Game started again')
      this.startGame()
    },

    finishTheGame() {
      this.stopTimer()
      this.resetSequence()
      clearTimeout(this.currentTimeout)
      this.checkProgress()
      this.gameIsRunning = false
      this.buttonsAreDisabled = true
    },

    onButtonClick(event) {
      if(this.buttonsAreDisabled) {
        return
      }

      this.playAudio()
      const clickedButtonIndex = +event.target.dataset.index
      this.checkSequence(clickedButtonIndex)
    },

    playAudio() {
      const audios = document.getElementsByClassName('audio')
      const randomAudioIndex = this.randomIntNumber(0, 2)
      const playPromise = audios[randomAudioIndex].play()

      if (playPromise !== undefined) {
        playPromise.then(() => {

        })
        .catch(error => {
          console.log(`playSound error: ${error}`)
        })
      }
    },

    checkSequence(buttonIndex) {
      const firstSequence = this.sequence.values().next().value

      if(firstSequence !== buttonIndex) {
        this.playerLost()
        return
      }

      this.sequence.delete(buttonIndex)

      if(!this.sequence.size) {
        this.playerWon()
        return
      }

      this.restartTimeout()
      this.stopTimer()
      this.startTimer()
    },

    playerWon() {
      alert('You won!')
      this.currentRound += 1
      this.finishTheGame()
    },

    playerLost() {
      alert('You lose! :(')
      this.currentRound = 0
      this.finishTheGame()
    }

  }
}
</script>

<style lang="scss">
.wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.button-list {
  display: flex;
  flex-wrap: wrap;
  max-width: 200px;
}

.button {
  width: 100px;
  height: 100px;
  &--highlighted {
    background-color: red;
  }
}
</style>
