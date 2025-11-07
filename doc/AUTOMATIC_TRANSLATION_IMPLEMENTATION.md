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

## Database Operations Based on LanguageId

### Save/Update Logic

#### 1. Save Operation (Create)

```csharp
// V3BookingEngine/Services/PropertyService/PropertyManagementService.cs

public async Task<string> CreatePropertyAsync(
    CreatePropertyViewModel model, 
    int ownerId, 
    int languageId)
{
    try
    {
        await _connection.OpenAsync();

        using var cmd = new SqlCommand("SP_Accommodations_Core", _connection)
        {
            CommandType = CommandType.StoredProcedure
        };

        cmd.Parameters.AddWithValue("@ProcedureType", "I"); // Insert
        cmd.Parameters.AddWithValue("@SessionType", "1");
        cmd.Parameters.AddWithValue("@p_MultiLanguageId", languageId); // ⭐ LanguageId parameter
        
        // ... other parameters ...

        // ⭐ DECISION: Save to main table or ML table based on LanguageId
        if (languageId == 1) // Default language (English)
        {
            // Save to main table: Accommodations
            cmd.Parameters.AddWithValue("@p_AccommodationName", model.AccommodationName);
            // ... other fields ...
        }
        else
        {
            // Save to ML table: Accommodations_ML
            cmd.Parameters.AddWithValue("@p_AccommodationName", model.AccommodationName);
            // ... other translatable fields ...
        }

        var messageOut = new SqlParameter("@MessageOut", SqlDbType.VarChar, 500)
        {
            Direction = ParameterDirection.Output
        };
        cmd.Parameters.Add(messageOut);

        await cmd.ExecuteNonQueryAsync();
        
        var result = messageOut.Value?.ToString() ?? "Failure";
        return result;
    }
    catch (Exception ex)
    {
        await _errorLoggingHelper.LogErrorAsync(_logger, ex, "CreatePropertyAsync", ownerId.ToString());
        return "Failure";
    }
    finally
    {
        if (_connection.State == ConnectionState.Open)
        {
            await _connection.CloseAsync();
        }
    }
}
```

#### 2. Update Operation

```csharp
public async Task<string> UpdatePropertyAsync(
    CreatePropertyViewModel model,
    int accommodationId,
    int ownerId,
    int languageId)
{
    try
    {
        await _connection.OpenAsync();

        using var cmd = new SqlCommand("SP_Accommodations_Core", _connection)
        {
            CommandType = CommandType.StoredProcedure
        };

        cmd.Parameters.AddWithValue("@ProcedureType", "U"); // Update
        cmd.Parameters.AddWithValue("@SessionType", "1");
        cmd.Parameters.AddWithValue("@p_AccommodationId", accommodationId);
        cmd.Parameters.AddWithValue("@p_MultiLanguageId", languageId); // ⭐ LanguageId parameter

        // ⭐ DECISION: Update main table or ML table based on LanguageId
        if (languageId == 1) // Default language
        {
            // Update main table: Accommodations
            cmd.Parameters.AddWithValue("@p_AccommodationName", model.AccommodationName);
        }
        else
        {
            // Update or Insert in ML table: Accommodations_ML
            // Stored procedure should handle: 
            // - If record exists: UPDATE
            // - If record doesn't exist: INSERT
            cmd.Parameters.AddWithValue("@p_AccommodationName", model.AccommodationName);
        }

        var messageOut = new SqlParameter("@MessageOut", SqlDbType.VarChar, 500)
        {
            Direction = ParameterDirection.Output
        };
        cmd.Parameters.Add(messageOut);

        await cmd.ExecuteNonQueryAsync();
        
        return messageOut.Value?.ToString() ?? "Failure";
    }
    catch (Exception ex)
    {
        await _errorLoggingHelper.LogErrorAsync(_logger, ex, "UpdatePropertyAsync", accommodationId.ToString());
        return "Failure";
    }
    finally
    {
        if (_connection.State == ConnectionState.Open)
        {
            await _connection.CloseAsync();
        }
    }
}
```

#### 3. Get Operation (Retrieve)

```csharp
public async Task<CreatePropertyViewModel> GetPropertyByIdAsync(
    int accommodationId, 
    int languageId)
{
    try
    {
        await _connection.OpenAsync();

        using var cmd = new SqlCommand("SP_Accommodations_Core", _connection)
        {
            CommandType = CommandType.StoredProcedure
        };

        cmd.Parameters.AddWithValue("@ProcedureType", "G"); // Get
        cmd.Parameters.AddWithValue("@SessionType", "1");
        cmd.Parameters.AddWithValue("@p_AccommodationId", accommodationId);
        cmd.Parameters.AddWithValue("@p_MultiLanguageId", languageId); // ⭐ LanguageId parameter

        using var adapter = new SqlDataAdapter(cmd);
        var dataSet = new DataSet();
        adapter.Fill(dataSet);

        var model = new CreatePropertyViewModel();

        if (dataSet.Tables.Count > 0 && dataSet.Tables[0].Rows.Count > 0)
        {
            var row = dataSet.Tables[0].Rows[0];
            
            // ⭐ Stored procedure should return data based on LanguageId:
            // - If LanguageId = 1: Get from Accommodations table
            // - If LanguageId != 1: Get from Accommodations_ML table with fallback to Accommodations
            
            model.AccommodationName = row["AccommodationName"]?.ToString() ?? string.Empty;
            model.AccommodationTypeId = Convert.ToInt32(row["AccommodationTypeId"] ?? 0);
            // ... other fields ...
        }

        return model;
    }
    catch (Exception ex)
    {
        await _errorLoggingHelper.LogErrorAsync(_logger, ex, "GetPropertyByIdAsync", accommodationId.ToString());
        return new CreatePropertyViewModel();
    }
    finally
    {
        if (_connection.State == ConnectionState.Open)
        {
            await _connection.CloseAsync();
        }
    }
}
```

