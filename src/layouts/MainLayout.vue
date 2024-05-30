<template>
  <q-layout view="lHh Lpr lFf">

    <q-header elevated>
      <q-toolbar class="text-center q-py-md bg-white text-black">
        <q-toolbar-title>
          Locate Hub
        </q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
    
  </q-layout>
</template>

<script>

  export default {
    name: 'MainLayout',
  }
</script>
