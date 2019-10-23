<template>
  <section class="todoapp">
    <!-- header -->
    <header class="header">
      <input class="new-todo" autocomplete="off" placeholder="Todo List" @keyup.enter="addTodo" />
    </header>
    <!-- main section -->
    <section v-show="todos.length" class="main">
      <input
        id="toggle-all"
        :checked="allChecked"
        class="toggle-all"
        type="checkbox"
        @change="toggleAll({ done: !allChecked })"
      />
      <label for="toggle-all" />
      <ul class="todo-list">
        <todo
          v-for="(todo, index) in filteredTodos"
          :key="index"
          :todo="todo"
          @toggleTodo="toggleTodo"
          @editTodo="editTodo"
          @deleteTodo="deleteTodo"
        />
      </ul>
    </section>
    <!-- footer -->
    <footer v-show="todos.length" class="footer">
      <span class="todo-count">
        <strong>{{ remaining }}</strong>
        {{ remaining | pluralize('item') }} left
      </span>
      <ul class="filters">
        <li v-for="(val, key) in filters" :key="key">
          <a
            :class="{ selected: visibility === key }"
            @click.prevent="visibility = key"
          >{{ key | capitalize }}</a>
        </li>
      </ul>
      <ul class="filters">
        <a>epaper IP :</a>
        <input id="set-ip-id" class="set-ip" :placeholder="epaper.epaperIP" @keyup.enter="setIP" />
        <!--<a>epaper Port :</a>-->
        <!--<input class="set-port" autocomplete="off" placeholder="6502" @keyup.enter="setPort" />-->
        <a>rotate :</a>
        <input class="rotate" :placeholder="picRotate" @keyup.enter="rotate" />
      </ul>
    </footer>
  </section>
</template>

<script>
import axios from "axios";
import Todo from "./Todo.vue";

const STORAGE_KEY = "todos";
const ipport_key = "ipport";
const rotate_key = "rotate";
const filters = {
  all: todos => todos,
  active: todos => todos.filter(todo => !todo.done),
  completed: todos => todos.filter(todo => todo.done)
};
const defalutList = [
  { text: "test", done: false },
  { text: "test1", done: false }
];
function processTodo(obj) {
  var list = [];
  for (var item in obj) {
    list.push(obj[item]);
  }
  return list;
}
function processIPPort(obj) {
  console.log(document.getElementById("set-ip-id"));
  console.log("epaper IP is : " + obj.epaperIP);
  document.getElementById("set-ip-id");
  //  .setAttribute("placeholder", obj.epaperIP);
  // { epaperIP: "127.0.0.1", epaperPort: "6502" },
  return { epaperIP: obj.epaperIP, epaperPort: "6502" };
}
function processRotate(obj) {
  console.log("now rotate is : " + obj);
  return obj;
}
export default {
  components: { Todo },
  filters: {
    pluralize: (n, w) => (n === 1 ? w : w + "s"),
    capitalize: s => s.charAt(0).toUpperCase() + s.slice(1)
  },
  data() {
    return {
      visibility: "all",
      filters,
      //defalutList,
      todos: processTodo(JSON.parse(window.localStorage.getItem(STORAGE_KEY))), //defalutList,//JSON.parse(window.localStorage.getItem(STORAGE_KEY)),
      epaper: processIPPort(
        JSON.parse(window.localStorage.getItem(ipport_key))
      ),
      picRotate: processRotate(window.localStorage.getItem(rotate_key))
    };
  },
  computed: {
    allChecked() {
      return this.todos.every(todo => todo.done);
    },
    filteredTodos() {
      return filters[this.visibility](this.todos);
    },
    remaining() {
      return this.todos.filter(todo => !todo.done).length;
    }
  },
  methods: {
    setLocalStorage() {
      window.localStorage.setItem(STORAGE_KEY, JSON.stringify(this.todos));

      var obj = [];
      obj.push({ rotate: this.picRotate });
      obj.push({
        rectangle: {
          colour: 4,
          fill: true,
          height: 26,
          position: [0, 0],
          wide: 640
        }
      });
      obj.push({
        rectangle: {
          colour: 4,
          fill: true,
          height: 26,
          position: [0, 200],
          wide: 640
        }
      });
      obj.push({
        string: {
          colour: 3,
          content: "---Max's inprogress list---",
          font: 20,
          position: [20, 3]
        }
      });

      obj.push({
        string: {
          colour: 3,
          content: "---Max's to do list---",
          font: 20,
          position: [20, 203]
        }
      });

      var todoindex = 0;
      var doneindex = 0;
      this.todos.forEach(todo => {
        if (todo.done) {
          doneindex++;
          if (doneindex > 5) {
            return;
          }
          obj.push({
            string: {
              colour: 0,
              content: doneindex + "." + todo.text,
              font: 20,
              position: [20, 26 * doneindex]
            }
          });
        } else {
          todoindex++;
          if (todoindex > 5) {
            return;
          }
          obj.push({
            string: {
              colour: 0,
              content: todoindex + "." + todo.text,
              font: 20,
              position: [20, 200 + 26 * todoindex]
            }
          });
        }
      });
      //console.log("obj is : " + JSON.stringify(obj));

      var strtosend = JSON.stringify(obj);
      const service = axios.create({
        headers: {
          "Content-Type": "application/json"
        }
      });
      service
        .post(
          "http://" +
            this.epaper.epaperIP +
            ":" +
            this.epaper.epaperPort +
            "/v1/api/epaper/group/",
          strtosend
        )
        .then(function(response) {
          console.log(response);
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    rotate(e) {
      this.picRotate = e.target.value % 4;
      window.localStorage.setItem(rotate_key, this.picRotate);
      console.log("now rotate is : " + this.picRotate);
    },
    setIP(e) {
      this.epaper.epaperIP = e.target.value;
      this.epaper.epaperPort = "6502";
      window.localStorage.setItem(ipport_key, JSON.stringify(this.epaper));
      //console.log("epaper IP is :" + this.epaper.epaperIP);
    },
    setPort(e) {
      this.epaper.epaperPort = e.target.value;
      //console.log("epaper port is : " + this.epaper.Port);
    },
    addTodo(e) {
      const text = e.target.value;
      if (text.trim()) {
        this.todos.push({
          text,
          done: false
        });
        this.setLocalStorage();
      }
      e.target.value = "";
    },
    toggleTodo(val) {
      val.done = !val.done;
      this.setLocalStorage();
    },
    deleteTodo(todo) {
      this.todos.splice(this.todos.indexOf(todo), 1);
      this.setLocalStorage();
    },
    editTodo({ todo, value }) {
      todo.text = value;
      this.setLocalStorage();
    },
    clearCompleted() {
      this.todos = this.todos.filter(todo => !todo.done);
      this.setLocalStorage();
    },
    toggleAll({ done }) {
      this.todos.forEach(todo => {
        todo.done = done;
        this.setLocalStorage();
      });
    }
  }
};
</script>

<style lang="scss">
@import "./index.scss";
</style>
