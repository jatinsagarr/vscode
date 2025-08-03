# 📐 VS Code Layout System - UI Positioning & Sizing Engine

## 🎯 Overview

VS Code's Layout System is the sophisticated engine that manages how all UI components are positioned, sized, and arranged on screen. It handles everything from window resizing to panel splitting, ensuring that every pixel is used efficiently and the interface remains responsive. Think of it as the master architect that designs and continuously redesigns the workspace layout.

## 🧠 What is the Layout System?

### **Simple Analogy: The Interior Designer**
Imagine VS Code as a modern office building with a smart interior designer:
- **Layout Service** = The interior designer who arranges all the furniture
- **Grid System** = The floor plan that defines where things can go
- **Responsive Design** = Automatically adjusting when the office space changes
- **Split Views** = Creating room dividers to organize different work areas
- **Flexible Panels** = Movable walls that can expand or contract as needed

The layout system ensures your workspace is always organized and efficient, no matter how you resize or rearrange it!

## 🏗️ Layout Architecture

### **The Complete Layout System**
```
VS Code Layout System
├── 🎯 Layout Service (Central Coordinator)
│   ├── Dimension Calculation (Size management)
│   ├── Position Management (Placement logic)
│   └── State Persistence (Layout memory)
├── 📐 Grid System (Spatial Organization)
│   ├── Main Grid (Primary layout areas)
│   ├── Editor Grid (Editor group arrangement)
│   └── Panel Grid (Panel organization)
├── 🔄 Responsive Engine (Dynamic Adjustment)
│   ├── Window Resize Handler (Size changes)
│   ├── Content Reflow (Content adjustment)
│   └── Minimum Size Enforcement (Usability limits)
├── 🎛️ Splitter System (User Control)
│   ├── Horizontal Splitters (Width adjustment)
│   ├── Vertical Splitters (Height adjustment)
│   └── Drag Handlers (Interactive resizing)
└── 💾 Layout Persistence (State Management)
    ├── Layout State Storage (Remember arrangements)
    ├── Workspace Layouts (Per-project layouts)
    └── Default Layouts (Fallback arrangements)
```

## 🎯 Layout Service Core (`src/vs/workbench/services/layout/`)

