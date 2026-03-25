# Canvas System Module

## Overview

The **canvas-system** module provides a sophisticated image manipulation and composition interface built on top of Konva.js and React. It enables users to upload, transform, layer, and edit images with real-time visual feedback through an interactive canvas workspace. The module serves as the core visual editing engine of the application, offering professional-grade image manipulation capabilities including transformations, filters, and layer management.

## Architecture

### High-Level Architecture

```mermaid
graph TB
    subgraph "Canvas System"
        Canvas[Canvas Component]
        LayerControls[Layer Controls]
        CanvasRef[Canvas Reference API]
    end
    
    subgraph "External Dependencies"
        Konva[Konva.js Library]
        ReactKonva[React-Konva]
        AntD[Ant Design UI]
    end
    
    subgraph "Type System"
        Layer[Layer Type]
        CanvasConfig[Canvas Config]
        Transform[Transform Data]
    end
    
    subgraph "User Interactions"
        Upload[Image Upload]
        Transform_UI[Transform Controls]
        Filters[Filter Controls]
        Export[Export Operations]
    end
    
    Canvas --> LayerControls
    Canvas --> CanvasRef
    Canvas --> Konva
    Canvas --> ReactKonva
    Canvas --> AntD
    Canvas --> Layer
    Canvas --> CanvasConfig
    
    Upload --> Canvas
    Transform_UI --> Canvas
    Filters --> Canvas
    Export --> Canvas
    
    LayerControls --> Layer
    LayerControls --> Transform
    
    style Canvas fill:#4A90E2,stroke:#2E5C8A,stroke-width:3px,color:#fff
    style CanvasRef fill:#50C878,stroke:#2E7D4E,stroke-width:2px,color:#fff
```

### Component Architecture

```mermaid
graph LR
    subgraph "Canvas Component"
        State[Component State]
        Refs[React Refs]
        Handlers[Event Handlers]
        Render[Render Logic]
    end
    
    subgraph "State Management"
        Layers[Layers Array]
        SelectedId[Selected Layer ID]
        Scale[Canvas Scale]
    end
    
    subgraph "Refs"
        StageRef[Stage Reference]
        TransformerRef[Transformer Reference]
        ContainerRef[Container Reference]
        DragPosRef[Drag Position Reference]
    end
    
    subgraph "Core Operations"
        ImageOps[Image Operations]
        TransformOps[Transform Operations]
        FilterOps[Filter Operations]
        LayerOps[Layer Operations]
    end
    
    State --> Layers
    State --> SelectedId
    State --> Scale
    
    Refs --> StageRef
    Refs --> TransformerRef
    Refs --> ContainerRef
    Refs --> DragPosRef
    
    Handlers --> ImageOps
    Handlers --> TransformOps
    Handlers --> FilterOps
    Handlers --> LayerOps
    
    Render --> State
    Render --> Refs
    
    style State fill:#E8F4F8,stroke:#4A90E2,stroke-width:2px
    style Refs fill:#FFF4E6,stroke:#F39C12,stroke-width:2px
    style Handlers fill:#E8F8F5,stroke:#50C878,stroke-width:2px
```

## Core Components

### CanvasRef Interface

The `CanvasRef` interface provides imperative access to canvas operations from parent components.

```typescript
export interface CanvasRef {
  getDataUrl: () => string | null;
}
```

**Purpose**: Exposes canvas export functionality to parent components through React's `forwardRef` and `useImperativeHandle` pattern.

**Key Method**:
- `getDataUrl()`: Exports the current canvas state as a base64-encoded PNG data URL with high quality (2x pixel ratio)

### Canvas Component

The main Canvas component is a complex React component that manages the entire image editing workspace.

**Props**:
```typescript
interface CanvasProps {
  config: CanvasConfig;
}
```

**Key Features**:
- Multi-layer image composition
- Real-time transformations (move, scale, rotate)
- Image filters (brightness, contrast, saturation, hue)
- Layer management (add, delete, duplicate, reorder)
- Responsive canvas scaling
- Keyboard shortcuts support
- High-quality export

## Data Flow

### Image Upload Flow

