# 🧩 VS Code Workbench Parts - Complete UI Components Guide

## 🎯 Overview

Workbench Parts are the individual UI components that make up VS Code's interface. Each part is responsible for a specific area of the screen and functionality. Think of them as specialized rooms in an office building, each designed for a particular purpose but all working together to create a productive workspace.

## 🧠 What are Workbench Parts?

### **Simple Analogy: The Office Building Rooms**
Imagine VS Code as a modern office building with specialized rooms:
- **Title Bar** = Building sign and security controls
- **Activity Bar** = Elevator buttons for quick navigation
- **Sidebar** = Your main filing cabinet and reference library
- **Editor Area** = Your primary workspace desk
- **Panel** = Conference room for meetings and collaboration
- **Status Bar** = Information display board
- **Auxiliary Bar** = Secondary storage and quick access area

Each room has its own purpose, but they're all connected and coordinated!

## 🏗️ Parts Architecture

### **The Complete Parts System**
```
VS Code Workbench Parts
├── 📋 Title Bar (Window Management)
│   ├── Window Controls (minimize, maximize, close)
│   ├── Menu Bar (File, Edit, View, etc.)
│   └── Title Display (current file/workspace)
├── 🎮 Activity Bar (Primary Navigation)
│   ├── Built-in Activities (Explorer, Search, SCM, Debug, Extensions)
│   ├── Extension Activities (from extensions)
│   └── Global Actions (Settings, Accounts)
├── 📁 Sidebar (Primary Side Panel)
│   ├── Viewlets (Explorer, Search, SCM, Debug, Extensions)
│   ├── Views (File tree, Search results, etc.)
│   └── View Actions (toolbar buttons)
├── 📝 Editor Area (Text Editing)
│   ├── Editor Groups (split editors)
│   ├── Editor Tabs (open files)
│   └── Editor Instances (Monaco editors)
├── 📊 Panel (Bottom Utilities)
│   ├── Built-in Panels (Terminal, Problems, Output, Debug Console)
│   ├── Extension Panels (from extensions)
│   └── Panel Actions (toolbar buttons)
├── 📋 Status Bar (Information Display)
│   ├── Left Items (language, branch, etc.)
│   ├── Right Items (position, encoding, etc.)
│   └── Background Tasks (progress indicators)
└── 🔧 Auxiliary Bar (Secondary Side Panel)
    ├── Secondary Views (Chat, Timeline, etc.)
    └── Overflow Items (when sidebar is full)
```

## 📋 Title Bar (`src/vs/workbench/browser/parts/titlebar/`)

### **The Window Header**
```typescript
export class TitlebarPart extends Part implements ITitleService {
    public static readonly ID = 'workbench.parts.titlebar';

    private _titleContainer: HTMLElement;
    private _menuBarContainer: HTMLElement;
    private _windowControls: HTMLElement;
    private _currentTitle: string = '';

    constructor(
        @IContextMenuService private readonly contextMenuService: IContextMenuService,
        @IConfigurationService private readonly configurationService: IConfigurationService,
        @IBrowserService private readonly browserService: IBrowserService,
        @IWorkbenchEnvironmentService private readonly environmentService: IWorkbenchEnvironmentService,
        @IWorkspaceContextService private readonly contextService: IWorkspaceContextService,
        @IInstantiationService private readonly instantiationService: IInstantiationService,
        @IThemeService themeService: IThemeService,
        @IStorageService storageService: IStorageService,
        @IWorkbenchLayoutService layoutService: IWorkbenchLayoutService
    ) {
        super(TitlebarPart.ID, { hasTitle: false }, themeService, storageService, layoutService);

        this.registerListeners();
    }

    createContentArea(parent: HTMLElement): HTMLElement {
        this.element = parent;

        // Create menu bar (File, Edit, View, etc.)
        this._menuBarContainer = this.createMenuBar(parent);

        // Create title display area
        this._titleContainer = this.createTitleContainer(parent);

        // Create window controls (minimize, maximize, close)
        this._windowControls = this.createWindowControls(parent);

        // Set initial title
        this.updateTitle();

        return parent;
    }

    private createMenuBar(parent: HTMLElement): HTMLElement {
        const menuBarContainer = document.createElement('div');
        menuBarContainer.className = 'menubar';
        parent.appendChild(menuBarContainer);

        // Create menu bar with standard menus
        const menuBar = this.instantiationService.createInstance(MenuBar, menuBarContainer);

        // Add standard menus
        menuBar.addMenu('File', this.createFileMenu());
        menuBar.addMenu('Edit', this.createEditMenu());
        menuBar.addMenu('View', this.createViewMenu());
        menuBar.addMenu('Go', this.createGoMenu());
        menuBar.addMenu('Run', this.createRunMenu());
        menuBar.addMenu('Terminal', this.createTerminalMenu());
        menuBar.addMenu('Help', this.createHelpMenu());

        return menuBarContainer;
    }

    private createTitleContainer(parent: HTMLElement): HTMLElement {
        const titleContainer = document.createElement('div');
        titleContainer.className = 'window-title';
        parent.appendChild(titleContainer);
        return titleContainer;
    }

    private createWindowControls(parent: HTMLElement): HTMLElement {
        const controlsContainer = document.createElement('div');
        controlsContainer.className = 'window-controls-container';
        parent.appendChild(controlsContainer);

        // Minimize button
        const minimizeButton = this.createWindowControl('minimize', '🗕', () => {
            this.browserService.minimizeWindow();
        });
        controlsContainer.appendChild(minimizeButton);

        // Maximize/Restore button
        const maximizeButton = this.createWindowControl('maximize', '🗖', () => {
            this.browserService.toggleMaximizeWindow();
        });
        controlsContainer.appendChild(maximizeButton);

        // Close button
        const closeButton = this.createWindowControl('close', '🗙', () => {
            this.browserService.closeWindow();
        });
        controlsContainer.appendChild(closeButton);

        return controlsContainer;
    }

    // Update the window title
    updateTitle(title?: string): void {
        if (title) {
            this._currentTitle = title;
        } else {
            // Generate title from workspace and active editor
            const workspace = this.contextService.getWorkspace();
            const activeEditor = this.editorService.activeEditor;

            let titleParts: string[] = [];

            if (activeEditor) {
                titleParts.push(activeEditor.getName());
            }

            if (workspace.name) {
                titleParts.push(workspace.name);
            }

            titleParts.push('Visual Studio Code');

            this._currentTitle = titleParts.join(' - ');
        }

        // Update DOM
        if (this._titleContainer) {
            this._titleContainer.textContent = this._currentTitle;
        }

        // Update browser title
        document.title = this._currentTitle;
    }
}
```

