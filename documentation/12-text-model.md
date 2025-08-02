# 📄 VS Code Text Model System - Complete Guide

## 🎯 Overview

The Text Model is the heart of VS Code's editor - it's the smart document that holds your code, tracks changes, manages undo/redo, and provides the foundation for all editor features. Think of it as the "brain" behind every file you edit.

## 🧠 What is the Text Model?

### **Simple Analogy: The Smart Notebook**
Imagine you have a magical notebook that:
- **Remembers everything**: Every change you make
- **Understands structure**: Knows about lines, words, and syntax
- **Tracks history**: Can undo/redo any change
- **Stays synchronized**: Updates all viewers when changed
- **Validates content**: Checks for errors as you type

That's exactly what VS Code's Text Model does for your code!

## 🏗️ Text Model Architecture

### **Core Components**
```
Text Model System
├── 📄 TextModel (Core Document)
├── 🔄 EditStack (Undo/Redo)
├── 📝 TextBuffer (Content Storage)
├── 🎯 ModelDecorations (Visual Markers)
├── 🔍 TokenizationSupport (Syntax Highlighting)
└── 📊 ModelServices (Language Features)
```

## 📄 Core Text Model (`src/vs/editor/common/model/textModel.ts`)

### **The Main TextModel Class**
```typescript
export class TextModel extends Disposable implements ITextModel {
    // The actual text content
    private readonly _buffer: ITextBuffer;

    // Undo/redo system
    private readonly _commandManager: EditStack;

    // Visual decorations (errors, highlights, etc.)
    private readonly _decorations: ModelDecorationOptions;

    // Language-specific features
    private _languageId: string;

    // Event system
    private readonly _onDidChangeContent = new Emitter<IModelContentChangedEvent>();
    readonly onDidChangeContent = this._onDidChangeContent.event;
}
```

### **Key Responsibilities**
1. **Content Management**: Store and modify text
2. **Change Tracking**: Track all modifications
3. **Event Broadcasting**: Notify when content changes
4. **Language Integration**: Connect to language services
5. **Decoration Management**: Handle visual markers

## 📝 Text Buffer System (`src/vs/editor/common/model/textBuffer.ts`)

### **How Text is Stored**
```typescript
export class TextBuffer implements ITextBuffer {
    // Efficient line-based storage
    private _lines: string[];

    // Line ending information
    private _lineEndings: LineEnding[];

    // Performance optimization
    private _BOM: string;

    // Get text in a range
    getValueInRange(range: Range): string {
        const startLine = range.startLineNumber - 1;
        const endLine = range.endLineNumber - 1;

        if (startLine === endLine) {
            // Single line
            return this._lines[startLine].substring(
                range.startColumn - 1,
                range.endColumn - 1
            );
        }

        // Multiple lines
        const result: string[] = [];
        result.push(this._lines[startLine].substring(range.startColumn - 1));

        for (let i = startLine + 1; i < endLine; i++) {
            result.push(this._lines[i]);
        }

        result.push(this._lines[endLine].substring(0, range.endColumn - 1));
        return result.join('\n');
    }
}
```

### **Why Line-Based Storage?**
- **Memory Efficient**: Only store changed lines
- **Fast Operations**: Quick line-based edits
- **Diff Friendly**: Easy to compare changes
- **Undo Optimized**: Efficient undo/redo operations

## 🔄 Edit Stack System (`src/vs/editor/common/model/editStack.ts`)

### **Undo/Redo Magic**
```typescript
export class EditStack {
    // Past operations (for undo)
    private past: EditStackElement[] = [];

    // Future operations (for redo)
    private future: EditStackElement[] = [];

    // Current operation being built
    private currentEditStackElement: EditStackElement | null = null;

    pushEditOperation(operations: IIdentifiedSingleEditOperation[]): void {
        // Group related operations together
        if (!this.currentEditStackElement) {
            this.currentEditStackElement = new EditStackElement();
        }

        this.currentEditStackElement.operations.push(...operations);
    }

    undo(): IIdentifiedSingleEditOperation[] | null {
        if (this.past.length === 0) {
            return null; // Nothing to undo
        }

        const element = this.past.pop()!;
        this.future.push(element);

        // Return inverse operations
        return element.getInverseOperations();
    }
}
```

