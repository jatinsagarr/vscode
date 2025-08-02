# 📁 VS Code Directory Structure - Complete Analysis

## 🎯 Overview

This document provides a comprehensive analysis of every directory and file in the VS Code repository. Understanding this structure is crucial for navigating the massive codebase effectively.

## 📊 Repository Statistics

- **Total Directories**: ~500+
- **Total Files**: ~50,000+
- **Source Code**: ~2.5M lines
- **Languages**: TypeScript (90%), JavaScript, CSS, JSON, Markdown

## 🏗️ Root Level Structure

### **Configuration Files**

#### **package.json** - Project Definition
```json
{
  "name": "code-oss-dev",
  "version": "1.98.0",
  "main": "./out/main.js",
  "type": "module"
}
```
**Purpose**: Defines dependencies, scripts, and project metadata

#### **product.json** - Product Configuration
```json
{
  "nameShort": "Code - OSS",
  "applicationName": "code-oss",
  "dataFolderName": ".vscode-oss"
}
```
**Purpose**: Product branding, URLs, and built-in extensions

#### **gulpfile.js** - Build System Entry
```javascript
import { createRequire } from 'node:module';
const require = createRequire(import.meta.url);
require('./build/gulpfile');
```
**Purpose**: Main build system entry point

#### **Other Configuration Files**
- **.nvmrc**: Node.js version (20.18.1)
- **.gitignore**: Git ignore patterns
- **.editorconfig**: Editor configuration
- **eslint.config.js**: Code linting rules
- **tsfmt.json**: TypeScript formatting
- **LICENSE.txt**: MIT license

### **Documentation Files**
- **README.md**: Project overview and setup
- **CONTRIBUTING.md**: Contribution guidelines
- **SECURITY.md**: Security policy
- **ThirdPartyNotices.txt**: Third-party licenses

## 📂 Core Directories

### **src/** - Source Code (Main Development)

#### **Entry Points**
```
src/
├── main.ts              # 🚀 Electron main process entry
├── cli.ts               # 🖥️  Command line interface
├── server-main.ts       # 🌐 Remote server entry
├── server-cli.ts        # 🔧 Server CLI entry
└── bootstrap-*.ts       # 🔄 Bootstrap modules
```

**Bootstrap Files:**
- **bootstrap-cli.ts**: CLI bootstrap
- **bootstrap-esm.ts**: ES Module setup
- **bootstrap-fork.ts**: Process forking
- **bootstrap-import.ts**: Import handling
- **bootstrap-meta.ts**: Metadata bootstrap
- **bootstrap-node.ts**: Node.js bootstrap
- **bootstrap-server.ts**: Server bootstrap
- **bootstrap-window.ts**: Window bootstrap

#### **Type Definitions (src/typings/)**
```
src/typings/
├── base-common.d.ts         # Base type definitions
├── crypto.d.ts              # Cryptography types
├── editContext.d.ts         # Edit context types
├── thenable.d.ts            # Promise-like types
├── vscode-globals-nls.d.ts  # Internationalization
├── vscode-globals-product.d.ts # Product globals
└── vscode-globals-ttp.d.ts  # Trusted Types
```

#### **VS Code API (src/vscode-dts/)**
```
src/vscode-dts/
├── vscode.d.ts              # 📋 Main VS Code API
├── vscode.proposed.*.d.ts   # 🧪 Experimental APIs (100+ files)
└── README.md                # API documentation
```

### **src/vs/** - Core VS Code Implementation

