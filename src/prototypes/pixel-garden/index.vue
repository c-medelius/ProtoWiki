<script setup lang="ts">
import { computed, ref, watch } from 'vue'
import { CdxButton, CdxSelect, CdxTextInput } from '@wikimedia/codex'

import ChromeWrapper from '@/components/ChromeWrapper.vue'

definePage({
  meta: {
    title: 'Pixel garden growth',
    description: 'A playful prototype where article-category edits grow a themed pixel garden.',
  },
})

type Category = 'history' | 'science' | 'arts' | 'geography'
type TimeOfDay = 'day' | 'night'
type Terrain = 'desert' | 'mountain'
type ArtsSubtype = 'music' | 'film' | 'literature' | 'visual-art' | 'theater'
type Subtype =
  | 'ancient'
  | 'medieval'
  | 'modern'
  | 'contemporary'
  | 'physics'
  | 'biology'
  | 'space'
  | 'computing'
  | ArtsSubtype
  | 'mountains'
  | 'oceans'
  | 'cities'
  | 'climate'

interface CategoryOption {
  value: Category
  label: string
}

interface OptionItem<T extends string> {
  value: T
  label: string
}

interface GrowthState {
  history: number
  science: number
  arts: number
  geography: number
}

interface GrowthLogItem {
  id: string
  category: Category
  subtype: Subtype
  note: string
  timestamp: string
}

interface FillerPlant {
  id: string
  className: string
  minEdits: number
}

interface OverflowPlant {
  id: string
  category: Category
  stage: number
  left: string
  bottom: string
  scale: number
}

interface OverflowSlot {
  left: string
  bottom: string
  scale: number
}

const categoryOptions: CategoryOption[] = [
  { value: 'history', label: 'History (oak tree)' },
  { value: 'science', label: 'Science (crystal bloom)' },
  { value: 'arts', label: 'Arts (flower patch)' },
  { value: 'geography', label: 'Geography (wild bush)' },
]

const subtypeOptionsByCategory: Record<Category, OptionItem<Subtype>[]> = {
  history: [
    { value: 'ancient', label: 'Ancient' },
    { value: 'medieval', label: 'Medieval' },
    { value: 'modern', label: 'Modern' },
    { value: 'contemporary', label: 'Contemporary' },
  ],
  science: [
    { value: 'physics', label: 'Physics' },
    { value: 'biology', label: 'Biology' },
    { value: 'space', label: 'Space' },
    { value: 'computing', label: 'Computing' },
  ],
  arts: [
    { value: 'music', label: 'Music' },
    { value: 'film', label: 'Film' },
    { value: 'literature', label: 'Literature' },
    { value: 'visual-art', label: 'Visual art' },
    { value: 'theater', label: 'Theater' },
  ],
  geography: [
    { value: 'mountains', label: 'Mountains' },
    { value: 'oceans', label: 'Oceans' },
    { value: 'cities', label: 'Cities' },
    { value: 'climate', label: 'Climate' },
  ],
}

const artsPaletteBySubtype: Record<ArtsSubtype, { petal: string; core: string; accent: string }> = {
  music: { petal: '#ff88c1', core: '#ffe17a', accent: '#ff5da5' },
  film: { petal: '#9ac5ff', core: '#fbe98e', accent: '#6f9bff' },
  literature: { petal: '#b899ff', core: '#ffe39c', accent: '#8d69dc' },
  'visual-art': { petal: '#ff9a70', core: '#ffe080', accent: '#ff6f54' },
  theater: { petal: '#ff6575', core: '#ffd58a', accent: '#d94f5d' },
}

const fillerPlants: FillerPlant[] = [
  { id: 'f1', className: 'pixel-garden__filler--1', minEdits: 2 },
  { id: 'f2', className: 'pixel-garden__filler--2', minEdits: 3 },
  { id: 'f3', className: 'pixel-garden__filler--3', minEdits: 4 },
  { id: 'f4', className: 'pixel-garden__filler--4', minEdits: 5 },
  { id: 'f5', className: 'pixel-garden__filler--5', minEdits: 7 },
  { id: 'f6', className: 'pixel-garden__filler--6', minEdits: 8 },
  { id: 'f7', className: 'pixel-garden__filler--7', minEdits: 10 },
  { id: 'f8', className: 'pixel-garden__filler--8', minEdits: 12 },
  { id: 'f9', className: 'pixel-garden__filler--9', minEdits: 14 },
  { id: 'f10', className: 'pixel-garden__filler--10', minEdits: 16 },
]

const timeOptions: OptionItem<TimeOfDay>[] = [
  { value: 'day', label: 'Day' },
  { value: 'night', label: 'Night' },
]

