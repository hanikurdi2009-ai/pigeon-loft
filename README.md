# pigeon-loft
A messaging app with calls, store, games, and AI features
import { useEffect, useRef, useState, type CSSProperties, type FormEvent } from 'react'
import { AssistantPage, GamesPage, NewsPage, StorePage } from './FeaturePages'
import { initialProducts, type GameTemplate, type StoreOrder, type StoreProduct } from './FeatureModels'
import './App.css'

type Profile = {
  name: string
  username: string
  birthday: string
  photo: string
}

type Message = {
  id: number
  text: string
  time: string
  mine: boolean
  mediaUrl?: string
  mediaType?: 'image' | 'video' | 'audio' | 'document'
  mediaName?: string
  pollOptions?: string[]
  contentKind?: 'gif' | 'sticker'
}

type Chat = {
  id: string
  name: string
  username: string
  preview: string
  time: string
  unread: number
  photo: string
  tint: string
  online: boolean
  group?: boolean
  messages: Message[]
}

type Page = 'chats' | 'calls' | 'settings' | 'ai' | 'store' | 'games' | 'news'
type GameUsage = { date: string; count: number }
type SettingsPrefs = { notifications: boolean; readReceipts: boolean; allowCalls: boolean; autoDownload: boolean }
type CallPhase = 'finding' | 'answered' | 'missed'

function readLocal<T>(key: string, fallback: T): T {
  try {
    const saved = localStorage.getItem(key)
    return saved ? JSON.parse(saved) as T : fallback
  } catch {
    return fallback
  }
}

const defaultChats: Chat[] = [
  {
    id: 'maya', name: 'Maya Chen', username: '@mayachen', preview: 'That little place by the water sounds perfect.', time: '9:41', unread: 2, photo: '', tint: 'coral', online: true,
    messages: [
      { id: 1, text: 'Found a quiet spot for Saturday. Sending you the pin now.', time: '9:36', mine: false },
      { id: 2, text: 'Oh this looks lovely. Is it walkable from the station?', time: '9:38', mine: true },
      { id: 3, text: 'About ten minutes, and there is a little bookshop on the way.', time: '9:39', mine: false },
      { id: 4, text: 'That little place by the water sounds perfect.', time: '9:41', mine: false },
    ],
  },
  {
    id: 'sam', name: 'Sam Rivera', username: '@samrivera', preview: 'Voice note · 0:18', time: '8:57', unread: 0, photo: '', tint: 'mint', online: false,
    messages: [
      { id: 1, text: 'Are we still on for the studio visit?', time: '8:48', mine: false },
      { id: 2, text: 'Absolutely. I will meet you outside at ten.', time: '8:52', mine: true },
      { id: 3, text: 'Voice note · 0:18', time: '8:57', mine: false },
    ],
  },
  {
    id: 'weekend', name: 'Weekend people', username: '4 members', preview: 'Noah: I can bring the speaker', time: 'Yesterday', unread: 0, photo: '', tint: 'blue', online: false, group: true,
    messages: [
      { id: 1, text: 'Saturday picnic is happening. Bring something to share!', time: 'Yesterday', mine: false },
      { id: 2, text: 'I can bring the speaker.', time: 'Yesterday', mine: false },
    ],
  },
]

function Icon({ name, size = 20 }: { name: string; size?: number }) {
  const paths: Record<string, string> = {
    chat: 'M20 11.5a7.5 7.5 0 0 1-7.5 7.5H5l-3 3v-6.5A7.5 7.5 0 1 1 20 11.5Z',
    plus: 'M12 5v14M5 12h14',
    phone: 'M22 16.9v3a2 2 0 0 1-2.2 2 19.8 19.8 0 0 1-8.6-3.1 19.4 19.4 0 0 1-6-6A19.8 19.8 0 0 1 2.1 4.2 2 2 0 0 1 4.1 2h3a2 2 0 0 1 2 1.7l.5 2.8a2 2 0 0 1-.6 1.8L7.7 9.6a16 16 0 0 0 6 6l1.3-1.3a2 2 0 0 1 1.8-.6l2.8.5a2 2 0 0 1 1.7 2.7Z',
    settings: 'M12.22 2h-.44a2 2 0 0 0-1.99 1.72l-.13.96a2 2 0 0 1-1.09 1.48l-.87.5a2 2 0 0 1-1.83.03l-.85-.46a2 2 0 0 0-2.7.73l-.22.38a2 2 0 0 0 .73 2.73l.82.48a2 2 0 0 1 1 1.45v1a2 2 0 0 1-1 1.45l-.82.48a2 2 0 0 0-.73 2.73l.22.38a2 2 0 0 0 2.7.73l.85-.46a2 2 0 0 1 1.83.03l.87.5a2 2 0 0 1 1.09 1.48l.13.96A2 2 0 0 0 11.78 22h.44a2 2 0 0 0 1.99-1.72l.13-.96a2 2 0 0 1 1.09-1.48l.87-.5a2 2 0 0 1 1.83-.03l.85.46a2 2 0 0 0 2.7-.73l.22-.38a2 2 0 0 0-.73-2.73l-.82-.48a2 2 0 0 1-1-1.45v-1a2 2 0 0 1 1-1.45l.82-.48a2 2 0 0 0 .73-2.73l-.22-.38a2 2 0 0 0-2.7-.73l-.85.46a2 2 0 0 1-1.83-.03l-.87-.5a2 2 0 0 1-1.09-1.48l-.13-.96A2 2 0 0 0 12.22 2Z M12 8a4 4 0 1 0 0 8 4 4 0 0 0 0-8Z',
    search: 'm21 21-4.3-4.3M10.8 18a7.2 7.2 0 1 0 0-14.4 7.2 7.2 0 0 0 0 14.4Z',
    send: 'm22 2-7 20-4-9-9-4 20-7ZM22 2 11 13',
    back: 'm15 18-6-6 6-6M9 12h12',
    sun: 'M12 3v2m0 14v2M5.6 5.6 7 7m10 10 1.4 1.4M3 12h2m14 0h2M5.6 18.4 7 17m10-10 1.4-1.4M16 12a4 4 0 1 1-8 0 4 4 0 0 1 8 0Z',
    moon: 'M20.9 13A9 9 0 0 1 11 3.1 9 9 0 1 0 20.9 13Z',
    close: 'M18 6 6 18M6 6l12 12',
    user: 'M20 21a8 8 0 0 0-16 0m8-10a4 4 0 1 0 0-8 4 4 0 0 0 0 8Z',
    check: 'm5 12 4 4L19 6',
    spark: 'm12 3 1.9 5.8L20 11l-6.1 2.2L12 19l-2-5.8L4 11l6-2.2L12 3Zm7 11 .9 2.1L22 17l-2.1.9L19 20l-.9-2.1L16 17l2.1-.9L19 14Z',
    up: 'm18 15-6-6-6 6',
    down: 'm6 9 6 6 6-6',
    mic: 'M12 2a3 3 0 0 0-3 3v7a3 3 0 0 0 6 0V5a3 3 0 0 0-3-3Zm7 10a7 7 0 0 1-14 0m7 7v3m-4 0h8',
    phoneOff: 'm3 3 18 18M10.6 10.6a2 2 0 0 0 2.8 2.8M8.3 5.2A15.8 15.8 0 0 0 7.7 9.6a16 16 0 0 0 6.7 6.7m3.7-1.2a2 2 0 0 1 1.8-.6l2.8.5a2 2 0 0 1 1.7 2.7v3a2 2 0 0 1-2.2 2 19.8 19.8 0 0 1-8.6-3.1',
    store: 'M3 9h18l-1.5 12h-15L3 9Zm3 0 1-6h10l1 6m-9 4v4m6-4v4',
    games: 'M4 4h7v7H4zM13 4h7v7h-7zM4 13h7v7H4zM13 13h7v7h-7z',
    news: 'M4 4h16v16H4zM8 8h8M8 12h8M8 16h5',
    droplet: 'M12 22a7 7 0 0 0 7-7c0-4-7-13-7-13S5 11 5 15a7 7 0 0 0 7 7Z',
  }

  return (
    <svg aria-hidden="true" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="1.8" strokeLinecap="round" strokeLinejoin="round">
      <path d={paths[name] ?? paths.chat} />
    </svg>
  )
}

