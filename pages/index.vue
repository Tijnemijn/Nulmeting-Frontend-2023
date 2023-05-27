<template>
  <div class="page-wrapper">
    <div class="element-wrapper">
      <div class="element-body">
        <h1>Welkom bij de Levarne Nulmeting</h1>
        <LevButton @clicked="todoListStore().GetTodo()">Krijg een nieuwe Todo</LevButton>
        <h4 id = "noNewTodo" style="display: none"> geen nieuwe Todo gevonden</h4>
        <br>
        <br>
        <h2>ToDo lijst:</h2>
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
    //list of the id's in the todolist
    idList: [] as string[],
    //the todo list on the page
    todoList: document.getElementById("todoList")!,
    noNewTodo: document.getElementById("noNewTodo")!,
  }),
  actions: {
    GetTodo(iteration: number = 0) {
      this.noNewTodo.style.display = "none";
      axios.request(config)
      .then((response) => {
        // check how often a new todo is aksed
        if (iteration < 10) {
          // check if the Todo item already exists in the list
          if (this.idList.includes(response.data.todo.id))
          {
            // get new item
            this.GetTodo(iteration + 1);
          }
          else
          {
            // add item to the list
            this.AddTodoToList(response.data.todo);
          }
        }
        else{
          this.noNewTodo.style.display = "block";
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
      todoItem.innerHTML = todo.assignee + ": " + todo.description + " (af voor: " + dueDateTime[0] + " om " + dueDateTime[1] +")";
      //create a button that will delete the Todo when it's done
      var button = document.createElement("button");
      button.style.backgroundColor = "lightgreen";
      button.innerHTML = "Done";
      //when the button is clicked delete the Todo
      button.onclick = function() 
      {
        todoListStore().RemoveTodo(todoItem, todo.id)
      };
      //add the Todo to the list
      todoItem.appendChild(button);
      this.todoList.appendChild(todoItem);
    },

    RemoveTodo(todoItem: Element, id: string) {
      todoItem.remove();
      this.noNewTodo.style.display = "none";
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