### **The Master Coordinator**
```typescript
export class WorkbenchLayoutService implements IWorkbenchLayoutService {
    private _dimension: Dimension;
    private _container: HTMLElement;
    private _parts = new Map<string, Part>();
    private _partDimensions = new Map<string, IPartLayoutInfo>();

    // Layout state
    private _sideBarHidden = false;
    private _panelHidden = false;
    private _activityBarHidden = false;
    private _statusBarHidden = false;
    private _auxiliaryBarHidden = true;

    // Panel positioning
    private _panelPosition: Position = Position.BOTTOM;
    private _sideBarPosition: Position = Position.LEFT;

    // Sizes
    private _sideBarWidth = 300;
    private _panelHeight = 300;
    private _auxiliaryBarWidth = 300;

    constructor(
        @IStorageService private readonly storageService: IStorageService,
        @IConfigurationService private readonly configurationService: IConfigurationService,
        @IThemeService private readonly themeService: IThemeService,
        @IContextKeyService private readonly contextKeyService: IContextKeyService
    ) {
        this.initializeContextKeys();
        this.restoreLayoutState();
        this.registerListeners();
    }

    // Initialize the layout with container
    initLayout(container: HTMLElement): void {
        this._container = container;

        // Set up container
        container.classList.add('monaco-workbench');

        // Create main layout structure
        this.createLayoutStructure(container);

        // Initial layout
        this.layout();
    }

    private createLayoutStructure(container: HTMLElement): void {
        // Create main areas
        const titleBarArea = document.createElement('div');
        titleBarArea.className = 'part titlebar';
        titleBarArea.id = 'workbench.parts.titlebar';
        container.appendChild(titleBarArea);

        const mainArea = document.createElement('div');
        mainArea.className = 'main-area';
        container.appendChild(mainArea);

        // Activity bar
        const activityBarArea = document.createElement('div');
        activityBarArea.className = 'part activitybar';
        activityBarArea.id = 'workbench.parts.activitybar';
        mainArea.appendChild(activityBarArea);

        // Sidebar
        const sideBarArea = document.createElement('div');
        sideBarArea.className = 'part sidebar';
        sideBarArea.id = 'workbench.parts.sidebar';
        mainArea.appendChild(sideBarArea);

        // Editor and panel container
        const editorPanelContainer = document.createElement('div');
        editorPanelContainer.className = 'editor-panel-container';
        mainArea.appendChild(editorPanelContainer);

        // Editor area
        const editorArea = document.createElement('div');
        editorArea.className = 'part editor';
        editorArea.id = 'workbench.parts.editor';
        editorPanelContainer.appendChild(editorArea);

        // Panel area
        const panelArea = document.createElement('div');
        panelArea.className = 'part panel';
        panelArea.id = 'workbench.parts.panel';
        editorPanelContainer.appendChild(panelArea);

        // Auxiliary bar
        const auxiliaryBarArea = document.createElement('div');
        auxiliaryBarArea.className = 'part auxiliarybar';
        auxiliaryBarArea.id = 'workbench.parts.auxiliarybar';
        mainArea.appendChild(auxiliaryBarArea);

        // Status bar
        const statusBarArea = document.createElement('div');
        statusBarArea.className = 'part statusbar';
        statusBarArea.id = 'workbench.parts.statusbar';
        container.appendChild(statusBarArea);

        // Create splitters
        this.createSplitters(mainArea);
    }

    private createSplitters(container: HTMLElement): void {
        // Sidebar splitter
        const sidebarSplitter = document.createElement('div');
        sidebarSplitter.className = 'split-view-view-separator sidebar-splitter';
        sidebarSplitter.addEventListener('mousedown', (e) => {
            this.startSidebarResize(e);
        });
        container.appendChild(sidebarSplitter);

        // Panel splitter
        const panelSplitter = document.createElement('div');
        panelSplitter.className = 'split-view-view-separator panel-splitter';
        panelSplitter.addEventListener('mousedown', (e) => {
            this.startPanelResize(e);
        });
        container.appendChild(panelSplitter);

        // Auxiliary bar splitter
        const auxiliarySplitter = document.createElement('div');
        auxiliarySplitter.className = 'split-view-view-separator auxiliary-splitter';
        auxiliarySplitter.addEventListener('mousedown', (e) => {
            this.startAuxiliaryResize(e);
        });
        container.appendChild(auxiliarySplitter);
    }

    // Main layout method
    layout(dimension?: Dimension): void {
        if (dimension) {
            this._dimension = dimension;
        }

        if (!this._dimension || !this._container) {
            return;
        }

        // Update container size
        this._container.style.width = `${this._dimension.width}px`;
        this._container.style.height = `${this._dimension.height}px`;

        // Calculate part dimensions
        const partDimensions = this.computePartDimensions();

        // Apply layout to each part
        this.layoutParts(partDimensions);

        // Update splitter positions
        this.updateSplitterPositions(partDimensions);

        // Fire layout event
        this._onDidLayout.fire(this._dimension);
    }

    private computePartDimensions(): Map<string, IPartLayoutInfo> {
        const dimensions = new Map<string, IPartLayoutInfo>();

        if (!this._dimension) {
            return dimensions;
        }

        const totalWidth = this._dimension.width;
        const totalHeight = this._dimension.height;

        // Constants
        const titleBarHeight = this.getTitleBarHeight();
        const statusBarHeight = this._statusBarHidden ? 0 : this.getStatusBarHeight();
        const activityBarWidth = this._activityBarHidden ? 0 : this.getActivityBarWidth();

        // Variable dimensions
        const sideBarWidth = this._sideBarHidden ? 0 : this._sideBarWidth;
        const auxiliaryBarWidth = this._auxiliaryBarHidden ? 0 : this._auxiliaryBarWidth;
        const panelHeight = this._panelHidden ? 0 : this._panelHeight;

        // Available space for main content
        const mainAreaHeight = totalHeight - titleBarHeight - statusBarHeight;
        const mainAreaWidth = totalWidth;

        // Title bar
        dimensions.set('workbench.parts.titlebar', {
            width: totalWidth,
            height: titleBarHeight,
            top: 0,
            left: 0,
            visible: true
        });

        // Status bar
        dimensions.set('workbench.parts.statusbar', {
            width: totalWidth,
            height: statusBarHeight,
            top: totalHeight - statusBarHeight,
            left: 0,
            visible: !this._statusBarHidden
        });

        // Activity bar
        let activityBarLeft = 0;
        if (this._sideBarPosition === Position.RIGHT) {
            activityBarLeft = totalWidth - activityBarWidth - auxiliaryBarWidth - sideBarWidth;
        }

        dimensions.set('workbench.parts.activitybar', {
            width: activityBarWidth,
            height: mainAreaHeight,
            top: titleBarHeight,
            left: activityBarLeft,
            visible: !this._activityBarHidden
        });

        // Sidebar
        let sideBarLeft = activityBarWidth;
        if (this._sideBarPosition === Position.RIGHT) {
            sideBarLeft = totalWidth - sideBarWidth - auxiliaryBarWidth;
        }

        dimensions.set('workbench.parts.sidebar', {
            width: sideBarWidth,
            height: mainAreaHeight,
            top: titleBarHeight,
            left: sideBarLeft,
            visible: !this._sideBarHidden
        });

        // Auxiliary bar
        let auxiliaryBarLeft = totalWidth - auxiliaryBarWidth;
        if (this._sideBarPosition === Position.RIGHT) {
            auxiliaryBarLeft = totalWidth - auxiliaryBarWidth;
        }

        dimensions.set('workbench.parts.auxiliarybar', {
            width: auxiliaryBarWidth,
            height: mainAreaHeight,
            top: titleBarHeight,
            left: auxiliaryBarLeft,
            visible: !this._auxiliaryBarHidden
        });

        // Editor and panel area
        const editorPanelLeft = this._sideBarPosition === Position.LEFT
            ? activityBarWidth + sideBarWidth
            : activityBarWidth;
        const editorPanelWidth = totalWidth - activityBarWidth - sideBarWidth - auxiliaryBarWidth;

        if (this._panelPosition === Position.BOTTOM) {
            // Panel at bottom
            dimensions.set('workbench.parts.editor', {
                width: editorPanelWidth,
                height: mainAreaHeight - panelHeight,
                top: titleBarHeight,
                left: editorPanelLeft,
                visible: true
            });

            dimensions.set('workbench.parts.panel', {
                width: editorPanelWidth,
                height: panelHeight,
                top: titleBarHeight + mainAreaHeight - panelHeight,
                left: editorPanelLeft,
                visible: !this._panelHidden
            });
        } else if (this._panelPosition === Position.RIGHT) {
            // Panel at right
            const panelWidth = Math.min(editorPanelWidth * 0.5, 400);

            dimensions.set('workbench.parts.editor', {
                width: editorPanelWidth - panelWidth,
                height: mainAreaHeight,
                top: titleBarHeight,
                left: editorPanelLeft,
                visible: true
            });

            dimensions.set('workbench.parts.panel', {
                width: panelWidth,
                height: mainAreaHeight,
                top: titleBarHeight,
                left: editorPanelLeft + editorPanelWidth - panelWidth,
                visible: !this._panelHidden
            });
        }

        return dimensions;
    }

    private layoutParts(dimensions: Map<string, IPartLayoutInfo>): void {
        for (const [partId, part] of this._parts) {
            const dimension = dimensions.get(partId);
            if (dimension) {
                // Update part visibility
                const element = this._container.querySelector(`#${partId}`) as HTMLElement;
                if (element) {
                    element.style.display = dimension.visible ? 'block' : 'none';

                    if (dimension.visible) {
                        element.style.width = `${dimension.width}px`;
                        element.style.height = `${dimension.height}px`;
                        element.style.top = `${dimension.top}px`;
                        element.style.left = `${dimension.left}px`;
                        element.style.position = 'absolute';
                    }
                }

                // Layout the part
                part.layout(dimension.width, dimension.height, dimension.top, dimension.left);

                // Store dimension info
                this._partDimensions.set(partId, dimension);
            }
        }
    }

    // Toggle sidebar visibility
    setSideBarHidden(hidden: boolean): void {
        if (this._sideBarHidden !== hidden) {
            this._sideBarHidden = hidden;
            this.updateContextKeys();
            this.layout();
            this.saveLayoutState();

            this._onDidChangeSideBarHidden.fire(hidden);
        }
    }

    // Toggle panel visibility
    setPanelHidden(hidden: boolean): void {
        if (this._panelHidden !== hidden) {
            this._panelHidden = hidden;
            this.updateContextKeys();
            this.layout();
            this.saveLayoutState();

            this._onDidChangePanelHidden.fire(hidden);
        }
    }

    // Set panel position
    setPanelPosition(position: Position): void {
        if (this._panelPosition !== position) {
            this._panelPosition = position;
            this.layout();
            this.saveLayoutState();

            this._onDidChangePanelPosition.fire(position);
        }
    }

    // Resize sidebar
    resizeSidebar(width: number): void {
        const minWidth = 170;
        const maxWidth = Math.floor(this._dimension.width * 0.5);

        this._sideBarWidth = Math.max(minWidth, Math.min(maxWidth, width));
        this.layout();
        this.saveLayoutState();
    }

    // Resize panel
    resizePanel(height: number): void {
        const minHeight = 100;
        const maxHeight = Math.floor(this._dimension.height * 0.7);

        this._panelHeight = Math.max(minHeight, Math.min(maxHeight, height));
        this.layout();
        this.saveLayoutState();
    }
}

