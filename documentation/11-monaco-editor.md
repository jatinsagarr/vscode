# 📝 Monaco Editor - The Heart of VS Code

## 🎯 What is Monaco Editor?

Monaco Editor is the **text editing engine** that powers VS Code. Think of it as the **engine in a car** - it's what actually handles all the text editing, syntax highlighting, IntelliSense, and everything you see when you're writing code.

**Fun fact**: Monaco Editor can run standalone! You can embed it in websites, which is exactly what GitHub, Azure DevOps, and many other services do.

## 📊 Monaco Editor Quick Facts

- **Location**: `src/vs/editor/`
- **Standalone**: Can run without the rest of VS Code
- **Used Everywhere**: GitHub, Azure DevOps, CodePen, and thousands of websites
- **Language Agnostic**: Supports any programming language
- **Highly Optimized**: Handles files with millions of lines

## 🏗️ Monaco Editor Architecture (Simple View)

```
Monaco Editor
├── 📄 Text Model        # The document (your code)
├── 🎨 View Layer        # What you see on screen
├── 🖱️ Controller       # Handles your input (typing, clicking)
├── 🧩 Contributions     # Features (find/replace, IntelliSense, etc.)
└── 🔌 Language Support # Syntax highlighting, autocomplete
```

Think of it like a **word processor for code**:
- **Text Model** = The document you're editing
- **View Layer** = The visual representation on screen
- **Controller** = Handles your keyboard and mouse
- **Contributions** = Features like spell-check, but for code
- **Language Support** = Knows about different programming languages

## 📄 Text Model - The Document Brain

### **What is a Text Model?**
The Text Model is like a **smart document** that knows about your code. It's not just storing text - it understands lines, positions, changes, and can efficiently handle huge files.

### **Simple Text Model Example**
```typescript
// src/vs/editor/common/model/textModel.ts

class TextModel implements ITextModel {
  private lines: string[] = [];
  private versionId: number = 1;

  // Get the entire document as a string
  getValue(): string {
    return this.lines.join('\n');
  }

  // Replace the entire document
  setValue(newValue: string): void {
    this.lines = newValue.split('\n');
    this.versionId++;

    // Tell everyone the content changed
    this._onDidChangeContent.fire({
      changes: [{ /* change details */ }],
      eol: '\n',
      versionId: this.versionId
    });
  }

  // Get text from a specific range
  getValueInRange(range: IRange): string {
    const startLine = range.startLineNumber - 1;
    const endLine = range.endLineNumber - 1;

    if (startLine === endLine) {
      // Single line
      const line = this.lines[startLine];
      return line.substring(range.startColumn - 1, range.endColumn - 1);
    } else {
      // Multiple lines
      const result: string[] = [];

      // First line (partial)
      result.push(this.lines[startLine].substring(range.startColumn - 1));

      // Middle lines (complete)
      for (let i = startLine + 1; i < endLine; i++) {
        result.push(this.lines[i]);
      }

      // Last line (partial)
      result.push(this.lines[endLine].substring(0, range.endColumn - 1));

      return result.join('\n');
    }
  }

  // Apply edits to the document
  applyEdits(operations: IIdentifiedSingleEditOperation[]): void {
    // Sort operations by position (end to start to avoid position shifts)
    const sortedOps = operations.sort((a, b) =>
      Range.compareRangesUsingStarts(b.range, a.range)
    );

    for (const op of sortedOps) {
      if (op.text === null) {
        // Delete operation
        this.deleteRange(op.range);
      } else {
        // Insert/Replace operation
        this.replaceRange(op.range, op.text);
      }
    }

    this.versionId++;
    this._onDidChangeContent.fire(/* change event */);
  }
}
```

### **Text Model Superpowers**
- **Efficient Storage**: Uses a "piece table" data structure for fast edits
- **Undo/Redo**: Tracks all changes for undo/redo functionality
- **Change Events**: Notifies when content changes
- **Position Tracking**: Knows exactly where everything is
- **Language Awareness**: Understands the programming language

