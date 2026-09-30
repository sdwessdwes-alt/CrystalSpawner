========================================================================
              Object Controller - User Manual
========================================================================

Version: v3.1.0
Author: expzzzz
Platform: BepInEx 5.x
Plugin GUID: com.dynroot.tree

------------------------------------------------------------------------
1. Overview
------------------------------------------------------------------------

This is a server-side plugin for multiplayer that spawns and drives
in-game objects based on an editable node tree. You can manually
place objects, generate large batches from text or images, brush
objects directly with your mouse, and make objects loop along a path
with custom rotations.

Spawning and driving only happen when you are the server and the game
world is generated. In single-player, no objects are spawned.

------------------------------------------------------------------------
2. Hotkeys
------------------------------------------------------------------------

These keys work anywhere in the game. They can be changed in the
Settings tab.

  F5          Spawn / Apply
              Edit tab: apply current parameters to selected nodes
              Print tab: start printing

  F9          Toggle panel (show / hide the whole window)

  F7          Clock pause / resume (works on all tabs except Settings)

  F8          Cancel task
              If printing, cancel it; otherwise clear the task state

  [           Step clock back one frame (1/60 second)
  ]           Step clock forward one frame (1/60 second)

  Backspace   Reset clock
              Clock = 0, all node offsets and pause states cleared

Arrow keys, Q, E are used for fine-tuning nodes (see Edit tab).
They do not work while a text field has focus.

On the Settings tab, only F9 is captured; other hotkeys are passed
through to the game.

------------------------------------------------------------------------
3. Interface Overview
------------------------------------------------------------------------

Press F9 to open the window. It has four regions:

  · Top: three tab buttons (Edit / Print / Settings); right side shows
    clock status and loop toggle

  · Left: tree panel, containing (top to bottom): World tree,
    Staging tree, Tree operations, Time control, Print control

  · Right: contents of the current tab, scrollable

  · Bottom status bar: current task, world object count, selected
    node name

------------------------------------------------------------------------
4. Nodes and Trees
------------------------------------------------------------------------

4.1 Two Trees

  World tree: objects here are actually spawned and driven in-game.
              Each node represents an object or a group.

  Staging tree: nothing is spawned. It is a template repository for
                saving your structures. Copy nodes from the world
                tree to save them here; move them back when needed.

4.2 Node Types

  Every node has one of two types:

  · Group node (empty prefab): does not spawn anything, only used to
    organize children. Can hold any children.

  · Leaf node (non-empty prefab): spawns the corresponding object.
    Cannot hold children.

4.3 Tree Panel Operations

  Click              select node
  Ctrl + click       add / remove from selection
  Shift + click      range select (from last anchor to current,
                     requires same parent)
  Double-click       select node and all its children
  Click ▾ / ▸        collapse / expand children

  Badges that may appear after a node name:

  ⏸        node is paused (time frozen)
  [Free]   node is not driven, fully controlled by game physics
  [Break]  node will not respawn after being destroyed
  [xxx]    prefab name inside square brackets

4.4 Node Properties

  Name            Display name, also used in spawned object names
  Prefab          Object type to spawn; empty means group node
  Pos sequence    List of position keyframes
  Rot sequence    List of rotation keyframes
  Pivot           Rotation axis point (local coordinates)
  Offset          Time offset relative to the global clock
  X/Y Scale       Object scale; parent scale multiplies to children

  Important: if Pos sequence has data, posX/posY is ignored; the
  sampler only reads the sequence. Same for rotation. Scalar values
  are only used when the sequence has 0 or 1 keyframes.

------------------------------------------------------------------------
5. Left Panel
------------------------------------------------------------------------

5.1 World Tree

  Shows all nodes in the world that spawn objects. The number at the
  top right is the total leaf count.

5.2 Staging Tree

  Shows saved structure templates. The ⊞ button expands all nodes;
  ⊟ collapses all. This is a pure repository — nothing spawns from
  here.

5.3 Tree Operations

  Merge      Wrap selected sibling nodes into a new group node.
             The group's position is the average of the selected
             nodes.
             Requirement: same level, not the root.

  Delete     Delete selected nodes and all their children.

  Copy       Duplicate selected nodes; the copy is offset by +1.5
             on the X axis.
             Requirement: not the root.

  Move       Two-step:
             First click  = detach selected nodes
             Second click = attach to target node
             Requirement: target must be a group node.

  Pause      Toggle pause. When paused, time is frozen; when
             resumed, phase continues from where it was paused.

