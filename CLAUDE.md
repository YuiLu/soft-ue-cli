# Unreal Engine Control — soft-ue-cli

This project uses **soft-ue-cli** to control a running Unreal Engine 5 editor via the SoftUEBridge plugin.

> **Prerequisites**: UE editor must be running with the SoftUEBridge plugin enabled (default port 8080).

## Quick Reference

```bash
soft-ue-cli --help                    # List all 60+ commands
soft-ue-cli <command> --help          # Detailed usage for any command
soft-ue-cli check-setup               # Verify plugin + bridge connectivity
soft-ue-cli status                    # Quick health check
```

## Command Categories

### 🎭 Actor Operations
```bash
soft-ue-cli spawn-actor <class> --location X,Y,Z [--rotation P,Y,R] [--label NAME]
soft-ue-cli query-level [--class-filter CLASS] [--search "Pattern*"] [--components]
soft-ue-cli set-property <actor> <prop> <value>
soft-ue-cli get-property <actor> <prop>
soft-ue-cli call-function <actor> <function> [--args '{"key": val}']
soft-ue-cli batch-spawn-actors --actors '[{...}]'
soft-ue-cli batch-modify-actors --modifications '[{...}]'
soft-ue-cli batch-delete-actors --actors '["Name1","Name2"]'
```

### 📐 Blueprint
```bash
soft-ue-cli query-blueprint <path> [--include functions,variables,components,defaults,graph,all]
soft-ue-cli query-blueprint-graph <path> [--node-guid GUID] [--callable-name NAME]
soft-ue-cli add-graph-node <path> <node_class> [--graph-name NAME] [--properties '{}']
soft-ue-cli connect-graph-pins <path> <src_guid> <src_pin> <dst_guid> <dst_pin>
soft-ue-cli disconnect-graph-pin <path> <node_guid> <pin_name>
soft-ue-cli insert-graph-node <path> <node_class> <src_guid> <src_pin> <dst_guid> <dst_pin>
soft-ue-cli remove-graph-node <path> <node_guid>
soft-ue-cli set-node-position <path> --positions '[{"guid":"...","x":0,"y":0}]'
soft-ue-cli set-node-property <path> <node_guid> '{"prop": val}'
soft-ue-cli modify-interface <path> <interface_class> --action add|remove
soft-ue-cli compile-blueprint <path>
soft-ue-cli save-asset <path>
```

### 🎨 Material
```bash
soft-ue-cli query-material <path> [--include graph,parameters,all]
soft-ue-cli query-mpc <path> [--write --param NAME --value VAL]
soft-ue-cli compile-material <path>
```

### 📦 Asset Management
```bash
soft-ue-cli query-asset --query "BP_*" [--class Blueprint] [--path /Game/...]
soft-ue-cli query-asset --asset-path /Game/Data/DT_Items
soft-ue-cli create-asset <path> <class> [--parent-class Actor] [--skeleton PATH]
soft-ue-cli set-asset-property <path> <prop> --value <val> [--component-name NAME]
soft-ue-cli open-asset <path>
soft-ue-cli delete-asset <path>
soft-ue-cli get-asset-diff <path>
soft-ue-cli get-asset-preview <path>
soft-ue-cli find-references --asset-path <path>
soft-ue-cli query-enum <path>
soft-ue-cli query-struct <path>
soft-ue-cli add-datatable-row <path> --row-name NAME --values '{...}'
```

### 🌳 StateTree
```bash
soft-ue-cli query-statetree <path> [--include states,transitions,tasks,all]
soft-ue-cli add-statetree-state <path> --name NAME [--parent-state PARENT]
soft-ue-cli add-statetree-task <path> --state NAME --task-class CLASS
soft-ue-cli add-statetree-transition <path> --source STATE --target STATE
soft-ue-cli remove-statetree-state <path> --name NAME
```

### 🖼️ Widget / UI
```bash
soft-ue-cli inspect-widget-blueprint <path>
soft-ue-cli inspect-runtime-widgets [--name NAME] [--class CLASS]
soft-ue-cli add-widget <path> <widget_class> [--parent NAME] [--slot-properties '{}']
```

### 🎬 Animation
```bash
soft-ue-cli inspect-anim-instance <actor> [--include state-machines,montages,all]
```

