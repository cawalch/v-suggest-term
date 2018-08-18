<template>
    <div class="term-menu-container">
        <div 
        class="term-menu-items-container"
        v-bind:style="menuPosition"
        ref="listContainer"
        >
            <ul :class="listClass">
                <li
                :class="[isSelected(index) ? termSelectedClass: '', termClass]"
                v-for="(result, index) in results"
                :key="index"
                @click="handleClick(index)"
                >
                {{result.name}}
                </li>
            </ul>
        </div>
        <slot name="input">
            <input type="text">
        </slot>
    </div>
</template>

<script>
  import { position, offset } from 'caret-pos'
  const MIN_SIZE = 3
  const STOP_WREG = new RegExp(/([\s\r\n:(])/)

  export default {
      name: 'vue-suggest-term',
      props: {
          filter: Function,
          fields: [Object, Array],
          limit: Number,
          listClass: {
              type: String,
              default: '',
          },
          termClass: {
              type: String,
              default: '',
          },
          termSelectedClass: {
              type: String,
              default: 'selected',
          },
      },
      computed: {
          menuPosition() {
                if (this.results.length === 0) {
                    return { display: 'none' }
                }
                const offs = offset(this.input)
                return {
                    display: 'block',
                    top: `${offs.top}px`,
                    left: `${offs.left}px`,
                }
            },
          results() {
              return this.filter(this.word, this.fields, this.limit)
          },
      },
      data() {
          return {
              input: undefined,
              selected: 0,
              word: undefined,
              lastKey: undefined,
          }
      },
      methods: {
          isSelected(index) {
              return index === this.selected
          },
          resetState() {
              this.word = undefined
              this.selected = 0
          },
          handleClick(index) {
              this.selectTerm(this.results[index])
          },
          handleKeyDown(evt) {
            this.lastKey = evt.key
            if (this.results.length === 0) {
                return
            }
            switch (evt.key) {
                case 'Backspace':
                case 'Escape':
                    this.resetState()
                    return
                case 'Tab':
                case 'ArrowDown':
                    if (this.selected < (this.results.length - 1)) {
                        this.selected += 1
                    } else {
                        this.selected = 0
                    }
                    evt.preventDefault()
                break
                case 'ArrowUp':
                    if (this.selected === 0) {
                        this.selected = this.results.length - 1
                    } else {
                        this.selected -= 1
                    }
                    evt.preventDefault()
                break
                case 'Enter':
                    this.selectTerm(this.results[this.selected])
                    evt.preventDefault()
                break
            }
          },
          selectTerm(term) {
            const tokens = this.input.value.split(STOP_WREG)
            if (tokens.length === 1) {
                tokens[0] = term.name
            } else {
                const { pos } = position(this.input)
                let offset = 0
                for (let i = 0; i < tokens.length; i += 1) {
                    offset += tokens[i].length
                    if (pos <= offset) {
                        tokens[i] = term.name
                    }
                }
            }
            this.$emit('selectTerm', tokens.join(''))
            this.resetState()
          },
          updateWord() {
            this.resetState()
            if (this.lastKey === 'Backspace') {
                return
            }
            const tokens = this.input.value.split(STOP_WREG)
            let newWord = undefined
            if (tokens.length === 1) {
                newWord = tokens[0]
            } else {
                const { pos } = position(this.input)
                let offset = 0
                for (let i = 0; i < tokens.length; i += 1) {
                    offset += tokens[i].length
                    if (pos <= offset) {
                        newWord = tokens[i]
                    }
                }
            }
            this.word = newWord.length >= MIN_SIZE ? newWord : undefined
          },
      },
      beforeDestroy() {
          this.input.removeEventListener('keydown', this.handleKeyDown)
          this.input.removeEventListener('input', this.updateWord)
      },
      mounted() {
          this.input = this.$el.children[1]
          this.input.addEventListener('keydown', this.handleKeyDown) 
          this.input.addEventListener('input', this.updateWord)
      },
  }
</script>

<style>
.term-menu-items-container {
    position: absolute;
    display: none;
}
.term-menu-items-container ul {
    list-style-type: none;
    padding: 0;
    margin: 0;
}
</style>