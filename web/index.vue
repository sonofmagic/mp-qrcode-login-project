<template>
  <SomPopup
    v-model="visible"
    title="登录"
    @closed="closed"
    @open="open"
    @close="close"
  >
    <div class="w-[350px] flex flex-col items-center">
      <div v-show="!toggle">
        <div class="flex justify-between mb-4 text-sm">
          请使用微信扫一扫登录
          <span class="hover:underline hover:text-primary-400" @click="refresh"
            >刷新<i class="el-icon-refresh-right"></i
          ></span>
        </div>
        <van-image
          v-loading="qrcodeLoading"
          width="188"
          height="188"
          :src="qrcodeUrl"
        >
          <template #loading>
            <van-loading type="spinner" size="20" />
          </template>
        </van-image>
      </div>

      <SignIn v-show="toggle" ref="signin" @success="visible = false"></SignIn>
      <div class="mt-6 underline" @click="toggleLoginType">
        切换{{ toggle ? '小程序码' : '账号密码' }}登录
      </div>
    </div>
  </SomPopup>
</template>

<script>
// 大概就是这样
import { io } from 'socket.io-client'
import { getBlogLoginQrcode } from '@/api/article'
export default {
  name: 'SignInPopup',
  components: {
    SignIn,
  },
  props: {
    value: {
      type: [Boolean],
      default: false,
    },
  },
  data() {
    return {
      qrcodeUrl: '',
      qrcodeLoading: false,
      toggle: false,
    }
  },
  computed: {
    visible: {
      get() {
        return this.value
      },
      set(nv) {
        this.$emit('input', nv)
      },
    },
  },
  methods: {
    toggleLoginType() {
      this.toggle = !this.toggle
    },
    refresh() {
      this.getQrcode(this.socketId)
    },
    async getQrcode(scene) {
      try {
        this.qrcodeLoading = true
        const { data: imgBuf } = await getBlogLoginQrcode(scene)
        this.qrcodeUrl = imgBuf
      } catch (err) {
        console.error(err)
      } finally {
        this.qrcodeLoading = false
      }
    },
    close() {},
    closed() {
      if (this.socket) {
        this.socket.close()
      }
      this.clearValidate('signin')
    },
    open() {
      const socket = (this.socket = io('http://127.0.0.1:9000'))
      socket.on('connect', () => {
        this.socketId = socket.id // x8WIv7-mJelg7on_ALbx
        this.getQrcode(this.socketId)
      })
      socket.on('success', (userinfo) => {
        console.log(userinfo)
        this.$store.dispatch('cache/setWsTempData', userinfo)
        this.$router.push('/ws-login-success')
        this.$message.success('微信登录成功!')
        this.visible = false
      })
      socket.on('disconnect', () => {
        this.socketId = socket.id // undefined
        console.log(socket.id, socket.connected)
      })
    },
    clearValidate(refName) {
      if (this.$refs[refName]) {
        this.$refs[refName].clearValidate()
      }
    },
  },
}
</script>
