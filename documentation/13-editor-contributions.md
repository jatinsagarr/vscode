# 🧩 VS Code Editor Contributions - Complete Feature Guide

## 🎯 Overview

Editor Contributions are the individual features that make VS Code's editor so powerful. Think of them as "plugins" built into the editor - each one adds a specific capability like find/replace, IntelliSense, or code folding. There are 40+ contributions working together to create the editing experience you know and love.

## 🧠 What are Editor Contributions?

### **Simple Analogy: The Toolbox**
Imagine VS Code's editor as a workshop, and each contribution is a specialized tool:
- **Find/Replace**: The search magnifying glass
- **IntelliSense**: The smart assistant that suggests tools
- **Code Folding**: The organizer that tidies up your workspace
- **Bracket Matching**: The helper that finds matching pairs
- **Format Document**: The cleaner that organizes everything

Each tool (contribution) knows exactly when and how to help you!

## 🏗️ Contribution Architecture

### **How Contributions Work**
```
Editor Contributions System
├── 📝 Editor Instance
├── 🔌 Contribution Registry
├── 🧩 Individual Contributions
│   ├── 🔍 Find Controller
│   ├── 💡 Suggest Controller
│   ├── 📁 Folding Controller
│   └── ⌨️  Keyboard Controller
└── 🎯 Feature Coordination
```

### **Contribution Lifecycle**
```typescript
// Every contribution follows this pattern
export class MyEditorContribution implements IEditorContribution {
    public static readonly ID = 'editor.contrib.myFeature';

    constructor(
        private readonly _editor: ICodeEditor,
        @IContextKeyService private readonly _contextKeyService: IContextKeyService
    ) {
        // Initialize the feature
        this._setupEventListeners();
        this._registerCommands();
    }

    // Called when editor is disposed
    dispose(): void {
        // Clean up resources
    }
}

// Register the contribution
registerEditorContribution(MyEditorContribution.ID, MyEditorContribution);
```

## 🔍 Core Editor Contributions

### **1. Find Controller (`src/vs/editor/contrib/find/`)**

#### **The Search Master**
```typescript
export class FindController implements IEditorContribution {
    public static readonly ID = 'editor.contrib.findController';

    private _findWidget: FindWidget;
    private _findModel: FindModelBoundToEditorModel;

    // Show find widget
    start(opts: IFindStartOptions): void {
        this._findWidget.reveal();

        if (opts.seedSearchStringFromSelection) {
            const selection = this._editor.getSelection();
            const selectedText = this._editor.getModel()?.getValueInRange(selection);
            this._findWidget.setSearchString(selectedText);
        }
    }

    // Find next match
    moveToNextMatch(): boolean {
        return this._findModel.findNextMatch();
    }
}
```

#### **Find Widget UI**
```typescript
export class FindWidget extends Widget {
    private _searchInput: FindInput;
    private _replaceInput: ReplaceInput;
    private _toggles: {
        caseSensitive: Checkbox;
        wholeWords: Checkbox;
        regex: Checkbox;
    };

    // Update search results
    private _updateMatchesCount(): void {
        const model = this._findModel;
        const count = model.getCount();
        const currentMatch = model.getCurrentMatch();

        this._matchesCount.textContent =
            count > 0 ? `${currentMatch} of ${count}` : 'No results';
    }
}
```

#### **What Find Controller Does**
- **Search**: Find text in the document
- **Replace**: Replace found text with new text
- **Navigation**: Jump between search results
- **Options**: Case sensitive, whole words, regex
- **UI**: Shows the find widget at the top

### **2. Suggest Controller (`src/vs/editor/contrib/suggest/`)**

#### **The IntelliSense Brain**
```typescript
export class SuggestController implements IEditorContribution {
    public static readonly ID = 'editor.contrib.suggestController';

    private _model: SuggestModel;
    private _widget: SuggestWidget;
    private _alternatives: SuggestAlternatives;

    // Trigger IntelliSense
    triggerSuggest(onlyFrom?: Set<CompletionItemProvider>): void {
        if (this._model.state !== State.Idle) {
            return; // Already showing suggestions
        }

        this._model.trigger({
            auto: false,
            shy: false,
            onlyFrom
        });
    }

    // Accept selected suggestion
    acceptSelectedSuggestion(): void {
        const item = this._widget.getFocusedItem();
        if (item) {
            this._model.accept(item);
        }
    }
}
```