### **Menu Bar System**
```typescript
export class MenuBar {
    private _menus = new Map<string, IMenu>();
    private _container: HTMLElement;

    constructor(
        container: HTMLElement,
        @IMenuService private readonly menuService: IMenuService,
        @ICommandService private readonly commandService: ICommandService
    ) {
        this._container = container;
    }

    addMenu(label: string, menu: IMenu): void {
        this._menus.set(label, menu);

        const menuElement = document.createElement('div');
        menuElement.className = 'menu-item';
        menuElement.textContent = label;

        // Handle menu click
        menuElement.addEventListener('click', () => {
            this.showMenu(label, menuElement);
        });

        this._container.appendChild(menuElement);
    }

    private showMenu(label: string, anchor: HTMLElement): void {
        const menu = this._menus.get(label);
        if (menu) {
            this.contextMenuService.showContextMenu({
                getAnchor: () => anchor,
                getActions: () => menu.getActions()
            });
        }
    }
}

// Example menu creation
private createFileMenu(): IMenu {
    return {
        getActions: () => [
            new Action('workbench.action.files.newUntitledFile', 'New File', undefined, true, () => {
                return this.commandService.executeCommand('workbench.action.files.newUntitledFile');
            }),
            new Action('workbench.action.files.openFile', 'Open File...', undefined, true, () => {
                return this.commandService.executeCommand('workbench.action.files.openFile');
            }),
            new Separator(),
            new Action('workbench.action.files.save', 'Save', undefined, true, () => {
                return this.commandService.executeCommand('workbench.action.files.save');
            }),
            new Action('workbench.action.files.saveAs', 'Save As...', undefined, true, () => {
                return this.commandService.executeCommand('workbench.action.files.saveAs');
            })
        ]
    };
}
```

## 🎮 Activity Bar (`src/vs/workbench/browser/parts/activitybar/`)

