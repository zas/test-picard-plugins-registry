# Test Plugin Registries

This repository contains multiple registry files for testing Picard plugin system features.

## Quick Start

```bash
# Test multi-ref auto-selection
export PICARD_PLUGIN_REGISTRY_URL="https://raw.githubusercontent.com/zas/test-picard-plugins-registry/refs/heads/main/registry-multi-ref.json"
picard plugins --refresh-registry --browse
picard plugins --install view-script-variables
# Should show: "Found View script variables in registry (using ref: main)"

# Test blacklist
export PICARD_PLUGIN_REGISTRY_URL="https://raw.githubusercontent.com/zas/test-picard-plugins-registry/refs/heads/main/registry-blacklist.json"
picard plugins --refresh-registry --check-blacklist https://github.com/badactor/malicious-plugin
# Should show: ✗ URL is blacklisted

# Reset to default
unset PICARD_PLUGIN_REGISTRY_URL
picard plugins --refresh-registry
```

**Tip:** `--refresh-registry` can be combined with other commands (e.g., `--refresh-registry --browse`) to refresh the cache and immediately use the new registry.

---

## Registry Files

### `registry.json` (Default)
Single plugin with basic refs format.

```bash
export PICARD_PLUGIN_REGISTRY_URL="https://raw.githubusercontent.com/zas/test-picard-plugins-registry/refs/heads/main/registry.json"
```

**Contents:**
- 1 plugin: view-script-variables (trusted)
- Single ref: main (API 3.0+)

---

### `registry-multi-ref.json`
Test multi-ref auto-selection based on Picard version.

```bash
export PICARD_PLUGIN_REGISTRY_URL="https://raw.githubusercontent.com/zas/test-picard-plugins-registry/refs/heads/main/registry-multi-ref.json"
```

**Contents:**
- 1 plugin with 3 refs:
  - `main` - For Picard 4.x (min: 4.0)
  - `beta` - Beta testing (min: 3.0)
  - `picard-v3` - Stable for Picard 3.x (min: 3.0, max: 3.99)

**Test:**
```bash
# Picard 3.x should auto-select 'picard-v3'
picard plugins --install view-script-variables

# Explicit ref override
picard plugins --install view-script-variables --ref beta
```

**Expected:** Picard 3.x auto-selects `picard-v3`, Picard 4.x auto-selects `main`

---

### `registry-blacklist.json`
Test blacklist functionality.

```bash
export PICARD_PLUGIN_REGISTRY_URL="https://raw.githubusercontent.com/zas/test-picard-plugins-registry/refs/heads/main/registry-blacklist.json"
```

**Contents:**
- 4 blacklist entries:
  - URL: `https://github.com/badactor/malicious-plugin`
  - URL regex: `^https://github\.com/badorg/.*`
  - UUID: `blacklisted-uuid-1234-5678-90ab-cdef`
  - Combined URL+UUID

**Test:**
```bash
picard plugins --check-blacklist https://github.com/badactor/malicious-plugin
# Expected: ✗ URL is blacklisted: Contains malicious code

picard plugins --check-blacklist https://github.com/badorg/any-plugin
# Expected: ✗ URL is blacklisted: Entire organization blacklisted

picard plugins --check-blacklist https://github.com/safe/plugin
# Expected: ✓ URL is not blacklisted
```

---

### `registry-trust-levels.json`
Test different trust levels and warnings.

```bash
export PICARD_PLUGIN_REGISTRY_URL="https://raw.githubusercontent.com/zas/test-picard-plugins-registry/refs/heads/main/registry-trust-levels.json"
```

**Contents:**
- 3 plugins with different trust levels:
  - `view-script-variables` - trusted
  - `official-plugin-example` - official
  - `community-plugin-example` - community

**Test:**
```bash
# Browse by trust level
picard plugins --browse --trust official
picard plugins --browse --trust community

# Install with different trust levels
picard plugins --install official-plugin-example        # No warning
picard plugins --install community-plugin-example       # Warning shown
picard plugins --install community-plugin-example --trust-community  # Skip warning
```

---

### `registry-comprehensive.json`
All features combined.

```bash
export PICARD_PLUGIN_REGISTRY_URL="https://raw.githubusercontent.com/zas/test-picard-plugins-registry/refs/heads/main/registry-comprehensive.json"
```

**Contents:**
- 3 plugins (official, trusted, community)
- Multi-ref support
- Blacklist entries

**Test:**
```bash
picard plugins --browse
picard plugins --install view-script-variables          # Auto-selects ref
picard plugins --install community-plugin --ref beta    # Explicit ref
picard plugins --check-blacklist https://github.com/badactor/malicious
```

---

## Testing Workflow

### 1. Basic Registry
```bash
export PICARD_PLUGIN_REGISTRY_URL="https://raw.githubusercontent.com/zas/test-picard-plugins-registry/refs/heads/main/registry.json"
picard plugins --browse
picard plugins --install view-script-variables
```

### 2. Multi-Ref Auto-Selection
```bash
export PICARD_PLUGIN_REGISTRY_URL="https://raw.githubusercontent.com/zas/test-picard-plugins-registry/refs/heads/main/registry-multi-ref.json"
picard plugins --refresh-registry
picard plugins --install view-script-variables
# Check which ref was selected in output
```

### 3. Blacklist
```bash
export PICARD_PLUGIN_REGISTRY_URL="https://raw.githubusercontent.com/zas/test-picard-plugins-registry/refs/heads/main/registry-blacklist.json"
picard plugins --refresh-registry
picard plugins --check-blacklist https://github.com/badactor/malicious-plugin
picard plugins --check-blacklist https://github.com/badorg/anything
```

### 4. Trust Levels
```bash
export PICARD_PLUGIN_REGISTRY_URL="https://raw.githubusercontent.com/zas/test-picard-plugins-registry/refs/heads/main/registry-trust-levels.json"
picard plugins --refresh-registry
picard plugins --browse --trust official
picard plugins --browse --trust community
picard plugins --install community-plugin-example
```

### 5. Comprehensive
```bash
export PICARD_PLUGIN_REGISTRY_URL="https://raw.githubusercontent.com/zas/test-picard-plugins-registry/refs/heads/main/registry-comprehensive.json"
picard plugins --refresh-registry
picard plugins --browse
picard plugins --install view-script-variables
picard plugins --install community-plugin --ref beta
```

---

## Notes

- All registries use the refs format (no top-level min_api_version)
- The actual plugin repository: https://github.com/rdswift/picard-v3-plugins
- Only view-script-variables is a real plugin, others are examples
- Use `--force-blacklisted` to bypass blacklist (testing only!)
- Use `--trust-community` to skip community plugin warnings
- Registry cache is stored per URL hash
- Always run `--refresh-registry` after changing `PICARD_PLUGIN_REGISTRY_URL`
