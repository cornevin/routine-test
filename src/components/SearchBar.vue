<template>
  <div
    class="searchBar"
    ref="input"
    contenteditable 
    v-html="parsedHtml" 
    @input="onInput"
  />
</template>

<script>
/* eslint-disable no-unused-vars */
// import axios from 'axios'

export default {
  name: 'SearchBar',
  model: {
    prop: 'search',
    event: 'change'
  },
  props: {
    search: {
      type: String,
      default: null,
    },
  },
  methods: {
    /**
     * Called on every input of the user 
     */
    onInput(event) {
      // Vue automaticaly cropped the white space, so we need to add it manually
      if (event.data === ' ') {
        this.$emit('change', `${this.search}&nbsp;`)
      } else {
        this.$emit('change', event.target.textContent)
      }
    },
  },
  computed: {
    /**
     * Formatted HTML after looking for the regexp 
     */
    parsedHtml() {
      const re = new RegExp('I pick you', 'i')

      let html = this.search.replace(new RegExp('(?<=I pick you).*', 'i'), match => {
        return `&#32<span class="highlightText">${match.trim()}</span>`
      })

      if (html) {
        html = html.replace(re, match => {
          return `<b>${match}</b>`
        })
      }
      
      return html
    }
  },
  watch: {
    search: {
      immediate: true,
      /**
       * Because Vue render the div on every change of 'search', we need to manually place the carret
       * otherwise the UX become a nightmare
       */
      handler() {
        const selection = window.getSelection()
        const { input } = this.$refs

        this.$nextTick(() => {
          if (input && document.activeElement === input) {
            const range = document.createRange()

            if (input.childNodes.length === 1) {
              range.setStart(input.childNodes[0], input.childNodes[0].textContent.length)
            } else {
              range.setStart(input, input.childNodes.length)
            }
            
            range.collapse(false)
            selection.removeAllRanges()
            selection.addRange(range)
          }
        })
      },
    }
  }
}
</script>

<style>
.searchBar {
  padding: 16px 24px;
  background: #F6F3EE;
  border-radius: 5px;
  width: 500px;
  margin: 0 auto;
}

.highlightText {
  background: yellow;
}
</style>
