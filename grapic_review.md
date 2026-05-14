# Graphics Project Review

## 1. The Big Picture

This project is a Windows desktop graphics demo built with classic Win32 tools. It is not using OpenGL or Direct3D; instead it uses the Windows GDI API to draw directly on a window. The program lets you choose between many drawing and fill techniques, then click inside the window to place shapes, curves, clipping examples, and smiley faces.

In plain terms, the application is a drawing sandbox for teaching computer graphics algorithms. The menu lets you pick a drawing method, then the program records mouse clicks and turns them into pixels, lines, circles, ellipses, fills, and clipped shapes.

The main idea is:
- The user chooses a tool from menus.
- The program listens to mouse clicks.
- Each click or pair of clicks becomes a shape or operation.
- The shape is drawn immediately and also stored in memory.
- When the window needs to refresh, the program redraws everything from that stored list.

## 2. Architecture & Data Flow

### Program start and event loop

The program starts in `main.cpp`, inside the `WinMain` function. That function:
- creates a window class,
- sets up a window procedure (`WndProc`),
- builds the menu,
- enters a Win32 message loop.

The message loop is the central control flow:
- Windows sends events such as `WM_PAINT`, `WM_LBUTTONDOWN`, `WM_RBUTTONDOWN`, and `WM_COMMAND`.
- `WndProc` handles those events.

### Main interaction flow

1. **User chooses a tool** from the menu. The program updates `g_currentSelection` and, for ellipses, `g_currentType`.
2. **User clicks in the window.** The code handles clicks in `WM_LBUTTONDOWN` and sometimes `WM_RBUTTONDOWN`.
3. **Shape parameters are built** from the collected click points.
4. **The selected drawing function is called** to draw immediately using a device context (`HDC`).
5. **A `ShapeRecord` is created** and stored in `g_shapes`.
6. **If the window needs redrawing**, `WM_PAINT` calls `RedrawShapes` to redraw all stored shapes from `g_shapes`.

This means the program has two related flows:
- immediate drawing to the screen,
- persistent shape storage for redraw.

### Data movement between files

- `main.cpp` decides what to draw and stores shape metadata.
- `task1_file_menu.cpp` manages the global shape list, saving, loading, clearing, and repainting.
- `Lines.cpp`, `Circles.cpp`, `task5_ellipse_algorithms.cpp`, `Filling.cpp`, `Clipping.cpp`, `Curves.cpp`, and `SmileyFace.cpp` each implement real drawing algorithms.
- `Preferences.cpp` manages color and cursor settings.

### Global data structure

`task1_file_menu.h` defines `ShapeRecord` and the `ShapeType` enum.
- `ShapeRecord` stores the shape kind, two coordinate pairs, a color, and an optional list of points.
- `g_shapes` is the global list of all shapes.
- `RedrawShapes` loops over `g_shapes` and redraws each shape using the appropriate algorithm.

## 3. Graphics Primer

Here are the top concepts for this project, explained with simple analogies.

### 1. Pixels and raster rendering

Think of the screen as a grid of tiny squares, like a sheet of graph paper. Each square is a pixel. The code does not use cameras or shaders; it paints individual pixels with `SetPixel` or draws lines with `MoveToEx`/`LineTo`.

- `SetPixel(hdc, x, y, color)` is like coloring a single square on graph paper.
- `HDC` is like the paintbrush and paint surface together: it is a handle to the place where drawing happens.

Why it matters: Every line, circle, and curve is broken down into pixels, and the job of the algorithms is to decide which pixels to color.

### 2. Rendering vs. drawing

Rendering is the process of turning shape descriptions into visible pixels. In this project, rendering happens whenever the program draws a line or circle and whenever `WM_PAINT` forces redraw.

- Draw once: a function like `DrawLineDDA` renders a line right away.
- Redraw later: `RedrawShapes` re-renders the stored shapes when the window is refreshed.

### 3. Device context (`HDC`) and the framebuffer

Imagine `HDC` as a painter’s workspace. It represents the window’s drawing surface and the current settings.

- The `HDC` is not the actual screen itself; it is a handle to a place where Windows can draw.
- The real framebuffer is the memory behind the window that stores the colored pixels until Windows shows them on screen.

### 4. Line and circle algorithms

This project teaches the idea that simple math can define complex shapes.

- A line is defined by two points. Algorithms like DDA, midpoint, and parametric decide which pixels best approximate the line.
- A circle is defined by a center and radius. The code uses direct equation methods, polar coordinates, iterative rotation, and midpoint decision methods.

Analogy: If you want to draw a curved path on graph paper, you can either connect points step-by-step or calculate each coordinate using a formula.

### 5. Clipping and filling

Clipping is like putting a stencil over your drawing and only allowing marks inside the holes.

- Rectangle clipping hides or removes pixels outside a rectangular stencil.
- Circle clipping hides everything outside a circular stencil.

Filling is like coloring the inside of a shape.

- Scanline filling colors every gap between the left and right edges on each row.
- Flood fill is like pouring paint into a closed region and letting it expand until it hits a border.

## 4. File-by-File Breakdown

### `main.cpp`
Primary purpose:
- Launches the Windows application.
- Defines the menu structure.
- Handles user input and mouse events.
- Constructs shapes from mouse clicks.
- Calls drawing functions and stores shapes.
- Redraws shapes on `WM_PAINT`.

Important elements:
- `WinMain` sets up the window and menu.
- `WndProc` handles `WM_PAINT`, `WM_LBUTTONDOWN`, `WM_RBUTTONDOWN`, and `WM_COMMAND`.
- `CreateAppMenu` builds the menu with options for ellipses, circles, lines, fills, clipping, curves, smiley faces, and preferences.
- The mouse click logic decides whether two clicks make a line, circle, ellipse, or fill boundary.

### `task1_file_menu.h` and `task1_file_menu.cpp`
Primary purpose:
- Define the shared shape data structure and store all shapes.
- Save and load shapes from disk.
- Clear the drawing and redraw stored shapes.

What it does:
- `g_shapes` is the global persistent list.
- `ShapeRecord` stores the shape type, coordinates, color, and additional points.
- `SaveToFile` writes shape data to a `.shp` file.
- `LoadFromFile` reads shape data back into `g_shapes`.
- `RedrawShapes` replays every shape when the window refreshes.

### `task5_ellipse_algorithms.h` and `task5_ellipse_algorithms.cpp`
Primary purpose:
- Implement ellipse drawing algorithms.

What it does:
- `DrawEllipseDirect` uses the ellipse equation and symmetry.
- `DrawEllipsePolar` uses angle steps and cosine/sine.
- `DrawEllipseMidpoint` uses a decision parameter to choose the next pixel.

Why this is interesting:
- The code teaches that ellipses can be drawn in different ways to trade off simplicity and smoothness.

### `Circles.h` and `Circles.cpp`
Primary purpose:
- Implement multiple circle drawing methods.

What it does:
- `CircleDirect` uses `x^2 + y^2 = R^2` and symmetry.
- `CirclePolar` uses `R*cos(theta)` and `R*sin(theta)`.
- `CircleIterativePolar` avoids repeated trig calculations.
- `CircleMidpoint` uses a midpoint decision strategy.
- `CircleModifiedMidpoint` uses an optimized incremental update.

### `Lines.h` and `Lines.cpp`
Primary purpose:
- Implement line drawing algorithms.

What it does:
- `DrawLineDDA` steps incrementally from one end to the other.
- `DrawLineMidpoint` uses an integer error term to choose the next pixel.
- `DrawLineParametric` computes each point from a parameter `t`.

Why it matters:
- These algorithms illustrate how a straight mathematical line gets converted into grid pixels.

### `Filling.h` and `Filling.cpp`
Primary purpose:
- Implement shape filling and curve-based fill effects.

What it does:
- `FillCircleWithLines` fills a circle by scanning horizontal lines inside it.
- `FillCircleWithCircles` fills a quarter circle with concentric partial circles.
- `FillSquareHermite` uses Hermite curve math to fill a square.
- `FillRectBezier` uses Bezier curve math to fill a rectangle.
- `FillConvexPolygon` and `FillNonConvexPolygon` use scanline polygon filling.
- `FloodFillRecursive` and `FloodFillIterative` fill areas by expanding from a seed point.

Important notes:
- Flood fill is like the paint-bucket tool in drawing apps.
- Scanline fill is like coloring row by row across a shape.

### `Clipping.h` and `Clipping.cpp`
Primary purpose:
- Implement clipping algorithms for points, lines, and polygons.

What it does:
- `ClipPointRect` and `ClipPointCircle` test whether a point lies inside a region.
- `ClipLineRect` implements the Cohen-Sutherland line clipping algorithm.
- `ClipLineCircle` finds the part of a line inside a circle using a quadratic equation.
- `PolygonClip` uses Sutherland-Hodgman polygon clipping to trim polygon edges.

Why it matters:
- Clipping is the process of only drawing what is visible inside a shape or window.
- This is a foundational graphics concept because real systems often only render what the camera can see.

### `Curves.h` and `Curves.cpp`
Primary purpose:
- Implement a Cardinal spline curve.

What it does:
- `DrawCardinalSpline` builds a smooth curve through control points.
- It uses cubic Hermite basis functions and automatically computes tangents.

Why it matters:
- This shows how smooth curves are generated from a few points.
- The idea is that a curve can be defined by control points and a tension parameter.

### `SmileyFace.h` and `SmileyFace.cpp`
Primary purpose:
- Draw smiley face icons.

What it does:
- `DrawSmiley` draws a face outline, eyes, a nose, and a mouth.
- Uses circle drawing and Win32 arc/line primitives.

This module is mainly a fun demonstration of combining shapes.

### `Preferences.h` and `Preferences.cpp`
Primary purpose:
- Offer UI preferences for background, cursor, and drawing color.

What it does:
- `PrefSetWhiteBackground` changes the window background to white.
- `PrefSetCursor` changes the cursor shape.
- `PrefChooseColor` opens the standard Windows color picker.

### Binaries and extras

- `GraphicsProject.exe` and `main.exe` are built executables; they are not source code.
- `Graphics_2025_20206_Project.pdf` is likely a project specification or report and is not part of the source logic.

## 5. Key Functions & Logic

### `WinMain` in `main.cpp`
Intent:
- Start the app and show the main window.
- Register a window class with a default white background.
- Build the menu and display the window.
- Enter the Windows message loop.

Why it matters:
- This is the entry point that makes the program a Windows GUI application.

### `WndProc` in `main.cpp`
Intent:
- Respond to events from Windows.
- Draw shapes when `WM_PAINT` occurs.
- Receive clicks and turn them into drawing commands.
- Handle menu commands.

Important logic blocks:
- `WM_PAINT`: calls `RedrawShapes(hdc)`.
- `WM_LBUTTONDOWN`: collects clicks and decides what to draw based on the current menu selection.
- `WM_RBUTTONDOWN`: finalizes multi-click operations like polygon fill, cardinal spline, and clipping.
- `WM_COMMAND`: switches the current tool or updates preferences.

How that relates to graphics:
- Mouse events become shape parameters.
- The program chooses the correct algorithm and draws directly to the window.
- Shapes are remembered so the image persists during refresh.

### `RedrawShapes` in `task1_file_menu.cpp`
Intent:
- Repaint the complete content from stored shape records.

How it works:
- Loops over `g_shapes`.
- For each shape, runs the appropriate drawing algorithm.
- Reconstructs curves, fills, clipped shapes, and smiley faces.

Why it matters:
- Windows may ask the program to repaint at any time.
- Without this function, the drawing would disappear when the window is minimized, resized, or covered.

### `DrawLineDDA`, `DrawLineMidpoint`, `DrawLineParametric` in `Lines.cpp`
Intent:
- Draw lines from point A to point B using different pixel-selection strategies.

How they differ:
- DDA: moves in small equal steps, like walking from one end to the other.
- Midpoint: uses error tracking to decide whether to step in x or y, like choosing the best next pixel.
- Parametric: uses a parameter `t` from 0 to 1 and calculates the exact position along the line.

Why it matters:
- These functions show the fundamental graphics problem of converting a continuous line into discrete pixels.

### `CircleDirect`, `CirclePolar`, `CircleMidpoint`, `CircleIterativePolar`, `CircleModifiedMidpoint` in `Circles.cpp`
Intent:
- Draw a circle using different mathematical methods.

How they differ:
- Direct: plugs pixel x into `x^2 + y^2 = R^2`.
- Polar: computes coordinates by angle.
- Iterative polar: rotates a point step-by-step.
- Midpoint: uses a decision variable to choose between two next pixels.
- Modified midpoint: refines the decision for efficiency.

Why it matters:
- Circles are a classic example of how geometry becomes pixel drawing.
- Symmetry is used heavily: the code draws eight mirror points at once.

### `DrawEllipseDirect`, `DrawEllipsePolar`, `DrawEllipseMidpoint` in `task5_ellipse_algorithms.cpp`
Intent:
- Draw ellipses with different mathematical strategies.

How they work:
- Direct uses the ellipse equation and draws symmetric points.
- Polar uses trigonometry to compute `(x,y)` from an angle.
- Midpoint uses region-based decision logic to move from one pixel step to the next.

Why it matters:
- Ellipses are a generalization of circles, and the code shows how the same ideas extend to stretched circles.

### `FillCircleWithLines`, `FillCircleWithCircles`, `FillSquareHermite`, `FillRectBezier` in `Filling.cpp`
Intent:
- Fill interior areas with different visual styles.

How they differ:
- Circle with lines: fills a circle by checking each possible point inside the radius.
- Circle with circles: draws many smaller concentric circles.
- Hermite/Bezier filling: draws curve-guided shapes.

Why it matters:
- Filling is not just drawing outlines; it is about covering an area.
- These functions also introduce basic curve math.

### `FillConvexPolygon`, `FillNonConvexPolygon`, `FloodFillRecursive`, `FloodFillIterative`
Intent:
- Fill polygons and areas.

How they work:
- Scanline polygon fill: for each row, find where the polygon intersects that row and fill between intersections.
- Flood fill: start at a point and expand until a boundary is reached.

Why it matters:
- These are two core fill strategies used in graphics programs.
- The code demonstrates both row-by-row filling and region-growing filling.

### `ClipLineRect`, `ClipLineCircle`, `PolygonClip` in `Clipping.cpp`
Intent:
- Restrict drawing to a visible region.

How they work:
- Rectangle line clipping uses Cohen-Sutherland.
- Circle clipping uses geometry and a quadratic equation.
- Polygon clipping trims a polygon edge by edge.

Why it matters:
- This is the first step toward understanding how real renderers hide what is outside the view.
- Clipping reduces wasted drawing work and avoids drawing outside the allowed area.

### `DrawCardinalSpline` in `Curves.cpp`
Intent:
- Draw a smooth curve through control points using Cardinal spline math.

How it works:
- It creates phantom endpoints for smooth beginning and end.
- It computes tangent vectors and cubic Hermite basis values.
- It plots a pixel for each sample along the curve.

Why it matters:
- This shows a basic curve generation technique used in computer graphics.

### `DrawSmiley` in `SmileyFace.cpp`
Intent:
- Render a simple happy or sad face.

How it works:
- Draws a large circle for the face.
- Draws two smaller circles for eyes.
- Draws a nose with a line.
- Uses `Arc` for a smile or frown.

Why it matters:
- It combines primitive shapes into a composite drawing.

### Preferences functions
Intent:
- Change the UI environment.

What they do:
- `PrefSetWhiteBackground` changes the window background.
- `PrefSetCursor` changes the mouse cursor shape.
- `PrefChooseColor` opens the color picker.

Why it matters:
- These functions are not core graphics algorithms, but they make the application easier to use.

## Final Notes