```mermaid
sequenceDiagram
    participant User
    participant FileInput
    participant Canvas
    participant FileReader
    participant ImageLoader
    participant State
    
    User->>FileInput: Select Image File
    FileInput->>Canvas: handleImageUpload(event)
    Canvas->>FileReader: readAsDataURL(file)
    FileReader->>ImageLoader: Load Image Data
    ImageLoader->>Canvas: Calculate Dimensions
    Canvas->>Canvas: calculateImageDimensions()
    Canvas->>State: Add New Layer
    State->>Canvas: Re-render with New Layer
    Canvas->>User: Display Image on Canvas
```

### Transform Operation Flow

```mermaid
sequenceDiagram
    participant User
    participant KonvaImage
    participant Transformer
    participant Canvas
    participant State
    
    User->>KonvaImage: Drag/Scale/Rotate
    KonvaImage->>Canvas: onDragEnd/onTransformEnd
    Canvas->>Canvas: handleTransform()
    Canvas->>State: Update Layer Properties
    State->>Canvas: Trigger Re-render
    Canvas->>Transformer: Update Transformer Position
    Transformer->>User: Visual Feedback
```

### Layer Selection Flow

```mermaid
stateDiagram-v2
    [*] --> NoSelection: Initial State
    NoSelection --> LayerSelected: Click on Layer
    LayerSelected --> NoSelection: Click on Empty Area
    LayerSelected --> LayerSelected: Click on Different Layer
    LayerSelected --> NoSelection: Delete Layer
    
    NoSelection: No Transformer Visible
    LayerSelected: Transformer Attached
    
    note right of LayerSelected
        - Transformer visible
        - Layer controls enabled
        - Keyboard shortcuts active
    end note
```

## State Management

### Component State Structure

```mermaid
graph TB
    subgraph "Canvas State"
        Layers["layers: Layer Array"]
        SelectedId["selectedId: string or null"]
        Scale["scale: number"]
    end
    
    subgraph "Derived State"
        SelectedLayer["selectedLayer: Layer or null"]
    end
    
    subgraph "Ref State"
        StageRef["stageRef: Konva.Stage"]
        TransformerRef["transformerRef: Konva.Transformer"]
        ContainerRef["containerRef: HTMLDivElement"]
        DragStartPos["dragStartPosRef: Position"]
    end
    
    Layers --> SelectedLayer
    SelectedId --> SelectedLayer
    
    SelectedLayer --> TransformerRef
    StageRef --> TransformerRef
    
    style Layers fill:#E3F2FD,stroke:#2196F3,stroke-width:2px
    style SelectedId fill:#FFF3E0,stroke:#FF9800,stroke-width:2px
    style Scale fill:#F3E5F5,stroke:#9C27B0,stroke-width:2px
```

### State Update Patterns

**Layer Updates**:
```typescript
// Immutable update pattern used throughout
setLayers(prev => prev.map(layer => 
  layer.id === selectedId
    ? { ...layer, ...updates }
    : layer
));
```

**Selection Management**:
```typescript
// Selection triggers transformer attachment
useEffect(() => {
  if (!selectedId) {
    transformerRef.current?.nodes([]);
    return;
  }
  
  const node = stageRef.current.findOne('#' + selectedId);
  if (node) {
    transformerRef.current.nodes([node]);
  }
}, [selectedId]);
```

## Core Operations

### Image Operations

```mermaid
graph LR
    subgraph "Image Upload"
        Upload[File Upload]
        Read[FileReader]
        Load[Image Load]
        Calc[Calculate Dimensions]
        Create[Create Layer]
    end
    
    Upload --> Read
    Read --> Load
    Load --> Calc
    Calc --> Create
    
    subgraph "Dimension Calculation"
        MaxSize[Max Size: 90% Canvas]
        AspectRatio[Maintain Aspect Ratio]
        Center[Center Position]
    end
    
    Calc --> MaxSize
    Calc --> AspectRatio
    Calc --> Center
    
    style Upload fill:#E8F5E9,stroke:#4CAF50,stroke-width:2px
    style Create fill:#E3F2FD,stroke:#2196F3,stroke-width:2px
```