const terrainOptions: OptionItem<Terrain>[] = [
  { value: 'desert', label: 'Desert' },
  { value: 'mountain', label: 'Mountain' },
]

const selectedCategory = ref<Category>('history')
const selectedSubtype = ref<Subtype>('ancient')
const activeArtsSubtype = ref<ArtsSubtype>('music')
const timeOfDay = ref<TimeOfDay>('day')
const terrain = ref<Terrain>('desert')
const editNote = ref('')

const growth = ref<GrowthState>({
  history: 0,
  science: 0,
  arts: 0,
  geography: 0,
})

const growthLog = ref<GrowthLogItem[]>([])

const stageCap = 10
const maxOverflowPerCategory = 18

const stageScales = [0, 0.78, 0.92, 1.05, 1.17, 1.29, 1.4, 1.5, 1.6, 1.69, 1.78]

const overflowSlots: OverflowSlot[] = Array.from({ length: 72 }, (_, index) => {
  const columns = 12
  const column = index % columns
  const row = Math.floor(index / columns)
  const left = 4 + column * 8 + (row % 2 === 0 ? 0 : 2)
  const bottom = 12 + row * 2.8 + (column % 3 === 0 ? 0.6 : 0)
  const scale = 0.66 + row * 0.07

  return {
    left: `${Math.min(left, 96)}%`,
    bottom: `${Math.min(bottom, 29)}%`,
    scale,
  }
})

const currentSubtypeOptions = computed(() => subtypeOptionsByCategory[selectedCategory.value])

watch(selectedCategory, () => {
  selectedSubtype.value = currentSubtypeOptions.value[0].value
})

const currentSubtypeLabel = computed(() => {
  const match = currentSubtypeOptions.value.find((item) => item.value === selectedSubtype.value)
  return match?.label ?? 'General'
})

const totalEdits = computed(
  () => growth.value.history + growth.value.science + growth.value.arts + growth.value.geography,
)

const completedBeds = computed(() => {
  let count = 0
  if (growth.value.history >= stageCap) count += 1
  if (growth.value.science >= stageCap) count += 1
  if (growth.value.arts >= stageCap) count += 1
  if (growth.value.geography >= stageCap) count += 1
  return count
})

const sceneClasses = computed(() => [
  `pixel-garden__scene--${timeOfDay.value}`,
  `pixel-garden__scene--${terrain.value}`,
])

const visibleFillerPlants = computed(() =>
  fillerPlants.filter((plant) => totalEdits.value >= plant.minEdits),
)

const overflowPlants = computed<OverflowPlant[]>(() => {
  const categories: Category[] = ['history', 'science', 'arts', 'geography']
  const plants: OverflowPlant[] = []

  categories.forEach((category, categoryIndex) => {
    const value = growth.value[category]
    const additionalPlants = Math.min(Math.max(Math.ceil(value / stageCap) - 1, 0), maxOverflowPerCategory)

    for (let extraIndex = 0; extraIndex < additionalPlants; extraIndex += 1) {
      const slot = overflowSlots[categoryIndex + extraIndex * categories.length]
      if (!slot) continue

      const stage = Math.min(Math.max(value - (extraIndex + 1) * stageCap, 1), stageCap)

      plants.push({
        id: `${category}-overflow-${extraIndex}`,
        category,
        stage,
        left: slot.left,
        bottom: slot.bottom,
        scale: slot.scale,
      })
    }
  })

  return plants
})

const artsSpriteStyle = computed(() => {
  const colors = artsPaletteBySubtype[activeArtsSubtype.value]
  return {
    '--arts-petal': colors.petal,
    '--arts-core': colors.core,
    '--arts-accent': colors.accent,
  }
})

const gardenMood = computed(() => {
  if (totalEdits.value === 0) return 'Your plot is waiting for the first seed.'
  if (completedBeds.value === 4) return 'All four category beds are thriving. This article is flourishing.'
  if (totalEdits.value < 8) return 'Sprouts are appearing across the whole plot.'
  if (totalEdits.value < 16) return 'The garden is growing denser and more varied.'
  return 'Lush and layered. Every patch is contributing.'
})

function stageClass(value: number): string {
  const level = Math.min(Math.max(value, 0), stageCap)
  return `stage-${level}`
}

function isArtsSubtype(value: Subtype): value is ArtsSubtype {
  return value === 'music' || value === 'film' || value === 'literature' || value === 'visual-art' || value === 'theater'
}

