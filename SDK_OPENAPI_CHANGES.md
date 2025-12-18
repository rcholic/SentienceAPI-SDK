# SDK OpenAPI Changes Assessment

**File:** `/Volumes/ExtremeSSD/Code/SentienceAPI/SDK-SentienceAPI/openapi.json`
**Date:** 2025-12-17
**Status:** 📋 Assessment Complete - Changes Needed

---

## Overview

The SDK OpenAPI file requires **identical schema changes** to the Gateway OpenAPI, but with one key difference:

**Authentication Method:**
- Gateway: Uses `bearerAuth` (JWT tokens)
- SDK: Uses `apiKeyAuth` (API keys in header)

**Schema Changes:** Same for both files (ObserveOptions, InteractableElement, VisualCues)

---

## Required Changes Summary

### 1. Add `include_visual_cues` to ObserveOptions
**Location:** Line 238-258 (`components.schemas.ObserveOptions`)

### 2. Add new fields to InteractableElement
**Location:** Line 510-560 (`components.schemas.InteractableElement`)

### 3. Create VisualCues schema (NEW)
**Location:** After `ElementStates` (around line 574)

---

## Detailed Changes

### Change 1: Update `ObserveOptions` Schema

**Current (lines 238-258):**
```json
"ObserveOptions": {
  "type": "object",
  "description": "Fine-tuning options.",
  "properties": {
    "render_quality": {
      "$ref": "#/components/schemas/RenderQuality"
    },
    "limit": {
      "type": "integer",
      "minimum": 10,
      "maximum": 500,
      "description": "Max elements to return (Map/Visual mode)."
    },
    "filter": {
      "$ref": "#/components/schemas/ElementFilter"
    },
    "viewport": {
      "$ref": "#/components/schemas/Viewport"
    }
  }
}
```

**Updated (add after `viewport` property):**
```json
"ObserveOptions": {
  "type": "object",
  "description": "Fine-tuning options.",
  "properties": {
    "render_quality": {
      "$ref": "#/components/schemas/RenderQuality"
    },
    "limit": {
      "type": "integer",
      "minimum": 10,
      "maximum": 500,
      "description": "Max elements to return (Map/Visual mode)."
    },
    "filter": {
      "$ref": "#/components/schemas/ElementFilter"
    },
    "viewport": {
      "$ref": "#/components/schemas/Viewport"
    },
    "include_visual_cues": {
      "type": "boolean",
      "default": false,
      "description": "Include minimal visual_cues object (4 fields: background_color_name, color_name, cursor, is_primary). Adds ~17 tokens per element. Useful for icon-heavy UIs and design tools."
    }
  }
}
```

---

### Change 2: Update `InteractableElement` Schema

**Current (lines 510-560):**
```json
"InteractableElement": {
  "type": "object",
  "required": [
    "id",
    "tag",
    "role",
    "bbox",
    "is_visible"
  ],
  "properties": {
    "id": { "type": "integer" },
    "uid": { "type": "string", "nullable": true },
    "tag": { "type": "string" },
    "role": { "type": "string" },
    "text": { "type": "string" },
    "selector": { "type": "string" },
    "bbox": { "$ref": "#/components/schemas/BoundingBox" },
    "is_visible": { "type": "boolean" },
    "z_index": { "type": "integer" },
    "attributes": {
      "type": "object",
      "additionalProperties": {
        "type": "string",
        "nullable": true
      }
    },
    "states": {
      "$ref": "#/components/schemas/ElementStates",
      "nullable": true
    }
  }
}
```

