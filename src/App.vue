<template>
  <div id="app">
      <section class="loading-game" v-show="loadingGame">
          <h1>LOADING GAME . . .</h1>
      </section>
      <section class="row" v-show="!loadingGame">
          <div class="small-6 columns">
              <h1 class="text-center"><span v-if="turn" style="color:blue;">&bull;</span>You</h1>
              <div class="healthbar" :style="youLose">
                  <div class="healthbar text-center" :style="youStyles">
                      <p>{{ youHealth > 0 ? youHealth : '' }}</p>
                  </div>
              </div>
              <div class="power-wrap">
                  <p>Power</p>
                  <div class="power">
                      <div class="power" :style="youPowerStyles"></div>
                  </div>
              </div>
          </div>
          <div class="small-6 columns">
              <h1 class="text-center"><span v-if="!turn" style="color:blue;">&bull;</span>{{ !swapiChar ? "Loading Opponent..." : swapiChar }}</h1>
              <div class="healthbar" :style="monsterLose">
                  <div class="healthbar text-center" :style="monsterStyles">
                          <p>{{ monsterHealth > 0 ? monsterHealth : '' }}</p>
                  </div>
              </div>
              <div class="power-wrap">
                  <p>Power</p>
                  <div class="power">
                      <div class="power" :style="monsterPowerStyles"></div>
                  </div>
              </div>
          </div>
      </section>
      <section class="row controls" v-show="!startGame" v-if="!loadingGame">
          <div class="small-12 columns">
              <button id="start-game" @click="startNewGame">START NEW GAME</button>
          </div>
      </section>
      <section class="row controls" v-show="startGame">
          <div class="small-12 columns">
              <button id="attack" @click="attack">ATTACK</button>
              <button id="special-attack" @click="specialAttack">SPECIAL ATTACK</button>
              <button id="block" @click="block">BLOCK</button>
              <button id="heal" @click="heal">HEAL</button>
              <button id="obliterate" :disabled="youPowerStatus" @click="obliterate" v-if="turn">OBLITERATE</button>
              <button id="obliterate" :disabled="monsterPowerStatus" @click="obliterate" v-else>OBLITERATE</button>
              <button id="give-up" @click="giveUp">GIVE UP</button>
          </div>
      </section>
      <section class="row log" v-show="startGame">
          <div class="small-12 columns">
              <ul>
                  <li v-for="(hit, i) in hits" :key="`hit${i}`" :class="i % 2 === 0 ? 'player-log' : 'monster-log'">{{ hit }}</li>
              </ul>
          </div>
      </section>
      <section class="modal-wrap" v-show="gameOver">
          <div class="modal">
              <h1>{{ monsterHealth &lt;= 0 ? "YOU WIN!" : "YOU LOSE." }}</h1>
              <button id="new-game" @click="startNewGame">New Game</button>
          </div>
      </section>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'app',
  data: function() {
      return {
        loadingGame: true,
        startGame: false,
        gameOver: false,
        turn: true,
        youHealth: 100,
        monsterHealth: 100,
        youPowerStatus: true,
        monsterPowerStatus: true,
        youPower: 0,
        monsterPower: 0,
        youStyles: {
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
            background: "green",
            margin: 0,
            color: "white",
            width: "100%",
        },
        monsterStyles: {
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
            background: "green",
            margin: 0,
            color: "white",
            width: "100%",
        },
        youPowerStyles: {
            background: "#197BBD",
            margin: 0,
            width: "0%"
        },
        monsterPowerStyles: {
            background: "#197BBD",
            margin: 0,
            width: "0%"
        },
        youLose: {
            background: "#eee"
        },
        monsterLose: {
            background: "#eee"
        },
        hits: [],
        swapiChar: ''
      }
    },
    created() {
        console.log('created');
        axios.get("https://swapi.co/api/people")
        .then(res => {
            this.swapiChar = res.data.results[Math.floor(Math.random()*10)].name;
            console.log(this.swapiChar);
            this.loadingGame = false;
        })
        .catch( err => {throw err});
    },
    methods: {
        startNewGame(){
            this.startGame = true;
            this.turn = true;
            this.youStyles.width = "100%";
            this.youHealth = 100;
            this.monsterStyles.width = "100%";
            this.monsterHealth = 100;
            this.youPower = 0
            this.youPowerStyles.width = "0%";
            this.monsterPower = 0
            this.monsterPowerStyles.width = "0%";
            this.gameOver = false;
            this.youPowerStatus = true;
            this.monsterPowerStatus = true;
            this.hits = [];
        },
        damage(min, max) {
            return Math.max(Math.floor(Math.random() * max) + 1, min);
        },
        attack() {
            if(this.turn){
                let damage = this.damage(1, 6);
                let attackPower = this.youPower >= 100 ? 100 : this.youPower += 10;
                let monsterWidth = this.monsterHealth -= damage;
                this.monsterStyles.width = `${monsterWidth}%`;
                this.youPowerStyles.width = `${attackPower}%`;
                this.youPower >= 100 ? this.youPowerStatus = false : this.youPowerStatus = true;
                this.turn = false;
                this.hits.push(`You hit ${this.swapiChar} for ${damage}. You gain 10 power.`);
            }
            else {
                let damage = this.damage(1, 6);
                let attackPower = this.monsterPower >= 100 ? 100 : this.monsterPower += 10;
                let youWidth = this.youHealth -= damage;
                this.youStyles.width = `${youWidth}%`;
                this.monsterPowerStyles.width = `${attackPower}%`;
                this.monsterPower >= 100 ? this.monsterPowerStatus = false : this.monsterPowerStatus = true;
                this.turn = true;
                this.hits.push(`${this.swapiChar} hit You for ${damage}. ${this.swapiChar} gains 10 power.`);
            }
        },
        specialAttack() {
            if (this.turn) {
                let damage = this.damage(7, 12);
                let attackPower = this.youPower >= 100 ? 100 : this.youPower += 20;
                let monsterWidth = this.monsterHealth -= damage;
                this.monsterStyles.width = `${monsterWidth}%`;
                this.youPowerStyles.width = `${attackPower}%`;
                this.youPower >= 100 ? this.youPowerStatus = false : this.youPowerStatus = true;
                this.turn = false;
                this.hits.push(`You hit ${this.swapiChar} for ${damage}. You gain 20 power.`);
            }
            else {
                let damage = this.damage(7, 12);
                let attackPower = this.monsterPower >= 100 ? 100 : this.monsterPower += 20;
                let youWidth = this.youHealth -= damage;
                this.youStyles.width = `${youWidth}%`;
                this.monsterPowerStyles.width = `${attackPower}%`;
                this.monsterPower >= 100 ? this.monsterPowerStatus = false : this.monsterPowerStatus = true;
                this.turn = true;
                this.hits.push(`${this.swapiChar} hit You for ${damage}. ${this.swapiChar} gains 20 power.`);
            }
        },
        block() {
            if(this.turn){
                let youWidth = this.youHealth += 5;
                this.youStyles.width = `${youWidth}%`;
                this.youPower >= 100 ? this.youPowerStatus = false : this.youPowerStatus = true;
                this.turn = false;
                this.hits.push(`You block ${this.swapiChar} and gain 5 health.`);
            }
            else {
                let monsterWidth = this.monsterHealth += 5;
                this.monsterStyles.width = `${monsterWidth}%`;
                this.monsterPower >= 100 ? this.monsterPowerStatus = false : this.monsterPowerStatus = true;
                this.turn = true;
                this.hits.push(`${this.swapiChar} blocks You and gains 5 health.`);
            }
        },
        heal() {
            if(this.turn){
                let attackPower = this.youPower -= 5;
                let youWidth = this.youHealth += 5;
                this.youStyles.width = `${youWidth}%`;
                this.youPowerStyles.width = `${attackPower}%`;
                this.youPower >= 100 ? this.youPowerStatus = false : this.youPowerStatus = true;
                this.turn = false;
                this.hits.push(`You heal and gain 5 health, but lose 5 power.`);
            }
            else {
                let attackPower = this.monsterPower -= 5;
                let monsterWidth = this.monsterHealth += 5;
                this.monsterStyles.width = `${monsterWidth}%`;
                this.monsterPowerStyles.width = `${attackPower}%`;
                this.monsterPower >= 100 ? this.monsterPowerStatus = false : this.monsterPowerStatus = true;
                this.turn = true;
                this.hits.push(`${this.swapiChar} heals and gains 5 health, but loses 5 power.`);
            }
        },
        obliterate() {
            if(this.turn) {
                this.monsterStyles.width = "0%";
                this.monsterHealth = 0;
                this.monsterPower = 0;
                this.monsterPowerStyles.width = "0%";
            }
            else {
                this.youStyles.width = "0%";
                this.youHealth = 0;
                this.youPower = 0;
                this.youPowerStyles.width = "0%";
            }
        },
        giveUp() {
            if(this.turn) {
                this.youStyles.width = "0%";
                this.youHealth = 0;
                this.youPower = 0
                this.youPowerStyles.width = "0%";
            }
            else {
                this.monsterStyles.width = "0%";
                this.monsterHealth = 0;
                this.monsterPower = 0
                this.monsterPowerStyles.width = "0%";
            }
        }
    },
    watch: {
        youHealth() {
            if(this.youHealth <= 0){
                this.youLose.background = "red";
                this.gameOver = true;
            }
            else {
                this.youLose.background = "#eee";
            }
        },
        monsterHealth() {
            if(this.monsterHealth <= 0) {
                this.monsterLose.background = "red";
                this.gameOver = true;
            }
            else {
                this.monsterLose.background = "#eee";
            }
        }
    }
}
</script>

