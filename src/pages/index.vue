<script setup lang="ts">
import championJson from '~/database/champion.json'
import earlyJson from '~/database/early.json'
type Champion = typeof championJson[0]

const champions = championJson.sort((a, b) => {
  return a.cost - b.cost
})
const cost1Champions = champions.filter(champion => champion.cost === 1)
const cost2Champions = champions.filter(champion => champion.cost === 2)
const cost3Champions = champions.filter(champion => champion.cost === 3)
const cost4Champions = champions.filter(champion => champion.cost === 4)
const cost5Champions = champions.filter(champion => champion.cost === 5)

const filteredChampions = [[...cost1Champions], [...cost2Champions], [...cost3Champions], [...cost4Champions], [...cost5Champions]]
const currentLineUp = $ref<Champion[]>([])
const analyzedLineups = $computed(() => findSuitableLineups(currentLineUp, earlyJson))

// 寻找最合适的阵容
function findSuitableLineups(currentLineup: Champion[], earlyLineup: Record<string, Champion[]>) {
  const allLineups = Object.keys(earlyLineup).map(key => earlyLineup[key])
  const occupyCounts = allLineups.map((earlyLineup) => {
    return getOccupyCount(currentLineup, earlyLineup)
  })
  const suitableSortedLineUps = occupyCounts.map((occupyCount, index) => {
    return {
      occupyCount,
      lineUp: { name: Object.keys(earlyLineup)[index], value: earlyLineup[Object.keys(earlyLineup)[index]] },
    }
  }).sort((a, b) => {
    return b.occupyCount - a.occupyCount
  })
  return suitableSortedLineUps
}

// 当前阵容在某个阵容中的棋子占用数量
function getOccupyCount(currentLineup: Champion[], earlyLineup: Champion[]) {
  return earlyLineup.filter(hero => currentLineup.some(v => v.id === hero.id)).length
}

function onClickChampion(champion: Champion) {
  if (currentLineUp.includes(champion))
    currentLineUp.splice(currentLineUp.indexOf(champion), 1)
  else
    currentLineUp.push(champion)
}

function getImageSrc(id: string) {
  const url = new URL(`../assets/images/champions/${id}.png`, import.meta.url).href
  return url
}

function isSelected(id: string) {
  return currentLineUp.some(v => v.id === id)
}
</script>
<template>
  <div w="90%" flex="~" flex-col>
    <div v-for="(everyCostChampions, i) in filteredChampions" :key="i" flex="~" flex-wrap w-full mb-16px>
      <div
        v-for="champion in everyCostChampions" :key="champion.id" w-100px h-100px border border-1 border-gray-500
        relative cursor-pointer class="group" @click="onClickChampion(champion)"
      >
        <img :src="getImageSrc(champion.id)" alt="" w-full h-full group-hover:opacity-80 transition>
        <div
          v-if="isSelected(champion.id)" absolute top-0 left-0 w-full h-full bg="gray-500/80" flex="~" items-center
          justify-center
        >
          <div i-ic-baseline-check text-white text-2xl />
        </div>
      </div>
    </div>
  </div>

  <div v-for="lineup in analyzedLineups" :key="lineup.lineUp.name">
    <div font-sans italic text-4xl>
      {{ lineup.lineUp.name }} {{ lineup.occupyCount }}
    </div>
    <div w-full flex="~">
      <div
        v-for="champion in lineup.lineUp.value" :key="champion.id" w-50px h-50px border border-1 border-gray-500
        cursor-pointer relative
      >
        <img :src="getImageSrc(champion.id)" alt="" w-full h-full>
        <div
          v-if="isSelected(champion.id)" absolute top-0 left-0 w-full h-full bg="green-300/50" flex="~" items-center
          justify-center
        >
          <div i-ic-baseline-check text-white text-2xl />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
</style>
