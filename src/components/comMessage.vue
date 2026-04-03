<template>
  <layout>
  <div class="chat-app">
     <!-- <div class="logo-section">
    <img src="/image/logoPetGram.png" alt="PetGram" class="logo-img" />
  </div>
    <nav class="left-bar">
      <div class="nav-icons">
        <router-link to="/home">
          <img src="/image/home.png" class="icon-img" />
        </router-link>
        <router-link to="/message">
          <img src="../assets/message.png" class="icon-img" />
        </router-link>
        <router-link to="/groups">
          <img src="../assets/community.png" class="icon-img" />
        </router-link>
        <router-link to="/friend">
          <img src="../assets/friends.png" class="icon-img" />
        </router-link>
        <router-link to="/profile">
          <img src="../assets/user.png" class="icon-img" />
        </router-link>
      </div>
      <img :src="currentUserAvatar" class="avatar-icon" />
    </nav> -->
    <div class="right-main">
    <aside class="sidebar" v-show="!isMobile || showSidebar">
      <div class="search-bar">
        <img src="/image/tim.png" alt="Tìm kiếm" />
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Tìm kiếm bạn bè..."
        />
      </div>
        <div class="tab-section" v-show="!isMobile || showSidebar">
          <button class="tab-btn" :class="{ active: activeTab === 'all' }" @click="switchTab('all')">
            Tất cả tin nhắn
          </button>
          <button class="tab-btn" :class="{ active: activeTab === 'friends' }" @click="switchTab('friends')">
            Bạn bè
          </button>
          <div class="tab-indicator" :style="indicatorStyle"></div>
        </div>
        <div class="friend-section">
          <div
            v-for="friend in filteredFriends"
            :key="friend.id"
            class="friend-item"
            :class="{ active: selectedFriend?.id === friend.id }"
            @click="selectFriend(friend)"
          >
            <img :src="friend.avatar" class="avatar" />
            <div class="friend-info">
              <div class="name">
                {{ friend.name }}
                <span v-if="friend.online" class="online-dot" title="Đang hoạt động"></span>
              </div>
              <div class="desc">{{ friend.desc }}</div>
            </div>
          </div>
        </div>
      </aside>
      <section class="main-chat" v-if="selectedFriend && (!isMobile || !showSidebar)">
        <header class="chat-header">
        <button v-if="isMobile" @click="showSidebar = true" class="back-btn">
          <img src="/image/back.png" alt="Back" class="back-icon" />
        </button>
          <img :src="selectedFriend.avatar" class="avatar" />
          <div>
            <div class="name">{{ selectedFriend.name }}</div>
            <div class="status" :class="{ online: selectedFriend.online }">
              {{ selectedFriend.online ? 'Đang hoạt động' : 'Không hoạt động' }}
              <span v-if="selectedFriend.online" class="online-dot" title="Đang hoạt động"></span>
            </div>
          </div>
        </header>
        <div class="chat-body" ref="chatBody">
          <div
            v-for="(msg, index) in currentMessages"
            :key="msg.id + '-' + msg.seen"
            :class="['msg-wrapper', msg.fromMe ? 'align-right' : 'align-left']"
          >
            <!-- Avatar người nhận -->
            <img
              v-if="!msg.fromMe"
              :src="selectedFriend.avatar"
              class="avatar-msg"
            />
            <!-- Nội dung tin nhắn -->
            <div class="msg-block">
              <div
                :class="[
                  'msg',
                  msg.fromMe ? 'from-me' : 'from-other',
                  msg.image && !msg.text ? 'image-only' : ''
                ]"
              >
              <div v-if="msg.image?.startsWith('http')">
                <img :src="msg.image" class="msg-image" />
              </div>
              <p v-if="msg.text" class="msg-text">{{ msg.text }}</p>
              </div>
              <!-- Trạng thái cuối cùng nếu là tin từ mình -->
              <small
                v-if="msg.fromMe && isLastSentByMe(index)"
                class="msg-status-outside"
              >
                {{ msg.seen ? 'Đã xem' : 'Đã gửi' }}
              </small>
            </div>
          </div>
        </div>
      <footer class="chat-input">
        <input
          v-model="newMessage"
          @keydown.enter.prevent="sendMessage"
          type="text"
          placeholder="Tin nhắn văn bản"
        />
        <div class="chat-actions">
          <label class="icon-btn">
            <input
              type="file"
              accept="image/*"
              @change="handleImageChange"
              style="display: none"
            />
            <img src="/image/image.png" alt="Gửi ảnh" />
          </label>
          <button type="button" class="icon-btn" @click="sendMessage">
            <img src="/image/send.png" alt="Gửi" />
          </button>
        </div>
      </footer>
      </section>
    </div>
  </div>
  </layout>
