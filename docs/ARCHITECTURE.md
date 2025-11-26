# Requestify Architecture Documentation

## System Architecture Overview

This document describes the architecture of the Requestify project, including the library, demo application, and CI/CD infrastructure.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           Requestify Ecosystem                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌────────────────────┐          ┌─────────────────────┐               │
│  │   Consumer Apps    │          │   RequestifyApp     │               │
│  │  (iOS/macOS/etc)   │          │   (Demo App)        │               │
│  └─────────┬──────────┘          └──────────┬──────────┘               │
│            │                                 │                          │
│            │                                 │                          │
│            └─────────────┬───────────────────┘                          │
│                          │                                              │
│                          ▼                                              │
│            ┌─────────────────────────────┐                              │
│            │   Requestify Library        │                              │
│            │   (Swift Package)           │                              │
│            └──────────────┬──────────────┘                              │
│                           │                                             │
│                           ▼                                             │
│            ┌─────────────────────────────┐                              │
│            │      Alamofire              │                              │
│            │   (HTTP Networking)         │                              │
│            └──────────────┬──────────────┘                              │
│                           │                                             │
│                           ▼                                             │
│            ┌─────────────────────────────┐                              │
│            │    URLSession / Network     │                              │
│            │       (Foundation)          │                              │
│            └──────────────┬──────────────┘                              │
│                           │                                             │
└───────────────────────────┼─────────────────────────────────────────────┘
                            │
                            ▼
                ┌───────────────────────┐
                │   Remote API Server   │
                │   (HTTP/HTTPS)        │
                └───────────────────────┘
