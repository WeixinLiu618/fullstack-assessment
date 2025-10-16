# Stackline Full Stack Assignment - Bug Fixes Documentation

<br>

**Author**: Weixin Liu<br>
**Date**: Oct 16, 2025

## Overview

This document summarizes the bugs identified and fixed in the Stackline eCommerce Full Stack assessment. The fixes address security vulnerabilities, UX inconsistencies, API error handling, and performance issues. Each fix is explained with location, cause, and resolution details.

## The files that were modified are:

1. `app/page.tsx` - Main product list page (bug fixes)
2. `app/product/page.tsx` - Product detail page (bug fixes)
3. `next.config.ts` - Image configuration (bug fix for Amazon image domains)
4. `README.md` - Documentation of all changes


## Bugs Identified and Fixed

### Bug #1 Security Issue: Product Data in URL

**Location**: `app/page.tsx` line 167-170

**Issue**: The entire product object was being passed through the URL as a JSON-stringified query parameter when navigating to the product detail page.

**Problems**:
- **Security Risk**: All product data exposed in the URL and browser history
- **Data Manipulation**: Users could modify the URL to inject arbitrary product data
- **URL Length Limits**: Could hit browser URL length limitations (typically 2000 characters)
- **Poor UX**: Extremely long, ugly URLs that are not shareable
- **SEO Impact**: Non-semantic URLs hurt search engine optimization


**Fix**: 
1. `app/page.tsx` (Line 167)
* Changed from passing entire product object to passing only the SKU:

```typescript
// Before:
href={{
  pathname: "/product",
  query: { product: JSON.stringify(product) },
}}

// After:
href={`/product?sku=${product.stacklineSku}`}
```

2. `app/product/page.tsx` (Lines 23-90)
* Updated to fetch product data from the API instead of parsing from URL
* Changed from productParam to sku parameter
* Added loading and error states
* Implemented API fetch to /api/products/${sku}
* Added proper error handling
* Added loading state UI
* Added error state UI with user-friendly message


---

### Bug #2 UX and Functionality Issue: Category & Subcategory Filtering

**Location**: `app/page.tsx` lines 55, 59, 98-132

**Issue**: Multiple related problems with category and subcategory filtering functionality and UX.

**Problems**:
- **Incorrect Filtering**: Subcategories dropdown showed ALL subcategories from ALL categories instead of filtering by selected category
- **No Individual Filter Clearing**: Users could only clear all filters at once, no way to select "All Categories" or "All Subcategories" individually
- **Invalid State Possible**: Users could select subcategories that didn't belong to the selected category
- **State Management**: Subcategory didn't auto-clear when switching categories, causing confusion

**Fix**:
1. **`app/page.tsx`** lines 55, 59, 98-132
* Added category parameter to subcategories API call for proper filtering:

    ```typescript
    // Before:
    fetch(`/api/subcategories`)
    
    // After:
    fetch(`/api/subcategories?category=${encodeURIComponent(selectedCategory)}`)
    ```

* Added auto-clear logic when category changes.
* Added "All Categories" option to category dropdown.
* Added "All Subcategories" option to subcategory dropdown.


---


### Bug #3 Functionality Issue: No Error Handling on API Calls

**Location**: `app/page.tsx` lines 46, 48-59, 61-79, 81-104, 178-187

**Issue**: None of the fetch calls had any error handling. If any API endpoint failed, the application would break silently.

**Problems**:
- **Silent Failures**: No error messages shown to users when API calls fail
- **Poor UX**: Application appears broken with no feedback or explanation
- **Stuck Loading States**: Loading indicators could remain visible indefinitely
- **No Recovery Option**: Users have no way to retry failed requests
- **Debugging Difficulty**: Only console errors available, not user-friendly

**Fix**:

1. **`app/page.tsx` ** (Line 46, 48-59, 61-79, 81-104, 178-187)
* Added error state management:
   - Added `error` state variable for tracking API failures.
* Added error handling to categories API call with user-friendly error message.
* Added error handling to subcategories API call with graceful degradation.
* Added error handling to products API call with proper state management.
* Added error UI with retry functionality.


---

### Bug #4 Performance Issue: Search Input Triggers on Every Keystroke

**Location**: `app/page.tsx` lines 39, 49-56, 95, 114

**Issue**: Search functionality triggered an API call on every single keystroke, causing excessive API requests.

**Problems**:
- **Excessive API Calls**: Typing "laptop" creates 6 API calls (one for each character: l, la, lap, lapt, lapto, laptop)
- **Server Overhead**: Wastes server resources processing intermediate/incomplete search queries
- **Poor Performance**: Rapid loading state changes create flickering UI
- **Potential Rate Limiting**: Could hit API rate limits with fast typing
- **Network Waste**: Unnecessary bandwidth usage for queries user didn't intend to search

**Fix**:

1. **`app/page.tsx`** lines 39, 49-56, 95, 114
* Added debounced search state:
   ```typescript
   const [debouncedSearch, setDebouncedSearch] = useState("");
   ```
* Implemented debounce logic with 300ms delay.
* Changed products API call to use debounced value.

---
### Additional Fix: Next.js Image Configuration

**Location**: `next.config.ts` lines 10-13

**Issue**: Product images from Amazon CDN were failing to load with error: "hostname is not configured under images in your next.config.js"


**Fix**:

**`next.config.ts` (Lines 10-13)**
Added Amazon image domain to Next.js remote patterns：
```
const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'm.media-amazon.com',
      },
      {
        protocol: 'https',
        hostname: 'images-na.ssl-images-amazon.com',
      },
    ],
  },
};
```

---

## Summary

This eCommerce application had several critical and non-critical bugs. Four major bugs have been identified and fixed:

1. **Security Vulnerability** - Product data exposed in URL query parameters
2. **Category & Subcategory Filtering** - Multiple UX and functionality issues with filters
3. **Error Handling** - Missing error handling on all API calls
4. **Search Performance** - Excessive API calls on every keystroke

**Additional Fix:**
- **Next.js Image Configuration** - Added Amazon CDN domain to allow product images to load properly

---

