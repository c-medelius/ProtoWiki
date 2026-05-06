<script setup lang="ts">
import { computed, ref } from 'vue'
import { CdxButton, CdxSelect, CdxTextInput } from '@wikimedia/codex'

import ChromeWrapper from '@/components/ChromeWrapper.vue'
import SpecialPageWrapper from '@/components/SpecialPageWrapper.vue'

definePage({
  meta: {
    title: 'Impact story share',
    description: 'A newcomer-homepage prototype that expands the impact module toward shareable story graphics.',
  },
})

type StoryPresetId = 'breakout' | 'fixer' | 'commons'
type StoryTemplateId = 'spotlight' | 'scrapbook'

interface StoryPreset {
  id: StoryPresetId
  label: string
  editorName: string
  articleTitle: string
  articleTag: string
  impactHeadline: string
  impactSummary: string
  readerCount: number
  editCount: number
  thanksCount: number
  rankLabel: string
  shareCaption: string
  palette: {
    top: string
    bottom: string
    accent: string
    accentSoft: string
    card: string
    ink: string
  }
}

interface StoryTemplateOption {
  value: StoryTemplateId
  label: string
}

interface ChecklistItem {
  label: string
  detail: string
  completed: boolean
}

interface HelpLink {
  label: string
  detail: string
}

const presetOptions = [
  { value: 'breakout', label: 'Breakout article' },
  { value: 'fixer', label: 'Steady fixer' },
  { value: 'commons', label: 'Visual contributor' },
]

const templateOptions: StoryTemplateOption[] = [
  { value: 'spotlight', label: 'Spotlight' },
  { value: 'scrapbook', label: 'Scrapbook' },
]

const checklistItems: ChecklistItem[] = [
  {
    label: 'Add an email address',
    detail: 'Recover your account and receive notifications.',
    completed: true,
  },
  {
    label: 'Try a newcomer task',
    detail: 'Pick a small article improvement to keep editing momentum.',
    completed: false,
  },
  {
    label: 'Create your user page',
    detail: 'Tell other editors a bit about your interests.',
    completed: false,
  },
]

const helpLinks: HelpLink[] = [
  {
    label: 'Step-by-step guide to editing',
    detail: 'A quick orientation to how article editing works.',
  },
  {
    label: 'Ask the help desk',
    detail: 'Get an answer when something feels confusing.',
  },
  {
    label: 'Find easy tasks',
    detail: 'Browse structured work that is good for newcomers.',
  },
]

const storyPresets: Record<StoryPresetId, StoryPreset> = {
  breakout: {
    id: 'breakout',
    label: 'Breakout article',
    editorName: 'sebastian',
    articleTitle: 'Wet Leg',
    articleTag: 'Music article',
    impactHeadline: 'Your edits hit a nerve.',
    impactSummary:
      'You tightened references, smoothed the lead, and helped an article readers suddenly cared about feel complete.',
    readerCount: 52800,
    editCount: 14,
    thanksCount: 231,
    rankLabel: 'Top 3% of edited pages this week',
    shareCaption: 'A few careful edits. Tens of thousands of curious readers.',
    palette: {
      top: '#f97360',
      bottom: '#f8d66d',
      accent: '#0f5fd7',
      accentSoft: 'rgba(15, 95, 215, 0.14)',
      card: 'rgba(255, 250, 240, 0.92)',
      ink: '#1e252b',
    },
  },
  fixer: {
    id: 'fixer',
    label: 'Steady fixer',
    editorName: 'sebastian',
    articleTitle: 'Chicago Manual of Style',
    articleTag: 'Reference article',
    impactHeadline: 'Quiet edits, huge usefulness.',
    impactSummary:
      'You kept a practical article tidy and trustworthy. The work was invisible in the best way: readers just got what they needed.',
    readerCount: 18400,
    editCount: 39,
    thanksCount: 87,
    rankLabel: 'Used heavily by students this month',
    shareCaption: 'Not every impact story is viral. Some are just relentlessly useful.',
    palette: {
      top: '#2448b8',
      bottom: '#83b6ff',
      accent: '#ffe07a',
      accentSoft: 'rgba(255, 224, 122, 0.18)',
      card: 'rgba(248, 251, 255, 0.9)',
      ink: '#122033',
    },
  },
  commons: {
    id: 'commons',
    label: 'Visual contributor',
    editorName: 'sebastian',
    articleTitle: 'Solar eclipse',
    articleTag: 'Illustrated article',
    impactHeadline: 'You made the page more shareable.',
    impactSummary:
      'A stronger image choice and cleaner captions made the article feel more vivid, faster to understand, and easier to pass around.',
    readerCount: 73100,
    editCount: 9,
    thanksCount: 149,
    rankLabel: 'High-interest page during a global event',
    shareCaption: 'A visual contribution can change whether an article gets remembered.',
    palette: {
      top: '#121a4f',
      bottom: '#7d64ff',
      accent: '#7ff0cf',
      accentSoft: 'rgba(127, 240, 207, 0.16)',
      card: 'rgba(247, 245, 255, 0.9)',
      ink: '#171b2d',
    },
  },
}

