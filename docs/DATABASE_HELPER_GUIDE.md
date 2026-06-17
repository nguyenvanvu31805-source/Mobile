# HƯỚNG DẪN CHI TIẾT: DATABASE HELPER CLASS

## 📋 TỔNG QUAN

DatabaseHelper là class core của hệ thống QLNS, kế thừa từ SQLiteOpenHelper để quản lý toàn bộ database operations. Class này chứa tất cả các phương thức CRUD, business logic, và data management cho 8 bảng chính của hệ thống.

## 🏗️ KIẾN TRÚC DATABASE

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/database/
└── DatabaseHelper.java                     # Core database management class
```

### Database Schema:
```
Database: qlns.db (Version 18)
├── ChucVu (Positions)                      # Quản lý chức vụ
├── PhongBan (Departments)                  # Quản lý phòng ban  
├── NhanVien (Employees)                    # Quản lý nhân viên
├── HopDongLaoDong (Contracts)              # Quản lý hợp đồng lao động
├── ChamCong (Attendance)                   # Quản lý chấm công
├── NghiPhep (Leave Requests)               # Quản lý nghỉ phép
├── Luong (Salary)                          # Quản lý lương
└── TaiKhoan (Accounts)                     # Quản lý tài khoản
```

## 📊 NGHIỆP VỤ DATABASE

### 1. Database Management:
- **Schema creation**: Tạo và quản lý cấu trúc 8 bảng
- **Version control**: Quản lý phiên bản database với migration
- **Sample data**: Tự động tạo dữ liệu mẫu khi khởi tạo
- **Data integrity**: Đảm bảo tính toàn vẹn dữ liệu với foreign keys

### 2. Business Operations:
- **Employee lifecycle**: Từ đăng ký, quản lý đến nghỉ việc
- **Attendance tracking**: Chấm công vào/ra với tính toán giờ làm
- **Leave management**: Quy trình xin nghỉ phép và duyệt
- **Salary calculation**: Tính lương tự động dựa trên chấm công

### 3. Security & Access Control:
- **Authentication**: Xác thực đăng nhập với username/password
- **Role-based access**: Phân quyền theo vai trò (Admin/HR/Manager/Employee)
- **Data protection**: Parameterized queries để tránh SQL injection

---

## 📱 CHI TIẾT CLASS IMPLEMENTATION

## 1️⃣ CLASS DECLARATION & CONSTANTS

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/database/DatabaseHelper.java`

### Mục đích:
Core database management class với tất cả constants, constructor và lifecycle methods.

### Chi tiết code:

#### Package và Import declarations:
```java
package com.example.btl_mobile_qlns.database;                             // Khai báo package chứa database classes

import android.content.ContentValues;                                      // Import ContentValues để insert/update data
import android.content.Context;                                            // Import Context để truy cập app context
import android.database.Cursor;                                            // Import Cursor để xử lý query results
import android.database.sqlite.SQLiteDatabase;                             // Import SQLiteDatabase để thao tác database
import android.database.sqlite.SQLiteOpenHelper;                           // Import SQLiteOpenHelper làm base class
```

#### Class declaration và constants:
```java
public class DatabaseHelper extends SQLiteOpenHelper {                     // Khai báo class kế thừa SQLiteOpenHelper
    
    private static final String DATABASE_NAME = "qlns.db";                 // Tên file database
    private static final int DATABASE_VERSION = 18;                        // Phiên bản database hiện tại
    
    // Table name constants - Khai báo tên các bảng
    public static final String TABLE_CHUC_VU = "ChucVu";                   // Bảng chức vụ
    public static final String TABLE_PHONG_BAN = "PhongBan";               // Bảng phòng ban
    public static final String TABLE_NHAN_VIEN = "NhanVien";               // Bảng nhân viên
    public static final String TABLE_HOP_DONG = "HopDongLaoDong";          // Bảng hợp đồng lao động
    public static final String TABLE_CHAM_CONG = "ChamCong";               // Bảng chấm công
    public static final String TABLE_NGHI_PHEP = "NghiPhep";               // Bảng nghỉ phép
    public static final String TABLE_LUONG = "Luong";                      // Bảng lương
    public static final String TABLE_TAI_KHOAN = "TaiKhoan";               // Bảng tài khoản
```

#### Constructor:
```java
    public DatabaseHelper(Context context) {                               // Constructor nhận Context
        super(context, DATABASE_NAME, null, DATABASE_VERSION);            // Gọi constructor của SQLiteOpenHelper
    }
```

---

## 2️⃣ DATABASE LIFECYCLE METHODS

### Method onCreate:
```java
    @Override
    public void onCreate(SQLiteDatabase db) {                              // Method được gọi khi tạo database lần đầu
        // Tạo bảng ChucVu (Positions)
        String createChucVuTable = "CREATE TABLE " + TABLE_CHUC_VU + " (" + // SQL tạo bảng chức vụ
                "MaChucVu TEXT PRIMARY KEY, " +                            // Mã chức vụ - Primary Key
                "TenChucVu TEXT NOT NULL, " +                              // Tên chức vụ - Bắt buộc
                "MucLuongCoBan REAL, " +                                   // Mức lương cơ bản - Số thực
                "TrangThai INTEGER DEFAULT 1)";                            // Trạng thái - Mặc định 1 (active)
        db.execSQL(createChucVuTable);                                     // Thực thi SQL tạo bảng
        
        // Tạo bảng PhongBan (Departments)
        String createPhongBanTable = "CREATE TABLE " + TABLE_PHONG_BAN + " (" + // SQL tạo bảng phòng ban
                "MaPhongBan TEXT PRIMARY KEY, " +                          // Mã phòng ban - Primary Key
                "TenPhongBan TEXT NOT NULL, " +                            // Tên phòng ban - Bắt buộc
                "TruongPhong TEXT, " +                                     // Trưởng phòng - Foreign Key đến NhanVien
                "TrangThai INTEGER DEFAULT 1)";                            // Trạng thái - Mặc định 1 (active)
        db.execSQL(createPhongBanTable);                                   // Thực thi SQL tạo bảng
        
        // Tạo bảng NhanVien (Employees)
        String createNhanVienTable = "CREATE TABLE " + TABLE_NHAN_VIEN + " (" + // SQL tạo bảng nhân viên
                "MaNhanVien TEXT PRIMARY KEY, " +                          // Mã nhân viên - Primary Key
                "HoTen TEXT NOT NULL, " +                                  // Họ tên - Bắt buộc
                "NgaySinh DATE, " +                                        // Ngày sinh - Kiểu DATE
                "GioiTinh TEXT DEFAULT 'Nam', " +                          // Giới tính - Mặc định 'Nam'
                "SoDienThoai TEXT, " +                                     // Số điện thoại
                "Email TEXT, " +                                           // Email
                "NgayVaoLam DATE NOT NULL, " +                             // Ngày vào làm - Bắt buộc
                "MaPhongBan TEXT, " +                                      // Mã phòng ban - Foreign Key
                "MaChucVu TEXT, " +                                        // Mã chức vụ - Foreign Key
                "HinhAnh TEXT, " +                                         // Đường dẫn hình ảnh
                "TrangThaiLamViec TEXT DEFAULT 'Đang làm việc')";          // Trạng thái làm việc - Mặc định 'Đang làm việc'
        db.execSQL(createNhanVienTable);                                   // Thực thi SQL tạo bảng
        
        // Tạo các bảng còn lại với execSQL trực tiếp
        db.execSQL("CREATE TABLE " + TABLE_HOP_DONG + " (MaHopDong TEXT PRIMARY KEY, MaNhanVien TEXT NOT NULL, LoaiHopDong TEXT NOT NULL, NgayBatDau DATE NOT NULL, NgayKetThuc DATE, MucLuong REAL NOT NULL, TrangThai TEXT DEFAULT 'Hiệu lực')"); // Bảng hợp đồng lao động
        db.execSQL("CREATE TABLE " + TABLE_CHAM_CONG + " (MaChamCong INTEGER PRIMARY KEY AUTOINCREMENT, MaNhanVien TEXT NOT NULL, NgayChamCong DATE NOT NULL, GioVao TIME, GioRa TIME, SoGioLam REAL DEFAULT 0, TrangThai TEXT DEFAULT 'Có mặt', GhiChu TEXT, UNIQUE (MaNhanVien, NgayChamCong))"); // Bảng chấm công với UNIQUE constraint
        db.execSQL("CREATE TABLE " + TABLE_NGHI_PHEP + " (MaNghiPhep INTEGER PRIMARY KEY AUTOINCREMENT, MaNhanVien TEXT NOT NULL, NgayBatDau DATE NOT NULL, NgayKetThuc DATE NOT NULL, SoNgayNghi INTEGER NOT NULL, LyDo TEXT, TrangThai TEXT DEFAULT 'Chờ duyệt', NguoiDuyet TEXT)"); // Bảng nghỉ phép
        db.execSQL("CREATE TABLE " + TABLE_LUONG + " (MaLuong INTEGER PRIMARY KEY AUTOINCREMENT, MaNhanVien TEXT NOT NULL, ThangNam TEXT NOT NULL, LuongCoBan REAL NOT NULL, PhuCap REAL DEFAULT 0, SoGioLam REAL DEFAULT 0, TongLuong REAL NOT NULL, TrangThai TEXT DEFAULT 'Chưa thanh toán', NgayTinhLuong DATE, UNIQUE (MaNhanVien, ThangNam))"); // Bảng lương với UNIQUE constraint
        db.execSQL("CREATE TABLE " + TABLE_TAI_KHOAN + " (MaTaiKhoan INTEGER PRIMARY KEY AUTOINCREMENT, MaNhanVien TEXT NOT NULL UNIQUE, TenDangNhap TEXT NOT NULL UNIQUE, MatKhau TEXT NOT NULL, VaiTro TEXT DEFAULT 'Employee', TrangThai INTEGER DEFAULT 1)"); // Bảng tài khoản với UNIQUE constraints
        
        insertSampleData(db);                                              // Gọi method chèn dữ liệu mẫu
    }
```
### Method onUpgrade:
```java
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) { // Method được gọi khi upgrade database version
        if (oldVersion < 2) {                                              // Nếu version cũ < 2
            db.execSQL("ALTER TABLE " + TABLE_NHAN_VIEN + " ADD COLUMN HinhAnh TEXT"); // Thêm cột HinhAnh vào bảng NhanVien
        }
        if (oldVersion < 5) {                                              // Nếu version cũ < 5
            db.execSQL("DELETE FROM " + TABLE_TAI_KHOAN);                  // Xóa tất cả dữ liệu tài khoản cũ
            db.execSQL("DELETE FROM " + TABLE_NHAN_VIEN);                  // Xóa tất cả dữ liệu nhân viên cũ
            db.execSQL("DELETE FROM " + TABLE_PHONG_BAN);                  // Xóa tất cả dữ liệu phòng ban cũ
            db.execSQL("DELETE FROM " + TABLE_CHUC_VU);                    // Xóa tất cả dữ liệu chức vụ cũ
            insertSampleData(db);                                          // Chèn lại dữ liệu mẫu mới
        }
        if (oldVersion < 8) {                                              // Nếu version cũ < 8
            // Thêm cột GhiChu nếu chưa có
            try {
                db.execSQL("ALTER TABLE " + TABLE_CHAM_CONG + " ADD COLUMN GhiChu TEXT"); // Thêm cột GhiChu vào bảng ChamCong
            } catch (Exception e) {                                        // Catch exception nếu cột đã tồn tại
                // Cột đã tồn tại
            }
            // Xóa dữ liệu chấm công cũ và thêm dữ liệu mẫu mới
            db.execSQL("DELETE FROM " + TABLE_CHAM_CONG);                  // Xóa dữ liệu chấm công cũ
            insertSampleAttendance(db);                                    // Chèn dữ liệu chấm công mẫu mới
        }
        if (oldVersion < 13) {                                             // Nếu version cũ < 13
            // Tạo đúng 100 dòng chấm công và sửa lỗi xem lương
            db.execSQL("DELETE FROM " + TABLE_CHAM_CONG);                  // Xóa dữ liệu chấm công cũ
            db.execSQL("DELETE FROM " + TABLE_LUONG);                      // Xóa dữ liệu lương cũ
            insertSampleAttendance(db);                                    // Chèn dữ liệu chấm công mẫu mới
        }
        if (oldVersion < 14) {                                             // Nếu version cũ < 14
            // Thêm dữ liệu chấm công cho 3 tháng (March, April, May 2026)
            db.execSQL("DELETE FROM " + TABLE_CHAM_CONG);                  // Xóa dữ liệu chấm công cũ
            db.execSQL("DELETE FROM " + TABLE_LUONG);                      // Xóa dữ liệu lương cũ
            insertSampleAttendance(db);                                    // Chèn dữ liệu chấm công 3 tháng
        }
        if (oldVersion < 15) {                                             // Nếu version cũ < 15
            // Force update dữ liệu chấm công 3 tháng
            db.execSQL("DELETE FROM " + TABLE_CHAM_CONG);                  // Xóa dữ liệu chấm công cũ
            db.execSQL("DELETE FROM " + TABLE_LUONG);                      // Xóa dữ liệu lương cũ
            insertSampleAttendance(db);                                    // Chèn dữ liệu chấm công mẫu mới
        }
        if (oldVersion < 16) {                                             // Nếu version cũ < 16
            // Sửa lỗi logic tính lương - getAttendanceStats
            db.execSQL("DELETE FROM " + TABLE_LUONG);                      // Xóa dữ liệu lương cũ để tính lại
        }
        if (oldVersion < 17) {                                             // Nếu version cũ < 17
            // Dữ liệu tháng 1, 2, 3 và tháng 4 đến ngày 17
            db.execSQL("DELETE FROM " + TABLE_CHAM_CONG);                  // Xóa dữ liệu chấm công cũ
            db.execSQL("DELETE FROM " + TABLE_LUONG);                      // Xóa dữ liệu lương cũ
            insertSampleAttendance(db);                                    // Chèn dữ liệu chấm công mới với format cập nhật
        }
    }
```