- This project is a good example of a classic graphics teaching application.
- It stays at the level of pixels and simple shapes rather than modern GPU shader pipelines.
- The most important idea is the separation of user input, shape storage, and redraw logic.
- If you want to refer to a specific concept in the discussion, the strongest ones are:
  - rasterization (`SetPixel`, `MoveToEx`/`LineTo`),
  - shape algorithms for lines/circles/ellipses,
  - clipping (rectangle/circle),
  - filling strategies,
  - and the redraw architecture using `WM_PAINT` and a persistent `g_shapes` list.

  ## 6. Line-by-Line Deep Dive — `main.cpp`

  I'll break `main.cpp` into small chunks and explain each line or short block in plain English. Whenever the code is touching a core graphics concept I will remind you of the simple analogy from the Graphics Primer ("graph paper" pixels, "painter's workspace" HDC, "stencil" clipping, etc.).

  ### File: `main.cpp` — top-level header, macros, and includes
  **Intent:** Explain file-wide settings, includes, and the initial menu comment that documents UI layout.

  ```cpp
  /* Menu layout:
     File          | Clear / Save / Load / Exit
     Task 5        | Direct / Polar / Midpoint Ellipse      (1 click)
     Circles       | Direct / Polar / Iterative / Midpoint / Modified   (2 clicks)
     Lines         | DDA / Midpoint / Parametric            (2 clicks)
     Filling       | Circle-Lines / Circle-Circles          (1 click + quarter dialog)
          | Square-Hermite / Rect-Bezier           (2 clicks)
           | Convex / Non-Convex polygon            (N clicks + Right-click)
           | Flood Fill Recursive / Iterative       (1 click)
     Curves        | Cardinal Spline                        (N clicks + Right-click)
     Clipping      | Rectangle / Square / Circle Window
     Smiley Faces  | Happy / Sad
     Preferences   | White Background / Change Cursor / Choose Color*/
  ```
  Explanation:
  - This block is a multi-line comment describing the program menu and how many mouse clicks each tool expects. It is documentation only and does not affect runtime.

  ```cpp
  #define _UNICODE
  #define UNICODE
  ```
  Explanation:
  - These preprocessor defines tell the Windows headers to expose Unicode (wide-character) APIs. They ensure Windows strings and APIs use `wchar_t` variants. Not graphics-specific—it's about text encoding.

  ```cpp
  #include <tchar.h>
  #include <vector>
  #include <cmath>
  #include <windows.h>
  #include <algorithm>
  ```
  Explanation:
  - `#include <tchar.h>`: portable character macros (helps if code used ANSI vs Unicode).
  - `#include <vector>`: brings in `std::vector` used for dynamic arrays (like lists of points).
  - `#include <cmath>`: math functions (sqrt, pow, cos, sin).
  - `#include <windows.h>`: the Win32 API header — provides `HWND`, `HDC`, GDI functions like `SetPixel`, `MoveToEx`, `LineTo`, and message constants (`WM_PAINT`, etc.). This is the bridge to the low-level drawing surface (HDC) — recall the painter's workspace analogy.
  - `#include <algorithm>`: utilities like `std::min`, `std::max`, `std::sort`.

  ```cpp
  #include "task1_file_menu.h"
  #include "task5_ellipse_algorithms.h"
  #include "Circles.h"
  #include "Clipping.h"
  #include "SmileyFace.h"
  #include "Lines.h"
  #include "Filling.h"
  #include "Curves.h"
  #include "Preferences.h"
  ```
  Explanation:
  - These bring in the project's headers so `main.cpp` can call the functions declared in them (draw circles, lines, handle saving/loading, etc.). They connect the central event loop to the algorithm implementations.

  ### File: `main.cpp` — Menu ID definitions
  **Intent:** Define integer IDs for each menu command used in `WM_COMMAND` handling.

  ```cpp
  // Menu IDs

  // File
  #define IDM_FILE_CLEAR      1001
  #define IDM_FILE_SAVE       1002
  #define IDM_FILE_LOAD       1003
  #define IDM_FILE_EXIT       1004
  ```
  Explanation:
  - Each `#define` assigns a numeric constant to a menu action. When the user selects a menu item, Windows sends `WM_COMMAND` with the ID; the code uses these constants to decide which action to run.

  ```cpp
  // Ellipses
  #define IDM_T5_DIRECT       2001
  #define IDM_T5_POLAR        2002
  #define IDM_T5_MIDPOINT     2003

  // Circles 
  #define IDM_CIRC_DIRECT     3001
  #define IDM_CIRC_POLAR      3002
  #define IDM_CIRC_IPOLAR     3003
  #define IDM_CIRC_MIDPOINT   3004
  #define IDM_CIRC_MOD_MID    3005

  // Clipping
  #define IDM_CLIP_RECT       4001
  #define IDM_CLIP_SQUARE     4002
  #define IDM_CLIP_CIRCLE     4003
  ```
  Explanation:
  - Continued menu ID definitions. Numbers are grouped in ranges per feature (2000s for ellipses, 3000s for circles, etc.) to keep them organized.

  ```cpp
  // Smiley Faces
  #define IDM_FACE_HAPPY      5001
  #define IDM_FACE_SAD        5002

  // Lines
  #define IDM_LINE_DDA        6001
  #define IDM_LINE_MIDPOINT   6002
  #define IDM_LINE_PARAMETRIC 6003
  ```
  Explanation:
  - More menu IDs for faces and line-drawing modes.

  ```cpp
  // Filling
  #define IDM_FILL_SQ_HERMITE   7003
  #define IDM_FILL_RECT_BEZIER  7004
  #define IDM_FILL_CONVEX       7005
  #define IDM_FILL_NONCONVEX    7006
  #define IDM_FILL_FLOOD_REC    7007
  #define IDM_FILL_FLOOD_ITER   7008
  #define IDM_FILL_CIRC_LINES_Q1   7011
  #define IDM_FILL_CIRC_LINES_Q2   7012
  #define IDM_FILL_CIRC_LINES_Q3   7013
  #define IDM_FILL_CIRC_LINES_Q4   7014
  #define IDM_FILL_CIRC_CIRC_Q1   7015
  #define IDM_FILL_CIRC_CIRC_Q2   7016
  #define IDM_FILL_CIRC_CIRC_Q3   7017
  #define IDM_FILL_CIRC_CIRC_Q4   7018
  ```
  Explanation:
  - IDs for many filling variants. The `_Q1..Q4` values indicate quarter-selection submenus used by fill-with-lines / fill-with-circles.

  ```cpp
  // Curves
  #define IDM_CURVE_CARDINAL    8001

  // Preferences
  #define IDM_PREF_BG_WHITE       9001
  #define IDM_PREF_CURSOR_ARROW   9002
  #define IDM_PREF_CURSOR_CROSS   9003
  #define IDM_PREF_CURSOR_HAND    9004
  #define IDM_PREF_CURSOR_IBEAM   9005
  #define IDM_PREF_COLOR          9006
  ```
  Explanation:
  - IDs for curve tools and preference options (background, cursor, color).

  ### File: `main.cpp` — Global state variables
  **Intent:** Track the current tool selection, collected mouse clicks, current shape type, drawing color, and fill-quarter.

  ```cpp
  // Global state
  static int  g_currentSelection = IDM_T5_DIRECT;
  static std::vector<Point> g_mouseClicks;
  static ShapeType g_currentType = ShapeType::ELLIPSE_DIRECT;
  ```
  Explanation:
  - `g_currentSelection`: stores the active menu ID (initially set to `IDM_T5_DIRECT` — the direct ellipse tool). Used inside `WndProc` to decide how to interpret mouse clicks.
  - `g_mouseClicks`: dynamic list collecting mouse click positions as `Point` instances. Many tools need multiple clicks for parameters (center + radius, polygon vertex list, etc.).
  - `g_currentType`: stores which `ShapeType` to record when creating `ShapeRecord`s — important because ellipses need both a selection id and a stored type for redraw.

  ```cpp
  // Active drawing colour 
  static COLORREF g_drawColor = RGB(0, 0, 0);

  // Quarter for circle-fill functions (1=top-right, 2=top-left, 3=bottom-left, 4=bottom-right)
  static int g_fillQuarter = 1;
  ```
  Explanation:
  - `g_drawColor` stores the current drawing color as a `COLORREF` (RGB). `RGB(0,0,0)` is black by default.
  - `g_fillQuarter` chooses which quarter is filled for certain fill functions (used by menu choices and `FillCircleWithLines`/`FillCircleWithCircles`).

  ### Function: CreateAppMenu
  **Intent:** Build the Win32 menu bar and submenus; returns an `HMENU` handle that gets attached to the window.

  ```cpp
  static HMENU CreateAppMenu()
  {
    HMENU hMenuBar = CreateMenu();
  ```
  Explanation:
  - `static HMENU CreateAppMenu()`: defines a helper function that constructs the application's menu.
  - `CreateMenu()` returns a new empty menu handle (`hMenuBar`) — the top-level menu bar.

  ```cpp
    //  File 
    HMENU hFile = CreatePopupMenu();
    AppendMenu(hFile, MF_STRING,    IDM_FILE_CLEAR, L"&Clear");
    AppendMenu(hFile, MF_STRING,    IDM_FILE_SAVE,  L"&Save...");
    AppendMenu(hFile, MF_STRING,    IDM_FILE_LOAD,  L"&Load...");
    AppendMenu(hFile, MF_SEPARATOR, 0,              NULL);
    AppendMenu(hFile, MF_STRING,    IDM_FILE_EXIT,  L"E&xit");
    AppendMenu(hMenuBar, MF_POPUP, (UINT_PTR)hFile, L"&File");
  ```
  Explanation:
  - `CreatePopupMenu()` creates a submenu handle `hFile`.
  - `AppendMenu(...)` adds items: Clear, Save..., Load..., a separator, and Exit. Each call attaches a string label and an ID constant — when selected, `WM_COMMAND` receives the ID.
  - `AppendMenu(hMenuBar, MF_POPUP, (UINT_PTR)hFile, L"&File")` inserts the `hFile` popup into the menubar under the "File" label.

  ```cpp
    // Ellipses 
    HMENU hT5 = CreatePopupMenu();
    AppendMenu(hT5, MF_STRING, IDM_T5_DIRECT,   L"Select Direct Ellipse");
    AppendMenu(hT5, MF_STRING, IDM_T5_POLAR,    L"Select Polar Ellipse");
    AppendMenu(hT5, MF_STRING, IDM_T5_MIDPOINT, L"Select Midpoint Ellipse");
    CheckMenuItem(hT5, IDM_T5_DIRECT, MF_CHECKED);
    AppendMenu(hMenuBar, MF_POPUP, (UINT_PTR)hT5, L"Ellipses");
  ```
  Explanation:
  - Builds the "Ellipses" submenu with three options tied to the earlier IDs.
  - `CheckMenuItem` pre-checks the `IDM_T5_DIRECT` item to show it's the default selected tool.
  - All menu creation is UI wiring — not directly graphics, but selecting items affects which drawing algorithms are used.

  <!-- The file creates many more submenus (Circles, Lines, Filling, Curves, Clipping, Smiley Faces, Preferences). Each uses the same pattern: CreatePopupMenu + AppendMenu (possibly nested for quarters). -->

  ### Function: WndProc
  **Intent:** Main Windows procedure handling painting, mouse events, menu commands, and closing.

  ```cpp
  LRESULT WINAPI WndProc(HWND hwnd, UINT mcode, WPARAM wp, LPARAM lp)
  {
    HDC hdc;
  ```
  Explanation:
  - `WndProc` is the callback Windows calls for window messages. `hwnd` is the window handle, `mcode` is the message code (`WM_PAINT`, `WM_LBUTTONDOWN`, etc.), `wp`/`lp` are message-specific parameters.
  - `HDC hdc;` declares a handle to the device context that will be used for drawing. This is the painter's workspace handle — when we draw pixels or lines we use an `HDC`.

  ```cpp
    switch (mcode)
    {
    case WM_PAINT:
    {
      PAINTSTRUCT ps;
      hdc = BeginPaint(hwnd, &ps);
      RedrawShapes(hdc);
      EndPaint(hwnd, &ps);
      break;
    }
  ```
  Explanation (line-by-line):
  - `case WM_PAINT:`: Windows asks the program to repaint all/some of the client area (e.g., window uncovered or resized).
  - `PAINTSTRUCT ps;` - structure filled by `BeginPaint` with clipping/invalid rect info.
  - `hdc = BeginPaint(hwnd, &ps);` obtains the `HDC` for painting and locks the invalidated region for drawing. This must be paired with `EndPaint`.
  - `RedrawShapes(hdc);` calls the central redraw routine that iterates `g_shapes` and re-renders everything from the stored state. This is the "replay" step that ensures the image persists — recall the redraw architecture analogy.
  - `EndPaint(hwnd, &ps);` signals painting is done and releases the `HDC`.
  - `break;` exits the switch case.

  ```cpp
    //  Left click 
    case WM_LBUTTONDOWN:
    {
      int mx = LOWORD(lp);
      int my = HIWORD(lp);
      hdc = GetDC(hwnd);
  ```
  Explanation:
  - `case WM_LBUTTONDOWN:`: left mouse button was pressed.
  - `int mx = LOWORD(lp); int my = HIWORD(lp);` extract the x,y pixel coordinates of the mouse from `lParam` (Windows encodes them as low/high words).
  - `hdc = GetDC(hwnd);` obtains a device context for immediate drawing outside `WM_PAINT`. This is allowed but must be released with `ReleaseDC`. Drawing with `GetDC` writes directly to the window's surface — think of painting directly on the visible canvas. Note: It's immediate drawing (not deferred) and won't be persistent unless also stored in `g_shapes` and redrawn later.

  ```cpp
      Point p;
      p.x = (double)mx;
      p.y = (double)my;
      g_mouseClicks.push_back(p);
  ```
  Explanation:
  - `Point p;` declares a point structure (defined elsewhere).
  - `p.x = (double)mx; p.y = (double)my;` converts mouse coordinates to `double` members of `Point` — the code uses `Point` both for integer pixel positions and to preserve fractional values when needed.
  - `g_mouseClicks.push_back(p);` records this click in the global click list; subsequent logic will decide whether enough clicks are present to form a shape.

  Now `WndProc` branches by `g_currentSelection` and number of clicks. The following subsections explain each branch.

  ### Handling Ellipses (2 clicks: centre then radius point)
  **Intent:** If the user selected an ellipse tool, use two clicks: first center, second point to define radii. Draw and store the ellipse.

  ```cpp
      if (g_currentSelection >= 2001 && g_currentSelection <= 2003)
      {
        if (g_mouseClicks.size() == 2)
        {
          int a = (int)abs(g_mouseClicks[1].x - g_mouseClicks[0].x); // horizontal radius
          int b = (int)abs(g_mouseClicks[1].y - g_mouseClicks[0].y); // vertical radius
          int cx = (int)g_mouseClicks[0].x;
          int cy = (int)g_mouseClicks[0].y;
  ```
  Explanation:
  - The outer `if` checks whether the active selection ID is in the ellipses range (2001-2003).
  - `if (g_mouseClicks.size() == 2)` ensures two clicks were recorded (center and a radius-defining point).
  - `a` is computed as absolute horizontal difference between clicks => horizontal semi-axis (radius along x).
  - `b` is vertical semi-axis (radius along y).
  - `cx, cy` are the center coordinates taken from the first click.

  ```cpp
          if (a < 1) a = 1;
          if (b < 1) b = 1;
  ```
  Explanation:
  - Enforce minimum radii of 1 to avoid degenerate ellipses (prevents division by zero or empty shapes).

  ```cpp
          ShapeRecord rec;
          rec.type  = g_currentType;
          rec.x1    = cx - a; rec.y1 = cy - b;
          rec.x2    = cx + a; rec.y2 = cy + b;
          rec.color = g_drawColor;
          g_shapes.push_back(rec);
  ```
  Explanation:
  - `ShapeRecord rec;` creates a record to store the shape for future redrawing.
  - `rec.type = g_currentType;` sets the type (which matches the selected ellipse algorithm).
  - `rec.x1 = cx - a; rec.y1 = cy - b; rec.x2 = cx + a; rec.y2 = cy + b;` store bounding-box coordinates rather than center+axes. Many drawing functions later recompute center and radii from these values.
  - `rec.color = g_drawColor;` stores the drawing color.
  - `g_shapes.push_back(rec);` appends the shape record to the global list — this is the persistence step so the drawing survives repaints.

  ```cpp
          if (g_currentSelection == IDM_T5_DIRECT)   DrawEllipseDirect  (hdc, cx, cy, a, b, g_drawColor);
          if (g_currentSelection == IDM_T5_POLAR)    DrawEllipsePolar   (hdc, cx, cy, a, b, g_drawColor);
          if (g_currentSelection == IDM_T5_MIDPOINT) DrawEllipseMidpoint(hdc, cx, cy, a, b, g_drawColor);
          g_mouseClicks.clear();
        }
      }
  ```
  Explanation:
  - Depending on which ellipse menu item is active, call the matching algorithm function to draw the ellipse immediately using the `hdc`, center `(cx,cy)`, radii `a,b`, and `g_drawColor`.
  - `g_mouseClicks.clear();` resets the click buffer for the next operation.

  Graphics analogy reminder:
  - Here the program converts two clicks into a shape description, then the chosen algorithm draws pixels that approximate the ellipse on the painter’s grid (rasterization). `DrawEllipse*` functions use symmetry and per-pixel decisions — akin to deciding which graph-paper squares to color.

  ### Handling Circles (2 clicks: center, radius point)
  **Intent:** Build circle parameters from two clicks, draw using selected circle algorithm, and store in `g_shapes`.

  ```cpp
      else if (g_currentSelection >= 3001 && g_currentSelection <= 3005)
      {
        if (g_mouseClicks.size() == 2)
        {
          int R  = (int)sqrt(pow(g_mouseClicks[1].x - g_mouseClicks[0].x, 2) +
                     pow(g_mouseClicks[1].y - g_mouseClicks[0].y, 2));
          int xc = (int)g_mouseClicks[0].x;
          int yc = (int)g_mouseClicks[0].y;
  ```
  Explanation:
  - `else if` checks whether the selection is in circles range (3001-3005).
  - When 2 clicks are available, compute `R` as Euclidean distance from center to second click (radius).
  - `xc`,`yc` set to the first click (center).

  ```cpp
          ShapeRecord rec;
          rec.x1    = xc;
          rec.y1    = yc;
          rec.x2    = R;
          rec.y2    = 0;
          rec.color = g_drawColor;
  ```
  Explanation:
  - For circles, the code stores center in `x1,y1` and radius in `x2` (with `y2` unused), as a convention for later redraw.

  ```cpp
          if (g_currentSelection == IDM_CIRC_DIRECT) {
            rec.type = ShapeType::CIRCLE_DIRECT;
            CircleDirect          (hdc, xc, yc, R, g_drawColor);
          }
          if (g_currentSelection == IDM_CIRC_POLAR) {
            rec.type = ShapeType::CIRCLE_POLAR;
            CirclePolar           (hdc, xc, yc, R, g_drawColor);
          }
          if (g_currentSelection == IDM_CIRC_MIDPOINT) {
            rec.type = ShapeType::CIRCLE_MIDPOINT;
            CircleMidpoint        (hdc, xc, yc, R, g_drawColor);
          }
          if (g_currentSelection == IDM_CIRC_IPOLAR) {
            rec.type = ShapeType::CIRCLE_IPOLAR;
            CircleIterativePolar  (hdc, xc, yc, R, g_drawColor);
          }
          if (g_currentSelection == IDM_CIRC_MOD_MID) {
            rec.type = ShapeType::CIRCLE_MOD_MID;
            CircleModifiedMidpoint(hdc, xc, yc, R, g_drawColor);
          }
          g_shapes.push_back(rec);
          g_mouseClicks.clear();
        }
      }
  ```
  Explanation:
  - For each circle mode, set the `rec.type` accordingly and call the matching circle drawing function immediately (write pixels via `SetPixel` or symmetric plotting).
  - Store the record and clear clicks.

  Graphics analogy reminder:
  - Circle algorithms compute which pixels approximate the round shape — often by calculating symmetric points around the center to reduce work (drawing 8 mirrored pixels at once is like coloring symmetric squares on graph paper).

  ### Handling Lines (2 clicks: endpoints)
  **Intent:** Create a line record from two clicks, draw using selected line algorithm, store record.

  ```cpp
      else if (g_currentSelection >= 6001 && g_currentSelection <= 6003)
      {
        if (g_mouseClicks.size() == 2)
        {
          int x1 = (int)g_mouseClicks[0].x, y1 = (int)g_mouseClicks[0].y;
          int x2 = (int)g_mouseClicks[1].x, y2 = (int)g_mouseClicks[1].y;
  ```
  Explanation:
  - Check line tool selection range 6001-6003 and ensure two clicks are present.
  - Extract the coordinates for the two endpoints.

  ```cpp
          ShapeRecord rec;
          rec.x1    = x1;
          rec.y1    = y1;
          rec.x2    = x2;
          rec.y2    = y2;
          rec.color = g_drawColor;
  ```
  Explanation:
  - Store endpoints and color into a `ShapeRecord`.

  ```cpp
          if (g_currentSelection == IDM_LINE_DDA) {
            rec.type = ShapeType::LINE_DDA;
            DrawLineDDA       (hdc, x1, y1, x2, y2, g_drawColor);
          }
          if (g_currentSelection == IDM_LINE_MIDPOINT) {
            rec.type = ShapeType::LINE_MIDPOINT;
            DrawLineMidpoint  (hdc, x1, y1, x2, y2, g_drawColor);
          }
          if (g_currentSelection == IDM_LINE_PARAMETRIC) {
            rec.type = ShapeType::LINE_PARAMETRIC;
            DrawLineParametric(hdc, x1, y1, x2, y2, g_drawColor);
          }
          g_shapes.push_back(rec);
          g_mouseClicks.clear();
        }
      }
  ```
  Explanation:
  - Depending on selected line algorithm, set `rec.type`, call the corresponding draw function (which will use `SetPixel` or `LineTo`), push the record into `g_shapes`, and clear clicks.

  Graphics analogy reminder:
  - Line rasterization transforms the continuous mathematical line into discrete pixels — the chosen algorithm (DDA, midpoint, parametric) is just a different way to pick which squares to color on graph paper.

  ### Handling Fill Circle With Lines/Circles (1 click)
  **Intent:** For the menu options that fill a quarter-circle with lines or circles using a single click as center.

  ```cpp
      else if (g_currentSelection >= 7011 && g_currentSelection <= 7018)
      {
        int quarter = ((g_currentSelection - 1) % 4) + 1;
        if (g_currentSelection <= 7014)
          FillCircleWithLines  (hdc, mx, my, 80, quarter, g_drawColor);
        else
          FillCircleWithCircles(hdc, mx, my, 80, quarter, g_drawColor);
        g_mouseClicks.clear();
      }
  ```
  Explanation:
  - If the selection is one of the quarter-circle fill options (7011–7018), compute `quarter` by mapping menu ID to 1..4.
  - For IDs ≤ 7014 call `FillCircleWithLines`; else call `FillCircleWithCircles`. Both use a fixed radius `80` here (hard-coded) and the `mx,my` mouse coords as center.
  - Immediately draw the fill and clear clicks.

  Graphics analogy reminder:
  - Filling is like coloring inside a stencil; the function computes which pixels are inside the quarter circle and sets them accordingly.

  ### Handling Fill Square Hermite and Fill Rect Bezier (2 clicks)
  **Intent:** When the user selects curve-based filling, two clicks define the rectangle/square; the program fills using parametric curve sampling and draws an outline.

  ```cpp
      else if (g_currentSelection == IDM_FILL_SQ_HERMITE)
      {
        if (g_mouseClicks.size() == 2)
        {
          int x0   = (int)(std::min)(g_mouseClicks[0].x, g_mouseClicks[1].x);
          int y0   = (int)(std::min)(g_mouseClicks[0].y, g_mouseClicks[1].y);
          int side = (int)(std::max)(
            std::abs(g_mouseClicks[1].x - g_mouseClicks[0].x),
            std::abs(g_mouseClicks[1].y - g_mouseClicks[0].y));
          FillSquareHermite(hdc, x0, y0, side, g_drawColor);
          HBRUSH hOld = (HBRUSH)SelectObject(hdc, GetStockObject(NULL_BRUSH));
          Rectangle(hdc, x0, y0, x0 + side, y0 + side);
          SelectObject(hdc, hOld);
  ```
  Explanation:
  - For Hermite square fill: compute `x0,y0` as top-left (min of both clicks), `side` as the larger of width/height to make a square.
  - `FillSquareHermite` draws the interior using curve math; then the code selects a `NULL_BRUSH` to draw a rectangle outline over the filled area (for edge clarity), using `Rectangle(hdc, ...)` which writes to `hdc`.
  - `SelectObject(hdc, hOld);` restores the previous brush.

  ```cpp
          ShapeRecord rec;
          rec.type  = ShapeType::FILL_SQ_HERMITE;
          rec.x1    = x0;
          rec.y1    = y0;
          rec.x2    = side;
          rec.y2    = 0;
          rec.color = g_drawColor;
          g_shapes.push_back(rec);

          g_mouseClicks.clear();
        }
      }
  ```
  Explanation:
  - Create a `ShapeRecord` storing type, position, side length, color and push into `g_shapes` to persist the filled shape for future redraws.

  (There is an analogous block for `IDM_FILL_RECT_BEZIER` that uses `FillRectBezier` and stores width/height.)

  ### Handling Polygon vertices, Flood Fill, Cardinal Spline, Clipping setup, Smiley faces
  **Intent:** Cover other selection cases — collecting vertices, handling single-click fills, finalizing shapes on right-clicks, and drawing smiley faces.

  Key points:
  - For polygon fill modes (`FILL_CONVEX` / `FILL_NONCONVEX`), each left click adds a vertex and draws a small dot using `Ellipse(hdc, ...)` for immediate feedback.
  - For flood fill, `GetPixel(hdc, mx, my)` samples the color at the seed point. If it's not already the fill color, the code calls `FloodFillRecursive` or `FloodFillIterative` which set pixels using `SetPixel` until a border is reached (paint-bucket analogy).
  - For `IDM_CURVE_CARDINAL`, clicks are control points for the spline. On right-click `DrawCardinalSpline` samples the cubic Hermite basis and uses `SetPixel` to rasterize the curve; it also stores the control points in `g_shapes`.
  - Clipping modes: first two clicks set the clipping window (rect/square/circle); subsequent clicks are the shape to clip. Right-click runs Cohen-Sutherland, circle clipping math, or Sutherland-Hodgman polygon clipping to draw only the visible segment(s).
  - `IDM_FACE_HAPPY` / `IDM_FACE_SAD`: single click draws a composite face using circle functions and GDI primitives, then stores a `ShapeRecord`.

  In every case the pattern is:
  1. collect clicks into `g_mouseClicks`;
  2. when enough clicks are present (or on right-click), compute parameters;
  3. call drawing routines on `hdc` (immediate painting) and push a `ShapeRecord` into `g_shapes` (persistence).

  Graphics reminders:
  - `GetPixel`/`SetPixel` are direct pixel operations (graph-paper analogy).
  - Clipping functions compute intersections and only draw inside the stencil/window (stencil analogy).
  - Curve/spline drawing samples points along parametric formulas and sets their pixels.

  ### WM_RBUTTONDOWN — finalize multi-click operations (right-click)
  **Intent:** When the user right-clicks, finalize polygon fills, curves, or clipping operations using the collected `g_mouseClicks`.

  ```cpp
    case WM_RBUTTONDOWN:
    {
      hdc = GetDC(hwnd);

      // Cardinal Spline: draw when >= 2 points collected
      if (g_currentSelection == IDM_CURVE_CARDINAL && g_mouseClicks.size() >= 2)
      {
        std::vector<CurvePoint> cpts;
        ShapeRecord rec;
        rec.type  = ShapeType::CARDINAL_SPLINE;
        rec.color = g_drawColor;

        for (auto& p : g_mouseClicks)
        {
          cpts.push_back({ (int)p.x, (int)p.y });
          rec.points.push_back({ (int)p.x, (int)p.y });
        }
        DrawCardinalSpline(hdc, cpts, 0.0, 60, g_drawColor);
        g_shapes.push_back(rec);
        g_mouseClicks.clear();
        printf("">> Cardinal Spline drawn through %d control points.\n", (int)cpts.size());
      }
  ```
  Explanation:
  - On right-click, get an `HDC`.
  - If the active tool is the Cardinal spline tool and there are at least 2 control points:
    - build a `cpts` vector and a `ShapeRecord` from `g_mouseClicks`;
    - call `DrawCardinalSpline` to rasterize points on `hdc` (which uses `SetPixel`), then store the record for redraw;
    - clear clicks and log to console.

  There are similar right-click branches for finalizing polygon fills, clipping point/line/polygon cases. Each extracts coordinates, calls appropriate clipping/filling function, stores a `ShapeRecord`, and clears `g_mouseClicks`.

  ### WM_COMMAND — menu actions and selection changes
  **Intent:** Process menu selections (clear, save, load, preferences) and tool switching.

  ```cpp
    case WM_COMMAND:
    {
      int id = LOWORD(wp);
      switch (id)
      {
      // File
      case IDM_FILE_CLEAR: ClearScreen(hwnd); break;
      case IDM_FILE_SAVE:  SaveToFile(hwnd);  break;
      case IDM_FILE_LOAD:  LoadFromFile(hwnd); break;
      case IDM_FILE_EXIT:  DestroyWindow(hwnd); break;
  ```
  Explanation:
  - `WM_COMMAND` arrives when a menu item is selected (or control notification).
  - `int id = LOWORD(wp);` extracts the command ID.
  - The switch handles file actions: clear (empties `g_shapes` and invalidates window), save (serializes `g_shapes`), load (reads shapes back), exit (closes window).

  ```cpp
      case IDM_PREF_COLOR:
        g_drawColor = PrefChooseColor(hwnd, g_drawColor);
        printf(">> Drawing color: R=%d G=%d B=%d\n",
             GetRValue(g_drawColor),
             GetGValue(g_drawColor),
             GetBValue(g_drawColor));
        break;
  ```
  Explanation:
  - The color chooser opens a standard Windows color dialog and returns a `COLORREF` to `g_drawColor`. `GetRValue`/`GetGValue`/`GetBValue` extract channels for logging.

  ```cpp
      // All drawing-tool selections
      default:
        g_currentSelection = id;
        g_mouseClicks.clear();

        // Ellipse type selection updates the current shape type for drawing and recording
        if (g_currentSelection == IDM_T5_DIRECT)   g_currentType = ShapeType::ELLIPSE_DIRECT;
        if (g_currentSelection == IDM_T5_POLAR)    g_currentType = ShapeType::ELLIPSE_POLAR;
        if (g_currentSelection == IDM_T5_MIDPOINT) g_currentType = ShapeType::ELLIPSE_MIDPOINT;

        // Update menu check states for ellipse selection 
        {
          HMENU hMenu = GetMenu(hwnd);
          CheckMenuItem(hMenu, IDM_T5_DIRECT,
            (g_currentSelection == IDM_T5_DIRECT)   ? MF_CHECKED : MF_UNCHECKED);
          CheckMenuItem(hMenu, IDM_T5_POLAR,
            (g_currentSelection == IDM_T5_POLAR)    ? MF_CHECKED : MF_UNCHECKED);
          CheckMenuItem(hMenu, IDM_T5_MIDPOINT,
            (g_currentSelection == IDM_T5_MIDPOINT) ? MF_CHECKED : MF_UNCHECKED);
        }
        break;
      }
      break;
    }
  ```
  Explanation:
  - For any other `WM_COMMAND` id (i.e., selecting a tool), set `g_currentSelection` and clear pending clicks.
  - Update `g_currentType` for ellipses and refresh menu check marks.

  ### End of WndProc: message defaults, close, destroy

  ```cpp
    case WM_CLOSE:   DestroyWindow(hwnd); break;
    case WM_DESTROY: PostQuitMessage(0);  break;
    default:         return DefWindowProc(hwnd, mcode, wp, lp);
    }
    return 0;
  }
  ```
  Explanation:
  - `WM_CLOSE`: destroy window.
  - `WM_DESTROY`: post quit message to exit the message loop.
  - `default`: call `DefWindowProc` for unhandled messages.

  ### Function: WinMain
  **Intent:** Program entry point that registers the window class, creates the main window, sets the menu, and runs the message loop.

  ```cpp
  int APIENTRY WinMain(HINSTANCE h, HINSTANCE p, LPSTR c, int nsh)
  {
    AllocConsole();
    freopen("CONOUT$", "w", stdout);
    printf("=== Computer Graphics Project ===\n");
    printf("Preferences menu: change background, cursor shape, or drawing color.\n");
  ```
  Explanation:
  - `WinMain` is the entry point for Win32 GUI programs.
  - `AllocConsole()` creates a console window so `printf` outputs can be seen.
  - `freopen("CONOUT$", "w", stdout);` redirects `stdout` to that console.

  ```cpp
    WNDCLASS wc{};
    wc.hbrBackground = (HBRUSH)GetStockObject(WHITE_BRUSH);
    wc.hCursor       = LoadCursor(NULL, IDC_ARROW);
    wc.hInstance     = h;
    wc.lpfnWndProc   = WndProc;
    wc.lpszClassName = L"GraphicsProjectClass";
    wc.style         = CS_HREDRAW | CS_VREDRAW;
    RegisterClass(&wc);
  ```
  Explanation:
  - Initialize a `WNDCLASS` describing the window class (background brush, cursor, instance handle, `WndProc`, class name, style flags) and call `RegisterClass` so Windows can create windows of this type.

  ```cpp
    HWND hwnd = CreateWindow(
      L"GraphicsProjectClass",
      L"Computer Graphics - All Tasks Integrated",
      WS_OVERLAPPEDWINDOW,
      CW_USEDEFAULT, CW_USEDEFAULT, 1024, 720,
      NULL, NULL, h, NULL);

    SetMenu(hwnd, CreateAppMenu());
    ShowWindow(hwnd, nsh);
    UpdateWindow(hwnd);
  ```
  Explanation:
  - `CreateWindow` makes the visible window; `SetMenu` attaches the menu; `ShowWindow` displays it; `UpdateWindow` forces an initial paint.

  ```cpp
    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0) > 0)
    {
      TranslateMessage(&msg);
      DispatchMessage(&msg);
    }
    return (int)msg.wParam;
  }
  ```
  Explanation:
  - The classic Win32 message loop: `GetMessage` retrieves, `TranslateMessage` helps with keyboard messages, `DispatchMessage` calls `WndProc`. When `WM_QUIT` arrives, `GetMessage` returns 0 and the loop ends.

  ---

  ## 7. Line-by-Line Deep Dive — `task1_file_menu.cpp`

  I'll break `task1_file_menu.cpp` into small chunks (1–5 lines) and explain each line or short block in plain English. Where the code touches graphics concepts (drawing, HDC, rasterization, clipping, filling), I'll remind you of the simple analogy from the Graphics Primer.

  ### File header macros and includes
  **Intent:** Configure Unicode mode and import OS, std and project headers used by file I/O and drawing helpers.

  ```cpp
  #ifndef UNICODE
  #define UNICODE
  #endif
  #ifndef _UNICODE
  #define _UNICODE
  #endif
  ```
  Explanation:
  - These preprocessor guards ensure the project builds with Windows Unicode APIs (`wchar_t` strings). They tell the compiler to use the `W` (wide-char) variants of Win32 functions.

  ```cpp
  #include "task1_file_menu.h"
  #include <commdlg.h>
  #include <cstdio>
  #include <fstream>
  #include <sstream>
  ```
  Explanation:
  - `task1_file_menu.h` declares the `ShapeRecord`, `g_shapes`, and the function prototypes used across the program.
  - `commdlg.h` is the Windows common dialogs header (used for `GetOpenFileName` / `GetSaveFileName`).
  - `<cstdio>`, `<fstream>`, `<sstream>` provide C and C++ file I/O utilities used by `SaveToFile` / `LoadFromFile`.

  ```cpp
  #include "task5_ellipse_algorithms.h"
  #include "Circles.h"
  #include "Lines.h"
  #include "Filling.h"
  #include "Clipping.h"
  #include "Curves.h"
  #include "Preferences.h"
  #include "SmileyFace.h"

  #pragma comment(lib, "comdlg32.lib")
  ```
  Explanation:
  - These includes bring in the drawing algorithm implementations so `RedrawShapes` can call them while repainting.
  - `#pragma comment(lib, "comdlg32.lib")` tells the MSVC linker to link the common-dialogs library (so the color/file dialogs work).

  ### Global state: `g_shapes`
  **Intent:** Store all shapes drawn so the UI can be redrawn at any time.

  ```cpp
  // global shape list
  std::vector<ShapeRecord> g_shapes;
  ```
  Explanation:
  - `g_shapes` is a global `std::vector` that holds `ShapeRecord` entries describing every shape/action the user performed. This is the program's persistent model: when the window needs repainting, `RedrawShapes` iterates over `g_shapes` and replays each shape.
  - Concept link: this is the program's recording of what to render later — like a list of painting instructions that can be replayed on the 'painter's workspace' (HDC).

  ### Function: `BuildOFN`
  **Intent:** Create and initialize an `OPENFILENAME` structure used by the Windows file-open/save dialogs.

  ```cpp
  static OPENFILENAME BuildOFN(HWND hwnd, wchar_t* szFile, bool forSave)
  {
    OPENFILENAME ofn{}; // zero-initialize the struct
    ofn.lStructSize  = sizeof(ofn); // حجم ال struct
    ofn.hwndOwner    = hwnd;
  ```
  Explanation:
  - `BuildOFN` returns a filled `OPENFILENAME` struct. Line: `OPENFILENAME ofn{};` zero-initializes the struct to safe defaults.
  - `ofn.lStructSize = sizeof(ofn);` stores the structure size (required by the API).
  - `ofn.hwndOwner = hwnd;` tells the dialog which window owns the dialog (modal relationship).

  ```cpp
    ofn.lpstrFile    = szFile; // buffer to receive the file path
    ofn.nMaxFile     = MAX_PATH; // maximum size of the file path
    ofn.lpstrFilter  = L"Shape Files (*.shp)\0*.shp\0All Files (*.*)\0*.*\0"; // filter for file types
  ```
  Explanation:
  - `ofn.lpstrFile = szFile;` provides the buffer where the chosen file path will be returned.
  - `ofn.nMaxFile = MAX_PATH;` protects against buffer overflow.
  - `ofn.lpstrFilter` sets the filter string for the dialog (first visible choice is `*.shp`).

  ```cpp
    ofn.nFilterIndex = 1; // default to the first filter (Shape Files)
    ofn.lpstrDefExt  = L"shp"; // default extension if user doesn't specify
    ofn.Flags        = forSave
               ? (OFN_PATHMUSTEXIST | OFN_OVERWRITEPROMPT)
               : (OFN_PATHMUSTEXIST | OFN_FILEMUSTEXIST);
    return ofn;
  }
  ```
  Explanation:
  - `nFilterIndex` sets which filter is selected initially.
  - `lpstrDefExt` supplies a default extension to append if user omits one.
  - `Flags` chooses different behaviors for save vs open (for save: prompt if overwriting; for open: require existing file).

  ### Function: `ShapeTypeToStr`
  **Intent:** Convert a `ShapeType` enum to a human-readable string for file serialization.

  ```cpp
  static const char* ShapeTypeToStr(ShapeType t)
  {
    switch(t){
    case ShapeType::ELLIPSE_DIRECT:    return "ELLIPSE_DIRECT";
    case ShapeType::ELLIPSE_POLAR:     return "ELLIPSE_POLAR";
    case ShapeType::ELLIPSE_MIDPOINT:  return "ELLIPSE_MIDPOINT";
  ```
  Explanation:
  - The function switches on `t` and returns a literal string for each enum value. Each `case` maps an internal enum to a textual label that will be written to the file.

  ```cpp
    case ShapeType::CIRCLE_DIRECT:     return "CIRCLE_DIRECT";
    case ShapeType::CIRCLE_POLAR:      return "CIRCLE_POLAR";
    case ShapeType::CIRCLE_IPOLAR:     return "CIRCLE_IPOLAR";
    case ShapeType::CIRCLE_MIDPOINT:   return "CIRCLE_MIDPOINT";
    case ShapeType::CIRCLE_MOD_MID:    return "CIRCLE_MOD_MID";
  ```
  Explanation:
  - Same mapping pattern for circle types — these strings must match what `StrToShapeType` expects when loading.

  ```cpp
    case ShapeType::LINE_DDA:          return "LINE_DDA";
    case ShapeType::LINE_MIDPOINT:     return "LINE_MIDPOINT";
    case ShapeType::LINE_PARAMETRIC:   return "LINE_PARAMETRIC";
  ```
  Explanation:
  - Line type mappings.

  ```cpp
    case ShapeType::FILL_CIRC_LINES:     return "FILL_CIRC_LINES";
    case ShapeType::FILL_CIRC_CIRCLES:   return "FILL_CIRC_CIRCLES";
    case ShapeType::FILL_SQ_HERMITE:     return "FILL_SQ_HERMITE";
    case ShapeType::FILL_RECT_BEZIER:    return "FILL_RECT_BEZIER";
    case ShapeType::FILL_FLOOD_RECURSIVE:return "FILL_FLOOD_RECURSIVE";
    case ShapeType::FILL_FLOOD_ITERATIVE:return "FILL_FLOOD_ITERATIVE";
  ```
  Explanation:
  - Fill-type mappings; used so fills can be reconstructed when loaded.

  ```cpp
    case ShapeType::FACE_HAPPY:          return "FACE_HAPPY";
    case ShapeType::FACE_SAD:            return "FACE_SAD";
    case ShapeType::CONVEX_POLYGON:      return "CONVEX_POLYGON";
    case ShapeType::NONCONVEX_POLYGON:   return "NONCONVEX_POLYGON";
    case ShapeType::CARDINAL_SPLINE:     return "CARDINAL_SPLINE";
  ```
  Explanation:
  - Misc/compound shapes and curves mapping.

  ```cpp
    case ShapeType::CLIP_POINT_RECT:     return "CLIP_POINT_RECT";
    case ShapeType::CLIP_LINE_RECT:      return "CLIP_LINE_RECT";
    case ShapeType::CLIP_POINT_CIRCLE:   return "CLIP_POINT_CIRCLE";
    case ShapeType::CLIP_LINE_CIRCLE:    return "CLIP_LINE_CIRCLE";
    case ShapeType::CLIP_POLYGON_RECT:   return "CLIP_POLYGON_RECT";

    default:                           return "UNKNOWN";
    }
  }
  ```
  Explanation:
  - Clipping-type mappings and a `default` returning "UNKNOWN" as a safe fallback.

  ### Function: `StrToShapeType`
  **Intent:** Convert a saved string back to a `ShapeType` enum when loading a file.

  ```cpp
  static ShapeType StrToShapeType(const std::string& s)
  {
    if (s == "ELLIPSE_DIRECT")      return ShapeType::ELLIPSE_DIRECT;
    if (s == "ELLIPSE_POLAR")       return ShapeType::ELLIPSE_POLAR;
    if (s == "ELLIPSE_MIDPOINT")    return ShapeType::ELLIPSE_MIDPOINT;
  ```
  Explanation:
  - For each expected string, `StrToShapeType` checks equality and returns the matching enum. This reverses `ShapeTypeToStr` during file parsing.

  ```cpp
    if (s == "CIRCLE_DIRECT")       return ShapeType::CIRCLE_DIRECT;
    if (s == "CIRCLE_POLAR")        return ShapeType::CIRCLE_POLAR;
    if (s == "CIRCLE_IPOLAR")       return ShapeType::CIRCLE_IPOLAR;
    if (s == "CIRCLE_MIDPOINT")     return ShapeType::CIRCLE_MIDPOINT;
    if (s == "CIRCLE_MOD_MID")      return ShapeType::CIRCLE_MOD_MID;
  ```
  Explanation:
  - Circle types restored from saved strings.

  ```cpp
    // ... other string checks for lines, fills, faces, clipping, polygons, spline ...
    return ShapeType::ELLIPSE_DIRECT; // default / fallback
  }
  ```
  Explanation:
  - If none of the checks match, the function returns `ELLIPSE_DIRECT` as a fallback default. That prevents crashes, but means unknown strings silently map to an ellipse type.

  ### Function: `RedrawShapes`
  **Intent:** Repaint every stored `ShapeRecord` into the provided `HDC` (used by `WM_PAINT`).

  ```cpp
  void RedrawShapes(HDC hdc)
  {
    for (const auto& sh : g_shapes)
    {
      int cx = (sh.x1 + sh.x2) / 2;
      int cy = (sh.y1 + sh.y2) / 2;
      int a  = abs(sh.x2 - sh.x1) / 2;
      int b  = abs(sh.y2 - sh.y1) / 2;
      int R  = a; // for circles x1=cx, y1=cy, x2=cx+R, y2=cy
  ```
  Explanation:
  - `RedrawShapes` loops over `g_shapes` to replay all drawn content.
  - `cx, cy, a, b` compute a center and half-extents for shapes that store bounding boxes. For circles the code reuses `a` as the radius `R` by convention.
  - Concept link: this function replays stored painting instructions on the "painter's workspace" (`HDC`) so the display can be reconstructed after minimize/resize.

  ```cpp
      switch (sh.type)
      {
      case ShapeType::ELLIPSE_DIRECT:   DrawEllipseDirect   (hdc,cx,cy,a,b,sh.color); break;
      case ShapeType::ELLIPSE_POLAR:    DrawEllipsePolar    (hdc,cx,cy,a,b,sh.color); break;
      case ShapeType::ELLIPSE_MIDPOINT: DrawEllipseMidpoint (hdc,cx,cy,a,b,sh.color); break;
  ```
  Explanation:
  - For ellipse types the code calls the corresponding algorithm with the computed center and radii. Each `DrawEllipse*` paints pixels into `hdc`.

  ```cpp
      case ShapeType::CIRCLE_DIRECT:    CircleDirect        (hdc,sh.x1,sh.y1,sh.x2,sh.color); break;
      case ShapeType::CIRCLE_POLAR:     CirclePolar         (hdc,sh.x1,sh.y1,sh.x2,sh.color); break;
      case ShapeType::CIRCLE_IPOLAR:    CircleIterativePolar(hdc,sh.x1,sh.y1,sh.x2,sh.color); break;
      case ShapeType::CIRCLE_MIDPOINT:  CircleMidpoint      (hdc,sh.x1,sh.y1,sh.x2,sh.color); break;
      case ShapeType::CIRCLE_MOD_MID:   CircleModifiedMidpoint(hdc,sh.x1,sh.y1,sh.x2,sh.color); break;
  ```
  Explanation:
  - For circle types the code passes center in `sh.x1,sh.y1` and radius in `sh.x2`. Each called function chooses pixels to color to form the circle.

  ```cpp
      case ShapeType::LINE_DDA:         DrawLineDDA         (hdc,sh.x1,sh.y1,sh.x2,sh.y2,sh.color); break;
      case ShapeType::LINE_MIDPOINT:    DrawLineMidpoint    (hdc,sh.x1,sh.y1,sh.x2,sh.y2,sh.color); break;
      case ShapeType::LINE_PARAMETRIC:  DrawLineParametric  (hdc,sh.x1,sh.y1,sh.x2,sh.y2,sh.color); break;
  ```
  Explanation:
  - Line cases call the appropriate rasterization algorithm with endpoints stored in the record.

  ```cpp
      case ShapeType::FILL_CIRC_LINES:    FillCircleWithLines  (hdc, sh.x1, sh.y1, sh.x2, sh.y2, sh.color); break;
      case ShapeType::FILL_CIRC_CIRCLES:  FillCircleWithCircles(hdc, sh.x1, sh.y1, sh.x2, sh.y2, sh.color); break;
      case ShapeType::FILL_SQ_HERMITE:     FillSquareHermite   (hdc, sh.x1, sh.y1, sh.x2, sh.color); break;
      case ShapeType::FILL_RECT_BEZIER:    FillRectBezier      (hdc, sh.x1, sh.y1, sh.x2, sh.y2, sh.color); break;
      case ShapeType::FILL_FLOOD_RECURSIVE: FloodFillRecursive (hdc, sh.x1, sh.y1, sh.color, sh.x2); break;
      case ShapeType::FILL_FLOOD_ITERATIVE: FloodFillIterative (hdc, sh.x1, sh.y1, sh.color, sh.x2); break;
  ```
  Explanation:
  - Fill cases call fill algorithms; note the parameter conventions differ slightly between functions (some use width/height, some use radius, some use `x2` as stored target color for flood fill).
  - Concept link: filling algorithms decide which pixels inside a region to mark — like coloring all graph-paper squares inside a stencil.

  ```cpp
      case ShapeType::FACE_HAPPY:          DrawSmiley          (hdc, sh.x1, sh.y1, true); break;
      case ShapeType::FACE_SAD:            DrawSmiley          (hdc, sh.x1, sh.y1, false); break;
  ```
  Explanation:
  - DrawSmiley composes circles/lines/arcs to render a face; it's a convenience function that issues more low-level pixel/primitive draws.

  ```cpp
      case ShapeType::CONVEX_POLYGON:
      {
        std::vector<FillPoint> poly;
        for (auto& p : sh.points)
          poly.push_back({p.first, p.second});
        FillConvexPolygon(hdc, poly, sh.color);
        MoveToEx(hdc, poly.back().x, poly.back().y, NULL);
        for (auto& p : poly) LineTo(hdc, p.x, p.y);
        break;
      }
  ```
  Explanation:
  - For polygons the stored `sh.points` contain vertex coordinates. The code converts them into `FillPoint` objects, calls the scanline fill, then draws the polygon outline with `MoveToEx`/`LineTo`.
  - Concept link: scanline fill computes intersections per row and fills between pairs — like coloring each row of graph paper between left/right edges.

  ```cpp
      case ShapeType::NONCONVEX_POLYGON:
      {
        std::vector<FillPoint> poly;
        for (auto& p : sh.points)
          poly.push_back({p.first, p.second});
        FillNonConvexPolygon(hdc, poly, sh.color);
        MoveToEx(hdc, poly.back().x, poly.back().y, NULL);
        for (auto& p : poly) LineTo(hdc, p.x, p.y);
        break;
      }
  ```
  Explanation:
  - Non-convex polygons use the same conversion; the filling implementation handles edge cases that arise with concavities.

  ```cpp
      case ShapeType::CARDINAL_SPLINE:
      {
        std::vector<CurvePoint> cpts;
        for (auto& p : sh.points)
          cpts.push_back({p.first, p.second});
        DrawCardinalSpline(hdc, cpts, 0.0, 60, sh.color);
        break;
      }
  ```
  Explanation:
  - Reconstruct control points and call the spline rasterizer to sample points along the curve and mark the corresponding pixels.

  ```cpp
      case ShapeType::CLIP_POINT_RECT:
        ClipPointRect(hdc, sh.x1, sh.y1,
                sh.x2, sh.y2,
                sh.points[0].first, sh.points[0].second,
                sh.color);
        break;
  ```
  Explanation:
  - For clipping records, the code uses stored clipping-window parameters and the shape coordinates to invoke the relevant clip routine. `ClipPointRect` will `SetPixel` only if the point lies inside the rectangle (stencil analogy).

  ```cpp
      case ShapeType::CLIP_POINT_CIRCLE:
        ClipPointCircle(hdc, sh.x1, sh.y1,
                 sh.x2, sh.y2,
                 sh.points[0].first,
                 sh.color);
        break;
  ```
  Explanation:
  - Circle point clipping checks distance from center and draws only when inside the circle.

  ```cpp
      case ShapeType::CLIP_LINE_RECT:
        ClipLineRect(hdc, sh.x1, sh.y1, sh.x2, sh.y2,
               sh.points[0].first, sh.points[0].second,
               sh.points[1].first, sh.points[1].second);
        break;
  ```
  Explanation:
  - Line clipping calls Cohen-Sutherland variant which computes intersections and draws the clipped segment with `LineTo`/`MoveToEx`.

  ```cpp
      case ShapeType::CLIP_LINE_CIRCLE:
        ClipLineCircle(hdc, sh.x1, sh.y1, sh.x2, sh.y2,
                 sh.points[0].first, sh.points[0].second,
                 sh.points[1].first); // R stored in points[1].first
        break;
  ```
  Explanation:
  - Circle-line clipping solves a quadratic to find the segment inside the circle and draws that segment.

  ```cpp
      case ShapeType::CLIP_POLYGON_RECT:
      {
        PointList poly;
        for (auto& p : sh.points)
          poly.push_back({p.first, p.second});
        // x1,y1 = window min, x2,y2 = window max
        PolygonClip(hdc, poly, sh.x1, sh.y1, sh.x2, sh.y2);
        break;
      }
      }
    }
  }
  ```
  Explanation:
  - Polygon clipping uses the Sutherland-Hodgman edge-by-edge trimming. After clipping, the function draws the resulting polygon edges.

  ### Function: `ClearScreen`
  **Intent:** Remove all stored shapes and force a repaint (clear the canvas).

  ```cpp
  void ClearScreen(HWND hwnd)
  {
    g_shapes.clear();
    InvalidateRect(hwnd, NULL, TRUE);   // erase background + repaint
    UpdateWindow(hwnd);
  }
  ```
  Explanation:
  - `g_shapes.clear()` empties the persistent record. `InvalidateRect(hwnd, NULL, TRUE)` marks the entire client area invalid and requests a background erase; `UpdateWindow` forces an immediate `WM_PAINT` so the window refreshes with no shapes.

  ### Function: `SaveToFile`
  **Intent:** Show a Save dialog and write the `g_shapes` list to a plain-text `.shp` file so drawings can be reloaded.

  ```cpp
  void SaveToFile(HWND hwnd)
  {
    wchar_t szFile[MAX_PATH] = {}; // buffer to receive the file path
    OPENFILENAME ofn = BuildOFN(hwnd, szFile, /*forSave=*/true);

    if (!GetSaveFileName(&ofn))
      return;  // user cancelled
  ```
  Explanation:
  - Build the `OPENFILENAME` and call `GetSaveFileName`. If the user cancels, return early.

  ```cpp
    FILE* file = nullptr;
    errno_t err = _wfopen_s(&file, szFile, L"w");
    if (err != 0 || !file)
    {
      MessageBox(hwnd, L"Could not open file for writing.", L"Save Error", MB_ICONERROR);
      return;
    }
  ```
  Explanation:
  - `_wfopen_s` opens the wide-char path for writing. On error, a message box is shown and the function returns.

  ```cpp
    for (const auto& sh : g_shapes)
    {
      unsigned int pointCount = static_cast<unsigned int>(sh.points.size());
      fprintf(file, "%s %d %d %d %d %u %u",
          ShapeTypeToStr(sh.type), sh.x1, sh.y1, sh.x2, sh.y2,
          static_cast<unsigned int>(sh.color), pointCount);

      for (const auto& pt : sh.points)
        fprintf(file, " %d %d", pt.first, pt.second);

      fprintf(file, "\n");
    }
  ```
  Explanation:
  - Each `ShapeRecord` is serialized as a line: `TYPE x1 y1 x2 y2 color pointCount [x y]...`.
  - `ShapeTypeToStr` produces a textual tag so load-time mapping is easy. The loop writes each point pair after the header fields.

  ```cpp
    fclose(file);
    MessageBox(hwnd, L"Shapes saved successfully.", L"Save", MB_ICONINFORMATION);
  }
  ```
  Explanation:
  - Close the file and inform the user with a message box.

  ### Function: `LoadFromFile`
  **Intent:** Show an Open dialog, read the `.shp` file lines, parse them into `ShapeRecord`s, and store them in `g_shapes`.

  ```cpp
  void LoadFromFile(HWND hwnd)
  {
    wchar_t szFile[MAX_PATH] = {};
    OPENFILENAME ofn = BuildOFN(hwnd, szFile, /*forSave=*/false);
    if (!GetOpenFileName(&ofn))
      return;  // user cancelled

    FILE* file = nullptr;
    errno_t err = _wfopen_s(&file, szFile, L"r");
    if (err != 0 || !file)
    {
      MessageBox(hwnd, L"Could not open file for reading.", L"Load Error", MB_ICONERROR);
      return;
    }
  ```
  Explanation:
  - Use `GetOpenFileName` to get a path and open the file for reading. On failure, show an error.

  ```cpp
    g_shapes.clear();

    char buffer[1024];
    while (fgets(buffer, sizeof(buffer), file))
    {
      if (buffer[0] == '\n' || buffer[0] == '\0')
        continue;

      std::istringstream ss(buffer);
      std::string token;
      unsigned int color = 0;
      unsigned int pointCount = 0;
      ShapeRecord rec{};

      ss >> token >> rec.x1 >> rec.y1 >> rec.x2 >> rec.y2 >> color;
      if (!(ss >> pointCount))
        pointCount = 0;
      rec.type  = StrToShapeType(token);
      rec.color = static_cast<COLORREF>(color);
      rec.points.resize(pointCount);
      for (unsigned int i = 0; i < pointCount; ++i)
        ss >> rec.points[i].first >> rec.points[i].second;

      g_shapes.push_back(rec);
    }

    fclose(file);

    // Repaint with loaded shapes
    InvalidateRect(hwnd, NULL, TRUE);
    UpdateWindow(hwnd);
  }
  ```
  Explanation:
  - Read each line into `buffer`, parse tokens with `std::istringstream` into `token` and numerical fields.
  - `StrToShapeType` converts the textual type back to an enum. The code resizes `rec.points` and fills them from the remaining tokens.
  - Finally push the reconstructed `ShapeRecord` into `g_shapes`, close the file, and invalidate the window to force a repaint.

  ---

  ## 8. Line-by-Line Deep Dive — `Circles.cpp`

  I'll break `Circles.cpp` into very small chunks and explain each line or short block in plain English, with quick references to the Graphics Primer analogies where helpful.

  ### File: `Circles.cpp` — includes

  ```cpp
  #include "Circles.h"
  #include <cmath>
  ```
  Explanation:
  - `#include "Circles.h"` brings in the header with function declarations and any type definitions used here.
  - `#include <cmath>` provides math functions like `sqrt`, `cos`, `sin`, and `round` used by multiple algorithms. In graphics terms, this file contains different ways to compute which graph-paper squares (pixels) form a circle.

  ### Function: `Draw8Points`

  ```cpp
  // Draw 8 symmetric points of the circle using circle symmetry around the center (xc, yc)
  void Draw8Points(HDC hdc, int xc, int yc, int x, int y, COLORREF color) {
    SetPixel(hdc, xc + x, yc + y, color);
    SetPixel(hdc, xc - x, yc + y, color);
    SetPixel(hdc, xc + x, yc - y, color);
    SetPixel(hdc, xc - x, yc - y, color);
    SetPixel(hdc, xc + y, yc + x, color);
    SetPixel(hdc, xc - y, yc + x, color);
    SetPixel(hdc, xc + y, yc - x, color);
    SetPixel(hdc, xc - y, yc - x, color);
  }
  ```
  Explanation (line-by-line):
  - Comment: describes intent — exploit the 8-way symmetry of a circle to plot eight pixels per computed offset.
  - `void Draw8Points(...){` declares a helper that takes the device context `hdc`, center `(xc,yc)`, an offset `(x,y)` in the first octant, and a `color`.
  - Each `SetPixel(hdc, xc +/- x, yc +/- y, color);` writes one pixel at a mirrored location around the center. The first four lines plot the (±x, ±y) combinations; the next four plot the swapped offsets (±y, ±x) which cover the remaining octants.
  - Why: by computing one `(x,y)` in the first octant and mirroring it, the code draws all eight symmetric points at once — an optimization that saves repeating the expensive math. Analogy: compute one square on the graph paper and copy it to its mirror positions instead of recalculating each.

  ### Function: `CircleDirect`

  ```cpp
  // Direct circle drawing algo using the circle equation: x² + y² = R²
  void CircleDirect(HDC hdc, int xc, int yc, int R, COLORREF color) {

    // Start from the top point of the circle
    int x = 0, y = R;

    // Draw initial symmetric points
    Draw8Points(hdc, xc, yc, x, y, color);

    // Continue until x reaches y (first octant only)
    while (x < y) {
      x++;
        
      // Calculate y from circle equation
      y = round(sqrt(R * R - x * x));

      Draw8Points(hdc, xc, yc, x, y, color);
    }
  }
  ```
  Explanation (small chunks):
  - Comment: states the method — use the circle equation directly to compute `y` for each `x`.
  - `int x = 0, y = R;` initialize at the top of the circle (offset (0,R) from the center). This picks the first point in the first octant.
  - `Draw8Points(...)` plots the starting symmetric pixels for (0,R).
  - `while (x < y) {` iterate across the first octant until the 45° line where x==y; beyond that symmetry flips.
  - `x++;` increment `x` to the next column (one more step across graph-paper horizontally).
  - `y = round(sqrt(R * R - x * x));` directly compute the corresponding `y` from the circle equation x^2 + y^2 = R^2, then round to the nearest integer pixel row. This uses a costly `sqrt` per step, so it's simple but not the fastest.
  - `Draw8Points(...)` mirrors and draws all eight symmetric pixels for the computed offset.
  - Analogy: for each column `x` on graph paper, compute the circle's y using the formula and color the symmetric squares — accurate but uses heavy arithmetic (sqrt) per step.

  Notes / edge case: if `R` is 0 this routine draws a single pixel at the center. For small radii the loop terminates quickly; for large radii the repeated `sqrt` can be expensive.

  ### Function: `CirclePolar`

  ```cpp
  // Polar circle drawing algo using the polar equation of a circle: x = R * cos(theta), y = R * sin(theta)
  void CirclePolar(HDC hdc, int xc, int yc, int R, COLORREF color) {

    //Small angle step for smooth drawing
    double dtheta = 1.0 / R;

    // Draw only first octant (from 0 to PI/4)
    for (double theta = 0; theta < 0.785; theta += dtheta) { 

      //Conversion to cartesian 
      int x = round(R * cos(theta));
      int y = round(R * sin(theta));

      Draw8Points(hdc, xc, yc, x, y, color);
    }
  }
  ```
  Explanation (small chunks):
  - Comment: this method parametrizes the circle by angle `theta` and computes `x,y` from `cos`/`sin`.
  - `double dtheta = 1.0 / R;` choose a step size inversely proportional to the radius so that larger circles take smaller angular steps (smoother). Note: if `R` is 0 this would divide by zero — the caller should avoid radius 0 for this method.
  - `for (double theta = 0; theta < 0.785; theta += dtheta) {` iterate theta from 0 up to approx π/4 (0.785 radians) — the first octant. The code only computes one octant and relies on symmetry.
  - `int x = round(R * cos(theta)); int y = round(R * sin(theta));` convert polar to cartesian and round to pixel coordinates.
  - `Draw8Points(...)` mirror and plot the eight symmetric pixels.
  - Analogy: imagine stepping around the circle by small angles and marking the corresponding squares on the graph paper; trig calls are accurate but relatively expensive.

  Performance note: this is straightforward but uses trigonometric functions per step; accuracy depends on `dtheta` choice and rounding.

  ### Function: `CircleIterativePolar`

  ```cpp
  // Iterative polar circle algo avoiding repeated sin/cos calculations
  void CircleIterativePolar(HDC hdc, int xc, int yc, int R, COLORREF color) {

    // Rotation angle increment
    double dtheta = 1.0 / R;
    double ct = cos(dtheta), st = sin(dtheta);
    
    // Start Point (R, 0)
    double x = R, y = 0;

    Draw8Points(hdc, xc, yc, R, 0, color);

    // Rotate point iteratively
    while (x > y) {

       // Rotation transformation
      double x1 = x * ct - y * st;
      y = x * st + y * ct;
      x = x1;

      Draw8Points(hdc, xc, yc, round(x), round(y), color);
    }
  }
  ```
  Explanation (small chunks):
  - Comment: describes intention — iterate a single point by rotation instead of calling `cos`/`sin` each time.
  - `double dtheta = 1.0 / R; double ct = cos(dtheta), st = sin(dtheta);` precompute cosine and sine of the small rotation angle once, to reuse in the iterative step.
  - `double x = R, y = 0;` initialize at (R,0) — the rightmost point on the circle — and plot it with `Draw8Points`.
  - `while (x > y) {` iterate until the point crosses the 45° line (x <= y), covering the first octant.
  - The rotation transform `x1 = x * ct - y * st; y = x * st + y * ct; x = x1;` applies the 2D rotation matrix to advance the point by `dtheta` radians.
  - `Draw8Points(hdc, xc, yc, round(x), round(y), color);` rounds the rotated coordinates to pixel integers and mirrors them.
  - Analogy & trade-offs: this avoids per-iteration trig calls (faster), but because it's iterative with floating-point updates it accumulates rounding error over many steps — a small drift can occur on large circles. The approach is like rotating a compass needle a tiny bit repeatedly instead of recalculating the endpoint from scratch.

  ### Function: `CircleMidpoint`

  ```cpp
  // Midpoint circle algo using decision parameter to choose next pixel
  void CircleMidpoint(HDC hdc, int xc, int yc, int R, COLORREF color) {
    // Start point (top)
    int x = 0, y = R;

    // First decision parameter
    int d = 1 - R;
    Draw8Points(hdc, xc, yc, x, y, color);

    // first octant
    while (x < y) {
      // move horizontal
      if (d < 0) d += 2 * x + 3;
      else {
        // move diagonally down
        d += 2 * (x - y) + 5;
        y--;
      }
      x++;
      Draw8Points(hdc, xc, yc, x, y, color);
    }
  }
  ```
  Explanation (small chunks):
  - Comment: describes algorithm — a midpoint (Bresenham-like) decision method that uses an integer `d` to decide the next pixel.
  - `int x = 0, y = R;` start at the top of the circle in first octant.
  - `int d = 1 - R;` initialize the decision parameter; derived from evaluating the midpoint condition relative to the circle equation — this single integer encodes whether the next step should change `y`.
  - `Draw8Points(...)` plot the initial eight symmetric pixels.
  - `while (x < y) {` iterate across the first octant.
  - `if (d < 0) d += 2 * x + 3;` if the midpoint is inside the circle, the next pixel is horizontally adjacent: update `d` by the increment for a horizontal move.
  - `else { d += 2 * (x - y) + 5; y--; }` otherwise the midpoint is outside (or on) and we must move diagonally (decrement `y`) and update `d` by the diagonal step increment.
  - `x++;` advance `x` each iteration; then `Draw8Points(...)` mirrors and plots the chosen pixel.
  - Analogy: uses an error accumulator (decision variable) to choose the best next graph-paper square without expensive math — like using a small tally to decide whether to step up or stay level while drawing the curve.

  Performance note: this is an efficient integer algorithm (no trig, no sqrt), making it suitable for real-time drawing.

  ### Function: `CircleModifiedMidpoint`

  ```cpp
  // Modified midpoint circle algo using incremental updates
  void CircleModifiedMidpoint(HDC hdc, int xc, int yc, int R, COLORREF color) {
    //Start point
    int x = 0, y = R;

    // First decision parameter
    int d = 1 - R;

    // Increment values 
    int d1 = 3;
    int d2 = 5 - 2 * R;

    Draw8Points(hdc, xc, yc, x, y, color);
    //draw first octant
    while (x < y) {
      // move horizontal
      if (d < 0) {
        d += d1;
        d2 += 2;
        d1 += 2;
      } else {
        // move diagonally down
        d += d2;
        d2 += 4;
        d1 += 2;
        y--;
      }
      x++;
      Draw8Points(hdc, xc, yc, x, y, color);
    }
  }
  ```
  Explanation (small chunks):
  - Comment: indicates a micro-optimized midpoint variant that precomputes and updates incremental values to reduce repeated arithmetic.
  - Initialization: `x=0, y=R`, `d = 1 - R` same as the classic midpoint.
  - `int d1 = 3; int d2 = 5 - 2 * R;` precompute two increment values used to update `d` depending on the chosen step. These track how `d` would change for horizontal vs diagonal moves and are themselves updated incrementally to avoid multiplications inside the loop.
  - Inside the loop: if `d < 0` we take a horizontal step and update `d` by `d1`; then increment `d2` and `d1` for the next iteration. Otherwise take a diagonal step, update `d` by `d2`, change `d2` and `d1` appropriately, and decrement `y`.
  - `x++` and `Draw8Points(...)` are the same plotting mechanics as earlier.
  - Why: this reduces arithmetic work per pixel and keeps all updates integer-only — good for old-school raster devices where multiplications/divisions are expensive.

  Analogy: like keeping small running counters of how your error will change if you step one way or the other, then updating those counters each step — cheaper than recomputing from scratch.

  ---


  ## 9. Line-by-Line Deep Dive — `task5_ellipse_algorithms.cpp`

  I'll break `task5_ellipse_algorithms.cpp` into very small chunks and explain each line or short block in plain English, with reminders of the graphics analogies where relevant.

  ### File header and includes

  ```cpp
  #include "task5_ellipse_algorithms.h"
  #include <cmath>
  ```
  Explanation:
  - `#include "task5_ellipse_algorithms.h"` brings in the declarations for the ellipse functions and any shared types used across files.
  - `#include <cmath>` provides `sqrt`, `cos`, `sin`, and `round` which appear in the algorithms below.

  ### Helper: `Draw4Points`

  ```cpp
  //draws the 4 symmetric points of the ellipse for a given (dx, dy) offset
  static inline void Draw4Points(HDC hdc, int cx, int cy,
                     int dx, int dy, COLORREF color)
  {
    SetPixel(hdc, cx + dx, cy + dy, color);
    SetPixel(hdc, cx - dx, cy + dy, color);
    SetPixel(hdc, cx + dx, cy - dy, color);
    SetPixel(hdc, cx - dx, cy - dy, color);
  }
  ```
  Explanation (line-by-line):
  - Comment: intent — use 4-way symmetry of an axis-aligned ellipse to draw four mirror points from one offset.
  - `static inline void Draw4Points(...){` declares a small helper to keep the drawing code concise and avoid duplicating pixel writes.
  - Each `SetPixel(hdc, cx ± dx, cy ± dy, color);` writes a pixel at the corresponding mirrored coordinate around the center `(cx,cy)`.
  - Analogy: compute a single square to color on graph paper and mirror it across the ellipse axes instead of recomputing symmetric points.

  ### Function: `DrawEllipseDirect`

  ```cpp
  // direct algo
  // a is the horizontal radius, b is the vertical radius, (cx, cy) is the center of the ellipse
  void DrawEllipseDirect(HDC hdc, int cx, int cy, int a, int b, COLORREF color)
  {
    // case where the ellipse collapses to a line or point
    if (a == 0 || b == 0) return;

    // Precompute squares to avoid repeated multiplication
    double a2 = static_cast<double>(a) * a;
    double b2 = static_cast<double>(b) * b;

    // iterate over x-axis to ensure no gaps
    for (int dx = -a; dx <= a; ++dx)
    {
      // Compute corresponding y using the ellipse equation: (x^2/a^2) + (y^2/b^2) = 1
      double inner = 1.0 - (static_cast<double>(dx) * dx) / a2;
      // Due to rounding errors, inner might become slightly negative, make it zero
      if (inner < 0.0) inner = 0.0;
      // Compute corresponding y using the ellipse equation: (x^2/a^2) + (y^2/b^2) = 1
      int dy = static_cast<int>(std::round(b * std::sqrt(inner)));
      Draw4Points(hdc, cx, cy, dx, dy, color);
    }

    // iterate over y-axis to ensure no gaps 
    for (int dy = -b; dy <= b; ++dy)
    {
      //same steps for y-axis
      double inner = 1.0 - (static_cast<double>(dy) * dy) / b2;
      if (inner < 0.0) inner = 0.0;
      int dx = static_cast<int>(std::round(a * std::sqrt(inner)));
      Draw4Points(hdc, cx, cy, dx, dy, color);
    }
  }
  ```
  Explanation (chunked):
  - `if (a == 0 || b == 0) return;` guard: if either radius is zero the ellipse degenerates to a line or point, so nothing to draw (early exit).
  - `double a2 = ...; double b2 = ...;` precompute squared radii as `double` to avoid repeated integer multiplication and allow floating arithmetic below.
  - `for (int dx = -a; dx <= a; ++dx)` sweep across every integer x-offset across the horizontal span of the ellipse so no horizontal column is missed.
  - `double inner = 1.0 - (dx*dx)/a2;` derives from rearranging (x^2/a^2) + (y^2/b^2) = 1 to get y^2/b^2 = 1 - x^2/a^2. `inner` is the fraction that times b^2 gives y^2.
  - `if (inner < 0.0) inner = 0.0;` protect against tiny negative values due to floating-point rounding which would make `sqrt` invalid.
  - `int dy = round(b * sqrt(inner));` compute the vertical offset `dy` by taking the square root and scaling by `b`, then round to nearest pixel row.
  - `Draw4Points(...)` mirror and plot 4 pixels for that (dx,dy).
  - After the x-sweep, the code does a y-sweep `for (dy = -b..b)` doing the symmetric computation `dx = round(a * sqrt(inner))` — this ensures vertical gaps (if any) are filled because the two passes complement each other. In practice, drawing both axis-aligned sweeps avoids missing pixels near steep slopes.
  - Analogy: compute which graph-paper squares belong to the ellipse column-by-column and row-by-row, then mirror each computed square to its symmetric positions.

  Performance note: this direct method is straightforward and accurate but does a square root per sample; the double pass ensures coverage but costs extra work.

  ### Function: `DrawEllipsePolar`

  ```cpp
  // polar algo
  void DrawEllipsePolar(HDC hdc, int cx, int cy, int a, int b, COLORREF color)
  {
    if (a == 0 || b == 0) return;     // case where the ellipse collapses to a line or point

    // Choose step size so that the arc length per step approximates 1 pixel
    int r = (a > b) ? a : b;
    double step = 1.0 / r;

    // Iterate over angle from 0 to π/2 and use symmetry to draw all 4 points, which is more efficient and avoids gaps at the poles
    const double HALF_PI = 3.14159265358979323846 / 2.0;

    for (double theta = 0.0; theta <= HALF_PI; theta += step)
    {
      // Parametric ellipse equations
      int dx = static_cast<int>(std::round(a * std::cos(theta)));
      int dy = static_cast<int>(std::round(b * std::sin(theta)));

      // Draw the 4 symmetric points
      Draw4Points(hdc, cx, cy, dx, dy, color);
    }
    
  }
  ```
  Explanation (chunked):
  - `if (a == 0 || b == 0) return;` guard as before.
  - `int r = (a > b) ? a : b; double step = 1.0 / r;` pick the larger radius to set an angular step such that the arc step is roughly one pixel — larger ellipses get smaller angular increments for smoothness.
  - The loop runs `theta` from `0` to `π/2` (first quadrant) and uses `Draw4Points` to mirror into all four quadrants; this reduces work and avoids redundant sampling at symmetrical angles.
  - `dx = round(a * cos(theta))` and `dy = round(b * sin(theta))` convert the parametric coordinates to integer pixel offsets.
  - Analogy: walk around one quarter of the ellipse like stepping by small angles on a protractor, compute the corresponding graph-paper squares, and mirror them; trig is accurate but more expensive than pure integer methods.

  Note: the commented-out 0..2π version in the source shows a simpler full-angle approach that was replaced by the symmetric quarter-angle loop for efficiency and to avoid gaps at poles.

  ### Function: `DrawEllipseMidpoint`

  ```cpp
  // midpoint algo
  void DrawEllipseMidpoint(HDC hdc, int cx, int cy, int a, int b, COLORREF color)
  {
    if (a == 0 || b == 0) return; // case where the ellipse collapses to a line or point

    // Precompute squares to avoid repeated multiplication
    long long a2 = static_cast<long long>(a) * a;
    long long b2 = static_cast<long long>(b) * b;

    int x = 0;
    int y = b;

    // case1 |slope| <= 1  
    // Initial decision parameter p1 = b^2 - a^2*b + a^2/4
    long long p1 = b2 - a2 * b + (a2 + 2) / 4;  

    //while we are in the first region, we step through x and decide whether to move vertically based on p1
    while (2LL * b2 * x < 2LL * a2 * y)
    {
      Draw4Points(hdc, cx, cy, x, y, color);

      if (p1 < 0)
      {
        // Move vertically only
        ++x;
        p1 += 2LL * b2 * x + b2;
      }
      else
      {
        // Move diagonally (x+1, y-1)
        ++x;
        --y;
        p1 += 2LL * b2 * x - 2LL * a2 * y + b2;
      }
    }

    // case2 |slope| > 1
    // Initial decision parameter p2 = b^2*(x+1/2)^2 + a^2*(y-1)^2 - a^2*b^2
    long long p2 = b2 * (2LL * x + 1) * (2LL * x + 1) / 4
           + a2 * (y - 1LL) * (y - 1LL)
           - a2 * b2;

    // step through y and decide whether to move horizontally based on p2
    while (y >= 0)
    {
      Draw4Points(hdc, cx, cy, x, y, color);

      if (p2 > 0)
      {
        // Move vertically only
        --y;
        p2 -= 2LL * a2 * y + a2;
      }
      else
      {
        // Move diagonally (x+1, y-1)
        ++x;
        --y;
        p2 += 2LL * b2 * x - 2LL * a2 * y + a2;
      }
    }
  }
  ```
  Explanation (chunked, careful):
  - Guard: `if (a == 0 || b == 0) return;` handles degenerate ellipses.
  - `long long a2 = (long long)a * a; long long b2 = ...;` precompute squared radii as 64-bit integers to avoid overflow for moderately large `a`/`b`, and to keep subsequent decision math integer-based for speed and determinism.
  - `int x = 0; int y = b;` start at the top of the ellipse in the first region.
  - `long long p1 = b2 - a2 * b + (a2 + 2) / 4;` initialize the first-region decision parameter. This expression approximates the classic midpoint formula (which includes an `a^2/4` term); the code uses integer arithmetic `(a2 + 2) / 4` to safely calculate a rounded quarter-term.
  - `while (2LL * b2 * x < 2LL * a2 * y)` loop condition defines region 1 where the ellipse slope magnitude is ≤ 1 (i.e., x increments faster than y). The multiplication by 2 on both sides avoids fractions and keeps the comparison integer-based.
  - Inside the loop:
    - `Draw4Points(...)` paints current symmetric pixels.
    - `if (p1 < 0)` means the midpoint test indicates the next pixel should be horizontally adjacent (increase `x` only): `++x; p1 += 2*b2*x + b2;` updates the decision variable by the incremental difference corresponding to that move.
    - `else` means the midpoint test suggests a diagonal move is better: `++x; --y; p1 += 2*b2*x - 2*a2*y + b2;` — perform diagonal step and update `p1` by the diagonal increment.
  - After region 1 completes, compute `p2` as the initial decision parameter for region 2 using integer algebra:
    - `p2 = b2 * (2*x + 1)^2 / 4 + a2 * (y - 1)^2 - a2*b2;` this matches the derived midpoint evaluation when switching regions and avoids floating arithmetic by scaling terms appropriately.
  - `while (y >= 0)` iterates while there are vertical steps left to process:
    - `Draw4Points(...)` paints the current symmetric pixels.
    - `if (p2 > 0)` means the decision favors a vertical move only: `--y; p2 -= 2*a2*y + a2;` update the decision variable for the vertical step.
    - `else` perform a diagonal move: `++x; --y; p2 += 2*b2*x - 2*a2*y + a2;` and update `p2` accordingly.
  - Why it matters: the midpoint algorithm avoids expensive square roots and trig by using integer decision parameters and incremental updates. Dividing the ellipse into two regions (slope ≤1 and slope >1) simplifies the incremental rules in each region.
  - Analogy: instead of computing the exact curve position each time (heavy arithmetic), the algorithm keeps a small running error value that tells you whether to step horizontally, vertically, or diagonally — like using a finger-counting trick to decide the next square to color on graph paper.

  ---

  ## 10. Line-by-Line Deep Dive — `Lines.cpp`

  I'll break `Lines.cpp` into very small chunks (1–5 lines) and explain each line or short block in plain English, and link back to the raster/graph-paper analogies when helpful.

  ### File header

  ```cpp
  #include "Lines.h"
  #include <cmath>
  #include <algorithm>
  ```
  Explanation:
  - `#include "Lines.h"` imports the declarations for the line-drawing functions.
  - `<cmath>` provides math helpers like `round` used by the DDA and parametric implementations.
  - `<algorithm>` gives utilities like `std::abs` used here.

  ### DDA Line Drawing: `DrawLineDDA`

  ```cpp
  // DDA Line Drawing Algo
  void DrawLineDDA(HDC hdc, int x1, int y1, int x2, int y2, COLORREF color)
  {
    // Calculate delta x and y
    int dx = x2 - x1;
    int dy = y2 - y1;
    // calculate the number of steps needed
    int steps = (std::abs(dx) > std::abs(dy)) ? std::abs(dx) : std::abs(dy);
    // if steps is 0 line is a single point
    if (steps == 0) { SetPixel(hdc, x1, y1, color); return; }

    // Calculate the increment in x and y for each step
    double xInc = (double)dx / steps;
    double yInc = (double)dy / steps;

    double x = x1;
    double y = y1;
    // Draw the line by incrementing x and y in each step
    for (int i = 0; i <= steps; ++i)
    {
      SetPixel(hdc, (int)round(x), (int)round(y), color);
      x += xInc;
      y += yInc;
    }
  }
  ```
  Explanation (line-by-line):
  - `int dx = x2 - x1; int dy = y2 - y1;` compute the vector from start to end.
  - `int steps = (abs(dx) > abs(dy)) ? abs(dx) : abs(dy);` choose the larger delta to ensure we step by 1 pixel in the dominant axis and sample the other coordinate proportionally — this prevents gaps.
  - `if (steps == 0) { SetPixel(...); return; }` handle degenerate case where both points are identical: draw a single pixel and exit.
  - `double xInc = dx / (double)steps; double yInc = dy / (double)steps;` compute per-step fractional increments so after `steps` iterations you move exactly from `(x1,y1)` to `(x2,y2)`.
  - `double x = x1; double y = y1;` start positions as floating values so increments accumulate fractions.
  - `for (int i = 0; i <= steps; ++i) { SetPixel(round(x), round(y)); x += xInc; y += yInc; }` loop `steps+1` times to include both endpoints, at each iteration round the floating coordinates to the nearest pixel and set it. Analogy: walk from start to end in equal steps and mark the nearest graph-paper square each time.

  Trade-offs: DDA is simple and works for all slopes, but it uses floating arithmetic and rounding per step which can be slower than integer-only algorithms.

  ### Midpoint Line Drawing: `DrawLineMidpoint`

  ```cpp
  // Midpoint Line Drawing Algo
  void DrawLineMidpoint(HDC hdc, int x1, int y1, int x2, int y2, COLORREF color)
  {
    // Calculate deltas and steps
    int dx = std::abs(x2 - x1);
    int dy = std::abs(y2 - y1);

    int sx = (x1 < x2) ? 1 : -1;
    int sy = (y1 < y2) ? 1 : -1;
    int err = dx - dy;

    while (true)
    {
      // Set the pixel for the current point
      SetPixel(hdc, x1, y1, color);
      // If we've reached the end point, break
      if (x1 == x2 && y1 == y2) break;
      // Calculate error and increment x and y accordingly
      int e2 = 2 * err;
      if (e2 > -dy) { err -= dy; x1 += sx; }
      if (e2 <  dx) { err += dx; y1 += sy; }
    }
  }
  ```
  Explanation (stepwise):
  - `int dx = abs(x2-x1); int dy = abs(y2-y1);` compute absolute deltas to drive the algorithm independently of point order.
  - `int sx = (x1 < x2) ? 1 : -1; int sy = (y1 < y2) ? 1 : -1;` determine the sign of steps (direction) for x and y so the loop can increment or decrement coordinates toward the endpoint.
  - `int err = dx - dy;` initialize the Bresenham-style error accumulator. This single integer represents how far off the current raster position is from the ideal line.
  - `while (true) { SetPixel(hdc, x1, y1, color); if (x1==x2 && y1==y2) break; int e2 = 2*err; if (e2 > -dy) { err -= dy; x1 += sx; } if (e2 < dx) { err += dx; y1 += sy; } }`
    - Each iteration paints the current pixel and checks for completion.
    - `e2 = 2*err` creates two branch conditions: if `e2 > -dy` we move in x direction (adjust `err` and x); if `e2 < dx` we move in y direction. Both moves can happen in the same iteration for steep slopes (diagonal step).
  - Analogy: use an integer error counter to decide whether to advance horizontally, vertically, or both — like tracking how far you have deviated from the ideal slope while stepping across graph paper.

  Why use midpoint/Bresenham: it uses only integer arithmetic and branches, which makes it fast and exact for raster devices.

  ### Parametric Line Drawing: `DrawLineParametric`

  ```cpp
  // Parametric Line Drawing Algo
  void DrawLineParametric(HDC hdc, int x1, int y1, int x2, int y2, COLORREF color)
  {
    // Calculate deltas and steps
    int dx = x2 - x1;
    int dy = y2 - y1;
    int steps = (std::abs(dx) > std::abs(dy)) ? std::abs(dx) : std::abs(dy);
    if (steps == 0) { SetPixel(hdc, x1, y1, color); return; } // single point case

    // Draw the line by varying t from 0 to 1
    double dt = 1.0 / steps;
    // calculate the corresponding point on the line and set the pixel
    for (int i = 0; i <= steps; ++i)
    {
      double t = i * dt;
      // Calculate the x and y coordinates using the parametric form of the line
      int x = (int)round(x1 + t * dx);
      int y = (int)round(y1 + t * dy);
      SetPixel(hdc, x, y, color);
    }
  }
  ```
  Explanation (chunked):
  - `int dx = x2-x1; int dy = y2-y1; int steps = max(abs(dx), abs(dy));` same step-count logic as DDA.
  - `if (steps == 0) ...` handle single point case.
  - `double dt = 1.0 / steps; for (i=0..steps) { t = i*dt; x = round(x1 + t*dx); y = round(y1 + t*dy); SetPixel(...) }` sample the parametric line `P(t) = (1-t)*P0 + t*P1` for `t` in [0,1]. This is mathematically direct and numerically stable; similar to DDA but expresses the interpolation explicitly in terms of a parameter `t`.
  - Analogy: slide a ruler from the start to the end point, picking evenly spaced fractional positions along the ruler and marking their nearest squares on the graph paper.

  ---

  ## 11. Line-by-Line Deep Dive — `Filling.cpp`

  I'll break `Filling.cpp` into small chunks and explain each function and key lines in plain English, linking to the graphics analogies where useful.

  ### File header and helpers

  ```cpp
  #define NOMINMAX
  #include "Filling.h"
  #include <cmath>
  #include <algorithm>
  #include <stack>
  #include <vector>
  ```
  Explanation:
  - `NOMINMAX` prevents Windows headers from defining `min`/`max` macros that conflict with `std::min`/`std::max`.
  - Includes pull in math, container and stack utilities used by the fill algorithms.

  ```cpp
  static inline bool InQuarter(int dx, int dy, int quarter)
  {
    switch (quarter)
    {
    case 3: return dx >= 0 && dy <= 0;
    case 4: return dx <= 0 && dy <= 0;
    case 1: return dx <= 0 && dy >= 0;
    case 2: return dx >= 0 && dy >= 0;
    default: return true; 
    }
  }
  ```
  Explanation (line-by-line):
  - `InQuarter` tests whether a point at offset `(dx,dy)` from a circle center lies in the requested quarter (1..4). The function maps quarters to simple sign tests on `dx`/`dy`.
  - Used by quarter-based fill routines to limit which octant/octant-symmetric pixels are painted (stencil analogy).

  ### `FillCircleWithLines` — scan-by-rows filling

  ```cpp
  void FillCircleWithLines(HDC hdc, int cx, int cy, int r, int quarter, COLORREF color)
  {
    // find intersection points with the circle
    for (int dy = -r; dy <= r; ++dy)
    {
      int dx_max = (int)sqrt((double)(r * r - dy * dy));

      // For each dy, find the max dx that satisfies the circle equation
      for (int dx = -dx_max; dx <= dx_max; ++dx)
      { 
        // Check if the point (cx+dx, cy+dy) is in the specified quarter and fill it
        if (InQuarter(dx, dy, quarter))
          SetPixel(hdc, cx + dx, cy + dy, color);
      }
    }
  }
  ```
  Explanation:
  - Outer loop: iterate vertical offsets `dy` through the circle.
  - `dx_max = sqrt(r^2 - dy^2)` gives the horizontal half-width at that row (circle equation), so the inner loop covers every pixel inside the circle on that row.
  - Each candidate pixel is tested with `InQuarter` and painted with `SetPixel` if allowed.
  - Analogy: for each row of graph paper through the circle, color all squares between the left and right intersection points.

  Performance note: simple and correct; uses sqrt per row but avoids complex bookkeeping.

  ### `FillCircleWithCircles` — concentric circle filling

  ```cpp
  void FillCircleWithCircles(HDC hdc, int cx, int cy, int r, int quarter, COLORREF color)
  {
    for (int ri = 1; ri <= r; ++ri)
    {
      // Use midpoint circle algorithm, draw only pixels in chosen quarter
      int x = 0, y = ri;
      int d = 1 - ri;

      auto PlotOctants = [&](int px, int py)
      {
        struct { int dx; int dy; } pts[8] = { ... };
        for (auto& p : pts)
          if (InQuarter(p.dx, p.dy, quarter))
            SetPixel(hdc, cx + p.dx, cy + p.dy, color);
      };

      PlotOctants(x, y);
      while (x < y)
      {
        if (d < 0) d += 2 * x + 3;
        else       { d += 2 * (x - y) + 5; --y; }
        ++x;
        PlotOctants(x, y);
      }
    }
  }
  ```
  Explanation (chunked):
  - For each radius `ri` from 1..r, the function rasterizes a circle of radius `ri` using a midpoint loop and plots only the pixels that fall into the requested quarter.
  - `PlotOctants` maps the 8-way symmetric offsets into the quarter test and calls `SetPixel` for allowed offsets.
  - This produces a filled effect by drawing many concentric outlines instead of scanning rows.

  Trade-off: visually interesting but more expensive than scanline for large `r` because it draws many circles.

  ### Hermite and Bezier filling: `FillSquareHermite` and `FillRectBezier`

  Both functions iterate across one axis (columns for Hermite square, rows for Bezier rectangle), evaluate a curve at many parameter `t` values per column/row, and set the resulting pixel samples.

  Key lines (Hermite):

  ```cpp
  int steps = (side + 1)*2;
  for (int col = 0; col <= side; col++) { ... for (int s = 0; s <= steps; ++s) { double t = (double)s / steps; HermitePoint(...); SetPixel(...); } }
  ```
  Explanation:
  - Each column defines a Hermite curve from top to bottom using fixed tangents; the code samples `steps+1` points along the Hermite interpolation and sets pixels at the rounded coordinates.

  Key lines (Bezier):

  ```cpp
  int steps = (w + 1)*2;
  for (int row = 0; row <= h; row++) { ... for (int s = 0; s <= steps; ++s) { double t = (double)s / steps; BezierPoint(...); SetPixel(...);} }
  ```
  Explanation:
  - For each scanline row the code defines a cubic Bezier from left to right and samples it; this creates a filled stylized rectangle by scanning rows with curve-guided samples.

  Note: these are artistic fills—each pixel written is sampled on the curve; the samples may leave holes unless sampling density (`steps`) is high enough.

  ### Scanline polygon fill utilities and `ScanlineFill`

  ```cpp
  static void ScanlineFill(HDC hdc, const std::vector<FillPoint>& pts, COLORREF color)
  {
    int n = (int)pts.size();
    if (n < 3) return;
    int ymin = pts[0].y, ymax = pts[0].y;
    for (auto& p : pts) { ymin = std::min(ymin, p.y); ymax = std::max(ymax, p.y); }

    for (int y = ymin; y <= ymax; ++y)
    {
      std::vector<int> xIntersect;
      for (int i = 0; i < n; ++i)
      {
        int j = (i + 1) % n; int yi = pts[i].y, yj = pts[j].y; int xi = pts[i].x, xj = pts[j].x;
        if ((yi <= y && yj > y) || (yj <= y && yi > y)) {
          int x = xi + (y - yi) * (xj - xi) / (yj - yi);
          xIntersect.push_back(x);
        }
      }
      std::sort(xIntersect.begin(), xIntersect.end());
      for (int k = 0; k + 1 < (int)xIntersect.size(); k += 2)
        for (int x = xIntersect[k]; x <= xIntersect[k+1]; ++x)
          SetPixel(hdc, x, y, color);
    }
  }
  ```
  Explanation (stepwise):
  - Compute `ymin,ymax` to bound scanlines.
  - For each scanline, collect x-intersections of polygon edges with that y.
  - Sort intersections and fill between each pair (assumes an even number of intersections per scanline).
  - This is a classic scanline polygon fill; it works for both convex and non-convex polygons if edges are handled consistently.

  ### Flood fill (recursive and iterative)

  Recursive:

  ```cpp
  void FloodFillRecursive(HDC hdc, int x, int y, COLORREF fillColor, COLORREF targetColor)
  {
    COLORREF cur = GetPixel(hdc, x, y);
    if (cur != targetColor) return;
    if (cur == fillColor)   return;
    SetPixel(hdc, x, y, fillColor);
    FloodFillRecursive(hdc, x+1, y,   fillColor, targetColor);
    FloodFillRecursive(hdc, x-1, y,   fillColor, targetColor);
    FloodFillRecursive(hdc, x,   y+1, fillColor, targetColor);
    FloodFillRecursive(hdc, x,   y-1, fillColor, targetColor);
  }
  ```
  Explanation:
  - Standard 4-way recursive flood fill: check current pixel, fill it, and recurse on 4 neighbors. Simple but can overflow the call stack on large regions.

  Iterative (stack-based):

  ```cpp
  void FloodFillIterative(HDC hdc, int x, int y, COLORREF fillColor, COLORREF targetColor)
  {
    if (GetPixel(hdc, x, y) == fillColor) return;
    struct Pt { int x, y; };
    std::stack<Pt> stk; stk.push({x, y});
    while (!stk.empty()) {
      Pt p = stk.top(); stk.pop();
      COLORREF cur = GetPixel(hdc, p.x, p.y);
      if (cur != targetColor) continue;
      SetPixel(hdc, p.x, p.y, fillColor);
      stk.push({p.x+1, p.y}); stk.push({p.x-1, p.y}); stk.push({p.x, p.y+1}); stk.push({p.x, p.y-1});
    }
  }
  ```
  Explanation:
  - Uses an explicit stack to avoid recursion and stack overflow; semantics match the recursive version.

  ---

  ## 12. Line-by-Line Deep Dive — `Clipping.cpp`

  I'll explain `Clipping.cpp` line-by-line and highlight the Cohen–Sutherland and Sutherland–Hodgman pieces.

  ### Headers and OutCode union

  ```cpp
  #include "Clipping.h"
  #include <cmath>
  #include <algorithm>

  using namespace std;

  union OutCode {
    unsigned ALL : 4;
    struct { unsigned left : 1, right : 1, bottom : 1, top : 1; };
  };
  ```
  Explanation:
  - `OutCode` is a 4-bit bitfield representing whether a point is outside the left/right/top/bottom edges. `ALL` provides easy bitwise checks.

  ### `GetOutCode`

  ```cpp
  OutCode GetOutCode(int x, int y, int xmin, int ymin, int xmax, int ymax) {
    OutCode res; res.ALL = 0;
    if (x < xmin) res.left = 1; else if (x > xmax) res.right = 1;
    if (y < ymin) res.top = 1;  else if (y > ymax) res.bottom = 1;
    return res;
  }
  ```
  Explanation:
  - Builds an `OutCode` for a point relative to the rectangular window. Note the top/bottom mapping uses `y < ymin` for top because Windows y increases downward; semantics must match caller conventions.

  ### `ClipLineRect` — Cohen–Sutherland

  Key structure and steps:

  ```cpp
  void ClipLineRect(HDC hdc, int x1, int y1, int x2, int y2, int xmin, int ymin, int xmax, int ymax) {
    OutCode out1 = GetOutCode(x1, y1, xmin, ymin, xmax, ymax);
    OutCode out2 = GetOutCode(x2, y2, xmin, ymin, xmax, ymax);
    while (true) {
      if (!(out1.ALL | out2.ALL)) { MoveToEx(hdc, x1, y1, NULL); LineTo(hdc, x2, y2); break; }
      else if (out1.ALL & out2.ALL) break; 
      else {
        double x, y; OutCode outout = out1.ALL ? out1 : out2;
        if (outout.top) { y = ymin; x = x1 + (double)(x2 - x1) * (ymin - y1) / (y2 - y1); }
        else if (outout.bottom) { y = ymax; x = x1 + (double)(x2 - x1) * (ymax - y1) / (x2 - x1); }
        else if (outout.right) { x = xmax; y = y1 + (double)(y2 - y1) * (xmax - x1) / (x2 - x1); }
        else if (outout.left) { x = xmin; y = y1 + (double)(y2 - y1) * (xmin - x1) / (x2 - x1); }
        if (outout.ALL == out1.ALL) { x1 = (int)x; y1 = (int)y; out1 = GetOutCode(x1, y1, xmin, ymin, xmax, ymax); }
        else { x2 = (int)x; y2 = (int)y; out2 = GetOutCode(x2, y2, xmin, ymin, xmax, ymax); }
      }
    }
  }
  ```
  Explanation (chunked):
  - Compute outcodes and loop until trivially accepted (both inside) or trivially rejected (share an outside bit).
  - When partially outside, pick an outside endpoint, compute intersection with the offending edge, replace that endpoint with intersection, and recompute outcode. Repeat until resolved.
  - Note: some divisions use `(y2 - y1)` or `(x2 - x1)` — calls must avoid division by zero; code relies on geometry to not hit invalid branches, but edge cases exist.

  ### `ClipPointRect` and `ClipPointCircle`

  ```cpp
  void ClipPointRect(...) { if (x >= x1 && x <= x2 && y >= y1 && y <= y2) SetPixel(...); }
  void ClipPointCircle(...) { double distSq = (x-xc)^2+(y-yc)^2; if (distSq <= R*R) SetPixel(...); }
  ```
  Explanation:
  - Simple containment tests: rectangle bounds test and circle distance-squared test.

  ### `ClipLineCircle` — quadratic intersection

  ```cpp
  // build quadratic a*u^2 + b*u + c = 0 where point(u) = P0 + u*(P1-P0)
  // solve discriminant, clamp intersection parameters to [0,1], draw segment between tmin and tmax
  ```
  Explanation:
  - Convert line to parametric `P(u) = P0 + u*(P1-P0)` and substitute into circle equation to get a quadratic in `u`.
  - If discriminant < 0: no intersection. Otherwise compute `u1,u2`, clamp to [0,1] and draw the portion inside the circle using `MoveToEx`/`LineTo`.

  ### Sutherland–Hodgman polygon clipping helpers and `PolygonClip`

  Key pieces:

  ```cpp
  bool InLeft(Point v, int edge) { return v.x >= edge; }
  Point VIntersect(Point v1, Point v2, int xedge) { res.x = xedge; res.y = v1.y + (xedge - v1.x) * (v2.y - v1.y) / (v2.x - v1.x); }

  PointList ClipWithEdge(PointList p, int edge, bool (*IsIn)(Point, int), Point (*Intersect)(Point, Point, int)) { ... }

  void PolygonClip(HDC hdc, PointList p, int xleft, int ytop, int xright, int ybottom) {
    p = ClipWithEdge(p, xleft, InLeft, VIntersect);
    p = ClipWithEdge(p, ytop, InTop, HIntersect);
    p = ClipWithEdge(p, xright, InRight, VIntersect);
    p = ClipWithEdge(p, ybottom, InBottom, HIntersect);
    if (p.empty()) return;
    // Draw polygon edges
  }
  ```
  Explanation:
  - `ClipWithEdge` iterates polygon edges and performs the four Sutherland–Hodgman cases (out→in add intersection+vertex, in→in add vertex, in→out add intersection, out→out add nothing).
  - `PolygonClip` applies `ClipWithEdge` for each rectangle edge in sequence and draws the resulting clipped polygon.

  ---

  ## 13. Line-by-Line Deep Dive — `Curves.cpp`

  ```cpp
  #include "Curves.h"
  #include <cmath>
  ```
  Explanation: header + math for Hermite basis calculations.

  ```cpp
  void DrawCardinalSpline(HDC hdc, const std::vector<CurvePoint>& pts, double tension, int steps, COLORREF color)
  {
    int n = (int)pts.size();
    if (n < 2) return;

    std::vector<CurvePoint> ep;
    ep.push_back({ 2*pts[0].x - pts[1].x, 2*pts[0].y - pts[1].y }); // phantom start
    for (auto& p : pts) ep.push_back(p);
    ep.push_back({ 2*pts[n-1].x - pts[n-2].x, 2*pts[n-1].y - pts[n-2].y }); // phantom end

    double s = (1.0 - tension) * 0.5;

    for (int i = 1; i + 2 < (int)ep.size(); ++i) {
      double p0x = ep[i-1].x, p0y = ep[i-1].y;
      double p1x = ep[i  ].x, p1y = ep[i  ].y;
      double p2x = ep[i+1].x, p2y = ep[i+1].y;
      double p3x = ep[i+2].x, p3y = ep[i+2].y;

      double t1x = s * (p2x - p0x), t1y = s * (p2y - p0y);
      double t2x = s * (p3x - p1x), t2y = s * (p3y - p1y);

      for (int k = 0; k <= steps; ++k) {
        double t  = (double)k / steps;
        double t2 = t * t, t3 = t2 * t;
        double h00 =  2*t3 - 3*t2 + 1;
        double h10 =    t3 - 2*t2 + t;
        double h01 = -2*t3 + 3*t2;
        double h11 =    t3 -   t2;

        double x = h00*p1x + h10*t1x + h01*p2x + h11*t2x;
        double y = h00*p1y + h10*t1y + h01*p2y + h11*t2y;
        SetPixel(hdc, (int)round(x), (int)round(y), color);
      }
    }
  }
  ```
  Explanation:
  - The function extends the control point list with phantom endpoints so the first and last segments have computed tangents.
  - `s = (1.0 - tension) * 0.5` scales tangent magnitude by tension; `tension=0` yields Catmull–Rom-like tangents, `tension=1` tightens the curve.
  - For each segment compute Hermite tangents `t1`, `t2` and sample the cubic Hermite basis at `steps+1` `t` values, evaluating `x,y` and plotting pixels. This produces a smooth curve through the original control points.

  ---

  ## 14. Line-by-Line Deep Dive — `SmileyFace.cpp`

  ```cpp
  #include "SmileyFace.h"
  #include "Circles.h"

  void DrawSmiley(HDC hdc, int xc, int yc, bool isHappy) {
    // 1. Draw Face Outline & Eyes
    CircleMidpoint(hdc, xc, yc, 60, RGB(0, 0, 0));
    CircleMidpoint(hdc, xc - 20, yc - 20, 7, RGB(0, 0, 0));
    CircleMidpoint(hdc, xc + 20, yc - 20, 7, RGB(0, 0, 0));
    // 2. Draw Nose
    MoveToEx(hdc, xc, yc - 5, NULL); LineTo(hdc, xc, yc + 15);
    // 3. Draw Mouth (Arc)
    if (isHappy) { Arc(hdc, xc - 30, yc + 10, xc + 30, yc + 40, xc - 30, yc + 25, xc + 30, yc + 25); }
    else { Arc(hdc, xc - 30, yc + 20, xc + 30, yc + 50, xc + 30, yc + 35, xc - 30, yc + 35); }
  }
  ```
  Explanation (chunked):
  - Draws a face using `CircleMidpoint` for outline and eyes (reuse of circle raster algorithm).
  - Nose: vertical line drawn with `MoveToEx`/`LineTo`.
  - Mouth: uses `Arc` (Win32 GDI) with different source/destination points to draw a smiling or frowning arc. The `Arc` parameters define the bounding rectangle and start/end points of the arc.

  Why it matters: demonstrates composing primitive algorithms (circles, lines, GDI arc) into a single drawable asset.

  ---

  ## 15. Line-by-Line Deep Dive — `Preferences.cpp`

  ```cpp
  #define UNICODE
  #define _UNICODE
  #include "Preferences.h"
  #include <commdlg.h>
  ```
  Explanation:
  - Ensures Unicode APIs and pulls in `commdlg.h` for the color chooser.

  ```cpp
  void PrefSetWhiteBackground(HWND hwnd)
  {
    SetClassLongPtr(hwnd, GCLP_HBRBACKGROUND, (LONG_PTR)GetStockObject(WHITE_BRUSH));
    HDC  hdc  = GetDC(hwnd);
    RECT rect; GetClientRect(hwnd, &rect);
    FillRect(hdc, &rect, (HBRUSH)GetStockObject(WHITE_BRUSH));
    ReleaseDC(hwnd, hdc);
    InvalidateRect(hwnd, NULL, TRUE);
  }
  ```
  Explanation:
  - Updates the window class background brush to `WHITE_BRUSH` so future erases are white, immediately fills the client area with white using `FillRect`, and invalidates the window to force repaint.

  ```cpp
  void PrefSetCursor(HWND hwnd, LPCWSTR cursorId)
  {
    HCURSOR hCur = LoadCursor(NULL, cursorId);
    if (!hCur) return;
    SetClassLongPtr(hwnd, GCLP_HCURSOR, (LONG_PTR)hCur);
    SetCursor(hCur);
  }
  ```
  Explanation:
  - Loads a system cursor by id (e.g., `IDC_ARROW`) and updates the window class cursor so the whole window uses it. `SetCursor` forces immediate update.

  ```cpp
  static COLORREF g_customColors[16] = { ... };
  COLORREF PrefChooseColor(HWND hwnd, COLORREF currentColor)
  {
    CHOOSECOLOR cc = {};
    cc.lStructSize = sizeof(cc); cc.hwndOwner = hwnd; cc.rgbResult = currentColor; cc.lpCustColors = g_customColors; cc.Flags = CC_FULLOPEN | CC_RGBINIT;
    if (ChooseColor(&cc)) return cc.rgbResult;
    return currentColor;
  }
  ```
  Explanation:
  - `g_customColors` provides defaults for the color dialog.
  - `PrefChooseColor` fills a `CHOOSECOLOR` struct, calls the standard `ChooseColor` dialog, and returns the user's selection or the previous color if canceled.

  ---

  ## 16. Header Files — Quick Line-by-Line Reference

  I'll highlight the purpose of each header and the declarations they expose (short form):

  - `Filling.h`: declares `FillPoint` and all fill routines (`FillCircleWithLines`, `FillCircleWithCircles`, `FillSquareHermite`, `FillRectBezier`, `FillConvexPolygon`, `FillNonConvexPolygon`, `FloodFillRecursive`, `FloodFillIterative`). These are the public entry points used by `main.cpp` and `task1_file_menu.cpp`.

  - `Clipping.h`: declares `Point`, `PointList`, and clipping APIs: `ClipPointRect`, `ClipLineRect`, `ClipPointCircle`, `ClipLineCircle`, `ClipWithEdge`, and `PolygonClip`.

  - `Curves.h`: declares `CurvePoint` and `DrawCardinalSpline` with tension and sampling `steps`.

  - `SmileyFace.h`: declares `DrawSmiley(HDC,int,int,bool)` — a composite demo that uses circle and primitive routines.

  - `Preferences.h`: declares preference APIs: `PrefSetWhiteBackground`, `PrefSetCursor`, and `PrefChooseColor`.

  - `Lines.h`: declares `DrawLineDDA`, `DrawLineMidpoint`, and `DrawLineParametric`.

  - `Circles.h`: declares the circle helper `Draw8Points` and all circle variants: `CircleDirect`, `CirclePolar`, `CircleIterativePolar`, `CircleMidpoint`, `CircleModifiedMidpoint`.

  - `task1_file_menu.h`: defines `ShapeType`, `ShapeRecord`, the global `g_shapes`, and the persistence/redraw functions `ClearScreen`, `SaveToFile`, `LoadFromFile`, and `RedrawShapes`.

  - `task5_ellipse_algorithms.h`: declares `DrawEllipseDirect`, `DrawEllipsePolar`, and `DrawEllipseMidpoint` with short comments describing their trade-offs.

  ---



