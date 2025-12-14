# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ARCED Collect is a specialized fork of ODK Collect, customized for ARCED Foundation (www.arced.foundation). It's an Android application designed for form filling in resource-constrained environments with unreliable connectivity or power infrastructure.

**Key Distinction**: This is a fork maintaining the same package name (`org.odk.collect.android`) for compatibility with existing ODK Collect integrations, but with ARCED Foundation branding and custom features.

## Build System and Commands

### Essential Build Commands

```bash
# Build tasks
./gradlew assembleDebug                    # Build debug APK
./gradlew assembleRelease                  # Build release APK
./gradlew assembleSelfSignedRelease       # Build self-signed ARCED release
./gradlew testDebug                        # Run unit tests
./gradlew connectedAndroidTest             # Run instrumented tests
./gradlew checkCode                        # Run quality checks (pmd, ktlint, etc.)

# Development tasks
./gradlew installDebug                    # Install debug build
./gradlew lintDebug                        # Run Android lint
./gradlew assembleDebug -Px86Libs         # Build with x86 libraries for emulators

# Memory-intensive builds (when encountering memory issues)
./gradlew assembleSelfSignedRelease -Dorg.gradle.jvmargs=-Xmx4g
```

### Build Variants

This project supports multiple build variants:

1. **`debug`** - Development builds with full debugging enabled
2. **`release`** - Production builds with code obfuscation via ProGuard
3. **`odkCollectRelease`** - Official ODK Collect release build (maintains original branding)
4. **`selfSignedRelease`** - Self-signed ARCED Collect release (primary ARCED variant)

**Important**: The `selfSignedRelease` variant is the primary ARCED Collect build that includes:
- ARCED Foundation branding
- Custom version format: `v2025.3.0-beta.3.arced.1.0`
- ARCED-specific features (entity management, etc.)
- Proper APK naming: `ARCED-Collect-v2025.3.0-beta.3.arced.1.0.apk`

## Version Management

### Version Format

ARCED Collect uses a custom version format:
```
v2025.3.0-beta.3.arced.1.0
```

**Breakdown**:
- `v2025.3.0` - Base version (aligned with ODK releases)
- `beta.3` - Beta number
- `arced.1.0` - ARCED-specific suffix (version increment for ARCED changes)

### Version Code

Current version code: `60001` (incremented for ARCED-specific releases)

### Version Display

The footer displays: "ARCED Collect v2025.3.0 Beta 3 ARCED 1.0"

**Implementation**:
- `VersionInformation.getVersionToDisplayWithBranding()` - Formats version with ARCED branding
- `MainMenuViewModel.versionWithBranding` - Exposes formatted version
- `MainMenuFragment` - Displays branded version in footer

## Architecture Overview

### Module Structure

The project uses a Gradle monorepo architecture with clear module separation:

**Core Application Modules**:
- `collect_app` - Main Android application module
- `projects` - Project management functionality
- `shared` - Shared utilities and common code
- `androidshared` - Android-specific shared components

**Feature Modules**:
- `forms` - Form parsing and management (JavaRosa 5.1.0)
- `analytics` - Firebase Analytics integration
- `location` - Location services and GPS functionality
- `geo` - Geographic features and map integration
- `audio-recorder` - Audio recording capabilities
- `camera` - Camera functionality with ML Kit
- `qr-code` - QR code scanning (ZXing integration)
- `entities` - ARCED-specific entity management system

**Infrastructure Modules**:
- `db` - Database layer (Room)
- `permissions` - Permission handling
- `settings` - Application settings management
- `upgrade` - Migration and upgrade utilities
- `crash-handler` - Crash reporting (Firebase Crashlytics)

### Dependency Injection

Uses Dagger 2.56.2 with Hilt-like pattern:
- Root component: `AppDependencyComponent`
- Feature-specific modules: Geo, Maps, Camera, etc.
- Located in `collect_app/src/main/java/org/odk/collect/android/injection/`

### Form Processing

- **Core Library**: JavaRosa 5.1.0 for XForms parsing
- **Standard**: ODK XForms standard compliance
- **Extensions**: Custom form extensions for ARCED-specific features

## Key Development Considerations

### Package Name Compatibility

**Critical**: Maintaining `org.odk.collect.android` package name for:
- Installation compatibility with original ODK Collect
- Integration compatibility with existing ODK Collect tools and workflows
- No need for users to uninstall existing ODK Collect installations

### URL Configuration Strategy