5.4 Time Control

  Play / Pause     Toggle whether the global clock advances
  Reset (↺)        Clock = 0; all node offsets and pauses cleared
  [ / ]            Step one frame back / forward
  Clock display    Current clock in seconds
  Loop toggle      When on, objects wrap to start after their
                   duration ends. When off, they stop at the end.
  Speed            Clock multiplier, range 0.1 ~ 16

5.5 Print Control

  Print         Start a print task
  Pause/Resume  Pause or resume an ongoing print
  Cancel        Abort the print
  Status text   Idle / Running / Paused

------------------------------------------------------------------------
6. Edit Tab
------------------------------------------------------------------------

The Edit tab is used to manually create and modify nodes.

6.1 Top Toggles

  Modify mode    When on, edits modify selected nodes
                 When off, you are in spawn mode

  Spawn mode     When on (Modify off), F5 creates a new node under
                 the selected node

  Auto           When on, changes to editor fields apply to
                 selected nodes immediately (no need to press F5)

6.2 Basic Info

  Name           Node name
  Prefab         Object type to spawn
                 Can be left empty in spawn mode for a group node
  Free physics   When on, this node is not driven; game physics
                 controls it
  Breakable      When on, the object will not respawn after being
                 destroyed

6.3 Position Sequence

  Each row is one keyframe:

       #  |  (X,Y)  |  Duration

  · Click the # button to select that row
  · (X,Y): position of the object at this frame (relative to parent)
  · Duration: time in seconds to move from this frame to the next
  · The last keyframe does not show a duration field; instead it
    shows the (End) label, because the last frame's duration is
    not part of the total time

  Total duration = sum of durations of the first n-1 keyframes

  Quick positioning: click a (X,Y) field to focus it, then right-
  click on the game canvas. The row's position will be set to the
  cursor location.

6.4 Rotation Sequence

  Each row is one keyframe:

       #  |  Angle  |  Dir  |  Duration

  · Angle: orientation in degrees
  · Dir: click to cycle through three modes
        CW   = rotate clockwise from this angle to the next
        CCW  = rotate counter-clockwise
        Sht  = take the shortest arc
  · The last keyframe also shows (End), with no Dir or Duration

  Quick rotation:
      Q                rotate left 5°
      E                rotate right 5°
      Q / E + Shift    step becomes 30°
      Q / E + Ctrl     step becomes 1°

6.5 Transform Parameters

  Pivot       Rotation axis point; Zero resets to (0,0)
  Offset      Time offset of this node; Zero resets to 0
  X/Y Scale   Object scale; slider 0~10 or type a value directly

6.6 Arrow Key Fine-Tuning

  With a node selected (and no input field focused), arrow keys
  move the position:

      Key           Default   Shift    Ctrl
      Arrow keys    0.5       2.0      0.1

  If a position sequence exists, all keyframes move together;
  otherwise posX/posY is changed.

6.7 Apply

  Click Apply, or press F5:

  Modify mode: write editor parameters into all selected nodes
  Spawn mode:  create a new node under the selected node
               if "spawn at cursor" is on, the new node is placed
               at the cursor's last right-click point

------------------------------------------------------------------------
7. Print Tab
------------------------------------------------------------------------

The Print tab generates large batches of objects from text or images.

7.1 Input Source

  Text mode:
      Content     Text to print
      Font        System font name; falls back to Arial
      Size        Font size
      Spacing     Character spacing
      Threshold   Pixel grayscale below this counts as a "dot",
                  default 0.5

  Image mode:
      Path        Absolute path of the image file
      Threshold   Same as above

  Invert toggles black and white.

7.2 Processing Methods

  Three methods correspond to three object layouts:

  [Dot Matrix]
      Split image into blocks. If the black ratio inside a block
      exceeds the fill threshold, spawn one object at the block
      center.

      Parameters:
          Step         Block side length (pixels)
          Fill         Minimum black ratio inside a block
          Per frame    Max spawns per frame
          Keep ratio   When on, use the smaller block dimension

  [Rectangle Cover]
      Greedily cover black regions with rectangles until coverage
      reaches the target.

      Parameters:
          Coverage     Target covered ratio, default 0.98
          Angle step   Angle interval for search, default 5°
          Max count    Max rectangles
          White        Max allowed white pixels per rectangle
          Min size     Minimum rectangle side length
          Per frame    Max spawns per frame

  [Fast Outline]
      Split image into connected components first; process each
      region. Optionally pre-split rectangles first.

      Parameters:
          Coverage     Same as above
          Angle step   Same as above
          Max count    Same as above
          Pre-split    Pre-generate N rectangles before covering
          Min area     Components smaller than this are skipped
          Per frame    Same as above

  Tip: Rectangle Cover and Fast Outline run on a worker thread
       and will not freeze the UI. Dot Matrix runs synchronously
       on the main thread — large images may lag for a few
       seconds.

