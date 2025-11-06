# Translation Service Implementation Plan

## Overview

This document outlines the implementation plan for adding multi-language translation support to the Booking Engine project. The system will support automatic translation using either **DeepL API** or **LibreTranslate**, with the ability to toggle between services via configuration.

---

## Database Schema

### Existing Tables

1. **MULTILanguages**
   - Stores available languages (English, Urdu, Arabic, Russian, Spanish)
   - Contains `LanguageId`, `LanguageCode`, `LanguageName`, etc.
   - `LanguageId = 1` represents English (default language)

2. **Accommodations**
   - Stores base/default property data in English
   - Used when `LanguageId = 1`
   - Contains all property fields (name, description, address, etc.)

3. **Accommodations_ML**
   - Stores translations for languages other than English
   - Used when `LanguageId != 1`
   - Contains only translatable fields (name, description, short description, address, city, etc.)
   - References `AccommodationId` from `Accommodations` table
   - References `LanguageId` from `MULTILanguages` table

### Translation Logic

- **English (LanguageId = 1)**: 
  - All CRUD operations use `Accommodations` table only
  - No translation needed
  
- **Other Languages (LanguageId != 1)**:
  - Base data comes from `Accommodations` table
  - Translatable fields stored/retrieved from `Accommodations_ML` table
  - Translation service will auto-translate missing fields

---

## Architecture Overview

### Service Layer Structure

```
V3BookingEngine/
├── Services/
│   └── TranslationService/
│       ├── ITranslationService.cs          (Interface)
│       ├── TranslationServiceFactory.cs     (Factory for service selection)
│       ├── DeepLTranslationService.cs      (DeepL implementation)
│       ├── LibreTranslateService.cs        (LibreTranslate implementation)
│       └── TranslationServiceBase.cs       (Base class with common logic)
```

### Key Components

1. **ITranslationService Interface**
   - Defines contract for translation operations
   - Methods: `TranslateAsync(string text, string targetLanguageCode)`
   - Ensures both services implement the same interface

2. **TranslationServiceFactory**
   - Reads configuration to determine which service to use
   - Returns appropriate implementation based on config
   - Handles service instantiation and dependency injection

3. **DeepLTranslationService**
   - Implements `ITranslationService`
   - Handles DeepL API communication
   - Manages API keys, rate limiting, error handling

4. **LibreTranslateService**
   - Implements `ITranslationService`
   - Handles LibreTranslate API communication
   - Supports both self-hosted and public instances

5. **TranslationServiceBase** (Optional)
   - Common functionality shared by both services
   - Caching, logging, retry logic
   - Base class for both implementations

---

## Configuration

### appsettings.json Structure

```json
{
  "TranslationService": {
    "Provider": "DeepL",  // Options: "DeepL" or "LibreTranslate"
    "EnableAutoTranslation": true,
    "CacheTranslations": true,
    "DefaultSourceLanguage": "en",
    
    "DeepL": {
      "ApiKey": "your-deepl-api-key-here",
      "ApiUrl": "https://api-free.deepl.com/v2/translate",
      "UsePro": false,
      "RateLimitDelay": 200
    },
    
    "LibreTranslate": {
      "ApiUrl": "http://localhost:5000/translate",
      "ApiKey": "",  // Optional, if API key is required
      "Timeout": 30
    }
  }
}
```

### Configuration Options

| Setting | Description | Default |
|---------|-------------|---------|
| `Provider` | Service to use: "DeepL" or "LibreTranslate" | "DeepL" |
| `EnableAutoTranslation` | Enable/disable automatic translation | `true` |
| `CacheTranslations` | Cache translated content to avoid re-translation | `true` |
| `DefaultSourceLanguage` | Default source language code | "en" |
| `DeepL.ApiKey` | DeepL API key (required if using DeepL) | - |
| `DeepL.ApiUrl` | DeepL API endpoint | Free tier URL |
| `DeepL.UsePro` | Use DeepL Pro API (paid) | `false` |
| `LibreTranslate.ApiUrl` | LibreTranslate instance URL | `http://localhost:5000` |
| `LibreTranslate.ApiKey` | LibreTranslate API key (if required) | Empty |

---