## 🎨 View Layer - What You See

### **What is the View Layer?**
The View Layer takes the Text Model and turns it into what you see on screen. It handles:
- **Rendering text** with proper fonts and colors
- **Line numbers** on the side
- **Syntax highlighting** (making keywords blue, strings green, etc.)
- **Scrolling** through large files
- **Cursor positioning** and selection

### **Simple View Example**
```typescript
// The View Layer is like a painter that draws the editor
class EditorView {
  private textModel: ITextModel;
  private container: HTMLElement;

  constructor(container: HTMLElement, model: ITextModel) {
    this.container = container;
    this.textModel = model;

    // Listen for model changes
    this.textModel.onDidChangeContent(() => {
      this.render(); // Repaint when content changes
    });

    this.render();
  }

  private render(): void {
    // Clear the container
    this.container.innerHTML = '';

    // Get all lines from the model
    const lines = this.textModel.getLinesContent();

    // Create HTML for each line
    lines.forEach((lineText, lineNumber) => {
      const lineElement = document.createElement('div');
      lineElement.className = 'editor-line';

      // Add line number
      const lineNumberElement = document.createElement('span');
      lineNumberElement.className = 'line-number';
      lineNumberElement.textContent = (lineNumber + 1).toString();

      // Add line content with syntax highlighting
      const contentElement = document.createElement('span');
      contentElement.className = 'line-content';
      contentElement.innerHTML = this.syntaxHighlight(lineText);

      lineElement.appendChild(lineNumberElement);
      lineElement.appendChild(contentElement);
      this.container.appendChild(lineElement);
    });
  }

  private syntaxHighlight(text: string): string {
    // Simple syntax highlighting (real version is much more complex)
    return text
      .replace(/\b(function|class|const|let|var)\b/g, '<span class="keyword">$1</span>')
      .replace(/"([^"]*)"/g, '<span class="string">"$1"</span>')
      .replace(/\/\/.*$/gm, '<span class="comment">$&</span>');
  }
}
```

### **View Layer Magic**
- **Virtual Scrolling**: Only renders visible lines (handles million-line files)
- **Syntax Highlighting**: Colors code based on language rules
- **Decorations**: Underlines, highlights, error squiggles
- **Minimap**: The small code overview on the right
- **Line Numbers**: Gutter with line numbers

## 🖱️ Controller - Handling Your Input

### **What is the Controller?**
The Controller is like the **translator** between your actions (typing, clicking, scrolling) and changes to the Text Model.

### **Simple Controller Example**
```typescript
class EditorController {
  private textModel: ITextModel;
  private view: EditorView;
  private cursor: Position = new Position(1, 1);

  constructor(textModel: ITextModel, view: EditorView) {
    this.textModel = textModel;
    this.view = view;

    this.setupEventListeners();
  }

  private setupEventListeners(): void {
    // Handle typing
    document.addEventListener('keypress', (e) => {
      if (e.key.length === 1) { // Regular character
        this.type(e.key);
      }
    });

    // Handle special keys
    document.addEventListener('keydown', (e) => {
      switch (e.key) {
        case 'Enter':
          this.insertNewLine();
          break;
        case 'Backspace':
          this.backspace();
          break;
        case 'ArrowLeft':
          this.moveCursor(-1, 0);
          break;
        case 'ArrowRight':
          this.moveCursor(1, 0);
          break;
        case 'ArrowUp':
          this.moveCursor(0, -1);
          break;
        case 'ArrowDown':
          this.moveCursor(0, 1);
          break;
      }
    });

    // Handle mouse clicks
    this.view.onClick((position) => {
      this.cursor = position;
      this.view.setCursor(position);
    });
  }

  private type(character: string): void {
    // Insert character at cursor position
    const edit = {
      range: Range.fromPositions(this.cursor),
      text: character
    };

    this.textModel.applyEdits([edit]);

    // Move cursor forward
    this.cursor = new Position(this.cursor.lineNumber, this.cursor.column + 1);
    this.view.setCursor(this.cursor);
  }

  private insertNewLine(): void {
    const edit = {
      range: Range.fromPositions(this.cursor),
      text: '\n'
    };

    this.textModel.applyEdits([edit]);

    // Move cursor to next line
    this.cursor = new Position(this.cursor.lineNumber + 1, 1);
    this.view.setCursor(this.cursor);
  }

  private backspace(): void {
    if (this.cursor.column > 1) {
      // Delete character before cursor
      const deleteRange = new Range(
        this.cursor.lineNumber,
        this.cursor.column - 1,
        this.cursor.lineNumber,
        this.cursor.column
      );

      const edit = {
        range: deleteRange,
        text: ''
      };

      this.textModel.applyEdits([edit]);

      // Move cursor back
      this.cursor = new Position(this.cursor.lineNumber, this.cursor.column - 1);
      this.view.setCursor(this.cursor);
    }
  }
}
```

