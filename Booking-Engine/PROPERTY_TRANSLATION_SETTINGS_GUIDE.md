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
  1. Gets the user's `PropertyId` from their account
  2. Queries the `Accommodations` table to get `AutoTranslateEnabled` and `EnableForProperty` values
  3. Adds these values to the user's authentication claims
  4. Claims are available throughout the user's session

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
- [ ] Create stored procedure `SP_Accommodations_GetTranslationSettings`

### Backend Changes
- [ ] Add `AutoTranslateEnabled` and `EnableForProperty` to `CreatePropertyViewModel`
- [ ] Update `CreatePropertyAsync` method to save these values
- [ ] Update `UpdatePropertyAsync` method to save these values
- [ ] Create `GetPropertyTranslationSettingsAsync` method
- [ ] Update `Login` method to get settings and add to claims
- [ ] Update `VerifyOtp` method to get settings and add to claims
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

## Future Enhancements

### Possible Additions
1. **Per-Language Settings**: Enable/disable translation for specific languages
2. **Translation Quality Settings**: Choose between different translation providers
3. **Translation Review Workflow**: Require approval before publishing translations
4. **Translation History**: Track when translations were created/updated
5. **Bulk Settings Update**: Update settings for multiple properties at once

---

## Summary

Property-level translation settings provide granular control over automatic translation functionality. By storing these settings in the database and caching them in user claims, the system can efficiently decide whether to run translation without impacting performance. Each property can independently control its translation behavior, giving property owners the flexibility they need while keeping costs and performance optimized.

---

**Last Updated**: 2024