#### **Suggestion Model**
```typescript
export class SuggestModel {
    // Get suggestions from language services
    async trigger(context: SuggestTriggerContext): Promise<void> {
        const position = this._editor.getPosition();
        const model = this._editor.getModel();

        // Ask all completion providers
        const suggestions = await provideSuggestionItems(
            model,
            position,
            this._completionOptions,
            context
        );

        // Filter and sort suggestions
        const filtered = this._filterSuggestions(suggestions);
        const sorted = this._sortSuggestions(filtered);

        // Show in widget
        this._widget.showSuggestions(sorted);
    }
}
```

#### **What Suggest Controller Does**
- **Triggers**: Shows IntelliSense popup automatically or on Ctrl+Space
- **Filters**: Narrows suggestions as you type
- **Sorts**: Orders suggestions by relevance
- **Accepts**: Inserts selected suggestion into code
- **UI**: Shows the suggestion popup with documentation

### **3. Folding Controller (`src/vs/editor/contrib/folding/`)**

#### **The Code Organizer**
```typescript
export class FoldingController implements IEditorContribution {
    public static readonly ID = 'editor.contrib.foldingController';

    private _foldingModel: FoldingModel;
    private _foldingDecorations: FoldingDecorations;

    // Fold a region
    fold(levels: number): void {
        const foldingRegions = this._foldingModel.getFoldingRegions();
        const toFold: FoldingRegion[] = [];

        for (const region of foldingRegions) {
            if (region.level <= levels && !region.isCollapsed) {
                toFold.push(region);
            }
        }

        this._foldingModel.toggleCollapseState(toFold);
    }

    // Unfold all
    unfoldAll(): void {
        const foldingRegions = this._foldingModel.getFoldingRegions();
        const toUnfold = foldingRegions.filter(r => r.isCollapsed);
        this._foldingModel.toggleCollapseState(toUnfold);
    }
}
```

#### **Folding Regions Detection**
```typescript
export class IndentRangeProvider implements FoldingRangeProvider {
    // Detect foldable regions based on indentation
    provideFoldingRanges(model: ITextModel): FoldingRange[] {
        const ranges: FoldingRange[] = [];
        const lines = model.getLinesContent();

        for (let i = 0; i < lines.length; i++) {
            const line = lines[i];
            const indent = this._getIndentLevel(line);

            // Find end of this indentation block
            const endLine = this._findBlockEnd(lines, i, indent);

            if (endLine > i + 1) {
                ranges.push({
                    start: i + 1,
                    end: endLine,
                    kind: FoldingRangeKind.Region
                });
            }
        }

        return ranges;
    }
}
```

#### **What Folding Controller Does**
- **Detects**: Finds foldable code regions (functions, classes, blocks)
- **Visualizes**: Shows fold/unfold icons in the gutter
- **Collapses**: Hides code sections to reduce clutter
- **Expands**: Shows hidden code when needed
- **Shortcuts**: Provides keyboard shortcuts for folding operations

### **4. Bracket Matching (`src/vs/editor/contrib/bracketMatching/`)**

#### **The Pair Finder**
```typescript
export class BracketMatchingController implements IEditorContribution {
    public static readonly ID = 'editor.contrib.bracketMatchingController';

    private _decorations: string[] = [];

    // Find and highlight matching brackets
    private _updateBrackets(): void {
        const position = this._editor.getPosition();
        const model = this._editor.getModel();

        if (!position || !model) {
            return;
        }

        // Find bracket at current position
        const bracket = this._findBracketAtPosition(model, position);
        if (!bracket) {
            this._removeBracketDecorations();
            return;
        }

        // Find matching bracket
        const matchingBracket = this._findMatchingBracket(model, bracket);
        if (!matchingBracket) {
            return;
        }

        // Highlight both brackets
        this._decorations = model.deltaDecorations(this._decorations, [
            {
                range: bracket.range,
                options: { className: 'bracket-match' }
            },
            {
                range: matchingBracket.range,
                options: { className: 'bracket-match' }
            }
        ]);
    }
}
```