---

## 3️⃣ SAMPLE DATA INSERTION METHODS

### Method insertSampleData:
```java
    private void insertSampleData(SQLiteDatabase db) {                     // Method chèn dữ liệu mẫu ban đầu
        // Thêm dữ liệu Chức vụ
        ContentValues cv1 = new ContentValues();                          // Tạo ContentValues cho chức vụ Giám đốc
        cv1.put("MaChucVu", "CV001");                                      // Mã chức vụ
        cv1.put("TenChucVu", "Giám đốc");                                  // Tên chức vụ
        cv1.put("MucLuongCoBan", 15000000);                                // Mức lương cơ bản 15 triệu
        cv1.put("TrangThai", 1);                                           // Trạng thái active
        db.insert(TABLE_CHUC_VU, null, cv1);                              // Insert vào bảng ChucVu
        
        ContentValues cv2 = new ContentValues();                          // Tạo ContentValues cho chức vụ Trưởng phòng
        cv2.put("MaChucVu", "CV002");                                      // Mã chức vụ
        cv2.put("TenChucVu", "Trưởng phòng");                              // Tên chức vụ
        cv2.put("MucLuongCoBan", 8000000);                                 // Mức lương cơ bản 8 triệu
        cv2.put("TrangThai", 1);                                           // Trạng thái active
        db.insert(TABLE_CHUC_VU, null, cv2);                              // Insert vào bảng ChucVu
        
        ContentValues cv3 = new ContentValues();                          // Tạo ContentValues cho chức vụ Nhân viên
        cv3.put("MaChucVu", "CV003");                                      // Mã chức vụ
        cv3.put("TenChucVu", "Nhân viên");                                 // Tên chức vụ
        cv3.put("MucLuongCoBan", 5000000);                                 // Mức lương cơ bản 5 triệu
        cv3.put("TrangThai", 1);                                           // Trạng thái active
        db.insert(TABLE_CHUC_VU, null, cv3);                              // Insert vào bảng ChucVu
        
        ContentValues cv4 = new ContentValues();                          // Tạo ContentValues cho chức vụ Phó phòng
        cv4.put("MaChucVu", "CV004");                                      // Mã chức vụ
        cv4.put("TenChucVu", "Phó phòng");                                 // Tên chức vụ
        cv4.put("MucLuongCoBan", 6500000);                                 // Mức lương cơ bản 6.5 triệu
        cv4.put("TrangThai", 1);                                           // Trạng thái active
        db.insert(TABLE_CHUC_VU, null, cv4);                              // Insert vào bảng ChucVu
        
        ContentValues cv5 = new ContentValues();                          // Tạo ContentValues cho chức vụ Thực tập sinh
        cv5.put("MaChucVu", "CV005");                                      // Mã chức vụ
        cv5.put("TenChucVu", "Thực tập sinh");                             // Tên chức vụ
        cv5.put("MucLuongCoBan", 3000000);                                 // Mức lương cơ bản 3 triệu
        cv5.put("TrangThai", 1);                                           // Trạng thái active
        db.insert(TABLE_CHUC_VU, null, cv5);                              // Insert vào bảng ChucVu
        
        // Thêm dữ liệu Phòng ban
        ContentValues pb1 = new ContentValues();                          // Tạo ContentValues cho Phòng Nhân sự
        pb1.put("MaPhongBan", "PB001");                                    // Mã phòng ban
        pb1.put("TenPhongBan", "Phòng Nhân sự");                           // Tên phòng ban
        pb1.put("TrangThai", 1);                                           // Trạng thái active
        db.insert(TABLE_PHONG_BAN, null, pb1);                            // Insert vào bảng PhongBan
        
        ContentValues pb2 = new ContentValues();                          // Tạo ContentValues cho Phòng Kế toán
        pb2.put("MaPhongBan", "PB002");                                    // Mã phòng ban
        pb2.put("TenPhongBan", "Phòng Kế toán");                           // Tên phòng ban
        pb2.put("TrangThai", 1);                                           // Trạng thái active
        db.insert(TABLE_PHONG_BAN, null, pb2);                            // Insert vào bảng PhongBan
        
        ContentValues pb3 = new ContentValues();                          // Tạo ContentValues cho Phòng Kỹ thuật
        pb3.put("MaPhongBan", "PB003");                                    // Mã phòng ban
        pb3.put("TenPhongBan", "Phòng Kỹ thuật");                          // Tên phòng ban
        pb3.put("TrangThai", 1);                                           // Trạng thái active
        db.insert(TABLE_PHONG_BAN, null, pb3);                            // Insert vào bảng PhongBan
        
        ContentValues pb4 = new ContentValues();                          // Tạo ContentValues cho Phòng Marketing
        pb4.put("MaPhongBan", "PB004");                                    // Mã phòng ban
        pb4.put("TenPhongBan", "Phòng Marketing");                         // Tên phòng ban
        pb4.put("TrangThai", 1);                                           // Trạng thái active
        db.insert(TABLE_PHONG_BAN, null, pb4);                            // Insert vào bảng PhongBan
        
        ContentValues pb5 = new ContentValues();                          // Tạo ContentValues cho Phòng Kinh doanh
        pb5.put("MaPhongBan", "PB005");                                    // Mã phòng ban
        pb5.put("TenPhongBan", "Phòng Kinh doanh");                        // Tên phòng ban
        pb5.put("TrangThai", 1);                                           // Trạng thái active
        db.insert(TABLE_PHONG_BAN, null, pb5);                            // Insert vào bảng PhongBan
        
        // Thêm dữ liệu Nhân viên (10 nhân viên)
        String[][] employees = {                                           // Mảng 2 chiều chứa thông tin 10 nhân viên mẫu
            {"NV002", "Trần Thị Bình", "1990-08-22", "Nữ", "0901234568", "binh@company.com", "2020-02-01", "PB001", "CV002"},
            {"NV003", "Lê Văn Cường", "1988-12-10", "Nam", "0901234569", "cuong@company.com", "2020-03-15", "PB002", "CV002"},
            {"NV004", "Phạm Thị Dung", "1992-03-25", "Nữ", "0901234570", "dung@company.com", "2020-04-01", "PB003", "CV003"},
            {"NV005", "Hoàng Văn Em", "1995-07-18", "Nam", "0901234571", "em@company.com", "2020-05-10", "PB003", "CV003"},
            {"NV006", "Nguyễn Thị Hoa", "1993-11-05", "Nữ", "0901234572", "hoa@company.com", "2021-01-15", "PB004", "CV003"},
            {"NV007", "Đặng Minh Khoa", "1987-09-30", "Nam", "0901234573", "khoa@company.com", "2021-02-20", "PB004", "CV004"},
            {"NV008", "Vũ Thị Lan", "1994-06-12", "Nữ", "0901234574", "lan@company.com", "2021-03-10", "PB005", "CV003"},
            {"NV009", "Bùi Văn Minh", "1991-04-08", "Nam", "0901234575", "minh@company.com", "2021-04-05", "PB005", "CV004"},
            {"NV010", "Lý Thị Nga", "1996-12-20", "Nữ", "0901234576", "nga@company.com", "2022-01-10", "PB001", "CV003"},
            {"NV011", "Trịnh Văn Phong", "1998-02-14", "Nam", "0901234577", "phong@company.com", "2023-06-01", "PB002", "CV005"}
        };
        
        for (String[] emp : employees) {                                   // Lặp qua từng nhân viên trong mảng
            ContentValues nv = new ContentValues();                       // Tạo ContentValues cho mỗi nhân viên
            nv.put("MaNhanVien", emp[0]);                                  // Mã nhân viên
            nv.put("HoTen", emp[1]);                                       // Họ tên
            nv.put("NgaySinh", emp[2]);                                    // Ngày sinh
            nv.put("GioiTinh", emp[3]);                                    // Giới tính
            nv.put("SoDienThoai", emp[4]);                                 // Số điện thoại
            nv.put("Email", emp[5]);                                       // Email
            nv.put("NgayVaoLam", emp[6]);                                  // Ngày vào làm
            nv.put("MaPhongBan", emp[7]);                                  // Mã phòng ban
            nv.put("MaChucVu", emp[8]);                                    // Mã chức vụ
            nv.put("TrangThaiLamViec", "Đang làm việc");                   // Trạng thái làm việc mặc định
            db.insert(TABLE_NHAN_VIEN, null, nv);                         // Insert vào bảng NhanVien
        }
        
        // Cập nhật Trưởng phòng
        ContentValues updatePB1 = new ContentValues();                    // Cập nhật trưởng phòng cho PB001
        updatePB1.put("TruongPhong", "NV002");                             // Set NV002 làm trưởng phòng PB001
        db.update(TABLE_PHONG_BAN, updatePB1, "MaPhongBan = ?", new String[]{"PB001"}); // Update bảng PhongBan
        
        ContentValues updatePB2 = new ContentValues();                    // Cập nhật trưởng phòng cho PB002
        updatePB2.put("TruongPhong", "NV003");                             // Set NV003 làm trưởng phòng PB002
        db.update(TABLE_PHONG_BAN, updatePB2, "MaPhongBan = ?", new String[]{"PB002"}); // Update bảng PhongBan
        
        // Thêm tài khoản ADMIN riêng
        ContentValues tkAdmin = new ContentValues();                      // Tạo tài khoản Admin đặc biệt
        tkAdmin.put("MaNhanVien", "ADMIN");                                // Mã nhân viên đặc biệt cho Admin
        tkAdmin.put("TenDangNhap", "admin");                               // Tên đăng nhập admin
        tkAdmin.put("MatKhau", "123456");                                  // Mật khẩu mặc định
        tkAdmin.put("VaiTro", "Admin");                                    // Vai trò Admin
        tkAdmin.put("TrangThai", 1);                                       // Trạng thái active
        db.insert(TABLE_TAI_KHOAN, null, tkAdmin);                         // Insert vào bảng TaiKhoan
        
        // Thêm tài khoản cho nhân viên
        String[][] accounts = {                                            // Mảng 2 chiều chứa thông tin tài khoản cho 10 nhân viên
            {"NV002", "hr", "123456", "HR"},                               // Tài khoản HR
            {"NV003", "manager1", "123456", "Manager"},                    // Tài khoản Manager
            {"NV004", "user1", "123456", "Employee"},                      // Tài khoản Employee
            {"NV005", "user2", "123456", "Employee"},                      // Tài khoản Employee
            {"NV006", "user3", "123456", "Employee"},                      // Tài khoản Employee
            {"NV007", "manager2", "123456", "Manager"},                    // Tài khoản Manager
            {"NV008", "user4", "123456", "Employee"},                      // Tài khoản Employee
            {"NV009", "manager3", "123456", "Manager"},                    // Tài khoản Manager
            {"NV010", "user5", "123456", "Employee"},                      // Tài khoản Employee
            {"NV011", "intern1", "123456", "Employee"}                     // Tài khoản Employee (thực tập sinh)
        };
        
        for (String[] acc : accounts) {                                    // Lặp qua từng tài khoản trong mảng
            ContentValues tk = new ContentValues();                       // Tạo ContentValues cho mỗi tài khoản
            tk.put("MaNhanVien", acc[0]);                                  // Mã nhân viên liên kết
            tk.put("TenDangNhap", acc[1]);                                 // Tên đăng nhập
            tk.put("MatKhau", acc[2]);                                     // Mật khẩu
            tk.put("VaiTro", acc[3]);                                      // Vai trò
            tk.put("TrangThai", 1);                                        // Trạng thái active
            db.insert(TABLE_TAI_KHOAN, null, tk);                          // Insert vào bảng TaiKhoan
        }
        
        // Thêm dữ liệu Hợp đồng lao động
        String[][] contracts = {                                           // Mảng 2 chiều chứa thông tin hợp đồng cho 10 nhân viên
            {"HD001", "NV002", "Không thời hạn", "2020-02-01", null, "8000000"},
            {"HD002", "NV003", "Không thời hạn", "2020-03-15", null, "8000000"},
            {"HD003", "NV004", "Có thời hạn", "2020-04-01", "2025-03-31", "5000000"},
            {"HD004", "NV005", "Có thời hạn", "2020-05-10", "2025-05-09", "5000000"},
            {"HD005", "NV006", "Có thời hạn", "2021-01-15", "2026-01-14", "5000000"},
            {"HD006", "NV007", "Không thời hạn", "2021-02-20", null, "6500000"},
            {"HD007", "NV008", "Có thời hạn", "2021-03-10", "2026-03-09", "5000000"},
            {"HD008", "NV009", "Không thời hạn", "2021-04-05", null, "6500000"},
            {"HD009", "NV010", "Có thời hạn", "2022-01-10", "2027-01-09", "5000000"},
            {"HD010", "NV011", "Có thời hạn", "2023-06-01", "2024-05-31", "3000000"}
        };
        
        for (String[] contract : contracts) {                              // Lặp qua từng hợp đồng trong mảng
            ContentValues hd = new ContentValues();                       // Tạo ContentValues cho mỗi hợp đồng
            hd.put("MaHopDong", contract[0]);                              // Mã hợp đồng
            hd.put("MaNhanVien", contract[1]);                             // Mã nhân viên
            hd.put("LoaiHopDong", contract[2]);                            // Loại hợp đồng
            hd.put("NgayBatDau", contract[3]);                             // Ngày bắt đầu
            if (contract[4] != null) hd.put("NgayKetThuc", contract[4]);   // Ngày kết thúc (nếu có)
            hd.put("MucLuong", Double.parseDouble(contract[5]));           // Mức lương
            hd.put("TrangThai", "Hiệu lực");                               // Trạng thái hợp đồng
            db.insert(TABLE_HOP_DONG, null, hd);                           // Insert vào bảng HopDongLaoDong
        }
        
        // Thêm dữ liệu nghỉ phép
        insertSampleLeaveRequests(db);                                     // Gọi method chèn dữ liệu nghỉ phép mẫu
        
        // Thêm dữ liệu chấm công mẫu
        insertSampleAttendance(db);                                        // Gọi method chèn dữ liệu chấm công mẫu
    }
```
### Method insertSampleLeaveRequests:
```java
    private void insertSampleLeaveRequests(SQLiteDatabase db) {            // Method chèn dữ liệu nghỉ phép mẫu
        java.time.LocalDate today = java.time.LocalDate.now();            // Lấy ngày hiện tại
        
        // Một số đơn nghỉ phép trong tháng hiện tại và tháng trước
        String[][] leaveRequests = {                                       // Mảng 2 chiều chứa thông tin đơn nghỉ phép mẫu
            {"NV004", today.minusDays(15).toString(), today.minusDays(15).toString(), "1", "Khám bệnh định kỳ", "Đã duyệt", "NV002"},
            {"NV005", today.minusDays(10).toString(), today.minusDays(8).toString(), "3", "Về quê thăm gia đình", "Đã duyệt", "NV003"},
            {"NV006", today.minusDays(5).toString(), today.minusDays(5).toString(), "1", "Bị cảm cúm", "Đã duyệt", "NV002"},
            {"NV008", today.minusDays(3).toString(), today.minusDays(2).toString(), "2", "Công việc cá nhân", "Chờ duyệt", null},
            {"NV010", today.plusDays(5).toString(), today.plusDays(7).toString(), "3", "Nghỉ lễ gia đình", "Chờ duyệt", null},
            {"NV011", today.minusDays(20).toString(), today.minusDays(20).toString(), "1", "Đi học", "Từ chối", "NV003"},
            {"NV007", today.minusDays(12).toString(), today.minusDays(11).toString(), "2", "Nghỉ phép năm", "Đã duyệt", "NV002"}
        };
        
        for (String[] leave : leaveRequests) {                             // Lặp qua từng đơn nghỉ phép trong mảng
            ContentValues np = new ContentValues();                       // Tạo ContentValues cho mỗi đơn nghỉ phép
            np.put("MaNhanVien", leave[0]);                                // Mã nhân viên
            np.put("NgayBatDau", leave[1]);                                // Ngày bắt đầu nghỉ
            np.put("NgayKetThuc", leave[2]);                               // Ngày kết thúc nghỉ
            np.put("SoNgayNghi", Integer.parseInt(leave[3]));              // Số ngày nghỉ
            np.put("LyDo", leave[4]);                                      // Lý do nghỉ phép
            np.put("TrangThai", leave[5]);                                 // Trạng thái đơn
            if (leave[6] != null) np.put("NguoiDuyet", leave[6]);          // Người duyệt (nếu có)
            db.insert(TABLE_NGHI_PHEP, null, np);                          // Insert vào bảng NghiPhep
        }
    }
```

