# Jetpack Common - Architecture & Patterns

A centralized knowledge base of architectural patterns and reusable UI components used across the Essentials and AirSync projects.

## Contents

- **[ui/](ui/)**: User interface components and layout strategies.
  - **[layout/](ui/layout/)**: High-level screen structures and animations.
    - [Edge-to-Edge Guide](ui/layout/edge-to-edge.md)
    - [Splash Screen Pattern](ui/layout/splash.md)
    - [Welcome & Onboarding](ui/layout/welcome.md)
    - [Swipe Navigation Pattern](ui/layout/swipe-navigation.md)
    - [App Menu Shortcuts](ui/layout/app-menu-shortcuts.md)
    - [Progressive Blur Implementation](ui/layout/progressive-blur.md)
  - **[Theming & AMOLED](ui/theme.md)**
  - **[components/](ui/components/)**: Reusable Material 3 components.
    - **[Containers](ui/components/containers/)**: Layout wrappers.
      - [Rounded Card Container](ui/components/containers/rounded-card-container.md)
    - **[Cards & Items](ui/components/cards/)**: List items and feature cards.
      - [List Item Variants](ui/components/cards/list-items.md)
      - [Expandable Section Toggle Button](ui/components/cards/list-expand-toggle-button.md)
    - **[About & Community](ui/components/about.md)**: App info and developer credits.
    - **[Bottom Sheets](ui/components/sheets/)**: Modal overlays and guides.
      - [General Layout](ui/components/sheets/general-layout.md)
      - [Help & Guides](ui/components/sheets/help-guide-sheet.md)
      - [Feature Info](ui/components/sheets/feature-info-sheet.md)
    - **[Pickers](ui/components/pickers/)**: Selection components.
      - [Segmented Picker Pattern](ui/components/pickers/picker.md)
    - **[Toolbars](ui/components/toolbar/)**: Variants of top and bottom bars.
      - [Floating Toolbar](ui/components/toolbar/floating-toolbar.md)
      - [Top App Bar](ui/components/toolbar/top-app-bar.md)
  - **[theme/](ui/theme/)**: Design system and theming guides.
- **[logic/](logic/)**: Business logic and utility patterns.
  - **[search/](logic/search/)**: Implementation of global search.
    - [Deep Search Pattern](logic/search/deep-search.md)
  - **[services/](logic/services/)**: Background services and system integrations.
    - [Quick Settings Tiles](logic/services/qs-tiles.md)
    - [Live Update Notifications](logic/services/live-updates.md)
    - [In-App Updates](logic/services/updates.md)
  - **[utils/](logic/utils/)**: Common utility functions.
    - [Haptic Feedback](logic/utils/haptics.md)
    - [Developer Mode](logic/utils/developer-mode.md)
    - [Language Switching](logic/utils/languages.md)
    - [App Selection Picker](logic/utils/app-picker.md)
    - [Flashlight & Brightness](logic/utils/flashlight.md)
    - [Media Playback Info](logic/utils/media-playback.md)
    - [Shell Command Execution](logic/utils/shell-cmd.md)
    - [WearOS Communication Logic](logic/utils/wearos.md)
    - [Full Backup & Restore](logic/utils/backup-restore.md)
    - [Cross-App Communication](logic/utils/cross-app-communication.md)
- **[wearos/](wearos/)**: Specialized WearOS components.
  - [Glanceable Tiles](wearos/wearos-tile.md)
  - [System Complications](wearos/wearos-complication.md)
  - [Enhanced Time View](wearos/wearos-time.md)
- **[permissions/](permissions/)**: Permission handling strategies.
  - [Shizuku Integration](permissions/shizuku.md)
- **[accessibility/](accessibility/)**: System-level interaction patterns.
  - [Accessibility Overlays](accessibility/accessibility-overlays.md)
- **[data/](data/)**: Data persistence and migration patterns.

## Getting Started

Refer to the specific categories to find implementation details and code snippets. This repository serves as the source of truth for the Essentials ecosystem design language.
