# 🌍 Multi-Language Implementation Guide

A comprehensive guide for implementing multi-language support in ASP.NET Core MVC applications with database-driven translations and API integration.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture Approach](#architecture-approach)
- [Database Schema Design](#database-schema-design)
- [Backend Implementation](#backend-implementation)
- [Frontend Implementation](#frontend-implementation)
- [API Implementation](#api-implementation)
- [Stored Procedures Update](#stored-procedures-update)
- [Implementation Steps](#implementation-steps)
- [Best Practices](#best-practices)
- [Example API Response](#example-api-response)

---

## Overview

This guide explains how to implement multi-language functionality where:
- **Admin Panel**: Data entry in multiple languages
- **Database**: Multi-language data storage
- **APIs**: Language-based data retrieval via `LanguageId` parameter
- **Website**: Consumes APIs and displays data based on `LanguageId`

### System Flow

```
Admin Panel → Database (Translations) → APIs (LanguageId) → Website (Display)
```

---

## Architecture Approach

### Option 1: Translation Table Pattern (Recommended) ⭐

Create separate translation tables for each content entity. This approach is:
- ✅ **Scalable**: Easy to add new languages
- ✅ **Maintainable**: Clear separation of concerns
- ✅ **Flexible**: Supports unlimited languages
- ✅ **Performance**: Optimized with proper indexing

### Option 2: Single Table with Language Columns

Store translations as columns in the main table (e.g., `Name_EN`, `Name_AR`, `Name_FR`).

**Limitations:**
- ❌ Schema changes required for new languages
- ❌ Not scalable
- ❌ Difficult to maintain

### Option 3: JSON Column Pattern

Store all translations in a single JSON column.

**Limitations:**
- ❌ Complex queries
- ❌ Limited indexing capabilities
- ❌ Harder to maintain

**Recommendation: Use Option 1 (Translation Table Pattern)**

---

## Database Schema Design

### 1. Language Master Table

```sql
CREATE TABLE Languages (
    LanguageId INT PRIMARY KEY IDENTITY(1,1),
    LanguageCode NVARCHAR(10) NOT NULL UNIQUE, -- en, ar, fr, es
    LanguageName NVARCHAR(100) NOT NULL, -- English, Arabic, French
    IsActive BIT DEFAULT 1,
    IsDefault BIT DEFAULT 0,
    DisplayOrder INT DEFAULT 0,
    CreatedDate DATETIME DEFAULT GETDATE()
);

-- Sample Data
INSERT INTO Languages (LanguageCode, LanguageName, IsDefault, DisplayOrder) VALUES
('en', 'English', 1, 1),
('ar', 'Arabic', 0, 2),
('fr', 'French', 0, 3),
('es', 'Spanish', 0, 4);
```

### 2. Property/Accommodation Translation Table

```sql
CREATE TABLE AccommodationTranslations (
    TranslationId INT PRIMARY KEY IDENTITY(1,1),
    AccommodationId INT NOT NULL,
    LanguageId INT NOT NULL,
    AccommodationName NVARCHAR(500),
    Description NVARCHAR(MAX),
    ShortDescription NVARCHAR(1000),
    Address NVARCHAR(500),
    CityName NVARCHAR(200),
    CreatedDate DATETIME DEFAULT GETDATE(),
    UpdatedDate DATETIME DEFAULT GETDATE(),
    
    FOREIGN KEY (AccommodationId) REFERENCES Accommodations(AccommodationId),
    FOREIGN KEY (LanguageId) REFERENCES Languages(LanguageId),
    UNIQUE (AccommodationId, LanguageId) -- One translation per language per property
);

-- Indexes for performance
CREATE INDEX IX_AccommodationTranslations_AccommodationId 
    ON AccommodationTranslations(AccommodationId);
CREATE INDEX IX_AccommodationTranslations_LanguageId 
    ON AccommodationTranslations(LanguageId);
CREATE INDEX IX_AccommodationTranslations_Composite 
    ON AccommodationTranslations(AccommodationId, LanguageId);
```

### 3. Room Type Translations

```sql
CREATE TABLE RoomTypeTranslations (
    TranslationId INT PRIMARY KEY IDENTITY(1,1),
    RoomTypeId INT NOT NULL,
    LanguageId INT NOT NULL,
    RoomTypeName NVARCHAR(200),
    Description NVARCHAR(MAX),
    BedType NVARCHAR(100),
    FOREIGN KEY (RoomTypeId) REFERENCES RoomTypes(RoomTypeId),
    FOREIGN KEY (LanguageId) REFERENCES Languages(LanguageId),
    UNIQUE (RoomTypeId, LanguageId)
);

CREATE INDEX IX_RoomTypeTranslations_RoomTypeId ON RoomTypeTranslations(RoomTypeId);
CREATE INDEX IX_RoomTypeTranslations_LanguageId ON RoomTypeTranslations(LanguageId);
```

### 4. Rate Plan Translations

```sql
CREATE TABLE RatePlanTranslations (
    TranslationId INT PRIMARY KEY IDENTITY(1,1),
    RatePlanId INT NOT NULL,
    LanguageId INT NOT NULL,
    RatePlanName NVARCHAR(200),
    Description NVARCHAR(MAX),
    CancellationPolicy NVARCHAR(MAX),
    FOREIGN KEY (RatePlanId) REFERENCES RatePlans(RatePlanId),
    FOREIGN KEY (LanguageId) REFERENCES Languages(LanguageId),
    UNIQUE (RatePlanId, LanguageId)
);

CREATE INDEX IX_RatePlanTranslations_RatePlanId ON RatePlanTranslations(RatePlanId);
CREATE INDEX IX_RatePlanTranslations_LanguageId ON RatePlanTranslations(LanguageId);
```

### 5. Facility Translations

```sql
CREATE TABLE FacilityTranslations (
    TranslationId INT PRIMARY KEY IDENTITY(1,1),
    FacilityId INT NOT NULL,
    LanguageId INT NOT NULL,
    FacilityName NVARCHAR(200),
    Description NVARCHAR(500),
    FOREIGN KEY (FacilityId) REFERENCES Facilities(FacilityId),
    FOREIGN KEY (LanguageId) REFERENCES Languages(LanguageId),
    UNIQUE (FacilityId, LanguageId)
);

CREATE INDEX IX_FacilityTranslations_FacilityId ON FacilityTranslations(FacilityId);
CREATE INDEX IX_FacilityTranslations_LanguageId ON FacilityTranslations(LanguageId);
```

### 6. Addon/Activity Translations

```sql
CREATE TABLE AddonTranslations (
    TranslationId INT PRIMARY KEY IDENTITY(1,1),
    AddonId INT NOT NULL,
    LanguageId INT NOT NULL,
    AddonName NVARCHAR(200),
    Description NVARCHAR(MAX),
    FOREIGN KEY (AddonId) REFERENCES Addons(AddonId),
    FOREIGN KEY (LanguageId) REFERENCES Languages(LanguageId),
    UNIQUE (AddonId, LanguageId)
);

CREATE INDEX IX_AddonTranslations_AddonId ON AddonTranslations(AddonId);
CREATE INDEX IX_AddonTranslations_LanguageId ON AddonTranslations(LanguageId);
```

---

## Backend Implementation

### 1. Models - Language Model

```csharp
// V3BookingEngine/CustomModel/Language.cs
namespace V3BookingEngine.CustomModel
{
    public class Language
    {
        public int LanguageId { get; set; }
        public string LanguageCode { get; set; } = string.Empty; // en, ar, fr
        public string LanguageName { get; set; } = string.Empty; // English, Arabic
        public bool IsActive { get; set; }
        public bool IsDefault { get; set; }
        public int DisplayOrder { get; set; }
    }
}
```

### 2. DTOs - Multi-Language ViewModels

```csharp
// V3BookingEngine/DTOs/Property/MultiLanguagePropertyViewModel.cs
namespace V3BookingEngine.DTOs.Property
{
    public class MultiLanguagePropertyViewModel
    {
        // Base Property Data (Language-independent)
        public int AccommodationId { get; set; }
        public int CountryId { get; set; }
        public int CurrencyId { get; set; }
        public int AccommodationTypeId { get; set; }
        // ... other non-translatable fields

        // Translations Dictionary: LanguageId -> Translation Data
        public Dictionary<int, PropertyTranslationViewModel> Translations { get; set; } 
            = new Dictionary<int, PropertyTranslationViewModel>();
    }

    public class PropertyTranslationViewModel
    {
        public int LanguageId { get; set; }
        public string LanguageCode { get; set; } = string.Empty;
        public string AccommodationName { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
        public string ShortDescription { get; set; } = string.Empty;
        public string Address { get; set; } = string.Empty;
        public string CityName { get; set; } = string.Empty;
    }
}
```

### 3. Service Layer - Language Service Interface

```csharp
// V3BookingEngine/Services/LanguageService/ILanguageService.cs
namespace V3BookingEngine.Services.LanguageService
{
    public interface ILanguageService
    {
        Task<List<CustomModel.Language>> GetAllLanguagesAsync();
        Task<CustomModel.Language> GetLanguageByIdAsync(int languageId);
        Task<CustomModel.Language> GetDefaultLanguageAsync();
        Task<bool> IsLanguageActiveAsync(int languageId);
    }
}
```

### 4. Service Layer - Language Service Implementation

```csharp
// V3BookingEngine/Services/LanguageService/LanguageService.cs
using Microsoft.Data.SqlClient;
using System.Data;
using V3BookingEngine.CustomModel;

namespace V3BookingEngine.Services.LanguageService
{
    public class LanguageService : ILanguageService
    {
        private readonly IConfiguration _config;
        private readonly SqlConnection _connection;

        public LanguageService(IConfiguration config)
        {
            _config = config;
            _connection = new SqlConnection(_config.GetConnectionString("V3DBConnection"));
        }

        public async Task<List<Language>> GetAllLanguagesAsync()
        {
            var languages = new List<Language>();
            try
            {
                await _connection.OpenAsync();
                using var cmd = new SqlCommand(
                    "SELECT * FROM Languages WHERE IsActive = 1 ORDER BY DisplayOrder", 
                    _connection);
                using var reader = await cmd.ExecuteReaderAsync();

                while (await reader.ReadAsync())
                {
                    languages.Add(new Language
                    {
                        LanguageId = reader.IsDBNull("LanguageId") ? 0 : Convert.ToInt32(reader["LanguageId"]),
                        LanguageCode = reader.IsDBNull("LanguageCode") ? string.Empty : (string)reader["LanguageCode"],
                        LanguageName = reader.IsDBNull("LanguageName") ? string.Empty : (string)reader["LanguageName"],
                        IsActive = !reader.IsDBNull("IsActive") && Convert.ToBoolean(reader["IsActive"]),
                        IsDefault = !reader.IsDBNull("IsDefault") && Convert.ToBoolean(reader["IsDefault"]),
                        DisplayOrder = reader.IsDBNull("DisplayOrder") ? 0 : Convert.ToInt32(reader["DisplayOrder"])
                    });
                }
            }
            finally
            {
                if (_connection.State == ConnectionState.Open)
                    _connection.Close();
            }
            return languages;
        }

        public async Task<Language> GetLanguageByIdAsync(int languageId)
        {
            try
            {
                await _connection.OpenAsync();
                using var cmd = new SqlCommand(
                    "SELECT * FROM Languages WHERE LanguageId = @LanguageId", 
                    _connection);
                cmd.Parameters.AddWithValue("@LanguageId", languageId);
                using var reader = await cmd.ExecuteReaderAsync();

                if (await reader.ReadAsync())
                {
                    return new Language
                    {
                        LanguageId = Convert.ToInt32(reader["LanguageId"]),
                        LanguageCode = (string)reader["LanguageCode"],
                        LanguageName = (string)reader["LanguageName"],
                        IsActive = Convert.ToBoolean(reader["IsActive"]),
                        IsDefault = Convert.ToBoolean(reader["IsDefault"]),
                        DisplayOrder = Convert.ToInt32(reader["DisplayOrder"])
                    };
                }
            }
            finally
            {
                if (_connection.State == ConnectionState.Open)
                    _connection.Close();
            }
            return null;
        }

        public async Task<Language> GetDefaultLanguageAsync()
        {
            try
            {
                await _connection.OpenAsync();
                using var cmd = new SqlCommand(
                    "SELECT TOP 1 * FROM Languages WHERE IsDefault = 1 AND IsActive = 1", 
                    _connection);
                using var reader = await cmd.ExecuteReaderAsync();

                if (await reader.ReadAsync())
                {
                    return new Language
                    {
                        LanguageId = Convert.ToInt32(reader["LanguageId"]),
                        LanguageCode = (string)reader["LanguageCode"],
                        LanguageName = (string)reader["LanguageName"],
                        IsActive = Convert.ToBoolean(reader["IsActive"]),
                        IsDefault = Convert.ToBoolean(reader["IsDefault"]),
                        DisplayOrder = Convert.ToInt32(reader["DisplayOrder"])
                    };
                }
            }
            finally
            {
                if (_connection.State == ConnectionState.Open)
                    _connection.Close();
            }
            return null;
        }

        public async Task<bool> IsLanguageActiveAsync(int languageId)
        {
            try
            {
                await _connection.OpenAsync();
                using var cmd = new SqlCommand(
                    "SELECT COUNT(*) FROM Languages WHERE LanguageId = @LanguageId AND IsActive = 1", 
                    _connection);
                cmd.Parameters.AddWithValue("@LanguageId", languageId);
                var count = Convert.ToInt32(await cmd.ExecuteScalarAsync());
                return count > 0;
            }
            finally
            {
                if (_connection.State == ConnectionState.Open)
                    _connection.Close();
            }
        }
    }
}
```

### 5. Update PropertyManagementService - Multi-Language Support

```csharp
// PropertyManagementService.cs - Add these methods:

// Get Property with all translations
public async Task<MultiLanguagePropertyViewModel> GetPropertyWithTranslationsAsync(int accommodationId)
{
    var property = new MultiLanguagePropertyViewModel();
    property.Translations = new Dictionary<int, PropertyTranslationViewModel>();

    try
    {
        await _connection.OpenAsync();

        // Get base property data (non-translatable) - use existing stored procedure
        using var baseCmd = new SqlCommand("SP_Accommodations_Core", _connection)
        {
            CommandType = CommandType.StoredProcedure
        };
        baseCmd.Parameters.AddWithValue("@ProcedureType", "G");
        baseCmd.Parameters.AddWithValue("@p_AccommodationId", accommodationId);
        // ... other parameters as needed

        // Get all translations
        using var transCmd = new SqlCommand(@"
            SELECT 
                at.*,
                l.LanguageCode,
                l.LanguageName
            FROM AccommodationTranslations at
            INNER JOIN Languages l ON at.LanguageId = l.LanguageId
            WHERE at.AccommodationId = @AccommodationId
        ", _connection);
        transCmd.Parameters.AddWithValue("@AccommodationId", accommodationId);

        using var reader = await transCmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            var langId = Convert.ToInt32(reader["LanguageId"]);
            property.Translations[langId] = new PropertyTranslationViewModel
            {
                LanguageId = langId,
                LanguageCode = reader.IsDBNull("LanguageCode") ? string.Empty : (string)reader["LanguageCode"],
                AccommodationName = reader.IsDBNull("AccommodationName") ? string.Empty : (string)reader["AccommodationName"],
                Description = reader.IsDBNull("Description") ? string.Empty : (string)reader["Description"],
                ShortDescription = reader.IsDBNull("ShortDescription") ? string.Empty : (string)reader["ShortDescription"],
                Address = reader.IsDBNull("Address") ? string.Empty : (string)reader["Address"],
                CityName = reader.IsDBNull("CityName") ? string.Empty : (string)reader["CityName"]
            };
        }
    }
    finally
    {
        if (_connection.State == ConnectionState.Open)
            _connection.Close();
    }

    return property;
}

// Save Property with translations
public async Task<string> SavePropertyWithTranslationsAsync(
    MultiLanguagePropertyViewModel model, 
    int userId, 
    int defaultLanguageId)
{
    try
    {
        await _connection.OpenAsync();
        using var transaction = _connection.BeginTransaction();

        try
        {
            // 1. Save base property (existing stored procedure)
            using var baseCmd = new SqlCommand("SP_Accommodations_Core", _connection, transaction)
            {
                CommandType = CommandType.StoredProcedure
            };
            baseCmd.Parameters.AddWithValue("@ProcedureType", model.AccommodationId > 0 ? "U" : "I");
            baseCmd.Parameters.AddWithValue("@p_AccommodationId", model.AccommodationId);
            // ... other base parameters
            await baseCmd.ExecuteNonQueryAsync();

            var newAccommodationId = model.AccommodationId;
            if (model.AccommodationId == 0)
            {
                // Get newly created ID
                using var getIdCmd = new SqlCommand("SELECT SCOPE_IDENTITY()", _connection, transaction);
                newAccommodationId = Convert.ToInt32(await getIdCmd.ExecuteScalarAsync());
            }

            // 2. Save/Update translations
            foreach (var translation in model.Translations.Values)
            {
                using var transCmd = new SqlCommand(@"
                    IF EXISTS (SELECT 1 FROM AccommodationTranslations 
                               WHERE AccommodationId = @AccommodationId AND LanguageId = @LanguageId)
                    BEGIN
                        UPDATE AccommodationTranslations
                        SET AccommodationName = @AccommodationName,
                            Description = @Description,
                            ShortDescription = @ShortDescription,
                            Address = @Address,
                            CityName = @CityName,
                            UpdatedDate = GETDATE()
                        WHERE AccommodationId = @AccommodationId AND LanguageId = @LanguageId
                    END
                    ELSE
                    BEGIN
                        INSERT INTO AccommodationTranslations 
                        (AccommodationId, LanguageId, AccommodationName, Description, ShortDescription, Address, CityName)
                        VALUES 
                        (@AccommodationId, @LanguageId, @AccommodationName, @Description, @ShortDescription, @Address, @CityName)
                    END
                ", _connection, transaction);

                transCmd.Parameters.AddWithValue("@AccommodationId", newAccommodationId);
                transCmd.Parameters.AddWithValue("@LanguageId", translation.LanguageId);
                transCmd.Parameters.AddWithValue("@AccommodationName", translation.AccommodationName ?? (object)DBNull.Value);
                transCmd.Parameters.AddWithValue("@Description", translation.Description ?? (object)DBNull.Value);
                transCmd.Parameters.AddWithValue("@ShortDescription", translation.ShortDescription ?? (object)DBNull.Value);
                transCmd.Parameters.AddWithValue("@Address", translation.Address ?? (object)DBNull.Value);
                transCmd.Parameters.AddWithValue("@CityName", translation.CityName ?? (object)DBNull.Value);

                await transCmd.ExecuteNonQueryAsync();
            }

            transaction.Commit();
            return "Success";
        }
        catch
        {
            transaction.Rollback();
            throw;
        }
    }
    finally
    {
        if (_connection.State == ConnectionState.Open)
            _connection.Close();
    }
}
```

### 6. Dependency Injection Setup

```csharp
// Program.cs or Startup.cs
services.AddScoped<ILanguageService, LanguageService>();
```

### 7. Controller Updates - Multi-Language Support

```csharp
// PropertyController.cs

[HttpGet]
public async Task<IActionResult> CreateProperty()
{
    try
    {
        var userLanguageId = int.TryParse(
            User.Claims.FirstOrDefault(c => c.Type == "LanguageId")?.Value, 
            out var langId) ? langId : 1;
        
        // Get all active languages
        var languages = await _languageService.GetAllLanguagesAsync();
        
        // Get base data
        var dataSets = await _propertyManagementService.GetCreatePropertyDataAsync(userLanguageId);
        
        var model = new CreatePropertyViewModel();
        model.PopulateDropdownData(dataSets);
        
        // Add languages to ViewBag for UI
        ViewBag.Languages = languages;
        ViewBag.DefaultLanguageId = languages.FirstOrDefault(l => l.IsDefault)?.LanguageId ?? 1;
        
        return View(model);
    }
    catch (Exception ex)
    {
        await _errorLoggingHelper.LogErrorAsync(_logger, ex, "CreateProperty", GetCurrentUserId());
        return RedirectToAction("Error", "Home");
    }
}

[HttpPost]
public async Task<IActionResult> CreateProperty(MultiLanguagePropertyViewModel model)
{
    try
    {
        var userId = int.TryParse(User.FindFirst("UserId")?.Value, out var uId) ? uId : 1;
        var defaultLanguageId = int.TryParse(
            User.Claims.FirstOrDefault(c => c.Type == "LanguageId")?.Value, 
            out var langId) ? langId : 1;
        
        // Validate: At least default language translation is required
        if (!model.Translations.ContainsKey(defaultLanguageId))
        {
            return Json(new { success = false, message = "Default language translation is required" });
        }
        
        var result = await _propertyManagementService.SavePropertyWithTranslationsAsync(
            model, userId, defaultLanguageId);
        
        if (result == "Success")
        {
            return Json(new { success = true, message = "Property created successfully" });
        }
        
        return Json(new { success = false, message = result });
    }
    catch (Exception ex)
    {
        await _errorLoggingHelper.LogErrorAsync(_logger, ex, "CreateProperty", GetCurrentUserId());
        return Json(new { success = false, message = "Error creating property" });
    }
}
```

---

## Frontend Implementation (Admin Panel)

### 1. Multi-Language Tabs UI

```html
<!-- CreateProperty.cshtml -->

<div class="language-tabs-container mb-4">
    <ul class="nav nav-tabs" id="languageTabs" role="tablist">
        @foreach (var lang in ViewBag.Languages as List<Language>)
        {
            <li class="nav-item" role="presentation">
                <button class="nav-link @(lang.IsDefault ? "active" : "")" 
                        id="@lang.LanguageCode-tab" 
                        data-bs-toggle="tab" 
                        data-bs-target="#@lang.LanguageCode" 
                        type="button" 
                        role="tab"
                        data-language-id="@lang.LanguageId"
                        data-language-code="@lang.LanguageCode">
                    @lang.LanguageName
                    @if (lang.IsDefault)
                    {
                        <span class="badge bg-primary ms-1">Default</span>
                    }
                </button>
            </li>
        }
    </ul>
</div>

<div class="tab-content" id="languageTabContent">
    @foreach (var lang in ViewBag.Languages as List<Language>)
    {
        <div class="tab-pane fade @(lang.IsDefault ? "show active" : "")" 
             id="@lang.LanguageCode" 
             role="tabpanel"
             data-language-id="@lang.LanguageId">
            
            <!-- Property Name -->
            <div class="form-group mb-3">
                <label for="AccommodationName_@lang.LanguageId">
                    Property Name (@lang.LanguageName) 
                    @if (lang.IsDefault)
                    {
                        <span class="text-danger">*</span>
                    }
                </label>
                <input type="text" 
                       class="form-control" 
                       id="AccommodationName_@lang.LanguageId"
                       name="Translations[@lang.LanguageId].AccommodationName"
                       data-language-id="@lang.LanguageId"
                       placeholder="Enter property name in @lang.LanguageName"
                       @(lang.IsDefault ? "required" : "")>
            </div>

            <!-- Description -->
            <div class="form-group mb-3">
                <label for="Description_@lang.LanguageId">
                    Description (@lang.LanguageName)
                </label>
                <textarea class="form-control" 
                          id="Description_@lang.LanguageId"
                          name="Translations[@lang.LanguageId].Description"
                          rows="5"
                          data-language-id="@lang.LanguageId"
                          placeholder="Enter description in @lang.LanguageName"></textarea>
            </div>

            <!-- Short Description -->
            <div class="form-group mb-3">
                <label for="ShortDescription_@lang.LanguageId">
                    Short Description (@lang.LanguageName)
                </label>
                <textarea class="form-control" 
                          id="ShortDescription_@lang.LanguageId"
                          name="Translations[@lang.LanguageId].ShortDescription"
                          rows="3"
                          data-language-id="@lang.LanguageId"></textarea>
            </div>

            <!-- Address -->
            <div class="form-group mb-3">
                <label for="Address_@lang.LanguageId">
                    Address (@lang.LanguageName)
                </label>
                <input type="text" 
                       class="form-control" 
                       id="Address_@lang.LanguageId"
                       name="Translations[@lang.LanguageId].Address"
                       data-language-id="@lang.LanguageId">
            </div>

            <!-- City Name -->
            <div class="form-group mb-3">
                <label for="CityName_@lang.LanguageId">
                    City Name (@lang.LanguageName)
                </label>
                <input type="text" 
                       class="form-control" 
                       id="CityName_@lang.LanguageId"
                       name="Translations[@lang.LanguageId].CityName"
                       data-language-id="@lang.LanguageId">
            </div>

            @if (!lang.IsDefault)
            {
                <div class="mb-3">
                    <button type="button" 
                            class="btn btn-sm btn-outline-secondary copy-from-default"
                            data-target-language-id="@lang.LanguageId">
                        <i class="fas fa-copy"></i> Copy from Default Language
                    </button>
                </div>
            }

        </div>
    }
</div>
```

### 2. JavaScript - Form Submission with Multi-Language Data

```javascript
// CreateProperty.cshtml or separate JS file

$(document).ready(function() {
    // Form submission handler
    $('#createPropertyForm').on('submit', function(e) {
        e.preventDefault();
        
        // Collect all language data
        var translations = {};
        
        $('.tab-pane').each(function() {
            var languageId = $(this).data('language-id');
            if (languageId) {
                translations[languageId] = {
                    LanguageId: languageId,
                    AccommodationName: $('#AccommodationName_' + languageId).val() || '',
                    Description: $('#Description_' + languageId).val() || '',
                    ShortDescription: $('#ShortDescription_' + languageId).val() || '',
                    Address: $('#Address_' + languageId).val() || '',
                    CityName: $('#CityName_' + languageId).val() || ''
                };
            }
        });
        
        // Collect base property data (non-translatable)
        var propertyData = {
            AccommodationId: $('#AccommodationId').val() || 0,
            CountryId: $('#CountryId').val(),
            CurrencyId: $('#CurrencyId').val(),
            AccommodationTypeId: $('#AccommodationTypeId').val(),
            // ... other base fields
            Translations: translations
        };
        
        // Submit via AJAX
        $.ajax({
            url: '/Property/CreateProperty',
            type: 'POST',
            contentType: 'application/json',
            data: JSON.stringify(propertyData),
            success: function(response) {
                if (response.success) {
                    Swal.fire({
                        icon: 'success',
                        title: 'Success',
                        text: response.message || 'Property created successfully'
                    }).then(() => {
                        window.location.href = '/Property/PropertyList';
                    });
                } else {
                    Swal.fire({
                        icon: 'error',
                        title: 'Error',
                        text: response.message || 'Failed to create property'
                    });
                }
            },
            error: function(xhr) {
                Swal.fire({
                    icon: 'error',
                    title: 'Error',
                    text: 'An error occurred. Please try again.'
                });
            }
        });
    });
    
    // Copy from default language functionality
    $('.copy-from-default').on('click', function() {
        var defaultLanguageId = @ViewBag.DefaultLanguageId;
        var targetLanguageId = $(this).data('target-language-id');
        
        // Copy values from default language to target language
        $('#AccommodationName_' + targetLanguageId).val($('#AccommodationName_' + defaultLanguageId).val());
        $('#Description_' + targetLanguageId).val($('#Description_' + defaultLanguageId).val());
        $('#ShortDescription_' + targetLanguageId).val($('#ShortDescription_' + defaultLanguageId).val());
        $('#Address_' + targetLanguageId).val($('#Address_' + defaultLanguageId).val());
        $('#CityName_' + targetLanguageId).val($('#CityName_' + defaultLanguageId).val());
        
        Swal.fire({
            icon: 'info',
            title: 'Copied',
            text: 'Content copied from default language',
            timer: 2000,
            showConfirmButton: false
        });
    });
});
```

---

## API Implementation (Website Integration)

### 1. API Controller - Language-based Data

```csharp
// V3BookingEngine/Controllers/Api/PropertyApiController.cs
using Microsoft.AspNetCore.Mvc;

namespace V3BookingEngine.Controllers.Api
{
    [ApiController]
    [Route("api/[controller]")]
    public class PropertyApiController : ControllerBase
    {
        private readonly IPropertyManagementService _propertyService;
        private readonly ILanguageService _languageService;

        public PropertyApiController(
            IPropertyManagementService propertyService,
            ILanguageService languageService)
        {
            _propertyService = propertyService;
            _languageService = languageService;
        }

        /// <summary>
        /// Get property details by ID and language
        /// </summary>
        /// <param name="id">Property/Accommodation ID</param>
        /// <param name="languageId">Language ID (default: 1)</param>
        /// <returns>Property details in requested language</returns>
        [HttpGet("GetProperty/{id}")]
        public async Task<IActionResult> GetProperty(int id, [FromQuery] int languageId = 1)
        {
            try
            {
                // Validate language
                if (!await _languageService.IsLanguageActiveAsync(languageId))
                {
                    // Fallback to default language
                    var defaultLang = await _languageService.GetDefaultLanguageAsync();
                    languageId = defaultLang?.LanguageId ?? 1;
                }

                // Get property with translation for requested language
                var property = await _propertyService.GetPropertyByLanguageAsync(id, languageId);
                
                if (property == null)
                {
                    return NotFound(new { 
                        success = false, 
                        message = "Property not found" 
                    });
                }

                return Ok(new
                {
                    success = true,
                    data = property,
                    languageId = languageId
                });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { 
                    success = false, 
                    message = ex.Message 
                });
            }
        }

        /// <summary>
        /// Get paginated list of properties by language
        /// </summary>
        /// <param name="languageId">Language ID (default: 1)</param>
        /// <param name="page">Page number (default: 1)</param>
        /// <param name="pageSize">Items per page (default: 10)</param>
        /// <returns>Paginated list of properties</returns>
        [HttpGet("GetPropertyList")]
        public async Task<IActionResult> GetPropertyList(
            [FromQuery] int languageId = 1,
            [FromQuery] int page = 1,
            [FromQuery] int pageSize = 10)
        {
            try
            {
                if (!await _languageService.IsLanguageActiveAsync(languageId))
                {
                    var defaultLang = await _languageService.GetDefaultLanguageAsync();
                    languageId = defaultLang?.LanguageId ?? 1;
                }

                var properties = await _propertyService.GetPropertyListByLanguageAsync(
                    languageId, page, pageSize);
                
                return Ok(new
                {
                    success = true,
                    data = properties.Items,
                    totalCount = properties.TotalCount,
                    page = page,
                    pageSize = pageSize,
                    totalPages = (int)Math.Ceiling(properties.TotalCount / (double)pageSize),
                    languageId = languageId
                });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { 
                    success = false, 
                    message = ex.Message 
                });
            }
        }

        /// <summary>
        /// Get all available languages
        /// </summary>
        /// <returns>List of active languages</returns>
        [HttpGet("GetAvailableLanguages")]
        public async Task<IActionResult> GetAvailableLanguages()
        {
            try
            {
                var languages = await _languageService.GetAllLanguagesAsync();
                return Ok(new
                {
                    success = true,
                    data = languages.Select(l => new
                    {
                        languageId = l.LanguageId,
                        languageCode = l.LanguageCode,
                        languageName = l.LanguageName,
                        isDefault = l.IsDefault
                    })
                });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { 
                    success = false, 
                    message = ex.Message 
                });
            }
        }
    }
}
```

### 2. Service Method - Get Property by Language

```csharp
// PropertyManagementService.cs - Add this method:

public async Task<PropertyApiDto> GetPropertyByLanguageAsync(int accommodationId, int languageId)
{
    try
    {
        await _connection.OpenAsync();
        
        using var cmd = new SqlCommand(@"
            SELECT 
                a.AccommodationId,
                a.CountryId,
                a.CurrencyId,
                a.AccommodationTypeId,
                -- Base fields (non-translatable)
                ISNULL(at.AccommodationName, a.AccommodationName) AS AccommodationName,
                ISNULL(at.Description, '') AS Description,
                ISNULL(at.ShortDescription, '') AS ShortDescription,
                ISNULL(at.Address, '') AS Address,
                ISNULL(at.CityName, '') AS CityName,
                ISNULL(l.LanguageCode, 'en') AS LanguageCode,
                ISNULL(l.LanguageName, 'English') AS LanguageName
            FROM Accommodations a
            LEFT JOIN AccommodationTranslations at 
                ON a.AccommodationId = at.AccommodationId 
                AND at.LanguageId = @LanguageId
            LEFT JOIN Languages l ON at.LanguageId = l.LanguageId
            WHERE a.AccommodationId = @AccommodationId
        ", _connection);
        
        cmd.Parameters.AddWithValue("@AccommodationId", accommodationId);
        cmd.Parameters.AddWithValue("@LanguageId", languageId);
        
        using var reader = await cmd.ExecuteReaderAsync();
        
        if (await reader.ReadAsync())
        {
            return new PropertyApiDto
            {
                AccommodationId = Convert.ToInt32(reader["AccommodationId"]),
                AccommodationName = reader.IsDBNull("AccommodationName") ? string.Empty : (string)reader["AccommodationName"],
                Description = reader.IsDBNull("Description") ? string.Empty : (string)reader["Description"],
                ShortDescription = reader.IsDBNull("ShortDescription") ? string.Empty : (string)reader["ShortDescription"],
                Address = reader.IsDBNull("Address") ? string.Empty : (string)reader["Address"],
                CityName = reader.IsDBNull("CityName") ? string.Empty : (string)reader["CityName"],
                LanguageCode = reader.IsDBNull("LanguageCode") ? "en" : (string)reader["LanguageCode"],
                LanguageName = reader.IsDBNull("LanguageName") ? "English" : (string)reader["LanguageName"],
                // ... other fields
            };
        }
        
        return null;
    }
    finally
    {
        if (_connection.State == ConnectionState.Open)
            _connection.Close();
    }
}
```

---

## Stored Procedures Update

### Update Existing Stored Procedures with LanguageId Parameter

```sql
-- Example: SP_Accommodations_Core update
ALTER PROCEDURE [dbo].[SP_Accommodations_Core]
    @ProcedureType VARCHAR(10),
    @p_AccommodationId INT = NULL,
    @p_MultiLanguageId INT = 1, -- LanguageId parameter
    @p_UserTypeId INT = NULL,
    @p_UserId INT = NULL
    -- ... other parameters
AS
BEGIN
    IF @ProcedureType = 'G' -- Get
    BEGIN
        SELECT 
            a.*,
            ISNULL(at.AccommodationName, a.AccommodationName) AS AccommodationName,
            ISNULL(at.Description, '') AS Description,
            ISNULL(at.ShortDescription, '') AS ShortDescription,
            ISNULL(at.Address, '') AS Address,
            ISNULL(at.CityName, '') AS CityName,
            @p_MultiLanguageId AS LanguageId
        FROM Accommodations a
        LEFT JOIN AccommodationTranslations at 
            ON a.AccommodationId = at.AccommodationId 
            AND at.LanguageId = @p_MultiLanguageId
        WHERE a.AccommodationId = @p_AccommodationId
    END
    -- ... other procedure types (Insert, Update, Delete)
END
```

---

## Implementation Steps

### Phase 1: Database Setup ✅

1. Create `Languages` table
2. Insert sample languages (en, ar, fr, es, etc.)
3. Create translation tables:
   - `AccommodationTranslations`
   - `RoomTypeTranslations`
   - `RatePlanTranslations`
   - `FacilityTranslations`
   - `AddonTranslations`
4. Add indexes for performance optimization

### Phase 2: Backend Services ✅

1. Create `Language` model in `CustomModel`
2. Create `ILanguageService` interface
3. Implement `LanguageService` class
4. Create `MultiLanguagePropertyViewModel` and related DTOs
5. Add multi-language methods to `PropertyManagementService`
6. Register `ILanguageService` in dependency injection

### Phase 3: Controllers ✅

1. Update `PropertyController` with multi-language support
2. Create `PropertyApiController` for website APIs
3. Add language validation and fallback logic

### Phase 4: Frontend (Admin Panel) ✅

1. Add language tabs UI to create/update forms
2. Organize form fields by language
3. Update JavaScript for form submission with multi-language data
4. Implement "Copy from Default Language" feature
5. Add validation for required default language fields

### Phase 5: APIs (Website) ✅

1. Create API controllers with language support
2. Implement `LanguageId` parameter handling
3. Add fallback to default language logic
4. Create API documentation

### Phase 6: Testing ✅

1. Test multi-language data entry in admin panel
2. Test API responses with different `LanguageId` values
3. Test fallback to default language when translation missing
4. Performance testing with indexes
5. Test edge cases (missing translations, invalid language IDs)

---

## Best Practices

### 1. Default Language Fallback

Always provide a fallback mechanism when requested language translation is not available:

```csharp
var translation = await GetTranslationAsync(entityId, requestedLanguageId);
if (translation == null || string.IsNullOrEmpty(translation.AccommodationName))
{
    var defaultLang = await GetDefaultLanguageAsync();
    translation = await GetTranslationAsync(entityId, defaultLang.LanguageId);
}
```

### 2. Caching

Cache language list as it rarely changes:

```csharp
// In LanguageService
private readonly IMemoryCache _cache;

public async Task<List<Language>> GetAllLanguagesAsync()
{
    return await _cache.GetOrCreateAsync("AllLanguages", async entry =>
    {
        entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(24);
        // ... fetch from database
    });
}
```

### 3. Validation

Ensure at least default language translation is provided:

```csharp
if (!model.Translations.ContainsKey(defaultLanguageId) || 
    string.IsNullOrWhiteSpace(model.Translations[defaultLanguageId].AccommodationName))
{
    ModelState.AddModelError("", "Default language translation is required");
    return BadRequest(ModelState);
}
```

### 4. Performance Optimization

- ✅ Use proper indexes on translation tables
- ✅ Use `LEFT JOIN` to show base data if translation is missing
- ✅ Implement pagination for large lists
- ✅ Cache frequently accessed language data
- ✅ Use parameterized queries to prevent SQL injection

### 5. Error Handling

```csharp
try
{
    // Language operations
}
catch (SqlException ex)
{
    _logger.LogError(ex, "Database error in language service");
    // Handle database errors
}
catch (Exception ex)
{
    _logger.LogError(ex, "Unexpected error in language service");
    // Handle general errors
}
```

---

## Example API Response

### Get Property by Language

**Request:**
```
GET /api/PropertyApi/GetProperty/123?languageId=2
```

**Response:**
```json
{
    "success": true,
    "data": {
        "accommodationId": 123,
        "accommodationName": "فندق الشاطئ",
        "description": "فندق فاخر على الشاطئ مع إطلالة رائعة على البحر",
        "shortDescription": "إقامة مريحة وفاخرة",
        "address": "شارع الكورنيش، دبي",
        "cityName": "دبي",
        "languageCode": "ar",
        "languageName": "Arabic",
        "countryId": 1,
        "currencyId": 1
    },
    "languageId": 2
}
```

### Get Property List

**Request:**
```
GET /api/PropertyApi/GetPropertyList?languageId=1&page=1&pageSize=10
```

**Response:**
```json
{
    "success": true,
    "data": [
        {
            "accommodationId": 123,
            "accommodationName": "Beach Hotel",
            "shortDescription": "Luxury beachfront accommodation",
            "cityName": "Dubai"
        }
    ],
    "totalCount": 50,
    "page": 1,
    "pageSize": 10,
    "totalPages": 5,
    "languageId": 1
}
```

### Get Available Languages

**Request:**
```
GET /api/PropertyApi/GetAvailableLanguages
```

**Response:**
```json
{
    "success": true,
    "data": [
        {
            "languageId": 1,
            "languageCode": "en",
            "languageName": "English",
            "isDefault": true
        },
        {
            "languageId": 2,
            "languageCode": "ar",
            "languageName": "Arabic",
            "isDefault": false
        },
        {
            "languageId": 3,
            "languageCode": "fr",
            "languageName": "French",
            "isDefault": false
        }
    ]
}
```

---

## Translation APIs Integration

For automatic translation of content, you can integrate third-party translation APIs. Here's a comparison of popular options:

### Translation API Comparison

| API Service | Quality | Price (per 1M characters) | Free Tier | Best For |
|------------|---------|---------------------------|-----------|----------|
| **DeepL API** | ⭐⭐⭐⭐⭐ Excellent | $25 | 500K chars/month | High-quality translations |
| **Google Cloud Translate** | ⭐⭐⭐⭐ Very Good | $20 | 500K chars/month | General purpose, many languages |
| **Azure Translator** | ⭐⭐⭐⭐ Very Good | $10 | 2M chars/month | Microsoft ecosystem |
| **AWS Translate** | ⭐⭐⭐⭐ Very Good | $15 | 2M chars/month | AWS infrastructure |
| **LibreTranslate** | ⭐⭐⭐ Good | Free (self-hosted) | Unlimited | Open source, privacy-focused |
| **MyMemory** | ⭐⭐ Basic | Free (limited) | 10K chars/day | Small projects, testing |

### Recommended: DeepL API (Best Quality) ⭐

**Why DeepL?**
- ✅ Highest translation quality (especially for European languages)
- ✅ Natural-sounding translations
- ✅ Good for business/professional content
- ✅ Supports 31+ languages
- ✅ Free tier: 500,000 characters/month

**Pricing:**
- Free: 500K characters/month
- Pro: $25 per 1M characters (after free tier)
- Enterprise: Custom pricing

**API Integration Example:**

```csharp
// V3BookingEngine/Services/TranslationService/DeepLTranslationService.cs
using System.Net.Http;
using System.Text;
using System.Text.Json;

namespace V3BookingEngine.Services.TranslationService
{
    public class DeepLTranslationService : ITranslationService
    {
        private readonly IConfiguration _config;
        private readonly HttpClient _httpClient;
        private readonly string _apiKey;
        private readonly string _apiUrl = "https://api-free.deepl.com/v2/translate"; // Free tier
        // For Pro: "https://api.deepl.com/v2/translate"

        public DeepLTranslationService(IConfiguration config, HttpClient httpClient)
        {
            _config = config;
            _httpClient = httpClient;
            _apiKey = _config["TranslationServices:DeepL:ApiKey"];
        }

        public async Task<string> TranslateAsync(string text, string sourceLanguage, string targetLanguage)
        {
            try
            {
                var requestBody = new
                {
                    text = new[] { text },
                    source_lang = sourceLanguage.ToUpper(),
                    target_lang = targetLanguage.ToUpper()
                };

                var json = JsonSerializer.Serialize(requestBody);
                var content = new StringContent(json, Encoding.UTF8, "application/json");

                var request = new HttpRequestMessage(HttpMethod.Post, _apiUrl)
                {
                    Content = content
                };
                request.Headers.Add("Authorization", $"DeepL-Auth-Key {_apiKey}");

                var response = await _httpClient.SendAsync(request);
                response.EnsureSuccessStatusCode();

                var responseContent = await response.Content.ReadAsStringAsync();
                var result = JsonSerializer.Deserialize<DeepLResponse>(responseContent);

                return result?.translations?.FirstOrDefault()?.text ?? text;
            }
            catch (Exception ex)
            {
                // Log error and return original text
                // _logger.LogError(ex, "DeepL translation failed");
                return text;
            }
        }

        public async Task<Dictionary<string, string>> TranslateBatchAsync(
            Dictionary<string, string> texts, 
            string sourceLanguage, 
            string targetLanguage)
        {
            var results = new Dictionary<string, string>();
            
            foreach (var kvp in texts)
            {
                results[kvp.Key] = await TranslateAsync(kvp.Value, sourceLanguage, targetLanguage);
                // Add delay to respect rate limits (free tier: 5 requests/second)
                await Task.Delay(200);
            }
            
            return results;
        }

        private class DeepLResponse
        {
            public List<Translation> translations { get; set; }
        }

        private class Translation
        {
            public string text { get; set; }
            public string detected_source_language { get; set; }
        }
    }

    public interface ITranslationService
    {
        Task<string> TranslateAsync(string text, string sourceLanguage, string targetLanguage);
        Task<Dictionary<string, string>> TranslateBatchAsync(
            Dictionary<string, string> texts, 
            string sourceLanguage, 
            string targetLanguage);
    }
}
```

**Configuration (appsettings.json):**

```json
{
  "TranslationServices": {
    "DeepL": {
      "ApiKey": "your-deepl-api-key-here",
      "UsePro": false
    }
  }
}
```

**Usage in Controller:**

```csharp
// PropertyController.cs
[HttpPost]
public async Task<IActionResult> AutoTranslateProperty(
    int accommodationId, 
    int sourceLanguageId, 
    int targetLanguageId)
{
    try
    {
        // Get source translation
        var sourceTranslation = await _propertyService
            .GetPropertyTranslationAsync(accommodationId, sourceLanguageId);
        
        // Get language codes
        var sourceLang = await _languageService.GetLanguageByIdAsync(sourceLanguageId);
        var targetLang = await _languageService.GetLanguageByIdAsync(targetLanguageId);
        
        // Translate using DeepL
        var translatedName = await _translationService.TranslateAsync(
            sourceTranslation.AccommodationName,
            sourceLang.LanguageCode,
            targetLang.LanguageCode
        );
        
        var translatedDescription = await _translationService.TranslateAsync(
            sourceTranslation.Description,
            sourceLang.LanguageCode,
            targetLang.LanguageCode
        );
        
        // Save translated content
        var translation = new PropertyTranslationViewModel
        {
            LanguageId = targetLanguageId,
            AccommodationName = translatedName,
            Description = translatedDescription,
            // ... other fields
        };
        
        await _propertyService.SaveTranslationAsync(accommodationId, translation);
        
        return Json(new { success = true, message = "Translation completed" });
    }
    catch (Exception ex)
    {
        return Json(new { success = false, message = ex.Message });
    }
}
```

### Alternative: Google Cloud Translate (Cost-Effective)

**Why Google Translate?**
- ✅ Supports 100+ languages
- ✅ Good pricing ($20 per 1M characters)
- ✅ Free tier: 500K characters/month
- ✅ Reliable and fast

**Pricing:**
- Free: 500K characters/month
- Paid: $20 per 1M characters

**Integration Example:**

```csharp
// GoogleCloudTranslationService.cs
using Google.Cloud.Translation.V2;

public class GoogleCloudTranslationService : ITranslationService
{
    private readonly TranslationClient _client;

    public GoogleCloudTranslationService(IConfiguration config)
    {
        var apiKey = config["TranslationServices:Google:ApiKey"];
        _client = TranslationClient.CreateFromApiKey(apiKey);
    }

    public async Task<string> TranslateAsync(string text, string sourceLanguage, string targetLanguage)
    {
        var response = await _client.TranslateTextAsync(
            text, 
            targetLanguage, 
            sourceLanguage: sourceLanguage
        );
        return response.TranslatedText;
    }
}
```

### Alternative: Azure Translator (Best Value)

**Why Azure Translator?**
- ✅ Excellent pricing ($10 per 1M characters)
- ✅ Free tier: 2M characters/month
- ✅ Good quality
- ✅ Integrates well with Microsoft services

**Pricing:**
- Free: 2M characters/month
- Paid: $10 per 1M characters

**Integration Example:**

```csharp
// AzureTranslationService.cs
using Azure;
using Azure.AI.Translation.Text;

public class AzureTranslationService : ITranslationService
{
    private readonly TextTranslationClient _client;

    public AzureTranslationService(IConfiguration config)
    {
        var endpoint = config["TranslationServices:Azure:Endpoint"];
        var apiKey = config["TranslationServices:Azure:ApiKey"];
        var credential = new AzureKeyCredential(apiKey);
        _client = new TextTranslationClient(credential, new Uri(endpoint));
    }

    public async Task<string> TranslateAsync(string text, string sourceLanguage, string targetLanguage)
    {
        var response = await _client.TranslateAsync(
            targetLanguage, 
            new InputTextItem[] { new InputTextItem(text) },
            sourceLanguage: sourceLanguage
        );
        return response.Value[0].Translations[0].Text;
    }
}
```

### Budget Option: LibreTranslate (Free & Open Source)

**Why LibreTranslate?**
- ✅ Completely free (self-hosted)
- ✅ Privacy-focused (no data sent to third parties)
- ✅ Open source
- ⚠️ Lower quality than paid services
- ⚠️ Requires self-hosting

**Self-Hosted Setup:**

```bash
# Docker installation
docker run -ti --rm -p 5000:5000 libretranslate/libretranslate
```

**Integration:**

```csharp
// LibreTranslateService.cs
public class LibreTranslateService : ITranslationService
{
    private readonly HttpClient _httpClient;
    private readonly string _apiUrl;

    public LibreTranslateService(IConfiguration config, HttpClient httpClient)
    {
        _httpClient = httpClient;
        _apiUrl = config["TranslationServices:LibreTranslate:ApiUrl"] 
            ?? "http://localhost:5000/translate";
    }

    public async Task<string> TranslateAsync(string text, string sourceLanguage, string targetLanguage)
    {
        var requestBody = new
        {
            q = text,
            source = sourceLanguage,
            target = targetLanguage,
            format = "text"
        };

        var json = JsonSerializer.Serialize(requestBody);
        var content = new StringContent(json, Encoding.UTF8, "application/json");

        var response = await _httpClient.PostAsync(_apiUrl, content);
        response.EnsureSuccessStatusCode();

        var result = await response.Content.ReadAsStringAsync();
        var translation = JsonSerializer.Deserialize<LibreTranslateResponse>(result);
        
        return translation?.translatedText ?? text;
    }
}
```

### Translation Service Selection Guide

**Choose DeepL if:**
- ✅ Quality is top priority
- ✅ Translating European languages
- ✅ Budget allows ($25/1M chars)
- ✅ Professional/business content

**Choose Google Translate if:**
- ✅ Need many languages (100+)
- ✅ Good balance of quality and price
- ✅ Already using Google Cloud services

**Choose Azure Translator if:**
- ✅ Best value for money ($10/1M chars)
- ✅ Using Microsoft ecosystem
- ✅ Need 2M free characters/month

**Choose LibreTranslate if:**
- ✅ Privacy is critical
- ✅ Budget is very limited
- ✅ Can self-host the service
- ✅ Quality requirements are moderate

### Implementation Strategy

1. **Start with Free Tier**: Use free tier of chosen service for testing
2. **Batch Translation**: Translate multiple fields in one API call to save costs
3. **Cache Translations**: Store translated content in database to avoid re-translating
4. **Manual Review**: Always review auto-translated content before publishing
5. **Fallback Strategy**: If API fails, show original language or prompt for manual translation

### Cost Estimation Example

**Scenario**: 100 properties, each with:
- Name: 50 characters
- Description: 500 characters
- Short Description: 200 characters
- Address: 100 characters
- City: 20 characters

**Total per property**: 870 characters
**Total for 100 properties**: 87,000 characters
**For 4 languages**: 348,000 characters

**Cost with DeepL**: 
- First 500K free → $0
- Remaining: 0 (within free tier)

**Cost with Azure**: 
- First 2M free → $0
- Remaining: 0 (within free tier)

**Cost with Google**: 
- First 500K free → $0
- Remaining: 0 (within free tier)

### Dependency Injection Setup

```csharp
// Program.cs
services.AddHttpClient<ITranslationService, DeepLTranslationService>();
// OR
services.AddScoped<ITranslationService, GoogleCloudTranslationService>();
// OR
services.AddScoped<ITranslationService, AzureTranslationService>();
```

---

## Additional Considerations

### RTL (Right-to-Left) Language Support

For languages like Arabic, consider:
- CSS direction: `direction: rtl;`
- Text alignment: `text-align: right;`
- UI mirroring for RTL layouts

### Date and Number Formatting

Different languages may require different formats:
- Date formats: `MM/DD/YYYY` (US) vs `DD/MM/YYYY` (UK)
- Number formats: `1,234.56` vs `1.234,56`
- Currency symbols and positions

### SEO Considerations

- Use language-specific URLs: `/en/property/123` vs `/ar/property/123`
- Add `hreflang` tags for multi-language content
- Language-specific meta descriptions and titles

---

## Troubleshooting

### Common Issues

1. **Translation not showing**: Check if translation exists in database and `LanguageId` is correct
2. **Default language fallback not working**: Verify default language is set in `Languages` table
3. **Performance issues**: Ensure indexes are created on translation tables
4. **API returning wrong language**: Verify `LanguageId` parameter is being passed correctly

---

## License

This implementation guide is provided as-is for reference purposes.

---

## Contributing

For improvements or suggestions, please create an issue or submit a pull request.

---

**Last Updated**: 2024