<style>
  @import './foundation.min.css';
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
  .text-center {
      text-align: center;
  }

  .healthbar {
      width: 80%;
      height: 40px;
      background: #eee;
      margin: auto;
      transition: width 500ms;
  }

  .controls, .log {
      margin-top: 20px;
      text-align: center;
      padding: 10px;
      border: 1px solid #ccc;
      box-shadow: 0px 3px 6px #ccc;
  }

  .power-wrap {
      width: 80%;
      display: flex;
      justify-content: center;
      align-items: center;
      margin: auto;
  }
  .power-wrap p {
      margin-right: 10px;
  }

  .power {
      width: 70%;
      height: 10px;
      background: #eee;
      transition: width 500ms;
  }

  .turn {
      margin-top: 20px;
      margin-bottom: 20px;
      font-weight: bold;
      font-size: 22px;
  }

  .log ul {
      list-style: none;
      font-weight: bold;
      text-transform: uppercase;
  }

  .player-log {
      color: blue;
      background: #e4e8ff;
  }

  .monster-log {
      color: red;
      background: #ffc0c1;
  }

  button {
      font-size: 20px;
      background: #eee;
      padding: 12px;
      box-shadow: 0 1px 1px black;
      margin: 10px;
  }

  #start-game {
      background: #197BBD;
      color: white;
  }

  #start-game:hover {
      background: #4293C9;
  }

  #attack {
      background: #ff7367;
  }

  #attack:hover {
      background: #ff3f43;
  }

  #special-attack {
      background: #ffaf4f;
  }

  #special-attack:hover {
      background: #ff9a2b;
  }
  #block {
      background: #6CABD5;
  }

  #block:hover {
      background: #4293C9;
  }

  #heal {
      background: #aaffb0;
  }

  #heal:hover {
      background: #76ff7e;
  }
  #obliterate {
      background: #0D0D0D;
      color: white;
  }
  #obliterate:hover {
      color: red;
  }
  #obliterate:disabled {
      background: #eee;
      color: #c7c7c7;
  }

  #give-up {
      background: #ffffff;
  }

  #give-up:hover {
      background: #c7c7c7;
  }

  #new-game {
      background: #197BBD;
      color: white;
  }

  #new-game:hover {
      background: #4293C9;
  }

  .modal-wrap {
      position: fixed;
      left: 0;
      top: 0;
      width: 100vw;
      height: 100vh;
      display: flex;
      justify-content: center;
      align-items: flex-start;
      background: #00000070;
  }
  .modal {
      background: white;
      width: 300px;
      height: 200px;
      border-radius: 5px;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      margin-top: 200px;
  }
  .loading-game {
      width: 100vw;
      display: flex;
      justify-content: center;
      align-items: flex-end;
      margin-top: 50px;
  }
  .loading-game span {
      margin-left: 10px;
  }
</style>
