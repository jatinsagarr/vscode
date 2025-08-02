# 💉 VS Code Dependency Injection System - Complete Guide

## 🎯 Overview

VS Code's Dependency Injection (DI) system is the architectural foundation that powers the entire application. It manages service lifecycles, resolves dependencies, and enables the modular, testable architecture that makes VS Code's massive codebase maintainable.

## 📊 DI System Statistics

- **100+ Services**: Managed across all layers
- **Decorator-Based**: Uses TypeScript decorators for injection
- **Hierarchical**: Child containers inherit parent services
- **Type-Safe**: Full TypeScript type safety
- **Performance**: Lazy instantiation and singleton management

## 🏗️ DI System Architecture

### **Core Components**

```
┌─────────────────────────────────────────────────────────────┐
│                    DI SYSTEM COMPONENTS                     │
├─────────────────┬─────────────────┬─────────────────────────┤
│ ServiceIdentifier│ ServiceCollection│ InstantiationService   │
│ • Type safety    │ • Service registry│ • Service creation     │
│ • Decoration     │ • Lifecycle mgmt │ • Dependency resolution│
│ • Branding       │ • Hierarchical   │ • Circular detection   │
└─────────────────┴─────────────────┴─────────────────────────┘
```

## 🔧 Service Identification System

### **Creating Service Identifiers**

#### **Service Decorator Creation**
```typescript
// src/vs/platform/instantiation/common/instantiation.ts

// Create a service identifier with type safety
export function createDecorator<T>(serviceId: string): ServiceIdentifier<T> {
    const id = function (target: Function, key: string, index: number): void {
        // Store dependency information for injection
        storeServiceDependency(id, target, index);
    };

    // Make the identifier debuggable
    id.toString = () => serviceId;

    // Store in global registry for debugging
    _util.serviceIds.set(serviceId, id);

    return id as ServiceIdentifier<T>;
}

// Service identifier interface
export interface ServiceIdentifier<T> {
    (...args: any[]): void;  // Decorator function
    type: T;                 // Type information
}
```

#### **Example Service Definitions**
```typescript
// File service identifier
export const IFileService = createDecorator<IFileService>('fileService');

export interface IFileService {
    readonly _serviceBrand: undefined; // Service branding for type safety

    // File operations
    readFile(resource: URI): Promise<IFileContent>;
    writeFile(resource: URI, content: VSBuffer): Promise<void>;
    exists(resource: URI): Promise<boolean>;

    // Events
    readonly onDidFilesChange: Event<FileChangesEvent>;
}

// Configuration service identifier
export const IConfigurationService = createDecorator<IConfigurationService>('configurationService');

export interface IConfigurationService {
    readonly _serviceBrand: undefined;

    // Configuration access
    getValue<T>(key: string): T;
    updateValue(key: string, value: any): Promise<void>;

    // Events
    readonly onDidChangeConfiguration: Event<IConfigurationChangeEvent>;
}

// Command service identifier
export const ICommandService = createDecorator<ICommandService>('commandService');

export interface ICommandService {
    readonly _serviceBrand: undefined;

    // Command execution
    executeCommand<T>(id: string, ...args: any[]): Promise<T>;

    // Command registration
    onWillExecuteCommand: Event<ICommandEvent>;
}
```

### **Service Branding**
```typescript
// Service branding prevents type confusion
export type BrandedService = { _serviceBrand: undefined };

// This ensures you can't accidentally use the wrong service type
interface IMyService extends BrandedService {
    doSomething(): void;
}

interface IAnotherService extends BrandedService {
    doSomethingElse(): void;
}

// TypeScript will catch this error:
// const myService: IMyService = someAnotherService; // ❌ Type error!
```

## 🏭 Service Collection System