// Layout information interface
interface IPartLayoutInfo {
    width: number;
    height: number;
    top: number;
    left: number;
    visible: boolean;
}

// Position enum
export enum Position {
    LEFT = 'left',
    RIGHT = 'right',
    TOP = 'top',
    BOTTOM = 'bottom'
}
```

## 📐 Grid System (`src/vs/base/browser/ui/grid/`)

### **The Spatial Organizer**
```typescript
export class Grid<T extends IView> implements IDisposable {
    private _root: GridNode<T>;
    private _views = new Map<T, GridNode<T>>();
    private _orientation: Orientation;

    constructor(view: T, options: IGridOptions = {}) {
        this._orientation = options.orientation || Orientation.VERTICAL;
        this._root = new GridLeafNode(view, options.proportionalLayout !== false);
        this._views.set(view, this._root);
    }

    // Add view to grid
    addView(newView: T, size: number | Sizing, referenceView: T, direction: Direction): void {
        const referenceNode = this._views.get(referenceView);
        if (!referenceNode) {
            throw new Error('Reference view not found');
        }

        const newNode = new GridLeafNode(newView, true);
        this._views.set(newView, newNode);

        // Determine split orientation
        const orientation = direction === Direction.Up || direction === Direction.Down
            ? Orientation.VERTICAL
            : Orientation.HORIZONTAL;

        // Create branch node
        const branchNode = new GridBranchNode(orientation, []);

        // Arrange nodes based on direction
        if (direction === Direction.Up || direction === Direction.Left) {
            branchNode.children = [newNode, referenceNode];
        } else {
            branchNode.children = [referenceNode, newNode];
        }

        // Replace reference node with branch
        this.replaceNode(referenceNode, branchNode);

        // Set sizes
        this.setViewSize(newView, size);
    }