### **Controller Responsibilities**
- **Keyboard Input**: Converts key presses to text changes
- **Mouse Input**: Handles clicks, selections, drag and drop
- **Cursor Management**: Tracks where the cursor is
- **Selection Handling**: Manages text selection
- **Command Execution**: Runs editor commands (copy, paste, etc.)

## 🧩 Contributions - Editor Features

### **What are Contributions?**
Contributions are like **plugins** that add features to the editor. Each feature (find/replace, IntelliSense, bracket matching) is a separate contribution.

### **Example: Find and Replace Contribution**
```typescript
// src/vs/editor/contrib/find/findController.ts

class FindController implements IEditorContribution {
  public static readonly ID = 'editor.contrib.findController';

  private editor: ICodeEditor;
  private findWidget: FindWidget;

  constructor(editor: ICodeEditor) {
    this.editor = editor;
    this.findWidget = new FindWidget(editor);

    // Register keyboard shortcuts
    this.editor.addAction({
      id: 'actions.find',
      label: 'Find',
      alias: 'Find',
      precondition: undefined,
      kbOpts: {
        kbExpr: null,
        primary: KeyMod.CtrlCmd | KeyCode.KeyF
      },
      run: () => this.start()
    });

    this.editor.addAction({
      id: 'actions.findNext',
      label: 'Find Next',
      alias: 'Find Next',
      precondition: undefined,
      kbOpts: {
        kbExpr: null,
        primary: KeyCode.F3
      },
      run: () => this.findNext()
    });
  }

  start(): void {
    // Show the find widget
    this.findWidget.show();

    // If text is selected, use it as search term
    const selection = this.editor.getSelection();
    if (selection && !selection.isEmpty()) {
      const selectedText = this.editor.getModel()?.getValueInRange(selection);
      if (selectedText) {
        this.findWidget.setSearchString(selectedText);
      }
    }
  }

  findNext(): void {
    const searchString = this.findWidget.getSearchString();
    if (!searchString) return;

    const model = this.editor.getModel();
    if (!model) return;

    // Find next occurrence
    const currentPosition = this.editor.getPosition();
    const match = model.findNextMatch(
      searchString,
      currentPosition,
      false, // not regex
      false, // not case sensitive
      null,  // no word separators
      false  // not capture matches
    );

    if (match) {
      // Select the found text
      this.editor.setSelection(match.range);
      this.editor.revealRangeInCenter(match.range);
    }
  }
}

// Register the contribution
EditorExtensionsRegistry.registerEditorContribution(
  FindController.ID,
  FindController,
  EditorContributionInstantiation.Eager
);
```

