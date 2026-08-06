---
name: TripleSix.Core
description: Bộ quy tắc lập trình, chuẩn thiết kế Domain Entities, Services, Unit Tests và quy chuẩn code cho các dự án sử dụng TripleSix.Core.
---

# TripleSixCore Coding Rules & Standards

Quy tắc thiết kế và cấu trúc mã nguồn đối với các thực thể (Entities), Services, Unit Tests trong hệ thống.

## 1. Quy tắc Tuyệt đối (Highest Priority Rule)

- **TUYỆT ĐỐI KHÔNG** tự động thực hiện hay tự chạy lệnh `dotnet ef database update` dưới bất kỳ hình thức nào.
- Mọi thao tác cập nhật Database/Migration phải do người dùng tự thực hiện hoặc trực tiếp yêu cầu.

## 2. Định nghĩa Giới tính (Genders Enum)

- Không sử dụng kiểu `bool` (`True/False`) để biểu diễn giới tính.
- Luôn sử dụng Enum `Genders` (nằm trong thư mục `Domain/Constants`) với các giá trị:
  - `Male = 1` (Nam)
  - `Female = 2` (Nữ)
- Gắn thêm thuộc tính `[Description("...")]` cho từng giá trị enum để hiển thị tiếng Việt.

## 3. Khởi tạo Collection (Collection Expressions)

- Mọi thuộc tính danh sách `ICollection<T>` phải được khởi tạo bằng cú pháp rút gọn C# 12 (`= [];`) thay vì `new List<T>()` hay `new HashSet<T>()`.

## 4. Cấu hình Quan hệ giữa các Entity (Data Annotations & Virtual Proxies)

- Ưu tiên cấu hình quan hệ bằng Data Annotations đặt trực tiếp trên Navigation Properties:
  - Sử dụng `[ForeignKey(nameof(ForeignId))]` để chỉ định khóa ngoại.
  - Sử dụng `[DeleteBehavior(DeleteBehavior.Restrict)]` để ngăn chặn hành vi xóa cascade mặc định của EF Core.
- **Tất cả các Navigation Properties và Collections (`ICollection<T>`) trong Entity BẮT BUỘC phải được khai báo dạng `virtual`** để tương thích với cơ chế EF Core `UseLazyLoadingProxies` / `UseChangeTrackingProxies`.
- Chỉ sử dụng Fluent API (`Configure` method) cho các trường hợp đặc biệt mà Data Annotations không hỗ trợ (như cấu hình Index hoặc cài đặt kiểu dữ liệu cột).

## 5. Cấu trúc và Phân đoạn thực thể (Entity Structure)

Bên trong một class Entity, code cần được sắp xếp và phân vùng theo thứ tự từ trên xuống dưới bằng các comment sau:

1. Các thuộc tính dữ liệu thông thường.
2. `// Navigation properties`: Chứa các thực thể liên kết đơn (kèm theo các attributes `[ForeignKey]`, `[DeleteBehavior]`).
3. `// Collections`: Chứa các danh sách thực thể `ICollection<T>` liên kết nhiều (khởi tạo bằng `= [];`).
4. `// Configuration`: Đặt ngay phía trên phương thức `Configure(EntityTypeBuilder<T> builder)`.

## 6. Comment cho thuộc tính bool

- Đối với các thuộc tính kiểu `bool` hoặc `bool?`, Comment mô tả phải được viết dưới dạng câu hỏi kết thúc bằng dấu chấm hỏi.
  - _Ví dụ tốt_: `[Comment("Là khách hàng VIP ?")]`, `[Comment("Có yêu cầu xuất hóa đơn ?")]`
  - _Ví dụ không tốt_: `[Comment("Khách hàng VIP (True: Đúng, False: Sai)")]`

## 7. Ghi chú cho thuộc tính Id và Code (Comment for Id vs Code)

- Các trường định danh (như `WorkflowId`, `SiteId`,...) phải sử dụng từ `"Id"` trong comment mô tả.
  - _Ví dụ tốt_: `[Comment("Id quy trình")]`, `[Comment("Id chi nhánh")]`
  - _Ví dụ không tốt_: `[Comment("Mã định danh quy trình")]`, `[Comment("Mã chi nhánh")]`
- Chỉ những thuộc tính mã hiển thị nghiệp vụ (như `Code`, `ProvinceCode`,...) mới sử dụng từ `"Mã"`.
  - _Ví dụ tốt_: `[Comment("Mã vụ việc")]`, `[Comment("Mã tỉnh thành")]`

## 8. Cấu hình Index cho thực thể (Database Indexes)

- Không sử dụng Data Annotation `[Index]` ở cấp class.
- Mọi Index (Unique hoặc Non-unique) phải được cấu hình bằng Fluent API trong phương thức `Configure` của Entity.
- **Không khai báo Index dư thừa cho thuộc tính khóa ngoại (`[ForeignKey]`)**: Các trường thuộc tính khóa ngoại đã được chỉ định trong `[ForeignKey(nameof(...))]` trên Navigation Property thì **tuyệt đối không** khai báo thêm `builder.HasIndex(x => x.ForeignId);` trong phương thức `Configure`.
- Định dạng cấu hình index phải được viết trên cùng một dòng.
  - _Ví dụ tốt_: `builder.HasIndex(x => x.Code).IsUnique();` hoặc `builder.HasIndex(x => x.Code);`
  - _Ví dụ không tốt_: tách `.IsUnique();` xuống dòng tiếp theo.

## 9. Quy tắc Git Commit & Push của Agent

