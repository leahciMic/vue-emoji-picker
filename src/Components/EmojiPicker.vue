<template>
  <div>
    <slot
      name="emoji-invoker"
      :events="{ click: toggle }"
    ></slot>
    <div
      v-if="visible"
      v-click-outside="hide"
    >
      <slot
        name="emoji-picker"
        :emojis="emojis"
        :insert="insert"
        :display="display"
      ></slot>
    </div>
  </div>
</template>

<script>
  import emojis from '../emojis'

  export default {
    model: {
      prop: 'visible',
      event: 'toggleVisible',
    },
    props: {
      visible: {
        type: Boolean,
        default: false,
      },
      search: {
        type: String,
        required: false,
        default: '',
      },
      emojiTable: {
        type: Object,
        required: false,
        default() {
          return emojis
        },
      },
    },
    data() {
      return {
        display: {
          x: 0,
          y: 0,
        },
      }
    },
    computed: {
      emojis() {
        if (this.search) {
          const obj = {}

          for (const category in this.emojiTable) {
            obj[category] = {}

            for (const emoji in this.emojiTable[category]) {
              if (new RegExp(`.*${this.search}.*`).test(emoji)) {
                obj[category][emoji] = this.emojiTable[category][emoji]
              }
            }

            if (Object.keys(obj[category]).length === 0) {
              delete obj[category]
            }
          }

          return obj
        }

        return this.emojiTable
      },
    },
    methods: {
      insert(emoji) {
        this.$emit('emoji', emoji)
      },
      toggle(e) {
        this.$emit('toggleVisible', !this.visible);
        this.display.x = e.clientX
        this.display.y = e.clientY
      },
      hide() {
        this.$emit('toggleVisible', false);
      },
      escape(e) {
        if (this.visible === true && e.keyCode === 27) {
          this.hide();
        }
      },
    },
    directives: {
      'click-outside': {
        bind(el, binding, vNode) {
          if (typeof binding.value !== 'function') {
            return
          }

          const bubble = binding.modifiers.bubble
          const handler = (e) => {
            if (bubble || (! el.contains(e.target) && el !== e.target)) {
              binding.value(e)
            }
          }
          el.__vueClickOutside__ = handler

          (vNode.context.$window || document).addEventListener('click', handler)
        },
        unbind(el, binding, vNode) {
          (vNode.context.$window || document).removeEventListener('click', el.__vueClickOutside__)

          el.__vueClickOutside__ = null
        },
      },
    },
    mounted() {
      (this.$window || document).addEventListener('keyup', this.escape)
    },
    destroyed() {
      (this.$window || document).removeEventListener('keyup', this.escape)
    },
  }
</script>
