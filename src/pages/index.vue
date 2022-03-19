<script setup lang="ts">
const name = ref('')

const router = useRouter()
const go = () => {
  if (name.value) router.push(`/progress/${encodeURIComponent(name.value)}`)
}
</script>

<template>
  <div>
    <p>
      <a
        rel="noreferrer"
        href="https://github.com/harshil1712/oss-commit"
        target="_blank"
      >OSS Commit</a>
    </p>
    <p>
      <em text-sm op75>Track your open-source contributions</em>
    </p>

    <div py-4 />

    <form @submit.prevent="go">
      <input
        id="input"
        v-model="name"
        placeholder="What's your name?"
        type="text"
        autocomplete="false"
        p="x-4 y-2"
        w="250px"
        text="center"
        bg="transparent"
        border="~ rounded gray-200 dark:gray-700"
        outline="none active:none"
      />

      <div>
        <button class="m-3 text-sm btn" :disabled="!name" type="submit">Go</button>
      </div>
    </form>
  </div>
</template>
