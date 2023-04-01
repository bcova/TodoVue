<script setup>
import { ref, onMounted, computed, watch } from "vue";
import { themeChange } from 'theme-change'
const todos = ref([]);
const name = ref("");


const input_content = ref("");
const input_category = ref(null);

const todos_asc = computed(() =>
  todos.value.sort((a, b) => {
    return b.createdAt - a.createdAt;
  })
);

const addTodo = () => {
  if (input_content.value.trim() === "" || input_category.value === null) {
    return;
  }

  let date = new Date();
  let year = date.getFullYear();
  let last2 = year.toString().slice(-2);
	let current_date = (date.getMonth()+1)+"-"+ date.getDate()+"-"+last2
	let current_time =  (date.getHours()+24)%12 +":"+(date.getMinutes()<10?'0':'') + date.getMinutes()
	let date_time = current_date+"  🕒: "+current_time;	


  todos.value.push({
    content: input_content.value,
    category: input_category.value,
    done: false,
    createdAt: new Date().getTime(),date_time
  });

  input_content.value = "";
  input_category.value = null;
};

const removeTodo = (todo) => {
  if (confirm(` Are you sure you would like to delete ${todo.content}`))
    todos.value = todos.value.filter((t) => t !== todo);
};


watch(
  todos,
  (newVal) => {
    localStorage.setItem("todos", JSON.stringify(newVal));
  },
  { deep: true }
);

onMounted(() => {
  name.value = localStorage.getItem("name") || "";
  todos.value = JSON.parse(localStorage.getItem("todos")) || [];
  themeChange(false)
});
</script>

<template>
  <main class="app bg-base-100 h-screen w-full flex justify-center items-center "  >
    <div
      class="rounded-lg w-full h-full p-4 border-4 border-secondary flex flex-col justify-between border-opacity-0 " 
    >
    <span class="self-end flex flex-col items-center"><span class="flex items-center">☀️<input data-choose-theme data-toggle-theme="halloween,cupcake" type="checkbox" className="toggle mx-2 bg-secondary border-primary " checked />🌙</span></span>
    
      <section class="greeting w-full text-center">
        <div class="title text-3xl mb-3">
          <div v-if="name">What's up, {{ name }}!</div>
          <div v-else>Hi, What's your name?</div>
</div>
        <input
          type="text"
          placeholder="Name here"
          v-model="name"
          className="input inputName border-b-2 border-t-0 border-l-0 border-r-0 border-primary w-full max-w-xs rounded-none rounded-tl-md rounded-tr-md focus:outline-none bg-gray-500 bg-opacity-20 caret-primary caret- focus:placeholder:invisible autofill:bg-secondary"
          id="name"
        />
      </section>

      <section class="create-todo flex flex-col justify-center items-center w-full ">
        <form @submit.prevent="addTodo">
          <h4 class="text-center my-4">What's on your todo list?</h4>
          <input
            type="text"
            placeholder="e.g make a video"
            v-model="input_content"
            className="input border-b-2 border-t-0 border-l-0 border-r-0 border-primary w-full max-w-xs rounded-none rounded-tl-md rounded-tr-md focus:outline-none bg-gray-500 bg-opacity-20 caret-primary my-4 focus:placeholder:invisible"
          />

          <h4 class="text-center">Pick a category</h4>

          <div class="options flex justify-evenly my-3">
            <label class="text-center">
              <input
                type="radio"
                name="category"
                className="radio checked:bg-primary peer/business"
                value="business"
                v-model="input_category"
              />
              <div class="peer-checked/business:text-primary transition-all duration-300">
                Business
              </div>
            </label>
            <label class="text-center">
              <input
                type="radio"
                name="category"
                className="radio checked:bg-secondary peer/personal"
                value="personal"
                v-model="input_category"
              />
              <div
                class="peer-checked/personal:text-secondary transition-all duration-300"
              >
                Personal
              </div>
            </label>
          </div>

          <input
            class="w-full hover:cursor-pointer btn btn-secondary text-white"
            type="submit"
            value="Add todo"
          />
        </form>
      </section>

      <section
        class="todo-list w-full h-fit grid grid-cols-1 justify-center overflow-y-scroll overflow-x-hidden items-center "
      >
        <div class="list w-full flex flex-col items-center h-64">
          <div
            class="w-full h-fit"
            v-for="todo in todos_asc"
            :class="`todo-item ${todo.done && 'done'}`"
          >
            <div class="todo-content flex justify-center items-center">
              <label type="checkbox" class="grid">
                <div
                  v-if="todo.category.charAt(0).toUpperCase() === 'B'"
                  className="badge badge-primary font-bold mb-3 "
                >
                  {{ todo.category.charAt(0).toUpperCase() }}
                </div>
                <div v-else className="badge badge-secondary font-bold mb-3">
                  {{ todo.category.charAt(0).toUpperCase() }}
                </div>
                <input
                  v-if="todo.category.charAt(0).toUpperCase() === 'B'"
                  className="radio checked:bg-accent checked:border-accent peer border-primary"
                  type="checkbox"
                  v-model="todo.done"
                />
                <input 
                  v-else
                  className="radio checked:bg-accent checked:border-accent peer border-secondary"
                  type="checkbox"
                  v-model="todo.done"
                />
                <span :class="`bubble ${todo.category}`"></span>
              </label>
              <input
                className="radio checked:bg-accent peer invisible"
                type="checkbox"
                v-model="todo.done"
              />
             
                <p v-if="todo.category.charAt(0).toUpperCase() === 'B'"
                @click="edit"
                  type="text"
                  
                  className="input border-b-2 border-t-0 border-l-0 border-r-0 border-primary w-full max-w-xs rounded-none rounded-tl-md rounded-tr-md focus:outline-none bg-gray-500 bg-opacity-20 caret-primary my-4 peer-checked:border-accent peer-checked:bg-accent peer-checked:bg-opacity-20 transition-colors duration-300" 
                  
                >{{ todo.content }}</p>
                <input v-else
                @click="edit"
                  type="text"
                  v-model="todo.content"
                  className="input border-b-2 border-t-0 border-l-0 border-r-0 border-secondary w-full max-w-xs rounded-none rounded-tl-md rounded-tr-md focus:outline-none bg-gray-500 bg-opacity-20 caret-secondary my-4 peer-checked:border-accent peer-checked:bg-accent peer-checked:bg-opacity-20 transition-colors duration-300"
                  
                />
                
            </div>
            <div className="  divider before:bg-zinc-700 after:bg-zinc-700 w-full px-10 ">
              📅: {{ todo.date_time }}
              <button class="" @click="removeTodo(todo)"><label htmlFor="my-modal-4" className="btn">open modal</label>


<input type="checkbox" id="my-modal-4" className="modal-toggle" />
<label htmlFor="my-modal-4" className="modal cursor-pointer">
  <label className="modal-box relative" htmlFor="">
    <h3 className="text-lg font-bold">Congratulations random Internet user!</h3>
    <p className="py-4">You've been selected for a chance to get one year of subscription to use Wikipedia for free!</p>
  </label>
</label></button>
            </div>
          </div>
        </div>
      </section>
    </div>
  </main>
</template>
