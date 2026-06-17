# HƯỚNG DẪN CHI TIẾT: CHỨC NĂNG QUẢN LÝ LƯƠNG

## 📋 TỔNG QUAN

Chức năng Quản lý Lương là một module quan trọng của hệ thống QLNS, cho phép tính toán, quản lý và theo dõi lương của nhân viên theo từng tháng. Module này tích hợp với dữ liệu chấm công để tính toán lương chính xác và cung cấp báo cáo chi tiết.

## 🏗️ KIẾN TRÚC HỆ THỐNG

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/
├── QuanLyLuongActivity.java                 # Activity chính - Quản lý lương
├── LuongAdapter.java                        # Adapter hiển thị danh sách lương
├── models/Luong.java                        # Model class cho đối tượng Lương
└── database/DatabaseHelper.java             # Xử lý database

app/src/main/res/layout/
├── activity_quan_ly_luong.xml               # Layout chính quản lý lương
└── item_luong.xml                           # Layout item trong ListView
```

## 📊 NGHIỆP VỤ QUẢN LÝ LƯƠNG

### 1. Quy trình nghiệp vụ:
- **Tính lương tự động**: Dựa trên dữ liệu chấm công và mức lương cơ bản
- **Quản lý theo tháng**: Chọn tháng/năm để xem và quản lý
- **Phân quyền truy cập**: Admin/HR/Manager/Employee có quyền khác nhau
- **Thanh toán lương**: Admin/HR có thể đánh dấu đã thanh toán
- **Xuất báo cáo**: Tạo báo cáo chi tiết và xuất PDF
- **Theo dõi trạng thái**: Quản lý trạng thái thanh toán

### 2. Công thức tính lương:
- **Lương cơ bản**: Theo chức vụ và số ngày làm thực tế
- **Phụ cấp**: 10% lương cơ bản
- **Lương tăng ca**: Giờ tăng ca × (Lương cơ bản / 208 giờ) × 1.5
- **Tổng lương**: Lương cơ bản + Phụ cấp + Lương tăng ca

### 3. Phân quyền:
- **Admin**: Full quyền tất cả chức năng
- **HR**: Full quyền tất cả chức năng
- **Manager**: Xem lương nhân viên, xuất báo cáo
- **Employee**: Chỉ xem lương cá nhân

---

## 📱 CHI TIẾT CÁC FILE

## 1️⃣ ACTIVITY CHÍNH - QuanLyLuongActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/QuanLyLuongActivity.java`

### Mục đích:
Activity chính quản lý lương, tính toán lương tự động, hiển thị danh sách và xuất báo cáo.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.app.AlertDialog;                                            // Import AlertDialog để hiển thị dialog
import android.database.Cursor;                                            // Import Cursor để xử lý kết quả query database
import android.os.Bundle;                                                  // Import Bundle để truyền dữ liệu giữa Activity
import android.view.View;                                                  // Import View để xử lý UI components
import android.widget.ArrayAdapter;                                        // Import ArrayAdapter cho Spinner
import android.widget.Button;                                              // Import Button widget
import android.widget.ListView;                                            // Import ListView widget
import android.widget.Spinner;                                             // Import Spinner widget
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo

import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity làm base class

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database
import com.example.btl_mobile_qlns.models.Luong;                           // Import model class Luong

import java.text.SimpleDateFormat;                                         // Import SimpleDateFormat để format ngày
import java.util.ArrayList;                                                // Import ArrayList để lưu danh sách
import java.util.Calendar;                                                 // Import Calendar để xử lý ngày tháng
import java.util.List;                                                     // Import List interface
import java.util.Locale;                                                   // Import Locale để định dạng theo vùng miền
```

#### Khai báo thuộc tính class:
```java
public class QuanLyLuongActivity extends AppCompatActivity {               // Khai báo class kế thừa AppCompatActivity

    private TextView tvTitle, tvThangHienTai;                              // TextView tiêu đề và tháng hiện tại
    private Spinner spThang, spNam;                                        // Spinner chọn tháng và năm
    private Button btnTinhLuong, btnXemLuong;                              // Button tính lương và xuất báo cáo
    private ListView lvLuong;                                              // ListView hiển thị danh sách lương
    private View layoutChonThang;                                          // Layout chọn tháng (không sử dụng trong code hiện tại)
    
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    private LuongAdapter adapter;                                          // Adapter cho ListView
    private List<Luong> listLuong;                                         // Danh sách đối tượng Lương
    private String currentRole;                                            // Vai trò người dùng hiện tại
    private String currentUsername;                                        // Tên đăng nhập người dùng hiện tại
    private boolean isInitialLoad = true;                                  // Flag kiểm tra lần load đầu tiên
```

#### Method onCreate:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi method onCreate của class cha
        setContentView(R.layout.activity_quan_ly_luong);                   // Set layout cho Activity
        
        initViews();                                                       // Khởi tạo các view components
        setupDatabase();                                                   // Thiết lập database và thông tin người dùng
        setupUI();                                                         // Thiết lập giao diện theo vai trò
        setupSpinners();                                                   // Thiết lập các Spinner
        setupButtons();                                                    // Thiết lập các Button
        loadCurrentMonthSalary();                                          // Tải dữ liệu lương tháng hiện tại
    }
```

#### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo và ánh xạ các view từ layout
        tvTitle = findViewById(R.id.tv_title);                             // Ánh xạ TextView tiêu đề
        tvThangHienTai = findViewById(R.id.tv_thang_hien_tai);              // Ánh xạ TextView tháng hiện tại
        spThang = findViewById(R.id.sp_thang);                             // Ánh xạ Spinner chọn tháng
        spNam = findViewById(R.id.sp_nam);                                 // Ánh xạ Spinner chọn năm
        btnTinhLuong = findViewById(R.id.btn_tinh_luong);                  // Ánh xạ Button tính lương
        btnXemLuong = findViewById(R.id.btn_xem_luong);                    // Ánh xạ Button xuất báo cáo
        lvLuong = findViewById(R.id.lv_luong);                             // Ánh xạ ListView danh sách lương
    }
```

#### Method setupDatabase:
```java
    private void setupDatabase() {                                         // Method thiết lập database và lấy thông tin người dùng
        dbHelper = new DatabaseHelper(this);                              // Khởi tạo DatabaseHelper với context hiện tại
        currentRole = getIntent().getStringExtra("role");                 // Lấy vai trò từ Intent
        currentUsername = getIntent().getStringExtra("username");         // Lấy username từ Intent
    }
```

#### Method setupUI:
```java
    private void setupUI() {                                               // Method thiết lập giao diện theo vai trò người dùng
        // Hiển thị tháng hiện tại
        Calendar calendar = Calendar.getInstance();                        // Lấy instance Calendar hiện tại
        SimpleDateFormat sdf = new SimpleDateFormat("MM/yyyy", Locale.getDefault()); // Tạo formatter cho tháng/năm
        tvThangHienTai.setText("Tháng hiện tại: " + sdf.format(calendar.getTime())); // Set text hiển thị tháng hiện tại
        
        if ("Employee".equals(currentRole)) {                              // Nếu là Employee
            // Employee: giao diện xem lương cá nhân
            tvTitle.setText("LƯƠNG CÁ NHÂN");                              // Set tiêu đề cho Employee
            btnTinhLuong.setVisibility(View.GONE);                         // Ẩn button tính lương
            btnXemLuong.setVisibility(View.GONE);                          // Ẩn button xuất báo cáo
        } else if ("Manager".equals(currentRole)) {                        // Nếu là Manager
            // Manager: xem lương nhân viên, không tính lương
            tvTitle.setText("XEM LƯƠNG NHÂN VIÊN");                        // Set tiêu đề cho Manager
            btnTinhLuong.setVisibility(View.GONE);                         // Ẩn button tính lương
            btnXemLuong.setVisibility(View.VISIBLE);                       // Hiển thị button xuất báo cáo
        } else {                                                           // Nếu là Admin/HR
            // Admin/HR: quản lý lương đầy đủ
            tvTitle.setText("QUẢN LÝ LƯƠNG");                              // Set tiêu đề cho Admin/HR
            btnTinhLuong.setVisibility(View.VISIBLE);                      // Hiển thị button tính lương
            btnXemLuong.setVisibility(View.VISIBLE);                       // Hiển thị button xuất báo cáo
        }
    }
