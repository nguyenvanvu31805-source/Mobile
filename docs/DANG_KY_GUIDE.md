# HƯỚNG DẪN CHI TIẾT: CHỨC NĂNG ĐĂNG KÝ

## 📋 TỔNG QUAN

Chức năng Đăng ký cho phép nhân viên mới tự tạo tài khoản trong hệ thống QLNS. Module này tích hợp việc tạo hồ sơ nhân viên và tài khoản đăng nhập trong một quy trình duy nhất, đảm bảo tính toàn vẹn dữ liệu và đơn giản hóa quy trình onboarding.

## 🏗️ KIẾN TRÚC HỆ THỐNG

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/
├── RegisterActivity.java                    # Activity chính - Xử lý đăng ký
└── database/DatabaseHelper.java             # Xử lý database và validation

app/src/main/res/layout/
└── activity_register.xml                    # Layout form đăng ký

Lưu ý: Module này không có model class riêng, sử dụng trực tiếp database
```

## 📊 NGHIỆP VỤ ĐĂNG KÝ

### 1. Quy trình nghiệp vụ:
- **Thu thập thông tin**: Form đầy đủ thông tin nhân viên và tài khoản
- **Auto-generation**: Tự động tạo mã nhân viên tiếp theo
- **Validation**: Kiểm tra dữ liệu đầu vào và tính duy nhất
- **Database transaction**: Tạo đồng thời record NhanVien và TaiKhoan
- **Role assignment**: Tự động gán vai trò "Employee" cho tài khoản mới
- **Success feedback**: Thông báo kết quả và quay về màn hình đăng nhập

### 2. Phân loại thông tin:
- **Thông tin nhân viên**: Mã NV, họ tên, ngày sinh, giới tính, SĐT, email, ngày vào làm, phòng ban
- **Thông tin tài khoản**: Tên đăng nhập, mật khẩu, xác nhận mật khẩu
- **Thông tin mặc định**: Chức vụ "Nhân viên", vai trò "Employee", trạng thái "Đang làm việc"

### 3. Bảo mật và validation:
- **Unique constraints**: Kiểm tra mã nhân viên và tên đăng nhập không trùng lặp
- **Password policy**: Mật khẩu tối thiểu 6 ký tự
- **Email validation**: Kiểm tra format email hợp lệ
- **Required fields**: Tất cả trường đều bắt buộc nhập

---

## 📱 CHI TIẾT CÁC FILE

## 1️⃣ ACTIVITY CHÍNH - RegisterActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/RegisterActivity.java`

### Mục đích:
Activity chính xử lý đăng ký nhân viên mới, bao gồm thu thập thông tin, validation và tạo tài khoản.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.app.DatePickerDialog;                                       // Import DatePickerDialog để chọn ngày
import android.database.Cursor;                                            // Import Cursor để xử lý kết quả query database
import android.os.Bundle;                                                  // Import Bundle để truyền dữ liệu giữa Activity
import android.view.View;                                                  // Import View để xử lý UI components
import android.widget.ArrayAdapter;                                        // Import ArrayAdapter cho Spinner
import android.widget.Button;                                              // Import Button widget
import android.widget.EditText;                                            // Import EditText widget
import android.widget.RadioButton;                                         // Import RadioButton widget
import android.widget.RadioGroup;                                          // Import RadioGroup widget
import android.widget.Spinner;                                             // Import Spinner widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo

import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity làm base class

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database

import java.text.SimpleDateFormat;                                         // Import SimpleDateFormat để format ngày
import java.util.ArrayList;                                                // Import ArrayList để lưu danh sách
import java.util.Calendar;                                                 // Import Calendar để xử lý ngày tháng
import java.util.List;                                                     // Import List interface
import java.util.Locale;                                                   // Import Locale để định dạng theo vùng miền
```

#### Khai báo thuộc tính class:
```java
public class RegisterActivity extends AppCompatActivity {                  // Khai báo class kế thừa AppCompatActivity
    
    private EditText etMaNhanVien, etHoTen, etNgaySinh, etSoDienThoai, etEmail, etNgayVaoLam; // EditText cho thông tin nhân viên
    private EditText etTenDangNhap, etMatKhau, etXacNhanMatKhau;           // EditText cho thông tin tài khoản
    private RadioGroup rgGioiTinh;                                         // RadioGroup cho giới tính
    private Spinner spPhongBan;                                            // Spinner chọn phòng ban
    private Button btnDangKy, btnHuy;                                      // Button đăng ký và hủy
    
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    private Calendar calendar;                                             // Calendar để xử lý ngày tháng
    private SimpleDateFormat dateFormat;                                   // SimpleDateFormat để format ngày
```

#### Method onCreate:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi method onCreate của class cha
        setContentView(R.layout.activity_register);                        // Set layout cho Activity
        
        initViews();                                                       // Khởi tạo các view components
        setupDatabase();                                                   // Thiết lập database và auto-generate mã NV
        setupSpinners();                                                   // Thiết lập Spinner phòng ban
        setupDatePickers();                                                // Thiết lập date picker cho ngày
        setupButtons();                                                    // Thiết lập các button events
    }
```

#### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo và ánh xạ các view từ layout
        etMaNhanVien = findViewById(R.id.et_ma_nhan_vien);                 // Ánh xạ EditText mã nhân viên
        etHoTen = findViewById(R.id.et_ho_ten);                            // Ánh xạ EditText họ tên
        etNgaySinh = findViewById(R.id.et_ngay_sinh);                      // Ánh xạ EditText ngày sinh
        etSoDienThoai = findViewById(R.id.et_so_dien_thoai);               // Ánh xạ EditText số điện thoại
        etEmail = findViewById(R.id.et_email);                             // Ánh xạ EditText email
        etNgayVaoLam = findViewById(R.id.et_ngay_vao_lam);                 // Ánh xạ EditText ngày vào làm
        etTenDangNhap = findViewById(R.id.et_ten_dang_nhap);               // Ánh xạ EditText tên đăng nhập
        etMatKhau = findViewById(R.id.et_mat_khau);                        // Ánh xạ EditText mật khẩu
        etXacNhanMatKhau = findViewById(R.id.et_xac_nhan_mat_khau);        // Ánh xạ EditText xác nhận mật khẩu
        
