<script lang="ts" setup>
import moment from "moment"
import { defineProps, ref, defineEmits, toRefs, watch, computed } from "vue";
const emit = defineEmits(["close", "todo"]);
const props = defineProps({
  isModalOpen: Boolean,
  todo: Object(),
  titleModal: String,
});
let selected: any = ref('');
const { isModalOpen, todo } = toRefs(props);
let todoList = ref({
  taskName: String(),
  description: String(),
  date: Date(),
  status: String(),
})
watch(props, () => {
  todoList.value = todo.value;
  selected.value = todo.value.status;
});

const isAdd = computed(()=> {
  if(todoList.value.taskName.length !== 0 && todoList.value.description.length !== 0 && selected.value.length !== 0){
    return false
  }
  return true
})

const inputtaskname = ref();
const inputdescription = ref();

const submitForm = async () => {
  if(!isAdd.value){
    const _date = moment(Date.now()).format('DD/MM/YYYY');
    todoList.value = {...todoList.value, date: `${_date}`, status: selected.value }
    const value = todoList.value;
    if (value.taskName && value.description) {
      emit("todo", todoList.value);
    }
    if (!value.taskName) {
      inputtaskname.value.focus();
    }
    if (!value.description) {
      inputdescription.value.focus();
    }
  }
};
</script>

<template>
  <transition name="fade">
    <div v-if="isModalOpen" :class="$style.overlay"></div>
  </transition>
  <transition name="slide-fade">
    <div v-if="isModalOpen" :class="$style.modalContainer">
      <div :class="$style.modal" ref="" role="dialog">
        <header :class="$style.formHeadline">{{ titleModal }}</header>
        <main>
          <form :class="$style.form">
            <div :class="$style.formRow">
              <label for="task">Task Name</label>
              <input
                ref="inputtaskname"
                v-model="todoList.taskName"
                placeholder="Task Name"
                type="text"
                name="taskname"
                id="task"
                required
              />
              <span v-if="todoList.taskName.length === 0" class="required-text">Required task name</span>
            </div>
            <div :class="$style.formRow">
              <label for="description">Description</label>
              <textarea
                id="description"
                ref="inputdescription"
                v-model="todoList.description"
                type="text"
                name="description"
                class="w-100"
                required
              />
              <span v-if="todoList.description.length === 0" class="required-text">Required description</span>
            </div>
            <div :class="$style.formRow">
              <label for="status">Status</label>
              <select
                v-model="selected"
                class="form-select"
                aria-label="Default select example"
                required
              >
              <option value="" disabled>Select status</option>
                <option value="todo">Todo</option>
                <option value="in progress">In progress</option>
                <option value="completed">Completed</option>
              </select>
              <span v-if="selected.length === 0" class="required-text">Required to select status</span>
            </div>

            <div :class="$style.formActions">
              <button @click.prevent="$emit('close')" class="btn-custom-close">Close</button>
              <button :disabled="isAdd" @click.prevent="submitForm()" :class="isAdd ? 'btn-custom-add-disabled': 'btn-custom-add'">Add</button>
            </div>
          </form>
        </main>
      </div>
    </div>
  </transition>
</template>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease-in-out;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all 0.2s ease-in-out;
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateY(2rem);
  opacity: 0;
}

.btn-custom-close {
  color: #343a40;
  outline: none;
  border: 1px solid #343a40;
  padding: 10px;
  border-radius: 4px;
  font-weight: 700;
  width: 100px;
}
.btn-custom-add {
  color: #28a745;
  outline: none;
  border: 1px solid #28a745;
  padding: 10px;
  border-radius: 4px;
  font-weight: 700;
  width: 100px;
}
.btn-custom-add-disabled {
  color: #28a745;
  outline: none;
  border: 1px solid #28a745;
  padding: 10px;
  border-radius: 4px;
  font-weight: 700;
  width: 100px;
  opacity: 0.5;
}
.required-text {
  color: red !important;
  padding: 2px;
}
</style>

<style module>
.overlay {
  background: rgba(0, 0, 0, 0.3);
  position: fixed;
  inset: 0;
}

.modalContainer {
  position: fixed;
  inset: 0;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal {
  width: 25rem;
  margin: 0 auto;
  padding: 2rem;
  z-index: 10;
  background-color: white;
  transform: translateY(-2rem);
}

.form {
  margin: 0;
}

.formHeadline {
  font-size: 1.6rem;
  margin-bottom: 2rem;
}

.formRow {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  margin-bottom: 1.5rem;
}

.formRow label {
  margin-bottom: 0.5rem;
  display: block;
  width: 100%;
  text-align: left;
  flex-basis: 100%;
}

.formRow input {
  flex-basis: 100%;
  padding: 0.5rem 0.75rem;
}

.formActions {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 1rem;
}
</style>