### Stored Procedure Example

```sql
-- SP_Accommodations_Core
ALTER PROCEDURE [dbo].[SP_Accommodations_Core]
    @ProcedureType VARCHAR(10),
    @SessionType VARCHAR(10) = '1',
    @p_AccommodationId INT = NULL,
    @p_MultiLanguageId INT = 1, -- ⭐ LanguageId parameter
    @p_AccommodationName NVARCHAR(500) = NULL,
    -- ... other parameters ...
    @MessageOut VARCHAR(500) OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    -- GET Operation
    IF @ProcedureType = 'G'
    BEGIN
        IF @p_MultiLanguageId = 1
        BEGIN
            -- Get from main table (English/Default)
            SELECT 
                a.AccommodationId,
                a.AccommodationName,
                a.AccommodationTypeId,
                -- ... other fields from Accommodations table
            FROM Accommodations a
            WHERE a.AccommodationId = @p_AccommodationId
        END
        ELSE
        BEGIN
            -- Get from ML table with fallback to main table
            SELECT 
                a.AccommodationId,
                ISNULL(at.AccommodationName, a.AccommodationName) AS AccommodationName,
                a.AccommodationTypeId,
                -- ... other fields
            FROM Accommodations a
            LEFT JOIN Accommodations_ML at 
                ON a.AccommodationId = at.AccommodationId 
                AND at.LanguageId = @p_MultiLanguageId
            WHERE a.AccommodationId = @p_AccommodationId
        END
    END

    -- INSERT Operation
    IF @ProcedureType = 'I'
    BEGIN
        IF @p_MultiLanguageId = 1
        BEGIN
            -- Insert into main table
            INSERT INTO Accommodations (
                AccommodationName,
                AccommodationTypeId,
                -- ... other fields
            )
            VALUES (
                @p_AccommodationName,
                @p_AccommodationTypeId,
                -- ... other values
            )
            
            SET @MessageOut = 'Success'
        END
        ELSE
        BEGIN
            -- First insert into main table (if not exists), then into ML table
            -- Get the AccommodationId from main table insert
            DECLARE @NewAccommodationId INT
            
            -- Insert into main table with default values
            INSERT INTO Accommodations (
                AccommodationName, -- Use default or empty
                AccommodationTypeId,
                -- ... other fields
            )
            VALUES (
                '', -- Empty for non-default language
                @p_AccommodationTypeId,
                -- ... other values
            )
            
            SET @NewAccommodationId = SCOPE_IDENTITY()
            
            -- Insert into ML table
            INSERT INTO Accommodations_ML (
                AccommodationId,
                LanguageId,
                AccommodationName
            )
            VALUES (
                @NewAccommodationId,
                @p_MultiLanguageId,
                @p_AccommodationName
            )
            
            SET @MessageOut = 'Success'
        END
    END

    -- UPDATE Operation
    IF @ProcedureType = 'U'
    BEGIN
        IF @p_MultiLanguageId = 1
        BEGIN
            -- Update main table
            UPDATE Accommodations
            SET AccommodationName = @p_AccommodationName,
                -- ... other fields
            WHERE AccommodationId = @p_AccommodationId
            
            SET @MessageOut = 'Success'
        END
        ELSE
        BEGIN
            -- Update or Insert in ML table (UPSERT)
            IF EXISTS (
                SELECT 1 FROM Accommodations_ML 
                WHERE AccommodationId = @p_AccommodationId 
                AND LanguageId = @p_MultiLanguageId
            )
            BEGIN
                -- UPDATE existing ML record
                UPDATE Accommodations_ML
                SET AccommodationName = @p_AccommodationName,
                    UpdatedDate = GETDATE()
                WHERE AccommodationId = @p_AccommodationId 
                AND LanguageId = @p_MultiLanguageId
            END
            ELSE
            BEGIN
                -- INSERT new ML record
                INSERT INTO Accommodations_ML (
                    AccommodationId,
                    LanguageId,
                    AccommodationName
                )
                VALUES (
                    @p_AccommodationId,
                    @p_MultiLanguageId,
                    @p_AccommodationName
                )
            END
            
            SET @MessageOut = 'Success'
        END
    END
END
```

---

## Frontend Language Dropdown Implementation

### Header Language Dropdown (Next to "Find Property")

#### 1. HTML Structure

```html
<!-- Views/Shared/_Layout.cshtml or Views/Shared/_Header.cshtml -->

<header class="main-header">
    <div class="container">
        <div class="header-content">
            <!-- Logo -->
            <div class="logo">
                <a href="/">BookingWhizz</a>
            </div>

            <!-- Navigation -->
            <nav class="main-nav">
                <ul>
                    <li><a href="/Property/PropertiesList">Find Property</a></li>
                    <!-- ... other menu items ... -->
                </ul>
            </nav>

            <!-- Language Dropdown -->
            <div class="language-selector">
                <select id="languageDropdown" class="form-select language-dropdown">
                    <option value="1" data-code="en">English</option>
                    <option value="2" data-code="ar">العربية (Arabic)</option>
                    <option value="3" data-code="ur">اردو (Urdu)</option>
                    <option value="4" data-code="fr">Français (French)</option>
                    <option value="5" data-code="es">Español (Spanish)</option>
                </select>
            </div>
        </div>
    </div>
</header>
```