</template>
<script setup>
import layout from '@/components/Layout.vue'
import { ref, onMounted, nextTick, computed, watch } from 'vue'
import { getFriendList } from '@/service/friendService'
import { getMessageList,createMessage  } from '@/service/messageService'
import { getAccountById } from '@/service/authService'
import { getProfileByAccountId } from '@/service/profileService'

// import socket from '@/socket'

const selectedImage = ref(null)
const currentUserAvatar = ref('/image/avata.jpg')

// import socket from '@/socket'
import { getNotificationList } from '@/service/notificationService'
import { createNotification } from '@/service/notificationService' // thêm import

const notifications = ref({
  unread: [],
  new: [],
  old: []
})

const friends = ref([])
const messages = ref({})
const selectedFriend = ref(null)
const chatBody = ref(null)
const activeTab = ref('all')
const isMobile = ref(window.innerWidth <= 426)
const showSidebar = ref(true)
const newMessage = ref('') 
const searchQuery = ref('')


const currentUserId = JSON.parse(localStorage.getItem('user'))?.id
const currentUser = ref({ id: null, name: 'Ẩn danh', avatar: '/image/avata.jpg' })
const currentMessages = computed(() => {
  return messages.value[selectedFriend.value?.id] || []
})

const filteredFriends = computed(() =>
  friends.value.filter((f) =>
    f.name.toLowerCase().includes(searchQuery.value.trim().toLowerCase())
  )
)
const handleImageChange = (event) => {
  const file = event.target.files[0]
  if (!file) return

  selectedImage.value = file
  sendMessage() // tự động gửi khi chọn ảnh
}
const handleResize = () => {
  isMobile.value = window.innerWidth <= 432
  if (!isMobile.value) showSidebar.value = true
}
window.addEventListener('resize', handleResize)

watch(selectedFriend, async (newVal) => {
  if (isMobile.value && newVal) showSidebar.value = false
  if (newVal) {
    await fetchMessages(newVal.id)
    socket.emit('seen message', {
      senderId: newVal.id,
      receiverId: currentUserId
    })
    nextTick(scrollToBottom)
  }
})
const switchTab = (tab) => {
  activeTab.value = tab
}

const indicatorStyle = computed(() => {
  const index = activeTab.value === 'all' ? 0 : 1
  return {
    transform: `translateX(${index * 100}%)`,
    width: '50%'
  }
})
const isLastSentByMe = (index) => {
  const list = messages.value[selectedFriend.value.id]
  if (!list || !list.length) return false

  // Nếu tin hiện tại không phải từ mình thì bỏ qua
  if (!list[index].fromMe) return false

  // Tìm vị trí cuối cùng mà mình gửi
  let lastIndexFromMe = -1
  for (let i = list.length - 1; i >= 0; i--) {
    if (list[i].fromMe) {
      lastIndexFromMe = i
      break
    }
  }

  // Kiểm tra có tin nào của người kia gửi sau tin đó không
  for (let i = lastIndexFromMe + 1; i < list.length; i++) {
    if (!list[i].fromMe) {
      return false
    }
  }

  return index === lastIndexFromMe
}

const scrollToBottom = () => {
  if (chatBody.value) {
    chatBody.value.scrollTop = chatBody.value.scrollHeight
  }
}

