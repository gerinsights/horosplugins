# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is `gerinsights/horosplugins` — a collection of 84 plugins for the Horos DICOM medical imaging workstation. All plugins require modernization for Apple Silicon, Metal rendering, and current macOS APIs. Sister repo: `gerinsights/horos`.

## Structure

Each plugin is a standalone directory with its own `.xcodeproj`. Plugins build as NSBundle (.horosplugin) loaded by Horos at runtime.

**Special directories:**
- `_obsolete/` — 3 deprecated plugins (64bit, AutoClean, Casimage)
- `_help/` — documentation (PluginsManual.pdf, OsiriX Plugin Generator, ITK plugins help)

## Building Plugins

Plugins link against `OsiriXAPI.framework` or `HorosAPI.framework` from the main Horos build. Each plugin builds independently via its Xcode project.

Prerequisites: A built copy of Horos (for the API framework).

## Plugin SDK

All plugins subclass `PluginFilter` from the Horos SDK:

```objc
#import <OsiriXAPI/PluginFilter.h>      // or HorosAPI/PluginFilter.h
#import <OsiriXAPI/ViewerController.h>
#import <OsiriXAPI/DCMPix.h>
#import <OsiriXAPI/DCMView.h>
#import <OsiriXAPI/ROI.h>
```

**Entry points:** `filterImage:`, `processFiles:`, `report:action:`, `initPlugin`, `willUnload`

**Plugin types** (set in Info.plist `pluginType` key):
- `imageFilter` — image processing filters
- `roiTool` — ROI creation/manipulation tools
- `Database` — database browser extensions
- `fusionFilter` — image fusion
- `other` — general purpose

**Info.plist keys:** `NSPrincipalClass`, `pluginType`, `MenuTitles` (array), `ToolbarIcon`, `allowToolbarIcon`

## Current State — Modernization Required

**Architecture:** No plugin has arm64 support configured. All target x86_64 or older (i386, ppc).

**Deprecated APIs in use:**
- NSOpenGLView (deprecated macOS 10.14) — used by plugins with custom rendering
- Carbon.framework — legacy C APIs
- QuickTime.framework / QTKit — removed from modern macOS
- NSMatrix — deprecated UI class
- Method swizzling patterns for extending BrowserController

**SDK transition:** Only 4 plugins reference HorosAPI; the rest use OsiriXAPI (same code, different name).

## Plugin Inventory by Complexity

**Heavy dependencies (ITK/VTK — need careful migration):**
- CMIV_CTA_TOOLS (39 source files, VTK)
- PetSpectFusion (ITK + VTK, registration algorithms)
- NMSegmentation (ITK, region growing)
- VoxelVolume (embedded OsiriXAPI framework, VTK)

**OpenGL rendering (need Metal migration):**
- OpenGL (demo plugin with NSOpenGLView subclass kGLView)
- Any plugin using `drawInOpenGLView:` protocol method

**Moderate complexity:**
- MIRC Teaching File (65 source files, WebKit, SeqGrab)
- HipArthroplastyTemplating (custom image manipulation)
- DiscPublishing (Growl, PTRobot framework)
- ExtraDatabaseColumnsSample (runtime method injection)

**Simple (minimal dependencies):**
- HelloWorld, CloseThisStudy, Duplicate, Cobb Angle, Sample Menu, WindowAnchoredAnnotations

## Migration Plan

Tracked under Sprint 8 (Plugin SDK Modernization) in `gerinsights/horos` issues:
1. New plugin SDK version 5.0 with Metal rendering protocol
2. Hard break — no OpenGL backward compatibility shim
3. Each plugin gets its own migration checklist
4. Common issues: OpenGL drawing, NSMatrix UI, Carbon deps, x86_64-only binaries
