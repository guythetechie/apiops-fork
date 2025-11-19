# DTO Changes in Azure APIM REST API 2025-03-01-preview

## Executive Summary

This document details the differences between Azure API Management REST API version **2024-05-01** (currently implemented) and **2025-03-01-preview** to help prepare for future updates.

**Key Finding:** The 2025-03-01-preview API introduces several new properties across multiple resource types, primarily focused on:
1. **Large Language Model (LLM) support** - New diagnostic settings for AI workloads
2. **Enhanced authentication** - Support for multiple OAuth2/OpenID providers
3. **Application-level authentication** - New OAuth2 application support for products
4. **Backend enhancements** - Carbon-aware load balancing, session affinity, and Azure region support

## New Properties by Resource

### 1. DiagnosticDto

**New Property: `largeLanguageModel`** (LLMDiagnosticSettings)

```csharp
[JsonPropertyName("largeLanguageModel")]
[JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
public LLMDiagnosticSettings? LargeLanguageModel { get; init; }
```

**Purpose:** Enables diagnostic logging specifically for Large Language Model API calls, including request/response message logging with configurable size limits.

**Supporting Types Needed:**
```csharp
public sealed record LLMDiagnosticSettings
{
    [JsonPropertyName("logs")]
    public string? Logs { get; init; } // "enabled" or "disabled"
    
    [JsonPropertyName("requests")]
    public LLMMessageDiagnosticSettings? Requests { get; init; }
    
    [JsonPropertyName("responses")]
    public LLMMessageDiagnosticSettings? Responses { get; init; }
}

public sealed record LLMMessageDiagnosticSettings
{
    [JsonPropertyName("maxSizeInBytes")]
    public int? MaxSizeInBytes { get; init; } // min: 1, max: 262144
    
    [JsonPropertyName("messages")]
    public string? Messages { get; init; } // "all"
}
```

**Impact:** MINOR - Optional property for AI/LLM scenarios. Existing diagnostic configurations remain valid.

---

### 2. BackendDto

#### New Property: `azureRegion` (string)

```csharp
[JsonPropertyName("azureRegion")]
[JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
public string? AzureRegion { get; init; }
```

**Purpose:** Specifies the Azure region where the backend is deployed. Enables carbon-optimized load balancing.

**Example:** `"northeurope"`, `"eastus"`

**Impact:** MINOR - Optional property for advanced load balancing scenarios.

#### Enhanced: `BackendPoolItem` - New Properties

```csharp
// EXISTING property (priority existed, but now documented with carbon emission)
[JsonPropertyName("priority")]
public int? Priority { get; init; } // 0-100

// NEW property
[JsonPropertyName("preferredCarbonEmission")]
[JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
public string? PreferredCarbonEmission { get; init; } // "VeryLow", "Low", "Medium", "High", "VeryHigh"
```

**Purpose:** Enable carbon-aware load balancing by routing traffic to backends with lower carbon emissions.

**Impact:** MINOR - Optional properties for sustainability-focused deployments.

#### Enhanced: `Pool` - New Properties

```csharp
// NEW property
[JsonPropertyName("failureResponse")]
[JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
public BackendFailureResponse? FailureResponse { get; init; }

// NEW property
[JsonPropertyName("sessionAffinity")]
[JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
public BackendSessionAffinity? SessionAffinity { get; init; }
```

**Supporting Types Needed:**
```csharp
public sealed record BackendFailureResponse
{
    [JsonPropertyName("statusCode")]
    public int? StatusCode { get; init; } // 100-599
}

public sealed record BackendSessionAffinity
{
    [JsonPropertyName("sessionId")]
    public BackendSessionId? SessionId { get; init; }
}

public sealed record BackendSessionId
{
    [JsonPropertyName("name")]
    public string? Name { get; init; }
    
    [JsonPropertyName("source")]
    public string? Source { get; init; } // "cookie"
}
```

**Purpose:** 
- `failureResponse`: Define custom HTTP status code when all backends in pool are inactive
- `sessionAffinity`: Enable session stickiness based on cookies

**Impact:** MINOR - Optional properties for advanced backend pool configurations.

#### Enhanced: `CircuitBreakerRule` - New Property