const selectFriend = (friend) => {
  selectedFriend.value = friend
}
const fetchMessages = async (friendId) => {
  try {
    if (!currentUserId || !friendId) {
      console.warn('❌ senderId hoặc receiverId bị thiếu')
      return
    }

    const msgList = await getMessageList(currentUserId, friendId)

    // Sắp xếp tin nhắn theo thời gian tăng dần
    const sorted = msgList.sort((a, b) => new Date(a.createdAt) - new Date(b.createdAt))

      messages.value[friendId] = sorted.map((msg) => ({
        id: msg.id,
        fromMe: msg.senderId === currentUserId,
        text: msg.content,
        image: msg.imageUrl || msg.mediaUrl || '',
        createdAt: msg.createdAt,
        seen: msg.seen || false // 👈 thêm thuộc tính seen
      }))
    await nextTick()
    scrollToBottom()

    //  Gửi tín hiệu đã xem ngay nếu đang xem đoạn chat
    if (selectedFriend.value?.id === friendId) {
      socket.emit('seen message', {
        senderId: friendId,
        receiverId: currentUserId
      })

      //  Đánh dấu seen ngay tại client (để không cần đợi server phản hồi)
      messages.value[friendId] = messages.value[friendId].map((msg) => ({
        ...msg,
        seen: msg.fromMe ? true : msg.seen
      }))
    }
  } catch (err) {
    console.error('❌ Lỗi khi lấy tin nhắn:', err)
  }
}

onMounted(async () => {
  try {
    const rawUser = localStorage.getItem('user')
    if (!rawUser) return
    const currentUserId = JSON.parse(rawUser)?.id
    const account = await getAccountById(currentUserId)
    const profile = await getProfileByAccountId(currentUserId)

    currentUser.value.name = profile?.fullName?.trim()
      ? profile.fullName
      : account?.username || 'Ẩn danh'

    currentUser.value.avatar = profile?.avatarUrl?.trim()
      ? profile.avatarUrl
      : account?.avatar || '/image/avata.jpg'
    if (!currentUserId) return

    const res = await getFriendList(currentUserId)

    const friendInfoPromises = res.map(async (friend) => {
      try {
        const [account, profile] = await Promise.all([
          getAccountById(friend.id),
          getProfileByAccountId(friend.id)
        ])

        const name = profile?.fullName?.trim()
          ? profile.fullName
          : account?.username || 'Ẩn danh'

        const avatar = profile?.avatarUrl?.trim()
          ? profile.avatarUrl
          : account?.avatar || '/image/avata.jpg'

        return {
          id: friend.id,
          name,
          avatar,
          desc: 'Hãy bắt đầu trò chuyện!',
          online: true
        }
      } catch (err) {
        console.warn(` Lỗi khi lấy thông tin người dùng ${friend.id}:`, err)
        return {
          id: friend.id,
          name: 'Ẩn danh',
          avatar: '/image/avata.jpg',
          desc: 'Không thể hiển thị người dùng',
          online: false
        }
      }
    })

    friends.value = await Promise.all(friendInfoPromises)
  } catch (err) {
    console.error('Không thể lấy danh sách bạn bè:', err)
  }
})
const sendMessage = async () => {
  if (!newMessage.value.trim() && !selectedImage.value) return
  if (!selectedFriend.value) return

  const formData = new FormData()
  formData.append('senderId', currentUserId)
  formData.append('receiverId', selectedFriend.value.id)
  formData.append('content', newMessage.value || '')
  if (selectedImage.value) {
    formData.append('image', selectedImage.value)
      console.log('Hình ảnh gửi đi:', selectedImage.value)
  }

  try {
    const sent = await createMessage(formData)
    console.log('📥 Server trả về tin nhắn:', sent)
    await createNotification({
      actorId: currentUserId,
      receptorId: selectedFriend.value.id,
      content: `Đã gửi tin nhắn mới đến cho bạn`,
      objectType: 'MESSAGE'
    })

    if (!messages.value[selectedFriend.value.id]) {
      messages.value[selectedFriend.value.id] = []
    }

      messages.value[selectedFriend.value.id].push({
        id: sent.id,
        fromMe: true,
        text: sent.content,
        image: sent.imageUrl || '',
        createdAt: sent.createdAt,
        seen: false // mặc định chưa xem
      })
    newMessage.value = ''
    selectedImage.value = null
    await nextTick()
    setTimeout(() => scrollToBottom(), 50)
  } catch (err) {
    console.error('Lỗi khi gửi tin nhắn:', err)
  }
}