    // Remove view from grid
    removeView(view: T): T {
        const node = this._views.get(view);
        if (!node) {
            throw new Error('View not found');
        }

        this._views.delete(view);

        // Handle removal based on node type
        if (node === this._root) {
            throw new Error('Cannot remove root view');
        }

        const parent = node.parent;
        if (parent instanceof GridBranchNode) {
            // Remove from parent's children
            const index = parent.children.indexOf(node);
            parent.children.splice(index, 1);

            // If parent has only one child left, replace parent with child
            if (parent.children.length === 1) {
                const remainingChild = parent.children[0];
                this.replaceNode(parent, remainingChild);
            }
        }

        return view;
    }

    // Layout the grid
    layout(width: number, height: number): void {
        this._root.layout(width, height, 0, 0);
    }

    // Set view size
    setViewSize(view: T, size: number | Sizing): void {
        const node = this._views.get(view);
        if (!node) {
            return;
        }

        if (typeof size === 'number') {
            node.size = size;
        } else {
            // Handle Sizing enum (Distribute, Split, etc.)
            this.applySizing(node, size);
        }

        this.layout(this._root.width, this._root.height);
    }

    // Get view size
    getViewSize(view: T): number {
        const node = this._views.get(view);
        return node ? node.size : 0;
    }

    private replaceNode(oldNode: GridNode<T>, newNode: GridNode<T>): void {
        if (oldNode === this._root) {
            this._root = newNode;
            newNode.parent = undefined;
        } else if (oldNode.parent instanceof GridBranchNode) {
            const index = oldNode.parent.children.indexOf(oldNode);
            oldNode.parent.children[index] = newNode;
            newNode.parent = oldNode.parent;
        }
    }
}

// Grid node base class
abstract class GridNode<T extends IView> {
    parent: GridBranchNode<T> | undefined;
    size: number = 0;
    width: number = 0;
    height: number = 0;
    top: number = 0;
    left: number = 0;

    abstract layout(width: number, height: number, top: number, left: number): void;
}

// Leaf node (contains a view)
class GridLeafNode<T extends IView> extends GridNode<T> {
    constructor(
        public readonly view: T,
        public readonly proportionalLayout: boolean
    ) {
        super();
    }

    layout(width: number, height: number, top: number, left: number): void {
        this.width = width;
        this.height = height;
        this.top = top;
        this.left = left;

        // Layout the view
        this.view.layout(width, height, top, left);
    }
}

// Branch node (contains child nodes)
class GridBranchNode<T extends IView> extends GridNode<T> {
    children: GridNode<T>[] = [];

    constructor(
        public readonly orientation: Orientation,
        children: GridNode<T>[]
    ) {
        super();
        this.children = children;

        // Set parent references
        for (const child of children) {
            child.parent = this;
        }
    }

