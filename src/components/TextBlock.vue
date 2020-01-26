<template>
    <div>
        <div ref="ed" class="editor" @keydown.tab.prevent="tab" @input="getInput" @keyup="getCurrentContext" spellcheck="false" contenteditable="true">
        </div>
    </div>
</template>

<script>
import uuidv1 from 'uuid/v1';
export default {
  name: 'TextBlock',
  data() {
    return {
      html: '',
      arrayOfWords: '',
      node: '',
      space: '',
      spaceText: '',
      el: '',
      elText: '',
      container: '',
      id: '',
      length: 0,
      currentText: ''
    };
  },
  methods: {
    tab() {
      /* ------------------------- get current node of DOM ------------------------ */
      this.node = this.getSelectionElement();
      /* ------------------- parse array of words base on space between each word ------------------- */
      this.arrayOfWords = this.getArrayOfWords();
      /* ------------------- returns object of command and words ------------------ */
      this.snapshot = this.getSnapshot(this.arrayOfWords, this.node);
      /* --------------------- creates html div for container --------------------- */
      this.container = document.createElement('div');
      /* -------------- iterates over all words and creates new html -------------- */
      this.snapshot.forEach(item => {
        this.el = document.createElement(item.command);
        this.elText = document.createTextNode(item.text);
        this.el.appendChild(this.elText);
        this.el.id = 'id-' + uuidv1();
        this.container.appendChild(this.el);
        this.getIdAndLengthOfWord(item);
      });
      /* ------------------------ update DOM with new html ------------------------ */
      this.syncDom();
    },
    syncDom() {
      if (this.node.className == 'editor') {
        this.node.innerHTML = this.container.innerHTML;
        this.setPos(this.id, this.length);
      } else {
        this.node.outerHTML = this.container.innerHTML;
        this.setPos(this.id, this.length);
      }
    },
    getArrayOfWords() {
      return this.node.firstChild.textContent.split(' ');
    },
    getIdAndLengthOfWord(item) {
      if (item.command.indexOf('h') == -1) {
        this.space = document.createElement('span');
        this.spaceText = document.createTextNode(' ');
        this.space.appendChild(this.spaceText);
        this.space.id = 'id-' + uuidv1();
        this.container.appendChild(this.space);
        this.id = this.space.id;
        this.length = this.space.textContent.length;
      } else {
        this.id = this.el.id;
        this.length = this.el.textContent.length;
      }
    },
    getSnapshot(raw, node) {
      let word;
      let command;
      let snapshot = [];
      raw.forEach((item, i) => {
        command = node.localName == 'div' ? 'span' : node.localName;
        let length = item.length;
        let matchIndex = item.indexOf('`');
        if (matchIndex !== -1) {
          word = item.substr(0, matchIndex);
          command = item.substr(matchIndex + 1, length);
          item = word;
        }
        if (command == '') {
          command = 'span';
        }
        snapshot[i] = { text: item, command: command };
      });
      return snapshot;
    },
    getSelectionElement() {
      var selection = window.getSelection();
      var container = selection.anchorNode;
      if (container.nodeType !== 3) {
        return container;
      } else {
        // return parent if text node
        return container.parentNode;
      }
    },
    getIfNotEmpty(collection) {
      if (collection.length > 0) {
        return collection;
      }
      return 'empty array';
    },
    getFirstIfNotEmpty(collection) {
      if (collection.length > 0) {
        return collection[0];
      }
      return 'empty array';
    },
    getInput(e) {
      this.html = e.target.innerHTML;
    },
    setPos(context, length) {
      var node = document.querySelector('#' + context);
      node.focus();
      var textNode = node.firstChild;
      var caret = length; // insert caret after the 10th character say
      var range = document.createRange();
      range.setStart(textNode, caret);
      range.setEnd(textNode, caret);
      var sel = window.getSelection();
      sel.removeAllRanges();
      sel.addRange(range);
    }
  },
  beforeDestroy() {
    this.$refs = null;
  }
};
</script>

<style>
span,
p,
h1,
h2,
h3,
h4,
h5 {
  line-height: 0rem;
}

.editor {
  margin: 0;
  padding: 0;
  width: 100%;
  white-space: pre-wrap;
  line-break: loose;
  overflow-x: hidden;
  word-wrap: break-word;
  min-height: 20px;
  background-color: whitesmoke;
}
</style>