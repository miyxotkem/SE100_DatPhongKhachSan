<script setup lang="ts">
import AppHeader from '@/components/layout/AppHeader.vue';
import AppSidebar from '@/components/layout/AppSidebar.vue';
</script>

<template>
  <div class="flex flex-col h-screen w-screen overflow-hidden font-sans select-none">
    <!-- Header -->
    <AppHeader />

    <!-- Body -->
    <div class="flex-1 flex min-h-0 overflow-hidden p-4 gap-4 bg-gradient-to-br from-slate-100 via-stone-50 to-amber-50">
      <!-- Sidebar -->
      <AppSidebar />

      <!-- Main Content -->
      <main
        class="flex-1 min-w-0 relative z-10 bg-white/95 backdrop-blur-md rounded-3xl shadow-2xl border border-white/60 overflow-y-auto p-6 flex flex-col"
      >
        <router-view />
      </main>
    </div>
  </div>
</template>
