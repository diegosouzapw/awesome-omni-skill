---
name: react-to-wx-miniprogram-migrator
description: Migrates a React + TailwindCSS H5 web application to a native WeChat Mini Program. Use when the user wants to convert their existing web project into a mini program, preserving structure, styling, and functionality.
license: MIT
metadata:
  author: Manus AI
  version: "1.0"
---

# React + TailwindCSS to WeChat Mini Program Migration Skill

This skill provides a systematic process for an AI agent to migrate a web application built with React and TailwindCSS into a native WeChat Mini Program. The goal is to achieve a functional and stylistically faithful conversion by mapping modern web development patterns to the specific architecture of WeChat Mini Programs.

## When to Use This Skill

Use this skill when the user explicitly requests to convert a React-based H5 website, especially one using TailwindCSS for styling, into a native WeChat Mini Program. The source code of the React project should be available for analysis.

## Migration Process Overview

The migration is a multi-step process that involves code transformation, style conversion, and API replacement. Follow these steps sequentially for a successful migration.

1.  **Analyze Source & Setup Project**: Analyze the React project structure and set up a corresponding WeChat Mini Program project.
2.  **Component & JSX to WXML Conversion**: Convert React components and JSX syntax into Mini Program components and WXML structure.
3.  **TailwindCSS to WXSS Conversion**: Transform TailwindCSS utility classes into equivalent WXSS styles.
4.  **Logic & State Management Migration**: Adapt React's state management (Hooks, Redux) and lifecycle to the Mini Program's `data` and `setData` system.
5.  **Routing & Navigation Replacement**: Replace web-based routing (e.g., React Router) with the Mini Program's page navigation API.
6.  **API & SDK Replacement**: Substitute browser/web APIs (e.g., `fetch`, `localStorage`) with their WeChat Mini Program counterparts (`wx.request`, `wx.setStorageSync`).
7.  **Asset & Resource Handling**: Migrate and correctly reference static assets like images and fonts.
8.  **Final Review & Debugging**: Thoroughly test the migrated application in the WeChat DevTools and on a physical device.

---

## Step-by-Step Migration Instructions

### Step 1: Analyze Source & Setup Project

1.  **Analyze React Project**: Use the `ls -R` or `tree` command to inspect the source React project's directory structure. Identify the main components, pages, utility functions, and static assets.
2.  **Create Mini Program Structure**: Based on the analysis, create a standard WeChat Mini Program directory structure. Refer to `references/MIGRATION_MAP.md` for the standard structure.
    -   Create the `app.js`, `app.json`, and `app.wxss` root files.
    -   Create a `pages` directory and subdirectories for each page identified in the React app.
    -   Create a `components` directory for reusable components.
    -   Create a `utils` directory for helper functions.
3.  **Configure `app.json`**: Populate the `app.json` file. List all the pages in the `pages` field. If the H5 app has a tab bar, configure the `tabBar` field accordingly.

### Step 2: Component & JSX to WXML Conversion

This is the core of the UI migration. Convert each React component into a Mini Program component folder containing `.js`, `.wxml`, `.wxss`, and `.json` files.

1.  **Map HTML Tags to WXML Components**: For each JSX file, systematically replace HTML tags with their corresponding WXML component equivalents. A detailed mapping can be found in `references/MIGRATION_MAP.md`.
    -   `div` -> `view`
    -   `span`, `p`, `h1`-`h6` -> `text`
    -   `img` -> `image`
    -   `a` -> `navigator`
    -   `button` -> `button`
2.  **Convert JSX Syntax**: Transform JSX control flow and expressions into WXML directives.
    -   **List Rendering**: Convert `.map()` calls to `wx:for` loops.
    -   **Conditional Rendering**: Convert `&&`, `? :` operators to `wx:if`, `wx:elif`, `wx:else` directives.
    -   **Data Binding**: Convert `{variable}` to `{{variable}}`.
    -   **Event Handling**: Convert `onClick={handler}` to `bindtap="handler"`.

