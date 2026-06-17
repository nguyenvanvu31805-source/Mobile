# HƯỚNG DẪN CHI TIẾT: THỐNG KÊ MODULE

## 📋 TỔNG QUAN

Module Thống kê cung cấp báo cáo tổng quan về tình hình hoạt động của hệ thống QLNS. Module này hiển thị các chỉ số quan trọng về nhân viên, tổ chức, chấm công, nghỉ phép và lương theo tháng/năm được chọn, giúp quản lý có cái nhìn toàn diện về hiệu quả hoạt động.

## 🏗️ KIẾN TRÚC MODULE

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/
└── ThongKeActivity.java                    # Main statistics activity class

app/src/main/res/layout/
└── activity_thong_ke.xml                   # Statistics UI layout
```

### Thành phần chính:
```
Statistics Module
├── Employee Statistics                     # Thống kê nhân viên
├── Organization Statistics                 # Thống kê tổ chức (phòng ban, chức vụ)
├── Attendance Statistics                   # Thống kê chấm công
├── Leave Request Statistics                # Thống kê nghỉ phép
├── Salary Statistics                       # Thống kê lương
└── Time Period Selection                   # Chọn tháng/năm thống kê
```

## 📊 NGHIỆP VỤ THỐNG KÊ

### 1. Employee Statistics:
- **Total Employees**: Tổng số nhân viên trong hệ thống
- **Active Employees**: Số nhân viên đang làm việc
- **Inactive Employees**: Số nhân viên đã nghỉ việc
- **Employee Status Tracking**: Theo dõi trạng thái nhân viên

### 2. Organization Statistics:
- **Department Count**: Số lượng phòng ban đang hoạt động
- **Position Count**: Số lượng chức vụ đang có hiệu lực
- **Organizational Structure**: Cấu trúc tổ chức tổng quan

### 3. Attendance Statistics:
- **Monthly Attendance**: Số lần chấm công trong tháng
- **Attendance Rate**: Tỷ lệ đi làm của nhân viên
- **Time-based Analysis**: Phân tích theo thời gian

### 4. Leave Request Statistics:
- **Total Requests**: Tổng số đơn nghỉ phép trong tháng
- **Pending Requests**: Số đơn chờ duyệt
- **Approved Requests**: Số đơn đã được duyệt
- **Request Status Tracking**: Theo dõi trạng thái đơn nghỉ phép

### 5. Salary Statistics:
- **Total Monthly Salary**: Tổng lương tháng của toàn công ty
- **Average Salary**: Lương trung bình của nhân viên
- **Salary Distribution**: Phân bổ lương theo tháng

---

## 📱 CHI TIẾT CLASS IMPLEMENTATION

## 1️⃣ CLASS DECLARATION & IMPORTS

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/ThongKeActivity.java`

### Mục đích:
Activity chính quản lý hiển thị thống kê tổng quan với các chỉ số quan trọng của hệ thống QLNS.

### Chi tiết code:

#### Package và Import declarations:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chính của ứng dụng

import android.database.Cursor;                                            // Import Cursor để xử lý dữ liệu từ database
import android.os.Bundle;                                                  // Import Bundle để truyền dữ liệu
import android.view.View;                                                  // Import View để xử lý UI components
import android.widget.AdapterView;                                         // Import AdapterView để xử lý Spinner events
import android.widget.ArrayAdapter;                                        // Import ArrayAdapter cho Spinner
import android.widget.Spinner;                                             // Import Spinner widget
import android.widget.TextView;                                            // Import TextView widget
import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity làm base class
import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để truy cập database
import java.util.ArrayList;                                                // Import ArrayList để quản lý danh sách
import java.util.Calendar;                                                 // Import Calendar để xử lý thời gian
import java.util.List;                                                     // Import List interface
import java.util.Locale;                                                   // Import Locale để format số
```

#### Class declaration và member variables:
```java
public class ThongKeActivity extends AppCompatActivity {                   // Khai báo class kế thừa AppCompatActivity

    // UI Components - Khai báo các thành phần giao diện
    private TextView tvTitle;                                              // TextView tiêu đề trang
    private TextView tvTongNhanVien, tvNhanVienDangLam, tvNhanVienNghiViec; // TextViews thống kê nhân viên
    private TextView tvTongPhongBan, tvTongChucVu;                         // TextViews thống kê tổ chức
    private TextView tvTongChamCongThangNay;                               // TextView thống kê chấm công
    private TextView tvTongDonNghiPhep, tvDonChoDuyet, tvDonDaDuyet;       // TextViews thống kê nghỉ phép
    private TextView tvTongLuongThangNay, tvLuongTrungBinh;                // TextViews thống kê lương
    private Spinner spThang, spNam;                                        // Spinners chọn tháng và năm
    private boolean isInitialLoad = true;                                  // Flag kiểm tra lần load đầu tiên
    
    // Business Logic Components - Các thành phần logic nghiệp vụ
    private DatabaseHelper dbHelper;                                       // Helper để truy cập database
    private String currentRole;                                            // Vai trò người dùng hiện tại
    private String currentUsername;                                        // Username người dùng hiện tại
```

---

## 2️⃣ ACTIVITY LIFECYCLE METHODS

### Method onCreate:
```java
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi constructor của class cha
        setContentView(R.layout.activity_thong_ke);                        // Set layout cho Activity
        
        initViews();                                                       // Khởi tạo các view components
        setupDatabase();                                                   // Thiết lập kết nối database
        setupSpinners();                                                   // Thiết lập Spinners tháng/năm
        isInitialLoad = false;                                             // Đánh dấu đã hoàn thành initial load
        loadStatistics();                                                  // Load dữ liệu thống kê
    }
