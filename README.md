# simplewebconsolescraper

## Overview

This script runs entirely in your browser's Console, fetching paginated bookmark listings, parsing each page's HTML into a DOM, extracting bookmark titles, links, and "Current Chapter" data, then packaging everything into downloadable **CSV** and **XML** files — all without external libraries.

## How It Works

1. **Pagination & Fetching**
   - Uses the **Fetch API** to request each page's HTML as text (`fetch(url)`).
   - Respects same-origin credentials by default.
   - Loops by checking for `.pagination .next:not(.disabled)` in the parsed DOM.

2. **Parsing HTML**
   - Uses `DOMParser().parseFromString(html, 'text/html')` to convert raw HTML into a live Document.

3. **Extracting Data**
   - `doc.querySelectorAll('.user-bookmark-item')` grabs bookmark items.
   - Iterates over the resulting **NodeList** with `forEach()`.
   - Uses optional chaining (`?.`) to safely access potentially missing elements.
   - Separately targets the `<span>...<a rel="nofollow">Current : Chapter</a></span>` link for “Current Chapter” info.

4. **Serializing to CSV & XML**
   - Builds a header row dynamically based on object keys.
   - Constructs a CSV string, escaping quotes as needed.
   - Creates an XML string with simple `<record>` elements.

5. **Creating Download Links**
   - Uses `URL.createObjectURL(blob)` to generate a temporary URL.
   - Appends `<a download="...">` elements to the DOM for easy download.

## Quick Start

1. Open your bookmarks page at `https://your.site/bookmarks?page=1`.
2. Press F12 (or Cmd+Option+I) to open DevTools → Console.
3. Paste the entire scraping snippet and hit Enter.
4. Click the newly appended **Download bookmarks.csv** and **Download bookmarks.xml** links.

## Customization

- **Selectors**
  - Change `'.user-bookmark-item'` to match your site's container for items.
  - Adjust `'.bm-title'` or the `querySelectorAll('a')[4]` if needed.

- **Pagination Logic**
  - Modify the selector:
    ```javascript
    doc.querySelector('.pagination .next:not(.disabled)')
    ```
    if your site uses a different structure.

- **Output Fields**
  - You can add new properties to the `results.push({ ... })` object.
  - Headers for CSV are generated automatically from object keys.

- **Filename & MIME Type**
  - Modify `'bookmarks.csv'` or `'bookmarks.xml'` if you want a different file name.
  - You can also adjust the MIME type if necessary.

## Making It General-Purpose

If you want to adapt this script for **any** website:

1. **Identify the List Container**  
   - Inspect the page and find the container that holds the repeating elements you want (e.g., articles, products, comments).
   - Update the selector:
     ```javascript
     doc.querySelectorAll('YOUR_ITEM_SELECTOR')
     ```
     Example: `.post`, `.product-item`, `.comment-row`.

2. **Identify Fields to Extract**  
   - For each field (title, link, date, etc.), update the inner query selectors.
   - Example:
     ```javascript
     item.querySelector('.title')?.textContent
     item.querySelector('a')?.href
     item.querySelector('.meta .date')?.textContent
     ```

3. **Pagination Navigation**  
   - Find how the site loads the next page.
   - Update how `nextLink` is found. It could be:
     - A `<a class="next">` link.
     - A button that fetches via JavaScript (this needs a more advanced method).

4. **Field Mapping**  
   - You can define a map object for fields to make it very flexible:
     ```javascript
     const fieldMap = {
       title: '.title',
       link: 'a',
       date: '.date'
     };
     ```
   - Then extract dynamically:
     ```javascript
     const record = {};
     for (const [key, selector] of Object.entries(fieldMap)) {
       record[key] = item.querySelector(selector)?.textContent?.trim() || '';
     }
     ```

5. **Testing**
   - Always run on one page first before letting it auto-scrape all pages.
   - Log each record to console with `console.table(results)` to verify.

6. **Performance**
   - Add small delays between page loads if you scrape hundreds of pages.
   - Example:
     ```javascript
     await new Promise(r => setTimeout(r, 500)); // 500ms delay
     ```

## Further Reading

- [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [DOMParser](https://developer.mozilla.org/en-US/docs/Web/API/DOMParser)
- [CSS Selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors)
- [NodeList.forEach](https://developer.mozilla.org/en-US/docs/Web/API/NodeList/forEach)
- [Optional Chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)
- [Blob API](https://developer.mozilla.org/en-US/docs/Web/API/Blob)
- [URL.createObjectURL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL)
- [Array.from](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from)
- [Location.origin](https://developer.mozilla.org/en-US/docs/Web/API/Location/origin)
- [Node.appendChild](https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild)

---

Use this README as your guide to quickly understand, extend, and adapt the scraper for **any site**.  
Pure vanilla browser APIs, no dependencies required!
