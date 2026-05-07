<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from 'vue'

definePage({
  meta: {
    title: 'Reader reactions',
    description:
      'Once the reader has been on this article for 15 seconds, it shows them a non-intrusive inline option to leave a reaction for the editor(s).',
  },
})

import Article from '@/components/Article.vue'
import ChromeWrapper from '@/components/ChromeWrapper.vue'

const INSERT_DELAY_MS = 5000

const articleContainerRef = ref<HTMLElement | null>(null)

let insertionTimeout: ReturnType<typeof window.setTimeout> | null = null
let paragraphObserver: MutationObserver | null = null

function sendReaction(callout: HTMLElement, button: HTMLElement) {
  callout.textContent = ''
  const message = document.createElement('span')
  message.textContent = 'Thanks for reacting!'
  
  const meta = document.createElement('p')
  meta.className = 'reader-thanks-callout__meta'

  const link = document.createElement('a')
  link.href = '#'
  link.className = 'reader-thanks-callout__meta-link'
  link.textContent = 'Who did my reaction go to?'
  link.title = "This could link to Ilana's 'behind the curtain' page"
  link.addEventListener('click', (e) => e.preventDefault())
  meta.append(link)

  callout.append(message, meta)
  callout.classList.add('reader-thanks-callout--dismissed')
}

function getReactionEmoji(name: 'thumbs-up' | 'heart' | 'fire' | 'clap'): string {
  const emojiMap = {
    'thumbs-up': '👍',
    'heart': '❤️',
    'fire': '🔥',
    'clap': '👏',
  }
  return emojiMap[name]
}

function buildReaderThanksCallout(): HTMLElement {
  const callout = document.createElement('p')
  callout.className = 'reader-thanks-callout'

  const intro = document.createElement('span')
  intro.textContent = 'Did this article help you? '

  const prompt = document.createElement('span')
  prompt.className = 'reader-thanks-callout__link'
  prompt.textContent = 'Send a reaction to the editors →'

  const reactions = document.createElement('span')
  reactions.className = 'reader-thanks-callout__reactions'

  const reactionOptions = [
    { icon: 'thumbs-up' as const, label: 'Thumbs up' },
    { icon: 'heart' as const, label: 'Heart' },
    { icon: 'fire' as const, label: 'Fire' },
    { icon: 'clap' as const, label: 'Clap' },
  ]

  for (const option of reactionOptions) {
    const button = document.createElement('button')
    button.type = 'button'
    button.className = 'reader-thanks-callout__reaction-btn'
    button.textContent = getReactionEmoji(option.icon)
    button.setAttribute('aria-label', option.label)
    button.addEventListener('click', () => {
      sendReaction(callout, button)
    })
    reactions.append(button)
  }

  const meta = document.createElement('p')
  meta.className = 'reader-thanks-callout__meta'

  const link = document.createElement('a')
  link.href = '#'
  link.className = 'reader-thanks-callout__meta-link'
  link.textContent = 'Who does my reaction go to?'
  link.title = "This could link to ilana's 'behind the curtain' page."
  link.addEventListener('click', (e) => e.preventDefault())
  meta.append(link)

  callout.append(intro, prompt, reactions, meta)
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
  </ChromeWrapper>
</template>

<style scoped>
.article-container {
  padding: 0 var(--spacing-100);
}

.article-container :deep(.reader-thanks-callout) {
  margin: var(--spacing-50, 8px) 0;
  padding: var(--spacing-50, 8px);
  border: 1px solid var(--border-color-base, #a2a9b1);
  border-radius: var(--border-radius-base, 2px);
  background: var(--background-color-interactive-subtle, #f8f9fa);
  font-size: var(--font-size-small, 0.875rem);
  line-height: 1.2;
}

.article-container :deep(.reader-thanks-callout__link) {
  margin-right: var(--spacing-35, 6px);
  font-weight: var(--font-weight-normal, 400);
}

.article-container :deep(.reader-thanks-callout__reactions) {
  display: inline-flex;
  align-items: center;
  gap: var(--spacing-25, 4px);
}

.article-container :deep(.reader-thanks-callout__reaction-btn) {
  border: 0;
  padding: 0;
  background: transparent;
  font-size: 1rem;
  line-height: 1;
  filter: grayscale(1) contrast(0.8) brightness(1.05);
  cursor: pointer;
}

.article-container :deep(.reader-thanks-callout__reaction-btn:hover) {
  filter: grayscale(0.3) contrast(1) brightness(1);
  transform: scale(1.08);
}

.article-container :deep(.reader-thanks-callout__meta) {
  margin: var(--spacing-50, 8px) 0 0;
  font-size: var(--font-size-x-small, 0.75rem);
}

.article-container :deep(.reader-thanks-callout__meta-link) {
  color: var(--color-progressive, #36c);
  cursor: pointer;
}

.article-container :deep(.reader-thanks-callout--dismissed) {
  opacity: 0.5;
  font-style: italic;
}
</style>
