<script setup lang="ts">
import { CdxField, CdxSelect } from '@wikimedia/codex'
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

definePage({
  meta: {
    title: 'Reader thanks',
    description: "Once the reader has been on this article for 15 seconds, it shows them a non-intrusive option to thank the editor(s).",
  },
})

import Article from '@/components/Article.vue'
import ChromeWrapper from '@/components/ChromeWrapper.vue'

const INSERT_DELAY_MS = 5000

const articleContainerRef = ref<HTMLElement | null>(null)
const showThanksDialog = ref(false)
const thanksMessage = ref('')
const thanksReason = ref('')
const thanksInputMode = ref<'message' | 'reason'>('message')
const thanksDismissed = ref(false)

const reasonOptions = [
  { value: 'school-topic', label: 'Helped me understand a school topic' },
  { value: 'settle-debate', label: 'Helped me verify a claim or settle a debate' },
  { value: 'interesting-info', label: 'I learned something new and interesting' },
  { value: 'quick-overview', label: 'Gave me a quick overview of an unfamiliar topic' },
  { value: 'further-reading', label: 'Pointed me to useful sources for further reading' },
]

const thanksInfoTooltip =
  'could link to info on recent editors of this section or info about editing wikipedia'

const dialogPrompt = computed(() => {
  if (thanksInputMode.value === 'reason') {
    return 'Your thanks has been recorded, but you can optionally let the editor know how this article helped you:'
  }
  return 'Your thanks has been recorded, but you can optionally add a short message. Let the editors who worked on this article know how it helped you!'
})

let insertionTimeout: ReturnType<typeof window.setTimeout> | null = null
let paragraphObserver: MutationObserver | null = null

function onThanksLinkClick(event: MouseEvent) {
  event.preventDefault()
  showThanksDialog.value = true
}

function dismissThanks() {
  thanksMessage.value = ''
  thanksReason.value = ''
  showThanksDialog.value = false
  thanksDismissed.value = true

  const root = articleContainerRef.value
  if (!root) return
  const callout = root.querySelector('.reader-thanks-callout')
  if (!callout) return
  callout.textContent = 'Your thanks has been sent!'
  callout.classList.add('reader-thanks-callout--dismissed')
}

function onSkipThanks() {
  dismissThanks()
}

function onSendThanks() {
  dismissThanks()
}

function buildReaderThanksCallout(): HTMLElement {
  const callout = document.createElement('p')
  callout.className = 'reader-thanks-callout'

  const intro = document.createElement('span')
  intro.textContent = 'Did this article help you? '

  const link = document.createElement('a')
  link.className = 'reader-thanks-callout__link'
  link.href = '#'
  link.textContent = 'Thank the editors!'
  link.addEventListener('click', onThanksLinkClick)

  callout.append(intro, link)
  return callout
}

function getParagraphPairIndex(paragraphs: HTMLParagraphElement[]): number {
  if (paragraphs.length < 2) return -1

  const viewportCenterY = window.innerHeight / 2
  let bestIndex = 0
  let bestDistance = Number.POSITIVE_INFINITY

  for (let i = 0; i < paragraphs.length - 1; i += 1) {
    const currentRect = paragraphs[i].getBoundingClientRect()
    const nextRect = paragraphs[i + 1].getBoundingClientRect()
    const pairMidpointY = (currentRect.bottom + nextRect.top) / 2
    const distance = Math.abs(pairMidpointY - viewportCenterY)

    if (distance < bestDistance) {
      bestDistance = distance
      bestIndex = i
    }
  }

  return bestIndex
}

function insertReaderThanksCallout(): boolean {
  const root = articleContainerRef.value
  if (!root) return false
  if (root.querySelector('.reader-thanks-callout')) return true

  const paragraphs = Array.from(
    root.querySelectorAll<HTMLParagraphElement>('.mw-parser-output p'),
  ).filter((p) => (p.textContent?.trim().length ?? 0) > 0)

  if (paragraphs.length < 2) return false

  const pairStartIndex = getParagraphPairIndex(paragraphs)
  if (pairStartIndex < 0) return false

  const targetParagraph = paragraphs[pairStartIndex]
  targetParagraph.insertAdjacentElement('afterend', buildReaderThanksCallout())
  return true
}

function stopObservingParagraphs() {
  if (!paragraphObserver) return
  paragraphObserver.disconnect()
  paragraphObserver = null
}

function startObservingParagraphs() {
  const root = articleContainerRef.value
  if (!root) return

  paragraphObserver = new MutationObserver(() => {
    if (insertReaderThanksCallout()) {
      stopObservingParagraphs()
    }
  })

  paragraphObserver.observe(root, {
    childList: true,
    subtree: true,
  })
}

onMounted(() => {
  insertionTimeout = window.setTimeout(() => {
    if (!insertReaderThanksCallout()) {
      startObservingParagraphs()
    }
  }, INSERT_DELAY_MS)
})

onBeforeUnmount(() => {
  if (insertionTimeout !== null) {
    window.clearTimeout(insertionTimeout)
    insertionTimeout = null
  }
  stopObservingParagraphs()
})
</script>

