<script setup lang="ts">
import { useRoute } from 'vue-router';
import { Home, Map, Compass, Globe } from 'lucide-vue-next';

const route = useRoute();

const menuItems = [
  { name: 'Trang chủ', path: '/dashboard', icon: Home },
  { name: 'Chuyến đi của tôi', path: '/trips', icon: Map },
  { name: 'Khám phá', path: '/explore', icon: Compass },
  { name: 'Bản đồ thế giới', path: '/map', icon: Globe },
];
</script>

<template>
  <aside
    class="w-56 flex-shrink-0 relative rounded-3xl shadow-2xl border border-white/40 flex flex-col justify-between p-4 text-white z-20 overflow-hidden bg-gradient-to-b from-stone-800 via-stone-900 to-neutral-900"
  >
    <!-- Navigation Menu -->
    <div class="relative z-10 space-y-3 pt-2">
      <nav class="space-y-2">
        <router-link
          v-for="item in menuItems"
          :key="item.path"
          :to="item.path"
          :class="[
            'flex items-center gap-3 px-4 py-2.5 rounded-full text-xs font-bold transition-all duration-200',
            route.path === item.path
              ? 'bg-white text-slate-800 shadow-lg'
              : 'text-white/80 hover:bg-white/20 hover:text-white drop-shadow-sm',
          ]"
        >
          <component
            :is="item.icon"
            :class="['w-4 h-4 flex-shrink-0', route.path === item.path ? 'text-amber-600' : 'text-white/80']"
          />
          <span class="truncate">{{ item.name }}</span>
        </router-link>
      </nav>
    </div>
    <div class="h-28 pointer-events-none"></div>
  </aside>
</template>