**Key Algorithm - Image Dimension Calculation**:
```typescript
calculateImageDimensions(imageWidth, imageHeight) {
  const maxWidth = config.width * 0.9;
  const maxHeight = config.height * 0.9;
  
  let newWidth = imageWidth;
  let newHeight = imageHeight;
  const aspectRatio = imageWidth / imageHeight;

  // Scale down if exceeds canvas bounds
  if (imageWidth > maxWidth || imageHeight > maxHeight) {
    if (maxWidth / maxHeight > aspectRatio) {
      newHeight = maxHeight;
      newWidth = maxHeight * aspectRatio;
    } else {
      newWidth = maxWidth;
      newHeight = maxWidth / aspectRatio;
    }
  }

  // Center on canvas
  return {
    width: newWidth,
    height: newHeight,
    scaleX: 1,
    scaleY: 1,
    x: (config.width - newWidth) / 2,
    y: (config.height - newHeight) / 2
  };
}
```

### Transform Operations

```mermaid
graph TB
    subgraph "Transform Types"
        Position[Position: x, y]
        Scale[Scale: scaleX, scaleY]
        Rotation[Rotation: degrees]
    end
    
    subgraph "Transform Sources"
        Drag[Drag Events]
        Transformer[Transformer Events]
        Controls[Layer Controls]
    end
    
    subgraph "Transform Handler"
        Handler[handleTransform]
        Update[Update Layer State]
        Render[Re-render Canvas]
    end
    
    Drag --> Handler
    Transformer --> Handler
    Controls --> Handler
    
    Handler --> Update
    Update --> Render
    
    Position -.-> Handler
    Scale -.-> Handler
    Rotation -.-> Handler
    
    style Handler fill:#FFF3E0,stroke:#FF9800,stroke-width:3px
```

**Transform Handler Implementation**:
```typescript
const handleTransform = useCallback((transform: Partial<LayerType>) => {
  if (!selectedId) return;
  
  setLayers(prev => prev.map(layer => 
    layer.id === selectedId
      ? { ...layer, ...transform }
      : layer
  ));
}, [selectedId]);
```

### Filter Operations

```mermaid
graph LR
    subgraph "Available Filters"
        Brightness[Brightness]
        Contrast[Contrast]
        Saturation[Saturation]
        Hue[Hue]
    end
    
    subgraph "Konva Filters"
        KBrighten[Konva.Filters.Brighten]
        KContrast[Konva.Filters.Contrast]
        KHSL[Konva.Filters.HSL]
    end
    
    subgraph "Application"
        LayerControls[Layer Controls UI]
        FilterChange[handleFilterChange]
        ImageNode[Konva Image Node]
    end
    
    Brightness --> KBrighten
    Contrast --> KContrast
    Saturation --> KHSL
    Hue --> KHSL
    
    LayerControls --> FilterChange
    FilterChange --> ImageNode
    
    KBrighten --> ImageNode
    KContrast --> ImageNode
    KHSL --> ImageNode
    
    style ImageNode fill:#E8F5E9,stroke:#4CAF50,stroke-width:2px
```

### Layer Management Operations

```mermaid
graph TB
    subgraph "Layer Operations"
        Add[Add Layer]
        Delete[Delete Layer]
        Duplicate[Duplicate Layer]
        Reorder[Reorder Layers]
        Visibility[Toggle Visibility]
    end
    
    subgraph "Reorder Operations"
        MoveUp[Move Up in Stack]
        MoveDown[Move Down in Stack]
    end
    
    subgraph "State Updates"
        LayersArray[Layers Array]
        Selection[Selected ID]
    end
    
    Add --> LayersArray
    Delete --> LayersArray
    Delete --> Selection
    Duplicate --> LayersArray
    Duplicate --> Selection
    
    Reorder --> MoveUp
    Reorder --> MoveDown
    MoveUp --> LayersArray
    MoveDown --> LayersArray
    
    Visibility --> LayersArray
    
    style LayersArray fill:#E3F2FD,stroke:#2196F3,stroke-width:2px
    style Selection fill:#FFF3E0,stroke:#FF9800,stroke-width:2px
```

