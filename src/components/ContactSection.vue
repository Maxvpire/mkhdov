<template>
  <section
    id="contact"
    ref="contactSection"
    class="contact-section"
    :class="{ 'is-visible': isVisible }"
  >
    <div class="contact-container">
      <aside class="contact-info reveal-panel">
        <p class="contact-kicker reveal-item">Contact</p>
        <h2 class="reveal-item">Let's build something useful together.</h2>
        <p class="contact-copy reveal-item">
          Send a live message here, or use the direct links if you prefer email
          or social channels.
        </p>

        <div class="contact-methods reveal-item">
          <a class="contact-method" href="mailto:mkhdov@yahoo.com">
            <span class="method-icon" aria-hidden="true">
              <svg viewBox="0 0 24 24" fill="none">
                <path d="M4 6.5h16v11H4v-11Z" />
                <path d="m4 7 8 6 8-6" />
              </svg>
            </span>
            <span>
              <strong>Email</strong>
              <small>mkhdov@yahoo.com</small>
            </span>
          </a>

          <div class="contact-method">
            <span class="method-icon" aria-hidden="true">
              <svg viewBox="0 0 24 24" fill="none">
                <path d="M12 21s7-4.7 7-11a7 7 0 1 0-14 0c0 6.3 7 11 7 11Z" />
                <path d="M12 12.5a2.5 2.5 0 1 0 0-5 2.5 2.5 0 0 0 0 5Z" />
              </svg>
            </span>
            <span>
              <strong>Location</strong>
              <small>Uzbekistan, remote friendly</small>
            </span>
          </div>

          <div class="contact-method">
            <span class="method-icon" aria-hidden="true">
              <svg viewBox="0 0 24 24" fill="none">
                <path d="M12 6v6l4 2" />
                <path d="M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z" />
              </svg>
            </span>
            <span>
              <strong>Response</strong>
              <small>I reply as soon as I am available</small>
            </span>
          </div>
        </div>

        <form class="visitor-card reveal-item" @submit.prevent="saveContactInfo">
          <label>
            <span>Your name</span>
            <input
              v-model="guestName"
              type="text"
              autocomplete="name"
              placeholder="How should I call you?"
            />
          </label>

          <label>
            <span>Email address</span>
            <input
              v-model="guestEmail"
              type="email"
              autocomplete="email"
              placeholder="you@example.com"
            />
          </label>

          <button v-if="profileNeedsSave" class="profile-submit" type="submit" :disabled="savingProfile">
            <span v-if="savingProfile" class="mini-spinner"></span>
            <span v-else>Send contact info</span>
          </button>
        </form>

        <div class="social-links reveal-item" aria-label="Social links">
          <a href="https://linkedin.com/in/mkhdov" target="_blank" rel="noopener noreferrer">
            LinkedIn
          </a>
          <a href="https://github.com/mkhdov" target="_blank" rel="noopener noreferrer">
            GitHub
          </a>
          <a href="mailto:mkhdov@yahoo.com">
            Email
          </a>
        </div>
      </aside>

      <div class="chat-panel reveal-panel reveal-panel--chat">
        <div class="chat-header">
          <div>
            <p class="chat-label">Live chat</p>
            <h3>Message me directly</h3>
          </div>
          <span class="chat-status">Online inbox</span>
        </div>

        <div class="chat-wrapper">
          <div class="chat-messages" ref="messagesContainer">
            <div v-if="loading" class="chat-state-msg">
              Loading messages...
            </div>

            <div v-else-if="messages.length === 0" class="chat-state-msg empty-state">
              Start the conversation.
            </div>

            <template v-else>
              <div
                v-for="msg in messages"
                :key="msg.id"
                :class="['message-bubble', msg.sender_type === 'guest' ? 'guest' : 'admin']"
              >
                <div class="bubble-content">{{ msg.content }}</div>
                <div class="bubble-time">
                  {{ new Date(msg.created_at).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }) }}
                </div>
              </div>
            </template>
          </div>

          <form class="chat-input-area" @submit.prevent="sendMessage">
            <input
              v-model="newMessage"
              type="text"
              placeholder="Type your message..."
              :disabled="sending"
              class="chat-input"
            />
            <button type="submit" class="btn-send" title="Send message" :disabled="!newMessage.trim() || sending">
              <svg
                v-if="!sending"
                width="20"
                height="20"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
              >
                <line x1="22" y1="2" x2="11" y2="13"></line>
                <polygon points="22 2 15 22 11 13 2 9 22 2"></polygon>
              </svg>
              <span v-else class="spinner"></span>
            </button>
          </form>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { computed, nextTick, onMounted, onUnmounted, ref } from 'vue'