```

---

## 3️⃣ INITIALIZATION METHODS

### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo tất cả view components
        tvTitle = findViewById(R.id.tv_title);                             // Khởi tạo TextView tiêu đề
        
        // Thống kê nhân viên - Khởi tạo các TextView cho thống kê nhân viên
        tvTongNhanVien = findViewById(R.id.tv_tong_nhan_vien);             // TextView tổng số nhân viên
        tvNhanVienDangLam = findViewById(R.id.tv_nhan_vien_dang_lam);      // TextView nhân viên đang làm việc
        tvNhanVienNghiViec = findViewById(R.id.tv_nhan_vien_nghi_viec);    // TextView nhân viên đã nghỉ việc
        
        // Thống kê tổ chức - Khởi tạo các TextView cho thống kê tổ chức
        tvTongPhongBan = findViewById(R.id.tv_tong_phong_ban);             // TextView tổng số phòng ban
        tvTongChucVu = findViewById(R.id.tv_tong_chuc_vu);                 // TextView tổng số chức vụ
        
        // Thống kê chấm công - Khởi tạo TextView cho thống kê chấm công
        tvTongChamCongThangNay = findViewById(R.id.tv_tong_cham_cong_thang_nay); // TextView tổng chấm công tháng
        
        // Thống kê nghỉ phép - Khởi tạo các TextView cho thống kê nghỉ phép
        tvTongDonNghiPhep = findViewById(R.id.tv_tong_don_nghi_phep);      // TextView tổng đơn nghỉ phép
        tvDonChoDuyet = findViewById(R.id.tv_don_cho_duyet);               // TextView đơn chờ duyệt
        tvDonDaDuyet = findViewById(R.id.tv_don_da_duyet);                 // TextView đơn đã duyệt
        
        // Thống kê lương - Khởi tạo các TextView cho thống kê lương
        tvTongLuongThangNay = findViewById(R.id.tv_tong_luong_thang_nay);  // TextView tổng lương tháng
        tvLuongTrungBinh = findViewById(R.id.tv_luong_trung_binh);         // TextView lương trung bình
        
        // Spinners - Khởi tạo các Spinner chọn thời gian
        spThang = findViewById(R.id.sp_thang_thong_ke);                    // Spinner chọn tháng
        spNam = findViewById(R.id.sp_nam_thong_ke);                        // Spinner chọn năm
    }
```

### Method setupDatabase:
```java
    private void setupDatabase() {                                         // Method thiết lập kết nối database và lấy thông tin user
        dbHelper = new DatabaseHelper(this);                              // Khởi tạo DatabaseHelper với context hiện tại
        currentRole = getIntent().getStringExtra("role");                  // Lấy vai trò từ Intent
        currentUsername = getIntent().getStringExtra("username");          // Lấy username từ Intent
        
        tvTitle.setText("THỐNG KÊ TỔNG QUAN");                             // Set tiêu đề cho trang thống kê
    }
```
### Method setupSpinners:
```java
    private void setupSpinners() {                                         // Method thiết lập Spinners cho chọn tháng và năm
        // Tạo danh sách tháng từ 01 đến 12
        List<String> listThang = new ArrayList<>();                        // Tạo ArrayList chứa danh sách tháng
        for (int i = 1; i <= 12; i++) {                                    // Lặp từ tháng 1 đến 12
            listThang.add(String.format(Locale.getDefault(), "%02d", i)); // Thêm tháng với format 2 chữ số (01, 02, ...)
        }
        ArrayAdapter<String> thangAdapter = new ArrayAdapter<>(this, android.R.layout.simple_spinner_item, listThang); // Tạo ArrayAdapter cho Spinner tháng
        thangAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item); // Set layout cho dropdown
        spThang.setAdapter(thangAdapter);                                  // Set adapter cho Spinner tháng
        
        // Tạo danh sách năm từ 2020 đến 2030
        List<String> listNam = new ArrayList<>();                          // Tạo ArrayList chứa danh sách năm
        for (int i = 2020; i <= 2030; i++) {                              // Lặp từ năm 2020 đến 2030
            listNam.add(String.valueOf(i));                               // Thêm năm vào danh sách
        }
        ArrayAdapter<String> namAdapter = new ArrayAdapter<>(this, android.R.layout.simple_spinner_item, listNam); // Tạo ArrayAdapter cho Spinner năm
        namAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item); // Set layout cho dropdown
        spNam.setAdapter(namAdapter);                                      // Set adapter cho Spinner năm
        
        // Set giá trị mặc định là tháng và năm hiện tại
        Calendar cal = Calendar.getInstance();                             // Lấy instance của Calendar
        int currentMonth = cal.get(Calendar.MONTH);                       // Lấy tháng hiện tại (0-based, nên tháng 1 = 0)
        int currentYear = cal.get(Calendar.YEAR);                         // Lấy năm hiện tại
        
        spThang.setSelection(currentMonth);                                // Set selection cho Spinner tháng (currentMonth đã là 0-based)
        int yearPosition = listNam.indexOf(String.valueOf(currentYear));   // Tìm vị trí của năm hiện tại trong danh sách
        if (yearPosition >= 0) {                                           // Nếu tìm thấy năm hiện tại
            spNam.setSelection(yearPosition);                              // Set selection cho Spinner năm
        }
        
        // Tạo listener cho sự kiện thay đổi selection của Spinner
        AdapterView.OnItemSelectedListener listener = new AdapterView.OnItemSelectedListener() { // Tạo OnItemSelectedListener
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) { // Method được gọi khi item được chọn
                if (!isInitialLoad) {                                      // Nếu không phải lần load đầu tiên
                    loadStatistics();                                      // Load lại dữ liệu thống kê với tháng/năm mới
                }
            }

            @Override
            public void onNothingSelected(AdapterView<?> parent) {}        // Method được gọi khi không có item nào được chọn (không cần xử lý)
        };
        
        spThang.setOnItemSelectedListener(listener);                       // Set listener cho Spinner tháng
        spNam.setOnItemSelectedListener(listener);                         // Set listener cho Spinner năm
    }
```

---

## 4️⃣ STATISTICS LOADING METHODS

### Method loadStatistics:
```java
    private void loadStatistics() {                                        // Method chính để load tất cả dữ liệu thống kê
        loadEmployeeStatistics();                                          // Load thống kê nhân viên
        loadOrganizationStatistics();                                      // Load thống kê tổ chức
        loadAttendanceStatistics();                                        // Load thống kê chấm công
        loadLeaveStatistics();                                             // Load thống kê nghỉ phép
        loadSalaryStatistics();                                            // Load thống kê lương
    }
```

