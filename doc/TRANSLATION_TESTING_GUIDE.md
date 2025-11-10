# 🧪 Translation System Testing Guide

**Complete Testing Guide for Multi-Language Translation System**

Yeh guide aapko step-by-step batayega ke translation system ko kaise test karna hai.

---

## 📋 Table of Contents

1. [Prerequisites / Setup](#prerequisites--setup)
2. [Module-wise Testing](#module-wise-testing)
3. [Translation History Testing](#translation-history-testing)
4. [Translation Analytics Testing](#translation-analytics-testing)
5. [Per-Language Settings Testing](#per-language-settings-testing)
6. [Bulk Translation Testing](#bulk-translation-testing)
7. [Language Selector Testing](#language-selector-testing)
8. [Revert Translation Testing](#revert-translation-testing)
9. [Common Issues & Troubleshooting](#common-issues--troubleshooting)

---

## Prerequisites / Setup

### 1. Database Setup

**Check karein:**
- ✅ `MultiLanguages` table mein languages available hain (English, Arabic, Turkish minimum)
- ✅ Property ke liye `AutoTranslateEnabled = 1` aur `EnableForProperty = 1` set hai
- ✅ `TranslationHistory` table exist karta hai
- ✅ DeepL API key `appsettings.json` mein configured hai

**SQL Query to Check:**
```sql
-- Check languages
SELECT * FROM BookingWhizz.dbo.MultiLanguages WHERE StatusId = 1;

-- Check property settings
SELECT AccommodationId, AccommodationName, AutoTranslateEnabled, EnableForProperty 
FROM BookingWhizz.dbo.Accommodations 
WHERE AccommodationId = YOUR_PROPERTY_ID;
```

### 2. Configuration Check

**File**: `appsettings.json`

**Check karein:**
```json
{
  "TranslationSettings": {
    "DeepLApiKey": "YOUR_DEEPL_API_KEY",
    "DeepLApiUrl": "https://api-free.deepl.com/v2/translate"
  }
}
```

### 3. User Claims Check

**Verify karein:**
- User ke claims mein `AutoTranslateEnabled` aur `EnableForProperty` values set hain
- `LanguageId` claim properly set hai
- `PropertyId` claim set hai

**Browser Console mein check karein:**
```javascript
// Check cookies
document.cookie

// Check if language cookie exists
// Should see: SelectedLanguageId, SelectedLanguageName, SelectedLanguageCode
```

---

## Module-wise Testing

### 1. Property Module Testing

#### Test Case 1.1: Create Property with Translation

**Steps:**
1. Login karein aur property create page par jayein: `/Property/CreateProperty`
2. Property details fill karein:
   - **AccommodationName**: "Grand Hotel"
   - **KeyCollectionComments**: "Check-in time is 2 PM"
   - **PetsAllowedComments**: "Pets are welcome"
3. Form submit karein
4. Success message check karein

**Expected Results:**
- ✅ Property successfully create ho
- ✅ Translation History mein entries add hon
- ✅ `Accommodations_ML` table mein translated data save ho
- ✅ Console mein translation logs dikhen

**Verification:**
```sql
-- Check ML table
SELECT * FROM BookingWhizz.dbo.Accommodations_ML 
WHERE AccommodationId = YOUR_PROPERTY_ID;

-- Check Translation History
SELECT * FROM BookingWhizz.dbo.TranslationHistory 
WHERE TableName = 'Accommodations' 
  AND RecordId = YOUR_PROPERTY_ID;
```

#### Test Case 1.2: Update Property with Translation

**Steps:**
1. Property update page par jayein: `/Property/UpdateProperty`
2. Existing property select karein
3. **AccommodationName** change karein: "Grand Hotel" → "Luxury Grand Hotel"
4. Form submit karein

**Expected Results:**
- ✅ Property update ho
- ✅ Sirf changed fields ka translation ho (field change detection)
- ✅ Translation History mein new entry add ho
- ✅ ML table update ho

**Verification:**
- Translation History page par jayein aur check karein ke sirf changed field ka translation hua hai

#### Test Case 1.3: Update Property without Translation (Non-translatable fields)

**Steps:**
1. Property update page par jayein
2. Sirf non-translatable fields change karein (e.g., Address, Phone)
3. Form submit karein

**Expected Results:**
- ✅ Property update ho
- ✅ Translation trigger NAHI hona chahiye
- ✅ Console logs mein "Translation skipped - no translatable fields changed" dikhe

---

### 2. Addon Module Testing

#### Test Case 2.1: Create Addon with Translation

**Steps:**
1. Addon create page par jayein: `/Property/CreateAddon`
2. Addon details fill karein:
   - **ActivityName**: "Swimming Pool Access"
   - **ShortDescription**: "Enjoy our beautiful pool"
   - **LongDescription**: "Our pool is open 24/7"
   - **CancellationPolicy**: "No cancellation fee"
   - **GuaranteePolicy**: "Credit card required"
3. Form submit karein

**Expected Results:**
- ✅ Addon create ho
- ✅ All translatable fields ka translation ho
- ✅ Translation History mein entries add hon

**Verification:**
```sql
-- Check Addons_ML table
SELECT * FROM BookingWhizz.dbo.Addons_ML 
WHERE AddonId = YOUR_ADDON_ID;
```

#### Test Case 2.2: Update Addon with Partial Field Change

**Steps:**
1. Addon update page par jayein: `/Property/UpdateAddon`
2. Sirf **ActivityName** change karein
3. Form submit karein

**Expected Results:**
- ✅ Sirf ActivityName ka translation ho
- ✅ Other fields ka translation skip ho

---

### 3. Room Module Testing

#### Test Case 3.1: Create Room with Translation

**Steps:**
1. Room create page par jayein: `/RoomManagement/CreateRoom`
2. Room details fill karein:
   - **RoomName**: "Deluxe Suite" (if RoomTypeId = 1)
   - **ApartmentName**: "Luxury Apartment" (if RoomTypeId = 2)
   - **RoomDescription**: "Spacious room with sea view"
3. Form submit karein

**Expected Results:**
- ✅ Room create ho
- ✅ Translation parallel processing se ho (Task.WhenAll)
- ✅ Translation History mein entries add hon

**Verification:**
```sql
-- Check Rooms_ML table
SELECT * FROM BookingWhizz.dbo.Rooms_ML 
WHERE RoomId = YOUR_ROOM_ID;
```

#### Test Case 3.2: Update Room with Translation

**Steps:**
1. Room update page par jayein: `/RoomManagement/UpdateRoom`
2. **RoomDescription** change karein
3. Form submit karein

**Expected Results:**
- ✅ Room update ho
- ✅ Field change detection properly kaam kare
- ✅ Translation History update ho

---

### 4. Rate Plan Module Testing

#### Test Case 4.1: Create Rate Plan with Translation

**Steps:**
1. Rate Plan create page par jayein: `/RoomManagement/CreateRatePlan`
2. Rate Plan details fill karein:
   - **RatePlanName**: "Standard Rate"
   - **DisplayRatePlanName**: "Best Value"
   - **Included**: "Breakfast included"
   - **Highlight**: "Free WiFi"
   - **MealDescription**: "Continental breakfast"
3. Form submit karein

**Expected Results:**
- ✅ Rate Plan create ho
- ✅ All 5 translatable fields ka translation ho
- ✅ Parallel translation processing ho

**Verification:**
```sql
-- Check RatePlans_ML table
SELECT * FROM BookingWhizz.dbo.RatePlans_ML 
WHERE RatePlanId = YOUR_RATE_PLAN_ID;
```

#### Test Case 4.2: Update Rate Plan with Multiple Field Changes

**Steps:**
1. Rate Plan update page par jayein: `/RoomManagement/UpdateRatePlan`
2. Multiple fields change karein:
   - **RatePlanName**: "Premium Rate"
   - **Highlight**: "Free WiFi and Parking"
3. Form submit karein

**Expected Results:**
- ✅ Both fields ka translation ho
- ✅ Translation History mein 2 entries add hon

---

### 5. Promotion Module Testing

#### Test Case 5.1: Create Promotion with Translation

**Steps:**
1. Promotion create page par jayein: `/PromotionManagement/CreatePromotion`
2. Promotion details fill karein:
   - **PromotionName**: "Summer Sale"
   - **Description**: "Get 20% off on all bookings"
3. Form submit karein

**Expected Results:**
- ✅ Promotion create ho
- ✅ Translation ho
- ✅ Translation History mein entries add hon

**Verification:**
```sql
-- Check Promotions_ML table
SELECT * FROM BookingWhizz.dbo.Promotions_ML 
WHERE PromotionId = YOUR_PROMOTION_ID;
```

---

### 6. Room Facilities Module Testing

#### Test Case 6.1: Create Room Group with Translation

**Steps:**
1. Room Facilities page par jayein: `/RoomManagement/RoomFacilities?roomId=1&roomName=Test`
2. New Group add karein:
   - **GroupName**: "Bathroom Facilities"
3. Submit karein

**Expected Results:**
- ✅ Group create ho
- ✅ Translation ho
- ✅ Translation History mein entry add ho

#### Test Case 6.2: Create Room Facility with Translation

**Steps:**
1. Room Facilities page par jayein
2. New Facility add karein:
   - **FacilityName**: "Hair Dryer"
3. Submit karein

**Expected Results:**
- ✅ Facility create ho
- ✅ Translation ho

---

## Translation History Testing

### Test Case: View Translation History

**Steps:**
1. Translation History page par jayein: `/Property/TranslationHistory`
2. Page load check karein

**Expected Results:**
- ✅ Table properly load ho
- ✅ All translations dikhen
- ✅ Filters (Language, Field Name, Date Range) available hon
- ✅ Pagination kaam kare

### Test Case: Filter by Language

**Steps:**
1. Translation History page par jayein
2. Language dropdown se "Arabic" select karein
3. Filter apply karein

**Expected Results:**
- ✅ Sirf Arabic translations dikhen
- ✅ Table properly update ho

### Test Case: Filter by Field Name

**Steps:**
1. Translation History page par jayein
2. Field Name dropdown se "AccommodationName" select karein
3. Filter apply karein

**Expected Results:**
- ✅ Sirf AccommodationName translations dikhen

### Test Case: Date Range Filter

**Steps:**
1. Translation History page par jayein
2. Start Date aur End Date select karein
3. Filter apply karein

**Expected Results:**
- ✅ Sirf selected date range ke translations dikhen

### Test Case: Search Functionality

**Steps:**
1. Translation History page par jayein
2. Search box mein text enter karein (e.g., "Hotel")
3. Search button click karein

**Expected Results:**
- ✅ Original text ya translated text mein "Hotel" wale results dikhen

### Test Case: Export to CSV

**Steps:**
1. Translation History page par jayein
2. Filters apply karein (optional)
3. "Export" dropdown se "Export as CSV" select karein

**Expected Results:**
- ✅ CSV file download ho
- ✅ File mein filtered data ho
- ✅ Excel mein properly open ho

### Test Case: Export to Excel

**Steps:**
1. Translation History page par jayein
2. "Export" dropdown se "Export as Excel" select karein

**Expected Results:**
- ✅ Excel file download ho
- ✅ Data properly formatted ho

---

## Translation Analytics Testing

### Test Case: View Analytics Dashboard

**Steps:**
1. Translation Analytics page par jayein: `/Property/TranslationAnalytics`
2. Page load check karein

**Expected Results:**
- ✅ 4 summary cards dikhen:
  - Total Translations
  - Auto Translations
  - Manual Translations
  - API Usage Cost
- ✅ Charts properly render hon
- ✅ Statistics table load ho

### Test Case: Verify Statistics

**Steps:**
1. Analytics dashboard par jayein
2. Numbers verify karein

**Expected Results:**
- ✅ Total Translations = Auto + Manual
- ✅ API Usage Cost properly calculate ho
- ✅ Charts accurate data show karein

**Verification:**
```sql
-- Manual count check
SELECT COUNT(*) FROM BookingWhizz.dbo.TranslationHistory 
WHERE AccommodationId = YOUR_PROPERTY_ID AND IsActive = 1;
```

---

## Per-Language Settings Testing

### Test Case: View Per-Language Settings

**Steps:**
1. Per-Language Settings page par jayein: `/Property/PerLanguageSettings`
2. Page load check karein

**Expected Results:**
- ✅ All enabled languages ki list dikhe
- ✅ Each language ke liye toggle switch ho
- ✅ Current status properly show ho

### Test Case: Enable Translation for Language

**Steps:**
1. Per-Language Settings page par jayein
2. Kisi language ke liye toggle ON karein (e.g., Arabic)
3. Save karein

**Expected Results:**
- ✅ Settings save hon
- ✅ Success message dikhe
- ✅ Next translation mein Arabic include ho

**Verification:**
- Property create/update karein aur check karein ke Arabic translation ho

### Test Case: Disable Translation for Language

**Steps:**
1. Per-Language Settings page par jayein
2. Kisi language ke liye toggle OFF karein
3. Save karein

**Expected Results:**
- ✅ Settings save hon
- ✅ Next translation mein wo language skip ho

---

## Bulk Translation Testing

### Test Case: Bulk Translate Room Groups

**Steps:**
1. Room Facilities page par jayein
2. "Bulk Translate Room Groups" button click karein (agar available hai)
   Ya direct API call karein:
   ```javascript
   $.post('/RoomManagement/BulkTranslateRoomGroups', function(response) {
       console.log(response);
   });
   ```

**Expected Results:**
- ✅ Success message dikhe
- ✅ Message mein count dikhe: "Bulk translation completed. Success: X, Skipped (duplicates): Y"
- ✅ Translation History mein entries add hon

**Verification:**
```sql
-- Check RFGroups_ML table
SELECT * FROM BookingWhizz.dbo.RFGroups_ML 
WHERE OwnerId = YOUR_OWNER_ID;
```

### Test Case: Bulk Translate Room Facilities

**Steps:**
1. Room Facilities page par jayein
2. "Bulk Translate Room Facilities" button click karein
   Ya direct API call karein:
   ```javascript
   $.post('/RoomManagement/BulkTranslateRoomFacilities', function(response) {
       console.log(response);
   });
   ```

**Expected Results:**
- ✅ Success message dikhe
- ✅ All facilities translate hon
- ✅ Duplicates skip hon

**Verification:**
```sql
-- Check RoomFacilities_ML table
SELECT * FROM BookingWhizz.dbo.RoomFacilities_ML 
WHERE OwnerId = YOUR_OWNER_ID;
```

---

## Language Selector Testing

### Test Case: Language Dropdown Display

**Steps:**
1. Kisi bhi page par jayein
2. Header mein language dropdown check karein

**Expected Results:**
- ✅ Dropdown properly dikhe
- ✅ Current language selected ho
- ✅ All available languages dikhen

### Test Case: Change Language

**Steps:**
1. Header mein language dropdown se different language select karein (e.g., Arabic)
2. Page reload check karein

**Expected Results:**
- ✅ Language change ho
- ✅ Cookie set ho
- ✅ Page data selected language mein dikhe
- ✅ Next request mein selected language use ho

**Verification:**
```javascript
// Browser console mein
document.cookie
// Should see: SelectedLanguageId=3; SelectedLanguageName=Arabic; etc.
```

### Test Case: Language Persistence

**Steps:**
1. Language change karein
2. Browser close karein
3. Browser open karein aur login karein

**Expected Results:**
- ✅ Previously selected language automatically load ho
- ✅ Cookie properly persist ho

---

## Revert Translation Testing

### Test Case: Revert Translation from History

**Steps:**
1. Translation History page par jayein: `/Property/TranslationHistory`
2. Kisi translation record ke liye "Revert" button click karein
3. Confirmation dialog accept karein

**Expected Results:**
- ✅ Success message dikhe
- ✅ Translation revert ho
- ✅ ML table update ho
- ✅ New revert entry Translation History mein add ho

**Verification:**
```sql
-- Check ML table before and after revert
SELECT * FROM BookingWhizz.dbo.Accommodations_ML 
WHERE AccommodationId = YOUR_PROPERTY_ID AND MultiLanguageId = LANGUAGE_ID;

-- Check Translation History
SELECT * FROM BookingWhizz.dbo.TranslationHistory 
WHERE TableName = 'Accommodations' 
  AND RecordId = YOUR_PROPERTY_ID
ORDER BY TranslationDate DESC;
```

---

## Common Issues & Troubleshooting

### Issue 1: Translation NAHI ho raha

**Symptoms:**
- Property/Room create/update ho raha hai but translation NAHI ho raha

**Check Points:**
1. ✅ Property settings check karein:
   ```sql
   SELECT AutoTranslateEnabled, EnableForProperty 
   FROM BookingWhizz.dbo.Accommodations 
   WHERE AccommodationId = YOUR_PROPERTY_ID;
   ```
   - Dono `1` hona chahiye

2. ✅ User claims check karein:
   - Browser console mein check karein
   - `AutoTranslateEnabled` aur `EnableForProperty` claims set hon

3. ✅ DeepL API key check karein:
   - `appsettings.json` mein API key valid hai
   - API key active hai

4. ✅ Logs check karein:
   - Application logs mein translation-related errors dikhen
   - Console logs check karein

**Solution:**
- Property settings update karein
- API key verify karein
- Logs check karein aur errors fix karein

---

### Issue 2: Translation History NAHI dikh raha

**Symptoms:**
- Translation ho raha hai but Translation History page par data NAHI dikh raha

**Check Points:**
1. ✅ `TranslationHistory` table check karein:
   ```sql
   SELECT * FROM BookingWhizz.dbo.TranslationHistory 
   WHERE AccommodationId = YOUR_PROPERTY_ID;
   ```

2. ✅ Property ID match check karein:
   - User ke claims mein `PropertyId` correct hai

**Solution:**
- Database directly check karein
- Property ID verify karein

---

### Issue 3: Language Selector NAHI dikh raha

**Symptoms:**
- Header mein language dropdown NAHI dikh raha

**Check Points:**
1. ✅ `_Header.cshtml` check karein:
   - Language selector code present hai

2. ✅ JavaScript file check karein:
   - `wwwroot/js/language-selector.js` exist karta hai

3. ✅ API endpoint check karein:
   - `/Home/GetLanguages` properly kaam kar raha hai

**Solution:**
- Header view check karein
- JavaScript file verify karein
- API endpoint test karein

---

### Issue 4: Parallel Translation Slow hai

**Symptoms:**
- Translation ho raha hai but slow hai

**Check Points:**
1. ✅ `Task.WhenAll` properly use ho raha hai
2. ✅ DeepL API response time check karein
3. ✅ Network issues check karein

**Solution:**
- Code review karein ke parallel processing properly implement hai
- API response time check karein

---

### Issue 5: Duplicate Translations

**Symptoms:**
- Same translation multiple times ho raha hai

**Check Points:**
1. ✅ Field change detection properly kaam kar raha hai
2. ✅ Update operations mein old model properly pass ho raha hai

**Solution:**
- Field change detection logic verify karein
- Old model comparison check karein

---

## Testing Checklist

### Pre-Testing Checklist

- [ ] Database properly configured hai
- [ ] DeepL API key valid hai
- [ ] Property settings enabled hain
- [ ] Languages database mein available hain
- [ ] User properly logged in hai

### Module Testing Checklist

- [ ] Property Create/Update translation
- [ ] Addon Create/Update translation
- [ ] Room Create/Update translation
- [ ] Rate Plan Create/Update translation
- [ ] Promotion Create/Update translation
- [ ] Room Facilities Create translation
- [ ] Bulk Translation (Room Groups)
- [ ] Bulk Translation (Room Facilities)

### Feature Testing Checklist

- [ ] Translation History view
- [ ] Translation History filters (Language, Field, Date, Search)
- [ ] Translation History export (CSV, Excel)
- [ ] Translation Analytics dashboard
- [ ] Per-Language Settings
- [ ] Language Selector
- [ ] Revert Translation
- [ ] Field Change Detection
- [ ] Parallel Translation Processing

### Post-Testing Checklist

- [ ] All translations properly save ho rahe hain
- [ ] Translation History complete hai
- [ ] Analytics accurate hain
- [ ] No errors in logs
- [ ] Performance acceptable hai

---

## Quick Test Commands

### Browser Console Commands

```javascript
// Check language cookie
document.cookie

// Check if language selector loaded
$('#languageSelector').length

// Manually trigger language change
$('#languageSelector').val(3).trigger('change');

// Check translation history API
$.get('/Property/TranslationHistory', function(data) {
    console.log(data);
});
```

### SQL Verification Queries

```sql
-- Check property translation settings
SELECT AccommodationId, AccommodationName, AutoTranslateEnabled, EnableForProperty 
FROM BookingWhizz.dbo.Accommodations 
WHERE AccommodationId = YOUR_PROPERTY_ID;

-- Check recent translations
SELECT TOP 10 * FROM BookingWhizz.dbo.TranslationHistory 
WHERE AccommodationId = YOUR_PROPERTY_ID 
ORDER BY TranslationDate DESC;

-- Check ML table data
SELECT * FROM BookingWhizz.dbo.Accommodations_ML 
WHERE AccommodationId = YOUR_PROPERTY_ID;

-- Check enabled languages for property
SELECT * FROM BookingWhizz.dbo.PropertyLanguageSettings 
WHERE AccommodationId = YOUR_PROPERTY_ID AND IsEnabled = 1;
```

---

## Performance Testing

### Test Case: Parallel Translation Performance

**Steps:**
1. Property create karein with 3+ enabled languages
2. Translation time measure karein

**Expected Results:**
- ✅ All languages ka translation parallel ho
- ✅ Total time sequential se kam ho
- ✅ Console logs mein parallel processing dikhe

### Test Case: Bulk Translation Performance

**Steps:**
1. 50+ room groups create karein
2. Bulk translation trigger karein
3. Time measure karein

**Expected Results:**
- ✅ All groups translate hon
- ✅ Duplicates properly skip hon
- ✅ Performance acceptable ho

---

## Security Testing

### Test Case: Unauthorized Access

**Steps:**
1. Translation History page directly access karein without login
2. Translation API endpoints directly call karein

**Expected Results:**
- ✅ Unauthorized access block ho
- ✅ Redirect to login page

### Test Case: Property Isolation

**Steps:**
1. Property A ke user se login karein
2. Property B ka translation history access karne ki koshish karein

**Expected Results:**
- ✅ Sirf Property A ka data dikhe
- ✅ Property B ka data NAHI dikhe

---

## Conclusion

Yeh testing guide complete hai. Agar koi issue aaye to:

1. **Logs check karein**: Application logs aur browser console
2. **Database verify karein**: Direct SQL queries se data check karein
3. **API endpoints test karein**: Postman ya browser console se
4. **Configuration verify karein**: appsettings.json aur database settings

**Happy Testing! 🎉**