#### 2. JavaScript Implementation

```javascript
// wwwroot/js/language-selector.js

$(document).ready(function() {
    // Get current language from cookie or default to 1 (English)
    var currentLanguageId = getCookie('SelectedLanguageId') || '1';
    $('#languageDropdown').val(currentLanguageId);

    // Language change event
    $('#languageDropdown').on('change', function() {
        var selectedLanguageId = $(this).val();
        var selectedLanguageCode = $(this).find('option:selected').data('code');
        
        // Save to cookie
        setCookie('SelectedLanguageId', selectedLanguageId, 365);
        
        // Update all API calls to use new LanguageId
        updateLanguageId(selectedLanguageId);
        
        // Reload current page data with new language
        reloadPageData(selectedLanguageId);
        
        // Optional: Show loading indicator
        showLanguageLoading();
    });

    function updateLanguageId(languageId) {
        // Update global variable
        window.currentLanguageId = languageId;
        
        // Update all AJAX calls to include LanguageId
        $.ajaxSetup({
            beforeSend: function(xhr, settings) {
                if (settings.url && settings.url.indexOf('/api/') !== -1) {
                    // Add LanguageId to API calls
                    if (settings.url.indexOf('?') !== -1) {
                        settings.url += '&languageId=' + languageId;
                    } else {
                        settings.url += '?languageId=' + languageId;
                    }
                }
            }
        });
    }

    function reloadPageData(languageId) {
        // Reload property list if on properties page
        if (window.location.pathname.indexOf('/Property/PropertiesList') !== -1) {
            loadPropertiesList(languageId);
        }
        
        // Reload property details if on property detail page
        if (window.location.pathname.indexOf('/Property/PropertyDetails') !== -1) {
            var propertyId = getPropertyIdFromUrl();
            loadPropertyDetails(propertyId, languageId);
        }
        
        // Add other page-specific reload logic here
    }

    function loadPropertiesList(languageId) {
        $.ajax({
            url: '/api/Property/GetPropertiesList',
            type: 'GET',
            data: { languageId: languageId },
            success: function(response) {
                if (response.success) {
                    renderPropertiesList(response.data);
                }
            },
            error: function(xhr, status, error) {
                console.error('Error loading properties:', error);
            }
        });
    }

    function loadPropertyDetails(propertyId, languageId) {
        $.ajax({
            url: '/api/Property/GetProperty/' + propertyId,
            type: 'GET',
            data: { languageId: languageId },
            success: function(response) {
                if (response.success) {
                    renderPropertyDetails(response.data);
                }
            },
            error: function(xhr, status, error) {
                console.error('Error loading property details:', error);
            }
        });
    }

    // Cookie helper functions
    function setCookie(name, value, days) {
        var expires = "";
        if (days) {
            var date = new Date();
            date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
            expires = "; expires=" + date.toUTCString();
        }
        document.cookie = name + "=" + (value || "") + expires + "; path=/";
    }

    function getCookie(name) {
        var nameEQ = name + "=";
        var ca = document.cookie.split(';');
        for (var i = 0; i < ca.length; i++) {
            var c = ca[i];
            while (c.charAt(0) == ' ') c = c.substring(1, c.length);
            if (c.indexOf(nameEQ) == 0) return c.substring(nameEQ.length, c.length);
        }
        return null;
    }

    function showLanguageLoading() {
        // Show loading indicator
        $('body').append('<div id="languageLoading" class="language-loading"><i class="fas fa-spinner fa-spin"></i> Loading...</div>');
        
        setTimeout(function() {
            $('#languageLoading').fadeOut(function() {
                $(this).remove();
            });
        }, 1000);
    }
});
```

#### 3. CSS Styling

```css
/* wwwroot/css/language-selector.css */

.language-selector {
    display: inline-block;
    margin-left: 20px;
}

.language-dropdown {
    min-width: 150px;
    padding: 8px 12px;
    border: 1px solid #ddd;
    border-radius: 4px;
    background-color: #fff;
    font-size: 14px;
    cursor: pointer;
}

.language-dropdown:focus {
    outline: none;
    border-color: #007bff;
    box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.language-loading {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background: rgba(0, 0, 0, 0.8);
    color: white;
    padding: 20px 40px;
    border-radius: 8px;
    z-index: 9999;
    font-size: 16px;
}

/* RTL Support for Arabic */
[dir="rtl"] .language-selector {
    margin-left: 0;
    margin-right: 20px;
}
```

#### 4. API Controller Update

```csharp
// V3BookingEngine/Controllers/Api/PropertyApiController.cs

[ApiController]
[Route("api/[controller]")]
public class PropertyApiController : ControllerBase
{
    private readonly IPropertyManagementService _propertyService;
    private readonly ILanguageService _languageService;

    [HttpGet("GetPropertiesList")]
    public async Task<IActionResult> GetPropertiesList(
        [FromQuery] int languageId = 1, // ⭐ Default to English
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 10)
    {
        try
        {
            // Validate languageId
            var language = await _languageService.GetLanguageByIdAsync(languageId);
            if (language == null)
            {
                languageId = 1; // Fallback to default
            }

            // Get properties with LanguageId
            var properties = await _propertyService.GetPropertiesListAsync(
                languageId, 
                page, 
                pageSize
            );

            return Ok(new 
            { 
                success = true, 
                data = properties,
                languageId = languageId,
                languageCode = language?.LanguageCode ?? "en"
            });
        }
        catch (Exception ex)
        {
            return StatusCode(500, new { success = false, message = ex.Message });
        }
    }

    [HttpGet("GetProperty/{id}")]
    public async Task<IActionResult> GetProperty(
        int id, 
        [FromQuery] int languageId = 1) // ⭐ LanguageId from query string
    {
        try
        {
            // Validate languageId
            var language = await _languageService.GetLanguageByIdAsync(languageId);
            if (language == null)
            {
                languageId = 1; // Fallback to default
            }

            // Get property with LanguageId
            var property = await _propertyService.GetPropertyByIdAsync(id, languageId);

            if (property == null)
            {
                return NotFound(new { success = false, message = "Property not found" });
            }

            return Ok(new 
            { 
                success = true, 
                data = property,
                languageId = languageId,
                languageCode = language?.LanguageCode ?? "en"
            });
        }
        catch (Exception ex)
        {
            return StatusCode(500, new { success = false, message = ex.Message });
        }
    }
}
```

