<template>
   <div class="wrapper">
     <!-- Floor decoration components -->
     <tpl />
   </div>
</template>
<script>
import tpl from "@/pages/tabbar/home/views.vue";

export default {

   data() {
     return {
       background: {
         backgroundColor: "#fff",
       },
     };
   },
   components: {
     tpl,
   },

};
</script>

<style lang="scss" scoped>

</style>