### Method loadEmployeeStatistics:
```java
    private void loadEmployeeStatistics() {                                // Method load thống kê nhân viên
        try {
            // Tổng số nhân viên - Query đếm tất cả nhân viên trong hệ thống
            Cursor cursor = dbHelper.getReadableDatabase().rawQuery(       // Thực hiện raw query
                "SELECT COUNT(*) FROM NhanVien", null);                    // Query đếm tổng số record trong bảng NhanVien
            int tongNhanVien = 0;                                          // Biến lưu tổng số nhân viên
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                tongNhanVien = cursor.getInt(0);                           // Lấy giá trị COUNT từ column đầu tiên
                cursor.close();                                            // Đóng cursor để giải phóng tài nguyên
            }
            
            // Nhân viên đang làm việc - Query đếm nhân viên có trạng thái 'Đang làm việc'
            cursor = dbHelper.getReadableDatabase().rawQuery(              // Thực hiện raw query
                "SELECT COUNT(*) FROM NhanVien WHERE TrangThaiLamViec = 'Đang làm việc'", null); // Query với WHERE condition
            int nhanVienDangLam = 0;                                       // Biến lưu số nhân viên đang làm việc
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                nhanVienDangLam = cursor.getInt(0);                        // Lấy giá trị COUNT
                cursor.close();                                            // Đóng cursor
            }
            
            // Nhân viên đã nghỉ việc - Tính bằng cách trừ tổng số với số đang làm việc
            int nhanVienNghiViec = tongNhanVien - nhanVienDangLam;         // Tính số nhân viên đã nghỉ việc
            
            // Cập nhật UI với dữ liệu đã tính toán
            tvTongNhanVien.setText(String.valueOf(tongNhanVien));          // Hiển thị tổng số nhân viên
            tvNhanVienDangLam.setText(String.valueOf(nhanVienDangLam));    // Hiển thị số nhân viên đang làm việc
            tvNhanVienNghiViec.setText(String.valueOf(nhanVienNghiViec));  // Hiển thị số nhân viên đã nghỉ việc
            
        } catch (Exception e) {                                           // Catch mọi exception có thể xảy ra
            e.printStackTrace();                                          // In stack trace để debug
        }
    }
```

### Method loadOrganizationStatistics:
```java
    private void loadOrganizationStatistics() {                            // Method load thống kê tổ chức
        try {
            // Tổng số phòng ban đang hoạt động - Query đếm phòng ban có TrangThai = 1
            Cursor cursor = dbHelper.getReadableDatabase().rawQuery(       // Thực hiện raw query
                "SELECT COUNT(*) FROM PhongBan WHERE TrangThai = 1", null); // Query đếm phòng ban đang hoạt động
            int tongPhongBan = 0;                                          // Biến lưu tổng số phòng ban
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                tongPhongBan = cursor.getInt(0);                           // Lấy giá trị COUNT
                cursor.close();                                            // Đóng cursor
            }
            
            // Tổng số chức vụ đang hoạt động - Query đếm chức vụ có TrangThai = 1
            cursor = dbHelper.getReadableDatabase().rawQuery(              // Thực hiện raw query
                "SELECT COUNT(*) FROM ChucVu WHERE TrangThai = 1", null);  // Query đếm chức vụ đang hoạt động
            int tongChucVu = 0;                                            // Biến lưu tổng số chức vụ
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                tongChucVu = cursor.getInt(0);                             // Lấy giá trị COUNT
                cursor.close();                                            // Đóng cursor
            }
            
            // Cập nhật UI với dữ liệu tổ chức
            tvTongPhongBan.setText(String.valueOf(tongPhongBan));          // Hiển thị tổng số phòng ban
            tvTongChucVu.setText(String.valueOf(tongChucVu));              // Hiển thị tổng số chức vụ
            
        } catch (Exception e) {                                           // Catch exception
            e.printStackTrace();                                          // In stack trace để debug
        }
    }
```

### Method loadAttendanceStatistics:
```java
    private void loadAttendanceStatistics() {                              // Method load thống kê chấm công
        try {
            String selectedMonth = spThang.getSelectedItem().toString();   // Lấy tháng được chọn từ Spinner
            String selectedYear = spNam.getSelectedItem().toString();      // Lấy năm được chọn từ Spinner
            String thangNam = selectedYear + "-" + selectedMonth;          // Tạo string tháng năm format YYYY-MM
            
            // Tổng số lần chấm công tháng này - Query đếm record chấm công trong tháng được chọn
            Cursor cursor = dbHelper.getReadableDatabase().rawQuery(       // Thực hiện raw query
                "SELECT COUNT(*) FROM ChamCong WHERE strftime('%Y-%m', NgayChamCong) = ?", // Query sử dụng strftime để extract tháng năm
                new String[]{thangNam});                                   // Parameter cho prepared statement
            int tongChamCong = 0;                                          // Biến lưu tổng số lần chấm công
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                tongChamCong = cursor.getInt(0);                           // Lấy giá trị COUNT
                cursor.close();                                            // Đóng cursor
            }
            
            // Cập nhật UI với dữ liệu chấm công
            tvTongChamCongThangNay.setText(String.valueOf(tongChamCong));  // Hiển thị tổng số lần chấm công
            
        } catch (Exception e) {                                           // Catch exception
            e.printStackTrace();                                          // In stack trace để debug
        }
    }
```
### Method loadLeaveStatistics:
```java
    private void loadLeaveStatistics() {                                   // Method load thống kê nghỉ phép
        try {
            String selectedMonth = spThang.getSelectedItem().toString();   // Lấy tháng được chọn từ Spinner
            String selectedYear = spNam.getSelectedItem().toString();      // Lấy năm được chọn từ Spinner
            String thangNam = selectedYear + "-" + selectedMonth;          // Tạo string tháng năm format YYYY-MM
            
            // Tổng số đơn nghỉ phép trong tháng - Query đếm đơn nghỉ phép có NgayBatDau trong tháng được chọn
            Cursor cursor = dbHelper.getReadableDatabase().rawQuery(       // Thực hiện raw query
                "SELECT COUNT(*) FROM NghiPhep WHERE strftime('%Y-%m', NgayBatDau) = ?", new String[]{thangNam}); // Query với strftime để extract tháng năm từ NgayBatDau
            int tongDonNghiPhep = 0;                                       // Biến lưu tổng số đơn nghỉ phép
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                tongDonNghiPhep = cursor.getInt(0);                        // Lấy giá trị COUNT
                cursor.close();                                            // Đóng cursor
            }
            
            // Đơn chờ duyệt - Query đếm đơn có TrangThai = 'Chờ duyệt' trong tháng được chọn
            cursor = dbHelper.getReadableDatabase().rawQuery(              // Thực hiện raw query
                "SELECT COUNT(*) FROM NghiPhep WHERE TrangThai = 'Chờ duyệt' AND strftime('%Y-%m', NgayBatDau) = ?", new String[]{thangNam}); // Query với WHERE conditions
            int donChoDuyet = 0;                                           // Biến lưu số đơn chờ duyệt
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                donChoDuyet = cursor.getInt(0);                            // Lấy giá trị COUNT
                cursor.close();                                            // Đóng cursor
            }
            
            // Đơn đã duyệt - Query đếm đơn có TrangThai = 'Đã duyệt' trong tháng được chọn
            cursor = dbHelper.getReadableDatabase().rawQuery(              // Thực hiện raw query
                "SELECT COUNT(*) FROM NghiPhep WHERE TrangThai = 'Đã duyệt' AND strftime('%Y-%m', NgayBatDau) = ?", new String[]{thangNam}); // Query với WHERE conditions
            int donDaDuyet = 0;                                            // Biến lưu số đơn đã duyệt
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                donDaDuyet = cursor.getInt(0);                             // Lấy giá trị COUNT
                cursor.close();                                            // Đóng cursor
            }
            
            // Cập nhật UI với dữ liệu nghỉ phép
            tvTongDonNghiPhep.setText(String.valueOf(tongDonNghiPhep));    // Hiển thị tổng số đơn nghỉ phép
            tvDonChoDuyet.setText(String.valueOf(donChoDuyet));            // Hiển thị số đơn chờ duyệt
            tvDonDaDuyet.setText(String.valueOf(donDaDuyet));              // Hiển thị số đơn đã duyệt
            
        } catch (Exception e) {                                           // Catch exception
            e.printStackTrace();                                          // In stack trace để debug
        }
    }
```

