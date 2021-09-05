<template>
  <div class="bg-gray-100 h-screen flex justify-center">

    <div v-if="status === 'login'">
      <p class="text-5xl p-8 text-center">(なんちゃって)ペンギン</p>
      <div class="text-gray-800">
        <div class="text-center">
          <div class="mb-4">
            <label class="block text-gray-700 text-xl font-bold mb-2">
              名前
            </label>
            <input class="shadow appearance-none border rounded w-1/2 py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" type="text" placeholder="名前"  v-model="name">
          </div>
          <button class="bg-gray-500 hover:bg-gray-700 text-white font-bold py-2 px-4 rounded" @click="join">
            参加
          </button>
        </div>
      </div>
    </div>

    <div class="text-center" v-if="status === 'ready'">
      <p class="text-5xl p-8">(なんちゃって)ペンギン</p>
      <div class="mb-4">
        <p class="text-xl font-semibold">プレイヤー</p>
        <div v-for="player in players" :key="player.name" class="m-2">
          <p>{{"- " + player.name}}</p>
        </div>
      </div>
      <button class="bg-gray-500 hover:bg-gray-700 text-white font-bold py-2 px-4 rounded" @click="start">
        すたーと！
      </button>
    </div>

    <div v-if="status === 'game'">
      <div class="w-1/4">
        <p class="text-4xl mt-5 text-center">テストペンギン</p>

        <div class="m-3 border border-gray-400 rounded-lg">
          <p class="text-2xl text-gray-700 m-2 text-center">{{"ぷれいや: " + player.name}}</p>
          <p class="text-lg text-gray-700 m-2 text-center">{{"あかんポイント: " + player.point}}</p>
        </div>

      </div>

      <div class="w-1/2">
        <div id="field" class="mt-10">
          <div v-for="(fieldCards, fieldCardsIndex) in showFieldCard()" :key="fieldCardsIndex" class="flex justify-center my-3">
            <div v-for="fieldCard in fieldCards" :key="fieldCard.id" class="mx-2" :id="fieldCard.id"
              draggable="true"
              @dragstart="dragStart(fieldCard, $event)"
              @dragend="dragEnd($event)"
              @dragover="dragOver($event)"
              @drop="drop($event)"
              >
              <CardComponent :color="fieldCard.color" :id="fieldCard.id"></CardComponent>
            </div>
          </div>
        </div>

        <div id="player" class="mt-10 mx-16 px-3 py-6 border border-gray-400 rounded-lg"
          @dragover="dragOver($event)"
          @drop="drop($event)"
          >
          <p class="text-lg text-gray-700 ml-6 mb-2">手札</p>
          <div class="flex justify-center">
            <div v-for="(playerCard, playerCardIndex) in playerCards" :key="playerCardIndex" class="mx-2"
              draggable="true"
              @dragstart="dragStart(playerCard, $event)"
              @dragend="dragEnd($event)"
              @dragover="dragOver($event)"
              @drop="drop($event)"
              >
              <CardComponent :color="playerCard.color" :id="playerCard.id"></CardComponent>
            </div>
          </div>
        </div>
      </div>

      <div class="w-1/4">
        <div v-for="player in players" :key="player.name" class="m-2 border border-gray-400 rounded-lg p-2">
          <p class="text-gray-700 text-xl my-2">{{player.name}}</p>
          <div class="flex justify-left">
            <div v-for="(otherCard, otherCardIndex) in player.hand" :key="otherCardIndex">
              <CardComponent v-if="player.visible" :color="otherCard" :id="otherCardIndex" class="ml-1"></CardComponent>
              <CardComponent v-else :color="'black'" :id="otherCardIndex" class="ml-1"></CardComponent>
            </div>
          </div>
          <p class="my-2 text-lg text-gray-700">{{'あかんポイント : ' + player.point}}</p>
        </div>
      </div>
    </div>

  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from '@vue/composition-api';
import CardComponent from '../components/CardComponent.vue';

interface Player {
  name: string;
  hand: string[];
  visible: boolean;
  point: number;
}

interface Card {
  color: Color;
  id: number;
  place: Place;
}

interface HTMLElementEvent<T extends HTMLElement> extends Event {
    target: T;
}

interface GameResponce {
  field: Color[];
  players: Player[];
}

type Color = 
  'red' |
  'green' |
  'blue' |
  'purple' |
  'yellow' |
  ''

type Place = 
  'field' |
  'player'