#### 5. Service Method Update

```csharp
// V3BookingEngine/Services/PropertyService/PropertyManagementService.cs

public async Task<List<PropertyViewModel>> GetPropertiesListAsync(
    int languageId, 
    int page = 1, 
    int pageSize = 10)
{
    try
    {
        await _connection.OpenAsync();

        using var cmd = new SqlCommand("SP_Accommodations_GetList", _connection)
        {
            CommandType = CommandType.StoredProcedure
        };

        cmd.Parameters.AddWithValue("@p_MultiLanguageId", languageId); // ⭐ LanguageId
        cmd.Parameters.AddWithValue("@p_Page", page);
        cmd.Parameters.AddWithValue("@p_PageSize", pageSize);

        using var adapter = new SqlDataAdapter(cmd);
        var dataSet = new DataSet();
        adapter.Fill(dataSet);

        var properties = new List<PropertyViewModel>();

        if (dataSet.Tables.Count > 0)
        {
            foreach (DataRow row in dataSet.Tables[0].Rows)
            {
                properties.Add(new PropertyViewModel
                {
                    AccommodationId = Convert.ToInt32(row["AccommodationId"]),
                    AccommodationName = row["AccommodationName"]?.ToString() ?? string.Empty,
                    // ... other fields based on LanguageId
                });
            }
        }

        return properties;
    }
    catch (Exception ex)
    {
        await _errorLoggingHelper.LogErrorAsync(_logger, ex, "GetPropertiesListAsync", languageId.ToString());
        return new List<PropertyViewModel>();
    }
    finally
    {
        if (_connection.State == ConnectionState.Open)
        {
            await _connection.CloseAsync();
        }
    }
}
```

### Language Dropdown with Dynamic Loading

```csharp
// V3BookingEngine/Controllers/HomeController.cs or PropertyController.cs

[HttpGet]
public async Task<IActionResult> GetLanguages()
{
    try
    {
        var languages = await _languageService.GetAllLanguagesAsync();
        
        var languageList = languages.Select(l => new
        {
            LanguageId = l.LanguageId,
            LanguageCode = l.LanguageCode,
            LanguageName = l.LanguageName,
            IsDefault = l.IsDefault
        }).ToList();

        return Json(new { success = true, data = languageList });
    }
    catch (Exception ex)
    {
        return Json(new { success = false, message = ex.Message });
    }
}
```

```javascript
// Load languages dynamically
$(document).ready(function() {
    loadLanguages();
});

function loadLanguages() {
    $.ajax({
        url: '/Home/GetLanguages',
        type: 'GET',
        success: function(response) {
            if (response.success) {
                var dropdown = $('#languageDropdown');
                dropdown.empty();
                
                response.data.forEach(function(lang) {
                    var option = $('<option></option>')
                        .attr('value', lang.LanguageId)
                        .attr('data-code', lang.LanguageCode)
                        .text(lang.LanguageName);
                    
                    if (lang.IsDefault) {
                        option.attr('selected', 'selected');
                    }
                    
                    dropdown.append(option);
                });
            }
        }
    });
}
```

---

## Complete Flow Diagram

```
User Selects Language from Dropdown
    ↓
JavaScript saves LanguageId to Cookie
    ↓
All API calls include ?languageId=X parameter
    ↓
API Controller receives LanguageId
    ↓
Service Method calls Stored Procedure with @p_MultiLanguageId
    ↓
Stored Procedure:
    - If LanguageId = 1 → Query Accommodations table
    - If LanguageId != 1 → Query Accommodations_ML table with LEFT JOIN to Accommodations (fallback)
    ↓
Return data in selected language
    ↓
Frontend displays translated content
```

---

## Property-Level Translation Settings

### Database Schema Changes

#### Add Columns to Accommodations Table

```sql
-- Add translation settings columns to Accommodations table
ALTER TABLE Accommodations
ADD AutoTranslateEnabled BIT DEFAULT 1,  -- Enable automatic translation for this property
    EnableForProperty BIT DEFAULT 0;      -- Property-level translation toggle

-- Add index for performance
CREATE INDEX IX_Accommodations_AutoTranslateEnabled 
ON Accommodations(AutoTranslateEnabled);

CREATE INDEX IX_Accommodations_EnableForProperty 
ON Accommodations(EnableForProperty);
```

### Update CreateProperty and UpdateProperty

#### 1. Update CreatePropertyViewModel

```csharp
// V3BookingEngine/DTOs/Property/CreatePropertyViewModel.cs

namespace V3BookingEngine.DTOs.Property
{
    public class CreatePropertyViewModel
    {
        // ... existing fields ...

        [Display(Name = "Auto Translate Enabled")]
        public bool AutoTranslateEnabled { get; set; } = true;

        [Display(Name = "Enable Translation for Property")]
        public bool EnableForProperty { get; set; } = false;
    }
}
```