### Method loadSalaryStatistics:
```java
    private void loadSalaryStatistics() {                                  // Method load thống kê lương
        try {
            String selectedMonth = spThang.getSelectedItem().toString();   // Lấy tháng được chọn từ Spinner
            String selectedYear = spNam.getSelectedItem().toString();      // Lấy năm được chọn từ Spinner
            String thangNam = selectedYear + "-" + selectedMonth;          // Tạo string tháng năm format YYYY-MM
            
            // Tổng lương tháng này và số nhân viên có lương - Query SUM và COUNT từ bảng Luong
            Cursor cursor = dbHelper.getReadableDatabase().rawQuery(       // Thực hiện raw query
                "SELECT SUM(TongLuong), COUNT(*) FROM Luong WHERE ThangNam = ?", // Query SUM tổng lương và COUNT số record
                new String[]{thangNam});                                   // Parameter cho prepared statement
            
            double tongLuong = 0;                                          // Biến lưu tổng lương
            int soNhanVienCoLuong = 0;                                     // Biến lưu số nhân viên có lương
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                tongLuong = cursor.getDouble(0);                           // Lấy giá trị SUM từ column đầu tiên
                soNhanVienCoLuong = cursor.getInt(1);                      // Lấy giá trị COUNT từ column thứ hai
                cursor.close();                                            // Đóng cursor
            }
            
            // Tính lương trung bình - Chia tổng lương cho số nhân viên có lương
            double luongTrungBinh = soNhanVienCoLuong > 0 ? tongLuong / soNhanVienCoLuong : 0; // Tránh chia cho 0
            
            // Cập nhật UI với dữ liệu lương đã format
            tvTongLuongThangNay.setText(formatCurrency(tongLuong));        // Hiển thị tổng lương với format tiền tệ
            tvLuongTrungBinh.setText(formatCurrency(luongTrungBinh));      // Hiển thị lương trung bình với format tiền tệ
            
        } catch (Exception e) {                                           // Catch exception
            e.printStackTrace();                                          // In stack trace để debug
        }
    }
```

---

## 5️⃣ UTILITY METHODS

### Method formatCurrency:
```java
    private String formatCurrency(double amount) {                         // Method format số tiền thành chuỗi tiền tệ VND
        return String.format("%,.0f đ", amount);                          // Format với dấu phẩy phân cách hàng nghìn và đơn vị đồng
    }
}
```

---

## 📱 CHI TIẾT LAYOUT IMPLEMENTATION

## 6️⃣ XML LAYOUT STRUCTURE

**Đường dẫn**: `app/src/main/res/layout/activity_thong_ke.xml`

### Mục đích:
Layout chính của module thống kê với thiết kế Material Design, sử dụng CardView và ScrollView để hiển thị các chỉ số thống kê một cách trực quan và dễ đọc.

### Chi tiết XML:

#### Root LinearLayout:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- XML declaration với encoding UTF-8 -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- Root LinearLayout với namespace Android -->
    xmlns:app="http://schemas.android.com/apk/res-auto"                    <!-- Namespace cho app attributes -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng full màn hình -->
    android:layout_height="match_parent"                                   <!-- Chiều cao full màn hình -->
    android:orientation="vertical"                                         <!-- Orientation dọc -->
    android:background="#f8f9fa">                                          <!-- Background màu xám nhạt -->
```

#### Header Section với Time Selection:
```xml
    <!-- Header -->
    <LinearLayout                                                          <!-- LinearLayout chứa header và time selection -->
        android:layout_width="match_parent"                                <!-- Chiều rộng full -->
        android:layout_height="wrap_content"                               <!-- Chiều cao wrap content -->
        android:orientation="vertical"                                     <!-- Orientation dọc -->
        android:background="#2196F3"                                       <!-- Background màu xanh Material Design -->
        android:padding="20dp"                                             <!-- Padding 20dp tất cả các cạnh -->
        android:elevation="8dp">                                           <!-- Elevation tạo shadow -->

        <TextView                                                          <!-- TextView tiêu đề chính -->
            android:id="@+id/tv_title"                                     <!-- ID để reference từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng full -->
            android:layout_height="wrap_content"                           <!-- Chiều cao wrap content -->
            android:text="THỐNG KÊ TỔNG QUAN"                              <!-- Text tiêu đề -->
            android:textSize="24sp"                                        <!-- Kích thước text 24sp -->
            android:textStyle="bold"                                       <!-- Style bold -->
            android:textColor="@android:color/white"                       <!-- Màu text trắng -->
            android:gravity="center"                                       <!-- Căn giữa -->
            android:layout_marginBottom="8dp" />                           <!-- Margin bottom 8dp -->

        <LinearLayout                                                      <!-- LinearLayout chứa Spinners chọn thời gian -->
            android:layout_width="wrap_content"                            <!-- Chiều rộng wrap content -->
            android:layout_height="wrap_content"                           <!-- Chiều cao wrap content -->
            android:layout_gravity="center"                                <!-- Căn giữa trong parent -->
            android:gravity="center_vertical"                              <!-- Căn giữa theo chiều dọc -->
            android:orientation="horizontal">                              <!-- Orientation ngang -->

            <TextView                                                      <!-- Label cho Spinner tháng -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng wrap content -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:text="Tháng: "                                     <!-- Text label -->
                android:textColor="@android:color/white"                   <!-- Màu text trắng -->
                android:textSize="16sp" />                                 <!-- Kích thước text 16sp -->

            <Spinner                                                       <!-- Spinner chọn tháng -->
                android:id="@+id/sp_thang_thong_ke"                        <!-- ID để reference từ Java code -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng wrap content -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:minHeight="48dp"                                   <!-- Chiều cao tối thiểu 48dp (accessibility) -->
                android:backgroundTint="@android:color/white" />           <!-- Background tint màu trắng -->

            <TextView                                                      <!-- Label cho Spinner năm -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng wrap content -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:text="Năm: "                                       <!-- Text label -->
                android:textColor="@android:color/white"                   <!-- Màu text trắng -->
                android:textSize="16sp"                                    <!-- Kích thước text 16sp -->
                android:layout_marginStart="16dp" />                       <!-- Margin start 16dp -->

            <Spinner                                                       <!-- Spinner chọn năm -->
                android:id="@+id/sp_nam_thong_ke"                          <!-- ID để reference từ Java code -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng wrap content -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:minHeight="48dp"                                   <!-- Chiều cao tối thiểu 48dp -->
                android:backgroundTint="@android:color/white" />           <!-- Background tint màu trắng -->

        </LinearLayout>

    </LinearLayout>