### **Smart Operation Grouping**
```typescript
// These operations get grouped together:
model.pushEditOperations([], [
    { range: new Range(1, 1, 1, 1), text: 'function ' },
    { range: new Range(1, 10, 1, 10), text: 'hello() {\n}' }
], () => null);

// Single undo will remove both operations
```

## 🎯 Model Decorations (`src/vs/editor/common/model/textModelDecorations.ts`)

### **Visual Markers System**
```typescript
export interface IModelDecoration {
    // Where to show the decoration
    range: Range;

    // What it looks like
    options: IModelDecorationOptions;
}

export interface IModelDecorationOptions {
    // CSS class for styling
    className?: string;

    // Gutter icon
    glyphMarginClassName?: string;

    // Hover message
    hoverMessage?: IMarkdownString;

    // How it behaves when text changes
    stickiness?: TrackedRangeStickiness;
}
```

### **Common Decoration Types**
```typescript
// Error squiggles
const errorDecoration: IModelDecoration = {
    range: new Range(5, 10, 5, 20),
    options: {
        className: 'squiggly-error',
        hoverMessage: { value: 'Syntax error: missing semicolon' }
    }
};

// Search highlights
const searchHighlight: IModelDecoration = {
    range: new Range(10, 5, 10, 15),
    options: {
        className: 'findMatch',
        overviewRuler: {
            color: 'rgba(246, 185, 77, 0.7)',
            position: OverviewRulerLane.Center
        }
    }
};

// Breakpoints
const breakpoint: IModelDecoration = {
    range: new Range(15, 1, 15, 1),
    options: {
        glyphMarginClassName: 'debug-breakpoint',
        stickiness: TrackedRangeStickiness.NeverGrowsWhenTypingAtEdges
    }
};
```

## 🔍 Tokenization Support (`src/vs/editor/common/languages/supports/tokenizationSupport.ts`)

### **Syntax Highlighting Engine**
```typescript
export class TokenizationSupport implements ITokenizationSupport {
    private readonly _grammar: IGrammar;

    tokenize(line: string, hasEOL: boolean, state: IState): TokenizationResult {
        // Use TextMate grammar to tokenize
        const result = this._grammar.tokenizeLine(line, state);

        const tokens: IToken[] = [];
        for (const token of result.tokens) {
            tokens.push({
                startIndex: token.startIndex,
                endIndex: token.endIndex,
                scopes: token.scopes
            });
        }

        return {
            tokens,
            endState: result.ruleStack
        };
    }
}
```

### **How Syntax Highlighting Works**
1. **Grammar Loading**: Load TextMate grammar for language
2. **Line Tokenization**: Break each line into tokens
3. **Scope Assignment**: Assign semantic scopes to tokens
4. **Theme Mapping**: Map scopes to colors via theme
5. **Rendering**: Display colored text in editor

## 📊 Model Services Integration

### **Language Features Connection**
```typescript
export class TextModel {
    // Connect to language services
    private _attachLanguageFeatures(): void {
        // IntelliSense
        this._register(this._languageService.registerCompletionProvider(
            this._languageId,
            new CompletionProvider(this)
        ));

        // Error checking
        this._register(this._languageService.registerDiagnosticsProvider(
            this._languageId,
            new DiagnosticsProvider(this)
        ));

        // Code formatting
        this._register(this._languageService.registerDocumentFormattingProvider(
            this._languageId,
            new FormattingProvider(this)
        ));
    }
}
```

## 🔄 Change Events System