### **Popular Editor Contributions**
- **Find/Replace** (`find/`) - Search and replace text
- **IntelliSense** (`suggest/`) - Code completion
- **Bracket Matching** (`bracketMatching/`) - Highlights matching brackets
- **Code Folding** (`folding/`) - Collapse/expand code blocks
- **Hover** (`hover/`) - Shows information when you hover
- **Go to Definition** (`gotoSymbol/`) - Jump to symbol definitions
- **Format Document** (`format/`) - Code formatting
- **Multi-cursor** (`multicursor/`) - Multiple cursors at once

## 🔌 Language Support - Making Code Smart

### **What is Language Support?**
Language Support makes the editor understand different programming languages. It provides:
- **Syntax highlighting** (keywords, strings, comments)
- **IntelliSense** (autocomplete, parameter hints)
- **Error detection** (red squiggles under errors)
- **Go to definition** (jump to where functions are defined)
- **Refactoring** (rename symbols, extract methods)

### **Simple Language Support Example**
```typescript
// Register language support for a custom language
class MyLanguageSupport {

  // Define the language
  static register(): void {
    // Register the language
    languages.register({ id: 'mylang' });

    // Set file extensions
    languages.setLanguageConfiguration('mylang', {
      comments: {
        lineComment: '//',
        blockComment: ['/*', '*/']
      },
      brackets: [
        ['{', '}'],
        ['[', ']'],
        ['(', ')']
      ],
      autoClosingPairs: [
        { open: '{', close: '}' },
        { open: '[', close: ']' },
        { open: '(', close: ')' },
        { open: '"', close: '"' }
      ]
    });

    // Provide syntax highlighting
    languages.setMonarchTokensProvider('mylang', {
      tokenizer: {
        root: [
          [/\b(function|class|if|else|for|while)\b/, 'keyword'],
          [/"([^"\\]|\\.)*$/, 'string.invalid'],
          [/"/, 'string', '@string'],
          [/\/\/.*$/, 'comment'],
          [/\d+/, 'number']
        ],
        string: [
          [/[^\\"]+/, 'string'],
          [/"/, 'string', '@pop']
        ]
      }
    });

    // Provide IntelliSense
    languages.registerCompletionItemProvider('mylang', {
      provideCompletionItems: (model, position) => {
        const suggestions = [
          {
            label: 'function',
            kind: languages.CompletionItemKind.Keyword,
            insertText: 'function ${1:name}() {\n\t$0\n}',
            insertTextRules: languages.CompletionItemInsertTextRule.InsertAsSnippet
          },
          {
            label: 'console.log',
            kind: languages.CompletionItemKind.Function,
            insertText: 'console.log(${1:message});',
            insertTextRules: languages.CompletionItemInsertTextRule.InsertAsSnippet
          }
        ];

        return { suggestions };
      }
    });

    // Provide hover information
    languages.registerHoverProvider('mylang', {
      provideHover: (model, position) => {
        const word = model.getWordAtPosition(position);
        if (word?.word === 'function') {
          return {
            range: new Range(position.lineNumber, word.startColumn, position.lineNumber, word.endColumn),
            contents: [
              { value: '**function** keyword' },
              { value: 'Declares a function in MyLang' }
            ]
          };
        }
        return null;
      }
    });
  }
}
```

## 🔄 How It All Works Together

Here's how all the pieces work together when you type in the editor:

```typescript
// Real example of what happens when you type a character
class EditorOrchestrator {

  onKeyPress(character: string): void {
    // 1. Controller receives the key press
    const position = this.cursor.getPosition();

    // 2. Controller creates an edit operation
    const edit = {
      range: Range.fromPositions(position),
      text: character
    };

    // 3. Text Model applies the edit
    this.textModel.applyEdits([edit]);

    // 4. Text Model fires change event
    this.textModel._onDidChangeContent.fire({
      changes: [edit],
      eol: '\n',
      versionId: this.textModel.getVersionId()
    });

    // 5. View Layer receives the change event
    this.view.onModelContentChanged(() => {
      // Re-render the affected lines
      this.renderLines(edit.range.startLineNumber, edit.range.endLineNumber);

      // Update syntax highlighting
      this.updateTokens(edit.range.startLineNumber);

      // Update decorations (error squiggles, etc.)
      this.updateDecorations();
    });

    // 6. Language Support analyzes the change
    this.languageService.onModelContentChanged(() => {
      // Update IntelliSense suggestions
      this.updateCompletions();

      // Check for errors
      this.validateDocument();

      // Update semantic highlighting
      this.updateSemanticTokens();
    });

    // 7. Contributions react to the change
    this.contributions.forEach(contrib => {
      if (contrib.onModelContentChanged) {
        contrib.onModelContentChanged();
      }
    });
  }
}
```

