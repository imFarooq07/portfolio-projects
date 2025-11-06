# Property-Level Translation Settings Guide

## Overview

This guide explains how property-level translation settings work in the BookingWhizz system. These settings allow each property to control whether automatic translation is enabled and whether multi-language support is active for that specific property.

---

## What Are These Settings?

### 1. AutoTranslateEnabled
- **Purpose**: Controls whether automatic translation should run when translatable fields are saved
- **Default Value**: `true` (enabled by default)
- **When It Matters**: When a user creates or updates property data in a non-default language
- **Behavior**: 
  - If `true`: System will automatically translate translatable fields using DeepL/LibreTranslate API
  - If `false`: System will skip translation, even if translatable fields are changed

### 2. EnableForProperty
- **Purpose**: Master toggle for multi-language support at the property level
- **Default Value**: `false` (disabled by default)
- **When It Matters**: Determines if the property should have multi-language capabilities
- **Behavior**:
  - If `true`: Property can have translations in multiple languages
  - If `false`: Property will only use default language (English)

---

## How It Works

### Step 1: Database Storage
- Both settings are stored in the `Accommodations` table
- Each property has its own values for these settings
- Values are saved when property is created or updated

### Step 2: Login Process
- When a user logs in, the system:
  1. Executes `SP_Admin_Core` stored procedure with `LEFT JOIN` to `Accommodations` table
  2. The query returns user data along with `AutoTranslateEnabled` and `EnableForProperty` from `Accommodations` table
  3. User object is populated with all values (UserId, PropertyId, LanguageId, AutoTranslateEnabled, EnableForProperty, etc.)
  4. All values are added to authentication claims (same way as PropertyId, UserId, etc.)
  5. Claims are available throughout the user's session
  
**Important**: No separate query is needed! Translation settings are fetched in the same login query, just like other user properties.

### Step 3: Using the Settings
- During property creation/update:
  - System reads values from claims (fast, no database query needed)
  - Uses these values to decide whether to run translation
  - Saves the values back to database when property is saved

### Step 4: Translation Decision
- Before translating any field, system checks:
  1. Is `EnableForProperty` = `true`? (If no, skip translation)
  2. Is `AutoTranslateEnabled` = `true`? (If no, skip translation)
  3. Has a translatable field actually changed? (If no, skip translation)
  4. Only if all three are true, translation API is called

---

## Benefits

### 1. Cost Control
- Properties can disable translation to avoid API costs
- Only properties that need translation will use the service

### 2. Performance
- Settings stored in claims = no database query on every request
- Faster decision-making for translation logic

### 3. Flexibility
- Each property can have different settings
- Property owners can enable/disable translation as needed

### 4. Granular Control
- Two-level control:
  - `EnableForProperty`: Master switch
  - `AutoTranslateEnabled`: Fine-grained control

---

## User Experience Flow

### Scenario 1: Property with Translation Enabled
1. User logs in → Settings loaded from database → Added to claims
2. User creates property → `EnableForProperty = true`, `AutoTranslateEnabled = true`
3. User updates property name in Arabic → System detects change → Calls translation API → Saves translated data
4. User views property in Arabic → Sees translated content

### Scenario 2: Property with Translation Disabled
1. User logs in → Settings loaded from database → Added to claims
2. User creates property → `EnableForProperty = false`
3. User updates property name → System skips translation (setting is false)
4. User views property → Sees only default language content

### Scenario 3: Property with Auto-Translate Disabled
1. User logs in → Settings loaded from database → Added to claims
2. User creates property → `EnableForProperty = true`, `AutoTranslateEnabled = false`
3. User updates property name → System skips automatic translation
4. User can manually add translations later if needed

---

## Implementation Checklist

### Database Changes
- [ ] Add `AutoTranslateEnabled` column to `Accommodations` table (BIT, DEFAULT 1)
- [ ] Add `EnableForProperty` column to `Accommodations` table (BIT, DEFAULT 0)
- [ ] Create indexes on both columns for performance
- [ ] Update `SP_Admin_Core` stored procedure to include `LEFT JOIN` with `Accommodations` table in login query

