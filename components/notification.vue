<template>
  <div ref="notification" class="rounded py-2 px-4 mb-4 animated fadeInUp faster shadow" :class="className">
    <span class="select-none text-sm">
      {{ notification.message }}
    </span>
  </div>
</template>

<script>
export default {
  props: {
    notification: {
      type: Object,
      required: true
    },
    duration: {
      type: Number,
      required: false,
      default: 0
    }
  },
  computed: {
    className() {
      const types = {
        default: 'bg-grey-dark text-grey-light',
        success: 'bg-green-dark text-white border border-green',
        info: 'bg-blue-dark text-white border border-blue',
        warning: 'bg-orange-dark text-white border border-orange',
        error: 'bg-red-dark text-white border border-red'
      }
      return types[this.notification.type || 'default']
    }
  },
  mounted() {
    const duration = this.notification.duration || 5
    window.setTimeout(() => {
      this.$refs.notification.classList.add('fadeOutDown')
    }, duration * 1000)
  }
}
</script>