```csharp
// NEW property
[JsonPropertyName("failureResponse")]
[JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
public BackendFailureResponse? FailureResponse { get; init; }
```

**Purpose:** Define custom HTTP response when circuit breaker is open.

**Impact:** MINOR - Optional property for circuit breaker customization.

---

### 3. ProductDto

#### New Property: `application` (Application)

```csharp
[JsonPropertyName("application")]
[JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
public Application? Application { get; init; }
```

**Supporting Types Needed:**
```csharp
public sealed record Application
{
    [JsonPropertyName("entra")]
    public Entra? Entra { get; init; }
}

public sealed record Entra
{
    [JsonPropertyName("applicationId")]
    public string? ApplicationId { get; init; }
    
    [JsonPropertyName("audience")]
    public string? Audience { get; init; }
}
```

**Purpose:** Specifies Microsoft Entra (Azure AD) settings for authorizing product API calls using client application OAuth tokens.

**Impact:** MODERATE - Enables new OAuth2 application-based authentication for products.

#### New Property: `authenticationType` (array)

```csharp
[JsonPropertyName("authenticationType")]
[JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
public ImmutableArray<string>? AuthenticationType { get; init; } // "subscription-key", "application-token"
```

**Purpose:** Specifies supported authentication types for the product. Default is subscription-key only.

**Impact:** MODERATE - Required for application-token authentication scenarios.

---

### 4. ApiDto (AuthenticationSettingsContract)

#### Enhanced: Multiple OAuth2/OpenID Providers

**New Properties:**
```csharp
// NEW property (in addition to existing single oAuth2)
[JsonPropertyName("oAuth2AuthenticationSettings")]
[JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
public ImmutableArray<OAuth2AuthenticationSettingsContract>? OAuth2AuthenticationSettings { get; init; }

// NEW property (in addition to existing single openid)
[JsonPropertyName("openidAuthenticationSettings")]
[JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
public ImmutableArray<OpenIdAuthenticationSettingsContract>? OpenIdAuthenticationSettings { get; init; }
```

**Purpose:** Allows APIs to support multiple OAuth2 authorization servers and OpenID Connect providers simultaneously. The existing single `oAuth2` and `openid` properties are maintained for backward compatibility (they return the first item in the array).

**Example Use Case:**
```json
{
  "authenticationSettings": {
    "oAuth2AuthenticationSettings": [
      {
        "authorizationServerId": "server1",
        "scope": "read write"
      },
      {
        "authorizationServerId": "server2",
        "scope": "admin"
      }
    ]
  }
}
```

**Impact:** MODERATE - Existing single-provider configurations remain valid, but multi-provider scenarios now supported.

---

### 5. SubscriptionDto

**No new properties** - The schema remains the same in 2025-03-01-preview.

---

## Minor Schema Changes

### Property Value Changes
- **No breaking changes** to existing property types or enums
- All existing properties remain backward compatible

### Read-Only Properties
Several properties are documented as read-only (not settable in PUT requests):
- `provisioningState` (ApiDto) - deployment status
- `isOnline` (ApiDto) - gateway accessibility status
- Various date fields in SubscriptionDto

These are response-only properties and correctly excluded from DTOs used for create/update operations.

---

## Summary of Changes by Impact Level

### HIGH IMPACT (Breaking or Major Feature)
- **None** - All changes are additive and backward compatible

### MODERATE IMPACT (New Feature Areas)
1. **Multiple OAuth2/OpenID Providers** (ApiDto)
   - Enables complex authentication scenarios
   - Backward compatible with existing single-provider setup

2. **Application Authentication** (ProductDto)
   - New OAuth2 application-based authentication model
   - Requires new `application` and `authenticationType` properties

### LOW IMPACT (Optional Enhancements)
1. **LLM Diagnostics** (DiagnosticDto)
   - Optional AI/LLM logging capabilities
   - Only relevant for AI Gateway scenarios

2. **Backend Enhancements** (BackendDto)
   - Carbon-aware load balancing
   - Session affinity for backend pools
   - Custom failure responses
   - Azure region attribution