function readProfile(): Profile | null {
  try {
    const saved = localStorage.getItem('pigeon-profile')
    return saved ? JSON.parse(saved) as Profile : null
  } catch {
    return null
  }
}

function Avatar({ chat, size = 'normal' }: { chat: Pick<Chat, 'name' | 'photo' | 'tint'>; size?: 'small' | 'normal' | 'large' }) {
  return (
    <div className={`avatar avatar-${chat.tint} avatar-${size}`}>
      {chat.photo ? <img src={chat.photo} alt="" /> : chat.tint === 'ai' ? <Icon name="spark" size={size === 'large' ? 24 : 19} /> : chat.name.split(' ').map((part) => part[0]).slice(0, 2).join('')}
    </div>
  )
}

function App() {
  const [profile, setProfile] = useState<Profile | null>(readProfile)
  const [stage, setStage] = useState<'sign-in' | 'verify' | 'profile' | 'app'>(() => readProfile() ? 'app' : 'sign-in')
  const [identifier, setIdentifier] = useState('')
  const [code, setCode] = useState('')
  const [error, setError] = useState('')
  const [photo, setPhoto] = useState('')
  const [chats, setChats] = useState<Chat[]>(defaultChats)
  const [selectedId, setSelectedId] = useState('maya')
  const [page, setPage] = useState<Page>('chats')
  const [search, setSearch] = useState('')
  const [message, setMessage] = useState('')
  const [flight, setFlight] = useState<{ id: number; messageId: number; kind: 'note' | 'photo' | 'video'; fromX: number; fromY: number; toX: number; toY: number } | null>(null)
  const [chatFilter, setChatFilter] = useState<'all' | 'groups'>('all')
  const [contactType, setContactType] = useState<'person' | 'group'>('person')
  const attachmentRef = useRef<HTMLInputElement>(null)
  const [theme, setTheme] = useState<'light' | 'dark' | 'liquid'>(() => {
    const saved = localStorage.getItem('pigeon-theme-v3')
    return saved === 'light' || saved === 'dark' || saved === 'liquid' ? saved : 'liquid'
  })
  const [contactOpen, setContactOpen] = useState(false)
  const [profileEditOpen, setProfileEditOpen] = useState(false)
  const [toolsOpen, setToolsOpen] = useState(false)
  const [picker, setPicker] = useState<'emoji' | 'gifs' | 'stickers' | ''>('')
  const [pollOpen, setPollOpen] = useState(false)
  const [recordingVoice, setRecordingVoice] = useState(false)
  const [pollVotes, setPollVotes] = useState<Record<number, string>>({})
  const [notice, setNotice] = useState('')
  const [callCount, setCallCount] = useState(0)
  const [activeCall, setActiveCall] = useState<Chat | null>(null)
  const [callPhase, setCallPhase] = useState<CallPhase>('finding')
  const [cutscene, setCutscene] = useState(false)
  const [microphoneEnabled, setMicrophoneEnabled] = useState(false)
  const [callSeconds, setCallSeconds] = useState(0)
  const [scrollState, setScrollState] = useState({ up: false, down: false })
  const [products, setProducts] = useState<StoreProduct[]>(() => readLocal('pigeon-market-products', initialProducts))
  const [wishlistIds, setWishlistIds] = useState<string[]>(() => readLocal('pigeon-wishlist', []))
  const [orders, setOrders] = useState<StoreOrder[]>(() => readLocal('pigeon-orders', []))
  const [subscribed, setSubscribed] = useState(() => localStorage.getItem('pigeon-subscriber') === 'true')
  const [storeRegion, setStoreRegion] = useState(() => localStorage.getItem('pigeon-store-region') || '')
  const [gameUsage, setGameUsage] = useState<GameUsage>(() => readLocal('pigeon-game-usage', { date: new Date().toISOString().slice(0, 10), count: 0 }))
  const [settingsPrefs, setSettingsPrefs] = useState<SettingsPrefs>(() => readLocal('pigeon-settings', { notifications: true, readReceipts: true, allowCalls: true, autoDownload: false }))
  const selectedChat = chats.find((chat) => chat.id === selectedId) ?? chats[0]
  const selectedMessageCount = selectedChat?.messages.length ?? 0
  const gamesCreatedToday = gameUsage.date === new Date().toISOString().slice(0, 10) ? gameUsage.count : 0
  const messageAreaRef = useRef<HTMLDivElement>(null)
  const microphoneStream = useRef<MediaStream | null>(null)
  const callTransitionTimer = useRef<number | null>(null)
  const flightTimer = useRef<number | null>(null)
  const sendButtonRef = useRef<HTMLButtonElement>(null)
  const cameraRef = useRef<HTMLInputElement>(null)
  const documentRef = useRef<HTMLInputElement>(null)
  const voiceRecorderStream = useRef<MediaStream | null>(null)
  const voiceRecorder = useRef<MediaRecorder | null>(null)
  const voiceChunks = useRef<Blob[]>([])

  useEffect(() => {
    document.documentElement.dataset.theme = theme
    localStorage.setItem('pigeon-theme', theme)
    localStorage.setItem('pigeon-theme-v3', theme)
  }, [theme])

  useEffect(() => {
    const frame = window.requestAnimationFrame(() => {
      const area = messageAreaRef.current
      if (!area) return
      area.scrollTo({ top: area.scrollHeight, behavior: 'smooth' })
      const canScroll = area.scrollHeight > area.clientHeight + 40
      setScrollState({ up: canScroll, down: false })
    })
    return () => window.cancelAnimationFrame(frame)
  }, [selectedId, selectedMessageCount])

  useEffect(() => {
    if (!flight || flight.toX !== flight.fromX || flight.toY !== flight.fromY) return
    const frame = window.requestAnimationFrame(() => {
      const bubble = document.querySelector<HTMLElement>(`[data-message-id="${flight.messageId}"]`)
      if (!bubble) return
      const rect = bubble.getBoundingClientRect()
      setFlight((current) => current?.id === flight.id ? { ...current, toX: rect.left + rect.width / 2 - 22, toY: rect.top + rect.height / 2 - 22 } : current)
    })
    return () => window.cancelAnimationFrame(frame)
  }, [flight])

  useEffect(() => {
    if (!microphoneEnabled) return
    const timer = window.setInterval(() => setCallSeconds((seconds) => seconds + 1), 1000)
    return () => window.clearInterval(timer)
  }, [microphoneEnabled])

  useEffect(() => () => {
    if (callTransitionTimer.current !== null) window.clearTimeout(callTransitionTimer.current)
    if (flightTimer.current !== null) window.clearTimeout(flightTimer.current)
    microphoneStream.current?.getTracks().forEach((track) => track.stop())
    voiceRecorderStream.current?.getTracks().forEach((track) => track.stop())
  }, [])

  function showNotice(text: string) {
    setNotice(text)
    window.setTimeout(() => setNotice(''), 2800)
  }

  function startCall(chat: Chat) {
    if (!settingsPrefs.allowCalls) {
      showNotice('Calls are disabled in your privacy settings.')
      return
    }
    setCallCount((count) => count + 1)
    setCallSeconds(0)
    setActiveCall(chat)
    setCallPhase('finding')
    setCutscene(true)
    if (callTransitionTimer.current !== null) window.clearTimeout(callTransitionTimer.current)
    callTransitionTimer.current = window.setTimeout(() => {
      setCutscene(false)
      callTransitionTimer.current = window.setTimeout(() => {
        if (chat.online) {
          setCallPhase('answered')
          callTransitionTimer.current = null
        } else {
          setCallPhase('missed')
          callTransitionTimer.current = window.setTimeout(() => {
            finishCall()
            callTransitionTimer.current = null
          }, 2200)
        }
      }, 1500)
    }, 1900)
  }

  function finishCall() {
    if (callTransitionTimer.current !== null) window.clearTimeout(callTransitionTimer.current)
    callTransitionTimer.current = null
    microphoneStream.current?.getTracks().forEach((track) => track.stop())
    microphoneStream.current = null
    setMicrophoneEnabled(false)
    setCutscene(false)
    setActiveCall(null)
    setCallPhase('finding')
  }

  function saveProfileEdits(event: FormEvent<HTMLFormElement>) {
    event.preventDefault()
    const form = new FormData(event.currentTarget)
    const nextProfile: Profile = {
      name: String(form.get('editName') || '').trim(),
      username: String(form.get('editUsername') || '').trim().replace(/^@/, ''),
      birthday: String(form.get('editBirthday') || ''),
      photo: photo || profile?.photo || '',
    }
    if (!nextProfile.name || !nextProfile.username || !nextProfile.birthday) {
      showNotice('Add your name, username, and birthday before saving.')
      return
    }
    try {
      localStorage.setItem('pigeon-profile', JSON.stringify(nextProfile))
      setProfile(nextProfile)
      setProfileEditOpen(false)
      showNotice('Your profile is updated on this device.')
    } catch {
      showNotice('That profile photo is too large to save.')
    }
  }

  function updateSetting(key: keyof SettingsPrefs) {
    setSettingsPrefs((current) => {
      const next = { ...current, [key]: !current[key] }
      localStorage.setItem('pigeon-settings', JSON.stringify(next))
      return next
    })
  }

  async function toggleMicrophone() {
    if (microphoneStream.current) {
      const track = microphoneStream.current.getAudioTracks()[0]
      if (track) track.enabled = !microphoneEnabled
      setMicrophoneEnabled(!microphoneEnabled)
      return
    }
    if (!navigator.mediaDevices?.getUserMedia) {
      showNotice('Microphone access is unavailable in this browser.')
      return
    }
    try {
      microphoneStream.current = await navigator.mediaDevices.getUserMedia({ audio: true })
      setMicrophoneEnabled(true)
      setCallSeconds(0)
    } catch {
      showNotice('Allow microphone access in your browser to try voice chat.')
    }
  }

  function updateScrollState(area: HTMLDivElement) {
    const remaining = area.scrollHeight - area.scrollTop - area.clientHeight
    setScrollState({ up: area.scrollTop > 40, down: remaining > 40 })
  }

  function scrollMessages(direction: 'up' | 'down') {
    const area = messageAreaRef.current
    if (!area) return
    area.scrollBy({ top: (direction === 'up' ? -1 : 1) * area.clientHeight * 0.75, behavior: 'smooth' })
  }

  function toggleWishlist(product: StoreProduct) {
    setWishlistIds((current) => {
      const next = current.includes(product.id) ? current.filter((id) => id !== product.id) : [...current, product.id]
      localStorage.setItem('pigeon-wishlist', JSON.stringify(next))
      return next
    })
  }

  function placeOrder(product: StoreProduct) {
    const order: StoreOrder = { id: `order-${Date.now()}`, product, status: 'Talabat delivery · on the way', orderedAt: new Date().toLocaleString([], { dateStyle: 'short', timeStyle: 'short' }) }
    setOrders((current) => {
      const next = [order, ...current]
      localStorage.setItem('pigeon-orders', JSON.stringify(next))
      return next
    })
    showNotice(`${product.name} is on its way in this delivery demo.`)
  }

  function sendProductToChat(product: StoreProduct) {
    const orderMessage: Message = { id: Date.now(), text: `I found ${product.name} in the Pigeon market · $${product.price.toFixed(2)} from ${product.seller}.`, time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }), mine: true }
    deliverMessage(orderMessage, `Market find · ${product.name}`, 'note')
    setPage('chats')
  }

  function listProduct(product: Omit<StoreProduct, 'id' | 'seller'>) {
    const listing: StoreProduct = { ...product, id: `listing-${Date.now()}`, seller: profile?.name || 'Your shop' }
    setProducts((current) => {
      const next = [listing, ...current]
      localStorage.setItem('pigeon-market-products', JSON.stringify(next))
      return next
    })
    showNotice('Your local demo listing is live in the catalog.')
  }

  function subscribeDemo() {
    setSubscribed(true)
    localStorage.setItem('pigeon-subscriber', 'true')
    showNotice('Loft Plus is active for this local demo.')
  }

  function createGame(name: string, template: GameTemplate) {
    if (!subscribed && gamesCreatedToday >= 1) return false
    const usage = { date: new Date().toISOString().slice(0, 10), count: gamesCreatedToday + 1 }
    setGameUsage(usage)
    localStorage.setItem('pigeon-game-usage', JSON.stringify(usage))
    showNotice(`${name} · ${template} is ready to play.`)
    return true
  }

  function sendGift(gift: string) {
    if (!subscribed) {
      setPage('settings')
      showNotice('Gift sending is a Loft Plus perk in this demo.')
      return
    }
    const giftMessage: Message = { id: Date.now(), text: `A little ${gift} gift is on its way to you 🎁`, time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }), mine: true }
    deliverMessage(giftMessage, `Gift sent · ${gift}`, 'note')
    setPage('chats')
  }

  function sendCode(event: FormEvent<HTMLFormElement>) {
    event.preventDefault()
    const value = identifier.trim()
    const isEmail = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)
    const phoneDigits = value.replace(/\D/g, '')
    if (!isEmail && (phoneDigits.length < 7 || phoneDigits.length > 15)) {
      setError('Enter a valid email address or phone number.')
      return
    }
    setError('')
    setCode('')
    setStage('verify')
  }

  function verifyCode(event: FormEvent<HTMLFormElement>) {
    event.preventDefault()
    if (code !== '246810') {
      setError('That code does not match. Use the demo code shown below.')
      return
    }
    setError('')
    setStage('profile')
  }

  function finishProfile(event: FormEvent<HTMLFormElement>) {
    event.preventDefault()
    const form = new FormData(event.currentTarget)
    const nextProfile: Profile = {
      name: String(form.get('name') || '').trim(),
      username: String(form.get('username') || '').trim().replace(/^@/, ''),
      birthday: String(form.get('birthday') || ''),
      photo,
    }
    if (!nextProfile.name || !nextProfile.username || !nextProfile.birthday) {
      setError('Complete each profile field to continue.')
      return
    }
    try {
      localStorage.setItem('pigeon-profile', JSON.stringify(nextProfile))
    } catch {
      setError('This photo is too large to save. Choose a smaller image and try again.')
      return
    }
    setProfile(nextProfile)
    setStage('app')
  }

  function changePhoto(file?: File) {
    if (!file) return
    if (!file.type.startsWith('image/')) {
      setError('Choose an image file for your profile photo.')
      return
    }
    if (file.size > 1_500_000) {
      setError('Choose a photo under 1.5 MB for this local preview.')
      return
    }
    const reader = new FileReader()
    reader.onload = () => setPhoto(String(reader.result))
    reader.readAsDataURL(file)
  }

  function sendMessage(event: FormEvent<HTMLFormElement>) {
    event.preventDefault()
    sendTextMessage(message)
  }

  function sendTextMessage(value: string) {
    const text = value.trim()
    if (!text) return
    const nextMessage: Message = { id: Date.now(), text, time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }), mine: true }
    deliverMessage(nextMessage, text, 'note')
    setMessage('')
  }

  function sendSticker(kind: 'gif' | 'sticker', label: string) {
    const art = label.includes('wave') ? '🕊️👋' : label.includes('Dancing') ? '🕊️✨' : label.includes('congratulations') ? '🎉🕊️' : label.includes('heart') ? '💛🕊️' : '🕊️🌷'
    const nextMessage: Message = { id: Date.now(), text: '', time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }), mine: true, contentKind: kind, mediaName: `${art} ${label}` }
    deliverMessage(nextMessage, `${kind === 'gif' ? 'GIF' : 'Sticker'} · ${label}`, 'note')
  }

  function deliverMessage(nextMessage: Message, preview: string, kind: 'note' | 'photo' | 'video') {
    setChats((current) => current.map((chat) => chat.id === selectedId ? { ...chat, preview, time: 'Now', messages: [...chat.messages, nextMessage] } : chat))
    const sendRect = sendButtonRef.current?.getBoundingClientRect()
    const fromX = sendRect ? sendRect.left + sendRect.width / 2 - 22 : window.innerWidth - 50
    const fromY = sendRect ? sendRect.top + sendRect.height / 2 - 22 : window.innerHeight - 100
    const flightId = Date.now()
    setFlight({ id: flightId, messageId: nextMessage.id, kind, fromX, fromY, toX: fromX, toY: fromY })
    if (flightTimer.current !== null) window.clearTimeout(flightTimer.current)
    flightTimer.current = window.setTimeout(() => {
      setFlight((current) => current?.id === flightId ? null : current)
      flightTimer.current = null
    }, 1750)
  }

  function sendAttachment(file?: File) {
    if (!file) return
    const extension = file.name.split('.').pop()?.toLowerCase() || ''
    const documentTypes = ['pdf', 'txt', 'csv', 'doc', 'docx', 'ppt', 'pptx', 'xls', 'xlsx', 'rtf']
    const mediaType = file.type.startsWith('image/') ? 'image' : file.type.startsWith('video/') ? 'video' : file.type.startsWith('audio/') ? 'audio' : documentTypes.includes(extension) ? 'document' : null
    if (!mediaType) {
      showNotice('Choose an image, video, audio clip, PDF, or text document.')
      return
    }
    if (file.size > 20_000_000) {
      showNotice('Choose a file smaller than 20 MB.')
      return
    }
    const preview = mediaType === 'image' ? 'Photo sent' : mediaType === 'video' ? 'Video sent' : mediaType === 'audio' ? 'Voice note sent' : `Document · ${file.name}`
    const nextMessage: Message = {
      id: Date.now(), text: '', time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }), mine: true,
      mediaUrl: URL.createObjectURL(file), mediaType, mediaName: file.name,
    }
    deliverMessage(nextMessage, preview, mediaType === 'image' ? 'photo' : mediaType === 'video' ? 'video' : 'note')
    setToolsOpen(false)
  }

  async function recordVoiceNote() {
    if (voiceRecorder.current?.state === 'recording') {
      voiceRecorder.current.stop()
      return
    }
    if (!navigator.mediaDevices?.getUserMedia || typeof MediaRecorder === 'undefined') {
      showNotice('Voice recording is unavailable in this browser.')
      return
    }
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true })
      voiceRecorderStream.current = stream
      voiceChunks.current = []
      const recorder = new MediaRecorder(stream)
      voiceRecorder.current = recorder
      recorder.ondataavailable = (event) => { if (event.data.size) voiceChunks.current.push(event.data) }
      recorder.onstop = () => {
        const audio = new Blob(voiceChunks.current, { type: recorder.mimeType || 'audio/webm' })
        const file = new File([audio], `voice-note-${Date.now()}.webm`, { type: audio.type })
        sendAttachment(file)
        stream.getTracks().forEach((track) => track.stop())
        voiceRecorderStream.current = null
        voiceRecorder.current = null
        setRecordingVoice(false)
      }
      recorder.start()
      setRecordingVoice(true)
    } catch {
      showNotice('Allow microphone access to record a voice note.')
    }
  }

  function sendLocation() {
    if (!navigator.geolocation) {
      showNotice('Location sharing is unavailable in this browser.')
      return
    }
    navigator.geolocation.getCurrentPosition((position) => {
      const { latitude, longitude } = position.coords
      sendTextMessage(`My location: https://maps.google.com/?q=${latitude},${longitude}`)
      setToolsOpen(false)
    }, () => showNotice('Allow location access to share your current location.'))
  }

  function sendPoll(event: FormEvent<HTMLFormElement>) {
    event.preventDefault()
    const form = new FormData(event.currentTarget)
    const question = String(form.get('pollQuestion') || '').trim()
    const options = String(form.get('pollOptions') || '').split('\n').map((option) => option.trim()).filter(Boolean).slice(0, 6)
    if (!question || options.length < 2) {
      showNotice('A poll needs a question and at least two options.')
      return
    }
    const poll: Message = { id: Date.now(), text: question, time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }), mine: true, pollOptions: options }
    deliverMessage(poll, `Poll · ${question}`, 'note')
    setPollOpen(false)
    setToolsOpen(false)
  }

  function handleChatTool(tool: 'photos' | 'camera' | 'document' | 'location' | 'poll' | 'emoji' | 'gifs' | 'stickers' | 'voice') {
    if (tool === 'photos') attachmentRef.current?.click()
    if (tool === 'camera') cameraRef.current?.click()
    if (tool === 'document') documentRef.current?.click()
    if (tool === 'location') sendLocation()
    if (tool === 'poll') { setPollOpen(true); setToolsOpen(false) }
    if (tool === 'emoji' || tool === 'gifs' || tool === 'stickers') setPicker(tool)
    if (tool === 'voice') recordVoiceNote()
    if (tool !== 'poll') setToolsOpen(false)
  }

  function addContact(event: FormEvent<HTMLFormElement>) {
    event.preventDefault()
    const form = new FormData(event.currentTarget)
    const name = String(form.get(contactType === 'group' ? 'groupName' : 'contactName') || '').trim()
    const handle = String(form.get('contactHandle') || '').trim()
    if (!name) return
    const members = String(form.get('groupMembers') || '').split(',').map((member) => member.trim()).filter(Boolean)
    const contact: Chat = {
      id: `contact-${Date.now()}`, name, username: contactType === 'group' ? `${members.length + 1} members` : handle || 'New contact', preview: contactType === 'group' ? 'Your group loft is ready.' : 'You can start a conversation.', time: 'Now', unread: 0, photo: '', tint: 'blue', online: false, group: contactType === 'group', messages: [],
    }
    setChats((current) => [contact, ...current])
    setSelectedId(contact.id)
    setPage('chats')
    setContactOpen(false)
    setContactType('person')
  }

  function logOut() {
    localStorage.removeItem('pigeon-profile')
    setProfile(null)
    setIdentifier('')
    setStage('sign-in')
  }

  const visibleChats = chats.filter((chat) => (chatFilter === 'all' || chat.group) && `${chat.name} ${chat.preview}`.toLowerCase().includes(search.toLowerCase()))

  if (stage !== 'app') {
    return (
      <main className="onboarding" data-stage={stage}>
        <div className="onboarding-art" aria-hidden="true"><span className="art-cloud cloud-one" /><span className="art-cloud cloud-two" /><div className="loft-house"><div className="loft-roof" /><div className="loft-body"><span className="loft-sign">P</span><span className="loft-hole">🕊️</span><span className="loft-perch" /></div><span className="loft-leg leg-one" /><span className="loft-leg leg-two" /></div><div className="loft-caption"><span>THE PIGEON LOFT</span><strong>Good messages<br />find their way home.</strong></div></div>
        <div className="onboarding-panel">
          <a className="brand-lockup" href="#top" aria-label="Pigeon home"><span className="brand-mark"><span aria-hidden="true">🕊</span></span><span>Pigeon <b>Loft</b></span></a>
          {stage === 'sign-in' && <section className="auth-content" id="top">
            <p className="eyebrow">A little closer, wherever you are</p>
            <h1>Good chats<br />start here.</h1>
            <p className="auth-copy">Your people, your moments, all in one place.</p>
            <form className="auth-form" onSubmit={sendCode}>
              <label htmlFor="identifier">Email or phone number</label>
              <input id="identifier" autoComplete="username" placeholder="you@example.com or +1 555 0100" value={identifier} onChange={(event) => setIdentifier(event.target.value)} />
              {error && <p className="form-error">{error}</p>}
              <button className="primary-button" type="submit">Continue <span>→</span></button>
            </form>
            <p className="fine-print">Demo account only. Your details stay in this browser; no email or text is sent.</p>
          </section>}
          {stage === 'verify' && <section className="auth-content" id="top">
            <button className="text-back" onClick={() => { setStage('sign-in'); setError('') }}><Icon name="back" size={17} /> Back</button>
            <p className="eyebrow">One quick check</p>
            <h1>Enter your<br />code.</h1>
            <p className="auth-copy">Enter the demo code for <strong>{identifier}</strong>. This preview does not send real messages.</p>
            <form className="auth-form" onSubmit={verifyCode}>
              <label htmlFor="verification-code">Verification code</label>
              <input id="verification-code" inputMode="numeric" autoComplete="one-time-code" maxLength={6} placeholder="6-digit code" value={code} onChange={(event) => setCode(event.target.value.replace(/\D/g, ''))} />
              <div className="demo-code"><span>DEMO CODE</span><strong>246810</strong></div>
              {error && <p className="form-error">{error}</p>}
              <button className="primary-button" type="submit">Verify and continue <span>→</span></button>
            </form>
          </section>}
          {stage === 'profile' && <section className="auth-content profile-content" id="top">
            <p className="eyebrow">Make yourself at home</p>
            <h1>Your profile,<br />your way.</h1>
            <form className="auth-form" onSubmit={finishProfile}>
              <label className="photo-picker" htmlFor="profile-photo">
                <span className="photo-preview">{photo ? <img src={photo} alt="Profile preview" /> : <Icon name="plus" size={22} />}</span><span>Add a profile photo <small>Optional</small></span>
                <input id="profile-photo" type="file" accept="image/*" onChange={(event) => changePhoto(event.target.files?.[0])} />
              </label>
              <label htmlFor="profile-name">Your name</label>
              <input id="profile-name" name="name" autoComplete="name" placeholder="Name people know you by" />
              <label htmlFor="profile-username">Username</label>
              <div className="username-field"><span>@</span><input id="profile-username" name="username" autoComplete="nickname" placeholder="yourname" /></div>
              <label htmlFor="profile-birthday">Date of birth</label>
              <input id="profile-birthday" name="birthday" type="date" autoComplete="bday" />
              {error && <p className="form-error">{error}</p>}
              <button className="primary-button" type="submit">Finish setup <span>→</span></button>
            </form>
          </section>}
          <div className="onboarding-foot"><span>Private by design</span><span>Built for your people</span></div>
        </div>
      </main>
    )
  }

  return (
    <main className="app-shell">
      <aside className="side-rail">
        <a className="rail-brand" href="#home" aria-label="Pigeon Loft"><span className="brand-mark"><span aria-hidden="true">🕊</span></span></a>
        <div className="rail-actions">
          <button className={`rail-button ${page === 'chats' ? 'active' : ''}`} title="Messages" aria-label="Messages" onClick={() => setPage('chats')}><Icon name="chat" /></button>
          <button className="rail-button" title="Add contact" aria-label="Add contact" onClick={() => setContactOpen(true)}><Icon name="plus" /></button>
          <button className={`rail-button ${page === 'calls' ? 'active' : ''}`} title="Calls" aria-label="Calls" onClick={() => setPage('calls')}><Icon name="phone" /></button>
          <button className={`rail-button ${page === 'ai' ? 'active' : ''}`} title="Your AI" aria-label="Your AI" onClick={() => setPage('ai')}><Icon name="spark" /></button>
          <button className={`rail-button ${page === 'store' ? 'active' : ''}`} title="Market store" aria-label="Market store" onClick={() => setPage('store')}><Icon name="store" /></button>
          <button className={`rail-button ${page === 'games' ? 'active' : ''}`} title="Game space" aria-label="Game space" onClick={() => setPage('games')}><Icon name="games" /></button>
          <button className={`rail-button ${page === 'news' ? 'active' : ''}`} title="News reports" aria-label="News reports" onClick={() => setPage('news')}><Icon name="news" /></button>
        </div>
        <div className="rail-bottom">
          <button className="rail-button" title={`Change theme · ${theme}`} aria-label={`Change theme · ${theme}`} onClick={() => setTheme(theme === 'liquid' ? 'dark' : theme === 'dark' ? 'light' : 'liquid')}><Icon name={theme === 'liquid' ? 'droplet' : theme === 'light' ? 'sun' : 'moon'} /></button>
          <button className={`rail-button ${page === 'settings' ? 'active' : ''}`} title="Settings" aria-label="Settings" onClick={() => setPage('settings')}><Icon name="settings" /></button>
          <button className="profile-chip" title={profile?.name || 'Profile'} onClick={() => setPage('settings')}>
            {profile?.photo ? <img src={profile.photo} alt="" /> : <span>{profile?.name?.[0] || 'K'}</span>}
          </button>
        </div>
      </aside>

      <section className={`chat-column ${page !== 'chats' ? 'chat-column-hidden' : ''} ${selectedId && page === 'chats' ? 'has-selection' : ''}`}>
        <header className="list-header">
          <div className="list-heading"><div><p className="eyebrow">YOUR CORNER OF THE WORLD</p><h1>Messages<span className="heading-count">{chats.length}</span></h1></div><button className="icon-button new-chat-button" title="Add contact" aria-label="Add contact" onClick={() => setContactOpen(true)}><Icon name="plus" /></button></div>
          <label className="search-box"><Icon name="search" size={18} /><input aria-label="Search conversations" placeholder="Search conversations" value={search} onChange={(event) => setSearch(event.target.value)} /><kbd>⌘ K</kbd></label>
          <div className="filter-row"><button className={`filter-pill ${chatFilter === 'all' ? 'selected' : ''}`} aria-pressed={chatFilter === 'all'} onClick={() => setChatFilter('all')}>All <span>{chats.length}</span></button><button className={`filter-pill ${chatFilter === 'groups' ? 'selected' : ''}`} aria-pressed={chatFilter === 'groups'} onClick={() => setChatFilter('groups')}>Groups <span>{chats.filter((chat) => chat.group).length}</span></button></div>
        </header>
        <div className="conversation-list">
          <p className="list-label">RECENT CONVERSATIONS</p>
          {visibleChats.map((chat) => <button key={chat.id} className={`conversation-item ${selectedId === chat.id ? 'conversation-active' : ''}`} onClick={() => { setSelectedId(chat.id); setPage('chats') }}>
            <span className="avatar-wrap"><Avatar chat={chat} />{chat.online && <span className="online-dot" />}</span>
            <span className="conversation-copy"><span className="conversation-name">{chat.name}{chat.group && <span className="ai-label">GROUP</span>}</span><span className="conversation-preview">{chat.preview}</span></span>
            <span className="conversation-meta"><time>{chat.time}</time>{chat.unread > 0 && <span className="unread-count">{chat.unread}</span>}</span>
          </button>)}
          {visibleChats.length === 0 && <p className="empty-search">No conversations match “{search}”.</p>}
        </div>
        <div className="list-footer"><span className="privacy-dot" /> Your conversations stay yours <span className="footer-lock">⌑</span></div>
      </section>

      <section className={`workspace ${page !== 'chats' ? 'workspace-page' : ''}`}>
        {page === 'chats' && selectedChat && selectedId && <>
          <header className="chat-header">
            <button className="mobile-back" title="Back to messages" aria-label="Back to messages" onClick={() => setSelectedId('')}><Icon name="back" /></button>
            <Avatar chat={selectedChat} size="small" />
            <div className="chat-heading"><strong>{selectedChat.name}</strong><span>{selectedChat.online ? 'Home and online' : selectedChat.username}</span></div>
            <div className="header-tools"><button className="icon-button" title="Start a voice chat" aria-label={`Start a voice chat with ${selectedChat.name}`} onClick={() => startCall(selectedChat)}><Icon name="phone" /></button><button className="icon-button" title="Conversation details" aria-label="Conversation details" onClick={() => showNotice(`${selectedChat.name} · ${selectedChat.username}`)}><Icon name="user" /></button></div>
          </header>
          <div className="message-area" ref={messageAreaRef} onScroll={(event) => updateScrollState(event.currentTarget)}>
            <div className="day-divider"><span>Today</span></div>
            <div className="message-stack">{selectedChat.messages.map((item) => <div key={item.id} className={`message-row ${item.mine ? 'message-mine' : ''}`}>
              {!item.mine && <Avatar chat={selectedChat} size="small" />}
              <div className="message-bubble" data-message-id={item.id}>{item.mediaUrl && item.mediaType === 'image' && <img className="message-media" src={item.mediaUrl} alt="Photo attachment" />}{item.mediaUrl && item.mediaType === 'video' && <video className="message-media" src={item.mediaUrl} controls aria-label="Video attachment" />}{item.mediaUrl && item.mediaType === 'audio' && <audio className="message-audio" src={item.mediaUrl} controls aria-label="Voice note" />}{item.mediaUrl && item.mediaType === 'document' && <a className="message-document" href={item.mediaUrl} download={item.mediaName}>{item.mediaName || 'Open document'}</a>}{item.contentKind && <div className={`chat-sticker chat-${item.contentKind}`}><span>{item.mediaName?.split(' ')[0]}</span><strong>{item.mediaName?.split(' ').slice(1).join(' ')}</strong></div>}{item.text && !item.pollOptions && <p>{item.text}</p>}{item.pollOptions && <div className="poll-message"><strong>{item.text}</strong>{item.pollOptions.map((option) => <button key={option} className={pollVotes[item.id] === option ? 'poll-voted' : ''} onClick={() => setPollVotes((current) => ({ ...current, [item.id]: option }))}>{option}<span>{pollVotes[item.id] === option ? '✓' : ''}</span></button>)}<small>{pollVotes[item.id] ? `You voted: ${pollVotes[item.id]}` : 'Tap an option to vote'}</small></div>}<time>{item.time}{item.mine && <Icon name="check" size={13} />}</time></div>
            </div>)}</div>
            {(scrollState.up || scrollState.down) && <div className="message-scroll-controls"><button type="button" title="Scroll to older messages" aria-label="Scroll to older messages" disabled={!scrollState.up} onClick={() => scrollMessages('up')}><Icon name="up" size={17} /></button><button type="button" title="Scroll to newer messages" aria-label="Scroll to newer messages" disabled={!scrollState.down} onClick={() => scrollMessages('down')}><Icon name="down" size={17} /></button></div>}
          </div>
          <div className="composer-area"><form className="composer" onSubmit={sendMessage}>
            <input ref={attachmentRef} className="attachment-input" type="file" accept="image/*,video/*" aria-label="Choose photos or videos" onChange={(event) => { sendAttachment(event.target.files?.[0]); event.target.value = '' }} />
            <input ref={cameraRef} className="attachment-input" type="file" accept="image/*" capture="environment" aria-label="Take a photo" onChange={(event) => { sendAttachment(event.target.files?.[0]); event.target.value = '' }} />
            <input ref={documentRef} className="attachment-input" type="file" accept=".pdf,.txt,.csv,.doc,.docx,.ppt,.pptx,.xls,.xlsx,.rtf" aria-label="Choose a document" onChange={(event) => { sendAttachment(event.target.files?.[0]); event.target.value = '' }} />
            {recordingVoice && <button className="recording-indicator" type="button" onClick={() => handleChatTool('voice')} aria-label="Stop voice note recording">● Stop recording</button>}
            <button className={`composer-tool ${recordingVoice ? 'voice-recording' : ''}`} type="button" title={recordingVoice ? 'Stop voice note' : 'More message options'} aria-label={recordingVoice ? 'Stop voice note' : 'More message options'} onClick={() => recordingVoice ? handleChatTool('voice') : setToolsOpen((current) => !current)}><Icon name={recordingVoice ? 'phoneOff' : 'plus'} /></button>
            <input aria-label="Write a message" placeholder={`Message ${selectedChat.name}`} value={message} onChange={(event) => setMessage(event.target.value)} />
            <button ref={sendButtonRef} className={`send-button ${message.trim() ? 'send-ready' : ''}`} type="submit" title="Send message" aria-label="Send message"><Icon name="send" size={18} /></button>
          </form>
          {toolsOpen && <div className="chat-tool-menu" aria-label="Message options">{[['photos', 'Photos & videos'], ['camera', 'Camera'], ['location', 'Location'], ['poll', 'Poll'], ['document', 'Document'], ['emoji', 'Emoji'], ['gifs', 'GIFs'], ['stickers', 'Stickers'], ['voice', recordingVoice ? 'Stop voice note' : 'Voice note']].map(([tool, label]) => <button type="button" key={tool} onClick={() => handleChatTool(tool as 'photos' | 'camera' | 'document' | 'location' | 'poll' | 'emoji' | 'gifs' | 'stickers' | 'voice')}>{label}</button>)}</div>}
          {picker && <div className="chat-picker-panel" aria-label={`${picker} picker`}>{(picker === 'emoji' ? ['😀', '😂', '🥲', '💛', '🕊️', '🙌', '🎉', '🌷'] : picker === 'gifs' ? ['Hello wave', 'Dancing pigeon', 'Big congratulations'] : ['Pigeon hello', 'Tiny heart', 'Garden friend']).map((item) => <button type="button" key={item} onClick={() => { if (picker === 'emoji') setMessage((current) => `${current}${item}`); else sendSticker(picker === 'gifs' ? 'gif' : 'sticker', item); setPicker('') }}>{item}</button>)}<button type="button" className="picker-close" onClick={() => setPicker('')} aria-label="Close picker">×</button></div>}
          </div>
          <div className="conversation-note">Messages are stored only in this browser preview.</div>
        </>}
        {page === 'calls' && <section className="utility-page calls-page">
          <p className="eyebrow">KEEP IN TOUCH</p><h1>Calls</h1><p className="utility-intro">A quick hello can change the whole day.</p>
          <div className="call-feature"><div className="call-feature-icon"><Icon name="phone" size={23} /></div><div><strong>Start a voice chat</strong><span>Open a local microphone preview with {selectedChat.name}.</span></div><button className="primary-button compact-button" onClick={() => startCall(selectedChat)}>Call <Icon name="phone" size={16} /></button></div>
          <div className="call-list-heading"><h2>Recent</h2><span>{callCount ? `${callCount} new` : 'This week'}</span></div>
          {chats.filter((chat) => !chat.group).slice(0, 3).map((chat, index) => <button className="call-item" key={chat.id} aria-label={`Call ${chat.name}`} onClick={() => startCall(chat)}><Avatar chat={chat} /><span><strong>{chat.name}</strong><small>{index === 1 ? 'Yesterday · Incoming' : 'Monday · Outgoing'}</small></span><Icon name="phone" /></button>)}
          <p className="integration-note">Try microphone access here. Live calls need a signaling server to connect both people.</p>
        </section>}
        {page === 'settings' && <section className="utility-page settings-page">
          <p className="eyebrow">MAKE IT YOURS</p><h1>Settings</h1><p className="utility-intro">A few small things, set just how you like them.</p>
          <div className="settings-section"><h2>Appearance</h2><div className="setting-row"><span className="setting-icon"><Icon name={theme === 'light' ? 'sun' : theme === 'liquid' ? 'droplet' : 'moon'} /></span><span className="setting-copy"><strong>Theme</strong><small>{theme === 'liquid' ? 'Liquid glass' : theme === 'light' ? 'Light and easy on the eyes' : 'A softer glow after dark'}</small></span><div className="segmented-control theme-segments"><button className={theme === 'light' ? 'segment-active' : ''} onClick={() => setTheme('light')} aria-label="Use light theme"><Icon name="sun" size={16} /></button><button className={theme === 'liquid' ? 'segment-active' : ''} onClick={() => setTheme('liquid')} aria-label="Use liquid theme"><Icon name="droplet" size={16} /></button><button className={theme === 'dark' ? 'segment-active' : ''} onClick={() => setTheme('dark')} aria-label="Use dark theme"><Icon name="moon" size={16} /></button></div></div></div>
          <div className="settings-section"><h2>Messages & privacy</h2>{([{ key: 'notifications', title: 'Notifications', note: 'Show message alerts on this device.' }, { key: 'readReceipts', title: 'Read receipts', note: 'Let people know when you have read a message.' }, { key: 'allowCalls', title: 'Allow calls', note: 'Let your contacts start a voice call.' }, { key: 'autoDownload', title: 'Auto-download media', note: 'Automatically prepare shared photos and videos.' }] as const).map((setting) => <div className="setting-row preference-row" key={setting.key}><span className="setting-copy"><strong>{setting.title}</strong><small>{setting.note}</small></span><button className={`toggle-switch ${settingsPrefs[setting.key] ? 'toggle-on' : ''}`} role="switch" aria-checked={settingsPrefs[setting.key]} aria-label={setting.title} onClick={() => updateSetting(setting.key)}><span /></button></div>)}</div>
          <div className="settings-section"><h2>Subscription</h2><div className="plan-row"><div><strong>{subscribed ? 'Loft Plus · active demo' : 'Free plan'}</strong><small>{subscribed ? 'More game creations, store discounts, gifts, and saved AI setups.' : 'Configure your own AI, create one game a day, and get occasional store discounts.'}</small></div>{subscribed ? <button className="text-button" onClick={() => { setSubscribed(false); localStorage.setItem('pigeon-subscriber', 'false'); showNotice('You are back on the free demo plan.') }}>Switch to free</button> : <button className="primary-button compact-button" onClick={subscribeDemo}>Try Loft Plus</button>}</div><p className="integration-note">Demo subscription only. No payment is collected or recurring plan started.</p></div>
          <div className="settings-section"><h2>Gifts for friends</h2>{subscribed ? <><p className="gift-intro">Send one to {selectedChat.name} in your open chat.</p><div className="gift-actions"><button onClick={() => sendGift('rose')}>🌹 <span>Rose</span></button><button onClick={() => sendGift('postcard')}>💌 <span>Postcard</span></button><button onClick={() => sendGift('tea')}>🍵 <span>Tea</span></button></div></> : <div className="gift-locked"><span>🎁</span><p>Gift sending is included with Loft Plus.</p><button className="quiet-button" onClick={subscribeDemo}>See subscription</button></div>}</div>
          <div className="settings-section account-section"><h2>Your account</h2><div className="account-row"><Avatar chat={{ name: profile?.name || 'Pigeon', photo: profile?.photo || '', tint: 'blue' }} /><span><strong>{profile?.name}</strong><small>@{profile?.username}</small></span><button className="text-button" onClick={() => { setPhoto(profile?.photo || ''); setProfileEditOpen(true) }}>Edit profile</button><button className="text-button" onClick={logOut}>Sign out</button></div></div>
        </section>}
        {page === 'store' && <StorePage products={products} wishlistIds={wishlistIds} orders={orders} subscribed={subscribed} region={storeRegion} onSetRegion={(region) => { setStoreRegion(region); localStorage.setItem('pigeon-store-region', region) }} onWishlist={toggleWishlist} onOrder={placeOrder} onSendToChat={sendProductToChat} onSell={listProduct} />}
        {page === 'ai' && <AssistantPage />}
        {page === 'games' && <GamesPage gamesCreatedToday={gamesCreatedToday} subscribed={subscribed} onCreate={createGame} onUpgrade={() => { setPage('settings'); showNotice('Loft Plus includes unlimited game creations.') }} />}
        {page === 'news' && <NewsPage />}
      </section>

      <nav className="mobile-nav" aria-label="Main navigation">
        <button className={page === 'chats' ? 'mobile-nav-active' : ''} onClick={() => setPage('chats')}><Icon name="chat" /><span>Chats</span></button>
        <button onClick={() => { setContactType('person'); setContactOpen(true) }}><Icon name="plus" /><span>New chat</span></button>
        <button className={page === 'calls' ? 'mobile-nav-active' : ''} onClick={() => setPage('calls')}><Icon name="phone" /><span>Calls</span></button>
        <button className={page === 'ai' ? 'mobile-nav-active' : ''} onClick={() => setPage('ai')}><Icon name="spark" /><span>AI</span></button>
        <button className={page === 'store' ? 'mobile-nav-active' : ''} onClick={() => setPage('store')}><Icon name="store" /><span>Store</span></button>
        <button className={page === 'games' ? 'mobile-nav-active' : ''} onClick={() => setPage('games')}><Icon name="games" /><span>Games</span></button>
        <button className={page === 'news' ? 'mobile-nav-active' : ''} onClick={() => setPage('news')}><Icon name="news" /><span>News</span></button>
        <button className={page === 'settings' ? 'mobile-nav-active' : ''} onClick={() => setPage('settings')}><Icon name="settings" /><span>Settings</span></button>
      </nav>
      {contactOpen && <div className="modal-backdrop" onMouseDown={(event) => { if (event.target === event.currentTarget) setContactOpen(false) }}><section className="contact-modal" role="dialog" aria-modal="true" aria-labelledby="add-contact-heading"><button className="icon-button modal-close" aria-label="Close" onClick={() => setContactOpen(false)}><Icon name="close" /></button><p className="eyebrow">GROW YOUR CIRCLE</p><h2 id="add-contact-heading">Open a new loft</h2><p>Start a one-to-one chat or gather your group.</p><div className="contact-tabs" role="tablist" aria-label="Conversation type"><button type="button" role="tab" aria-selected={contactType === 'person'} className={contactType === 'person' ? 'contact-tab-active' : ''} onClick={() => setContactType('person')}>Person</button><button type="button" role="tab" aria-selected={contactType === 'group'} className={contactType === 'group' ? 'contact-tab-active' : ''} onClick={() => setContactType('group')}>Group</button></div><form className="auth-form" onSubmit={addContact}>{contactType === 'person' ? <><label htmlFor="contact-name">Their name</label><input id="contact-name" name="contactName" autoFocus placeholder="Name" required /><label htmlFor="contact-handle">Email or phone</label><input id="contact-handle" name="contactHandle" placeholder="Optional for this preview" /></> : <><label htmlFor="group-name">Group name</label><input id="group-name" name="groupName" autoFocus placeholder="Sunday garden club" required /><label htmlFor="group-members">Invite people</label><input id="group-members" name="groupMembers" placeholder="Names, separated by commas" /></>}<button className="primary-button" type="submit">{contactType === 'group' ? 'Create group' : 'Add person'} <span>→</span></button></form><small className="integration-note">Chats and groups in this prototype are stored in this browser session.</small></section></div>}
      {profileEditOpen && <div className="modal-backdrop" onMouseDown={(event) => { if (event.target === event.currentTarget) setProfileEditOpen(false) }}><section className="contact-modal" role="dialog" aria-modal="true" aria-labelledby="edit-profile-heading"><button className="icon-button modal-close" aria-label="Close profile editor" onClick={() => setProfileEditOpen(false)}><Icon name="close" /></button><p className="eyebrow">YOUR ACCOUNT</p><h2 id="edit-profile-heading">Edit profile</h2><form className="auth-form" onSubmit={saveProfileEdits}><label className="photo-picker" htmlFor="edit-profile-photo"><span className="photo-preview">{photo ? <img src={photo} alt="Profile preview" /> : <Icon name="plus" size={22} />}</span><span>Change profile photo<small>Image under 1.5 MB</small></span><input id="edit-profile-photo" type="file" accept="image/*" onChange={(event) => changePhoto(event.target.files?.[0])} /></label><label htmlFor="edit-profile-name">Name</label><input id="edit-profile-name" name="editName" defaultValue={profile?.name} required /><label htmlFor="edit-profile-username">Username</label><input id="edit-profile-username" name="editUsername" defaultValue={profile?.username} required /><label htmlFor="edit-profile-birthday">Birthday</label><input id="edit-profile-birthday" name="editBirthday" type="date" defaultValue={profile?.birthday} required /><button className="primary-button" type="submit">Save profile</button></form></section></div>}
      {pollOpen && <div className="modal-backdrop" onMouseDown={(event) => { if (event.target === event.currentTarget) setPollOpen(false) }}><section className="contact-modal" role="dialog" aria-modal="true" aria-labelledby="poll-heading"><button className="icon-button modal-close" aria-label="Close poll" onClick={() => setPollOpen(false)}><Icon name="close" /></button><p className="eyebrow">ASK YOUR CHAT</p><h2 id="poll-heading">Create a poll</h2><form className="auth-form" onSubmit={sendPoll}><label htmlFor="poll-question">Question</label><input id="poll-question" name="pollQuestion" placeholder="Where should we meet?" required /><label htmlFor="poll-options">Options · one per line</label><textarea id="poll-options" name="pollOptions" rows={4} placeholder={'The garden\nBy the lake'} required /><button className="primary-button" type="submit">Share poll <span>→</span></button></form></section></div>}
      {flight && <div className={`flight-overlay flight-${flight.kind}`} style={{ '--flight-from-x': `${flight.fromX}px`, '--flight-from-y': `${flight.fromY}px`, '--flight-to-x': `${flight.toX}px`, '--flight-to-y': `${flight.toY}px` } as CSSProperties} aria-hidden="true"><span className="flight-bird">🕊️</span><span className="flight-cargo">{flight.kind === 'note' ? '✉' : flight.kind === 'photo' ? '▧' : '▶'}</span></div>}
      {activeCall && !cutscene && <section className="call-room" role="dialog" aria-modal="true" aria-labelledby="call-room-title">
        <div className="call-room-top"><span className="call-live-indicator" /><span>VOICE ROOM · LOCAL PREVIEW</span><button type="button" className="icon-button" aria-label="End voice chat" title="End voice chat" onClick={finishCall}><Icon name="close" /></button></div>
        <div className="call-room-center"><div className={`call-avatar-wrap ${microphoneEnabled ? 'call-speaking' : ''} ${callPhase === 'answered' ? 'call-arrived' : ''}`}><Avatar chat={activeCall} size="large" />{callPhase === 'finding' && <span className="call-arrival-pigeon" aria-hidden="true">🕊️</span>}</div><p className="eyebrow">VOICE CHAT</p><h2 id="call-room-title">{activeCall.name}</h2><p className="call-status">{callPhase === 'finding' ? 'Your pigeon is finding them…' : callPhase === 'missed' ? 'No answer · your pigeon is heading home.' : microphoneEnabled ? `Connected · ${String(Math.floor(callSeconds / 60)).padStart(2, '0')}:${String(callSeconds % 60).padStart(2, '0')}` : 'They answered · enable your microphone'}</p><div className={`voice-wave ${microphoneEnabled ? 'voice-wave-active' : ''}`} aria-hidden="true">{Array.from({ length: 21 }, (_, index) => <span key={index} />)}</div><p className="call-disclaimer">This preview can access your microphone, but it cannot transmit audio to {activeCall.name}. A signaling server is needed for live calls.</p></div>
        <div className="call-controls">{callPhase === 'answered' && <button type="button" className={`call-mic-button ${microphoneEnabled ? 'mic-enabled' : ''}`} onClick={toggleMicrophone}><Icon name="mic" size={21} /><span>{microphoneStream.current && !microphoneEnabled ? 'Unmute' : microphoneEnabled ? 'Mute' : 'Enable mic'}</span></button>}<button type="button" className="call-end-button" aria-label="End voice chat" title="End voice chat" onClick={finishCall}><Icon name="phoneOff" size={21} /><span>{callPhase === 'missed' ? 'Close' : 'End'}</span></button></div>
      </section>}
      {activeCall && callPhase === 'missed' && <span className="call-phase-pigeon" aria-hidden="true">🕊️</span>}
      {cutscene && <div className="call-cutscene" role="status" aria-live="polite"><div className="cutscene-sky"><span className="cutscene-cloud cloud-a" /><span className="cutscene-cloud cloud-b" /><div className="cutscene-house"><span>⌂</span></div><span className="cutscene-pigeon">🕊️</span><p>Your pigeon is searching for {activeCall?.name}</p></div></div>}
      {notice && <div className="toast" role="status">{notice}</div>}
    </main>
  )
}

export default App