### Method insertSampleAttendance:
```java
    private void insertSampleAttendance(SQLiteDatabase db) {               // Method chèn dữ liệu chấm công mẫu cho 4 tháng
        // Dữ liệu tháng 1, 2, 3 và tháng 4 đến ngày 17
        String[] employees = {"NV002", "NV003", "NV004", "NV005", "NV006", "NV007", "NV008", "NV009", "NV010", "NV011"}; // Mảng mã nhân viên
        String[] startTimes = {"08:00:00", "08:15:00", "08:30:00", "08:00:00", "08:10:00", "08:00:00", "08:15:00", "08:30:00", "08:00:00", "08:10:00"}; // Giờ vào làm mẫu
        String[] endTimes = {"17:00:00", "17:15:00", "17:30:00", "16:45:00", "17:20:00", "17:00:00", "17:15:00", "17:30:00", "16:45:00", "17:20:00"}; // Giờ ra về mẫu
        double[] baseHours = {8.0, 8.0, 8.0, 7.75, 8.17, 8.0, 8.0, 8.0, 7.75, 8.17}; // Số giờ làm cơ bản cho mỗi nhân viên
        double[] variations = {0.0, 0.5, 1.0, 0.0, 0.5, 0.0, 1.5, 0.0, 0.5, 0.0, 1.0, 0.0, 0.5, 0.0, 1.0, 0.0, 0.5, 0.0, 1.5, 0.0}; // Biến thiên giờ làm

        int[] months = {1, 2, 3, 4};                                       // Mảng các tháng cần tạo dữ liệu

        for (int m = 0; m < months.length; m++) {                          // Lặp qua từng tháng
            int month = months[m];                                         // Lấy tháng hiện tại
            int daysInMonth = 31;                                          // Số ngày trong tháng mặc định
            if (month == 2) daysInMonth = 28;                              // Tháng 2 có 28 ngày
            if (month == 4) daysInMonth = 17;                              // Tháng 4 chỉ đến ngày 17

            int varIndex = 0;                                              // Index cho mảng variations
            for (int day = 1; day <= daysInMonth; day++) {                 // Lặp qua từng ngày trong tháng
                java.time.LocalDate date = java.time.LocalDate.of(2026, month, day); // Tạo LocalDate cho ngày hiện tại
                // Nghỉ T7, CN để có khoảng 20-22 ngày công 1 tháng
                if (date.getDayOfWeek() == java.time.DayOfWeek.SUNDAY || date.getDayOfWeek() == java.time.DayOfWeek.SATURDAY) { // Nếu là thứ 7 hoặc chủ nhật
                    continue;                                              // Bỏ qua ngày nghỉ
                }

                String dateStr = String.format(java.util.Locale.US, "%04d-%02d-%02d", 2026, month, day); // Format ngày thành string yyyy-MM-dd

                for (int i = 0; i < employees.length; i++) {               // Lặp qua từng nhân viên
                    double variation = variations[varIndex % variations.length]; // Lấy biến thiên giờ làm
                    double actualHours = baseHours[i] + variation;         // Tính số giờ làm thực tế
                    String[] time = calculateWorkingTime(startTimes[i], endTimes[i], actualHours); // Tính giờ vào/ra dựa trên số giờ làm
                    String note = getWorkingNote(actualHours, baseHours[i] < 8.0 ? "Về sớm" : ""); // Tạo ghi chú dựa trên số giờ làm

                    ContentValues cv = new ContentValues();               // Tạo ContentValues cho record chấm công
                    cv.put("MaNhanVien", employees[i]);                    // Mã nhân viên
                    cv.put("NgayChamCong", dateStr);                       // Ngày chấm công
                    cv.put("GioVao", time[0]);                             // Giờ vào
                    cv.put("GioRa", time[1]);                              // Giờ ra
                    cv.put("SoGioLam", actualHours);                       // Số giờ làm
                    cv.put("TrangThai", "Có mặt");                         // Trạng thái có mặt
                    cv.put("GhiChu", note);                                // Ghi chú
                    db.insert(TABLE_CHAM_CONG, null, cv);                  // Insert vào bảng ChamCong
                }
                varIndex++;                                                // Tăng index cho variation
            }
        }
    }
```

### Helper methods cho sample data:
```java
    private String getRandomAbsentReason() {                               // Method tạo lý do vắng mặt ngẫu nhiên
        String[] reasons = {                                               // Mảng các lý do vắng mặt
            "Nghỉ ốm",
            "Nghỉ phép",
            "Công việc cá nhân",
            "Nghỉ không phép",
            "Khám bệnh",
            "Việc gia đình"
        };
        return reasons[(int) (Math.random() * reasons.length)];           // Trả về lý do ngẫu nhiên
    }
    
    private String[] calculateWorkingTime(String baseStart, String baseEnd, double actualHours) { // Method tính giờ vào/ra dựa trên số giờ làm thực tế
        try {
            // Giữ nguyên giờ vào cơ bản, điều chỉnh giờ ra
            String[] startParts = baseStart.split(":");                   // Tách giờ vào thành giờ và phút
            int startHour = Integer.parseInt(startParts[0]);              // Lấy giờ vào
            int startMinute = Integer.parseInt(startParts[1]);            // Lấy phút vào
            
            // Tính giờ ra dựa trên số giờ làm thực tế
            int totalMinutes = (int) (actualHours * 60);                  // Chuyển số giờ làm thành phút
            int endHour = startHour + (totalMinutes / 60);                // Tính giờ ra
            int endMinute = startMinute + (totalMinutes % 60);            // Tính phút ra
            
            if (endMinute >= 60) {                                        // Nếu phút >= 60
                endHour += endMinute / 60;                                // Cộng thêm giờ
                endMinute = endMinute % 60;                               // Lấy phần dư phút
            }
            
            // Đảm bảo giờ ra không quá 23:59
            if (endHour >= 24) {                                          // Nếu giờ ra >= 24
                endHour = 23;                                             // Set giờ ra = 23
                endMinute = 59;                                           // Set phút ra = 59
            }
            
            String endTime = String.format("%02d:%02d:00", endHour, endMinute); // Format giờ ra thành string HH:mm:ss
            return new String[]{baseStart, endTime};                      // Trả về mảng [giờ vào, giờ ra]
        } catch (Exception e) {                                           // Catch exception
            return new String[]{baseStart, baseEnd};                      // Trả về giờ mặc định nếu có lỗi
        }
    }
    
    private String getWorkingNote(double actualHours, String baseNote) {   // Method tạo ghi chú dựa trên số giờ làm
        if (actualHours > 10.0) {                                         // Nếu làm > 10 giờ
            return "Tăng ca rất nhiều";                                   // Ghi chú tăng ca rất nhiều
        } else if (actualHours > 9.0) {                                   // Nếu làm > 9 giờ
            return "Tăng ca nhiều";                                       // Ghi chú tăng ca nhiều
        } else if (actualHours > 8.5) {                                   // Nếu làm > 8.5 giờ
            return "Tăng ca";                                             // Ghi chú tăng ca
        } else if (actualHours < 7.0) {                                   // Nếu làm < 7 giờ
            return "Thiếu giờ";                                           // Ghi chú thiếu giờ
        } else if (actualHours < 7.5) {                                   // Nếu làm < 7.5 giờ
            return "Đi muộn/về sớm";                                      // Ghi chú đi muộn/về sớm
        } else {                                                          // Nếu làm bình thường
            return baseNote.isEmpty() ? "Làm việc bình thường" : baseNote; // Trả về ghi chú bình thường hoặc baseNote
        }
    }
```

---

## 4️⃣ EMPLOYEE MANAGEMENT METHODS

### Method addEmployee:
```java
    public boolean addEmployee(String maNV, String hoTen, String ngaySinh, String gioiTinh, // Method thêm nhân viên mới
                              String sdt, String email, String maPB, String maCV, String hinhAnh) {
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues cv = new ContentValues();                           // Tạo ContentValues để chứa dữ liệu
        cv.put("MaNhanVien", maNV);                                        // Thêm mã nhân viên
        cv.put("HoTen", hoTen);                                            // Thêm họ tên
        cv.put("NgaySinh", ngaySinh);                                      // Thêm ngày sinh
        cv.put("GioiTinh", gioiTinh);                                      // Thêm giới tính
        cv.put("SoDienThoai", sdt);                                        // Thêm số điện thoại
        cv.put("Email", email);                                            // Thêm email
        cv.put("NgayVaoLam", java.time.LocalDate.now().toString());       // Thêm ngày vào làm (ngày hiện tại)
        cv.put("MaPhongBan", maPB);                                        // Thêm mã phòng ban
        cv.put("MaChucVu", maCV);                                          // Thêm mã chức vụ
        cv.put("HinhAnh", hinhAnh);                                        // Thêm đường dẫn hình ảnh
        cv.put("TrangThaiLamViec", "Đang làm việc");                       // Set trạng thái mặc định
        long result = db.insert(TABLE_NHAN_VIEN, null, cv);               // Insert vào bảng NhanVien
        return result != -1;                                              // Trả về true nếu insert thành công
    }
```

### Method updateEmployee:
```java
    public boolean updateEmployee(String maNV, String hoTen, String ngaySinh, String gioiTinh, // Method cập nhật thông tin nhân viên
                                 String sdt, String email, String maPB, String maCV, String hinhAnh) {
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues cv = new ContentValues();                           // Tạo ContentValues để chứa dữ liệu cập nhật
        cv.put("HoTen", hoTen);                                            // Cập nhật họ tên
        cv.put("NgaySinh", ngaySinh);                                      // Cập nhật ngày sinh
        cv.put("GioiTinh", gioiTinh);                                      // Cập nhật giới tính
        cv.put("SoDienThoai", sdt);                                        // Cập nhật số điện thoại
        cv.put("Email", email);                                            // Cập nhật email
        cv.put("MaPhongBan", maPB);                                        // Cập nhật mã phòng ban
        cv.put("MaChucVu", maCV);                                          // Cập nhật mã chức vụ
        cv.put("HinhAnh", hinhAnh);                                        // Cập nhật đường dẫn hình ảnh
        int result = db.update(TABLE_NHAN_VIEN, cv, "MaNhanVien = ?", new String[]{maNV}); // Update WHERE mã nhân viên
        return result > 0;                                                // Trả về true nếu có record được cập nhật
    }
```