#### **What Bracket Matching Does**
- **Highlights**: Shows matching brackets when cursor is near one
- **Jumps**: Ctrl+Shift+\ to jump to matching bracket
- **Validates**: Helps identify mismatched brackets
- **Visual**: Provides visual feedback for code structure

### **5. Code Action Controller (`src/vs/editor/contrib/codeAction/`)**

#### **The Problem Solver**
```typescript
export class CodeActionController implements IEditorContribution {
    public static readonly ID = 'editor.contrib.codeActionController';

    private _lightBulbWidget: LightBulbWidget;
    private _codeActionMenu: CodeActionMenu;

    // Show available code actions
    async showCodeActions(trigger: CodeActionTriggerType): Promise<void> {
        const model = this._editor.getModel();
        const position = this._editor.getPosition();
        const selection = this._editor.getSelection();

        // Get code actions from language services
        const codeActions = await getCodeActions(
            model,
            selection || Range.fromPositions(position),
            { type: trigger }
        );

        if (codeActions.validActions.length > 0) {
            // Show light bulb
            this._lightBulbWidget.show();

            // Show menu when clicked
            this._codeActionMenu.show(codeActions.validActions);
        }
    }

    // Apply a code action
    async applyCodeAction(action: CodeAction): Promise<void> {
        if (action.edit) {
            await this._bulkEditService.apply(action.edit);
        }

        if (action.command) {
            await this._commandService.executeCommand(
                action.command.id,
                ...action.command.arguments
            );
        }
    }
}
```

#### **What Code Action Controller Does**
- **Detects**: Finds available quick fixes and refactorings
- **Shows**: Displays light bulb icon when actions are available
- **Lists**: Shows menu of available actions
- **Applies**: Executes selected code actions
- **Examples**: "Add missing import", "Extract method", "Fix typo"

## 🎨 UI Enhancement Contributions

### **6. Hover Controller (`src/vs/editor/contrib/hover/`)**

#### **The Information Provider**
```typescript
export class ModesHoverController implements IEditorContribution {
    public static readonly ID = 'editor.contrib.hover';

    private _hoverWidget: ModesContentHoverWidget;
    private _isMouseDown: boolean = false;

    // Show hover information
    private async _showHover(position: Position): Promise<void> {
        const model = this._editor.getModel();

        // Get hover information from language services
        const hovers = await getHover(model, position);

        if (hovers && hovers.length > 0) {
            const hover = hovers[0];

            this._hoverWidget.showAt(position, {
                contents: hover.contents,
                range: hover.range
            });
        }
    }

    // Handle mouse hover
    private _onEditorMouseMove(e: IEditorMouseEvent): void {
        if (this._isMouseDown) {
            return; // Don't show hover while dragging
        }

        const position = e.target.position;
        if (position) {
            // Debounce hover requests
            this._hoverTimer.cancelAndSet(() => {
                this._showHover(position);
            }, 300);
        }
    }
}
```

#### **What Hover Controller Does**
- **Shows**: Displays information when hovering over code
- **Content**: Type information, documentation, error messages
- **Timing**: Appears after a short delay
- **Positioning**: Smart positioning to avoid screen edges

### **7. Parameter Hints (`src/vs/editor/contrib/parameterHints/`)**

#### **The Function Helper**
```typescript
export class ParameterHintsController implements IEditorContribution {
    public static readonly ID = 'editor.contrib.parameterHintsController';

    private _widget: ParameterHintsWidget;
    private _model: ParameterHintsModel;

    // Trigger parameter hints
    trigger(): void {
        const position = this._editor.getPosition();
        const model = this._editor.getModel();

        // Check if we're inside a function call
        const signatureHelp = this._getSignatureHelp(model, position);

        if (signatureHelp && signatureHelp.signatures.length > 0) {
            this._widget.show();
            this._widget.render(signatureHelp);
        }
    }

    // Navigate between overloads
    previous(): void {
        this._model.previous();
    }

    next(): void {
        this._model.next();
    }
}
```

#### **What Parameter Hints Does**
- **Triggers**: Shows when typing function calls
- **Information**: Function signature and parameter info
- **Navigation**: Arrow keys to see different overloads
- **Highlighting**: Shows current parameter being typed

