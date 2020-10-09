<template>
  <div class="p-8">
      <div class="flex justify-around">
        <!-- Rather than one huge giant list, I had the list ids be buttons you could select to view the ones in that list, which seemed easier to use and grok than a whole big list of items -->
        <button 
          v-for="group in state.list" :key="group.listId" 
          @click="state.selectedList = group.listId" 
          class="flex border rounded border-black m-4 text-2xl p-4 transition-all duration-500" 
          :class="isActive(group.listId)">
          List {{group.listId}}
        </button>
      </div>
      <hr class="my-2">
      <!-- I added the transition to ease the appearance of the large lists changing over -->
    <transition name="fade" mode="out-in">
      <div class="flex flex-wrap justify-center" v-if="state.selectedList != null" :key="state.selectedList">
        <p v-for="item in state.list[state.selectedList].items" :key="item.id" 
        class="text-xl border rounded shadow-md p-2 w-1/5 m-4">
          {{item.name}}
        </p>
      </div>
    </transition>
  </div>
</template>

<style>

.fade-enter-active, .fade-leave-active {
  transition: opacity .5s ease;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
</style>

<script>
import { onMounted, reactive } from 'vue'

const getInt = (string) => {
  //This was needed to sort the items correctly, since it would put 31 after 300, for example, if you just compared the strings
  return parseInt(string.match(/(\d+)/)[0])
}

export default {
  name: 'App',
  setup(){
    const isActive = (num) => {
      //Controls the classes on the active button, tried to use shadow to accentuate it so it wasn't just a color change
      if (state.selectedList == num) return ['text-white','bg-green-400', 'shadow-lg']
      else return ['text-black', 'bg-white', 'shadow-none']
    }
    let state = reactive({
      list: {},
      selectedList: null
    })
    onMounted(() => {
      //There was a CORS issue with the link given, so I downloaded the file to pull locally rather than try to worry about that getting changed
      fetch('/hiring.json')
      .then(res => res.json())
      .then(data => {
        //Fetching, then first filtering out any nameless items. This made the code a bit more readable
        const filtered = data.filter((item) => item.name != null && item.name.length > 0)
        for (let item of filtered) {
          if (state.list[item.listId]) {
            for (let [i,x] of state.list[item.listId].items.entries()) {
              if ( getInt(item.name) < getInt(x.name)) {
                state.list[item.listId].items.splice(i, 0, item)
                break
              }
            }
          } else {
            state.list[item.listId] = {listId: item.listId, items: [item]}
          }
        }
      })
    })
    return {state, isActive}
  }
}
</script>