**Layer Reordering Algorithm**:
```typescript
// Move layer up in z-index
handleMoveUp() {
  const index = layers.findIndex(l => l.id === selectedId);
  if (index < layers.length - 1) {
    const newLayers = [...layers];
    [newLayers[index], newLayers[index + 1]] = 
      [newLayers[index + 1], newLayers[index]];
    setLayers(newLayers);
  }
}

// Move layer down in z-index
handleMoveDown() {
  const index = layers.findIndex(l => l.id === selectedId);
  if (index > 0) {
    const newLayers = [...layers];
    [newLayers[index], newLayers[index - 1]] = 
      [newLayers[index - 1], newLayers[index]];
    setLayers(newLayers);
  }
}
```

## Export System

### Export Process Flow

```mermaid
sequenceDiagram
    participant Parent
    participant CanvasRef
    participant Canvas
    participant Transformer
    participant Stage
    participant Output
    
    Parent->>CanvasRef: getDataUrl()
    CanvasRef->>Transformer: Hide Transformer
    Transformer->>Transformer: nodes([])
    CanvasRef->>Stage: toDataURL(options)
    Stage->>Stage: Render at 2x Resolution
    Stage->>Output: Generate PNG Data URL
    Output->>CanvasRef: Return Data URL
    CanvasRef->>Transformer: Restore Transformer
    CanvasRef->>Parent: Return Data URL
```

**Export Configuration**:
```typescript
{
  pixelRatio: 2,      // 2x resolution for high quality
  mimeType: 'image/png',
  quality: 1          // Maximum quality
}
```

### Export Features

- **High Resolution**: 2x pixel ratio for crisp output
- **Clean Export**: Transformer controls hidden during export
- **State Preservation**: Transformer restored after export
- **Format**: PNG with maximum quality
- **Transparency**: Supports transparent backgrounds

## Responsive Design

### Canvas Scaling System

```mermaid
graph TB
    subgraph "Container Dimensions"
        Container[Container Size]
        Padding[Padding: 40px]
        Available[Available Space]
    end
    
    subgraph "Canvas Dimensions"
        ConfigWidth[Config Width]
        ConfigHeight[Config Height]
    end
    
    subgraph "Scale Calculation"
        ScaleX["Scale X = Available Width / Config Width"]
        ScaleY["Scale Y = Available Height / Config Height"]
        FinalScale["Final Scale = minimum of scaleX, scaleY, 1"]
    end
    
    Container --> Available
    Padding --> Available
    
    Available --> ScaleX
    ConfigWidth --> ScaleX
    
    Available --> ScaleY
    ConfigHeight --> ScaleY
    
    ScaleX --> FinalScale
    ScaleY --> FinalScale
    
    FinalScale --> Stage[Apply to Stage]
    
    style FinalScale fill:#FFF3E0,stroke:#FF9800,stroke-width:2px
    style Stage fill:#E8F5E9,stroke:#4CAF50,stroke-width:2px
```

**Scaling Implementation**:
```typescript
useEffect(() => {
  if (containerRef.current) {
    const padding = 40;
    const containerWidth = containerRef.current.offsetWidth - padding;
    const containerHeight = containerRef.current.offsetHeight - padding;
    const scaleX = containerWidth / config.width;
    const scaleY = containerHeight / config.height;
    const newScale = Math.min(scaleX, scaleY, 1);
    setScale(newScale);
  }
}, [config.width, config.height]);
```

## User Interactions

### Interaction Flow Map

```mermaid
graph TB
    subgraph "Mouse Interactions"
        Click[Click]
        Drag[Drag]
        Transform[Transform Handles]
    end
    
    subgraph "Keyboard Interactions"
        Delete[Delete/Backspace]
    end
    
    subgraph "UI Controls"
        Upload[Upload Button]
        Export[Export Button]
        LayerPanel[Layer Controls Panel]
    end
    
    subgraph "Actions"
        SelectLayer[Select Layer]
        Deselect[Deselect]
        MoveLayer[Move Layer]
        ScaleRotate[Scale/Rotate Layer]
        DeleteLayer[Delete Layer]
        AddImage[Add Image]
        ExportImage[Export Image]
        AdjustFilters[Adjust Filters]
    end
    
    Click --> SelectLayer
    Click --> Deselect
    Drag --> MoveLayer
    Transform --> ScaleRotate
    Delete --> DeleteLayer
    Upload --> AddImage
    Export --> ExportImage
    LayerPanel --> AdjustFilters
    
    style SelectLayer fill:#E3F2FD,stroke:#2196F3,stroke-width:2px
    style MoveLayer fill:#E8F5E9,stroke:#4CAF50,stroke-width:2px
    style ScaleRotate fill:#FFF3E0,stroke:#FF9800,stroke-width:2px
```

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Delete` / `Backspace` | Delete selected layer |

### Drag and Drop Behavior

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> DragStart: Mouse Down on Layer
    DragStart --> Dragging: Mouse Move
    Dragging --> Dragging: Continue Moving
    Dragging --> DragEnd: Mouse Up
    DragEnd --> PositionChanged: Position Different
    DragEnd --> NoChange: Position Same
    PositionChanged --> Idle: Update State
    NoChange --> Idle: No Update
    
    note right of DragStart
        Store initial position
    end note
    
    note right of PositionChanged
        Call handleTransform
        Update layer state
    end note
```

