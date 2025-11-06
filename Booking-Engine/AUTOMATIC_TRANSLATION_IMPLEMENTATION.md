# 🤖 Automatic Multi-Language Translation Implementation Guide

This guide provides the structure, architecture, and method signatures for implementing automatic translation support in BookingWhizz using DeepL API or LibreTranslate.

## 📋 Table of Contents

- [Overview](#overview)
- [Configuration Structure](#configuration-structure)
- [Folder Structure](#folder-structure)
- [Service Architecture](#service-architecture)
- [Translation Field Mapping](#translation-field-mapping)
- [Field Change Detection](#field-change-detection) ⭐ **NEW**
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
6. **Conditional Translation**: ⭐ **Translation only runs when translatable fields are changed**

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
      "ApiKey": "",
      "RateLimitDelayMs": 100
    }
  }
}
```

---

## Folder Structure

```
V3BookingEngine/
├── Services/
│   └── TranslationService/
│       ├── ITranslationService.cs
│       ├── TranslationServiceFactory.cs
│       ├── TranslationServiceBase.cs
│       ├── Providers/
│       │   ├── DeepLTranslationProvider.cs
│       │   ├── LibreTranslateProvider.cs
│       │   └── ITranslationProvider.cs
│       ├── Models/
│       │   ├── TranslationRequest.cs
│       │   ├── TranslationResponse.cs
│       │   └── TranslationResult.cs
│       └── ServiceRegistration.cs
│
├── Services/
│   ├── PropertyService/
│   │   └── PropertyTranslationHelper.cs
│   ├── RoomManagementService/
│   │   └── RoomTranslationHelper.cs
│   └── ...
│
└── Helpers/
    └── TranslationHelper.cs
```

---

## Translation Field Mapping

### Property Module - Translatable Fields

| Field Name | Model Property | Translatable |
|------------|---------------|-------------|
| Property Name | `AccommodationName` | ✅ YES |
| Property Type | `AccommodationTypeId` | ❌ NO |
| Property Category | `PropertyCategory` | ❌ NO |
| Time Zone | `TimeZoneId` | ❌ NO |
| Country | `Country` | ❌ NO |
| City | `AddressId` | ❌ NO |
| Post Code | `PostCode` | ❌ NO |
| Currency | `CurrencyId` | ❌ NO |
| Contact Number | `ContactNo` | ❌ NO |
| Email | `EmailId` | ❌ NO |
| Latitude | `Latitude` | ❌ NO |
| Longitude | `Longitude` | ❌ NO |
| Web Search | `WebSearch` | ❌ NO |
| Home Shopping | `HomeShopping` | ❌ NO |

### Additional Detail Module - Translatable Fields

| Field Name | Model Property | Translatable |
|------------|---------------|-------------|
| Key Collection Comments | `KeyCollectionComments` | ✅ YES |
| Pets Allowed Comments | `PetsAllowedComments` | ✅ YES |

### Addon Module - Translatable Fields

| Field Name | Model Property | Translatable |
|------------|---------------|-------------|
| Addon Name | `AddonName` | ✅ YES |
| Short Description | `ShortDescription` | ✅ YES |
| Cancellation Policy | `CancellationPolicy` | ✅ YES |
| Guarantee Policy | `GuaranteePolicy` | ✅ YES |
| Long Description | `LongDescription` | ✅ YES |

### Room Module - Translatable Fields

| Field Name | Model Property | Translatable |
|------------|---------------|-------------|
| Room Name | `RoomName` | ✅ YES |
| Room Description | `RoomDescription` | ✅ YES |

### Rate Plan Module - Translatable Fields

| Field Name | Model Property | Translatable |
|------------|---------------|-------------|
| Rate Plan Name | `RatePlanName` | ✅ YES |
| Display Rate Plan Name | `DisplayRatePlanName` | ✅ YES |
| Included | `Included` | ✅ YES |
| Highlight | `Highlight` | ✅ YES |
| Meal Description | `MealDescription` | ✅ YES |

### Policy Module - Translatable Fields

| Field Name | Model Property | Translatable |
|------------|---------------|-------------|
| Policy Name | `PolicyName` | ✅ YES |
| Cancellation Policy Description | `CancellationPolicyDescription` | ✅ YES |
| Booking Policy Description | `BookingPolicyDescription` | ✅ YES |
| No-Show Policy Description | `NoShowPolicyDescription` | ✅ YES |

### Promotion Module - Translatable Fields

| Field Name | Model Property | Translatable |
|------------|---------------|-------------|
| Promotion Name | `PromotionName` | ✅ YES |
| Description | `Description` | ✅ YES |

---

## Non-Translatable Fields (Translation Will NOT Run) ❌

These fields are **NOT translatable**. If only these fields are changed, translation will **NOT** run.

### Property Module - Non-Translatable Fields

| Field Name | Model Property | Reason |
|------------|---------------|--------|
| Property Type | `AccommodationTypeId` | ID field (lookup) |
| Property Category | `PropertyCategory` | ID list (lookup) |
| Time Zone | `TimeZoneId` | ID field (lookup) |
| Country | `Country` | ID field (lookup) |
| City | `AddressId` | ID field (lookup) |
| Post Code | `PostCode` | Alphanumeric code (not translatable) |
| Currency | `CurrencyId` | ID field (lookup) |
| Contact Number | `ContactNo` | Phone number (not translatable) |
| Email | `EmailId` | Email address (not translatable) |
| Latitude | `Latitude` | Numeric coordinate |
| Longitude | `Longitude` | Numeric coordinate |
| Web Search | `WebSearch` | Boolean flag |
| Home Shopping | `HomeShopping` | Boolean flag |

### Additional Detail Module - Non-Translatable Fields

| Field Name | Model Property | Reason |
|------------|---------------|--------|
| All other fields except `KeyCollectionComments` and `PetsAllowedComments` | Various | IDs, dates, booleans, etc. |

### Addon Module - Non-Translatable Fields

| Field Name | Model Property | Reason |
|------------|---------------|--------|
| Activity ID | `ActivityId` | ID field |
| Bookable With Rateplan | `BookableWithRateplan` | Boolean flag |
| Bookable Individual | `BookableIndividual` | Boolean flag |
| Max Bookable Quantity | `MaxBookableQty` | Numeric value |
| Currency | `CurrencyId` | ID field (lookup) |
| Currency Code | `CurrencyCode` | Code (not translatable) |
| Activity Bookable Type | `ActivityBookableTypeId` | ID field (lookup) |
| Activity Category Type IDs | `ActivityCategoryTypeIds` | ID list (lookup) |
| Variation Values | `VariationValues` | Price variations (numeric) |
| Variation Prices | `VariationPrices` | Price list (numeric) |
| Rates | `Rates` | Price (numeric) |
| Status ID | `StatusId` | ID field (lookup) |
| Image | `Image` | File path |
| Temp Image | `TempImage` | File path |

### Room Module - Non-Translatable Fields

| Field Name | Model Property | Reason |
|------------|---------------|--------|
| Copy Room ID | `CopyRoomId` | ID field |
| Room Type | `RoomTypeId` | ID field (lookup) |
| Apartment Name | `ApartmentName` | (Note: Only `RoomName` is translatable for Room Type 1) |
| Total Guest | `TotalGuest` | Numeric value |
| Rack Rate | `RackRate` | Price (numeric) |
| Max Persons | `MaxPersons` | Numeric value |
| Number of Beds | `NoOfBeds` | Numeric value |
| Bed Size | `BedSize` | Size specification (not translatable) |
| Room Size | `RoomSize` | Numeric value (square meters/feet) |
| Bathroom Detail | `BathRoomDetail` | Numeric value |
| Room Address | `RoomAddress` | Address (may be translatable in future, currently not) |
| All Array Bed | `AllArrayBed` | JSON/Array data |
| Stay Room ID | `StayRoomId` | ID field |

### Rate Plan Module - Non-Translatable Fields

| Field Name | Model Property | Reason |
|------------|---------------|--------|
| Rate Plan ID | `RatePlanId` | ID field |
| Room ID | `RoomId` | ID field (lookup) |
| Default Currency | `RatePlan_DefualtCurrencyId` | ID field (lookup) |
| Other Currency | `RatePlan_OtherCurrencyId` | ID field (lookup) |
| Default Rates | `DefaultRates` | Price (numeric) |
| Guest Quantity | `GuestQuantity` | Numeric value |
| Adult Quantity | `AdultQuantity` | Numeric value |
| Child Quantity | `ChildQuantity` | Numeric value |
| Default Minimum Stay | `DefaultMinStay` | Numeric value |
| Default Maximum Stay | `DefaultMaxStay` | Numeric value |
| Booking Window From | `BookingWindowFrom` | Numeric value (days) |
| Booking Window To | `BookingWindowTo` | Numeric value (days) |
| Rate Code | `RateCode` | Code (not translatable) |
| Meal ID | `MealId` | ID field (lookup) |
| Meal Type ID | `MealTypeId` | ID field (lookup) |
| On Request | `OnRequest` | Boolean flag |
| Is Activities Mapped | `IsActivitiesMapped` | Boolean flag |
| Is Activities | `IsActivities` | Boolean flag |
| Is Occupancy | `IsOccupancy` | Boolean flag |
| Is Loyalty | `IsLoyalty` | Boolean flag |
| Advance Payment | `AdvancePayment` | Boolean flag |
| Web | `Web` | Boolean flag |
| Mobile | `Mobile` | Boolean flag |
| Tablet | `Tablet` | Boolean flag |
| Registered User | `Registered_User` | Boolean flag |
| Mobile App | `MobileApp` | Boolean flag |
| CRS | `CRS` | Boolean flag |
| Policy IDs | `PolicyId` | ID list (lookup) |
| Policy Start Dates | `PolicyStartDate` | Date list |
| Policy End Dates | `PolicyEndDate` | Date list |
| Policy Days (Monday-Sunday) | `PolicyMonday`, `PolicyTuesday`, etc. | Boolean flags |

### Policy Module - Non-Translatable Fields

| Field Name | Model Property | Reason |
|------------|---------------|--------|
| Accommodation ID | `AccommodationId` | ID field |
| Owner ID | `OwnerId` | ID field |
| Language ID | `LanguageId` | ID field |
| Cancellation Policy Type | `CancellationPolicyTypeId` | ID field (lookup) |
| Booking Policy Type | `BookingPolicyTypeId` | ID field (lookup) |
| No-Show Policy Type | `NoShowPolicyTypeId` | ID field (lookup) |

### Promotion Module - Non-Translatable Fields

| Field Name | Model Property | Reason |
|------------|---------------|--------|
| Policy ID | `PolicyId` | ID field (lookup) |
| Promotion Type | `PromotionTypeId` | ID field (lookup) |
| Book Start Date | `BookStartDate` | Date |
| Book End Date | `BookEndDate` | Date |
| Stay Start Date | `StayStartDate` | Date |
| Stay End Date | `StayEndDate` | Date |
| Black Date Type | `BlackDateTypeId` | ID field (lookup) |
| Block Dates | `BlockDates` | Date list |
| Check In | `CheckIn` | Boolean flag |
| Checkout | `Checkout` | Boolean flag |
| Is Multi | `IsMulti` | Boolean flag |
| Voucher Code | `VoucherCode` | Code (not translatable) |
| Min Stay | `MinStay` | Numeric value |
| Max Stay | `MaxStay` | Numeric value |
| Device ID | `DeviceId` | ID field (lookup) |
| Region ID | `RegionId` | ID field (lookup) |
| Country ID | `CountryId` | ID field (lookup) |
| City ID | `CityId` | ID field (lookup) |
| Currency ID | `CurrencyId` | ID field (lookup) |
| Early Days | `EarlyDays` | Numeric value |
| Early Hours | `EarlyHours` | Numeric value |
| Free Nights | `FreeNights` | Numeric value |
| Free Days | `FreeDays` | Numeric value |
| Multi Region ID | `MultiRegionId` | ID field (lookup) |
| Multi Country IDs | `MultiCountryIds` | ID list (lookup) |
| Multi City IDs | `MultiCityIds` | ID list (lookup) |
| Per Nights | `PerNights` | Numeric list |
| Days | `Days` | Numeric list |
| Percentages | `Percentages` | Numeric list |
| Discount Value | `DiscountValue` | Price (numeric) |
| Discount Type | `DiscountType` | ID field (0 = %, 1 = Fixed) |
| Monday-Sunday | `Monday`, `Tuesday`, etc. | Boolean flags |
| Room Rate Plan IDs | `RoomRatePlanIds` | ID list (lookup) |

### Property Facilities Module - Non-Translatable Fields

| Field Name | Model Property | Reason |
|------------|---------------|--------|
| Type | `Type` | ID field (1 = Group, 2 = Facility) |
| Group ID | `GroupId` | ID field (lookup) |
| Facility IDs | `FacilityIds` | ID array (lookup) |
| Group IDs | `GroupIds` | ID array (lookup) |
| Is Checked | `IsChecked` | Boolean array |
| Comments | `Comments` | (Note: Comments may be translatable in future) |
| Group Facility IDs | `GroupFacilityIds` | ID array |

### Room Facilities Module - Non-Translatable Fields

| Field Name | Model Property | Reason |
|------------|---------------|--------|
| Type | `Type` | ID field (1 = Group, 2 = Facility) |
| Group ID | `GroupId` | ID field (lookup) |
| Facility IDs | `FacilityIds` | ID array (lookup) |
| Group IDs | `GroupIds` | ID array (lookup) |
| Is Checked | `IsChecked` | Boolean array |
| Comments | `Comments` | (Note: Comments may be translatable in future) |
| Group Facility IDs | `GroupFacilityIds` | ID array |
| Room ID | `RoomId` | ID field |
| Room ID Drag | `RoomIdDrag` | ID field |
| GF ID | `GFId` | ID field |

---

## Field Change Detection Rules ⚠️

### Translation Will Run ✅
- If **ANY** translatable field is changed
- Example: `AccommodationName` changed → Translation runs

### Translation Will NOT Run ❌
- If **ONLY** non-translatable fields are changed
- Example: Only `TimeZoneId` changed → Translation skipped
- Example: Only `CurrencyId` changed → Translation skipped
- Example: Only `MaxPersons` changed → Translation skipped

### Mixed Changes
- If both translatable AND non-translatable fields changed:
  - Translation **WILL** run (because translatable field changed)
  - Only translatable fields will be translated
  - Non-translatable fields will be saved as-is

### Examples

#### Example 1: Property Update
```
Changed Fields:
- AccommodationName: "Grand Hotel" → "Grand Hotel Premium" ✅ (Translatable)
- TimeZoneId: 1 → 2 ❌ (Non-translatable)

Result: ✅ TRANSLATE (AccommodationName changed)
```

#### Example 2: Property Update
```
Changed Fields:
- TimeZoneId: 1 → 2 ❌ (Non-translatable)
- CurrencyId: 1 → 3 ❌ (Non-translatable)
- AccommodationName: "Grand Hotel" (no change)

Result: ❌ SKIP TRANSLATION (No translatable fields changed)
```

#### Example 3: Room Update
```
Changed Fields:
- RoomName: "Standard Room" → "Deluxe Suite" ✅ (Translatable)
- MaxPersons: 2 → 4 ❌ (Non-translatable)
- RackRate: 100 → 150 ❌ (Non-translatable)

Result: ✅ TRANSLATE (RoomName changed)
Action: Only RoomName will be translated, MaxPersons and RackRate saved as-is
```

#### Example 4: Rate Plan Update
```
Changed Fields:
- DefaultRates: 100 → 150 ❌ (Non-translatable)
- GuestQuantity: 2 → 3 ❌ (Non-translatable)
- RatePlanName: "Standard Rate" (no change)

Result: ❌ SKIP TRANSLATION (No translatable fields changed)
```

---

## Field Change Detection ⭐

### Overview

Translation should **ONLY** run when translatable fields are changed. If only non-translatable fields (like `TimeZoneId`, `CurrencyId`, etc.) are modified, translation should **NOT** run.

### Implementation Strategy

#### 1. Field Change Detection Helper

```csharp
// V3BookingEngine/Helpers/FieldChangeDetector.cs
namespace V3BookingEngine.Helpers
{
    public static class FieldChangeDetector
    {
        /// <summary>
        /// Checks if any translatable fields have changed
        /// </summary>
        public static bool HasTranslatableFieldsChanged<T>(
            T newModel, 
            T oldModel, 
            List<string> translatableFields) where T : class
        {
            if (oldModel == null) return true; // New record, translate if fields have values
            
            foreach (var fieldName in translatableFields)
            {
                var newValue = GetPropertyValue(newModel, fieldName);
                var oldValue = GetPropertyValue(oldModel, fieldName);
                
                if (!AreEqual(newValue, oldValue))
                {
                    return true; // Field changed
                }
            }
            
            return false; // No translatable fields changed
        }

        /// <summary>
        /// Gets list of changed translatable fields
        /// </summary>
        public static List<string> GetChangedTranslatableFields<T>(
            T newModel, 
            T oldModel, 
            List<string> translatableFields) where T : class
        {
            var changedFields = new List<string>();
            
            if (oldModel == null) 
            {
                // New record - return all fields that have values
                foreach (var fieldName in translatableFields)
                {
                    var value = GetPropertyValue(newModel, fieldName);
                    if (!string.IsNullOrWhiteSpace(value?.ToString()))
                    {
                        changedFields.Add(fieldName);
                    }
                }
                return changedFields;
            }
            
            foreach (var fieldName in translatableFields)
            {
                var newValue = GetPropertyValue(newModel, fieldName);
                var oldValue = GetPropertyValue(oldModel, fieldName);
                
                if (!AreEqual(newValue, oldValue))
                {
                    changedFields.Add(fieldName);
                }
            }
            
            return changedFields;
        }

        private static object? GetPropertyValue<T>(T obj, string propertyName) where T : class
        {
            var property = typeof(T).GetProperty(propertyName);
            return property?.GetValue(obj);
        }

        private static bool AreEqual(object? value1, object? value2)
        {
            if (value1 == null && value2 == null) return true;
            if (value1 == null || value2 == null) return false;
            
            // Handle string comparison (trim whitespace)
            if (value1 is string str1 && value2 is string str2)
            {
                return str1.Trim().Equals(str2.Trim(), StringComparison.OrdinalIgnoreCase);
            }
            
            return value1.Equals(value2);
        }
    }
}
```

#### 2. Translatable Fields Configuration

```csharp
// V3BookingEngine/Helpers/TranslatableFieldsConfig.cs
namespace V3BookingEngine.Helpers
{
    public static class TranslatableFieldsConfig
    {
        // Property Module
        public static readonly List<string> PropertyTranslatableFields = new()
        {
            "AccommodationName"
        };

        // Additional Detail Module
        public static readonly List<string> AdditionalDetailTranslatableFields = new()
        {
            "KeyCollectionComments",
            "PetsAllowedComments"
        };

        // Addon Module (AddonFormModel)
        public static readonly List<string> AddonTranslatableFields = new()
        {
            "ActivityName",  // Addon Name
            "ShortDescription",
            "CancellationPolicy",
            "GuaranteePolicy",
            "LongDescription"
        };

        // Room Module (CreateRoomFormModel)
        public static readonly List<string> RoomTranslatableFields = new()
        {
            "RoomName",
            "RoomDescription"
        };

        // Rate Plan Module (UpdateRatePlanFormModel / CreateRatePlanFormModel)
        public static readonly List<string> RatePlanTranslatableFields = new()
        {
            "RatePlanName",
            "DisplayRatePlanName",
            "Included",
            "Highlight",
            "MealDescription"
        };

        // Policy Module (CreatePolicyViewModel)
        public static readonly List<string> PolicyTranslatableFields = new()
        {
            "PolicyName",
            "CancellationPolicyDescription",
            "BookingPolicyDescription",
            "NoShowPolicyDescription"
        };

        // Promotion Module (CreatePromotionFormModel)
        public static readonly List<string> PromotionTranslatableFields = new()
        {
            "PromotionName",
            "Description"
        };

        // Property Facilities Module (AddGroupFacilityModel)
        public static readonly List<string> PropertyFacilitiesTranslatableFields = new()
        {
            "GroupName",
            "FacilityName",
            "OtherGroupName",
            "OtherFacilityName"
        };

        // Room Facilities Module (AddGroupFacilityModel)
        public static readonly List<string> RoomFacilitiesTranslatableFields = new()
        {
            "GroupName",
            "FacilityName",
            "OtherGroupName",
            "OtherFacilityName"
        };
    }
}
```

#### 3. Updated Translation Helper with Change Detection

```csharp
// V3BookingEngine/Services/PropertyService/PropertyTranslationHelper.cs
namespace V3BookingEngine.Services.PropertyService
{
    public class PropertyTranslationHelper
    {
        private readonly ITranslationService _translationService;
        private readonly ILanguageService _languageService;
        private readonly IPropertyManagementService _propertyService;

        /// <summary>
        /// Translates property fields ONLY if translatable fields changed
        /// </summary>
        public async Task<bool> ShouldTranslatePropertyAsync(
            CreatePropertyViewModel newModel,
            CreatePropertyViewModel? oldModel,
            int accommodationId = 0)
        {
            // Check if translation is enabled for this property
            if (accommodationId > 0)
            {
                var isEnabled = await TranslationHelper.IsTranslationEnabledForPropertyAsync(
                    accommodationId, 
                    _propertyService.GetConnection());
                if (!isEnabled) return false;
            }

            // Check if any translatable fields changed
            var hasChanged = FieldChangeDetector.HasTranslatableFieldsChanged(
                newModel,
                oldModel,
                TranslatableFieldsConfig.PropertyTranslatableFields
            );

            return hasChanged;
        }

        /// <summary>
        /// Translates property fields when creating/updating property
        /// </summary>
        public async Task<CreatePropertyViewModel> TranslatePropertyAsync(
            CreatePropertyViewModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            int accommodationId = 0,
            CreatePropertyViewModel? oldModel = null)
        {
            // ⭐ CHECK: Only translate if translatable fields changed
            if (accommodationId > 0 && oldModel != null)
            {
                var shouldTranslate = await ShouldTranslatePropertyAsync(sourceModel, oldModel, accommodationId);
                if (!shouldTranslate)
                {
                    // No translatable fields changed, return original model
                    return sourceModel;
                }
            }

            // Get source and target language codes
            var sourceLang = await _languageService.GetLanguageByIdAsync(sourceLanguageId);
            var targetLang = await _languageService.GetLanguageByIdAsync(targetLanguageId);

            if (sourceLang == null || targetLang == null)
            {
                return sourceModel; // Can't translate without language info
            }

            // Get changed fields only
            var changedFields = FieldChangeDetector.GetChangedTranslatableFields(
                sourceModel,
                oldModel,
                TranslatableFieldsConfig.PropertyTranslatableFields
            );

            // Translate only changed fields
            if (changedFields.Contains("AccommodationName") && 
                !string.IsNullOrWhiteSpace(sourceModel.AccommodationName))
            {
                sourceModel.AccommodationName = await _translationService.TranslateTextAsync(
                    sourceModel.AccommodationName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            return sourceModel;
        }
    }
}
```

---

## Service Architecture

### Main Translation Service Interface

```csharp
// V3BookingEngine/Services/TranslationService/ITranslationService.cs
namespace V3BookingEngine.Services.TranslationService
{
    public interface ITranslationService
    {
        Task<string> TranslateTextAsync(
            string text, 
            string sourceLanguageCode, 
            string targetLanguageCode);

        Task<Dictionary<string, string>> TranslateBatchAsync(
            Dictionary<string, string> texts, 
            string sourceLanguageCode, 
            string targetLanguageCode);

        Task<bool> IsServiceAvailableAsync();
        string GetProviderName();
    }
}
```

---

## Integration Points

### Updated Property Management Service

```csharp
// V3BookingEngine/Services/PropertyService/PropertyManagementService.cs

// Existing method - keep as is for backward compatibility
public async Task<string> CreatePropertyAsync(
    CreatePropertyViewModel model, 
    int ownerId, 
    int languageId)

// NEW: Method with translation support and change detection
public async Task<string> CreatePropertyWithTranslationAsync(
    CreatePropertyViewModel model,
    int ownerId,
    int sourceLanguageId,
    int targetLanguageId,
    bool autoTranslate = false)
{
    try
    {
        // 1. Save to main table if sourceLanguageId = 1 (default/English)
        if (sourceLanguageId == 1)
        {
            var result = await CreatePropertyAsync(model, ownerId, sourceLanguageId);
            if (string.IsNullOrEmpty(result) || result == "Failure")
            {
                return result;
            }
            
            // Extract accommodation ID from result if possible
            var accommodationId = ExtractAccommodationId(result);
            
            // 2. If targetLanguageId != 1 and autoTranslate = true:
            if (targetLanguageId != 1 && autoTranslate && accommodationId > 0)
            {
                // ⭐ CHECK: Only translate if translatable fields have values
                var hasTranslatableFields = HasTranslatableFields(model, 
                    TranslatableFieldsConfig.PropertyTranslatableFields);
                
                if (hasTranslatableFields)
                {
                    // Translate and save to ML table
                    var translatedModel = await _propertyTranslationHelper.TranslatePropertyAsync(
                        model,
                        sourceLanguageId,
                        targetLanguageId,
                        accommodationId
                    );
                    
                    await SavePropertyTranslationAsync(accommodationId, translatedModel, targetLanguageId);
                }
            }
            
            return result;
        }
        else
        {
            // Save directly to ML table (non-default language)
            return await SavePropertyTranslationAsync(0, model, sourceLanguageId);
        }
    }
    catch (Exception ex)
    {
        // Log error
        return "Failure";
    }
}

// NEW: Update method with change detection
public async Task<string> UpdatePropertyWithTranslationAsync(
    CreatePropertyViewModel newModel,
    int accommodationId,
    int ownerId,
    int sourceLanguageId,
    int targetLanguageId,
    bool autoTranslate = false)
{
    try
    {
        // 1. Get existing property data
        var oldModel = await GetPropertyByIdAsync(accommodationId, sourceLanguageId);
        
        // 2. Update main table if sourceLanguageId = 1
        if (sourceLanguageId == 1)
        {
            var result = await UpdatePropertyAsync(newModel, accommodationId, ownerId, sourceLanguageId);
            if (string.IsNullOrEmpty(result) || result == "Failure")
            {
                return result;
            }
        }
        
        // 3. ⭐ CHECK: Only translate if translatable fields changed
        if (targetLanguageId != 1 && autoTranslate)
        {
            var shouldTranslate = await _propertyTranslationHelper.ShouldTranslatePropertyAsync(
                newModel,
                oldModel,
                accommodationId
            );
            
            if (shouldTranslate)
            {
                // Translate only changed fields
                var translatedModel = await _propertyTranslationHelper.TranslatePropertyAsync(
                    newModel,
                    sourceLanguageId,
                    targetLanguageId,
                    accommodationId,
                    oldModel
                );
                
                await SavePropertyTranslationAsync(accommodationId, translatedModel, targetLanguageId);
            }
        }
        
        return "Success";
    }
    catch (Exception ex)
    {
        // Log error
        return "Failure";
    }
}

// Helper method to check if model has translatable fields with values
private bool HasTranslatableFields<T>(T model, List<string> translatableFields) where T : class
{
    foreach (var fieldName in translatableFields)
    {
        var property = typeof(T).GetProperty(fieldName);
        var value = property?.GetValue(model);
        
        if (value is string str && !string.IsNullOrWhiteSpace(str))
        {
            return true;
        }
    }
    return false;
}
```

### 4. Room Translation Helper with Change Detection

```csharp
// V3BookingEngine/Services/RoomManagementService/RoomTranslationHelper.cs
namespace V3BookingEngine.Services.RoomManagementService
{
    public class RoomTranslationHelper
    {
        private readonly ITranslationService _translationService;
        private readonly ILanguageService _languageService;

        /// <summary>
        /// Checks if translatable fields changed for Room
        /// </summary>
        public bool ShouldTranslateRoom(
            CreateRoomFormModel newModel,
            CreateRoomFormModel? oldModel)
        {
            return FieldChangeDetector.HasTranslatableFieldsChanged(
                newModel,
                oldModel,
                TranslatableFieldsConfig.RoomTranslatableFields
            );
        }

        /// <summary>
        /// Translates room fields ONLY if translatable fields changed
        /// </summary>
        public async Task<CreateRoomFormModel> TranslateRoomAsync(
            CreateRoomFormModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            CreateRoomFormModel? oldModel = null)
        {
            // ⭐ CHECK: Only translate if translatable fields changed
            if (oldModel != null)
            {
                var shouldTranslate = ShouldTranslateRoom(sourceModel, oldModel);
                if (!shouldTranslate)
                {
                    return sourceModel; // No translatable fields changed
                }
            }

            var sourceLang = await _languageService.GetLanguageByIdAsync(sourceLanguageId);
            var targetLang = await _languageService.GetLanguageByIdAsync(targetLanguageId);

            if (sourceLang == null || targetLang == null)
            {
                return sourceModel;
            }

            // Get changed fields only
            var changedFields = FieldChangeDetector.GetChangedTranslatableFields(
                sourceModel,
                oldModel,
                TranslatableFieldsConfig.RoomTranslatableFields
            );

            // Translate only changed fields
            if (changedFields.Contains("RoomName") && !string.IsNullOrWhiteSpace(sourceModel.RoomName))
            {
                sourceModel.RoomName = await _translationService.TranslateTextAsync(
                    sourceModel.RoomName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("RoomDescription") && !string.IsNullOrWhiteSpace(sourceModel.RoomDescription))
            {
                sourceModel.RoomDescription = await _translationService.TranslateTextAsync(
                    sourceModel.RoomDescription,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            return sourceModel;
        }
    }
}
```

### 5. Rate Plan Translation Helper with Change Detection

```csharp
// V3BookingEngine/Services/RoomManagementService/RatePlanTranslationHelper.cs
namespace V3BookingEngine.Services.RoomManagementService
{
    public class RatePlanTranslationHelper
    {
        private readonly ITranslationService _translationService;
        private readonly ILanguageService _languageService;

        /// <summary>
        /// Checks if translatable fields changed for Rate Plan
        /// </summary>
        public bool ShouldTranslateRatePlan(
            UpdateRatePlanFormModel newModel,
            UpdateRatePlanFormModel? oldModel)
        {
            return FieldChangeDetector.HasTranslatableFieldsChanged(
                newModel,
                oldModel,
                TranslatableFieldsConfig.RatePlanTranslatableFields
            );
        }

        /// <summary>
        /// Translates rate plan fields ONLY if translatable fields changed
        /// </summary>
        public async Task<UpdateRatePlanFormModel> TranslateRatePlanAsync(
            UpdateRatePlanFormModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            UpdateRatePlanFormModel? oldModel = null)
        {
            // ⭐ CHECK: Only translate if translatable fields changed
            if (oldModel != null)
            {
                var shouldTranslate = ShouldTranslateRatePlan(sourceModel, oldModel);
                if (!shouldTranslate)
                {
                    return sourceModel;
                }
            }

            var sourceLang = await _languageService.GetLanguageByIdAsync(sourceLanguageId);
            var targetLang = await _languageService.GetLanguageByIdAsync(targetLanguageId);

            if (sourceLang == null || targetLang == null)
            {
                return sourceModel;
            }

            var changedFields = FieldChangeDetector.GetChangedTranslatableFields(
                sourceModel,
                oldModel,
                TranslatableFieldsConfig.RatePlanTranslatableFields
            );

            // Translate only changed fields
            if (changedFields.Contains("RatePlanName") && !string.IsNullOrWhiteSpace(sourceModel.RatePlanName))
            {
                sourceModel.RatePlanName = await _translationService.TranslateTextAsync(
                    sourceModel.RatePlanName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("DisplayRatePlanName") && !string.IsNullOrWhiteSpace(sourceModel.DisplayRatePlanName))
            {
                sourceModel.DisplayRatePlanName = await _translationService.TranslateTextAsync(
                    sourceModel.DisplayRatePlanName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("Included") && !string.IsNullOrWhiteSpace(sourceModel.Included))
            {
                sourceModel.Included = await _translationService.TranslateTextAsync(
                    sourceModel.Included,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("Highlight") && !string.IsNullOrWhiteSpace(sourceModel.Highlight))
            {
                sourceModel.Highlight = await _translationService.TranslateTextAsync(
                    sourceModel.Highlight,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("MealDescription") && !string.IsNullOrWhiteSpace(sourceModel.MealDescription))
            {
                sourceModel.MealDescription = await _translationService.TranslateTextAsync(
                    sourceModel.MealDescription,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            return sourceModel;
        }
    }
}
```

### 6. Addon Translation Helper with Change Detection

```csharp
// V3BookingEngine/Services/PropertyService/AddonTranslationHelper.cs
namespace V3BookingEngine.Services.PropertyService
{
    public class AddonTranslationHelper
    {
        private readonly ITranslationService _translationService;
        private readonly ILanguageService _languageService;

        /// <summary>
        /// Checks if translatable fields changed for Addon
        /// </summary>
        public bool ShouldTranslateAddon(
            AddonFormModel newModel,
            AddonFormModel? oldModel)
        {
            return FieldChangeDetector.HasTranslatableFieldsChanged(
                newModel,
                oldModel,
                TranslatableFieldsConfig.AddonTranslatableFields
            );
        }

        /// <summary>
        /// Translates addon fields ONLY if translatable fields changed
        /// </summary>
        public async Task<AddonFormModel> TranslateAddonAsync(
            AddonFormModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            AddonFormModel? oldModel = null)
        {
            // ⭐ CHECK: Only translate if translatable fields changed
            if (oldModel != null)
            {
                var shouldTranslate = ShouldTranslateAddon(sourceModel, oldModel);
                if (!shouldTranslate)
                {
                    return sourceModel;
                }
            }

            var sourceLang = await _languageService.GetLanguageByIdAsync(sourceLanguageId);
            var targetLang = await _languageService.GetLanguageByIdAsync(targetLanguageId);

            if (sourceLang == null || targetLang == null)
            {
                return sourceModel;
            }

            var changedFields = FieldChangeDetector.GetChangedTranslatableFields(
                sourceModel,
                oldModel,
                TranslatableFieldsConfig.AddonTranslatableFields
            );

            // Translate only changed fields
            if (changedFields.Contains("ActivityName") && !string.IsNullOrWhiteSpace(sourceModel.ActivityName))
            {
                sourceModel.ActivityName = await _translationService.TranslateTextAsync(
                    sourceModel.ActivityName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("ShortDescription") && !string.IsNullOrWhiteSpace(sourceModel.ShortDescription))
            {
                sourceModel.ShortDescription = await _translationService.TranslateTextAsync(
                    sourceModel.ShortDescription,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("LongDescription") && !string.IsNullOrWhiteSpace(sourceModel.LongDescription))
            {
                sourceModel.LongDescription = await _translationService.TranslateTextAsync(
                    sourceModel.LongDescription,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("CancellationPolicy") && !string.IsNullOrWhiteSpace(sourceModel.CancellationPolicy))
            {
                sourceModel.CancellationPolicy = await _translationService.TranslateTextAsync(
                    sourceModel.CancellationPolicy,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("GuaranteePolicy") && !string.IsNullOrWhiteSpace(sourceModel.GuaranteePolicy))
            {
                sourceModel.GuaranteePolicy = await _translationService.TranslateTextAsync(
                    sourceModel.GuaranteePolicy,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            return sourceModel;
        }
    }
}
```

### 7. Policy Translation Helper with Change Detection

```csharp
// V3BookingEngine/Services/PolicyManagementService/PolicyTranslationHelper.cs
namespace V3BookingEngine.Services.PolicyManagementService
{
    public class PolicyTranslationHelper
    {
        private readonly ITranslationService _translationService;
        private readonly ILanguageService _languageService;

        /// <summary>
        /// Checks if translatable fields changed for Policy
        /// </summary>
        public bool ShouldTranslatePolicy(
            CreatePolicyViewModel newModel,
            CreatePolicyViewModel? oldModel)
        {
            return FieldChangeDetector.HasTranslatableFieldsChanged(
                newModel,
                oldModel,
                TranslatableFieldsConfig.PolicyTranslatableFields
            );
        }

        /// <summary>
        /// Translates policy fields ONLY if translatable fields changed
        /// </summary>
        public async Task<CreatePolicyViewModel> TranslatePolicyAsync(
            CreatePolicyViewModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            CreatePolicyViewModel? oldModel = null)
        {
            // ⭐ CHECK: Only translate if translatable fields changed
            if (oldModel != null)
            {
                var shouldTranslate = ShouldTranslatePolicy(sourceModel, oldModel);
                if (!shouldTranslate)
                {
                    return sourceModel;
                }
            }

            var sourceLang = await _languageService.GetLanguageByIdAsync(sourceLanguageId);
            var targetLang = await _languageService.GetLanguageByIdAsync(targetLanguageId);

            if (sourceLang == null || targetLang == null)
            {
                return sourceModel;
            }

            var changedFields = FieldChangeDetector.GetChangedTranslatableFields(
                sourceModel,
                oldModel,
                TranslatableFieldsConfig.PolicyTranslatableFields
            );

            // Translate only changed fields
            if (changedFields.Contains("PolicyName") && !string.IsNullOrWhiteSpace(sourceModel.PolicyName))
            {
                sourceModel.PolicyName = await _translationService.TranslateTextAsync(
                    sourceModel.PolicyName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("CancellationPolicyDescription") && !string.IsNullOrWhiteSpace(sourceModel.CancellationPolicyDescription))
            {
                sourceModel.CancellationPolicyDescription = await _translationService.TranslateTextAsync(
                    sourceModel.CancellationPolicyDescription,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("BookingPolicyDescription") && !string.IsNullOrWhiteSpace(sourceModel.BookingPolicyDescription))
            {
                sourceModel.BookingPolicyDescription = await _translationService.TranslateTextAsync(
                    sourceModel.BookingPolicyDescription,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("NoShowPolicyDescription") && !string.IsNullOrWhiteSpace(sourceModel.NoShowPolicyDescription))
            {
                sourceModel.NoShowPolicyDescription = await _translationService.TranslateTextAsync(
                    sourceModel.NoShowPolicyDescription,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            return sourceModel;
        }
    }
}
```

### 8. Promotion Translation Helper with Change Detection

```csharp
// V3BookingEngine/Services/PromotionManagementService/PromotionTranslationHelper.cs
namespace V3BookingEngine.Services.PromotionManagementService
{
    public class PromotionTranslationHelper
    {
        private readonly ITranslationService _translationService;
        private readonly ILanguageService _languageService;

        /// <summary>
        /// Checks if translatable fields changed for Promotion
        /// </summary>
        public bool ShouldTranslatePromotion(
            CreatePromotionFormModel newModel,
            CreatePromotionFormModel? oldModel)
        {
            return FieldChangeDetector.HasTranslatableFieldsChanged(
                newModel,
                oldModel,
                TranslatableFieldsConfig.PromotionTranslatableFields
            );
        }

        /// <summary>
        /// Translates promotion fields ONLY if translatable fields changed
        /// </summary>
        public async Task<CreatePromotionFormModel> TranslatePromotionAsync(
            CreatePromotionFormModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            CreatePromotionFormModel? oldModel = null)
        {
            // ⭐ CHECK: Only translate if translatable fields changed
            if (oldModel != null)
            {
                var shouldTranslate = ShouldTranslatePromotion(sourceModel, oldModel);
                if (!shouldTranslate)
                {
                    return sourceModel;
                }
            }

            var sourceLang = await _languageService.GetLanguageByIdAsync(sourceLanguageId);
            var targetLang = await _languageService.GetLanguageByIdAsync(targetLanguageId);

            if (sourceLang == null || targetLang == null)
            {
                return sourceModel;
            }

            var changedFields = FieldChangeDetector.GetChangedTranslatableFields(
                sourceModel,
                oldModel,
                TranslatableFieldsConfig.PromotionTranslatableFields
            );

            // Translate only changed fields
            if (changedFields.Contains("PromotionName") && !string.IsNullOrWhiteSpace(sourceModel.PromotionName))
            {
                sourceModel.PromotionName = await _translationService.TranslateTextAsync(
                    sourceModel.PromotionName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("Description") && !string.IsNullOrWhiteSpace(sourceModel.Description))
            {
                sourceModel.Description = await _translationService.TranslateTextAsync(
                    sourceModel.Description,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            return sourceModel;
        }
    }
}
```

### 9. Property Facilities Translation Helper with Change Detection

```csharp
// V3BookingEngine/Services/PropertyService/PropertyFacilitiesTranslationHelper.cs
namespace V3BookingEngine.Services.PropertyService
{
    public class PropertyFacilitiesTranslationHelper
    {
        private readonly ITranslationService _translationService;
        private readonly ILanguageService _languageService;

        /// <summary>
        /// Checks if translatable fields changed for Property Facilities
        /// </summary>
        public bool ShouldTranslatePropertyFacilities(
            AddGroupFacilityModel newModel,
            AddGroupFacilityModel? oldModel)
        {
            return FieldChangeDetector.HasTranslatableFieldsChanged(
                newModel,
                oldModel,
                TranslatableFieldsConfig.PropertyFacilitiesTranslatableFields
            );
        }

        /// <summary>
        /// Translates property facilities fields ONLY if translatable fields changed
        /// </summary>
        public async Task<AddGroupFacilityModel> TranslatePropertyFacilitiesAsync(
            AddGroupFacilityModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            AddGroupFacilityModel? oldModel = null)
        {
            // ⭐ CHECK: Only translate if translatable fields changed
            if (oldModel != null)
            {
                var shouldTranslate = ShouldTranslatePropertyFacilities(sourceModel, oldModel);
                if (!shouldTranslate)
                {
                    return sourceModel;
                }
            }

            var sourceLang = await _languageService.GetLanguageByIdAsync(sourceLanguageId);
            var targetLang = await _languageService.GetLanguageByIdAsync(targetLanguageId);

            if (sourceLang == null || targetLang == null)
            {
                return sourceModel;
            }

            var changedFields = FieldChangeDetector.GetChangedTranslatableFields(
                sourceModel,
                oldModel,
                TranslatableFieldsConfig.PropertyFacilitiesTranslatableFields
            );

            // Translate only changed fields
            if (changedFields.Contains("GroupName") && !string.IsNullOrWhiteSpace(sourceModel.GroupName))
            {
                sourceModel.GroupName = await _translationService.TranslateTextAsync(
                    sourceModel.GroupName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("FacilityName") && !string.IsNullOrWhiteSpace(sourceModel.FacilityName))
            {
                sourceModel.FacilityName = await _translationService.TranslateTextAsync(
                    sourceModel.FacilityName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("OtherGroupName") && !string.IsNullOrWhiteSpace(sourceModel.OtherGroupName))
            {
                sourceModel.OtherGroupName = await _translationService.TranslateTextAsync(
                    sourceModel.OtherGroupName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("OtherFacilityName") && !string.IsNullOrWhiteSpace(sourceModel.OtherFacilityName))
            {
                sourceModel.OtherFacilityName = await _translationService.TranslateTextAsync(
                    sourceModel.OtherFacilityName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            return sourceModel;
        }
    }
}
```

### 10. Room Facilities Translation Helper with Change Detection

```csharp
// V3BookingEngine/Services/RoomManagementService/RoomFacilitiesTranslationHelper.cs
namespace V3BookingEngine.Services.RoomManagementService
{
    public class RoomFacilitiesTranslationHelper
    {
        private readonly ITranslationService _translationService;
        private readonly ILanguageService _languageService;

        /// <summary>
        /// Checks if translatable fields changed for Room Facilities
        /// </summary>
        public bool ShouldTranslateRoomFacilities(
            AddGroupFacilityModel newModel,
            AddGroupFacilityModel? oldModel)
        {
            return FieldChangeDetector.HasTranslatableFieldsChanged(
                newModel,
                oldModel,
                TranslatableFieldsConfig.RoomFacilitiesTranslatableFields
            );
        }

        /// <summary>
        /// Translates room facilities fields ONLY if translatable fields changed
        /// </summary>
        public async Task<AddGroupFacilityModel> TranslateRoomFacilitiesAsync(
            AddGroupFacilityModel sourceModel,
            int sourceLanguageId,
            int targetLanguageId,
            AddGroupFacilityModel? oldModel = null)
        {
            // ⭐ CHECK: Only translate if translatable fields changed
            if (oldModel != null)
            {
                var shouldTranslate = ShouldTranslateRoomFacilities(sourceModel, oldModel);
                if (!shouldTranslate)
                {
                    return sourceModel;
                }
            }

            var sourceLang = await _languageService.GetLanguageByIdAsync(sourceLanguageId);
            var targetLang = await _languageService.GetLanguageByIdAsync(targetLanguageId);

            if (sourceLang == null || targetLang == null)
            {
                return sourceModel;
            }

            var changedFields = FieldChangeDetector.GetChangedTranslatableFields(
                sourceModel,
                oldModel,
                TranslatableFieldsConfig.RoomFacilitiesTranslatableFields
            );

            // Translate only changed fields
            if (changedFields.Contains("GroupName") && !string.IsNullOrWhiteSpace(sourceModel.GroupName))
            {
                sourceModel.GroupName = await _translationService.TranslateTextAsync(
                    sourceModel.GroupName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("FacilityName") && !string.IsNullOrWhiteSpace(sourceModel.FacilityName))
            {
                sourceModel.FacilityName = await _translationService.TranslateTextAsync(
                    sourceModel.FacilityName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("OtherGroupName") && !string.IsNullOrWhiteSpace(sourceModel.OtherGroupName))
            {
                sourceModel.OtherGroupName = await _translationService.TranslateTextAsync(
                    sourceModel.OtherGroupName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            if (changedFields.Contains("OtherFacilityName") && !string.IsNullOrWhiteSpace(sourceModel.OtherFacilityName))
            {
                sourceModel.OtherFacilityName = await _translationService.TranslateTextAsync(
                    sourceModel.OtherFacilityName,
                    sourceLang.LanguageCode,
                    targetLang.LanguageCode
                );
            }

            return sourceModel;
        }
    }
}
```

### Updated Controller with Change Detection

```csharp
// V3BookingEngine/Controllers/PropertyController.cs

[HttpPost]
public async Task<IActionResult> CreateProperty(CreatePropertyViewModel model)
{
    try
    {
        // ... existing validation code ...

        var ownerId = userId;
        var userlanguageId = int.TryParse(User.Claims.FirstOrDefault(c => c.Type == "LanguageId")?.Value, out var langId) ? langId : 1;
        
        // ⭐ NEW: Check if translatable fields have values before translating
        var hasTranslatableFields = TranslationHelper.HasTranslatableFields(
            model, 
            TranslatableFieldsConfig.PropertyTranslatableFields
        );
        
        string result;
        
        if (userlanguageId == 1)
        {
            // Default language - save to main table
            result = await _propertyManagementService.CreatePropertyAsync(model, ownerId, userlanguageId);
        }
        else
        {
            // Non-default language - check if translation enabled and fields exist
            var isTranslationEnabled = await TranslationHelper.IsTranslationEnabledForPropertyAsync(
                0, // New property, check global setting
                _propertyManagementService.GetConnection()
            );
            
            if (isTranslationEnabled && hasTranslatableFields)
            {
                // Translate and save to ML table
                result = await _propertyManagementService.CreatePropertyWithTranslationAsync(
                    model,
                    ownerId,
                    1, // Source: English
                    userlanguageId, // Target: Selected language
                    autoTranslate: true
                );
            }
            else
            {
                // Save directly to ML table without translation
                result = await _propertyManagementService.CreatePropertyAsync(model, ownerId, userlanguageId);
            }
        }
        
        // ... rest of the code ...
    }
    catch (Exception ex)
    {
        // ... error handling ...
    }
}

[HttpPost]
public async Task<IActionResult> UpdateProperty(CreatePropertyViewModel model, int accommodationId)
{
    try
    {
        // ... existing validation code ...

        var ownerId = userId;
        var userlanguageId = int.TryParse(User.Claims.FirstOrDefault(c => c.Type == "LanguageId")?.Value, out var langId) ? langId : 1;
        
        // ⭐ NEW: Get old model for comparison
        var oldModel = await _propertyManagementService.GetPropertyByIdAsync(accommodationId, userlanguageId);
        
        // ⭐ NEW: Check if translatable fields changed
        var hasTranslatableFieldsChanged = FieldChangeDetector.HasTranslatableFieldsChanged(
            model,
            oldModel,
            TranslatableFieldsConfig.PropertyTranslatableFields
        );
        
        string result;
        
        if (userlanguageId == 1)
        {
            // Default language - update main table
            result = await _propertyManagementService.UpdatePropertyAsync(model, accommodationId, ownerId, userlanguageId);
        }
        else
        {
            // Non-default language
            var isTranslationEnabled = await TranslationHelper.IsTranslationEnabledForPropertyAsync(
                accommodationId,
                _propertyManagementService.GetConnection()
            );
            
            if (isTranslationEnabled && hasTranslatableFieldsChanged)
            {
                // ⭐ Only translate if fields changed
                result = await _propertyManagementService.UpdatePropertyWithTranslationAsync(
                    model,
                    accommodationId,
                    ownerId,
                    1, // Source: English
                    userlanguageId, // Target: Selected language
                    autoTranslate: true
                );
            }
            else
            {
                // Update ML table without translation
                result = await _propertyManagementService.UpdatePropertyAsync(model, accommodationId, ownerId, userlanguageId);
            }
        }
        
        // ... rest of the code ...
    }
    catch (Exception ex)
    {
        // ... error handling ...
    }
}
```

---

## Implementation Flow with Change Detection

### Flow Diagram

```
Admin Creates/Updates Data
    ↓
Check LanguageId
    ↓
Is LanguageId = 1 (Default/English)?
    ├─ YES → Save/Update Main Table
    │         ↓
    │    Check if other languages need translation
    │         ↓
    │    ⭐ Check: Are translatable fields changed?
    │         ├─ YES → Translate → Save to ML Table
    │         └─ NO → Skip translation
    │
    └─ NO → ⭐ Check: Are translatable fields changed?
            ↓
        Is Translation Enabled for Property?
            ├─ YES → ⭐ Check: Do translatable fields have values?
            │         ├─ YES → Translate → Save to ML Table
            │         └─ NO → Save Empty to ML Table
            └─ NO → Save Empty to ML Table (Manual entry)
```

### Decision Logic

```csharp
// Pseudocode for translation decision
bool shouldTranslate = false;

if (languageId != 1) // Not default language
{
    if (isTranslationEnabledForProperty)
    {
        if (isUpdateOperation)
        {
            // For updates: check if translatable fields changed
            shouldTranslate = HasTranslatableFieldsChanged(newModel, oldModel);
        }
        else
        {
            // For creates: check if translatable fields have values
            shouldTranslate = HasTranslatableFields(newModel);
        }
    }
}

if (shouldTranslate)
{
    // Run translation
    TranslateAndSave();
}
else
{
    // Skip translation
    SaveWithoutTranslation();
}
```

---

## Example Scenarios

### Scenario 1: Create Property - Only PropertyName Changed
```
Input:
- AccommodationName: "Grand Hotel" ✅ (Translatable - Changed)
- TimeZoneId: 1 (Non-translatable)
- CurrencyId: 2 (Non-translatable)

Decision: ✅ TRANSLATE (PropertyName has value)
Action: Translate "Grand Hotel" and save to ML table
```

### Scenario 2: Update Property - Only TimeZone Changed
```
Input:
- AccommodationName: "Grand Hotel" (No change)
- TimeZoneId: 2 (Changed from 1) ❌ (Non-translatable)

Decision: ❌ SKIP TRANSLATION (No translatable fields changed)
Action: Update main table only, skip ML table translation
```

### Scenario 3: Update Property - PropertyName Changed
```
Input:
- AccommodationName: "Grand Hotel Premium" ✅ (Changed from "Grand Hotel")
- TimeZoneId: 1 (No change)

Decision: ✅ TRANSLATE (PropertyName changed)
Action: Translate "Grand Hotel Premium" and update ML table
```

### Scenario 4: Create Property - No Translatable Fields
```
Input:
- AccommodationName: "" (Empty)
- TimeZoneId: 1
- CurrencyId: 2

Decision: ❌ SKIP TRANSLATION (No translatable fields with values)
Action: Save to main table only
```

---

## Helper Methods Summary

```csharp
// Check if translatable fields changed (for updates)
FieldChangeDetector.HasTranslatableFieldsChanged(newModel, oldModel, translatableFields)

// Get list of changed translatable fields
FieldChangeDetector.GetChangedTranslatableFields(newModel, oldModel, translatableFields)

// Check if model has translatable fields with values (for creates)
TranslationHelper.HasTranslatableFields(model, translatableFields)

// Check if translation enabled for property
TranslationHelper.IsTranslationEnabledForPropertyAsync(accommodationId, connection)
```

---

## Modules with Field Change Detection Summary ⭐

All modules now support **conditional translation** - translation only runs when translatable fields are changed.

| Module | Translatable Fields | Non-Translatable Fields | Change Detection | Translation Helper |
|--------|-------------------|------------------------|------------------|-------------------|
| **Property** | `AccommodationName` | `AccommodationTypeId`, `PropertyCategory`, `TimeZoneId`, `Country`, `AddressId`, `PostCode`, `CurrencyId`, `ContactNo`, `EmailId`, `Latitude`, `Longitude`, `WebSearch`, `HomeShopping` | ✅ Yes | `PropertyTranslationHelper` |
| **Additional Detail** | `KeyCollectionComments`, `PetsAllowedComments` | All other fields (IDs, dates, booleans) | ✅ Yes | `PropertyTranslationHelper` |
| **Addon** | `ActivityName`, `ShortDescription`, `LongDescription`, `CancellationPolicy`, `GuaranteePolicy` | `ActivityId`, `BookableWithRateplan`, `BookableIndividual`, `MaxBookableQty`, `CurrencyId`, `ActivityBookableTypeId`, `ActivityCategoryTypeIds`, `VariationValues`, `VariationPrices`, `Rates`, `StatusId`, `Image`, `TempImage` | ✅ Yes | `AddonTranslationHelper` |
| **Room** | `RoomName`, `RoomDescription` | `CopyRoomId`, `RoomTypeId`, `ApartmentName`, `TotalGuest`, `RackRate`, `MaxPersons`, `NoOfBeds`, `BedSize`, `RoomSize`, `BathRoomDetail`, `RoomAddress`, `AllArrayBed`, `StayRoomId` | ✅ Yes | `RoomTranslationHelper` |
| **Rate Plan** | `RatePlanName`, `DisplayRatePlanName`, `Included`, `Highlight`, `MealDescription` | `RatePlanId`, `RoomId`, `RatePlan_DefualtCurrencyId`, `RatePlan_OtherCurrencyId`, `DefaultRates`, `GuestQuantity`, `AdultQuantity`, `ChildQuantity`, `DefaultMinStay`, `DefaultMaxStay`, `BookingWindowFrom`, `BookingWindowTo`, `RateCode`, `MealId`, `MealTypeId`, all boolean flags, `PolicyId`, dates, etc. | ✅ Yes | `RatePlanTranslationHelper` |
| **Policy** | `PolicyName`, `CancellationPolicyDescription`, `BookingPolicyDescription`, `NoShowPolicyDescription` | `AccommodationId`, `OwnerId`, `LanguageId`, `CancellationPolicyTypeId`, `BookingPolicyTypeId`, `NoShowPolicyTypeId` | ✅ Yes | `PolicyTranslationHelper` |
| **Promotion** | `PromotionName`, `Description` | `PolicyId`, `PromotionTypeId`, all dates, `BlackDateTypeId`, `BlockDates`, all boolean flags, `VoucherCode`, `MinStay`, `MaxStay`, all ID fields, `DiscountValue`, `DiscountType`, etc. | ✅ Yes | `PromotionTranslationHelper` |
| **Property Facilities** | `GroupName`, `FacilityName`, `OtherGroupName`, `OtherFacilityName` | `Type`, `GroupId`, `FacilityIds`, `GroupIds`, `IsChecked`, `Comments`, `GroupFacilityIds` | ✅ Yes | `PropertyFacilitiesTranslationHelper` |
| **Room Facilities** | `GroupName`, `FacilityName`, `OtherGroupName`, `OtherFacilityName` | `Type`, `GroupId`, `FacilityIds`, `GroupIds`, `IsChecked`, `Comments`, `GroupFacilityIds`, `RoomId`, `RoomIdDrag`, `GFId` | ✅ Yes | `RoomFacilitiesTranslationHelper` |

### How It Works

1. **For Create Operations**:
   - Check if translatable fields have values
   - If YES → Translate and save
   - If NO → Skip translation

2. **For Update Operations**:
   - Compare new model with old model
   - Check if any translatable fields changed
   - If YES → Translate only changed fields
   - If NO → Skip translation (even if other non-translatable fields changed)

### Example: Room Update

```csharp
// Scenario 1: Only RoomName changed
newModel.RoomName = "Deluxe Suite" (changed)
newModel.MaxPersons = 2 (changed, but non-translatable)
oldModel.RoomName = "Standard Room"

Result: ✅ TRANSLATE (RoomName is translatable and changed)

// Scenario 2: Only MaxPersons changed
newModel.MaxPersons = 3 (changed, but non-translatable)
oldModel.MaxPersons = 2
newModel.RoomName = "Deluxe Suite" (no change)
oldModel.RoomName = "Deluxe Suite"

Result: ❌ SKIP TRANSLATION (No translatable fields changed)
```

---

## Performance Benefits

1. **Reduced API Calls**: Only translate when necessary
2. **Faster Updates**: Skip translation for non-translatable field changes
3. **Cost Savings**: Fewer API calls = lower costs
4. **Better UX**: Faster response times

---

## Next Steps

1. ✅ Implement `FieldChangeDetector` helper
2. ✅ Define `TranslatableFieldsConfig` for all modules
3. ✅ Update translation helpers with change detection
4. ✅ Update service methods to use change detection
5. ✅ Update controllers to check field changes
6. ✅ Test with various scenarios
7. ✅ Add logging for translation decisions

---

**Last Updated**: 2024