- **Tuyệt đối không** tự động thực hiện các lệnh `git commit` / `git push` trên dự án mã nguồn (source code project) dưới bất kỳ hình thức nào trừ khi có yêu cầu trực tiếp từ người dùng.
- Việc cập nhật nội dung các tệp tin trong Skill (ví dụ: `C:\Users\tripl\.gemini\config\skills\TripleSix.Core\`) được phép ghi nhận trực tiếp.


## 10. Sử dụng Primary Constructor

- Ưu tiên sử dụng cú pháp Primary Constructor (C# 12) cho các class khi định nghĩa constructor (ví dụ: các services kế thừa `BaseService<T>`) để giữ code ngắn gọn và nhất quán.

## 11. Câu lệnh điều kiện đơn & XML Documentation (Shorthand If & Comments)

- Đối với các câu lệnh điều kiện `if` chỉ chứa một câu lệnh thực thi duy nhất, tuyệt đối không sử dụng cặp dấu ngoặc nhọn `{}`.
- Hãy ưu tiên viết trên cùng một dòng. Nếu câu lệnh quá dài, có thể xuống dòng (thụt lề 1 tab) nhưng vẫn không sử dụng dấu ngoặc nhọn `{}`.
  - _Ví dụ tốt_:
    `if (siteCode.IsNullOrEmpty()) throw new Exception("Tài khoản chưa được cấu hình chi nhánh.");`
    Hoặc khi câu lệnh dài:
    ```csharp
    if (siteCode.IsNullOrEmpty())
        throw new Exception("Tài khoản chưa được cấu hình chi nhánh.");
    ```
  - _Ví dụ không tốt_:
    ```csharp
    if (siteCode.IsNullOrEmpty())
    {
        throw new Exception("Tài khoản chưa được cấu hình chi nhánh.");
    }
    ```
- **Định dạng XML Documentation**: Tất cả XML summary/docstring comment cho class, property, method phải **luôn kết thúc bằng dấu chấm `.`**.

## 12. Cấu trúc lớp Service (Service Class Structure)

- Mọi Service nên được chia làm 2 `partial class`:
  - `partial class` thứ nhất: Chứa định nghĩa Primary Constructor (C# 12) kế thừa `BaseService<T>` (hoặc `StrongService<T>` đối với thực thể `StrongEntity`), khai báo các dependencies, biến thành viên, các properties/fields được inject (private, protected, hoặc public) và **tất cả các phương thức helper `protected` / `private`** không nằm trong interface.
  - `partial class` thứ hai: Kế thừa và cài đặt các hàm được định nghĩa trong interface của Service.
- **Bắt buộc Kế thừa StrongService cho StrongEntity**:
  - Đối với mọi thực thể (Entity) kế thừa `StrongEntity<T>` (hoặc triển khai `IStrongEntity`), lớp Service tương ứng **BẮT BUỘC** phải kế thừa `StrongService<T>(db)` thay vì `BaseService<T>(db)`.
  - _Ví dụ tốt_: `public partial class CustomerService(IDbDataContext db) : StrongService<Customer>(db)`
  - _Ví dụ không tốt_: `public partial class CustomerService(IDbDataContext db) : BaseService<Customer>(db)`
- **Lớp cơ sở StrongCodeService cho các Thực thể có trường Mã (Code)**:
  - Đối với các thực thể `StrongEntity` có thuộc tính `Code`, Interface của Service kế thừa `IStrongCodeService<TEntity>` và Class Service kế thừa `StrongCodeService<TEntity>(db)` (nằm tại thư mục `Src/Application/Abstracts/StrongCodeService.cs`).
  - `StrongCodeService<TEntity>` tự động tích hợp sẵn các phương thức `GetFirstByCode` và `GetFirstOrDefaultByCode` (cho cả Entity và generic `TResult`), giúp loại bỏ mã trùng lặp giữa các Service.
- **Kế thừa Strong Service Interfaces trong Service Interface**:
  - Đối với các Service có bộ DTOs Admin tương ứng (`FilterAdminDto`, `CreateAdminDto`, `UpdateAdminDto`), interface của Service phải kế thừa đầy đủ các strong interfaces từ `TripleSix.Core`:
    ```csharp
    public interface I[Entity]Service : IService<[Entity]>,
        IStrongServiceRead<[Entity], [Entity]FilterAdminDto>,
        IStrongServiceCreate<[Entity], [Entity]CreateAdminDto>,
        IStrongServiceUpdate<[Entity], [Entity]UpdateAdminDto>
    ```
- **Chỉ tạo Service tương ứng với Entity có thật**:
  - Chỉ tạo lớp Service (kế thừa `BaseService<T>` / `StrongService<T>` / `StrongCodeService<T>`) cho các thực thể Entity thực sự tồn tại trong Domain (ví dụ: `Product` -> `ProductService`, `Customer` -> `CustomerService`). Tránh tạo Service khi không có thực thể tương ứng.
- **Ưu tiên tiêm và sử dụng Lớp Service thay vì truy vấn Db trực tiếp**:
  - Khi một Service thao tác hoặc tương tác với thực thể khác (ngoài thực thể chính của Service đó), nếu thực thể đó đã có lớp Service tương ứng (ví dụ: `Ticket` -> `ITicketService`, `WorkflowState` -> `IWorkflowStateService`, `Customer` -> `ICustomerService`), **BẮT BUỘC** phải tiêm (inject) và gọi các phương thức thông qua lớp Service đó (như `TicketService.GetFirstById(...)`, `WorkflowStateService.GetFirstByCode(...)`, `CustomerService.Upsert(...)`, `TicketService.Update(...)`) thay vì truy vấn EF Core trực tiếp qua `Db.[Entity].FirstOrDefaultAsync(...)` nhằm giữ tính đóng gói nghiệp vụ và đảm bảo tính nhất quán.
- **Autofac Property Injection & Không kiểm tra Null dư thừa**: Các dependency Service thuộc container được Autofac tiêm tự động thông qua `PropertiesAutowired()`. **Tuyệt đối không viết các câu lệnh kiểm tra null dư thừa** như `if (Service != null)` trước khi gọi phương thức của Service đó.
- **Tái sử dụng logic giữa phương thức Generic và Non-generic overload**:
  - Khi Service định nghĩa cặp phương thức overload cùng thực hiện một tác vụ tra cứu (ví dụ: `GetListNextState(...)` trả về `List<Entity>` và `GetListNextState<TResult>(...)` trả về `List<TResult>`), phương thức non-generic **bắt buộc phải gọi lại phương thức generic** với type parameter là chính thực thể Entity đó (`return await GetListNextState<WorkflowState>(...);`) để tập trung duy nhất một logic truy vấn, tránh lặp mã nguồn (DRY) và dễ bảo trì.

## 13. Sắp xếp Alphabet và Thứ tự Inject (Alphabetical Sorting & Injection Order)

- Đối với các thuộc tính/dependencies được tiêm (inject / ProjectInject) trong Service:
  - `IApplicationDbContext Db` luôn được ưu tiên đặt ở vị trí **trên cùng** (đầu tiên).
  - Tiếp theo vị trí thứ 2 là `IBackgroundJobClient BackgroundJobClient` (nếu có).
  - Các dependencies là các Lớp Service khác (`I[Entity]Service`) được sắp xếp theo thứ tự bảng chữ cái (Alphabetical order).
  - Các dependencies là Request Client (`I[Name]RequestClient`) **bắt buộc phải nằm ở vị trí bên dưới tất cả các Service** và được sắp xếp theo thứ tự bảng chữ cái giữa các Request Client với nhau.
- **Sắp xếp API Endpoints trong Controller**:
  - Tất cả các API Endpoints trong Controller **bắt buộc phải được sắp xếp theo thứ tự bảng chữ cái (Alphabetical order - ABC)** dựa trên đường dẫn Route URL (`[HttpPost("{id}/...")]`, `[HttpGet("{id}/...")]`, `[HttpDelete("{id}/...")]`).
  - Các phương thức cùng thuộc một tài nguyên (ví dụ: `Install/Material`) được gom nhóm lại gần nhau theo thứ tự HTTP Method (`GET`, `POST`, `DELETE`).
- Các hàm khai báo trong interface và định nghĩa trong class (Service, Controller, RequestClient,...) đều phải được sắp xếp theo thứ tự bảng chữ cái (Alphabetical order - ABC).
- **Vị trí của các hàm xử lý Background Job (Hangfire Jobs)**:
  - Tất cả các phương thức xử lý Job (Hangfire Jobs - thường bắt đầu với tên `Job...` hoặc có thuộc tính `[Queue]`, `[DisplayName]`) **BẮT BUỘC** luôn được đặt ở vị trí **DƯỚI CÙNG** trong cả khai báo `interface` và định nghĩa `class` của Service.

## 14. Toán tử gán kết hợp Null (Null-coalescing Compound Assignment)

- Ưu tiên sử dụng toán tử gán kết hợp null (`??=`) thay vì sử dụng câu lệnh `if` kiểm tra null để gán giá trị mặc định hoặc khởi tạo đối tượng khi biến nhận giá trị null.
  - _Ví dụ tốt_: `site ??= await SiteService.Create(...);`
  - _Ví dụ không tốt_:
    ```csharp
    if (site == null)
    {
        site = await SiteService.Create(...);
    }
    ```
- Đồng thời, ưu tiên sử dụng biểu thức gán kết hợp ném ngoại lệ (`?? throw`) để ném Exception trực tiếp thay vì sử dụng câu lệnh `if` kiểm tra null riêng biệt. Nếu biểu thức quá dài, xuống dòng tại toán tử `??` (thụt lề 1 tab so với dòng gán).
  - _Ví dụ tốt (ngắn)_: `var site = await GetSite() ?? throw new Exception("...");`
  - _Ví dụ tốt (dài)_:
    ```csharp
    site = await SiteService.GetFirstOrDefaultByCode(siteCode)
        ?? throw new Exception("Không tìm thấy chi nhánh tương ứng.");
    ```
  - _Ví dụ không tốt_:
    ```csharp
    site = await SiteService.GetFirstOrDefaultByCode(siteCode);
    if (site == null) throw new Exception("Không tìm thấy chi nhánh tương ứng.");
    ```

## 15. Sử dụng thuộc tính Query trong Service

- Trong các lớp Service (kế thừa `BaseService<T>`), khi viết các câu truy vấn LINQ/EF Core cho thực thể chính của Service đó, luôn ưu tiên sử dụng thuộc tính `Query` thay vì truy cập trực tiếp qua `Db.[Entity]`.
  - _Ví dụ tốt_: `Query.Where(x => x.Code == code)`
  - _Ví dụ không tốt_: `Db.Staff.Where(x => x.Code == code)`

## 16. Sử dụng Extension Methods kiểm tra Chuỗi (String Extension Methods)

- Ưu tiên sử dụng các extension methods `.IsNullOrEmpty()` và `.IsNotNullOrEmpty()` (từ `TripleSixCore`) thay vì gọi phương thức tĩnh `string.IsNullOrEmpty(...)` hay `!string.IsNullOrEmpty(...)`.
  - _Ví dụ tốt_: `if (connectionString.IsNotNullOrEmpty())` hoặc `if (code.IsNullOrEmpty())`
  - _Ví dụ không tốt_: `if (!string.IsNullOrEmpty(connectionString))` hoặc `if (string.IsNullOrEmpty(code))`

## 17. Cấu trúc Thư mục Dự án Unit Test (Unit Test Project Structure)

- Dự án Unit Test phải được đặt trực tiếp dưới thư mục `Tests/` (ví dụ: `Tests/DMCL.FSM.Tests.csproj`), tuyệt đối không tạo thêm thư mục trùng tên lồng nhau (ví dụ: `Tests/DMCL.FSM.Tests/DMCL.FSM.Tests.csproj`).

## 18. Cấu trúc lớp Unit Test (Unit Test Class Structure)

- Mọi lớp kiểm thử (Test Class) nên được chia làm các `partial class`:
  - `partial class` thứ nhất: Chứa các phương thức khởi tạo mock/setup (như `CreateAutoSubstitute`), cấu hình DI container, và các lớp giả lập phụ trợ (như `TestDbContext`).
  - `partial class` thứ hai (hoặc các file partial tiếp theo): Chứa các phương thức kiểm thử (`[Fact]`, `[Theory]`).
- **Bỏ qua Unit Test cho các phương thức cơ sở từ StrongCodeService**:
  - Các hàm dùng chung được định nghĩa sẵn trong `StrongCodeService` (như `GetFirstByCode`, `GetFirstOrDefaultByCode`) không cần viết Unit Test lặp lại cho từng Service kế thừa, trừ trường hợp lớp Service con override hoặc tùy biến lại logic.

## 19. Quy định về Exception, Querying & Controller DbContext

- **Sử dụng Exception thay cho AppException**: Trong các Application Service, ưu tiên ném `Exception` tiêu chuẩn cho các kiểm tra điều kiện nghiệp vụ hoặc validation thay vì `AppException`.
- **Truy vấn bản ghi bắt buộc với `FirstAsync`**: Sử dụng `FirstAsync(predicate)` trực tiếp khi tìm kiếm bản ghi bắt buộc phải tồn tại thay cho mẫu `FirstOrDefaultAsync(...) ?? throw ...` để mã nguồn ngắn gọn và tự động ném exception thích hợp khi không tìm thấy.
- **Sử dụng DbContext trong MigrationController**: Trong `MigrationController` (hoặc các controller chuyên xử lý migration/seed/đồng bộ dữ liệu hệ thống), ưu tiên tiêm và sử dụng trực tiếp `IApplicationDbContext Db { get; set; }` để thao tác trực tiếp với Database Context thay vì tiêm và gọi thông qua lớp Service (`IService<T>`).
- **Tối ưu hóa truy vấn EF Core - Hạn chế Include dư thừa**:
  - Không đính kèm `.Include(...)` đối với các thuộc tính liên kết (Navigation Properties) nếu:
    1. Thuộc tính liên kết đó chỉ được sử dụng trong mệnh đề điều kiện `.Where(...)` (EF Core sẽ tự động dịch thành câu lệnh `JOIN` tối ưu dưới SQL).
    2. Hệ thống đã bật Lazy Loading Proxies và mã nguồn C# chỉ truy cập trực tiếp các trường khóa ngoại Foreign Key ID (`ToStateId`, `SiteId`,...) mà không truy cập sâu vào navigation object.

## 20. Quy chuẩn Thiết kế DTOs, Types & Service Methods (DTO & Service Design Standards)

- **Ràng buộc Kế thừa BaseDto trên DTO**:
  - Mọi lớp DTO trong toàn bộ hệ thống (Admin DTOs, Integrate DTOs, Service DTOs, Client Request DTOs...) đều **bắt buộc phải kế thừa ít nhất lớp `BaseDto`** (hoặc các lớp DTO cơ sở kế thừa `BaseDto` như `BaseDataAdminDto<T>`, `BaseInputDto<T>`, `BaseFilterAdminDto<T>`).
- **Phân biệt DTO và Domain Types**:
  - Các lớp DTO (`[Entity]DataAdminDto`, `[Entity]FilterAdminDto`, `[Entity]UpdateAdminDto`...) phục vụ giao tiếp API/Client.
  - Các lớp chứa tùy chọn/tham số điều khiển cho Service (ví dụ: `TransitionTicketOption`) không phải là DTO giao tiếp API thì đặt tại lớp **Types** trong Domain (`DMCL.FSM.Domain.Types`) và **không** đính kèm suffix `Dto`.
- **Truyền DTO vào Service Method**:
  - Khi phương thức Service xử lý nghiệp vụ tương ứng với một DTO đầu vào (ví dụ `UpdateResult`), hãy khai báo nhận trực tiếp object DTO (`Task UpdateResult(Guid ticketId, TicketResultUpdateDto input);`) thay vì giải nén thành các tham số rời rạc.
- **Không hardcode kiểu DTO cụ thể trong Service Method**:
  - Các phương thức tra cứu / lấy dữ liệu trong Service (ví dụ: `GetFirstById`, `GetListByCase`, `GetListNextState`...) **không được giới hạn cứng một kiểu DTO cố định** (như `WorkflowStateDataAppDto`).
  - Bắt buộc phải cung cấp cặp phương thức (hoặc overload generic `<TResult>`):
    1. Phương thức non-generic trả về kiểu Entity mặc định.
    2. Phương thức generic `<TResult>` trả về `TResult` / `List<TResult>` (với `where TResult : class`) để phía Controller / API Caller tự do lựa chọn DTO phù hợp với từng phân hệ (Web Admin, App Client,...).
- **Quy tắc Đặt tên Class DTO theo Phân hệ (DTO Suffix Conventions)**:
  - **Phân hệ Web Admin (`Dto/Admins/`)**: Luôn kết thúc bằng hậu tố `AdminDto` (Ví dụ: `[Entity]DataAdminDto`, `[Entity]DetailAdminDto`, `[Entity]CreateAdminDto`, `[Entity]UpdateAdminDto`, `[Entity]FilterAdminDto`).
  - **Phân hệ App Client (`Dto/Apps/`)**: Luôn kết thúc bằng hậu tố `AppDto` (Ví dụ: `[Entity]DataAppDto`, `[Entity]DetailAppDto`, `[Entity]CreateAppDto`, `[Entity]FilterListAppDto`, `[Entity]FilterDetailAppDto`).
  - **Phân hệ Tích hợp (`Dto/Integrates/`)**: Luôn kết thúc bằng hậu tố `IntegrateDto` (Ví dụ: `CaseUpsertIntegrateDto`, `SyncDeliveryTicketIntegrateDto`).
  - **Phân hệ RequestClient (`Dto/RequestClients/[Source]/`)**: Luôn kết thúc bằng tên nguồn dịch vụ `[Source]Dto` (Ví dụ: `InstallMaterialDataDienMayDto`, `InstallMaterialInputDienMayDto`, `ProvinceMasterDataDto`).
  - **Phân hệ Headers (`Dto/Apps/Headers/`)**: Luôn kết thúc bằng hậu tố `HeaderAppDto` (Ví dụ: `LocationHeaderAppDto`).
  - **Phân hệ Common (`Dto/Commons/`)**: Chỉ kết thúc bằng `Dto` (Ví dụ: `SettingDataDto`).
- **Tổ chức và Đóng gói Header DTOs (`Dto/Apps/Headers/`)**:
  - Các tham số vị trí tọa độ (`Lat`, `Lng`) hoặc tham số dùng chung truyền qua HTTP Header không lặp lại trong Request Body mà được đóng gói thành các DTO chuyên biệt đặt tại thư mục `Src/Application/Dto/Apps/Headers/` (ví dụ: `LocationHeaderAppDto.cs`).
  - Phía Controller tiếp nhận các Header DTOs bằng thuộc tính `[FromHeader]` (ví dụ: `[FromHeader] LocationHeaderAppDto location`).
- **Tổ chức File và Namespace DTO**:
  - Các DTO của một phân hệ/thực thể được đặt trong thư mục con tương ứng với tên thư mục trùng khớp chính xác tên của Entity (ví dụ: `Src/Application/Dto/Admins/Form/FormCreateAdminDto.cs`, `Src/Application/Dto/Admins/Feedback/FeedbackCreateAdminDto.cs` với tên thư mục ở dạng số ít theo tên Entity như `Form`, `Feedback`, `RatingCriteria`...). Mỗi class DTO nằm trên 1 file riêng biệt trùng tên với class.
  - Namespace trong các file DTO ở thư mục con vẫn duy trì namespace gốc (ví dụ: `namespace DMCL.FSM.Application.Dto.Admins`), tránh tạo thêm namespace lồng nhau.
  - Cảnh báo analyzer `IDE0130` (_Namespace does not match folder structure_) đối với thư mục DTO được gỡ bỏ (suppress) trong `.editorconfig` và `GlobalSuppressions.cs`.
- **Ràng buộc Validation trên DTO**:
  - Các thuộc tính validation như `[Required]`, `[MustPhone]`, `[MustNoSpace]`... **BẮT BUỘC** phải luôn được đặt **bên trên** các thuộc tính mô tả `[DisplayName("...")]` và `[Comment("...")]`.
  - Không sử dụng khởi tạo mặc định `= string.Empty;` cho các thuộc tính kiểu `string`.
  - Đối với thuộc tính danh sách `List<T>` có gán `[Required]`: **Không sử dụng khởi tạo mặc định `= [];`** để tránh lỗi ép kiểu của Swashbuckle Swagger Generator khi sinh tài liệu OpenAPI/Swagger.
  - Sử dụng ngắn gọn `[Required]` thay vì truyền thêm `ErrorMessage = "..."`.
  - Đối với DTO Cập nhật (`UpdateAdminDto`): **Tuyệt đối không sử dụng `[Required]`** trên các thuộc tính và để các thuộc tính ở dạng nullable (`string?`, `bool?`...) vì hệ thống Core chỉ cập nhật những trường được truyền lên (partial update).
  - Thuộc tính mã (Code): Sử dụng thêm validation attribute `[MustNoSpace]` trên DTO Create và Update.
  - Thuộc tính số điện thoại (Phone): Sử dụng thêm validation attribute `[MustPhone]` trên DTO Create và Update.
- **Sử dụng các Extension Methods hỗ trợ LINQ Query (`WhereIf`, `WhereOrs`, `WhereIfOrs`, `WhereIfElse`)**:
  - Ưu tiên sử dụng nạp chuỗi các phương thức mở rộng LINQ của TripleSixCore để giữ câu truy vấn gọn gàng, súc tích thay vì viết các câu lệnh `if (...) query = query.Where(...)` rời rạc:
    - `.WhereIf(condition, predicate)`: Áp dụng điều kiện lọc khi `condition == true`.
    - `.WhereOrs(predicates...)`: Áp dụng tập hợp các điều kiện kết hợp nhóm OR (`||`).
    - `.WhereIfOrs(condition, predicates...)`: Áp dụng tập hợp các điều kiện kết hợp nhóm OR khi `condition == true`.
    - `.WhereIfElse(condition, truePredicate, falsePredicate)`: Áp dụng `truePredicate` nếu `condition == true`, ngược lại áp dụng `falsePredicate` nếu `condition == false`.
    ```csharp
    var query = Db.Ticket
        .Where(x => x.Staff.Code == staffCode)
        .WhereIf(input.StartAt.HasValue, x => x.CreateAt >= input.StartAt!.Value)
        .WhereIfElse(isClosed == true, x => x.CloseAt != null, x => x.CloseAt == null)
        .WhereIfOrs(Keyword.IsNotNullOrEmpty(),
            x => EF.Functions.Like(x.Code, $"%{Keyword}%"),
            x => EF.Functions.Like(x.Name, $"%{Keyword}%"))
        .OrderBy(x => x.CreateAt);
    ```
- **Thuộc tính liên kết trong Data DTO (`[Entity]DataAdminDto`)**:
  - Khi Entity có mối quan hệ liên kết với các Entity khác (như `Site`, `Customer`, `Workflow`, `State`, `Staff`), trong Data DTO **tuyệt đối không trải phẳng** các trường ID/Tên dạng `SiteId`, `SiteName`, `CustomerId`, `CustomerName`.
  - Phải thay thế bằng thuộc tính Data DTO của thực thể liên kết tương ứng (ví dụ: `public SiteDataAdminDto? Site { get; set; }`, `public CustomerDataAdminDto? Customer { get; set; }`, `public WorkflowDataAdminDto? Workflow { get; set; }`).
- **Tùy biến Mapped Fields trong Data DTO qua `FromEntity`**:
  - Khi Data DTO cần tùy biến map các trường dữ liệu qua tập hợp/thực thể liên kết (ví dụ: trích xuất danh sách `List<TicketDataAdminDto>` từ tập hợp `StaffLocationTickets`), thực hiện override phương thức `FromEntity(IServiceProvider serviceProvider, TEntity source)` trực tiếp trong Data DTO:

    ```csharp
    public override async Task FromEntity(IServiceProvider serviceProvider, StaffLocationLog source)
    {
        await base.FromEntity(serviceProvider, source);

        var mapper = serviceProvider.GetRequiredService<IMapper>();
        Tickets = mapper.MapData<List<TicketDataAdminDto>>(source.StaffLocationTickets.Select(x => x.Ticket));
    }
    ```

- **Tùy biến Mapped Fields trong Input DTO qua `ToEntity`**:
  - Khi Input DTO (`BaseInputDto<TEntity>`, `BaseCreateAdminDto<TEntity>`, `BaseUpdateAdminDto<TEntity>`) cần tùy biến chuyển đổi từ DTO thành Entity (ví dụ: mã hóa thông tin, xử lý bổ sung các thuộc tính trước khi lưu), thực hiện override phương thức `ToEntity(IServiceProvider serviceProvider, TEntity? entity = null)` trực tiếp trong Input DTO:

    ```csharp
    public override async Task<Customer> ToEntity(IServiceProvider serviceProvider, Customer? entity = null)
    {
        var result = await base.ToEntity(serviceProvider, entity);

        // Tùy biến thêm dữ liệu vào entity
        return result;
    }
    ```

- **Quy định Đặt tên trong `[SwaggerTag]` và `[DisplayName]`**:
  - Sử dụng tiếng Việt ngắn gọn làm mô tả cho Controller / DTO.
  - **Tuyệt đối không đính kèm tên tiếng Anh của Entity trong dấu ngoặc đơn** đằng sau (ví dụ: viết `[SwaggerTag("Quản lý sự vụ")]` thay vì `[SwaggerTag("Quản lý sự vụ (Case)")]`).
- **Ưu tiên Lọc theo Mã (`Code`) trong Filter DTO (`[Entity]FilterAdminDto`)**:
  - Đối với các tham số lọc theo các đối tượng liên quan (như `Site`, `Staff`, `State`), **ưu tiên khai báo và tiếp nhận lọc theo Mã dạng chuỗi** (ví dụ: `SiteCode`, `StaffCode`, `StateCode`) thay vì dùng GUID Primary Key (`SiteId`, `StaffId`, `StateId`) để phục vụ tiện lợi cho các dịch vụ bên ngoài tích hợp API.
  - Trong phương thức `ToQueryable(...)`, thực hiện lọc qua navigation property: `result.Where(x => x.Site.Code == SiteCode)`.
- **Quy định Đặt tên URL Endpoint Route**:
  - Các động từ HTTP (`[HttpPost]`, `[HttpPut]`, `[HttpDelete]`) đã thể hiện rõ ngữ nghĩa (Post: thêm/ghi nhận, Put: sửa, Delete: xóa).
  - **Tuyệt đối không đính kèm các tiền tố dư thừa như `Add`, `Remove`, `Update` vào đường dẫn URL Route** (Ví dụ: dùng `[HttpPost("{id}/Result")]` thay vì `[HttpPost("{id}/UpdateResult")]`, dùng `[HttpPost("{id}/InstallMaterial")]` và `[HttpDelete("{id}/InstallMaterial/{materialId}")]` thay cho `AddInstallMaterial` / `RemoveInstallMaterial`). Tên của phương thức C# vẫn giữ tên mô tả rõ ràng (`AddInstallMaterial`, `RemoveInstallMaterial`, `UpdateResult`).
- **Khởi tạo DbContext trong Unit Test**:
  - Trong lớp giả lập `TestDbContext`, thuộc tính `Identity` luôn được khởi tạo (`Identity = new IdentityContext(null, Substitute.For<IConfiguration>())`) để tránh `NullReferenceException` khi thực thi `SaveChangesAsync()` auto-auditing.

## 21. Cấu trúc Block-scoped Namespace

- Luôn sử dụng block-scoped namespace (sử dụng cặp ngoặc nhọn `{}`) thay vì file-scoped namespace (sử dụng dấu chấm phẩy `;`) cho tất cả các tệp mã nguồn C# (DTOs, Services, Controllers, Entities, v.v.).
  - _Ví dụ tốt_:
    ```csharp
    namespace DMCL.FSM.Application.Dto.Integrates
    {
        public class SyncCustomerIntegrateDto
        {
        }
    }
    ```
  - _Ví dụ không tốt_:
    ```csharp
    namespace DMCL.FSM.Application.Dto.Integrates;
    ```

## 22. Lược bỏ Comment thừa trong DTOs

- Tránh viết các khối comment XML summary (`/// <summary>`) cho các thuộc tính của DTO nếu tên thuộc tính đã thể hiện đầy đủ ý nghĩa (self-explanatory), nhằm giữ tệp tin DTO ngắn gọn và tập trung.

## 23. Comment cho DTO Tích hợp (Integrate DTOs)

- **Bắt buộc Viết Comment cho DTO Tích hợp**:
  - Mọi DTO thuộc thư mục `Src/Application/Dto/Integrates/` phải được bổ sung comment mô tả chi tiết ý nghĩa tiếng Việt cho từng thuộc tính/trường dữ liệu nhằm phục vụ việc làm rõ nghiệp vụ khi tích hợp giữa các hệ thống.

## 24. Quản lý Global Usings (\_GlobalUsings.cs)

- Tất cả các namespace dùng chung (như DTOs `Admins`, `Apps`, `Integrates`, `RequestClients`, `Services`, các lớp `Services`, `Helpers`, v.v.) phải được khai báo tập trung tại tệp `_GlobalUsings.cs` của từng dự án (`Src/Application/_GlobalUsings.cs`, `Src/WebApi/_GlobalUsings.cs`, `Src/Infrastructure/_GlobalUsings.cs`, `Tests/_GlobalUsings.cs`).
- Tuyệt đối không thêm các câu lệnh `using` cục bộ trong từng file mã nguồn đối với các namespace này để giữ code sạch và tránh lặp lại.

## 25. Lưu trữ Số điện thoại chuẩn E.164 (E.164 Phone Storage)

- Tất cả các thuộc tính lưu trữ số điện thoại (`Phone`, `ReceiverPhone`,...) dưới Database **BẮT BUỘC** phải luôn được chuẩn hóa và lưu trữ ở dạng chuẩn E.164 quốc tế (`+84...`) thông qua `PhoneHelper.NationalPhoneToE164Phone(...)`.
- Khi thực thi các câu truy vấn LINQ/EF Core theo số điện thoại (ví dụ `GetFirstOrDefaultByPhone`), số điện thoại đầu vào phải được chuyển đổi về dạng E.164 trước khi truy vấn trực tiếp dưới Database (`x.Phone == e164Phone`).

## 26. Sử dụng StartsWith(char) cho Kí tự Đơn (Single Character StartsWith)

- Khi kiểm tra chuỗi bắt đầu bằng một kí tự đơn, luôn ưu tiên sử dụng overload nhận tham số `char` (`'...'`) thay vì `string` (`"..."`) để tối ưu hiệu năng và tuân thủ quy chuẩn C#.
  - _Ví dụ tốt_: `if (trimmed.StartsWith('+'))` hoặc `if (trimmed.StartsWith('0'))`
  - _Ví dụ không tốt_: `if (trimmed.StartsWith("+"))` hoặc `if (trimmed.StartsWith("0"))`
  - _Lưu ý_: Đối với chuỗi từ 2 kí tự trở lên (như `"84"` hay `"+84"`), tiếp tục sử dụng dạng chuỗi `"..."`.

## 27. Quy định không truy cập IdentityContext trong Service Layer

- Các lớp Service (trong `Src/Application/Services/`) **tuyệt đối không được tiêm hoặc truy cập trực tiếp `IdentityContext`**.
- Chỉ có API Controllers (trong `Src/WebApi/Controllers/`) mới được phép truy cập `IdentityContext`.
- Khi Service cần thông tin căn cước/tài khoản (như `StaffCode`, `UserId`, `SiteCode`...), phương thức Service phải nhận thông tin này thông qua các tham số (hoặc DTO) do API Controller truyền vào.

## 28. Quy định về việc tạo Unit Test

- **Không tự động sinh/viết thêm Unit Test mới** trong quá trình phát triển hay điều chỉnh tính năng trừ khi người dùng có yêu cầu cụ thể, nhằm tránh lãng phí token và tài nguyên.
- Chỉ cập nhật/sửa các Unit Test đã có sẵn nếu signature hoặc logic phương thức thay đổi khiến test bị lỗi compilation/failure.

## 29. Quy định gán [Transactional] cho API (Transactional Attribute Rule)

- Tất cả các phương thức API Endpoint làm thay đổi dữ liệu hoặc sử dụng các động từ HTTP `[HttpPost]`, `[HttpPut]`, `[HttpDelete]` phía Controller **BẮT BUỘC** phải được gán thuộc tính `[Transactional]` để tự động mở Database Transaction và lưu/rollback dữ liệu một cách an toàn.

## 30. Quy định Lưu trữ Tọa độ và Dữ liệu Không gian khi sử dụng GIST Index

- **Điều kiện áp dụng**: Quy định này áp dụng khi Thực thể (Domain Entity) có lưu trữ tọa độ địa lý dạng `Lat`/`Lng` và dự án có nhu cầu truy vấn dữ liệu không gian sử dụng chỉ mục **GIST Index** (PostGIS / Spatial Query):
  - **Khai báo thuộc tính Point**: Bổ sung thuộc tính vị trí không gian `LocationPoint` (hoặc `LastLocationPoint` đối với Staff) kiểu `Point?` thuộc `NetTopologySuite.Geometries` đi kèm Data Annotation `[Column(TypeName = "geography(Point, 4326)")]`.
  - **Cấu hình GIST Index**: Khai báo chỉ mục GIST index cho thuộc tính `LocationPoint` trong phương thức `Configure(EntityTypeBuilder<T> builder)` bằng cú pháp: `builder.HasIndex(x => x.LocationPoint).HasMethod("gist");`.
  - **Khởi tạo/Cập nhật trong Service Layer**: Trong các phương thức Service (thêm mới/cập nhật), luôn đồng bộ khởi tạo `LocationPoint` theo thứ tự `(lng, lat)` với SRID 4326 khi `Lat` và `Lng` hợp lệ: `new Point(lng, lat) { SRID = 4326 }` (Lưu ý: tham số đầu tiên của `Point` là Kinh độ `Lng`, tham số thứ hai là Vĩ độ `Lat`).

## 31. Quy định sử dụng Db.Set<T>().Add(...) (Synchronous Add in EF Core)

- Khi thêm một thực thể (entity) mới vào EF Core DbContext ChangeTracker, luôn ưu tiên sử dụng phương thức đồng bộ `Db.[Entity].Add(entity)` (hoặc `Db.Set<T>().Add(entity)`) thay vì `AddAsync(...)`.
- _Lý do_: Phương thức `Add` trong EF Core chỉ đơn thuần gắn trạng thái `EntityState.Added` lên đối tượng trên RAM mà không thực hiện kết nối hay I/O call xuống Database. Việc thực thi câu lệnh SQL chỉ diễn ra khi gọi `await Db.SaveChangesAsync()`. Phương thức `AddAsync` chỉ thực sự cần thiết khi sử dụng các Value Generator đặc biệt có truy vấn DB ngay tại thời điểm Add (ví dụ `HiLoValueGenerator`).

## 32. Quy định Trả về đối với Phương thức Service Thao tác (Add/Delete Return Types)

- Đối với các phương thức Service thực hiện thao tác thêm mới hoặc xóa dữ liệu mà phía API Controller không cần trả về payload dữ liệu chi tiết (ví dụ: các API thêm đính kèm, xóa đính kèm, cập nhật flag...):
  - Khai báo kiểu trả về của Service Method là **`Task`** (void async) thay vì trả về `Task<TEntity>` hay `Task<TDto>`.
  - Phía API Controller tương ứng sẽ gọi `await Service.Method(...)` và trả về kết quả chuẩn **`SuccessResult()`**.

## 33. Sắp xếp Enum Value theo Giá trị (Enum Values Sorting)

- **Tên Enum luôn ở dạng số nhiều (Plural Enum Naming)**: Tất cả tên lớp Enum (Class/Enum Name) và tệp tin mã nguồn tương ứng thuộc thư mục `Domain/Constants/` **BẮT BUỘC** phải luôn ở dạng số nhiều (kết thúc bằng `s` hoặc `es`, ví dụ: `FormTargetTypes`, `RatingCriteriaValueTypes`, `Genders`, `StatusFilters`...) thay vì dùng từ ở dạng số ít (`FormTargetType`, `RatingCriteriaValueType`).
- Tất cả các giá trị của Enum (Enum Values) **BẮT BUỘC** phải được sắp xếp tăng dần theo giá trị số (numerical value) của chúng.

## 34. Thứ tự Fields trong DTO phải theo thứ tự Entity (DTO Field Ordering)

- Khi khai báo các thuộc tính trong Data DTO (`[Entity]DataAdminDto`, `[Entity]DataAppDto`,...), **bắt buộc** phải sắp xếp các fields theo **đúng thứ tự** như chúng được khai báo trong Entity tương ứng.
- Khi thêm field mới vào DTO, phải đặt đúng vị trí tương ứng trong Entity, không được thêm vào cuối danh sách.

## 35. Quy định Exception trong Hangfire Job

- Trong các Hangfire job, **chỉ throw `JobException` đối với các kiểm tra điều kiện nghiệp vụ / validation rõ ràng** (nơi mà việc thử lại không có ý nghĩa). `JobException` giúp ngắt job và không tự động Retry.
- **Tuyệt đối không bọc toàn bộ body của Job trong khối `try-catch` chỉ để ném `JobException`**, vì làm như vậy sẽ nuốt các ngoại lệ hệ thống/kết nối DB tạm thời và làm mất đi cơ chế tự động Retry mặc định của Hangfire. Để các ngoại lệ không mong muốn được ném tự nhiên để Hangfire tự động retry.

## 36. Quy định Múi giờ trong Cron Schedule cho Hangfire Job (Timezone for Hangfire Cron)

- Server ứng dụng vận hành mặc định ở múi giờ UTC (+0).
- Khi người dùng đề cập đến mốc thời gian (ví dụ: 12h đêm / 00:00 giờ Việt Nam UTC+7), **bắt buộc phải quy đổi về múi giờ UTC (+0)** trước khi cấu hình biểu thức Cron (ví dụ: 00:00 VN = 17:00 UTC hôm trước -> `"0 17 * * *"`).

## 37. Quy định Tên Hangfire Queue trong [Queue("...")] Attribute (Hangfire Queue Attribute Rule)

- Các dự án TripleSixCore sử dụng Hangfire Jobs luôn khai báo tên queue tương ứng (ví dụ: `"fsm"`, `"delivery"`,...) tại mục `Hangfire:Queues` trong `appsettings.json`.
- Mọi thuộc tính `[Queue("...")]` đặt phía trên phương thức Hangfire Job (`Job...`) **BẮT BUỘC** phải chỉ định đúng tên queue mà dự án khai báo trong `appsettings.json` (ví dụ: `[Queue("fsm")]`), **tuyệt đối không** sử dụng `[Queue("default")]`.

## 38. Quy định Tham số Thời gian & Múi giờ UTC trong Service & Hangfire Job (UTC Time & Nullable Date Parameter Rule)

- **Sử dụng `DateTime.UtcNow`**: Server ứng dụng vận hành mặc định ở múi giờ UTC (+0). Mọi thao tác lấy mốc thời gian hiện tại **BẮT BUỘC** phải sử dụng `DateTime.UtcNow` để tối ưu hiệu năng và giữ code sạch, tuyệt đối không dùng `DateTime.Now` hay chuyển đổi phức tạp.
- **Khai báo Tham số `date` Nullable**: Các phương thức Service hoặc Hangfire Job tiếp nhận tham số thời gian **BẮT BUỘC** khai báo ở dạng nullable có giá trị mặc định `null` (`DateTime? date = null`). Phía bên trong phương thức, sử dụng toán tử gán null kết hợp súc tích:
  ```csharp
  var targetDate = date ?? DateTime.UtcNow;
  ```

## 39. Quy định Tối ưu hóa EF Core với Eager Loading (.Include) và AsNoTracking (EF Core Query Optimization Rule)

- **Sử dụng Eager Loading (`.Include(...)`) thay cho Lazy Loading Proxies**:
  - Khi cần lấy dữ liệu thuộc tính của thực thể liên kết (Navigation Property) để map ra DTO hoặc trả về response, **BẮT BUỘC** phải sử dụng `.Include(...)` để EF Core phát sinh duy nhất 1 câu SQL `JOIN` tối ưu dưới CSDL.
  - Tuyệt đối không phụ thuộc vào `UseLazyLoadingProxies` khi truy vấn danh sách để tránh gây ra sự cố nghiêm trọng **N+1 Query Problem** (thực thi 1 + N câu SQL riêng biệt xuống DB).
  - _Ngoại lệ_: Nếu thuộc tính liên kết chỉ dùng trong mệnh đề lọc `.Where(...)` hoặc chỉ lấy trường khóa ngoại ID (`SiteId`, `StateId`), không đính kèm `.Include(...)` dư thừa vì EF Core đã tự gộp JOIN dưới SQL.
- **Sử dụng `AsNoTracking()` cho Truy vấn Đọc (Read-Only Queries)**:
  - Tất cả các câu truy vấn tra cứu danh sách, xuất báo cáo, đọc dữ liệu trả về cho API Read-Only mà không thực hiện sửa đổi và gọi `SaveChangesAsync()` **BẮT BUỘC** phải áp dụng `.AsNoTracking()` (hoặc sử dụng thuộc tính `Query` của `BaseService` đã được cấu hình tối ưu).
  - `.AsNoTracking()` giúp bỏ qua bộ nhớ Change Tracker trên RAM, giảm chi phí CPU/bộ nhớ và tăng tốc độ đọc dữ liệu từ 20% - 40%.

## 40. Quy định Cấu hình Cột JSONB trong Entity (JSONB Column Mapping)

- Các thuộc tính lưu trữ dữ liệu dạng JSONB dưới Database PostgreSQL **BẮT BUỘC** phải được khai báo kiểu dữ liệu trong C# là **`JsonElement`** (từ namespace `System.Text.Json`).
- Đính kèm Data Annotation **`[Column(TypeName = "jsonb")]`** trực tiếp phía trên thuộc tính đó trong Entity.
