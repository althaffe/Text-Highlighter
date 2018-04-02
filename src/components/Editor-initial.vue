<template>
  <div id="editor" v-html="content"></div>
</template>

<script>
import rangy from '../../node_modules/rangy/lib/rangy-core.js';
export default {
  name: 'Editor',
  data () {
    return {
      editorId: 'editor',

       /* align: ['left'|'center'|'right']
       * When the __static__ option is true, this aligns the static toolbar
       * relative to the medium-editor element.
       */
      align: 'center',

      /* diffLeft: [Number]
       * value in pixels to be added to the X axis positioning of the toolbar.
       */
      diffLeft: 0,

      /* diffTop: [Number]
       * value in pixels to be added to the Y axis positioning of the toolbar.
       */
      diffTop: -10,
      sources: [],
      facts: [],
      content: `Immigration activists protest in Washington on March 5 in support of the Deferred Action for Childhood Arrivals (DACA) program\n\nProtections against deportation for some 700,000 undocumented immigrants who came to the US as children remain in place two days after the DACA program was to expire, the Department of Homeland Security announced Wednesday.\n\nDHA spokesman Tyler Houlton said that people under the Deferred Action for Childhood Arrivals can continue to renew their registrations with the US Citizenship and Immigration Services, earning a two-year extension of protection.\n\n"In compliance with court injunctions, USCIS is accepting and adjudicating DACA requests for renewals as they are submitted," Houlton said in a statement.\n\n"This process is in accordance with the longstanding DACA policies on renewals."\n\nLast September President Donald Trump ordered DACA to be shut down on March 5, threatening the status in the United States of 690,000 people registered under the program, so-called "Dreamers."\n\nBut a federal judge issued a nationwide injunction ordering the government to continue offering DACA protection. With the government's appeal of that injunction unlikely to be ruled on for months, DHS said DACA recipients could continue to renew their protections, though no one outside the program would be able to join.\n\nTrump, along with a majority of Democrats and Republicans in Congress, has repeatedly pledged to pass legislation that would offer paths to citizenship for DACA registrants and possibly more than a million other people who entered the country illegally as youths and grew up in the United States.\n\nTrump has offered a plan for eventual citizenship to a total of 1.8 million of these Dreamers.\n\nBut a series of bills on the issue have failed to pass Congress, with Democrats rejecting Trump's proposal to tie DACA legislation to a sharp cutback in legal immigration and to funding for a wall along the Mexico border.\n\nOn Wednesday, Trump said he still hopes for legislation to replace DACA.\n\n"We're trying to have a DACA victory," he said in a speech to the Latino Coalition, a conservative business group.\n\n"The Democrats are nowhere to be found... We're ready, willing and able," he said.`
    }
  },
  mounted() {
    rangy.init();
  },
  methods: {
    init: function () {
           document.body.appendChild(this.getToolbarElement());
           this.positionToolbarIfShown();

           this.sourceClassApplier = rangy.createClassApplier('data-source', {
                   elementTagName: 'span',
                   normalize: true,
                   elementAttributes: {'data-disable-editing': true},
                   onElementCreate: function(el) {
                     var id = 'source-' + uuid();
                     el.id = id;
                     saveSource(id);
                     el.addEventListener('click', function() {
                       Toolbar.selectText(id);
                       Toolbar.getToolbarElement().classList.add('source-options');
                       Toolbar.showAndUpdateToolbar();
                     });
                   }
               });

           this.factClassApplier = rangy.createClassApplier('data-fact', {
                   elementTagName: 'span',
                   normalize: true,
                   elementAttributes: {'data-disable-editing': true},
                   onElementCreate: function(el) {
                     var id = 'fact-' + uuid();
                     el.id = id;
                     saveFact(id);
                     el.addEventListener('click', function() {
                       Toolbar.selectText(id);
                       Toolbar.getToolbarElement().classList.add('fact-options');
                       Toolbar.showAndUpdateToolbar();
                       var fact = facts.filter(function(fact) {
                         return fact.id === id;
                       });
                     });
                   }
               });

       },

       // Toolbar creation/deletion

       createToolbar: function () {
           var toolbar = document.createElement('div');

           toolbar.className = 'medium-editor-toolbar';
           toolbar.className += ' medium-editor-stalker-toolbar';
           toolbar.appendChild(this.createToolbarButtons());

           this.attachEventHandlers();

           return toolbar;
       },

       createToolbarButtons: function () {
           var ul = document.createElement('ul'),
               li,
               btn,
               buttons,
               extension,
               buttonName,
               buttonOpts;

           ul.className = 'medium-editor-toolbar-actions';
           ul.style.display = 'block';

           var fact = document.createElement('li');
           fact.classList.add('fact-button');
           var source = document.createElement('li');
           source.classList.add('source-button');
           var remove = document.createElement('li');
           remove.classList.add('remove-button');
           var selectSource = document.createElement('li');
           selectSource.classList.add('select-source-button');
           var showFacts = document.createElement('li');
           showFacts.classList.add('show-facts-button');



           var factButton = document.createElement('button')
           factButton.classList.add('medium-editor-action');
           factButton.innerHTML = 'Mark Fact';

           var sourceButton = document.createElement('button')
           sourceButton.classList.add('medium-editor-action');
           sourceButton.innerHTML = 'Mark Source';

           var selectSourceButton = document.createElement('button')
           selectSourceButton.classList.add('medium-editor-action');
           selectSourceButton.innerHTML = 'Select Source';

           var showFactsButton = document.createElement('button')
           showFactsButton.classList.add('medium-editor-action');
           showFactsButton.innerHTML = 'Show Facts';

           var removeButton = document.createElement('button')
           removeButton.classList.add('medium-editor-action');
           removeButton.innerHTML = 'Remove';

           sourceButton.addEventListener('click', function() {
             Toolbar.sourceClassApplier.applyToSelection();
             Toolbar.hideToolbar();
           })

           factButton.addEventListener('click', function() {
             Toolbar.factClassApplier.applyToSelection();
             Toolbar.hideToolbar();
           })

           removeButton.addEventListener('click', function() {
             var el = Toolbar.getSelectionParentElement();
             if(el.className !== 'data-fact' && el.className !== 'data-source') return false;

             if(el.className === 'data-fact') {
                removeFact(el.id)
              } else if (el.className === 'data-source') {
                removeSource(el.id)
              }

             var parent = el.parentNode;

             // move all children out of the element
             while (el.firstChild) parent.insertBefore(el.firstChild, el);

             // remove the empty element
             parent.removeChild(el);

             Toolbar.hideToolbar();
           })

           selectSourceButton.addEventListener('click', function() {
             var factId = Toolbar.getSelectionParentElement().id;
             var sourceList = document.querySelectorAll('.data-source');
             var fact = facts.filter(function(fact) {return fact.id == factId;})[0];
             if(fact.source) var selectedSource = fact.source;
             sourceList.forEach(function(source) {
               source.classList.add('source-selectable');
               if (source.id == selectedSource) source.classList.add('source-selected');
             })

             var editor = document.getElementById(Toolbar.editorId);

             var detectSource = function (factId, event) {
               if(event.target.classList.contains('data-source')) {
               setTimeout(Toolbar.hideToolbar(), 300)

                 facts.forEach(function(fact) {
                   if (fact.id === factId) {
                     if (fact.source === event.target.id) {
                       delete fact.source;
                     } else {
                       fact.source = event.target.id;
                       event.target.classList.add('source-selected')
                     }
                   }
                 })
                 console.log(facts)
               }

               var sourceList = document.querySelectorAll('.data-source');
               sourceList.forEach(function(source) {
                 source.classList.remove('source-selectable', 'source-selected');
               })
               editor.removeEventListener('click', detectSource);
             }.bind(this, factId)

             editor.addEventListener('click', detectSource);

             Toolbar.hideToolbar();

           })

           showFactsButton.addEventListener('click', function() {
             var sourceId = Toolbar.getSelectionParentElement().id;
             facts.forEach(function(fact) {
               if(fact.source === sourceId) {
                 document.getElementById(fact.id).classList.add('fact-selected');
               }
               var editor = document.getElementById(Toolbar.editorId);

               var removeFactHighlight = function() {
                 var factList = document.querySelectorAll('.data-fact');
                 factList.forEach(function(fact) {
                   fact.classList.remove('fact-selected');
                 });
                 editor.removeEventListener('click', removeFactHighlight);
               }
               editor.addEventListener('click', removeFactHighlight);
             })
           })



           fact.appendChild(factButton);
           source.appendChild(sourceButton);
           remove.appendChild(removeButton);
           selectSource.appendChild(selectSourceButton);
           showFacts.appendChild(showFactsButton);

           ul.appendChild(fact);
           ul.appendChild(source);
           ul.appendChild(selectSource);
           ul.appendChild(showFacts);
           ul.appendChild(remove);

           return ul;
       },

       getToolbarElement: function () {
           if (!this.toolbar) {
               this.toolbar = this.createToolbar();
           }

           return this.toolbar;
       },

       attachEventHandlers: function () {

           var editor = document.getElementById(this.editorId);

           // Handle mouseup on editor for updating the selection in the toolbar
           editor.addEventListener('mouseup', this.handleDocumentMouseup.bind(this))

           // On resize, re-position the toolbar
           window.addEventListener('resize', this.handleWindowResize.bind(this));
       },

       handleWindowResize: function () {
           this.positionToolbarIfShown();
       },

       handleDocumentMouseup: function (event) {
           this.getToolbarElement().classList.remove('fact-options', 'source-options')

           setTimeout(function () {
               this.checkState();
           }.bind(this), 0);
       },

       // Hiding/showing toolbar

       isDisplayed: function () {
           return this.getToolbarElement().classList.contains('medium-editor-toolbar-active');
       },

       showToolbar: function () {
           if (!this.isDisplayed()) {
               this.getToolbarElement().classList.add('medium-editor-toolbar-active');
           }
       },

       hideToolbar: function () {
           if (this.isDisplayed()) {
               this.getToolbarElement().classList.remove('medium-editor-toolbar-active');
           }
       },

       checkState: function () {
           // If there's no selection element, selection element doesn't belong to this editor
           // or toolbar is disabled for this selection element
           // hide toolbar
           // var selectionElement = MediumEditor.selection.getSelectionElement(window);
           // if (!selectionElement ||
           //         this.getEditorElements().indexOf(selectionElement) === -1) {
           //     return this.hideToolbar();
           // }

           // If we don't have a 'valid' selection -> hide toolbar
           if (!this.selectionContainsContent()) {
               return this.hideToolbar();
           }

           this.showAndUpdateToolbar();
       },

       selectionContainsContent: function () {
           var sel = document.getSelection();

           // collapsed selection or selection withour range doesn't contain content
           if (!sel || sel.isCollapsed || !sel.rangeCount) {
               return false;
           }

           // if toString() contains any text, the selection contains some content
           if (sel.toString().trim() !== '') {
               return true;
           }

           return false;
       },

       // Updating the toolbar

       showAndUpdateToolbar: function () {
           var editor = document.getElementById(this.editorId);
           var event = new Event('positionToolbar');

           editor.dispatchEvent(event);
           this.showToolbar();
           this.setToolbarPosition();
       },

       // Positioning toolbar

       positionToolbarIfShown: function () {
           if (this.isDisplayed()) {
               this.setToolbarPosition();
           }
       },

       setToolbarPosition: function () {
           // var container = this.base.getFocusedElement(),
               var selection = window.getSelection();

           // If there isn't a valid selection, bail
           // if (!container) {
           //     return this;
           // }

           if (!selection.isCollapsed) {
               this.showToolbar();
               this.positionToolbar(selection);
           }
       },

       positionToolbar: function (selection) {
           // position the toolbar at left 0, so we can get the real width of the toolbar
           this.getToolbarElement().style.left = '0';
           this.getToolbarElement().style.right = 'initial';

           var range = selection.getRangeAt(0),
               boundary = range.getBoundingClientRect();

           // Handle selections with just images
           if (!boundary || ((boundary.height === 0 && boundary.width === 0) && range.startContainer === range.endContainer)) {
               // If there's a nested image, use that for the bounding rectangle
               if (range.startContainer.nodeType === 1 && range.startContainer.querySelector('img')) {
                   boundary = range.startContainer.querySelector('img').getBoundingClientRect();
               } else {
                   boundary = range.startContainer.getBoundingClientRect();
               }
           }

           var containerWidth = window.innerWidth,
               toolbarElement = this.getToolbarElement(),
               toolbarHeight = toolbarElement.offsetHeight,
               toolbarWidth = toolbarElement.offsetWidth,
               halfOffsetWidth = toolbarWidth / 2,
               buttonHeight = 50,
               defaultLeft = this.diffLeft - halfOffsetWidth,
               elementsContainer = document.body,
               elementsContainerAbsolute = ['absolute', 'fixed'].indexOf(window.getComputedStyle(elementsContainer).getPropertyValue('position')) > -1,
               positions = {},
               relativeBoundary = {},
               middleBoundary, elementsContainerBoundary;

           // If container element is absolute / fixed, recalculate boundaries to be relative to the container
           if (elementsContainerAbsolute) {
               elementsContainerBoundary = elementsContainer.getBoundingClientRect();
               ['top', 'left'].forEach(function (key) {
                   relativeBoundary[key] = boundary[key] - elementsContainerBoundary[key];
               });

               relativeBoundary.width = boundary.width;
               relativeBoundary.height = boundary.height;
               boundary = relativeBoundary;

               containerWidth = elementsContainerBoundary.width;

               // Adjust top position according to container scroll position
               positions.top = elementsContainer.scrollTop;
           } else {
               // Adjust top position according to window scroll position
               positions.top = window.pageYOffset;
           }

           middleBoundary = boundary.left + boundary.width / 2;
           positions.top += boundary.top - toolbarHeight;

           if (boundary.top < buttonHeight) {
               toolbarElement.classList.add('medium-toolbar-arrow-over');
               toolbarElement.classList.remove('medium-toolbar-arrow-under');
               positions.top += buttonHeight + boundary.height - this.diffTop;
           } else {
               toolbarElement.classList.add('medium-toolbar-arrow-under');
               toolbarElement.classList.remove('medium-toolbar-arrow-over');
               positions.top += this.diffTop;
           }

           if (middleBoundary < halfOffsetWidth) {
               positions.left = defaultLeft + halfOffsetWidth;
               positions.right = 'initial';
           } else if ((containerWidth - middleBoundary) < halfOffsetWidth) {
               positions.left = 'auto';
               positions.right = 0;
           } else {
               positions.left = defaultLeft + middleBoundary;
               positions.right = 'initial';
           }

           ['top', 'left', 'right'].forEach(function (key) {
               toolbarElement.style[key] = positions[key] + (isNaN(positions[key]) ? '' : 'px');
           });
       },
       selectText: function (element) {
           var doc = document
               , text = doc.getElementById(element)
               , range, selection
           ;
           if (doc.body.createTextRange) {
               range = document.body.createTextRange();
               range.moveToElementText(text);
               range.select();
           } else if (window.getSelection) {
               selection = window.getSelection();
               range = document.createRange();
               range.selectNodeContents(text);
               selection.removeAllRanges();
               selection.addRange(range);
           }
       },
       getSelectionParentElement: function () {
           var parentEl = null, sel;
           if (window.getSelection) {
               sel = window.getSelection();
               if (sel.rangeCount) {
                   parentEl = sel.getRangeAt(0).commonAncestorContainer;
                   if (parentEl.nodeType != 1) {
                       parentEl = parentEl.parentNode;
                   }
               }
           } else if ( (sel = document.selection) && sel.type != "Control") {
               parentEl = sel.createRange().parentElement();
           }
           return parentEl;
       }
  }
}
</script>

