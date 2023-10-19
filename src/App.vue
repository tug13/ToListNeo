<script lang="ts" setup>
import axios from 'axios';
import { computed, ref, onMounted } from 'vue';
let isNotification = ref(false)
const totalnotification = ref(Number())
let data = ref([]) as any;
onMounted(() => {
  getApi();
});
const groupByData = (data: any) => {
  let result = [...data].reduce((accumulator, current)=> {
    let isGroup = accumulator.find((item: any)=> item.status === current.status);
    if(isGroup){
      isGroup.data.push(current)
    }else{
      accumulator = [...accumulator, {status: current.status,data: [current]}]
    }
    return accumulator
  }, []);
  return result;
}
const getApi = async () => {
   let res =  await axios.get("http://localhost:3001/todo");
   if(res){
    totalnotification.value = res.data.length;
    data.value = groupByData(res.data);
   }
  
    
};
</script>
<template>
  <nav class="navbar navbar-light position-relative bg-light">
    <a class="navbar-brand p-1" href="#">Welcome</a>
    <button @click="isNotification=!isNotification" type="button" class="btn btn-primary">
      Notifications <span class="badge text-bg-secondary">{{ data.length }}</span>
    </button>
    <div v-if="isNotification" class="position-absolute custom-card">
      <div class="card notification" style="width: 18rem;">
      <div class="card-body">
        <div v-for="(item, index) of data" :key="index">
          <span>{{ item.status.toUpperCase() }} {{ `${item.data.length}`.padStart(2, '0') }}</span>
        </div>
      </div>
    </div>
    </div>
    
  </nav>
  <router-view />
</template>

<style>
.notification{
 float: right;
 z-index: 1000;
 position:absolute;
}
.custom-card {
  display: flex;
  justify-content: flex-end !important;
  padding-left: 12%;
  width: 100%;
  margin-top: 126px;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