```
#### ScrollView với Statistics Cards:
```xml
    <ScrollView                                                            <!-- ScrollView để cuộn khi có nhiều thống kê -->
        android:layout_width="match_parent"                                <!-- Chiều rộng full -->
        android:layout_height="match_parent"                               <!-- Chiều cao full -->
        android:padding="16dp">                                            <!-- Padding 16dp -->

        <LinearLayout                                                      <!-- LinearLayout chứa tất cả statistics cards -->
            android:layout_width="match_parent"                            <!-- Chiều rộng full -->
            android:layout_height="wrap_content"                           <!-- Chiều cao wrap content -->
            android:orientation="vertical">                                <!-- Orientation dọc -->
```

#### Employee Statistics Card:
```xml
            <!-- Thống kê nhân viên -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho thống kê nhân viên -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp tạo shadow -->
                app:cardBackgroundColor="@android:color/white">            <!-- Background màu trắng -->

                <LinearLayout                                              <!-- LinearLayout chứa nội dung card -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao wrap content -->
                    android:orientation="vertical"                         <!-- Orientation dọc -->
                    android:padding="20dp">                                <!-- Padding 20dp -->

                    <TextView                                              <!-- TextView tiêu đề section -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="THỐNG KÊ NHÂN VIÊN"                  <!-- Text tiêu đề -->
                        android:textSize="18sp"                            <!-- Kích thước text 18sp -->
                        android:textStyle="bold"                           <!-- Style bold -->
                        android:textColor="#2196F3"                        <!-- Màu text xanh -->
                        android:layout_marginBottom="16dp" />              <!-- Margin bottom 16dp -->

                    <LinearLayout                                          <!-- LinearLayout chứa 2 statistics boxes -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:orientation="horizontal">                  <!-- Orientation ngang -->

                        <LinearLayout                                      <!-- Statistics box cho tổng nhân viên -->
                            android:layout_width="0dp"                     <!-- Chiều rộng 0dp để sử dụng weight -->
                            android:layout_height="wrap_content"           <!-- Chiều cao wrap content -->
                            android:layout_weight="1"                      <!-- Weight 1 để chia đều không gian -->
                            android:orientation="vertical"                 <!-- Orientation dọc -->
                            android:gravity="center"                       <!-- Căn giữa nội dung -->
                            android:background="#E3F2FD"                   <!-- Background màu xanh nhạt -->
                            android:padding="16dp"                         <!-- Padding 16dp -->
                            android:layout_marginEnd="8dp">                <!-- Margin end 8dp -->

                            <TextView                                      <!-- TextView hiển thị số liệu -->
                                android:id="@+id/tv_tong_nhan_vien"        <!-- ID để reference từ Java code -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="0"                           <!-- Text mặc định -->
                                android:textSize="32sp"                    <!-- Kích thước text lớn 32sp -->
                                android:textStyle="bold"                   <!-- Style bold -->
                                android:textColor="#1976D2" />             <!-- Màu text xanh đậm -->

                            <TextView                                      <!-- TextView label -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="Tổng nhân viên"              <!-- Text label -->
                                android:textSize="14sp"                    <!-- Kích thước text 14sp -->
                                android:textColor="#666"                   <!-- Màu text xám -->
                                android:gravity="center" />                <!-- Căn giữa -->

                        </LinearLayout>

                        <LinearLayout                                      <!-- Statistics box cho nhân viên đang làm việc -->
                            android:layout_width="0dp"                     <!-- Chiều rộng 0dp để sử dụng weight -->
                            android:layout_height="wrap_content"           <!-- Chiều cao wrap content -->
                            android:layout_weight="1"                      <!-- Weight 1 để chia đều không gian -->
                            android:orientation="vertical"                 <!-- Orientation dọc -->
                            android:gravity="center"                       <!-- Căn giữa nội dung -->
                            android:background="#E8F5E8"                   <!-- Background màu xanh lá nhạt -->
                            android:padding="16dp"                         <!-- Padding 16dp -->
                            android:layout_marginStart="8dp">              <!-- Margin start 8dp -->

                            <TextView                                      <!-- TextView hiển thị số liệu -->
                                android:id="@+id/tv_nhan_vien_dang_lam"    <!-- ID để reference từ Java code -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="0"                           <!-- Text mặc định -->
                                android:textSize="32sp"                    <!-- Kích thước text lớn 32sp -->
                                android:textStyle="bold"                   <!-- Style bold -->
                                android:textColor="#388E3C" />             <!-- Màu text xanh lá -->

                            <TextView                                      <!-- TextView label -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="Đang làm việc"               <!-- Text label -->
                                android:textSize="14sp"                    <!-- Kích thước text 14sp -->
                                android:textColor="#666"                   <!-- Màu text xám -->
                                android:gravity="center" />                <!-- Căn giữa -->

                        </LinearLayout>

                    </LinearLayout>

                    <LinearLayout                                          <!-- LinearLayout cho statistics box nhân viên nghỉ việc -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:orientation="horizontal"                   <!-- Orientation ngang -->
                        android:layout_marginTop="12dp">                   <!-- Margin top 12dp -->

                        <LinearLayout                                      <!-- Statistics box cho nhân viên nghỉ việc -->
                            android:layout_width="match_parent"            <!-- Chiều rộng full -->
                            android:layout_height="wrap_content"           <!-- Chiều cao wrap content -->
                            android:orientation="vertical"                 <!-- Orientation dọc -->
                            android:gravity="center"                       <!-- Căn giữa nội dung -->
                            android:background="#FFEBEE"                   <!-- Background màu đỏ nhạt -->
                            android:padding="16dp">                        <!-- Padding 16dp -->

                            <TextView                                      <!-- TextView hiển thị số liệu -->
                                android:id="@+id/tv_nhan_vien_nghi_viec"   <!-- ID để reference từ Java code -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="0"                           <!-- Text mặc định -->
                                android:textSize="28sp"                    <!-- Kích thước text 28sp -->
                                android:textStyle="bold"                   <!-- Style bold -->
                                android:textColor="#D32F2F" />             <!-- Màu text đỏ -->

                            <TextView                                      <!-- TextView label -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="Đã nghỉ việc"                <!-- Text label -->
                                android:textSize="14sp"                    <!-- Kích thước text 14sp -->
                                android:textColor="#666" />                <!-- Màu text xám -->

                        </LinearLayout>

                    </LinearLayout>

                </LinearLayout>

            </androidx.cardview.widget.CardView>
