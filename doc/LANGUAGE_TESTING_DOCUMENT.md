# 🌐 Language & Translation Testing Document

**Complete Testing Guide for Multi-Language Translation System**

This document provides comprehensive test cases and scenarios for testing the language selection, translation functionality, master switch, per-language settings, and all related features.

**Version**: 3.0  
**Last Updated**: 2025  
**Status**: Production Ready ✅

---

## 📋 Table of Contents

1. [Pre-Testing Setup](#pre-testing-setup)
2. [Language Selection Testing](#language-selection-testing)
3. [Master Switch Testing](#master-switch-testing)
4. [Per-Language Settings Testing](#per-language-settings-testing)
5. [Translation Functionality Testing](#translation-functionality-testing)
6. [Translation History Testing](#translation-history-testing)
7. [Translation Analytics Testing](#translation-analytics-testing)
8. [Edge Cases & Error Scenarios](#edge-cases--error-scenarios)
9. [Performance Testing](#performance-testing)
10. [Regression Testing Checklist](#regression-testing-checklist)

---

## Pre-Testing Setup

### Prerequisites

1. **Database Setup**:
   - ✅ `MultiLanguages` table has active languages (English, Arabic, Turkish, etc.)
   - ✅ `Accommodations` table has test properties
   - ✅ `PerLanguageSettings` table is accessible
   - ✅ `TranslationHistory` table is accessible

2. **User Setup**:
   - ✅ Test user with property access
   - ✅ User has `AutoTranslateEnabled` claim set
   - ✅ User can access translation management pages

3. **API Setup**:
   - ✅ DeepL API key is configured and valid
   - ✅ API quota is available for testing

4. **Browser Setup**:
   - ✅ Clear browser cookies before testing
   - ✅ Use incognito/private mode for clean testing
   - ✅ Enable browser console for error checking

### Test Data Requirements

- **Test Property**: At least 1 property with `AutoTranslateEnabled = true`
- **Test Languages**: At least 3 languages (English, Arabic, Turkish)
- **Test Content**: Property with translatable fields (name, description, etc.)

---

## Language Selection Testing

### Test Case 1: Language Dropdown Display

**Objective**: Verify language dropdown appears and displays all active languages

**Steps**:
1. Navigate to any page (e.g., Dashboard)
2. Locate language dropdown in header
3. Click on language dropdown

**Expected Results**:
- ✅ Dropdown opens and displays all active languages from database
- ✅ Each language shows flag icon (🇬🇧, 🇸🇦, 🇹🇷, etc.)
- ✅ Language name is displayed correctly
- ✅ Current selected language is highlighted/checked
- ✅ Dropdown is searchable (search input appears)

**Test Data**: 
- Languages: English (en), Arabic (ar), Turkish (tr)

---

### Test Case 2: Language Search Functionality

**Objective**: Verify search functionality in language dropdown

**Steps**:
1. Open language dropdown
2. Type "ara" in search box
3. Verify filtered results
4. Clear search and verify all languages appear

**Expected Results**:
- ✅ Search input is visible and functional
- ✅ Typing filters languages in real-time
- ✅ Only matching languages are shown
- ✅ Search is case-insensitive
- ✅ Clearing search shows all languages
- ✅ Search box auto-focuses when dropdown opens

---

### Test Case 3: Language Selection & Cookie Persistence

**Objective**: Verify language selection is saved in cookies and persists

**Steps**:
1. Select a language (e.g., Arabic) from dropdown
2. Verify page reloads or updates
3. Navigate to another page
4. Verify selected language is still active
5. Close browser and reopen
6. Verify language selection persists

**Expected Results**:
- ✅ Language changes immediately after selection
- ✅ Cookie `SelectedLanguageId` is set correctly
- ✅ Cookie `SelectedLanguageName` is set correctly
- ✅ Cookie `SelectedLanguageCode` is set correctly
- ✅ Language persists across page navigation
- ✅ Language persists after browser restart
- ✅ Selected language flag appears in header

---

### Test Case 4: Language Claim Update

**Objective**: Verify LanguageId claim is updated via middleware

**Steps**:
1. Select a language (e.g., Turkish)
2. Check browser cookies (should have `SelectedLanguageId`)
3. Make any API call or page request
4. Verify LanguageId claim is updated in backend

**Expected Results**:
- ✅ Cookie is set correctly
- ✅ Middleware reads cookie on each request
- ✅ LanguageId claim is updated in user claims
- ✅ Controllers can access updated LanguageId from claims
- ✅ Data is fetched in selected language

**How to Verify**:
- Check network tab for cookie in requests
- Check backend logs for claim updates
- Verify data displayed matches selected language

---

## Master Switch Testing

### Test Case 5: Master Switch Display

**Objective**: Verify master switch appears on UpdateProperty page

**Steps**:
1. Navigate to `/Property/UpdateProperty`
2. Locate "Auto Translate Enabled" checkbox/switch
3. Verify current state is displayed correctly

**Expected Results**:
- ✅ Master switch field is visible
- ✅ Current value from database is displayed
- ✅ Switch/checkbox is functional
- ✅ Label is clear and descriptive

---

### Test Case 6: Enable Master Switch

**Objective**: Verify enabling master switch works correctly

**Steps**:
1. Navigate to UpdateProperty page
2. Check "Auto Translate Enabled" checkbox
3. Save property
4. Verify database update
5. Verify auto-initialization of PerLanguageSettings

**Expected Results**:
- ✅ Checkbox can be checked
- ✅ Form saves successfully
- ✅ `Accommodations.AutoTranslateEnabled` is set to `true` in database
- ✅ `PerLanguageSettings` entries are created for all active languages
- ✅ All per-language settings default to `AutoTranslateEnabled = true`
- ✅ Success message is displayed

**Database Verification**:
```sql
-- Check Accommodations table
SELECT AccommodationId, AutoTranslateEnabled 
FROM Accommodations 
WHERE AccommodationId = @TestPropertyId;

-- Check PerLanguageSettings table
SELECT AccommodationId, LanguageId, AutoTranslateEnabled, IsActive
FROM PerLanguageSettings
WHERE AccommodationId = @TestPropertyId;
```

---

### Test Case 7: Disable Master Switch

**Objective**: Verify disabling master switch works correctly

**Steps**:
1. Navigate to UpdateProperty page (with master switch enabled)
2. Uncheck "Auto Translate Enabled" checkbox
3. Save property
4. Verify database update
5. Verify translation does not run

**Expected Results**:
- ✅ Checkbox can be unchecked
- ✅ Form saves successfully
- ✅ `Accommodations.AutoTranslateEnabled` is set to `false` in database
- ✅ Translation does not run on next update
- ✅ Success message is displayed

---

### Test Case 8: Master Switch on AllPropertiesLanguageSettings Page

**Objective**: Verify master switch toggle on consolidated page

**Steps**:
1. Navigate to `/Property/AllPropertiesLanguageSettings`
2. Locate property card
3. Find "Master Switch (Auto Translate Enabled)" toggle
4. Toggle master switch ON
5. Verify AJAX call succeeds
6. Verify database update
7. Toggle master switch OFF
8. Verify AJAX call succeeds

**Expected Results**:
- ✅ Master switch toggle is visible in property card
- ✅ Toggle is functional (can switch ON/OFF)
- ✅ AJAX call to `/Property/UpdatePropertyAutoTranslateEnabled` succeeds
- ✅ Database is updated correctly
- ✅ Success toast notification appears
- ✅ UI updates immediately (no page reload needed)
- ✅ Auto-initialization happens when enabled

---

## Per-Language Settings Testing

### Test Case 9: Per-Language Settings Display

**Objective**: Verify per-language toggles appear on AllPropertiesLanguageSettings page

**Steps**:
1. Navigate to `/Property/AllPropertiesLanguageSettings`
2. Locate a property card
3. Verify language toggles are displayed

**Expected Results**:
- ✅ Property card shows all active languages
- ✅ Each language has a toggle switch
- ✅ Language name and flag icon are displayed
- ✅ Current state (enabled/disabled) is shown correctly
- ✅ Toggles are functional

---

### Test Case 10: Enable/Disable Per-Language Setting

**Objective**: Verify per-language toggle works correctly

**Steps**:
1. Navigate to `/Property/AllPropertiesLanguageSettings`
2. Find a property with master switch enabled
3. Toggle a specific language (e.g., Arabic) OFF
4. Verify AJAX call succeeds
5. Verify database update
6. Toggle the same language ON
7. Verify update

**Expected Results**:
- ✅ Toggle switches ON/OFF correctly
- ✅ AJAX call to `/Property/UpdatePropertyLanguageSetting` succeeds
- ✅ `PerLanguageSettings` table is updated correctly
- ✅ Success toast notification appears
- ✅ UI updates immediately
- ✅ Translation respects per-language setting

**Database Verification**:
```sql
SELECT AccommodationId, LanguageId, AutoTranslateEnabled, IsActive
FROM PerLanguageSettings
WHERE AccommodationId = @TestPropertyId AND LanguageId = @TestLanguageId;
```

---

### Test Case 11: Fallback Logic Testing

**Objective**: Verify fallback to master switch when per-language setting doesn't exist

**Steps**:
1. Create a new property (or use one without PerLanguageSettings)
2. Enable master switch (`AutoTranslateEnabled = true`)
3. Do NOT initialize PerLanguageSettings
4. Update a translatable field (e.g., property name)
5. Verify translation runs for all active languages

**Expected Results**:
- ✅ Translation runs even without PerLanguageSettings entries
- ✅ System falls back to master switch value
- ✅ All active languages are translated (if master switch is ON)
- ✅ No errors occur

**How to Verify**:
- Check `TranslationHistory` table for translation entries
- Verify ML tables have translated content
- Check logs for fallback logic execution

---

### Test Case 12: Auto-Initialization Testing

**Objective**: Verify auto-initialization when master switch is enabled

**Steps**:
1. Navigate to UpdateProperty page
2. Enable master switch (`AutoTranslateEnabled = true`)
3. Save property
4. Verify PerLanguageSettings are created automatically

**Expected Results**:
- ✅ `PerLanguageSettings` entries are created for all active languages
- ✅ All entries have `AutoTranslateEnabled = true` (default)
- ✅ All entries have `IsActive = 1`
- ✅ No duplicate entries are created
- ✅ Initialization happens automatically (no manual action needed)

**Database Verification**:
```sql
-- Should return count equal to number of active languages
SELECT COUNT(*) 
FROM PerLanguageSettings
WHERE AccommodationId = @TestPropertyId AND IsActive = 1;

-- Should match active languages count
SELECT COUNT(*) 
FROM MultiLanguages
WHERE StatusId = 1;
```

---

### Test Case 13: Initialize All Languages (Default Enabled)

**Objective**: Verify bulk initialization with default enabled

**Steps**:
1. Navigate to `/Property/AllPropertiesLanguageSettings`
2. Locate "Initialize Section for Current Property"
3. Click "Initialize All Languages (Default: Enabled)"
4. Verify initialization

**Expected Results**:
- ✅ Button is visible (only if user has a property)
- ✅ Clicking button initializes all active languages
- ✅ All languages are set to `AutoTranslateEnabled = true`
- ✅ Success message is displayed
- ✅ Language toggles appear after initialization

---

### Test Case 14: Initialize All Languages (Default Disabled)

**Objective**: Verify bulk initialization with default disabled

**Steps**:
1. Navigate to `/Property/AllPropertiesLanguageSettings`
2. Click "Initialize All Languages (Default: Disabled)"
3. Verify initialization

**Expected Results**:
- ✅ Button is visible and functional
- ✅ All active languages are initialized
- ✅ All languages are set to `AutoTranslateEnabled = false`
- ✅ Success message is displayed
- ✅ Language toggles appear with disabled state

---

## Translation Functionality Testing

### Test Case 15: Property Translation on Create

**Objective**: Verify translation runs when creating property in non-English language

**Steps**:
1. Select Arabic language from dropdown
2. Navigate to `/Property/CreateProperty`
3. Enter property name in English
4. Submit form
5. Verify translation runs

**Expected Results**:
- ✅ Property is created successfully
- ✅ Property name is translated to Arabic
- ✅ Translation is saved to `Accommodations_ML` table
- ✅ Translation history entry is created
- ✅ Original English text is preserved in main table

---

### Test Case 16: Property Translation on Update

**Objective**: Verify translation runs when updating translatable field

**Steps**:
1. Navigate to `/Property/UpdateProperty`
2. Change property name (translatable field)
3. Save property
4. Verify translation runs

**Expected Results**:
- ✅ Property is updated successfully
- ✅ Field change is detected
- ✅ Translation runs for all enabled languages
- ✅ ML tables are updated with translations
- ✅ Translation history entries are created
- ✅ Original language content is preserved

**Prerequisites**:
- Master switch must be enabled
- At least one language must be enabled in PerLanguageSettings
- Old model must be available for comparison

---

### Test Case 17: Translation Skipped When No Field Change

**Objective**: Verify translation does not run when no translatable fields change

**Steps**:
1. Navigate to `/Property/UpdateProperty`
2. Change a non-translatable field (e.g., email, phone)
3. Save property
4. Verify translation does NOT run

**Expected Results**:
- ✅ Property is updated successfully
- ✅ Translation does NOT run (field change detection works)
- ✅ No translation history entries are created
- ✅ Logs show "Translation skipped - no translatable fields changed"

---

### Test Case 18: Translation Skipped When Master Switch Disabled

**Objective**: Verify translation does not run when master switch is disabled

**Steps**:
1. Disable master switch for a property
2. Navigate to `/Property/UpdateProperty`
3. Change property name (translatable field)
4. Save property
5. Verify translation does NOT run

**Expected Results**:
- ✅ Property is updated successfully
- ✅ Translation does NOT run
- ✅ No translation history entries are created
- ✅ Logs show "Translation skipped - AutoTranslateEnabled: false"

---

### Test Case 19: Translation Skipped for Disabled Language

**Objective**: Verify translation does not run for disabled languages

**Steps**:
1. Enable master switch
2. Disable Arabic language in PerLanguageSettings
3. Update property name
4. Verify translation runs for enabled languages only

**Expected Results**:
- ✅ Translation runs for enabled languages (e.g., Turkish)
- ✅ Translation does NOT run for disabled languages (e.g., Arabic)
- ✅ Only enabled languages have entries in ML tables
- ✅ Translation history shows only enabled languages

---

### Test Case 20: Parallel Translation Processing

**Objective**: Verify multiple languages are translated in parallel

**Steps**:
1. Enable master switch
2. Enable 5+ languages in PerLanguageSettings
3. Update property name
4. Monitor translation process

**Expected Results**:
- ✅ All enabled languages are translated
- ✅ Translations happen in parallel (Task.WhenAll)
- ✅ Translation completes faster than sequential processing
- ✅ All ML tables are updated correctly
- ✅ Translation history has entries for all languages

**Performance Check**:
- Sequential: ~5 seconds per language × 5 languages = 25 seconds
- Parallel: ~5 seconds total (all languages together)

---

### Test Case 21: English Always Included in Translation

**Objective**: Verify English is always included when current language is not English

**Steps**:
1. Select Arabic language
2. Update property name
3. Verify English is also translated

**Expected Results**:
- ✅ Current language (Arabic) is translated
- ✅ English (LanguageId = 1) is also translated
- ✅ Both languages have entries in ML tables
- ✅ Logs show "Added English (LanguageId 1) to translation targets"

---

### Test Case 22: Room Translation

**Objective**: Verify room translation works correctly

**Steps**:
1. Navigate to `/RoomManagement/CreateRoom`
2. Enter room name and description
3. Save room
4. Verify translation runs

**Expected Results**:
- ✅ Room is created successfully
- ✅ Translation runs for all enabled languages
- ✅ `Rooms_ML` table has translated entries
- ✅ Translation history entries are created

---

### Test Case 23: Rate Plan Translation

**Objective**: Verify rate plan translation works correctly

**Steps**:
1. Navigate to `/RoomManagement/CreateRatePlan`
2. Enter rate plan name, included, highlight, meal description
3. Save rate plan
4. Verify translation runs

**Expected Results**:
- ✅ Rate plan is created successfully
- ✅ All translatable fields are translated
- ✅ `RatePlans_ML` table has translated entries
- ✅ Translation history entries are created

---

### Test Case 24: Promotion Translation

**Objective**: Verify promotion translation works correctly

**Steps**:
1. Navigate to `/PromotionManagement/CreatePromotion`
2. Enter promotion name and description
3. Save promotion
4. Verify translation runs

**Expected Results**:
- ✅ Promotion is created successfully
- ✅ Promotion name and description are translated
- ✅ `Promotions_ML` table has translated entries
- ✅ Translation history entries are created

---

### Test Case 25: Addon Translation

**Objective**: Verify addon translation works correctly

**Steps**:
1. Navigate to `/Property/CreateAddon`
2. Enter addon name, descriptions, policies
3. Save addon
4. Verify translation runs

**Expected Results**:
- ✅ Addon is created successfully
- ✅ All translatable fields are translated
- ✅ `Activities_ML` table has translated entries
- ✅ Translation history entries are created

---

## Translation History Testing

### Test Case 26: Translation History Display

**Objective**: Verify translation history page displays correctly

**Steps**:
1. Navigate to `/Property/TranslationHistory`
2. Verify history table is displayed
3. Verify columns are correct

**Expected Results**:
- ✅ Page loads successfully
- ✅ Translation history table is displayed
- ✅ Columns: Date & Time, Language, Field Name, Original Text, Translated Text, Method, Provider, Actions
- ✅ Data is paginated (if many records)
- ✅ Empty state is handled (if no records)

---

### Test Case 27: Translation History Filtering

**Objective**: Verify filtering works correctly

**Steps**:
1. Navigate to `/Property/TranslationHistory`
2. Select a language from filter dropdown
3. Select a field name from filter dropdown
4. Set date range (start date and end date)
5. Apply filters
6. Verify filtered results

**Expected Results**:
- ✅ Language filter works correctly
- ✅ Field name filter works correctly
- ✅ Date range filter works correctly
- ✅ Filtered results match criteria
- ✅ Clear filters button resets all filters

---

### Test Case 28: Translation History Search

**Objective**: Verify search functionality works

**Steps**:
1. Navigate to `/Property/TranslationHistory`
2. Enter search text in search box
3. Verify search results
4. Clear search

**Expected Results**:
- ✅ Search box is functional
- ✅ Search searches in original text, translated text, and field names
- ✅ Search is case-insensitive
- ✅ Results update in real-time
- ✅ Clear search shows all records

---

### Test Case 29: Translation History Pagination

**Objective**: Verify pagination works correctly

**Steps**:
1. Navigate to `/Property/TranslationHistory`
2. Verify pagination controls are visible
3. Navigate to next page
4. Navigate to previous page
5. Change page size

**Expected Results**:
- ✅ Pagination controls are visible
- ✅ Next/Previous buttons work correctly
- ✅ Page numbers are clickable
- ✅ Page size can be changed
- ✅ Current page is highlighted
- ✅ Total records count is displayed

---

### Test Case 30: Translation History Export (CSV)

**Objective**: Verify CSV export works correctly

**Steps**:
1. Navigate to `/Property/TranslationHistory`
2. Apply filters (optional)
3. Click "Export" dropdown
4. Select "Export as CSV"
5. Verify file download

**Expected Results**:
- ✅ CSV file downloads successfully
- ✅ File contains all filtered records
- ✅ Headers are correct
- ✅ Data is formatted correctly
- ✅ File can be opened in Excel

---

### Test Case 31: Translation History Export (Excel)

**Objective**: Verify Excel export works correctly

**Steps**:
1. Navigate to `/Property/TranslationHistory`
2. Apply filters (optional)
3. Click "Export" dropdown
4. Select "Export as Excel"
5. Verify file download

**Expected Results**:
- ✅ Excel file downloads successfully
- ✅ File contains all filtered records
- ✅ Headers are correct
- ✅ Data is formatted correctly
- ✅ File can be opened in Excel

---

### Test Case 32: Revert Translation

**Objective**: Verify revert functionality works correctly

**Steps**:
1. Navigate to `/Property/TranslationHistory`
2. Find a translation record
3. Click "Revert" button
4. Confirm revert action
5. Verify revert succeeds

**Expected Results**:
- ✅ Revert button is visible for supported records
- ✅ Confirmation dialog appears
- ✅ Revert action succeeds
- ✅ Original text is restored to ML table
- ✅ New history entry is created for revert action
- ✅ Success message is displayed
- ✅ Page reloads with updated data

**Database Verification**:
```sql
-- Check ML table has original text
SELECT * FROM Accommodations_ML 
WHERE AccommodationId = @TestPropertyId AND LanguageId = @TestLanguageId;

-- Check history has revert entry
SELECT * FROM TranslationHistory 
WHERE AccommodationId = @TestPropertyId 
ORDER BY TranslationDate DESC;
```

---

## Translation Analytics Testing

### Test Case 33: Translation Analytics Display

**Objective**: Verify analytics dashboard displays correctly

**Steps**:
1. Navigate to `/Property/TranslationAnalytics`
2. Verify dashboard loads
3. Verify all charts and cards are displayed

**Expected Results**:
- ✅ Page loads successfully
- ✅ Summary cards are displayed (Total Translations, Auto Translations, Manual Translations, API Calls)
- ✅ Language chart (Doughnut) is displayed
- ✅ Provider chart (Pie) is displayed
- ✅ Field chart (Bar) is displayed
- ✅ Trends chart (Line) is displayed
- ✅ Statistics tables are displayed

---

### Test Case 34: Translation Analytics Data Accuracy

**Objective**: Verify analytics data is accurate

**Steps**:
1. Perform some translations
2. Navigate to `/Property/TranslationAnalytics`
3. Verify counts match actual translations

**Expected Results**:
- ✅ Total translations count matches `TranslationHistory` table
- ✅ Auto translations count matches API translations
- ✅ Manual translations count matches manual entries
- ✅ API calls count is accurate
- ✅ Cost estimation is reasonable

**Database Verification**:
```sql
-- Compare with analytics
SELECT COUNT(*) FROM TranslationHistory WHERE AccommodationId = @TestPropertyId;
SELECT COUNT(*) FROM TranslationHistory WHERE TranslationMethod = 'API_AUTO';
SELECT COUNT(*) FROM TranslationHistory WHERE TranslationMethod = 'MANUAL';
```

---

## Edge Cases & Error Scenarios

### Test Case 35: Missing DeepL API Key

**Objective**: Verify graceful handling when API key is missing

**Steps**:
1. Remove or invalidate DeepL API key in configuration
2. Attempt to translate a property
3. Verify error handling

**Expected Results**:
- ✅ Error is logged
- ✅ User-friendly error message is displayed
- ✅ Property update still succeeds (translation failure doesn't break main flow)
- ✅ No application crash

---

### Test Case 36: API Quota Exceeded

**Objective**: Verify handling when API quota is exceeded

**Steps**:
1. Exceed DeepL API quota (if possible)
2. Attempt to translate
3. Verify error handling

**Expected Results**:
- ✅ Error is logged
- ✅ User-friendly error message is displayed
- ✅ Property update still succeeds
- ✅ Error is recorded in translation history (if applicable)

---

### Test Case 37: Invalid Language Code

**Objective**: Verify handling of invalid language codes

**Steps**:
1. Manually insert invalid language code in database
2. Attempt to translate
3. Verify error handling

**Expected Results**:
- ✅ Error is logged
- ✅ Invalid language is skipped
- ✅ Other languages are still translated
- ✅ No application crash

---

### Test Case 38: Empty Translatable Field

**Objective**: Verify handling when translatable field is empty

**Steps**:
1. Create/update property with empty name
2. Verify translation behavior

**Expected Results**:
- ✅ Empty fields are skipped (not translated)
- ✅ No API call is made for empty fields
- ✅ No errors occur
- ✅ Other non-empty fields are still translated

---

### Test Case 39: Very Long Text Translation

**Objective**: Verify handling of very long text

**Steps**:
1. Enter very long property name (5000+ characters)
2. Attempt to translate
3. Verify translation succeeds

**Expected Results**:
- ✅ Long text is translated successfully
- ✅ Translation is saved correctly
- ✅ No truncation occurs
- ✅ Performance is acceptable

---

### Test Case 40: Special Characters Translation

**Objective**: Verify handling of special characters

**Steps**:
1. Enter text with special characters (emojis, symbols, etc.)
2. Attempt to translate
3. Verify translation handles special characters

**Expected Results**:
- ✅ Special characters are preserved or translated appropriately
- ✅ No encoding errors occur
- ✅ Text is saved correctly in database

---

### Test Case 41: Concurrent Translation Requests

**Objective**: Verify handling of concurrent translation requests

**Steps**:
1. Open multiple browser tabs
2. Update same property simultaneously in different tabs
3. Verify no conflicts occur

**Expected Results**:
- ✅ Both updates succeed
- ✅ Translations run correctly
- ✅ No database conflicts
- ✅ Translation history has entries for both

---

### Test Case 42: Network Failure During Translation

**Objective**: Verify handling of network failure

**Steps**:
1. Disconnect network during translation
2. Verify error handling
3. Reconnect and verify recovery

**Expected Results**:
- ✅ Error is logged
- ✅ Property update still succeeds
- ✅ Translation failure doesn't break main flow
- ✅ User can retry translation later

---

## Performance Testing

### Test Case 43: Translation Performance with Multiple Languages

**Objective**: Verify translation performance with many languages

**Steps**:
1. Enable 10+ languages
2. Update property name
3. Measure translation time

**Expected Results**:
- ✅ Translation completes within reasonable time (< 30 seconds for 10 languages)
- ✅ Parallel processing is used (Task.WhenAll)
- ✅ No timeout errors
- ✅ All languages are translated

---

### Test Case 44: Large Dataset Pagination Performance

**Objective**: Verify pagination performance with large history

**Steps**:
1. Create 1000+ translation history entries
2. Navigate to Translation History page
3. Verify pagination works smoothly

**Expected Results**:
- ✅ Page loads quickly (< 2 seconds)
- ✅ Pagination is responsive
- ✅ No performance degradation
- ✅ Server-side pagination is used

---

### Test Case 45: AllPropertiesLanguageSettings Performance

**Objective**: Verify performance with many properties

**Steps**:
1. Create 100+ properties
2. Navigate to `/Property/AllPropertiesLanguageSettings`
3. Verify page loads and pagination works

**Expected Results**:
- ✅ Page loads quickly (< 3 seconds)
- ✅ Server-side pagination works correctly
- ✅ Search and filters are responsive
- ✅ No performance issues

---

## Regression Testing Checklist

### Pre-Release Testing

Before releasing to production, verify:

- [ ] All test cases pass
- [ ] No console errors in browser
- [ ] No database errors in logs
- [ ] All API calls succeed
- [ ] All UI elements are functional
- [ ] Mobile responsiveness works
- [ ] Cross-browser compatibility (Chrome, Firefox, Edge)
- [ ] Performance is acceptable
- [ ] Security is maintained
- [ ] Error handling is graceful

### Critical Path Testing

Must test these critical paths:

1. **Language Selection → Translation → History**
   - [ ] Select language → Update property → Check history

2. **Master Switch → Per-Language → Translation**
   - [ ] Enable master switch → Configure languages → Update property → Verify translation

3. **Create → Translate → Revert**
   - [ ] Create property → Verify translation → Revert translation → Verify revert

4. **Analytics → Export**
   - [ ] View analytics → Export history → Verify export

---

## Test Data Setup Scripts

### SQL Scripts for Test Data

```sql
-- Create test property
INSERT INTO Accommodations (AccommodationName, AutoTranslateEnabled, StatusId)
VALUES ('Test Property', 1, 1);

-- Get test property ID
DECLARE @TestPropertyId INT = SCOPE_IDENTITY();

-- Initialize PerLanguageSettings for test property
INSERT INTO PerLanguageSettings (AccommodationId, LanguageId, AutoTranslateEnabled, IsActive)
SELECT @TestPropertyId, MultiLanguageId, 1, 1
FROM MultiLanguages
WHERE StatusId = 1;

-- Verify setup
SELECT * FROM Accommodations WHERE AccommodationId = @TestPropertyId;
SELECT * FROM PerLanguageSettings WHERE AccommodationId = @TestPropertyId;
```

---

## Test Execution Log Template

### Test Execution Log

| Test Case # | Test Case Name | Status | Notes | Tester | Date |
|-------------|----------------|--------|-------|--------|------|
| TC-01 | Language Dropdown Display | ✅ Pass | - | - | - |
| TC-02 | Language Search Functionality | ⏳ Pending | - | - | - |
| TC-03 | Language Selection & Cookie Persistence | ⏳ Pending | - | - | - |
| ... | ... | ... | ... | ... | ... |

**Status Legend**:
- ✅ Pass
- ❌ Fail
- ⏳ Pending
- ⚠️ Blocked
- 🔄 Retest

---

## Known Issues & Limitations

### Current Limitations

1. **Translation Provider**: Currently only DeepL API is supported
2. **Language Support**: Limited to languages supported by DeepL API
3. **Batch Size**: Translation is done per field, not in batches
4. **Retry Logic**: No automatic retry on API failure

### Known Issues

- None currently reported

---

## Support & Contact

For issues or questions regarding testing:

- **Email**: imfarooq1995@gmail.com
- **Documentation**: See `AUTOMATIC_TRANSLATION_IMPLEMENTATION.md`
- **Code Repository**: Check code comments for implementation details

---

**Document Version**: 3.0  
**Last Updated**: 2025  
**Status**: Production Ready ✅

**Happy Testing! 🧪**