```

#### Method setupSpinners:
```java
    private void setupSpinners() {                                         // Method thiết lập các Spinner chọn tháng và năm
        // Setup spinner tháng
        List<String> months = new ArrayList<>();                           // Tạo danh sách tháng
        for (int i = 1; i <= 12; i++) {                                    // Vòng lặp từ tháng 1 đến 12
            months.add(String.format("%02d", i));                          // Thêm tháng với format 2 chữ số (01, 02, ...)
        }
        ArrayAdapter<String> monthAdapter = new ArrayAdapter<>(this,       // Tạo ArrayAdapter cho Spinner tháng
            android.R.layout.simple_spinner_item, months);                 // Sử dụng layout mặc định và danh sách tháng
        monthAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item); // Set layout cho dropdown
        spThang.setAdapter(monthAdapter);                                  // Set adapter cho Spinner tháng
        
        // Setup spinner năm
        List<String> years = new ArrayList<>();                            // Tạo danh sách năm
        int currentYear = Calendar.getInstance().get(Calendar.YEAR);       // Lấy năm hiện tại
        for (int i = currentYear - 2; i <= currentYear + 1; i++) {         // Vòng lặp từ 2 năm trước đến 1 năm sau
            years.add(String.valueOf(i));                                  // Thêm năm vào danh sách
        }
        ArrayAdapter<String> yearAdapter = new ArrayAdapter<>(this,        // Tạo ArrayAdapter cho Spinner năm
            android.R.layout.simple_spinner_item, years);                  // Sử dụng layout mặc định và danh sách năm
        yearAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item); // Set layout cho dropdown
        spNam.setAdapter(yearAdapter);                                     // Set adapter cho Spinner năm
        
        // Set tháng và năm hiện tại
        Calendar calendar = Calendar.getInstance();                        // Lấy instance Calendar hiện tại
        spThang.setSelection(calendar.get(Calendar.MONTH));                // Set selection cho tháng hiện tại (0-based)
        spNam.setSelection(2);                                             // Set selection cho năm hiện tại (index 2 trong danh sách)
        
        // Auto load khi thay đổi spinner
        spThang.setOnItemSelectedListener(new android.widget.AdapterView.OnItemSelectedListener() { // Set listener cho Spinner tháng
            @Override
            public void onItemSelected(android.widget.AdapterView<?> parent, android.view.View view, int position, long id) { // Method được gọi khi chọn item
                if (!isInitialLoad) {                                      // Nếu không phải lần load đầu tiên
                    loadSalaryByMonth();                                   // Tải dữ liệu lương theo tháng được chọn
                }
            }
            @Override
            public void onNothingSelected(android.widget.AdapterView<?> parent) {} // Method được gọi khi không chọn gì (không sử dụng)
        });
        
        spNam.setOnItemSelectedListener(new android.widget.AdapterView.OnItemSelectedListener() { // Set listener cho Spinner năm
            @Override
            public void onItemSelected(android.widget.AdapterView<?> parent, android.view.View view, int position, long id) { // Method được gọi khi chọn item
                if (!isInitialLoad) {                                      // Nếu không phải lần load đầu tiên
                    loadSalaryByMonth();                                   // Tải dữ liệu lương theo tháng được chọn
                }
            }
            @Override
            public void onNothingSelected(android.widget.AdapterView<?> parent) {} // Method được gọi khi không chọn gì (không sử dụng)
        });
    }
```

#### Method setupButtons:
```java
    private void setupButtons() {                                          // Method thiết lập các sự kiện cho button
        btnTinhLuong.setOnClickListener(v -> showCalculateSalaryDialog()); // Set listener cho button tính lương
        btnXemLuong.setOnClickListener(v -> exportSalaryReport());         // Set listener cho button xuất báo cáo
    }
```

#### Method showCalculateSalaryDialog:
```java
    private void showCalculateSalaryDialog() {                             // Method hiển thị dialog xác nhận tính lương
        String thang = spThang.getSelectedItem().toString();               // Lấy tháng được chọn
        String nam = spNam.getSelectedItem().toString();                   // Lấy năm được chọn
        
        // Đảm bảo định dạng tháng đúng
        if (thang.length() == 1) {                                         // Nếu tháng chỉ có 1 chữ số
            thang = "0" + thang;                                           // Thêm số 0 phía trước
        }
        
        String thangNam = nam + "-" + thang;                               // Tạo string tháng năm format yyyy-MM
        
        // Validation: Không cho tính lương tháng tương lai
        Calendar current = Calendar.getInstance();                         // Lấy Calendar hiện tại
        Calendar selected = Calendar.getInstance();                        // Tạo Calendar cho tháng được chọn
        selected.set(Integer.parseInt(nam), Integer.parseInt(thang) - 1, 1); // Set tháng được chọn (month 0-based)
        
        if (selected.after(current)) {                                     // Nếu tháng được chọn sau tháng hiện tại
            Toast.makeText(this, "Không thể tính lương cho tháng tương lai!", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            return;                                                        // Thoát method
        }
        
        // Kiểm tra có dữ liệu chấm công không
        if (!hasAttendanceData(thangNam)) {                                // Nếu không có dữ liệu chấm công
            Toast.makeText(this, "Chưa có dữ liệu chấm công cho tháng " + thang + "/" + nam, Toast.LENGTH_SHORT).show(); // Hiển thị thông báo
            return;                                                        // Thoát method
        }
        
        new AlertDialog.Builder(this)                                      // Tạo AlertDialog builder
            .setTitle("Tính lương tháng " + thang + "/" + nam)             // Set tiêu đề dialog
            .setMessage("Bạn có chắc muốn tính lương cho tất cả nhân viên trong tháng này?\n\n" + // Set message xác nhận
                       "Lưu ý: Nếu đã có dữ liệu lương tháng này, hệ thống sẽ cập nhật lại.")
            .setPositiveButton("Tính lương", (dialog, which) -> calculateSalary(thangNam)) // Set button "Tính lương" với action
            .setNegativeButton("Hủy", null)                                // Set button "Hủy" không có action
            .show();                                                       // Hiển thị dialog
    }
```

#### Method hasAttendanceData:
```java
    private boolean hasAttendanceData(String thangNam) {                   // Method kiểm tra có dữ liệu chấm công không
        Cursor cursor = null;                                              // Khai báo cursor
        try {
            cursor = dbHelper.getReadableDatabase().rawQuery(              // Thực hiện query kiểm tra dữ liệu chấm công
                "SELECT COUNT(*) FROM ChamCong WHERE strftime('%Y-%m', NgayChamCong) = ? AND SoGioLam > 0", // Query đếm số record chấm công có giờ làm > 0
                new String[]{thangNam}                                     // Tham số tháng năm
            );
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                return cursor.getInt(0) > 0;                               // Trả về true nếu có dữ liệu chấm công
            }
        } catch (Exception e) {                                            // Bắt exception
            e.printStackTrace();                                           // In stack trace
        } finally {
            if (cursor != null) cursor.close();                           // Đóng cursor trong finally block
        }
        return false;                                                      // Trả về false nếu không có dữ liệu
    }