**Mixed URL Approach** (as defined by ARCED requirements):
- **Brand URLs**: `https://arced.foundation` (ARCED Foundation branding)
- **Technical URLs**: `https://getodk.org` (ODK technical documentation)
- **Forum URL**: `https://forum.getodk.org` (no ARCED forum available)

### Secrets Management

Create `secrets.properties` in project root with required keys:

```properties
# Required for Google Maps integration
GOOGLE_MAPS_API_KEY=your_api_key_here

# Required for Mapbox integration
MAPBOX_ACCESS_TOKEN=your_token_here

# ARCED-specific configurations
ENTITIES_FILTER_PROJECT_URL=https://your-arced-server.com/api/entities
ENTITIES_FILTER_SEARCH_PROJECT_URL=https://your-arced-server.com/api/entities/search
THOUSAND_MEDIA_FILE_PROJECT_URL=https://your-arced-server.com/api/media
THOUSAND_MEDIA_FILE_ENTITY_LIST_PROJECT_URL=https://your-arced-server.com/api/entities

# Release signing (for production builds)
RELEASE_STORE_FILE=/path/to/keystore
RELEASE_STORE_PASSWORD=store_password
RELEASE_KEY_ALIAS=key_alias
RELEASE_KEY_PASSWORD=key_password
```

### ARCED-Specific Features

1. **Entity Management System**:
   - Configurable project URLs for entity filtering
   - Entity list management for large-scale data collection
   - Media file handling with entity associations

2. **Custom Version Display**:
   - Footer shows "ARCED Collect" branding
   - Custom version format with ARCED suffix
   - Version extraction and formatting in `VersionInformation.java`

3. **Branding Elements**:
   - ARCED logo assets (already in place)
   - ARCED Foundation color scheme
   - Custom app name in untranslated.xml

### Testing Strategy

**Unit Tests**:
- Location: `src/test/java`
- Framework: JUnit 4, Mockito, Robolectric
- Command: `./gradlew testDebug`

**Instrumented Tests**:
- Location: `src/androidTest/java`
- Framework: Espresso, AndroidX Test
- Command: `./gradlew connectedAndroidTest`

**Test Forms**:
- Location: `test-forms/` directory
- Used for integration testing with real form definitions

### Quality Assurance

**Static Analysis**:
- PMD, Checkstyle, ktlint enabled via `./gradlew checkCode`
- Automatic enforcement in CI/CD

**Code Coverage**:
- Automated via CircleCI
- Target coverage maintained for core functionality

### Common Development Issues

**Memory Issues**:
- Use `-Dorg.gradle.jvmargs=-Xmx4g` for builds
- Configure in `gradle.properties` for persistent settings

**Android SDK Issues**:
- Ensure Java 17 compatibility
- Set `JAVA_HOME` for proper SDK integration

**Robolectric Tests**:
- Use `./download-robolectric-deps.sh` if dependency issues occur
- Configure working directory in Android Studio to `$MODULE_DIR$`

## Release Process

### ARCED Collect Release Steps

1. **Update Version**:
   - Modify `getVersionName()` in `collect_app/build.gradle`
   - Update `versionCode` if needed
   - Follow format: `v2025.3.0-beta.3.arced.1.0`

2. **Build Self-Signed Release**:
   ```bash
   ./gradlew assembleSelfSignedRelease -Dorg.gradle.jvmargs=-Xmx4g
   ```

3. **Verify APK**:
   - Check APK name: `ARCED-Collect-v2025.3.0-beta.3.arced.1.0.apk`
   - Verify version display in app footer
   - Test ARCED-specific features

4. **Commit Changes**:
   - Include version update commit
   - Test build verification

### Continuous Integration

**CircleCI Configuration**:
- Docker: cimg/android:2025.04
- Automated testing across multiple device configurations
- APK artifacts available on successful builds
- Firebase Test Lab integration for device testing

## Legal and Compliance

### Apache License 2.0 Compliance

- Maintain proper attribution to ODK project
- Keep license headers in all source files
- Preserve ODK references in comments (user-facing strings only updated)

### Privacy Policy

- Comprehensive privacy policy in `PRIVACY_POLICY.md`
- Covers data collection, usage, security, and compliance requirements
- Aligns with GDPR and CCPA standards

### ARCED Foundation Branding

- Use "ARCED Collect" for user-facing references
- Maintain "ODK Collect" for technical documentation and attribution
- Preserve ODK forum URL (no ARCED forum available)