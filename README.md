# Flutter Environment Management Guide

## Overview

This (demo) project implements a robust environment management system for Flutter applications using a combination of build flavors and runtime configuration. The system supports multiple environments with secure configuration handling and seamless development workflow.

## Environment Strategy

- Two environments: **staging** (default) and **production**
- Multiple package names for simultaneous installation:
  - Staging: `com.ourapp.staging`
  - Production: `com.ourapp`

## Native Configuration

### Android (app/build.gradle)

```gradle
android {
    flavorDimensions "environment"
    productFlavors {
        staging {
            dimension "environment"
            applicationIdSuffix ".staging"
            resValue "string", "app_name", "AppName Staging"
        }
        production {
            dimension "environment"
            resValue "string", "app_name", "AppName"
        }
    }
}
```

### iOS (Xcode Setup)

1. Duplicate the "Runner" scheme to create "Staging" and "Production" schemes
2. Configure Bundle Identifiers:
   - Staging: `com.ourapp.staging`
   - Production: `com.ourapp`
3. Set appropriate display names for each scheme

## Security Measures

### Environment File Security

- Environment files located in `assets/env/` use **Base64 encoding**
- File naming: `env.staging.urls.json.encoded`, `env.production.urls.json.encoded`
- Decoding occurs at runtime within the application

### .flutter_env Strategy

```bash
# Initial commit contains placeholder values
FLUTTER_ENV=development
API_BASE_URL=https://example.com/api/v1/
API_KEY=api_key_here
DEBUG=true

# Added to .gitignore after initial commit
```

### CI/CD Security

- **No .flutter_env usage** in pipelines
- Uses **environment variables** from CI/CD platforms
- Secrets securely stored in **Bitrise Secrets/GitHub Secrets**
- Secure builds using `--dart-define` parameters

## Development Workflow

### Terminal Commands

```bash
# Auto environment detection (defaults to staging)
fvm flutter run

# Explicit flavor selection
fvm flutter run --flavor staging
fvm flutter run --flavor production

# Distribution builds
fvm flutter build apk --flavor staging
fvm flutter build appbundle --flavor production
```

### VS Code Integration (`.vscode/launch.json`)

```json
{
  "configurations": [
    {
      "name": "Staging",
      "request": "launch",
      "type": "dart",
      "program": "lib/main.dart",
      "args": ["--flavor", "staging"]
    },
    {
      "name": "Production",
      "request": "launch",
      "type": "dart", 
      "program": "lib/main.dart",
      "args": ["--flavor", "production"]
    }
  ]
}
```

## Environment Detection Logic

Priority-based environment detection:

1. Dart defines (`--dart-define=FLAVOR=...`)
2. CI/CD environment variables
3. .flutter_env file (local development)
4. Package name detection (staging suffix)
5. Default to staging

## Team Collaboration & Onboarding

### Documentation

```markdown
## Development Environment Setup

1. **Clone repository**
2. **Setup FVM**: `fvm install && fvm use`
3. **Get dependencies**: `fvm flutter pub get`
4. **Run application**: `fvm flutter run`

## Environment Configuration

- `.flutter_env` file available with placeholder values
- Request real values from team lead for environment access
- Edit `.flutter_env` with appropriate values

## Distribution Building

# Staging
fvm flutter build apk --flavor staging

# Production
fvm flutter build appbundle --flavor production
```

## CI/CD Implementation

### Bitrise Example

```yaml
workflows:
  staging:
    envs:
      - FLAVOR: staging
      - ENCRYPTED_ENV: true
    steps:
      - flutter-build:
          inputs:
            - platform: android
            - additional_flutter_args: |
                --flavor staging
                --dart-define=FLAVOR=staging
                --dart-define=API_KEY=$STAGING_API_KEY
```

### GitHub Actions Example

```yaml
- name: Build Production
  run: |
    flutter build appbundle \
      --flavor production \
      --dart-define=FLAVOR=production \
      --dart-define=API_KEY=${{ secrets.PRODUCTION_API_KEY }}
```

## Advantages

1. **Multiple Installations** - staging & production coexist on device
2. **Zero Script Development** - no wrapper scripts needed
3. **Full IDE Integration** - VS Code debugging support
4. **Security First** - no secrets in repository
5. **Team Scalability** - consistent onboarding process
6. **Build Flexibility** - support both APK and App Bundle
7. **CI/CD Ready** - seamless automation integration

## Implementation Checklist

1. [ ] Configure Android build.gradle with flavors
2. [ ] Setup iOS schemes in Xcode
3. [ ] Implement environment detection and parsing
4. [ ] Configure dependency injection with GetIt
5. [ ] Create encoded environment files
6. [ ] Create .flutter_env template and add to .gitignore
7. [ ] Setup VS Code launch configurations
8. [ ] Implement CI/CD pipeline
9. [ ] Create comprehensive documentation

This implementation provides a secure, flexible, and easy-to-use environment management system for scalable team development.