### **Content Change Events**
```typescript
export interface IModelContentChangedEvent {
    // What changed
    changes: IModelContentChange[];

    // Document version after changes
    versionId: number;

    // Was this an undo/redo?
    isUndoing: boolean;
    isRedoing: boolean;

    // Flush event (major change)
    isFlush: boolean;
}

export interface IModelContentChange {
    // Where the change happened
    range: Range;

    // Length of text that was replaced
    rangeLength: number;

    // New text that was inserted
    text: string;

    // Offset in the document
    rangeOffset: number;
}
```

### **Event Flow Example**
```typescript
// User types "hello" at position (1,1)
const changeEvent: IModelContentChangedEvent = {
    changes: [{
        range: new Range(1, 1, 1, 1),    // Insert at (1,1)
        rangeLength: 0,                   // Nothing replaced
        text: 'hello',                    // Text inserted
        rangeOffset: 0                    // Start of document
    }],
    versionId: 42,                        // New version
    isUndoing: false,
    isRedoing: false,
    isFlush: false
};
```

## 🎯 Real-World Examples

### **Creating a Text Model**
```typescript
// Create a new text model
const model = monaco.editor.createModel(
    'console.log("Hello World");',  // Initial content
    'javascript',                   // Language
    monaco.Uri.file('/hello.js')    // URI
);

// Listen for changes
model.onDidChangeContent((e) => {
    console.log('Content changed:', e.changes);
});

// Make an edit
model.pushEditOperations([], [{
    range: new Range(1, 1, 1, 1),
    text: '// Comment\n'
}], () => null);
```

### **Working with Decorations**
```typescript
// Add error decoration
const decorations = model.deltaDecorations([], [{
    range: new Range(1, 8, 1, 13),  // Highlight "log"
    options: {
        className: 'error-highlight',
        hoverMessage: { value: 'Unknown method' },
        glyphMarginClassName: 'error-glyph'
    }
}]);

// Update decoration
model.deltaDecorations(decorations, [{
    range: new Range(1, 8, 1, 13),
    options: {
        className: 'warning-highlight',
        hoverMessage: { value: 'Deprecated method' }
    }
}]);

// Remove decorations
model.deltaDecorations(decorations, []);
```

### **Undo/Redo Operations**
```typescript
// Make some changes
model.pushEditOperations([], [
    { range: new Range(1, 1, 1, 1), text: 'const ' },
    { range: new Range(1, 7, 1, 7), text: 'message = ' }
], () => null);

// Undo the changes
model.undo();

// Redo the changes
model.redo();

// Check if undo/redo is available
console.log('Can undo:', model.canUndo());
console.log('Can redo:', model.canRedo());
```

## 🔧 Advanced Text Model Features

### **Version Management**
```typescript
export class TextModel {
    private _versionId: number = 1;
    private _alternativeVersionId: number = 1;

    getVersionId(): number {
        return this._versionId;
    }

    getAlternativeVersionId(): number {
        // Used for tracking "clean" state
        return this._alternativeVersionId;
    }

    private _increaseVersionId(): void {
        this._versionId++;
        // Alternative version only increases for user changes
        if (!this._isUndoing && !this._isRedoing) {
            this._alternativeVersionId = this._versionId;
        }
    }
}
```

### **Position and Range Utilities**
```typescript
// Convert offset to position
const position = model.getPositionAt(100);  // { lineNumber: 5, column: 10 }

// Convert position to offset
const offset = model.getOffsetAt(new Position(5, 10));  // 100

// Get word at position
const wordInfo = model.getWordAtPosition(new Position(5, 10));
// { word: 'console', startColumn: 5, endColumn: 12 }

// Find text in model
const matches = model.findMatches('console', false, false, false, null, false);
// Array of Range objects where 'console' appears
```

### **Line Operations**
```typescript
// Get line content
const lineContent = model.getLineContent(5);  // "console.log('hello');"

// Get line count
const lineCount = model.getLineCount();  // 100

// Get line length
const lineLength = model.getLineLength(5);  // 21

// Get line first/last non-whitespace column
const firstNonWhitespace = model.getLineFirstNonWhitespaceColumn(5);  // 5
const lastNonWhitespace = model.getLineLastNonWhitespaceColumn(5);   // 21
```