### Backend Changes
- [ ] Add `AutoTranslateEnabled` and `EnableForProperty` to `User` model (`CustomModel/User.cs`)
- [ ] Add `AutoTranslateEnabled` and `EnableForProperty` to `CreatePropertyViewModel`
- [ ] Update `UserService.ValidateUserAsync` to read translation settings from query result
- [ ] Update `UserService.GetUserbyLoginIdAsync` to read translation settings from query result
- [ ] Update `CreatePropertyAsync` method to save these values
- [ ] Update `UpdatePropertyAsync` method to save these values
- [ ] Update `Login` method to add translation settings to claims (from user object)
- [ ] Update `VerifyOtp` method to add translation settings to claims (from user object)
- [ ] Create `TranslationHelper` class with helper methods
- [ ] Update stored procedure `SP_Accommodations_Core` to handle new columns

### Frontend Changes (Optional)
- [ ] Add checkboxes in CreateProperty form for both settings
- [ ] Add checkboxes in UpdateProperty form for both settings
- [ ] Add tooltips/help text explaining what each setting does

### Translation Logic Updates
- [ ] Update translation service to check `EnableForProperty` from claims
- [ ] Update translation service to check `AutoTranslateEnabled` from claims
- [ ] Only call translation API if both are `true` AND field has changed

---

## Default Behavior

### New Properties
- `AutoTranslateEnabled` = `true` (automatic translation enabled)
- `EnableForProperty` = `false` (multi-language disabled by default)

### Existing Properties
- If columns don't exist yet: System will use default values (`true` and `false`)
- After migration: Existing properties will have `AutoTranslateEnabled = 1`, `EnableForProperty = 0`

---

## Best Practices

### 1. When to Enable Translation
- Enable `EnableForProperty` when:
  - Property serves international customers
  - Property wants to support multiple languages
  - Property has content ready for translation

### 2. When to Disable Translation
- Disable `EnableForProperty` when:
  - Property only serves local market
  - Property doesn't need multi-language support
  - Want to reduce API costs

### 3. When to Disable Auto-Translate
- Disable `AutoTranslateEnabled` when:
  - Want manual control over translations
  - Want to review translations before saving
  - Want to use professional translators instead of API

### 4. Performance Tips
- Settings are cached in claims, so no performance impact
- Only check settings when actually needed (during create/update)
- Use indexes on database columns for faster queries

---

## Troubleshooting

### Issue: Translation not working even when enabled
**Check:**
1. Is `EnableForProperty` = `true` in database?
2. Is `AutoTranslateEnabled` = `true` in database?
3. Are values correctly added to claims after login?
4. Is translation service checking both settings?

### Issue: Translation running when it shouldn't
**Check:**
1. Are settings correctly read from claims?
2. Is the logic checking both settings before translating?
3. Are default values correct?

### Issue: Settings not saving
**Check:**
1. Are columns added to database?
2. Is stored procedure updated to handle new columns?
3. Are service methods passing the parameters correctly?

---

## Security Considerations

### 1. Claims Storage
- Settings are stored in authentication claims
- Claims are encrypted in cookies
- Only logged-in users can access their property's settings

### 2. Authorization
- Users can only modify settings for their own property
- PropertyId is validated before updating settings
- Admin users may have additional permissions

### 3. Data Integrity
- Settings are validated before saving
- Default values are used if invalid data is provided
- Database constraints ensure data consistency

---

## Per-Language Settings

### Overview

Per-Language Settings allow property administrators to enable or disable translation for specific languages. This provides granular control over which languages should have automatic translation enabled.

### Database Schema

```sql
-- Create PerLanguageSettings table
CREATE TABLE PerLanguageSettings (
    Id INT PRIMARY KEY IDENTITY(1,1),
    AccommodationId INT NOT NULL,
    LanguageId INT NOT NULL,
    AutoTranslateEnabled BIT DEFAULT 1,
    IsActive BIT DEFAULT 1,
    CreatedDate DATETIME DEFAULT GETDATE(),
    UpdatedDate DATETIME DEFAULT GETDATE(),
    FOREIGN KEY (AccommodationId) REFERENCES Accommodations(AccommodationId),
    FOREIGN KEY (LanguageId) REFERENCES MultiLanguages(MultiLanguageId),
    UNIQUE (AccommodationId, LanguageId)
);
```

### Use Cases

1. **Selective Language Translation**: Enable translation for Urdu and Arabic, but disable for French
2. **Cost Control**: Only translate to languages that generate revenue
3. **Quality Control**: Disable automatic translation for languages that need manual review
4. **Gradual Rollout**: Enable translation for one language at a time

### Implementation Flow

