<script lang="ts" setup>
import { ref, watch } from "vue";
import axios from "axios";
import Modaltest from "../components/ModalTodo.vue";
import { useToast } from "vue-toast-notification";
import 'vue-toast-notification/dist/theme-bootstrap.css'

const toast = useToast();
const isModalOpen = ref(false);
const titleModal = ref("");
const todo = ref({
  id: Number(),
  taskName: String(),
  description: String(),
  status: String(),
  date: Date(),
});
const dataTable = ref([]) as any;
let _dataTable = ref([]) as any;
let textSearch = ref('');
watch(textSearch, (newValue, oldValue) => {
      if(newValue.length === 0){
        dataTable.value = _dataTable.value
      }
    });
const getApi = async () => {
  try {
   let res =  await axios.get("http://localhost:3001/todo");
   if(res){
    dataTable.value = res.data;
    _dataTable.value = res.data;
   }
  }catch(e){
    throw new Error('Request error')
  }
    
};
const postApi = async (todo:any) => {
  isModalOpen.value = false;
  if (titleModal.value === "Add new task") {
    try {
      await axios
        .post("http://localhost:3001/todo", todo)
        .then(() => {
          toast.success("Add with success");
          getApi();
        });
    } catch (e) {
      toast.error("Error to add");
    }
  }
  if (titleModal.value === "Update task") {
    try {
      await axios
        .patch(`${`http://localhost:3001/todo`}/${todo.id}`, todo)
        .then(() => {
          toast.success("Update with success");
          getApi();
        });
    } catch (e) {
      toast.error("Error to update");
    }
  }
};
const handleSearch = () => {
  let data = _dataTable.value;
  if(textSearch.value.length !== 0) {
    let name = textSearch.value.toUpperCase();
    dataTable.value = [...data].filter((item: any)=> {
      if(item.taskName.toUpperCase().toString().includes(name) 
      || item.description.toUpperCase().toString().includes(name) 
      || item.date.toUpperCase().toString().includes(name) 
      || item.status.toUpperCase().toString().includes(name)){
        return true
      }
      return false
    })
  }else{
    dataTable.value = data
  }
}
const handleAdd = () => {
  titleModal.value = "Add new task";
  isModalOpen.value = true;
  todo.value = {
    taskName: String(),
    description: String(),
    date: Date(),
    status: String(),
    id:0
  };
};
const openmodal = () => {
  isModalOpen.value = true;
};
const convertStatus = (status: any) => {
  return status.split(' ').join('-')
}
const modifier = (row: any) => {
  todo.value = {
    taskName: row.taskName,
    description: row.description,
    date: row.date,
    status: row.status,
    id:row.id
  };
};
const supprimer = (todo:any) => {
  try {
    axios.delete(`${`http://localhost:3001/todo`}/${todo.id}`).then(() => {
      toast.success("Delete with success");
      getApi();
    });
  } catch (e) {
    toast.error("Error to delete");
  }
};
const closeModal = () => {
  isModalOpen.value = false;
};
getApi();
</script>
<template>
  <div class="d-flex felx-row">
    <div class="d-flex flex-row justify-content-start w-50 p-1">
      <input class="w-100" v-model="textSearch" placeholder="Search..." @keyup.enter.prevent="handleSearch"/>
      <button @click="handleSearch" type="button" class="btn-custom-primary">
        Search
      </button>
    </div>
    <div class="d-flex flex-row justify-content-end w-50 p-1">
      <button @click="handleAdd" type="button" class="btn-custom-secondary">
        <i class="fa fa-plus" aria-hidden="true"></i>
        Add
      </button>
    </div>
  </div>
  <Modaltest
    :isModalOpen="isModalOpen"
    :titleModal="titleModal"
    :todo="todo"
    @close="closeModal()"
    @todo="postApi"
  />
  <div>
    <table class="table">
      <thead class="thead-dark">
        <tr>
          <th scope="col">ID</th>
          <th scope="col">Task name</th>
          <th scope="col">Description</th>
          <th scope="col">Date</th>
          <th scope="col">status</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item,index) in dataTable" :key="index">
                  <td scope="row">{{ item.id }}</td>
                  <td>{{ item?.taskName }}</td>
                  <td>{{ item?.description }}</td>
                  <td>{{ item?.date }}</td>
                  <td>
                    <span :class="`type-${convertStatus(item?.status)}`">{{ item?.status }}</span>
                  </td>
                  <td class="d-flex justify-content-center flex-row gap-10">
                    <button
                      @click="
                        openmodal();
                        modifier(item);
                        titleModal = 'Update task';
                      "
                      type="button"
                      class="btn btn-info mr-1">
                      <i class="fa fa-pencil" aria-hidden="true"></i>
                    </button>
                    <button
                      @click="supprimer(item)"
                      type="button"
                      class="btn btn-danger">
                      <i class="fa-solid fa-trash"></i>
                    </button>
                  </td>
                </tr>
      </tbody>
    </table>
  </div>
</template>
<style>
thead {
  background-color: black !important;
}


svg {
  padding: 0 !important;
  background: transparent !important;
  color: white !important;
}
.btn-custom-primary {
  background-color: #28a745;
  outline: none;
  color: #ffff;
  font-weight: 700;
  border-radius: 0px 5px 5px 0px;
  border: 1px solid #28a745;
  padding: 10px;
}
.btn-custom-secondary {
  background-color: #007bff;
  outline: none;
  color: #ffff;
  font-weight: 700;
  border-radius: 5px;
  border: 1px solid #007bff;
  padding: 10px;
}
.gap-10 {
  gap: 5px;
}
.type-in-progress {
  color: #17a2b8 !important;
  border: 1px solid #17a2b8;
  padding: 4px;
  border-radius: 4px;
}
.type-todo {
  color: #ffc107 !important;
  border: 1px solid #ffc107;
  padding: 4px;
  border-radius: 4px;
}
.type-completed {
  color: #28a745 !important;
  border: 1px solid #28a745;
  padding: 4px;
  border-radius: 4px;
}
</style>