const selectedPresetId = ref<StoryPresetId>('breakout')
const selectedTemplateId = ref<StoryTemplateId>('spotlight')
const customEditorName = ref('sebastian')

const selectedPreset = computed(() => storyPresets[selectedPresetId.value])

const editorName = computed(() => {
  const trimmedName = customEditorName.value.trim()
  return trimmedName.length > 0 ? trimmedName : selectedPreset.value.editorName
})

const spotlightNarrative = computed(() => {
  const preset = selectedPreset.value
  return `${editorName.value} helped ${preset.articleTitle} reach ${formatCompactNumber(preset.readerCount)} readers.`
})

const scrapbookNarrative = computed(() => {
  const preset = selectedPreset.value
  return `${preset.editCount} edits. ${preset.thanksCount} thanks. One page people trusted more.`
})

const activeTemplateLabel = computed(
  () => templateOptions.find((option) => option.value === selectedTemplateId.value)?.label ?? 'Spotlight',
)

const moduleStatus = computed(() => {
  const preset = selectedPreset.value
  return `People viewed pages you edited ${formatCompactNumber(preset.readerCount)} times recently.`
})

function cyclePreset() {
  const currentIndex = presetOptions.findIndex((option) => option.value === selectedPresetId.value)
  const nextOption = presetOptions[(currentIndex + 1) % presetOptions.length]
  selectedPresetId.value = nextOption.value as StoryPresetId
}

function formatCompactNumber(value: number): string {
  return new Intl.NumberFormat('en', {
    notation: 'compact',
    maximumFractionDigits: value >= 10000 ? 1 : 0,
  }).format(value)
}

function cardStyle(preset: StoryPreset) {
  return {
    '--story-top': preset.palette.top,
    '--story-bottom': preset.palette.bottom,
    '--story-accent': preset.palette.accent,
    '--story-accent-soft': preset.palette.accentSoft,
    '--story-card': preset.palette.card,
    '--story-ink': preset.palette.ink,
  }
}
</script>