## Performance Optimizations

### Optimization Strategies

```mermaid
graph LR
    subgraph "React Optimizations"
        Memo[React.memo]
        Callbacks[useCallback]
        MemoValues[useMemo]
        ImperativeHandle[useImperativeHandle]
    end
    
    subgraph "Konva Optimizations"
        PerfectDraw[perfectDrawEnabled: false]
        TransformEnabled[Conditional transformsEnabled]
        BatchDraw[Layer.batchDraw]
        ShouldOverdraw[shouldOverdrawWholeArea: false]
    end
    
    subgraph "State Optimizations"
        ImmutableUpdates[Immutable Updates]
        DerivedState[Derived State with useMemo]
        RefState[Refs for Non-Render State]
    end
    
    Memo --> Component[Canvas Component]
    Callbacks --> Handlers[Event Handlers]
    MemoValues --> SelectedLayer[Selected Layer]
    
    PerfectDraw --> ImageNodes[Image Nodes]
    TransformEnabled --> ImageNodes
    BatchDraw --> Transformer[Transformer Updates]
    
    style Memo fill:#E8F5E9,stroke:#4CAF50,stroke-width:2px
    style PerfectDraw fill:#E3F2FD,stroke:#2196F3,stroke-width:2px
```

**Key Optimizations**:

1. **Component Memoization**: Entire Canvas wrapped in `React.memo`
2. **Callback Memoization**: All event handlers use `useCallback`
3. **Derived State**: `selectedLayer` computed with `useMemo`
4. **Konva Performance**:
   - `perfectDrawEnabled: false` - Faster rendering
   - Conditional `transformsEnabled` - Only selected layer fully transformable
   - `shouldOverdrawWholeArea: false` - Optimized transformer rendering
5. **Ref Usage**: Non-render state (drag position) stored in refs

## Integration Points

### Type System Integration

The canvas-system heavily depends on types from the [type-system](type-system.md) module:

```mermaid
graph LR
    subgraph "Type System"
        Layer[Layer Type]
        CanvasConfig[CanvasConfig Type]
        Transform[Transform Type]
    end
    
    subgraph "Canvas System"
        Canvas[Canvas Component]
        State[Component State]
        Props[Component Props]
    end
    
    Layer --> State
    CanvasConfig --> Props
    Transform --> Handlers[Transform Handlers]
    
    style Layer fill:#E3F2FD,stroke:#2196F3,stroke-width:2px
    style CanvasConfig fill:#E3F2FD,stroke:#2196F3,stroke-width:2px
```

**Key Type Dependencies**:
- `Layer`: Defines the structure of each image layer with transform and filter properties
- `CanvasConfig`: Configures canvas dimensions and background
- Transform properties: Used in `handleTransform` operations

### External Library Dependencies

```mermaid
graph TB
    subgraph "Canvas System"
        Canvas[Canvas Component]
    end
    
    subgraph "Konva.js Ecosystem"
        Konva[Konva Core]
        ReactKonva[React-Konva]
        Stage[Stage Component]
        Layer[Layer Component]
        Image[Image Component]
        Transformer[Transformer Component]
        Filters[Konva Filters]
    end
    
    subgraph "React Ecosystem"
        React[React]
        Hooks[React Hooks]
    end
    
    subgraph "UI Library"
        AntD[Ant Design]
        Button[Button Component]
        Space[Space Component]
        Icons[Icons]
    end
    
    Canvas --> ReactKonva
    ReactKonva --> Konva
    Canvas --> Stage
    Canvas --> Layer
    Canvas --> Image
    Canvas --> Transformer
    Canvas --> Filters
    
    Canvas --> React
    Canvas --> Hooks
    
    Canvas --> AntD
    Canvas --> Button
    Canvas --> Icons
    
    style Canvas fill:#4A90E2,stroke:#2E5C8A,stroke-width:3px,color:#fff
```

