# 🤖 Automatic Multi-Language Translation Implementation Guide

This guide provides the structure, architecture, and method signatures for implementing automatic translation support in BookingWhizz using DeepL API or LibreTranslate.

## 📋 Table of Contents

- [Overview](#overview)
- [Configuration Structure](#configuration-structure)
- [Folder Structure](#folder-structure)
- [Service Architecture](#service-architecture)
- [Translation Field Mapping](#translation-field-mapping)
- [Integration Points](#integration-points)
- [Implementation Flow](#implementation-flow)

---

## Overview

### Requirements

1. **Config-based Provider Toggle**: Switch between DeepL and LibreTranslate via `appsettings.json`
2. **Default Language**: English (MultiLanguageId = 1) - saved to main tables
3. **Other Languages**: Saved to `*_ML` tables (e.g., `Accommodations_ML`, `Rooms_ML`)
4. **Selective Translation**: Only specific fields are translated per module
5. **Property-level Toggle**: Translation only happens if admin enables it for a property

### Tables Structure

| Main Table | ML Table | Purpose |
|------------|----------|---------|
| `Accommodations` | `Accommodations_ML` | Property data |
| `Rooms` | `Rooms_ML` | Room data |
| `RatePlans` | `RatePlans_ML` | Rate plan data |
| `Activities` | `Activities_ML` | Activity data |
| `Policies` | `Policies_ML` | Policy data |
| `Promotions` | `Promotions_ML` | Promotion data |

---

## Configuration Structure

### appsettings.json

```json
{
  "TranslationSettings": {
    "Provider": "deepl",  // "deepl" or "libre"
    "DefaultLanguageId": 1,
    "DefaultLanguageCode": "en",
    "AutoTranslateEnabled": true,
    "EnableForProperty": false  // Property-level toggle (stored in DB)
  },
  "TranslationServices": {
    "DeepL": {
      "ApiKey": "your-deepl-api-key-here",
      "ApiUrl": "https://api-free.deepl.com/v2/translate",
      "UsePro": false,
      "RateLimitDelayMs": 200
    },
    "LibreTranslate": {
      "ApiUrl": "http://localhost:5000/translate",
      "ApiKey": "",  // Optional for self-hosted
      "RateLimitDelayMs": 100
    }
  }
}
```

### TranslationSettings Model

```csharp
// V3BookingEngine/Models/TranslationSettings.cs
namespace V3BookingEngine.Models
{
    public class TranslationSettings
    {
        public string Provider { get; set; } = "deepl"; // "deepl" or "libre"
        public int DefaultLanguageId { get; set; } = 1;
        public string DefaultLanguageCode { get; set; } = "en";
        public bool AutoTranslateEnabled { get; set; } = true;
        public bool EnableForProperty { get; set; } = false;
    }

    public class DeepLSettings
    {
        public string ApiKey { get; set; } = string.Empty;
        public string ApiUrl { get; set; } = "https://api-free.deepl.com/v2/translate";
        public bool UsePro { get; set; } = false;
        public int RateLimitDelayMs { get; set; } = 200;
    }

    public class LibreTranslateSettings
    {
        public string ApiUrl { get; set; } = "http://localhost:5000/translate";
        public string ApiKey { get; set; } = string.Empty;
        public int RateLimitDelayMs { get; set; } = 100;
    }
}
```

---

## Folder Structure

```
V3BookingEngine/
├── Services/
│   └── TranslationService/
│       ├── ITranslationService.cs              # Main translation interface
│       ├── TranslationServiceFactory.cs        # Factory to create provider instances
│       ├── TranslationServiceBase.cs           # Base class with common logic
│       ├── Providers/
│       │   ├── DeepLTranslationProvider.cs     # DeepL implementation
│       │   ├── LibreTranslateProvider.cs       # LibreTranslate implementation
│       │   └── ITranslationProvider.cs         # Provider interface
│       ├── Models/
│       │   ├── TranslationRequest.cs           # Request model
│       │   ├── TranslationResponse.cs          # Response model
│       │   └── TranslationResult.cs            # Result wrapper
│       └── ServiceRegistration.cs              # DI registration
│
├── Services/
│   ├── PropertyService/
│   │   └── PropertyTranslationHelper.cs       # Property-specific translation logic
│   ├── RoomManagementService/
│   │   └── RoomTranslationHelper.cs           # Room-specific translation logic
│   ├── PolicyManagementService/
│   │   └── PolicyTranslationHelper.cs         # Policy-specific translation logic
│   └── PromotionManagementService/
│       └── PromotionTranslationHelper.cs      # Promotion-specific translation logic
│
├── Helpers/
│   └── TranslationHelper.cs                    # Generic translation utilities
│
└── Models/
    └── TranslationSettings.cs                  # Configuration models
```

---

## Service Architecture

### 1. Main Translation Service Interface

```csharp
// V3BookingEngine/Services/TranslationService/ITranslationService.cs
namespace V3BookingEngine.Services.TranslationService
{
    public interface ITranslationService
    {
        /// <summary>
        /// Translates a single text from source language to target language
        /// </summary>
        Task<string> TranslateTextAsync(
            string text, 
            string sourceLanguageCode, 
            string targetLanguageCode);

        /// <summary>
        /// Translates multiple texts in batch (more efficient)
        /// </summary>
        Task<Dictionary<string, string>> TranslateBatchAsync(
            Dictionary<string, string> texts, 
            string sourceLanguageCode, 
            string targetLanguageCode);

        /// <summary>
        /// Checks if translation service is available
        /// </summary>
        Task<bool> IsServiceAvailableAsync();

        /// <summary>
        /// Gets the current provider name
        /// </summary>
        string GetProviderName();
    }
}
```

### 2. Translation Provider Interface

```csharp
// V3BookingEngine/Services/TranslationService/Providers/ITranslationProvider.cs
namespace V3BookingEngine.Services.TranslationService.Providers
{
    public interface ITranslationProvider
    {
        Task<string> TranslateAsync(
            string text, 
            string sourceLanguageCode, 
            string targetLanguageCode);

        Task<Dictionary<string, string>> TranslateBatchAsync(
            Dictionary<string, string> texts, 
            string sourceLanguageCode, 
            string targetLanguageCode);

        Task<bool> IsAvailableAsync();
        string ProviderName { get; }
    }
}
```

### 3. Translation Service Factory

```csharp
// V3BookingEngine/Services/TranslationService/TranslationServiceFactory.cs
namespace V3BookingEngine.Services.TranslationService
{
    public class TranslationServiceFactory
    {
        public static ITranslationProvider CreateProvider(
            string providerName,
            IConfiguration configuration,
            HttpClient httpClient,
            ILogger logger)
        {
            return providerName.ToLower() switch
            {
                "deepl" => new DeepLTranslationProvider(configuration, httpClient, logger),
                "libre" => new LibreTranslateProvider(configuration, httpClient, logger),
                _ => throw new ArgumentException($"Unknown translation provider: {providerName}")
            };
        }
    }
}
```

### 4. DeepL Provider Implementation (Signature)

```csharp
// V3BookingEngine/Services/TranslationService/Providers/DeepLTranslationProvider.cs
namespace V3BookingEngine.Services.TranslationService.Providers
{
    public class DeepLTranslationProvider : ITranslationProvider
    {
        private readonly DeepLSettings _settings;
        private readonly HttpClient _httpClient;
        private readonly ILogger<DeepLTranslationProvider> _logger;

        public string ProviderName => "DeepL";

        public DeepLTranslationProvider(
            IConfiguration configuration,
            HttpClient httpClient,
            ILogger<DeepLTranslationProvider> logger)
        {
            // Initialize from configuration
        }

        public async Task<string> TranslateAsync(
            string text, 
            string sourceLanguageCode, 
            string targetLanguageCode)
        {
            // Implementation: Call DeepL API
            // Return translated text
        }

        public async Task<Dictionary<string, string>> TranslateBatchAsync(
            Dictionary<string, string> texts, 
            string sourceLanguageCode, 
            string targetLanguageCode)
        {
            // Implementation: Batch translate with rate limiting
        }

        public async Task<bool> IsAvailableAsync()
        {
            // Implementation: Health check
        }
    }
}
```

### 5. LibreTranslate Provider Implementation (Signature)

```csharp
// V3BookingEngine/Services/TranslationService/Providers/LibreTranslateProvider.cs
namespace V3BookingEngine.Services.TranslationService.Providers
{
    public class LibreTranslateProvider : ITranslationProvider
    {
        private readonly LibreTranslateSettings _settings;
        private readonly HttpClient _httpClient;
        private readonly ILogger<LibreTranslateProvider> _logger;

        public string ProviderName => "LibreTranslate";

        public LibreTranslateProvider(
            IConfiguration configuration,
            HttpClient httpClient,
            ILogger<LibreTranslateProvider> logger)
        {
            // Initialize from configuration
        }

        public async Task<string> TranslateAsync(
            string text, 
            string sourceLanguageCode, 
            string targetLanguageCode)
        {
            // Implementation: Call LibreTranslate API
        }

        public async Task<Dictionary<string, string>> TranslateBatchAsync(
            Dictionary<string, string> texts, 
            string sourceLanguageCode, 
            string targetLanguageCode)
        {
            // Implementation: Batch translate
        }

        public async Task<bool> IsAvailableAsync()
        {
            // Implementation: Health check
        }
    }
}
```

### 6. Main Translation Service Implementation

```csharp
// V3BookingEngine/Services/TranslationService/TranslationService.cs
namespace V3BookingEngine.Services.TranslationService
{
    public class TranslationService : ITranslationService
    {
        private readonly ITranslationProvider _provider;
        private readonly TranslationSettings _settings;
        private readonly ILogger<TranslationService> _logger;

        public TranslationService(
            IConfiguration configuration,
            TranslationServiceFactory factory,
            HttpClient httpClient,
            ILogger<TranslationService> logger)
        {
            _settings = configuration.GetSection("TranslationSettings")
                .Get<TranslationSettings>() ?? new TranslationSettings();
            
            _provider = factory.CreateProvider(
                _settings.Provider,
                configuration,
                httpClient,
                logger);
            
            _logger = logger;
        }

        public async Task<string> TranslateTextAsync(
            string text, 
            string sourceLanguageCode, 
            string targetLanguageCode)
        {
            // Validate input
            // Call provider
            // Handle errors
            // Return translated text
        }

        public async Task<Dictionary<string, string>> TranslateBatchAsync(
            Dictionary<string, string> texts, 
            string sourceLanguageCode, 
            string targetLanguageCode)
        {
            // Batch translation logic
        }

        public async Task<bool> IsServiceAvailableAsync()
        {
            return await _provider.IsAvailableAsync();
        }

        public string GetProviderName()
        {
            return _provider.ProviderName;
        }
    }
}
```

---

## Translation Field Mapping

### Property Module

#### CreateProperty
- **Field**: `AccommodationName`
- **Source Table**: `Accommodations` (if LanguageId = 1)
- **Target Table**: `Accommodations_ML` (if LanguageId != 1)

#### AdditionalDetail
- **Fields**: 
  - `KeyCollectionComments`
  - `PetsAllowedComments`
- **Source Table**: `Accommodations` or `Accommodations_ML`
- **Target Table**: `Accommodations_ML`

#### PropertyFacilities
- **Fields**:
  - `Property Group` (from lookup table)
  - `Property Facilities` (from lookup table)
- **Source Table**: Facility lookup tables
- **Target Table**: `Facilities_ML` or `PropertyFacilities_ML`

#### CreateAddon
- **Fields**:
  - `AddonName`
  - `ShortDescription`
  - `CancellationPolicy`
  - `GuaranteePolicy`
  - `LongDescription`
- **Source Table**: `Addons` (if LanguageId = 1)
- **Target Table**: `Addons_ML` (if LanguageId != 1)

### Room Module

#### CreateRoom
- **Fields**:
  - `RoomName`
  - `RoomDescription`
- **Source Table**: `Rooms` (if LanguageId = 1)
- **Target Table**: `Rooms_ML` (if LanguageId != 1)

#### RoomFacilities
- **Fields**:
  - `Room Group` (from lookup table)
  - `Room Facilities` (from lookup table)
- **Source Table**: Facility lookup tables
- **Target Table**: `RoomFacilities_ML`

#### CreateRatePlan
- **Fields**:
  - `RatePlanName`
  - `DisplayRatePlanName`
  - `Included`
  - `Highlight`
  - `MealDescription`
- **Source Table**: `RatePlans` (if LanguageId = 1)
- **Target Table**: `RatePlans_ML` (if LanguageId != 1)

### Policy Module

#### CreatePolicy
- **Fields**:
  - `PolicyName`
  - `CancellationPolicyDescription`
  - `BookingPolicyDescription`
  - `NoShowPolicyDescription`
- **Source Table**: `Policies` (if LanguageId = 1)
- **Target Table**: `Policies_ML` (if LanguageId != 1)

### Promotion Module

#### CreatePromotion
- **Fields**:
  - `PromotionName`
  - `Description`
- **Source Table**: `Promotions` (if LanguageId = 1)
- **Target Table**: `Promotions_ML` (if LanguageId != 1)

---

## Integration Points

### 1. Property Translation Helper

```csharp
// V3BookingEngine/Services/PropertyService/PropertyTranslationHelper.cs
namespace V3BookingEngine.Services.PropertyService
{
    public class PropertyTranslationHelper
    {
        private readonly ITranslationService _translationService;
        private readonly ILanguageService _languageService;
        private readonly IConfiguration _configuration;

        public PropertyTranslationHelper(
            ITranslationService translationService,
            ILanguageService languageService,
            IConfiguration configuration)
        {
            // Initialize
        }

        /// <summary>
        /// Translates property fields when creating/updating property
        /// </summary>
        public async Task<CreatePropertyViewModel> TranslatePropertyAsync(
            CreatePropertyViewModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            int accommodationId = 0)
        {
            // Check if translation is enabled for this property
            // Get source and target language codes
            // Translate AccommodationName
            // Return translated model
        }

        /// <summary>
        /// Translates additional detail fields
        /// </summary>
        public async Task<AdditionalDetailsViewModel> TranslateAdditionalDetailsAsync(
            AdditionalDetailsViewModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            int accommodationId)
        {
            // Translate KeyCollectionComments, PetsAllowedComments
        }

        /// <summary>
        /// Translates addon fields
        /// </summary>
        public async Task<AddonViewModel> TranslateAddonAsync(
            AddonViewModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            int addonId = 0)
        {
            // Translate AddonName, ShortDescription, CancellationPolicy, etc.
        }
    }
}
```

### 2. Room Translation Helper

```csharp
// V3BookingEngine/Services/RoomManagementService/RoomTranslationHelper.cs
namespace V3BookingEngine.Services.RoomManagementService
{
    public class RoomTranslationHelper
    {
        private readonly ITranslationService _translationService;
        private readonly ILanguageService _languageService;

        /// <summary>
        /// Translates room fields
        /// </summary>
        public async Task<CreateRoomViewModel> TranslateRoomAsync(
            CreateRoomViewModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            int roomId = 0)
        {
            // Translate RoomName, RoomDescription
        }

        /// <summary>
        /// Translates rate plan fields
        /// </summary>
        public async Task<RatePlanViewModel> TranslateRatePlanAsync(
            RatePlanViewModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            int ratePlanId = 0)
        {
            // Translate RatePlanName, DisplayRatePlanName, Included, Highlight, MealDescription
        }
    }
}
```

### 3. Policy Translation Helper

```csharp
// V3BookingEngine/Services/PolicyManagementService/PolicyTranslationHelper.cs
namespace V3BookingEngine.Services.PolicyManagementService
{
    public class PolicyTranslationHelper
    {
        private readonly ITranslationService _translationService;
        private readonly ILanguageService _languageService;

        /// <summary>
        /// Translates policy fields
        /// </summary>
        public async Task<CreatePolicyViewModel> TranslatePolicyAsync(
            CreatePolicyViewModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            int policyId = 0)
        {
            // Translate PolicyName, CancellationPolicyDescription, etc.
        }
    }
}
```

### 4. Promotion Translation Helper

```csharp
// V3BookingEngine/Services/PromotionManagementService/PromotionTranslationHelper.cs
namespace V3BookingEngine.Services.PromotionManagementService
{
    public class PromotionTranslationHelper
    {
        private readonly ITranslationService _translationService;
        private readonly ILanguageService _languageService;

        /// <summary>
        /// Translates promotion fields
        /// </summary>
        public async Task<CreatePromotionViewModel> TranslatePromotionAsync(
            CreatePromotionViewModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            int promotionId = 0)
        {
            // Translate PromotionName, Description
        }
    }
}
```

### 5. Generic Translation Helper

```csharp
// V3BookingEngine/Helpers/TranslationHelper.cs
namespace V3BookingEngine.Helpers
{
    public static class TranslationHelper
    {
        /// <summary>
        /// Determines if data should be saved to ML table or main table
        /// </summary>
        public static bool ShouldSaveToMLTable(int languageId, int defaultLanguageId = 1)
        {
            return languageId != defaultLanguageId;
        }

        /// <summary>
        /// Checks if translation is enabled for a property
        /// </summary>
        public static async Task<bool> IsTranslationEnabledForPropertyAsync(
            int accommodationId,
            IDbConnection connection)
        {
            // Query Accommodations table for AutoTranslateEnabled flag
            // Return true/false
        }

        /// <summary>
        /// Gets language code from language ID
        /// </summary>
        public static async Task<string> GetLanguageCodeAsync(
            int languageId,
            ILanguageService languageService)
        {
            var language = await languageService.GetLanguageByIdAsync(languageId);
            return language?.LanguageCode ?? "en";
        }
    }
}
```

---

## Implementation Flow

### Flow Diagram

```
Admin Creates/Updates Data
    ↓
Check LanguageId
    ↓
Is LanguageId = 1 (Default/English)?
    ├─ YES → Save to Main Table (Accommodations, Rooms, etc.)
    └─ NO → Check if Translation Enabled for Property
            ↓
        Is Translation Enabled?
            ├─ YES → Translate Fields → Save to ML Table
            └─ NO → Save Empty/Null to ML Table (Manual entry required)
```

### Method Signature Examples

#### Property Management Service Update

```csharp
// V3BookingEngine/Services/PropertyService/PropertyManagementService.cs

// Existing method signature (keep as is)
public async Task<string> CreatePropertyAsync(
    CreatePropertyViewModel model, 
    int ownerId, 
    int languageId)

// NEW: Add translation support
public async Task<string> CreatePropertyWithTranslationAsync(
    CreatePropertyViewModel model,
    int ownerId,
    int sourceLanguageId,
    int targetLanguageId,
    bool autoTranslate = false)
{
    // 1. Save to main table if sourceLanguageId = 1
    // 2. If targetLanguageId != 1 and autoTranslate = true:
    //    - Get source language code
    //    - Get target language code
    //    - Translate AccommodationName using ITranslationService
    //    - Save to Accommodations_ML table
    // 3. Return result
}
```

#### Room Management Service Update

```csharp
// V3BookingEngine/Services/RoomManagementService/RoomManagementService.cs

// NEW: Add translation support
public async Task<string> CreateRoomWithTranslationAsync(
    CreateRoomViewModel model,
    int accommodationId,
    int sourceLanguageId,
    int targetLanguageId,
    bool autoTranslate = false)
{
    // Similar flow as Property
}
```

---

## Service Registration

```csharp
// V3BookingEngine/Services/TranslationService/ServiceRegistration.cs
namespace V3BookingEngine.Services.TranslationService
{
    public static class ServiceRegistration
    {
        public static IServiceCollection AddTranslationServices(
            this IServiceCollection services,
            IConfiguration configuration)
        {
            // Register HttpClient for translation APIs
            services.AddHttpClient<ITranslationService, TranslationService>();

            // Register translation service
            services.AddScoped<ITranslationService, TranslationService>();

            // Register translation helpers
            services.AddScoped<PropertyTranslationHelper>();
            services.AddScoped<RoomTranslationHelper>();
            services.AddScoped<PolicyTranslationHelper>();
            services.AddScoped<PromotionTranslationHelper>();

            return services;
        }
    }
}
```

### Program.cs Registration

```csharp
// Program.cs
using V3BookingEngine.Services.TranslationService;

// Add translation services
builder.Services.AddTranslationServices(builder.Configuration);
```

---

## Database Schema Considerations

### Add Translation Toggle to Accommodations Table

```sql
-- Add column to enable/disable translation per property
ALTER TABLE Accommodations
ADD AutoTranslateEnabled BIT DEFAULT 0;

-- Index for performance
CREATE INDEX IX_Accommodations_AutoTranslateEnabled 
ON Accommodations(AutoTranslateEnabled);
```

### ML Tables Structure (Example: Accommodations_ML)

```sql
CREATE TABLE Accommodations_ML (
    TranslationId INT PRIMARY KEY IDENTITY(1,1),
    AccommodationId INT NOT NULL,
    LanguageId INT NOT NULL,
    AccommodationName NVARCHAR(500),
    -- Add other translatable fields
    CreatedDate DATETIME DEFAULT GETDATE(),
    UpdatedDate DATETIME DEFAULT GETDATE(),
    
    FOREIGN KEY (AccommodationId) REFERENCES Accommodations(AccommodationId),
    FOREIGN KEY (LanguageId) REFERENCES Languages(LanguageId),
    UNIQUE (AccommodationId, LanguageId)
);
```

---

## Error Handling Strategy

### Translation Service Error Handling

```csharp
// If translation fails:
// 1. Log error
// 2. Return original text (fallback)
// 3. Notify admin (optional)
// 4. Continue with save operation (don't block user)
```

### Method Signature for Error Handling

```csharp
public class TranslationResult
{
    public bool Success { get; set; }
    public string TranslatedText { get; set; } = string.Empty;
    public string OriginalText { get; set; } = string.Empty;
    public string ErrorMessage { get; set; } = string.Empty;
    public string ProviderName { get; set; } = string.Empty;
}

public interface ITranslationService
{
    Task<TranslationResult> TranslateTextWithResultAsync(
        string text,
        string sourceLanguageCode,
        string targetLanguageCode);
}
```

---

## Testing Strategy

### Unit Tests Structure

```
Tests/
├── TranslationService.Tests/
│   ├── DeepLTranslationProviderTests.cs
│   ├── LibreTranslateProviderTests.cs
│   └── TranslationServiceTests.cs
├── PropertyTranslationHelper.Tests/
│   └── PropertyTranslationHelperTests.cs
└── Integration.Tests/
    └── TranslationIntegrationTests.cs
```

### Test Method Signatures

```csharp
[Fact]
public async Task TranslateTextAsync_WithValidInput_ReturnsTranslatedText()

[Fact]
public async Task TranslateTextAsync_WithInvalidApiKey_ReturnsOriginalText()

[Fact]
public async Task TranslatePropertyAsync_WithLanguageId1_SavesToMainTable()

[Fact]
public async Task TranslatePropertyAsync_WithLanguageId2_SavesToMLTable()
```

---

## Performance Considerations

1. **Batch Translation**: Use `TranslateBatchAsync` for multiple fields
2. **Caching**: Cache translated content to avoid re-translating
3. **Rate Limiting**: Respect API rate limits (DeepL: 5 req/sec free tier)
4. **Async Operations**: All translation operations should be async
5. **Background Jobs**: Consider queuing translations for large datasets

---

## Security Considerations

1. **API Key Storage**: Store API keys in `appsettings.json` (development) or Azure Key Vault (production)
2. **Input Validation**: Validate text length and content before translation
3. **Error Messages**: Don't expose API keys in error messages
4. **Rate Limiting**: Implement client-side rate limiting to prevent abuse

---

## Next Steps

1. ✅ Create folder structure
2. ✅ Implement `ITranslationService` and providers
3. ✅ Create translation helpers for each module
4. ✅ Update existing service methods to support translation
5. ✅ Add database columns for translation toggle
6. ✅ Update stored procedures to handle ML tables
7. ✅ Add UI toggle for enabling translation per property
8. ✅ Implement error handling and logging
9. ✅ Add unit tests
10. ✅ Performance testing and optimization

---

**Last Updated**: 2024