#### **Base Layer (src/vs/base/)**
```
src/vs/base/
├── browser/                 # 🌐 Browser-specific utilities
│   ├── ui/                 # UI components
│   │   ├── tree/           # Tree view components
│   │   ├── list/           # List components
│   │   ├── menu/           # Menu components
│   │   ├── button/         # Button components
│   │   └── inputbox/       # Input components
│   ├── dom.ts              # DOM manipulation
│   ├── event.ts            # Event handling
│   ├── keyboardEvent.ts    # Keyboard events
│   ├── mouseEvent.ts       # Mouse events
│   └── window.ts           # Window management
├── common/                  # 🔧 Platform-agnostic utilities
│   ├── arrays.ts           # Array utilities
│   ├── async.ts            # Async utilities
│   ├── buffer.ts           # Buffer handling
│   ├── cache.ts            # Caching mechanisms
│   ├── cancellation.ts     # Cancellation tokens
│   ├── collections.ts      # Collection utilities
│   ├── color.ts            # Color handling
│   ├── date.ts             # Date utilities
│   ├── decorators.ts       # TypeScript decorators
│   ├── errors.ts           # Error handling
│   ├── event.ts            # Event system
│   ├── glob.ts             # File globbing
│   ├── hash.ts             # Hashing utilities
│   ├── json.ts             # JSON utilities
│   ├── lifecycle.ts        # Object lifecycle
│   ├── map.ts              # Map utilities
│   ├── network.ts          # Network utilities
│   ├── objects.ts          # Object utilities
│   ├── path.ts             # Path utilities
│   ├── performance.ts      # Performance monitoring
│   ├── platform.ts         # Platform detection
│   ├── process.ts          # Process utilities
│   ├── resources.ts        # Resource management
│   ├── strings.ts          # String utilities
│   ├── types.ts            # Type utilities
│   ├── uri.ts              # URI handling
│   └── uuid.ts             # UUID generation
├── node/                    # 🖥️  Node.js-specific utilities
│   ├── config.ts           # Configuration handling
│   ├── console.ts          # Console utilities
│   ├── crypto.ts           # Cryptography
│   ├── encoding.ts         # Text encoding
│   ├── extpath.ts          # Extended path utilities
│   ├── flow.ts             # Control flow
│   ├── id.ts               # ID generation
│   ├── languagePacks.ts    # Language pack handling
│   ├── nls.ts              # Internationalization
│   ├── pfs.ts              # Promised file system
│   ├── ports.ts            # Port management
│   ├── processes.ts        # Process management
│   ├── proxy.ts            # Proxy handling
│   ├── request.ts          # HTTP requests
│   ├── shell.ts            # Shell integration
│   ├── unc.ts              # UNC path handling
│   ├── userDataPath.ts     # User data paths
│   └── zip.ts              # ZIP file handling
├── parts/                   # 🧩 Reusable UI parts
│   ├── quickinput/         # Quick input components
│   ├── sandbox/            # Sandboxed components
│   └── tree/               # Tree view components
├── test/                    # 🧪 Base layer tests
└── worker/                  # 👷 Web worker utilities
```

