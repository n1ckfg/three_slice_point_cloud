# Architecture Overview: Three Slice Point Cloud

This document outlines the architecture and technical design of the `three_slice_point_cloud` project.

## 1. System Overview

The project is a lightweight, frontend-only web application that visualizes a 3D point cloud and projects it onto a dynamically moving 2D plane (contour slice). It relies on Vanilla JavaScript and Three.js for 3D rendering, without the need for a complex build step or modern frontend framework (like React or Vue). 

## 2. Technology Stack

*   **Frontend Environment:** HTML5, CSS3, Vanilla JavaScript (ES6+).
*   **3D Rendering Engine:** [Three.js](https://threejs.org/) (included as a static asset).
*   **Controls:** `OrbitControls` (for Three.js) allowing the user to rotate, pan, and zoom the camera.
*   **Local Server:** `http-server` (Node.js package) used for serving the static files locally to bypass CORS restrictions when loading data files.

## 3. Directory Structure

*   **`/` (Root):** Contains the main entry point (`index.html`) and execution scripts.
*   **`/css/`:** Contains stylesheets (`main.css`).
*   **`/files/`:** Contains the raw point cloud data files (e.g., `20180527_111027.xyz`).
*   **`/js/libraries/threejs/`:** Contains the statically vendored dependencies (`three.min.js`, `OrbitControls.js`).

## 4. Core Components & Logic

All of the core application logic is embedded directly within `<script>` tags in `index.html`.

### 4.1. Scene Initialization
The app initializes a standard Three.js pipeline:
*   `THREE.WebGLRenderer` attached to the document body.
*   `THREE.Scene` and `THREE.PerspectiveCamera`.
*   `THREE.OrbitControls` to handle mouse/touch interactions.

### 4.2. Data Loading (`loadPointCloud`)
*   Data is loaded using `THREE.FileLoader` which fetches an `.xyz` text file containing 3D coordinates.
*   The raw text is parsed by splitting lines and spaces, converting the string values into a flat `Float32Array`.
*   This array is used to construct a `THREE.BufferGeometry`, which is then rendered as a `THREE.Points` object with a simple white material.

### 4.3. Dynamic Contour Slicing (`projectPointCloudOntoSlice`)
The most significant logic in the application involves the dynamic "contour slice".
*   A `middlePlanePosition` variable is continuously updated in the `animate` loop using a sine wave based on the current time (`Math.sin(Date.now() * 0.005) * 2.0`). This creates a smooth oscillating vertical movement.
*   During each frame, the `projectPointCloudOntoSlice` function iterates through all vertices of the original point cloud and keeps only those whose `y` falls within `sliceThickness` of `middlePlanePosition` — producing an actual cross-section of the cloud at that height.
*   The surviving points retain their original `(x, y, z)` coordinates, so the slice preserves the true geometry of the cloud at that height rather than flattening it.
*   `contourSlice` is a red `THREE.Points` object whose geometry is rebuilt each frame from the filtered vertices.

## 5. Execution Scripts

To run the application, the project provides OS-specific scripts that handle the setup of a local web server:
*   **`run.bat` (Windows):** A batch script that uses `netstat` to find an open port starting at `8080`, launches the default browser, and starts `http-server`.
*   **`run.command` (macOS / Linux):** A bash script that uses `lsof` to find an open port starting at `8080`, opens the browser, and starts `http-server`.

These scripts ensure a frictionless "click-to-run" developer/user experience without requiring manual port configuration.