```

#### Organization Statistics Card:
```xml
            <!-- Thống kê tổ chức -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho thống kê tổ chức -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp -->
                app:cardBackgroundColor="@android:color/white">            <!-- Background màu trắng -->

                <LinearLayout                                              <!-- LinearLayout chứa nội dung card -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao wrap content -->
                    android:orientation="vertical"                         <!-- Orientation dọc -->
                    android:padding="20dp">                                <!-- Padding 20dp -->

                    <TextView                                              <!-- TextView tiêu đề section -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="THỐNG KÊ TỔ CHỨC"                    <!-- Text tiêu đề -->
                        android:textSize="18sp"                            <!-- Kích thước text 18sp -->
                        android:textStyle="bold"                           <!-- Style bold -->
                        android:textColor="#FF9800"                        <!-- Màu text cam -->
                        android:layout_marginBottom="16dp" />              <!-- Margin bottom 16dp -->

                    <LinearLayout                                          <!-- LinearLayout chứa 2 statistics boxes -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:orientation="horizontal">                  <!-- Orientation ngang -->

                        <LinearLayout                                      <!-- Statistics box cho phòng ban -->
                            android:layout_width="0dp"                     <!-- Chiều rộng 0dp để sử dụng weight -->
                            android:layout_height="wrap_content"           <!-- Chiều cao wrap content -->
                            android:layout_weight="1"                      <!-- Weight 1 -->
                            android:orientation="vertical"                 <!-- Orientation dọc -->
                            android:gravity="center"                       <!-- Căn giữa nội dung -->
                            android:background="#FFF3E0"                   <!-- Background màu cam nhạt -->
                            android:padding="16dp"                         <!-- Padding 16dp -->
                            android:layout_marginEnd="8dp">                <!-- Margin end 8dp -->

                            <TextView                                      <!-- TextView hiển thị số liệu phòng ban -->
                                android:id="@+id/tv_tong_phong_ban"        <!-- ID để reference từ Java code -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="0"                           <!-- Text mặc định -->
                                android:textSize="32sp"                    <!-- Kích thước text lớn 32sp -->
                                android:textStyle="bold"                   <!-- Style bold -->
                                android:textColor="#F57C00" />             <!-- Màu text cam đậm -->

                            <TextView                                      <!-- TextView label phòng ban -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="Phòng ban"                   <!-- Text label -->
                                android:textSize="14sp"                    <!-- Kích thước text 14sp -->
                                android:textColor="#666" />                <!-- Màu text xám -->

                        </LinearLayout>

                        <LinearLayout                                      <!-- Statistics box cho chức vụ -->
                            android:layout_width="0dp"                     <!-- Chiều rộng 0dp để sử dụng weight -->
                            android:layout_height="wrap_content"           <!-- Chiều cao wrap content -->
                            android:layout_weight="1"                      <!-- Weight 1 -->
                            android:orientation="vertical"                 <!-- Orientation dọc -->
                            android:gravity="center"                       <!-- Căn giữa nội dung -->
                            android:background="#F3E5F5"                   <!-- Background màu tím nhạt -->
                            android:padding="16dp"                         <!-- Padding 16dp -->
                            android:layout_marginStart="8dp">              <!-- Margin start 8dp -->

                            <TextView                                      <!-- TextView hiển thị số liệu chức vụ -->
                                android:id="@+id/tv_tong_chuc_vu"          <!-- ID để reference từ Java code -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="0"                           <!-- Text mặc định -->
                                android:textSize="32sp"                    <!-- Kích thước text lớn 32sp -->
                                android:textStyle="bold"                   <!-- Style bold -->
                                android:textColor="#7B1FA2" />             <!-- Màu text tím -->

                            <TextView                                      <!-- TextView label chức vụ -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="Chức vụ"                     <!-- Text label -->
                                android:textSize="14sp"                    <!-- Kích thước text 14sp -->
                                android:textColor="#666" />                <!-- Màu text xám -->

                        </LinearLayout>

                    </LinearLayout>

                </LinearLayout>

            </androidx.cardview.widget.CardView>