### Method getAllEmployees:
```java
    public Cursor getAllEmployees() {                                      // Method lấy tất cả nhân viên với thông tin JOIN
        String query = "SELECT nv.*, cv.TenChucVu, pb.TenPhongBan FROM " + TABLE_NHAN_VIEN + " nv " + // Query JOIN 3 bảng
                      "LEFT JOIN " + TABLE_CHUC_VU + " cv ON nv.MaChucVu = cv.MaChucVu " + // LEFT JOIN với bảng ChucVu
                      "LEFT JOIN " + TABLE_PHONG_BAN + " pb ON nv.MaPhongBan = pb.MaPhongBan " + // LEFT JOIN với bảng PhongBan
                      "WHERE nv.TrangThaiLamViec = 'Đang làm việc'";       // WHERE chỉ lấy nhân viên đang làm việc
        return getReadableDatabase().rawQuery(query, null);               // Thực hiện query và trả về Cursor
    }
```

### Method searchEmployees:
```java
    public Cursor searchEmployees(String keyword) {                        // Method tìm kiếm nhân viên theo keyword
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT nv.*, cv.TenChucVu, pb.TenPhongBan FROM " + TABLE_NHAN_VIEN + " nv " + // Query JOIN 3 bảng
                      "LEFT JOIN " + TABLE_CHUC_VU + " cv ON nv.MaChucVu = cv.MaChucVu " + // LEFT JOIN với bảng ChucVu
                      "LEFT JOIN " + TABLE_PHONG_BAN + " pb ON nv.MaPhongBan = pb.MaPhongBan " + // LEFT JOIN với bảng PhongBan
                      "WHERE nv.TrangThaiLamViec = 'Đang làm việc' AND " +  // WHERE nhân viên đang làm việc
                      "(nv.HoTen LIKE ? OR nv.MaNhanVien LIKE ? OR pb.TenPhongBan LIKE ?)"; // AND tìm kiếm theo họ tên, mã NV, tên phòng ban
        String wildcardKeyword = "%" + keyword + "%";                      // Thêm wildcard % vào keyword
        return db.rawQuery(query, new String[]{wildcardKeyword, wildcardKeyword, wildcardKeyword}); // Thực hiện query với 3 parameter
    }
```

### Method getNextEmployeeCode:
```java
    public String getNextEmployeeCode() {                                  // Method tự động tạo mã nhân viên tiếp theo
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT MaNhanVien FROM " + TABLE_NHAN_VIEN +       // Query lấy mã nhân viên lớn nhất
                      " WHERE MaNhanVien LIKE 'NV%' " +                    // WHERE mã nhân viên có format NVxxx
                      " ORDER BY length(MaNhanVien) DESC, MaNhanVien DESC LIMIT 1"; // ORDER BY độ dài và giá trị giảm dần, LIMIT 1
        Cursor cursor = db.rawQuery(query, null);                         // Thực hiện query
        String lastCode = null;                                            // Biến lưu mã cuối cùng
        if (cursor != null) {                                              // Nếu cursor không null
            if (cursor.moveToFirst()) lastCode = cursor.getString(0).trim(); // Lấy mã đầu tiên và trim khoảng trắng
            cursor.close();                                                // Đóng cursor
        }
        if (lastCode == null) return "NV002";                             // Nếu không có mã nào thì trả về NV002 (bỏ qua NV001 cho admin)
        
        // Parse mã để tăng số thứ tự
        java.util.regex.Matcher matcher = java.util.regex.Pattern.compile("^([a-zA-Z]+)(\\d+)$").matcher(lastCode); // Regex pattern để tách prefix và số
        if (matcher.find()) {                                              // Nếu match được pattern
            String prefix = matcher.group(1);                             // Lấy prefix (NV)
            String numberStr = matcher.group(2);                          // Lấy phần số (002, 003, ...)
            try {
                int nextNumber = Integer.parseInt(numberStr) + 1;         // Tăng số lên 1
                return String.format("%s%0" + numberStr.length() + "d", prefix, nextNumber); // Format với leading zeros
            } catch (Exception e) { return "NV002"; }                     // Nếu có lỗi parse thì trả về NV002
        }
        return "NV002";                                                    // Default return NV002
    }
```

### Method deleteEmployee:
```java
    public boolean deleteEmployee(String maNhanVien) {                     // Method xóa nhân viên (soft delete)
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để cập nhật trạng thái
        values.put("TrangThaiLamViec", "Đã nghỉ việc");                    // Set trạng thái thành "Đã nghỉ việc" thay vì xóa
        return db.update(TABLE_NHAN_VIEN, values, "MaNhanVien = ?", new String[]{maNhanVien}) > 0; // Update và trả về kết quả
    }
```
---

## 5️⃣ AUTHENTICATION & ACCOUNT MANAGEMENT METHODS

### Method checkLogin:
```java
    public boolean checkLogin(String username, String password) {          // Method kiểm tra đăng nhập
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        Cursor cursor = db.rawQuery("SELECT * FROM " + TABLE_TAI_KHOAN +   // Query SELECT với WHERE conditions
                " WHERE TenDangNhap = ? AND MatKhau = ? AND TrangThai = 1", // WHERE username, password và trạng thái active
                new String[]{username, password});                         // Parameters để tránh SQL injection
        boolean success = (cursor.getCount() > 0);                        // Kiểm tra có record nào trả về không
        cursor.close();                                                    // Đóng cursor
        return success;                                                    // Trả về true nếu đăng nhập thành công
    }
```

### Method checkCurrentPassword:
```java
    public boolean checkCurrentPassword(String username, String currentPassword) { // Method kiểm tra mật khẩu hiện tại
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        Cursor cursor = db.rawQuery("SELECT * FROM " + TABLE_TAI_KHOAN +   // Query SELECT với WHERE conditions
                " WHERE TenDangNhap = ? AND MatKhau = ?",                  // WHERE username và password hiện tại
                new String[]{username, currentPassword});                  // Parameters để tránh SQL injection
        boolean isCorrect = (cursor.getCount() > 0);                      // Kiểm tra có record nào trả về không
        cursor.close();                                                    // Đóng cursor
        return isCorrect;                                                  // Trả về true nếu mật khẩu đúng
    }
```

### Method changePassword:
```java
    public boolean changePassword(String username, String newPassword) {   // Method đổi mật khẩu
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues cv = new ContentValues();                           // Tạo ContentValues để chứa dữ liệu cập nhật
        cv.put("MatKhau", newPassword);                                    // Cập nhật mật khẩu mới
        int result = db.update(TABLE_TAI_KHOAN, cv, "TenDangNhap = ?", new String[]{username}); // Update WHERE username
        return result > 0;                                                // Trả về true nếu có record được cập nhật
    }
```

### Method getUserInfo:
```java
    public Cursor getUserInfo(String username) {                           // Method lấy thông tin user
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT " +                                         // Query SELECT với CASE WHEN
                "CASE WHEN tk.MaNhanVien = 'ADMIN' THEN 'Administrator' ELSE nv.HoTen END as HoTen, " + // Nếu là ADMIN thì hiển thị 'Administrator', không thì lấy HoTen
                "tk.VaiTro " +                                             // Lấy vai trò từ bảng TaiKhoan
                "FROM " + TABLE_TAI_KHOAN + " tk " +                       // FROM bảng TaiKhoan với alias tk
                "LEFT JOIN " + TABLE_NHAN_VIEN + " nv ON tk.MaNhanVien = nv.MaNhanVien " + // LEFT JOIN với bảng NhanVien
                "WHERE tk.TenDangNhap = ?";                                // WHERE username
        return db.rawQuery(query, new String[]{username});                // Thực hiện query với parameter
    }
```

### Method registerUser:
```java
    public boolean registerUser(String maNV, String hoTen, String ngaySinh, String gioiTinh, // Method đăng ký user mới với transaction
                               String sdt, String email, String ngayVaoLam, String maPB,     // Nhận tất cả thông tin cần thiết
                               String maCV, String username, String password) {              // Bao gồm cả thông tin nhân viên và tài khoản
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        db.beginTransaction();                                             // Bắt đầu transaction để đảm bảo tính toàn vẹn dữ liệu
        try {
            // Tạo record nhân viên
            ContentValues nvValues = new ContentValues();                 // Tạo ContentValues cho bảng NhanVien
            nvValues.put("MaNhanVien", maNV);                             // Thêm mã nhân viên
            nvValues.put("HoTen", hoTen);                                 // Thêm họ tên
            nvValues.put("NgaySinh", ngaySinh);                           // Thêm ngày sinh
            nvValues.put("GioiTinh", gioiTinh);                           // Thêm giới tính
            nvValues.put("SoDienThoai", sdt);                             // Thêm số điện thoại
            nvValues.put("Email", email);                                 // Thêm email
            nvValues.put("NgayVaoLam", ngayVaoLam);                       // Thêm ngày vào làm
            nvValues.put("MaPhongBan", maPB);                             // Thêm mã phòng ban
            nvValues.put("MaChucVu", maCV);                               // Thêm mã chức vụ
            nvValues.put("TrangThaiLamViec", "Đang làm việc");            // Set trạng thái mặc định
            
            long nvResult = db.insert(TABLE_NHAN_VIEN, null, nvValues);   // Insert record vào bảng NhanVien
            if (nvResult == -1) return false;                            // Nếu insert thất bại thì return false

            // Tạo record tài khoản
            ContentValues tkValues = new ContentValues();                // Tạo ContentValues cho bảng TaiKhoan
            tkValues.put("MaNhanVien", maNV);                             // Liên kết với mã nhân viên
            tkValues.put("TenDangNhap", username);                        // Thêm tên đăng nhập
            tkValues.put("MatKhau", password);                            // Thêm mật khẩu (plain text)
            tkValues.put("VaiTro", "Employee");                           // Set vai trò mặc định là Employee
            tkValues.put("TrangThai", 1);                                 // Set trạng thái active (1)
            
            long tkResult = db.insert(TABLE_TAI_KHOAN, null, tkValues);   // Insert record vào bảng TaiKhoan
            if (tkResult == -1) return false;                            // Nếu insert thất bại thì return false

            db.setTransactionSuccessful();                                // Đánh dấu transaction thành công
            return true;                                                  // Trả về true nếu tất cả thành công
        } catch (Exception e) {                                           // Catch mọi exception
            return false;                                                 // Trả về false nếu có lỗi
        } finally {
            db.endTransaction();                                          // Kết thúc transaction (commit hoặc rollback)
        }
    }
```

### Method checkEmployeeExists & checkUsernameExists:
```java
    public boolean checkEmployeeExists(String maNhanVien) {                // Method kiểm tra mã nhân viên đã tồn tại
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        Cursor cursor = db.rawQuery("SELECT * FROM " + TABLE_NHAN_VIEN + " WHERE MaNhanVien = ?", new String[]{maNhanVien}); // Query kiểm tra mã nhân viên
        boolean exists = (cursor.getCount() > 0);                         // Kiểm tra có record nào trả về không
        cursor.close();                                                    // Đóng cursor
        return exists;                                                     // Trả về true nếu mã nhân viên đã tồn tại
    }

    public boolean checkUsernameExists(String username) {                  // Method kiểm tra tên đăng nhập đã tồn tại
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        Cursor cursor = db.rawQuery("SELECT * FROM " + TABLE_TAI_KHOAN + " WHERE TenDangNhap = ?", new String[]{username}); // Query kiểm tra tên đăng nhập
        boolean exists = (cursor.getCount() > 0);                         // Kiểm tra có record nào trả về không
        cursor.close();                                                    // Đóng cursor
        return exists;                                                     // Trả về true nếu tên đăng nhập đã tồn tại
    }
```

---

## 6️⃣ ATTENDANCE MANAGEMENT METHODS

### Method getMaNhanVienByUsername:
```java
    public String getMaNhanVienByUsername(String username) {               // Method lấy mã nhân viên từ username
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT MaNhanVien FROM " + TABLE_TAI_KHOAN + " WHERE TenDangNhap = ?"; // Query lấy mã nhân viên
        Cursor cursor = db.rawQuery(query, new String[]{username});       // Thực hiện query với parameter
        String maNV = null;                                                // Biến lưu mã nhân viên
        if (cursor != null && cursor.moveToFirst()) {                      // Nếu cursor không null và có dữ liệu
            maNV = cursor.getString(0);                                    // Lấy mã nhân viên từ column đầu tiên
            cursor.close();                                                // Đóng cursor
        }
        return maNV;                                                       // Trả về mã nhân viên
    }
```

### Method getTodayAttendanceStatus:
```java
    public boolean[] getTodayAttendanceStatus(String maNhanVien, String date) { // Method kiểm tra trạng thái chấm công trong ngày
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT GioVao, GioRa FROM " + TABLE_CHAM_CONG +    // Query lấy giờ vào và giờ ra
                      " WHERE MaNhanVien = ? AND NgayChamCong = ?";        // WHERE theo mã nhân viên và ngày
        Cursor cursor = db.rawQuery(query, new String[]{maNhanVien, date}); // Thực hiện query với parameters
        
        boolean hasCheckedIn = false;                                      // Biến kiểm tra đã chấm công vào chưa
        boolean hasCheckedOut = false;                                     // Biến kiểm tra đã chấm công ra chưa
        
        if (cursor != null && cursor.moveToFirst()) {                      // Nếu cursor không null và có dữ liệu
            String gioVao = cursor.getString(0);                           // Lấy giờ vào
            String gioRa = cursor.getString(1);                            // Lấy giờ ra
            
            hasCheckedIn = (gioVao != null && !gioVao.isEmpty());         // Kiểm tra đã có giờ vào chưa
            hasCheckedOut = (gioRa != null && !gioRa.isEmpty());          // Kiểm tra đã có giờ ra chưa
            
            cursor.close();                                                // Đóng cursor
        }
        
        return new boolean[]{hasCheckedIn, hasCheckedOut};                 // Trả về mảng boolean [đã vào, đã ra]
    }
```

