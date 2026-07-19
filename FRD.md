# Functional Requirements Document (FRD)

## FrostPlan Pro: Visual Freezer Planogram Tool

**Version:** 1.0
**Date:** July 19, 2026
**Project Scope:** A single-page web application (SPA) designed to help grocery store planners visually arrange inventory within a top-down view of a freezer chest, outputting a printable schematic and stacking report.

---

## 1. Introduction

### 1.1 Purpose

The purpose of this document is to outline the functional and non-functional requirements for the FrostPlan Pro web application. This document is intended for developers, QA testers, and product managers to understand the expected behavior and scope of the software.

### 1.2 Target Audience

- Grocery Store Planners and Managers
- Merchandising Teams
- Inventory Analysts

### 1.3 Key Features

- Configurable physical dimensions and measurement units (inches/cm).
- Inventory import via CSV or auto-generation.
- Data table for inline editing of inventory specifications.
- Visual, drag-and-drop planning workspace.
- Automatic collision detection and boundary enforcement.
- Automatic vertical stacking calculation based on item depth.
- Print-optimized schematic and inventory report generation.
- Responsive design supporting desktop and touchscreen mobile devices.

---

## 2. System Architecture & Technology Stack

- **Architecture:** Single Page Application (SPA)
- **Core Library:** React 18 (Client-side rendering via Babel standalone for the current implementation)
- **Styling:** Tailwind CSS (via CDN)
- **Icons:** Lucide Icons (via CDN)
- **Data Storage:** Transient (In-memory component state). No backend database is currently implemented.

---

## 3. Functional Requirements

### 3.1 Setup Phase (Configuration & Import)

#### REQ-1.1: Measurement Unit Selection

- The system shall allow the user to select the primary unit of measurement: Inches (in) or Centimeters (cm).
- Changing this setting shall update all UI labels accordingly.

#### REQ-1.2: Freezer Dimension Configuration

- The system shall require the user to input the internal dimensions of the freezer: Length (horizontal axis), Width (vertical axis), and Depth (vertical stacking axis).
- Inputs must accept positive numerical values.

#### REQ-1.3: Inventory Data Ingestion

- **REQ-1.3.1 (Auto-Generate):** The system shall provide a mechanism to populate a sample inventory dataset for testing purposes. This dataset must include predefined items with images, shapes (box/cylinder), and dimensions.
- **REQ-1.3.2 (CSV Import):** The system shall accept manual CSV text input.
  - Expected headers: `Name, SKU, Shape, L, W, D, Radius, ImgURL`
  - The system shall parse the CSV and map it to the inventory state.
  - The system shall display error messages for malformed data or missing required fields (e.g., Box requires L, W, D; Cylinder requires Radius, D).

#### REQ-1.4: Inventory Data Editor (Table)

- Upon loading inventory, the system shall display the data in a tabular format.
- The system shall allow inline editing of SKU, Name, Shape, Dimensions (L/W/D/Radius).
- The system shall allow the removal of individual items from the inventory list.
- The system shall allow clearing the entire inventory list.
- The system shall automatically assign a fallback color if no image is provided.

#### REQ-1.5: Validation to Proceed

- The "Start Planning" action shall remain disabled until valid freezer dimensions (L>0, W>0, D>0) are entered and at least one item exists in the inventory.

### 3.2 Planning Phase (Workspace)

#### REQ-2.1: UI Layout & Responsiveness

- The workspace shall consist of two main areas: an Inventory Panel and a Freezer Canvas.
- **Desktop:** Inventory panel on the left (vertical scroll), Freezer Canvas on the right.
- **Mobile (Touch):** Inventory panel on the top (horizontal scroll), Freezer Canvas below.

#### REQ-2.2: Inventory Panel

- The panel shall display all available inventory items as discrete cards.
- Cards shall display: Image/Color, Name, SKU, Dimensions, and the calculated "Max Stack" value.
  - Max Stack calculation: `Floor(Freezer Depth / Item Depth)`
- Cards shall inherit the background color tint and border color assigned to the item to match its on-canvas representation.

#### REQ-2.3: Freezer Canvas (Top-Down View)

- The canvas shall render a rectangular container maintaining an aspect ratio corresponding to the inputted Freezer Length and Width.
- The canvas shall display a visual grid pattern as a scaling aid.
- The canvas shall feature a static label indicating "Customer Stands Here ↓" to define orientation.