### **The Navigation Hub**
```typescript
export class ActivityBar {
    private _activities = new Map<string, IActivityBarItem>();
    private _container: HTMLElement;
    private _activeActivity: string | undefined;

    constructor(
        container: HTMLElement,
        @IInstantiationService private readonly instantiationService: IInstantiationService,
        @IActivityBarService private readonly activityBarService: IActivityBarService,
        @IThemeService private readonly themeService: IThemeService
    ) {
        this._container = container;
        this.createActivityBar();
    }

    private createActivityBar(): void {
        // Create main activities container
        const activitiesContainer = document.createElement('div');
        activitiesContainer.className = 'activities-container';
        this._container.appendChild(activitiesContainer);

        // Add built-in activities
        this.addBuiltInActivities(activitiesContainer);

        // Create global actions container (settings, accounts, etc.)
        const globalActionsContainer = document.createElement('div');
        globalActionsContainer.className = 'global-actions-container';
        this._container.appendChild(globalActionsContainer);

        this.addGlobalActions(globalActionsContainer);
    }

    private addBuiltInActivities(container: HTMLElement): void {
        const builtInActivities = [
            {
                id: 'workbench.view.explorer',
                name: 'Explorer',
                iconClass: 'codicon-files',
                keybinding: 'Ctrl+Shift+E',
                order: 1
            },
            {
                id: 'workbench.view.search',
                name: 'Search',
                iconClass: 'codicon-search',
                keybinding: 'Ctrl+Shift+F',
                order: 2
            },
            {
                id: 'workbench.view.scm',
                name: 'Source Control',
                iconClass: 'codicon-source-control',
                keybinding: 'Ctrl+Shift+G',
                order: 3
            },
            {
                id: 'workbench.view.debug',
                name: 'Run and Debug',
                iconClass: 'codicon-debug-alt',
                keybinding: 'Ctrl+Shift+D',
                order: 4
            },
            {
                id: 'workbench.view.extensions',
                name: 'Extensions',
                iconClass: 'codicon-extensions',
                keybinding: 'Ctrl+Shift+X',
                order: 5
            }
        ];

        for (const activity of builtInActivities) {
            this.addActivity(activity, container);
        }
    }

    addActivity(activity: IActivity, container?: HTMLElement): void {
        const activityContainer = container || this._container.querySelector('.activities-container');

        const activityElement = document.createElement('div');
        activityElement.className = 'activity-bar-item';
        activityElement.setAttribute('data-activity-id', activity.id);
        activityElement.setAttribute('title', `${activity.name} (${activity.keybinding})`);

        // Create icon
        const iconElement = document.createElement('div');
        iconElement.className = `activity-icon ${activity.iconClass}`;
        activityElement.appendChild(iconElement);

        // Create badge container (for notifications)
        const badgeContainer = document.createElement('div');
        badgeContainer.className = 'activity-badge';
        activityElement.appendChild(badgeContainer);

        // Handle click
        activityElement.addEventListener('click', () => {
            this.setActiveActivity(activity.id);
        });

        // Store activity item
        const activityItem: IActivityBarItem = {
            element: activityElement,
            activity: activity,
            badge: badgeContainer
        };

        this._activities.set(activity.id, activityItem);

        // Insert in correct order
        this.insertActivityInOrder(activityContainer, activityElement, activity.order);
    }

    setActiveActivity(activityId: string): void {
        // Remove active class from previous activity
        if (this._activeActivity) {
            const previousItem = this._activities.get(this._activeActivity);
            if (previousItem) {
                previousItem.element.classList.remove('active');
            }
        }

        // Add active class to new activity
        const newItem = this._activities.get(activityId);
        if (newItem) {
            newItem.element.classList.add('active');
            this._activeActivity = activityId;

            // Notify activity bar service
            this.activityBarService.showActivity(activityId);
        }
    }

    // Update activity badge (for notifications)
    updateActivityBadge(activityId: string, badge: IBadge | undefined): void {
        const item = this._activities.get(activityId);
        if (item && item.badge) {
            if (badge) {
                item.badge.textContent = badge.value.toString();
                item.badge.className = `activity-badge ${badge.type}`;
                item.badge.style.display = 'block';
            } else {
                item.badge.style.display = 'none';
            }
        }
    }
}
```

### **Activity Badge System**
```typescript
export interface IBadge {
    value: number | string;    // Badge content
    type: BadgeType;          // Badge style
    tooltip?: string;         // Hover tooltip
}

export enum BadgeType {
    Info = 'info',           // Blue badge
    Warning = 'warning',     // Yellow badge
    Error = 'error',         // Red badge
    Success = 'success'      // Green badge
}

// Example: Update SCM badge with number of changes
this.activityBar.updateActivityBadge('workbench.view.scm', {
    value: 5,
    type: BadgeType.Info,
    tooltip: '5 changes'
});
```

## 📁 Sidebar (`src/vs/workbench/browser/parts/sidebar/`)