socket.on('chat messsage', (data) => {
  const { message, from } = data
  console.log('📥 Tin nhắn realtime:', message)

  if (!messages.value[from]) {
    messages.value[from] = []
  }

  messages.value[from].push({
    id: message.id,
    fromMe: false,
    text: message.content,
    image: message.imageUrl || '',
    createdAt: message.createdAt,
    seen: false
  })

  // Nếu đang mở đoạn chat này thì đánh dấu đã xem
if (selectedFriend.value?.id === from) {
  console.log('👁 Đang trong đoạn chat, gửi seen message')

  socket.emit('seen message', {
    senderId: from,
    receiverId: currentUserId
  })

  const updated = messages.value[from].map((msg) => {
    if (msg.fromMe) return { ...msg, seen: true }
    return msg
  })
  messages.value = {
    ...messages.value,
    [from]: updated
  }

  nextTick(scrollToBottom)
}
})
onMounted(() => {
  const userId = JSON.parse(localStorage.getItem('user'))?.id
  if (userId) {
    socket.emit('join room', `user_${userId}`)

    // socket.on('chat messsage', (data) => {
    //   const { message, from } = data
    //   console.log('Tin nhắn realtime:', message)

    //   if (!messages.value[from]) {
    //     messages.value[from] = []
    //   }

    //   messages.value[from].push({
    //     id: message.id,
    //     fromMe: false,
    //     text: message.content,
    //     image: message.imageUrl || '',
    //     createdAt: message.createdAt
    //   })

    //   if (selectedFriend.value?.id === from) {
    //     nextTick(scrollToBottom)
    //   }
    // })

    // CHÈN Ở ĐÂY
    socket.on('update-user-status', ({ userId, online }) => {
      const index = friends.value.findIndex((f) => f.id === Number(userId));
      if (index !== -1) {
        friends.value[index].online = online;
        friends.value = [...friends.value]; // Bắt buộc để force re-render
      }
    });
    socket.on('message seen', ({ senderId }) => {
      console.log('✅ Tin nhắn đã được xem bởi:', senderId)
      if (messages.value[senderId]) {
        const updated = messages.value[senderId].map((msg) => {
          if (msg.fromMe) return { ...msg, seen: true }
          return msg
        })

        //  Nếu đang mở đúng đoạn chat, thì ép Vue cập nhật lại đúng phần hiển thị
        if (Number(selectedFriend.value?.id) === Number(senderId)) {
          messages.value = {
            ...messages.value,
            [senderId]: updated
          }
        } else {
          // Nếu không phải đoạn chat đang mở, chỉ update ở memory
          messages.value[senderId] = updated
        }
      }
    })
  }
})

</script>
<style scoped>
.chat-app {
  display: flex;
  height: 100vh;
  font-family: 'Segoe UI', roboto;
  background: #FFFFFF;
}
.left-bar {
  width: 60px;
  background: #F9F9F9;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 32px 6px;
  border-right: 2px solid #ddd;
  height: 260 px;
  position: relative;
  border-radius: 10px;
  margin-left:-54px;
  margin-top: 72px;;
  box-shadow: 0px 4px 15px rgba(0, 0, 0, 0.1);
}
.msg.image-only {
  background: transparent !important;
  box-shadow: none !important;
  padding: 0 !important;
  border-radius: 0 !important;
}

.msg.image-only img.msg-image {
  border-radius: 12px;
  max-width: 240px;
  height: auto;
}
.msg-image {
  max-width: 200px;
  max-height: 200px;
  border-radius: 10px;
  margin-bottom: 4px;
}

