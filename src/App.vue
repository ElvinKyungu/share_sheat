<script setup lang="ts">
import { Download, Check } from 'lucide-vue-next'
import { animate } from 'motion'
import { ref, nextTick, useTemplateRef } from 'vue'
import elvin from '@/assets/avatar.jpg'
import deb from '@/assets/deb.jpg'
import gab from '@/assets/gab.jpg'
import marden from '@/assets/marden.jpg'

const isExpanded = ref(false)
const selectedUser = ref<{ name: string; avatar: string } | null>(null)
// Variable pour afficher l’icône Check
const showCheck = ref(false)

const containerList = useTemplateRef('containerList')
const downloadIcon = useTemplateRef('downloadIcon')
const selectedAvatar = useTemplateRef('selectedAvatar')
const borderCircle = useTemplateRef('borderCircle')
const checkIcon = useTemplateRef('checkIcon')

const toggleExpand = async () => {
  if (!isExpanded.value) {
    isExpanded.value = true
    await nextTick()
    if (containerList.value) {
      animate(containerList.value, {
        height: 'auto',
        opacity: [0, 1],
        scale: [0.8, 1]
      }, {
        duration: 1,
        type: 'spring'
      })
    } else {
      animate(containerList.value, {
        height: '0px',
        opacity: [1, 0],
        scale: [1, 0.9]
      }, {
        duration: 0.4,
        type: 'spring'
      })
    }
  }
}

const selectUser = async (user: { name: string; avatar: string }) => {
  if (containerList.value) {
    await animate(containerList.value, {
      height: 0,
      opacity: [1, 0],
      scale: [1, 0.8]
    }, {
      duration: 0.7,
      type: 'spring'
    })
  }

  if (downloadIcon.value) {
    await animate(downloadIcon.value, {
      y: [0, -60],
      opacity: [1, 0],
      scale: [1, 0.8]
    }, {
      duration: 0.4
    })
  }

  await nextTick()
  isExpanded.value = false
  selectedUser.value = user

  if (selectedAvatar.value) {
    await animate(selectedAvatar.value, {
      y: [40, 0],
      opacity: [0, 1],
      scale: 1
    }, {
      duration: 0.5
    })
  }

  // Animation du contour via le cercle SVG
  if (borderCircle.value) {
    // Définir une circonférence. Ici, pour un cercle de rayon 55 dans un SVG 120x120,
    // la circonférence est approximativement 2 * PI * 55 ≈ 345.
    const circumference = 345
    borderCircle.value.style.strokeDasharray = circumference
    borderCircle.value.style.strokeDashoffset = circumference

    await animate(borderCircle.value, {
      strokeDashoffset: 0
    }, {
      duration: 1,
      easing: 'ease-in-out'
    })
  }

  // Une fois le bord rempli, afficher l’icône Check
  showCheck.value = true
  if (checkIcon.value) {
    await animate(checkIcon.value, {
      opacity: [0, 1],
      scale: [0.5, 1]
    }, {
      duration: 0.5
    })
  }

 await new Promise(resolve => setTimeout(resolve, 1000))
  if (selectedAvatar.value && checkIcon.value) {
    await animate([selectedAvatar.value, checkIcon.value], {
      y: [0, 100],
      opacity: [1, 0]
    }, {
      duration: 0.5
    })
  }

  // Réinitialisation de l’état
  selectedUser.value = null
  showCheck.value = false

  if (downloadIcon.value) {
    await animate(downloadIcon.value, {
      y: [-60, 0],
      opacity: [0, 1],
      scale: [0.8, 1]
    }, {
      duration: 0.4
    })
  }
}

const users = [
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
        <!-- Contour SVG -->
        <svg class="absolute top-0 left-0 w-full h-full" viewBox="0 0 120 120">
          <circle
            ref="borderCircle"
            cx="60"
            cy="60"
            r="55"
            fill="none"
            stroke="#4caf50"
            stroke-width="5"
          />
        </svg>
        <!-- Icône Check -->
        <div
          v-if="showCheck"
          ref="checkIcon"
          class="absolute inset-0 flex items-center justify-center"
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
      >
        <li
          v-for="user in users"
          :key="user.name"
          @click="selectUser(user)"
          class="flex gap-3 items-center cursor-pointer hover:bg-white/50 p-2 rounded-lg transition-colors"
        >
          <div>
            <img
              :src="user.avatar"
              class="w-8 h-8 rounded-full"
              :alt="user.name"
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
