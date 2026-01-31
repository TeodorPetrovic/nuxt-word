# Nuxt Word

A powerful Microsoft Word-style editor built with Nuxt.js v4 and TipTap editor. Create, edit, and share Word-compatible documents in your browser.

## Features

### Rich Text Formatting
- **Text Styling**: Bold, Italic, Underline, Strikethrough
- **Headings**: 6 levels of headings with customizable styles
- **Text Alignment**: Left, Center, Right, Justify
- **Lists**: Bulleted and Numbered lists

### Advanced Features
- **Tables**: Insert and edit tables with resizable columns
- **Images**: Insert images from URLs
- **Page Breaks**: Add page breaks for multi-page documents
- **Page Margins**: Proper A4 page layout with margins

### Document Management
- **Find & Replace**: Search for text and replace it throughout the document
- **Comments**: Add comments to selected text for collaboration
- **Revision History**: Undo/Redo support for tracking changes
- **Word Count & Character Count**: Real-time statistics in the status bar

### Import/Export
- **DOCX Import**: Open and edit existing Word documents
- **DOCX Export**: Save your documents in Word format
- **PDF Export**: Export documents to PDF format

## Tech Stack

- **Nuxt.js v4**: Modern Vue.js framework
- **TipTap**: Powerful rich text editor based on ProseMirror
- **docx**: For DOCX file generation
- **mammoth**: For DOCX file import
- **jsPDF & html2canvas**: For PDF export

## Setup

Make sure to install dependencies:

```bash
npm install
```

## Development Server

Start the development server on `http://localhost:3000`:

```bash
npm run dev
```

## Production

Build the application for production:

```bash
npm run build
```

Locally preview production build:

```bash
npm run preview
```

Check out the [deployment documentation](https://nuxt.com/docs/getting-started/deployment) for more information.

## Usage

### Creating a New Document
1. Click the "📄 New" button in the toolbar
2. Start typing in the editor

### Opening an Existing DOCX File
1. Click the "📂 Open" button
2. Select a .docx file from your computer
3. The document will be loaded into the editor

### Formatting Text
- Use the toolbar buttons to apply formatting
- Select text and click formatting buttons (Bold, Italic, etc.)
- Choose heading levels from the dropdown
- Align text using alignment buttons

### Adding Tables
1. Click the "📊 Table" button to insert a 3x3 table
2. Click inside table cells to edit content
3. Use "❌ Table" to delete the entire table

### Inserting Images
1. Click the "🖼️ Image" button
2. Enter the image URL
3. The image will be inserted at the cursor position

### Find and Replace
1. Click the "🔍 Find" button
2. Enter the text to find
3. Use "Find Next" to locate occurrences
4. Enter replacement text and use "Replace" or "Replace All"

### Adding Comments
1. Select text you want to comment on
2. Click the "💬 Comment" button
3. Enter your comment
4. View comments in the sidebar by clicking "👁️ Show"

### Saving Your Work
- **Save as DOCX**: Click "💾 Save DOCX" to download in Word format
- **Export as PDF**: Click "📕 Export PDF" to create a PDF version

## Supported Features (~50% of Word)

✅ **Text Formatting**
- Bold, Italic, Underline, Strikethrough
- Font styles and colors
- Text alignment

✅ **Headings and Styles**
- 6 heading levels
- Normal paragraph style

✅ **Lists**
- Bulleted lists
- Numbered lists

✅ **Tables**
- Insert tables
- Edit table content
- Delete tables

✅ **Images**
- Insert images from URLs

✅ **Page Layout**
- Page breaks
- Margins (96px/72px)
- A4 page format

✅ **Document Features**
- Find and replace
- Comments system
- Basic revision history (Undo/Redo)

✅ **Import/Export**
- Import DOCX files
- Export to DOCX
- Export to PDF

## Browser Support

- Chrome/Edge (latest)
- Firefox (latest)
- Safari (latest)

## License

MIT

