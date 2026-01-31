<script setup lang="ts">
const isLoaded = ref(false)
const editor = ref<any>(null)
let EditorContent: any = null

// UI state
const showFindReplace = ref(false)
const showComments = ref(false)
const editorStatus = ref('Ready')
const fileInput = ref(null)

// Find & Replace
const findText = ref('')
const replaceText = ref('')

// Comments
const comments = ref<any[]>([])

// Statistics
const wordCount = computed(() => {
  if (!editor.value) return 0
  const text = editor.value.getText()
  return text.split(/\s+/).filter((word: string) => word.length > 0).length
})

const characterCount = computed(() => {
  if (!editor.value) return 0
  return editor.value.getText().length
})

// Only run on client
onNuxtReady(async () => {
  try {
    const { useEditor, EditorContent: EC } = await import('@tiptap/vue-3')
    EditorContent = EC
    const StarterKit = (await import('@tiptap/starter-kit')).default

    editor.value = useEditor({
      extensions: [
        StarterKit,
      ],
      content: '<p><strong>Welcome to Nuxt Word!</strong> This is a Microsoft Word-style editor built with Nuxt.js v4 and TipTap. Try these features:</p><ul><li>Type and format text</li><li>Use the toolbar buttons for formatting</li><li>Insert tables, images, and lists</li><li>Save as DOCX or PDF</li></ul><p>Start creating your document...</p>',
      editorProps: {
        attributes: {
          class: 'prose prose-sm sm:prose lg:prose-lg xl:prose-2xl focus:outline-none',
        },
      },
    })
    isLoaded.value = true
  } catch (error) {
    console.error('Error loading editor:', error)
  }
})

// Toolbar functions
function setHeading(event: any) {
  if (!editor.value) return
  const level = parseInt(event.target.value)
  if (level) {
    editor.value.chain().focus().toggleHeading({ level }).run()
  } else {
    editor.value.chain().focus().setParagraph().run()
  }
  event.target.value = ''
}

function insertTable() {
  if (!editor.value) return
  editor.value.chain().focus().insertTable({ rows: 3, cols: 3, withHeaderRow: true }).run()
}

function addImage() {
  if (!editor.value) return
  const url = prompt('Enter image URL:')
  if (url) {
    editor.value.chain().focus().setImage({ src: url }).run()
  }
}

function insertPageBreak() {
  if (!editor.value) return
  editor.value.chain().focus().setHardBreak().run()
  editor.value.chain().focus().insertContent('<hr class="page-break" />').run()
}

function toggleFindReplace() {
  showFindReplace.value = !showFindReplace.value
}

function findNext() {
  if (!editor || !findText.value) return
  const content = editor.value.getText() || ''
  const index = content.indexOf(findText.value)
  if (index !== -1) {
    editorStatus.value = `Found at position ${index}`
  } else {
    editorStatus.value = 'Not found'
  }
}

function findPrevious() {
  findNext()
}

function replaceOne() {
  if (!editor || !findText.value) return
  const html = editor.value.getHTML() || ''
  const newHtml = html.replace(findText.value, replaceText.value)
  editor.value.commands.setContent(newHtml)
  editorStatus.value = 'Replaced 1 occurrence'
}

function replaceAll() {
  if (!editor || !findText.value) return
  const html = editor.value.getHTML() || ''
  const regex = new RegExp(findText.value, 'g')
  const newHtml = html.replace(regex, replaceText.value)
  editor.value.commands.setContent(newHtml)
  const count = (html.match(regex) || []).length
  editorStatus.value = `Replaced ${count} occurrences`
}

function addComment() {
  if (!editor.value) return
  const selection = editor.value.state.selection
  if (!selection || selection.empty) {
    alert('Please select some text to comment on')
    return
  }

  const commentText = prompt('Enter your comment:')
  if (commentText) {
    comments.value.push({
      text: commentText,
      author: 'User',
      timestamp: Date.now(),
      selection: editor.value.getText().substring(selection.from, selection.to),
    })
    showComments.value = true
    editorStatus.value = 'Comment added'
  }
}

function deleteComment(index: number) {
  comments.value.splice(index, 1)
}

function toggleComments() {
  showComments.value = !showComments.value
}

function formatTime(timestamp: number) {
  return new Date(timestamp).toLocaleString()
}

