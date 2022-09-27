<script lang="ts" setup>
import { InitApp } from '~/utils/initApp'
InitApp()
const locale = useState<string>('locale.i18n')
</script>

<template>
  <Html :lang="locale">
    <Body
      class="bg-white text-gray-800 antialiased transition-colors duration-300"
    >
      <NuxtLayout>
        <NuxtPage />
      </NuxtLayout>
      <div class="scrollbar">
        <div class="force-overflow" />
      </div>
    </Body>
  </Html>
</template>

<style lang="postcss">
body {
  @apply bg-gray-50;
}

::-webkit-scrollbar {
  width: 5px;
}

::-webkit-scrollbar-track {
  background-color: #ebebeb;
  -webkit-border-radius: 10px;
  border-radius: 10px;
}

::-webkit-scrollbar-thumb {
  -webkit-border-radius: 10px;
  border-radius: 10px;
  background: #6d6d6d;
}

.global-text {
  @apply text-gray-900;
}
</style>
