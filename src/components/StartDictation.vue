<script setup lang="ts">
import { ref } from 'vue'
let content = ref('')
let lastContent = ref('')
let shuffledArr = ref([])
let timer = null
const speekIndex = ref(0)
const showMask = ref(false)

const handleStart = ()=>{
  const originArr = splitContent()
  if(originArr.length){
    shuffledArr.value = shuffleArray([...originArr])
    saveToLocal()
    speakContent()
  }
}
const handleEnd = ()=>{
  console.log('end')
  clearTimeout(timer)
  shuffledArr.value = []
}
const handleAgain = () =>{
  content.value = lastContent.value
}
// const handlePause = () => {
//    clearTimeout(timer)
// }
// 处理输入的文本 返回词语数组
const splitContent = () => {
  const resultContent = content.value.replace(/[\t\n]/g,' ').trim()
  return resultContent? resultContent.split(/\s+/):[]
}
// 把数组随机打乱
function shuffleArray(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * ( i + 1 ));

    const temp  = array[i]
    array[i] = array[j]
    array[j] = temp
    // [array[i], array[j]] = [array[j], array[i]];
  }
  return array;
}

const count = ref(3)
//每隔n秒读一个单词
const speakContent = () =>{
  showMask.value = true;
  count.value = 3
  const timer = setInterval(()=>{
    count.value--
    if(!count.value){
      count.value = '听写中'
      speekTimer()
      clearInterval(timer)
    }
  },1000)
}

const speekTimer = ()=>{
  const word = shuffledArr.value[speekIndex.value]
  speekOneWord(word)
  console.log(1, speekIndex.value, shuffledArr.value.length)
  speekIndex.value++
  timer =  setTimeout( () => {
     if(speekIndex.value < shuffledArr.value.length){ 
        speekTimer()
      }else{
        showMask.value = false;
        clearTimeout(timer)
      }
  }, word.length * 4000)
 
}

// 读一个词语，两遍
const speekOneWord = (word) =>{
  var utterThis = new window.SpeechSynthesisUtterance(word + '\n' +word);
  window.speechSynthesis.speak(utterThis);
}

const saveToLocal = () => {
  localStorage.setItem('lastContent', content.value)
  getLocal()
}
const getLocal = ()=>{
  lastContent.value = localStorage.getItem('lastContent')
}
getLocal()

</script>

<template>
  <div>
      <h3>输入听写内容</h3>
      <div>
        <textarea v-model="content" class="textarea"/>
      </div>
      <button @click="handleStart">开始</button>
     
      <!-- <button @click="handlePause">暂停</button> -->
      <h3>上一次听写记录</h3>
      <div>
        {{lastContent}}
        <button @click="handleAgain">重新听写</button>
      </div>
      <div class="mask" v-if="showMask">
        {{count}}
       
      </div> 
  </div>
</template>

<style scoped>

h3 {
  font-size: 1.2rem;
}

.textarea{
  width: 100%;
  height: 16rem;
}
.mask{
  position: fixed;
  top:0;
  left:0;
  right:0;
  bottom: 0;
  background:rgba(0,0,0,.5);
  color: #fff;
  font-size: 3rem;
  text-align: center;
}


</style>