#### 2. Update CreateProperty Service Method

```csharp
// V3BookingEngine/Services/PropertyService/PropertyManagementService.cs

public async Task<string> CreatePropertyAsync(
    CreatePropertyViewModel model, 
    int ownerId, 
    int languageId)
{
    try
    {
        await _connection.OpenAsync();

        using var cmd = new SqlCommand("SP_Accommodations_Core", _connection)
        {
            CommandType = CommandType.StoredProcedure
        };

        cmd.Parameters.AddWithValue("@ProcedureType", "I");
        cmd.Parameters.AddWithValue("@SessionType", "1");
        cmd.Parameters.AddWithValue("@p_MultiLanguageId", languageId);
        cmd.Parameters.AddWithValue("@p_OwnerId", ownerId);
        cmd.Parameters.AddWithValue("@p_AccommodationName", model.AccommodationName.Trim());
        
        // ⭐ NEW: Add translation settings
        cmd.Parameters.AddWithValue("@p_AutoTranslateEnabled", model.AutoTranslateEnabled);
        cmd.Parameters.AddWithValue("@p_EnableForProperty", model.EnableForProperty);
        
        // ... other parameters ...

        var messageOut = new SqlParameter("@MessageOut", SqlDbType.VarChar, 500)
        {
            Direction = ParameterDirection.Output
        };
        cmd.Parameters.Add(messageOut);

        await cmd.ExecuteNonQueryAsync();
        
        var result = messageOut.Value?.ToString() ?? "Failure";
        return result;
    }
    catch (Exception ex)
    {
        await _errorLoggingHelper.LogErrorAsync(_logger, ex, "CreatePropertyAsync", ownerId.ToString());
        return "Failure";
    }
    finally
    {
        if (_connection.State == ConnectionState.Open)
        {
            await _connection.CloseAsync();
        }
    }
}
```

#### 3. Update UpdateProperty Service Method

```csharp
public async Task<string> UpdatePropertyAsync(
    CreatePropertyViewModel model,
    int accommodationId,
    int ownerId,
    int languageId)
{
    try
    {
        await _connection.OpenAsync();

        using var cmd = new SqlCommand("SP_Accommodations_Core", _connection)
        {
            CommandType = CommandType.StoredProcedure
        };

        cmd.Parameters.AddWithValue("@ProcedureType", "U");
        cmd.Parameters.AddWithValue("@SessionType", "1");
        cmd.Parameters.AddWithValue("@p_AccommodationId", accommodationId);
        cmd.Parameters.AddWithValue("@p_MultiLanguageId", languageId);
        cmd.Parameters.AddWithValue("@p_AccommodationName", model.AccommodationName.Trim());
        
        // ⭐ NEW: Add translation settings
        cmd.Parameters.AddWithValue("@p_AutoTranslateEnabled", model.AutoTranslateEnabled);
        cmd.Parameters.AddWithValue("@p_EnableForProperty", model.EnableForProperty);
        
        // ... other parameters ...

        var messageOut = new SqlParameter("@MessageOut", SqlDbType.VarChar, 500)
        {
            Direction = ParameterDirection.Output
        };
        cmd.Parameters.Add(messageOut);

        await cmd.ExecuteNonQueryAsync();
        
        return messageOut.Value?.ToString() ?? "Failure";
    }
    catch (Exception ex)
    {
        await _errorLoggingHelper.LogErrorAsync(_logger, ex, "UpdatePropertyAsync", accommodationId.ToString());
        return "Failure";
    }
    finally
    {
        if (_connection.State == ConnectionState.Open)
        {
            await _connection.CloseAsync();
        }
    }
}
```

#### 4. Update Stored Procedure

```sql
-- SP_Accommodations_Core
ALTER PROCEDURE [dbo].[SP_Accommodations_Core]
    @ProcedureType VARCHAR(10),
    @SessionType VARCHAR(10) = '1',
    @p_AccommodationId INT = NULL,
    @p_MultiLanguageId INT = 1,
    @p_OwnerId INT = NULL,
    @p_AccommodationName NVARCHAR(500) = NULL,
    @p_AutoTranslateEnabled BIT = 1,  -- ⭐ NEW parameter
    @p_EnableForProperty BIT = 0,     -- ⭐ NEW parameter
    -- ... other parameters ...
    @MessageOut VARCHAR(500) OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    -- INSERT Operation
    IF @ProcedureType = 'I'
    BEGIN
        INSERT INTO Accommodations (
            AccommodationName,
            OwnerId,
            AutoTranslateEnabled,      -- ⭐ NEW column
            EnableForProperty,         -- ⭐ NEW column
            -- ... other columns ...
        )
        VALUES (
            @p_AccommodationName,
            @p_OwnerId,
            @p_AutoTranslateEnabled,   -- ⭐ NEW value
            @p_EnableForProperty,      -- ⭐ NEW value
            -- ... other values ...
        )
        
        SET @MessageOut = 'Success'
    END

    -- UPDATE Operation
    IF @ProcedureType = 'U'
    BEGIN
        UPDATE Accommodations
        SET AccommodationName = @p_AccommodationName,
            AutoTranslateEnabled = @p_AutoTranslateEnabled,  -- ⭐ NEW column
            EnableForProperty = @p_EnableForProperty,        -- ⭐ NEW column
            -- ... other fields ...
        WHERE AccommodationId = @p_AccommodationId
        
        SET @MessageOut = 'Success'
    END

    -- GET Operation
    IF @ProcedureType = 'G'
    BEGIN
        SELECT 
            a.AccommodationId,
            a.AccommodationName,
            a.AutoTranslateEnabled,    -- ⭐ NEW column
            a.EnableForProperty,        -- ⭐ NEW column
            -- ... other fields ...
        FROM Accommodations a
        WHERE a.AccommodationId = @p_AccommodationId
    END
END
```