7.3 Output Settings

  Prefab            Object type to spawn. Six built-in types,
                    or "Custom" for a manual name and size.
  Keep ratio        Scale uniformly by the smaller ratio
  Print width       World width the image is mapped to
  Center            World position (X,Y) of the image center
  Item adjust       Format: sx,sy rot
                    Example: "1,1 0"    no extra scale or rotation
                             "0.5,2 45" X to 0.5, Y to 2, +45°

7.4 Execution

  Print     Start task (errors if one is already running)
  Pause     Temporarily stop spawning
  Resume    Continue spawning
  Cancel    Abort task; if nothing was spawned yet, the empty
            group node is cleaned up

  Each print creates a group node named Print_HHmmss in the world
  tree. All spawned leaves hang under it.

------------------------------------------------------------------------
8. Brush
------------------------------------------------------------------------

The brush lets you quickly paint objects onto the canvas.

Requirements:
    · "Enable brush" is checked
    · No print task is running
    · Mouse is not inside the panel window

Usage:
    Hold the left mouse button and drag on the game canvas.
    A group node named Brush_HHmmss is created automatically
    when you begin; it is saved when you release the button.

Parameters:
    Spacing     Minimum distance between two objects
    Size        Scale multiplier (supports decimals, e.g. 0.5)
    Max         Maximum objects per drag

The brush uses the prefab and item-adjust values from the Print tab.

------------------------------------------------------------------------
9. Settings Tab
------------------------------------------------------------------------

9.1 Appearance

  Font size     9~16; affects all UI text and control heights
  Spacing       0.2~2.5; panel spacing multiplier
  Alpha         0.3~1; overall window opacity
  Window W/H    Enter a number or "Auto"
  Language      Chinese / English; changes apply immediately

  Click Save to write to the config file. The title shows "Save *"
  when there are unsaved changes. Reset restores appearance only
  (not language).

9.2 Hotkeys

  Shows all current bindings. Click a key name to enter capture
  mode; press any key to rebind, or ESC to cancel.

9.3 Log

  Shows recent messages and errors. Max 30 messages, 20 errors.
  Entries fade out after 8 seconds, down to 35% opacity.
  Click Clear to remove all logs.

------------------------------------------------------------------------
10. Common Workflows
------------------------------------------------------------------------

10.1 Create a Moving Object

  1. Edit tab, switch to "Spawn mode"
  2. Name = Cube, Prefab = oxygencrystal
  3. Click the world tree's root node, press F5
  4. Select the Cube you just created, enable "Auto", switch to
     "Modify mode"
  5. Add a few position keyframes, press F7 to play the clock

10.2 Print Text as a Wall

  1. Print tab, choose "Text" as input source
  2. Enter text, tune font size and threshold
  3. Method = Dot Matrix, Step = 4, Fill = 0.5
  4. Output: Prefab = oxygencrystal, Print width = 20,
     Center = 0,0
  5. Press F5 to start

10.3 Save a Structure to Staging

  1. Select a group of nodes in the world tree (Ctrl / Shift)
  2. Click "Copy" to duplicate them
  3. Click any node in the staging tree to make it the active tree
  4. Click "Move", then click the staging root to attach the copies

10.4 Paint Objects Quickly

  1. Print tab: choose prefab and item-adjust
  2. Check "Enable brush"; tune spacing and size
  3. Close the panel and hold left mouse button on the canvas

------------------------------------------------------------------------
11. Notes
------------------------------------------------------------------------

  · All tree modifications (add / delete / edit / move / tweak)
    are saved immediately.

  · Node IDs are not saved to disk; they are reassigned on load.

  · Rectangle-cover prints are not reproducible run-to-run due to
    randomized seeds. This is expected.

  · With thousands of leaf nodes, game frame rate may suffer.
    Consider enabling "Free physics" for objects that don't need
    precise control.

  · Logs keep 30 messages and 20 errors max; older ones are
    discarded.

  · Hotkeys do not trigger while a text field has focus.

------------------------------------------------------------------------
12. Config Files
------------------------------------------------------------------------

  BepInEx/Config/DynRoot/theme.json          Window, font, language
  BepInEx/Config/DynRoot/trees/world.json    World tree
  BepInEx/Config/DynRoot/trees/staging.json  Staging tree

========================================================================