## ⌨️ Input and Navigation Contributions

### **8. Multi-Cursor Controller (`src/vs/editor/contrib/multicursor/`)**

#### **The Multi-Selection Master**
```typescript
export class MultiCursorSelectionController implements IEditorContribution {
    public static readonly ID = 'editor.contrib.multiCursorController';

    // Add cursor above
    addCursorUp(): void {
        const selections = this._editor.getSelections();
        const newSelections: Selection[] = [];

        for (const selection of selections) {
            newSelections.push(selection);

            // Add cursor one line up
            const newPosition = new Position(
                Math.max(1, selection.startLineNumber - 1),
                selection.startColumn
            );
            newSelections.push(new Selection(
                newPosition.lineNumber,
                newPosition.column,
                newPosition.lineNumber,
                newPosition.column
            ));
        }

        this._editor.setSelections(newSelections);
    }

    // Select all occurrences of current word
    selectAll(): void {
        const selection = this._editor.getSelection();
        const model = this._editor.getModel();
        const word = model.getWordAtPosition(selection.getStartPosition());

        if (word) {
            const matches = model.findMatches(
                word.word,
                false, // searchOnlyEditableRange
                false, // isRegex
                true,  // matchCase
                true,  // matchWholeWord
                false  // captureMatches
            );

            const selections = matches.map(match =>
                Selection.fromPositions(
                    model.getPositionAt(match.range.startOffset),
                    model.getPositionAt(match.range.endOffset)
                )
            );

            this._editor.setSelections(selections);
        }
    }
}
```

#### **What Multi-Cursor Controller Does**
- **Multiple Cursors**: Ctrl+Alt+Up/Down to add cursors
- **Word Selection**: Ctrl+D to select next occurrence
- **All Occurrences**: Ctrl+Shift+L to select all occurrences
- **Editing**: Type once, edit everywhere

### **9. Word Operations (`src/vs/editor/contrib/wordOperations/`)**

#### **The Word Navigator**
```typescript
export class WordOperations {
    // Move cursor to next word
    static moveWordLeft(editor: ICodeEditor): void {
        const position = editor.getPosition();
        const model = editor.getModel();

        const newPosition = this._findPreviousWordStart(model, position);
        editor.setPosition(newPosition);
    }

    // Select to next word
    static selectWordRight(editor: ICodeEditor): void {
        const selection = editor.getSelection();
        const model = editor.getModel();

        const newPosition = this._findNextWordEnd(model, selection.getEndPosition());
        const newSelection = selection.setEndPosition(
            newPosition.lineNumber,
            newPosition.column
        );

        editor.setSelection(newSelection);
    }

    // Delete word
    static deleteWordLeft(editor: ICodeEditor): void {
        const position = editor.getPosition();
        const model = editor.getModel();

        const wordStart = this._findPreviousWordStart(model, position);
        const range = Range.fromPositions(wordStart, position);

        editor.executeEdits('deleteWordLeft', [{
            range: range,
            text: ''
        }]);
    }
}
```

#### **What Word Operations Does**
- **Navigation**: Ctrl+Left/Right to jump by words
- **Selection**: Ctrl+Shift+Left/Right to select words
- **Deletion**: Ctrl+Backspace/Delete to delete words
- **Smart**: Understands camelCase and snake_case

## 🎯 Advanced Feature Contributions

### **10. Rename Controller (`src/vs/editor/contrib/rename/`)**

#### **The Symbol Renamer**
```typescript
export class RenameController implements IEditorContribution {
    public static readonly ID = 'editor.contrib.renameController';

    private _renameInputField: RenameInputField;

    // Start rename operation
    async run(): Promise<void> {
        const position = this._editor.getPosition();
        const model = this._editor.getModel();

        // Check if symbol can be renamed
        const renameLocation = await this._languageService.prepareRename(
            model.uri,
            position
        );

        if (!renameLocation) {
            this._notificationService.info('Cannot rename this symbol');
            return;
        }

        // Show rename input box
        this._renameInputField.show(renameLocation.range, renameLocation.text);
    }

    // Apply rename
    async acceptRenameInput(newName: string): Promise<void> {
        const position = this._editor.getPosition();
        const model = this._editor.getModel();

        // Get all rename locations
        const edit = await this._languageService.provideRenameEdits(
            model.uri,
            position,
            newName
        );

        if (edit) {
            // Apply changes across all files
            await this._bulkEditService.apply(edit);
        }
    }
}
```