### **ServiceCollection Class**
```typescript
// src/vs/platform/instantiation/common/serviceCollection.ts
export class ServiceCollection {
    private _entries = new Map<ServiceIdentifier<any>, any>();

    constructor(...entries: [ServiceIdentifier<any>, any][]) {
        for (const [id, service] of entries) {
            this.set(id, service);
        }
    }

    // Set a service (instance or descriptor)
    set<T>(id: ServiceIdentifier<T>, instanceOrDescriptor: T | SyncDescriptor<T>): T | SyncDescriptor<T> {
        const result = this._entries.get(id);
        this._entries.set(id, instanceOrDescriptor);
        return result;
    }

    // Check if service exists
    has(id: ServiceIdentifier<any>): boolean {
        return this._entries.has(id);
    }

    // Get service or descriptor
    get<T>(id: ServiceIdentifier<T>): T | SyncDescriptor<T> {
        return this._entries.get(id);
    }
}
```

### **Service Registration Patterns**

#### **Direct Instance Registration**
```typescript
// Register a service instance directly
const services = new ServiceCollection();

// Create and register file service instance
const fileService = new FileService();
services.set(IFileService, fileService);

// Register configuration service instance
const configService = new ConfigurationService();
services.set(IConfigurationService, configService);
```

#### **Descriptor-Based Registration**
```typescript
// Register services using descriptors for lazy instantiation
import { SyncDescriptor } from './descriptors.js';

const services = new ServiceCollection();

// Register with descriptor - service created when first requested
services.set(IFileService, new SyncDescriptor(FileService));
services.set(IConfigurationService, new SyncDescriptor(ConfigurationService, [arg1, arg2]));

// Register with static arguments
services.set(ILogService, new SyncDescriptor(LogService, ['main'], true)); // supports delayed instantiation
```

## 🏭 Service Descriptors

### **SyncDescriptor Class**
```typescript
// src/vs/platform/instantiation/common/descriptors.ts
export class SyncDescriptor<T> {
    readonly ctor: any;                        // Constructor function
    readonly staticArguments: any[];           // Arguments passed to constructor
    readonly supportsDelayedInstantiation: boolean; // Can be instantiated later

    constructor(
        ctor: new (...args: any[]) => T,
        staticArguments: any[] = [],
        supportsDelayedInstantiation: boolean = false
    ) {
        this.ctor = ctor;
        this.staticArguments = staticArguments;
        this.supportsDelayedInstantiation = supportsDelayedInstantiation;
    }
}
```

### **Descriptor Usage Examples**
```typescript
// Basic descriptor
const fileServiceDescriptor = new SyncDescriptor(FileService);

// Descriptor with static arguments
const logServiceDescriptor = new SyncDescriptor(
    LogService,
    ['main', LogLevel.Info], // Static arguments
    true // Supports delayed instantiation
);

// Descriptor for complex service
const editorServiceDescriptor = new SyncDescriptor(
    EditorService,
    [/* static args */],
    false // Must be instantiated immediately when requested
);
```

## 🏭 Instantiation Service

### **IInstantiationService Interface**
```typescript
// src/vs/platform/instantiation/common/instantiation.ts
export interface IInstantiationService {
    readonly _serviceBrand: undefined;

    // Create instances with automatic dependency injection
    createInstance<T>(descriptor: SyncDescriptor0<T>): T;
    createInstance<Ctor extends new (...args: any[]) => unknown, R extends InstanceType<Ctor>>(
        ctor: Ctor,
        ...args: GetLeadingNonServiceArgs<ConstructorParameters<Ctor>>
    ): R;

    // Invoke functions with service accessor
    invokeFunction<R, TS extends any[] = []>(
        fn: (accessor: ServicesAccessor, ...args: TS) => R,
        ...args: TS
    ): R;

    // Create child containers
    createChild(services: ServiceCollection, store?: DisposableStore): IInstantiationService;

    // Dispose the service
    dispose(): void;
}
```