function logEdit() {
  const category = selectedCategory.value
  const subtype = selectedSubtype.value

  growth.value[category] += 1

  if (category === 'arts' && isArtsSubtype(subtype)) {
    activeArtsSubtype.value = subtype
  }

  const note = editNote.value.trim().length > 0 ? editNote.value.trim() : 'Quick improvement pass'
  const categoryLabel = categoryOptions.find((item) => item.value === category)?.label ?? category
  const subtypeLabel = currentSubtypeLabel.value

  const entry: GrowthLogItem = {
    id: crypto.randomUUID(),
    category,
    subtype,
    note,
    timestamp: `${categoryLabel} - ${subtypeLabel} - ${new Date().toLocaleTimeString([], {
      hour: '2-digit',
      minute: '2-digit',
    })}`,
  }

  growthLog.value = [entry, ...growthLog.value].slice(0, 10)
  editNote.value = ''
}

function resetGarden() {
  growth.value = { history: 0, science: 0, arts: 0, geography: 0 }
  growthLog.value = []
  editNote.value = ''
  selectedCategory.value = 'history'
  selectedSubtype.value = 'ancient'
  activeArtsSubtype.value = 'music'
}

function overflowSpriteStyle(plant: OverflowPlant): Record<string, string> {
  const style: Record<string, string> = {
    '--sprite-left': plant.left,
    '--sprite-bottom': plant.bottom,
    '--sprite-scale': String((stageScales[plant.stage] ?? stageScales[stageCap]) * plant.scale),
  }

  if (plant.category === 'arts') {
    const colors = artsPaletteBySubtype[activeArtsSubtype.value]
    style['--arts-petal'] = colors.petal
    style['--arts-core'] = colors.core
    style['--arts-accent'] = colors.accent
  }

  return style
}
</script>

<template>
  <ChromeWrapper>
    <section class="pixel-garden" aria-labelledby="pixel-garden-title">
      <header class="pixel-garden__header">
        <p class="pixel-garden__eyebrow">Edit feedback concept</p>
        <h1 id="pixel-garden-title" class="pixel-garden__title">Pixel garden growth</h1>
        <p class="pixel-garden__subtitle">
          Article-category edits grow themed patches, and arts subtypes recolor the flower bed.
        </p>

        <div class="pixel-garden__style-controls" aria-label="Garden appearance controls">
          <div class="pixel-garden__style-item">
            <span class="pixel-garden__style-label">Time</span>
            <CdxSelect v-model:selected="timeOfDay" :menu-items="timeOptions" />
          </div>
          <div class="pixel-garden__style-item">
            <span class="pixel-garden__style-label">Terrain</span>
            <CdxSelect v-model:selected="terrain" :menu-items="terrainOptions" />
          </div>
        </div>
      </header>

      <div class="pixel-garden__layout">
        <section class="pixel-garden__panel pixel-garden__panel--visual" aria-label="Pixel garden preview">
          <div
            class="pixel-garden__scene"
            :class="sceneClasses"
            role="img"
            aria-label="A pixel garden that changes with your category edits"
          >
            <div class="pixel-garden__sky" />
            <div class="pixel-garden__stars" />
            <div class="pixel-garden__sun" />
            <div class="pixel-garden__moon" />
            <div class="pixel-garden__terrain pixel-garden__terrain--far" />
            <div class="pixel-garden__terrain pixel-garden__terrain--mid" />

            <div class="pixel-garden__bed pixel-garden__bed--history" :class="stageClass(growth.history)">
              <div class="pixel-garden__sprite pixel-garden__sprite--history" />
            </div>

            <div class="pixel-garden__bed pixel-garden__bed--science" :class="stageClass(growth.science)">
              <div class="pixel-garden__sprite pixel-garden__sprite--science" />
            </div>

            <div class="pixel-garden__bed pixel-garden__bed--arts" :class="stageClass(growth.arts)">
              <div class="pixel-garden__sprite pixel-garden__sprite--arts" :style="artsSpriteStyle" />
            </div>

            <div class="pixel-garden__bed pixel-garden__bed--geography" :class="stageClass(growth.geography)">
              <div class="pixel-garden__sprite pixel-garden__sprite--geography" />
            </div>

            <div
              v-for="plant in overflowPlants"
              :key="plant.id"
              class="pixel-garden__sprite pixel-garden__sprite--overflow"
              :class="`pixel-garden__sprite--${plant.category}`"
              :style="overflowSpriteStyle(plant)"
            />

            <div
              v-for="plant in visibleFillerPlants"
              :key="plant.id"
              class="pixel-garden__filler"
              :class="plant.className"
            />

            <div class="pixel-garden__ground" />
          </div>

          <div class="pixel-garden__legend">
            <span>History: {{ growth.history }} edits</span>
            <span>Science: {{ growth.science }} edits</span>
            <span>Arts: {{ growth.arts }} edits</span>
            <span>Geography: {{ growth.geography }} edits</span>
          </div>

          <dl class="pixel-garden__stats" aria-label="Garden status">
            <div>
              <dt>Total edits</dt>
              <dd>{{ totalEdits }}</dd>
            </div>
            <div>
              <dt>Completed beds</dt>
              <dd>{{ completedBeds }}/4</dd>
            </div>
          </dl>

          <p class="pixel-garden__mood">{{ gardenMood }}</p>
        </section>

        <section class="pixel-garden__panel pixel-garden__panel--log" aria-label="Recent growth log">
          <h2 class="pixel-garden__panel-title">Recent growth</h2>

          <ul v-if="growthLog.length > 0" class="pixel-garden__log-list">
            <li v-for="entry in growthLog" :key="entry.id" class="pixel-garden__log-item">
              <p class="pixel-garden__log-note">{{ entry.note }}</p>
              <p class="pixel-garden__log-meta">{{ entry.timestamp }}</p>
            </li>
          </ul>
          <p v-else class="pixel-garden__empty">No edits yet. Plant your first seed.</p>
        </section>
      </div>

      <form class="pixel-garden__dev-controls" @submit.prevent="logEdit">
        <p class="pixel-garden__dev-title">Prototype simulator only</p>

        <div class="pixel-garden__dev-row">
          <CdxSelect
            v-model:selected="selectedCategory"
            class="pixel-garden__dev-select"
            aria-label="Edit category"
            :menu-items="categoryOptions"
          />
          <CdxSelect
            v-model:selected="selectedSubtype"
            class="pixel-garden__dev-select"
            aria-label="Edit subtype"
            :menu-items="currentSubtypeOptions"
          />
          <CdxTextInput
            v-model="editNote"
            class="pixel-garden__dev-note"
            aria-label="Edit note"
            placeholder="Optional note"
          />
          <CdxButton action="progressive" weight="quiet" type="submit">Grow</CdxButton>
          <CdxButton weight="quiet" type="button" @click="resetGarden">Reset</CdxButton>
        </div>
      </form>
    </section>
  </ChromeWrapper>
