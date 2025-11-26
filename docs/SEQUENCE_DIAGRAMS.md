# Requestify Sequence Diagrams

This document contains detailed sequence diagrams for various Requestify operations and workflows.

## Table of Contents
1. [Standard Request Flow](#1-standard-request-flow)
2. [Multipart Upload Flow](#2-multipart-upload-flow)
3. [Error Handling Flow](#3-error-handling-flow)
4. [CI/CD Pipeline Flow](#4-cicd-pipeline-flow)
5. [Code Signing Flow](#5-code-signing-flow)

---

## 1. Standard Request Flow

### GET Request with Logging

```mermaid
sequenceDiagram
    participant App as iOS Application
    participant Req as Requestify
    participant Log as Logger
    participant AF as Alamofire
    participant API as API Server

    App->>Req: Requestify()
    App->>Req: setURL("https://api.example.com/users")
    App->>Req: setMethod(.get)
    App->>Req: setPrintLog(true)
    App->>Req: setPrintResponse(true)
    
    App->>Req: await request([User].self)
    
    Req->>Req: Validate URL
    alt URL is invalid
        Req-->>App: Throw RequestBuilderError
    end
    
    Req->>Log: Print request details
    Log-->>App: Console output (URL, Method, Headers)
    
    Req->>AF: AF.request(url, method: .get)
    AF->>API: HTTP GET /users
    
    alt Network Error
        API-->>AF: Connection failed
        AF-->>Req: AFError
        Req->>Log: Print error
        Req-->>App: Throw network error
    end
    
    API-->>AF: 200 OK + JSON data
    AF-->>Req: DataResponse
    
    Req->>Req: Decode JSON to [User]
    alt Decoding Error
        Req->>Log: Print decode error
        Req-->>App: Throw DecodingError
    end
    
    Req->>Log: Print response data
    Log-->>App: Console output (Status, Data)
    
    Req-->>App: Return [User] array
```

### POST Request with Parameters

```mermaid
sequenceDiagram
    participant App as iOS Application
    participant Params as Params Builder
    participant Req as Requestify
    participant AF as Alamofire
    participant API as API Server

    App->>Params: Params()
    App->>Params: add("name", "John Doe")
    App->>Params: add("email", "john@example.com")
    App->>Params: add("age", 30)
    Params-->>App: Return self (chaining)
    
    App->>Params: build()
    Params-->>App: [String: Any] dictionary
    
    App->>Req: Requestify()
    App->>Req: setURL("https://api.example.com/users")
    App->>Req: setMethod(.post)
    App->>Req: setParameters(params)
    App->>Req: setHeaders(["Authorization": "Bearer token"])
    
    App->>Req: await request(User.self)
    
    Req->>Req: Validate configuration
    Req->>AF: AF.request(url, method: .post, parameters: params, headers: headers)
    
    AF->>AF: Encode parameters to JSON
    AF->>API: HTTP POST /users + JSON body
    
    API->>API: Process request
    API-->>AF: 201 Created + User JSON
    
    AF-->>Req: DataResponse<User>
    Req->>Req: Decode JSON to User
    Req-->>App: Return User object
```

---

## 2. Multipart Upload Flow

### Image Upload with Metadata

```mermaid
sequenceDiagram
    participant App as iOS Application
    participant UI as UIImage
    participant Obj as Encodable Object
    participant Req as Requestify
    participant AF as Alamofire
    participant API as API Server

    App->>UI: UIImage(named: "photo")
    UI-->>App: UIImage instance
    
    App->>Obj: UserProfile(name: "John", bio: "...")
    Obj-->>App: Encodable object
    
    App->>Req: Requestify()
    App->>Req: setURL("https://api.example.com/upload")
    App->>Req: setMethod(.post)
    App->>Req: addImages([image], withName: "profile_pic")
    App->>Req: addObject(profile, withName: "metadata")
    
    App->>Req: await upload(UploadResponse.self)
    
    Req->>Req: Validate configuration
    Req->>Req: Convert object to dictionary
    
    Req->>AF: AF.upload(multipartFormData: { formData in ... })
    
    AF->>AF: Create multipart boundary
    
    loop For each image
        AF->>AF: Append image data with JPEG encoding
    end
    
    AF->>AF: Append JSON metadata
    AF->>AF: Finalize multipart form
    
    AF->>API: HTTP POST /upload + multipart/form-data
    
    API->>API: Process upload
    API->>API: Save files
    API->>API: Store metadata
    
    API-->>AF: 200 OK + UploadResponse JSON
    AF-->>Req: DataResponse<UploadResponse>
    
    Req->>Req: Decode response
    Req-->>App: Return UploadResponse
```

---

## 3. Error Handling Flow

### Complete Error Handling Sequence

```mermaid
sequenceDiagram
    participant App as iOS Application
    participant Req as Requestify
    participant Val as Validator
    participant AF as Alamofire
    participant Dec as JSONDecoder
    participant API as API Server

    App->>Req: await request(User.self)
    
    Req->>Val: Validate URL
    alt URL is nil or empty
        Val-->>Req: Invalid URL
        Req-->>App: Throw RequestBuilderError.missingURL
    end
    
    Req->>AF: Create request
    AF->>API: HTTP Request
    
    alt Network Error
        API-->>AF: No connection / Timeout
        AF-->>Req: AFError.sessionTaskFailed
        Req->>Req: Log error details
        Req-->>App: Throw network error
    end
    
    alt HTTP Error
        API-->>AF: 4xx or 5xx status
        AF-->>Req: Response with error code
        Req->>Req: Log HTTP status
        Req-->>App: Throw HTTP error
    end
    
    API-->>AF: 200 OK + Invalid JSON
    AF-->>Req: Data response
    
    Req->>Dec: Decode data to User.self
    alt Decoding Error
        Dec-->>Req: DecodingError.typeMismatch
        Req->>Req: Log decoding error
        Req-->>App: Throw DecodingError
    end
    
    Dec-->>Req: Decoded User object
    Req-->>App: Return User
```

### Error Recovery Pattern

```mermaid
sequenceDiagram
    participant App as iOS Application
    participant Req as Requestify
    participant Retry as Retry Logic
    participant API as API Server

    App->>Req: await request(User.self)
    
    loop Retry attempts (3 max)
        Req->>API: HTTP Request
        
        alt Success
            API-->>Req: 200 OK + Data
            Req-->>App: Return User
        else Retryable Error (5xx, timeout)
            API-->>Req: Error
            Retry->>Retry: Wait with exponential backoff
            Note over Retry: Delay: 1s, 2s, 4s
        else Non-retryable Error (4xx)
            API-->>Req: 4xx Client Error
            Req-->>App: Throw error immediately
        end
    end
    
    alt Max retries exceeded
        Req-->>App: Throw final error
    end
```

---

## 4. CI/CD Pipeline Flow

### Complete GitHub Actions Workflow

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Git as GitHub
    participant GHA as GitHub Actions
    participant SPM as Swift PM
    participant XCG as XcodeGen
    participant XCB as xcodebuild
    participant Sign as Code Signing
    participant Art as Artifacts

    Dev->>Git: git push origin main
    Git->>GHA: Trigger workflow
    
    GHA->>GHA: Checkout repository
    GHA->>SPM: swift package resolve
    SPM-->>GHA: Dependencies resolved
    
    GHA->>SPM: swift build -v
    SPM->>SPM: Compile sources
    SPM-->>GHA: Build successful
    
    GHA->>SPM: swift test --enable-code-coverage
    SPM->>SPM: Run unit tests
    SPM-->>GHA: Tests passed ✓
    
    GHA->>XCG: brew install xcodegen
    XCG-->>GHA: Installed
    
    GHA->>XCG: xcodegen generate
    XCG->>XCG: Parse project.yml
    XCG->>XCG: Generate .xcodeproj
    XCG-->>GHA: Project created
    
    alt Secrets configured
        GHA->>Sign: Setup code signing
        Sign->>Sign: Create keychain
        Sign->>Sign: Import certificate
        Sign->>Sign: Install provisioning profile
        Sign-->>GHA: Signing ready
        
        GHA->>XCB: xcodebuild archive (signed)
        XCB->>XCB: Build app with signing
        XCB-->>GHA: Archive created
        
        GHA->>XCB: xcodebuild -exportArchive
        XCB->>XCB: Export signed IPA
        XCB-->>GHA: IPA file created
    else No secrets
        GHA->>XCB: xcodebuild archive (unsigned)
        XCB->>XCB: Build app without signing
        XCB-->>GHA: Archive created
        
        GHA->>GHA: Fallback ZIP packaging
        GHA->>GHA: Create Payload/ folder
        GHA->>GHA: Copy .app to Payload/
        GHA->>GHA: ZIP as .ipa
        GHA-->>GHA: Unsigned IPA created
    end
    
    GHA->>Art: Upload IPA artifact
    Art-->>GHA: Upload complete
    
    alt Export failed
        GHA->>Art: Upload export logs
        Art-->>GHA: Logs uploaded
    end
    
    GHA-->>Git: Workflow completed ✓
    Git-->>Dev: Notification
    
    Dev->>Git: Download artifacts
    Git-->>Dev: IPA file
```

---

## 5. Code Signing Flow

### Development Certificate Setup

```mermaid
sequenceDiagram
    participant GHA as GitHub Actions
    participant Sec as GitHub Secrets
    participant KC as Keychain
    participant Cert as Certificate
    participant Prof as Provisioning Profile
    participant XCB as xcodebuild

    GHA->>Sec: Read SIGNING_CERT_BASE64
    Sec-->>GHA: Base64 encoded .p12
    
    GHA->>GHA: base64 --decode > signing.p12
    
    GHA->>Sec: Read PROVISIONING_PROFILE_BASE64
    Sec-->>GHA: Base64 encoded .mobileprovision
    
    GHA->>GHA: base64 --decode > profile.mobileprovision
    
    GHA->>KC: security create-keychain
    KC-->>GHA: Keychain created
    
    GHA->>KC: security default-keychain
    KC-->>GHA: Set as default
    
    GHA->>KC: security unlock-keychain
    KC-->>GHA: Keychain unlocked
    
    GHA->>Cert: security import signing.p12
    Note over GHA,Cert: Password from SIGNING_CERT_PASSWORD
    Cert->>KC: Store certificate
    KC-->>GHA: Certificate imported
    
    GHA->>KC: security set-key-partition-list
    KC-->>GHA: Partition list updated
    
    GHA->>Prof: security cms -D -i profile.mobileprovision
    Prof-->>GHA: UUID extracted
    
    GHA->>Prof: Copy to ~/Library/MobileDevice/Provisioning Profiles/
    Prof-->>GHA: Profile installed
    
    GHA->>GHA: Generate exportOptions.plist
    Note over GHA: Contains: method, teamID, signingStyle, provisioningProfiles
    
    GHA->>XCB: xcodebuild archive
    XCB->>KC: Request signing identity
    KC-->>XCB: Provide certificate
    XCB->>Prof: Request provisioning profile
    Prof-->>XCB: Provide profile
    
    XCB->>XCB: Sign application
    XCB-->>GHA: Signed archive created
    
    GHA->>XCB: xcodebuild -exportArchive
    XCB->>XCB: Verify signing
    XCB->>XCB: Create IPA
    XCB-->>GHA: Signed IPA exported
    
    GHA->>KC: security delete-keychain
    KC-->>GHA: Keychain deleted (cleanup)
```

---

## 6. Parameter Building Flow

### Complex Parameter Construction

```mermaid
sequenceDiagram
    participant App as Application
    participant P1 as Params (User)
    participant P2 as Params (Settings)
    participant Req as Requestify
    participant Enc as Encodable Extension

    Note over App: Building nested parameters
    
    App->>P1: Params()
    App->>P1: add("name", "John")
    App->>P1: add("email", "john@example.com")
    
    App->>P2: Params()
    App->>P2: add("notifications", true)
    App->>P2: add("theme", "dark")
    P2->>P2: build()
    P2-->>App: settingsDict
    
    App->>P1: add("settings", settingsDict)
    App->>P1: add("tags", ["swift", "ios"])
    P1->>P1: build()
    P1-->>App: userParams
    
    App->>Req: setParameters(userParams)
    
    Note over Req: Final parameters structure:
    Note over Req: { "name": "John",
    Note over Req:   "email": "john@example.com",
    Note over Req:   "settings": {
    Note over Req:     "notifications": true,
    Note over Req:     "theme": "dark"
    Note over Req:   },
    Note over Req:   "tags": ["swift", "ios"]
    Note over Req: }
    
    Req->>Enc: Convert to JSON
    Enc-->>Req: JSON data
    Req->>Req: Ready for request
```

---

## 7. Logging Flow

### Request/Response Logging Sequence

```mermaid
sequenceDiagram
    participant App as Application
    participant Req as Requestify
    participant Log as printApiLog
    participant AF as Alamofire
    participant Con as Console

    App->>Req: setPrintLog(true)
    App->>Req: setPrintResponse(true)
    App->>Req: await request(User.self)
    
    Req->>Log: logRequest(url, method, headers, params)
    Log->>Con: Print "🌐 REQUEST:"
    Log->>Con: Print "URL: https://..."
    Log->>Con: Print "Method: POST"
    Log->>Con: Print "Headers: {...}"
    Log->>Con: Print "Parameters: {...}"
    Con-->>App: Console output
    
    Req->>AF: Send request
    AF->>AF: Execute HTTP call
    
    AF-->>Req: Response received
    
    Req->>Log: logResponse(statusCode, data, error)
    Log->>Con: Print "📥 RESPONSE:"
    Log->>Con: Print "Status: 200"
    Log->>Con: Print "Data: {...}"
    Con-->>App: Console output
    
    Req-->>App: Return parsed object
```

---

## Legend

### Sequence Diagram Symbols

- **Solid Arrow (→)**: Synchronous call
- **Dashed Arrow (- -)**: Return value
- **Note box**: Additional information
- **Alt box**: Alternative/conditional flow
- **Loop box**: Repeated operations
- **Participant**: System component or actor

### Common Abbreviations

- **App**: Application (consumer of Requestify)
- **Req**: Requestify library
- **AF**: Alamofire framework
- **API**: Remote API server
- **GHA**: GitHub Actions
- **XCB**: xcodebuild tool
- **SPM**: Swift Package Manager
- **KC**: macOS Keychain

---

**Document Version**: 1.0  
**Last Updated**: November 2025  
**Maintained By**: Requestify Team
