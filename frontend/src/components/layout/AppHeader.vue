<script setup lang="ts">
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { Search, SlidersHorizontal, Bell, Moon, LogOut, ChevronDown } from 'lucide-vue-next';

const router = useRouter();

const showUserMenu = ref(false);
const searchQuery = ref('');

// User profile state
const user = ref({
  name: 'Thinh Phat Ho',
  email: 'hothinhphat06@gmail.com',
});

const handleLogout = () => {
  localStorage.removeItem('access_token');
  localStorage.removeItem('user_info');
  router.push('/login');
};
</script>

<template>
  <header class="h-16 bg-white border-b border-slate-200/80 px-6 flex items-center justify-between shadow-xs z-30 flex-shrink-0">
    <!-- Brand Logo -->
    <div class="flex items-center gap-2 cursor-pointer" @click="router.push('/dashboard')">
      <span class="text-2xl font-black tracking-tight text-[#0F172A] hover:text-brand-blue transition">
        Trip<span class="text-brand-blue">Planner</span>
      </span>
    </div>

    <!-- Center: Pill Search Bar with Sliders/Filter icon -->
    <div class="relative w-full max-w-md mx-6">
      <div class="relative flex items-center">
        <Search class="w-4 h-4 text-slate-400 absolute left-4 pointer-events-none" />
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Tìm kiếm địa điểm, thành phố, ..."
          class="w-full bg-slate-50 hover:bg-slate-100/80 text-slate-800 text-xs rounded-full pl-11 pr-11 py-2.5 transition focus:bg-white focus:outline-none focus:ring-2 focus:ring-brand-blue/30 border border-slate-200 focus:border-brand-blue placeholder:text-slate-400"
        />
        <button
          class="absolute right-3.5 p-1 rounded-full text-slate-400 hover:text-slate-700 hover:bg-slate-200 transition"
          title="Bộ lọc tìm kiếm"
        >
          <SlidersHorizontal class="w-3.5 h-3.5" />
        </button>
      </div>
    </div>

    <!-- Right Actions: Darkmode, Bell, Profile Badge -->
    <div class="flex items-center gap-3">
      <!-- Moon (Darkmode Toggle) -->
      <button
        class="w-9 h-9 rounded-full border border-slate-200 text-slate-600 hover:text-slate-900 hover:bg-slate-100 flex items-center justify-center transition"
        title="Chế độ tối"
      >
        <Moon class="w-4 h-4" />
      </button>

      <!-- Bell (Notification) -->
      <button
        class="relative w-9 h-9 rounded-full border border-slate-200 text-slate-600 hover:text-slate-900 hover:bg-slate-100 flex items-center justify-center transition"
        title="Thông báo"
      >
        <Bell class="w-4 h-4" />
        <span class="absolute top-1.5 right-1.5 w-2 h-2 bg-brand-orange rounded-full ring-2 ring-white"></span>
      </button>

      <!-- User Profile Pill Button -->
      <div class="relative">
        <button
          class="flex items-center gap-2 pl-1 pr-3 py-1 rounded-full bg-slate-50 hover:bg-slate-100 border border-slate-200 transition focus:outline-none"
          @click="showUserMenu = !showUserMenu"
        >
          <div class="w-7 h-7 rounded-full bg-[#0D9488] text-white flex items-center justify-center font-bold text-xs ring-2 ring-emerald-500/20">
            PH
          </div>
          <span class="text-xs font-bold text-slate-800 hidden sm:inline">{{ user.name }}</span>
          <ChevronDown class="w-3.5 h-3.5 text-slate-400" />
        </button>

        <!-- Dropdown Menu -->
        <div
          v-if="showUserMenu"
          class="absolute right-0 mt-2 w-48 bg-white rounded-2xl shadow-xl border border-slate-100 py-1.5 z-50 animate-in fade-in slide-in-from-top-2 duration-150"
        >
          <div class="px-4 py-2 border-b border-slate-100">
            <p class="text-[11px] text-slate-400">Đăng nhập với</p>
            <p class="text-xs font-bold text-slate-900 truncate">{{ user.email }}</p>
          </div>
          <button
            class="w-full px-4 py-2 text-left text-xs text-red-600 hover:bg-red-50 flex items-center gap-2 transition"
            @click="handleLogout"
          >
            <LogOut class="w-3.5 h-3.5" />
            <span>Đăng xuất</span>
          </button>
        </div>
      </div>
    </div>
  </header>
</template>