    layout(width: number, height: number, top: number, left: number): void {
        this.width = width;
        this.height = height;
        this.top = top;
        this.left = left;

        if (this.children.length === 0) {
            return;
        }

        // Calculate child sizes
        const childSizes = this.calculateChildSizes(
            this.orientation === Orientation.HORIZONTAL ? width : height
        );

        // Layout children
        let currentOffset = 0;

        for (let i = 0; i < this.children.length; i++) {
            const child = this.children[i];
            const childSize = childSizes[i];

            if (this.orientation === Orientation.HORIZONTAL) {
                child.layout(childSize, height, top, left + currentOffset);
            } else {
                child.layout(width, childSize, top + currentOffset, left);
            }

            currentOffset += childSize;
        }
    }

    private calculateChildSizes(totalSize: number): number[] {
        const sizes: number[] = [];
        let remainingSize = totalSize;

        // First pass: fixed sizes
        for (const child of this.children) {
            if (child.size > 0) {
                sizes.push(child.size);
                remainingSize -= child.size;
            } else {
                sizes.push(0);
            }
        }

        // Second pass: distribute remaining space
        const flexChildren = sizes.filter(size => size === 0).length;
        if (flexChildren > 0) {
            const flexSize = Math.floor(remainingSize / flexChildren);

            for (let i = 0; i < sizes.length; i++) {
                if (sizes[i] === 0) {
                    sizes[i] = flexSize;
                }
            }
        }

        return sizes;
    }
}

// Grid enums
export enum Orientation {
    HORIZONTAL = 'horizontal',
    VERTICAL = 'vertical'
}

export enum Direction {
    Up = 'up',
    Down = 'down',
    Left = 'left',
    Right = 'right'
}

export enum Sizing {
    Distribute = 'distribute',
    Split = 'split',
    Invisible = 'invisible'
}
```

## 🔄 Responsive Engine

### **The Dynamic Adjuster**
```typescript
export class ResponsiveLayoutEngine {
    private _breakpoints = {
        mobile: 768,
        tablet: 1024,
        desktop: 1440
    };

    private _currentBreakpoint: string = 'desktop';
    private _layoutRules = new Map<string, ILayoutRule[]>();

    constructor(
        @ILayoutService private readonly layoutService: ILayoutService,
        @IConfigurationService private readonly configurationService: IConfigurationService
    ) {
        this.registerLayoutRules();
        this.registerListeners();
    }

    // Handle window resize
    onWindowResize(dimension: Dimension): void {
        const newBreakpoint = this.getBreakpoint(dimension.width);

        if (newBreakpoint !== this._currentBreakpoint) {
            this._currentBreakpoint = newBreakpoint;
            this.applyBreakpointRules(newBreakpoint);
        }

        // Apply responsive adjustments
        this.applyResponsiveAdjustments(dimension);
    }

    private getBreakpoint(width: number): string {
        if (width < this._breakpoints.mobile) {
            return 'mobile';
        } else if (width < this._breakpoints.tablet) {
            return 'tablet';
        } else if (width < this._breakpoints.desktop) {
            return 'tablet';
        } else {
            return 'desktop';
        }
    }

    private applyBreakpointRules(breakpoint: string): void {
        const rules = this._layoutRules.get(breakpoint);
        if (!rules) {
            return;
        }

        for (const rule of rules) {
            switch (rule.action) {
                case 'hide':
                    this.layoutService.setPanelHidden(true);
                    break;
                case 'show':
                    this.layoutService.setPanelHidden(false);
                    break;
                case 'collapse':
                    this.layoutService.setSideBarHidden(true);
                    break;
                case 'expand':
                    this.layoutService.setSideBarHidden(false);
                    break;
                case 'reposition':
                    if (rule.target === 'panel' && rule.position) {
                        this.layoutService.setPanelPosition(rule.position);
                    }
                    break;
            }
        }
    }

    private applyResponsiveAdjustments(dimension: Dimension): void {
        // Adjust sidebar width based on screen size
        const sidebarWidth = this.calculateResponsiveSidebarWidth(dimension.width);
        this.layoutService.resizeSidebar(sidebarWidth);

        // Adjust panel height based on screen size
        const panelHeight = this.calculateResponsivePanelHeight(dimension.height);
        this.layoutService.resizePanel(panelHeight);

        // Handle minimum sizes
        this.enforceMinimumSizes(dimension);
    }

    private calculateResponsiveSidebarWidth(screenWidth: number): number {
        if (screenWidth < this._breakpoints.mobile) {
            return 250; // Smaller sidebar on mobile
        } else if (screenWidth < this._breakpoints.tablet) {
            return 280; // Medium sidebar on tablet
        } else {
            return Math.min(350, screenWidth * 0.25); // Proportional on desktop
        }
    }

    private calculateResponsivePanelHeight(screenHeight: number): number {
        if (screenHeight < 600) {
            return 150; // Smaller panel on small screens
        } else if (screenHeight < 900) {
            return 200; // Medium panel
        } else {
            return Math.min(400, screenHeight * 0.3); // Proportional on large screens
        }
    }

