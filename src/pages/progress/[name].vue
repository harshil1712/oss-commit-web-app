<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { supabase } from '../../api/supabase'

const props = defineProps<{ name: string }>()
const commit_goals = ref('')
const pr_goals = ref('')
const issue_goals = ref('')

async function getUser() {
  try {
    const { data } = await supabase.from('commits_goal').select('*').eq('gh_username', props.name).single()
    if (data)
      commit_goals.value = data.commit_goals
    pr_goals.value = data.pr_goals
    issue_goals.value = data.issue_goals
  }
  catch (error) {
    console.log(error)
  }
}

onMounted(() => {
  getUser()
})

</script>

<template>
  <div>
    <div i-carbon-pedestrian text-4xl inline-block />
    <p>Hi, {{ props.name }}</p>
    <p text-sm op50>
      <em>Bio from GitHub</em>
    </p>
    <div>
      <h1>Goals</h1>
      <div flex justify-between w-72 m-auto>
        <div>
          <p>{{ commit_goals }}</p>
          <p>Commits</p>
        </div>
        <div>
          <p>{{ pr_goals }}</p>
          <p>PR</p>
        </div>
        <div>
          <p>{{ issue_goals }}</p>
          <p>Issues</p>
        </div>
      </div>
    </div>
  </div>
</template>