.left-bar::before {
  content: "";
  position: absolute;
  top: -40px;
  left: 50%;
  transform: translateX(-50%);
  /* background-image: url('/image/logoPetGram.png'); */
  background-repeat: no-repeat;
  background-position: top center;
  background-size: 60px 40px;
  width: 60px;
  height: 60px;
}


.icon-img {
  width: 24px;
  height: 24px;
  cursor: pointer;
  opacity: 0.9;
  transition: 0.2s;
  margin-block: 2px;  /* thêm 2px khoảng trên/dưới mỗi icon */
}
.icon-img:hover {
  opacity: 1;
  transform: scale(1.05);
}

.nav-icons {
  display: flex;
  flex-direction: column;
  gap: 8px;       /* trước là 20px, giờ giảm xuống 8px */
}
.icon {
  font-size: 20px;
  cursor: pointer;
}
.avatar-icon {
  width: 35px;
  height: 35px;
  border-radius: 50%;
  margin-top: auto;
  margin-top: 20px;
}
.right-main {
  display: flex;
  flex: 1;
  background: #F9F9F9;
  margin-left:10px;
}
.sidebar {
  width: 290px;
  height:100%;
  margin-right:0;
  background: #F9F9F9;
  padding: 10px;
  display: flex;
  flex-direction: column;
  box-shadow: 0px 4px 15px rgba(0, 0, 0, 0.3);
}
.search-bar {
  position: relative;
  display: flex;
  align-items: center;
  background-color: #FFFFFF;
  border: 1px solid #ccc;
  border-radius: 10px;
  padding: 6px 12px;
  height: 35px;
  gap: 8px;
}

.search-bar img {
  width: 18px;
  height: 18px;
  opacity: 0.6;
}

.search-bar input {
  border: none;
  outline: none;
  flex: 1;
  font-size: 14px;
  background: transparent;
}

.tab-section {
  display: flex;
  position: relative;
  border-bottom: 1px solid #ddd;
  margin: 20px 0;
}
.tab-btn {
  flex: 1;
  text-align: center;
  padding: 10px 0;
  background: transparent;
  border: none;
  cursor: pointer;
  font-size: 14px;
  color: #666;
  position: relative;
  z-index: 1;
  transition: color 0.3s;
}

.tab-btn.active {
  color: #000;
  font-weight: bold;
}

.tab-indicator {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 3px;
  background-color: #FFA500;
  width: 50%;
  transition: transform 0.3s ease;
  border-radius: 2px;
}
/* Block chứa logo */
.logo-section {
  width: 60px;
  height: 60px;
  margin-bottom: 30px;      /* khoảng cách đến nav-icons */
  display: flex;
  align-items: center;
  justify-content: center;
}

.logo-img {
  width: 40px;
  height: 40px;
  object-fit: contain;
  margin-right: -12px;

}