```
User Updates Property in Language (e.g., Urdu - LanguageId = 2)
    ↓
Check PerLanguageSettings table:
    - AccommodationId = 11481
    - LanguageId = 2
    - AutoTranslateEnabled = true?
    ↓
If true → Run translation
If false → Skip translation (save original text)
    ↓
If record doesn't exist → Use property-level AutoTranslateEnabled setting
```

### UI Considerations

- Add language-specific toggles in property settings page
- Show enabled/disabled status for each language
- Allow bulk enable/disable for multiple languages
- Display cost estimates based on enabled languages

---

## Translation History

### Overview

Translation History tracks when translations were created, updated, and by which method (automatic API or manual entry). This provides audit trail and helps with debugging translation issues.

### Database Schema

```sql
-- Create TranslationHistory table
CREATE TABLE TranslationHistory (
    Id BIGINT PRIMARY KEY IDENTITY(1,1),
    AccommodationId INT NOT NULL,
    TableName VARCHAR(100) NOT NULL,  -- e.g., 'Accommodations_ML', 'Rooms_ML'
    RecordId INT NOT NULL,            -- Primary key of the translated record
    LanguageId INT NOT NULL,
    FieldName VARCHAR(100) NOT NULL,   -- e.g., 'AccommodationName', 'RoomName'
    OriginalText NVARCHAR(MAX),        -- Original English text
    TranslatedText NVARCHAR(MAX),     -- Translated text
    TranslationMethod VARCHAR(50),     -- 'API_AUTO', 'MANUAL', 'BULK_IMPORT'
    TranslationProvider VARCHAR(50),  -- 'DeepL', 'LibreTranslate', 'Google', 'Manual'
    ApiCallId VARCHAR(100),            -- For tracking API calls
    TranslatedBy INT,                  -- UserId who triggered translation
    TranslationDate DATETIME DEFAULT GETDATE(),
    IsActive BIT DEFAULT 1,
    FOREIGN KEY (AccommodationId) REFERENCES Accommodations(AccommodationId),
    FOREIGN KEY (LanguageId) REFERENCES MultiLanguages(MultiLanguageId),
    FOREIGN KEY (TranslatedBy) REFERENCES Users(UserId)
);
```

### Use Cases

1. **Audit Trail**: Track who translated what and when
2. **Debugging**: Identify translation quality issues
3. **Cost Tracking**: Monitor API usage per property/language
4. **Revert Changes**: Restore previous translations
5. **Analytics**: Analyze translation patterns and usage

### Implementation Flow

```
Translation Occurs (API or Manual)
    ↓
Save to *_ML table (e.g., Accommodations_ML)
    ↓
Log to TranslationHistory table:
    - TableName: 'Accommodations_ML'
    - RecordId: 12345
    - FieldName: 'AccommodationName'
    - OriginalText: 'Grand Hotel'
    - TranslatedText: 'گرانڈ ہوٹل'
    - TranslationMethod: 'API_AUTO'
    - TranslationProvider: 'DeepL'
    - TranslatedBy: 101
    - TranslationDate: GETDATE()
    ↓
History available for viewing/auditing
```

### UI Features

1. **Translation History Page**: 
   - Filter by property, language, date range
   - Show original vs translated text
   - Display translation method and provider
   - Show who translated and when

2. **Revert Functionality**:
   - View previous translations
   - Restore to a previous version
   - Compare translations side-by-side

3. **Analytics Dashboard**:
   - Total translations per property
   - API usage statistics
   - Translation quality metrics
   - Cost per language/property

### Integration Points

- **Translation Service**: Log after successful API translation
- **Manual Translation**: Log when admin manually edits translation
- **Bulk Import**: Log when translations are imported from file
- **Update Operations**: Log when existing translation is updated

---

## Future Enhancements

### Possible Additions
1. **Translation Quality Settings**: Choose between different translation providers
2. **Translation Review Workflow**: Require approval before publishing translations
3. **Bulk Settings Update**: Update settings for multiple properties at once
4. **Translation Analytics**: Advanced reporting and insights
5. **Multi-Provider Support**: Use different providers for different languages

---

## Summary

Property-level translation settings provide granular control over automatic translation functionality. By storing these settings in the database and caching them in user claims, the system can efficiently decide whether to run translation without impacting performance. Each property can independently control its translation behavior, giving property owners the flexibility they need while keeping costs and performance optimized.

---

**Last Updated**: 2024


