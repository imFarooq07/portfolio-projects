# 🌐 Language System Support Guide

**Complete Guide for Support Team - How Language & Translation System Works**

This guide explains how the language system works, which fields are translated, and how to manage language settings for customers.

**Version**: 3.0  
**Last Updated**: 2025  
**For**: Support Team

---

## 📋 Table of Contents

1. [Overview - Language System Kya Hai?](#overview---language-system-kya-hai)
2. [Language Selection - User Kaise Language Change Kare?](#language-selection---user-kaise-language-change-kare)
3. [Translation Fields - Konsa Fields Translate Hote Hain?](#translation-fields---konsa-fields-translate-hote-hain)
4. [Master Switch - Translation Enable/Disable Kaise Kare?](#master-switch---translation-enabledisable-kaise-kare)
5. [Per-Language Settings - Specific Language Enable/Disable](#per-language-settings---specific-language-enabledisable)
6. [Translation Management Pages - Konsa Pages Use Kare?](#translation-management-pages---konsa-pages-use-kare)
7. [Common Issues & Solutions](#common-issues--solutions)
8. [Step-by-Step Instructions](#step-by-step-instructions)
9. [FAQ - Frequently Asked Questions](#faq---frequently-asked-questions)

---

## Overview - Language System Kya Hai?

### What is Language System?

Language system allows properties to display content in multiple languages. For example:
- Property name in English: "Grand Hotel"
- Property name in Arabic: "فندق جراند"
- Property name in Turkish: "Grand Otel"

### How It Works?

1. **Main Language (English)**: Content is saved in main tables (e.g., `Accommodations`, `Rooms`)
2. **Other Languages**: Translated content is saved in ML tables (e.g., `Accommodations_ML`, `Rooms_ML`)
3. **Automatic Translation**: System automatically translates content using DeepL API
4. **Language Selection**: Users can switch languages using dropdown in header

### Key Concepts

- **Master Switch**: Controls overall translation for a property (ON/OFF)
- **Per-Language Settings**: Control translation for specific languages (Arabic ON, Turkish OFF, etc.)
- **Field Change Detection**: Translation only runs when translatable fields are changed
- **Auto-Translation**: Happens automatically when enabled

---

## Language Selection - User Kaise Language Change Kare?

### For End Users (Frontend)

**Location**: Header (top right corner)

**Steps**:
1. Click on language dropdown in header
2. Search for language (optional - type to filter)
3. Select desired language
4. Page will reload/update with selected language

**Available Languages**: 
- English (Default)
- Arabic
- Turkish
- And other active languages from database

**Features**:
- ✅ Searchable dropdown (type to find language)
- ✅ Flag icons for each language
- ✅ Current language is highlighted
- ✅ Selection persists across sessions (saved in cookies)

**Screenshot Locations**:
- Header dropdown: Top right corner of any page
- Language selector: Shows current language with flag icon

---

## Translation Fields - Konsa Fields Translate Hote Hain?

### Complete List of Translatable Fields

### 1. Property Module

**Page**: Create/Update Property (`/Property/CreateProperty`, `/Property/UpdateProperty`)

| Field Name | Database Column | Description |
|------------|-----------------|-------------|
| Property Name | `AccommodationName` | Name of the property/hotel |

**Example**:
- English: "Grand Hotel"
- Arabic: "فندق جراند"
- Turkish: "Grand Otel"

---

### 2. Additional Details Module

**Page**: Additional Details (`/Property/AdditionalDetail`)

| Field Name | Database Column | Description |
|------------|-----------------|-------------|
| Key Collection Comments | `KeyCollectionComments` | Instructions for key collection |
| Pets Allowed Comments | `PetsAllowedComments` | Comments about pet policy |

---

### 3. Addon Module

**Page**: Create/Update Addon (`/Property/CreateAddon`, `/Property/UpdateAddon`)

| Field Name | Database Column | Description |
|------------|-----------------|-------------|
| Addon Name | `ActivityName` | Name of the addon/activity |
| Short Description | `ShortDescription` | Brief description |
| Cancellation Policy | `CancellationPolicy` | Cancellation policy text |
| Guarantee Policy | `GuaranteePolicy` | Guarantee policy text |
| Long Description | `LongDescription` | Detailed description |

---

### 4. Room Module

**Page**: Create/Update Room (`/RoomManagement/CreateRoom`, `/RoomManagement/UpdateRoom`)

| Field Name | Database Column | Condition | Description |
|------------|-----------------|-----------|-------------|
| Room Name | `RoomName` | When Room Type = Room | Name of the room |
| Apartment Name | `ApartmentName` | When Room Type = Apartment | Name of the apartment |
| Room Description | `RoomDescription` | Always | Description of room/apartment |

**Note**: `RoomName` and `ApartmentName` are used based on room type.

---

### 5. Rate Plan Module

**Page**: Create/Update Rate Plan (`/RoomManagement/CreateRatePlan`, `/RoomManagement/UpdateRatePlan`)

| Field Name | Database Column | Description |
|------------|-----------------|-------------|
| Rate Plan Name | `RatePlanName` | Name of the rate plan |
| Display Rate Plan Name | `DisplayRatePlanName` | Display name for customers |
| Included | `Included` | What's included in rate plan |
| Highlight | `Highlight` | Highlighted features |
| Meal Description | `MealDescription` | Description of meal options |

---

### 6. Promotion Module

**Page**: Create/Update Promotion (`/PromotionManagement/CreatePromotion`, `/PromotionManagement/UpdatePromotion`)

| Field Name | Database Column | Description |
|------------|-----------------|-------------|
| Promotion Name | `PromotionName` | Name of the promotion |
| Description | `Description` | Detailed description |

---

### 7. Facilities Module

**Page**: Property Facilities (`/Property/PropertyFacilities`)  
**Page**: Room Facilities (`/RoomManagement/RoomFacilities`)

| Field Name | Database Column | Description |
|------------|-----------------|-------------|
| Group Name | `GroupName` | Name of facility group |
| Facility Name | `FacilityName` | Name of the facility |
| Other Group Name | `OtherGroupName` | Custom group name |
| Other Facility Name | `OtherFacilityName` | Custom facility name |

---

### Fields That Are NOT Translated

These fields are **NOT** translated (always in original language):

- Email addresses
- Phone numbers
- URLs/Websites
- Dates and times
- Numbers (prices, quantities)
- Country names
- Currency codes
- Time zones
- Technical fields (IDs, codes)

---

## Master Switch - Translation Enable/Disable Kaise Kare?

### What is Master Switch?

Master Switch is a single ON/OFF control that enables or disables translation for entire property.

**Field Name**: `AutoTranslateEnabled`  
**Location**: 
- Update Property page (`/Property/UpdateProperty`)
- All Properties Language Settings page (`/Property/AllPropertiesLanguageSettings`)

### How to Enable/Disable Master Switch?

#### Method 1: Update Property Page

**Steps**:
1. Navigate to `/Property/UpdateProperty`
2. Find "Auto Translate Enabled" checkbox
3. Check to enable, uncheck to disable
4. Click "Save" or "Update Property" button
5. Success message will appear

**Result**:
- ✅ If enabled: Translation will run when translatable fields are changed
- ✅ If disabled: Translation will NOT run (even if fields change)

#### Method 2: All Properties Language Settings Page

**Steps**:
1. Navigate to `/Property/AllPropertiesLanguageSettings`
2. Find property card
3. Locate "Master Switch (Auto Translate Enabled)" toggle
4. Toggle ON to enable, OFF to disable
5. Success toast notification will appear

**Result**:
- ✅ Changes are saved immediately (no page reload needed)
- ✅ Auto-initialization happens if enabled (creates per-language settings)

### When to Use Master Switch?

**Enable Master Switch When**:
- ✅ Customer wants automatic translation for their property
- ✅ Customer wants content in multiple languages
- ✅ Customer has enabled languages in Per-Language Settings

**Disable Master Switch When**:
- ✅ Customer doesn't want translation
- ✅ Customer only wants English content
- ✅ Translation is causing issues

### Important Notes

- ⚠️ **Master Switch must be ON** for translation to work
- ⚠️ If Master Switch is OFF, translation will NOT run even if per-language settings are enabled
- ⚠️ When Master Switch is enabled, Per-Language Settings are auto-initialized (all languages enabled by default)

---

## Per-Language Settings - Specific Language Enable/Disable

### What is Per-Language Settings?

Per-Language Settings allow you to enable/disable translation for **specific languages** per property.

**Example**:
- Master Switch: ON
- Arabic: ON ✅ (will translate)
- Turkish: OFF ❌ (will NOT translate)
- French: ON ✅ (will translate)

### How to Manage Per-Language Settings?

#### Page: All Properties Language Settings

**URL**: `/Property/AllPropertiesLanguageSettings`

**Steps**:
1. Navigate to `/Property/AllPropertiesLanguageSettings`
2. Find the property card (use search/filter if needed)
3. Locate language toggles within the property card
4. Toggle ON/OFF for each language
5. Success toast notification will appear

**Features**:
- ✅ Card-based UI (easy to use)
- ✅ Master switch toggle per property
- ✅ Individual language toggles
- ✅ Search and filter properties
- ✅ Server-side pagination (for many properties)

### Initialize All Languages

If per-language settings don't exist, you can initialize them:

**Steps**:
1. Navigate to `/Property/AllPropertiesLanguageSettings`
2. Scroll to "Initialize Section for Current Property"
3. Choose one:
   - **"Initialize All Languages (Default: Enabled)"** - All languages ON
   - **"Initialize All Languages (Default: Disabled)"** - All languages OFF
4. Click button
5. Success message will appear

**When to Use**:
- ✅ When Master Switch is enabled but per-language settings don't exist
- ✅ When you want to set up all languages at once
- ✅ When customer wants to enable/disable all languages quickly

### Fallback Logic

**How It Works**:
1. If Per-Language Setting exists → Use that setting
2. If Per-Language Setting doesn't exist → Use Master Switch value

**Example**:
- Master Switch: ON
- Arabic: No per-language setting → Uses Master Switch (ON) ✅
- Turkish: Per-language setting = OFF → Uses per-language setting (OFF) ❌

---

## Translation Management Pages - Konsa Pages Use Kare?

### 1. All Properties Language Settings

**URL**: `/Property/AllPropertiesLanguageSettings`  
**Purpose**: Manage translation settings for all properties

**Features**:
- ✅ View all properties in card-based layout
- ✅ Toggle Master Switch per property
- ✅ Enable/disable specific languages per property
- ✅ Search properties by name/ID
- ✅ Filter by status and group
- ✅ Initialize all languages for a property
- ✅ Server-side pagination

**When to Use**:
- ✅ Customer wants to manage translation for multiple properties
- ✅ Customer wants to enable/disable specific languages
- ✅ Customer wants to see all properties at once

**Access**: 
- Menu: Sidebar → "Language Settings"
- Direct URL: `/Property/AllPropertiesLanguageSettings`

---

### 2. Translation History

**URL**: `/Property/TranslationHistory`  
**Purpose**: View all translation activities and history

**Features**:
- ✅ View all translation records
- ✅ Filter by language, field, date range
- ✅ Search in original/translated text
- ✅ Pagination for large datasets
- ✅ Export to CSV/Excel
- ✅ Revert translation (restore previous version)

**When to Use**:
- ✅ Customer wants to see translation history
- ✅ Customer wants to track what was translated
- ✅ Customer wants to revert a translation
- ✅ Customer wants to export translation data

**Access**:
- Menu: Sidebar → "Translation History"
- Direct URL: `/Property/TranslationHistory`

---

### 3. Translation Analytics

**URL**: `/Property/TranslationAnalytics`  
**Purpose**: View translation statistics and analytics

**Features**:
- ✅ Total translations count
- ✅ Auto vs Manual translation breakdown
- ✅ API usage statistics
- ✅ Cost estimation
- ✅ Charts (Language, Provider, Field, Trends)
- ✅ Statistics tables

**When to Use**:
- ✅ Customer wants to see translation statistics
- ✅ Customer wants to track API usage
- ✅ Customer wants to see translation costs
- ✅ Customer wants to analyze translation trends

**Access**:
- Menu: Sidebar → "Translation Analytics"
- Direct URL: `/Property/TranslationAnalytics`

---

### 4. Update Property Page

**URL**: `/Property/UpdateProperty`  
**Purpose**: Update property details and Master Switch

**Features**:
- ✅ Update property information
- ✅ Enable/disable Master Switch
- ✅ Auto-initialization when Master Switch is enabled

**When to Use**:
- ✅ Customer wants to update property details
- ✅ Customer wants to enable/disable translation for single property
- ✅ Customer wants to manage property settings

**Access**:
- Menu: Sidebar → "Properties" → Select property → "Update"
- Direct URL: `/Property/UpdateProperty?accommodationId={id}`

---

## Common Issues & Solutions

### Issue 1: Translation Not Working

**Symptoms**:
- Content is not translating
- ML tables are empty
- No translation history entries

**Possible Causes & Solutions**:

1. **Master Switch is OFF**
   - ✅ Check Master Switch on Update Property page
   - ✅ Enable Master Switch
   - ✅ Try updating a translatable field again

2. **No Per-Language Settings**
   - ✅ Go to All Properties Language Settings
   - ✅ Initialize all languages (Default: Enabled)
   - ✅ Try updating a translatable field again

3. **All Languages Disabled**
   - ✅ Go to All Properties Language Settings
   - ✅ Enable at least one language
   - ✅ Try updating a translatable field again

4. **Field Not Changed**
   - ✅ Translation only runs when translatable fields are changed
   - ✅ Make sure you're changing a translatable field (see list above)
   - ✅ Try changing property name or description

5. **DeepL API Issue**
   - ✅ Check if API key is valid
   - ✅ Check if API quota is available
   - ✅ Check error logs for API errors

---

### Issue 2: Language Dropdown Not Showing

**Symptoms**:
- Language dropdown is missing
- Dropdown doesn't open
- No languages are listed

**Possible Causes & Solutions**:

1. **JavaScript Error**
   - ✅ Check browser console for errors
   - ✅ Refresh page
   - ✅ Clear browser cache

2. **API Not Responding**
   - ✅ Check if `/Home/GetLanguages` API is working
   - ✅ Check network tab in browser
   - ✅ Verify database has active languages

3. **Database Issue**
   - ✅ Check `MultiLanguages` table has active languages
   - ✅ Verify `StatusId = 1` for languages

---

### Issue 3: Selected Language Not Persisting

**Symptoms**:
- Language resets to English after page reload
- Cookie is not saving
- Language claim not updating

**Possible Causes & Solutions**:

1. **Cookie Blocked**
   - ✅ Check browser cookie settings
   - ✅ Allow cookies for the site
   - ✅ Try in incognito/private mode

2. **Middleware Issue**
   - ✅ Check if `LanguageClaimUpdateMiddleware` is registered
   - ✅ Check middleware logs
   - ✅ Verify middleware is running

---

### Issue 4: Translation History Not Showing

**Symptoms**:
- Translation History page is empty
- No records displayed
- Filters not working

**Possible Causes & Solutions**:

1. **No Translation History**
   - ✅ Verify translations have actually run
   - ✅ Check if Master Switch was enabled when updates were made
   - ✅ Check `TranslationHistory` table in database

2. **Filter Issue**
   - ✅ Clear all filters
   - ✅ Try different date range
   - ✅ Check if property ID is correct

3. **Pagination Issue**
   - ✅ Try different page
   - ✅ Check page size
   - ✅ Verify total records count

---

### Issue 5: Revert Translation Not Working

**Symptoms**:
- Revert button not visible
- Revert fails
- Original text not restored

**Possible Causes & Solutions**:

1. **Revert Not Supported**
   - ✅ Not all tables/fields support revert
   - ✅ Check if revert is available for that field
   - ✅ Try reverting a different record

2. **Database Issue**
   - ✅ Check ML table exists
   - ✅ Verify record exists in ML table
   - ✅ Check database permissions

---

## Step-by-Step Instructions

### How to Enable Translation for a Property?

**Scenario**: Customer wants to enable translation for their property

**Steps**:

1. **Enable Master Switch**
   - Go to `/Property/UpdateProperty?accommodationId={id}`
   - Find "Auto Translate Enabled" checkbox
   - Check the checkbox
   - Click "Save" or "Update Property"
   - Wait for success message

2. **Initialize Per-Language Settings** (if needed)
   - Go to `/Property/AllPropertiesLanguageSettings`
   - Find the property (use search if needed)
   - Scroll to "Initialize Section for Current Property"
   - Click "Initialize All Languages (Default: Enabled)"
   - Wait for success message

3. **Configure Specific Languages** (optional)
   - On All Properties Language Settings page
   - Find the property card
   - Toggle ON/OFF for each language as needed
   - Changes save automatically

4. **Test Translation**
   - Go to Update Property page
   - Change property name (translatable field)
   - Save property
   - Check Translation History to verify translation ran

**Result**: Translation is now enabled and will run automatically when translatable fields are changed.

---

### How to Disable Translation for a Property?

**Scenario**: Customer wants to disable translation

**Steps**:

1. **Disable Master Switch**
   - Go to `/Property/UpdateProperty?accommodationId={id}`
   - Find "Auto Translate Enabled" checkbox
   - Uncheck the checkbox
   - Click "Save" or "Update Property"
   - Wait for success message

**Result**: Translation is now disabled. No translations will run even if fields are changed.

---

### How to Enable Translation for Specific Language Only?

**Scenario**: Customer wants translation only for Arabic, not Turkish

**Steps**:

1. **Ensure Master Switch is ON**
   - Go to `/Property/UpdateProperty`
   - Enable "Auto Translate Enabled"
   - Save

2. **Configure Per-Language Settings**
   - Go to `/Property/AllPropertiesLanguageSettings`
   - Find the property
   - Toggle Arabic: ON ✅
   - Toggle Turkish: OFF ❌
   - Toggle other languages as needed

**Result**: Only Arabic will be translated. Turkish and other disabled languages will not be translated.

---

### How to Check Translation History?

**Steps**:

1. Go to `/Property/TranslationHistory`
2. View all translation records
3. Use filters if needed:
   - Select language from dropdown
   - Select field name from dropdown
   - Set date range
   - Enter search text
4. Click "Apply Filters" or search
5. View filtered results

**To Export**:
- Click "Export" dropdown
- Select "Export as CSV" or "Export as Excel"
- File will download

---

### How to Revert a Translation?

**Steps**:

1. Go to `/Property/TranslationHistory`
2. Find the translation record you want to revert
3. Click "Revert" button in Actions column
4. Confirm revert action in dialog
5. Wait for success message
6. Page will reload with updated data

**Result**: Original text is restored. New history entry is created for revert action.

---

## FAQ - Frequently Asked Questions

### Q1: Translation kaise enable kare?

**Answer**: 
1. Go to Update Property page
2. Check "Auto Translate Enabled" checkbox
3. Save property
4. Go to All Properties Language Settings
5. Initialize all languages (Default: Enabled)
6. Translation ab automatically hoga jab translatable fields change honge

---

### Q2: Kya sabhi fields translate hote hain?

**Answer**: 
Nahi, sirf specific fields translate hote hain:
- Property Name ✅
- Room Name/Description ✅
- Rate Plan Name/Description ✅
- Promotion Name/Description ✅
- Addon Name/Description ✅
- Facilities Name ✅

**NOT Translated**:
- Email ❌
- Phone ❌
- Prices ❌
- Dates ❌
- Technical fields ❌

---

### Q3: Master Switch kya hai?

**Answer**: 
Master Switch ek single ON/OFF control hai jo entire property ke liye translation enable/disable karta hai. Agar Master Switch OFF hai, toh translation bilkul nahi hoga, chahe per-language settings kuch bhi ho.

---

### Q4: Per-Language Settings kya hai?

**Answer**: 
Per-Language Settings allow karte hain ki aap specific languages ke liye translation enable/disable kar sakein. Example:
- Master Switch: ON
- Arabic: ON ✅
- Turkish: OFF ❌

Is case mein sirf Arabic translate hoga, Turkish nahi.

---

### Q5: Translation automatically kaise hota hai?

**Answer**: 
1. Master Switch ON hona chahiye
2. At least ek language enabled hona chahiye
3. Translatable field change hona chahiye
4. System automatically DeepL API se translate karega
5. Translation ML tables mein save hoga

---

### Q6: Translation history kahan dekhein?

**Answer**: 
Go to `/Property/TranslationHistory` - yahan sabhi translation records dikhenge with filters, search, and export options.

---

### Q7: Agar translation fail ho jaye toh?

**Answer**: 
- Property update phir bhi successful hoga
- Translation failure se main update affect nahi hota
- Error logs mein check karein
- Translation history mein check karein
- Retry kar sakte hain by updating field again

---

### Q8: Kaise pata chale translation hua ya nahi?

**Answer**: 
1. Check Translation History page - agar entry hai toh translation hua
2. Check ML tables in database - agar data hai toh translation hua
3. Language change karke check karein - agar translated content dikh raha hai toh translation hua

---

### Q9: Language dropdown kahan hai?

**Answer**: 
Header mein top right corner mein - current language ke saath flag icon dikhega. Click karke language change kar sakte hain.

---

### Q10: Kaise pata chale konsa language enabled hai?

**Answer**: 
1. Go to `/Property/AllPropertiesLanguageSettings`
2. Find property card
3. Language toggles dekh sakte hain - ON/OFF status dikhega
4. Green/checked = Enabled ✅
5. Gray/unchecked = Disabled ❌

---

## Quick Reference

### Important URLs

| Page | URL | Purpose |
|------|-----|---------|
| All Properties Language Settings | `/Property/AllPropertiesLanguageSettings` | Manage translation settings |
| Translation History | `/Property/TranslationHistory` | View translation history |
| Translation Analytics | `/Property/TranslationAnalytics` | View statistics |
| Update Property | `/Property/UpdateProperty` | Enable/disable Master Switch |

### Important Fields

| Field | Location | Purpose |
|-------|----------|---------|
| AutoTranslateEnabled | Update Property, All Properties Language Settings | Master Switch |
| Per-Language Toggles | All Properties Language Settings | Enable/disable specific languages |

### Support Contacts

- **Technical Issues**: Check error logs, database
- **API Issues**: Check DeepL API key, quota
- **Translation Issues**: Check Master Switch, Per-Language Settings

---

## Summary

### Key Points to Remember

1. ✅ **Master Switch must be ON** for translation to work
2. ✅ **At least one language must be enabled** in Per-Language Settings
3. ✅ **Only translatable fields** are translated (see list above)
4. ✅ **Translation runs automatically** when translatable fields change
5. ✅ **Translation History** shows all translation activities
6. ✅ **Language dropdown** in header for users to switch languages

### Common Tasks

- **Enable Translation**: Master Switch ON → Initialize Languages → Done
- **Disable Translation**: Master Switch OFF → Done
- **Check Translation**: Go to Translation History → View records
- **Revert Translation**: Translation History → Click Revert → Confirm

---

**Document Version**: 3.0  
**Last Updated**: 2025  
**For Support Team**: Use this guide to help customers with language and translation issues

**Happy Supporting! 🎉**

