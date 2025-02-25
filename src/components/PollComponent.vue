<script setup>
import { ref, computed, onMounted } from 'vue'
import { CheckIcon, Users } from 'lucide-vue-next'

// Question reference for customization
const question = ref('What is your favorite programming language?')

// Unique ID for this poll to identify it in localStorage
const POLL_ID = 'favorite-programming-language-poll'
const USER_VOTE_KEY = `${POLL_ID}-user-vote`

const options = ref([
  {
    id: 'js',
    text: 'JavaScript',
    votes: 0,
    percentage: 0,
    checked: false,
    displayVotes: 0,
    displayPercentage: 0,
  },
  {
    id: 'py',
    text: 'Python',
    votes: 0,
    percentage: 0,
    checked: false,
    displayVotes: 0,
    displayPercentage: 0,
  },
  {
    id: 'java',
    text: 'Java',
    votes: 0,
    percentage: 0,
    checked: false,
    displayVotes: 0,
    displayPercentage: 0,
  },
  {
    id: 'cpp',
    text: 'C++',
    votes: 0,
    percentage: 0,
    checked: false,
    displayVotes: 0,
    displayPercentage: 0,
  },
])

const hasVoted = ref(false)
const isNewVisitor = ref(true)
const animatingResults = ref(false)
const error = ref(null)
let animationInterval = null

// Computed property to find the leading option(s)
const leadingOptionIds = computed(() => {
  if (!hasVoted.value) return []

  // Find the maximum vote count
  const maxVotes = Math.max(...options.value.map((option) => option.votes))

  // If no votes yet, return empty array
  if (maxVotes <= 0) return []

  // Return IDs of all options with the maximum vote count (could be a tie)
  return options.value.filter((option) => option.votes === maxVotes).map((option) => option.id)
})

// Generate random votes for initial/refresh scenario
const generateRandomVotes = () => {
  try {
    // Only generate between 10-100 votes per option for realistic data
    return options.value.map((option) => {
      return {
        ...option,
        votes: Math.floor(Math.random() * 90) + 10,
        displayVotes: 0,
        displayPercentage: 0,
      }
    })
  } catch (err) {
    handleError('Error generating random votes', err)
    return options.value
  }
}

// Load poll data from localStorage or initialize it
const loadPollData = () => {
  try {
    const storedData = localStorage.getItem(POLL_ID)

    if (storedData) {
      // We have existing poll data
      const parsedData = JSON.parse(storedData)

      // Validate the stored data structure
      if (
        !Array.isArray(parsedData) ||
        !parsedData.every((item) => item.id && typeof item.votes === 'number')
      ) {
        throw new Error('Invalid data format in localStorage')
      }

      options.value = parsedData.map((option) => ({
        ...option,
        displayVotes: hasVoted.value ? option.votes : 0,
        displayPercentage: hasVoted.value ? option.percentage : 0,
      }))
      isNewVisitor.value = false
    } else {
      // First time or cleared localStorage, generate random votes
      options.value = generateRandomVotes()
      savePollData()
      isNewVisitor.value = true
    }

    // Check if user has already voted
    const userVote = localStorage.getItem(USER_VOTE_KEY)
    if (userVote) {
      const votedOptionId = userVote
      const votedOption = options.value.find((opt) => opt.id === votedOptionId)
      if (votedOption) {
        votedOption.checked = true
        hasVoted.value = true

        // If user has already voted, animate the numbers immediately
        // Use setTimeout to ensure UI is rendered first
        setTimeout(() => {
          animateNumbers()
        }, 0)
      }
    }

    // Update percentages based on loaded data
    updatePercentages()
  } catch (err) {
    handleError('Failed to load poll data', err)
    resetToDefault()
  }
}