### **The Primary Side Panel**
```typescript
export class SidebarPart extends Part {
    private _viewletContainer: HTMLElement;
    private _titleContainer: HTMLElement;
    private _activeViewlet: IViewlet | undefined;
    private _viewlets = new Map<string, IViewlet>();

    createContentArea(parent: HTMLElement): HTMLElement {
        this.element = parent;

        // Create title area
        this._titleContainer = this.createTitleArea(parent);

        // Create viewlet container
        this._viewletContainer = this.createViewletContainer(parent);

        return parent;
    }

    private createTitleArea(parent: HTMLElement): HTMLElement {
        const titleContainer = document.createElement('div');
        titleContainer.className = 'sidebar-title';
        parent.appendChild(titleContainer);

        // Title text
        const titleText = document.createElement('h3');
        titleText.className = 'title-text';
        titleContainer.appendChild(titleText);

        // Title actions (toolbar buttons)
        const titleActions = document.createElement('div');
        titleActions.className = 'title-actions';
        titleContainer.appendChild(titleActions);

        return titleContainer;
    }

    private createViewletContainer(parent: HTMLElement): HTMLElement {
        const container = document.createElement('div');
        container.className = 'viewlet-container';
        parent.appendChild(container);
        return container;
    }

    // Open a specific viewlet
    async openViewlet(id: string, focus?: boolean): Promise<IViewlet | undefined> {
        // Hide current viewlet
        if (this._activeViewlet && this._activeViewlet.getId() !== id) {
            this._activeViewlet.setVisible(false);
        }

        // Get or create viewlet
        let viewlet = this._viewlets.get(id);
        if (!viewlet) {
            viewlet = await this.createViewlet(id);
            if (viewlet) {
                this._viewlets.set(id, viewlet);
            }
        }

        if (viewlet) {
            // Show viewlet
            await viewlet.create(this._viewletContainer);
            viewlet.setVisible(true);

            if (focus) {
                viewlet.focus();
            }

            this._activeViewlet = viewlet;
            this.updateTitle(viewlet.getTitle());
            this.updateTitleActions(viewlet.getActions());
        }

        return viewlet;
    }

    private updateTitle(title: string): void {
        const titleText = this._titleContainer.querySelector('.title-text');
        if (titleText) {
            titleText.textContent = title;
        }
    }

    private updateTitleActions(actions: IAction[]): void {
        const titleActions = this._titleContainer.querySelector('.title-actions');
        if (titleActions) {
            // Clear existing actions
            titleActions.innerHTML = '';

            // Add new actions
            for (const action of actions) {
                const actionButton = document.createElement('button');
                actionButton.className = 'action-button';
                actionButton.title = action.tooltip || action.label;
                actionButton.innerHTML = action.icon || action.label;

                actionButton.addEventListener('click', () => {
                    action.run();
                });

                titleActions.appendChild(actionButton);
            }
        }
    }
}
```

### **Explorer Viewlet Example**
```typescript
export class ExplorerViewlet extends Viewlet {
    public static readonly ID = 'workbench.view.explorer';

    private _fileTree: FileTree;
    private _openEditorsView: OpenEditorsView;

    constructor(
        @IInstantiationService private readonly instantiationService: IInstantiationService,
        @IWorkspaceContextService private readonly contextService: IWorkspaceContextService,
        @IFileService private readonly fileService: IFileService,
        @IEditorService private readonly editorService: IEditorService
    ) {
        super(ExplorerViewlet.ID, 'Explorer');
    }

    async create(parent: HTMLElement): Promise<void> {
        this.element = parent;

        // Create open editors view
        this._openEditorsView = this.instantiationService.createInstance(OpenEditorsView);
        await this._openEditorsView.create(parent);

        // Create file tree
        this._fileTree = this.instantiationService.createInstance(FileTree);
        await this._fileTree.create(parent);

        // Load workspace files
        await this.loadWorkspaceFiles();
    }

    private async loadWorkspaceFiles(): Promise<void> {
        const workspace = this.contextService.getWorkspace();

        for (const folder of workspace.folders) {
            const files = await this.fileService.resolve(folder.uri);
            this._fileTree.setInput(files);
        }
    }

    getActions(): IAction[] {
        return [
            new Action('explorer.newFile', 'New File', 'codicon-new-file', true, () => {
                return this.commandService.executeCommand('explorer.newFile');
            }),
            new Action('explorer.newFolder', 'New Folder', 'codicon-new-folder', true, () => {
                return this.commandService.executeCommand('explorer.newFolder');
            }),
            new Action('explorer.refresh', 'Refresh', 'codicon-refresh', true, () => {
                return this.loadWorkspaceFiles();
            })
        ];
    }

    focus(): void {
        if (this._fileTree) {
            this._fileTree.focus();
        }
    }
}
```

## 📝 Editor Area (`src/vs/workbench/browser/parts/editor/`)