<template>
  <ChromeWrapper>
    <SpecialPageWrapper title="Homepage">
      <template #notices>
        <div class="impact-story-share__notice">Only you can see your homepage and impact summary.</div>
      </template>

      <section class="impact-story-share">
        <header class="impact-story-share__page-header">
          <div>
            <p class="impact-story-share__eyebrow">Newcomer homepage prototype</p>
            <h2>Welcome, {{ editorName }}</h2>
            <p class="impact-story-share__lede">
              This version re-frames the story-sharing idea inside a homepage module. The page looks like a
              newcomer dashboard first, and the impact module is where we can gradually add shareable
              graphics.
            </p>
          </div>

          <aside class="impact-story-share__homepage-summary">
            <p class="impact-story-share__summary-label">Your recent activity</p>
            <dl class="impact-story-share__metrics">
              <div>
                <dt>Impact</dt>
                <dd>{{ formatCompactNumber(selectedPreset.readerCount) }} views</dd>
              </div>
              <div>
                <dt>Edits</dt>
                <dd>{{ selectedPreset.editCount }} improvements</dd>
              </div>
              <div>
                <dt>Next step</dt>
                <dd>Review your story card</dd>
              </div>
            </dl>
          </aside>
        </header>

        <div class="impact-story-share__layout">
          <section class="impact-story-share__main-column">
            <article class="homepage-module homepage-module--impact">
              <header class="homepage-module__header">
                <div>
                  <p class="homepage-module__eyebrow">Impact</p>
                  <h3>People are reading pages you improved</h3>
                  <p class="homepage-module__lede">{{ moduleStatus }}</p>
                </div>
              </header>

              <div class="impact-module__summary-strip">
                <div class="impact-module__summary-card">
                  <span class="impact-module__summary-label">Current article</span>
                  <strong>{{ selectedPreset.articleTitle }}</strong>
                  <p>{{ selectedPreset.articleTag }}</p>
                </div>
                <div class="impact-module__summary-card">
                  <span class="impact-module__summary-label">Encouragement</span>
                  <strong>{{ selectedPreset.rankLabel }}</strong>
                  <p>{{ selectedPreset.impactSummary }}</p>
                </div>
              </div>

              <div class="impact-module__controls">
                <div class="impact-story-share__field">
                  <span class="impact-story-share__label">Impact example</span>
                  <CdxSelect
                    v-model:selected="selectedPresetId"
                    default-label="Choose a preset"
                    :menu-items="presetOptions"
                  />
                </div>
                <div class="impact-story-share__field">
                  <span class="impact-story-share__label">Graphic style</span>
                  <CdxSelect
                    v-model:selected="selectedTemplateId"
                    default-label="Choose a template"
                    :menu-items="templateOptions"
                  />
                </div>
                <div class="impact-story-share__field impact-story-share__field--name">
                  <span class="impact-story-share__label">Display name</span>
                  <CdxTextInput v-model="customEditorName" placeholder="Use a display name" />
                </div>
                <div class="impact-story-share__actions">
                  <CdxButton action="progressive" weight="primary" @click="cyclePreset">
                    Try another example
                  </CdxButton>
                </div>
              </div>

              <div class="impact-module__expanded">
                <div class="impact-module__copy">
                  <p class="impact-story-share__summary-label">Expanded impact module</p>
                  <p class="impact-story-share__summary-headline">{{ selectedPreset.impactHeadline }}</p>
                  <p class="impact-story-share__summary-copy">
                    {{ selectedPreset.shareCaption }} This is the surface we can extend with share actions,
                    export states, and template switching.
                  </p>
                  <ul class="impact-module__bullet-list">
                    <li>{{ selectedPreset.editCount }} recent edits on {{ selectedPreset.articleTitle }}</li>
                    <li>{{ selectedPreset.thanksCount }} thanks or acknowledgements from others</li>
                    <li>{{ activeTemplateLabel }} is the current story treatment</li>
                  </ul>
                </div>

                <article class="story-preview story-preview--embedded">
                  <header class="story-preview__header">
                    <div>
                      <p class="story-preview__label">Impact graphic preview</p>
                      <h4>{{ activeTemplateLabel }}</h4>
                    </div>
                    <span class="story-preview__badge">Future share surface</span>
                  </header>

                  <div
                    class="story-preview__phone"
                    :class="`story-preview__phone--${selectedTemplateId}`"
                    :style="cardStyle(selectedPreset)"
                  >
                    <div class="story-preview__safe-zone story-preview__safe-zone--top">
                      <span></span>
                      <span></span>
                      <span></span>
                    </div>

                    <div class="story-preview__safe-zone story-preview__safe-zone--bottom"></div>

                    <template v-if="selectedTemplateId === 'spotlight'">
                      <p class="story-preview__kicker">Wikipedia impact</p>
                      <h3>{{ selectedPreset.impactHeadline }}</h3>
                      <p class="story-preview__narrative">{{ spotlightNarrative }}</p>

                      <div class="story-preview__hero-stat">
                        <span>{{ formatCompactNumber(selectedPreset.readerCount) }}</span>
                        <small>readers reached after recent improvements</small>
                      </div>

                      <div class="story-preview__article-card">
                        <p>{{ selectedPreset.articleTag }}</p>
                        <strong>{{ selectedPreset.articleTitle }}</strong>
                        <span>{{ selectedPreset.rankLabel }}</span>
                      </div>

                      <div class="story-preview__chip-row">
                        <span>{{ selectedPreset.editCount }} edits</span>
                        <span>{{ selectedPreset.thanksCount }} thanks</span>
                        <span>@{{ editorName.toLowerCase().replace(/\s+/g, '') }}</span>
                      </div>
                    </template>

                    <template v-else>
                      <div class="story-preview__scrapbook-top">
                        <div class="story-preview__scrapbook-card story-preview__scrapbook-card--quote">
                          <p>"{{ selectedPreset.shareCaption }}"</p>
                        </div>
                        <div class="story-preview__scrapbook-card story-preview__scrapbook-card--stat">
                          <strong>{{ formatCompactNumber(selectedPreset.readerCount) }}</strong>
                          <span>people used this page recently</span>
                        </div>
                      </div>

                      <div class="story-preview__scrapbook-title">
                        <p>{{ selectedPreset.articleTag }}</p>
                        <h3>{{ selectedPreset.articleTitle }}</h3>
                      </div>

                      <div class="story-preview__scrapbook-grid">
                        <div class="story-preview__tile">
                          <strong>{{ selectedPreset.editCount }}</strong>
                          <span>edits</span>
                        </div>
                        <div class="story-preview__tile">
                          <strong>{{ selectedPreset.thanksCount }}</strong>
                          <span>thanks</span>
                        </div>
                        <div class="story-preview__tile story-preview__tile--wide">
                          <strong>{{ scrapbookNarrative }}</strong>
                        </div>
                      </div>

                      <p class="story-preview__footer-line">Shared by {{ editorName }} · Wikipedia volunteer</p>
                    </template>
                  </div>
                </article>
              </div>
            </article>

            <article class="homepage-module homepage-module--notes">
              <header class="homepage-module__header">
                <div>
                  <p class="homepage-module__eyebrow">Prototype direction</p>
                  <h3>How this module can evolve</h3>
                </div>
              </header>
              <p>
                The real product path is: show meaningful impact first, expand into graphic variations second,
                and only then add sharing actions. That keeps the module useful even before export exists.
              </p>
            </article>
          </section>

          <aside class="impact-story-share__side-column">
            <article class="homepage-module">
              <header class="homepage-module__header">
                <div>
                  <p class="homepage-module__eyebrow">Mentorship</p>
                  <h3>Your mentor</h3>
                </div>
              </header>
              <p class="homepage-module__body-copy">
                Ask ExampleMentor when you want feedback on article quality, sources, or where to make your
                next edit.
              </p>
              <CdxButton weight="primary">Ask your mentor</CdxButton>
            </article>

            <article class="homepage-module">
              <header class="homepage-module__header">
                <div>
                  <p class="homepage-module__eyebrow">Start</p>
                  <h3>Complete your setup</h3>
                </div>
              </header>
              <ul class="homepage-module__checklist">
                <li
                  v-for="item in checklistItems"
                  :key="item.label"
                  :class="{ 'homepage-module__checklist-item--done': item.completed }"
                >
                  <span class="homepage-module__checkmark">{{ item.completed ? '✓' : '○' }}</span>
                  <span>
                    <strong>{{ item.label }}</strong>
                    <small>{{ item.detail }}</small>
                  </span>
                </li>
              </ul>
            </article>

            <article class="homepage-module">
              <header class="homepage-module__header">
                <div>
                  <p class="homepage-module__eyebrow">Help</p>
                  <h3>Learn what to do next</h3>
                </div>
              </header>
              <ul class="homepage-module__link-list">
                <li v-for="item in helpLinks" :key="item.label">
                  <strong>{{ item.label }}</strong>
                  <small>{{ item.detail }}</small>
                </li>
              </ul>
            </article>
          </aside>
        </div>
      </section>
    </SpecialPageWrapper>
  </ChromeWrapper>