## Integration Points

### 1. Stored Procedures

**Current State:**
- Stored procedures handle CRUD for `Accommodations` table
- Translation logic not yet implemented

**Future Updates Required:**
- Add `@LanguageId` parameter to relevant stored procedures
- Modify SELECT queries to join with `Accommodations_ML` when `LanguageId != 1`
- Modify INSERT/UPDATE queries to save translations to `Accommodations_ML`
- Implement fallback logic: if translation doesn't exist, return English from `Accommodations`

**Example Stored Procedure Pattern:**
```
SP_Accommodations_GetById
  @AccommodationId INT,
  @LanguageId INT = 1
  
  IF @LanguageId = 1
    SELECT * FROM Accommodations WHERE AccommodationId = @AccommodationId
  ELSE
    SELECT 
      a.*,  -- Base fields from Accommodations
      COALESCE(ml.Name, a.Name) AS Name,  -- Translation or fallback
      COALESCE(ml.Description, a.Description) AS Description
    FROM Accommodations a
    LEFT JOIN Accommodations_ML ml 
      ON a.AccommodationId = ml.AccommodationId 
      AND ml.LanguageId = @LanguageId
    WHERE a.AccommodationId = @AccommodationId
```

### 2. Service Layer

**PropertyManagementService Updates:**
- Inject `ITranslationService` via dependency injection
- Add method: `TranslateAccommodationAsync(int accommodationId, int targetLanguageId)`
- Add method: `AutoTranslateMissingFieldsAsync(int accommodationId, int languageId)`
- Check `Accommodations_ML` for existing translations before calling translation service
- Save translated content to `Accommodations_ML` after translation

**Translation Workflow:**
1. User creates/updates property in English (saves to `Accommodations`)
2. User selects target language (e.g., Arabic, Urdu)
3. System checks `Accommodations_ML` for existing translation
4. If missing, calls `ITranslationService.TranslateAsync()` for each translatable field
5. Saves translated content to `Accommodations_ML`
6. Returns translated data to user

### 3. Controller Layer

**PropertyController Updates:**
- Add endpoint: `[HttpPost] TranslateProperty(int accommodationId, int targetLanguageId)`
- Add endpoint: `[HttpPost] AutoTranslateAllLanguages(int accommodationId)`
- Add UI button in admin panel: "Translate to All Languages"
- Add UI button per language: "Auto Translate"

**API Endpoints:**
```
POST /Property/TranslateProperty
  Body: { "accommodationId": 123, "targetLanguageId": 2 }
  Response: { "success": true, "message": "Translation completed" }

POST /Property/AutoTranslateAllLanguages
  Body: { "accommodationId": 123 }
  Response: { "success": true, "translatedLanguages": [2, 3, 4, 5] }
```

### 4. Frontend (Admin Panel)

**UI Components:**
- Language selector dropdown in property create/edit form
- "Auto Translate" button next to each language tab
- "Translate All Languages" button in property list/edit page
- Translation status indicator (showing which languages have translations)
- Manual translation override option (edit translated text directly)

**Translation Status Display:**
- Green checkmark: Translation exists
- Yellow warning: Partial translation (some fields missing)
- Red X: No translation available

---

## Translation Service Methods

### ITranslationService Interface

```csharp
// This interface will be implemented later
public interface ITranslationService
{
    Task<string> TranslateAsync(string text, string targetLanguageCode);
    Task<string> TranslateAsync(string text, string sourceLanguageCode, string targetLanguageCode);
    Task<Dictionary<string, string>> TranslateBatchAsync(Dictionary<string, string> texts, string targetLanguageCode);
    Task<bool> IsServiceAvailableAsync();
    string GetServiceName();
}
```

### Method Details

**TranslateAsync(string text, string targetLanguageCode)**
- Translates text from default language (English) to target language
- Used when source is always English
- Returns translated text

**TranslateAsync(string text, string sourceLanguageCode, string targetLanguageCode)**
- Translates text from source language to target language
- Used for cross-language translation
- Returns translated text

**TranslateBatchAsync(Dictionary<string, string> texts, string targetLanguageCode)**
- Translates multiple texts in one call (if API supports)
- More efficient than multiple single calls
- Returns dictionary with same keys and translated values