        rgGioiTinh = findViewById(R.id.rg_gioi_tinh);                      // Ánh xạ RadioGroup giới tính
        spPhongBan = findViewById(R.id.sp_phong_ban);                      // Ánh xạ Spinner phòng ban
        
        btnDangKy = findViewById(R.id.btn_dang_ky);                        // Ánh xạ Button đăng ký
        btnHuy = findViewById(R.id.btn_huy);                               // Ánh xạ Button hủy
        
        calendar = Calendar.getInstance();                                 // Khởi tạo Calendar instance
        dateFormat = new SimpleDateFormat("yyyy-MM-dd", Locale.getDefault()); // Khởi tạo SimpleDateFormat với format yyyy-MM-dd
    }
```

#### Method setupDatabase:
```java
    private void setupDatabase() {                                         // Method thiết lập database và auto-generate mã nhân viên
        dbHelper = new DatabaseHelper(this);                              // Khởi tạo DatabaseHelper với context hiện tại
        
        // Tự động tạo mã nhân viên và disable input
        etMaNhanVien.setText(dbHelper.getNextEmployeeCode());             // Set mã nhân viên tiếp theo từ database
        etMaNhanVien.setEnabled(false);                                    // Disable EditText mã nhân viên (không cho sửa)
    }
```

#### Method setupSpinners:
```java
    private void setupSpinners() {                                         // Method thiết lập Spinner phòng ban
        // Setup Phòng ban spinner
        List<String> phongBanList = new ArrayList<>();                     // Tạo danh sách tên phòng ban
        List<String> phongBanIds = new ArrayList<>();                      // Tạo danh sách mã phòng ban
        
        Cursor cursor = dbHelper.getAllDepartments();                      // Lấy tất cả phòng ban từ database
        if (cursor.moveToFirst()) {                                        // Nếu cursor có dữ liệu
            do {
                String maPhongBan = cursor.getString(cursor.getColumnIndexOrThrow("MaPhongBan"));     // Lấy mã phòng ban từ cursor
                String tenPhongBan = cursor.getString(cursor.getColumnIndexOrThrow("TenPhongBan"));   // Lấy tên phòng ban từ cursor
                phongBanList.add(tenPhongBan);                             // Thêm tên phòng ban vào danh sách hiển thị
                phongBanIds.add(maPhongBan);                               // Thêm mã phòng ban vào danh sách ID
            } while (cursor.moveToNext());                                 // Lặp đến record tiếp theo
        }
        cursor.close();                                                    // Đóng cursor
        
        ArrayAdapter<String> phongBanAdapter = new ArrayAdapter<>(this,    // Tạo ArrayAdapter cho Spinner
                android.R.layout.simple_spinner_item, phongBanList);       // Sử dụng layout mặc định và danh sách tên phòng ban
        phongBanAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item); // Set layout cho dropdown
        spPhongBan.setAdapter(phongBanAdapter);                            // Set adapter cho Spinner
        spPhongBan.setTag(phongBanIds);                                    // Lưu danh sách ID vào tag để sử dụng sau
    }
```

#### Method setupDatePickers:
```java
    private void setupDatePickers() {                                      // Method thiết lập date picker cho các EditText ngày
        etNgaySinh.setOnClickListener(v -> showDatePicker(etNgaySinh));    // Set listener cho EditText ngày sinh
        etNgayVaoLam.setOnClickListener(v -> showDatePicker(etNgayVaoLam)); // Set listener cho EditText ngày vào làm
        
        // Set ngày vào làm mặc định là hôm nay
        etNgayVaoLam.setText(dateFormat.format(calendar.getTime()));       // Set ngày vào làm mặc định là ngày hiện tại
    }
```

#### Method showDatePicker:
```java
    private void showDatePicker(EditText editText) {                       // Method hiển thị DatePickerDialog
        DatePickerDialog datePickerDialog = new DatePickerDialog(          // Tạo DatePickerDialog
                this,                                                      // Context
                (view, year, month, dayOfMonth) -> {                       // Callback khi chọn ngày
                    calendar.set(year, month, dayOfMonth);                 // Set ngày được chọn vào Calendar
                    editText.setText(dateFormat.format(calendar.getTime())); // Format và set ngày vào EditText
                },
                calendar.get(Calendar.YEAR),                               // Năm hiện tại
                calendar.get(Calendar.MONTH),                              // Tháng hiện tại
                calendar.get(Calendar.DAY_OF_MONTH)                        // Ngày hiện tại
        );
        datePickerDialog.show();                                           // Hiển thị DatePickerDialog
    }
```

#### Method setupButtons:
```java
    private void setupButtons() {                                          // Method thiết lập các sự kiện cho button
        btnDangKy.setOnClickListener(v -> registerUser());                // Set listener cho button đăng ký, gọi method registerUser()
        btnHuy.setOnClickListener(v -> finish());                         // Set listener cho button hủy, đóng Activity
    }