</template>

<style scoped>
.pixel-garden {
  max-width: 74rem;
  margin: 0 auto;
  padding: var(--spacing-125) var(--spacing-100) var(--spacing-150);
}

.pixel-garden__header {
  margin-bottom: var(--spacing-125);
}

.pixel-garden__eyebrow {
  margin: 0 0 var(--spacing-25);
  font-size: var(--font-size-small);
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--color-subtle);
}

.pixel-garden__title {
  margin: 0;
  font-size: clamp(1.6rem, 3.4vw, 2.4rem);
  line-height: 1.2;
}

.pixel-garden__subtitle {
  margin: var(--spacing-50) 0 0;
  color: var(--color-subtle);
  max-width: 42rem;
}

.pixel-garden__style-controls {
  margin-top: var(--spacing-75);
  display: flex;
  flex-wrap: wrap;
  gap: var(--spacing-75);
}

.pixel-garden__style-item {
  min-width: 12rem;
}

.pixel-garden__style-label {
  display: block;
  margin-bottom: var(--spacing-25);
  font-size: var(--font-size-small);
  color: var(--color-subtle);
}

.pixel-garden__layout {
  display: grid;
  gap: var(--spacing-100);
  grid-template-columns: minmax(24rem, 1fr) minmax(17rem, 20rem);
  align-items: start;
}

.pixel-garden__panel {
  border: var(--border-width-base) solid var(--border-color-subtle);
  border-radius: var(--border-radius-base);
  background: var(--background-color-base);
  padding: var(--spacing-100);
}

.pixel-garden__panel-title {
  margin: 0 0 var(--spacing-100);
  font-size: var(--font-size-large);
}

.pixel-garden__panel--visual {
  padding: var(--spacing-75);
}