<style scoped>
#editor {
  width: 80%;
  padding: 30px;
  border: 1px solid;
  white-space: pre-line;
  font-family: sans-serif;
  line-height: 1.5;
}
.data-source {
  background: #d6eff7;
}
.data-source:hover {
  background: #c5f1ff;
}
.data-fact {
  background: #ffffbd;
}
.data-fact:hover {
  background: #ffff99;
}
.data-source, .data-fact {
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  cursor: pointer;
  padding: 6px 0 0;
}
/* Toolbar styles */
.medium-editor-toolbar li button {
    box-sizing: border-box;
    cursor: pointer;
    display: block;
    font-size: 14px;
    line-height: 1.33;
    margin: 0;
    padding: 15px;
    text-decoration: none;
}
.medium-editor-toolbar li button {
    background-color: transparent;
    border: none;
    border-right: 1px solid #357ebd;
    box-sizing: border-box;
    color: #fff;
    height: 60px;
    min-width: 60px;
    transition: background-color .2s ease-in, color .2s ease-in;
}
.medium-editor-toolbar li {
  float: left;
  list-style: none;
  margin: 0;
  padding: 0;
}
.medium-editor-toolbar ul {
  margin: 0;
  padding: 0;
}

.medium-toolbar-arrow-under:after, .medium-toolbar-arrow-over:before {
  border-style: solid;
  content: '';
  display: block;
  height: 0;
  left: 50%;
  margin-left: -8px;
  position: absolute;
  width: 0;
}
.medium-editor-toolbar-active.medium-editor-stalker-toolbar {
  animation: medium-editor-pop-upwards 160ms forwards linear;
}