## 🎯 Why Monaco is Brilliant

### **1. Separation of Concerns**
Each part has a specific job:
- **Model** = Data storage and manipulation
- **View** = Visual representation
- **Controller** = User input handling
- **Contributions** = Feature plugins
- **Language Support** = Language-specific intelligence

### **2. Performance**
- **Virtual Scrolling**: Only renders what you can see
- **Incremental Updates**: Only re-renders what changed
- **Efficient Data Structures**: Fast text operations
- **Web Workers**: Heavy computation doesn't block UI

### **3. Extensibility**
- **Contributions**: Easy to add new features
- **Language Support**: Easy to add new languages
- **Themes**: Easy to customize appearance
- **Standalone**: Can be embedded anywhere

### **4. Cross-Platform**
- **Same Code**: Works in Electron, browsers, and web workers
- **Consistent Behavior**: Same editing experience everywhere
- **Responsive**: Adapts to different screen sizes

## 🧪 Testing Monaco Editor

Monaco Editor is extensively tested:

```typescript
describe('TextModel', () => {
  test('should handle basic text operations', () => {
    const model = createTextModel('Hello World');

    // Test getting value
    expect(model.getValue()).toBe('Hello World');

    // Test editing
    model.applyEdits([{
      range: new Range(1, 7, 1, 12), // "World"
      text: 'Monaco'
    }]);

    expect(model.getValue()).toBe('Hello Monaco');
  });

  test('should track cursor position correctly', () => {
    const model = createTextModel('Line 1\nLine 2\nLine 3');
    const controller = new EditorController(model);

    // Move cursor to line 2, column 3
    controller.setCursorPosition(new Position(2, 3));

    // Type a character
    controller.type('X');

    // Should insert at the correct position
    expect(model.getLineContent(2)).toBe('LiXne 2');
  });
});
```

## 🎓 Key Takeaways

### **What You Should Remember**
1. **Monaco = Text Editor Engine** - The core editing functionality
2. **Model-View-Controller** - Clean separation of concerns
3. **Contributions = Features** - Pluggable feature system
4. **Language Support** - Makes code smart
5. **Standalone Capable** - Can run without VS Code

### **The Big Components**
- **Text Model** - The document and its data
- **View Layer** - What you see on screen
- **Controller** - Handles your input
- **Contributions** - Editor features
- **Language Support** - Programming language intelligence

## 📚 What's Next?

Now that you understand the editor engine, let's see how it fits into VS Code:

1. **[12-text-model.md](./12-text-model.md)** - Deep dive into document representation
2. **[13-editor-contributions.md](./13-editor-contributions.md)** - All the editor features
3. **[16-workbench-core.md](./16-workbench-core.md)** - How the workbench uses Monaco

Think of your learning journey:
- ✅ **Foundation** (Base Layer) - The building blocks
- ✅ **Services** (Platform Layer) - The core functionality
- ✅ **Editor Engine** (Monaco) - The text editing heart
- ⏭️ **User Interface** (Workbench) - The complete IDE experience

Monaco Editor is the beating heart of VS Code. Every time you type, every syntax highlight, every IntelliSense suggestion - it all flows through this beautifully architected system. Now you know the magic behind the best code editor in the world! ✨