import { supabase } from '../lib/supabase'

interface Message {
  id: string
  conversation_id: string
  sender_type: 'guest' | 'admin'
  content: string
  created_at: string
}

interface Conversation {
  id: string
  visitor_id: string
  guest_name: string | null
  guest_email: string | null
}

const VISITOR_KEY = 'mkhdov_visitor_id'
const VISITOR_NAME_KEY = 'mkhdov_guest_name'
const VISITOR_EMAIL_KEY = 'mkhdov_guest_email'
const initialGuestName = localStorage.getItem(VISITOR_NAME_KEY) ?? ''
const initialGuestEmail = localStorage.getItem(VISITOR_EMAIL_KEY) ?? ''

const messages = ref<Message[]>([])
const newMessage = ref('')
const guestName = ref(initialGuestName)
const guestEmail = ref(initialGuestEmail)
const savedGuestName = ref(initialGuestName.trim())
const savedGuestEmail = ref(initialGuestEmail.trim())
const loading = ref(true)
const sending = ref(false)
const savingProfile = ref(false)
const messagesContainer = ref<HTMLElement | null>(null)
const contactSection = ref<HTMLElement | null>(null)
const isVisible = ref(false)

let visitorId = ''
let currentConversationId = ''
let realtimeSubscription: ReturnType<typeof supabase.channel> | null = null
let observer: IntersectionObserver | null = null

const profileNeedsSave = computed(() => {
  return (
    guestName.value.trim() !== savedGuestName.value ||
    guestEmail.value.trim() !== savedGuestEmail.value
  )
})

function getOrCreateVisitorId() {
  let id = localStorage.getItem(VISITOR_KEY)

  if (!id) {
    id = crypto.randomUUID
      ? crypto.randomUUID()
      : Math.random().toString(36).substring(2) + Date.now().toString(36)
    localStorage.setItem(VISITOR_KEY, id)
  }

  return id
}

function profilePayload() {
  return {
    guest_name: guestName.value.trim() || null,
    guest_email: guestEmail.value.trim() || null,
  }
}

async function fetchOrCreateConversation() {
  const { data: convData, error: convError } = await supabase
    .from('conversations')
    .select('id, visitor_id, guest_name, guest_email')
    .eq('visitor_id', visitorId)
    .single()

  if (convError && convError.code !== 'PGRST116') {
    console.error('Error fetching conversation:', convError)
    return null
  }

  if (convData) {
    hydrateProfile(convData as Conversation)
    return convData.id
  }

  const { data: newConv, error: createError } = await supabase
    .from('conversations')
    .insert([
      {
        visitor_id: visitorId,
        ...profilePayload(),
      },
    ])
    .select('id')
    .single()

  if (createError) {
    console.error('Error creating conversation:', createError)
    return null
  }

  return newConv?.id ?? null
}

function hydrateProfile(conversation: Conversation) {
  if (!guestName.value && conversation.guest_name) {
    guestName.value = conversation.guest_name
    localStorage.setItem(VISITOR_NAME_KEY, conversation.guest_name)
  }

  if (!guestEmail.value && conversation.guest_email) {
    guestEmail.value = conversation.guest_email
    localStorage.setItem(VISITOR_EMAIL_KEY, conversation.guest_email)
  }

  savedGuestName.value = guestName.value.trim()
  savedGuestEmail.value = guestEmail.value.trim()
}

async function updateConversationProfile() {
  localStorage.setItem(VISITOR_NAME_KEY, guestName.value.trim())
  localStorage.setItem(VISITOR_EMAIL_KEY, guestEmail.value.trim())

  if (!currentConversationId) return false

  const { error } = await supabase
    .from('conversations')
    .update(profilePayload())
    .eq('id', currentConversationId)

  if (error) {
    console.error('Error updating conversation profile:', error)
    return false
  }

  savedGuestName.value = guestName.value.trim()
  savedGuestEmail.value = guestEmail.value.trim()
  return true
}

async function saveContactInfo() {
  savingProfile.value = true

  if (!currentConversationId) {
    const cid = await fetchOrCreateConversation()
    if (cid) {
      currentConversationId = cid
      subscribeToMessages()
    }
  }

  await updateConversationProfile()
  savingProfile.value = false
}

async function loadMessages() {
  if (!currentConversationId) {
    loading.value = false
    return
  }

  const { data, error } = await supabase
    .from('messages')
    .select('*')
    .eq('conversation_id', currentConversationId)
    .order('created_at', { ascending: true })

  if (error) {
    console.error('Error fetching messages:', error)
  } else {
    messages.value = (data ?? []) as Message[]
    scrollToBottom()
  }

  loading.value = false
}