3. **No changes** to:
   - NamedValueDto
   - LoggerDto
   - GatewayDto
   - TagDto
   - GroupDto
   - VersionSetDto
   - ApiReleaseDto
   - ApiSchemaDto
   - PolicyFragmentDto

---

## Recommended Action Plan

### Phase 1: Assessment (Current)
- ✅ Document all differences between 2024-05-01 and 2025-03-01-preview
- ✅ Identify which new features are relevant to use cases

### Phase 2: Preparation
1. **Monitor API version graduation** - Track when 2025-03-01-preview becomes GA
2. **Evaluate feature adoption**:
   - Do you need LLM diagnostic capabilities?
   - Do you need multiple OAuth2 providers?
   - Do you need application-based product authentication?
   - Do you need carbon-aware load balancing?

### Phase 3: Implementation (When needed)
1. **Add new optional properties** to relevant DTOs
2. **Add new supporting types** (LLMDiagnosticSettings, Application, etc.)
3. **Test backward compatibility** - Ensure existing configs work
4. **Update tests** - Add test coverage for new properties
5. **Update documentation** - Document new capabilities

### Phase 4: Migration (Future)
- When 2025-03-01-preview becomes GA (stable)
- When older API versions are deprecated
- When new features become business requirements

---

## Implementation Checklist

If you decide to implement 2025-03-01-preview support, here's the checklist:

### DiagnosticDto
- [ ] Add `LargeLanguageModel` property
- [ ] Add `LLMDiagnosticSettings` record type
- [ ] Add `LLMMessageDiagnosticSettings` record type

### BackendDto / BackendContract
- [ ] Add `AzureRegion` property
- [ ] Add `PreferredCarbonEmission` to `BackendPoolItem`
- [ ] Add `FailureResponse` to `Pool`
- [ ] Add `SessionAffinity` to `Pool`
- [ ] Add `FailureResponse` to `CircuitBreakerRule`
- [ ] Add `BackendFailureResponse` record type
- [ ] Add `BackendSessionAffinity` record type
- [ ] Add `BackendSessionId` record type

### ProductDto / ProductContract
- [ ] Add `Application` property
- [ ] Add `AuthenticationType` property
- [ ] Add `Application` record type
- [ ] Add `Entra` record type

### ApiDto / AuthenticationSettingsContract
- [ ] Add `OAuth2AuthenticationSettings` array property
- [ ] Add `OpenIdAuthenticationSettings` array property

---

## Conclusion

The 2025-03-01-preview API version introduces valuable new capabilities while maintaining full backward compatibility. All changes are **additive** - no existing properties are removed or changed.

**Key takeaways:**
1. ✅ Current DTOs (2024-05-01) remain fully functional
2. ✅ No breaking changes requiring immediate action
3. ✅ New features are optional and can be adopted incrementally
4. ✅ Migration can be planned based on business needs

**Recommendation:** Monitor the API version for GA status and evaluate which new features align with your use cases. Implementation can be deferred until the preview API graduates to stable or until specific features become requirements.

---

## References

- [Diagnostic - Create Or Update (2025-03-01-preview)](https://learn.microsoft.com/en-us/rest/api/apimanagement/diagnostic/create-or-update?view=rest-apimanagement-2025-03-01-preview)
- [Backend - Create Or Update (2025-03-01-preview)](https://learn.microsoft.com/en-us/rest/api/apimanagement/backend/create-or-update?view=rest-apimanagement-2025-03-01-preview)
- [Product - Create Or Update (2025-03-01-preview)](https://learn.microsoft.com/en-us/rest/api/apimanagement/product/create-or-update?view=rest-apimanagement-2025-03-01-preview)
- [Apis - Create Or Update (2025-03-01-preview)](https://learn.microsoft.com/en-us/rest/api/apimanagement/apis/create-or-update?view=rest-apimanagement-2025-03-01-preview)
- [Subscription - Create Or Update (2025-03-01-preview)](https://learn.microsoft.com/en-us/rest/api/apimanagement/subscription/create-or-update?view=rest-apimanagement-2025-03-01-preview)

---

**Document Version:** 1.0  
**Date:** 2025-11-19  
**API Versions Compared:** 2024-05-01 (stable) vs 2025-03-01-preview