#### **Platform Layer (src/vs/platform/)**
```
src/vs/platform/
├── accessibility/           # ♿ Accessibility services
├── action/                  # 🎬 Action system
├── actions/                 # 🎯 Action registry
├── backup/                  # 💾 Backup services
├── clipboard/               # 📋 Clipboard operations
├── commands/                # ⚡ Command system
│   ├── common/
│   │   ├── commands.ts     # Command definitions
│   │   └── commandService.ts # Command execution
├── configuration/           # ⚙️  Configuration management
│   ├── common/
│   │   ├── configuration.ts # Configuration interfaces
│   │   ├── configurationService.ts # Configuration service
│   │   └── configurationRegistry.ts # Configuration registry
├── contextkey/              # 🔑 Context key system
├── contextview/             # 👁️  Context view system
├── dialogs/                 # 💬 Dialog system
├── editor/                  # 📝 Editor platform services
├── environment/             # 🌍 Environment detection
├── extensionManagement/     # 🔌 Extension management
├── extensions/              # 🧩 Extension system
├── files/                   # 📁 File system abstraction
│   ├── common/
│   │   ├── files.ts        # File interfaces
│   │   ├── fileService.ts  # File service
│   │   └── fileSystemProvider.ts # File system providers
├── instantiation/           # 💉 Dependency injection
│   ├── common/
│   │   ├── instantiation.ts    # Main DI container
│   │   ├── serviceCollection.ts # Service collection
│   │   └── descriptors.ts      # Service descriptors
├── ipc/                     # 📡 Inter-process communication
├── keybinding/              # ⌨️  Keyboard shortcuts
├── lifecycle/               # 🔄 Application lifecycle
├── log/                     # 📊 Logging system
├── markers/                 # 🏷️  Problem markers
├── native/                  # 🖥️  Native platform integration
├── notification/            # 🔔 Notification system
├── opener/                  # 🔗 URL/file opening
├── product/                 # 📦 Product information
├── progress/                # ⏳ Progress indication
├── quickinput/              # ⚡ Quick input system
├── registry/                # 📚 Service registry
├── remote/                  # 🌐 Remote development
├── request/                 # 🌍 HTTP request service
├── secrets/                 # 🔐 Secret storage
├── state/                   # 💾 State management
├── storage/                 # 🗄️  Storage services
├── telemetry/               # 📈 Telemetry system
├── terminal/                # 💻 Terminal services
├── theme/                   # 🎨 Theme system
├── update/                  # 🔄 Update system
├── url/                     # 🔗 URL handling
├── userData/                # 👤 User data management
├── userDataProfile/         # 👤 User profiles
├── userDataSync/            # 🔄 Settings sync
├── webview/                 # 🌐 Webview system
├── window/                  # 🪟 Window management
├── workspace/               # 📂 Workspace management
└── workspaces/              # 📂 Multi-workspace support
```

#### **Editor Layer (src/vs/editor/)**
```
src/vs/editor/
├── browser/                 # 🌐 Browser editor implementation
│   ├── controller/         # Input controllers
│   ├── services/           # Editor services
│   ├── view/               # View layer
│   ├── widget/             # Editor widgets
│   ├── codeEditor.ts       # Main code editor
│   ├── editorBrowser.ts    # Browser-specific editor
│   └── editorExtensions.ts # Editor extensions
├── common/                  # 🔧 Editor core logic
│   ├── config/             # Editor configuration
│   ├── controller/         # Core controllers
│   ├── core/               # Core editor logic
│   ├── diff/               # Diff editor
│   ├── languages/          # Language support
│   ├── model/              # Text model
│   ├── modes/              # Language modes
│   ├── services/           # Editor services
│   ├── standalone/         # Standalone editor
│   ├── view/               # View model
│   └── viewModel/          # View model implementation
├── contrib/                 # 🧩 Editor contributions
│   ├── anchorSelect/       # Anchor selection
│   ├── bracketMatching/    # Bracket matching
│   ├── caretOperations/    # Caret operations
│   ├── clipboard/          # Clipboard operations
│   ├── codeAction/         # Code actions
│   ├── codelens/           # Code lens
│   ├── colorPicker/        # Color picker
│   ├── comment/            # Comment operations
│   ├── contextmenu/        # Context menu
│   ├── cursorUndo/         # Cursor undo
│   ├── dnd/                # Drag and drop
│   ├── documentSymbols/    # Document symbols
│   ├── find/               # Find and replace
│   ├── folding/            # Code folding
│   ├── format/             # Code formatting
│   ├── gotoError/          # Go to error
│   ├── gotoLine/           # Go to line
│   ├── gotoSymbol/         # Go to symbol
│   ├── hover/              # Hover information
│   ├── inPlaceReplace/     # In-place replace
│   ├── indentation/        # Indentation
│   ├── inlineCompletions/  # Inline completions
│   ├── inlineEdit/         # Inline editing
│   ├── linesOperations/    # Line operations
│   ├── linkedEditing/      # Linked editing
│   ├── links/              # Link detection
│   ├── multicursor/        # Multi-cursor
│   ├── parameterHints/     # Parameter hints
│   ├── quickAccess/        # Quick access
│   ├── rename/             # Symbol renaming
│   ├── smartSelect/        # Smart selection
│   ├── snippet/            # Code snippets
│   ├── suggest/            # IntelliSense
│   ├── toggleTabFocusMode/ # Tab focus mode
│   ├── unusualLineTerminators/ # Line terminators
│   ├── viewportSemanticTokens/ # Semantic tokens
│   ├── wordHighlighter/    # Word highlighting
│   ├── wordOperations/     # Word operations
│   └── wordPartOperations/ # Word part operations
├── standalone/              # 🏃 Standalone Monaco editor
│   ├── browser/            # Browser standalone
│   ├── common/             # Common standalone
│   └── test/               # Standalone tests
├── test/                    # 🧪 Editor tests
├── editor.all.ts           # All editor imports
├── editor.api.ts           # Editor API
├── editor.main.ts          # Editor main entry
└── editor.worker.ts        # Editor web worker
```