function newDocument() {
  if (!editor.value) return
  if (confirm('Create a new document? Unsaved changes will be lost.')) {
    editor.value.commands.setContent('<p></p>')
    comments.value = []
    editorStatus.value = 'New document created'
  }
}

function openDocument() {
  fileInput.value?.click()
}

async function handleFileUpload(event: any) {
  if (!editor.value) return
  const file = event.target.files?.[0]
  if (!file) return

  try {
    editorStatus.value = 'Loading document...'
    const arrayBuffer = await file.arrayBuffer()
    
    const mammoth = await import('mammoth')
    const result = await mammoth.convertToHtml({ arrayBuffer })
    
    editor.value.commands.setContent(result.value)
    editorStatus.value = 'Document loaded successfully'
  } catch (error) {
    console.error('Error loading document:', error)
    editorStatus.value = 'Error loading document'
    alert('Failed to load document. Please try again.')
  }
}

async function saveAsDocx() {
  if (!editor.value) return
  try {
    editorStatus.value = 'Saving document...'
    
    const { Document, Packer, Paragraph, TextRun } = await import('docx')
    const { saveAs } = await import('file-saver')
    
    const doc = new Document({
      sections: [{
        properties: {},
        children: [
          new Paragraph({
            children: [
              new TextRun({
                text: editor.value.getText() || '',
              }),
            ],
          }),
        ],
      }],
    })
    
    const blob = await Packer.toBlob(doc)
    saveAs(blob, 'document.docx')
    editorStatus.value = 'Document saved as DOCX'
  } catch (error) {
    console.error('Error saving document:', error)
    editorStatus.value = 'Error saving document'
    alert('Failed to save document. Please try again.')
  }
}

async function exportToPdf() {
  if (!editor.value) return
  try {
    editorStatus.value = 'Exporting to PDF...'
    
    const jsPDF = (await import('jspdf')).default
    const html2canvas = (await import('html2canvas')).default
    
    const content = document.querySelector('.editor-content')
    if (!content) return
    
    const canvas = await html2canvas(content as HTMLElement, {
      scale: 2,
      useCORS: true,
      logging: false,
    })
    
    const imgData = canvas.toDataURL('image/png')
    const pdf = new jsPDF({
      orientation: 'portrait',
      unit: 'mm',
      format: 'a4',
    })
    
    const imgWidth = 210
    const imgHeight = (canvas.height * imgWidth) / canvas.width
    
    pdf.addImage(imgData, 'PNG', 0, 0, imgWidth, imgHeight)
    pdf.save('document.pdf')
    
    editorStatus.value = 'Document exported as PDF'
  } catch (error) {
    console.error('Error exporting to PDF:', error)
    editorStatus.value = 'Error exporting to PDF'
    alert('Failed to export to PDF. Please try again.')
  }
}
</script>