.all-msg-btn {
  width: 100%;
  padding: 10px;
  background: #ffcc80;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}
.friend-section h3,
.stranger-section h3 {
  margin: 15px 0 10px;
  font-size: 14px;
}
.friend-item,
.stranger-item {
  display: flex;
  align-items: center;
  padding: 8px;
  border-radius: 8px;
  cursor: pointer;
}

.friend-item.active {
  background: #ffe0b2;
}
.avatar {
  width: 40px;
  height: 40px;
   object-fit: cover; 
  border-radius: 50%;
}
.friend-info,
.stranger-item > div {
  margin-left: 10px;
}
.name {
  font-weight: 600;
  font-size: 14px;
}
.desc {
  font-size: 12px;
  color: #555;
}
.main-chat {
  flex: 1;
  display: flex;
  flex-direction: column;
  border-left: 2px solid black;
}
.chat-header {
  padding: 20px;
  height:55px;
  display: flex;
  align-items: center;
  gap: 10px;
  border-bottom: 2px solid black;
}
.status {
  font-size: 12px;
  color: green;
}
.chat-body {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}
.msg {
  max-width: 60%;
  padding: 10px;
  border-radius: 10px;
}
.msg-wrapper {
  display: flex;
  margin-bottom: 10px;
  gap: 8px;
}

.avatar-msg {
  width: 36px;
  height: 36px;
   object-fit: cover; 
  border-radius: 50%;
  margin: 0 8px 4px 8px;
    flex-shrink: 0;
}
/* .align-right {
  display: flex;
  justify-content: flex-end;
} */
.align-left {
  display: flex;
  justify-content: flex-start;
}
.from-me {
  align-self: flex-end;
  background: #3b6eee;
  color: white;
}
.from-other {
  align-self: flex-start;
  background: #cecaca;
}
.msg-img {
  width: 200px;
  border-radius: 10px;
  object-fit: cover;
  border: none; 
  background: none;
  padding: 0;
}
.msg-image-wrapper {
  margin: 8px 0;
}
.msg-text {
  margin: 0;
}
.chat-input {
  display: flex;
  align-items: center;
  padding: 10px 15px;
  background-color: #DCD9D4;
}

.chat-input input {
  flex: 1;
  padding: 10px 12px;
  border-radius: 6px;
  border: none;
  outline: none;
  font-size: 14px;
  background: #DCD9D4;
}

.chat-actions {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-left: 12px;
}
.chat-actions img {
  width: 20px;
  height: 20px;
  cursor: pointer;
  opacity: 0.8;
  transition: transform 0.2s ease, opacity 0.2s;
}
.chat-input button {
  /* margin-left: 10px;
  padding: 0 12px; */
  padding-right:20px;
  background: #DCD9D4;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}
.main-placeholder {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #999;
  font-size: 18px;
}
.online-dot {
  display: inline-block;
  width: 8px;
  height: 8px;
  background-color: #00cc00;
  border-radius: 50%;
  margin-left: 6px;
}

.status {
  font-size: 12px;
  color: gray;
}
.status.online {
  color: green;
}
.icon-btn {
  background: transparent;
  border: none;
  padding: 0;
  cursor: pointer;
  display: flex;
  align-items: center;
}

.icon-btn img {
  width: 20px;
  height: 20px;
  opacity: 0.8;
  transition: opacity 0.2s, transform 0.2s;
}
.msg-block {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  max-width: 60%;
}
.align-right .msg-block {
  align-items: flex-end;
}

.align-right {
  justify-content: flex-end;
}
.msg {
  padding: 10px;
  border-radius: 12px;
  word-wrap: break-word;
  max-width: 100%;
}
.msg-status-outside {
  font-size: 12px;
  color: gray;
  margin-top: 2px;
}
@media (max-width: 432px) {
  .chat-app {
    display: flex;
    flex-direction: row;
    height: 100vh;
    overflow: hidden;
  }

  .left-bar {
  width: 60px;
  background: #F9F9F9;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px 0;
  border-right: 2px solid #ddd;
  border-radius: 10px;
  margin-left: 10px;
  
}
  .left-bar::before {
    bottom: 20px;
  }



  .right-main {
    flex: 1;
    display: flex;
    flex-direction: column;
    margin-left: 0;
    height: 100vh;
    overflow: hidden;
  }

  .sidebar {
    width: 100%;
    border-right: none;
    border-bottom: 1px solid #ccc;
    overflow-y: auto;
    flex-shrink: 0;
  }

  .main-chat {
    width: 100%;
    height: 100%;
    flex: 1;
    display: flex;
    flex-direction: column;
  }

  .chat-body {
    flex: 1;
    padding: 10px;
    overflow-y: auto;
    min-height: 0;
  }

  .chat-input {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 10px;
    padding: 10px;
    flex-shrink: 0;
    background-color: #DCD9D4;
  }

  .chat-actions {
    display: flex;
    align-items: center;
    gap: 1px;
    margin-left: 10px;
  }

  .chat-header {
    padding: 10px;
    flex-shrink: 0;
  }

  .back-btn {
    background: none;
    border: none;
    font-size: 20px;
    margin-right: 10px;
    cursor: pointer;
  }

  .back-icon {
    width: 20px;
    height: 20px;
    opacity: 0.8;
    transition: opacity 0.2s;
  }

  .back-icon:hover {
    opacity: 1;
  }
}
</style>