### 📸 Visual Confirmation
```bash
soft-ue-cli capture-screenshot viewport          # Editor viewport
soft-ue-cli capture-viewport [--source editor|game]
soft-ue-cli set-viewport-camera --preset top|front|right|perspective
soft-ue-cli set-viewport-camera --location X,Y,Z --rotation P,Y,R
```

### ▶️ Play-In-Editor (PIE)
```bash
soft-ue-cli pie-session start|stop|pause|resume|get-state
soft-ue-cli pie-tick --frames N [--delta-seconds 0.016]
soft-ue-cli trigger-input --action press --key SpaceBar
soft-ue-cli trigger-input --action move-to --location X,Y,Z
```

### ⚙️ Configuration
```bash
soft-ue-cli config tree [--format ini|xml|project]
soft-ue-cli config get "[Section]Key" [--trace] [--layer LAYER]
soft-ue-cli config set "[Section]Key" "Value" --layer ProjectDefault
soft-ue-cli config diff --layers ProjectDefault Base
soft-ue-cli config audit
```

### 📊 Profiling & Debugging
```bash
soft-ue-cli get-logs [--lines 50] [--filter error] [--category LogAI]
soft-ue-cli get-console-var <name>
soft-ue-cli set-console-var <name> <value>
soft-ue-cli insights-capture start|stop|status [--channels cpu,gpu,memory]
soft-ue-cli insights-list-traces
soft-ue-cli insights-analyze <trace_path>
```

### ⏪ Rewind Debugger (Animation Recording)
```bash
soft-ue-cli rewind-start [--channels skeletal-mesh,montage]
soft-ue-cli rewind-stop
soft-ue-cli rewind-status
soft-ue-cli rewind-list-tracks
soft-ue-cli rewind-overview <actor>
soft-ue-cli rewind-snapshot <actor> --time 2.5
soft-ue-cli rewind-save [--file path.utrace]
```

### 🏗️ Build & Code
```bash
soft-ue-cli build-and-relaunch                   # Full rebuild + relaunch editor
soft-ue-cli trigger-live-coding                  # Live Coding hot reload (Windows)
soft-ue-cli run-python-script --script "print('hello')"
soft-ue-cli class-hierarchy <class> [--direction parents|children|both]
soft-ue-cli get-project-info [--include-settings]
```

### 🔧 Offline Inspection (no editor needed)
```bash
soft-ue-cli inspect-uasset <file.uasset> [--sections all]
soft-ue-cli diff-uasset <old.uasset> <new.uasset> [--sections all]
```

### 🧠 Skills (Built-in Workflow Prompts)
```bash
soft-ue-cli skills list                          # List all available skills
soft-ue-cli skills get <name>                    # Get full skill prompt
```

Available skills: `level-from-image`, `blueprint-to-cpp`, `replay-changes`, `test-tools`, `author-test`, `author-anim-state-test`, `author-bp-parity-test`, `author-invariant-test`, `author-regression-test`, `run-test`.

### 🔄 Batch Operations
```bash
soft-ue-cli batch-call --calls '[{"tool":"query-level","args":{}},{"tool":"get-logs","args":{}}]'
soft-ue-cli batch-call --calls-file scenarios.json --continue-on-error
```

## Workflow Conventions

1. **All commands output JSON** — exit code 0 = success, 1 = error.
2. **Always capture a screenshot after visual changes** to verify results:
   ```bash
   soft-ue-cli capture-screenshot viewport
   ```
3. **Save assets after mutations** (add-graph-node, set-asset-property, etc.):
   ```bash
   soft-ue-cli save-asset /Game/Blueprints/BP_Player
   ```
4. **Compile after Blueprint edits**:
   ```bash
   soft-ue-cli compile-blueprint /Game/Blueprints/BP_Player
   ```
5. **Coordinate values are in cm** (Unreal units). Default origin is (0,0,0).
6. **Rotation is in degrees**: Pitch, Yaw, Roll order.
7. **Bridge server** runs at `http://127.0.0.1:8080` by default.
8. When unsure about a command, run `soft-ue-cli <command> --help` first.
9. Use `soft-ue-cli skills get <name>` to load a built-in workflow for complex tasks.