#### REQ-2.4: Drag and Drop Interactions

- **REQ-2.4.1 (Initiation):** Users shall be able to initiate a drag operation from the Inventory Panel or by selecting an already-placed item on the canvas.
- **REQ-2.4.2 (Ghost Cursor):** While dragging, a scaled visual representation (ghost) of the item shall follow the user's pointer (mouse or touch).
- **REQ-2.4.3 (Placement):** Releasing the drag within the canvas bounds shall drop the item at that location.
- **REQ-2.4.4 (Touch Support):** All drag operations must support capacitive touch events (`touchstart`, `touchmove`, `touchend`) without triggering default browser scrolling behaviors.

#### REQ-2.5: Spatial Logic & Collision Detection

- **REQ-2.5.1 (Boundary Enforcement):** The system shall prevent items from being placed if any part of the item's bounding box exceeds the freezer canvas boundaries.
- **REQ-2.5.2 (Overlap Prevention):** The system shall utilize geometric bounding box collision detection (AABB) to prevent an item from being dropped if it overlaps with any previously placed item.
- **REQ-2.5.3 (Visual Feedback):** During a drag, the item ghost shall turn Red if the current position is invalid (collision/out-of-bounds) and Green (or its native color) if valid.
- **REQ-2.5.4 (Shape Handling):** Bounding boxes for "Cylinders" shall be calculated dynamically as a square with width/height equal to `Radius * 2`.

#### REQ-2.6: Placed Item Management

- Placed items shall display their SKU, stack count, and a background image/color.
- Placed items shall feature an explicit "X" button in the top right corner to remove them from the canvas. Clicking/tapping this button must not trigger a drag operation.

#### REQ-2.7: Session Management

- The system shall provide a "Start Over" button.
- Clicking "Start Over" shall prompt an inline confirmation UI (Yes/No) to prevent accidental data loss.
- Confirming "Start Over" shall clear all inventory and placed items and return the user to the Setup Phase.

### 3.3 Print / Export Phase

#### REQ-3.1: Print Layout Mode

- Entering the "Print" phase shall re-render the DOM to prioritize printable content.
- All interactive UI elements (navigation buttons, "X" delete buttons, drag handlers) shall be hidden or disabled.
- The system shall automatically trigger the browser's native `window.print()` dialog after a short delay (e.g., 500ms) upon entering this view.

#### REQ-3.2: Report Content

- **REQ-3.2.1 (Header):** Must include a title, generation date, freezer specifications (L x W x D), total unique SKUs, and total unit capacity.
- **REQ-3.2.2 (Schematic Diagram):** Must render a static, scaled version of the Freezer Canvas showing the exact placement of all items, matching the planning view. Background colors and images must be configured to force printing (e.g., `WebkitPrintColorAdjust: 'exact'`).
- **REQ-3.2.3 (Inventory Table):** Must generate an aggregated list of all items currently placed on the canvas.
- **REQ-3.2.4 (Aggregations):** The table must calculate and display:
  - Number of discrete placements (footprints) for each SKU.
  - Max stack size for that SKU.
  - Total Quantity: `(Number of Placements) * (Max Stack Size)`
  - A Grand Total of all units at the bottom of the table.

---

## 4. Non-Functional Requirements

- **Usability:** The interface must be intuitive, requiring no specialized training. Drag operations must feel responsive.
- **Performance:** Collision detection algorithms must run efficiently within the `mousemove` / `touchmove` event loops without causing UI stuttering.
- **Compatibility:** Must function on modern browsers (Chrome, Firefox, Safari, Edge) on desktop and mobile operating systems.
- **State Management:** As a client-side only application, refreshing the page will result in a total loss of data. (Future enhancement consideration: LocalStorage implementation.)

---

## 5. Future Enhancement Considerations

- **Save/Load Functionality:** Implement localStorage or a JSON export/import to save planogram layouts between sessions.
- **Rotation:** Allow items (specifically boxes) to be rotated 90 degrees to optimize packing efficiency.
- **Variable Stacking:** Allow users to override the "Max Stack" with a custom number (e.g., placing a smaller box on top of a larger one, if spatial logic permits).
- **Snap-to-Grid:** Implement an optional magnetic grid to align items perfectly.