#### **Code Layer (src/vs/code/)**
```
src/vs/code/
├── browser/                 # 🌐 Browser-specific code
│   ├── workbench/          # Browser workbench
│   └── sharedProcess/      # Shared process (browser)
├── electron-main/           # ⚡ Electron main process
│   ├── app.ts              # Main application
│   ├── auth.ts             # Authentication
│   ├── backup.ts           # Backup management
│   ├── keyboard.ts         # Keyboard handling
│   ├── lifecycle.ts        # App lifecycle
│   ├── logUploader.ts      # Log uploading
│   ├── main.ts             # Main entry point
│   ├── menu.ts             # Application menu
│   ├── protocol.ts         # Protocol handling
│   ├── sharedProcess.ts    # Shared process management
│   ├── update.ts           # Update management
│   ├── window.ts           # Window management
│   └── windows.ts          # Multi-window support
├── electron-sandbox/        # 🏖️  Electron sandbox
│   ├── issue/              # Issue reporting
│   ├── processExplorer/    # Process explorer
│   ├── proxy/              # Proxy configuration
│   ├── sharedProcess/      # Shared process (sandbox)
│   └── workbench/          # Workbench (sandbox)
├── electron-utility/        # 🔧 Electron utility process
│   ├── sharedProcess.ts    # Shared process utility
│   └── utilityProcess.ts   # Utility process
└── node/                    # 🖥️  Node.js specific code
    ├── cli.ts              # CLI implementation
    ├── cliProcessMain.ts   # CLI process main
    └── sharedProcess.ts    # Shared process (Node)
```

#### **Server Layer (src/vs/server/)**
```
src/vs/server/
├── node/                    # 🖥️  Server Node.js implementation
│   ├── server.main.ts      # Server main entry
│   ├── remoteExtensionHostAgentServer.ts # Remote extension host
│   └── serverServices.ts   # Server services
└── test/                    # 🧪 Server tests
```