### Method chamCongVao:
```java
    public boolean chamCongVao(String maNhanVien, String date, String time) { // Method chấm công vào
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        
        String checkQuery = "SELECT MaChamCong FROM " + TABLE_CHAM_CONG +  // Query kiểm tra đã có record chấm công chưa
                           " WHERE MaNhanVien = ? AND NgayChamCong = ?";   // WHERE theo mã nhân viên và ngày
        Cursor cursor = db.rawQuery(checkQuery, new String[]{maNhanVien, date}); // Thực hiện query kiểm tra
        
        boolean exists = cursor.getCount() > 0;                           // Kiểm tra đã có record chưa
        cursor.close();                                                    // Đóng cursor
        
        if (exists) {                                                      // Nếu đã có record
            ContentValues values = new ContentValues();                   // Tạo ContentValues để cập nhật
            values.put("GioVao", time);                                    // Cập nhật giờ vào
            values.put("TrangThai", "Có mặt");                             // Cập nhật trạng thái
            
            int result = db.update(TABLE_CHAM_CONG, values,                // Update record hiện có
                                 "MaNhanVien = ? AND NgayChamCong = ?",    // WHERE theo mã nhân viên và ngày
                                 new String[]{maNhanVien, date});          // Parameters
            return result > 0;                                             // Trả về true nếu update thành công
        } else {                                                           // Nếu chưa có record
            ContentValues values = new ContentValues();                   // Tạo ContentValues để insert
            values.put("MaNhanVien", maNhanVien);                          // Thêm mã nhân viên
            values.put("NgayChamCong", date);                              // Thêm ngày chấm công
            values.put("GioVao", time);                                    // Thêm giờ vào
            values.put("SoGioLam", 0);                                     // Set số giờ làm = 0 (chưa ra)
            values.put("TrangThai", "Có mặt");                             // Set trạng thái có mặt
            
            long result = db.insert(TABLE_CHAM_CONG, null, values);       // Insert record mới
            return result != -1;                                           // Trả về true nếu insert thành công
        }
    }
```

### Method chamCongRa:
```java
    public boolean chamCongRa(String maNhanVien, String date, String time) { // Method chấm công ra
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        
        String query = "SELECT GioVao FROM " + TABLE_CHAM_CONG +           // Query lấy giờ vào để tính số giờ làm
                      " WHERE MaNhanVien = ? AND NgayChamCong = ?";        // WHERE theo mã nhân viên và ngày
        Cursor cursor = db.rawQuery(query, new String[]{maNhanVien, date}); // Thực hiện query
        
        double soGioLam = 0;                                               // Biến lưu số giờ làm
        if (cursor != null && cursor.moveToFirst()) {                      // Nếu cursor không null và có dữ liệu
            String gioVao = cursor.getString(0);                           // Lấy giờ vào
            if (gioVao != null) {                                          // Nếu có giờ vào
                soGioLam = tinhSoGioLam(gioVao, time);                     // Tính số giờ làm
            }
            cursor.close();                                                // Đóng cursor
        }
        
        ContentValues values = new ContentValues();                       // Tạo ContentValues để cập nhật
        values.put("GioRa", time);                                         // Cập nhật giờ ra
        values.put("SoGioLam", soGioLam);                                  // Cập nhật số giờ làm
        
        int result = db.update(TABLE_CHAM_CONG, values,                    // Update record
                             "MaNhanVien = ? AND NgayChamCong = ?",        // WHERE theo mã nhân viên và ngày
                             new String[]{maNhanVien, date});              // Parameters
        return result > 0;                                                 // Trả về true nếu update thành công
    }
```

### Method tinhSoGioLam:
```java
    private double tinhSoGioLam(String gioVao, String gioRa) {             // Method tính số giờ làm từ giờ vào và giờ ra
        try {
            String[] vao = gioVao.split(":");                             // Tách giờ vào thành mảng [giờ, phút, giây]
            String[] ra = gioRa.split(":");                               // Tách giờ ra thành mảng [giờ, phút, giây]
            
            int gioVaoMinutes = Integer.parseInt(vao[0]) * 60 + Integer.parseInt(vao[1]); // Chuyển giờ vào thành phút
            int gioRaMinutes = Integer.parseInt(ra[0]) * 60 + Integer.parseInt(ra[1]);    // Chuyển giờ ra thành phút
            
            int diffMinutes = gioRaMinutes - gioVaoMinutes;               // Tính chênh lệch phút
            return diffMinutes / 60.0;                                    // Chuyển về giờ (số thực)
        } catch (Exception e) {                                           // Catch exception nếu có lỗi parse
            return 0;                                                     // Trả về 0 nếu có lỗi
        }
    }
```

### Method tinhGioTangCa:
```java
    public double tinhGioTangCa(double soGioLam) {                         // Method tính giờ tăng ca (giờ làm > 8 giờ)
        if (soGioLam > 8.0) {                                             // Nếu số giờ làm > 8 giờ
            return soGioLam - 8.0;                                        // Trả về số giờ tăng ca
        }
        return 0;                                                         // Trả về 0 nếu không có tăng ca
    }
```

### Method getAttendanceHistory:
```java
    public Cursor getAttendanceHistory(String maNhanVien, int limit) {     // Method lấy lịch sử chấm công của một nhân viên
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        try {
            String query = "SELECT * FROM " + TABLE_CHAM_CONG +            // Query SELECT tất cả columns
                          " WHERE MaNhanVien = ? ORDER BY NgayChamCong DESC LIMIT ?"; // WHERE theo mã nhân viên, ORDER BY ngày giảm dần, LIMIT
            return db.rawQuery(query, new String[]{maNhanVien, String.valueOf(limit)}); // Thực hiện query với parameters
        } catch (Exception e) {                                           // Catch exception
            e.printStackTrace();                                          // In stack trace
            return null;                                                  // Trả về null nếu có lỗi
        }
    }
```

### Method getAllAttendanceHistory:
```java
    public Cursor getAllAttendanceHistory(int limit) {                     // Method lấy lịch sử chấm công của tất cả nhân viên
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        try {
            String query = "SELECT * FROM " + TABLE_CHAM_CONG + " ORDER BY NgayChamCong DESC, MaNhanVien ASC LIMIT ?"; // Query SELECT tất cả, ORDER BY ngày giảm dần và mã NV tăng dần, LIMIT
            return db.rawQuery(query, new String[]{String.valueOf(limit)}); // Thực hiện query với parameter
        } catch (Exception e) {                                           // Catch exception
            e.printStackTrace();                                          // In stack trace
            return null;                                                  // Trả về null nếu có lỗi
        }
    }
```

### Method updateAttendance & deleteAttendance:
```java
    public boolean updateAttendance(String maNhanVien, String ngayChamCong, String gioVao, String gioRa, String ghiChu) { // Method cập nhật thông tin chấm công
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        try {
            ContentValues values = new ContentValues();                   // Tạo ContentValues để chứa dữ liệu cập nhật
            
            if (gioVao != null && !gioVao.isEmpty()) {                     // Nếu có giờ vào
                values.put("GioVao", gioVao);                              // Cập nhật giờ vào
            }
            
            if (gioRa != null && !gioRa.isEmpty()) {                       // Nếu có giờ ra
                values.put("GioRa", gioRa);                                // Cập nhật giờ ra
                
                // Tính lại số giờ làm nếu có cả giờ vào và giờ ra
                if (gioVao != null && !gioVao.isEmpty()) {                 // Nếu có cả giờ vào và giờ ra
                    double soGioLam = tinhSoGioLam(gioVao, gioRa);         // Tính số giờ làm
                    values.put("SoGioLam", soGioLam);                      // Cập nhật số giờ làm
                }
            }
            
            // Cập nhật ghi chú
            if (ghiChu != null) {                                          // Nếu có ghi chú
                values.put("GhiChu", ghiChu);                              // Cập nhật ghi chú
            }
            
            int result = db.update(TABLE_CHAM_CONG, values,                // Update record
                                 "MaNhanVien = ? AND NgayChamCong = ?",    // WHERE theo mã nhân viên và ngày
                                 new String[]{maNhanVien, ngayChamCong});  // Parameters
            return result > 0;                                             // Trả về true nếu update thành công
        } catch (Exception e) {                                           // Catch exception
            e.printStackTrace();                                          // In stack trace
            return false;                                                 // Trả về false nếu có lỗi
        }
    }
    
    // Phương thức cũ để tương thích (không có ghi chú)
    public boolean updateAttendance(String maNhanVien, String ngayChamCong, String gioVao, String gioRa) { // Method overload không có ghi chú
        return updateAttendance(maNhanVien, ngayChamCong, gioVao, gioRa, null); // Gọi method chính với ghiChu = null
    }

    public boolean deleteAttendance(String maNhanVien, String ngayChamCong) { // Method xóa record chấm công
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        try {
            int result = db.delete(TABLE_CHAM_CONG,                        // Delete từ bảng ChamCong
                                 "MaNhanVien = ? AND NgayChamCong = ?",    // WHERE theo mã nhân viên và ngày
                                 new String[]{maNhanVien, ngayChamCong});  // Parameters
            return result > 0;                                             // Trả về true nếu delete thành công
        } catch (Exception e) {                                           // Catch exception
            e.printStackTrace();                                          // In stack trace
            return false;                                                 // Trả về false nếu có lỗi
        }
    }
```
---

## 7️⃣ LEAVE REQUEST MANAGEMENT METHODS

### Method addLeaveRequest & submitLeaveRequest:
```java
    public boolean addLeaveRequest(String maNhanVien, String ngayBatDau, String ngayKetThuc, // Method thêm đơn nghỉ phép mới
                                  int soNgayNghi, String lyDo) {                           // Nhận tất cả thông tin cần thiết
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để chứa dữ liệu
        values.put("MaNhanVien", maNhanVien);                             // Thêm mã nhân viên
        values.put("NgayBatDau", ngayBatDau);                             // Thêm ngày bắt đầu
        values.put("NgayKetThuc", ngayKetThuc);                           // Thêm ngày kết thúc
        values.put("SoNgayNghi", soNgayNghi);                             // Thêm số ngày nghỉ
        values.put("LyDo", lyDo);                                         // Thêm lý do nghỉ phép
        values.put("TrangThai", "Chờ duyệt");                             // Set trạng thái mặc định là "Chờ duyệt"
        
        long result = db.insert(TABLE_NGHI_PHEP, null, values);           // Insert record vào bảng NghiPhep
        return result != -1;                                              // Trả về true nếu insert thành công
    }

    public boolean submitLeaveRequest(String maNhanVien, String ngayBatDau, String ngayKetThuc, // Method gửi đơn nghỉ phép (wrapper)
                                    int soNgayNghi, String lyDo) {                         // Nhận tất cả thông tin cần thiết
        return addLeaveRequest(maNhanVien, ngayBatDau, ngayKetThuc, soNgayNghi, lyDo);    // Gọi method addLeaveRequest
    }
```

### Method getLeaveRequests & getLeaveHistory:
```java
    public Cursor getLeaveRequests(String maNhanVien) {                    // Method lấy danh sách đơn nghỉ phép của một nhân viên
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT * FROM " + TABLE_NGHI_PHEP +                // Query SELECT tất cả columns
                      " WHERE MaNhanVien = ? ORDER BY NgayBatDau DESC";    // WHERE theo mã nhân viên, ORDER BY ngày bắt đầu giảm dần
        return db.rawQuery(query, new String[]{maNhanVien});              // Thực hiện query với parameter
    }

    public Cursor getLeaveHistory(String maNhanVien) {                     // Method lấy lịch sử nghỉ phép (wrapper)
        return getLeaveRequests(maNhanVien);                              // Gọi method getLeaveRequests
    }
```

### Method getAllLeaveRequests:
```java
    public Cursor getAllLeaveRequests() {                                  // Method lấy tất cả đơn nghỉ phép (cho Admin/HR/Manager)
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT np.*, nv.HoTen FROM " + TABLE_NGHI_PHEP + " np " + // Query JOIN với bảng NhanVien để lấy tên
                      "LEFT JOIN " + TABLE_NHAN_VIEN + " nv ON np.MaNhanVien = nv.MaNhanVien " + // LEFT JOIN để lấy thông tin nhân viên
                      "ORDER BY np.NgayBatDau DESC";                       // ORDER BY ngày bắt đầu giảm dần
        return db.rawQuery(query, null);                                  // Thực hiện query không có parameter
    }
```

### Method updateLeaveRequestStatus & approveLeaveRequest:
```java
    public boolean updateLeaveRequestStatus(int maNghiPhep, String trangThai, String nguoiDuyet) { // Method cập nhật trạng thái đơn nghỉ phép
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để chứa dữ liệu cập nhật
        values.put("TrangThai", trangThai);                               // Cập nhật trạng thái
        values.put("NguoiDuyet", nguoiDuyet);                             // Cập nhật người duyệt
        
        int result = db.update(TABLE_NGHI_PHEP, values,                   // Update record
                             "MaNghiPhep = ?", new String[]{String.valueOf(maNghiPhep)}); // WHERE theo mã nghỉ phép
        return result > 0;                                                // Trả về true nếu có record được cập nhật
    }

    public boolean approveLeaveRequest(int maNghiPhep, String trangThai) { // Method duyệt/từ chối đơn nghỉ phép
        return updateLeaveRequestStatus(maNghiPhep, trangThai, "Admin");   // Gọi method updateLeaveRequestStatus với người duyệt là "Admin"
    }
```

