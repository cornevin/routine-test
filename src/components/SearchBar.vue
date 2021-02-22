<template>
  <div>
    <div
      class="searchBar"
      ref="input"
      contenteditable 
      v-html="parsedHtml" 
      @input="onInput"
    />

    <div v-if="showResults" class="results">
      <div 
        v-for="(result, id) in resultList" 
        :key="id" 
        class="resultItem"
        @click="onResultClick(result)"
      >
        {{ result.phrase }}
      </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable no-unused-vars */
import axios from 'axios'

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
  data() {
    return {
      isSearching: false,
      /**
       * List of all the results fetched according to what the user typed
       */
      resultList: [],
      showResults: false,
    }
  },
  methods: {
    /**
     * Called on every input of the user 
     */
    onInput(event) {
      const text = event.target.textContent
      
      // Vue automaticaly cropped the white space, so we need to add it manually
      if (event.data === ' ') {
        this.$emit('change', `${this.search}&nbsp;`)
      } else {
        this.$emit('change', text)
      }

      const myRegexp = new RegExp('(?<=I pick you).*', 'i')

      if (myRegexp.test(text)) {
        this.showResults = true

        axios.get(`https://bleeding.routine.co:8080/completion?query=${myRegexp.exec(text)[0].trim()}`)
          .then(res => { this.resultList = res.data.completions })
      }
    },
    /**
     * Called when the user click on a result
     */
    onResultClick(result) {
      const { search } = this
      this.showResults = false
      
      const myRegexp = new RegExp('(?<=I pick you).*', 'i')
      this.$emit('change', `${search.split(myRegexp)} ${result.phrase}`)
    }
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
          return `<div class="pick">${match}</div>`
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
  height: 71px;
  box-sizing: border-box;
  display: flex;
  padding: 16px 24px;
  background: #F6F3EE;
  border-radius: 5px;
  width: 800px;
  margin: 0 auto;
  font-family: GT America;
  font-style: normal;
  font-weight: normal;
  font-size: 26px;
  line-height: 150%;
}

.pick {
  font-family: GT America;
  font-style: normal;
  font-weight: normal;
  font-size: 26px;
  line-height: 150%;
  color: #4754dc;
}

.highlightText {
  padding: 2px 10px;
  font-family: GT America;
  font-style: normal;
  font-weight: 600;
  font-size: 22px;
  line-height: 160%;
  background: rgba(68, 68, 68, 0.1);
  border-radius: 3px;
}

.results {
  padding: 16px 32px 32px 16px;
  margin: 0 auto;
  width: 500px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  padding: 16px;
  background: #F6F3EE;
  border: 1px solid #D3CDB8;
  box-sizing: border-box;
  box-shadow: 0px 1px 14px rgba(0, 0, 0, 0.05);
  border-radius: 3px;
}

.resultItem {
  width: calc(100% - 28px);
  margin-right: auto;
  display: flex;
  padding: 8px 14px 10px 14px;
  margin: 4px 0px;
  font-family: GT America;
  font-style: normal;
  font-weight: 600;
  font-size: 14px;
  line-height: 120%;
  line-height: 120%;
}

.resultItem:hover {
  background: rgba(68, 68, 68, 0.1);
  border-radius: 1px;
}
</style>