#### **Workbench Layer (src/vs/workbench/)**
```
src/vs/workbench/
├── api/                     # 🔌 Extension API implementation
│   ├── browser/            # Browser API
│   ├── common/             # Common API
│   ├── node/               # Node API
│   └── test/               # API tests
├── browser/                 # 🌐 Browser workbench
│   ├── actions/            # Workbench actions
│   ├── codicons/           # Icon system
│   ├── contextkeys.ts      # Context keys
│   ├── layout.ts           # Layout management
│   ├── parts/              # UI parts
│   │   ├── activitybar/    # Activity bar
│   │   ├── auxiliarybar/   # Auxiliary bar
│   │   ├── banner/         # Banner
│   │   ├── editor/         # Editor area
│   │   ├── panel/          # Panel area
│   │   ├── sidebar/        # Sidebar
│   │   ├── statusbar/      # Status bar
│   │   └── titlebar/       # Title bar
│   ├── web.api.ts          # Web API
│   ├── web.factory.ts      # Web factory
│   ├── web.main.ts         # Web main
│   └── workbench.ts        # Main workbench
├── common/                  # 🔧 Common workbench code
│   ├── activity.ts         # Activity management
│   ├── contextkeys.ts      # Context keys
│   ├── editor.ts           # Editor commons
│   ├── memento.ts          # State persistence
│   ├── panel.ts            # Panel commons
│   ├── theme.ts            # Theme commons
│   └── views.ts            # View commons
├── contrib/                 # 🧩 Workbench contributions (Features)
│   ├── accessibility/      # Accessibility features
│   ├── audioCues/          # Audio cues
│   ├── backup/             # Backup features
│   ├── bulkEdit/           # Bulk editing
│   ├── callHierarchy/      # Call hierarchy
│   ├── chat/               # Chat features
│   ├── cli/                # CLI integration
│   ├── codeActions/        # Code actions
│   ├── codeEditor/         # Code editor features
│   ├── comments/           # Comments system
│   ├── debug/              # Debugging
│   ├── editSessions/       # Edit sessions
│   ├── emmet/              # Emmet support
│   ├── execution/          # Code execution
│   ├── externalTerminal/   # External terminal
│   ├── extensions/         # Extension management UI
│   ├── files/              # File explorer
│   ├── format/             # Formatting
│   ├── gettingStarted/     # Getting started
│   ├── git/                # Git integration
│   ├── inlineChat/         # Inline chat
│   ├── issue/              # Issue reporting
│   ├── keybindings/        # Keybinding editor
│   ├── languageDetection/  # Language detection
│   ├── languageStatus/     # Language status
│   ├── localHistory/       # Local history
│   ├── localization/       # Localization
│   ├── logs/               # Log viewer
│   ├── markdown/           # Markdown support
│   ├── markers/            # Problem markers
│   ├── mergeEditor/        # Merge editor
│   ├── multiDiffEditor/    # Multi-diff editor
│   ├── notebook/           # Notebook support
│   ├── outline/            # Outline view
│   ├── output/             # Output panel
│   ├── performance/        # Performance tools
│   ├── preferences/        # Settings editor
│   ├── profiles/           # User profiles
│   ├── quickaccess/        # Quick access
│   ├── relauncher/         # App relauncher
│   ├── remote/             # Remote development
│   ├── scm/                # Source control
│   ├── search/             # Search functionality
│   ├── searchEditor/       # Search editor
│   ├── share/              # Sharing features
│   ├── snippets/           # Code snippets
│   ├── speech/             # Speech features
│   ├── splash/             # Splash screen
│   ├── surveys/            # User surveys
│   ├── tasks/              # Task system
│   ├── terminal/           # Integrated terminal
│   ├── terminalContrib/    # Terminal contributions
│   ├── testing/            # Testing framework
│   ├── themes/             # Theme management
│   ├── timeline/           # Timeline view
│   ├── typeHierarchy/      # Type hierarchy
│   ├── update/             # Update notifications
│   ├── userDataProfile/    # User data profiles
│   ├── userDataSync/       # Settings sync UI
│   ├── watermark/          # Watermark
│   ├── webview/            # Webview system
│   ├── webviewPanel/       # Webview panels
│   ├── webviewView/        # Webview views
│   └── welcome/            # Welcome experience
├── electron-sandbox/        # 🏖️  Electron sandbox workbench
│   ├── desktop.contribution.ts # Desktop contributions
│   ├── desktop.main.ts     # Desktop main
│   └── parts/              # Desktop-specific parts
├── services/                # 🔧 Workbench services
│   ├── accessibility/      # Accessibility services
│   ├── activity/           # Activity services
│   ├── aiEmbeddingVector/  # AI embedding
│   ├── aiRelatedInformation/ # AI related info
│   ├── authentication/     # Authentication
│   ├── backup/             # Backup services
│   ├── banner/             # Banner services
│   ├── clipboard/          # Clipboard services
│   ├── commands/           # Command services
│   ├── configuration/      # Configuration services
│   ├── configurationResolver/ # Config resolver
│   ├── contextmenu/        # Context menu services
│   ├── decorations/        # Decoration services
│   ├── dialogs/            # Dialog services
│   ├── editor/             # Editor services
│   ├── encryption/         # Encryption services
│   ├── environment/        # Environment services
│   ├── extensionManagement/ # Extension management
│   ├── extensionRecommendations/ # Extension recommendations
│   ├── extensions/         # Extension services
│   ├── files/              # File services
│   ├── history/            # History services
│   ├── host/               # Host services
│   ├── hover/              # Hover services
│   ├── issue/              # Issue services
│   ├── keybinding/         # Keybinding services
│   ├── label/              # Label services
│   ├── language/           # Language services
│   ├── languageDetection/  # Language detection
│   ├── languageStatus/     # Language status
│   ├── layout/             # Layout services
│   ├── lifecycle/          # Lifecycle services
│   ├── localization/       # Localization services
│   ├── log/                # Log services
│   ├── menubar/            # Menubar services
│   ├── model/              # Model services
│   ├── notebook/           # Notebook services
│   ├── notification/       # Notification services
│   ├── opener/             # Opener services
│   ├── outline/            # Outline services
│   ├── output/             # Output services
│   ├── panel/              # Panel services
│   ├── path/               # Path services
│   ├── preferences/        # Preferences services
│   ├── progress/           # Progress services
│   ├── quickinput/         # Quick input services
│   ├── remote/             # Remote services
│   ├── request/            # Request services
│   ├── search/             # Search services
│   ├── statusbar/          # Status bar services
│   ├── storage/            # Storage services
│   ├── telemetry/          # Telemetry services
│   ├── terminal/           # Terminal services
│   ├── textfile/           # Text file services
│   ├── textmodelResolver/  # Text model resolver
│   ├── textresourceProperties/ # Text resource properties
│   ├── themes/             # Theme services
│   ├── timer/              # Timer services
│   ├── title/              # Title services
│   ├── tunnel/             # Tunnel services
│   ├── update/             # Update services
│   ├── url/                # URL services
│   ├── userDataProfile/    # User data profile
│   ├── userDataSync/       # User data sync
│   ├── viewlet/            # Viewlet services
│   ├── views/              # View services
│   ├── workingCopy/        # Working copy services
│   └── workspaces/         # Workspace services
├── test/                    # 🧪 Workbench tests
├── workbench.common.main.ts # Common workbench entry
├── workbench.desktop.main.ts # Desktop workbench entry
├── workbench.web.main.internal.ts # Internal web entry
└── workbench.web.main.ts    # Web workbench entry
```