#### **What Rename Controller Does**
- **Triggers**: F2 to rename symbol under cursor
- **Validation**: Checks if symbol can be renamed
- **Preview**: Shows all locations that will be changed
- **Apply**: Renames across all files in workspace

### **11. Go to Definition (`src/vs/editor/contrib/gotoSymbol/`)**

#### **The Code Navigator**
```typescript
export class GotoDefinitionAtPositionEditorContribution implements IEditorContribution {
    public static readonly ID = 'editor.contrib.gotoDefinitionAtPosition';

    // Go to definition
    async gotoDefinition(position: Position): Promise<void> {
        const model = this._editor.getModel();

        // Get definition locations
        const definitions = await this._languageService.provideDefinition(
            model.uri,
            position
        );

        if (definitions && definitions.length > 0) {
            if (definitions.length === 1) {
                // Single definition - jump directly
                this._gotoLocation(definitions[0]);
            } else {
                // Multiple definitions - show peek widget
                this._showPeekWidget(definitions);
            }
        }
    }

    // Navigate to location
    private _gotoLocation(location: Location): void {
        this._editorService.openCodeEditor({
            resource: location.uri,
            options: {
                selection: Range.collapseToStart(location.range)
            }
        }, this._editor);
    }
}
```

#### **What Go to Definition Does**
- **Navigation**: F12 or Ctrl+Click to go to definition
- **Peek**: Alt+F12 to peek definition inline
- **Multiple**: Shows list when multiple definitions exist
- **Cross-file**: Works across different files

## 🔧 Utility Contributions

### **12. Comment Controller (`src/vs/editor/contrib/comment/`)**

#### **The Comment Manager**
```typescript
export class CommentController implements IEditorContribution {
    public static readonly ID = 'editor.contrib.commentController';

    // Toggle line comment
    toggleLineComment(): void {
        const selections = this._editor.getSelections();
        const model = this._editor.getModel();
        const languageId = model.getLanguageId();

        // Get comment configuration for language
        const commentConfig = this._languageConfigurationService
            .getLanguageConfiguration(languageId).comments;

        if (!commentConfig?.lineComment) {
            return; // Language doesn't support line comments
        }

        const lineCommentStr = commentConfig.lineComment;
        const edits: IIdentifiedSingleEditOperation[] = [];

        for (const selection of selections) {
            const startLine = selection.startLineNumber;
            const endLine = selection.endLineNumber;

            for (let line = startLine; line <= endLine; line++) {
                const lineContent = model.getLineContent(line);
                const trimmed = lineContent.trim();

                if (trimmed.startsWith(lineCommentStr)) {
                    // Remove comment
                    const commentIndex = lineContent.indexOf(lineCommentStr);
                    edits.push({
                        range: new Range(line, commentIndex + 1, line, commentIndex + 1 + lineCommentStr.length),
                        text: ''
                    });
                } else {
                    // Add comment
                    const firstNonWhitespace = model.getLineFirstNonWhitespaceColumn(line);
                    edits.push({
                        range: new Range(line, firstNonWhitespace, line, firstNonWhitespace),
                        text: lineCommentStr + ' '
                    });
                }
            }
        }

        this._editor.executeEdits('toggleLineComment', edits);
    }
}
```

#### **What Comment Controller Does**
- **Toggle**: Ctrl+/ to toggle line comments
- **Block**: Ctrl+Shift+A for block comments
- **Smart**: Preserves indentation and formatting
- **Language-aware**: Uses correct comment syntax for each language

## 📊 All Editor Contributions List

### **Core Editing**
1. **Find Controller** - Search and replace
2. **Multi-Cursor** - Multiple selections
3. **Word Operations** - Word-based navigation
4. **Line Operations** - Line-based editing
5. **Comment Controller** - Comment toggling
6. **Clipboard** - Copy/paste operations
7. **Undo/Redo** - Edit history management

