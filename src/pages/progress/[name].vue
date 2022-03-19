<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { supabase } from '../../api/supabase'

const props = defineProps<{ name: string }>()
const GITHUB_TOKEN = import.meta.env.VITE_GITHUB_TOKEN

const userData = ref({
  commitGoals: 0,
  prGoals: 0,
  issueGoals: 0,
  avatarUrl: '',
  bio: '',
  currentCommitCount: 0,
  currentPrCount: 0,
  currentIssueCount: 0,
  commitPercent: 0,
  prPercent: 0,
  issuePercent: 0,
})

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
      userData.value.commitGoals = data.commit_goals
      userData.value.prGoals = data.pr_goals
      userData.value.issueGoals = data.issue_goals
      userData.value.avatarUrl = ghData.user.avatarUrl
      userData.value.bio = ghData.user.bio
      userData.value.currentCommitCount = ghData.user.contributionsCollection.totalCommitContributions
      userData.value.currentIssueCount = ghData.user.contributionsCollection.totalIssueContributions
      userData.value.commitPercent = Math.round((userData.value.currentCommitCount / userData.value.commitGoals) * 100)
      userData.value.currentPrCount = ghData.user.contributionsCollection.totalPullRequestContributions
      userData.value.prPercent = Math.round((userData.value.currentPrCount / userData.value.prGoals) * 100)
      userData.value.issuePercent = Math.round((userData.value.currentIssueCount / userData.value.issueGoals) * 100)
    }
    else {
      // TO Do: Display Register optionuser
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
        :src="`${userData.avatarUrl}`"
        :alt="`GitHub Avatar of ${props.name}`"
        w-32
        h-32
        rounded-full
        mb-6
      />
      <div ml-4>
        <h1 text-4xl>{{ props.name }}</h1>
        <p text-md op50 mt-2 w-64>
          <em>{{ userData.bio }}</em>
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
            <span text-6xl>{{ userData.commitPercent }}</span>%
          </p>
          <div>
            <p>{{ userData.currentCommitCount }} / {{ userData.commitGoals }}</p>
            <p>Commits</p>
          </div>
        </div>
        <div text-center py-8 rounded border-2 border-white flex-auto h-80 flex flex-col>
          <p flex-auto>
            <span text-6xl>{{ userData.prPercent }}</span>%
          </p>
          <div>
            <p>{{ userData.currentPrCount }} / {{ userData.prGoals }}</p>
            <p>PR</p>
          </div>
        </div>
        <div text-center py-8 rounded border-2 border-white flex-auto h-80 flex flex-col>
          <p flex-auto>
            <span text-6xl>{{ userData.issuePercent }}</span>%
          </p>
          <div>
            <p>{{ userData.currentIssueCount }} / {{ userData.issueGoals }}</p>
            <p>Issues</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
