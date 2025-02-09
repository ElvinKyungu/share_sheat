<script setup lang="ts">
import { Download, Check } from 'lucide-vue-next'
import { animate } from 'motion'
import { ref, nextTick, useTemplateRef } from 'vue'
import elvin from '@/assets/avatar.jpg'
import deb from '@/assets/deb.jpg'
import gab from '@/assets/gab.jpg'
import marden from '@/assets/marden.jpg'

// Define interfaces for better type safety
interface User {
  name: string
  avatar: string
}

const isExpanded = ref(false)
const selectedUser = ref<User | null>(null)
const showCheck = ref(false)

// Group refs for better organization
const refs = {
  containerList: useTemplateRef('containerList'),
  downloadIcon: useTemplateRef('downloadIcon'),
  selectedAvatar: useTemplateRef('selectedAvatar'),
  borderCircle: useTemplateRef('borderCircle'),
  checkIcon: useTemplateRef('checkIcon')
}

// Animation configurations for reusability
const ANIMATION_CONFIG = {
  spring: { type: 'spring' as const },
  fade: { duration: 0.4 },
  circle: {
    radius: 55,
    circumference: Math.PI * 2 * 55
  }
}

const toggleExpand = async () => {
  if (!isExpanded.value) {
    isExpanded.value = true
    await nextTick()

    if (refs.containerList.value) {
      await animate(refs.containerList.value, {
        height: 'auto',
        opacity: [0, 1],
        scale: [0.8, 1]
      }, {
        duration: 1,
        ...ANIMATION_CONFIG.spring
      })
    }
  } else {
    if (refs.containerList.value) {
      await animate(refs.containerList.value, {
        height: '0px',
        opacity: [1, 0],
        scale: [1, 0.9]
      }, {
        duration: 0.4,
        ...ANIMATION_CONFIG.spring
      })
      isExpanded.value = false
    }
  }
}

const selectUser = async (user: User) => {
  // Close list animation
  if (refs.containerList.value) {
    await animate(refs.containerList.value, {
      height: 0,
      opacity: [1, 0],
      scale: [1, 0.8]
    }, {
      duration: 0.7,
      ...ANIMATION_CONFIG.spring
    })
  }

  // Hide download icon
  if (refs.downloadIcon.value) {
    await animate(refs.downloadIcon.value, {
      y: [0, -60],
      opacity: [1, 0],
      scale: [1, 0.8]
    }, ANIMATION_CONFIG.fade)
  }

  await nextTick()
  isExpanded.value = false
  selectedUser.value = user

  // Show selected avatar
  if (refs.selectedAvatar.value) {
    await animate(refs.selectedAvatar.value, {
      y: [40, 0],
      opacity: [0, 1],
      scale: 1
    }, {
      duration: 0.5
    })
  }

  // Animate circle border
  if (refs.borderCircle.value) {
    const { circumference } = ANIMATION_CONFIG.circle
    const element = refs.borderCircle.value as SVGCircleElement
    element.style.strokeDasharray = `${circumference}`
    element.style.strokeDashoffset = `${circumference}`

    await animate(element, {
      strokeDashoffset: 0
    }, {
      duration: 1
    })
  }

  // Show check icon
  showCheck.value = true
  if (refs.checkIcon.value) {
    await animate(refs.checkIcon.value, {
      opacity: [0, 1],
      scale: [0.5, 1]
    }, {
      duration: 0.5
    })
  }

  // Wait and hide everything
  await new Promise(resolve => setTimeout(resolve, 1000))
  if (refs.selectedAvatar.value && refs.checkIcon.value) {
    await animate([refs.selectedAvatar.value, refs.checkIcon.value], {
      y: [0, 100],
      opacity: [1, 0]
    }, {
      duration: 0.5
    })
  }

  // Reset state
  selectedUser.value = null
  showCheck.value = false

  // Show download icon again
  if (refs.downloadIcon.value) {
    await animate(refs.downloadIcon.value, {
      y: [-60, 0],
      opacity: [0, 1],
      scale: [0.8, 1]
    }, ANIMATION_CONFIG.fade)
  }
}

const users: User[] = [
  { name: 'Elvin Code', avatar: elvin },
  { name: 'Gabriel Delattre', avatar: gab },
  { name: 'Deborah Yambenu', avatar: deb },
  { name: 'Marden Mbiya', avatar: marden }
]
</script>

<template>
  <main class="flex flex-col h-screen w-full relative justify-center items-center">
    <div
      @click="toggleExpand"
      class="flex items-center justify-center p-3 h-14 w-14 rounded-full bg-black text-white cursor-pointer hover:bg-gray-800 transition-colors relative overflow-hidden"
    >
      <Download ref="downloadIcon" v-if="!selectedUser" />
      <div v-if="selectedUser" class="relative">
        <img
          ref="selectedAvatar"
          :src="selectedUser.avatar"
          class="w-[70%] h-[70%] rounded-full object-cover"
          :alt="selectedUser.name"
        />
        <svg
          class="absolute top-0 left-0 w-full h-full"
          viewBox="0 0 120 120"
          aria-hidden="true"
        >
          <circle
            ref="borderCircle"
            cx="60"
            cy="60"
            :r="ANIMATION_CONFIG.circle.radius"
            fill="none"
            stroke="#4caf50"
            stroke-width="5"
          />
        </svg>
        <div
          v-if="showCheck"
          ref="checkIcon"
          class="absolute inset-0 flex items-center justify-center"
          aria-label="Selection confirmed"
        >
          <Check class="w-6 h-6 text-green-500" />
        </div>
      </div>
    </div>
    <div>
      <ul
        v-if="isExpanded"
        ref="containerList"
        class="user-list p-2 bg-gray-100 space-y-2 rounded-2xl absolute left-1/2 -translate-x-1/2 top-1/2 -translate-y-1/2 min-w-[17rem] shadow-lg"
        role="listbox"
      >
        <li
          v-for="user in users"
          :key="user.name"
          @click="selectUser(user)"
          class="flex gap-3 items-center cursor-pointer hover:bg-white/50 p-2 rounded-lg transition-colors"
          role="option"
        >
          <div>
            <img
              :src="user.avatar"
              class="w-8 h-8 rounded-full"
              :alt="`Avatar of ${user.name}`"
            />
          </div>
          <div>
            <p class="text-lg">{{ user.name }}</p>
          </div>
        </li>
      </ul>
    </div>
  </main>
</template>

<style scoped>
.user-list {
  transform-origin: top;
}
</style>
