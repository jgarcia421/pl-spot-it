<script>
  export default {
    name: 'SpotIt',
    data() {
      return {
        mode: 'intro',
        playerInput: '',
        players: [],
        answerStatus: '',
        symbols: [
          'arsenal.png',
          'aston-villa.png',
          'bournemouth.png',
          'brentford.png',
          'brighton.png',
          'burnley.png',
          'chelsea.png',
          'crystal-palace.png',
          'everton.png',
          'fulham.png',
          'liverpool.png',
          'luton-town.png',
          'manchester-city.png',
          'manchester-united.png',
          'newcastle.png',
          'nottingham-forest.png',
          'sheffield.png',
          'tottenham.png',
          'west-ham.png',
          'wolverhampton.png'
        ],
        selectedSymbols: []
      }
    },
    watch: {
      query: function(q) {
        this.searchAttendees();
      }
    },
    methods: {
      start() {
      },
      addPlayer() {
        this.players.push({
          name: this.playerInput,
          score: 0
        });
        this.playerInput = '';
        this.mode = 'playerReview';
      },
      startRound() {
        this.answerStatus = '';
        this.mode = 'game';
        this.selectedSymbols = this.generateObjectsArray();
        var matchingIndex = this.pickRandomObjectIndex(this.selectedSymbols);
        this.selectedSymbols[matchingIndex].is_match = true;
        this.selectedSymbols.push(JSON.parse(JSON.stringify(this.selectedSymbols[matchingIndex])));
        this.selectedSymbols[matchingIndex].size = this.getRandomSize();
        this.selectedSymbols = this.shuffleArray(this.selectedSymbols);
      },
      getRandomIndex(excludedIndices) {
          let randomIndex;
          do {
              randomIndex = Math.floor(Math.random() * 20);
          } while (excludedIndices.includes(randomIndex));
          return randomIndex;
      },
      getRandomSize() {
          const sizes = ["small", "medium", "large"];
          return sizes[Math.floor(Math.random() * sizes.length)];
      },
      generateObjectsArray() {
          const objectsArray = [];
          const usedIndices = [];
          for (let i = 0; i < 14; i++) {
              const uniqueIndex = this.getRandomIndex(usedIndices);
              usedIndices.push(uniqueIndex);
              objectsArray.push({
                  index: uniqueIndex,
                  size: this.getRandomSize()
              });
          }
          return objectsArray;
      },
      pickRandomObjectIndex(objectsArray) {
          const randomIndex = Math.floor(Math.random() * objectsArray.length);
          return randomIndex;
      },
      shuffleArray(array) {
          for (let i = array.length - 1; i > 0; i--) {
              const j = Math.floor(Math.random() * (i + 1));
              [array[i], array[j]] = [array[j], array[i]]; // Swap elements
          }
          return array;
      },
      checkAnswer(element) {
        if(element.is_match) {
          this.playYes();
          this.answerStatus = 'correct';
          this.mode = 'selectWinner';
          
          /*
          setTimeout(() => {
              this.startRound();
          }, 1000);
          */
        } else {
          this.playNo();
          this.answerStatus = 'incorrect';
          setTimeout(() => {
              this.answerStatus = '';
          }, 1000);
          
        }
      },
      playYes() {
        this.playAudio('yes');
      },
      playNo() {
        this.playAudio('no');
      },
      playAudio(yesOrNo) {
          // Create a new audio element
          const audio = new Audio('/audio/' + yesOrNo + '.mp3');
      
          // Play the audio
          audio.play();
      
          // Add an event listener to remove the audio element after it ends
          audio.addEventListener('ended', () => {
              audio.remove(); // Clean up the audio element after playing
          });
      },
      incrementScore(p) {
        p.score++;
        console.log(p);
        this.startRound();
      }
    },
    mounted() {
      this.start();
    },
  }
</script>

