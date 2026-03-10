# Extension System Design

**Purpose:** How to design extensions that can be added/removed anytime.

---

## Extension Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     reClaw Core                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   Channel   │    │    Agent    │    │   Memory    │     │
│  │   Manager   │    │ Orchestrator│   │   Manager   │     │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘     │
│         │                  │                  │            │
│         └──────────────────┼──────────────────┘            │
│                            │                                 │
│                    ┌──────┴──────┐                          │
│                    │  Extension │                          │
│                    │   Registry  │                          │
│                    └──────┬──────┘                          │
│                           │                                   │
│    ┌──────────────────────┼──────────────────────┐          │
│    │                      │                      │          │
│    ▼                      ▼                      ▼          │
│ ┌─────────┐        ┌─────────┐        ┌─────────┐        │
│ │Multi-   │        │DM       │        │Hybrid   │        │
│ │Account  │        │Policies │        │Search   │        │
│ │Extension│        │Extension│        │Extension│        │
│ └─────────┘        └─────────┘        └─────────┘        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Extension Interface

Every extension implements this interface:

```python
from abc import ABC, abstractmethod
from typing import Any

class ReclawExtension(ABC):
    """Base class for all reClaw extensions"""
    
    # Required properties
    name: str           # "multi-account"
    version: str        # "1.0.0"
    description: str   # Human readable description
    
    # Installation
    @abstractmethod
    async def install(self, config: dict) -> None:
        """Called when extension is added to project"""
        pass
    
    @abstractmethod
    async def uninstall(self) -> None:
        """Called when extension is removed from project"""
        pass
    
    # Runtime
    @abstractmethod
    async def load(self, config: dict) -> None:
        """Called when reClaw starts"""
        pass
    
    @abstractmethod
    async def unload(self) -> None:
        """Called when reClaw stops"""
        pass
    
    # Metadata
    def get_dependencies(self) -> list[str]:
        """Return required pip/npm packages"""
        return []
    
    def get_config_schema(self) -> dict:
        """Return config options for this extension"""
        return {}
```

---

## Extension Registry

```python
class ExtensionRegistry:
    """Manages all installed extensions"""
    
    def __init__(self):
        self.extensions: dict[str, ReclawExtension] = {}
        self.enabled: set[str] = set()
    
    def register(self, extension: ReclawExtension) -> None:
        """Register an extension"""
        self.extensions[extension.name] = extension
    
    def enable(self, name: str) -> None:
        """Enable an extension"""
        if name in self.extensions:
            self.enabled.add(name)
    
    def disable(self, name: str) -> None:
        """Disable an extension"""
        self.enabled.discard(name)
    
    async def load_all(self, config: dict) -> None:
        """Load all enabled extensions"""
        for name in self.enabled:
            await self.extensions[name].load(config)
    
    async def unload_all(self) -> None:
        """Unload all extensions"""
        for name in self.enabled:
            await self.extensions[name].unload()
```

---

## Example: Multi-Account Extension

```python
class MultiAccountExtension(ReclawExtension):
    name = "multi-account"
    version = "1.0.0"
    description = "Support multiple accounts per channel"
    
    async def install(self, config: dict) -> None:
        # 1. Add config template
        # 2. Add to config.yaml
        # 3. Set up account session handling
        pass
    
    async def uninstall(self) -> None:
        # 1. Remove config
        # 2. Clean up sessions
        pass
    
    async def load(self, config: dict) -> None:
        # 1. Load account configs
        # 2. Set up per-account session keys
        pass
    
    async def unload(self) -> None:
        # Clean up
        pass
    
    def get_dependencies(self) -> list[str]:
        return []  # No extra dependencies
    
    def get_config_schema(self) -> dict:
        return {
            "channels": {
                "*": {
                    "accounts": {
                        "type": "map",
                        "properties": {
                            "token": {"type": "string"},
                            "allowFrom": {"type": "list"}
                        }
                    }
                }
            }
        }
```

---

## Extension Discovery

### Auto-discover
```python
# Extensions are auto-discovered from extensions/ folder
EXTENSIONS_DIR = "reclaw/extensions"

def discover_extensions():
    for filename in os.listdir(EXTENSIONS_DIR):
        if filename.endswith(".py"):
            module = import_module(f"reclaw.extensions.{filename[:-3]}")
            for name in dir(module):
                obj = getattr(module, name)
                if isinstance(obj, type) and issubclass(obj, ReclawExtension):
                    registry.register(obj())
```

### Manual Registration
```python
# Or register manually in main
from reclaw.extensions.multi_account import MultiAccountExtension
from reclaw.extensions.dm_policies import DmPoliciesExtension

registry.register(MultiAccountExtension())
registry.register(DmPoliciesExtension())
```

---

## Config Integration

Extensions are configured in config.yaml:

```yaml
extensions:
  enabled:
    - multi-account
    - dm-policies
  
  multi-account:
    default_account: default
  
  dm-policies:
    default_policy: pairing
    allowlist:
      - "+15555551234"
```

---

## File Structure

```
reclaw/
├── src/
│   ├── core/
│   │   ├── main.py
│   │   ├── registry.py      # Extension registry
│   │   └── ...
│   │
│   └── extensions/
│       ├── __init__.py      # Auto-discover
│       ├── base.py          # Extension interface
│       ├── multi_account.py
│       ├── dm_policies.py
│       ├── hybrid_search.py
│       └── edge_tts.py
│
├── config.yaml              # Extension config
└── extensions/               # Docs for extensions
    ├── multi-account.md
    ├── dm-policies.md
    └── ...
```

---

## Loading Order

1. Load config.yaml
2. Discover all extensions
3. Enable/load based on config
4. Initialize core components
5. Start message processing

```
Config → Registry → Extensions → Core → Start
```

---

## Adding New Extension

### 1. Create extension file

```python
# src/extensions/my_extension.py
from .base import ReclawExtension

class MyExtension(ReclawExtension):
    name = "my-extension"
    version = "1.0.0"
    description = "Description here"
    
    async def install(self, config): ...
    async def uninstall(self): ...
    async def load(self, config): ...
    async def unload(self): ...
```

### 2. Add config schema

```python
def get_config_schema(self):
    return {
        "my_extension": {
            "option1": {"type": "string"},
            "option2": {"type": "boolean"}
        }
    }
```

### 3. Create docs

```markdown
# My Extension

Description of what it does...
```

### 4. Update skill

Add to `skills/reclaw-extensions/SKILL.md`