// Reset to default state if something goes wrong
const resetToDefault = () => {
  options.value = [
    {
      id: 'js',
      text: 'JavaScript',
      votes: 25,
      percentage: 0,
      checked: false,
      displayVotes: 0,
      displayPercentage: 0,
    },
    {
      id: 'py',
      text: 'Python',
      votes: 30,
      percentage: 0,
      checked: false,
      displayVotes: 0,
      displayPercentage: 0,
    },
    {
      id: 'java',
      text: 'Java',
      votes: 15,
      percentage: 0,
      checked: false,
      displayVotes: 0,
      displayPercentage: 0,
    },
    {
      id: 'cpp',
      text: 'C++',
      votes: 20,
      percentage: 0,
      checked: false,
      displayVotes: 0,
      displayPercentage: 0,
    },
  ]
  updatePercentages()
  hasVoted.value = false
  error.value = null
}

// Error handling function
const handleError = (message, err) => {
  console.error(message, err)
  error.value = `${message}: ${err.message}`

  // Auto-clear error after 5 seconds
  setTimeout(() => {
    error.value = null
  }, 5000)
}

// Save poll data to localStorage
const savePollData = () => {
  try {
    localStorage.setItem(POLL_ID, JSON.stringify(options.value))
  } catch (err) {
    handleError('Could not save poll data', err)
  }
}

// Save user's vote
const saveUserVote = (optionId) => {
  try {
    localStorage.setItem(USER_VOTE_KEY, optionId)
  } catch (err) {
    handleError('Could not save your vote', err)
  }
}

const totalVotes = computed(() => {
  return options.value.reduce((sum, option) => sum + option.votes, 0)
})

// Clean up any existing animation
const cleanupAnimation = () => {
  if (animationInterval) {
    clearInterval(animationInterval)
    animationInterval = null
  }
}

// Animation function for counting up numbers
const animateNumbers = () => {
  if (animatingResults.value) return
  animatingResults.value = true

  // Clean up any existing animation
  cleanupAnimation()

  // Set initial display values to 0
  options.value.forEach((option) => {
    option.displayVotes = 0
    option.displayPercentage = 0
  })

  // Determine animation duration
  const duration = 1200 // 1.2 seconds total
  const steps = 50 // Number of steps
  const interval = duration / steps

  let currentStep = 0

  try {
    animationInterval = setInterval(() => {
      currentStep++
      const progress = currentStep / steps

      // Update each option's display values based on progress
      options.value.forEach((option) => {
        option.displayVotes = Math.round(option.votes * progress)
        option.displayPercentage = parseFloat((option.percentage * progress).toFixed(1))
      })

      // Stop animation when complete
      if (currentStep >= steps) {
        options.value.forEach((option) => {
          option.displayVotes = option.votes
          option.displayPercentage = option.percentage
        })
        cleanupAnimation()
        animatingResults.value = false
      }
    }, interval)
  } catch (err) {
    handleError('Animation error', err)
    animatingResults.value = false

    // Set final values directly in case of error
    options.value.forEach((option) => {
      option.displayVotes = option.votes
      option.displayPercentage = option.percentage
    })
  }
}

const vote = (index) => {
  if (!hasVoted.value) {
    try {
      if (index < 0 || index >= options.value.length) {
        throw new Error('Invalid option selected')
      }

      options.value[index].votes++
      options.value[index].checked = true
      hasVoted.value = true

      // Save the updated poll data
      updatePercentages()
      savePollData()

      // Record that this user has voted
      saveUserVote(options.value[index].id)

      // Animate the results
      animateNumbers()
    } catch (err) {
      handleError('Failed to register vote', err)
    }
  }
}

const updatePercentages = () => {
  try {
    const total = totalVotes.value
    if (total <= 0) {
      options.value.forEach((option) => {
        option.percentage = 0
      })
      return
    }

    options.value.forEach((option) => {
      option.percentage = (option.votes / total) * 100
    })
  } catch (err) {
    handleError('Failed to update percentages', err)
  }
}

const resetPoll = () => {
  try {
    // Clean up any existing animation
    cleanupAnimation()

    // Only reset the user's vote, not the overall poll data
    options.value.forEach((option) => {
      option.checked = false
      option.displayVotes = 0
      option.displayPercentage = 0
    })
    hasVoted.value = false
    error.value = null
    localStorage.removeItem(USER_VOTE_KEY)

    // Regenerate random votes for demo purposes
    options.value = generateRandomVotes()
    updatePercentages()
    savePollData()
  } catch (err) {
    handleError('Failed to reset poll', err)
  }
}