export default defineComponent({
  components: {
    CardComponent
  },
  setup() {
    const name = ref<string>();
    const player = ref<Player>();
    const status = ref<string>("login");
    const players = ref<Player[]>();
    const playerCards = ref<Card[]>([]);
    const field = ref<Card[]>([]);
    const move = ref<boolean>(false);
    const dragItem = ref<Card>();

    // let idNum = 0
    // for(let i = 0; i < 36; i++){
    //   field.value.push({
    //     color: '',
    //     id: idNum,
    //     place: 'field'
    //   })
    //   idNum++
    // }

    let idNum = 0
    for(let i = 1; i < 7; i++){
      playerCards.value.push({
        color: 'yellow',
        id: idNum,
        place: 'player'
      })
      idNum++
    }

    // players.value = [
    //   {
    //   name: "まっさん",
    //   hand: ['red', 'blue', 'green', 'purple', 'yellow'],
    //   visible: false,
    //   point: 2,
    //   },
    //   {
    //   name: "うっさん",
    //   hand: ['red', 'blue', 'green', 'purple', 'yellow'],
    //   visible: true,
    //   point: 7,
    //   },
    // ]

    // player.value = {
    //   name: "massann",
    //   hand: ['red', 'blue', 'green', 'purple', 'yellow'],
    //   visible: true,
    //   point: 4,
    // }
    

    const showFieldCard = () => {
      let retArray: Card[][] = [];
      let nowNumber = 0;
      for(let i = 1; i < 9; i++){
        let array: Card[] = [];
        for(let j = 0; j < i; j++){
          if(field.value[nowNumber]){
            array.push(field.value[nowNumber])
            nowNumber++
          }
        }
        retArray.push(array)
      }
      return retArray;
    }
    
    const dragStart = (item: Card, e: DragEvent) => {
      if(e.dataTransfer){
        e.dataTransfer.effectAllowed = 'move'
      }
      dragItem.value = item
    }

    const dragEnd = () => {
      // console.log(e)
      // console.log(e.dataTransfer)
      // console.log(e.currentTarget)
    }

    const drop = (e: HTMLElementEvent<HTMLInputElement>) => {
      e.preventDefault();

      if(!dragItem.value || !e.target.id){
        return
      }

      if(dragItem.value.place === 'field' && e.target.id === 'player'){
        playerCards.value.push(dragItem.value)
        playerCards.value = playerCards.value.map((v, i) => {return {color: v.color, id:i, place: 'player'}})
        field.value[dragItem.value.id].color = ''
      }

      if(dragItem.value.place === 'player' && e.target.id !== 'player'){
        if(field.value[parseInt(e.target.id)].color !== ''){
          return
        }

        field.value[parseInt(e.target.id)].color = dragItem.value.color;
        playerCards.value.splice(dragItem.value.id, 1)
        playerCards.value = playerCards.value.map((v, i) => {return {color: v.color, id:i, place: v.place}})
      }
    }

    const dragOver = (e: DragEvent) => {
      e.preventDefault();
    }


    // const ws = new WebSocket('wss://87422686d3d4.ngrok.io');
    const ws = new WebSocket('ws://localhost:12345');

    // setInterval(() => {
    //   const data = {
    //     status: 'interval'
    //   }
    //   ws.send(JSON.stringify(data));
    // }, 60000);

    // setInterval(() => {
    //   timer.value = timer.value - 1;
    //   if(timer.value <= 0){
    //     timer.value = 0
    //   }
    // }, 1000)

    const join = () => {
      const data = {
        name: name.value,
        status: 'login',
      }
      ws.send(JSON.stringify(data));
    }

    const start = () => {
      const data = {
        status: 'ready'
      }
      ws.send(JSON.stringify(data));
    }

    ws.onmessage = (event) => {
      const obj = JSON.parse(event.data)

      if(obj['next_status'] === "ready"){
        status.value = 'ready';
        players.value = obj.data;
      }

      if(obj['next_status'] === "game"){
        const res: GameResponce = obj;
        status.value = 'game';

        // player の判断 -> playerCards の処理
        players.value = res.players;

        if(res.field){
          field.value = res.field.map((v, i) => {return {color: v, id: i, place: 'field'}})
        }
      }

      console.log(obj)
    }

    // ws.onmessage = (event) => {
    //   const obj = JSON.parse(event.data)
    //   if(obj['next_status'] === "ready"){
    //     status.value = 'ready';
    //     players.value = obj.data
    //   }

    //   if(obj['next_status'] === "start"){
    //     status.value = 'start';
    //     player.value = obj.data;
    //     answer.value = obj.answer;
    //   }

    //   if(obj['next_status'] === "vote"){
    //     status.value = 'vote';
    //     players.value = obj.data;
    //     reserve.value = obj.reserve;
    //   }

    //   if(obj['next_status'] === "waitResult"){
    //     status.value = 'waitResult';
    //   }

    //   if(obj['next_status'] === "result"){
    //     status.value = 'result';
    //     players.value = obj.data;
    //     gameResult.value = obj.result;
    //   }
    // }

    return {
      CardComponent,
      name,
      status,
      player,
      players,
      field,
      move,
      playerCards,
      dragItem,
      join,
      start,
      showFieldCard,
      dragStart,
      dragEnd,
      drop,
      dragOver
    }
  },
});
</script>
