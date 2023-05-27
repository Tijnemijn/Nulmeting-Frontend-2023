<template>
  <div class="page-wrapper">
    <div class="element-wrapper">
      <div class="element-body">
        <h1>Welkom bij de Levarne Nulmeting</h1>
        <LevButton @clicked="todoListStore().GetTodo()">Get a new ToDo</LevButton>
        <br>
        <br>
        <h2>ToDo list:</h2>
        <ul id="todoList" display="table-cell">
        </ul>
        <br>
        <br>
        <NuxtLink to="/about">Over ons</NuxtLink>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { assertTSModuleBlock, removePropertiesDeep } from '@babel/types';
import axios from 'axios';
import data from '~/assets/data.json';
import { defineStore } from 'pinia';
import { createElementBlock } from 'vue';
const api_key = data.api_key;
//list of all the id's in the toDoList
let config = {
  method: 'get',
  maxBodyLength: Infinity,
  url: 'https://86a4h9y007.execute-api.eu-west-1.amazonaws.com/development/nulmeting/todo',
  headers: { 
    'x-api-key': api_key,
  }
};

const todoListStore = defineStore('todoList', {
  state: () => ({
    idList: [] as string[],
    todoList: document.getElementById("todoList")!,
  }),
  actions: {
    GetTodo() {
      axios.request(config)
      .then((response) => {
        // check if the Todo item already exists in the list
        if (this.idList.includes(response.data.todo.id))
        {
          // get new item
          this.GetTodo();
        }
        else
        {
          // add item to the list
          this.AddTodoToList(response.data.todo);
        }
      })
      .catch((error) => {
        console.log(error);
      });
    },
    AddTodoToList(todo: any) {
      //add the id to the list of id's
      this.idList.push(todo.id);
      //create a new list item
      const todoItem = document.createElement("li");
      todoItem.style.display = "flex";
      todoItem.style.justifyContent = "space-between";
      //get the date and time in in seperate variables and in the correct format
      const dueDateTime = this.GetCorrectDateTime(todo.dueDateTime);
      todoItem.innerHTML = todo.assignee + ": " + todo.description + " (due: " + dueDateTime[0] + " at " + dueDateTime[1] +")";
      //create a button that will delete the Todo when it's done
      var button = document.createElement("button");
      button.style.backgroundColor = "lightgreen";
      button.innerHTML = "Done";
      button.onclick = function() 
      {
        todoItem.remove();
        todoListStore().RemoveId(todo.id);
        
      };
      todoItem.appendChild(button);
      this.todoList.appendChild(todoItem);
    },
    RemoveId(id: string) {
      // get the index of the id in the list
      const localIndex = this.idList.indexOf(id, 0);
      // check if it is a valid index and remove it from the list
      if (localIndex > -1) 
      {
        this.idList.splice(localIndex, 1);
      }
    },
    GetCorrectDateTime(dateTime: string) {
      //seperate the date and time
      const dueDate = dateTime.split("T")[0];
      var dueTime = dateTime.split("T")[1] ;
      //clean up the time so you only get the ours and minutes
      dueTime = (dueTime.split(".")[0]);
      dueTime = (dueTime.split(":")[0] + ":" + dueTime.split(":")[1]);
      return [dueDate, dueTime];
    }
  }
});
</script>