### **IntelliSense & Language**
8. **Suggest Controller** - Auto-completion
9. **Parameter Hints** - Function signatures
10. **Hover** - Information on hover
11. **Code Action** - Quick fixes and refactoring
12. **Rename** - Symbol renaming
13. **Go to Definition** - Navigation to definitions
14. **Go to Symbol** - Symbol navigation
15. **Document Symbols** - Outline view
16. **References** - Find all references

### **Visual Enhancements**
17. **Bracket Matching** - Matching bracket highlighting
18. **Folding** - Code region folding
19. **Word Highlighter** - Highlight word occurrences
20. **Selection Highlighter** - Highlight selections
21. **Indentation Guides** - Visual indentation
22. **Color Picker** - Color value editing
23. **Links** - Clickable links in code

### **Advanced Features**
24. **Snippet Controller** - Code snippets
25. **Format** - Code formatting
26. **Smart Select** - Expand/shrink selection
27. **Toggle Tab Focus** - Tab navigation mode
28. **Contextmenu** - Right-click menu
29. **Drag and Drop** - Text drag and drop
30. **Cursor Undo** - Cursor position history

### **Accessibility**
31. **Accessibility Help** - Screen reader support
32. **Unusual Line Terminators** - Line ending detection
33. **Viewport Semantic Tokens** - Semantic highlighting

### **Performance & Optimization**
34. **Inline Completions** - Ghost text suggestions
35. **Linked Editing** - Synchronized editing
36. **Caret Operations** - Cursor movement
37. **Quick Access** - Quick command access

## 🎯 How Contributions Work Together

### **Example: IntelliSense Flow**
```
User types "console." → Suggest Controller triggers
                    ↓
Language Service provides completions → Suggest Model filters
                    ↓
Suggest Widget shows popup → User selects "log"
                    ↓
Text Model applies edit → All other contributions update
```

### **Example: Find and Replace**
```
User presses Ctrl+F → Find Controller shows widget
                   ↓
User types search term → Find Model searches document
                   ↓
Text Model decorations highlight matches → User navigates results
                   ↓
User replaces text → Text Model applies changes → Undo stack updated
```

## 🔧 Creating Custom Contributions

### **Basic Contribution Template**
```typescript
export class MyCustomContribution implements IEditorContribution {
    public static readonly ID = 'editor.contrib.myCustom';

    constructor(
        private readonly _editor: ICodeEditor,
        @IContextKeyService private readonly _contextKeyService: IContextKeyService
    ) {
        this._register();
    }

    private _register(): void {
        // Register commands
        this._editor.addCommand(KeyMod.CtrlCmd | KeyCode.KeyK, () => {
            this._doSomething();
        });

        // Listen to events
        this._editor.onDidChangeModelContent(() => {
            this._onContentChanged();
        });
    }

    private _doSomething(): void {
        // Your custom logic here
    }

    private _onContentChanged(): void {
        // React to content changes
    }

    dispose(): void {
        // Clean up resources
    }
}

// Register the contribution
registerEditorContribution(MyCustomContribution.ID, MyCustomContribution);
```

## 📚 Next Steps

Now that you understand Editor Contributions:

1. **[14-language-support.md](./14-language-support.md)** - Learn how languages integrate
2. **[15-syntax-highlighting.md](./15-syntax-highlighting.md)** - Understand tokenization
3. **[16-workbench-core.md](./16-workbench-core.md)** - Move to the workbench layer

## 🎯 Key Takeaways

Editor Contributions are the building blocks of VS Code's editing experience:

- **Modular**: Each feature is a separate, focused contribution
- **Coordinated**: They work together seamlessly
- **Extensible**: New contributions can be added easily
- **Event-driven**: They respond to user actions and editor events
- **Language-aware**: Many integrate with language services

Understanding contributions helps you:
- **Debug editor issues** by knowing which contribution handles what
- **Build extensions** that integrate properly with existing features
- **Customize behavior** by understanding how features work
- **Contribute to VS Code** by adding new editor capabilities

Each contribution is like a specialized tool in VS Code's workshop, and together they create the powerful editing experience that millions of developers rely on! 🧩✨
