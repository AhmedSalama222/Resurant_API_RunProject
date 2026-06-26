# Restaurant API — باك اند مطعم Le Gourmet

باك اند كامل بـ ASP.NET Core (.NET 10) + Entity Framework Core + SQL Server + JWT Auth،
مبني بهيكل مبسط: **Controllers → Services → Repositories**.

---

## 1) المتطلبات قبل ما تبدأ

- **.NET 10 SDK** — تأكد إنه متظبت: `dotnet --version`
- **SQL Server** — أسهل حل هو **SQL Server LocalDB** (بييجي مع Visual Studio تلقائي)، أو SQL Server Express، أو حتى Docker لو حابب
- **Visual Studio 2026** أو **VS Code + C# Dev Kit**
- أداة `dotnet-ef` للـ Migrations:
  ```bash
  dotnet tool install --global dotnet-ef
  ```

---

## 2) إنشاء المشروع

```bash
dotnet new webapi -n RestaurantAPI --use-controllers
cd RestaurantAPI
```

الفلاج `--use-controllers` مهم، لأن بدونه .NET بيطلعلك Minimal API بدل Controllers.

شيل الملفين الافتراضيين اللي مش محتاجهم:
```bash
rm Controllers/WeatherForecastController.cs
rm WeatherForecast.cs
```

---

## 3) تركيب الـ NuGet Packages

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add package BCrypt.Net-Next
dotnet add package Swashbuckle.AspNetCore
```

---

## 4) نسخ الملفات

نسخ كل الفولدرات والملفات اللي جوه الزيب ده (Models, DTOs, Data, Repositories,
Services, Helpers, Middlewares, Exceptions, Controllers) جوه فولدر المشروع `RestaurantAPI/`،
وكذلك استبدل `Program.cs` و `appsettings.json` بالموجودين هنا.

شكل المشروع النهائي المفروض يكون كده:

```
RestaurantAPI/
├── Program.cs
├── appsettings.json
├── Controllers/
│   ├── AuthController.cs
│   ├── CategoriesController.cs
│   ├── MenuItemsController.cs
│   ├── OrdersController.cs
│   ├── ReviewsController.cs
│   └── ContactController.cs
├── Models/
│   ├── Enums/ (UserRole, OrderStatus)
│   ├── User.cs / Category.cs / MenuItem.cs
│   └── Order.cs / OrderItem.cs / Review.cs / ContactMessage.cs
├── DTOs/  (Auth, Categories, MenuItems, Orders, Reviews, Contact)
├── Data/  (AppDbContext.cs, SeedData.cs)
├── Repositories/ (Generic + خاص بكل كيان محتاج Include)
├── Services/ (المنطق البرمجي لكل feature)
├── Helpers/ (JwtSettings, JwtTokenGenerator)
├── Middlewares/ (ExceptionMiddleware)
└── Exceptions/ (NotFoundException, BadRequestException)
```

---

## 5) تعديل appsettings.json

الملف جاهز بقيم افتراضية، بس لازم تتأكد من:

- **Connection String**: لو عندك LocalDB فمفيش حاجة تتغير. لو SQL Server Express أو سيرفر تاني، غيّر `Server=...` على حسب اسم السيرفر بتاعك.
- **JwtSettings.Key**: غيّرها لقيمة سرية خاصة بيك (32 حرف على الأقل)، خصوصًا قبل أي رفع حقيقي (Production).

---

## 6) إنشاء قاعدة البيانات (Migrations)

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

ده هيعمل الجداول كلها في SQL Server، وأول ما تشغل المشروع هيتعمل Seed أوتوماتيك لـ:
- 5 تصنيفات (بيتزا، برجر، أطباق رئيسية، حلويات، مشروبات) — مطابقة لتصنيفات المنيو في الموقع
- مستخدم Admin افتراضي:
  - **Email:** `admin@legourmet.com`
  - **Password:** `Admin@123`

> غيّر باسورد الأدمن أول ما تجرب المشروع.

---

## 7) تشغيل المشروع

```bash
dotnet run
```

افتح المتصفح على رابط Swagger اللي هيظهر في الترمينال (مثلاً `https://localhost:7xxx/swagger`)
وجرب الـ Endpoints مباشرة من هناك. لو حابب تجرب Endpoint محتاج تسجيل دخول:
1. اعمل Login من `/api/auth/login`
2. خد الـ Token من الرد
3. دوس على زرار **Authorize** فوق في Swagger واكتب: `Bearer {التوكن}`

---

## 8) ملخص الـ Endpoints

### Auth (`/api/auth`)
| Method | Route | Auth | الوظيفة |
|---|---|---|---|
| POST | /register | عام | تسجيل حساب جديد (Customer) |
| POST | /login | عام | تسجيل دخول وإرجاع JWT |

### Categories (`/api/categories`)
| Method | Route | Auth | الوظيفة |
|---|---|---|---|
| GET | / | عام | كل التصنيفات |
| GET | /{id} | عام | تصنيف واحد |
| POST | / | Admin | إضافة تصنيف |
| PUT | /{id} | Admin | تعديل تصنيف |
| DELETE | /{id} | Admin | حذف تصنيف |

### MenuItems (`/api/menuitems`)
| Method | Route | Auth | الوظيفة |
|---|---|---|---|
| GET | /?categoryId=&featured= | عام | كل الأطباق (مع فلترة اختيارية) |
| GET | /{id} | عام | طبق واحد |
| POST | / | Admin | إضافة طبق |
| PUT | /{id} | Admin | تعديل طبق |
| DELETE | /{id} | Admin | حذف طبق |

### Orders (`/api/orders`)
| Method | Route | Auth | الوظيفة |
|---|---|---|---|
| POST | / | مسجل دخول | إنشاء طلب جديد من السلة |
| GET | /my | مسجل دخول | طلباتي |
| GET | /{id} | مسجل دخول (المالك أو Admin) | تفاصيل طلب |
| GET | /?status= | Admin | كل الطلبات (مع فلترة بالحالة) |
| PUT | /{id}/status | Admin | تغيير حالة الطلب |

### Reviews (`/api/reviews`)
| Method | Route | Auth | الوظيفة |
|---|---|---|---|
| GET | / | عام | التقييمات المعتمدة بس (تظهر في الهوم بيدج) |
| GET | /all | Admin | كل التقييمات (حتى غير المعتمدة) |
| POST | / | مسجل دخول | إضافة تقييم |
| PUT | /{id}/approve | Admin | اعتماد تقييم |
| DELETE | /{id} | Admin | حذف تقييم |

### Contact (`/api/contact`)
| Method | Route | Auth | الوظيفة |
|---|---|---|---|
| POST | / | عام | إرسال رسالة من فورم التواصل |
| GET | / | Admin | كل الرسائل |
| PUT | /{id}/read | Admin | تعليم رسالة كمقروءة |
| DELETE | /{id} | Admin | حذف رسالة |

---

## 9) ملاحظة عن CORS

في `Program.cs` مضاف Policy اسمه `AllowFrontend` بيسمح لدومين الفرونت إند
(`restaurant-three-puce.vercel.app`) ولـ localhost وقت التطوير. لو غيرت دومين الفرونت إند
أو شغلته على بورت تاني، عدّل القائمة في `Program.cs`.

---

## 10) خطوات تالية ممكن تضيفها بعدين

- Refresh Tokens (عشان الـ JWT حاليًا صالح يوم واحد بس بدون تجديد)
- رفع صور الأطباق فعليًا (دلوقتي بس `ImageUrl` كنص)
- ربط الفرونت إند الموجود (HTML/JS) بالـ API ده عن طريق `fetch`