    private enforceMinimumSizes(dimension: Dimension): void {
        const minWorkspaceWidth = 600;
        const minWorkspaceHeight = 400;

        if (dimension.width < minWorkspaceWidth || dimension.height < minWorkspaceHeight) {
            // Hide non-essential parts to preserve workspace
            this.layoutService.setPanelHidden(true);

            if (dimension.width < 800) {
                this.layoutService.setSideBarHidden(true);
            }
        }
    }

    private registerLayoutRules(): void {
        // Mobile rules
        this._layoutRules.set('mobile', [
            { action: 'collapse', target: 'sidebar' },
            { action: 'hide', target: 'panel' },
            { action: 'reposition', target: 'panel', position: Position.BOTTOM }
        ]);

        // Tablet rules
        this._layoutRules.set('tablet', [
            { action: 'show', target: 'sidebar' },
            { action: 'reposition', target: 'panel', position: Position.BOTTOM }
        ]);

        // Desktop rules
        this._layoutRules.set('desktop', [
            { action: 'expand', target: 'sidebar' },
            { action: 'show', target: 'panel' }
        ]);
    }
}

interface ILayoutRule {
    action: 'hide' | 'show' | 'collapse' | 'expand' | 'reposition';
    target: 'sidebar' | 'panel' | 'activitybar';
    position?: Position;
}
```

## 🎛️ Splitter System

### **The Interactive Resizer**
```typescript
export class SplitterController {
    private _isDragging = false;
    private _dragStartPosition = { x: 0, y: 0 };
    private _dragStartSize = 0;
    private _currentSplitter: HTMLElement | null = null;

    constructor(
        @ILayoutService private readonly layoutService: ILayoutService,
        @IThemeService private readonly themeService: IThemeService
    ) {
        this.registerGlobalListeners();
    }

    // Start sidebar resize
    startSidebarResize(event: MouseEvent): void {
        this._isDragging = true;
        this._dragStartPosition = { x: event.clientX, y: event.clientY };
        this._dragStartSize = this.layoutService.getSideBarWidth();
        this._currentSplitter = event.target as HTMLElement;

        // Add dragging class
        document.body.classList.add('dragging-sidebar');

        // Prevent text selection
        event.preventDefault();
    }

    // Start panel resize
    startPanelResize(event: MouseEvent): void {
        this._isDragging = true;
        this._dragStartPosition = { x: event.clientX, y: event.clientY };
        this._dragStartSize = this.layoutService.getPanelHeight();
        this._currentSplitter = event.target as HTMLElement;

        // Add dragging class
        document.body.classList.add('dragging-panel');

        event.preventDefault();
    }

    // Handle mouse move during drag
    private onMouseMove(event: MouseEvent): void {
        if (!this._isDragging || !this._currentSplitter) {
            return;
        }

        const deltaX = event.clientX - this._dragStartPosition.x;
        const deltaY = event.clientY - this._dragStartPosition.y;

        if (this._currentSplitter.classList.contains('sidebar-splitter')) {
            // Sidebar resize
            const newWidth = this._dragStartSize + deltaX;
            this.layoutService.resizeSidebar(newWidth);
        } else if (this._currentSplitter.classList.contains('panel-splitter')) {
            // Panel resize
            const newHeight = this._dragStartSize - deltaY; // Negative because panel grows upward
            this.layoutService.resizePanel(newHeight);
        }

        // Update cursor
        this.updateCursor(event);
    }

    // Handle mouse up (end drag)
    private onMouseUp(event: MouseEvent): void {
        if (!this._isDragging) {
            return;
        }

        this._isDragging = false;
        this._currentSplitter = null;

        // Remove dragging classes
        document.body.classList.remove('dragging-sidebar', 'dragging-panel');

        // Reset cursor
        document.body.style.cursor = '';
    }

    private updateCursor(event: MouseEvent): void {
        if (this._currentSplitter?.classList.contains('sidebar-splitter')) {
            document.body.style.cursor = 'col-resize';
        } else if (this._currentSplitter?.classList.contains('panel-splitter')) {
            document.body.style.cursor = 'row-resize';
        }
    }

    private registerGlobalListeners(): void {
        // Global mouse move and up listeners for drag operations
        document.addEventListener('mousemove', (e) => this.onMouseMove(e));
        document.addEventListener('mouseup', (e) => this.onMouseUp(e));

        // Prevent context menu during drag
        document.addEventListener('contextmenu', (e) => {
            if (this._isDragging) {
                e.preventDefault();
            }
        });
    }