### Method updateLeaveRequest & deleteLeaveRequest:
```java
    public boolean updateLeaveRequest(int maNghiPhep, String ngayBatDau, String ngayKetThuc, int soNgayNghi, String lyDo) { // Method cập nhật thông tin đơn nghỉ phép
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để chứa dữ liệu cập nhật
        values.put("NgayBatDau", ngayBatDau);                             // Cập nhật ngày bắt đầu
        values.put("NgayKetThuc", ngayKetThuc);                           // Cập nhật ngày kết thúc
        values.put("SoNgayNghi", soNgayNghi);                             // Cập nhật số ngày nghỉ
        values.put("LyDo", lyDo);                                         // Cập nhật lý do
        int result = db.update(TABLE_NGHI_PHEP, values, "MaNghiPhep = ?", new String[]{String.valueOf(maNghiPhep)}); // Update WHERE theo mã nghỉ phép
        return result > 0;                                                // Trả về true nếu có record được cập nhật
    }

    public boolean deleteLeaveRequest(int maNghiPhep) {                    // Method xóa đơn nghỉ phép
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        int result = db.delete(TABLE_NGHI_PHEP, "MaNghiPhep = ?", new String[]{String.valueOf(maNghiPhep)}); // Delete WHERE theo mã nghỉ phép
        return result > 0;                                                // Trả về true nếu có record được xóa
    }
```

---

## 8️⃣ SALARY MANAGEMENT METHODS

### AttendanceStats Inner Class:
```java
    // Class để lưu thống kê chấm công
    public static class AttendanceStats {                                  // Inner class chứa thống kê chấm công
        public double soGioLam = 0;                                        // Tổng số giờ làm
        public double soGioTangCa = 0;                                     // Tổng số giờ tăng ca
        public int soNgayLam = 0;                                          // Tổng số ngày làm
    }
```

### Method calculateMonthlySalary:
```java
    public int calculateMonthlySalary(String thangNam) {                   // Method tính lương tháng cho tất cả nhân viên
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        int count = 0;                                                     // Biến đếm số nhân viên được tính lương
        
        try {
            // Lấy danh sách nhân viên và mức lương từ hợp đồng mới nhất (nếu có)
            String query = "SELECT nv.MaNhanVien, " +                      // Query lấy mã nhân viên và mức lương
                          "COALESCE((SELECT hd.MucLuong FROM " + TABLE_HOP_DONG + " hd " + // COALESCE để lấy mức lương từ hợp đồng hoặc chức vụ
                          "WHERE hd.MaNhanVien = nv.MaNhanVien AND hd.TrangThai = 'Hiệu lực' " + // WHERE hợp đồng hiệu lực
                          "ORDER BY hd.NgayBatDau DESC LIMIT 1), cv.MucLuongCoBan) " + // ORDER BY ngày bắt đầu giảm dần, LIMIT 1, fallback về lương chức vụ
                          "FROM " + TABLE_NHAN_VIEN + " nv " +              // FROM bảng NhanVien
                          "LEFT JOIN " + TABLE_CHUC_VU + " cv ON nv.MaChucVu = cv.MaChucVu " + // LEFT JOIN với bảng ChucVu
                          "WHERE nv.TrangThaiLamViec = 'Đang làm việc'";    // WHERE nhân viên đang làm việc
            Cursor cursor = db.rawQuery(query, null);                     // Thực hiện query
            
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                do {
                    String maNhanVien = cursor.getString(0);               // Lấy mã nhân viên
                    double luongCoBan = cursor.getDouble(1);               // Lấy mức lương cơ bản
                    
                    // Tính tổng số giờ làm và giờ tăng ca trong tháng
                    AttendanceStats stats = getAttendanceStats(maNhanVien, thangNam); // Lấy thống kê chấm công
                    
                    // Tính lương theo ngày làm việc thực tế
                    // Lương ngày = lương cơ bản / 26 ngày (1 tháng tiêu chuẩn)
                    double luongNgay = luongCoBan / 26.0;                  // Tính lương theo ngày
                    double luongTheoNgay = stats.soNgayLam * luongNgay;    // Lương theo số ngày làm thực tế
                    
                    // Tính lương giờ = lương cơ bản / 208 giờ (26 ngày × 8 giờ)
                    double luongGio = luongCoBan / 208.0;                  // Tính lương theo giờ
                    
                    // Phụ cấp tính theo tỷ lệ ngày làm việc
                    double phuCapCoBan = luongCoBan * 0.1;                 // Phụ cấp = 10% lương cơ bản
                    double phuCap = (stats.soNgayLam / 26.0) * phuCapCoBan; // Phụ cấp theo tỷ lệ ngày làm
                    
                    // Lương tăng ca = số giờ tăng ca × lương giờ × 1.5
                    double luongTangCa = stats.soGioTangCa * luongGio * 1.5; // Lương tăng ca với hệ số 1.5
                    
                    // Tổng lương = lương theo ngày + phụ cấp + lương tăng ca
                    double tongLuong = luongTheoNgay + phuCap + luongTangCa; // Tính tổng lương
                    
                    // Kiểm tra xem đã có bản ghi lương chưa
                    String checkQuery = "SELECT MaLuong FROM " + TABLE_LUONG + // Query kiểm tra đã có bản ghi lương chưa
                                      " WHERE MaNhanVien = ? AND ThangNam = ?"; // WHERE theo mã nhân viên và tháng năm
                    Cursor checkCursor = db.rawQuery(checkQuery, new String[]{maNhanVien, thangNam}); // Thực hiện query kiểm tra
                    
                    ContentValues values = new ContentValues();           // Tạo ContentValues để chứa dữ liệu lương
                    values.put("MaNhanVien", maNhanVien);                  // Thêm mã nhân viên
                    values.put("ThangNam", thangNam);                      // Thêm tháng năm
                    values.put("LuongCoBan", luongTheoNgay);               // Thêm lương cơ bản theo ngày thực tế
                    values.put("PhuCap", phuCap);                          // Thêm phụ cấp
                    values.put("SoGioLam", stats.soGioLam);                // Thêm số giờ làm
                    values.put("TongLuong", tongLuong);                    // Thêm tổng lương
                    values.put("TrangThai", "Chưa thanh toán");            // Set trạng thái mặc định
                    values.put("NgayTinhLuong", java.time.LocalDate.now().toString()); // Thêm ngày tính lương
                    
                    if (checkCursor != null && checkCursor.moveToFirst()) { // Nếu đã có bản ghi lương
                        // Cập nhật nếu đã tồn tại
                        int maLuong = checkCursor.getInt(0);               // Lấy mã lương
                        db.update(TABLE_LUONG, values, "MaLuong = ?", new String[]{String.valueOf(maLuong)}); // Update bản ghi hiện có
                    } else {                                               // Nếu chưa có bản ghi lương
                        // Thêm mới nếu chưa tồn tại
                        db.insert(TABLE_LUONG, null, values);             // Insert bản ghi mới
                    }
                    
                    if (checkCursor != null) checkCursor.close();          // Đóng cursor kiểm tra
                    count++;                                               // Tăng biến đếm
                    
                } while (cursor.moveToNext());                             // Lặp đến nhân viên tiếp theo
                cursor.close();                                            // Đóng cursor chính
            }
        } catch (Exception e) {                                           // Catch exception
            e.printStackTrace();                                          // In stack trace
        }
        
        return count;                                                     // Trả về số nhân viên được tính lương
    }
```

### Method getAttendanceStats:
```java
    public AttendanceStats getAttendanceStatsForSalary(String maNhanVien, String thangNam) { // Method wrapper để lấy thống kê chấm công cho tính lương
        return getAttendanceStats(maNhanVien, thangNam);                  // Gọi method getAttendanceStats
    }
    
    private AttendanceStats getAttendanceStats(String maNhanVien, String thangNam) { // Method lấy thống kê chấm công của nhân viên trong tháng
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        AttendanceStats stats = new AttendanceStats();                     // Tạo object AttendanceStats
        
        try {
            // Sửa query để lấy từng dòng chấm công thay vì dùng COUNT(*)
            String query = "SELECT SoGioLam FROM " + TABLE_CHAM_CONG +     // Query lấy số giờ làm từng ngày
                          " WHERE MaNhanVien = ? AND strftime('%Y-%m', NgayChamCong) = ? " + // WHERE theo mã nhân viên và tháng năm (format YYYY-MM)
                          " AND SoGioLam > 0";                             // AND số giờ làm > 0
            Cursor cursor = db.rawQuery(query, new String[]{maNhanVien, thangNam}); // Thực hiện query
            
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                do {
                    double gioLam = cursor.getDouble(0);                   // Lấy số giờ làm trong ngày
                    stats.soGioLam += gioLam;                              // Cộng vào tổng số giờ làm
                    stats.soNgayLam++;                                     // Tăng số ngày làm
                    
                    // Tính giờ tăng ca (> 8 giờ/ngày)
                    if (gioLam > 8.0) {                                    // Nếu làm > 8 giờ/ngày
                        stats.soGioTangCa += (gioLam - 8.0);               // Cộng giờ tăng ca vào tổng
                    }
                } while (cursor.moveToNext());                             // Lặp đến record tiếp theo
                cursor.close();                                            // Đóng cursor
            }
        } catch (Exception e) {                                           // Catch exception
            e.printStackTrace();                                          // In stack trace
        }
        
        return stats;                                                     // Trả về object thống kê
    }
```

### Method getTotalWorkingHours:
```java
    private double getTotalWorkingHours(String maNhanVien, String thangNam) { // Method lấy tổng số giờ làm trong tháng
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        double totalHours = 0;                                             // Biến lưu tổng số giờ
        
        try {
            String query = "SELECT SUM(SoGioLam) FROM " + TABLE_CHAM_CONG + // Query SUM số giờ làm
                          " WHERE MaNhanVien = ? AND strftime('%Y-%m', NgayChamCong) = ?"; // WHERE theo mã nhân viên và tháng năm
            Cursor cursor = db.rawQuery(query, new String[]{maNhanVien, thangNam}); // Thực hiện query
            
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                totalHours = cursor.getDouble(0);                          // Lấy tổng số giờ
                cursor.close();                                            // Đóng cursor
            }
        } catch (Exception e) {                                           // Catch exception
            e.printStackTrace();                                          // In stack trace
        }
        
        return totalHours;                                                // Trả về tổng số giờ
    }
```

### Salary Query Methods:
```java
    public Cursor getSalaryByMonth(String thangNam) {                      // Method lấy lương theo tháng
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT * FROM " + TABLE_LUONG +                    // Query SELECT tất cả columns
                      " WHERE ThangNam = ? ORDER BY MaNhanVien";           // WHERE theo tháng năm, ORDER BY mã nhân viên
        return db.rawQuery(query, new String[]{thangNam});                // Thực hiện query với parameter
    }

    public Cursor getSalaryByEmployee(String maNhanVien, String thangNam) { // Method lấy lương của một nhân viên trong tháng
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT * FROM " + TABLE_LUONG +                    // Query SELECT tất cả columns
                      " WHERE MaNhanVien = ? AND ThangNam = ?";            // WHERE theo mã nhân viên và tháng năm
        return db.rawQuery(query, new String[]{maNhanVien, thangNam});    // Thực hiện query với parameters
    }

    public boolean updateSalaryStatus(int maLuong, String trangThai) {     // Method cập nhật trạng thái thanh toán lương
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để chứa dữ liệu cập nhật
        values.put("TrangThai", trangThai);                               // Cập nhật trạng thái
        
        int result = db.update(TABLE_LUONG, values,                       // Update record
                             "MaLuong = ?", new String[]{String.valueOf(maLuong)}); // WHERE theo mã lương
        return result > 0;                                                // Trả về true nếu có record được cập nhật
    }
```
---

## 9️⃣ DEPARTMENT & POSITION MANAGEMENT METHODS