**IsServiceAvailableAsync()**
- Checks if translation service is available/accessible
- Used for health checks and error handling
- Returns `true` if service is reachable

**GetServiceName()**
- Returns name of active service ("DeepL" or "LibreTranslate")
- Used for logging and UI display

---

## Translation Workflow

### Scenario 1: Creating New Property

1. Admin creates property in English (LanguageId = 1)
2. Data saved to `Accommodations` table
3. Admin clicks "Translate to All Languages"
4. System:
   - Gets all active languages from `MULTILanguages` (excluding English)
   - For each language:
     - Calls translation service for each translatable field
     - Saves translations to `Accommodations_ML`
   - Shows success message with list of translated languages

### Scenario 2: Editing Existing Property

1. Admin edits property in English
2. Updated data saved to `Accommodations` table
3. System checks if translations exist in `Accommodations_ML`
4. If translations exist:
   - Option A: Mark translations as "outdated" (flag in database)
   - Option B: Auto-retranslate all languages
   - Option C: Show notification "Translations may be outdated"

### Scenario 3: Viewing Property in Non-English Language

1. User/Admin selects language (e.g., Arabic, LanguageId = 2)
2. System:
   - Checks `Accommodations_ML` for translation
   - If exists: Returns translated data
   - If missing: Returns English data from `Accommodations` (fallback)
   - Optionally: Shows "Translation not available" indicator

### Scenario 4: Manual Translation Override

1. Admin views property in Arabic
2. Auto-translated text is displayed
3. Admin edits translated text manually
4. Updated translation saved to `Accommodations_ML`
5. System marks translation as "manually edited" (prevents auto-overwrite)

---

## Service Toggle Mechanism

### Factory Pattern Implementation

**TranslationServiceFactory.cs** (Structure)
- Reads `TranslationService:Provider` from configuration
- Instantiates appropriate service based on config value
- Handles dependency injection for both services
- Provides single point of service selection

**Service Registration (Program.cs)**
```csharp
// This will be implemented later
// Register both services
services.AddHttpClient<DeepLTranslationService>();
services.AddHttpClient<LibreTranslateService>();

// Register factory
services.AddSingleton<ITranslationService>(serviceProvider => 
{
    var config = serviceProvider.GetRequiredService<IConfiguration>();
    var provider = config["TranslationService:Provider"];
    
    return provider switch
    {
        "LibreTranslate" => serviceProvider.GetRequiredService<LibreTranslateService>(),
        "DeepL" => serviceProvider.GetRequiredService<DeepLTranslationService>(),
        _ => serviceProvider.GetRequiredService<DeepLTranslationService>() // Default
    };
});
```

### Switching Between Services

**Method 1: Configuration Change**
- Update `appsettings.json`: Change `Provider` from "DeepL" to "LibreTranslate"
- Restart application
- System automatically uses new service

**Method 2: Runtime Toggle (Future Enhancement)**
- Add admin setting to toggle service
- Store preference in database
- Reload service without restart (advanced)

---

## Translatable Fields

### Accommodations Table Fields (English - Base)

| Field | Type | Translatable? |
|-------|------|---------------|
| AccommodationId | INT | No (Primary Key) |
| AccommodationName | NVARCHAR | ✅ Yes |
| Description | NVARCHAR(MAX) | ✅ Yes |
| ShortDescription | NVARCHAR | ✅ Yes |
| Address | NVARCHAR | ✅ Yes |
| CityName | NVARCHAR | ✅ Yes |
| CountryId | INT | No (Reference) |
| CurrencyId | INT | No (Reference) |
| StarRating | INT | No |
| Latitude | DECIMAL | No |
| Longitude | DECIMAL | No |
| IsActive | BIT | No |

### Accommodations_ML Table Fields (Translations)

| Field | Type | Notes |
|-------|------|-------|
| TranslationId | INT | Primary Key |
| AccommodationId | INT | Foreign Key to Accommodations |
| LanguageId | INT | Foreign Key to MULTILanguages |
| AccommodationName | NVARCHAR(500) | Translated name |
| Description | NVARCHAR(MAX) | Translated description |
| ShortDescription | NVARCHAR(1000) | Translated short description |
| Address | NVARCHAR(500) | Translated address |
| CityName | NVARCHAR(200) | Translated city name |
| IsManuallyEdited | BIT | Flag for manual edits |
| CreatedDate | DATETIME | Translation creation date |
| UpdatedDate | DATETIME | Last update date |