function subscribeToMessages() {
  if (!currentConversationId) return

  if (realtimeSubscription) {
    supabase.removeChannel(realtimeSubscription)
  }

  realtimeSubscription = supabase
    .channel(`messages_channel_${currentConversationId}`)
    .on(
      'postgres_changes',
      {
        event: 'INSERT',
        schema: 'public',
        table: 'messages',
        filter: `conversation_id=eq.${currentConversationId}`,
      },
      (payload) => {
        const incomingMessage = payload.new as Message

        if (!messages.value.some((message) => message.id === incomingMessage.id)) {
          messages.value.push(incomingMessage)
          scrollToBottom()
        }
      }
    )
    .subscribe()
}

async function sendMessage() {
  const content = newMessage.value.trim()
  if (!content) return

  sending.value = true

  if (!currentConversationId) {
    const cid = await fetchOrCreateConversation()

    if (cid) {
      currentConversationId = cid
      subscribeToMessages()
    } else {
      console.error('Failed to initialize conversation')
      sending.value = false
      return
    }
  }

  await updateConversationProfile()

  newMessage.value = ''

  const { data, error } = await supabase
    .from('messages')
    .insert([
      {
        conversation_id: currentConversationId,
        sender_type: 'guest',
        content,
      },
    ])
    .select('*')
    .single()

  if (error) {
    console.error('Error sending message:', error)
    newMessage.value = content
  } else if (data && !messages.value.some((message) => message.id === data.id)) {
    messages.value.push(data as Message)
    scrollToBottom()
  }

  sending.value = false
}

function scrollToBottom() {
  nextTick(() => {
    if (messagesContainer.value) {
      messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
    }
  })
}

onMounted(async () => {
  const section = contactSection.value
  if (!section || !('IntersectionObserver' in window)) {
    isVisible.value = true
  } else {
    observer = new IntersectionObserver(
      ([entry]) => {
        if (!entry.isIntersecting) return

        isVisible.value = true
        observer?.disconnect()
        observer = null
      },
      {
        rootMargin: '0px 0px -16% 0px',
        threshold: 0.18,
      }
    )

    observer.observe(section)
  }

  visitorId = getOrCreateVisitorId()
  const cid = await fetchOrCreateConversation()

  if (cid) {
    currentConversationId = cid
    await loadMessages()
    subscribeToMessages()
  } else {
    loading.value = false
  }
})

onUnmounted(() => {
  observer?.disconnect()

  if (realtimeSubscription) {
    supabase.removeChannel(realtimeSubscription)
  }
})
</script>