### **InstantiationService Implementation**
```typescript
// src/vs/platform/instantiation/common/instantiationService.ts
export class InstantiationService implements IInstantiationService {
    private readonly _services: ServiceCollection;
    private readonly _activeInstantiations = new Set<ServiceIdentifier<any>>();

    constructor(services: ServiceCollection = new ServiceCollection(), strict: boolean = false) {
        this._services = services;
        this._services.set(IInstantiationService, this);
    }

    createInstance<T>(ctorOrDescriptor: any, ...rest: any[]): T {
        let result: T;

        if (ctorOrDescriptor instanceof SyncDescriptor) {
            // Create from descriptor
            result = this._createInstance(ctorOrDescriptor.ctor, ctorOrDescriptor.staticArguments.concat(rest));
        } else {
            // Create from constructor
            result = this._createInstance(ctorOrDescriptor, rest);
        }

        return result;
    }

    private _createInstance<T>(ctor: any, args: any[] = []): T {
        // Get service dependencies from decorator metadata
        const serviceDependencies = _util.getServiceDependencies(ctor);
        const serviceArgs: any[] = [];

        // Resolve each service dependency
        for (const dependency of serviceDependencies) {
            const service = this._getOrCreateServiceInstance(dependency.id);
            serviceArgs[dependency.index] = service;
        }

        // Merge static args with service args
        const allArgs = [...args];
        for (let i = 0; i < serviceArgs.length; i++) {
            if (serviceArgs[i] !== undefined) {
                allArgs[i] = serviceArgs[i];
            }
        }

        // Create instance
        return new ctor(...allArgs);
    }

    private _getOrCreateServiceInstance<T>(id: ServiceIdentifier<T>): T {
        // Check for circular dependencies
        if (this._activeInstantiations.has(id)) {
            throw new Error(`Circular dependency detected: ${id}`);
        }

        // Get service from collection
        let thing = this._services.get(id);

        if (thing instanceof SyncDescriptor) {
            // Mark as being instantiated
            this._activeInstantiations.add(id);

            try {
                // Create instance from descriptor
                thing = this._createInstance(thing.ctor, thing.staticArguments);

                // Store instance for reuse (singleton behavior)
                this._services.set(id, thing);
            } finally {
                this._activeInstantiations.delete(id);
            }
        }

        return thing;
    }
}
```

## 🎯 Dependency Injection Usage

### **Service Consumer Pattern**
```typescript
// Example service that depends on other services
export class WorkbenchEditorService implements IEditorService {
    readonly _serviceBrand: undefined;

    constructor(
        @IFileService private readonly fileService: IFileService,
        @IConfigurationService private readonly configurationService: IConfigurationService,
        @ICommandService private readonly commandService: ICommandService,
        @IInstantiationService private readonly instantiationService: IInstantiationService
    ) {
        // Service is automatically injected by the DI system
        this.initialize();
    }

    async openEditor(resource: URI): Promise<IEditor> {
        // Use injected file service
        const content = await this.fileService.readFile(resource);

        // Use injected configuration service
        const config = this.configurationService.getValue('editor');

        // Create editor using instantiation service
        const editor = this.instantiationService.createInstance(CodeEditor, content, config);

        return editor;
    }

    private initialize(): void {
        // Listen to configuration changes
        this.configurationService.onDidChangeConfiguration(e => {
            if (e.affectsConfiguration('editor')) {
                this.updateEditorConfiguration();
            }
        });
    }
}
```

### **Service Registration in Bootstrap**
```typescript
// Example from workbench startup
function createWorkbenchServices(): ServiceCollection {
    const services = new ServiceCollection();

    // Core platform services
    services.set(IFileService, new SyncDescriptor(FileService));
    services.set(IConfigurationService, new SyncDescriptor(ConfigurationService));
    services.set(ICommandService, new SyncDescriptor(CommandService));

    // Workbench services
    services.set(IEditorService, new SyncDescriptor(WorkbenchEditorService));
    services.set(IViewletService, new SyncDescriptor(ViewletService));
    services.set(IPanelService, new SyncDescriptor(PanelService));

    // Create instantiation service
    const instantiationService = new InstantiationService(services);
    services.set(IInstantiationService, instantiationService);

    return services;
}
```

### **Service Accessor Pattern**
```typescript
// Using service accessor for functional programming
export function registerCommand(id: string, handler: (accessor: ServicesAccessor, ...args: any[]) => any): void {
    CommandsRegistry.registerCommand(id, (accessor, ...args) => {
        // Get services from accessor
        const fileService = accessor.get(IFileService);
        const editorService = accessor.get(IEditorService);
        const notificationService = accessor.get(INotificationService);

        // Execute command logic
        return handler(accessor, ...args);
    });
}

// Example command registration
registerCommand('workbench.action.files.openFile', async (accessor) => {
    const fileService = accessor.get(IFileService);
    const editorService = accessor.get(IEditorService);

    // Show file picker
    const files = await fileService.showOpenDialog();

    // Open selected files
    for (const file of files) {
        await editorService.openEditor({ resource: file });
    }
});
```