<template>
  <ClientOnly>
    <div v-if="isLoaded" class="word-editor-container">
      <!-- Toolbar -->
      <div class="toolbar">
        <!-- File Operations -->
        <div class="toolbar-group">
          <button @click="newDocument" class="toolbar-btn" title="New Document">
            <span>📄</span> New
          </button>
          <button @click="openDocument" class="toolbar-btn" title="Open DOCX">
            <span>📂</span> Open
          </button>
          <button @click="saveAsDocx" class="toolbar-btn" title="Save as DOCX">
            <span>💾</span> Save DOCX
          </button>
          <button @click="exportToPdf" class="toolbar-btn" title="Export to PDF">
            <span>��</span> Export PDF
          </button>
          <input
            ref="fileInput"
            type="file"
            accept=".docx"
            @change="handleFileUpload"
            style="display: none"
          />
        </div>

        <div class="toolbar-divider"></div>

        <!-- Text Formatting -->
        <div class="toolbar-group">
          <button
            @click="editor?.chain().focus().toggleBold().run()"
            :class="{ 'is-active': editor?.isActive('bold') }"
            class="toolbar-btn"
            title="Bold"
          >
            <strong>B</strong>
          </button>
          <button
            @click="editor?.chain().focus().toggleItalic().run()"
            :class="{ 'is-active': editor?.isActive('italic') }"
            class="toolbar-btn"
            title="Italic"
          >
            <em>I</em>
          </button>
          <button
            @click="editor?.chain().focus().toggleStrike().run()"
            :class="{ 'is-active': editor?.isActive('strike') }"
            class="toolbar-btn"
            title="Strikethrough"
          >
            <s>S</s>
          </button>
        </div>

        <div class="toolbar-divider"></div>

        <!-- Headings -->
        <div class="toolbar-group">
          <select @change="setHeading" class="toolbar-select">
            <option value="">Normal</option>
            <option value="1">Heading 1</option>
            <option value="2">Heading 2</option>
            <option value="3">Heading 3</option>
          </select>
        </div>

        <div class="toolbar-divider"></div>

        <!-- Lists -->
        <div class="toolbar-group">
          <button
            @click="editor?.chain().focus().toggleBulletList().run()"
            :class="{ 'is-active': editor?.isActive('bulletList') }"
            class="toolbar-btn"
            title="Bullet List"
          >
            • List
          </button>
          <button
            @click="editor?.chain().focus().toggleOrderedList().run()"
            :class="{ 'is-active': editor?.isActive('orderedList') }"
            class="toolbar-btn"
            title="Numbered List"
          >
            1. List
          </button>
        </div>

        <div class="toolbar-divider"></div>

        <!-- Table -->
        <div class="toolbar-group">
          <button @click="insertTable" class="toolbar-btn" title="Insert Table">
            📊 Table
          </button>
        </div>

        <div class="toolbar-divider"></div>

        <!-- Image -->
        <div class="toolbar-group">
          <button @click="addImage" class="toolbar-btn" title="Insert Image">
            🖼️ Image
          </button>
        </div>

        <div class="toolbar-divider"></div>

        <!-- Find & Replace -->
        <div class="toolbar-group">
          <button @click="toggleFindReplace" class="toolbar-btn" title="Find & Replace">
            🔍 Find
          </button>
        </div>

        <div class="toolbar-divider"></div>

        <!-- Comments -->
        <div class="toolbar-group">
          <button @click="addComment" class="toolbar-btn" title="Add Comment">
            💬 Comment
          </button>
          <button @click="toggleComments" class="toolbar-btn" title="Toggle Comments">
            {{ showComments ? '👁️ Hide' : '👁️ Show' }}
          </button>
        </div>

        <div class="toolbar-divider"></div>

        <!-- History -->
        <div class="toolbar-group">
          <button
            @click="editor?.chain().focus().undo().run()"
            class="toolbar-btn"
            title="Undo"
          >
            ↶
          </button>
          <button
            @click="editor?.chain().focus().redo().run()"
            class="toolbar-btn"
            title="Redo"
          >
            ↷
          </button>
        </div>
      </div>

      <!-- Find & Replace Panel -->
      <div v-if="showFindReplace" class="find-replace-panel">
        <div class="find-replace-row">
          <input
            v-model="findText"
            type="text"
            placeholder="Find..."
            class="find-input"
            @keyup.enter="findNext"
          />
          <button @click="findNext" class="toolbar-btn">Find Next</button>
          <button @click="findPrevious" class="toolbar-btn">Find Previous</button>
        </div>
        <div class="find-replace-row">
          <input
            v-model="replaceText"
            type="text"
            placeholder="Replace with..."
            class="find-input"
          />
          <button @click="replaceOne" class="toolbar-btn">Replace</button>
          <button @click="replaceAll" class="toolbar-btn">Replace All</button>
          <button @click="toggleFindReplace" class="toolbar-btn">Close</button>
        </div>
      </div>

      <!-- Main Editor Area -->
      <div class="editor-wrapper">
        <!-- Editor Content -->
        <div class="editor-content-wrapper">
          <div class="page-container">
            <component :is="EditorContent" :editor="editor" class="editor-content" />
          </div>
        </div>

        <!-- Comments Sidebar -->
        <div v-if="showComments" class="comments-sidebar">
          <h3>Comments</h3>
          <div v-for="(comment, index) in comments" :key="index" class="comment">
            <div class="comment-header">
              <strong>{{ comment.author }}</strong>
              <span class="comment-time">{{ formatTime(comment.timestamp) }}</span>
            </div>
            <p class="comment-text">{{ comment.text }}</p>
            <button @click="deleteComment(index)" class="comment-delete">Delete</button>
          </div>
          <div v-if="comments.length === 0" class="no-comments">
            No comments yet. Select text and click "Add Comment" to add one.
          </div>
        </div>
      </div>

      <!-- Status Bar -->
      <div class="status-bar">
        <span>Words: {{ wordCount }}</span>
        <span>Characters: {{ characterCount }}</span>
        <span>{{ editorStatus }}</span>
      </div>
    </div>
    <div v-else style="display: flex; justify-content: center; align-items: center; height: 100vh; font-size: 24px; background: #f0f0f0;">
      <div style="text-align: center;">
        <div style="font-size: 48px; margin-bottom: 20px;">📝</div>
        <div>Loading Nuxt Word Editor...</div>
      </div>
    </div>
  </ClientOnly>