.pixel-garden__scene {
  --pixel-size: clamp(5px, 0.9vw, 9px);
  position: relative;
  width: 100%;
  aspect-ratio: 4 / 3;
  overflow: hidden;
  border: var(--border-width-base) solid var(--border-color-base);
  border-radius: var(--border-radius-base);
  background: linear-gradient(180deg, #a9d3ff 0%, #cbe9ff 60%, #8abd66 60%, #6ea34d 100%);
}

.pixel-garden__sky {
  position: absolute;
  inset: 0;
  background:
    radial-gradient(circle at 20% 26%, rgba(255, 255, 255, 0.33), transparent 30%),
    radial-gradient(circle at 74% 28%, rgba(255, 255, 255, 0.22), transparent 25%);
}

.pixel-garden__stars {
  position: absolute;
  inset: 0;
  opacity: 0;
  transition: opacity 240ms ease;
  background:
    radial-gradient(circle at 12% 18%, rgba(255, 255, 255, 0.95) 0 1px, transparent 2px),
    radial-gradient(circle at 24% 34%, rgba(255, 255, 255, 0.8) 0 1px, transparent 2px),
    radial-gradient(circle at 40% 13%, rgba(255, 255, 255, 0.85) 0 1px, transparent 2px),
    radial-gradient(circle at 57% 18%, rgba(255, 255, 255, 0.8) 0 1px, transparent 2px),
    radial-gradient(circle at 78% 24%, rgba(255, 255, 255, 0.9) 0 1px, transparent 2px);
}

.pixel-garden__sun,
.pixel-garden__moon {
  position: absolute;
  top: 11%;
  right: 10%;
  width: 10%;
  aspect-ratio: 1;
  border-radius: 50%;
  transition: opacity 240ms ease;
}

.pixel-garden__sun {
  background: #ffd159;
  box-shadow: 0 0 0 6px rgba(255, 209, 89, 0.22);
}

.pixel-garden__moon {
  opacity: 0;
  background: #f2f0d7;
  box-shadow: inset -8px -2px 0 #d3d1bf;
}

.pixel-garden__terrain {
  position: absolute;
  inset: auto 0 0;
  z-index: 0;
  opacity: 0.64;
  filter: blur(0.6px) saturate(0.9);
}

.pixel-garden__terrain--far {
  bottom: 48%;
  height: 18%;
}

.pixel-garden__terrain--mid {
  bottom: 40%;
  height: 12%;
}

.pixel-garden__scene--desert .pixel-garden__terrain--far {
  background:
    repeating-linear-gradient(
      90deg,
      rgba(200, 164, 99, 0.58),
      rgba(200, 164, 99, 0.58) 8%,
      rgba(183, 147, 86, 0.58) 8%,
      rgba(183, 147, 86, 0.58) 16%
    );
  clip-path: polygon(0 100%, 8% 82%, 20% 86%, 31% 74%, 44% 82%, 60% 72%, 73% 84%, 84% 80%, 100% 100%);
}

.pixel-garden__scene--desert .pixel-garden__terrain--mid {
  background:
    repeating-linear-gradient(
      90deg,
      rgba(221, 184, 117, 0.56),
      rgba(221, 184, 117, 0.56) 10%,
      rgba(204, 165, 100, 0.56) 10%,
      rgba(204, 165, 100, 0.56) 20%
    );
  clip-path: polygon(0 100%, 10% 86%, 24% 90%, 40% 78%, 57% 86%, 71% 77%, 89% 90%, 100% 100%);
}

.pixel-garden__scene--mountain .pixel-garden__terrain--far {
  background:
    repeating-linear-gradient(
      90deg,
      rgba(95, 114, 131, 0.6),
      rgba(95, 114, 131, 0.6) 8%,
      rgba(79, 97, 113, 0.6) 8%,
      rgba(79, 97, 113, 0.6) 16%
    );
  clip-path: polygon(0 100%, 7% 76%, 21% 52%, 36% 77%, 51% 46%, 67% 79%, 80% 58%, 100% 100%);
}

.pixel-garden__scene--mountain .pixel-garden__terrain--mid {
  background:
    repeating-linear-gradient(
      90deg,
      rgba(126, 142, 157, 0.58),
      rgba(126, 142, 157, 0.58) 10%,
      rgba(112, 128, 144, 0.58) 10%,
      rgba(112, 128, 144, 0.58) 20%
    );
  clip-path: polygon(0 100%, 9% 84%, 23% 64%, 37% 85%, 52% 60%, 69% 86%, 83% 67%, 100% 100%);
}

.pixel-garden__ground {
  position: absolute;
  inset: auto 0 0;
  height: 38%;
  z-index: 1;
  background:
    repeating-linear-gradient(
      90deg,
      rgba(76, 119, 55, 0.26),
      rgba(76, 119, 55, 0.26) 7%,
      rgba(85, 134, 61, 0.26) 7%,
      rgba(85, 134, 61, 0.26) 14%
    );
}

.pixel-garden__scene--night {
  background: linear-gradient(180deg, #283357 0%, #3f4f84 62%, #536842 62%, #405433 100%);
}

.pixel-garden__scene--night .pixel-garden__sun {
  opacity: 0;
}

.pixel-garden__scene--night .pixel-garden__moon,
.pixel-garden__scene--night .pixel-garden__stars {
  opacity: 1;
}

.pixel-garden__bed {
  position: absolute;
  bottom: 24%;
  width: 22%;
  height: 56%;
}

.pixel-garden__bed--history {
  left: 2%;
}

.pixel-garden__bed--science {
  left: 26%;
}

.pixel-garden__bed--arts {
  left: 50%;
}

.pixel-garden__bed--geography {
  left: 74%;
}

.pixel-garden__sprite {
  position: absolute;
  inset: auto 50% calc(var(--pixel-size) * 1.75) auto;
  width: calc(var(--pixel-size) * 18);
  height: calc(var(--pixel-size) * 18);
  transform: translateX(50%) scale(0.5);
  transform-origin: bottom center;
  opacity: 0;
  z-index: 2;
  filter: saturate(1.08) contrast(1.12) drop-shadow(0 1px 0 rgba(16, 38, 12, 0.65)) drop-shadow(0 0 3px rgba(245, 255, 220, 0.25));
  transition: transform 220ms ease, opacity 220ms ease;
}

.pixel-garden__sprite--overflow {
  inset: auto auto var(--sprite-bottom) var(--sprite-left);
  transform: translateX(-50%) scale(var(--sprite-scale));
  opacity: 1;
}

.pixel-garden__bed.stage-1 .pixel-garden__sprite,
.pixel-garden__bed.stage-2 .pixel-garden__sprite,
.pixel-garden__bed.stage-3 .pixel-garden__sprite,
.pixel-garden__bed.stage-4 .pixel-garden__sprite,
.pixel-garden__bed.stage-5 .pixel-garden__sprite,
.pixel-garden__bed.stage-6 .pixel-garden__sprite,
.pixel-garden__bed.stage-7 .pixel-garden__sprite,
.pixel-garden__bed.stage-8 .pixel-garden__sprite,
.pixel-garden__bed.stage-9 .pixel-garden__sprite,
.pixel-garden__bed.stage-10 .pixel-garden__sprite {
  opacity: 1;
}

.pixel-garden__bed.stage-1 .pixel-garden__sprite {
  transform: translateX(50%) scale(0.78);
}

.pixel-garden__bed.stage-2 .pixel-garden__sprite {
  transform: translateX(50%) scale(0.92);
}

.pixel-garden__bed.stage-3 .pixel-garden__sprite {
  transform: translateX(50%) scale(1.05);
}

.pixel-garden__bed.stage-4 .pixel-garden__sprite {
  transform: translateX(50%) scale(1.17);
}

.pixel-garden__bed.stage-5 .pixel-garden__sprite {
  transform: translateX(50%) scale(1.29);
}

.pixel-garden__bed.stage-6 .pixel-garden__sprite {
  transform: translateX(50%) scale(1.40);
}

.pixel-garden__bed.stage-7 .pixel-garden__sprite {
  transform: translateX(50%) scale(1.50);
}

.pixel-garden__bed.stage-8 .pixel-garden__sprite {
  transform: translateX(50%) scale(1.60);
}

.pixel-garden__bed.stage-9 .pixel-garden__sprite {
  transform: translateX(50%) scale(1.69);
}

.pixel-garden__bed.stage-10 .pixel-garden__sprite {
  transform: translateX(50%) scale(1.78);
}

.pixel-garden__sprite::before,
.pixel-garden__sprite::after {
  content: '';
  position: absolute;
  width: var(--pixel-size);
  height: var(--pixel-size);
  image-rendering: pixelated;
}

.pixel-garden__sprite--history::after {
  left: 50%;
  bottom: calc(var(--pixel-size) * 0.5);
  transform: translateX(-50%);
  background: #7a4b2b;
  box-shadow:
    calc(var(--pixel-size) * -1) 0 #7a4b2b,
    calc(var(--pixel-size) * 1) 0 #7a4b2b,
    0 calc(var(--pixel-size) * -1) #7a4b2b,
    0 calc(var(--pixel-size) * -2) #7a4b2b,
    0 calc(var(--pixel-size) * -3) #7a4b2b,
    0 calc(var(--pixel-size) * -4) #7a4b2b;
}

.pixel-garden__sprite--history::before {
  left: 50%;
  bottom: calc(var(--pixel-size) * 6);
  transform: translateX(-50%);
  background: #4f8b38;
  box-shadow:
    calc(var(--pixel-size) * -3) calc(var(--pixel-size) * 1) #6dbd4d,
    calc(var(--pixel-size) * -2) 0 #63ab46,
    calc(var(--pixel-size) * -1) calc(var(--pixel-size) * -1) #63ab46,
    0 calc(var(--pixel-size) * -2) #5ca63f,
    calc(var(--pixel-size) * 1) calc(var(--pixel-size) * -1) #63ab46,
    calc(var(--pixel-size) * 2) 0 #63ab46,
    calc(var(--pixel-size) * 3) calc(var(--pixel-size) * 1) #6dbd4d;
}

.pixel-garden__sprite--science::after {
  left: 50%;
  bottom: calc(var(--pixel-size) * 0.5);
  transform: translateX(-50%);
  background: #50724a;
  box-shadow:
    calc(var(--pixel-size) * -1) 0 #50724a,
    calc(var(--pixel-size) * 1) 0 #50724a,
    0 calc(var(--pixel-size) * -1) #4c6b46,
    0 calc(var(--pixel-size) * -2) #4c6b46;
}

.pixel-garden__sprite--science::before {
  left: 50%;
  bottom: calc(var(--pixel-size) * 4.5);
  transform: translateX(-50%);
  background: #6de9f0;
  box-shadow:
    calc(var(--pixel-size) * -1) calc(var(--pixel-size) * 1) #54c7d7,
    calc(var(--pixel-size) * 1) calc(var(--pixel-size) * 1) #54c7d7,
    calc(var(--pixel-size) * -2) calc(var(--pixel-size) * 2) #3caebf,
    calc(var(--pixel-size) * 2) calc(var(--pixel-size) * 2) #3caebf,
    0 calc(var(--pixel-size) * -1) #8bf6ff,
    0 calc(var(--pixel-size) * -2) #b3fcff;
}

.pixel-garden__sprite--arts::after {
  left: 50%;
  bottom: calc(var(--pixel-size) * 0.5);
  transform: translateX(-50%);
  background: #4d8f40;
  box-shadow:
    0 calc(var(--pixel-size) * -1) #4d8f40,
    0 calc(var(--pixel-size) * -2) #4d8f40,
    calc(var(--pixel-size) * -1) calc(var(--pixel-size) * -1) #4d8f40,
    calc(var(--pixel-size) * 1) calc(var(--pixel-size) * -1) #4d8f40;
}

.pixel-garden__sprite--arts::before {
  left: 50%;
  bottom: calc(var(--pixel-size) * 4.2);
  transform: translateX(-50%);
  background: var(--arts-core, #ffd95f);
  box-shadow:
    calc(var(--pixel-size) * -2) 0 var(--arts-petal, #ff8fb6),
    calc(var(--pixel-size) * 2) 0 var(--arts-petal, #ff8fb6),
    0 calc(var(--pixel-size) * -2) var(--arts-petal, #ff8fb6),
    0 calc(var(--pixel-size) * 2) var(--arts-petal, #ff8fb6),
    calc(var(--pixel-size) * -1) calc(var(--pixel-size) * -1) var(--arts-accent, #ffa5c5),
    calc(var(--pixel-size) * 1) calc(var(--pixel-size) * -1) var(--arts-accent, #ffa5c5),
    calc(var(--pixel-size) * -1) calc(var(--pixel-size) * 1) var(--arts-accent, #ffa5c5),
    calc(var(--pixel-size) * 1) calc(var(--pixel-size) * 1) var(--arts-accent, #ffa5c5);
}

.pixel-garden__sprite--geography::after {
  left: 50%;
  bottom: calc(var(--pixel-size) * 0.5);
  transform: translateX(-50%);
  background: #5e8441;
  box-shadow:
    calc(var(--pixel-size) * -2) 0 #5e8441,
    calc(var(--pixel-size) * -1) calc(var(--pixel-size) * -1) #5e8441,
    calc(var(--pixel-size) * 1) calc(var(--pixel-size) * -1) #5e8441,
    calc(var(--pixel-size) * 2) 0 #5e8441;
}

.pixel-garden__sprite--geography::before {
  left: 50%;
  bottom: calc(var(--pixel-size) * 3.2);
  transform: translateX(-50%);
  background: #4e973c;
  box-shadow:
    calc(var(--pixel-size) * -3) calc(var(--pixel-size) * 1) #6bb953,
    calc(var(--pixel-size) * -2) 0 #5ba846,
    calc(var(--pixel-size) * -1) calc(var(--pixel-size) * -1) #4e973c,
    0 calc(var(--pixel-size) * -2) #4e973c,
    calc(var(--pixel-size) * 1) calc(var(--pixel-size) * -1) #4e973c,
    calc(var(--pixel-size) * 2) 0 #5ba846,
    calc(var(--pixel-size) * 3) calc(var(--pixel-size) * 1) #6bb953;
}

.pixel-garden__filler {
  position: absolute;
  width: calc(var(--pixel-size) * 1.2);
  height: calc(var(--pixel-size) * 1.2);
  bottom: 23%;
}

.pixel-garden__filler::before,
.pixel-garden__filler::after {
  content: '';
  position: absolute;
  width: 100%;
  height: 100%;
}

.pixel-garden__filler::before {
  background: #4a8e3b;
  box-shadow:
    calc(var(--pixel-size) * -1) 0 #63ab46,
    calc(var(--pixel-size) * 1) 0 #63ab46;
}

.pixel-garden__filler::after {
  bottom: calc(var(--pixel-size) * 1.5);
  background: #f7c16d;
}

.pixel-garden__filler--1 {
  left: 8%;
}

.pixel-garden__filler--2 {
  left: 15%;
  bottom: 25%;
}

.pixel-garden__filler--3 {
  left: 22%;
}

.pixel-garden__filler--4 {
  left: 34%;
  bottom: 22%;
}

.pixel-garden__filler--5 {
  left: 41%;
}

.pixel-garden__filler--6 {
  left: 58%;
  bottom: 25%;
}

.pixel-garden__filler--7 {
  left: 65%;
}

.pixel-garden__filler--8 {
  left: 72%;
  bottom: 22%;
}

.pixel-garden__filler--9 {
  left: 84%;
}

.pixel-garden__filler--10 {
  left: 90%;
  bottom: 25%;
}

.pixel-garden__filler--2::after,
.pixel-garden__filler--6::after,
.pixel-garden__filler--9::after {
  background: #8fd3ff;
}

.pixel-garden__filler--4::after,
.pixel-garden__filler--7::after,
.pixel-garden__filler--10::after {
  background: #ff9fc0;
}

.pixel-garden__legend {
  margin-top: var(--spacing-75);
  display: flex;
  flex-wrap: wrap;
  gap: var(--spacing-50);
  font-size: var(--font-size-small);
  color: var(--color-subtle);
}

.pixel-garden__stats {
  margin: var(--spacing-100) 0 0;
  display: grid;
  gap: var(--spacing-50);
  grid-template-columns: repeat(2, minmax(0, 1fr));
}

.pixel-garden__stats dt {
  font-size: var(--font-size-small);
  color: var(--color-subtle);
}

.pixel-garden__stats dd {
  margin: var(--spacing-25) 0 0;
  font-size: 1.25rem;
  font-weight: var(--font-weight-bold);
}

.pixel-garden__mood {
  margin: var(--spacing-100) 0 0;
  padding-top: var(--spacing-75);
  border-top: var(--border-width-base) dashed var(--border-color-subtle);
  font-size: var(--font-size-small);
}

.pixel-garden__panel--log {
  max-height: 36rem;
  overflow: auto;
}

.pixel-garden__log-list {
  margin: 0;
  padding: 0;
  list-style: none;
  display: grid;
  gap: var(--spacing-75);
}

.pixel-garden__log-item {
  padding-bottom: var(--spacing-75);
  border-bottom: var(--border-width-base) solid var(--border-color-subtle);
}

.pixel-garden__log-note {
  margin: 0;
}

.pixel-garden__log-meta {
  margin: var(--spacing-25) 0 0;
  font-size: var(--font-size-small);
  color: var(--color-subtle);
}

.pixel-garden__empty {
  margin: 0;
  color: var(--color-subtle);
}

.pixel-garden__dev-controls {
  margin-top: var(--spacing-75);
  padding: var(--spacing-50) var(--spacing-75);
  border: var(--border-width-base) dashed var(--border-color-subtle);
  border-radius: var(--border-radius-base);
  background: color-mix(in srgb, var(--background-color-base) 70%, var(--background-color-interactive-subtle));
}

.pixel-garden__dev-title {
  margin: 0 0 var(--spacing-35);
  font-size: var(--font-size-x-small);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--color-subtle);
}

.pixel-garden__dev-row {
  display: flex;
  gap: var(--spacing-35);
  align-items: center;
}

.pixel-garden__dev-select {
  width: min(16rem, 32vw);
}

.pixel-garden__dev-note {
  width: min(16rem, 35vw);
}

@media (max-width: 960px) {
  .pixel-garden__layout {
    grid-template-columns: 1fr;
  }

  .pixel-garden__panel--log {
    max-height: none;
  }

  .pixel-garden__dev-row {
    flex-wrap: wrap;
  }

  .pixel-garden__dev-select,
  .pixel-garden__dev-note {
    width: 100%;
  }
}
</style>
