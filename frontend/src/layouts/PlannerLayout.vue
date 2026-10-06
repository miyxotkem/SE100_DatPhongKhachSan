<script setup lang="ts">
import { useRouter } from 'vue-router';
import { Home, Map, Compass } from 'lucide-vue-next';
import AppHeader from '@/components/layout/AppHeader.vue';

const router = useRouter();
</script>

<template>
  <div class="flex flex-col h-screen w-screen overflow-hidden font-sans select-none">
    <!-- Header (dùng chung với MainLayout) -->
    <AppHeader />

    <!-- Body -->
    <div class="flex-1 flex min-h-0 overflow-hidden p-4 gap-4 bg-gradient-to-br from-slate-100 via-stone-50 to-amber-50">

      <!-- Sidebar thu gọn (icon only) dành riêng cho Planner -->
      <aside class="w-16 flex-shrink-0 relative rounded-3xl shadow-2xl border border-white/40 flex flex-col items-center py-6 justify-between text-white z-20 overflow-hidden bg-gradient-to-b from-stone-800 via-stone-900 to-neutral-900">
        <!-- Navigation Icon Buttons -->
        <nav class="relative z-10 flex flex-col items-center gap-4">
          <button
            class="w-10 h-10 rounded-full flex items-center justify-center text-white/80 hover:bg-white/20 hover:text-white transition drop-shadow-sm"
            title="Trang chủ"
            @click="router.push('/dashboard')"
          >
            <Home class="w-5 h-5" />
          </button>

          <!-- Active: Planner -->
          <button
            class="w-10 h-10 rounded-full flex items-center justify-center bg-white text-slate-800 shadow-md font-bold"
            title="Lập lịch trình (Planner)"
          >
            <Map class="w-5 h-5 text-amber-600" />
          </button>

          <button
            class="w-10 h-10 rounded-full flex items-center justify-center text-white/80 hover:bg-white/20 hover:text-white transition drop-shadow-sm"
            title="Chuyến đi của tôi"
            @click="router.push('/trips')"
          >
            <Compass class="w-5 h-5" />
          </button>
        </nav>

        <div class="h-20 pointer-events-none"></div>
      </aside>

      <!-- Main Content -->
      <main class="flex-1 min-w-0 relative z-10 bg-white/95 backdrop-blur-md rounded-3xl shadow-2xl border border-white/60 overflow-hidden flex flex-col">
        <router-view />
      </main>
    </div>
  </div>
</template>