### **The Text Editing Space**
```typescript
export class EditorPart extends Part implements IEditorGroupsService {
    private _groups = new Map<GroupIdentifier, EditorGroup>();
    private _activeGroup: EditorGroup;
    private _container: HTMLElement;
    private _dimension: Dimension;

    createContentArea(parent: HTMLElement): HTMLElement {
        this._container = parent;

        // Create initial editor group
        this._activeGroup = this.createEditorGroup();
        this._groups.set(this._activeGroup.id, this._activeGroup);

        // Create group container
        this._activeGroup.create(this._container);

        return parent;
    }

    // Create a new editor group
    createEditorGroup(direction?: GroupDirection, referenceGroup?: EditorGroup): EditorGroup {
        const group = this.instantiationService.createInstance(
            EditorGroup,
            this.generateGroupId()
        );

        if (direction && referenceGroup) {
            this.layoutGroups(group, direction, referenceGroup);
        }

        this._groups.set(group.id, group);

        return group;
    }

    // Split editor group
    splitGroup(group: EditorGroup, direction: GroupDirection): EditorGroup {
        const newGroup = this.createEditorGroup(direction, group);

        // Update layout
        this.layoutGroups();

        return newGroup;
    }

    // Layout multiple groups
    private layoutGroups(): void {
        const groups = Array.from(this._groups.values());

        if (groups.length === 1) {
            // Single group - full width
            groups[0].layout(this._dimension);
        } else if (groups.length === 2) {
            // Two groups - split horizontally
            const halfWidth = Math.floor(this._dimension.width / 2);

            groups[0].layout(new Dimension(halfWidth, this._dimension.height));
            groups[1].layout(new Dimension(halfWidth, this._dimension.height));

            // Position second group
            groups[1].element.style.left = `${halfWidth}px`;
        } else {
            // Multiple groups - grid layout
            this.layoutGroupsInGrid(groups);
        }
    }

    // Open editor in specific group
    async openEditor(editor: IEditorInput, group?: EditorGroup): Promise<IEditor | undefined> {
        const targetGroup = group || this._activeGroup;
        return targetGroup.openEditor(editor);
    }

    // Close editor
    async closeEditor(editor: IEditorInput, group?: EditorGroup): Promise<void> {
        const targetGroup = group || this._activeGroup;
        return targetGroup.closeEditor(editor);
    }

    // Get all open editors
    get editors(): IEditorInput[] {
        const allEditors: IEditorInput[] = [];

        for (const group of this._groups.values()) {
            allEditors.push(...group.editors);
        }

        return allEditors;
    }
}
```

### **Editor Group Implementation**
```typescript
export class EditorGroup implements IEditorGroup {
    private _editors: IEditorInput[] = [];
    private _activeEditor: IEditorInput | undefined;
    private _tabsContainer: HTMLElement;
    private _editorContainer: HTMLElement;
    private _tabs = new Map<IEditorInput, HTMLElement>();

    constructor(
        public readonly id: GroupIdentifier,
        @IInstantiationService private readonly instantiationService: IInstantiationService,
        @IEditorService private readonly editorService: IEditorService,
        @IContextMenuService private readonly contextMenuService: IContextMenuService
    ) {}

    create(parent: HTMLElement): void {
        this.element = parent;

        // Create tabs container
        this._tabsContainer = document.createElement('div');
        this._tabsContainer.className = 'tabs-container';
        parent.appendChild(this._tabsContainer);

        // Create editor container
        this._editorContainer = document.createElement('div');
        this._editorContainer.className = 'editor-container';
        parent.appendChild(this._editorContainer);
    }

    // Open editor in this group
    async openEditor(editor: IEditorInput, options?: IEditorOptions): Promise<IEditor | undefined> {
        // Add to editors list if not present
        if (!this._editors.includes(editor)) {
            this._editors.push(editor);
            this.createTab(editor);
        }

        // Set as active
        this.setActiveEditor(editor);

        // Create editor instance
        const editorInstance = await this.editorService.createEditor(editor);
        if (editorInstance) {
            await editorInstance.create(this._editorContainer);
            editorInstance.setVisible(true);

            if (options?.focus !== false) {
                editorInstance.focus();
            }
        }

        return editorInstance;
    }

    private createTab(editor: IEditorInput): void {
        const tab = document.createElement('div');
        tab.className = 'tab';

        // Tab icon
        const icon = document.createElement('div');
        icon.className = `tab-icon ${this.getEditorIcon(editor)}`;
        tab.appendChild(icon);

        // Tab label
        const label = document.createElement('span');
        label.className = 'tab-label';
        label.textContent = editor.getName();
        tab.appendChild(label);

        // Close button
        const closeButton = document.createElement('div');
        closeButton.className = 'tab-close codicon-close';
        closeButton.addEventListener('click', (e) => {
            e.stopPropagation();
            this.closeEditor(editor);
        });
        tab.appendChild(closeButton);

        // Tab click handler
        tab.addEventListener('click', () => {
            this.openEditor(editor);
        });

        // Context menu
        tab.addEventListener('contextmenu', (e) => {
            this.showTabContextMenu(e, editor);
        });

        this._tabs.set(editor, tab);
        this._tabsContainer.appendChild(tab);
    }

    private setActiveEditor(editor: IEditorInput): void {
        // Remove active class from previous tab
        if (this._activeEditor) {
            const previousTab = this._tabs.get(this._activeEditor);
            if (previousTab) {
                previousTab.classList.remove('active');
            }
        }

        // Add active class to new tab
        const newTab = this._tabs.get(editor);
        if (newTab) {
            newTab.classList.add('active');
        }

        this._activeEditor = editor;
    }

    private showTabContextMenu(event: MouseEvent, editor: IEditorInput): void {
        const actions = [
            new Action('workbench.action.closeActiveEditor', 'Close', undefined, true, () => {
                return this.closeEditor(editor);
            }),
            new Action('workbench.action.closeOtherEditors', 'Close Others', undefined, true, () => {
                return this.closeOtherEditors(editor);
            }),
            new Action('workbench.action.closeEditorsToTheRight', 'Close to the Right', undefined, true, () => {
                return this.closeEditorsToTheRight(editor);
            }),
            new Separator(),
            new Action('workbench.action.splitEditor', 'Split Editor', undefined, true, () => {
                return this.splitEditor(editor);
            })
        ];

        this.contextMenuService.showContextMenu({
            getAnchor: () => ({ x: event.clientX, y: event.clientY }),
            getActions: () => actions
        });
    }
}
```