### Get Translation Settings After Login

**Important**: `AutoTranslateEnabled` and `EnableForProperty` values are fetched in the **same login query** (`SP_Admin_Core`) by joining with the `Accommodations` table. No separate query is needed.

#### 1. Update User Model

```csharp
// V3BookingEngine/CustomModel/User.cs

namespace V3BookingEngine.CustomModel
{
    public class User
    {
        // ... existing fields ...
        public int PropertyId { get; set; }
        public int LanguageId { get; set; }
        
        // ⭐ NEW: Add translation settings fields
        public bool AutoTranslateEnabled { get; set; } = true;
        public bool EnableForProperty { get; set; } = false;
        
        // ... other fields ...
    }
}
```

#### 2. Update SP_Admin_Core Stored Procedure

```sql
-- SP_Admin_Core (Update the SELECT query to include translation settings)
-- In the login query (ProcedureType = 'V' or 'G'), add LEFT JOIN with Accommodations:

ALTER PROCEDURE [dbo].[SP_Admin_Core]
    @ProcedureType VARCHAR(10),
    @SessionType VARCHAR(10) = '103',
    @p_LoginId VARCHAR(100) = NULL,
    @p_UserId INT = NULL,
    -- ... other parameters ...
AS
BEGIN
    SET NOCOUNT ON;

    -- Login/Get User Query
    IF @ProcedureType = 'V' OR @ProcedureType = 'G'
    BEGIN
        SELECT 
            u.UserId,
            u.UserName,
            u.LoginId,
            u.GroupId,
            u.RoleId,
            u.PropertyId,
            u.LanguageId,
            -- ... other user fields ...
            
            -- ⭐ NEW: Add translation settings from Accommodations table
            ISNULL(a.AutoTranslateEnabled, 1) AS AutoTranslateEnabled,
            ISNULL(a.EnableForProperty, 0) AS EnableForProperty
        FROM Users u
        LEFT JOIN Accommodations a ON u.PropertyId = a.AccommodationId
        WHERE (@p_LoginId IS NOT NULL AND u.LoginId = @p_LoginId)
           OR (@p_UserId IS NOT NULL AND u.UserId = @p_UserId)
    END
    
    -- ... other procedure types ...
END
```

#### 3. Update UserService to Read Translation Settings

```csharp
// V3BookingEngine/Services/AuthService/UserService.cs

public async Task<User?> ValidateUserAsync(string username, string password)
{
    try
    {
        // ... existing code ...
        
        using var reader = await cmd.ExecuteReaderAsync();

        if (!reader.HasRows || !await reader.ReadAsync())
        {
            return new User { Message = "User not found" };
        }

        var user = new User
        {
            // ... existing fields ...
            PropertyId = reader.IsDBNull("PropertyId") ? 0 : Convert.ToInt32(reader["PropertyId"]),
            LanguageId = reader.IsDBNull("LanguageId") ? 1 : Convert.ToInt32(reader["LanguageId"]),
            
            // ⭐ NEW: Read translation settings from query result
            AutoTranslateEnabled = reader.IsDBNull("AutoTranslateEnabled") 
                ? true 
                : Convert.ToBoolean(reader["AutoTranslateEnabled"]),
            EnableForProperty = reader.IsDBNull("EnableForProperty") 
                ? false 
                : Convert.ToBoolean(reader["EnableForProperty"]),
            
            // ... other fields ...
        };
        
        // ... password validation code ...
        
        return user;
    }
    catch (Exception ex)
    {
        // ... error handling ...
    }
}

public async Task<User> GetUserbyLoginIdAsync(string username)
{
    try
    {
        // ... existing code ...
        
        var user = new User
        {
            // ... existing fields ...
            
            // ⭐ NEW: Read translation settings from query result
            AutoTranslateEnabled = reader.IsDBNull("AutoTranslateEnabled") 
                ? true 
                : Convert.ToBoolean(reader["AutoTranslateEnabled"]),
            EnableForProperty = reader.IsDBNull("EnableForProperty") 
                ? false 
                : Convert.ToBoolean(reader["EnableForProperty"]),
            
            // ... other fields ...
        };
        
        return user;
    }
    catch (Exception ex)
    {
        // ... error handling ...
    }
}
```

#### 4. Update Login Method to Add Claims