<style scoped>
.contact-section {
  scroll-margin-top: 88px;
  padding: 100px 40px;
  background:
    linear-gradient(180deg, #ffffff 0%, #f8fafc 45%, #ffffff 100%);
}

.contact-container {
  display: grid;
  grid-template-columns: minmax(300px, 0.9fr) minmax(360px, 1.1fr);
  gap: 28px;
  width: 100%;
  max-width: 1180px;
  margin: 0 auto;
  align-items: stretch;
}

.contact-info,
.chat-panel {
  border: 1px solid rgba(108, 99, 255, 0.13);
  border-radius: 24px;
  background: rgba(255, 255, 255, 0.92);
  box-shadow: 0 20px 50px rgba(15, 23, 42, 0.07);
}

.reveal-panel,
.reveal-item {
  opacity: 0;
  transition:
    opacity 0.75s cubic-bezier(0.16, 1, 0.3, 1),
    transform 0.75s cubic-bezier(0.16, 1, 0.3, 1);
}

.reveal-panel {
  transform: translateY(34px) scale(0.98);
}

.reveal-panel--chat {
  transform: translateY(34px) translateX(18px) scale(0.98);
}

.reveal-item {
  transform: translateY(18px);
}

.contact-section.is-visible .reveal-panel,
.contact-section.is-visible .reveal-item {
  opacity: 1;
  transform: none;
}

.contact-section.is-visible .contact-info {
  transition-delay: 0.02s;
}

.contact-section.is-visible .reveal-panel--chat {
  transition-delay: 0.14s;
}

.contact-section.is-visible .contact-kicker {
  transition-delay: 0.14s;
}

.contact-section.is-visible .contact-info h2 {
  transition-delay: 0.2s;
}

.contact-section.is-visible .contact-copy {
  transition-delay: 0.27s;
}

.contact-section.is-visible .contact-methods {
  transition-delay: 0.34s;
}

.contact-section.is-visible .visitor-card {
  transition-delay: 0.42s;
}

.contact-section.is-visible .social-links {
  transition-delay: 0.5s;
}

.contact-info {
  display: flex;
  flex-direction: column;
  gap: 22px;
  padding: 34px;
}

.contact-kicker,
.chat-label {
  margin: 0;
  color: #6c63ff;
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  font-weight: 800;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.contact-info h2 {
  margin: 0;
  color: #1a1a2e;
  font-family: 'Space Grotesk', sans-serif;
  font-size: clamp(30px, 4vw, 48px);
  line-height: 1.05;
}

.contact-copy {
  margin: 0;
  color: #64748b;
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  line-height: 1.7;
}

.contact-methods {
  display: grid;
  gap: 12px;
}

.contact-method {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 14px;
  border: 1px solid #e8edf5;
  border-radius: 18px;
  color: inherit;
  background: #ffffff;
  text-decoration: none;
  transform: translateY(0);
  transition:
    border-color 0.2s,
    box-shadow 0.2s,
    transform 0.2s;
}

.contact-section.is-visible .contact-method:nth-child(1) {
  animation: contactCardIn 0.62s cubic-bezier(0.16, 1, 0.3, 1) 0.42s both;
}

.contact-section.is-visible .contact-method:nth-child(2) {
  animation: contactCardIn 0.62s cubic-bezier(0.16, 1, 0.3, 1) 0.48s both;
}

.contact-section.is-visible .contact-method:nth-child(3) {
  animation: contactCardIn 0.62s cubic-bezier(0.16, 1, 0.3, 1) 0.54s both;
}

.contact-method:hover {
  border-color: rgba(108, 99, 255, 0.22);
  box-shadow: 0 10px 24px rgba(15, 23, 42, 0.05);
  transform: translateY(-2px);
}

.method-icon {
  display: grid;
  place-items: center;
  width: 42px;
  height: 42px;
  flex-shrink: 0;
  border-radius: 14px;
  color: #6c63ff;
  background: rgba(108, 99, 255, 0.09);
}

.method-icon svg {
  width: 21px;
  height: 21px;
  stroke: currentColor;
  stroke-width: 2;
  stroke-linecap: round;
  stroke-linejoin: round;
}

.contact-method strong,
.contact-method small {
  display: block;
}

.contact-method strong {
  color: #1a1a2e;
  font-size: 14px;
}

.contact-method small {
  margin-top: 2px;
  color: #64748b;
  font-size: 13px;
}

.visitor-card {
  display: grid;
  gap: 12px;
  padding: 16px;
  border: 1px solid rgba(108, 99, 255, 0.13);
  border-radius: 20px;
  background: #f8fafc;
}

.visitor-card label {
  display: grid;
  gap: 7px;
  color: #64748b;
  font-size: 13px;
  font-weight: 700;
}

.visitor-card input {
  width: 100%;
  min-width: 0;
  border: 1px solid #dbe3ef;
  border-radius: 14px;
  outline: 0;
  padding: 12px 13px;
  color: #1e293b;
  background: #ffffff;
  font-size: 14px;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.visitor-card input:focus {
  border-color: #6c63ff;
  box-shadow: 0 0 0 4px rgba(108, 99, 255, 0.1);
}

.profile-submit {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-height: 44px;
  border: 0;
  border-radius: 14px;
  color: #ffffff;
  background: #6c63ff;
  font: inherit;
  font-size: 14px;
  font-weight: 800;
  cursor: pointer;
  transition: transform 0.2s, background 0.2s, opacity 0.2s;
}

.profile-submit:hover:not(:disabled) {
  background: #5a52d5;
  transform: translateY(-1px);
}

.profile-submit:disabled {
  cursor: not-allowed;
  opacity: 0.72;
}

.mini-spinner {
  width: 18px;
  height: 18px;
  border: 2px solid rgba(255, 255, 255, 0.35);
  border-top-color: #ffffff;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

.social-links {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: auto;
}

.social-links a {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-height: 38px;
  padding: 9px 14px;
  border: 1px solid rgba(108, 99, 255, 0.18);
  border-radius: 14px;
  color: #6c63ff;
  background: rgba(108, 99, 255, 0.07);
  font-size: 13px;
  font-weight: 800;
  text-decoration: none;
  transition: transform 0.2s, background 0.2s;
}

.social-links a:hover {
  transform: translateY(-1px);
  background: rgba(108, 99, 255, 0.11);
}

.chat-panel {
  display: flex;
  min-height: 680px;
  overflow: hidden;
  flex-direction: column;
}

.chat-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 18px;
  padding: 26px 30px 22px;
  border-bottom: 1px solid rgba(108, 99, 255, 0.12);
  background: #ffffff;
}

.chat-header h3 {
  margin: 4px 0 0;
  color: #1a1a2e;
  font-family: 'Space Grotesk', sans-serif;
  font-size: 25px;
}

.chat-status {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  border-radius: 999px;
  color: #047857;
  background: rgba(16, 185, 129, 0.1);
  font-size: 12px;
  font-weight: 800;
  white-space: nowrap;
}

.chat-status::before {
  content: '';
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #10b981;
}

.chat-wrapper {
  display: flex;
  flex: 1;
  min-height: 0;
  flex-direction: column;
  background: #f8fafc;
}

.chat-messages {
  display: flex;
  flex: 1;
  min-height: 0;
  flex-direction: column;
  gap: 16px;
  overflow-y: auto;
  padding: 26px 30px;
  scroll-behavior: smooth;
}

.chat-state-msg {
  margin: auto;
  color: #94a3b8;
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  text-align: center;
}

.message-bubble {
  display: flex;
  max-width: 76%;
  flex-direction: column;
  gap: 4px;
}

.message-bubble.guest {
  align-self: flex-end;
}

.message-bubble.admin {
  align-self: flex-start;
}

.bubble-content {
  padding: 12px 18px;
  border-radius: 18px;
  font-family: 'Inter', sans-serif;
  font-size: 15px;
  line-height: 1.5;
  overflow-wrap: anywhere;
}

.message-bubble.guest .bubble-content {
  color: #ffffff;
  border-bottom-right-radius: 4px;
  background: linear-gradient(135deg, #6c63ff 0%, #818cf8 100%);
  box-shadow: 0 10px 24px rgba(108, 99, 255, 0.18);
}

.message-bubble.admin .bubble-content {
  color: #1e293b;
  border: 1px solid #e2e8f0;
  border-bottom-left-radius: 4px;
  background: #ffffff;
}

.bubble-time {
  padding: 0 4px;
  color: #94a3b8;
  font-family: 'Inter', sans-serif;
  font-size: 11px;
}

.message-bubble.guest .bubble-time {
  text-align: right;
}

.chat-input-area {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 20px 30px;
  border-top: 1px solid #e2e8f0;
  background: #ffffff;
}

.chat-input {
  flex: 1;
  min-width: 0;
  border: 1px solid #cbd5e1;
  border-radius: 15px;
  outline: none;
  padding: 14px 16px;
  color: #1e293b;
  background: #f8fafc;
  font-family: 'Inter', sans-serif;
  font-size: 15px;
  transition: border-color 0.2s, box-shadow 0.2s, background 0.2s;
}

.chat-input:focus {
  border-color: #6c63ff;
  background: #ffffff;
  box-shadow: 0 0 0 4px rgba(108, 99, 255, 0.1);
}

.chat-input:disabled {
  cursor: not-allowed;
  background: #f1f5f9;
}

.btn-send {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 50px;
  height: 50px;
  flex-shrink: 0;
  border: none;
  border-radius: 16px;
  color: #ffffff;
  background: #6c63ff;
  cursor: pointer;
  transition: transform 0.2s, background 0.2s;
}

.btn-send:hover:not(:disabled) {
  background: #5a52d5;
  transform: translateY(-1px);
}

.btn-send:active:not(:disabled) {
  transform: scale(0.96);
}

.btn-send:disabled {
  cursor: not-allowed;
  background: #cbd5e1;
}

.spinner {
  width: 20px;
  height: 20px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top-color: #ffffff;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

@keyframes contactCardIn {
  from {
    opacity: 0;
    transform: translateY(16px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (max-width: 900px) {
  .contact-container {
    grid-template-columns: 1fr;
  }

  .chat-panel {
    min-height: 600px;
  }
}

@media (max-width: 640px) {
  .contact-section {
    padding: 70px 18px;
  }

  .contact-info,
  .chat-panel {
    border-radius: 20px;
  }

  .contact-info {
    padding: 24px;
  }

  .chat-header {
    align-items: flex-start;
    flex-direction: column;
    padding: 22px 22px 18px;
  }

  .chat-messages {
    padding: 20px;
  }

  .message-bubble {
    max-width: 88%;
  }

  .chat-input-area {
    padding: 16px;
  }
}

@media (prefers-reduced-motion: reduce) {
  .reveal-panel,
  .reveal-item,
  .contact-method,
  .social-links a,
  .profile-submit,
  .btn-send {
    opacity: 1;
    transform: none;
    animation: none !important;
    transition: none;
  }
}
</style>