<template>
  <ChromeWrapper>
    <div ref="articleContainerRef" class="article-container">
      <Article content-type="mock" />
    </div>

    <div v-if="showThanksDialog" class="reader-thanks-modal" role="dialog" aria-modal="true">
      <div class="reader-thanks-modal__backdrop" @click="onSkipThanks" />
      <section class="reader-thanks-modal__panel" aria-label="Thank editors">
        <div class="reader-thanks-modal__modes" role="tablist" aria-label="Response format">
          <button
            type="button"
            role="tab"
            class="reader-thanks-modal__mode-btn"
            :class="{
              'reader-thanks-modal__mode-btn--active': thanksInputMode === 'message',
            }"
            :aria-selected="thanksInputMode === 'message'"
            @click="thanksInputMode = 'message'"
          >
            free text
          </button>
          <button
            type="button"
            role="tab"
            class="reader-thanks-modal__mode-btn"
            :class="{
              'reader-thanks-modal__mode-btn--active': thanksInputMode === 'reason',
            }"
            :aria-selected="thanksInputMode === 'reason'"
            @click="thanksInputMode = 'reason'"
          >
            dropdown
          </button>
        </div>

        <p class="reader-thanks-modal__prompt">{{ dialogPrompt }}</p>

        <textarea
          v-if="thanksInputMode === 'message'"
          v-model="thanksMessage"
          class="reader-thanks-modal__textarea"
          rows="4"
          placeholder="Optional message"
        />

        <CdxField v-else class="reader-thanks-modal__reason-field">
          <template #label>How this article helped</template>
          <CdxSelect
            v-model:selected="thanksReason"
            default-label="Choose one (optional)"
            :menu-items="reasonOptions"
          />
        </CdxField>

        <div class="reader-thanks-modal__actions">
          <button type="button" class="reader-thanks-modal__btn" @click="onSkipThanks">skip</button>
          <button
            type="button"
            class="reader-thanks-modal__btn reader-thanks-modal__btn--primary"
            @click="onSendThanks"
          >
            send
          </button>
        </div>

        <p class="reader-thanks-modal__meta">
          <a
            href="#"
            class="reader-thanks-modal__meta-link"
            :title="thanksInfoTooltip"
            @click.prevent
          >
            who does my thanks go to?
          </a>
        </p>
      </section>
    </div>
  </ChromeWrapper>
</template>

<style scoped>
.article-container {
  padding: 0 var(--spacing-100);
}

.article-container :deep(.reader-thanks-callout) {
  margin: var(--spacing-100, 16px) 0;
  padding: var(--spacing-75, 12px);
  border: 1px solid var(--border-color-base, #a2a9b1);
  border-radius: var(--border-radius-base, 2px);
  background: var(--background-color-interactive-subtle, #f8f9fa);
  font-size: var(--font-size-small, 0.875rem);
  line-height: var(--line-height-medium, 1.6);
}

.article-container :deep(.reader-thanks-callout__link) {
  font-weight: var(--font-weight-bold, 700);
}

.article-container :deep(.reader-thanks-callout--dismissed) {
  opacity: 0.5;
  font-style: italic;
}

.reader-thanks-modal {
  position: fixed;
  inset: 0;
  z-index: 200;
  display: grid;
  place-items: end center;
  padding: var(--spacing-100, 16px);
}

.reader-thanks-modal__backdrop {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.4);
}

.reader-thanks-modal__panel {
  position: relative;
  width: min(100%, 36rem);
  padding: var(--spacing-125, 20px);
  border-radius: var(--border-radius-base, 2px);
  background: var(--background-color-base, #fff);
  box-shadow: var(--box-shadow-drop-medium);
}

.reader-thanks-modal__prompt {
  margin: 0 0 var(--spacing-100, 16px);
  font-size: var(--font-size-small, 0.875rem);
  line-height: var(--line-height-medium, 1.6);
}

.reader-thanks-modal__modes {
  display: inline-flex;
  gap: var(--spacing-25, 4px);
  margin-bottom: var(--spacing-100, 16px);
  padding: var(--spacing-25, 4px);
  border-radius: var(--border-radius-base, 2px);
  background: var(--background-color-interactive-subtle, #f8f9fa);
}

.reader-thanks-modal__mode-btn {
  border: 1px solid transparent;
  border-radius: var(--border-radius-base, 2px);
  background: transparent;
  color: var(--color-subtle, #54595d);
  font-size: var(--font-size-small, 0.875rem);
  padding: var(--spacing-35, 6px) var(--spacing-75, 12px);
  text-transform: lowercase;
  cursor: pointer;
}

.reader-thanks-modal__mode-btn--active {
  border-color: var(--border-color-base, #a2a9b1);
  background: var(--background-color-base, #fff);
  color: var(--color-base, #202122);
}

.reader-thanks-modal__textarea {
  box-sizing: border-box;
  width: 100%;
  min-height: 6rem;
  margin: 0;
  padding: var(--spacing-75, 12px);
  border: 1px solid var(--border-color-base, #a2a9b1);
  border-radius: var(--border-radius-base, 2px);
  font: inherit;
  color: var(--color-base, #202122);
  background: var(--background-color-base, #fff);
}

.reader-thanks-modal__reason-field {
  margin: 0;
}

.reader-thanks-modal__actions {
  display: flex;
  justify-content: flex-end;
  gap: var(--spacing-50, 8px);
  margin-top: var(--spacing-100, 16px);
}

.reader-thanks-modal__btn {
  padding: var(--spacing-50, 8px) var(--spacing-100, 16px);
  border: 1px solid var(--border-color-base, #a2a9b1);
  border-radius: var(--border-radius-base, 2px);
  background: var(--background-color-base, #fff);
  color: var(--color-base, #202122);
  text-transform: lowercase;
  cursor: pointer;
}

.reader-thanks-modal__btn--primary {
  border-color: var(--color-progressive, #36c);
  background: var(--color-progressive, #36c);
  color: var(--color-inverted, #fff);
}

.reader-thanks-modal__meta {
  margin: var(--spacing-75, 12px) 0 0;
  font-size: var(--font-size-x-small, 0.75rem);
}

.reader-thanks-modal__meta-link {
  color: var(--color-progressive, #36c);
}
</style>