### Department Management Methods:
```java
    public Cursor getAllDepartments() {                                    // Method lấy tất cả phòng ban đang hoạt động
        return getReadableDatabase().rawQuery("SELECT * FROM " + TABLE_PHONG_BAN + " WHERE TrangThai = 1", null); // Query SELECT WHERE trạng thái = 1 (active)
    }

    public Cursor getAllDepartmentsWithDetails() {                         // Method lấy tất cả phòng ban với thông tin chi tiết
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT pb.*, " +                                   // Query SELECT tất cả columns từ PhongBan
                      "COALESCE(nv.HoTen, '') as TenTruongPhong, " +       // COALESCE để lấy tên trưởng phòng hoặc chuỗi rỗng
                      "COUNT(nv2.MaNhanVien) as SoNhanVien " +             // COUNT số nhân viên trong phòng ban
                      "FROM " + TABLE_PHONG_BAN + " pb " +                 // FROM bảng PhongBan với alias pb
                      "LEFT JOIN " + TABLE_NHAN_VIEN + " nv ON pb.TruongPhong = nv.MaNhanVien " + // LEFT JOIN để lấy thông tin trưởng phòng
                      "LEFT JOIN " + TABLE_NHAN_VIEN + " nv2 ON pb.MaPhongBan = nv2.MaPhongBan AND nv2.TrangThaiLamViec = 'Đang làm việc' " + // LEFT JOIN để đếm nhân viên đang làm việc
                      "WHERE pb.TrangThai = 1 " +                          // WHERE phòng ban đang hoạt động
                      "GROUP BY pb.MaPhongBan " +                          // GROUP BY mã phòng ban
                      "ORDER BY pb.MaPhongBan";                            // ORDER BY mã phòng ban
        return db.rawQuery(query, null);                                  // Thực hiện query
    }

    public Cursor searchDepartments(String keyword) {                      // Method tìm kiếm phòng ban theo keyword
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT pb.*, " +                                   // Query SELECT với JOIN và search
                      "COALESCE(nv.HoTen, '') as TenTruongPhong, " +       // COALESCE để lấy tên trưởng phòng
                      "COUNT(nv2.MaNhanVien) as SoNhanVien " +             // COUNT số nhân viên
                      "FROM " + TABLE_PHONG_BAN + " pb " +                 // FROM bảng PhongBan
                      "LEFT JOIN " + TABLE_NHAN_VIEN + " nv ON pb.TruongPhong = nv.MaNhanVien " + // LEFT JOIN trưởng phòng
                      "LEFT JOIN " + TABLE_NHAN_VIEN + " nv2 ON pb.MaPhongBan = nv2.MaPhongBan AND nv2.TrangThaiLamViec = 'Đang làm việc' " + // LEFT JOIN nhân viên
                      "WHERE pb.TrangThai = 1 AND (pb.TenPhongBan LIKE ? OR pb.MaPhongBan LIKE ? OR nv.HoTen LIKE ?) " + // WHERE với search conditions
                      "GROUP BY pb.MaPhongBan " +                          // GROUP BY mã phòng ban
                      "ORDER BY pb.MaPhongBan";                            // ORDER BY mã phòng ban
        String wildcardKeyword = "%" + keyword + "%";                      // Thêm wildcard % vào keyword
        return db.rawQuery(query, new String[]{wildcardKeyword, wildcardKeyword, wildcardKeyword}); // Thực hiện query với 3 parameters
    }

    public String getNextDepartmentCode() {                                // Method tự động tạo mã phòng ban tiếp theo
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT MaPhongBan FROM " + TABLE_PHONG_BAN +       // Query lấy mã phòng ban lớn nhất
                      " WHERE MaPhongBan LIKE 'PB%' " +                    // WHERE mã phòng ban có format PBxxx
                      " ORDER BY length(MaPhongBan) DESC, MaPhongBan DESC LIMIT 1"; // ORDER BY độ dài và giá trị giảm dần, LIMIT 1
        Cursor cursor = db.rawQuery(query, null);                         // Thực hiện query
        String lastCode = null;                                            // Biến lưu mã cuối cùng
        if (cursor != null) {                                              // Nếu cursor không null
            if (cursor.moveToFirst()) lastCode = cursor.getString(0).trim(); // Lấy mã đầu tiên và trim
            cursor.close();                                                // Đóng cursor
        }
        if (lastCode == null) return "PB001";                             // Nếu không có mã nào thì trả về PB001
        
        // Parse mã để tăng số thứ tự (tương tự getNextEmployeeCode)
        java.util.regex.Matcher matcher = java.util.regex.Pattern.compile("^([a-zA-Z]+)(\\d+)$").matcher(lastCode); // Regex pattern
        if (matcher.find()) {                                              // Nếu match được pattern
            String prefix = matcher.group(1);                             // Lấy prefix (PB)
            String numberStr = matcher.group(2);                          // Lấy phần số
            try {
                int nextNumber = Integer.parseInt(numberStr) + 1;         // Tăng số lên 1
                return String.format("%s%0" + numberStr.length() + "d", prefix, nextNumber); // Format với leading zeros
            } catch (Exception e) { return "PB001"; }                     // Nếu có lỗi parse thì trả về PB001
        }
        return "PB001";                                                    // Default return PB001
    }

    public boolean addDepartment(String maPhongBan, String tenPhongBan, String truongPhong, int trangThai) { // Method thêm phòng ban mới
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để chứa dữ liệu
        values.put("MaPhongBan", maPhongBan);                             // Thêm mã phòng ban
        values.put("TenPhongBan", tenPhongBan);                           // Thêm tên phòng ban
        values.put("TruongPhong", truongPhong);                           // Thêm trưởng phòng
        values.put("TrangThai", trangThai);                               // Thêm trạng thái
        
        long result = db.insert(TABLE_PHONG_BAN, null, values);           // Insert vào bảng PhongBan
        return result != -1;                                              // Trả về true nếu insert thành công
    }

    public boolean updateDepartment(String maPhongBan, String tenPhongBan, String truongPhong, int trangThai) { // Method cập nhật thông tin phòng ban
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để chứa dữ liệu cập nhật
        values.put("TenPhongBan", tenPhongBan);                           // Cập nhật tên phòng ban
        values.put("TruongPhong", truongPhong);                           // Cập nhật trưởng phòng
        values.put("TrangThai", trangThai);                               // Cập nhật trạng thái
        
        int result = db.update(TABLE_PHONG_BAN, values,                   // Update record
                             "MaPhongBan = ?", new String[]{maPhongBan}); // WHERE theo mã phòng ban
        return result > 0;                                                // Trả về true nếu có record được cập nhật
    }

    public boolean deleteDepartment(String maPhongBan) {                   // Method xóa phòng ban (soft delete)
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để cập nhật trạng thái
        values.put("TrangThai", 0);                                       // Set trạng thái = 0 (ngừng hoạt động) thay vì xóa
        
        int result = db.update(TABLE_PHONG_BAN, values,                   // Update record
                             "MaPhongBan = ?", new String[]{maPhongBan}); // WHERE theo mã phòng ban
        return result > 0;                                                // Trả về true nếu có record được cập nhật
    }
```

### Position Management Methods:
```java
    public Cursor getAllPositions() {                                      // Method lấy tất cả chức vụ đang hoạt động
        return getReadableDatabase().rawQuery("SELECT * FROM " + TABLE_CHUC_VU + " WHERE TrangThai = 1", null); // Query SELECT WHERE trạng thái = 1
    }

    public Cursor getAllPositionsWithDetails() {                           // Method lấy tất cả chức vụ với thông tin chi tiết
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT cv.*, " +                                   // Query SELECT tất cả columns từ ChucVu
                      "COUNT(nv.MaNhanVien) as SoNhanVien " +              // COUNT số nhân viên có chức vụ này
                      "FROM " + TABLE_CHUC_VU + " cv " +                   // FROM bảng ChucVu với alias cv
                      "LEFT JOIN " + TABLE_NHAN_VIEN + " nv ON cv.MaChucVu = nv.MaChucVu AND nv.TrangThaiLamViec = 'Đang làm việc' " + // LEFT JOIN để đếm nhân viên đang làm việc
                      "WHERE cv.TrangThai = 1 " +                          // WHERE chức vụ đang hoạt động
                      "GROUP BY cv.MaChucVu " +                            // GROUP BY mã chức vụ
                      "ORDER BY cv.MaChucVu";                              // ORDER BY mã chức vụ
        return db.rawQuery(query, null);                                  // Thực hiện query
    }

    public Cursor searchPositions(String keyword) {                        // Method tìm kiếm chức vụ theo keyword
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT cv.*, " +                                   // Query SELECT với search
                      "COUNT(nv.MaNhanVien) as SoNhanVien " +              // COUNT số nhân viên
                      "FROM " + TABLE_CHUC_VU + " cv " +                   // FROM bảng ChucVu
                      "LEFT JOIN " + TABLE_NHAN_VIEN + " nv ON cv.MaChucVu = nv.MaChucVu AND nv.TrangThaiLamViec = 'Đang làm việc' " + // LEFT JOIN nhân viên
                      "WHERE cv.TrangThai = 1 AND (cv.TenChucVu LIKE ? OR cv.MaChucVu LIKE ?) " + // WHERE với search conditions
                      "GROUP BY cv.MaChucVu " +                            // GROUP BY mã chức vụ
                      "ORDER BY cv.MaChucVu";                              // ORDER BY mã chức vụ
        String wildcardKeyword = "%" + keyword + "%";                      // Thêm wildcard % vào keyword
        return db.rawQuery(query, new String[]{wildcardKeyword, wildcardKeyword}); // Thực hiện query với 2 parameters
    }

    public String getNextPositionCode() {                                  // Method tự động tạo mã chức vụ tiếp theo
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT MaChucVu FROM " + TABLE_CHUC_VU +           // Query lấy mã chức vụ lớn nhất
                      " WHERE MaChucVu LIKE 'CV%' " +                      // WHERE mã chức vụ có format CVxxx
                      " ORDER BY length(MaChucVu) DESC, MaChucVu DESC LIMIT 1"; // ORDER BY độ dài và giá trị giảm dần, LIMIT 1
        Cursor cursor = db.rawQuery(query, null);                         // Thực hiện query
        String lastCode = null;                                            // Biến lưu mã cuối cùng
        if (cursor != null) {                                              // Nếu cursor không null
            if (cursor.moveToFirst()) lastCode = cursor.getString(0).trim(); // Lấy mã đầu tiên và trim
            cursor.close();                                                // Đóng cursor
        }
        if (lastCode == null) return "CV001";                             // Nếu không có mã nào thì trả về CV001
        
        // Parse mã để tăng số thứ tự (tương tự các method khác)
        java.util.regex.Matcher matcher = java.util.regex.Pattern.compile("^([a-zA-Z]+)(\\d+)$").matcher(lastCode); // Regex pattern
        if (matcher.find()) {                                              // Nếu match được pattern
            String prefix = matcher.group(1);                             // Lấy prefix (CV)
            String numberStr = matcher.group(2);                          // Lấy phần số
            try {
                int nextNumber = Integer.parseInt(numberStr) + 1;         // Tăng số lên 1
                return String.format("%s%0" + numberStr.length() + "d", prefix, nextNumber); // Format với leading zeros
            } catch (Exception e) { return "CV001"; }                     // Nếu có lỗi parse thì trả về CV001
        }
        return "CV001";                                                    // Default return CV001
    }

    public boolean addPosition(String maChucVu, String tenChucVu, double mucLuongCoBan, int trangThai) { // Method thêm chức vụ mới
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để chứa dữ liệu
        values.put("MaChucVu", maChucVu);                                 // Thêm mã chức vụ
        values.put("TenChucVu", tenChucVu);                               // Thêm tên chức vụ
        values.put("MucLuongCoBan", mucLuongCoBan);                       // Thêm mức lương cơ bản
        values.put("TrangThai", trangThai);                               // Thêm trạng thái
        
        long result = db.insert(TABLE_CHUC_VU, null, values);             // Insert vào bảng ChucVu
        return result != -1;                                              // Trả về true nếu insert thành công
    }

    public boolean updatePosition(String maChucVu, String tenChucVu, double mucLuongCoBan, int trangThai) { // Method cập nhật thông tin chức vụ
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để chứa dữ liệu cập nhật
        values.put("TenChucVu", tenChucVu);                               // Cập nhật tên chức vụ
        values.put("MucLuongCoBan", mucLuongCoBan);                       // Cập nhật mức lương cơ bản
        values.put("TrangThai", trangThai);                               // Cập nhật trạng thái
        
        int result = db.update(TABLE_CHUC_VU, values,                     // Update record
                             "MaChucVu = ?", new String[]{maChucVu});     // WHERE theo mã chức vụ
        return result > 0;                                                // Trả về true nếu có record được cập nhật
    }

    public boolean deletePosition(String maChucVu) {                       // Method xóa chức vụ (soft delete)
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để cập nhật trạng thái
        values.put("TrangThai", 0);                                       // Set trạng thái = 0 (ngừng hoạt động) thay vì xóa
        
        int result = db.update(TABLE_CHUC_VU, values,                     // Update record
                             "MaChucVu = ?", new String[]{maChucVu});     // WHERE theo mã chức vụ
        return result > 0;                                                // Trả về true nếu có record được cập nhật
    }
```
---

## 🔟 CONTRACT & ACCOUNT MANAGEMENT METHODS

### Contract Management Methods:
```java
    public Cursor getAllHopDong() {                                        // Method lấy tất cả hợp đồng
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        return db.rawQuery("SELECT * FROM " + TABLE_HOP_DONG + " ORDER BY MaHopDong DESC", null); // Query SELECT ORDER BY mã hợp đồng giảm dần
    }

    public Cursor searchHopDong(String query) {                            // Method tìm kiếm hợp đồng theo keyword
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        return db.rawQuery("SELECT * FROM " + TABLE_HOP_DONG + " WHERE MaHopDong LIKE ? OR MaNhanVien LIKE ?", // Query search theo mã hợp đồng hoặc mã nhân viên
                new String[]{"%" + query + "%", "%" + query + "%"});      // Parameters với wildcard
    }

    public boolean insertHopDong(String maHD, String maNV, String loaiHD, String ngayBD, String ngayKT, double mucLuong) { // Method thêm hợp đồng mới
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues cv = new ContentValues();                           // Tạo ContentValues để chứa dữ liệu
        cv.put("MaHopDong", maHD);                                         // Thêm mã hợp đồng
        cv.put("MaNhanVien", maNV);                                        // Thêm mã nhân viên
        cv.put("LoaiHopDong", loaiHD);                                     // Thêm loại hợp đồng
        cv.put("NgayBatDau", ngayBD);                                      // Thêm ngày bắt đầu
        cv.put("NgayKetThuc", ngayKT);                                     // Thêm ngày kết thúc
        cv.put("MucLuong", mucLuong);                                      // Thêm mức lương
        cv.put("TrangThai", "Hiệu lực");                                   // Set trạng thái mặc định
        long result = db.insert(TABLE_HOP_DONG, null, cv);                // Insert vào bảng HopDongLaoDong
        return result != -1;                                              // Trả về true nếu insert thành công
    }

    public boolean updateHopDong(String maHD, String maNV, String loaiHD, String ngayBD, String ngayKT, double mucLuong, String trangThai) { // Method cập nhật hợp đồng
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues cv = new ContentValues();                           // Tạo ContentValues để chứa dữ liệu cập nhật
        cv.put("MaNhanVien", maNV);                                        // Cập nhật mã nhân viên
        cv.put("LoaiHopDong", loaiHD);                                     // Cập nhật loại hợp đồng
        cv.put("NgayBatDau", ngayBD);                                      // Cập nhật ngày bắt đầu
        cv.put("NgayKetThuc", ngayKT);                                     // Cập nhật ngày kết thúc
        cv.put("MucLuong", mucLuong);                                      // Cập nhật mức lương
        cv.put("TrangThai", trangThai);                                    // Cập nhật trạng thái
        int result = db.update(TABLE_HOP_DONG, cv, "MaHopDong = ?", new String[]{maHD}); // Update WHERE mã hợp đồng
        return result > 0;                                                // Trả về true nếu có record được cập nhật
    }

    public boolean deleteHopDong(String maHD) {                            // Method xóa hợp đồng (hard delete)
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        int result = db.delete(TABLE_HOP_DONG, "MaHopDong = ?", new String[]{maHD}); // Delete WHERE mã hợp đồng
        return result > 0;                                                // Trả về true nếu có record được xóa
    }

    public Cursor getHopDongById(String maHD) {                            // Method lấy hợp đồng theo mã
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        return db.rawQuery("SELECT * FROM " + TABLE_HOP_DONG + " WHERE MaHopDong = ?", new String[]{maHD}); // Query SELECT WHERE mã hợp đồng
    }

    public Cursor getEmployeeListForSpinner() {                            // Method lấy danh sách nhân viên cho Spinner
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        return db.rawQuery("SELECT MaNhanVien as _id, MaNhanVien || ' - ' || HoTen as DisplayName FROM " + TABLE_NHAN_VIEN, null); // Query SELECT với format hiển thị
    }

    public double getSalaryByEmployee(String maNV) {                       // Method lấy mức lương của nhân viên từ chức vụ
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT cv.MucLuongCoBan FROM " + TABLE_NHAN_VIEN + " nv " + // Query JOIN để lấy mức lương cơ bản
                "JOIN " + TABLE_CHUC_VU + " cv ON nv.MaChucVu = cv.MaChucVu " + // JOIN với bảng ChucVu
                "WHERE nv.MaNhanVien = ?";                                 // WHERE theo mã nhân viên
        Cursor cursor = db.rawQuery(query, new String[]{maNV});           // Thực hiện query
        double salary = 0;                                                 // Biến lưu mức lương
        if (cursor.moveToFirst()) {                                        // Nếu có dữ liệu
            salary = cursor.getDouble(0);                                  // Lấy mức lương
        }
        cursor.close();                                                    // Đóng cursor
        return salary;                                                     // Trả về mức lương
    }
```

