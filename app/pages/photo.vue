<template>
  <div>
    <client-only>
      TEST
    </client-only>
  </div>
</template>

<script setup lang="ts">
  const route = useRoute()
  const router = useRouter()
  onMounted(() => {
    if (route.query.frame) {
      console.log(route.query.frame)
    } else {
      router.push('/frame-selection')
    }
  })
</script>