### **extensions/** - Built-in Extensions

```
extensions/
├── bat/                     # Batch file support
├── clojure/                 # Clojure support
├── coffeescript/            # CoffeeScript support
├── configuration-editing/   # Configuration editing
├── cpp/                     # C++ support
├── csharp/                  # C# support
├── css/                     # CSS support
├── css-language-features/   # CSS language features
├── dart/                    # Dart support
├── debug-auto-launch/       # Debug auto launch
├── debug-server-ready/      # Debug server ready
├── diff/                    # Diff support
├── docker/                  # Docker support
├── emmet/                   # Emmet support
├── extension-editing/       # Extension editing
├── fsharp/                  # F# support
├── git/                     # Git integration
├── git-base/                # Git base functionality
├── github/                  # GitHub integration
├── go/                      # Go support
├── groovy/                  # Groovy support
├── grunt/                   # Grunt support
├── gulp/                    # Gulp support
├── handlebars/              # Handlebars support
├── hlsl/                    # HLSL support
├── html/                    # HTML support
├── html-language-features/  # HTML language features
├── ini/                     # INI file support
├── ipynb/                   # Jupyter notebook support
├── jake/                    # Jake support
├── java/                    # Java support
├── javascript/              # JavaScript support
├── json/                    # JSON support
├── json-language-features/  # JSON language features
├── julia/                   # Julia support
├── latex/                   # LaTeX support
├── less/                    # Less support
├── log/                     # Log file support
├── lua/                     # Lua support
├── make/                    # Makefile support
├── markdown-basics/         # Markdown support
├── markdown-language-features/ # Markdown language features
├── markdown-math/           # Markdown math support
├── media-preview/           # Media preview
├── merge-conflict/          # Merge conflict resolution
├── microsoft-authentication/ # Microsoft authentication
├── npm/                     # npm support
├── objective-c/             # Objective-C support
├── perl/                    # Perl support
├── php/                     # PHP support
├── php-language-features/   # PHP language features
├── powershell/              # PowerShell support
├── pug/                     # Pug support
├── python/                  # Python support
├── r/                       # R support
├── razor/                   # Razor support
├── references-view/         # References view
├── restructuredtext/        # reStructuredText support
├── ruby/                    # Ruby support
├── rust/                    # Rust support
├── scss/                    # SCSS support
├── search-result/           # Search result handling
├── shaderlab/               # ShaderLab support
├── shellscript/             # Shell script support
├── simple-browser/          # Simple browser
├── sql/                     # SQL support
├── swift/                   # Swift support
├── theme-abyss/             # Abyss theme
├── theme-defaults/          # Default themes
├── theme-kimbie-dark/       # Kimbie Dark theme
├── theme-monokai/           # Monokai theme
├── theme-monokai-dimmed/    # Monokai Dimmed theme
├── theme-quietlight/        # Quiet Light theme
├── theme-red/               # Red theme
├── theme-seti/              # Seti theme
├── theme-solarized-dark/    # Solarized Dark theme
├── theme-solarized-light/   # Solarized Light theme
├── theme-tomorrow-night-blue/ # Tomorrow Night Blue theme
├── tunnel-forwarding/       # Tunnel forwarding
├── typescript-basics/       # TypeScript support
├── typescript-language-features/ # TypeScript language features
├── vb/                      # Visual Basic support
├── vscode-api-tests/        # VS Code API tests
├── vscode-colorize-tests/   # Colorization tests
├── vscode-test-resolver/    # Test resolver
├── xml/                     # XML support
└── yaml/                    # YAML support
```