```

#### Method registerUser:
```java
    private void registerUser() {                                          // Method xử lý logic đăng ký người dùng
        // Lấy dữ liệu từ form
        String maNhanVien = etMaNhanVien.getText().toString().trim();      // Lấy mã nhân viên từ EditText và trim khoảng trắng
        String hoTen = etHoTen.getText().toString().trim();                // Lấy họ tên từ EditText và trim khoảng trắng
        String ngaySinh = etNgaySinh.getText().toString().trim();          // Lấy ngày sinh từ EditText và trim khoảng trắng
        String soDienThoai = etSoDienThoai.getText().toString().trim();    // Lấy số điện thoại từ EditText và trim khoảng trắng
        String email = etEmail.getText().toString().trim();                // Lấy email từ EditText và trim khoảng trắng
        String ngayVaoLam = etNgayVaoLam.getText().toString().trim();      // Lấy ngày vào làm từ EditText và trim khoảng trắng
        String tenDangNhap = etTenDangNhap.getText().toString().trim();    // Lấy tên đăng nhập từ EditText và trim khoảng trắng
        String matKhau = etMatKhau.getText().toString().trim();            // Lấy mật khẩu từ EditText và trim khoảng trắng
        String xacNhanMatKhau = etXacNhanMatKhau.getText().toString().trim(); // Lấy xác nhận mật khẩu từ EditText và trim khoảng trắng
        
        // Lấy giới tính
        String gioiTinh = "Nam";                                           // Giá trị mặc định là "Nam"
        int selectedGender = rgGioiTinh.getCheckedRadioButtonId();         // Lấy ID của RadioButton được chọn
        if (selectedGender != -1) {                                        // Nếu có RadioButton được chọn
            RadioButton rbSelected = findViewById(selectedGender);         // Tìm RadioButton được chọn
            gioiTinh = rbSelected.getText().toString();                    // Lấy text của RadioButton làm giới tính
        }
        
        // Lấy phòng ban
        List<String> phongBanIds = (List<String>) spPhongBan.getTag();     // Lấy danh sách mã phòng ban từ tag của Spinner
        String maPhongBan = phongBanIds.get(spPhongBan.getSelectedItemPosition()); // Lấy mã phòng ban được chọn
        
        // Validate dữ liệu
        if (!validateInput(maNhanVien, hoTen, ngaySinh, soDienThoai, email, // Gọi method validation với tất cả dữ liệu
                          ngayVaoLam, tenDangNhap, matKhau, xacNhanMatKhau)) {
            return;                                                        // Nếu validation thất bại thì thoát method
        }
        
        // Kiểm tra mã nhân viên đã tồn tại
        if (dbHelper.checkEmployeeExists(maNhanVien)) {                    // Kiểm tra mã nhân viên đã tồn tại trong database
            Toast.makeText(this, "Mã nhân viên đã tồn tại!", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            return;                                                        // Thoát method
        }
        
        // Kiểm tra username đã tồn tại
        if (dbHelper.checkUsernameExists(tenDangNhap)) {                   // Kiểm tra tên đăng nhập đã tồn tại trong database
            Toast.makeText(this, "Tên đăng nhập đã tồn tại!", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            return;                                                        // Thoát method
        }
        
        // Đăng ký user
        boolean success = dbHelper.registerUser(maNhanVien, hoTen, ngaySinh, gioiTinh, // Gọi method database đăng ký user
                soDienThoai, email, ngayVaoLam, maPhongBan, "CV003", tenDangNhap, matKhau); // CV003 = Nhân viên
        
        if (success) {                                                     // Nếu đăng ký thành công
            Toast.makeText(this, "Đăng ký thành công!", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
            finish();                                                      // Đóng Activity và quay về màn hình trước
        } else {                                                           // Nếu đăng ký thất bại
            Toast.makeText(this, "Đăng ký thất bại! Vui lòng thử lại.", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
        }
    }
```
#### Method validateInput:
```java
    private boolean validateInput(String maNhanVien, String hoTen, String ngaySinh, // Method validation tất cả dữ liệu đầu vào
                                 String soDienThoai, String email, String ngayVaoLam,
                                 String tenDangNhap, String matKhau, String xacNhanMatKhau) {
        
        if (maNhanVien.isEmpty()) {                                        // Kiểm tra mã nhân viên không được rỗng
            etMaNhanVien.setError("Vui lòng nhập mã nhân viên");           // Set error message cho EditText mã nhân viên
            etMaNhanVien.requestFocus();                                   // Focus vào EditText mã nhân viên
            return false;                                                  // Trả về false (validation thất bại)
        }
        
        if (hoTen.isEmpty()) {                                             // Kiểm tra họ tên không được rỗng
            etHoTen.setError("Vui lòng nhập họ tên");                      // Set error message cho EditText họ tên
            etHoTen.requestFocus();                                        // Focus vào EditText họ tên
            return false;                                                  // Trả về false (validation thất bại)
        }
        
        if (ngaySinh.isEmpty()) {                                          // Kiểm tra ngày sinh không được rỗng
            etNgaySinh.setError("Vui lòng chọn ngày sinh");                // Set error message cho EditText ngày sinh
            etNgaySinh.requestFocus();                                     // Focus vào EditText ngày sinh
            return false;                                                  // Trả về false (validation thất bại)
        }
        
        if (soDienThoai.isEmpty()) {                                       // Kiểm tra số điện thoại không được rỗng
            etSoDienThoai.setError("Vui lòng nhập số điện thoại");         // Set error message cho EditText số điện thoại
            etSoDienThoai.requestFocus();                                  // Focus vào EditText số điện thoai
            return false;                                                  // Trả về false (validation thất bại)
        }
        
        if (email.isEmpty()) {                                             // Kiểm tra email không được rỗng
            etEmail.setError("Vui lòng nhập email");                       // Set error message cho EditText email
            etEmail.requestFocus();                                        // Focus vào EditText email
            return false;                                                  // Trả về false (validation thất bại)
        }
        
        if (!email.contains("@")) {                                        // Kiểm tra email có chứa ký tự @ (validation cơ bản)
            etEmail.setError("Email không hợp lệ");                        // Set error message cho EditText email
            etEmail.requestFocus();                                        // Focus vào EditText email
            return false;                                                  // Trả về false (validation thất bại)
        }
        
        if (ngayVaoLam.isEmpty()) {                                        // Kiểm tra ngày vào làm không được rỗng
            etNgayVaoLam.setError("Vui lòng chọn ngày vào làm");           // Set error message cho EditText ngày vào làm
            etNgayVaoLam.requestFocus();                                   // Focus vào EditText ngày vào làm
            return false;                                                  // Trả về false (validation thất bại)
        }
        
        if (tenDangNhap.isEmpty()) {                                       // Kiểm tra tên đăng nhập không được rỗng
            etTenDangNhap.setError("Vui lòng nhập tên đăng nhập");         // Set error message cho EditText tên đăng nhập
            etTenDangNhap.requestFocus();                                  // Focus vào EditText tên đăng nhập
            return false;                                                  // Trả về false (validation thất bại)
        }
        
        if (matKhau.isEmpty()) {                                           // Kiểm tra mật khẩu không được rỗng
            etMatKhau.setError("Vui lòng nhập mật khẩu");                  // Set error message cho EditText mật khẩu
            etMatKhau.requestFocus();                                      // Focus vào EditText mật khẩu
            return false;                                                  // Trả về false (validation thất bại)
        }
        
        if (matKhau.length() < 6) {                                        // Kiểm tra mật khẩu phải có ít nhất 6 ký tự
            etMatKhau.setError("Mật khẩu phải có ít nhất 6 ký tự");        // Set error message cho EditText mật khẩu
            etMatKhau.requestFocus();                                      // Focus vào EditText mật khẩu
            return false;                                                  // Trả về false (validation thất bại)
        }
        
        if (!matKhau.equals(xacNhanMatKhau)) {                             // Kiểm tra mật khẩu và xác nhận mật khẩu phải giống nhau
            etXacNhanMatKhau.setError("Mật khẩu xác nhận không khớp");     // Set error message cho EditText xác nhận mật khẩu
            etXacNhanMatKhau.requestFocus();                               // Focus vào EditText xác nhận mật khẩu
            return false;                                                  // Trả về false (validation thất bại)
        }
        
        return true;                                                       // Trả về true nếu tất cả validation đều pass
    }
}
```

---

## 📋 CHI TIẾT CÁC FILE LAYOUT

## 2️⃣ LAYOUT CHÍNH - activity_register.xml

**Đường dẫn**: `app/src/main/res/layout/activity_register.xml`

### Mục đích:
Layout form đăng ký với ScrollView để chứa nhiều trường thông tin, sử dụng Material Design components.

### Chi tiết code:

#### Khai báo XML và ScrollView chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"    <!-- ScrollView root để có thể cuộn khi nội dung dài -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent -->
    android:layout_height="match_parent"                                   <!-- Chiều cao bằng parent -->
    android:padding="16dp"                                                 <!-- Padding 16dp cho tất cả các cạnh -->
    android:background="#f5f5f5">                                          <!-- Background màu xám nhạt -->

    <LinearLayout                                                          <!-- LinearLayout chính chứa tất cả form elements -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="vertical">                                    <!-- Orientation dọc -->
```

#### Header Section:
```xml
        <!-- Header -->
        <TextView                                                          <!-- TextView tiêu đề chính -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="ĐĂNG KÝ TÀI KHOẢN NHÂN VIÊN"                     <!-- Text tiêu đề -->
            android:textSize="24sp"                                        <!-- Kích thước font 24sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:textColor="#2196F3"                                    <!-- Màu chữ xanh dương -->
            android:gravity="center"                                       <!-- Căn giữa text -->
            android:layout_marginBottom="8dp" />                           <!-- Margin bottom 8dp -->

        <TextView                                                          <!-- TextView mô tả phụ -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Chỉ dành cho nhân viên mới"                      <!-- Text mô tả -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textColor="#666"                                       <!-- Màu chữ xám -->
            android:gravity="center"                                       <!-- Căn giữa text -->
            android:layout_marginBottom="24dp" />                          <!-- Margin bottom 24dp -->
```

#### Employee Information Section:
```xml
        <!-- Thông tin nhân viên -->
        <TextView                                                          <!-- TextView tiêu đề section thông tin nhân viên -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="THÔNG TIN NHÂN VIÊN"                             <!-- Text tiêu đề section -->
            android:textSize="16sp"                                        <!-- Kích thước font 16sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:textColor="#333"                                       <!-- Màu chữ xám đậm -->
            android:layout_marginBottom="16dp" />                          <!-- Margin bottom 16dp -->

        <!-- Mã nhân viên -->
        <com.google.android.material.textfield.TextInputLayout             <!-- TextInputLayout Material Design cho mã nhân viên -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:layout_marginBottom="12dp">                            <!-- Margin bottom 12dp -->

            <EditText                                                      <!-- EditText nhập mã nhân viên -->
                android:id="@+id/et_ma_nhan_vien"                          <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:hint="Mã nhân viên (VD: NV001)"                    <!-- Hint text hướng dẫn -->
                android:inputType="text"                                   <!-- Input type text thường -->
                android:maxLength="10" />                                  <!-- Giới hạn tối đa 10 ký tự -->
        </com.google.android.material.textfield.TextInputLayout>

        <!-- Họ tên -->
        <com.google.android.material.textfield.TextInputLayout             <!-- TextInputLayout Material Design cho họ tên -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:layout_marginBottom="12dp">                            <!-- Margin bottom 12dp -->

            <EditText                                                      <!-- EditText nhập họ tên -->
                android:id="@+id/et_ho_ten"                                <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:hint="Họ và tên"                                   <!-- Hint text hướng dẫn -->
                android:inputType="textPersonName" />                      <!-- Input type tên người -->
        </com.google.android.material.textfield.TextInputLayout>

        <!-- Ngày sinh -->
        <com.google.android.material.textfield.TextInputLayout             <!-- TextInputLayout Material Design cho ngày sinh -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:layout_marginBottom="12dp">                            <!-- Margin bottom 12dp -->

            <EditText                                                      <!-- EditText chọn ngày sinh -->
                android:id="@+id/et_ngay_sinh"                             <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:hint="Ngày sinh (yyyy-mm-dd)"                      <!-- Hint text hướng dẫn với format -->
                android:focusable="false"                                  <!-- Không cho phép focus (chỉ click để mở DatePicker) -->
                android:clickable="true" />                                <!-- Cho phép click để mở DatePicker -->
        </com.google.android.material.textfield.TextInputLayout>
```

#### Gender Selection Section:
```xml
        <!-- Giới tính -->
        <TextView                                                          <!-- TextView label cho giới tính -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Giới tính:"                                      <!-- Text label -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:layout_marginBottom="8dp" />                           <!-- Margin bottom 8dp -->

        <RadioGroup                                                        <!-- RadioGroup chứa các RadioButton giới tính -->
            android:id="@+id/rg_gioi_tinh"                                 <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="horizontal"                               <!-- Orientation ngang -->
            android:layout_marginBottom="12dp">                            <!-- Margin bottom 12dp -->

            <RadioButton                                                   <!-- RadioButton cho giới tính Nam -->
                android:id="@+id/rb_nam"                                   <!-- ID để truy cập từ Java code -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ chứa text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:text="Nam"                                         <!-- Text hiển thị -->
                android:checked="true"                                     <!-- Checked mặc định -->
                android:layout_marginEnd="24dp" />                         <!-- Margin end 24dp -->

            <RadioButton                                                   <!-- RadioButton cho giới tính Nữ -->
                android:id="@+id/rb_nu"                                    <!-- ID để truy cập từ Java code -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ chứa text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:text="Nữ" />                                       <!-- Text hiển thị -->
        </RadioGroup>
```

#### Contact Information Section:
```xml
        <!-- Số điện thoại -->
        <com.google.android.material.textfield.TextInputLayout             <!-- TextInputLayout Material Design cho số điện thoại -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:layout_marginBottom="12dp">                            <!-- Margin bottom 12dp -->

            <EditText                                                      <!-- EditText nhập số điện thoại -->
                android:id="@+id/et_so_dien_thoai"                         <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:hint="Số điện thoại"                               <!-- Hint text hướng dẫn -->
                android:inputType="phone" />                               <!-- Input type phone (hiển thị bàn phím số) -->
        </com.google.android.material.textfield.TextInputLayout>

        <!-- Email -->
        <com.google.android.material.textfield.TextInputLayout             <!-- TextInputLayout Material Design cho email -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:layout_marginBottom="12dp">                            <!-- Margin bottom 12dp -->

            <EditText                                                      <!-- EditText nhập email -->
                android:id="@+id/et_email"                                 <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:hint="Email"                                       <!-- Hint text hướng dẫn -->
                android:inputType="textEmailAddress" />                    <!-- Input type email (hiển thị bàn phím email) -->
        </com.google.android.material.textfield.TextInputLayout>
```

#### Work Information Section:
```xml
        <!-- Ngày vào làm -->
        <com.google.android.material.textfield.TextInputLayout             <!-- TextInputLayout Material Design cho ngày vào làm -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:layout_marginBottom="12dp">                            <!-- Margin bottom 12dp -->

            <EditText                                                      <!-- EditText chọn ngày vào làm -->
                android:id="@+id/et_ngay_vao_lam"                          <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:hint="Ngày vào làm (yyyy-mm-dd)"                   <!-- Hint text hướng dẫn với format -->
                android:focusable="false"                                  <!-- Không cho phép focus (chỉ click để mở DatePicker) -->
                android:clickable="true" />                                <!-- Cho phép click để mở DatePicker -->
        </com.google.android.material.textfield.TextInputLayout>

        <!-- Phòng ban -->
        <TextView                                                          <!-- TextView label cho phòng ban -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Phòng ban:"                                      <!-- Text label -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:layout_marginBottom="8dp" />                           <!-- Margin bottom 8dp -->

        <Spinner                                                           <!-- Spinner chọn phòng ban -->
            android:id="@+id/sp_phong_ban"                                 <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="48dp"                                   <!-- Chiều cao cố định 48dp -->
            android:layout_marginBottom="12dp"                             <!-- Margin bottom 12dp -->
            android:background="@android:drawable/btn_dropdown" />         <!-- Background dropdown mặc định -->

        <!-- Chức vụ mặc định -->
        <TextView                                                          <!-- TextView hiển thị chức vụ mặc định -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Chức vụ: Nhân viên (mặc định)"                   <!-- Text hiển thị chức vụ mặc định -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textColor="#666"                                       <!-- Màu chữ xám -->
            android:background="@android:color/white"                      <!-- Background màu trắng -->
            android:padding="12dp"                                         <!-- Padding 12dp cho tất cả các cạnh -->
            android:layout_marginBottom="24dp" />                          <!-- Margin bottom 24dp -->
```

#### Account Information Section:
```xml
        <!-- Thông tin tài khoản -->
        <TextView                                                          <!-- TextView tiêu đề section thông tin tài khoản -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="THÔNG TIN TÀI KHOẢN"                             <!-- Text tiêu đề section -->
            android:textSize="16sp"                                        <!-- Kích thước font 16sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:textColor="#333"                                       <!-- Màu chữ xám đậm -->
            android:layout_marginBottom="16dp" />                          <!-- Margin bottom 16dp -->

        <!-- Tên đăng nhập -->
        <com.google.android.material.textfield.TextInputLayout             <!-- TextInputLayout Material Design cho tên đăng nhập -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:layout_marginBottom="12dp">                            <!-- Margin bottom 12dp -->

            <EditText                                                      <!-- EditText nhập tên đăng nhập -->
                android:id="@+id/et_ten_dang_nhap"                         <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:hint="Tên đăng nhập"                               <!-- Hint text hướng dẫn -->
                android:inputType="text" />                                <!-- Input type text thường -->
        </com.google.android.material.textfield.TextInputLayout>

        <!-- Mật khẩu -->
        <com.google.android.material.textfield.TextInputLayout             <!-- TextInputLayout Material Design cho mật khẩu -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:layout_marginBottom="12dp">                            <!-- Margin bottom 12dp -->

            <EditText                                                      <!-- EditText nhập mật khẩu -->
                android:id="@+id/et_mat_khau"                              <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:hint="Mật khẩu (tối thiểu 6 ký tự)"                <!-- Hint text hướng dẫn với yêu cầu -->
                android:inputType="textPassword" />                        <!-- Input type password (ẩn ký tự) -->
        </com.google.android.material.textfield.TextInputLayout>

        <!-- Xác nhận mật khẩu -->
        <com.google.android.material.textfield.TextInputLayout             <!-- TextInputLayout Material Design cho xác nhận mật khẩu -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:layout_marginBottom="24dp">                            <!-- Margin bottom 24dp -->

            <EditText                                                      <!-- EditText nhập xác nhận mật khẩu -->
                android:id="@+id/et_xac_nhan_mat_khau"                     <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:hint="Xác nhận mật khẩu"                           <!-- Hint text hướng dẫn -->
                android:inputType="textPassword" />                        <!-- Input type password (ẩn ký tự) -->
        </com.google.android.material.textfield.TextInputLayout>
```

#### Action Buttons Section:
```xml
        <!-- Buttons -->
        <LinearLayout                                                      <!-- LinearLayout chứa các button action -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="horizontal"                               <!-- Orientation ngang -->
            android:gravity="center">                                      <!-- Căn giữa -->

            <Button                                                        <!-- Button hủy -->
                android:id="@+id/btn_huy"                                  <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:layout_weight="1"                                  <!-- Weight 1 để chia đều không gian -->
                android:text="HỦY"                                         <!-- Text button -->
                android:textColor="#666"                                   <!-- Màu chữ xám -->
                android:background="@android:color/transparent"            <!-- Background trong suốt -->
                android:layout_marginEnd="8dp" />                          <!-- Margin end 8dp -->

            <Button                                                        <!-- Button đăng ký -->
                android:id="@+id/btn_dang_ky"                              <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:layout_weight="1"                                  <!-- Weight 1 để chia đều không gian -->
                android:text="ĐĂNG KÝ"                                     <!-- Text button -->
                android:textColor="@android:color/white"                   <!-- Màu chữ trắng -->
                android:background="#2196F3"                               <!-- Background màu xanh dương -->
                android:layout_marginStart="8dp" />                        <!-- Margin start 8dp -->
        </LinearLayout>

    </LinearLayout>
</ScrollView>
```

---

## 🗄️ CHI TIẾT CÁC PHƯƠNG THỨC DATABASE

## 3️⃣ DATABASE METHODS - DatabaseHelper.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/database/DatabaseHelper.java`

### Mục đích:
Các phương thức database hỗ trợ chức năng đăng ký, bao gồm validation, auto-generation và transaction processing.

### Chi tiết các methods:

#### Method checkEmployeeExists:
```java
    public boolean checkEmployeeExists(String maNhanVien) {               // Method kiểm tra mã nhân viên đã tồn tại
        SQLiteDatabase db = this.getReadableDatabase();                   // Lấy database instance ở chế độ đọc
        Cursor cursor = db.rawQuery("SELECT * FROM " + TABLE_NHAN_VIEN + // Thực hiện query SELECT với WHERE condition
                " WHERE MaNhanVien = ?", new String[]{maNhanVien});       // Sử dụng parameter để tránh SQL injection
        boolean exists = (cursor.getCount() > 0);                        // Kiểm tra có record nào trả về không
        cursor.close();                                                   // Đóng cursor để giải phóng memory
        return exists;                                                    // Trả về true nếu mã nhân viên đã tồn tại
    }
```

#### Method checkUsernameExists:
```java
    public boolean checkUsernameExists(String username) {                 // Method kiểm tra tên đăng nhập đã tồn tại
        SQLiteDatabase db = this.getReadableDatabase();                   // Lấy database instance ở chế độ đọc
        Cursor cursor = db.rawQuery("SELECT * FROM " + TABLE_TAI_KHOAN +  // Thực hiện query SELECT với WHERE condition
                " WHERE TenDangNhap = ?", new String[]{username});        // Sử dụng parameter để tránh SQL injection
        boolean exists = (cursor.getCount() > 0);                        // Kiểm tra có record nào trả về không
        cursor.close();                                                   // Đóng cursor để giải phóng memory
        return exists;                                                    // Trả về true nếu tên đăng nhập đã tồn tại
    }
```

#### Method registerUser:
```java
    public boolean registerUser(String maNV, String hoTen, String ngaySinh, String gioiTinh, // Method đăng ký user với transaction
                               String sdt, String email, String ngayVaoLam, String maPB,     // Nhận tất cả thông tin cần thiết
                               String maCV, String username, String password) {              // Bao gồm cả thông tin nhân viên và tài khoản
        SQLiteDatabase db = this.getWritableDatabase();                   // Lấy database instance ở chế độ ghi
        db.beginTransaction();                                            // Bắt đầu transaction để đảm bảo tính toàn vẹn dữ liệu
        try {
            // Tạo record nhân viên
            ContentValues nvValues = new ContentValues();                 // Tạo ContentValues cho bảng NhanVien
            nvValues.put("MaNhanVien", maNV);                            // Thêm mã nhân viên
            nvValues.put("HoTen", hoTen);                                // Thêm họ tên
            nvValues.put("NgaySinh", ngaySinh);                          // Thêm ngày sinh
            nvValues.put("GioiTinh", gioiTinh);                          // Thêm giới tính
            nvValues.put("SoDienThoai", sdt);                            // Thêm số điện thoại
            nvValues.put("Email", email);                                // Thêm email
            nvValues.put("NgayVaoLam", ngayVaoLam);                      // Thêm ngày vào làm
            nvValues.put("MaPhongBan", maPB);                            // Thêm mã phòng ban
            nvValues.put("MaChucVu", maCV);                              // Thêm mã chức vụ
            nvValues.put("TrangThaiLamViec", "Đang làm việc");           // Set trạng thái mặc định
            
            long nvResult = db.insert(TABLE_NHAN_VIEN, null, nvValues);  // Insert record vào bảng NhanVien
            if (nvResult == -1) return false;                           // Nếu insert thất bại thì return false

            // Tạo record tài khoản
            ContentValues tkValues = new ContentValues();                // Tạo ContentValues cho bảng TaiKhoan
            tkValues.put("MaNhanVien", maNV);                            // Liên kết với mã nhân viên
            tkValues.put("TenDangNhap", username);                       // Thêm tên đăng nhập
            tkValues.put("MatKhau", password);                           // Thêm mật khẩu (plain text)
            tkValues.put("VaiTro", "Employee");                          // Set vai trò mặc định là Employee
            tkValues.put("TrangThai", 1);                                // Set trạng thái active (1)
            
            long tkResult = db.insert(TABLE_TAI_KHOAN, null, tkValues);  // Insert record vào bảng TaiKhoan
            if (tkResult == -1) return false;                           // Nếu insert thất bại thì return false

            db.setTransactionSuccessful();                               // Đánh dấu transaction thành công
            return true;                                                 // Trả về true nếu tất cả thành công
        } catch (Exception e) {                                          // Catch mọi exception
            return false;                                                // Trả về false nếu có lỗi
        } finally {
            db.endTransaction();                                         // Kết thúc transaction (commit hoặc rollback)
        }
    }
```

#### Method getNextEmployeeCode:
```java
    public String getNextEmployeeCode() {                                // Method tự động tạo mã nhân viên tiếp theo
        SQLiteDatabase db = this.getReadableDatabase();                  // Lấy database instance ở chế độ đọc
        String query = "SELECT MaNhanVien FROM " + TABLE_NHAN_VIEN +     // Query lấy mã nhân viên lớn nhất
                      " WHERE MaNhanVien LIKE 'NV%' " +                  // Chỉ lấy mã có format NVxxx
                      " ORDER BY length(MaNhanVien) DESC, MaNhanVien DESC LIMIT 1"; // Sắp xếp theo độ dài và giá trị giảm dần
        Cursor cursor = db.rawQuery(query, null);                       // Thực hiện query
        String lastCode = null;                                          // Biến lưu mã cuối cùng
        if (cursor != null) {                                            // Nếu cursor không null
            if (cursor.moveToFirst()) lastCode = cursor.getString(0).trim(); // Lấy mã đầu tiên và trim khoảng trắng
            cursor.close();                                              // Đóng cursor
        }
        if (lastCode == null) return "NV002";                           // Nếu không có mã nào thì trả về NV002 (bỏ qua NV001 cho admin)
        
        // Parse mã để tăng số thứ tự
        java.util.regex.Matcher matcher = java.util.regex.Pattern.compile("^([a-zA-Z]+)(\\d+)$").matcher(lastCode); // Regex pattern để tách prefix và số
        if (matcher.find()) {                                            // Nếu match được pattern
            String prefix = matcher.group(1);                           // Lấy prefix (NV)
            String numberStr = matcher.group(2);                        // Lấy phần số (001, 002, ...)
            try {
                int nextNumber = Integer.parseInt(numberStr) + 1;       // Tăng số lên 1
                return String.format("%s%0" + numberStr.length() + "d", prefix, nextNumber); // Format với leading zeros
            } catch (Exception e) { return "NV002"; }                   // Nếu có lỗi parse thì trả về NV002
        }
        return "NV002";                                                  // Default return NV002
    }
```

---

## 🔗 TÍCH HỢP VÀ NAVIGATION

### 1. Tích hợp với MainActivity:
- **Navigation flow**: MainActivity (public) → LoginActivity → RegisterActivity
- **Button integration**: Button "ĐĂNG KÝ" trong MainActivity navigate đến RegisterActivity
- **Intent handling**: Sử dụng Intent để chuyển đổi giữa các Activity

### 2. Tích hợp với LoginActivity:
- **Return flow**: Sau khi đăng ký thành công, user quay về LoginActivity
- **Data consistency**: Tài khoản mới tạo có thể đăng nhập ngay lập tức
- **Error handling**: Thông báo lỗi nếu đăng ký thất bại

### 3. Tích hợp với Database:
- **Transaction safety**: Sử dụng database transaction để đảm bảo tính toàn vẹn
- **Foreign key constraints**: Liên kết chính xác giữa NhanVien và TaiKhoan
- **Data validation**: Kiểm tra unique constraints trước khi insert

---

## 📋 QUY TẮC NGHIỆP VỤ

### 1. Quy tắc đăng ký:
- **Role restriction**: Chỉ cho phép đăng ký với vai trò "Employee"
- **Auto-generation**: Mã nhân viên được tự động tạo, không cho phép nhập thủ công
- **Default values**: Chức vụ mặc định là "Nhân viên" (CV003)
- **Status default**: Trạng thái làm việc mặc định là "Đang làm việc"

### 2. Quy tắc validation:
- **Required fields**: Tất cả trường đều bắt buộc nhập
- **Password policy**: Mật khẩu tối thiểu 6 ký tự
- **Email format**: Kiểm tra email có chứa ký tự "@"
- **Password confirmation**: Mật khẩu và xác nhận mật khẩu phải giống nhau

### 3. Quy tắc bảo mật:
- **Unique constraints**: Mã nhân viên và tên đăng nhập phải duy nhất
- **Plain text password**: Mật khẩu lưu dạng plain text (theo yêu cầu đơn giản)
- **Input sanitization**: Trim khoảng trắng và validate dữ liệu đầu vào
- **SQL injection prevention**: Sử dụng parameterized queries

---

## 🔒 BẢO MẬT VÀ VALIDATION

### 1. Client-side validation:
- **Real-time feedback**: Error messages hiển thị ngay khi có lỗi
- **Focus management**: Tự động focus vào field có lỗi
- **Input constraints**: Giới hạn độ dài và format của các trường

### 2. Server-side validation:
- **Database constraints**: Kiểm tra unique constraints trước khi insert
- **Data integrity**: Sử dụng transaction để đảm bảo tính toàn vẹn
- **Error handling**: Xử lý exception và rollback khi có lỗi

### 3. Security considerations:
- **Parameter binding**: Sử dụng parameterized queries để tránh SQL injection
- **Input validation**: Validate tất cả input trước khi xử lý
- **Error messages**: Thông báo lỗi rõ ràng nhưng không tiết lộ thông tin nhạy cảm

---

## 📱 TRẢI NGHIỆM NGƯỜI DÙNG

### 1. UI/UX Design:
- **Material Design**: Sử dụng Material Design components cho giao diện hiện đại
- **Responsive layout**: ScrollView đảm bảo form hiển thị tốt trên mọi kích thước màn hình
- **Visual hierarchy**: Phân chia rõ ràng giữa thông tin nhân viên và tài khoản
- **Color scheme**: Sử dụng màu sắc nhất quán với theme của ứng dụng

### 2. Interaction Design:
- **Date pickers**: DatePickerDialog cho việc chọn ngày sinh và ngày vào làm
- **Dropdown selection**: Spinner cho việc chọn phòng ban
- **Radio buttons**: RadioGroup cho việc chọn giới tính
- **Button states**: Visual feedback khi nhấn button

### 3. Error Handling:
- **Inline validation**: Error messages hiển thị trực tiếp trên các field
- **Toast notifications**: Thông báo kết quả đăng ký
- **Focus management**: Tự động focus vào field có lỗi đầu tiên

---

## 🎯 KẾT LUẬN

Module Đăng ký là một phần quan trọng của hệ thống QLNS, cung cấp:

### Tính năng chính:
- **Self-registration**: Cho phép nhân viên mới tự đăng ký tài khoản
- **Auto-generation**: Tự động tạo mã nhân viên để tránh trùng lặp
- **Comprehensive validation**: Kiểm tra đầy đủ dữ liệu đầu vào
- **Transaction safety**: Đảm bảo tính toàn vẹn dữ liệu với database transaction

### Ưu điểm:
- **User-friendly**: Giao diện đơn giản, dễ sử dụng
- **Secure**: Validation đầy đủ và sử dụng parameterized queries
- **Maintainable**: Code được tổ chức rõ ràng, dễ bảo trì
- **Scalable**: Có thể mở rộng thêm tính năng validation và business rules

### Tích hợp hệ thống:
- **Seamless flow**: Tích hợp mượt mà với flow đăng nhập
- **Database consistency**: Đảm bảo tính nhất quán dữ liệu
- **Role-based access**: Hỗ trợ hệ thống phân quyền của ứng dụng

Module này đóng vai trò là cửa ngõ cho nhân viên mới gia nhập hệ thống, đảm bảo quy trình onboarding hiệu quả và an toàn.