```

## Component Architecture

### 1. Requestify Library (Core)

```
┌────────────────────────────────────────────────────────┐
│                  Requestify Library                     │
├────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────────────────────────────────┐     │
│  │            Requestify.swift                   │     │
│  │  ┌────────────────────────────────────────┐  │     │
│  │  │  - url: String?                        │  │     │
│  │  │  - method: HTTPMethod                  │  │     │
│  │  │  - headers: HTTPHeaders?               │  │     │
│  │  │  - parameters: [String: Any]?          │  │     │
│  │  │  - printLog: Bool                      │  │     │
│  │  │  - printResponse: Bool                 │  │     │
│  │  │  - multipartImages: [UIImage]?         │  │     │
│  │  │  - multipartObject: Encodable?         │  │     │
│  │  └────────────────────────────────────────┘  │     │
│  │                                               │     │
│  │  Methods:                                     │     │
│  │  ├─ setURL(_ url: String)                    │     │
│  │  ├─ setMethod(_ method: HTTPMethod)          │     │
│  │  ├─ setHeaders(_ headers: HTTPHeaders)       │     │
│  │  ├─ setParameters(_ params: [String: Any])   │     │
│  │  ├─ setPrintLog(_ print: Bool)               │     │
│  │  ├─ setPrintResponse(_ print: Bool)          │     │
│  │  ├─ addImages(_ images: [UIImage])           │     │
│  │  ├─ addObject<T: Encodable>(_ obj: T)        │     │
│  │  ├─ request<T: Decodable>(_ type: T.Type)    │     │
│  │  ├─ requeust() -> HTTPURLResponse            │     │
│  │  └─ upload<T: Decodable>(_ type: T.Type)     │     │
│  └──────────────────────────────────────────────┘     │
│                                                         │
│  ┌──────────────────────────────────────────────┐     │
│  │              Params.swift                     │     │
│  │  ┌────────────────────────────────────────┐  │     │
│  │  │  - parameters: [String: Any]           │  │     │
│  │  └────────────────────────────────────────┘  │     │
│  │                                               │     │
│  │  Methods:                                     │     │
│  │  ├─ add(_ key: String, value: Any)           │     │
│  │  ├─ build() -> [String: Any]                 │     │
│  │  └─ Builder Pattern Support                  │     │
│  └──────────────────────────────────────────────┘     │
│                                                         │
│  ┌──────────────────────────────────────────────┐     │
│  │                 Utils/                        │     │
│  │  ┌────────────────────────────────────────┐  │     │
│  │  │  Encodable+.swift                      │  │     │
│  │  │  - Dictionary conversion extensions    │  │     │
│  │  └────────────────────────────────────────┘  │     │
│  │  ┌────────────────────────────────────────┐  │     │
│  │  │  printApiLog.swift                     │  │     │
│  │  │  - Request logging                     │  │     │
│  │  │  - Response logging                    │  │     │
│  │  │  - Error logging                       │  │     │
│  │  └────────────────────────────────────────┘  │     │
│  │  ┌────────────────────────────────────────┐  │     │
│  │  │  RequestBuilderError.swift             │  │     │
│  │  │  - Error definitions                   │  │     │
│  │  │  - Custom error types                  │  │     │
│  │  └────────────────────────────────────────┘  │     │
│  └──────────────────────────────────────────────┘     │
└────────────────────────────────────────────────────────┘
```

### 2. RequestifyApp (Demo Application)

```
┌────────────────────────────────────────────────────────┐
│                  RequestifyApp                          │
├────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────────────────────────────────┐     │
│  │          AppDelegate.swift                    │     │
│  │  - Application lifecycle management          │     │
│  │  - Window setup                               │     │
│  │  - Root view controller configuration         │     │
│  └──────────────────────────────────────────────┘     │
│                          │                              │
│                          ▼                              │
│  ┌──────────────────────────────────────────────┐     │
│  │        ViewController.swift                   │     │
│  │  - Demo UI implementation                     │     │
│  │  - Requestify usage examples                  │     │
│  │  - Status display                             │     │
│  └──────────────────────────────────────────────┘     │
│                          │                              │
│                          ▼                              │
│  ┌──────────────────────────────────────────────┐     │
│  │         Requestify Library                    │     │
│  │  (Swift Package Dependency)                   │     │
│  └──────────────────────────────────────────────┘     │
└────────────────────────────────────────────────────────┘
```

### 3. Build & CI/CD Infrastructure

```
┌──────────────────────────────────────────────────────────────────┐
│                  Build & Deployment Pipeline                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌────────────────────────────────────────────────────────┐     │
│  │             GitHub Repository                           │     │
│  │  - Source code (Sources/, Tests/)                      │     │
│  │  - Package.swift (SPM manifest)                        │     │
│  │  - project.yml (XcodeGen spec)                         │     │
│  └───────────────────┬────────────────────────────────────┘     │
│                      │                                           │
│                      ▼                                           │
│  ┌────────────────────────────────────────────────────────┐     │
│  │         GitHub Actions (macOS-latest)                  │     │
│  │                                                         │     │
│  │  Stage 1: Package Build                                │     │
│  │  ├─ swift package resolve                              │     │
│  │  ├─ swift build -v                                     │     │
│  │  └─ swift test --enable-code-coverage -v               │     │
│  │                                                         │     │
│  │  Stage 2: Project Generation                           │     │
│  │  ├─ brew install xcodegen                              │     │
│  │  └─ xcodegen generate (creates .xcodeproj)             │     │
│  │                                                         │     │
│  │  Stage 3: Code Signing (Optional)                      │     │
│  │  ├─ Create keychain                                    │     │
│  │  ├─ Import certificate                                 │     │
│  │  ├─ Install provisioning profile                       │     │
│  │  └─ Generate exportOptions.plist                       │     │
│  │                                                         │     │
│  │  Stage 4: Archive                                      │     │
│  │  ├─ Detect scheme                                      │     │
│  │  └─ xcodebuild archive                                 │     │
│  │                                                         │     │
│  │  Stage 5: IPA Export                                   │     │
│  │  ├─ xcodebuild -exportArchive (signed)                 │     │
│  │  └─ Manual ZIP packaging (fallback)                    │     │
│  │                                                         │     │
│  │  Stage 6: Artifact Upload                              │     │
│  │  ├─ Upload IPA (retention: 5 days)                     │     │
│  │  └─ Upload logs (on failure)                           │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

## Data Flow Architecture

### Request Flow