```csharp
// V3BookingEngine/Controllers/AccountController.cs

[HttpPost]
public async Task<IActionResult> Login(LoginViewModel model)
{
    try
    {
        // ... existing login validation code ...

        // Validate user
        var user = await _userService.ValidateUserAsync(model.LoginId, model.Password);
        
        if (user == null)
        {
            ModelState.AddModelError("", "Invalid login credentials");
            return View(model);
        }

        // Create claims (translation settings are already in user object from login query)
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.NameIdentifier, user.UserId.ToString()),
            new Claim("UserId", user.UserId.ToString()),
            new Claim("UserName", user.UserName ?? ""),
            new Claim("LoginId", user.LoginId ?? ""),
            new Claim("GroupId", user.GroupId.ToString()),
            new Claim("RoleId", user.RoleId.ToString()),
            new Claim("LanguageId", user.LanguageId.ToString()),
            new Claim("PropertyId", user.PropertyId.ToString()),
            
            // ⭐ NEW: Add translation settings to claims (from user object, same as other claims)
            new Claim("AutoTranslateEnabled", user.AutoTranslateEnabled.ToString()),
            new Claim("EnableForProperty", user.EnableForProperty.ToString())
        };

        var claimsIdentity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);
        var authProperties = new AuthenticationProperties
        {
            IsPersistent = model.RememberMe
        };

        await HttpContext.SignInAsync(
            CookieAuthenticationDefaults.AuthenticationScheme,
            new ClaimsPrincipal(claimsIdentity),
            authProperties);

        return RedirectToAction("Index", "Home");
    }
    catch (Exception ex)
    {
        await _errorLoggingHelper.LogErrorAsync(_logger, ex, "Login", "Anonymous");
        ModelState.AddModelError("", "System Error: Contact Technical Support");
        return View(model);
    }
}
```

#### 5. Update OTP Verification Method (if OTP is used)

```csharp
[HttpPost]
public async Task<IActionResult> VerifyOtp(VerifyOtpViewModel model)
{
    try
    {
        // ... existing OTP verification code ...

        // Get user (translation settings are already in user object from query)
        var user = await _userService.GetUserbyLoginIdAsync(model.LoginId);
        
        if (user == null)
        {
            return Json(new { success = false, message = "User not found" });
        }

        // Create claims (translation settings are already in user object from login query)
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.NameIdentifier, user.UserId.ToString()),
            new Claim("UserId", user.UserId.ToString()),
            new Claim("UserName", user.UserName ?? ""),
            new Claim("LoginId", user.LoginId ?? ""),
            new Claim("GroupId", user.GroupId.ToString()),
            new Claim("RoleId", user.RoleId.ToString()),
            new Claim("LanguageId", user.LanguageId.ToString()),
            new Claim("PropertyId", user.PropertyId.ToString()),
            
            // ⭐ NEW: Add translation settings to claims (from user object, same as other claims)
            new Claim("AutoTranslateEnabled", user.AutoTranslateEnabled.ToString()),
            new Claim("EnableForProperty", user.EnableForProperty.ToString())
        };

        var claimsIdentity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);
        
        await HttpContext.SignInAsync(
            CookieAuthenticationDefaults.AuthenticationScheme,
            new ClaimsPrincipal(claimsIdentity),
            new AuthenticationProperties());

        return Json(new { success = true, redirectUrl = "/Home/Index" });
    }
    catch (Exception ex)
    {
        await _errorLoggingHelper.LogErrorAsync(_logger, ex, "VerifyOtp", "Anonymous");
        return Json(new { success = false, message = "System Error" });
    }
}
```

#### 6. Helper Method to Get Translation Settings from Claims

```csharp
// V3BookingEngine/Helpers/TranslationHelper.cs

namespace V3BookingEngine.Helpers
{
    public static class TranslationHelper
    {
        /// <summary>
        /// Gets AutoTranslateEnabled from claims
        /// </summary>
        public static bool GetAutoTranslateEnabled(ClaimsPrincipal user)
        {
            var claim = user.FindFirst("AutoTranslateEnabled");
            if (claim != null && bool.TryParse(claim.Value, out var value))
            {
                return value;
            }
            return true; // Default to true
        }

        /// <summary>
        /// Gets EnableForProperty from claims
        /// </summary>
        public static bool GetEnableForProperty(ClaimsPrincipal user)
        {
            var claim = user.FindFirst("EnableForProperty");
            if (claim != null && bool.TryParse(claim.Value, out var value))
            {
                return value;
            }
            return false; // Default to false
        }

        /// <summary>
        /// Checks if translation is enabled for property (from claims or database)
        /// </summary>
        public static async Task<bool> IsTranslationEnabledForPropertyAsync(
            int accommodationId,
            IDbConnection connection,
            ClaimsPrincipal? user = null)
        {
            // First check claims if user is logged in
            if (user != null)
            {
                var propertyIdClaim = user.FindFirst("PropertyId");
                if (propertyIdClaim != null && 
                    int.TryParse(propertyIdClaim.Value, out var userPropertyId) &&
                    userPropertyId == accommodationId)
                {
                    var enableForProperty = GetEnableForProperty(user);
                    var autoTranslateEnabled = GetAutoTranslateEnabled(user);
                    
                    return enableForProperty && autoTranslateEnabled;
                }
            }

            // Fallback: Query database
            // ... existing database query logic ...
            return false;
        }
    }
}
```

#### 7. Update Controller to Use Claims

```csharp
// V3BookingEngine/Controllers/PropertyController.cs

[HttpPost]
public async Task<IActionResult> CreateProperty(CreatePropertyViewModel model)
{
    try
    {
        // ... existing validation code ...

        var ownerId = userId;
        var userlanguageId = int.TryParse(User.FindFirst("LanguageId")?.Value, out var langId) ? langId : 1;
        
        // ⭐ NEW: Get translation settings from claims
        var autoTranslateEnabled = TranslationHelper.GetAutoTranslateEnabled(User);
        var enableForProperty = TranslationHelper.GetEnableForProperty(User);
        
        // Set model values from claims (or let user override in form)
        model.AutoTranslateEnabled = autoTranslateEnabled;
        model.EnableForProperty = enableForProperty;
        
        // ... rest of the code ...
    }
    catch (Exception ex)
    {
        // ... error handling ...
    }
}

[HttpGet]
public async Task<IActionResult> UpdateProperty(int accommodationId)
{
    try
    {
        // ... existing code ...
        
        var userlanguageId = int.TryParse(User.FindFirst("LanguageId")?.Value, out var langId) ? langId : 1;
        var model = await _propertyManagementService.GetPropertyByIdAsync(accommodationId, userlanguageId);
        
        // ⭐ NEW: Get translation settings from claims
        model.AutoTranslateEnabled = TranslationHelper.GetAutoTranslateEnabled(User);
        model.EnableForProperty = TranslationHelper.GetEnableForProperty(User);
        
        return View(model);
    }
    catch (Exception ex)
    {
        // ... error handling ...
    }
}
```

