# CLAUDE.md - AI Assistant Guide for myTag

## Project Overview

myTag is a React Native mobile app (iOS/Android/Web) that automatically tags images using on-device machine learning (TensorFlow.js + COCO-SSD). Users can browse their photo library organized by detected objects (person, dog, car, etc.). Tags are computed once and cached locally via AsyncStorage.

**Version:** 1.0.4 (v1.1.0 in progress)
**License:** MIT

## Tech Stack

- **Language:** JavaScript (ES6+)
- **Framework:** React Native via Expo (SDK 40)
- **ML:** TensorFlow.js + COCO-SSD object detection model
- **Navigation:** React Navigation 5.x (Stack navigator)
- **State:** React Context API + Hooks
- **Storage:** AsyncStorage (local device cache)
- **Build:** Expo CLI + EAS (Expo Application Services)
- **Testing:** Jest (via jest-expo)
- **Linting:** ESLint (Standard JS + React + Jest plugins)
- **Git Hooks:** Husky + lint-staged (pre-commit: eslint --fix + jest)

## Common Commands

```bash
# Development
npm start              # Start Expo dev server
npm run android        # Start on Android
npm run ios            # Start on iOS
npm run web            # Start on web

# Testing & Quality
npm test               # Run Jest test suite
npm run fix            # Auto-fix ESLint issues in src/ and App.js

# Docker development
docker-compose run expo  # Then run: expo start
```

## Project Structure

```
App.js                          # Entry point: theming + navigation setup
src/
  screens/
    HomeScreen.js               # Main view: scans images, displays tag grid
    BrowseImagesScreen.js       # Shows images for a selected tag
    ImageScreen.js              # Single image detail with detected objects
  components/
    TagsGrid.js                 # Grid of tag categories
    ImageGrid.js                # 3-column image grid layout
    ImageItem.js                # Individual image thumbnail tile
    DetectedImage.js            # Image with detection overlay boxes
    LoadingClassificationsBar.js # ML scanning progress bar
    ShareButton.js              # Share image functionality
    LogoTitle.js                # Header logo component
  models/
    ObjectDetector.js           # TensorFlow/COCO-SSD model wrapper (singleton)
    Tag.js                      # Tag data model (name + metadata[])
    TaggedImage.js              # Image + tags data model
    TagsCollection.js           # In-memory tag collection with observer pattern
    services/
      AutoTagService.js         # Generates tags via ML predictions
      TagFinderService.js       # Orchestrates tag retrieval (cache-first)
  persistence/
    DetectionStorage.js         # Caches raw ML predictions per image URI
    TagsStorage.js              # Caches computed tags per image URI
    adapters/
      TaggedImageAdapter.js     # JSON serialization for TaggedImage
  config/
    themes.js                   # Light/dark theme color definitions
  utils/
    TagsContext.js              # React Context provider for global tag state
    ToTitleCase.js              # String utility
tests/
  models/
    Tag.test.js                 # Unit tests for Tag model
```

## Architecture

### Layer Pattern

1. **Screens** - React Navigation screens (HomeScreen, BrowseImagesScreen, ImageScreen)
2. **Components** - Reusable UI (TagsGrid, ImageGrid, DetectedImage, etc.)
3. **Services** - Business logic (TagFinderService, AutoTagService)
4. **Models** - Data models (Tag, TaggedImage, TagsCollection, ObjectDetector)
5. **Persistence** - AsyncStorage caching (DetectionStorage, TagsStorage)

### Data Flow

```
HomeScreen → tagsCollection.useImages()
  → tagFinder.findTags(image)
    → TagsStorage cache hit? → return cached tags
    → Cache miss → AutoTagService.generateTags()
      → ObjectDetector.classifyImage()
        → DetectionStorage cache hit? → return cached predictions
        → Cache miss → COCO-SSD inference → cache & return
```

### Key Patterns

- **Singleton:** ObjectDetector instance, TagsCollection
- **Observer:** TagsCollection notifies listeners on tag updates
- **Cache-first:** Two-level caching (predictions + tags) for performance
- **Context API:** TagsContext provides global tag state to all screens

## Code Conventions

- **Style:** Standard JS (enforced by ESLint) - 2-space indentation, semicolons
- **Components:** Functional components with hooks (no class components)
- **State:** React hooks (useState, useContext, useEffect, useLayoutEffect)
- **Props:** prop-types validation not enforced (rule disabled)
- **Naming:** PascalCase for components/classes, camelCase for functions/variables
- **File naming:** PascalCase for components/models/screens, camelCase for utils

## CI/CD

- **main.yml:** Runs on push/PR - `npm ci`, ESLint linting, Jest tests
- **Android-Build.yml:** Manual/push-to-main - builds APK (preview) and app-bundle (production) via EAS
- **Claude.yml:** AI assistant integration - triggered by `@claude` mentions on issues/PRs

## Pre-commit Checks

Husky + lint-staged runs on every commit for `*.js` files:
1. `eslint --cache --fix` (auto-fixes style issues)
2. `jest --bail --findRelatedTests` (runs relevant tests)

Both must pass for the commit to succeed.

## Environment Setup

- Copy `.env.template` to `.env` and fill in Expo credentials
- Node.js required (v20 for CI)
- Java 17 required for Android builds
- Docker alternative available via `docker-compose.yml`

## Testing

- Test files go in `tests/` directory mirroring `src/` structure
- Use Jest assertions and patterns
- Current coverage is minimal - only `tests/models/Tag.test.js` exists
- Pre-commit hook runs related tests automatically

## Important Notes

- The app requests READ/WRITE_EXTERNAL_STORAGE permissions on Android
- COCO-SSD model is loaded lazily on first image classification
- All ML inference happens on-device (no network calls for tagging)
- Portrait orientation only (configured in app.json)
- Supports automatic light/dark theme via system preferences
