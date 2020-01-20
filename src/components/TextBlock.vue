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
      currentText: '',
      id: '',
      index: {
        lines: 0,
        snapshot: [{}]
      }
    };
  },
  methods: {
    tab() {
      let node = this.getSelectionElement();
      let raw = node.firstChild.textContent.split(' ');
      let snapshot = this.getSnapshot(raw, node);
      let el;
      let text;
      let free;
      let textfree;
      let master = document.createElement('div');
      snapshot.forEach(item => {
        el = document.createElement(item.command);
        text = document.createTextNode(item.text);
        free = document.createElement('span');
        textfree = document.createTextNode(' ');
        el.appendChild(text);
        free.appendChild(textfree);
        free.id = 'id-' + uuidv1();
        master.appendChild(el);
        master.appendChild(free);
      });
      if (node.className == 'editor') {
        node.innerHTML = master.innerHTML;
        this.setPos(free.id, free.textContent.length);
      } else {
        node.outerHTML = master.innerHTML;
        this.setPos(free.id, free.textContent.length);
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
    getCaretPosOffset(node) {
      if (node.className == 'editor') {
        return 0;
      } else {
        return 0;
      }
    },
    getCurrentContext() {
      let editor = this.$refs.ed;
      this.index.lines = editor.childNodes.length;
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
    updateDOM(new_html, e) {
      this.html = new_html;
      e.target.innerHTML = new_html;
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
    },
    getPos(element) {
      var doc = element.ownerDocument || element.document;
      var win = doc.defaultView || doc.parentWindow;
      var sel;
      if (typeof win.getSelection != 'undefined') {
        sel = win.getSelection();
        if (sel.rangeCount > 0) {
          var range = win.getSelection().getRangeAt(0);
          var preCaretRange = range.cloneRange();
          preCaretRange.selectNodeContents(element);
          preCaretRange.setEnd(range.startContainer, range.startOffset);
          this.pos.start = preCaretRange.toString().length;
          preCaretRange.setEnd(range.endContainer, range.endOffset);
          this.pos.end = preCaretRange.toString().length;
        }
      } else if ((sel = doc.selection) && sel.type != 'Control') {
        var textRange = sel.createRange();
        var preCaretTextRange = doc.body.createTextRange();
        preCaretTextRange.moveToElementText(element);
        preCaretTextRange.setEndPoint('EndToStart', textRange);
        this.pos.start = preCaretTextRange.text.length;
        preCaretTextRange.setEndPoint('EndToEnd', textRange);
        this.pos.end = preCaretTextRange.text.length;
      }
      return { start: this.pos.start, end: this.pos.end };
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
  display: inline-block;
}
.editor {
  min-height: 20px;
  white-space: pre-wrap;
  background-color: rgb(255, 247, 207);
}
</style>