## 📊 Panel (`src/vs/workbench/browser/parts/panel/`)

### **The Bottom Utilities Area**
```typescript
export class PanelPart extends Part implements IPanelService {
    private _panelContainer: HTMLElement;
    private _panelTabs: HTMLElement;
    private _activePanel: IPanel | undefined;
    private _panels = new Map<string, IPanel>();

    createContentArea(parent: HTMLElement): HTMLElement {
        this.element = parent;

        // Create panel tabs
        this._panelTabs = this.createPanelTabs(parent);

        // Create panel container
        this._panelContainer = this.createPanelContainer(parent);

        // Add built-in panels
        this.registerBuiltInPanels();

        return parent;
    }

    private createPanelTabs(parent: HTMLElement): HTMLElement {
        const tabsContainer = document.createElement('div');
        tabsContainer.className = 'panel-tabs';
        parent.appendChild(tabsContainer);
        return tabsContainer;
    }

    private createPanelContainer(parent: HTMLElement): HTMLElement {
        const container = document.createElement('div');
        container.className = 'panel-container';
        parent.appendChild(container);
        return container;
    }

    private registerBuiltInPanels(): void {
        const builtInPanels = [
            {
                id: 'workbench.panel.terminal',
                name: 'Terminal',
                iconClass: 'codicon-terminal',
                order: 1
            },
            {
                id: 'workbench.panel.problems',
                name: 'Problems',
                iconClass: 'codicon-warning',
                order: 2
            },
            {
                id: 'workbench.panel.output',
                name: 'Output',
                iconClass: 'codicon-output',
                order: 3
            },
            {
                id: 'workbench.panel.debug.console',
                name: 'Debug Console',
                iconClass: 'codicon-debug-console',
                order: 4
            }
        ];

        for (const panelInfo of builtInPanels) {
            this.addPanelTab(panelInfo);
        }
    }

    private addPanelTab(panelInfo: IPanelInfo): void {
        const tab = document.createElement('div');
        tab.className = 'panel-tab';
        tab.setAttribute('data-panel-id', panelInfo.id);

        // Tab icon
        const icon = document.createElement('div');
        icon.className = `panel-icon ${panelInfo.iconClass}`;
        tab.appendChild(icon);

        // Tab label
        const label = document.createElement('span');
        label.className = 'panel-label';
        label.textContent = panelInfo.name;
        tab.appendChild(label);

        // Tab click handler
        tab.addEventListener('click', () => {
            this.openPanel(panelInfo.id);
        });

        this._panelTabs.appendChild(tab);
    }

    // Open a specific panel
    async openPanel(id: string, focus?: boolean): Promise<IPanel | undefined> {
        // Hide current panel
        if (this._activePanel && this._activePanel.getId() !== id) {
            this._activePanel.setVisible(false);
        }

        // Get or create panel
        let panel = this._panels.get(id);
        if (!panel) {
            panel = await this.createPanel(id);
            if (panel) {
                this._panels.set(id, panel);
            }
        }

        if (panel) {
            // Show panel
            await panel.create(this._panelContainer);
            panel.setVisible(true);

            if (focus) {
                panel.focus();
            }

            this._activePanel = panel;
            this.updateActiveTab(id);
        }

        return panel;
    }

    private updateActiveTab(panelId: string): void {
        // Remove active class from all tabs
        const tabs = this._panelTabs.querySelectorAll('.panel-tab');
        tabs.forEach(tab => tab.classList.remove('active'));

        // Add active class to current tab
        const activeTab = this._panelTabs.querySelector(`[data-panel-id="${panelId}"]`);
        if (activeTab) {
            activeTab.classList.add('active');
        }
    }
}
```