## Usage Example

### Basic Usage

```typescript
import Canvas, { CanvasRef } from './components/Canvas/Canvas';
import { CanvasConfig } from './types';

function App() {
  const canvasRef = useRef<CanvasRef>(null);
  
  const config: CanvasConfig = {
    width: 1024,
    height: 768,
    background: '#ffffff'
  };
  
  const handleExport = () => {
    const dataUrl = canvasRef.current?.getDataUrl();
    if (dataUrl) {
      // Use the exported image
      console.log('Exported:', dataUrl);
    }
  };
  
  return (
    <div>
      <Canvas ref={canvasRef} config={config} />
      <button onClick={handleExport}>Export Canvas</button>
    </div>
  );
}
```

### Advanced Usage with AI Integration

```typescript
import Canvas, { CanvasRef } from './components/Canvas/Canvas';
import { generateImage } from './services/fal';

function AIImageEditor() {
  const canvasRef = useRef<CanvasRef>(null);
  
  const handleAIGenerate = async () => {
    // Get current canvas state
    const currentImage = canvasRef.current?.getDataUrl();
    
    if (currentImage) {
      // Send to AI service for processing
      const result = await generateImage({
        image_url: currentImage,
        // ... other AI parameters
      });
      
      // Result can be loaded back into canvas
      console.log('AI processed:', result);
    }
  };
  
  return (
    <div>
      <Canvas ref={canvasRef} config={config} />
      <button onClick={handleAIGenerate}>AI Enhance</button>
    </div>
  );
}
```

## Component Lifecycle

### Lifecycle Flow

```mermaid
sequenceDiagram
    participant Mount
    participant Canvas
    participant Effects
    participant Konva
    participant User
    
    Mount->>Canvas: Component Mounts
    Canvas->>Effects: Run Initial Effects
    Effects->>Canvas: Calculate Scale
    Effects->>Konva: Initialize Stage
    Canvas->>User: Ready for Interaction
    
    User->>Canvas: Upload Image
    Canvas->>Effects: Update Layers State
    Effects->>Konva: Render New Layer
    
    User->>Canvas: Select Layer
    Canvas->>Effects: Update Selection
    Effects->>Konva: Attach Transformer
    
    User->>Canvas: Transform Layer
    Canvas->>Effects: Update Layer State
    Effects->>Konva: Re-render Layer
    
    User->>Canvas: Export
    Canvas->>Konva: Generate Data URL
    Konva->>Canvas: Return Image Data
    Canvas->>User: Provide Export
```

### Effect Dependencies

```mermaid
graph TB
    subgraph "Effect 1: Transformer Attachment"
        Dep1[Dependencies: selectedId]
        Action1[Attach/Detach Transformer]
    end
    
    subgraph "Effect 2: Responsive Scaling"
        Dep2[Dependencies: config.width, config.height]
        Action2[Calculate and Apply Scale]
    end
    
    subgraph "Effect 3: Keyboard Shortcuts"
        Dep3[Dependencies: selectedId, handleDelete]
        Action3[Register/Unregister Listeners]
    end
    
    Dep1 --> Action1
    Dep2 --> Action2
    Dep3 --> Action3
    
    Action1 --> Render1[Trigger Re-render]
    Action2 --> Render2[Update Scale State]
    Action3 --> Cleanup[Cleanup on Unmount]
    
    style Action1 fill:#E3F2FD,stroke:#2196F3,stroke-width:2px
    style Action2 fill:#E8F5E9,stroke:#4CAF50,stroke-width:2px
    style Action3 fill:#FFF3E0,stroke:#FF9800,stroke-width:2px
```

## Error Handling and Edge Cases

### Handled Edge Cases

