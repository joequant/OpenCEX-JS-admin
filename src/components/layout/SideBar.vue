<template>
  <v-app-bar flat color="blue lighten-5">
    <div class="pl-8" style="font-size: 18px;">Admin panel</div>
    <div style="position: absolute; right: 10px; top: 5px;">
      <v-menu>
        <template v-slot:activator="{ props }">
          <v-btn icon="mdi-account" v-bind="props">
          </v-btn>
        </template>
        <v-list>
          <v-list-item>
            <v-list-item-title @click="logout()">Logout</v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>
    </div>
  </v-app-bar>

  <v-navigation-drawer permanent>
    <v-list>
      <template v-for="item in navigation">
        <template v-if="item.icon">
          <v-list-item :to="`/page/${ item.link === '/' ? 'dashboard' : item.link.name}`" :title="item.text" :prepend-icon="item.icon" :value="item.text">
          </v-list-item>
        </template>
        <template>
          <v-divider></v-divider>
        </template>
      </template>
    </v-list>
  </v-navigation-drawer>
</template>
<script setup>
import axios from '../../plugins/axios';
import { onBeforeMount, ref } from 'vue'
import { useNavStore } from '@/stores/nav'
import initRouter from '@/router/'
import localConfig from "@/local_config"
import { useRoute } from 'vue-router'
const nav = useNavStore()

const apiKey = localConfig.api

const navigation = ref([])
const resources = ref([])

const logout = () => {
  localStorage.setItem('jwt_token', "")
  initRouter.push({name: 'Login'})
}
const route = useRoute()
const param = ref(route.path)
const getNavigation = async () => {
  if(!window.location.href.includes('/login'))
    try {
      const response = await axios.get(`${apiKey}navigation/`)
      navigation.value = response.data
      nav.setNavigation(response.data)
    } catch (error) {
      if(error?.response?.data?.type === "authentication_failed" || error?.response?.data?.code?.code === "token_not_valid" || error?.response?.data?.type === "not_authenticated") {
        initRouter.push({name: 'Login'})
      }
    }
} 

const getResources = async () => {
  try {
    const response = await axios.get(`${apiKey}resources/`);
    resources.value = response.data
    nav.setResources(response.data)
  } catch (error) {
    console.log(error.type);
  }
} 

onBeforeMount(() => {
  getNavigation()
  getResources()
})
</script>

<style>
.v-icon {
  opacity: 1 !important;
  visibility: visible !important;
}
</style>