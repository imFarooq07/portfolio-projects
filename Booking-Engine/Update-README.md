# 🏨 V3 Booking Engine - Enterprise Hotel Management System

[![.NET](https://img.shields.io/badge/.NET-8.0-purple.svg)](https://dotnet.microsoft.com/)
[![ASP.NET Core](https://img.shields.io/badge/ASP.NET%20Core-MVC-blue.svg)](https://dotnet.microsoft.com/apps/aspnet)
[![SQL Server](https://img.shields.io/badge/Database-SQL%20Server-red.svg)](https://www.microsoft.com/sql-server)
[![License](https://img.shields.io/badge/License-Proprietary-lightgrey.svg)]()

> A comprehensive, enterprise-grade hotel booking and property management system built with ASP.NET Core MVC, featuring advanced booking management, room inventory control, promotion management, multi-language translation support, and role-based access control.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [Technology Stack](#-technology-stack)
- [System Architecture](#-system-architecture)
- [Development Approach](#-development-approach)
- [Core Modules](#-core-modules)
- [Security Features](#-security-features)
- [Project Structure](#-project-structure)
- [Setup Instructions](#-setup-instructions)
- [Tools & Libraries Used](#-tools--libraries-used)
- [Key Achievements](#-key-achievements)
- [Contributions](#-contributions)

---

## 🎯 Overview

**V3 Booking Engine** is a full-featured hotel management system designed to handle complete property operations, from room management and inventory control to booking reservations and payment processing. The system provides a robust, scalable solution for hotel owners and managers to efficiently manage their properties, guests, and business operations.

### What This System Does

- **Property Management**: Create and manage multiple hotel properties with detailed configurations
- **Room Management**: Handle room types, rate plans, facilities, and pricing
- **Booking Management**: Process reservations, cancellations, manual bookings, and payment tracking
- **Inventory Control**: Manage room availability and rate inventories in real-time
- **Promotion Management**: Create and manage promotional offers and discounts
- **User & Role Management**: Implement role-based access control with hierarchical permissions
- **Policy Management**: Configure cancellation policies, payment terms, and booking rules
- **Multi-Language Translation**: Automatic translation system with DeepL API integration for multiple languages
- **Dashboard Analytics**: Real-time metrics and reporting for business insights

---

## ✨ Key Features

### 🏗️ Core Functionality

- **Multi-Property Support**: Manage multiple properties from a single platform
- **Real-Time Inventory**: Track room availability and update rates dynamically
- **Advanced Booking System**: Manual bookings, cancellations, no-shows, and status management
- **Tax Calculation Engine**: Property-specific tax calculations with complex business rules
- **Payment Processing**: Track payments, card details (with OTP security), and payment history
- **Email Integration**: Automated email notifications for bookings and confirmations
- **Image Management**: Upload and manage property and room images
- **Bulk Operations**: Bulk inventory updates and batch processing
- **Multi-Language Support**: Dynamic language selection with cookie persistence and claims-based context

### 🔐 Security Features

- **Role-Based Access Control (RBAC)**: Hierarchical role system with granular permissions
- **OTP-Based Authentication**: Secure login with One-Time Password verification
- **Device Token Management**: Remember trusted devices to skip OTP on subsequent logins
- **Session Management**: Sliding expiration with 24-hour cookie persistence
- **Card Details Security**: OTP-protected card viewing with 10-minute time-limited access
- **Authorization Middleware**: Custom page-level authorization with component-based permissions
- **Global Exception Handling**: Centralized error handling with user-friendly messages

### 📊 Business Logic

- **Property-Specific Tax Rules**: Different calculation methods per property
- **Complex Rate Calculations**: Support for promotions, discounts, and seasonal pricing
- **Booking Validation**: Business rule validation for check-in/check-out dates
- **Inventory Synchronization**: Real-time sync between bookings and availability
- **Abandoned Booking Tracking**: Monitor and recover abandoned reservations

### 🌐 Multi-Language Translation Features

- **Automatic Translation**: DeepL API integration for automatic translation of translatable fields
- **Dynamic Language Selection**: Language dropdown in header with cookie-based persistence
- **Per-Language Settings**: Enable/disable translation for specific languages per property
- **Translation History**: Complete audit trail of all translations with search, filter, and export
- **Translation Analytics**: Dashboard with charts showing translation statistics and cost tracking
- **Revert Functionality**: Restore previous translation versions from history
- **Field Change Detection**: Translation only runs when translatable fields are changed
- **Parallel Processing**: Fast translation using Task.WhenAll for multiple languages
- **Multi-Module Support**: Translation for Property, Room, Rate Plan, Promotion, Addon, and Facilities

---

## 🛠️ Technology Stack

### Backend
- **.NET 8.0**: Latest .NET framework for high performance
- **ASP.NET Core MVC**: Model-View-Controller architecture
- **C#**: Primary programming language
- **Entity Framework Core**: (Via Stored Procedures)
- **SQL Server**: Database with stored procedures

### Frontend
- **Razor Pages (CSHTML)**: Server-side rendering
- **jQuery**: DOM manipulation and AJAX calls
- **Bootstrap 4/5**: Responsive UI framework
- **DataTables**: Advanced table functionality with pagination, sorting, filtering
- **Select2**: Enhanced dropdown components
- **Summernote**: Rich text editor for content management
- **SweetAlert2**: Beautiful alert dialogs
- **Chart.js**: Data visualization for dashboards

### Security & Authentication
- **Cookie Authentication**: Secure session management
- **BCrypt**: Password hashing
- **OTP Service**: Email-based OTP generation
- **Claims-Based Identity**: User claims for authorization

### Infrastructure
- **Dependency Injection**: Built-in IoC container
- **Middleware Pipeline**: Custom middleware for authentication, exceptions, sidebar data, language claims
- **Service Layer Pattern**: Separation of concerns
- **Repository Pattern**: (Via Stored Procedures)

### Translation & Localization
- **DeepL API**: Professional translation service integration
- **Multi-Language Tables**: `*_ML` tables for storing translated content
- **Language Service**: Dynamic language loading from database
- **Translation Helpers**: Module-specific translation helper classes
- **Translation History Service**: Complete audit trail and analytics

---

## 🏛️ System Architecture

### MVC Architecture Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                        Presentation Layer                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │   Views      │  │  Controllers │  │   ViewModels │    │
│  │  (CSHTML)    │  │  (Routing)   │  │   (DTOs)     │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                        Business Logic Layer                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │   Services   │  │  Interfaces  │  │  Helpers     │    │
│  │  (Business)  │  │  (Contracts) │  │  (Utilities) │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                        Data Access Layer                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │  Stored      │  │   Database   │  │  Connection  │    │
│  │  Procedures  │  │   (SQL)      │  │   Management │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Request Flow

```
User Request → Controller → Service Layer → Stored Procedure → Database
                ↓              ↓
            ViewModel      Business Logic
                ↓
            View (CSHTML) → Response
```

### Data Flow

1. **User Action**: User interacts with UI (form submission, button click)
2. **Controller**: Receives HTTP request, validates model, calls service
3. **Service Layer**: Executes business logic, calls stored procedures
4. **Database**: Stored procedure executes, returns data
5. **Response**: Data flows back through service → controller → view
6. **UI Update**: View renders with data, AJAX updates DOM if needed

---

## 🚀 Development Approach

### 1. **Layered Architecture**
- **Separation of Concerns**: Clear separation between presentation, business, and data layers
- **Service Pattern**: Business logic encapsulated in service classes
- **DTO Pattern**: Data Transfer Objects for clean data passing
- **Dependency Injection**: Loose coupling through IoC container

### 2. **Stored Procedure Strategy**
- **Database-First Approach**: Business logic in SQL stored procedures
- **Performance Optimization**: Pre-compiled queries for better performance
- **Security**: Parameterized queries prevent SQL injection
- **Maintainability**: Complex queries centralized in database

### 3. **Frontend Development**
- **AJAX-Based Interactions**: Seamless user experience without page reloads
- **FormData Collection**: Manual form data collection for complex forms (Select2, Summernote)
- **Client-Side Validation**: jQuery Validation with Data Annotations
- **Responsive Design**: Mobile-first approach with Bootstrap

### 4. **Error Handling Strategy**
- **Global Exception Middleware**: Centralized error handling
- **User-Friendly Messages**: Custom error messages instead of raw SQL errors
- **Error Logging**: Database logging with email notifications
- **Graceful Degradation**: AJAX errors handled with JSON responses

### 5. **Security Implementation**
- **Authorization Attributes**: `[Authorize]` on controllers, `[PageAuthorization]` for granular control
- **Claims-Based Identity**: User information stored in claims
- **Session Management**: Sliding expiration with cookie persistence
- **OTP Security**: Multi-factor authentication for sensitive operations

### 6. **Code Quality Practices**
- **Null-Safe Operations**: Extensive use of null-conditional operators and TryParse
- **DBNull Handling**: Safe conversion methods for database values
- **Async/Await**: Asynchronous operations throughout for scalability
- **Clean Code**: Meaningful variable names, helper methods, code organization

---

## 📦 Core Modules

### 1. **Property Management**
- **Create/Update Property**: Full property configuration with images, facilities, taxes
- **Additional Details**: Extended property information and settings
- **Addon Management**: Create and manage property add-ons (activities, services)
- **Property Mapping**: Map properties to users and groups

**Key Files:**
- `Controllers/PropertyController.cs`
- `Services/PropertyService/PropertyManagementService.cs`
- `Views/Property/CreateProperty.cshtml`, `UpdateProperty.cshtml`, `CreateAddon.cshtml`

### 2. **Room Management**
- **Room CRUD**: Create, update, delete room types
- **Rate Plan Management**: Create rate plans with pricing, facilities, and variations
- **Room Images**: Upload and manage room images
- **Copy Rate Plan**: Duplicate existing rate plans for quick setup

**Key Files:**
- `Controllers/RoomManagementController.cs`
- `Services/RoomManagementService/RoomManagementService.cs`
- `Views/RoomManagement/CreateRoom.cshtml`, `CreateRatePlan.cshtml`

### 3. **Booking Management**
- **Reservation List**: View all bookings with filtering and pagination
- **Manual Booking**: Create bookings manually with guest details
- **Booking Operations**: Cancel, no-show, complete status updates
- **Payment Tracking**: View payment history and card details (OTP-protected)
- **Confirmation Booking**: Generate booking confirmations with tax calculations
- **Abandoned Bookings**: Track and recover abandoned reservations

**Key Files:**
- `Controllers/BookingManagementController.cs`
- `Services/ReservationService/ReservationService.cs`
- `Views/BookingManagement/ReservationList.cshtml`, `ManualBooking.cshtml`

### 4. **Inventory Management**
- **Bulk Inventory Updates**: Update multiple rate inventories at once
- **Availability Management**: Real-time availability tracking
- **Rate Inventory**: Manage room rates and availability dates

**Key Files:**
- `Controllers/InventoryManagementController.cs`
- `Services/InventoryManagementService/InventoryManagementService.cs`
- `Views/InventoryManagement/BulkInventory.cshtml`

### 5. **Promotion Management**
- **Create Promotions**: Set up promotional offers and discounts
- **Promotion Rules**: Configure promotion conditions and validity periods
- **Rate Plan Integration**: Link promotions to rate plans

**Key Files:**
- `Controllers/PromotionManagementController.cs`
- `Services/PromotionManagementService/PromotionManagementService.cs`
- `Views/PromotionManagement/CreatePromotion.cshtml`

### 6. **Policy Management**
- **Cancellation Policies**: Define cancellation rules and penalties
- **Payment Terms**: Configure payment terms and conditions
- **Booking Policies**: Set booking rules and restrictions

**Key Files:**
- `Controllers/PolicyManagementController.cs`
- `Services/PolicyManagementService/PolicyManagementService.cs`
- `Views/PolicyManagement/CreatePolicy.cshtml`

### 7. **User Management**
- **User CRUD**: Create, update, delete users with role assignment
- **Role Management**: Create roles with hierarchical parent-child relationships
- **Group Management**: Organize users into groups
- **Permission Assignment**: Assign page-level permissions to roles

**Key Files:**
- `Controllers/UserManagementController.cs`
- `Services/AuthService/UserService.cs`
- `Services/PermissionService/PermissionService.cs`
- `Views/UserManagement/CreateUser.cshtml`, `CreateRole.cshtml`

### 8. **Dashboard & Analytics**
- **Real-Time Metrics**: Booking statistics, revenue, occupancy rates
- **Monthly Bookings Chart**: Visual representation of booking trends
- **Key Performance Indicators**: Revenue, bookings, cancellations

**Key Files:**
- `Controllers/HomeController.cs`
- `Services/DashboardService/DashboardService.cs`
- `Views/Home/Dashboard.cshtml`

### 9. **Multi-Language Translation Management**
- **Automatic Translation**: DeepL API integration for automatic translation
- **Language Selection**: Dynamic language dropdown with cookie persistence
- **Translation History**: Complete audit trail with search, filters, and export
- **Translation Analytics**: Dashboard with charts and cost tracking
- **Per-Language Settings**: Enable/disable translation per language per property
- **Revert Translation**: Restore previous translation versions
- **Bulk Translation**: Translate existing room groups and facilities
- **Supported Modules**: Property, Room, Rate Plan, Promotion, Addon, Facilities

**Translatable Fields:**
- **Property**: AccommodationName, KeyCollectionComments, PetsAllowedComments
- **Addon**: ActivityName, ShortDescription, CancellationPolicy, GuaranteePolicy, LongDescription
- **Room**: RoomName, ApartmentName, RoomDescription
- **Rate Plan**: RatePlanName, DisplayRatePlanName, Included, Highlight, MealDescription
- **Promotion**: PromotionName, Description
- **Facilities**: GroupName, FacilityName, OtherGroupName, OtherFacilityName

**Key Files:**
- `Controllers/PropertyController.cs` (TranslationHistory, TranslationAnalytics, PerLanguageSettings)
- `Services/TranslationService/TranslationService.cs`
- `Services/TranslationService/TranslationHistoryService.cs`
- `Services/TranslationService/TranslationAnalyticsService.cs`
- `Services/LanguageService/LanguageService.cs`
- `Services/PropertyService/PropertyTranslationHelper.cs`
- `Services/PropertyService/AddonTranslationHelper.cs`
- `Services/RoomManagementService/RoomTranslationHelper.cs`
- `Services/RoomManagementService/RatePlanTranslationHelper.cs`
- `Services/PromotionManagementService/PromotionTranslationHelper.cs`
- `Middleware/LanguageClaimUpdateMiddleware.cs`
- `Views/Property/TranslationHistory.cshtml`
- `Views/Property/TranslationAnalytics.cshtml`
- `Views/Property/PerLanguageSettings.cshtml`
- `wwwroot/js/language-selector.js`

**URLs:**
- Translation History: `/Property/TranslationHistory`
- Translation Analytics: `/Property/TranslationAnalytics`
- Per-Language Settings: `/Property/PerLanguageSettings`
- Get Languages API: `/Home/GetLanguages`

---

## 🔒 Security Features

### Authentication & Authorization

1. **Cookie-Based Authentication**
   - Secure cookie with HttpOnly, Secure, SameSite flags
   - 24-hour sliding expiration
   - Automatic session renewal on activity

2. **OTP Verification**
   - Email-based OTP for login
   - Device token persistence (skip OTP on trusted devices)
   - 6-digit numeric OTP with expiration

3. **Role-Based Access Control**
   - Hierarchical role system
   - Page-level permissions
   - Component-based authorization
   - Session-cached permissions

4. **Card Details Security**
   - OTP-protected card viewing
   - 10-minute time-limited access
   - One-time use OTP
   - Session-based storage

5. **Global Authorization**
   - All controllers protected with `[Authorize]` (except Account)
   - Custom `PageAuthorizationAttribute` for granular control
   - Graceful handling of expired sessions

---

## 📁 Project Structure

```
V3BookingEngine/
├── Controllers/              # MVC Controllers (10 controllers)
│   ├── AccountController.cs
│   ├── PropertyController.cs
│   ├── RoomManagementController.cs
│   ├── BookingManagementController.cs
│   └── ...
│
├── Services/                 # Business Logic Layer (15+ services)
│   ├── PropertyService/
│   ├── RoomManagementService/
│   ├── ReservationService/
│   ├── AuthService/
│   ├── PermissionService/
│   └── ...
│
├── DTOs/                     # Data Transfer Objects
│   ├── Property/
│   ├── RoomManagement/
│   ├── Reservations/
│   ├── Users/
│   └── ...
│
├── Views/                    # Razor Views (CSHTML)
│   ├── Property/
│   ├── RoomManagement/
│   ├── BookingManagement/
│   ├── UserManagement/
│   └── ...
│
├── Middleware/               # Custom Middleware
│   ├── GlobalExceptionMiddleware.cs
│   ├── AuthenticationRehydrationMiddleware.cs
│   ├── SidebarDataMiddleware.cs
│   └── LanguageClaimUpdateMiddleware.cs
│
├── Services/                 # Business Logic Layer
│   ├── TranslationService/   # Translation Services
│   │   ├── TranslationService.cs
│   │   ├── TranslationHistoryService.cs
│   │   ├── TranslationAnalyticsService.cs
│   │   └── Providers/
│   │       └── DeepLTranslationProvider.cs
│   ├── LanguageService/      # Language Management
│   │   └── LanguageService.cs
│   ├── PropertyService/      # Property Services
│   │   ├── PropertyTranslationHelper.cs
│   │   └── AddonTranslationHelper.cs
│   ├── RoomManagementService/
│   │   ├── RoomTranslationHelper.cs
│   │   └── RatePlanTranslationHelper.cs
│   └── PromotionManagementService/
│       └── PromotionTranslationHelper.cs
│
├── Helpers/                  # Utility Classes
│   ├── ErrorLoggingHelper.cs
│   ├── ServiceRegistration.cs
│   ├── TranslatableFieldsConfig.cs
│   └── FieldChangeDetector.cs
│
├── Attributes/               # Custom Validation Attributes
│   ├── AmountValidationAttribute.cs
│   └── DuplicatePropertyNameAttribute.cs
│
├── wwwroot/                  # Static Files
│   ├── theme/
│   ├── css/
│   ├── js/
│   └── lib/
│
└── Database_Script/          # Database Scripts
    └── SP_*.sql
```

---

## 📚 Tools & Libraries Used

### Development Tools
- **Visual Studio 2022**: Primary IDE
- **Git**: Version control
- **SQL Server Management Studio**: Database management
- **Postman**: API testing (if applicable)

### NuGet Packages

| Package | Version | Purpose |
|---------|---------|---------|
| `Microsoft.AspNetCore.Authentication.Cookies` | 2.3.0 | Cookie authentication |
| `Microsoft.AspNetCore.Session` | 2.3.0 | Session management |
| `Microsoft.Data.SqlClient` | 6.1.1 | SQL Server connectivity |
| `BCrypt.Net-Next` | 4.0.3 | Password hashing |
| `MailKit` | 4.13.0 | Email sending |
| `Newtonsoft.Json` | 13.0.4 | JSON serialization |
| `System.Drawing.Common` | 9.0.9 | Image processing |

### Frontend Libraries

| Library | Purpose |
|---------|---------|
| **jQuery** | DOM manipulation, AJAX |
| **Bootstrap 4/5** | Responsive UI framework |
| **DataTables** | Advanced table functionality |
| **Select2** | Enhanced dropdowns |
| **Summernote** | Rich text editor |
| **SweetAlert2** | Alert dialogs |
| **Chart.js** | Data visualization |
| **DeepL API** | Professional translation service |

---

## 🎯 Key Achievements

### Technical Achievements

1. **✅ Complex Form Data Handling**
   - Implemented manual FormData collection for Select2, Summernote, and dynamic fields
   - Resolved model binding issues with custom field name cleaning
   - Fixed checkbox state handling for backend binding

2. **✅ Advanced Error Handling**
   - Custom SQL exception handling with user-friendly messages
   - Global exception middleware for centralized error management
   - Graceful session expiration handling without error pages

3. **✅ Security Implementation**
   - Implemented OTP-based authentication with device token persistence
   - Role-based access control with hierarchical permissions
   - OTP-protected card details viewing system

4. **✅ Data Validation & Safety**
   - Null-safe operations throughout the codebase
   - DBNull handling with helper methods
   - Custom validation attributes for business rules

5. **✅ UI/UX Improvements**
   - Converted forms to AJAX submission with SweetAlert2
   - Consistent styling across all pages
   - Responsive design with mobile support

6. **✅ Performance Optimization**
   - Stored procedure usage for database operations
   - Async/await patterns for scalability
   - Session caching for permissions
   - Parallel translation processing using Task.WhenAll

7. **✅ Multi-Language Translation System**
   - DeepL API integration for automatic translation
   - Dynamic language selection with cookie persistence
   - Translation history tracking with complete audit trail
   - Translation analytics dashboard with cost tracking
   - Per-language translation settings per property
   - Revert translation functionality
   - Field change detection for optimized translation
   - Support for 8+ modules (Property, Room, Rate Plan, Promotion, Addon, Facilities)

### Business Logic Achievements

1. **Property-Specific Tax Calculation**: Complex tax calculation engine supporting different properties
2. **Rate Plan Copy Feature**: Quick duplication of rate plans for efficiency
3. **Bulk Inventory Management**: Batch updates for inventory operations
4. **Abandoned Booking Recovery**: Track and recover lost bookings
5. **Manual Booking Creation**: Admin capability to create bookings manually
6. **Multi-Language Translation System**: Complete translation infrastructure with DeepL API integration, supporting automatic translation for 8+ modules with history tracking and analytics

---

## 💻 Development Highlights

### Approach Used

1. **MVC Pattern**: Clear separation of concerns
2. **Service Layer Architecture**: Business logic in services
3. **Stored Procedure Strategy**: Database-first approach
4. **AJAX-Based UI**: Seamless user experience
5. **Component-Based Security**: Granular permission control

### Code Quality

- **Async/Await**: Asynchronous operations throughout
- **Dependency Injection**: Loose coupling
- **Error Handling**: Comprehensive exception handling
- **Null Safety**: Extensive null checking
- **Clean Code**: Well-organized, readable code

### Problem-Solving

- **Form Data Collection**: Custom FormData collection for complex forms
- **Model Binding**: Fixed binding issues with manual data mapping
- **Session Management**: Implemented sliding expiration
- **Error Messages**: User-friendly error messages instead of technical errors
- **Security**: Multi-layer authentication and authorization

---

## 📝 Contributions

### Developer: [M.Farooq]

**Role**: Full-Stack Developer  
**Responsibilities**:
- Backend development (Controllers, Services, Business Logic)
- Frontend development (Views, JavaScript, AJAX)
- Database interaction (Stored Procedures)
- Security implementation (Authentication, Authorization)
- Error handling and logging
- UI/UX improvements
- Multi-language translation system implementation

### Key Contributions

1. **Form Data Collection Fix**: Implemented manual FormData collection for Select2, Summernote, and dynamic fields
2. **Model Binding Resolution**: Fixed checkbox and array binding issues
3. **Error Handling Enhancement**: User-friendly error messages with SQL exception handling
4. **Security Features**: OTP authentication, role-based access control, card details protection
5. **UI Consistency**: Converted all forms to AJAX with SweetAlert2
6. **Session Management**: Implemented sliding expiration and graceful session handling
7. **Authorization System**: Global authorization with page-level permissions
8. **Multi-Language Translation**: Complete translation system with DeepL API, translation history, analytics dashboard, per-language settings, and revert functionality

---

## 📄 License

This project is proprietary software. All rights reserved.

---

## 🔗 Links

- **Live Demo**: [https://v3-bw.bookingwhizz.com](https://v3-bw.bookingwhizz.com)
- **Repository**: https://github.com/imFarooq07/portfolio-projects/tree/main/

---

## 📞 Contact

For questions or inquiries about this project, please contact:
- **Email**: imfarooq1995@gmail.com
- **LinkedIn**: https://www.linkedin.com/in/muhammad-farooq-4598a3135 

---

**Built with ❤️ using ASP.NET Core MVC**

*Last Updated: Nov 2025*