.medium-toolbar-arrow-under:after {
  border-width: 8px 8px 0 8px;
  border-color: #428bca transparent transparent transparent;
  top: 60px;
}

.medium-toolbar-arrow-over:before {
  border-width: 0 8px 8px 8px;
  top: -8px;
}

.medium-editor-toolbar {
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
  font-size: 16px;
  left: 0;
  position: absolute;
  top: 0;
  visibility: hidden;
  z-index: 2000;
  background: #428bca;
  border: 1px solid #357ebd;
  border-radius: 4px;
}
.medium-editor-toolbar-active {
  visibility: visible;
}
.medium-editor-toolbar li button:hover {
  background-color: #3276b1;
  color: #fff;
}
.medium-editor-toolbar li.remove-button, .medium-editor-toolbar li.select-source-button, .medium-editor-toolbar li.show-facts-button {
  display: none;
}

.medium-editor-toolbar.fact-options li.fact-button, .medium-editor-toolbar.fact-options li.source-button,
.medium-editor-toolbar.source-options li.fact-button, .medium-editor-toolbar.source-options li.source-button {
  display: none;
}

.medium-editor-toolbar.fact-options li.remove-button, .medium-editor-toolbar.source-options li.remove-button{
  display: initial;
}
.medium-editor-toolbar.fact-options li.select-source-button {
  display: initial;
}
.medium-editor-toolbar.source-options li.show-facts-button {
  display: initial;
}

.source-selectable {
  border: 1px solid green;
}
.source-selected, .data-source.source-selected:hover {
  background: green;
}

.fact-selected {
  background: yellow;
}
</style>
