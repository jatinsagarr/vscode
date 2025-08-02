# 🛠️ VS Code Development Setup - Complete Guide

## 🎯 Overview

This guide will walk you through setting up a complete VS Code development environment. By the end, you'll be able to build, run, debug, and modify VS Code from source.

## 📋 Prerequisites

### **System Requirements**

#### **Hardware**
- **RAM**: Minimum 8GB, recommended 16GB+
- **Storage**: 10GB+ free space for source code and dependencies
- **CPU**: Multi-core processor (compilation is CPU-intensive)

#### **Operating Systems**
- **Windows**: Windows 10/11 (x64, ARM64)
- **macOS**: macOS 10.15+ (Intel, Apple Silicon)
- **Linux**: Ubuntu 18.04+, Fedora, SUSE, etc.

### **Required Software**

#### **1. Node.js (Exact Version Required)**
```bash
# VS Code requires Node.js 20.18.1 (check .nvmrc file)
node --version  # Should output: v20.18.1
```

**Installation Options:**

**Using NVM (Recommended):**
```bash
# Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Install and use the correct Node.js version
nvm install 20.18.1
nvm use 20.18.1
nvm alias default 20.18.1
```

#### **2. Git**
```bash
git --version  # Should be 2.0+
```

#### **3. Python (for native modules)**
- **Windows**: Python 3.8+ with Visual Studio Build Tools
- **macOS**: Xcode Command Line Tools
- **Linux**: build-essential package

## 🚀 Getting Started

### **Step 1: Clone the Repository**

```bash
# Clone VS Code repository
git clone https://github.com/microsoft/vscode.git
cd vscode

# Check the current branch (usually 'main')
git branch
```

### **Step 2: Install Dependencies**

```bash
# Install all dependencies (this takes 5-10 minutes)
npm install

# Verify installation
npm list --depth=0
```

### **Step 3: Build VS Code**

#### **Full Compilation**
```bash
# Complete build (takes 10-20 minutes first time)
npm run compile

# Or using Gulp directly
npx gulp compile
```

#### **Development Build (Faster)**
```bash
# Watch mode for development (rebuilds on changes)
npm run watch

# This runs in background - keep terminal open
```

### **Step 4: Run VS Code**

#### **Desktop Version**
```bash
# Run the development version
./scripts/code.sh        # Linux/macOS
./scripts/code.bat       # Windows

# Or with specific arguments
./scripts/code.sh --verbose --log debug
```

## 🔧 Development Workflow

### **Recommended Setup**

#### **1. Use VS Code to Develop VS Code**
```bash
# Open VS Code source in VS Code
./scripts/code.sh .
```

### **Development Commands**

#### **Build Commands**
```bash
# Full compilation
npm run compile

# Watch mode (development)
npm run watch

# Clean build
npm run gulp -- clean
npm run compile
```

#### **Testing Commands**
```bash
# Run unit tests
npm run test-node          # Node.js tests
npm run test-browser       # Browser tests

# Run integration tests
npm run test-extension     # Extension tests
```

## 🐛 Debugging Setup

### **Debug VS Code in VS Code**

#### **1. Debug Main Process**
```bash
# Launch with debugging enabled
./scripts/code.sh --inspect-brk=5874
```

#### **2. Debug Renderer Process**
```bash
# Launch VS Code
./scripts/code.sh

# Open Developer Tools
# Help → Toggle Developer Tools
# Or Ctrl+Shift+I / Cmd+Option+I
```

## 🧪 Testing Your Setup

### **Verification Steps**

#### **1. Build Verification**
```bash
# Should complete without errors
npm run compile

# Check output directory
ls -la out/
ls -la out/vs/
```

#### **2. Runtime Verification**
```bash
# Launch should work
./scripts/code.sh

# Check version in Help → About
# Should show development version
```

## 📚 Next Steps

### **Development Workflow**
1. **Make Changes**: Edit source files in `src/`
2. **Watch Mode**: Keep `npm run watch` running
3. **Test Changes**: Launch with `./scripts/code.sh`
4. **Debug Issues**: Use VS Code debugger
5. **Run Tests**: Verify with `npm run test`

### **Learning Path**
1. **[03-directory-structure.md](./03-directory-structure.md)** - Understand the codebase structure
2. **[04-build-system.md](./04-build-system.md)** - Deep dive into the build process
3. **[06-architecture-overview.md](./06-architecture-overview.md)** - Learn the architecture

## 🎯 Success Checklist

- [ ] Node.js 20.18.1 installed and active
- [ ] Repository cloned successfully
- [ ] Dependencies installed without errors
- [ ] Full compilation completes successfully
- [ ] VS Code launches from source
- [ ] Watch mode works for development
- [ ] Debugger attaches successfully
- [ ] Tests run without failures

Congratulations! You now have a complete VS Code development environment. You're ready to explore the codebase and start contributing! 🚀