// Clean up resources when component is unmounted
const cleanup = () => {
  cleanupAnimation()
}

// Initialize on component mount
onMounted(() => {
  loadPollData()

  // Clean up when component is unmounted
  return cleanup
})
</script>

<template>
  <div class="max-w-lg mx-auto p-6 bg-gray-100 border-[1px] rounded-lg shadow-md">
    <!-- Error notification -->
    <div
      v-if="error"
      class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded mb-4 flex justify-between items-center"
    >
      <span>{{ error }}</span>
      <button @click="error = null" class="text-red-700">&times;</button>
    </div>

    <h2 class="text-2xl font-bold mb-4 leading-8 tracking-wide text-gray-800">
      {{ question }}
    </h2>
    <p class="text-sm font-semibold text-gray-600 text-left mb-4 flex items-center">
      <Users class="w-4 h-4 mr-1" />
      {{ totalVotes }} {{ totalVotes === 1 ? 'vote' : 'votes' }}
      <span v-if="hasVoted" class="ml-2 text-emerald-600">(You voted)</span>
    </p>

    <ul class="space-y-4">
      <li
        v-for="(option, index) in options"
        :key="option.id"
        class="group border p-2 rounded-md"
        :class="{ 'cursor-pointer hover:shadow-md': !hasVoted, 'opacity-75': animatingResults }"
        @click="!hasVoted && vote(index)"
      >
        <div class="mb-1 flex justify-between items-start">
          <div class="flex items-center">
            <!-- Only show check indicator after voting -->
            <div
              v-if="hasVoted"
              :class="[
                'w-5 h-5 rounded-full flex items-center justify-center mr-2',
                option.checked
                  ? 'bg-emerald-600 text-white'
                  : 'bg-transparent border border-gray-500',
              ]"
            >
              <CheckIcon v-if="option.checked" class="w-3 h-3" />
            </div>
            <span class="font-medium mr-1">{{ option.text }}</span>
            <!-- Trophy - only shown for options with the most votes (could be multiple in case of ties) -->
            <img
              v-if="hasVoted && leadingOptionIds.includes(option.id)"
              src="/trophy.svg"
              alt="Most voted"
              class="w-4 h-4 transition-opacity duration-700"
              :class="{ 'opacity-0': animatingResults, 'opacity-100': !animatingResults }"
            />
          </div>

          <!-- Only show animated counts after voting -->
          <div v-if="hasVoted" class="flex flex-col items-end space-y-1">
            <div class="flex items-center text-gray-500 space-x-1">
              <span class="text-xs min-w-[20px] text-right">{{ option.displayVotes }}</span>
              <Users class="w-3" />
            </div>
            <span class="text-gray-700 text-sm font-medium min-w-[40px] text-right">
              {{ option.displayPercentage.toFixed(1) }}%
            </span>
          </div>
        </div>

        <!-- The bar element - initially empty, fills after voting -->
        <div
          class="h-2 w-full bg-gray-200 rounded-md overflow-hidden"
          :class="{ 'cursor-pointer': !hasVoted }"
        >
          <div
            class="h-full transition-all duration-1200 ease-out"
            :style="{ width: hasVoted ? `${option.displayPercentage}%` : '0%' }"
            :class="{
              'bg-emerald-500': option.checked,
              'bg-gray-300': !option.checked && hasVoted,
            }"
          ></div>
        </div>
      </li>
    </ul>

    <div class="flex flex-row items-center mt-6">
      <button
        v-if="hasVoted"
        @click="resetPoll"
        class="inline-flex flex-shrink-0 flex-wrap items-center justify-center border-[1px] cursor-pointer p-2 bg-[#121212] rounded-md text-sm text-gray-100 hover:bg-gray-600 mx-auto disabled:opacity-50 disabled:cursor-not-allowed"
        :disabled="animatingResults"
      >
        Reset Poll
      </button>
    </div>
  </div>
</template>