</template>

<style scoped>
.word-editor-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background: #f0f0f0;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.toolbar {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  padding: 12px;
  background: white;
  border-bottom: 1px solid #ddd;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  align-items: center;
}

.toolbar-group {
  display: flex;
  gap: 4px;
  align-items: center;
}

.toolbar-btn {
  padding: 6px 12px;
  border: 1px solid #ccc;
  background: white;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
  white-space: nowrap;
}

.toolbar-btn:hover:not(:disabled) {
  background: #f0f0f0;
  border-color: #999;
}

.toolbar-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.toolbar-btn.is-active {
  background: #e3f2fd;
  border-color: #2196f3;
  color: #2196f3;
}

.toolbar-select {
  padding: 6px 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background: white;
  font-size: 14px;
  cursor: pointer;
}

.toolbar-divider {
  width: 1px;
  height: 24px;
  background: #ddd;
}

.find-replace-panel {
  padding: 12px;
  background: #fff9e6;
  border-bottom: 1px solid #ddd;
}

.find-replace-row {
  display: flex;
  gap: 8px;
  margin-bottom: 8px;
  align-items: center;
}

.find-replace-row:last-child {
  margin-bottom: 0;
}

.find-input {
  flex: 1;
  padding: 6px 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
}

.editor-wrapper {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.editor-content-wrapper {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
  background: #e0e0e0;
}

.page-container {
  max-width: 816px;
  margin: 0 auto;
  background: white;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  min-height: 1056px;
  padding: 96px 72px;
}

.editor-content {
  min-height: 800px;
}

.editor-content :deep(.ProseMirror) {
  outline: none;
  min-height: 800px;
}

.editor-content :deep(.ProseMirror p.is-editor-empty:first-child::before) {
  content: attr(data-placeholder);
  float: left;
  color: #adb5bd;
  pointer-events: none;
  height: 0;
}

.editor-content :deep(table) {
  border-collapse: collapse;
  margin: 16px 0;
  width: 100%;
}

.editor-content :deep(th),
.editor-content :deep(td) {
  border: 1px solid #ddd;
  padding: 8px;
  min-width: 100px;
}

.editor-content :deep(th) {
  background: #f5f5f5;
  font-weight: bold;
}

.editor-content :deep(img) {
  max-width: 100%;
  height: auto;
  display: block;
  margin: 16px 0;
}

.editor-content :deep(.page-break) {
  page-break-after: always;
  border: none;
  border-top: 2px dashed #ccc;
  margin: 40px 0;
}

.comments-sidebar {
  width: 300px;
  background: white;
  border-left: 1px solid #ddd;
  padding: 16px;
  overflow-y: auto;
}

.comments-sidebar h3 {
  margin: 0 0 16px 0;
  font-size: 18px;
  color: #333;
}

.comment {
  padding: 12px;
  background: #f9f9f9;
  border-left: 3px solid #2196f3;
  margin-bottom: 12px;
  border-radius: 4px;
}

.comment-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
  font-size: 12px;
}

.comment-time {
  color: #666;
}

.comment-text {
  margin: 8px 0;
  font-size: 14px;
  color: #333;
}

.comment-delete {
  padding: 4px 8px;
  border: 1px solid #ccc;
  background: white;
  border-radius: 3px;
  cursor: pointer;
  font-size: 12px;
}

.comment-delete:hover {
  background: #f44336;
  color: white;
  border-color: #f44336;
}

.no-comments {
  color: #999;
  font-style: italic;
  font-size: 14px;
  text-align: center;
  padding: 20px;
}

.status-bar {
  display: flex;
  gap: 24px;
  padding: 8px 16px;
  background: white;
  border-top: 1px solid #ddd;
  font-size: 12px;
  color: #666;
}
</style>