### **Terminal Panel Example**
```typescript
export class TerminalPanel extends Panel {
    public static readonly ID = 'workbench.panel.terminal';

    private _terminalContainer: HTMLElement;
    private _terminals = new Map<number, ITerminalInstance>();
    private _activeTerminal: ITerminalInstance | undefined;

    constructor(
        @ITerminalService private readonly terminalService: ITerminalService,
        @IInstantiationService private readonly instantiationService: IInstantiationService
    ) {
        super(TerminalPanel.ID, 'Terminal');
    }

    async create(parent: HTMLElement): Promise<void> {
        this.element = parent;

        // Create terminal container
        this._terminalContainer = document.createElement('div');
        this._terminalContainer.className = 'terminal-container';
        parent.appendChild(this._terminalContainer);

        // Create initial terminal
        await this.createTerminal();
    }

    private async createTerminal(): Promise<ITerminalInstance> {
        const terminal = await this.terminalService.createTerminal({
            name: `Terminal ${this._terminals.size + 1}`
        });

        this._terminals.set(terminal.id, terminal);

        // Create terminal in DOM
        await terminal.create(this._terminalContainer);

        // Set as active
        this.setActiveTerminal(terminal);

        return terminal;
    }

    private setActiveTerminal(terminal: ITerminalInstance): void {
        // Hide previous terminal
        if (this._activeTerminal) {
            this._activeTerminal.setVisible(false);
        }

        // Show new terminal
        terminal.setVisible(true);
        this._activeTerminal = terminal;
    }

    focus(): void {
        if (this._activeTerminal) {
            this._activeTerminal.focus();
        }
    }

    getActions(): IAction[] {
        return [
            new Action('terminal.new', 'New Terminal', 'codicon-plus', true, () => {
                return this.createTerminal();
            }),
            new Action('terminal.split', 'Split Terminal', 'codicon-split-horizontal', true, () => {
                return this.splitTerminal();
            }),
            new Action('terminal.kill', 'Kill Terminal', 'codicon-trash', true, () => {
                return this.killActiveTerminal();
            })
        ];
    }
}
```

## 📋 Status Bar (`src/vs/workbench/browser/parts/statusbar/`)

### **The Information Display**
```typescript
export class StatusbarPart extends Part implements IStatusbarService {
    private _leftItemsContainer: HTMLElement;
    private _rightItemsContainer: HTMLElement;
    private _items = new Map<string, StatusbarItem>();

    createContentArea(parent: HTMLElement): HTMLElement {
        this.element = parent;

        // Create left items container
        this._leftItemsContainer = document.createElement('div');
        this._leftItemsContainer.className = 'statusbar-left';
        parent.appendChild(this._leftItemsContainer);

        // Create right items container
        this._rightItemsContainer = document.createElement('div');
        this._rightItemsContainer.className = 'statusbar-right';
        parent.appendChild(this._rightItemsContainer);

        // Add built-in status bar items
        this.addBuiltInItems();

        return parent;
    }

    private addBuiltInItems(): void {
        // Language mode
        this.addEntry({
            text: 'JavaScript',
            tooltip: 'Select Language Mode',
            command: 'workbench.action.editor.changeLanguageMode'
        }, 'editor.languageMode', StatusbarAlignment.RIGHT, 100);

        // Line/Column position
        this.addEntry({
            text: 'Ln 1, Col 1',
            tooltip: 'Go to Line/Column',
            command: 'workbench.action.gotoLine'
        }, 'editor.selection', StatusbarAlignment.RIGHT, 90);

        // Encoding
        this.addEntry({
            text: 'UTF-8',
            tooltip: 'Select Encoding',
            command: 'workbench.action.editor.changeEncoding'
        }, 'editor.encoding', StatusbarAlignment.RIGHT, 80);

        // End of Line
        this.addEntry({
            text: 'LF',
            tooltip: 'Select End of Line Sequence',
            command: 'workbench.action.editor.changeEOL'
        }, 'editor.eol', StatusbarAlignment.RIGHT, 70);
    }

    // Add status bar entry
    addEntry(entry: IStatusbarEntry, id: string, alignment: StatusbarAlignment, priority?: number): IStatusbarEntryAccessor {
        const item = new StatusbarItem(entry, id, alignment, priority);
        this._items.set(id, item);

        // Add to appropriate container
        const container = alignment === StatusbarAlignment.LEFT
            ? this._leftItemsContainer
            : this._rightItemsContainer;

        this.insertItemInOrder(container, item.element, priority);

        return {
            update: (entry: IStatusbarEntry) => item.update(entry),
            dispose: () => this.removeEntry(id)
        };
    }

    // Update existing entry
    updateEntry(id: string, entry: IStatusbarEntry): void {
        const item = this._items.get(id);
        if (item) {
            item.update(entry);
        }
    }

    // Remove entry
    removeEntry(id: string): void {
        const item = this._items.get(id);
        if (item) {
            item.dispose();
            this._items.delete(id);
        }
    }
}

class StatusbarItem {
    public readonly element: HTMLElement;

    constructor(
        private _entry: IStatusbarEntry,
        private _id: string,
        private _alignment: StatusbarAlignment,
        private _priority?: number
    ) {
        this.element = this.createElement();
        this.update(_entry);
    }

    private createElement(): HTMLElement {
        const element = document.createElement('div');
        element.className = 'statusbar-item';
        element.setAttribute('data-item-id', this._id);

        // Handle click
        element.addEventListener('click', () => {
            if (this._entry.command) {
                this.commandService.executeCommand(this._entry.command, ...(this._entry.arguments || []));
            }
        });

        return element;
    }

    update(entry: IStatusbarEntry): void {
        this._entry = entry;

        // Update text
        this.element.textContent = entry.text;

        // Update tooltip
        this.element.title = entry.tooltip || '';

        // Update colors
        if (entry.color) {
            this.element.style.color = entry.color.toString();
        }

        if (entry.backgroundColor) {
            this.element.style.backgroundColor = entry.backgroundColor.toString();
        }

        // Update accessibility
        if (entry.ariaLabel) {
            this.element.setAttribute('aria-label', entry.ariaLabel);
        }

        if (entry.role) {
            this.element.setAttribute('role', entry.role);
        }
    }

    dispose(): void {
        if (this.element.parentNode) {
            this.element.parentNode.removeChild(this.element);
        }
    }
}
```