```

#### Method calculateSalary:
```java
    private void calculateSalary(String thangNam) {                        // Method thực hiện tính lương
        try {
            int count = dbHelper.calculateMonthlySalary(thangNam);         // Gọi method database tính lương tháng
            if (count > 0) {                                               // Nếu tính lương thành công cho ít nhất 1 nhân viên
                Toast.makeText(this, "Đã tính lương cho " + count + " nhân viên", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                loadSalaryByMonth();                                       // Tải lại dữ liệu lương
            } else {                                                       // Nếu không có nhân viên nào được tính lương
                Toast.makeText(this, "Không có nhân viên nào để tính lương", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo
            }
        } catch (Exception e) {                                            // Bắt exception
            e.printStackTrace();                                           // In stack trace
            Toast.makeText(this, "Lỗi khi tính lương: " + e.getMessage(), Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
        }
    }
```

#### Method loadCurrentMonthSalary:
```java
    private void loadCurrentMonthSalary() {                                // Method tải dữ liệu lương tháng hiện tại
        Calendar calendar = Calendar.getInstance();                        // Lấy Calendar hiện tại
        String thang = String.format("%02d", calendar.get(Calendar.MONTH) + 1); // Format tháng hiện tại (1-based)
        String nam = String.valueOf(calendar.get(Calendar.YEAR));          // Lấy năm hiện tại
        String thangNam = nam + "-" + thang;                               // Tạo string tháng năm
        
        loadSalary(thangNam);                                              // Tải dữ liệu lương
        isInitialLoad = false;                                             // Set flag không còn là lần load đầu tiên
    }
```

#### Method loadSalaryByMonth:
```java
    private void loadSalaryByMonth() {                                     // Method tải dữ liệu lương theo tháng được chọn
        String thang = spThang.getSelectedItem().toString();               // Lấy tháng từ Spinner
        String nam = spNam.getSelectedItem().toString();                   // Lấy năm từ Spinner
        
        // Đảm bảo định dạng tháng đúng (01, 02, ..., 12)
        if (thang.length() == 1) {                                         // Nếu tháng chỉ có 1 chữ số
            thang = "0" + thang;                                           // Thêm số 0 phía trước
        }
        
        String thangNam = nam + "-" + thang;                               // Tạo string tháng năm
        loadSalary(thangNam);                                              // Tải dữ liệu lương
    }
```
#### Method loadSalary:
```java
    private void loadSalary(String thangNam) {                             // Method tải dữ liệu lương từ database
        try {
            listLuong = new ArrayList<>();                                 // Khởi tạo danh sách lương mới
            Cursor cursor;                                                 // Khai báo cursor
            
            if ("Employee".equals(currentRole)) {                          // Nếu là Employee
                // Employee chỉ xem lương của mình
                String maNhanVien = dbHelper.getMaNhanVienByUsername(currentUsername); // Lấy mã nhân viên từ username
                cursor = dbHelper.getSalaryByEmployee(maNhanVien, thangNam); // Lấy lương của nhân viên cụ thể
            } else {                                                       // Nếu là Admin/HR/Manager
                // Admin/HR/Manager xem tất cả
                cursor = dbHelper.getSalaryByMonth(thangNam);              // Lấy lương tất cả nhân viên trong tháng
            }
            
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                do {
                    int maLuong = cursor.getInt(cursor.getColumnIndexOrThrow("MaLuong"));         // Lấy mã lương từ cursor
                    String maNV = cursor.getString(cursor.getColumnIndexOrThrow("MaNhanVien"));   // Lấy mã nhân viên từ cursor
                    String thang = cursor.getString(cursor.getColumnIndexOrThrow("ThangNam"));    // Lấy tháng năm từ cursor
                    double luongCoBan = cursor.getDouble(cursor.getColumnIndexOrThrow("LuongCoBan")); // Lấy lương cơ bản từ cursor
                    double phuCap = cursor.getDouble(cursor.getColumnIndexOrThrow("PhuCap"));     // Lấy phụ cấp từ cursor
                    double soGioLam = cursor.getDouble(cursor.getColumnIndexOrThrow("SoGioLam")); // Lấy số giờ làm từ cursor
                    double tongLuong = cursor.getDouble(cursor.getColumnIndexOrThrow("TongLuong")); // Lấy tổng lương từ cursor
                    String trangThai = cursor.getString(cursor.getColumnIndexOrThrow("TrangThai")); // Lấy trạng thái từ cursor
                    String ngayTinhLuong = cursor.getString(cursor.getColumnIndexOrThrow("NgayTinhLuong")); // Lấy ngày tính lương từ cursor
                    
                    Luong luong = new Luong(maLuong, maNV, thang, luongCoBan, phuCap, soGioLam, tongLuong, trangThai); // Tạo đối tượng Luong
                    luong.setNgayTinhLuong(ngayTinhLuong);                 // Set ngày tính lương
                    
                    // Tính toán thông tin chi tiết
                    DatabaseHelper.AttendanceStats stats = dbHelper.getAttendanceStatsForSalary(maNV, thang); // Lấy thống kê chấm công
                    luong.setSoGioTangCa(stats.soGioTangCa);               // Set số giờ tăng ca
                    luong.setSoNgayLam(stats.soNgayLam);                   // Set số ngày làm
                    
                    // Lấy lương cơ bản gốc từ chức vụ để tính lương tăng ca
                    double luongCoBanGoc = getLuongCoBanGocByMaNV(maNV);   // Lấy lương cơ bản gốc theo chức vụ
                    double luongGio = luongCoBanGoc / 208.0;               // Tính lương theo giờ (26 ngày × 8 giờ)
                    double luongTangCa = stats.soGioTangCa * luongGio * 1.5; // Tính lương tăng ca (x1.5)
                    luong.setLuongTangCa(luongTangCa);                     // Set lương tăng ca
                    
                    // Lấy tên nhân viên (luôn lấy để hiển thị cho cả Employee)
                    String hoTen = dbHelper.getEmployeeNameByMa(maNV);     // Lấy họ tên nhân viên từ mã
                    luong.setHoTen(hoTen);                                 // Set họ tên vào đối tượng Luong
                    
                    listLuong.add(luong);                                  // Thêm đối tượng Luong vào danh sách
                } while (cursor.moveToNext());                             // Lặp đến record tiếp theo
                cursor.close();                                            // Đóng cursor
            }
            
            updateUI();                                                    // Cập nhật giao diện
        } catch (Exception e) {                                            // Bắt exception
            e.printStackTrace();                                           // In stack trace
            Toast.makeText(this, "Lỗi khi tải dữ liệu lương: " + e.getMessage(), Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
        }
    }
```

#### Method updateUI:
```java
    private void updateUI() {                                              // Method cập nhật giao diện ListView
        if (adapter == null) {                                             // Nếu adapter chưa được khởi tạo
            adapter = new LuongAdapter(this, listLuong, currentRole);      // Tạo adapter mới với danh sách lương và vai trò
            lvLuong.setAdapter(adapter);                                   // Set adapter cho ListView
        } else {                                                           // Nếu adapter đã tồn tại
            adapter.updateData(listLuong);                                 // Cập nhật dữ liệu cho adapter
        }
    }
```
#### Method exportSalaryReport:
```java
    private void exportSalaryReport() {                                    // Method xuất báo cáo lương chi tiết
        String thang = spThang.getSelectedItem().toString();               // Lấy tháng từ Spinner
        String nam = spNam.getSelectedItem().toString();                   // Lấy năm từ Spinner
        
        // Đảm bảo định dạng tháng đúng
        if (thang.length() == 1) {                                         // Nếu tháng chỉ có 1 chữ số
            thang = "0" + thang;                                           // Thêm số 0 phía trước
        }
        
        String thangNam = nam + "-" + thang;                               // Tạo string tháng năm
        
        // Tạo báo cáo chi tiết
        StringBuilder report = new StringBuilder();                        // Tạo StringBuilder để xây dựng báo cáo
        report.append("           CÔNG TY QUẢN LÝ NHÂN SỰ           \n");   // Thêm header công ty
        report.append("=============================================\n");   // Thêm đường phân cách
        report.append(String.format("          BÁO CÁO LƯƠNG THÁNG %s/%s          \n", thang, nam)); // Thêm tiêu đề báo cáo
        report.append("=============================================\n\n"); // Thêm đường phân cách và xuống dòng
        
        try {
            Cursor cursor;                                                 // Khai báo cursor
            if ("Employee".equals(currentRole)) {                          // Nếu là Employee
                String maNhanVien = dbHelper.getMaNhanVienByUsername(currentUsername); // Lấy mã nhân viên từ username
                cursor = dbHelper.getSalaryByEmployee(maNhanVien, thangNam); // Lấy lương của nhân viên cụ thể
            } else {                                                       // Nếu là Admin/HR/Manager
                cursor = dbHelper.getSalaryByMonth(thangNam);              // Lấy lương tất cả nhân viên
            }
            
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                double tongLuongCongTy = 0;                                // Biến tính tổng lương công ty
                int soNhanVien = 0;                                        // Biến đếm số nhân viên
                int daThanhToan = 0;                                       // Biến đếm số nhân viên đã thanh toán
                int chuaThanhToan = 0;                                     // Biến đếm số nhân viên chưa thanh toán
                
                do {
                    String maNV = cursor.getString(cursor.getColumnIndexOrThrow("MaNhanVien"));     // Lấy mã nhân viên
                    double luongCoBan = cursor.getDouble(cursor.getColumnIndexOrThrow("LuongCoBan")); // Lấy lương cơ bản
                    double phuCap = cursor.getDouble(cursor.getColumnIndexOrThrow("PhuCap"));       // Lấy phụ cấp
                    double tongLuong = cursor.getDouble(cursor.getColumnIndexOrThrow("TongLuong")); // Lấy tổng lương
                    String trangThai = cursor.getString(cursor.getColumnIndexOrThrow("TrangThai")); // Lấy trạng thái
                    
                    // Lấy thông tin chi tiết
                    DatabaseHelper.AttendanceStats stats = dbHelper.getAttendanceStatsForSalary(maNV, thangNam); // Lấy thống kê chấm công
                    String hoTen = dbHelper.getEmployeeNameByMa(maNV);     // Lấy họ tên nhân viên
                    if (hoTen == null) hoTen = "Không xác định";           // Set default nếu không có tên
                    
                    // Tính lương tăng ca
                    double luongCoBanGoc = getLuongCoBanGocByMaNV(maNV);   // Lấy lương cơ bản gốc
                    double luongGio = luongCoBanGoc / 208.0;               // Tính lương theo giờ
                    double luongTangCa = stats.soGioTangCa * luongGio * 1.5; // Tính lương tăng ca
                    
                    report.append(String.format("NHÂN VIÊN: %s (%s)\n", hoTen.toUpperCase(), maNV)); // Thêm thông tin nhân viên
                    report.append("---------------------------------------------\n"); // Thêm đường phân cách
                    report.append(String.format(" Ngày làm : %-5d | Tổng giờ : %.1f\n", stats.soNgayLam, stats.soGioLam)); // Thêm thông tin ngày làm và giờ làm
                    report.append(String.format(" Tăng ca  : %-5.1f |\n", stats.soGioTangCa)); // Thêm thông tin giờ tăng ca
                    report.append("---------------------------------------------\n"); // Thêm đường phân cách
                    report.append(String.format(" Lương cơ bản  :%15s\n", formatCurrency(luongCoBan))); // Thêm lương cơ bản
                    report.append(String.format(" Lương tăng ca :%15s\n", formatCurrency(luongTangCa))); // Thêm lương tăng ca
                    report.append(String.format(" Phụ cấp       :%15s\n", formatCurrency(phuCap))); // Thêm phụ cấp
                    report.append("---------------------------------------------\n"); // Thêm đường phân cách
                    report.append(String.format(" TỔNG NHẬN     :%15s\n", formatCurrency(tongLuong))); // Thêm tổng lương
                    report.append(String.format(" TRẠNG THÁI    : %s\n", trangThai.toUpperCase())); // Thêm trạng thái
                    report.append("=============================================\n\n"); // Thêm đường phân cách và xuống dòng
                    
                    tongLuongCongTy += tongLuong;                          // Cộng dồn tổng lương công ty
                    soNhanVien++;                                          // Tăng số lượng nhân viên
                    if ("Đã thanh toán".equals(trangThai)) daThanhToan++;  // Tăng số nhân viên đã thanh toán
                    else chuaThanhToan++;                                  // Tăng số nhân viên chưa thanh toán
                    
                } while (cursor.moveToNext());                             // Lặp đến record tiếp theo
                cursor.close();                                            // Đóng cursor
                
                // Thống kê tổng kết
                report.append("TỔNG KẾT DOANH NGHIỆP\n");                  // Thêm tiêu đề tổng kết
                report.append("---------------------------------------------\n"); // Thêm đường phân cách
                report.append(String.format(" Tổng NV         : %d người\n", soNhanVien)); // Thêm tổng số nhân viên
                report.append(String.format(" Tổng chi lương  :%16s\n", formatCurrency(tongLuongCongTy))); // Thêm tổng chi lương
                if (soNhanVien > 0) {                                      // Nếu có nhân viên
                    report.append(String.format(" Lương trung bình:%16s\n", formatCurrency(tongLuongCongTy / soNhanVien))); // Thêm lương trung bình
                }
                report.append(String.format(" Đã thanh toán   : %d/%d\n", daThanhToan, soNhanVien)); // Thêm số nhân viên đã thanh toán
                report.append(String.format(" Chưa thanh toán : %d/%d\n", chuaThanhToan, soNhanVien)); // Thêm số nhân viên chưa thanh toán
                report.append("=============================================\n"); // Thêm đường phân cách
                
            } else {                                                       // Nếu không có dữ liệu
                report.append("Không có dữ liệu lương cho tháng này!\n");  // Thêm thông báo không có dữ liệu
                report.append("Vui lòng tính lương trước khi xuất báo cáo.\n"); // Thêm hướng dẫn
            }
            
        } catch (Exception e) {                                            // Bắt exception
            report.append("Lỗi khi tạo báo cáo: ").append(e.getMessage()).append("\n"); // Thêm thông báo lỗi
        }
        
        report.append("\nNgày xuất: ").append(new java.text.SimpleDateFormat("dd/MM/yyyy HH:mm", java.util.Locale.getDefault()).format(new java.util.Date())); // Thêm ngày xuất báo cáo
        
        // Hiển thị báo cáo trong dialog
        showReportDialog(report.toString(), thang, nam);                   // Gọi method hiển thị dialog báo cáo
    }
```
#### Method showReportDialog:
```java
    private void showReportDialog(String report, String thang, String nam) { // Method hiển thị dialog báo cáo
        AlertDialog.Builder builder = new AlertDialog.Builder(this);       // Tạo AlertDialog builder
        builder.setTitle("Báo cáo lương " + thang + "/" + nam);            // Set tiêu đề dialog
        
        // Tạo ScrollView cho nội dung dài
        android.widget.ScrollView scrollView = new android.widget.ScrollView(this); // Tạo ScrollView để cuộn nội dung
        TextView textView = new TextView(this);                            // Tạo TextView hiển thị báo cáo
        textView.setText(report);                                          // Set nội dung báo cáo
        textView.setTextSize(12);                                          // Set kích thước font 12sp
        textView.setTypeface(android.graphics.Typeface.MONOSPACE);         // Set font monospace để căn chỉnh đều
        textView.setPadding(24, 24, 24, 24);                              // Set padding 24dp
        textView.setTextIsSelectable(true);                                // Cho phép select text
        textView.setLineSpacing(4, 1);                                     // Set khoảng cách dòng
        
        scrollView.addView(textView);                                      // Thêm TextView vào ScrollView
        builder.setView(scrollView);                                       // Set ScrollView làm view của dialog
        
        // Nút Copy
        builder.setPositiveButton("Copy", (dialog, which) -> {             // Set button "Copy"
            android.content.ClipboardManager clipboard = (android.content.ClipboardManager) getSystemService(android.content.Context.CLIPBOARD_SERVICE); // Lấy ClipboardManager
            android.content.ClipData clip = android.content.ClipData.newPlainText("Báo cáo lương", report); // Tạo ClipData với nội dung báo cáo
            clipboard.setPrimaryClip(clip);                                // Set clip vào clipboard
            Toast.makeText(this, "Đã copy báo cáo vào clipboard!", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
        });
        
        // Nút Xuất PDF
        builder.setNeutralButton("Xuất PDF", (dialog, which) -> {          // Set button "Xuất PDF"
            exportToPdf(report, thang, nam);                               // Gọi method xuất PDF
        });
        
        builder.setNegativeButton("Đóng", null);                           // Set button "Đóng" không có action
        
        AlertDialog dialog = builder.create();                             // Tạo dialog
        dialog.show();                                                     // Hiển thị dialog
        
        // Điều chỉnh kích thước dialog
        if (dialog.getWindow() != null) {                                  // Nếu window không null
            dialog.getWindow().setLayout(                                  // Set layout cho window
                (int) (getResources().getDisplayMetrics().widthPixels * 0.95),  // Chiều rộng 95% màn hình
                (int) (getResources().getDisplayMetrics().heightPixels * 0.8)   // Chiều cao 80% màn hình
            );
        }
    }
```

#### Method exportToPdf:
```java
    private void exportToPdf(String reportContent, String thang, String nam) { // Method xuất báo cáo ra file PDF
        android.graphics.pdf.PdfDocument document = new android.graphics.pdf.PdfDocument(); // Tạo PdfDocument mới
        int pageNumber = 1;                                                // Số trang bắt đầu từ 1
        android.graphics.pdf.PdfDocument.PageInfo pageInfo = new android.graphics.pdf.PdfDocument.PageInfo.Builder(595, 842, pageNumber).create(); // Tạo PageInfo với kích thước A4
        android.graphics.pdf.PdfDocument.Page page = document.startPage(pageInfo); // Bắt đầu trang mới

        android.graphics.Canvas canvas = page.getCanvas();                 // Lấy Canvas để vẽ
        android.graphics.Paint paint = new android.graphics.Paint();       // Tạo Paint để set style
        paint.setTypeface(android.graphics.Typeface.MONOSPACE);            // Set font monospace
        paint.setTextSize(12);                                             // Set kích thước font 12
        paint.setColor(android.graphics.Color.BLACK);                      // Set màu chữ đen

        int x = 40, y = 50;                                                // Tọa độ bắt đầu vẽ text
        for (String line : reportContent.split("\n")) {                    // Vòng lặp qua từng dòng trong báo cáo
            canvas.drawText(line, x, y, paint);                            // Vẽ dòng text lên canvas
            y += paint.descent() - paint.ascent();                         // Tăng tọa độ y cho dòng tiếp theo
            if (y > 800) {                                                 // Nếu vượt quá chiều cao trang
                document.finishPage(page);                                 // Kết thúc trang hiện tại
                pageNumber++;                                              // Tăng số trang
                pageInfo = new android.graphics.pdf.PdfDocument.PageInfo.Builder(595, 842, pageNumber).create(); // Tạo PageInfo cho trang mới
                page = document.startPage(pageInfo);                       // Bắt đầu trang mới
                canvas = page.getCanvas();                                 // Lấy Canvas mới
                y = 50;                                                    // Reset tọa độ y
            }
        }
        document.finishPage(page);                                         // Kết thúc trang cuối

        String fileName = "BaoCaoLuong_" + thang + "_" + nam + ".pdf";     // Tạo tên file PDF
        try {
            java.io.OutputStream fos;                                      // Khai báo OutputStream
            if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.Q) { // Nếu Android 10 trở lên
                android.content.ContentValues values = new android.content.ContentValues(); // Tạo ContentValues
                values.put(android.provider.MediaStore.Downloads.DISPLAY_NAME, fileName); // Set tên file
                values.put(android.provider.MediaStore.Downloads.MIME_TYPE, "application/pdf"); // Set MIME type
                values.put(android.provider.MediaStore.Downloads.RELATIVE_PATH, android.os.Environment.DIRECTORY_DOWNLOADS); // Set đường dẫn
                android.net.Uri uri = getContentResolver().insert(android.provider.MediaStore.Downloads.EXTERNAL_CONTENT_URI, values); // Insert vào MediaStore
                fos = getContentResolver().openOutputStream(uri);          // Mở OutputStream từ URI
            } else {                                                       // Nếu Android 9 trở xuống
                java.io.File dir = android.os.Environment.getExternalStoragePublicDirectory(android.os.Environment.DIRECTORY_DOWNLOADS); // Lấy thư mục Downloads
                if (!dir.exists()) dir.mkdirs();                          // Tạo thư mục nếu chưa tồn tại
                java.io.File file = new java.io.File(dir, fileName);      // Tạo File object
                fos = new java.io.FileOutputStream(file);                  // Tạo FileOutputStream
            }
            document.writeTo(fos);                                         // Ghi document vào OutputStream
            document.close();                                              // Đóng document
            if (fos != null) fos.close();                                  // Đóng OutputStream
            Toast.makeText(this, "Đã lưu tệp PDF thành công vào thư mục Tải Xuống (Downloads)!", Toast.LENGTH_LONG).show(); // Hiển thị thông báo thành công
        } catch (Exception e) {                                            // Bắt exception
            e.printStackTrace();                                           // In stack trace
            Toast.makeText(this, "Lỗi khi lưu PDF: " + e.getMessage(), Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
        }
    }
```

#### Method formatCurrency và các method khác:
```java
    private String formatCurrency(double amount) {                         // Method format tiền tệ
        return String.format("%,.0f đ", amount);                          // Trả về string format tiền tệ VNĐ
    }
    
    @Override
    protected void onResume() {                                            // Method được gọi khi Activity quay lại foreground
        super.onResume();                                                  // Gọi method của class cha
        if (!isInitialLoad) {                                              // Nếu không phải lần load đầu tiên
            loadSalaryByMonth();                                           // Tải lại dữ liệu lương
        }
    }
    
    private double getLuongCoBanGocByMaNV(String maNhanVien) {             // Method lấy lương cơ bản gốc theo chức vụ
        Cursor cursor = null;                                              // Khai báo cursor
        try {
            cursor = dbHelper.getReadableDatabase().rawQuery(              // Thực hiện query lấy lương cơ bản từ chức vụ
                "SELECT cv.MucLuongCoBan FROM NhanVien nv " +              // SELECT lương cơ bản từ bảng ChucVu
                "LEFT JOIN ChucVu cv ON nv.MaChucVu = cv.MaChucVu " +      // JOIN với bảng NhanVien
                "WHERE nv.MaNhanVien = ?",                                 // WHERE theo mã nhân viên
                new String[]{maNhanVien}                                   // Tham số mã nhân viên
            );
            if (cursor != null && cursor.moveToFirst()) {                  // Nếu cursor không null và có dữ liệu
                return cursor.getDouble(0);                                // Trả về lương cơ bản
            }
        } catch (Exception e) {                                            // Bắt exception
            e.printStackTrace();                                           // In stack trace
        } finally {
            if (cursor != null) cursor.close();                           // Đóng cursor trong finally block
        }
        return 0;                                                          // Trả về 0 nếu không tìm thấy
    }
}
```

---

## 2️⃣ ADAPTER - LuongAdapter.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/LuongAdapter.java`

### Mục đích:
Adapter quản lý hiển thị danh sách lương trong ListView, xử lý thanh toán và hiển thị thông tin theo vai trò.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.app.AlertDialog;                                            // Import AlertDialog để hiển thị dialog xác nhận
import android.content.Context;                                            // Import Context để truy cập resources và services
import android.view.LayoutInflater;                                        // Import LayoutInflater để inflate layout
import android.view.View;                                                  // Import View để xử lý UI components
import android.view.ViewGroup;                                             // Import ViewGroup để quản lý layout container
import android.widget.BaseAdapter;                                         // Import BaseAdapter làm base class
import android.widget.Button;                                              // Import Button widget
import android.widget.LinearLayout;                                        // Import LinearLayout widget
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database
import com.example.btl_mobile_qlns.models.Luong;                           // Import model class Luong

import java.text.NumberFormat;                                             // Import NumberFormat để định dạng số
import java.util.List;                                                     // Import List interface
import java.util.Locale;                                                   // Import Locale để định dạng theo vùng miền
```
#### Khai báo thuộc tính class:
```java
public class LuongAdapter extends BaseAdapter {                            // Khai báo class kế thừa BaseAdapter
    
    private Context context;                                               // Context của Activity sử dụng adapter
    private List<Luong> listLuong;                                         // Danh sách đối tượng Lương
    private String currentRole;                                            // Vai trò người dùng hiện tại
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    private NumberFormat currencyFormat;                                   // Formatter cho tiền tệ
```

#### Constructor:
```java
    public LuongAdapter(Context context, List<Luong> listLuong, String currentRole) { // Constructor khởi tạo adapter
        this.context = context;                                            // Gán context
        this.listLuong = listLuong;                                        // Gán danh sách lương
        this.currentRole = currentRole;                                    // Gán vai trò người dùng
        this.dbHelper = new DatabaseHelper(context);                      // Khởi tạo DatabaseHelper
        this.currencyFormat = NumberFormat.getCurrencyInstance(new Locale("vi", "VN")); // Khởi tạo formatter tiền tệ Việt Nam
    }
```

#### Method getCount:
```java
    @Override
    public int getCount() {                                                // Method trả về số lượng item trong danh sách
        return listLuong != null ? listLuong.size() : 0;                  // Trả về size của danh sách, nếu null thì trả về 0
    }
```

#### Method getItem:
```java
    @Override
    public Object getItem(int position) {                                  // Method trả về item tại vị trí position
        return listLuong.get(position);                                    // Trả về đối tượng Luong tại vị trí position
    }
```

#### Method getItemId:
```java
    @Override
    public long getItemId(int position) {                                  // Method trả về ID của item tại vị trí position
        return position;                                                   // Trả về chính position làm ID
    }
```

#### Method getView:
```java
    @Override
    public View getView(int position, View convertView, ViewGroup parent) { // Method tạo và trả về View cho item tại vị trí position
        if (convertView == null) {                                         // Kiểm tra convertView có null không (chưa được tạo)
            convertView = LayoutInflater.from(context).inflate(R.layout.item_luong, parent, false); // Inflate layout item_luong
        }
        
        Luong luong = listLuong.get(position);                             // Lấy đối tượng Luong tại vị trí position
        
        TextView tvMaNV = convertView.findViewById(R.id.tv_ma_nv);          // Ánh xạ TextView mã nhân viên
        TextView tvHoTen = convertView.findViewById(R.id.tv_ho_ten);        // Ánh xạ TextView họ tên
        TextView tvThangNam = convertView.findViewById(R.id.tv_thang_nam);  // Ánh xạ TextView tháng năm
        TextView tvLuongCoBan = convertView.findViewById(R.id.tv_luong_co_ban); // Ánh xạ TextView lương cơ bản
        TextView tvPhuCap = convertView.findViewById(R.id.tv_phu_cap);      // Ánh xạ TextView phụ cấp
        TextView tvSoGioLam = convertView.findViewById(R.id.tv_so_gio_lam); // Ánh xạ TextView số giờ làm
        TextView tvSoNgayLam = convertView.findViewById(R.id.tv_so_ngay_lam); // Ánh xạ TextView số ngày làm
        TextView tvGioTangCa = convertView.findViewById(R.id.tv_gio_tang_ca); // Ánh xạ TextView giờ tăng ca
        TextView tvLuongTangCa = convertView.findViewById(R.id.tv_luong_tang_ca); // Ánh xạ TextView lương tăng ca
        TextView tvTongLuong = convertView.findViewById(R.id.tv_tong_luong); // Ánh xạ TextView tổng lương
        TextView tvTrangThai = convertView.findViewById(R.id.tv_trang_thai); // Ánh xạ TextView trạng thái
        LinearLayout layoutButtons = convertView.findViewById(R.id.layout_buttons); // Ánh xạ LinearLayout chứa buttons
        Button btnThanhToan = convertView.findViewById(R.id.btn_thanh_toan); // Ánh xạ Button thanh toán
        
        // Hiển thị thông tin nhân viên
        if (!"Employee".equals(currentRole)) {                             // Nếu không phải Employee
            // Admin/HR/Manager: hiển thị mã NV và họ tên
            tvMaNV.setText("Mã NV: " + luong.getMaNhanVien());             // Set text mã nhân viên
            tvHoTen.setText("Họ tên: " + (luong.getHoTen() != null ? luong.getHoTen() : "N/A")); // Set text họ tên (hoặc N/A nếu null)
            tvMaNV.setVisibility(View.VISIBLE);                            // Hiển thị TextView mã nhân viên
            tvHoTen.setVisibility(View.VISIBLE);                           // Hiển thị TextView họ tên
        } else {                                                           // Nếu là Employee
            // Employee: hiển thị tên của mình
            tvMaNV.setVisibility(View.GONE);                               // Ẩn TextView mã nhân viên
            tvHoTen.setText(luong.getHoTen() != null ? luong.getHoTen() : ""); // Set text họ tên của Employee
            tvHoTen.setVisibility(View.VISIBLE);                           // Hiển thị TextView họ tên
        }
        
        // Hiển thị thông tin lương
        tvThangNam.setText("Tháng: " + luong.getThangNam());               // Set text tháng năm
        tvLuongCoBan.setText("Lương cơ bản: " + currencyFormat.format(luong.getLuongCoBan())); // Set text lương cơ bản với format tiền tệ
        tvPhuCap.setText("Phụ cấp: " + currencyFormat.format(luong.getPhuCap())); // Set text phụ cấp với format tiền tệ
        tvSoGioLam.setText("Số giờ làm: " + String.format("%.1f", luong.getSoGioLam()) + " giờ"); // Set text số giờ làm với 1 chữ số thập phân
        
        // Hiển thị thông tin chi tiết
        tvSoNgayLam.setText("Số ngày: " + luong.getSoNgayLam() + " ngày"); // Set text số ngày làm
        tvGioTangCa.setText("Giờ tăng ca: " + String.format("%.1f", luong.getSoGioTangCa()) + " giờ"); // Set text giờ tăng ca với 1 chữ số thập phân
        tvLuongTangCa.setText("Lương tăng ca: " + currencyFormat.format(luong.getLuongTangCa())); // Set text lương tăng ca với format tiền tệ
        
        tvTongLuong.setText("Tổng lương: " + currencyFormat.format(luong.getTongLuong())); // Set text tổng lương với format tiền tệ
        tvTrangThai.setText("Trạng thái: " + luong.getTrangThai());        // Set text trạng thái
        
        // Thiết lập màu sắc cho trạng thái
        if ("Đã thanh toán".equals(luong.getTrangThai())) {                // Nếu trạng thái là "Đã thanh toán"
            tvTrangThai.setTextColor(context.getResources().getColor(android.R.color.holo_green_dark)); // Set màu xanh lá
        } else {                                                           // Nếu trạng thái khác
            tvTrangThai.setTextColor(context.getResources().getColor(android.R.color.holo_orange_dark)); // Set màu cam
        }
        
        // Hiển thị nút thanh toán cho Admin/HR
        if (("Admin".equals(currentRole) || "HR".equals(currentRole)) &&   // Nếu là Admin hoặc HR
            "Chưa thanh toán".equals(luong.getTrangThai())) {              // Và trạng thái là "Chưa thanh toán"
            layoutButtons.setVisibility(View.VISIBLE);                     // Hiển thị layout buttons
            btnThanhToan.setOnClickListener(v -> showPaymentDialog(luong, position)); // Set listener cho button thanh toán
        } else {                                                           // Nếu không đủ điều kiện
            layoutButtons.setVisibility(View.GONE);                        // Ẩn layout buttons
        }
        
        return convertView;                                                // Trả về View đã được setup
    }
```

#### Method showPaymentDialog:
```java
    private void showPaymentDialog(Luong luong, int position) {            // Method hiển thị dialog xác nhận thanh toán
        new AlertDialog.Builder(context)                                   // Tạo AlertDialog builder
            .setTitle("Xác nhận thanh toán")                               // Set tiêu đề dialog
            .setMessage("Xác nhận thanh toán lương cho nhân viên " + luong.getMaNhanVien() + // Set message xác nhận với thông tin chi tiết
                       " tháng " + luong.getThangNam() + "?\n\n" +
                       "Số tiền: " + currencyFormat.format(luong.getTongLuong()))
            .setPositiveButton("Thanh toán", (dialog, which) -> {          // Set button "Thanh toán" với action
                boolean success = dbHelper.updateSalaryStatus(luong.getMaLuong(), "Đã thanh toán"); // Gọi method database cập nhật trạng thái
                if (success) {                                             // Nếu cập nhật thành công
                    Toast.makeText(context, "Đã thanh toán lương thành công", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                    refreshData();                                         // Refresh dữ liệu
                } else {                                                   // Nếu cập nhật thất bại
                    Toast.makeText(context, "Lỗi khi thanh toán lương", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
                }
            })
            .setNegativeButton("Hủy", null)                                // Set button "Hủy" không có action
            .show();                                                       // Hiển thị dialog
    }
```

#### Method refreshData và updateData:
```java
    private void refreshData() {                                           // Method refresh dữ liệu
        if (context instanceof QuanLyLuongActivity) {                      // Nếu context là QuanLyLuongActivity
            ((QuanLyLuongActivity) context).onResume();                    // Gọi method onResume để reload dữ liệu
        }
    }
    
    public void updateData(List<Luong> newData) {                          // Method cập nhật dữ liệu mới cho adapter
        this.listLuong = newData;                                          // Gán danh sách mới
        notifyDataSetChanged();                                            // Thông báo adapter dữ liệu đã thay đổi để refresh UI
    }
}
```

---

## 3️⃣ MODEL CLASS - Luong.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/models/Luong.java`

### Mục đích:
Model class đại diện cho đối tượng Lương với các thuộc tính và phương thức getter/setter.

### Chi tiết code:

#### Khai báo package và class:
```java
package com.example.btl_mobile_qlns.models;                               // Khai báo package chứa model classes

public class Luong {                                                      // Khai báo class Luong
    private int maLuong;                                                   // Mã lương (primary key)
    private String maNhanVien;                                             // Mã nhân viên (foreign key)
    private String hoTen;                                                  // Họ tên nhân viên (không lưu DB, chỉ để hiển thị)
    private String thangNam;                                               // Tháng năm (format yyyy-MM)
    private double luongCoBan;                                             // Lương cơ bản theo chức vụ
    private double phuCap;                                                 // Phụ cấp (10% lương cơ bản)
    private double soGioLam;                                               // Tổng số giờ làm trong tháng
    private double soGioTangCa;                                            // Số giờ tăng ca (> 8 giờ/ngày)
    private double luongTangCa;                                            // Lương tăng ca (giờ tăng ca × lương giờ × 1.5)
    private double tongLuong;                                              // Tổng lương = lương cơ bản + phụ cấp + lương tăng ca
    private String trangThai;                                              // Trạng thái thanh toán (Chưa thanh toán/Đã thanh toán)
    private String ngayTinhLuong;                                          // Ngày tính lương
    private int soNgayLam;                                                 // Số ngày làm việc thực tế
```

#### Constructor:
```java
    public Luong(int maLuong, String maNhanVien, String thangNam, double luongCoBan, // Constructor khởi tạo đối tượng Luong
                 double phuCap, double soGioLam, double tongLuong, String trangThai) {
        this.maLuong = maLuong;                                            // Gán mã lương
        this.maNhanVien = maNhanVien;                                      // Gán mã nhân viên
        this.thangNam = thangNam;                                          // Gán tháng năm
        this.luongCoBan = luongCoBan;                                      // Gán lương cơ bản
        this.phuCap = phuCap;                                              // Gán phụ cấp
        this.soGioLam = soGioLam;                                          // Gán số giờ làm
        this.tongLuong = tongLuong;                                        // Gán tổng lương
        this.trangThai = trangThai;                                        // Gán trạng thái
    }
```

#### Getter Methods:
```java
    // Getters
    public int getMaLuong() { return maLuong; }                            // Getter cho mã lương
    public String getMaNhanVien() { return maNhanVien; }                   // Getter cho mã nhân viên
    public String getHoTen() { return hoTen; }                             // Getter cho họ tên
    public String getThangNam() { return thangNam; }                       // Getter cho tháng năm
    public double getLuongCoBan() { return luongCoBan; }                   // Getter cho lương cơ bản
    public double getPhuCap() { return phuCap; }                           // Getter cho phụ cấp
    public double getSoGioLam() { return soGioLam; }                       // Getter cho số giờ làm
    public double getSoGioTangCa() { return soGioTangCa; }                 // Getter cho số giờ tăng ca
    public double getLuongTangCa() { return luongTangCa; }                 // Getter cho lương tăng ca
    public double getTongLuong() { return tongLuong; }                     // Getter cho tổng lương
    public String getTrangThai() { return trangThai; }                     // Getter cho trạng thái
    public String getNgayTinhLuong() { return ngayTinhLuong; }             // Getter cho ngày tính lương
    public int getSoNgayLam() { return soNgayLam; }                        // Getter cho số ngày làm
```

#### Setter Methods:
```java
    // Setters
    public void setMaLuong(int maLuong) { this.maLuong = maLuong; }        // Setter cho mã lương
    public void setMaNhanVien(String maNhanVien) { this.maNhanVien = maNhanVien; } // Setter cho mã nhân viên
    public void setHoTen(String hoTen) { this.hoTen = hoTen; }             // Setter cho họ tên
    public void setThangNam(String thangNam) { this.thangNam = thangNam; } // Setter cho tháng năm
    public void setLuongCoBan(double luongCoBan) { this.luongCoBan = luongCoBan; } // Setter cho lương cơ bản
    public void setPhuCap(double phuCap) { this.phuCap = phuCap; }         // Setter cho phụ cấp
    public void setSoGioLam(double soGioLam) { this.soGioLam = soGioLam; } // Setter cho số giờ làm
    public void setSoGioTangCa(double soGioTangCa) { this.soGioTangCa = soGioTangCa; } // Setter cho số giờ tăng ca
    public void setLuongTangCa(double luongTangCa) { this.luongTangCa = luongTangCa; } // Setter cho lương tăng ca
    public void setTongLuong(double tongLuong) { this.tongLuong = tongLuong; } // Setter cho tổng lương
    public void setTrangThai(String trangThai) { this.trangThai = trangThai; } // Setter cho trạng thái
    public void setNgayTinhLuong(String ngayTinhLuong) { this.ngayTinhLuong = ngayTinhLuong; } // Setter cho ngày tính lương
    public void setSoNgayLam(int soNgayLam) { this.soNgayLam = soNgayLam; } // Setter cho số ngày làm
}
```
---

## 📋 CHI TIẾT CÁC FILE LAYOUT

## 4️⃣ LAYOUT CHÍNH - activity_quan_ly_luong.xml

**Đường dẫn**: `app/src/main/res/layout/activity_quan_ly_luong.xml`

### Mục đích:
Layout chính cho Activity quản lý lương, bao gồm header, chọn tháng/năm, buttons và ListView.

### Chi tiết code:

#### Khai báo XML và LinearLayout chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- LinearLayout root với namespace Android -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent (full width) -->
    android:layout_height="match_parent"                                   <!-- Chiều cao bằng parent (full height) -->
    android:orientation="vertical"                                         <!-- Orientation dọc (các child xếp từ trên xuống) -->
    android:background="#f5f5f5">                                          <!-- Background màu xám nhạt (#f5f5f5) -->
```

#### Header Section:
```xml
    <!-- Header -->
    <LinearLayout                                                          <!-- LinearLayout chứa phần header -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="vertical"                                     <!-- Orientation dọc -->
        android:background="@android:color/white"                          <!-- Background màu trắng -->
        android:padding="16dp"                                             <!-- Padding 16dp cho tất cả các cạnh -->
        android:elevation="2dp">                                           <!-- Elevation 2dp tạo shadow -->

        <TextView                                                          <!-- TextView tiêu đề chính -->
            android:id="@+id/tv_title"                                     <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="QUẢN LÝ LƯƠNG"                                   <!-- Text tiêu đề mặc định -->
            android:textSize="24sp"                                        <!-- Kích thước font 24sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:textColor="#2196F3"                                    <!-- Màu chữ xanh dương -->
            android:gravity="center"                                       <!-- Căn giữa text -->
            android:layout_marginBottom="16dp" />                          <!-- Margin bottom 16dp -->

        <TextView                                                          <!-- TextView hiển thị tháng hiện tại -->
            android:id="@+id/tv_thang_hien_tai"                            <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Tháng hiện tại: 01/2024"                         <!-- Text mẫu hiển thị tháng hiện tại -->
            android:textSize="16sp"                                        <!-- Kích thước font 16sp -->
            android:textColor="#666"                                       <!-- Màu chữ xám -->
            android:gravity="center"                                       <!-- Căn giữa text -->
            android:layout_marginBottom="16dp" />                          <!-- Margin bottom 16dp -->
```

#### Month/Year Selection Section:
```xml
        <!-- Chọn tháng/năm -->
        <LinearLayout                                                      <!-- LinearLayout chứa phần chọn tháng/năm -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="horizontal"                               <!-- Orientation ngang -->
            android:layout_marginBottom="12dp">                            <!-- Margin bottom 12dp -->

            <TextView                                                      <!-- Label cho Spinner tháng -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ chứa text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="Tháng:"                                      <!-- Text label -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:textColor="#333"                                   <!-- Màu chữ xám đậm -->
                android:layout_gravity="center_vertical"                   <!-- Căn giữa theo chiều dọc -->
                android:layout_marginEnd="8dp" />                          <!-- Margin end 8dp -->

            <Spinner                                                       <!-- Spinner chọn tháng -->
                android:id="@+id/sp_thang"                                 <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:layout_weight="1"                                  <!-- Weight 1 để chia đều không gian -->
                android:background="@drawable/edit_text_background"        <!-- Background custom từ drawable -->
                android:padding="12dp"                                     <!-- Padding 12dp cho tất cả các cạnh -->
                android:layout_marginEnd="8dp" />                          <!-- Margin end 8dp -->

            <TextView                                                      <!-- Label cho Spinner năm -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ chứa text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="Năm:"                                        <!-- Text label -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:textColor="#333"                                   <!-- Màu chữ xám đậm -->
                android:layout_gravity="center_vertical"                   <!-- Căn giữa theo chiều dọc -->
                android:layout_marginEnd="8dp" />                          <!-- Margin end 8dp -->

            <Spinner                                                       <!-- Spinner chọn năm -->
                android:id="@+id/sp_nam"                                   <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:layout_weight="1"                                  <!-- Weight 1 để chia đều không gian -->
                android:background="@drawable/edit_text_background"        <!-- Background custom từ drawable -->
                android:padding="12dp" />                                  <!-- Padding 12dp cho tất cả các cạnh -->

        </LinearLayout>
```

#### Buttons Section:
```xml
        <!-- Buttons -->
        <LinearLayout                                                      <!-- LinearLayout chứa các button -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="horizontal">                              <!-- Orientation ngang -->

            <Button                                                        <!-- Button tính lương -->
                android:id="@+id/btn_tinh_luong"                           <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:layout_weight="1"                                  <!-- Weight 1 để chia đều không gian -->
                android:text="TÍNH LƯƠNG"                                  <!-- Text button -->
                android:textColor="@color/white"                           <!-- Màu chữ trắng -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:background="@drawable/btn_primary"                 <!-- Background custom từ drawable -->
                android:layout_marginEnd="8dp"                             <!-- Margin end 8dp -->
                android:visibility="gone" />                               <!-- Visibility gone (ẩn mặc định) -->

            <Button                                                        <!-- Button xuất báo cáo -->
                android:id="@+id/btn_xem_luong"                            <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:layout_weight="1"                                  <!-- Weight 1 để chia đều không gian -->
                android:text="XUẤT BÁO CÁO"                                <!-- Text button -->
                android:textColor="@color/white"                           <!-- Màu chữ trắng -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:background="@drawable/btn_success"                 <!-- Background custom màu xanh lá từ drawable -->
                android:layout_marginStart="8dp" />                        <!-- Margin start 8dp -->

        </LinearLayout>

    </LinearLayout>
```

#### ListView Section:
```xml
    <!-- Danh sách lương -->
    <ListView                                                              <!-- ListView hiển thị danh sách lương -->
        android:id="@+id/lv_luong"                                         <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="0dp"                                        <!-- Chiều cao 0dp để sử dụng layout_weight -->
        android:layout_weight="1"                                          <!-- Weight 1 để chiếm không gian còn lại -->
        android:layout_margin="16dp"                                       <!-- Margin 16dp cho tất cả các cạnh -->
        android:background="@android:color/white"                          <!-- Background màu trắng -->
        android:divider="#E0E0E0"                                          <!-- Màu divider xám nhạt -->
        android:dividerHeight="1dp"                                        <!-- Chiều cao divider 1dp -->
        android:elevation="2dp" />                                         <!-- Elevation 2dp tạo shadow -->

</LinearLayout>
```

---

## 5️⃣ LAYOUT ITEM - item_luong.xml

**Đường dẫn**: `app/src/main/res/layout/item_luong.xml`

### Mục đích:
Layout cho từng item lương trong ListView, hiển thị thông tin chi tiết và button thanh toán.

### Chi tiết code:

#### Khai báo XML và LinearLayout chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- LinearLayout root với namespace Android -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent -->
    android:layout_height="wrap_content"                                   <!-- Chiều cao vừa đủ chứa content -->
    android:orientation="vertical"                                         <!-- Orientation dọc -->
    android:padding="16dp"                                                 <!-- Padding 16dp cho tất cả các cạnh -->
    android:background="@android:color/white">                             <!-- Background màu trắng -->
```

#### Employee Information Section:
```xml
    <!-- Thông tin nhân viên (chỉ hiện với Admin/HR) -->
    <TextView                                                              <!-- TextView hiển thị mã nhân viên -->
        android:id="@+id/tv_ma_nv"                                         <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Mã NV: NV001"                                        <!-- Text mẫu hiển thị -->
        android:textSize="14sp"                                            <!-- Kích thước font 14sp -->
        android:textStyle="bold"                                           <!-- Style chữ đậm -->
        android:textColor="#2196F3"                                        <!-- Màu chữ xanh dương -->
        android:layout_marginBottom="4dp"                                  <!-- Margin bottom 4dp -->
        android:visibility="gone" />                                       <!-- Visibility gone (ẩn mặc định) -->

    <TextView                                                              <!-- TextView hiển thị họ tên nhân viên -->
        android:id="@+id/tv_ho_ten"                                        <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Họ tên: Nguyễn Văn A"                                <!-- Text mẫu hiển thị -->
        android:textSize="16sp"                                            <!-- Kích thước font 16sp -->
        android:textStyle="bold"                                           <!-- Style chữ đậm -->
        android:textColor="#333"                                           <!-- Màu chữ xám đậm -->
        android:layout_marginBottom="8dp"                                  <!-- Margin bottom 8dp -->
        android:visibility="gone" />                                       <!-- Visibility gone (ẩn mặc định) -->
```

#### Salary Period Section:
```xml
    <!-- Thông tin lương -->
    <TextView                                                              <!-- TextView hiển thị tháng năm -->
        android:id="@+id/tv_thang_nam"                                     <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Tháng: 2024-01"                                      <!-- Text mẫu hiển thị -->
        android:textSize="14sp"                                            <!-- Kích thước font 14sp -->
        android:textStyle="bold"                                           <!-- Style chữ đậm -->
        android:textColor="#333"                                           <!-- Màu chữ xám đậm -->
        android:layout_marginBottom="8dp" />                               <!-- Margin bottom 8dp -->
```

#### Salary Details Section:
```xml
    <LinearLayout                                                          <!-- LinearLayout chứa chi tiết lương với background -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="vertical"                                     <!-- Orientation dọc -->
        android:background="#f5f5f5"                                       <!-- Background màu xám nhạt -->
        android:padding="12dp"                                             <!-- Padding 12dp cho tất cả các cạnh -->
        android:layout_marginBottom="8dp">                                 <!-- Margin bottom 8dp -->

        <TextView                                                          <!-- TextView hiển thị lương cơ bản -->
            android:id="@+id/tv_luong_co_ban"                              <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Lương cơ bản: 12.000.000 ₫"                      <!-- Text mẫu hiển thị -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textColor="#666"                                       <!-- Màu chữ xám -->
            android:layout_marginBottom="4dp" />                           <!-- Margin bottom 4dp -->

        <TextView                                                          <!-- TextView hiển thị phụ cấp -->
            android:id="@+id/tv_phu_cap"                                   <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Phụ cấp: 2.000.000 ₫"                            <!-- Text mẫu hiển thị -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textColor="#666"                                       <!-- Màu chữ xám -->
            android:layout_marginBottom="4dp" />                           <!-- Margin bottom 4dp -->

        <TextView                                                          <!-- TextView hiển thị số giờ làm -->
            android:id="@+id/tv_so_gio_lam"                                <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Số giờ làm: 176.0 giờ"                           <!-- Text mẫu hiển thị -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textColor="#666"                                       <!-- Màu chữ xám -->
            android:layout_marginBottom="4dp" />                           <!-- Margin bottom 4dp -->

        <LinearLayout                                                      <!-- LinearLayout chứa số ngày làm và giờ tăng ca -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="horizontal">                              <!-- Orientation ngang -->

            <TextView                                                      <!-- TextView hiển thị số ngày làm -->
                android:id="@+id/tv_so_ngay_lam"                           <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:layout_weight="1"                                  <!-- Weight 1 để chia đều không gian -->
                android:text="Số ngày: 22 ngày"                            <!-- Text mẫu hiển thị -->
                android:textSize="14sp"                                    <!-- Kích thước font 14sp -->
                android:textColor="#666" />                                <!-- Màu chữ xám -->

            <TextView                                                      <!-- TextView hiển thị giờ tăng ca -->
                android:id="@+id/tv_gio_tang_ca"                           <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:layout_weight="1"                                  <!-- Weight 1 để chia đều không gian -->
                android:text="Giờ tăng ca: 8.0 giờ"                        <!-- Text mẫu hiển thị -->
                android:textSize="14sp"                                    <!-- Kích thước font 14sp -->
                android:textColor="#FF9800"                                <!-- Màu chữ cam -->
                android:gravity="end" />                                   <!-- Căn phải -->

        </LinearLayout>

        <TextView                                                          <!-- TextView hiển thị lương tăng ca -->
            android:id="@+id/tv_luong_tang_ca"                             <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Lương tăng ca: 865.385 ₫"                        <!-- Text mẫu hiển thị -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textColor="#FF9800"                                    <!-- Màu chữ cam -->
            android:layout_marginTop="4dp" />                              <!-- Margin top 4dp -->

    </LinearLayout>
```

#### Total Salary and Status Section:
```xml
    <!-- Tổng lương -->
    <TextView                                                              <!-- TextView hiển thị tổng lương -->
        android:id="@+id/tv_tong_luong"                                    <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Tổng lương: 14.000.000 ₫"                           <!-- Text mẫu hiển thị -->
        android:textSize="18sp"                                            <!-- Kích thước font 18sp -->
        android:textStyle="bold"                                           <!-- Style chữ đậm -->
        android:textColor="#4CAF50"                                        <!-- Màu chữ xanh lá -->
        android:layout_marginBottom="8dp" />                               <!-- Margin bottom 8dp -->

    <!-- Trạng thái -->
    <TextView                                                              <!-- TextView hiển thị trạng thái thanh toán -->
        android:id="@+id/tv_trang_thai"                                    <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Trạng thái: Chưa thanh toán"                         <!-- Text mẫu hiển thị -->
        android:textSize="14sp"                                            <!-- Kích thước font 14sp -->
        android:textStyle="bold" />                                        <!-- Style chữ đậm -->
```

#### Payment Button Section:
```xml
    <!-- Button thanh toán -->
    <LinearLayout                                                          <!-- LinearLayout chứa button thanh toán -->
        android:id="@+id/layout_buttons"                                   <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="horizontal"                                   <!-- Orientation ngang -->
        android:gravity="end"                                              <!-- Căn phải -->
        android:layout_marginTop="12dp"                                    <!-- Margin top 12dp -->
        android:visibility="gone">                                         <!-- Visibility gone (ẩn mặc định) -->

        <Button                                                            <!-- Button thanh toán -->
            android:id="@+id/btn_thanh_toan"                               <!-- ID để truy cập từ Java code -->
            android:layout_width="wrap_content"                            <!-- Chiều rộng vừa đủ chứa text -->
            android:layout_height="32dp"                                   <!-- Chiều cao cố định 32dp -->
            android:text="THANH TOÁN"                                      <!-- Text button -->
            android:textSize="12sp"                                        <!-- Kích thước font 12sp -->
            android:textColor="@android:color/white"                       <!-- Màu chữ trắng -->
            android:background="@drawable/btn_primary"                     <!-- Background custom từ drawable -->
            android:minWidth="100dp" />                                    <!-- Chiều rộng tối thiểu 100dp -->

    </LinearLayout>

</LinearLayout>
```
---

## 🔗 TÍCH HỢP VÀ ĐIỀU HƯỚNG

### 1. Tích hợp với Dashboard:
- **Quyền truy cập**: Admin/HR có full quyền, Manager xem và xuất báo cáo, Employee chỉ xem lương cá nhân
- **Button điều hướng**: "QUẢN LÝ LƯƠNG" trong DashboardActivity
- **Intent navigation**: Truyền role và username qua Intent để phân quyền

### 2. Tích hợp với Database:
- **Salary calculation methods**: calculateMonthlySalary(), getSalaryByMonth(), getSalaryByEmployee()
- **Attendance integration**: getAttendanceStatsForSalary() để lấy thống kê chấm công
- **Employee integration**: getEmployeeNameByMa(), getMaNhanVienByUsername()
- **Status management**: updateSalaryStatus() để cập nhật trạng thái thanh toán

### 3. Navigation Flow:
```
DashboardActivity → QuanLyLuongActivity
                         ↓
                   LuongAdapter (ListView items)
                         ↓
                   Payment Dialog / Report Dialog
```

---

## 📋 CÁC PHƯƠNG THỨC DATABASE LIÊN QUAN

### 1. Salary Calculation Methods:
- `calculateMonthlySalary(String thangNam)`: Tính lương tự động cho tất cả nhân viên trong tháng
- `getSalaryByMonth(String thangNam)`: Lấy danh sách lương tất cả nhân viên trong tháng
- `getSalaryByEmployee(String maNV, String thangNam)`: Lấy lương của nhân viên cụ thể
- `updateSalaryStatus(int maLuong, String trangThai)`: Cập nhật trạng thái thanh toán

### 2. Attendance Integration Methods:
- `getAttendanceStatsForSalary(String maNV, String thangNam)`: Lấy thống kê chấm công để tính lương
- `getTotalWorkingHours(String maNV, String thangNam)`: Tính tổng giờ làm việc
- Tích hợp với bảng ChamCong để lấy dữ liệu thực tế

### 3. Employee Integration Methods:
- `getEmployeeNameByMa(String maNV)`: Lấy tên nhân viên từ mã
- `getMaNhanVienByUsername(String username)`: Lấy mã nhân viên từ username
- `getSalaryByEmployee(String maNV)`: Lấy mức lương cơ bản theo chức vụ

---

## 🎯 QUY TẮC NGHIỆP VỤ

### 1. Salary Calculation Rules:
- **Lương cơ bản**: Dựa trên chức vụ và số ngày làm thực tế (tỷ lệ theo 26 ngày/tháng)
- **Phụ cấp**: 10% của lương cơ bản
- **Lương tăng ca**: (Giờ tăng ca) × (Lương cơ bản / 208 giờ) × 1.5
- **Tổng lương**: Lương cơ bản + Phụ cấp + Lương tăng ca

### 2. Business Logic:
- **Validation**: Không cho tính lương tháng tương lai
- **Data dependency**: Phải có dữ liệu chấm công trước khi tính lương
- **Auto-update**: Nếu đã có lương tháng đó, hệ thống sẽ cập nhật lại
- **Role-based access**: Phân quyền rõ ràng theo vai trò

### 3. Payment Management:
- **Status tracking**: "Chưa thanh toán" / "Đã thanh toán"
- **Authorization**: Chỉ Admin/HR mới có quyền thanh toán
- **Confirmation**: Hiển thị dialog xác nhận trước khi thanh toán
- **Audit trail**: Lưu ngày tính lương để theo dõi

---

## 🎨 THIẾT KẾ UI/UX

### 1. Design Principles:
- **Role-based UI**: Giao diện thay đổi theo vai trò người dùng
- **Information hierarchy**: Thông tin quan trọng được highlight
- **Color coding**: Màu sắc phân biệt trạng thái (xanh lá = đã thanh toán, cam = chưa thanh toán)
- **Responsive layout**: Tự động điều chỉnh theo kích thước màn hình

### 2. Interactive Elements:
- **Auto-reload**: Tự động tải lại khi thay đổi tháng/năm
- **Confirmation dialogs**: Xác nhận trước các thao tác quan trọng
- **Progress feedback**: Toast messages thông báo kết quả
- **Export functionality**: Xuất báo cáo PDF và copy clipboard

### 3. Data Visualization:
- **Detailed breakdown**: Hiển thị chi tiết từng khoản lương
- **Summary statistics**: Thống kê tổng quan trong báo cáo
- **Currency formatting**: Format tiền tệ VNĐ chuẩn
- **Professional reports**: Báo cáo có header và footer chuyên nghiệp

---

## 📊 TÍNH NĂNG XUẤT BÁO CÁO

### 1. Report Features:
- **Comprehensive data**: Thông tin chi tiết từng nhân viên
- **Company statistics**: Tổng kết doanh nghiệp
- **Professional formatting**: Layout chuyên nghiệp với monospace font
- **Multiple export options**: Copy clipboard và xuất PDF

### 2. PDF Export:
- **Multi-page support**: Tự động chia trang khi nội dung dài
- **Standard format**: Kích thước A4 chuẩn
- **File naming**: Tên file theo format BaoCaoLuong_MM_YYYY.pdf
- **Storage location**: Lưu vào thư mục Downloads

### 3. Report Content:
- **Employee details**: Mã NV, họ tên, số ngày làm, giờ tăng ca
- **Salary breakdown**: Lương cơ bản, tăng ca, phụ cấp, tổng lương
- **Payment status**: Trạng thái thanh toán từng nhân viên
- **Company summary**: Tổng NV, tổng chi lương, lương trung bình

---

## 🔧 KẾT LUẬN

Module Quản lý Lương là một trong những module phức tạp và quan trọng nhất của hệ thống QLNS. Với khả năng tính toán lương tự động, phân quyền chi tiết và báo cáo chuyên nghiệp, module này đảm bảo:

- **Tính chính xác**: Công thức tính lương rõ ràng, tích hợp với dữ liệu chấm công thực tế
- **Bảo mật**: Phân quyền nghiêm ngặt theo vai trò, Employee chỉ xem được lương của mình
- **Tiện dụng**: Giao diện thân thiện, tự động tải dữ liệu, xuất báo cáo PDF
- **Chuyên nghiệp**: Báo cáo chi tiết, format chuẩn, phù hợp với yêu cầu doanh nghiệp
- **Khả năng mở rộng**: Dễ dàng thêm các loại phụ cấp, bonus khác

Module này tích hợp seamlessly với module Chấm công và cung cấp foundation vững chắc cho việc quản lý lương bổng trong doanh nghiệp, đáp ứng đầy đủ nhu cầu từ tính toán đến báo cáo và thanh toán.