```
#### Attendance, Leave và Salary Statistics Cards:
```xml
            <!-- Thống kê chấm công -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho thống kê chấm công -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp -->
                app:cardBackgroundColor="@android:color/white">            <!-- Background màu trắng -->

                <LinearLayout                                              <!-- LinearLayout chứa nội dung card -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao wrap content -->
                    android:orientation="vertical"                         <!-- Orientation dọc -->
                    android:padding="20dp">                                <!-- Padding 20dp -->

                    <TextView                                              <!-- TextView tiêu đề section -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="THỐNG KÊ CHẤM CÔNG"                  <!-- Text tiêu đề -->
                        android:textSize="18sp"                            <!-- Kích thước text 18sp -->
                        android:textStyle="bold"                           <!-- Style bold -->
                        android:textColor="#00BCD4"                        <!-- Màu text cyan -->
                        android:layout_marginBottom="16dp" />              <!-- Margin bottom 16dp -->

                    <LinearLayout                                          <!-- LinearLayout chứa statistics box -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:orientation="vertical"                     <!-- Orientation dọc -->
                        android:gravity="center"                           <!-- Căn giữa nội dung -->
                        android:background="#E0F2F1"                       <!-- Background màu cyan nhạt -->
                        android:padding="16dp">                            <!-- Padding 16dp -->

                        <TextView                                          <!-- TextView hiển thị số liệu chấm công -->
                            android:id="@+id/tv_tong_cham_cong_thang_nay"  <!-- ID để reference từ Java code -->
                            android:layout_width="wrap_content"            <!-- Chiều rộng wrap content -->
                            android:layout_height="wrap_content"           <!-- Chiều cao wrap content -->
                            android:text="0"                               <!-- Text mặc định -->
                            android:textSize="32sp"                        <!-- Kích thước text lớn 32sp -->
                            android:textStyle="bold"                       <!-- Style bold -->
                            android:textColor="#00ACC1" />                 <!-- Màu text cyan đậm -->

                        <TextView                                          <!-- TextView label -->
                            android:layout_width="wrap_content"            <!-- Chiều rộng wrap content -->
                            android:layout_height="wrap_content"           <!-- Chiều cao wrap content -->
                            android:text="Lần chấm công"                   <!-- Text label -->
                            android:textSize="14sp"                        <!-- Kích thước text 14sp -->
                            android:textColor="#666"                       <!-- Màu text xám -->
                            android:gravity="center" />                    <!-- Căn giữa -->

                    </LinearLayout>

                </LinearLayout>

            </androidx.cardview.widget.CardView>

            <!-- Thống kê nghỉ phép -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho thống kê nghỉ phép -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp -->
                app:cardBackgroundColor="@android:color/white">            <!-- Background màu trắng -->

                <LinearLayout                                              <!-- LinearLayout chứa nội dung card -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao wrap content -->
                    android:orientation="vertical"                         <!-- Orientation dọc -->
                    android:padding="20dp">                                <!-- Padding 20dp -->

                    <TextView                                              <!-- TextView tiêu đề section -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="THỐNG KÊ NGHỈ PHÉP"                  <!-- Text tiêu đề -->
                        android:textSize="18sp"                            <!-- Kích thước text 18sp -->
                        android:textStyle="bold"                           <!-- Style bold -->
                        android:textColor="#795548"                        <!-- Màu text nâu -->
                        android:layout_marginBottom="16dp" />              <!-- Margin bottom 16dp -->

                    <LinearLayout                                          <!-- LinearLayout chứa 3 statistics boxes -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:orientation="horizontal">                  <!-- Orientation ngang -->

                        <LinearLayout                                      <!-- Statistics box cho tổng đơn -->
                            android:layout_width="0dp"                     <!-- Chiều rộng 0dp để sử dụng weight -->
                            android:layout_height="wrap_content"           <!-- Chiều cao wrap content -->
                            android:layout_weight="1"                      <!-- Weight 1 -->
                            android:orientation="vertical"                 <!-- Orientation dọc -->
                            android:gravity="center"                       <!-- Căn giữa nội dung -->
                            android:background="#EFEBE9"                   <!-- Background màu nâu nhạt -->
                            android:padding="16dp"                         <!-- Padding 16dp -->
                            android:layout_marginEnd="4dp">                <!-- Margin end 4dp -->

                            <TextView                                      <!-- TextView hiển thị tổng đơn -->
                                android:id="@+id/tv_tong_don_nghi_phep"    <!-- ID để reference từ Java code -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="0"                           <!-- Text mặc định -->
                                android:textSize="28sp"                    <!-- Kích thước text 28sp -->
                                android:textStyle="bold"                   <!-- Style bold -->
                                android:textColor="#5D4037" />             <!-- Màu text nâu đậm -->

                            <TextView                                      <!-- TextView label -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="Tổng đơn"                    <!-- Text label -->
                                android:textSize="12sp"                    <!-- Kích thước text nhỏ 12sp -->
                                android:textColor="#666" />                <!-- Màu text xám -->

                        </LinearLayout>

                        <LinearLayout                                      <!-- Statistics box cho đơn chờ duyệt -->
                            android:layout_width="0dp"                     <!-- Chiều rộng 0dp để sử dụng weight -->
                            android:layout_height="wrap_content"           <!-- Chiều cao wrap content -->
                            android:layout_weight="1"                      <!-- Weight 1 -->
                            android:orientation="vertical"                 <!-- Orientation dọc -->
                            android:gravity="center"                       <!-- Căn giữa nội dung -->
                            android:background="#FFF3E0"                   <!-- Background màu cam nhạt -->
                            android:padding="16dp"                         <!-- Padding 16dp -->
                            android:layout_marginStart="4dp"               <!-- Margin start 4dp -->
                            android:layout_marginEnd="4dp">                <!-- Margin end 4dp -->

                            <TextView                                      <!-- TextView hiển thị đơn chờ duyệt -->
                                android:id="@+id/tv_don_cho_duyet"          <!-- ID để reference từ Java code -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="0"                           <!-- Text mặc định -->
                                android:textSize="28sp"                    <!-- Kích thước text 28sp -->
                                android:textStyle="bold"                   <!-- Style bold -->
                                android:textColor="#F57C00" />             <!-- Màu text cam -->

                            <TextView                                      <!-- TextView label -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="Chờ duyệt"                   <!-- Text label -->
                                android:textSize="12sp"                    <!-- Kích thước text nhỏ 12sp -->
                                android:textColor="#666" />                <!-- Màu text xám -->

                        </LinearLayout>

                        <LinearLayout                                      <!-- Statistics box cho đơn đã duyệt -->
                            android:layout_width="0dp"                     <!-- Chiều rộng 0dp để sử dụng weight -->
                            android:layout_height="wrap_content"           <!-- Chiều cao wrap content -->
                            android:layout_weight="1"                      <!-- Weight 1 -->
                            android:orientation="vertical"                 <!-- Orientation dọc -->
                            android:gravity="center"                       <!-- Căn giữa nội dung -->
                            android:background="#E8F5E8"                   <!-- Background màu xanh lá nhạt -->
                            android:padding="16dp"                         <!-- Padding 16dp -->
                            android:layout_marginStart="4dp">              <!-- Margin start 4dp -->

                            <TextView                                      <!-- TextView hiển thị đơn đã duyệt -->
                                android:id="@+id/tv_don_da_duyet"          <!-- ID để reference từ Java code -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="0"                           <!-- Text mặc định -->
                                android:textSize="28sp"                    <!-- Kích thước text 28sp -->
                                android:textStyle="bold"                   <!-- Style bold -->
                                android:textColor="#388E3C" />             <!-- Màu text xanh lá -->

                            <TextView                                      <!-- TextView label -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="Đã duyệt"                    <!-- Text label -->
                                android:textSize="12sp"                    <!-- Kích thước text nhỏ 12sp -->
                                android:textColor="#666" />                <!-- Màu text xám -->

                        </LinearLayout>

                    </LinearLayout>

                </LinearLayout>

            </androidx.cardview.widget.CardView>

            <!-- Thống kê lương -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho thống kê lương -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp -->
                app:cardBackgroundColor="@android:color/white">            <!-- Background màu trắng -->

                <LinearLayout                                              <!-- LinearLayout chứa nội dung card -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao wrap content -->
                    android:orientation="vertical"                         <!-- Orientation dọc -->
                    android:padding="20dp">                                <!-- Padding 20dp -->

                    <TextView                                              <!-- TextView tiêu đề section -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="THỐNG KÊ LƯƠNG"                      <!-- Text tiêu đề -->
                        android:textSize="18sp"                            <!-- Kích thước text 18sp -->
                        android:textStyle="bold"                           <!-- Style bold -->
                        android:textColor="#E91E63"                        <!-- Màu text hồng -->
                        android:layout_marginBottom="16dp" />              <!-- Margin bottom 16dp -->

                    <LinearLayout                                          <!-- LinearLayout chứa 2 statistics boxes -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:orientation="vertical">                    <!-- Orientation dọc -->

                        <LinearLayout                                      <!-- Statistics box cho tổng lương tháng -->
                            android:layout_width="match_parent"            <!-- Chiều rộng full -->
                            android:layout_height="wrap_content"           <!-- Chiều cao wrap content -->
                            android:orientation="vertical"                 <!-- Orientation dọc -->
                            android:gravity="center"                       <!-- Căn giữa nội dung -->
                            android:background="#FCE4EC"                   <!-- Background màu hồng nhạt -->
                            android:padding="20dp"                         <!-- Padding 20dp -->
                            android:layout_marginBottom="12dp">            <!-- Margin bottom 12dp -->

                            <TextView                                      <!-- TextView hiển thị tổng lương -->
                                android:id="@+id/tv_tong_luong_thang_nay"  <!-- ID để reference từ Java code -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="0 đ"                         <!-- Text mặc định với đơn vị -->
                                android:textSize="24sp"                    <!-- Kích thước text 24sp -->
                                android:textStyle="bold"                   <!-- Style bold -->
                                android:textColor="#C2185B" />             <!-- Màu text hồng đậm -->

                            <TextView                                      <!-- TextView label -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="Tổng lương tháng"            <!-- Text label -->
                                android:textSize="14sp"                    <!-- Kích thước text 14sp -->
                                android:textColor="#666"                   <!-- Màu text xám -->
                                android:layout_marginTop="4dp" />          <!-- Margin top 4dp -->

                        </LinearLayout>

                        <LinearLayout                                      <!-- Statistics box cho lương trung bình -->
                            android:layout_width="match_parent"            <!-- Chiều rộng full -->
                            android:layout_height="wrap_content"           <!-- Chiều cao wrap content -->
                            android:orientation="vertical"                 <!-- Orientation dọc -->
                            android:gravity="center"                       <!-- Căn giữa nội dung -->
                            android:background="#F3E5F5"                   <!-- Background màu tím nhạt -->
                            android:padding="20dp">                        <!-- Padding 20dp -->

                            <TextView                                      <!-- TextView hiển thị lương trung bình -->
                                android:id="@+id/tv_luong_trung_binh"      <!-- ID để reference từ Java code -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="0 đ"                         <!-- Text mặc định với đơn vị -->
                                android:textSize="24sp"                    <!-- Kích thước text 24sp -->
                                android:textStyle="bold"                   <!-- Style bold -->
                                android:textColor="#7B1FA2" />             <!-- Màu text tím -->

                            <TextView                                      <!-- TextView label -->
                                android:layout_width="wrap_content"        <!-- Chiều rộng wrap content -->
                                android:layout_height="wrap_content"       <!-- Chiều cao wrap content -->
                                android:text="Lương trung bình"            <!-- Text label -->
                                android:textSize="14sp"                    <!-- Kích thước text 14sp -->
                                android:textColor="#666"                   <!-- Màu text xám -->
                                android:layout_marginTop="4dp" />          <!-- Margin top 4dp -->

                        </LinearLayout>

                    </LinearLayout>

                </LinearLayout>

            </androidx.cardview.widget.CardView>

        </LinearLayout>                                                    <!-- Kết thúc LinearLayout chứa tất cả cards -->

    </ScrollView>                                                          <!-- Kết thúc ScrollView -->