```
User App                Requestify              Alamofire           API Server
   │                        │                       │                    │
   ├─ Initialize ──────────>│                       │                    │
   │                        │                       │                    │
   ├─ Configure ───────────>│                       │                    │
   │  (URL, method,         │                       │                    │
   │   headers, params)     │                       │                    │
   │                        │                       │                    │
   ├─ request(Type.self) ──>│                       │                    │
   │                        │                       │                    │
   │                        ├─ Validate ────────────┤                    │
   │                        │                       │                    │
   │                        ├─ Create Request ─────>│                    │
   │                        │                       │                    │
   │                        │                       ├─ HTTP Request ────>│
   │                        │                       │                    │
   │                        │                       │<── HTTP Response ──┤
   │                        │                       │                    │
   │                        │<─── Response Data ────┤                    │
   │                        │                       │                    │
   │                        ├─ Decode JSON ─────────┤                    │
   │                        │                       │                    │
   │                        ├─ Log (if enabled)     │                    │
   │                        │                       │                    │
   │<── Return Object ──────┤                       │                    │
   │                        │                       │                    │
```

### Error Handling Flow

```
Request ──> Validation ──> Network Call ──> Response Processing
    │            │               │                    │
    │            │               │                    │
    ▼            ▼               ▼                    ▼
    │            │               │                    │
    │     Invalid Config    Network Error      Decode Error
    │            │               │                    │
    └────────────┴───────────────┴────────────────────┘
                          │
                          ▼
                   Log Error (if enabled)
                          │
                          ▼
                   Throw Error to Caller
```

## Design Patterns

### 1. Builder Pattern
Used in `Requestify` and `Params` classes for fluent interface:
```swift
let request = Requestify()
    .setURL("https://api.example.com")
    .setMethod(.post)
    .setHeaders(headers)
    .setParameters(params)
```

### 2. Async/Await Pattern
Modern Swift concurrency for asynchronous operations:
```swift
let response: User = try await requestify.request(User.self)
```

### 3. Dependency Injection
Alamofire is injected as a dependency via Swift Package Manager

### 4. Error Handling
Custom error types with Swift's Result and throw/catch pattern

## Security Architecture

```
┌────────────────────────────────────────────────────┐
│              Security Layers                        │
├────────────────────────────────────────────────────┤
│                                                     │
│  1. Transport Security                             │
│     └─ HTTPS enforcement                           │
│     └─ TLS 1.2+ support                            │
│                                                     │
│  2. Code Signing (Production)                      │
│     └─ Apple Developer certificate                 │
│     └─ Provisioning profiles                       │
│     └─ Team ID verification                        │
│                                                     │
│  3. Secrets Management (CI/CD)                     │
│     └─ GitHub Secrets encryption                   │
│     └─ Base64 encoding for certificates            │
│     └─ Temporary keychain (auto-deleted)           │
│                                                     │
│  4. Data Validation                                │
│     └─ Codable protocol enforcement                │
│     └─ Type-safe JSON decoding                     │
│     └─ URL validation                              │
│                                                     │
└────────────────────────────────────────────────────┘
```

## Platform Support Matrix

| Platform | Minimum Version | Status | Notes |
|----------|----------------|--------|-------|
| iOS | 13.0+ | ✅ Fully Supported | Primary target |
| macOS | 10.15+ | ✅ Fully Supported | Catalyst compatible |
| tvOS | 12.0+ | ✅ Supported | Limited testing |
| watchOS | 4.0+ | ✅ Supported | Limited testing |

## Performance Considerations

### 1. Request Lifecycle
- **Setup**: O(1) - Constant time configuration
- **Network Call**: Variable (network dependent)
- **JSON Decoding**: O(n) - Linear with response size
- **Logging**: O(1) - Minimal overhead when disabled

### 2. Memory Management
- ARC (Automatic Reference Counting) for memory cleanup
- Async/await prevents retain cycles
- Weak references in closures where appropriate

### 3. Threading
- Network calls execute on background queues
- Main thread updates handled automatically
- No manual thread management required

## Scalability

The architecture supports:
- Multiple simultaneous requests
- Different API endpoints
- Custom authentication schemes
- Middleware integration
- Custom interceptors (via Alamofire)

## Future Architecture Enhancements

### Planned Improvements
1. **Caching Layer** - Response caching with configurable policies
2. **Retry Logic** - Automatic retry with exponential backoff
3. **Rate Limiting** - Request throttling support
4. **Metrics Collection** - Performance monitoring
5. **GraphQL Support** - Alternative to REST
6. **WebSocket Support** - Real-time communication

---

**Document Version**: 1.0  
**Last Updated**: November 2025  
**Maintained By**: Requestify Team