    // Create splitter element
    createSplitter(type: 'sidebar' | 'panel' | 'auxiliary', orientation: 'horizontal' | 'vertical'): HTMLElement {
        const splitter = document.createElement('div');
        splitter.className = `split-view-view-separator ${type}-splitter`;

        // Add orientation class
        splitter.classList.add(orientation === 'horizontal' ? 'horizontal' : 'vertical');

        // Add hover effects
        splitter.addEventListener('mouseenter', () => {
            splitter.classList.add('hover');
        });

        splitter.addEventListener('mouseleave', () => {
            splitter.classList.remove('hover');
        });

        // Set cursor style
        splitter.style.cursor = orientation === 'horizontal' ? 'row-resize' : 'col-resize';

        return splitter;
    }
}
```

## 💾 Layout Persistence

### **The Memory Keeper**
```typescript
export class LayoutPersistenceService {
    private _layoutState: ILayoutState;
    private _workspaceLayouts = new Map<string, ILayoutState>();

    constructor(
        @IStorageService private readonly storageService: IStorageService,
        @IWorkspaceContextService private readonly contextService: IWorkspaceContextService,
        @ILayoutService private readonly layoutService: ILayoutService
    ) {
        this.restoreLayoutState();
        this.registerListeners();
    }

    // Save current layout state
    saveLayoutState(): void {
        const state: ILayoutState = {
            // Part visibility
            sideBarHidden: this.layoutService.isSideBarHidden(),
            panelHidden: this.layoutService.isPanelHidden(),
            activityBarHidden: this.layoutService.isActivityBarHidden(),
            statusBarHidden: this.layoutService.isStatusBarHidden(),
            auxiliaryBarHidden: this.layoutService.isAuxiliaryBarHidden(),

            // Positions
            panelPosition: this.layoutService.getPanelPosition(),
            sideBarPosition: this.layoutService.getSideBarPosition(),

            // Sizes
            sideBarWidth: this.layoutService.getSideBarWidth(),
            panelHeight: this.layoutService.getPanelHeight(),
            auxiliaryBarWidth: this.layoutService.getAuxiliaryBarWidth(),

            // Editor layout
            editorLayout: this.getEditorLayoutState(),

            // Timestamp
            timestamp: Date.now()
        };

        this._layoutState = state;

        // Save to storage
        const workspace = this.contextService.getWorkspace();
        if (workspace.id) {
            // Workspace-specific layout
            this._workspaceLayouts.set(workspace.id, state);
            this.storageService.store(
                `workbench.layout.${workspace.id}`,
                JSON.stringify(state),
                StorageScope.WORKSPACE
            );
        } else {
            // Global layout
            this.storageService.store(
                'workbench.layout.global',
                JSON.stringify(state),
                StorageScope.GLOBAL
            );
        }
    }

    // Restore layout state
    restoreLayoutState(): void {
        const workspace = this.contextService.getWorkspace();
        let stateJson: string | undefined;

        if (workspace.id) {
            // Try workspace-specific layout first
            stateJson = this.storageService.get(`workbench.layout.${workspace.id}`, StorageScope.WORKSPACE);
        }

        if (!stateJson) {
            // Fall back to global layout
            stateJson = this.storageService.get('workbench.layout.global', StorageScope.GLOBAL);
        }

        if (stateJson) {
            try {
                const state: ILayoutState = JSON.parse(stateJson);
                this.applyLayoutState(state);
            } catch (error) {
                // Invalid state, use defaults
                this.applyDefaultLayout();
            }
        } else {
            this.applyDefaultLayout();
        }
    }

    private applyLayoutState(state: ILayoutState): void {
        // Apply part visibility
        this.layoutService.setSideBarHidden(state.sideBarHidden);
        this.layoutService.setPanelHidden(state.panelHidden);
        this.layoutService.setActivityBarHidden(state.activityBarHidden);
        this.layoutService.setStatusBarHidden(state.statusBarHidden);
        this.layoutService.setAuxiliaryBarHidden(state.auxiliaryBarHidden);

        // Apply positions
        this.layoutService.setPanelPosition(state.panelPosition);
        this.layoutService.setSideBarPosition(state.sideBarPosition);

        // Apply sizes
        this.layoutService.resizeSidebar(state.sideBarWidth);
        this.layoutService.resizePanel(state.panelHeight);
        this.layoutService.resizeAuxiliaryBar(state.auxiliaryBarWidth);

        // Apply editor layout
        if (state.editorLayout) {
            this.applyEditorLayoutState(state.editorLayout);
        }
    }