## 🔧 Auxiliary Bar (`src/vs/workbench/browser/parts/auxiliarybar/`)

### **The Secondary Side Panel**
```typescript
export class AuxiliaryBarPart extends Part implements IAuxiliaryBarService {
    public static readonly ID = 'workbench.parts.auxiliarybar';

    private _viewContainer: HTMLElement;
    private _activeView: IView | undefined;
    private _views = new Map<string, IView>();

    constructor(
        @IInstantiationService private readonly instantiationService: IInstantiationService,
        @ILayoutService layoutService: IWorkbenchLayoutService,
        @IThemeService themeService: IThemeService,
        @IStorageService storageService: IStorageService
    ) {
        super(AuxiliaryBarPart.ID, { hasTitle: true }, themeService, storageService, layoutService);
    }

    createContentArea(parent: HTMLElement): HTMLElement {
        this.element = parent;

        // Create view container
        this._viewContainer = document.createElement('div');
        this._viewContainer.className = 'auxiliary-view-container';
        parent.appendChild(this._viewContainer);

        return parent;
    }

    // Show a view in the auxiliary bar
    async showView(id: string): Promise<IView | undefined> {
        // Hide current view
        if (this._activeView && this._activeView.getId() !== id) {
            this._activeView.setVisible(false);
        }

        // Get or create view
        let view = this._views.get(id);
        if (!view) {
            view = await this.createView(id);
            if (view) {
                this._views.set(id, view);
            }
        }

        if (view) {
            // Show view
            await view.create(this._viewContainer);
            view.setVisible(true);

            this._activeView = view;
        }

        return view;
    }

    // Hide the auxiliary bar
    hide(): void {
        if (this._activeView) {
            this._activeView.setVisible(false);
            this._activeView = undefined;
        }

        this.setVisible(false);
    }
}
```

## 🎯 Parts Coordination Example

### **How Parts Work Together**
```typescript
// Example: Opening a file and updating all parts
export class FileOpeningCoordination {
    async openFile(uri: URI): Promise<void> {
        // 1. Title Bar - Update with file name
        this.titleService.updateTitle(path.basename(uri.fsPath));

        // 2. Activity Bar - Ensure Explorer is active
        this.activityBarService.showActivity('workbench.view.explorer');

        // 3. Sidebar - Show Explorer viewlet
        const explorer = await this.sidebarService.openViewlet('workbench.view.explorer');

        // 4. Editor Area - Open file in editor
        const editorInput = this.editorService.createEditorInput(uri);
        const editor = await this.editorService.openEditor(editorInput);

        // 5. Status Bar - Update with file info
        this.statusBarService.updateEntry('editor.languageMode', {
            text: this.getLanguageMode(uri),
            tooltip: 'Select Language Mode'
        });

        this.statusBarService.updateEntry('editor.encoding', {
            text: await this.getFileEncoding(uri),
            tooltip: 'Select Encoding'
        });

        // 6. Panel - Show problems if any
        const problems = await this.problemsService.getProblems(uri);
        if (problems.length > 0) {
            await this.panelService.openPanel('workbench.panel.problems');
        }
    }
}
```

## 📚 Next Steps

Now that you understand all Workbench Parts:

1. **[18-workbench-services.md](./18-workbench-services.md)** - Learn about workbench services
2. **[19-layout-system.md](./19-layout-system.md)** - Understand the layout system
3. **[20-theme-system.md](./20-theme-system.md)** - Master the theming system

## 🎯 Key Takeaways

Workbench Parts are the building blocks of VS Code's UI:

- **Specialized Components**: Each part has a specific purpose and responsibility
- **Coordinated System**: Parts work together to create a cohesive experience
- **Extensible Architecture**: New parts can be added and existing ones extended
- **State Management**: Parts save and restore their state across sessions
- **Event-Driven**: Parts communicate through events and services

Understanding workbench parts helps you:
- **Debug UI issues** by knowing which part handles what
- **Build better extensions** that integrate properly with the UI
- **Customize VS Code** by understanding how the interface works
- **Contribute to VS Code** by adding new UI components

Each part is like a specialized room in VS Code's office building, and together they create the productive workspace that millions of developers love! 🧩✨