### Step 3: TailwindCSS to WXSS Conversion

This is a critical and potentially complex step. The primary challenge is converting utility-first CSS into static WXSS stylesheets, as Mini Programs do not have a JIT compiler for styles.

1.  **Identify All Tailwind Classes**: Scan all JSX files and extract every TailwindCSS class name used.
2.  **Convert Classes to CSS**: Use an online tool or a script to convert the list of Tailwind classes into standard CSS. You can use this [Tailwind to CSS Converter](https://tailwind-to-css.vercel.app/) for this purpose.
3.  **Adapt CSS for WXSS**: Manually adjust the generated CSS to be compatible with WXSS.
    -   **Convert Units**: Change `px` or `rem` units to `rpx`. A common baseline is `1rem` (16px) = `32rpx`. Use this as a general rule but adjust for precision.
    -   **Handle Unsupported Selectors**: Replace unsupported selectors like the universal selector (`*`) or complex descendant selectors.
    -   **Address `gap` Property**: Since `gap` is not supported, implement spacing between flex/grid items using `margin` on the child elements.
4.  **Organize Styles**: Paste the converted styles into the appropriate `.wxss` files. Global styles (from `index.css` or `App.css`) go into `app.wxss`, and component-specific styles go into the component's `.wxss` file.

### Step 4: Logic & State Management Migration

1.  **Component Logic**: Move the logic from the React component function body into the `Page({...})` or `Component({...})` object in the `.js` file.
2.  **State Management**: Refactor React state to the Mini Program's `data` object.
    -   Convert `useState` declarations to initial values in the `data` object.
    -   Replace state update calls (e.g., `setCount(1)`) with `this.setData({ count: 1 })`.
3.  **Lifecycle Methods**: Map React's lifecycle hooks to Mini Program lifecycle methods. See `references/MIGRATION_MAP.md` for a detailed mapping.
    -   `useEffect` with an empty dependency array `[]` maps to `onLoad`.
    -   `useEffect` with no dependency array maps roughly to `onShow`.
    -   The return function from `useEffect` maps to `onUnload`.

### Step 5: Routing & Navigation Replacement

Replace all instances of React Router or other routing libraries with the Mini Program's built-in navigation API.

-   `<Link to="/path">` or `navigate('/path')` becomes `wx.navigateTo({ url: '/pages/path/index' })`.
-   Use `wx.switchTab` for navigating to tab bar pages.
-   Use `wx.redirectTo` for redirects.
-   Use `wx.navigateBack` for going back.

### Step 6: API & SDK Replacement

Replace all browser-specific APIs with their `wx.` counterparts.

-   **Network Requests**: Replace `fetch` or `axios` with `wx.request`.
-   **Local Storage**: Replace `localStorage` with `wx.setStorageSync` and `wx.getStorageSync`.
-   **DOM Manipulation**: Remove all direct DOM manipulation code (e.g., `document.getElementById`). Instead, use data binding to control the view.

### Step 7: Asset & Resource Handling

1.  **Images**: Move all image assets into a dedicated directory (e.g., `/images`). Update all `<image>` `src` paths to be absolute paths from the project root (e.g., `/images/logo.png`).
2.  **Fonts**: If custom fonts are used, they need to be loaded using `wx.loadFontFace` in `app.js`.

### Step 8: Final Review & Debugging

1.  **Use WeChat DevTools**: This is your primary environment for testing and debugging. Check the console for errors and use the WXML panel to inspect the UI structure.
2.  **Test on Device**: Preview the Mini Program on a physical device to check for performance issues and styling inconsistencies that may not appear in the simulator.
3.  **Iterate and Refine**: The migration process is iterative. Expect to go back and forth between steps to fix bugs and refine the user experience.

---

## Bundled Resources

-   **`references/MIGRATION_MAP.md`**: Contains detailed mapping tables for components, events, lifecycles, and APIs to assist in the conversion process.