</LinearLayout>                                                            <!-- Kết thúc root LinearLayout -->
```

---

## 🔗 TÍCH HỢP VÀ KIẾN TRÚC

### 1. Database Integration:
- **Raw SQL Queries**: Sử dụng raw SQL với aggregate functions (COUNT, SUM)
- **Date Functions**: Sử dụng strftime() để filter theo tháng/năm
- **Multiple Tables**: Query từ 8 bảng khác nhau để tổng hợp thống kê
- **Performance**: Optimized queries với proper indexing

### 2. Time-Based Analysis:
- **Dynamic Period Selection**: Spinner cho phép chọn tháng/năm
- **Real-time Updates**: Auto-reload khi thay đổi thời gian
- **Current Period Default**: Mặc định hiển thị tháng/năm hiện tại
- **Historical Data**: Hỗ trợ xem thống kê các tháng trước

### 3. UI/UX Design:
- **Material Design**: CardView với elevation và corner radius
- **Color Coding**: Mỗi loại thống kê có màu sắc riêng biệt
- **Visual Hierarchy**: Typography và spacing tạo hierarchy rõ ràng
- **Responsive Layout**: ScrollView hỗ trợ nhiều kích thước màn hình

### 4. Business Intelligence:
- **KPI Tracking**: Theo dõi các chỉ số quan trọng
- **Trend Analysis**: So sánh dữ liệu theo thời gian
- **Decision Support**: Cung cấp thông tin để ra quyết định
- **Performance Monitoring**: Giám sát hiệu quả hoạt động

### 5. Security & Access Control:
- **Role-Based Access**: Chỉ Admin/HR/Manager mới truy cập được
- **Data Privacy**: Không hiển thị thông tin cá nhân nhạy cảm
- **Aggregated Data**: Chỉ hiển thị dữ liệu tổng hợp, không chi tiết cá nhân

---

## 📋 BUSINESS RULES & CALCULATIONS

### 1. Employee Statistics:
```
Tổng nhân viên = COUNT(*) FROM NhanVien
Nhân viên đang làm = COUNT(*) WHERE TrangThaiLamViec = 'Đang làm việc'
Nhân viên nghỉ việc = Tổng nhân viên - Nhân viên đang làm
```

### 2. Organization Statistics:
```
Tổng phòng ban = COUNT(*) FROM PhongBan WHERE TrangThai = 1
Tổng chức vụ = COUNT(*) FROM ChucVu WHERE TrangThai = 1
```

### 3. Attendance Statistics:
```
Tổng chấm công = COUNT(*) FROM ChamCong WHERE tháng = selected_month
```

### 4. Leave Statistics:
```
Tổng đơn nghỉ phép = COUNT(*) FROM NghiPhep WHERE tháng = selected_month
Đơn chờ duyệt = COUNT(*) WHERE TrangThai = 'Chờ duyệt'
Đơn đã duyệt = COUNT(*) WHERE TrangThai = 'Đã duyệt'
```

### 5. Salary Statistics:
```
Tổng lương tháng = SUM(TongLuong) FROM Luong WHERE ThangNam = selected_period
Lương trung bình = Tổng lương / COUNT(nhân viên có lương)
```

Module Thống kê hoàn thành với đầy đủ chức năng phân tích dữ liệu, hiển thị trực quan và hỗ trợ ra quyết định cho ban quản lý.