### **build/** - Build System

```
build/
├── azure-pipelines/         # 🔄 CI/CD pipelines
├── builtin/                 # Built-in extension handling
├── checksums/               # File checksums
├── darwin/                  # macOS-specific build
├── lib/                     # 🔧 Build utilities
│   ├── compilation.ts      # Compilation logic
│   ├── bundle.ts           # Bundling logic
│   ├── optimize.ts         # Optimization
│   ├── treeshaking.ts      # Tree shaking
│   └── util.ts             # Build utilities
├── linux/                   # Linux-specific build
├── monaco/                  # Monaco editor build
├── npm/                     # npm-related scripts
├── win32/                   # Windows-specific build
├── gulpfile.*.js           # 🏗️  Gulp build tasks
├── package.json            # Build dependencies
└── tsconfig.build.json     # Build TypeScript config
```

### **out/** - Compiled Output

```
out/
├── vs/                      # Compiled VS Code source
│   ├── base/               # Compiled base layer
│   ├── platform/           # Compiled platform layer
│   ├── editor/             # Compiled editor layer
│   └── workbench/          # Compiled workbench layer
├── main.js                 # Compiled main entry
├── cli.js                  # Compiled CLI
├── server-main.js          # Compiled server
└── bootstrap-*.js          # Compiled bootstrap files
```