</template>

<style scoped>
.impact-story-share {
  padding-bottom: var(--spacing-150, 24px);
}

.impact-story-share__notice {
  padding: var(--spacing-75, 12px) 0;
  font-size: var(--font-size-small, 0.875rem);
  color: var(--color-notice, #202122);
}

.impact-story-share__page-header {
  display: grid;
  grid-template-columns: minmax(0, 1.5fr) minmax(260px, 0.9fr);
  gap: var(--spacing-150, 24px);
  align-items: start;
  margin-bottom: var(--spacing-150, 24px);
}

.impact-story-share__page-header h2 {
  margin: 0;
  font-size: clamp(2rem, 4vw, 2.75rem);
  line-height: 1.05;
  color: var(--color-base, #202122);
}

.impact-story-share__eyebrow,
.impact-story-share__summary-label,
.story-preview__label,
.story-preview__kicker,
.homepage-module__eyebrow {
  margin: 0 0 10px;
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: #54606c;
}

.impact-story-share__lede,
.impact-story-share__summary-copy,
.homepage-module__body-copy,
.homepage-module--notes p {
  margin: 12px 0 0;
  max-width: 60ch;
  font-size: 1rem;
  line-height: 1.6;
  color: var(--color-subtle, #54595d);
}

.impact-story-share__homepage-summary,
.homepage-module,
.story-preview {
  border: 1px solid var(--border-color-base, #c8ccd1);
  border-radius: 12px;
  background: var(--background-color-base, #fff);
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}

.impact-story-share__homepage-summary,
.homepage-module {
  padding: var(--spacing-125, 20px);
}

.impact-story-share__layout {
  display: grid;
  grid-template-columns: minmax(0, 1.8fr) minmax(280px, 0.9fr);
  gap: var(--spacing-150, 24px);
  align-items: start;
}

.impact-story-share__main-column,
.impact-story-share__side-column {
  display: grid;
  gap: var(--spacing-125, 20px);
}

.homepage-module__header {
  display: flex;
  justify-content: space-between;
  gap: var(--spacing-100, 16px);
  align-items: start;
  margin-bottom: var(--spacing-100, 16px);
}

.homepage-module__header h3,
.impact-story-share__summary-headline {
  margin: 0;
  font-size: 1.5rem;
  line-height: 1.15;
  color: var(--color-base, #202122);
}

.homepage-module__lede {
  margin: 8px 0 0;
  color: var(--color-subtle, #54595d);
}

.impact-module__summary-strip,
.impact-module__expanded {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: var(--spacing-100, 16px);
}

.impact-module__summary-card {
  padding: var(--spacing-100, 16px);
  border-radius: 8px;
  background: var(--background-color-interactive-subtle, #f8f9fa);
}

.impact-module__summary-label {
  display: block;
  margin-bottom: 8px;
  font-size: 0.8125rem;
  color: var(--color-subtle, #54595d);
}

.impact-module__summary-card strong {
  display: block;
  font-size: 1.125rem;
  color: var(--color-base, #202122);
}

.impact-module__summary-card p {
  margin: 8px 0 0;
  font-size: 0.9375rem;
  line-height: 1.5;
  color: var(--color-subtle, #54595d);
}

.impact-module__controls,
.impact-story-share__metrics {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: var(--spacing-100, 16px);
  margin-top: var(--spacing-125, 20px);
}

.impact-story-share__field {
  min-width: 0;
}

.impact-story-share__label {
  display: block;
  margin-bottom: 8px;
  font-size: 0.875rem;
  font-weight: 600;
  color: var(--color-base, #202122);
}

.impact-story-share__actions {
  display: flex;
  align-items: end;
}

.impact-story-share__metrics div {
  padding: var(--spacing-100, 16px);
  border-radius: 8px;
  background: var(--background-color-interactive-subtle, #f8f9fa);
}

.impact-story-share__metrics dt {
  margin: 0 0 6px;
  font-size: 0.8125rem;
  color: var(--color-subtle, #54595d);
}

.impact-story-share__metrics dd {
  margin: 0;
  font-size: 1.125rem;
  font-weight: 700;
  color: var(--color-base, #202122);
}

.impact-module__copy {
  padding-right: var(--spacing-75, 12px);
}

.impact-module__bullet-list,
.homepage-module__checklist,
.homepage-module__link-list {
  margin: 0;
  padding: 0;
  list-style: none;
}

.impact-module__bullet-list {
  margin-top: var(--spacing-100, 16px);
  display: grid;
  gap: 10px;
}

.impact-module__bullet-list li {
  position: relative;
  padding-left: 18px;
  font-size: 0.95rem;
  line-height: 1.5;
  color: var(--color-subtle, #54595d);
}

.impact-module__bullet-list li::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0.55em;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--color-progressive, #36c);
}

.story-preview {
  padding: 16px;
}

.story-preview__header {
  display: flex;
  justify-content: space-between;
  gap: 16px;
  align-items: flex-start;
  margin-bottom: 16px;
}

.story-preview__header h4 {
  margin: 0;
  font-size: 1.25rem;
  line-height: 1.1;
  color: var(--color-base, #202122);
}

.story-preview__badge {
  display: inline-flex;
  align-items: center;
  padding: 7px 12px;
  border-radius: 999px;
  background: #eef1f5;
  font-size: 0.78rem;
  color: #54595d;
}

.story-preview__phone {
  position: relative;
  overflow: hidden;
  aspect-ratio: 9 / 16;
  min-height: 620px;
  padding: 26px;
  border-radius: 28px;
  color: var(--story-ink);
  background:
    radial-gradient(circle at top right, rgba(255, 255, 255, 0.52), transparent 24%),
    linear-gradient(180deg, var(--story-top) 0%, var(--story-bottom) 100%);
  box-shadow: inset 0 0 0 1px rgba(255, 255, 255, 0.25);
}

.story-preview__phone::before,
.story-preview__phone::after {
  content: '';
  position: absolute;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.18);
  filter: blur(2px);
}

.story-preview__phone::before {
  width: 180px;
  height: 180px;
  top: 80px;
  right: -30px;
}

.story-preview__phone::after {
  width: 220px;
  height: 220px;
  bottom: -30px;
  left: -60px;
}

.story-preview__safe-zone {
  position: absolute;
  left: 18px;
  right: 18px;
  border-radius: 999px;
  border: 1px dashed rgba(255, 255, 255, 0.35);
}

.story-preview__safe-zone--top {
  top: 16px;
  height: 22px;
  display: flex;
  gap: 6px;
  align-items: center;
  padding: 0 12px;
}

.story-preview__safe-zone--top span {
  width: 32px;
  height: 4px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.35);
}

.story-preview__safe-zone--top span:first-child {
  width: 52px;
}

.story-preview__safe-zone--bottom {
  bottom: 16px;
  height: 28px;
}

.story-preview__phone h3,
.story-preview__phone p,
.story-preview__phone strong,
.story-preview__phone span,
.story-preview__phone small {
  position: relative;
  z-index: 1;
}

.story-preview__phone--spotlight {
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  gap: 16px;
}

.story-preview__phone--spotlight h3 {
  margin: 0;
  max-width: 8ch;
  font-size: clamp(2.6rem, 4.6vw, 3.6rem);
  line-height: 0.95;
  letter-spacing: -0.05em;
}

.story-preview__narrative {
  margin: 0;
  max-width: 24ch;
  font-size: 1.06rem;
  line-height: 1.45;
}

.story-preview__hero-stat,
.story-preview__article-card,
.story-preview__scrapbook-card,
.story-preview__scrapbook-title,
.story-preview__tile {
  border-radius: 24px;
  background: var(--story-card);
  backdrop-filter: blur(12px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.08);
}

.story-preview__hero-stat {
  padding: 22px;
}

.story-preview__hero-stat span {
  display: block;
  font-size: clamp(3rem, 5vw, 4.3rem);
  font-weight: 800;
  line-height: 0.9;
  letter-spacing: -0.06em;
}

.story-preview__hero-stat small {
  display: block;
  margin-top: 10px;
  font-size: 0.95rem;
  line-height: 1.35;
  color: rgba(0, 0, 0, 0.66);
}

.story-preview__article-card {
  display: grid;
  gap: 6px;
  padding: 18px;
}

.story-preview__article-card p,
.story-preview__scrapbook-title p,
.story-preview__footer-line {
  margin: 0;
  font-size: 0.84rem;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: rgba(0, 0, 0, 0.6);
}

.story-preview__article-card strong,
.story-preview__scrapbook-title h3 {
  font-size: 1.45rem;
  line-height: 1.05;
}

.story-preview__article-card span {
  font-size: 0.95rem;
  line-height: 1.35;
}

.story-preview__chip-row {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  padding-bottom: 30px;
}

.story-preview__chip-row span {
  display: inline-flex;
  align-items: center;
  padding: 10px 14px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.22);
  backdrop-filter: blur(12px);
  font-size: 0.9rem;
}

.story-preview__phone--scrapbook {
  display: grid;
  align-content: center;
  gap: 18px;
}

.story-preview__scrapbook-top {
  display: grid;
  grid-template-columns: 1.15fr 0.85fr;
  gap: 14px;
}

.story-preview__scrapbook-card {
  min-height: 120px;
  padding: 18px;
}

.story-preview__scrapbook-card--quote {
  display: flex;
  align-items: end;
}

.story-preview__scrapbook-card--quote p {
  margin: 0;
  font-size: 1.18rem;
  line-height: 1.35;
}

.story-preview__scrapbook-card--stat {
  display: grid;
  align-content: space-between;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.94), var(--story-accent-soft));
}

.story-preview__scrapbook-card--stat strong {
  font-size: 2.4rem;
  line-height: 0.92;
  letter-spacing: -0.05em;
}

.story-preview__scrapbook-card--stat span {
  font-size: 0.9rem;
  line-height: 1.35;
}

.story-preview__scrapbook-title {
  padding: 20px;
}

.story-preview__scrapbook-title h3 {
  margin: 8px 0 0;
}

.story-preview__scrapbook-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 14px;
}

.story-preview__tile {
  display: grid;
  gap: 8px;
  min-height: 132px;
  padding: 18px;
}

.story-preview__tile strong {
  font-size: 2.2rem;
  line-height: 0.95;
  letter-spacing: -0.04em;
}

.story-preview__tile span {
  font-size: 0.95rem;
}

.story-preview__tile--wide {
  grid-column: span 2;
  min-height: auto;
  background: rgba(255, 255, 255, 0.72);
}

.story-preview__tile--wide strong {
  font-size: 1.18rem;
  line-height: 1.35;
  letter-spacing: -0.01em;
}

.homepage-module__checklist,
.homepage-module__link-list {
  display: grid;
  gap: 12px;
}

.homepage-module__checklist li,
.homepage-module__link-list li {
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 12px;
  padding: 12px;
  border-radius: 8px;
  background: var(--background-color-interactive-subtle, #f8f9fa);
}

.homepage-module__link-list li {
  grid-template-columns: 1fr;
}

.homepage-module__checkmark {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  font-size: 0.875rem;
  background: #eaecf0;
  color: #54595d;
}

.homepage-module__checklist-item--done .homepage-module__checkmark {
  background: #d5fdf4;
  color: #00755e;
}

.homepage-module__checklist strong,
.homepage-module__link-list strong {
  display: block;
  font-size: 0.95rem;
  color: var(--color-base, #202122);
}

.homepage-module__checklist small,
.homepage-module__link-list small {
  display: block;
  margin-top: 4px;
  font-size: 0.85rem;
  line-height: 1.45;
  color: var(--color-subtle, #54595d);
}

@media (max-width: 1100px) {
  .impact-story-share__page-header,
  .impact-story-share__layout,
  .impact-module__summary-strip,
  .impact-module__expanded {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 860px) {
  .impact-module__controls,
  .impact-story-share__metrics {
    grid-template-columns: 1fr;
  }

  .story-preview__phone {
    min-height: 560px;
    padding: 22px;
  }
}

@media (max-width: 560px) {
  .story-preview {
    padding: 12px;
    border-radius: 24px;
  }

  .story-preview__header {
    flex-direction: column;
  }

  .story-preview__phone {
    min-height: 520px;
    border-radius: 22px;
  }

  .story-preview__scrapbook-top,
  .story-preview__scrapbook-grid {
    grid-template-columns: 1fr;
  }

  .story-preview__tile--wide {
    grid-column: auto;
  }
}
</style>