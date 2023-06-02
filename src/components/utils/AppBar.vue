<template>
  <div
      :class="{'bg-white': !dark, 'bg-transparent': dark}" class="navigator  py-4 mb-9 px-3 w-full relative flex items-center"
     >
       <v-btn @click="$router.go(-1)" fab text small class="z-10"
         ><v-icon v-if="dark" color="white">mdi-arrow-left</v-icon>
         <v-icon v-else color="black">mdi-arrow-left</v-icon>
         
         </v-btn
       >
       <p :class="{'text-black': !dark, 'text-white': dark}" class="ma-0 pa-0 text-center w-full absolute text-title">
         {{ title }}
       </p>
 
     </div>
</template>

<script>
export default {
    name: "AppBar",
    props:['title', 'dark']

}
</script>

<style>

</style>