### **test/** - Testing Infrastructure

```
test/
├── automation/             # UI automation tests
├── integration/            # Integration tests
├── leaks/                  # Memory leak tests
├── monaco/                 # Monaco editor tests
├── smoke/                  # Smoke tests
├── unit/                   # Unit tests
│   ├── browser/           # Browser unit tests
│   └── node/              # Node.js unit tests
└── package.json           # Test dependencies
```

### **scripts/** - Development Scripts

```
scripts/
├── code.sh                 # Launch development VS Code (Linux/macOS)
├── code.bat                # Launch development VS Code (Windows)
├── code-web.sh             # Launch web version
├── code-server.sh          # Launch server version
├── test.sh                 # Run tests
├── test-integration.sh     # Run integration tests
└── package.json            # Script dependencies
```

## 🔍 Key Directory Insights

### **Development Workflow Directories**

#### **.vscode/** - Workspace Configuration
```
.vscode/
├── extensions.json         # Recommended extensions
├── launch.json            # Debug configurations
├── settings.json          # Workspace settings
├── tasks.json             # Build tasks
└── shared.code-snippets   # Code snippets
```

#### **.devcontainer/** - Development Container
```
.devcontainer/
├── devcontainer.json      # Container configuration
├── Dockerfile             # Container image
├── post-create.sh         # Post-creation script
└── README.md              # Container documentation
```

### **Quality Assurance Directories**

#### **.eslint-plugin-local/** - Custom ESLint Rules
Contains 30+ custom ESLint rules specific to VS Code:
- **code-layering.ts**: Enforces architectural layers
- **code-import-patterns.ts**: Controls import patterns
- **code-no-unexternalized-strings.ts**: Ensures i18n compliance
- **vscode-dts-*.ts**: API definition rules

#### **.github/** - GitHub Configuration
```
.github/
├── workflows/             # GitHub Actions
├── ISSUE_TEMPLATE/        # Issue templates
├── commands.json          # Bot commands
├── CODEOWNERS            # Code ownership
└── pull_request_template.md # PR template
```

### **Build Output Directories**

#### **.build/** - Build Artifacts
```
.build/
├── builtInExtensions/     # Downloaded built-in extensions
├── electron/              # Electron binaries
├── node/                  # Node.js binaries
└── log*                   # Build logs
```

#### **node_modules/** - Dependencies
Contains ~1000+ npm packages including:
- **Electron**: Desktop app framework
- **TypeScript**: Language compiler
- **Monaco Editor**: Text editor component
- **Gulp**: Build system
- **Webpack**: Module bundler

## 🎯 Navigation Tips

### **Finding Specific Features**
1. **Editor Features**: Look in `src/vs/editor/contrib/`
2. **Workbench Features**: Look in `src/vs/workbench/contrib/`
3. **Platform Services**: Look in `src/vs/platform/`
4. **Extension API**: Look in `src/vs/workbench/api/`

### **Understanding Dependencies**
1. **Base Layer**: No dependencies on other layers
2. **Platform Layer**: Depends only on Base
3. **Editor Layer**: Depends on Base and Platform
4. **Workbench Layer**: Depends on all lower layers

### **Common File Patterns**
- **\*.contribution.ts**: Feature registration
- **\*.service.ts**: Service implementations
- **\*.test.ts**: Unit tests
- **\*.d.ts**: Type definitions
- **\*.css**: Stylesheets

## 📚 Next Steps

Now that you understand the directory structure:

1. **[04-build-system.md](./04-build-system.md)** - Learn how everything gets compiled
2. **[06-architecture-overview.md](./06-architecture-overview.md)** - Understand the 4-layer architecture
3. **[09-base-layer.md](./09-base-layer.md)** - Start with the foundation layer

This directory structure is the roadmap to VS Code's 2.5 million lines of code. Use it to navigate efficiently and understand how everything fits together! 🗺️