**Updated (add new fields after `z_index` and update `attributes` description):**
```json
"InteractableElement": {
  "type": "object",
  "required": [
    "id",
    "tag",
    "role",
    "bbox",
    "is_visible"
  ],
  "properties": {
    "id": { "type": "integer" },
    "uid": { "type": "string", "nullable": true },
    "tag": { "type": "string" },
    "role": { "type": "string" },
    "text": { "type": "string" },
    "selector": { "type": "string" },
    "bbox": { "$ref": "#/components/schemas/BoundingBox" },
    "is_visible": { "type": "boolean" },
    "z_index": { "type": "integer" },
    "in_viewport": {
      "type": "boolean",
      "description": "Whether element is within the visible viewport (Phase 1 feature)"
    },
    "is_occluded": {
      "type": "boolean",
      "description": "Whether element is covered by another element, detected via elementFromPoint raycasting. Always included (Phase 1 feature)."
    },
    "attributes": {
      "type": "object",
      "description": "Element attributes. URL attributes (href, src, action) are resolved to absolute URLs.",
      "additionalProperties": {
        "type": "string",
        "nullable": true
      }
    },
    "states": {
      "$ref": "#/components/schemas/ElementStates",
      "nullable": true
    },
    "visual_cues": {
      "$ref": "#/components/schemas/VisualCues",
      "nullable": true,
      "description": "Visual styling hints for icon recognition and prominence detection. Only included when include_visual_cues=true in request options."
    }
  }
}
```

---

### Change 3: Add `VisualCues` Schema (NEW)

**Location:** Insert after `ElementStates` schema (after line 574)

```json
"VisualCues": {
  "type": "object",
  "description": "Minimal visual styling hints for icon-only button recognition and action prioritization. Only 4 high-value fields to minimize token overhead.",
  "properties": {
    "background_color_name": {
      "type": "string",
      "description": "Human-readable background color name for semantic understanding (e.g., 'blue' = primary action, 'red' = delete/danger, 'green' = success)",
      "example": "blue",
      "enum": [
        "black", "white", "red", "blue", "green", "yellow", "orange",
        "purple", "pink", "gray", "brown", "lime", "cyan", "magenta",
        "maroon", "navy", "olive", "teal", "gold", "indigo", "violet",
        "khaki", "lavender", "salmon", "lightseagreen", "skyblue",
        "plum", "wheat", "lightgray", "darkgray", "dimgray", "silver"
      ]
    },
    "color_name": {
      "type": "string",
      "description": "Human-readable foreground/text color name. Combined with background helps identify high-contrast CTAs (e.g., 'white on blue' = primary button)",
      "example": "white",
      "enum": [
        "black", "white", "red", "blue", "green", "yellow", "orange",
        "purple", "pink", "gray", "brown", "lime", "cyan", "magenta",
        "maroon", "navy", "olive", "teal", "gold", "indigo", "violet",
        "khaki", "lavender", "salmon", "lightseagreen", "skyblue",
        "plum", "wheat", "lightgray", "darkgray", "dimgray", "silver"
      ]
    },
    "cursor": {
      "type": "string",
      "description": "CSS cursor value. 'pointer' is the strongest clickability signal.",
      "example": "pointer",
      "enum": ["pointer", "default", "text", "move", "not-allowed", "grab", "crosshair", "wait", "help"]
    },
    "is_primary": {
      "type": "boolean",
      "description": "Whether element appears visually prominent based on viewport-relative size (>1% of viewport), color (primary colors with bold/large text), and typography. Helps AI prioritize primary actions over secondary/tertiary.",
      "example": true
    }
  }
}
```

---

## Exact Line Locations for Changes

### Change 1: ObserveOptions
- **File:** `SDK-SentienceAPI/openapi.json`
- **Line:** 256 (after `viewport` property, before closing brace at line 257)
- **Action:** Add comma after line 256, insert `include_visual_cues` property

### Change 2: InteractableElement
- **File:** `SDK-SentienceAPI/openapi.json`
- **Lines:** 545-558
- **Actions:**
  1. After line 547 (`"z_index": { "type": "integer" }`), add `in_viewport` and `is_occluded`
  2. Line 548-554: Update `attributes` to add description
  3. After line 558 (`states` property), add `visual_cues` property

### Change 3: VisualCues Schema
- **File:** `SDK-SentienceAPI/openapi.json`
- **Line:** 574 (after `ElementStates` closing brace)
- **Action:** Insert entire `VisualCues` schema before `BoundingBox` schema

---

## Differences Between Gateway and SDK OpenAPI