#### 8. Update CreateProperty View (Optional - if you want user to edit these fields)

```html
<!-- Views/Property/CreateProperty.cshtml -->

<div class="form-group">
    <div class="form-check">
        <input asp-for="AutoTranslateEnabled" class="form-check-input" type="checkbox" id="AutoTranslateEnabled" />
        <label class="form-check-label" for="AutoTranslateEnabled">
            Enable Automatic Translation
        </label>
        <small class="form-text text-muted">
            When enabled, translatable fields will be automatically translated when saved in non-default languages.
        </small>
    </div>
</div>

<div class="form-group">
    <div class="form-check">
        <input asp-for="EnableForProperty" class="form-check-input" type="checkbox" id="EnableForProperty" />
        <label class="form-check-label" for="EnableForProperty">
            Enable Translation for This Property
        </label>
        <small class="form-text text-muted">
            Enable multi-language support for this property. This setting will be saved to your account claims.
        </small>
    </div>
</div>
```

### Complete Flow

```
User Logs In
    ↓
SP_Admin_Core Query (with LEFT JOIN Accommodations)
    ↓
User Object Contains:
    - UserId, PropertyId, LanguageId, etc.
    - AutoTranslateEnabled (from Accommodations table)
    - EnableForProperty (from Accommodations table)
    ↓
Add All Values to Claims (same way as PropertyId, UserId, etc.)
    ↓
User Creates/Updates Property
    ↓
Get values from Claims (or form if editable)
    ↓
Save to Accommodations table
    ↓
Next Login: Values loaded from database again (same query)
```

**Key Point**: No separate query needed! Translation settings are fetched in the same login query by joining with `Accommodations` table, just like `PropertyId` and other user properties.

---

## Performance Benefits

1. **Reduced API Calls**: Only translate when necessary
2. **Faster Updates**: Skip translation for non-translatable field changes
3. **Cost Savings**: Fewer API calls = lower costs
4. **Better UX**: Faster response times

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

-- Index for performance
CREATE INDEX IX_PerLanguageSettings_AccommodationId 
ON PerLanguageSettings(AccommodationId);

CREATE INDEX IX_PerLanguageSettings_LanguageId 
ON PerLanguageSettings(LanguageId);
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

### Service Method Signature

```csharp
// V3BookingEngine/Services/PropertyService/PropertyManagementService.cs

public async Task<bool> IsTranslationEnabledForLanguageAsync(
    int accommodationId, 
    int languageId)
{
    // 1. Check PerLanguageSettings table first
    // 2. If not found, fallback to property-level AutoTranslateEnabled
    // 3. Return true/false
}
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

-- Indexes for performance
CREATE INDEX IX_TranslationHistory_AccommodationId 
ON TranslationHistory(AccommodationId);

CREATE INDEX IX_TranslationHistory_TableName_RecordId 
ON TranslationHistory(TableName, RecordId);

CREATE INDEX IX_TranslationHistory_LanguageId 
ON TranslationHistory(LanguageId);

CREATE INDEX IX_TranslationHistory_TranslationDate 
ON TranslationHistory(TranslationDate DESC);
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

### Service Method Signature

```csharp
// V3BookingEngine/Services/TranslationService/TranslationHistoryService.cs

public interface ITranslationHistoryService
{
    Task LogTranslationAsync(TranslationHistoryLog log);
    Task<List<TranslationHistory>> GetHistoryByPropertyAsync(int accommodationId);
    Task<List<TranslationHistory>> GetHistoryByLanguageAsync(int accommodationId, int languageId);
    Task<List<TranslationHistory>> GetHistoryByFieldAsync(string tableName, int recordId, string fieldName);
    Task<TranslationHistory> GetLatestTranslationAsync(string tableName, int recordId, string fieldName, int languageId);
    Task RevertTranslationAsync(long historyId);
}

public class TranslationHistoryLog
{
    public int AccommodationId { get; set; }
    public string TableName { get; set; }
    public int RecordId { get; set; }
    public int LanguageId { get; set; }
    public string FieldName { get; set; }
    public string OriginalText { get; set; }
    public string TranslatedText { get; set; }
    public string TranslationMethod { get; set; } // 'API_AUTO', 'MANUAL', 'BULK_IMPORT'
    public string TranslationProvider { get; set; } // 'DeepL', 'LibreTranslate', 'Manual'
    public string ApiCallId { get; set; }
    public int TranslatedBy { get; set; }
}
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

## Next Steps

1. ✅ Implement `FieldChangeDetector` helper
2. ✅ Define `TranslatableFieldsConfig` for all modules
3. ✅ Update translation helpers with change detection
4. ✅ Update service methods to use change detection
5. ✅ Update controllers to check field changes
6. ✅ Test with various scenarios
7. ✅ Add logging for translation decisions
8. ⏳ Implement Per-Language Settings table and service
9. ⏳ Implement Translation History table and service
10. ⏳ Create UI for managing per-language settings
11. ⏳ Create UI for viewing translation history

---

**Last Updated**: 2025
