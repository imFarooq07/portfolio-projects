# 🌐 Language & Translation Testing Document

**Quick 1-Hour System Testing Guide for Developers**

This document provides streamlined test cases to verify the entire multi-language translation system in approximately 1 hour.

**Version**: 4.0  
**Last Updated**: 2025  
**Target Time**: 1 Hour  
**Status**: Production Ready ✅

---

## 📋 Quick Navigation

1. [⚡ 1-Hour Complete System Test](#-1-hour-complete-system-test) - **START HERE**
2. [📊 Translatable Fields Reference](#-translatable-fields-reference)
3. [🔍 SQL Verification Queries](#-sql-verification-queries)
4. [✅ Quick Test Checklist](#-quick-test-checklist)

---

## ⚡ 1-Hour Complete System Test

**Goal**: Verify entire translation system works correctly  
**Time**: ~60 minutes  
**API Cost**: ~20 calls (optimized)

### Phase 1: Setup & Configuration (10 minutes)

#### Step 1.1: Language Selection (2 min)
- [ ] Login to system
- [ ] Click language dropdown in header
- [ ] Verify search works (type "ara" to filter Arabic)
- [ ] Select Arabic language
- [ ] Verify page reloads with Arabic
- [ ] Verify cookie is set (check browser DevTools → Application → Cookies → `SelectedLanguageId`)

#### Step 1.2: Master Switch Setup (3 min)
- [ ] Go to `/Property/UpdateProperty`
- [ ] Find "Auto Translate Enabled" checkbox
- [ ] Enable it (check the box)
- [ ] Save property
- [ ] Verify success message

#### Step 1.3: Per-Language Settings (5 min)
- [ ] Go to `/Property/AllPropertiesLanguageSettings`
- [ ] Find your property card
- [ ] Verify "Master Switch" is ON (green badge)
- [ ] Verify all languages are displayed in grid
- [ ] Verify English (EN) is always ON and disabled (cannot turn off)
- [ ] Toggle Arabic (AR) ON if not already ON
- [ ] Toggle Turkish (TR) ON
- [ ] Verify success toast appears for each toggle

**SQL Check** (Run in SSMS):
```sql
-- Verify master switch and per-language settings
SELECT a.AccommodationId, a.AutoTranslateEnabled, 
       pls.LanguageId, ml.MultiLanguageName, pls.AutoTranslateEnabled
FROM BookingWhizz.dbo.Accommodations a
LEFT JOIN BookingWhizz.dbo.PerLanguageSettings pls ON a.AccommodationId = pls.AccommodationId
LEFT JOIN BookingWhizz.dbo.MultiLanguages ml ON pls.LanguageId = ml.MultiLanguageId
WHERE a.AccommodationId = @YourPropertyId
ORDER BY pls.LanguageId;
```

---

### Phase 2: Translation Testing (30 minutes)

#### Step 2.1: Property Translation (5 min) ⭐
- [ ] Select Arabic from header
- [ ] Go to `/Property/UpdateProperty`
- [ ] Change property name to: **"Hotel ABC"** (short text)
- [ ] Save property
- [ ] Check Translation History page - should show 1 entry
- [ ] **SQL Verification**:
```sql
SELECT AccommodationId, MultiLanguageId, AccommodationName
FROM BookingWhizz.dbo.Accommodations_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY MultiLanguageId;
-- Expected: 2 rows (English + Arabic)
```

#### Step 2.2: Addon Translation (5 min)
- [ ] Go to `/Property/CreateAddon`
- [ ] Enter Addon Name: **"Spa"** (short text only)
- [ ] **Skip other fields** (to save API cost)
- [ ] Save addon
- [ ] **SQL Verification**:
```sql
SELECT ActivityId, MultiLanguageId, ActivityName
FROM BookingWhizz.dbo.Activities_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY ActivityId, MultiLanguageId;
-- Expected: 2 rows per addon (English + Arabic)
```

#### Step 2.3: Room Translation (5 min)
- [ ] Go to `/RoomManagement/CreateRoom`
- [ ] Select Room Type = 1 (Room)
- [ ] Enter Room Name: **"Deluxe"** (short text only)
- [ ] **Skip description** (to save cost)
- [ ] Save room
- [ ] **SQL Verification**:
```sql
SELECT RoomId, MultiLanguageId, RoomName
FROM BookingWhizz.dbo.Rooms_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY RoomId, MultiLanguageId;
-- Expected: 2 rows per room (English + Arabic)
```

#### Step 2.4: Rate Plan Translation (5 min)
- [ ] Go to `/RoomManagement/CreateRatePlan`
- [ ] Enter Rate Plan Name: **"Standard"** (short text only)
- [ ] **Skip other fields** (to save cost)
- [ ] Save rate plan
- [ ] **SQL Verification**:
```sql
SELECT RatePlanId, MultiLanguageId, RatePlanName
FROM BookingWhizz.dbo.RatePlans_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY RatePlanId, MultiLanguageId;
-- Expected: 2 rows per rate plan (English + Arabic)
```

#### Step 2.5: Promotion Translation (5 min)
- [ ] Go to `/PromotionManagement/CreatePromotion`
- [ ] Enter Promotion Name: **"Summer Sale"** (short text only)
- [ ] **Skip description** (to save cost)
- [ ] Save promotion
- [ ] **SQL Verification**:
```sql
SELECT PromotionId, MultiLanguageId, PromotionName
FROM BookingWhizz.dbo.Promotions_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY PromotionId, MultiLanguageId;
-- Expected: 2 rows per promotion (English + Arabic)
```

#### Step 2.6: Field Change Detection (5 min)
- [ ] Go to `/Property/UpdateProperty`
- [ ] Change NON-translatable field (e.g., Email, Phone)
- [ ] Save property
- [ ] Check Translation History - **should show NO new entries**
- [ ] Verify logs show: "Translation skipped - no translatable fields changed"

---

### Phase 3: Edge Cases & Validation (15 minutes)

#### Step 3.1: English Language Protection (3 min)
- [ ] Go to `/Property/AllPropertiesLanguageSettings`
- [ ] Find your property
- [ ] Try to toggle English (EN) OFF
- [ ] Verify: SweetAlert warning appears
- [ ] Verify: English toggle remains ON
- [ ] Verify: Cannot disable English

#### Step 3.2: Master Switch OFF (3 min)
- [ ] Go to `/Property/UpdateProperty`
- [ ] Disable "Auto Translate Enabled"
- [ ] Save property
- [ ] Change property name to: **"Test Hotel"**
- [ ] Save property
- [ ] Check Translation History - **should show NO new entries**
- [ ] Verify logs show: "Translation skipped - AutoTranslateEnabled: false"

#### Step 3.3: Per-Language Disable (3 min)
- [ ] Go to `/Property/AllPropertiesLanguageSettings`
- [ ] Toggle Arabic (AR) OFF
- [ ] Go to `/Property/UpdateProperty`
- [ ] Change property name to: **"Hotel XYZ"**
- [ ] Save property
- [ ] **SQL Verification**:
```sql
SELECT MultiLanguageId, AccommodationName
FROM BookingWhizz.dbo.Accommodations_ML 
WHERE AccommodationId = @YourPropertyId AND MultiLanguageId = 2;
-- Expected: Should NOT have Arabic translation (or old value)
```

#### Step 3.4: Non-English to English Pre-Translation (6 min)
- [ ] Select Arabic from header
- [ ] Go to `/Property/CreateProperty`
- [ ] Enter property name in Arabic: **"فندق جديد"** (or any Arabic text)
- [ ] Enable "Auto Translate Enabled"
- [ ] Select languages: English + Arabic + Turkish
- [ ] Save property
- [ ] **SQL Verification**:
```sql
-- Check main table has English (LanguageId = 1)
SELECT AccommodationId, AccommodationName
FROM BookingWhizz.dbo.Accommodations 
WHERE AccommodationName LIKE '%Hotel%' OR AccommodationName LIKE '%New%'
ORDER BY AccommodationId DESC;

-- Check ML table has original Arabic
SELECT AccommodationId, MultiLanguageId, AccommodationName
FROM BookingWhizz.dbo.Accommodations_ML 
WHERE AccommodationId = @NewPropertyId
ORDER BY MultiLanguageId;
-- Expected: Main table = English, ML table = Arabic (original) + English + Turkish
```

---

### Phase 4: Bulk Translation & Facilities (5 minutes)

#### Step 4.1: Bulk Translate Accommodation Groups (2 min)
- [ ] Go to `/Property/PropertyFacilities`
- [ ] Click "Bulk Translate Accommodation Groups" button (if available)
- [ ] Verify success message
- [ ] **SQL Verification**:
```sql
SELECT OwnerId, MultiLanguageId, GroupId, GroupName
FROM BookingWhizz.dbo.AFGroups_ML 
WHERE OwnerId = @YourOwnerId
ORDER BY GroupId, MultiLanguageId;
-- Expected: All groups have entries for enabled languages
```

#### Step 4.2: Bulk Translate Room Groups (3 min)
- [ ] Go to `/RoomManagement/RoomFacilities`
- [ ] Click "Bulk Translate Room Groups" button
- [ ] Verify success message
- [ ] **SQL Verification**:
```sql
SELECT OwnerId, MultiLanguageId, GroupId, GroupName
FROM BookingWhizz.dbo.RFGroups_ML 
WHERE OwnerId = @YourOwnerId
ORDER BY GroupId, MultiLanguageId;
-- Expected: All groups have entries for enabled languages
```

---

### Phase 5: Final Verification (5 minutes)

#### Step 5.1: Complete ML Tables Check (3 min)
Run this **one combined SQL query** to verify all tables:
```sql
-- Quick verification - All main ML tables
SELECT 'Accommodations_ML' AS TableName, COUNT(*) AS TotalRecords,
       COUNT(CASE WHEN MultiLanguageId = 1 THEN 1 END) AS EnglishCount,
       COUNT(CASE WHEN MultiLanguageId = 2 THEN 1 END) AS ArabicCount
FROM BookingWhizz.dbo.Accommodations_ML 
WHERE AccommodationId = @YourPropertyId

UNION ALL

SELECT 'Rooms_ML', COUNT(*),
       COUNT(CASE WHEN MultiLanguageId = 1 THEN 1 END),
       COUNT(CASE WHEN MultiLanguageId = 2 THEN 1 END)
FROM BookingWhizz.dbo.Rooms_ML 
WHERE AccommodationId = @YourPropertyId

UNION ALL

SELECT 'RatePlans_ML', COUNT(*),
       COUNT(CASE WHEN MultiLanguageId = 1 THEN 1 END),
       COUNT(CASE WHEN MultiLanguageId = 2 THEN 1 END)
FROM BookingWhizz.dbo.RatePlans_ML 
WHERE AccommodationId = @YourPropertyId

UNION ALL

SELECT 'Activities_ML', COUNT(*),
       COUNT(CASE WHEN MultiLanguageId = 1 THEN 1 END),
       COUNT(CASE WHEN MultiLanguageId = 2 THEN 1 END)
FROM BookingWhizz.dbo.Activities_ML 
WHERE AccommodationId = @YourPropertyId

UNION ALL

SELECT 'Promotions_ML', COUNT(*),
       COUNT(CASE WHEN MultiLanguageId = 1 THEN 1 END),
       COUNT(CASE WHEN MultiLanguageId = 2 THEN 1 END)
FROM BookingWhizz.dbo.Promotions_ML 
WHERE AccommodationId = @YourPropertyId;
```

**Expected Results**:
- ✅ All 5 tables have records
- ✅ Each item has 2 records (English + Arabic)
- ✅ EnglishCount = ArabicCount for each table

#### Step 5.2: Translation History Check (2 min)
- [ ] Go to `/Property/TranslationHistory`
- [ ] Verify all translation entries are listed
- [ ] Filter by language (Arabic)
- [ ] Verify filtered results
- [ ] Check API calls count matches expected (~20 calls)

---

## 📊 Translatable Fields Reference

### ⚡ Important: Field Names for API Cost Reduction

**Note**: Field names are kept short to minimize API costs. Use these exact names in test cases.

### Complete Field Mapping

#### 1. Property Module (`/Property/CreateProperty`, `/Property/UpdateProperty`)

| Short Name | Full Field Name | Description | API Cost Impact |
|------------|----------------|-------------|-----------------|
| `PropName` | `AccommodationName` | Property name | ⭐ High (most used) |

**Total Fields**: 1

---

#### 2. Additional Details Module (`/Property/AdditionalDetail`)

| Short Name | Full Field Name | Description | API Cost Impact |
|------------|----------------|-------------|-----------------|
| `KeyComments` | `KeyCollectionComments` | Key collection instructions | ⭐ Medium |
| `PetsComments` | `PetsAllowedComments` | Pet policy comments | ⭐ Medium |

**Total Fields**: 2

---

#### 3. Addon Module (`/Property/CreateAddon`, `/Property/UpdateAddon`)

| Short Name | Full Field Name | Description | API Cost Impact |
|------------|----------------|-------------|-----------------|
| `AddonName` | `ActivityName` | Addon/Activity name | ⭐ High |
| `ShortDesc` | `ShortDescription` | Short description | ⭐ High |
| `LongDesc` | `LongDescription` | Long description | ⭐ High |
| `CancelPolicy` | `CancellationPolicy` | Cancellation policy | ⭐ Medium |
| `GuaranteePolicy` | `GuaranteePolicy` | Guarantee policy | ⭐ Medium |

**Total Fields**: 5

---

#### 4. Room Module (`/RoomManagement/CreateRoom`, `/RoomManagement/UpdateRoom`)

| Short Name | Full Field Name | Description | API Cost Impact |
|------------|----------------|-------------|-----------------|
| `RoomName` | `RoomName` | Room name (RoomTypeId=1) | ⭐ High |
| `AptName` | `ApartmentName` | Apartment name (RoomTypeId=2) | ⭐ High |
| `RoomDesc` | `RoomDescription` | Room description | ⭐ High |

**Total Fields**: 3

---

#### 5. Rate Plan Module (`/RoomManagement/CreateRatePlan`, `/RoomManagement/UpdateRatePlan`)

| Short Name | Full Field Name | Description | API Cost Impact |
|------------|----------------|-------------|-----------------|
| `RatePlanName` | `RatePlanName` | Rate plan name | ⭐ High |
| `DisplayName` | `DisplayRatePlanName` | Display name | ⭐ High |
| `Included` | `Included` | Included items | ⭐ Medium |
| `Highlight` | `Highlight` | Highlights | ⭐ Medium |
| `MealDesc` | `MealDescription` | Meal description | ⭐ Medium |

**Total Fields**: 5

---

#### 6. Promotion Module (`/PromotionManagement/CreatePromotion`, `/PromotionManagement/UpdatePromotion`)

| Short Name | Full Field Name | Description | API Cost Impact |
|------------|----------------|-------------|-----------------|
| `PromoName` | `PromotionName` | Promotion name | ⭐ High |
| `PromoDesc` | `Description` | Promotion description | ⭐ High |

**Total Fields**: 2

---

#### 7. Property Facilities Module (`/Property/PropertyFacilities`)

| Short Name | Full Field Name | Description | API Cost Impact |
|------------|----------------|-------------|-----------------|
| `GroupName` | `GroupName` | Facility group name | ⭐ Low |
| `FacilityName` | `FacilityName` | Facility name | ⭐ Low |
| `OtherGroupName` | `OtherGroupName` | Other group name | ⭐ Low |
| `OtherFacilityName` | `OtherFacilityName` | Other facility name | ⭐ Low |

**Total Fields**: 4

---

#### 8. Room Facilities Module (`/RoomManagement/RoomFacilities`)

| Short Name | Full Field Name | Description | API Cost Impact |
|------------|----------------|-------------|-----------------|
| `GroupName` | `GroupName` | Room facility group name | ⭐ Low |
| `FacilityName` | `FacilityName` | Room facility name | ⭐ Low |
| `OtherGroupName` | `OtherGroupName` | Other group name | ⭐ Low |
| `OtherFacilityName` | `OtherFacilityName` | Other facility name | ⭐ Low |

**Total Fields**: 4

---

### 📊 Summary

| Module | Total Fields | High Impact Fields | Medium Impact Fields | Low Impact Fields |
|--------|-------------|-------------------|---------------------|-------------------|
| Property | 1 | 1 | 0 | 0 |
| Additional Details | 2 | 0 | 2 | 0 |
| Addon | 5 | 3 | 2 | 0 |
| Room | 3 | 3 | 0 | 0 |
| Rate Plan | 5 | 2 | 3 | 0 |
| Promotion | 2 | 2 | 0 | 0 |
| Property Facilities | 4 | 0 | 0 | 4 |
| Room Facilities | 4 | 0 | 0 | 4 |
| **TOTAL** | **26** | **11** | **7** | **8** |

### 💡 API Cost Optimization Tips

1. **Use Short Text**: Use short words like "Hotel", "Spa", "Deluxe" instead of long descriptions
2. **Test One Field Per Module**: Test only `PropName`, `AddonName`, `RoomName` (not all fields)
3. **Enable Only 2-3 Languages**: Enable English + 1-2 languages for testing (not all 33 languages)
4. **Test High Impact Fields First**: Focus on fields marked with ⭐ High
5. **Monitor API Usage**: Check Translation History for actual API calls made

### 💰 Cost Comparison Examples

| Test Scenario | Fields | Languages | API Calls | Cost Level |
|--------------|--------|-----------|-----------|------------|
| **Property Name Only** | 1 (PropName) | 2 (EN+AR) | 2 | ✅ **Lowest** |
| **Property Name Only** | 1 (PropName) | 5 (EN+AR+TR+FR+DE) | 5 | ⚠️ Medium |
| **Addon All Fields** | 5 (All) | 3 (EN+AR+TR) | 15 | ❌ High |
| **Addon Name Only** | 1 (AddonName) | 2 (EN+AR) | 2 | ✅ **Lowest** |
| **Room All Fields** | 3 (All) | 3 (EN+AR+TR) | 9 | ⚠️ Medium |
| **Room Name Only** | 1 (RoomName) | 2 (EN+AR) | 2 | ✅ **Lowest** |

---

### 🔍 Quick SQL Reference for ML Tables Testing

**Copy-paste ready SQL queries for testing** (Replace `11481` with your test property ID):

```sql
-- 1. Accommodations_ML (Property translations)
SELECT AccommodationId, MultiLanguageId, AccommodationName, 
       KeyCollectionComments, PetsAllowedComments
FROM BookingWhizz.dbo.Accommodations_ML 
WHERE AccommodationId = 11481
ORDER BY MultiLanguageId;

-- 2. Rooms_ML (Room translations)
SELECT RoomId, AccommodationId, MultiLanguageId, 
       RoomName, ApartmentName, RoomDescription
FROM BookingWhizz.dbo.Rooms_ML 
WHERE AccommodationId = 11481
ORDER BY RoomId, MultiLanguageId;

-- 3. RatePlans_ML (Rate plan translations)
SELECT RatePlanId, AccommodationId, MultiLanguageId, 
       RatePlanName, DisplayRatePlanName, Included, 
       Highlight, MealDescription
FROM BookingWhizz.dbo.RatePlans_ML 
WHERE AccommodationId = 11481
ORDER BY RatePlanId, MultiLanguageId;

-- 4. Activities_ML (Addon translations)
SELECT ActivityId, AccommodationId, MultiLanguageId, 
       ActivityName, ShortDescription, LongDescription,
       CancellationPolicy, GuaranteePolicy
FROM BookingWhizz.dbo.Activities_ML 
WHERE AccommodationId = 11481
ORDER BY ActivityId, MultiLanguageId;

-- 5. Promotions_ML (Promotion translations)
SELECT PromotionId, AccommodationId, MultiLanguageId, 
       PromotionName, Description
FROM BookingWhizz.dbo.Promotions_ML 
WHERE AccommodationId = 11481
ORDER BY PromotionId, MultiLanguageId;

-- 6. AFGroups_ML (Accommodation Groups) - Replace @YourOwnerId
SELECT OwnerId, MultiLanguageId, GroupId, GroupName, OtherGroupName
FROM BookingWhizz.dbo.AFGroups_ML 
WHERE OwnerId = @YourOwnerId
ORDER BY GroupId, MultiLanguageId;

-- 7. AccommodationsFacilities_ML (Accommodation Facilities) - Replace @YourOwnerId
SELECT OwnerId, MultiLanguageId, GroupId, FacilityId, 
       FacilityName, OtherFacilityName
FROM BookingWhizz.dbo.AccommodationsFacilities_ML 
WHERE OwnerId = @YourOwnerId
ORDER BY FacilityId, MultiLanguageId;

-- 8. RFGroups_ML (Room Groups) - Replace @YourOwnerId
SELECT OwnerId, MultiLanguageId, GroupId, GroupName, OtherGroupName
FROM BookingWhizz.dbo.RFGroups_ML 
WHERE OwnerId = @YourOwnerId
ORDER BY GroupId, MultiLanguageId;

-- 9. RoomFacilities_ML (Room Facilities) - Replace @YourOwnerId
SELECT OwnerId, MultiLanguageId, GroupId, FacilityId, 
       FacilityName, OtherFacilityName
FROM BookingWhizz.dbo.RoomFacilities_ML 
WHERE OwnerId = @YourOwnerId
ORDER BY FacilityId, MultiLanguageId;

-- 10. Complete Verification (All tables count)
SELECT 'Accommodations_ML' AS TableName, COUNT(*) AS RecordCount
FROM BookingWhizz.dbo.Accommodations_ML WHERE AccommodationId = 11481
UNION ALL
SELECT 'Rooms_ML', COUNT(*) FROM BookingWhizz.dbo.Rooms_ML WHERE AccommodationId = 11481
UNION ALL
SELECT 'RatePlans_ML', COUNT(*) FROM BookingWhizz.dbo.RatePlans_ML WHERE AccommodationId = 11481
UNION ALL
SELECT 'Activities_ML', COUNT(*) FROM BookingWhizz.dbo.Activities_ML WHERE AccommodationId = 11481
UNION ALL
SELECT 'Promotions_ML', COUNT(*) FROM BookingWhizz.dbo.Promotions_ML WHERE AccommodationId = 11481;
```

---

## 🔍 SQL Verification Queries

### Quick Reference - All ML Tables

**Replace `@YourPropertyId` with your test property ID**

```sql
-- 1. Accommodations_ML (Property)
SELECT AccommodationId, MultiLanguageId, AccommodationName
FROM BookingWhizz.dbo.Accommodations_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY MultiLanguageId;

-- 2. Rooms_ML
SELECT RoomId, MultiLanguageId, RoomName
FROM BookingWhizz.dbo.Rooms_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY RoomId, MultiLanguageId;

-- 3. RatePlans_ML
SELECT RatePlanId, MultiLanguageId, RatePlanName
FROM BookingWhizz.dbo.RatePlans_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY RatePlanId, MultiLanguageId;

-- 4. Activities_ML (Addons)
SELECT ActivityId, MultiLanguageId, ActivityName
FROM BookingWhizz.dbo.Activities_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY ActivityId, MultiLanguageId;

-- 5. Promotions_ML
SELECT PromotionId, MultiLanguageId, PromotionName
FROM BookingWhizz.dbo.Promotions_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY PromotionId, MultiLanguageId;

-- 6. AFGroups_ML (Accommodation Groups) - Replace @YourOwnerId
SELECT OwnerId, MultiLanguageId, GroupId, GroupName
FROM BookingWhizz.dbo.AFGroups_ML 
WHERE OwnerId = @YourOwnerId
ORDER BY GroupId, MultiLanguageId;

-- 7. AccommodationsFacilities_ML - Replace @YourOwnerId
SELECT OwnerId, MultiLanguageId, FacilityId, FacilityName
FROM BookingWhizz.dbo.AccommodationsFacilities_ML 
WHERE OwnerId = @YourOwnerId
ORDER BY FacilityId, MultiLanguageId;

-- 8. RFGroups_ML (Room Groups) - Replace @YourOwnerId
SELECT OwnerId, MultiLanguageId, GroupId, GroupName
FROM BookingWhizz.dbo.RFGroups_ML 
WHERE OwnerId = @YourOwnerId
ORDER BY GroupId, MultiLanguageId;

-- 9. RoomFacilities_ML - Replace @YourOwnerId
SELECT OwnerId, MultiLanguageId, FacilityId, FacilityName
FROM BookingWhizz.dbo.RoomFacilities_ML 
WHERE OwnerId = @YourOwnerId
ORDER BY FacilityId, MultiLanguageId;
```

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

---

## 📝 Detailed Test Cases (Reference Only)

*Note: The 1-Hour Test above covers all critical paths. Use these detailed cases only if you need to test specific scenarios.*

### Language Selection Testing

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

### Test Case 15: Property Name Translation (PropName) ⭐ LOWEST COST

**Field**: `PropName` (AccommodationName)  
**Page**: `/Property/UpdateProperty`  
**Time**: 3 minutes  
**API Cost**: **2 calls** (1 field × 2 languages)

**Steps**:
1. ✅ Enable master switch (if not enabled)
2. ✅ Enable only 2 languages: English + Arabic (on AllPropertiesLanguageSettings page)
3. ✅ Select Arabic language from header
4. ✅ Go to `/Property/UpdateProperty`
5. ✅ Change property name to short text: **"Hotel ABC"** (use short text to reduce cost)
6. ✅ Click Save
7. ✅ Check Translation History page - should show 1 entry
8. ✅ **Run SQL to verify table records**:

**SQL Verification**:
```sql
-- Check if record inserted/updated correctly
SELECT AccommodationId, MultiLanguageId, AccommodationName,
       CASE 
           WHEN MultiLanguageId = 1 THEN 'English'
           WHEN MultiLanguageId = 2 THEN 'Arabic'
           ELSE 'Other'
       END AS LanguageName
FROM BookingWhizz.dbo.Accommodations_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY MultiLanguageId;

-- Expected: 2 rows (English + Arabic)
-- English (MultiLanguageId = 1): "Hotel ABC" (original)
-- Arabic (MultiLanguageId = 2): Translated text in Arabic
```

**Expected Results**:
- ✅ Property saved successfully
- ✅ Translation entry in history (Field: `PropName`)
- ✅ **2 records in `Accommodations_ML` table** (English + Arabic)
- ✅ English record has original text
- ✅ Arabic record has translated text
- ✅ API cost: **2 calls only** (lowest cost)

---

### Test Case 16: Addon Translation (AddonName Only) ⭐ LOWEST COST

**Field**: `AddonName` (ActivityName) - **Only 1 field to reduce cost**  
**Page**: `/Property/CreateAddon` or `/Property/UpdateAddon`  
**Time**: 3 minutes  
**API Cost**: **2 calls** (1 field × 2 languages)

**Steps**:
1. ✅ Enable only 2 languages: English + Arabic
2. ✅ Go to `/Property/CreateAddon`
3. ✅ Enter **short text**:
   - Addon Name (`AddonName`): **"Spa"** (use short word to reduce cost)
   - **Skip Short Description and Long Description** (to save API cost)
4. ✅ Save addon
5. ✅ Check Translation History - should show 1 entry
6. ✅ **Run SQL to verify table records**:

**SQL Verification**:
```sql
-- Check if record inserted correctly
SELECT ActivityId, AccommodationId, MultiLanguageId, ActivityName,
       CASE 
           WHEN MultiLanguageId = 1 THEN 'English'
           WHEN MultiLanguageId = 2 THEN 'Arabic'
           ELSE 'Other'
       END AS LanguageName
FROM BookingWhizz.dbo.Activities_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY ActivityId, MultiLanguageId;

-- Expected: 2 rows per addon (English + Arabic)
-- English (MultiLanguageId = 1): "Spa" (original)
-- Arabic (MultiLanguageId = 2): Translated text in Arabic
```

**Expected Results**:
- ✅ Addon created successfully
- ✅ **1 translation entry in history** (only AddonName field)
- ✅ **2 records in `Activities_ML` table** (English + Arabic)
- ✅ English record has original text "Spa"
- ✅ Arabic record has translated text
- ✅ API cost: **2 calls only** (lowest cost)

**💡 Cost Saving Tip**: Skip `ShortDesc` and `LongDesc` fields during testing to save 4 API calls per language!

---

### Test Case 17: Room Translation (RoomName Only) ⭐ LOWEST COST

**Field**: `RoomName` (RoomName) - **Only 1 field to reduce cost**  
**Page**: `/RoomManagement/CreateRoom`  
**Time**: 3 minutes  
**API Cost**: **2 calls** (1 field × 2 languages)

**Steps**:
1. ✅ Enable only 2 languages: English + Arabic
2. ✅ Go to `/RoomManagement/CreateRoom`
3. ✅ Select Room Type (1 = Room)
4. ✅ Enter **short text**:
   - Room Name (`RoomName`): **"Deluxe"** (use short word to reduce cost)
   - **Skip Room Description** (to save API cost)
5. ✅ Save room
6. ✅ Check Translation History - should show 1 entry
7. ✅ **Run SQL to verify table records**:

**SQL Verification**:
```sql
-- Check if record inserted correctly
SELECT RoomId, AccommodationId, MultiLanguageId, RoomName,
       CASE 
           WHEN MultiLanguageId = 1 THEN 'English'
           WHEN MultiLanguageId = 2 THEN 'Arabic'
           ELSE 'Other'
       END AS LanguageName
FROM BookingWhizz.dbo.Rooms_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY RoomId, MultiLanguageId;

-- Expected: 2 rows per room (English + Arabic)
-- English (MultiLanguageId = 1): "Deluxe" (original)
-- Arabic (MultiLanguageId = 2): Translated text in Arabic
```

**Expected Results**:
- ✅ Room created successfully
- ✅ **1 translation entry in history** (only RoomName field)
- ✅ **2 records in `Rooms_ML` table** (English + Arabic)
- ✅ English record has original text "Deluxe"
- ✅ Arabic record has translated text
- ✅ API cost: **2 calls only** (lowest cost)

**💡 Cost Saving Tip**: Skip `RoomDesc` field during testing to save 2 API calls per language!

---

### Test Case 18: Rate Plan Translation (RatePlanName Only) ⭐ LOWEST COST

**Field**: `RatePlanName` (RatePlanName) - **Only 1 field to reduce cost**  
**Page**: `/RoomManagement/CreateRatePlan`  
**Time**: 3 minutes  
**API Cost**: **2 calls** (1 field × 2 languages)

**Steps**:
1. ✅ Enable only 2 languages: English + Arabic
2. ✅ Go to `/RoomManagement/CreateRatePlan`
3. ✅ Enter **short text**:
   - Rate Plan Name (`RatePlanName`): **"Standard"** (use short word to reduce cost)
   - **Skip Display Name and other fields** (to save API cost)
4. ✅ Save rate plan
5. ✅ Check Translation History - should show 1 entry
6. ✅ **Run SQL to verify table records**:

**SQL Verification**:
```sql
-- Check if record inserted correctly
SELECT RatePlanId, AccommodationId, MultiLanguageId, RatePlanName,
       CASE 
           WHEN MultiLanguageId = 1 THEN 'English'
           WHEN MultiLanguageId = 2 THEN 'Arabic'
           ELSE 'Other'
       END AS LanguageName
FROM BookingWhizz.dbo.RatePlans_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY RatePlanId, MultiLanguageId;

-- Expected: 2 rows per rate plan (English + Arabic)
-- English (MultiLanguageId = 1): "Standard" (original)
-- Arabic (MultiLanguageId = 2): Translated text in Arabic
```

**Expected Results**:
- ✅ Rate plan created successfully
- ✅ **1 translation entry in history** (only RatePlanName field)
- ✅ **2 records in `RatePlans_ML` table** (English + Arabic)
- ✅ English record has original text "Standard"
- ✅ Arabic record has translated text
- ✅ API cost: **2 calls only** (lowest cost)

**💡 Cost Saving Tip**: Skip `DisplayName`, `Included`, `Highlight`, `MealDesc` fields during testing to save 8 API calls per language!

---

### Test Case 19: Promotion Translation (PromoName Only) ⭐ LOWEST COST

**Field**: `PromoName` (PromotionName) - **Only 1 field to reduce cost**  
**Page**: `/PromotionManagement/CreatePromotion`  
**Time**: 3 minutes  
**API Cost**: **2 calls** (1 field × 2 languages)

**Steps**:
1. ✅ Enable only 2 languages: English + Arabic
2. ✅ Go to `/PromotionManagement/CreatePromotion`
3. ✅ Enter **short text**:
   - Promotion Name (`PromoName`): **"Summer Sale"** (use short text to reduce cost)
   - **Skip Description field** (to save API cost)
4. ✅ Save promotion
5. ✅ Check Translation History - should show 1 entry
6. ✅ **Run SQL to verify table records**:

**SQL Verification**:
```sql
-- Check if record inserted correctly
SELECT PromotionId, AccommodationId, MultiLanguageId, PromotionName,
       CASE 
           WHEN MultiLanguageId = 1 THEN 'English'
           WHEN MultiLanguageId = 2 THEN 'Arabic'
           ELSE 'Other'
       END AS LanguageName
FROM BookingWhizz.dbo.Promotions_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY PromotionId, MultiLanguageId;

-- Expected: 2 rows per promotion (English + Arabic)
-- English (MultiLanguageId = 1): "Summer Sale" (original)
-- Arabic (MultiLanguageId = 2): Translated text in Arabic
```

**Expected Results**:
- ✅ Promotion created successfully
- ✅ **1 translation entry in history** (only PromoName field)
- ✅ **2 records in `Promotions_ML` table** (English + Arabic)
- ✅ English record has original text "Summer Sale"
- ✅ Arabic record has translated text
- ✅ API cost: **2 calls only** (lowest cost)

**💡 Cost Saving Tip**: Skip `PromoDesc` field during testing to save 2 API calls per language!

---

### Test Case 20: Additional Details Translation (KeyComments, PetsComments)

**Fields**: `KeyComments`, `PetsComments`  
**Page**: `/Property/AdditionalDetail`  
**Time**: 2 minutes

**Steps**:
1. ✅ Go to `/Property/AdditionalDetail`
2. ✅ Enter:
   - Key Collection Comments (`KeyComments`)
   - Pets Allowed Comments (`PetsComments`)
3. ✅ Save
4. ✅ Check Translation History

**Expected Results**:
- ✅ Details saved successfully
- ✅ 2 translation entries in history
- ✅ Translations in `Accommodations_ML` table

---

### Test Case 21: Translation Skipped - No Field Change

**Objective**: Verify translation doesn't run when no translatable fields change

**Steps**:
1. ✅ Go to `/Property/UpdateProperty`
2. ✅ Change NON-translatable field (e.g., Email, Phone, Address)
3. ✅ Save property
4. ✅ Check Translation History

**Expected Results**:
- ✅ Property updated successfully
- ✅ NO translation entries in history
- ✅ No API calls made
- ✅ Logs show: "Translation skipped - no translatable fields changed"

---

### Test Case 22: Translation Skipped - Master Switch OFF

**Objective**: Verify translation doesn't run when master switch is disabled

**Steps**:
1. ✅ Disable master switch on UpdateProperty page
2. ✅ Change property name (PropName)
3. ✅ Save property
4. ✅ Check Translation History

**Expected Results**:
- ✅ Property updated successfully
- ✅ NO translation entries in history
- ✅ No API calls made
- ✅ Logs show: "Translation skipped - AutoTranslateEnabled: false"

---

### Test Case 23: Translation for Specific Languages Only

**Objective**: Verify translation runs only for enabled languages

**Steps**:
1. ✅ Enable master switch
2. ✅ Go to `/Property/AllPropertiesLanguageSettings`
3. ✅ Disable Arabic language (keep Turkish enabled)
4. ✅ Update property name (PropName)
5. ✅ Check Translation History

**Expected Results**:
- ✅ Translation runs for Turkish only
- ✅ NO translation for Arabic
- ✅ Only Turkish entry in history
- ✅ Only Turkish entry in `Accommodations_ML` table

---

### Test Case 24: Multiple Fields Translation (Batch Test)

**Objective**: Test multiple fields in one update to verify batch translation

**Steps**:
1. ✅ Go to `/Property/UpdateProperty`
2. ✅ Change multiple translatable fields:
   - Property Name (`PropName`)
   - (If Additional Details page exists, update `KeyComments` and `PetsComments`)
3. ✅ Save property
4. ✅ Check Translation History

**Expected Results**:
- ✅ All fields translated
- ✅ Multiple entries in history (one per field)
- ✅ All translations in ML tables
- ✅ API cost: Number of fields × Number of enabled languages

---

### Test Case 25: English Always Translated

**Objective**: Verify English is always included in translation

**Steps**:
1. ✅ Select Arabic language from header
2. ✅ Update property name (PropName)
3. ✅ Check Translation History

**Expected Results**:
- ✅ Arabic translation created
- ✅ English translation also created (LanguageId = 1)
- ✅ Both entries in history
- ✅ Both entries in `Accommodations_ML` table

---

### Test Case 26: Accommodations_ML Table Verification ⭐ SIMPLE CHECK

**Table**: `Accommodations_ML`  
**Field**: `PropName` only (to reduce cost)  
**Time**: 2 minutes

**Steps**:
1. ✅ Update property name (`PropName`) to **"Hotel ABC"** on UpdateProperty page
2. ✅ Save property
3. ✅ **Run SQL query to verify records**:

```sql
-- Simple verification query
SELECT AccommodationId, MultiLanguageId, AccommodationName,
       CASE 
           WHEN MultiLanguageId = 1 THEN 'English'
           WHEN MultiLanguageId = 2 THEN 'Arabic'
           WHEN MultiLanguageId = 3 THEN 'Turkish'
           ELSE 'Other'
       END AS LanguageName
FROM BookingWhizz.dbo.Accommodations_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY MultiLanguageId;

-- Check record count (should match enabled languages)
SELECT COUNT(*) AS TotalRecords,
       COUNT(CASE WHEN MultiLanguageId = 1 THEN 1 END) AS EnglishCount,
       COUNT(CASE WHEN MultiLanguageId = 2 THEN 1 END) AS ArabicCount
FROM BookingWhizz.dbo.Accommodations_ML 
WHERE AccommodationId = @YourPropertyId;
```

**Expected Results**:
- ✅ **2 records** (if 2 languages enabled: English + Arabic)
- ✅ English (MultiLanguageId = 1): "Hotel ABC" (original text)
- ✅ Arabic (MultiLanguageId = 2): Translated text in Arabic
- ✅ Both records have `AccommodationName` filled
- ✅ No NULL values in `AccommodationName` column

---

### Test Case 27: Rooms_ML Table Verification ⭐ SIMPLE CHECK

**Table**: `Rooms_ML`  
**Field**: `RoomName` only (to reduce cost)  
**Time**: 2 minutes

**Steps**:
1. ✅ Create or update a room
2. ✅ Enter room name (`RoomName`): **"Deluxe"** (short text)
3. ✅ **Skip room description** (to save cost)
4. ✅ Save room
5. ✅ **Run SQL query to verify records**:

```sql
-- Simple verification query
SELECT RoomId, AccommodationId, MultiLanguageId, RoomName,
       CASE 
           WHEN MultiLanguageId = 1 THEN 'English'
           WHEN MultiLanguageId = 2 THEN 'Arabic'
           ELSE 'Other'
       END AS LanguageName
FROM BookingWhizz.dbo.Rooms_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY RoomId, MultiLanguageId;

-- Check record count per room
SELECT RoomId, COUNT(*) AS LanguageCount
FROM BookingWhizz.dbo.Rooms_ML 
WHERE AccommodationId = @YourPropertyId
GROUP BY RoomId;
```

**Expected Results**:
- ✅ **2 records per room** (if 2 languages enabled)
- ✅ English (MultiLanguageId = 1): "Deluxe" (original text)
- ✅ Arabic (MultiLanguageId = 2): Translated text in Arabic
- ✅ Each room has entries for all enabled languages
- ✅ No NULL values in `RoomName` column

---

### Test Case 28: RatePlans_ML Table Verification ⭐ SIMPLE CHECK

**Table**: `RatePlans_ML`  
**Field**: `RatePlanName` only (to reduce cost)  
**Time**: 2 minutes

**Steps**:
1. ✅ Create or update a rate plan
2. ✅ Enter rate plan name (`RatePlanName`): **"Standard"** (short text)
3. ✅ **Skip all other fields** (to save cost)
4. ✅ Save rate plan
5. ✅ **Run SQL query to verify records**:

```sql
-- Simple verification query
SELECT RatePlanId, AccommodationId, MultiLanguageId, RatePlanName,
       CASE 
           WHEN MultiLanguageId = 1 THEN 'English'
           WHEN MultiLanguageId = 2 THEN 'Arabic'
           ELSE 'Other'
       END AS LanguageName
FROM BookingWhizz.dbo.RatePlans_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY RatePlanId, MultiLanguageId;

-- Check record count per rate plan
SELECT RatePlanId, COUNT(*) AS LanguageCount
FROM BookingWhizz.dbo.RatePlans_ML 
WHERE AccommodationId = @YourPropertyId
GROUP BY RatePlanId;
```

**Expected Results**:
- ✅ **2 records per rate plan** (if 2 languages enabled)
- ✅ English (MultiLanguageId = 1): "Standard" (original text)
- ✅ Arabic (MultiLanguageId = 2): Translated text in Arabic
- ✅ Each rate plan has entries for all enabled languages
- ✅ No NULL values in `RatePlanName` column

---

### Test Case 29: Activities_ML Table Verification ⭐ SIMPLE CHECK

**Table**: `Activities_ML`  
**Field**: `AddonName` only (to reduce cost)  
**Time**: 2 minutes

**Steps**:
1. ✅ Create or update an addon
2. ✅ Enter addon name (`AddonName`): **"Spa"** (short text)
3. ✅ **Skip all other fields** (to save cost)
4. ✅ Save addon
5. ✅ **Run SQL query to verify records**:

```sql
-- Simple verification query
SELECT ActivityId, AccommodationId, MultiLanguageId, ActivityName,
       CASE 
           WHEN MultiLanguageId = 1 THEN 'English'
           WHEN MultiLanguageId = 2 THEN 'Arabic'
           ELSE 'Other'
       END AS LanguageName
FROM BookingWhizz.dbo.Activities_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY ActivityId, MultiLanguageId;

-- Check record count per addon
SELECT ActivityId, COUNT(*) AS LanguageCount
FROM BookingWhizz.dbo.Activities_ML 
WHERE AccommodationId = @YourPropertyId
GROUP BY ActivityId;
```

**Expected Results**:
- ✅ **2 records per addon** (if 2 languages enabled)
- ✅ English (MultiLanguageId = 1): "Spa" (original text)
- ✅ Arabic (MultiLanguageId = 2): Translated text in Arabic
- ✅ Each addon has entries for all enabled languages
- ✅ No NULL values in `ActivityName` column

---

### Test Case 30: Promotions_ML Table Verification ⭐ SIMPLE CHECK

**Table**: `Promotions_ML`  
**Field**: `PromoName` only (to reduce cost)  
**Time**: 2 minutes

**Steps**:
1. ✅ Create or update a promotion
2. ✅ Enter promotion name (`PromoName`): **"Summer Sale"** (short text)
3. ✅ **Skip description field** (to save cost)
4. ✅ Save promotion
5. ✅ **Run SQL query to verify records**:

```sql
-- Simple verification query
SELECT PromotionId, AccommodationId, MultiLanguageId, PromotionName,
       CASE 
           WHEN MultiLanguageId = 1 THEN 'English'
           WHEN MultiLanguageId = 2 THEN 'Arabic'
           ELSE 'Other'
       END AS LanguageName
FROM BookingWhizz.dbo.Promotions_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY PromotionId, MultiLanguageId;

-- Check record count per promotion
SELECT PromotionId, COUNT(*) AS LanguageCount
FROM BookingWhizz.dbo.Promotions_ML 
WHERE AccommodationId = @YourPropertyId
GROUP BY PromotionId;
```

**Expected Results**:
- ✅ **2 records per promotion** (if 2 languages enabled)
- ✅ English (MultiLanguageId = 1): "Summer Sale" (original text)
- ✅ Arabic (MultiLanguageId = 2): Translated text in Arabic
- ✅ Each promotion has entries for all enabled languages
- ✅ No NULL values in `PromotionName` column

---

### Test Case 31: AFGroups_ML Table Verification (Accommodations Groups)

**Table**: `AFGroups_ML`  
**Fields**: `GroupName`, `OtherGroupName`  
**Time**: 3 minutes

**Steps**:
1. ✅ Go to `/Property/PropertyFacilities`
2. ✅ Add a new facility group
3. ✅ Enter group name (`GroupName`)
4. ✅ Enter other group name (`OtherGroupName`) - optional
5. ✅ Save group
6. ✅ Run SQL query to verify:

```sql
SELECT OwnerId, MultiLanguageId, GroupId, GroupName, OtherGroupName
FROM BookingWhizz.dbo.AFGroups_ML 
WHERE OwnerId = @YourOwnerId
ORDER BY GroupId, MultiLanguageId;
```

**Expected Results**:
- ✅ Group is created in main table (`AFGroups`)
- ✅ Translation runs for all enabled languages
- ✅ Entries created in `AFGroups_ML` for all enabled languages
- ✅ `GroupName` is translated
- ✅ `OtherGroupName` is translated (if provided)

**Bulk Translation Test**:
1. ✅ Go to `/Property/PropertyFacilities`
2. ✅ Click "Bulk Translate Accommodation Groups" button (if available)
3. ✅ Verify all groups are translated
4. ✅ Run SQL query to verify all groups have ML entries

---

### Test Case 32: AccommodationsFacilities_ML Table Verification (Accommodations Facilities)

**Table**: `AccommodationsFacilities_ML`  
**Fields**: `FacilityName`, `OtherFacilityName`  
**Time**: 3 minutes

**Steps**:
1. ✅ Go to `/Property/PropertyFacilities`
2. ✅ Add a new facility
3. ✅ Enter facility name (`FacilityName`)
4. ✅ Enter other facility name (`OtherFacilityName`) - optional
5. ✅ Save facility
6. ✅ Run SQL query to verify:

```sql
SELECT OwnerId, MultiLanguageId, GroupId, FacilityId, 
       FacilityName, OtherFacilityName
FROM BookingWhizz.dbo.AccommodationsFacilities_ML 
WHERE OwnerId = @YourOwnerId
ORDER BY FacilityId, MultiLanguageId;
```

**Expected Results**:
- ✅ Facility is created in main table (`AccommodationsFacilities`)
- ✅ Translation runs for all enabled languages
- ✅ Entries created in `AccommodationsFacilities_ML` for all enabled languages
- ✅ `FacilityName` is translated
- ✅ `OtherFacilityName` is translated (if provided)

**Bulk Translation Test**:
1. ✅ Go to `/Property/PropertyFacilities`
2. ✅ Click "Bulk Translate Accommodation Facilities" button (if available)
3. ✅ Verify all facilities are translated
4. ✅ Run SQL query to verify all facilities have ML entries

---

### Test Case 33: RFGroups_ML Table Verification (Room Groups)

**Table**: `RFGroups_ML`  
**Fields**: `GroupName`, `OtherGroupName`  
**Time**: 3 minutes

**Steps**:
1. ✅ Go to `/RoomManagement/RoomFacilities`
2. ✅ Add a new room facility group
3. ✅ Enter group name (`GroupName`)
4. ✅ Enter other group name (`OtherGroupName`) - optional
5. ✅ Save group
6. ✅ Run SQL query to verify:

```sql
SELECT OwnerId, MultiLanguageId, GroupId, GroupName, OtherGroupName
FROM BookingWhizz.dbo.RFGroups_ML 
WHERE OwnerId = @YourOwnerId
ORDER BY GroupId, MultiLanguageId;
```

**Expected Results**:
- ✅ Group is created in main table (`RFGroups`)
- ✅ Translation runs for all enabled languages
- ✅ Entries created in `RFGroups_ML` for all enabled languages
- ✅ `GroupName` is translated
- ✅ `OtherGroupName` is translated (if provided)

**Bulk Translation Test**:
1. ✅ Go to `/RoomManagement/RoomFacilities`
2. ✅ Click "Bulk Translate Room Groups" button
3. ✅ Verify all groups are translated
4. ✅ Run SQL query to verify all groups have ML entries

---

### Test Case 34: RoomFacilities_ML Table Verification (Room Facilities)

**Table**: `RoomFacilities_ML`  
**Fields**: `FacilityName`, `OtherFacilityName`  
**Time**: 3 minutes

**Steps**:
1. ✅ Go to `/RoomManagement/RoomFacilities`
2. ✅ Add a new room facility
3. ✅ Enter facility name (`FacilityName`)
4. ✅ Enter other facility name (`OtherFacilityName`) - optional
5. ✅ Save facility
6. ✅ Run SQL query to verify:

```sql
SELECT OwnerId, MultiLanguageId, GroupId, FacilityId, 
       FacilityName, OtherFacilityName
FROM BookingWhizz.dbo.RoomFacilities_ML 
WHERE OwnerId = @YourOwnerId
ORDER BY FacilityId, MultiLanguageId;
```

**Expected Results**:
- ✅ Facility is created in main table (`RoomFacilities`)
- ✅ Translation runs for all enabled languages
- ✅ Entries created in `RoomFacilities_ML` for all enabled languages
- ✅ `FacilityName` is translated
- ✅ `OtherFacilityName` is translated (if provided)

**Bulk Translation Test**:
1. ✅ Go to `/RoomManagement/RoomFacilities`
2. ✅ Click "Bulk Translate Room Facilities" button
3. ✅ Verify all facilities are translated
4. ✅ Run SQL query to verify all facilities have ML entries

---

### Test Case 35: Complete ML Tables Verification (All Tables) ⭐ QUICK CHECK

**Objective**: Verify all ML tables have correct data for a property

**Time**: 5 minutes  
**API Cost**: Already tested (no new API calls)

**Steps**:
1. ✅ Run **one combined SQL query** to check all tables at once:

```sql
-- Quick verification - All tables count
SELECT 'Accommodations_ML' AS TableName, COUNT(*) AS RecordCount,
       COUNT(CASE WHEN MultiLanguageId = 1 THEN 1 END) AS EnglishCount,
       COUNT(CASE WHEN MultiLanguageId = 2 THEN 1 END) AS ArabicCount
FROM BookingWhizz.dbo.Accommodations_ML 
WHERE AccommodationId = @YourPropertyId

UNION ALL

SELECT 'Rooms_ML', COUNT(*),
       COUNT(CASE WHEN MultiLanguageId = 1 THEN 1 END),
       COUNT(CASE WHEN MultiLanguageId = 2 THEN 1 END)
FROM BookingWhizz.dbo.Rooms_ML 
WHERE AccommodationId = @YourPropertyId

UNION ALL

SELECT 'RatePlans_ML', COUNT(*),
       COUNT(CASE WHEN MultiLanguageId = 1 THEN 1 END),
       COUNT(CASE WHEN MultiLanguageId = 2 THEN 1 END)
FROM BookingWhizz.dbo.RatePlans_ML 
WHERE AccommodationId = @YourPropertyId

UNION ALL

SELECT 'Activities_ML', COUNT(*),
       COUNT(CASE WHEN MultiLanguageId = 1 THEN 1 END),
       COUNT(CASE WHEN MultiLanguageId = 2 THEN 1 END)
FROM BookingWhizz.dbo.Activities_ML 
WHERE AccommodationId = @YourPropertyId

UNION ALL

SELECT 'Promotions_ML', COUNT(*),
       COUNT(CASE WHEN MultiLanguageId = 1 THEN 1 END),
       COUNT(CASE WHEN MultiLanguageId = 2 THEN 1 END)
FROM BookingWhizz.dbo.Promotions_ML 
WHERE AccommodationId = @YourPropertyId;
```

2. ✅ **Verify each table**:
   - RecordCount should be **2 × number of items** (if 2 languages enabled)
   - EnglishCount should match number of items
   - ArabicCount should match number of items

**Expected Results**:
- ✅ All 5 main ML tables have records
- ✅ Each item has **2 records** (English + Arabic)
- ✅ English (MultiLanguageId = 1) exists in all tables
- ✅ Arabic (MultiLanguageId = 2) exists in all tables
- ✅ No missing translations

**Example Output**:
```
TableName           | RecordCount | EnglishCount | ArabicCount
Accommodations_ML   | 2           | 1            | 1
Rooms_ML            | 4           | 2            | 2
RatePlans_ML        | 6           | 3            | 3
Activities_ML       | 4           | 2            | 2
Promotions_ML       | 2           | 1            | 1
```

---

## Translation History Testing

### Test Case 36: Translation History Display

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

### Test Case 37: Translation History Filtering

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

### Test Case 38: Translation History Search

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

### Test Case 39: Translation History Pagination

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

### Test Case 40: Translation History Export (CSV)

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

### Test Case 41: Translation History Export (Excel)

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

### Test Case 42: Revert Translation

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

---

## ✅ Quick Test Checklist

### ⚡ 5-Minute Smoke Test (Critical Path Only)

**Use this for quick verification after deployment:**

- [ ] Language dropdown works (select Arabic)
- [ ] Master switch toggle works (UpdateProperty page)
- [ ] Property name translation works (change to "Hotel ABC")
- [ ] SQL check: 2 records in `Accommodations_ML` table
- [ ] Translation History shows 1 entry

**SQL**:
```sql
SELECT AccommodationId, MultiLanguageId, AccommodationName
FROM BookingWhizz.dbo.Accommodations_ML 
WHERE AccommodationId = @YourPropertyId
ORDER BY MultiLanguageId;
```

**API Cost**: 2 calls only

---

### 🎯 15-Minute Quick Test (All Modules)

**Use this for weekly regression testing:**

- [ ] **Setup** (3 min):
  - [ ] Enable master switch
  - [ ] Enable 2 languages (English + Arabic)
  
- [ ] **Translation Tests** (10 min):
  - [ ] Property: "Hotel ABC"
  - [ ] Addon: "Spa"
  - [ ] Room: "Deluxe"
  - [ ] Rate Plan: "Standard"
  - [ ] Promotion: "Summer Sale"

- [ ] **Verification** (2 min):
  - [ ] Run combined SQL query (see Phase 5.1 above)
  - [ ] Check Translation History (should show 5 entries)

**API Cost**: 10 calls (5 fields × 2 languages)

---

### 📊 API Cost Calculation Guide

**Formula**: `API Calls = Number of Fields × Number of Enabled Languages`

**Examples**:

1. **Property Name Update** (`PropName`):
   - Fields: 1
   - Enabled Languages: 3 (English, Arabic, Turkish)
   - **API Calls**: 1 × 3 = **3 calls**

2. **Addon Creation** (`AddonName`, `ShortDesc`, `LongDesc`):
   - Fields: 3
   - Enabled Languages: 3
   - **API Calls**: 3 × 3 = **9 calls**

3. **Room Creation** (`RoomName`, `RoomDesc`):
   - Fields: 2
   - Enabled Languages: 5
   - **API Calls**: 2 × 5 = **10 calls**

**Cost Optimization Tips**:
- ✅ Test High Impact fields first (marked with ⭐ High)
- ✅ Disable unnecessary languages before testing
- ✅ Use batch updates (multiple fields in one save)
- ✅ Monitor Translation History for actual API usage

---

### 🎯 Priority Testing Order

**Test in this order for maximum efficiency**:

1. **Priority 1** (Must Test Daily):
   - `PropName` - Most used field
   - Master Switch toggle
   - Language selection

2. **Priority 2** (Test Weekly):
   - `AddonName`, `ShortDesc`, `LongDesc`
   - `RoomName`, `RoomDesc`
   - `RatePlanName`, `DisplayName`

3. **Priority 3** (Test Monthly):
   - `PromoName`, `PromoDesc`
   - `KeyComments`, `PetsComments`
   - Facilities fields

---

### 📝 Test Execution Log Template

| Test # | Test Case | Table/Field | Status | API Calls | Notes |
|--------|-----------|-------------|--------|-----------|-------|
| TC-15 | Property Name Translation | `PropName` | ✅ Pass | 3 | - |
| TC-16 | Addon Translation | `AddonName` | ✅ Pass | 9 | - |
| TC-17 | Room Translation | `RoomName` | ✅ Pass | 6 | - |
| TC-26 | Accommodations_ML | `Accommodations_ML` | ✅ Pass | - | SQL verified |
| TC-27 | Rooms_ML | `Rooms_ML` | ✅ Pass | - | SQL verified |
| TC-28 | RatePlans_ML | `RatePlans_ML` | ✅ Pass | - | SQL verified |
| TC-29 | Activities_ML | `Activities_ML` | ✅ Pass | - | SQL verified |
| TC-30 | Promotions_ML | `Promotions_ML` | ✅ Pass | - | SQL verified |
| TC-31 | AFGroups_ML | `AFGroups_ML` | ✅ Pass | - | SQL verified |
| TC-32 | AccommodationsFacilities_ML | `AccommodationsFacilities_ML` | ✅ Pass | - | SQL verified |
| TC-33 | RFGroups_ML | `RFGroups_ML` | ✅ Pass | - | SQL verified |
| TC-34 | RoomFacilities_ML | `RoomFacilities_ML` | ✅ Pass | - | SQL verified |
| TC-35 | Complete ML Tables | All 9 tables | ✅ Pass | - | All verified |
| ... | ... | ... | ... | ... | ... |

**Status Legend**:
- ✅ Pass
- ❌ Fail
- ⏳ Pending
- ⚠️ Blocked

---

---

## 📊 Test Summary

### Time Breakdown

| Phase | Description | Time |
|-------|-------------|------|
| Phase 1 | Setup & Configuration | 10 min |
| Phase 2 | Translation Testing | 30 min |
| Phase 3 | Edge Cases & Validation | 15 min |
| Phase 4 | Bulk Translation | 5 min |
| Phase 5 | Final Verification | 5 min |
| **Total** | **Complete System Test** | **~60 min** |

### API Cost Summary

| Module | Fields Tested | Languages | API Calls |
|--------|---------------|-----------|-----------|
| Property | 1 (PropName) | 2 (EN+AR) | 2 |
| Addon | 1 (AddonName) | 2 (EN+AR) | 2 |
| Room | 1 (RoomName) | 2 (EN+AR) | 2 |
| Rate Plan | 1 (RatePlanName) | 2 (EN+AR) | 2 |
| Promotion | 1 (PromoName) | 2 (EN+AR) | 2 |
| **Total** | **5 fields** | **2 languages** | **~10 calls** |

*Note: Additional calls may occur for edge case testing*

---

## 🚨 Critical Issues to Watch For

1. **English Language Disabled**: English should ALWAYS be enabled
2. **Translation Skipped When It Should Run**: Check master switch and per-language settings
3. **Translation Runs When It Shouldn't**: Check field change detection
4. **Missing ML Table Records**: Verify SQL queries return expected counts
5. **API Errors**: Check Translation History for failed translations

---

**Document Version**: 4.0  
**Last Updated**: 2025  
**Target Time**: 1 Hour  
**Status**: Production Ready ✅

**Happy Testing! 🧪**