### Similarities (Same Changes Needed)
✅ `ObserveOptions` schema update
✅ `InteractableElement` schema update
✅ `VisualCues` schema creation
✅ Field descriptions and types

### Differences (SDK-Specific)
🔑 **Authentication:**
- Gateway: `bearerAuth` (JWT)
- SDK: `apiKeyAuth` (API Key in header)

📍 **Server URL:**
- Gateway: Internal service URL
- SDK: Public API URL (`https://api.sentienceapi.com`)

📝 **Documentation Audience:**
- Gateway: Backend developers
- SDK: External API consumers (developers using the SDK)

---

## Implementation Checklist

### Phase 1: Schema Updates
- [ ] **Update `ObserveOptions`** (line 238-258)
  - [ ] Add `include_visual_cues` property after `viewport`
  - [ ] Verify JSON syntax (comma after `viewport`)

- [ ] **Update `InteractableElement`** (line 510-560)
  - [ ] Add `in_viewport` property after `z_index`
  - [ ] Add `is_occluded` property
  - [ ] Update `attributes` description
  - [ ] Add `visual_cues` property reference

- [ ] **Create `VisualCues` schema** (after line 574)
  - [ ] Add complete schema with all 4 properties
  - [ ] Include descriptions and enum values
  - [ ] Verify placement before `BoundingBox`

### Phase 2: Validation
- [ ] **Validate JSON syntax**
  ```bash
  npx @apidevtools/swagger-cli validate SDK-SentienceAPI/openapi.json
  ```

- [ ] **Check for duplicate schemas**
  - Ensure no duplicate `VisualCues` definition
  - Verify all `$ref` references are valid

- [ ] **Verify backward compatibility**
  - New fields are optional (not in `required` array)
  - `include_visual_cues` has default value

### Phase 3: Documentation
- [ ] **Update SDK documentation**
  - Add usage examples for `include_visual_cues` parameter
  - Document token cost implications
  - Add code samples for different languages

- [ ] **Update changelog**
  - Document new fields in SDK release notes
  - Highlight opt-in nature of `visual_cues`

---

## Request/Response Examples for SDK Docs

### Example 1: Default Request (Node.js SDK)

```javascript
import { SentienceClient } from '@sentience/sdk';

const client = new SentienceClient('sk_live_51Mz...');

const response = await client.observe({
  url: 'https://amazon.com/product/12345',
  mode: 'map'
});

// Response includes in_viewport and is_occluded
console.log(response.interactable_elements[0]);
// {
//   id: 1,
//   tag: 'button',
//   text: 'Add to Cart',
//   in_viewport: true,
//   is_occluded: false
// }
```

### Example 2: Request with visual_cues (Python SDK)

```python
from sentience import SentienceClient

client = SentienceClient(api_key='sk_live_51Mz...')

response = client.observe(
    url='https://figma.com/design/123',
    mode='map',
    options={
        'include_visual_cues': True  # Opt-in for visual cues
    }
)

# Response includes visual_cues for icon-only buttons
for element in response.interactable_elements:
    if element.visual_cues:
        print(f"Element {element.id}: {element.visual_cues.background_color_name} button")
        # "Element 42: red button" (delete action inferred)
```

### Example 3: cURL Request

```bash
curl -X POST https://api.sentienceapi.com/v1/observe \
  -H "Authorization: Bearer sk_live_51Mz..." \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://figma.com/design/123",
    "mode": "map",
    "options": {
      "include_visual_cues": true
    }
  }'
```

**Response:**
```json
{
  "mode": "map",
  "status": "success",
  "interactable_elements": [
    {
      "id": 42,
      "tag": "button",
      "text": "",
      "bbox": { "x": 450, "y": 200, "w": 24, "h": 24 },
      "is_visible": true,
      "in_viewport": true,
      "is_occluded": false,
      "visual_cues": {
        "background_color_name": "red",
        "color_name": "white",
        "cursor": "pointer",
        "is_primary": false
      }
    }
  ]
}
```

---

## Testing Strategy

