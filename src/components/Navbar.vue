<template>
  <nav class="navbar fixed w-full flex items-center justify-between px-8 py-4 bg-[#8f63d6] shadow">
    <div class="text-white font-bold text-xl">
      {{ $t('app.title') }}
    </div>
    <ul class="flex gap-8">
      <li><a class="text-white font-semibold text-lg hover:text-[#d1c4e9] transition-colors" href="#section1">Section 1</a></li>
      <li><a class="text-white font-semibold text-lg hover:text-[#d1c4e9] transition-colors" href="#section2">Section 2</a></li>
      <li><a class="text-white font-semibold text-lg hover:text-[#d1c4e9] transition-colors" href="#section3">Section 3</a></li>
    </ul>
  </nav>
</template>

<script setup>
// Navbar logic (if needed)
</script>