<template>
  
  <div id="intro" v-show="mode=='intro'">
    <div class="logo"><img src="/img/premier-league.jpg" /></div>
    <h2>Premier League Spot-It</h2>
    
    <button v-on:click="mode = 'players'">Start</button>
  </div>
  
  <div id="players" v-show="mode=='players'">
    <h2>Enter Player Name</h2>
    <div><input v-model="playerInput" /></div>
    <button v-on:click="addPlayer">Continue</button>
  </div>
  
  <div id="player-review" v-show="mode=='playerReview'">
    <div v-for="p in players">
      <h2>{{ p.name }}</h2>
    </div>
    <button v-on:click="mode='players'">Add Another</button>
    <button v-on:click="startRound()">Start Game</button>
  </div>
  
  <div id="game" v-show="mode=='game'">
    <div class="tiles">
      <div class="tile" v-for="s in selectedSymbols" v-bind:class="s.size" v-on:click="checkAnswer(s)">
        <img v-bind:src="'/img/' + symbols[s.index]">
        {{ s.match }}
      </div>
    </div>
    
    <div class="answer answer-correct" v-show="answerStatus == 'correct'" v-on:click="startRound()">
      <h2>Correct!</h2>
    </div>
    
    <div class="answer answer-incorrect" v-show="answerStatus == 'incorrect'" v-on:click="answerStatus = ''">
      <h2>Wrong!</h2>
    </div>
    
  </div>
  
  <div class="select-winner" v-show="mode=='selectWinner'">
    <div class="content">
      <h2>Correct!</h2>
      <h3>Who guessed it?</h3>
      <button v-for="p in players" v-on:click="incrementScore(p)">{{ p.name }}</button>
    </div>
  </div>
  
  <div class="scores">
    <span class="score" v-for="p in players">
      <strong>{{ p.name }}:</strong> {{ p.score }}
    </span>
  </div>
  
</template>

<style scoped>
  
  button {
    cursor:pointer;
    background-color:#3d195b;
    color:#fff;
    -webkit-appearance: none;
    font-size:1.5em;
    font-weight:bold;
    border:none;
    outline:none;
    padding:0.75em 1.5em;
    border-radius:6px;
    margin:0.5em 0.25em;
    display:inline-block;
  }
  
  .scores {
    position:fixed;
    bottom:0;
    width:100%;
    background-color:#3d195b;
    color:#fff;
    text-align:center;
    padding:0.5em;
    
    .score {
      font-size:2em;
      margin:0 2em;
    }
  }
  
  #intro {
    text-align:center;
    background-color:#2e093d;
    height:100vh;
    
    .logo {
      max-width:400px;
      margin:0 auto;
      
      img {
        width:100%;
      }
    }
    
    h2 {
      font-size:3em;
      color:#fff;
    }
    
    button {
      background-color:#fff;
      color:#2e093d;
    }
    
  }
  
  #players {
    text-align:center;
    
    input {
      font-size:1.5em;
      width:20em;
      text-align:center;
      border-radius:6px;
      border:solid 2px #ccc; 
      padding:0.75em;
    }
  }
  
  
  
  .select-winner {
    position:absolute;
    top:0;
    left:0;
    width:100%;
    height:100vh;
    text-align:center;
    background-color:#00a747;
    position:relative;
    
    .content {
      position:absolute;
      top:50%;
      width:100%;
      transform:translateY(-50%);
    }
    
    h2 {
      font-weight:bold;
      color:#fff;
    }
    
    h3 {
      color:#fff;
    }
    
    button {
      background-color:#fff;
      color:#3d195b;
    }
  }
  
  #game {
    
    .answer {
      position:absolute;
      top:0;
      left:0;
      width:100%;
      height:100%;
      text-align:center;
      
      h2 {
        position:absolute;
        top:50%;
        transform:translateY(-50%);
        width:100%;
        font-size:3em;
        font-weight:bold;
      }
      
      &.answer-correct {
        background-color:#00a747;
        color:#fff;
      }
      
      &.answer-incorrect {
        background-color:#a70000;
        color:#fff;
      }
    }
    
    .tiles {
      display:grid;
      grid-template-columns:1fr 1fr 1fr 1fr 1fr;
      grid-template-rows:1fr 1fr 1fr;
      text-align:center;
      height:100vh;
      
      .tile {
        overflow:hidden;
        padding:10%;
        
        img {
          object-fit:contain;
          width:80%;
          height:80%;
        }
        
        &.large {
          padding:10%;
          
          img {
            width:80%;
            height:80%;
          }
          
        }
        
        &.medium {
          padding:20%;
          
          img {
            width:60%;
            height:60%;
          }
          
        }
        
        &.small {
          padding:25%;
          
          img {
            width:50%;
            height:50%;
          }
          
        }
      }
    }
  }
</style>