```mermaid
graph TB
    subgraph "Transform Validation"
        MinSize[Minimum Size Check]
        BoundBox[Bound Box Validation]
    end
    
    subgraph "Selection Safety"
        NullCheck[Null Selection Check]
        NodeExists[Node Existence Check]
    end
    
    subgraph "Drag Optimization"
        PositionChange[Position Change Detection]
        NoUpdate[Skip Update if No Change]
    end
    
    subgraph "Export Safety"
        StageExists[Stage Existence Check]
        TransformerHide[Hide Transformer Before Export]
        TransformerRestore[Restore After Export]
    end
    
    MinSize --> Safe1[Prevent Invalid Transforms]
    BoundBox --> Safe1
    
    NullCheck --> Safe2[Prevent Null Reference Errors]
    NodeExists --> Safe2
    
    PositionChange --> Safe3[Optimize Performance]
    NoUpdate --> Safe3
    
    StageExists --> Safe4[Ensure Clean Export]
    TransformerHide --> Safe4
    TransformerRestore --> Safe4
    
    style Safe1 fill:#E8F5E9,stroke:#4CAF50,stroke-width:2px
    style Safe2 fill:#E8F5E9,stroke:#4CAF50,stroke-width:2px
    style Safe3 fill:#E8F5E9,stroke:#4CAF50,stroke-width:2px
    style Safe4 fill:#E8F5E9,stroke:#4CAF50,stroke-width:2px
```

**Key Safety Checks**:

1. **Transform Bounds**: Prevents layers from being scaled below 5x5 pixels
2. **Null Safety**: All operations check for `selectedId` existence
3. **Node Validation**: Verifies Konva nodes exist before manipulation
4. **Drag Optimization**: Only updates state if position actually changed
5. **Export Safety**: Ensures transformer is hidden and stage exists

## Future Enhancements

### Potential Improvements

```mermaid
mindmap
  root((Canvas System<br/>Enhancements))
    Performance
      Virtual Layer Rendering
      Web Workers for Filters
      Canvas Caching
      Lazy Loading
    Features
      Undo/Redo System
      Layer Groups
      Blend Modes
      Custom Filters
      Text Layers
      Shape Tools
    UX
      Touch Gestures
      Keyboard Shortcuts
      Context Menus
      Drag to Upload
      Layer Thumbnails
    Integration
      Cloud Storage
      Real-time Collaboration
      AI Filter Presets
      Template System
```

### Recommended Additions

1. **History Management**: Implement undo/redo with command pattern
2. **Layer Groups**: Support hierarchical layer organization
3. **Blend Modes**: Add Photoshop-style blend modes
4. **Performance**: Implement virtual rendering for many layers
5. **Collaboration**: Add real-time multi-user editing
6. **Templates**: Pre-configured canvas templates
7. **Gestures**: Touch and multi-touch support for mobile
8. **Accessibility**: Keyboard navigation and screen reader support

## Related Modules

- **[type-system](type-system.md)**: Provides core type definitions (`Layer`, `CanvasConfig`, `Transform`)
- **ai-service**: Can consume canvas exports for AI processing (see `CanvasRef.getDataUrl()`)

## Technical Specifications

### Browser Compatibility

- **Modern Browsers**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Required APIs**: Canvas API, FileReader API, Blob API
- **Optional APIs**: Clipboard API (for future copy/paste)

### Performance Characteristics

- **Layer Limit**: Recommended max 50 layers for optimal performance
- **Image Size**: Supports images up to 8192x8192 pixels
- **Export Time**: ~100-500ms depending on canvas size and layer count
- **Memory Usage**: ~50-200MB depending on image sizes and layer count

### Dependencies

```json
{
  "react": "^18.x",
  "react-konva": "^18.x",
  "konva": "^9.x",
  "antd": "^5.x"
}
```

## Conclusion

The canvas-system module provides a robust, performant, and feature-rich image editing interface. Built on Konva.js and React, it offers professional-grade capabilities while maintaining excellent performance through careful optimization. The module's clean API design (via `CanvasRef`) makes it easy to integrate with other systems, particularly AI services that can process canvas exports. Its comprehensive layer management, real-time transformations, and filter system make it suitable for a wide range of image editing applications.
