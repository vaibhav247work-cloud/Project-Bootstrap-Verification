# Data Table and Enterprise Grid Standard

## Purpose

Data tables and list grids are the primary interaction surface for ERP, SaaS, booking, and administrative applications. This standard establishes requirements for virtualization, URL synchronization, mobile responsiveness, bulk actions, and accessibility.

---

## 1. URL State Synchronization (Shareable Views)

All table state (search query, filter parameters, current page, page size, sort column, sort direction) **MUST** be synchronized with URL query parameters.

```text
/admin/bookings?page=2&size=50&status=PENDING&sort=-created_at&search=acme
```

### Rules:
1. **Bookmarkability:** A user must be able to copy the URL, send it to a colleague, and the colleague must see the exact same filtered and sorted view.
2. **Browser Navigation:** Pressing the browser's Back button must step backward through previous search/filter states predictably.
3. **Reset to Page 1 on Filter Change:** Modifying any filter or search keyword must automatically reset the page parameter to `page=1`.

---

## 2. Virtualization for Large Datasets

When rendering lists exceeding 100 rows in DOM simultaneously, client performance degrades rapidly.

1. **Client-Side Virtualization:**
   - Use windowing/virtualization (e.g., `@tanstack/react-virtual`) when rendering large lists in the browser.
   - Render only the DOM nodes visible in the current viewport plus a small overscan buffer (e.g., 5 rows above/below).
2. **Bounded Server Pagination:**
   - Combine virtualization with bounded server-side cursor or offset pagination (as mandated in `04-backend/spring-boot/api-standard.md`).
   - Default page size: 25 or 50 items. Maximum page size: 100 items. Never request unbounded collections (`size=10000`).

---

## 3. Responsive Table-to-Card Fallback

Traditional wide multi-column tables are unusable on mobile screens (< 768px).

1. **Viewport Adaptation (< 768px):**
   - Transform table rows into stacked, readable **Data Cards** or provide an explicit horizontal scroll indicator with pinned key columns.
   - Key attributes (Status badge, Title, Primary action) are elevated to the card header.
   - Secondary attributes are formatted into a clean label/value list.
2. **Horizontal Scroll Table Rule:**
   - If a dense table must remain a grid (e.g., financial ledger), pin the identifier column on the left and display an explicit shadow or indicator showing more columns exist off-screen.

---

## 4. Bulk Selection and Batch Actions

1. **Selection Model:**
   - Checkbox header supports three states: **Unchecked**, **Indeterminate** (some rows selected), and **Checked** (all rows on current page selected).
   - "Select all across all pages" requires an explicit banner confirmation showing total count (e.g., *"Select all 1,420 bookings across all pages"*).
2. **Batch Actions Bar:**
   - Floating action toolbar appears above the table when 1 or more rows are selected (e.g., *Export Selected*, *Bulk Delete*, *Update Status*).
   - Any destructive bulk action (e.g., Delete, Cancel) **MUST** require a confirmation modal displaying the count and scope before execution.

---

## 5. Accessibility and UX States

1. **Loading State:**
   - Render table skeleton rows matching the expected column widths while fetching data. Do not show a blank white box.
2. **Empty & Filtered Zero-State:**
   - Separate "No records exist yet" (prompting creation) from "No records match your active filters" (providing a 1-click "Clear All Filters" button).
3. **Keyboard Accessibility:**
   - Table headers used for sorting must be interactive `<button>` elements with `aria-sort="ascending"` or `aria-sort="descending"`.
   - Grid cells containing interactive controls must be navigable via keyboard Tab/Arrow keys without focus traps.