## 🔄 Service Lifecycle Management

### **Service Creation Lifecycle**
```typescript
// Service lifecycle phases
enum ServiceLifecycle {
    NotCreated,    // Service not yet instantiated
    Creating,      // Service is being created (prevents circular deps)
    Created,       // Service has been created
    Disposed       // Service has been disposed
}

class ServiceLifecycleManager {
    private serviceStates = new Map<ServiceIdentifier<any>, ServiceLifecycle>();

    createService<T>(id: ServiceIdentifier<T>): T {
        const currentState = this.serviceStates.get(id) || ServiceLifecycle.NotCreated;

        switch (currentState) {
            case ServiceLifecycle.NotCreated:
                this.serviceStates.set(id, ServiceLifecycle.Creating);
                const service = this.instantiateService(id);
                this.serviceStates.set(id, ServiceLifecycle.Created);
                return service;

            case ServiceLifecycle.Creating:
                throw new Error(`Circular dependency detected for service: ${id}`);

            case ServiceLifecycle.Created:
                return this.getExistingService(id);

            case ServiceLifecycle.Disposed:
                throw new Error(`Cannot create disposed service: ${id}`);
        }
    }
}
```

### **Child Container Pattern**
```typescript
// Create child containers for scoped services
export class ScopedInstantiationService {
    constructor(
        private parent: IInstantiationService,
        private scopedServices: ServiceCollection
    ) {}

    createChild(additionalServices?: ServiceCollection): IInstantiationService {
        // Merge parent services with scoped services
        const childServices = new ServiceCollection();

        // Copy parent services
        for (const [id, service] of this.parent.services) {
            childServices.set(id, service);
        }

        // Override with scoped services
        for (const [id, service] of this.scopedServices) {
            childServices.set(id, service);
        }

        // Add additional services
        if (additionalServices) {
            for (const [id, service] of additionalServices) {
                childServices.set(id, service);
            }
        }

        return new InstantiationService(childServices);
    }
}
```

## 🧪 Testing with DI

### **Mock Service Creation**
```typescript
// Create mock services for testing
class MockFileService implements IFileService {
    readonly _serviceBrand: undefined;

    private files = new Map<string, string>();

    async readFile(resource: URI): Promise<IFileContent> {
        const content = this.files.get(resource.toString());
        if (!content) {
            throw new Error('File not found');
        }

        return {
            resource,
            value: content,
            etag: '1',
            mtime: Date.now(),
            ctime: Date.now(),
            size: content.length
        };
    }

    async writeFile(resource: URI, content: VSBuffer): Promise<void> {
        this.files.set(resource.toString(), content.toString());
    }

    // Mock implementation of other methods...
}

// Test setup
function createTestServices(): ServiceCollection {
    const services = new ServiceCollection();

    // Use mock services
    services.set(IFileService, new MockFileService());
    services.set(IConfigurationService, new MockConfigurationService());

    // Use real services where needed
    services.set(IInstantiationService, new InstantiationService(services));

    return services;
}
```

### **Test Example**
```typescript
// Example test using DI
describe('EditorService', () => {
    let instantiationService: IInstantiationService;
    let editorService: IEditorService;

    beforeEach(() => {
        const services = createTestServices();
        instantiationService = new InstantiationService(services);
        editorService = instantiationService.createInstance(EditorService);
    });

    it('should open editor', async () => {
        const resource = URI.file('/test.txt');

        // Mock file content
        const fileService = instantiationService.get(IFileService) as MockFileService;
        await fileService.writeFile(resource, VSBuffer.fromString('test content'));

        // Test editor opening
        const editor = await editorService.openEditor({ resource });

        expect(editor).toBeDefined();
        expect(editor.getModel()?.getValue()).toBe('test content');
    });
});
```

## 🔍 DI System Debugging