## 🚀 Performance Optimizations

### **Efficient Text Storage**
```typescript
// Text buffer uses piece table for large files
class PieceTreeTextBuffer implements ITextBuffer {
    private _pieceTree: PieceTreeBase;

    // O(log n) insertions and deletions
    applyEdits(operations: IIdentifiedSingleEditOperation[]): void {
        for (const op of operations) {
            this._pieceTree.delete(op.range.startOffset, op.range.endOffset);
            this._pieceTree.insert(op.range.startOffset, op.text);
        }
    }
}
```

### **Lazy Tokenization**
```typescript
// Only tokenize visible lines
class TokenizationSupport {
    tokenizeViewport(startLine: number, endLine: number): void {
        for (let line = startLine; line <= endLine; line++) {
            if (!this._tokenizedLines.has(line)) {
                this._tokenizeLine(line);
            }
        }
    }
}
```

### **Decoration Optimization**
```typescript
// Decorations are stored in interval trees for fast range queries
class DecorationsTree {
    private _intervalTree: IntervalTree<ModelDecoration>;

    // O(log n + k) where k is number of results
    getDecorationsInRange(range: Range): ModelDecoration[] {
        return this._intervalTree.queryInterval(
            range.startOffset,
            range.endOffset
        );
    }
}
```

## 🎯 Text Model in Action

### **Real Editor Integration**
```typescript
// How the editor uses the text model
export class CodeEditorWidget {
    private _model: ITextModel | null = null;

    setModel(model: ITextModel | null): void {
        // Disconnect from old model
        if (this._model) {
            this._modelListeners.dispose();
        }

        this._model = model;

        if (this._model) {
            // Connect to new model
            this._modelListeners.add(
                this._model.onDidChangeContent((e) => {
                    this._onModelContentChanged(e);
                })
            );

            this._modelListeners.add(
                this._model.onDidChangeDecorations((e) => {
                    this._onModelDecorationsChanged(e);
                })
            );
        }

        // Update view
        this._view.setModel(this._model);
    }
}
```

## 🔍 Debugging Text Model

### **Common Issues and Solutions**

#### **Memory Leaks**
```typescript
// Always dispose models when done
const model = monaco.editor.createModel(content, language);

// Use the model...

// Clean up
model.dispose();
```

#### **Performance Issues**
```typescript
// Batch operations for better performance
model.pushEditOperations([], [
    // Multiple operations at once
    { range: range1, text: 'text1' },
    { range: range2, text: 'text2' },
    { range: range3, text: 'text3' }
], () => null);

// Instead of multiple separate operations
```

#### **Decoration Problems**
```typescript
// Always use deltaDecorations for updates
let decorationIds: string[] = [];

// Add decorations
decorationIds = model.deltaDecorations([], newDecorations);

// Update decorations
decorationIds = model.deltaDecorations(decorationIds, updatedDecorations);

// Remove decorations
model.deltaDecorations(decorationIds, []);
```

## 📚 Next Steps

Now that you understand the Text Model system:

1. **[13-editor-contributions.md](./13-editor-contributions.md)** - Learn about all editor features
2. **[14-language-support.md](./14-language-support.md)** - Understand language integration
3. **[15-syntax-highlighting.md](./15-syntax-highlighting.md)** - Deep dive into tokenization

## 🎯 Key Takeaways

The Text Model is VS Code's foundation because it:

- **Manages Content**: Efficiently stores and modifies text
- **Tracks Changes**: Provides undo/redo and change history
- **Enables Features**: Powers IntelliSense, errors, and highlighting
- **Optimizes Performance**: Uses smart data structures for speed
- **Coordinates Everything**: Acts as the central hub for all editor features

Understanding the Text Model helps you:
- **Debug editor issues** more effectively
- **Build better extensions** that work with the model
- **Optimize performance** by understanding the underlying system
- **Implement custom editors** using Monaco

The Text Model is truly the "smart document" that makes VS Code's editing experience so powerful! 📄✨