### Unit Tests (SDK-Level)
```javascript
// Test deserialization of new fields
describe('ObserveResponse', () => {
  it('should parse in_viewport and is_occluded', () => {
    const json = {
      id: 1,
      tag: 'button',
      in_viewport: true,
      is_occluded: false
    };
    const element = InteractableElement.fromJSON(json);
    expect(element.in_viewport).toBe(true);
    expect(element.is_occluded).toBe(false);
  });

  it('should parse visual_cues when present', () => {
    const json = {
      id: 1,
      tag: 'button',
      visual_cues: {
        background_color_name: 'blue',
        cursor: 'pointer',
        is_primary: true
      }
    };
    const element = InteractableElement.fromJSON(json);
    expect(element.visual_cues?.background_color_name).toBe('blue');
  });
});
```

### Integration Tests
- [ ] Test request with `include_visual_cues: false` (default)
- [ ] Test request with `include_visual_cues: true`
- [ ] Verify backward compatibility (omitting the field)
- [ ] Test with icon-heavy sites (Figma, Canva)

---

## Migration Guide for SDK Users

### Non-Breaking Changes ✅

All changes are backward compatible:

1. **New request parameter** (`include_visual_cues`)
   - Optional field with default value `false`
   - Existing code continues to work without changes

2. **New response fields** (`in_viewport`, `is_occluded`, `visual_cues`)
   - Optional fields in response
   - Existing parsers will ignore unknown fields
   - No changes required to existing client code

### Upgrade Path

**Version 1.x (Current):**
```javascript
// Works as before
const response = await client.observe({
  url: 'https://example.com',
  mode: 'map'
});
```

**Version 2.0 (With visual_cues):**
```javascript
// Still works (backward compatible)
const response = await client.observe({
  url: 'https://example.com',
  mode: 'map'
});

// New feature (opt-in)
const enhancedResponse = await client.observe({
  url: 'https://figma.com',
  mode: 'map',
  options: {
    include_visual_cues: true  // NEW: Get color and prominence hints
  }
});
```

---

## Validation Commands

### Validate OpenAPI Schema
```bash
cd /Volumes/ExtremeSSD/Code/SentienceAPI/SDK-SentienceAPI
npx @apidevtools/swagger-cli validate openapi.json
```

### Compare with Gateway OpenAPI
```bash
# Show differences in schema definitions
diff -u \
  <(jq '.components.schemas.ObserveOptions' SDK-SentienceAPI/openapi.json) \
  <(jq '.components.schemas.ObserveOptions' gateway/openapi.json)
```

### Generate SDK from OpenAPI
```bash
# Example: Generate TypeScript SDK
npx @openapitools/openapi-generator-cli generate \
  -i SDK-SentienceAPI/openapi.json \
  -g typescript-fetch \
  -o sdk-typescript
```

---

## Summary

### Changes Required: **3 Schema Updates**

1. ✅ **ObserveOptions:** Add `include_visual_cues` field
2. ✅ **InteractableElement:** Add 3 new fields (`in_viewport`, `is_occluded`, `visual_cues`)
3. ✅ **VisualCues (NEW):** Create schema with 4 fields

### Impact: **Non-Breaking**
- All new fields are optional
- Default behavior unchanged
- Backward compatible with existing SDK users

### Estimated Time: **30-45 minutes**
- Schema updates: 15 minutes
- Validation: 10 minutes
- Documentation: 15 minutes
- Testing: Additional time as needed

---

## References

- **Gateway OpenAPI Changes:** See `gateway/GATEWAY_INTEGRATION_PLAN.md`
- **Technical Spec:** See `servo-sentience/documentation/challenges/challenge2.md`
- **Final Changes:** See `servo-sentience/documentation/challenges/FINAL_CHANGES.md`

---

**Status:** 📋 **ASSESSMENT COMPLETE - READY FOR IMPLEMENTATION**
**File:** `/Volumes/ExtremeSSD/Code/SentienceAPI/SDK-SentienceAPI/openapi.json`
**Last Updated:** 2025-12-17