### **Service Resolution Tracing**
```typescript
// Debug service resolution
class DebuggingInstantiationService extends InstantiationService {
    private resolutionStack: string[] = [];

    createInstance<T>(ctorOrDescriptor: any, ...rest: any[]): T {
        const serviceName = ctorOrDescriptor.name || ctorOrDescriptor.toString();

        console.log(`Creating service: ${serviceName}`);
        console.log(`Resolution stack: ${this.resolutionStack.join(' -> ')}`);

        this.resolutionStack.push(serviceName);

        try {
            const result = super.createInstance(ctorOrDescriptor, ...rest);
            console.log(`✅ Created service: ${serviceName}`);
            return result;
        } catch (error) {
            console.error(`❌ Failed to create service: ${serviceName}`, error);
            throw error;
        } finally {
            this.resolutionStack.pop();
        }
    }
}
```

### **Circular Dependency Detection**
```typescript
// Enhanced circular dependency detection
class CircularDependencyDetector {
    private static activeCreations = new Set<string>();

    static checkCircularDependency(serviceId: string): void {
        if (this.activeCreations.has(serviceId)) {
            const chain = Array.from(this.activeCreations).join(' -> ');
            throw new Error(`Circular dependency detected: ${chain} -> ${serviceId}`);
        }
    }

    static startCreation(serviceId: string): void {
        this.activeCreations.add(serviceId);
    }

    static endCreation(serviceId: string): void {
        this.activeCreations.delete(serviceId);
    }
}
```

## 🎯 DI Best Practices

### **1. Service Interface Design**
```typescript
// ✅ Good: Clear, focused interface
export interface IFileService {
    readonly _serviceBrand: undefined;

    // Core operations
    readFile(resource: URI): Promise<IFileContent>;
    writeFile(resource: URI, content: VSBuffer): Promise<void>;

    // Events
    readonly onDidFilesChange: Event<FileChangesEvent>;
}

// ❌ Bad: Too many responsibilities
export interface IBadService {
    // File operations
    readFile(resource: URI): Promise<IFileContent>;

    // Network operations
    makeHttpRequest(url: string): Promise<Response>;

    // UI operations
    showDialog(message: string): Promise<void>;
}
```

### **2. Dependency Management**
```typescript
// ✅ Good: Minimal dependencies
export class EditorService {
    constructor(
        @IFileService private fileService: IFileService,
        @IConfigurationService private configService: IConfigurationService
    ) {}
}

// ❌ Bad: Too many dependencies (God object)
export class BadService {
    constructor(
        @IFileService private fileService: IFileService,
        @IConfigurationService private configService: IConfigurationService,
        @ICommandService private commandService: ICommandService,
        @INotificationService private notificationService: INotificationService,
        @IDialogService private dialogService: IDialogService,
        @IWorkspaceService private workspaceService: IWorkspaceService,
        // ... 10 more dependencies
    ) {}
}
```

### **3. Service Registration**
```typescript
// ✅ Good: Register services at appropriate level
function registerPlatformServices(services: ServiceCollection): void {
    // Platform-level services
    services.set(IFileService, new SyncDescriptor(FileService));
    services.set(IConfigurationService, new SyncDescriptor(ConfigurationService));
}

function registerWorkbenchServices(services: ServiceCollection): void {
    // Workbench-level services
    services.set(IEditorService, new SyncDescriptor(EditorService));
    services.set(IViewletService, new SyncDescriptor(ViewletService));
}

// ❌ Bad: Mix service levels
function registerAllServices(services: ServiceCollection): void {
    // Don't mix platform and workbench services in one place
    services.set(IFileService, new SyncDescriptor(FileService));        // Platform
    services.set(IEditorService, new SyncDescriptor(EditorService));    // Workbench
}
```

## 📚 Next Steps

Now that you understand the DI system:

1. **[09-base-layer.md](./09-base-layer.md)** - Explore the foundation layer
2. **[10-platform-layer.md](./10-platform-layer.md)** - Understand core services
3. **[11-monaco-editor.md](./11-monaco-editor.md)** - Dive into the editor layer

Understanding the DI system is crucial for:
- **Service development**: Creating new services correctly
- **Testing**: Mocking dependencies effectively
- **Debugging**: Understanding service resolution issues
- **Architecture**: Maintaining clean dependencies

The DI system is the glue that holds VS Code's architecture together! 💉