    private applyDefaultLayout(): void {
        // Default layout configuration
        this.layoutService.setSideBarHidden(false);
        this.layoutService.setPanelHidden(true);
        this.layoutService.setActivityBarHidden(false);
        this.layoutService.setStatusBarHidden(false);
        this.layoutService.setAuxiliaryBarHidden(true);

        this.layoutService.setPanelPosition(Position.BOTTOM);
        this.layoutService.setSideBarPosition(Position.LEFT);

        this.layoutService.resizeSidebar(300);
        this.layoutService.resizePanel(300);
        this.layoutService.resizeAuxiliaryBar(300);
    }

    private getEditorLayoutState(): IEditorLayoutState {
        // Get current editor group layout
        const editorGroupsService = this.layoutService.getEditorGroupsService();

        return {
            groups: editorGroupsService.groups.map(group => ({
                id: group.id,
                editors: group.editors.map(editor => ({
                    resource: editor.getResource()?.toString(),
                    options: editor.getOptions()
                })),
                activeEditor: group.activeEditor?.getResource()?.toString()
            })),
            activeGroup: editorGroupsService.activeGroup.id
        };
    }

    private applyEditorLayoutState(state: IEditorLayoutState): void {
        // Restore editor groups and their editors
        // This would involve recreating the editor layout
        // Implementation depends on editor group service
    }

    private registerListeners(): void {
        // Save layout when it changes
        this.layoutService.onDidLayout(() => {
            this.saveLayoutState();
        });

        // Save layout when parts are toggled
        this.layoutService.onDidChangeSideBarHidden(() => {
            this.saveLayoutState();
        });

        this.layoutService.onDidChangePanelHidden(() => {
            this.saveLayoutState();
        });

        // Save layout when workspace changes
        this.contextService.onDidChangeWorkspace(() => {
            this.restoreLayoutState();
        });
    }
}

// Layout state interfaces
interface ILayoutState {
    sideBarHidden: boolean;
    panelHidden: boolean;
    activityBarHidden: boolean;
    statusBarHidden: boolean;
    auxiliaryBarHidden: boolean;
    panelPosition: Position;
    sideBarPosition: Position;
    sideBarWidth: number;
    panelHeight: number;
    auxiliaryBarWidth: number;
    editorLayout?: IEditorLayoutState;
    timestamp: number;
}

interface IEditorLayoutState {
    groups: IEditorGroupState[];
    activeGroup: number;
}

interface IEditorGroupState {
    id: number;
    editors: IEditorState[];
    activeEditor?: string;
}

interface IEditorState {
    resource?: string;
    options?: any;
}
```

## 🎯 Layout System in Action

### **Complete Layout Flow Example**
```typescript
// Example: Window resize handling
export class WindowResizeHandler {
    constructor(
        @ILayoutService private readonly layoutService: ILayoutService,
        @IResponsiveLayoutEngine private readonly responsiveEngine: IResponsiveLayoutEngine,
        @ILayoutPersistenceService private readonly persistenceService: ILayoutPersistenceService
    ) {
        this.registerWindowListeners();
    }

    private registerWindowListeners(): void {
        // Handle window resize
        window.addEventListener('resize', () => {
            const dimension = new Dimension(window.innerWidth, window.innerHeight);

            // 1. Update layout service
            this.layoutService.layout(dimension);

            // 2. Apply responsive adjustments
            this.responsiveEngine.onWindowResize(dimension);

            // 3. Save new layout state
            this.persistenceService.saveLayoutState();
        });

        // Handle orientation change (mobile)
        window.addEventListener('orientationchange', () => {
            setTimeout(() => {
                const dimension = new Dimension(window.innerWidth, window.innerHeight);
                this.layoutService.layout(dimension);
                this.responsiveEngine.onWindowResize(dimension);
            }, 100); // Small delay for orientation change to complete
        });
    }
}
```

## 📚 Next Steps

Now that you understand the Layout System:

1. **[20-theme-system.md](./20-theme-system.md)** - Master the theming system
2. **[21-extension-architecture.md](./21-extension-architecture.md)** - Move to extension system
3. **[26-file-explorer.md](./26-file-explorer.md)** - Learn about specific UI features

## 🎯 Key Takeaways

VS Code's Layout System is a sophisticated spatial management engine:

- **Flexible Grid**: Supports complex layouts with resizable panels and split views
- **Responsive Design**: Automatically adapts to different screen sizes and orientations
- **Interactive Resizing**: Users can drag splitters to customize their workspace
- **State Persistence**: Remembers layout preferences across sessions and workspaces
- **Performance Optimized**: Efficient layout calculations and minimal DOM manipulation

Understanding the layout system helps you:
- **Debug layout issues** by understanding how positioning works
- **Build responsive extensions** that work well at different screen sizes
- **Customize VS Code** by understanding how the UI can be arranged
- **Optimize performance** by understanding layout calculation costs

The layout system is like having a master architect constantly redesigning your workspace to be as efficient and comfortable as possible! 📐✨
