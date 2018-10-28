<template>
    <div id="container">
        <div class="content">
            <div class="players">
                <div class="player">
                    <div class="label">YOU</div>
                    <div class="bar">
                        <template v-if="game.playerHealth > 10">
                            <div class="barValue" :style="{width: `${playerBarValue}px`}">{{game.playerHealth}}</div>
                        </template>
                        <template v-else>
                            <div class="barValue" :style="{width: `${playerBarValue}px`}"></div>
                            <div class="barValueLow">{{game.playerHealth}}</div>
                        </template>
                    </div>
                </div>
                <div class="player">
                    <div class="label">MONSTER</div>
                    <div class="bar">
                        <template v-if="game.monsterHealth > 10">
                            <div class="barValue" :style="{width: `${monsterBarValue}px`}">{{game.monsterHealth}}</div>
                        </template>
                        <template v-else>
                            <div class="barValue" :style="{width: `${monsterBarValue}px`}"></div>
                            <div class="barValueLow">{{game.monsterHealth}}</div>
                        </template>
                    </div>
                </div>
            </div>
            <div class="actions" >
                <template v-if="!game.started">
                    <button class="button" @click="startGame()">START NEW GAME</button>
                </template>
                <template v-else>
                    <button :disabled="gameOver || actionsDisabled" class="button attack" :class="{ disabled: gameOver || actionsDisabled }" @click="playerAttack()">Attack</button>
                    <button :disabled="gameOver ||actionsDisabled" class="button special" :class="{ disabled: gameOver || actionsDisabled }" @click="special()">Special Attack</button>
                    <button :disabled="gameOver ||actionsDisabled" class="button heal" :class="{ disabled: gameOver || actionsDisabled }" @click="heal()">Heal</button>
                    <button class="button giveUp" @click="giveUp()">{{ gameOver ? "New Game":"Give up" }}</button>
                </template>
            </div>
            <div class="logs">
                <template v-for="(log,idx) in logs">
                    <div v-if="log.attacker === 'Monster'" class="log monster">{{idx + 1}}. {{log.message}}</div>
                    <div v-if="log.attacker === 'Player'" class="log player">{{idx + 1}}. {{log.message}}</div>
                </template>

            </div>
        </div>
    </div>


</template>

<script lang="ts">
    export default {
        data: ()=>({
            game: {
                started: false,
                monsterHealth: 100,
                playerHealth: 100,
            },
            actionsDisabled: false,
            logs: []
        }),
        computed: {
            monsterBarValue: function()  {
                console.log(this.game.monsterHealth);
                console.log(Math.floor(this.game.monsterHealth / 100 * 200));
                return Math.floor(this.game.monsterHealth / 100 * 200)
            },
            playerBarValue:function() {
                return Math.floor(this.game.playerHealth / 100 * 200)
            },
            gameOver: function() {
                return !this.game.playerHealth || !this.game.monsterHealth;
            }
        },
        methods: {
            monsterAttack: function() {
                this.actionsDisabled = true;
                console.log("monsterAttack");
                setTimeout(()=> {
                    if(!this.gameOver) {
                        const dmg = Math.floor(Math.random()*20)
                        this.game.playerHealth = Math.max(0, this.game.playerHealth - dmg);
                        this.logs.push({message:`Monster hits Player for ${dmg} Damage.`, attacker: 'Monster'});
                        this.actionsDisabled = false;
                    }
                }, 500);
            },
            playerAttack: function() {
                console.log("playerAttack");
                const dmg = Math.floor(Math.random()*10);
                this.game.monsterHealth = Math.max(0, this.game.monsterHealth - dmg);
                this.logs.push({message:`Player hits Monster for ${dmg} Damage.`, attacker: 'Player'});
                this.monsterAttack();
            },
            special: function()  {
                const dmg = Math.floor(Math.random()*10) + 10;
                this.game.monsterHealth = Math.max(0, this.game.monsterHealth - dmg);
                this.logs.push({message:`Player hits Monster for ${dmg} Damage.`, attacker: 'Player'});
                this.monsterAttack();
            },
            heal: function() {
                const heal = Math.floor(Math.random()*20);
                this.game.playerHealth = Math.min(100, this.game.playerHealth + heal);
                this.logs.push({message:`Player heals ${heal} Damage.`, attacker: 'Player'});
                this.monsterAttack();
            },
            giveUp: function() {
                this.game = {
                    started: false,
                    monsterHealth: 100,
                    playerHealth: 100,
                };
                this.logs = [];
            },
            startGame:function() {
                console.log("startGame")
                this.game.started = true;
                this.monsterAttack();
            }
        },
        watch: {
            'game.playerHealth':
                function(h) {
                    if (h === 0) {
                        this.logs.push({
                            attacker: 'Player',
                            message: "Player died."
                        });
                    }
                },
                'game.monsterHealth': function(h) {
                    if(h === 0) {
                        this.logs.push({
                            attacker: 'Monster',
                            message: "Monster died."
                        });
                }
            }
        }
    }
</script>

<style scoped>
    #container {
        display: flex;
        height: 100vh;
        flex-direction: column;
        align-items: center;
        font-weight: bold;
    }

    .content {
        width: 700px;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .logs {
        display: flex;
        justify-content: center;
    }

    .players {
        display: flex;
        flex-direction: row;
        justify-content: center;
        font-size: 20px;
        min-height: 120px;
    }

    .player{
        padding: 20px;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .bar {
        width: 200px;
        height: 30px;
        background-color: lightgray;
        position: relative;
    }

    .barValue {
        background-color: green;
        color: white;
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .barValueLow {
        position: absolute;
        width: 100%;
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        color: black;
        top:0;
    }

    .label {
        padding: 10px;
    }

    .actions {
        justify-content: center;
        display: flex;
        width: 100%;
        box-shadow: 2px 5px 10px 2px darkgray;
        border: 1px solid darkgray;
        height: 50px;
        align-items: center;
        min-height: 50px;
    }

    .button {
        padding-bottom: 5px;
        padding-top: 2px;
        height: 30px;
        margin-right: 10px;
        margin-left: 10px;
    }

    .logs {
        width: 100%;
        display: flex;
        flex-direction: column;
        align-items: stretch;
        padding-bottom: 20px;
        padding-top: 20px;
        margin-top: 30px;
        box-shadow: 2px 5px 10px 2px darkgray;
        border: 1px solid darkgray;
        max-height: 400px;
        overflow-y: scroll;
    }

    .log {
        display: flex;
        justify-content: center;
        margin: 5px 50px;
        padding: 5px;
        min-height: 20px;
    }

    .log.player {
        background-color: lightskyblue;
    }

    .monster {
        background-color: pink;
    }

    button.attack {
        background-color: red;
    }

    button.heal {
        background-color: lightgreen;
    }

    button.special {
        background-color: orange;
    }

    button.disabled {
        background-color: gray;
    }
</style>