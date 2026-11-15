# Understanding Semantic Versioning (SemVer) & The Tilde (`~`) Operator in npm

Semantic Versioning (SemVer) is a standard three-part numbering system (`MAJOR.MINOR.PATCH`) used in npm packages to communicate the impact of code changes in each release.

---

## 1. Semantic Versioning Explained (`MAJOR.MINOR.PATCH`)

Every standard npm package version follows the **`1.0.0`** format:

- **MAJOR (`1`.x.x):** Incompatible API changes (breaking changes).
- **MINOR (x.`0`.x):** New functionality added in a backward-compatible manner.
- **PATCH (x.x.`0`):** Backward-compatible bug fixes.

### Practical Example: `store-product-api`

Imagine maintaining an e-commerce npm package called `store-product-api`, initially launched at version **`1.0.0`**:

```javascript
// Version 1.0.0 usage
const product = await api.getProduct(101);
// Returns: { id: 101, name: "Wireless Mouse", price: 25 }
```

#### A. PATCH Update (`1.0.0` → `1.0.1`) — Bug Fix
- **Scenario:** `api.getProduct()` crashes when a product is out of stock instead of returning `null`. You fix this bug internally.
- **Impact:** Existing code works identically without any changes.
- **Result:** Increment the **PATCH** number (`1.0.1`).

#### B. MINOR Update (`1.0.1` → `1.1.0`) — New Feature
- **Scenario:** You add a new `searchProducts()` method and include an optional `stock` field in the product response.
- **Impact:** Backward-compatible. Existing `getProduct()` calls continue working normally.
- **Result:** Increment the **MINOR** number and reset Patch to `0` (`1.1.0`).

```javascript
// Version 1.1.0 usage
const product = await api.getProduct(101);
// Returns: { id: 101, name: "Wireless Mouse", price: 25, stock: 12 }

// New feature available
const laptops = await api.searchProducts({ category: "laptops" });
```

#### C. MAJOR Update (`1.1.0` → `2.0.0`) — Breaking Change
- **Scenario:** You rename `getProduct()` to `fetchProductById()` and change `price` from a plain number (`25`) to an object (`{ amount: 25, currency: "USD" }`).
- **Impact:** **Breaking change.** Apps updating to `2.0.0` without code refactoring will break.
- **Result:** Increment the **MAJOR** number and reset Minor and Patch to `0` (`2.0.0`).

```javascript
// Version 2.0.0 usage
const product = await api.fetchProductById(101);
// Returns: { id: 101, name: "Wireless Mouse", price: { amount: 25, currency: "USD" } }
```

### SemVer Summary Table

| Version | Change in `store-product-api` | Safe to Auto-Update? |
| :--- | :--- | :--- |
| **`1.0.1`** | Fixed crash on missing product ID | **Yes** (`^1.0.0` or `~1.0.0`) |
| **`1.1.0`** | Added `searchProducts()` & `stock` field | **Yes** (`^1.0.0`) |
| **`2.0.0`** | Renamed `getProduct()` & changed `price` format | **No** (Requires code changes) |

---

## 2. The Tilde (`~`) Operator in `package.json`

The **tilde (`~`)** character restricts npm to installing **PATCH updates only**, locking both the **MAJOR** and **MINOR** version numbers.

```json
{
  "dependencies": {
    "store-product-api": "~1.1.0"
  }
}
```

### How `~1.1.0` Behaves During `npm update`

- **`1.1.1` (Bug Fix) → Installed:** Only the PATCH number changed (`1.1.0` → `1.1.1`).
- **`1.1.9` (Bug Fix) → Installed:** Remains within the `1.1.x` series.
- **`1.2.0` (New Feature) → Blocked:** The MINOR number changed (`1.1` → `1.2`).
- **`2.0.0` (Breaking Change) → Blocked:** The MAJOR number changed (`1` → `2`).

### Tilde (`~`) vs. Caret (`^`) Comparison

| Feature | Tilde (`~1.1.0`) | Caret (`^1.1.0`) |
| :--- | :--- | :--- |
| **Allowed Version Range** | `>=1.1.0` and `<1.2.0` | `>=1.1.0` and `<2.0.0` |
| **Permitted Updates** | **PATCH** only (bug fixes) | **MINOR** & **PATCH** (features + bug fixes) |
| **Best Use Case** | Maximum stability; prevents unexpected feature behavior. | Receiving non-breaking features and fixes automatically. |

### Shorthand Edge Cases
- **`~1.1`:** Equivalent to `~1.1.0` (allows `1.1.x` patch releases only).
- **`~1`:** Equivalent to `^1.0.0` (allows all `1.x.x` minor and patch releases).
