<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { supabase } from '../../api/supabase'

const props = defineProps<{ name: string }>()
const GITHUB_TOKEN = import.meta.env.VITE_GITHUB_TOKEN

const commitGoals = ref(0)
const prGoals = ref(0)
const issueGoals = ref(0)
const ghData = ref({})
const avatarUrl = ref('')
const bio = ref('')
const currentCommitCount = ref(0)
const currentIssueCount = ref(0)
const currentPrCount = ref(0)
const commitPercent = ref(0)
const prPercent = ref(0)
const issuePercent = ref(0)

async function getUser() {
  try {
    const { data } = await supabase.from('commits_goal').select('*').eq('gh_username', props.name).single()
    const { data: ghData } = await (await fetch('https://api.github.com/graphql', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `token ${GITHUB_TOKEN}`,
      },
      body: JSON.stringify({
        query: `
        query {
          user(login: "${props.name}") {
            avatarUrl
            bio
            contributionsCollection(from: "2022-01-01T00:00:00.000Z") {
              totalCommitContributions
              totalIssueContributions
              totalPullRequestContributions
            }
          }
        }
      `,
      }),
    },
    )).json()
    if (data && ghData) {
      commitGoals.value = data.commit_goals
      prGoals.value = data.pr_goals
      issueGoals.value = data.issue_goals
      avatarUrl.value = ghData.user.avatarUrl
      bio.value = ghData.user.bio
      currentCommitCount.value = ghData.user.contributionsCollection.totalCommitContributions
      currentIssueCount.value = ghData.user.contributionsCollection.totalIssueContributions
      currentPrCount.value = ghData.user.contributionsCollection.totalPullRequestContributions
      commitPercent.value = Math.round((currentCommitCount.value / commitGoals.value) * 100)
      prPercent.value = Math.round((currentPrCount.value / prGoals.value) * 100)
      issuePercent.value = Math.round((currentIssueCount.value / issueGoals.value) * 100)
    }
    else {
      // TO Do: Display Register option
    }
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
    <!-- Header Content -->
    <section flex>
      <img
        :src="`${avatarUrl}`"
        :alt="`GitHub Avatar of ${props.name}`"
        w-32
        h-32
        rounded-full
        mb-6
      />
      <div ml-4>
        <h1 text-4xl>{{ props.name }}</h1>
        <p text-md op50 mt-2 w-64>
          <em>{{ bio }}</em>
        </p>
      </div>
    </section>
    <!-- Progress -->
    <section>
      <h2 text-3xl text-left>
        Progress
        <span aria-label="rocket emoji">🚀</span>
      </h2>
    </section>
    <div mt-4>
      <div flex justify-between m-auto>
        <div text-center py-8 rounded border-2 border-white flex-auto h-80 flex flex-col>
          <p flex-auto my-auto>
            <span text-6xl>{{ commitPercent }}</span>%
          </p>
          <div>
            <p>{{ currentCommitCount }} / {{ commitGoals }}</p>
            <p>Commits</p>
          </div>
        </div>
        <div text-center py-8 rounded border-2 border-white flex-auto h-80 flex flex-col>
          <p flex-auto>
            <span text-6xl>{{ prPercent }}</span>%
          </p>
          <div>
            <p>{{ currentPrCount }} / {{ prGoals }}</p>
            <p>PR</p>
          </div>
        </div>
        <div text-center py-8 rounded border-2 border-white flex-auto h-80 flex flex-col>
          <p flex-auto>
            <span text-6xl>{{ issuePercent }}</span>%
          </p>
          <div>
            <p>{{ currentIssueCount }} / {{ issueGoals }}</p>
            <p>Issues</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