### Account Management Methods:
```java
    public Cursor getAllAccounts() {                                       // Method lấy tất cả tài khoản với thông tin nhân viên
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT tk.*, nv.HoTen FROM " + TABLE_TAI_KHOAN + " tk " + // Query JOIN để lấy thông tin tài khoản và tên nhân viên
                      "LEFT JOIN " + TABLE_NHAN_VIEN + " nv ON tk.MaNhanVien = nv.MaNhanVien " + // LEFT JOIN với bảng NhanVien
                      "ORDER BY tk.TenDangNhap";                           // ORDER BY tên đăng nhập
        return db.rawQuery(query, null);                                  // Thực hiện query
    }

    public boolean deleteAccount(String username) {                        // Method xóa tài khoản (hard delete)
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        // Không cho phép xóa tài khoản admin chính
        if ("admin".equals(username)) return false;                       // Bảo vệ tài khoản admin chính
        
        int result = db.delete(TABLE_TAI_KHOAN, "TenDangNhap = ?", new String[]{username}); // Delete WHERE tên đăng nhập
        return result > 0;                                                // Trả về true nếu có record được xóa
    }

    public boolean updateAccountStatus(String username, int status) {      // Method cập nhật trạng thái tài khoản
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để chứa dữ liệu cập nhật
        values.put("TrangThai", status);                                   // Cập nhật trạng thái (1 = active, 0 = inactive)
        int result = db.update(TABLE_TAI_KHOAN, values, "TenDangNhap = ?", new String[]{username}); // Update WHERE tên đăng nhập
        return result > 0;                                                // Trả về true nếu có record được cập nhật
    }

    public boolean updateAccountInfo(String username, String fullName, String role) { // Method cập nhật thông tin tài khoản và nhân viên
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        db.beginTransaction();                                             // Bắt đầu transaction để đảm bảo tính toàn vẹn
        try {
            // 1. Cập nhật VaiTro trong bảng TaiKhoan
            ContentValues accountValues = new ContentValues();            // Tạo ContentValues cho bảng TaiKhoan
            accountValues.put("VaiTro", role);                            // Cập nhật vai trò
            db.update(TABLE_TAI_KHOAN, accountValues, "TenDangNhap = ?", new String[]{username}); // Update vai trò

            // 2. Tìm MaNhanVien liên kết với tài khoản này
            String maNV = "";                                              // Biến lưu mã nhân viên
            Cursor cursor = db.rawQuery("SELECT MaNhanVien FROM " + TABLE_TAI_KHOAN + " WHERE TenDangNhap = ?", new String[]{username}); // Query lấy mã nhân viên
            if (cursor.moveToFirst()) {                                    // Nếu có dữ liệu
                maNV = cursor.getString(0);                                // Lấy mã nhân viên
            }
            cursor.close();                                                // Đóng cursor

            // 3. Cập nhật HoTen trong bảng NhanVien
            if (!maNV.isEmpty()) {                                         // Nếu có mã nhân viên
                ContentValues employeeValues = new ContentValues();       // Tạo ContentValues cho bảng NhanVien
                employeeValues.put("HoTen", fullName);                     // Cập nhật họ tên
                db.update(TABLE_NHAN_VIEN, employeeValues, "MaNhanVien = ?", new String[]{maNV}); // Update họ tên
            }

            db.setTransactionSuccessful();                                 // Đánh dấu transaction thành công
            return true;                                                   // Trả về true nếu thành công
        } catch (Exception e) {                                           // Catch exception
            e.printStackTrace();                                          // In stack trace
            return false;                                                 // Trả về false nếu có lỗi
        } finally {
            db.endTransaction();                                          // Kết thúc transaction
        }
    }
```

### Personal Information Methods:
```java
    public Cursor getEmployeeByMa(String maNhanVien) {                     // Method lấy thông tin nhân viên theo mã với JOIN
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT nv.*, cv.TenChucVu, pb.TenPhongBan FROM " + TABLE_NHAN_VIEN + " nv " + // Query JOIN 3 bảng
                      "LEFT JOIN " + TABLE_CHUC_VU + " cv ON nv.MaChucVu = cv.MaChucVu " + // LEFT JOIN với bảng ChucVu
                      "LEFT JOIN " + TABLE_PHONG_BAN + " pb ON nv.MaPhongBan = pb.MaPhongBan " + // LEFT JOIN với bảng PhongBan
                      "WHERE nv.MaNhanVien = ?";                           // WHERE theo mã nhân viên
        return db.rawQuery(query, new String[]{maNhanVien});              // Thực hiện query với parameter
    }

    public boolean updateEmployeePersonalInfo(String maNhanVien, String hoTen, String ngaySinh, // Method cập nhật thông tin cá nhân nhân viên
                                             String gioiTinh, String soDienThoai, String email, String hinhAnh) {
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để chứa dữ liệu cập nhật
        values.put("HoTen", hoTen);                                        // Cập nhật họ tên
        values.put("NgaySinh", ngaySinh);                                  // Cập nhật ngày sinh
        values.put("GioiTinh", gioiTinh);                                  // Cập nhật giới tính
        values.put("SoDienThoai", soDienThoai);                            // Cập nhật số điện thoại
        values.put("Email", email);                                        // Cập nhật email
        if (hinhAnh != null) {                                             // Nếu có hình ảnh
            values.put("HinhAnh", hinhAnh);                                // Cập nhật đường dẫn hình ảnh
        }
        
        int result = db.update(TABLE_NHAN_VIEN, values,                   // Update record
                             "MaNhanVien = ?", new String[]{maNhanVien}); // WHERE theo mã nhân viên
        return result > 0;                                                // Trả về true nếu có record được cập nhật
    }

    public String getEmployeeNameByMa(String maNhanVien) {                 // Method lấy tên nhân viên theo mã
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT HoTen FROM " + TABLE_NHAN_VIEN + " WHERE MaNhanVien = ?"; // Query SELECT họ tên
        Cursor cursor = db.rawQuery(query, new String[]{maNhanVien});     // Thực hiện query
        String hoTen = null;                                               // Biến lưu họ tên
        if (cursor != null && cursor.moveToFirst()) {                      // Nếu cursor không null và có dữ liệu
            hoTen = cursor.getString(0);                                   // Lấy họ tên
            cursor.close();                                                // Đóng cursor
        }
        return hoTen;                                                      // Trả về họ tên
    }

    public Cursor getManagerCandidates() {                                 // Method lấy danh sách ứng viên làm trưởng phòng
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT nv.*, cv.TenChucVu FROM " + TABLE_NHAN_VIEN + " nv " + // Query JOIN để lấy nhân viên và chức vụ
                      "LEFT JOIN " + TABLE_CHUC_VU + " cv ON nv.MaChucVu = cv.MaChucVu " + // LEFT JOIN với bảng ChucVu
                      "WHERE nv.TrangThaiLamViec = 'Đang làm việc' " +     // WHERE nhân viên đang làm việc
                      "AND (cv.TenChucVu LIKE '%Trưởng phòng%' OR cv.TenChucVu LIKE '%Giám đốc%') " + // AND chức vụ là trưởng phòng hoặc giám đốc
                      "ORDER BY nv.HoTen";                                 // ORDER BY họ tên
        return db.rawQuery(query, null);                                  // Thực hiện query
    }
}
```

---

## 🔗 TÍCH HỢP VÀ KIẾN TRÚC

### 1. Database Architecture:
- **Single Source of Truth**: DatabaseHelper là class duy nhất quản lý tất cả database operations
- **Centralized Management**: Tất cả CRUD operations được tập trung trong một class
- **Version Control**: Quản lý phiên bản database với migration strategy
- **Data Integrity**: Sử dụng foreign keys và constraints để đảm bảo tính toàn vẹn

### 2. Business Logic Integration:
- **Employee Lifecycle**: Từ registration, management đến termination
- **Attendance Workflow**: Check-in/out với automatic calculation
- **Leave Management**: Complete approval workflow
- **Salary Calculation**: Automatic calculation based on attendance data

### 3. Security Implementation:
- **Parameterized Queries**: Tất cả queries sử dụng parameters để tránh SQL injection
- **Role-based Data Access**: Methods kiểm tra permissions dựa trên role
- **Transaction Safety**: Sử dụng transactions cho operations quan trọng
- **Admin Protection**: Bảo vệ tài khoản admin khỏi bị xóa

---

## 📋 QUY TẮC NGHIỆP VỤ

### 1. Data Management Rules:
- **Soft Delete**: Phòng ban và chức vụ sử dụng soft delete (set TrangThai = 0)
- **Hard Delete**: Hợp đồng và tài khoản sử dụng hard delete
- **Auto Generation**: Mã nhân viên, phòng ban, chức vụ được tự động tạo
- **Unique Constraints**: Đảm bảo tính duy nhất của các key fields

### 2. Business Logic Rules:
- **Employee Status**: Chỉ nhân viên "Đang làm việc" được tính trong statistics
- **Salary Calculation**: Dựa trên attendance data và contract information
- **Leave Approval**: Workflow với 3 trạng thái (Chờ duyệt, Đã duyệt, Từ chối)
- **Attendance Tracking**: Unique constraint (MaNhanVien, NgayChamCong)

### 3. Security Rules:
- **Admin Protection**: Tài khoản admin không thể bị xóa
- **Password Policy**: Plain text passwords (theo yêu cầu đơn giản)
- **Role Management**: 4 roles (Admin, HR, Manager, Employee)
- **Data Access Control**: Methods kiểm tra permissions theo role

---

## 🔒 BẢO MẬT VÀ PERFORMANCE

### 1. Security Measures:
- **SQL Injection Prevention**: Tất cả queries sử dụng parameterized statements
- **Input Validation**: Validation data trước khi insert/update
- **Transaction Management**: Rollback khi có lỗi để đảm bảo data consistency
- **Access Control**: Role-based method access

### 2. Performance Optimization:
- **Efficient Queries**: Sử dụng JOIN thay vì multiple queries
- **Proper Indexing**: Primary keys và unique constraints
- **Cursor Management**: Đóng cursor sau khi sử dụng
- **Connection Reuse**: Reuse database connections

### 3. Error Handling:
- **Exception Catching**: Try-catch blocks cho tất cả database operations
- **Graceful Degradation**: Return false/null khi có lỗi
- **Logging**: printStackTrace() để debug
- **Transaction Rollback**: Automatic rollback khi có exception

---

## 🎯 KẾT LUẬN

DatabaseHelper class là backbone của hệ thống QLNS, cung cấp:

### Tính năng chính:
- **Complete CRUD Operations**: Đầy đủ operations cho 8 bảng chính
- **Business Logic Integration**: Tích hợp logic nghiệp vụ phức tạp
- **Data Integrity**: Đảm bảo tính toàn vẹn và nhất quán dữ liệu
- **Security Implementation**: Bảo mật toàn diện với parameterized queries

### Ưu điểm:
- **Centralized Management**: Quản lý tập trung tất cả database operations
- **Scalable Architecture**: Có thể mở rộng dễ dàng
- **Maintainable Code**: Code được tổ chức rõ ràng, dễ bảo trì
- **Performance Optimized**: Queries được tối ưu cho performance

### Vai trò trong hệ thống:
- **Data Layer**: Lớp dữ liệu chính của ứng dụng
- **Business Logic**: Chứa logic nghiệp vụ quan trọng
- **Security Gateway**: Kiểm soát truy cập dữ liệu
- **Integration Hub**: Kết nối tất cả modules với database

Class này đóng vai trò quan trọng nhất trong kiến trúc hệ thống, đảm bảo tất cả operations với database được thực hiện một cách an toàn, hiệu quả và nhất quán.