---

## Error Handling & Fallbacks

### Translation Service Errors

**Scenario: API Unavailable**
- Log error to database
- Return original English text
- Show warning to admin: "Translation service unavailable, showing English"

**Scenario: API Rate Limit Exceeded**
- Queue translation requests
- Retry after delay
- Show progress indicator to admin

**Scenario: Invalid Language Code**
- Validate language code before calling API
- Return error message
- Skip translation for that language

### Data Fallback Strategy

1. **Primary**: Use translation from `Accommodations_ML` if exists
2. **Secondary**: Use English text from `Accommodations` table
3. **Tertiary**: Show placeholder text "[Translation not available]"

---

## Performance Considerations

### Caching Strategy

- Cache translated content in `Accommodations_ML` (database cache)
- Avoid re-translating same content
- Check cache before calling translation service

### Batch Translation

- Translate multiple fields in single API call (if supported)
- Reduce number of API requests
- Improve performance and reduce costs

### Lazy Loading

- Translate on-demand (when user selects language)
- Don't translate all languages upfront
- Show "Translate" button for untranslated languages

---

## Testing Strategy

### Unit Tests

- Test `ITranslationService` implementations separately
- Mock API responses
- Test error handling and fallbacks

### Integration Tests

- Test with real API (use test keys)
- Test service toggle mechanism
- Test database save/retrieve operations

### Manual Testing Checklist

- [ ] Toggle between DeepL and LibreTranslate
- [ ] Translate single property to one language
- [ ] Translate property to all languages
- [ ] Edit translated content manually
- [ ] View property in different languages
- [ ] Handle API errors gracefully
- [ ] Verify fallback to English when translation missing

---

## Future Enhancements

1. **Translation Memory**: Store common translations to avoid re-translation
2. **Bulk Translation**: Translate multiple properties at once
3. **Translation Status Dashboard**: Show translation coverage per language
4. **Auto-Update**: Auto-retranslate when English content changes
5. **Quality Check**: Compare translations from different services
6. **Cost Tracking**: Track API usage and costs per service
7. **Multi-Source Translation**: Use DeepL for some languages, LibreTranslate for others

---

## Implementation Phases

### Phase 1: Service Infrastructure
- Create `ITranslationService` interface
- Implement `DeepLTranslationService`
- Implement `LibreTranslateService`
- Create `TranslationServiceFactory`
- Add configuration structure

### Phase 2: Database Integration
- Update stored procedures to handle `LanguageId` parameter
- Add methods to save/retrieve translations from `Accommodations_ML`
- Implement fallback logic

### Phase 3: Service Layer
- Integrate translation service into `PropertyManagementService`
- Add translation methods
- Implement caching logic

### Phase 4: Controller & API
- Add translation endpoints
- Handle translation requests
- Return translated data

### Phase 5: Frontend
- Add translation UI components
- Show translation status
- Add manual translation override

### Phase 6: Testing & Optimization
- Test all scenarios
- Optimize performance
- Add error handling
- Document usage

---

## Notes

- **No Code Implementation Yet**: This document is a planning guide. Actual code implementation will be done in subsequent phases.
- **Translate Function**: The `Translate(string text, string targetLanguageCode)` function will be created as part of `ITranslationService` interface implementation.
- **Service Toggle**: Switching between DeepL and LibreTranslate will be controlled via `appsettings.json` configuration.
- **Backward Compatibility**: Existing functionality (English-only) must continue to work without changes.

---

## Questions & Decisions Needed

1. Should translations be auto-updated when English content changes?
2. Should we allow manual editing of auto-translated content?
3. Should we show translation quality/confidence scores?
4. Should we support translation of user-generated content (reviews, comments)?
5. What is the priority order for languages to translate?

---

**Last Updated**: 2024  
**Status**: Planning Phase  
**Next Step**: Review and approve this plan before implementation

