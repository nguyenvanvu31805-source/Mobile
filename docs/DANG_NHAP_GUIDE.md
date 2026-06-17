# HƯỚNG DẪN CHI TIẾT: CHỨC NĂNG ĐĂNG NHẬP

## 📋 TỔNG QUAN

Chức năng Đăng nhập là cổng vào chính của hệ thống QLNS, cho phép người dùng xác thực danh tính và truy cập vào các chức năng tương ứng với vai trò của họ. Module này đảm bảo bảo mật và phân quyền truy cập cho toàn bộ hệ thống.

## 🏗️ KIẾN TRÚC HỆ THỐNG

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/
├── LoginActivity.java                       # Activity chính - Xử lý đăng nhập
└── database/DatabaseHelper.java             # Xử lý xác thực database

app/src/main/res/layout/
├── activity_login.xml                       # Layout form đăng nhập
└── drawable/
    └── button_login_bg.xml                  # Background button đăng nhập

Lưu ý: Module này không có model class riêng, sử dụng trực tiếp database
```

## 📊 NGHIỆP VỤ ĐĂNG NHẬP

### 1. Quy trình nghiệp vụ:
- **Nhập thông tin**: Tên đăng nhập và mật khẩu
- **Validation**: Kiểm tra dữ liệu đầu vào không được rỗng
- **Xác thực**: Kiểm tra thông tin đăng nhập với database
- **Phân quyền**: Xác định vai trò và chuyển hướng phù hợp
- **Session management**: Truyền thông tin người dùng qua Intent
- **Chuyển hướng**: Đến DashboardActivity sau khi đăng nhập thành công

### 2. Bảo mật:
- **Account status check**: Chỉ cho phép đăng nhập tài khoản đang hoạt động (TrangThai = 1)
- **Plain text password**: Sử dụng mật khẩu plain text (theo yêu cầu đơn giản hóa)
- **Input validation**: Kiểm tra và hiển thị lỗi cho các trường bắt buộc
- **Error handling**: Thông báo lỗi rõ ràng khi đăng nhập thất bại

### 3. Vai trò hệ thống:
- **Admin**: Quản trị viên - full quyền tất cả chức năng
- **HR**: Nhân sự - quản lý nhân viên, lương, hợp đồng
- **Manager**: Quản lý - quản lý nhân viên, xem báo cáo
- **Employee**: Nhân viên - chỉ truy cập chức năng cá nhân

---

## 📱 CHI TIẾT CÁC FILE

## 1️⃣ ACTIVITY CHÍNH - LoginActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/LoginActivity.java`

### Mục đích:
Activity chính xử lý đăng nhập, validation và chuyển hướng đến Dashboard sau khi xác thực thành công.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.content.Intent;                                             // Import Intent để chuyển Activity
import android.os.Bundle;                                                  // Import Bundle để truyền dữ liệu giữa Activity
import android.widget.Button;                                              // Import Button widget
import android.widget.EditText;                                            // Import EditText widget
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo

import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity làm base class

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database
```

#### Khai báo thuộc tính class:
```java
public class LoginActivity extends AppCompatActivity {                     // Khai báo class kế thừa AppCompatActivity
    
    private EditText etUsername, etPassword;                               // EditText cho tên đăng nhập và mật khẩu
    private Button btnLogin;                                               // Button đăng nhập
    private TextView tvRegister;                                           // TextView link đăng ký
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
```

#### Method onCreate:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi method onCreate của class cha
        setContentView(R.layout.activity_login);                           // Set layout cho Activity
        
        initViews();                                                       // Khởi tạo các view components
        setupDatabase();                                                   // Thiết lập database
        setupButtons();                                                    // Thiết lập các button events
    }
```

#### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo và ánh xạ các view từ layout
        etUsername = findViewById(R.id.et_username);                       // Ánh xạ EditText tên đăng nhập
        etPassword = findViewById(R.id.et_password);                       // Ánh xạ EditText mật khẩu
        btnLogin = findViewById(R.id.btn_login);                           // Ánh xạ Button đăng nhập
        tvRegister = findViewById(R.id.tv_register);                       // Ánh xạ TextView link đăng ký
    }
```

#### Method setupDatabase:
```java
    private void setupDatabase() {                                         // Method thiết lập database helper
        dbHelper = new DatabaseHelper(this);                              // Khởi tạo DatabaseHelper với context hiện tại
    }
```

#### Method setupButtons:
```java
    private void setupButtons() {                                          // Method thiết lập các sự kiện cho button
        btnLogin.setOnClickListener(v -> login());                        // Set listener cho button đăng nhập, gọi method login()
        tvRegister.setOnClickListener(v -> {                              // Set listener cho TextView đăng ký
            Intent intent = new Intent(LoginActivity.this, RegisterActivity.class); // Tạo Intent chuyển đến RegisterActivity
            startActivity(intent);                                         // Start RegisterActivity
        });
    }
```

#### Method login:
```java
    private void login() {                                                 // Method xử lý logic đăng nhập
        String username = etUsername.getText().toString().trim();         // Lấy tên đăng nhập từ EditText và trim khoảng trắng
        String password = etPassword.getText().toString().trim();         // Lấy mật khẩu từ EditText và trim khoảng trắng
        
        if (username.isEmpty()) {                                          // Kiểm tra tên đăng nhập không được rỗng
            etUsername.setError("Vui lòng nhập tên đăng nhập");            // Set error message cho EditText tên đăng nhập
            etUsername.requestFocus();                                     // Focus vào EditText tên đăng nhập
            return;                                                        // Thoát method
        }
        
        if (password.isEmpty()) {                                          // Kiểm tra mật khẩu không được rỗng
            etPassword.setError("Vui lòng nhập mật khẩu");                 // Set error message cho EditText mật khẩu
            etPassword.requestFocus();                                     // Focus vào EditText mật khẩu
            return;                                                        // Thoát method
        }
        
        boolean loginSuccess = dbHelper.checkLogin(username, password);    // Gọi method database kiểm tra đăng nhập
        
        if (loginSuccess) {                                                // Nếu đăng nhập thành công
            Toast.makeText(this, "Đăng nhập thành công!", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
            // Chuyển đến DashboardActivity (trang chính sau khi đăng nhập)
            Intent intent = new Intent(LoginActivity.this, DashboardActivity.class); // Tạo Intent chuyển đến DashboardActivity
            intent.putExtra("username", username);                         // Truyền username qua Intent
            startActivity(intent);                                         // Start DashboardActivity
            finish();                                                      // Đóng LoginActivity để không thể quay lại bằng back button
        } else {                                                           // Nếu đăng nhập thất bại
            Toast.makeText(this, "Tên đăng nhập hoặc mật khẩu không đúng!", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
        }
    }
}
```

---

## 📋 CHI TIẾT CÁC FILE LAYOUT

## 2️⃣ LAYOUT CHÍNH - activity_login.xml

**Đường dẫn**: `app/src/main/res/layout/activity_login.xml`

### Mục đích:
Layout form đăng nhập với thiết kế Material Design, bao gồm logo, form input và navigation.

### Chi tiết code:

#### Khai báo XML và LinearLayout chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- LinearLayout root với namespace Android -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent (full width) -->
    android:layout_height="match_parent"                                   <!-- Chiều cao bằng parent (full height) -->
    android:orientation="vertical"                                         <!-- Orientation dọc (các child xếp từ trên xuống) -->
    android:padding="24dp"                                                 <!-- Padding 24dp cho tất cả các cạnh -->
    android:gravity="center"                                               <!-- Căn giữa tất cả content -->
    android:background="#f5f5f5">                                          <!-- Background màu xám nhạt (#f5f5f5) -->
```

#### Logo Section:
```xml
    <!-- Logo/Icon -->
    <ImageView                                                             <!-- ImageView hiển thị logo ứng dụng -->
        android:layout_width="300dp"                                       <!-- Chiều rộng cố định 300dp -->
        android:layout_height="200dp"                                      <!-- Chiều cao cố định 200dp -->
        android:src="@drawable/quan_tri_nhan_su"                           <!-- Source image từ drawable -->
        android:scaleType="centerInside"                                   <!-- Scale type để giữ tỷ lệ và căn giữa -->
        android:layout_marginBottom="16dp" />                              <!-- Margin bottom 16dp -->
```

#### Title Section:
```xml
    <!-- Logo/Title -->
    <TextView                                                              <!-- TextView tiêu đề chính -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="QUẢN LÝ NHÂN SỰ"                                     <!-- Text tiêu đề ứng dụng -->
        android:textSize="28sp"                                            <!-- Kích thước font 28sp -->
        android:textStyle="bold"                                           <!-- Style chữ đậm -->
        android:textColor="#2196F3"                                        <!-- Màu chữ xanh dương -->
        android:gravity="center"                                           <!-- Căn giữa text -->
        android:layout_marginBottom="8dp" />                               <!-- Margin bottom 8dp -->

    <TextView                                                              <!-- TextView mô tả phụ -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Đăng nhập để tiếp tục"                               <!-- Text mô tả -->
        android:textSize="16sp"                                            <!-- Kích thước font 16sp -->
        android:textColor="#666"                                           <!-- Màu chữ xám -->
        android:gravity="center"                                           <!-- Căn giữa text -->
        android:layout_marginBottom="32dp" />                              <!-- Margin bottom 32dp -->
```

#### Login Form Section:
```xml
    <!-- Login Form -->
    <LinearLayout                                                          <!-- LinearLayout chứa form đăng nhập -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="vertical"                                     <!-- Orientation dọc -->
        android:background="@android:color/white"                          <!-- Background màu trắng -->
        android:padding="24dp"                                             <!-- Padding 24dp cho tất cả các cạnh -->
        android:elevation="4dp">                                           <!-- Elevation 4dp tạo shadow -->
```

#### Username Input Section:
```xml
        <!-- Username -->
        <com.google.android.material.textfield.TextInputLayout             <!-- TextInputLayout Material Design cho username -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:layout_marginBottom="16dp">                            <!-- Margin bottom 16dp -->

            <EditText                                                      <!-- EditText nhập tên đăng nhập -->
                android:id="@+id/et_username"                              <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:hint="Tên đăng nhập"                               <!-- Hint text hướng dẫn -->
                android:inputType="text"                                   <!-- Input type text thường -->
                android:drawableStart="@drawable/ic_person"                <!-- Icon người dùng ở đầu (bên trái) -->
                android:drawablePadding="12dp"                             <!-- Padding giữa icon và text 12dp -->
                android:background="@android:color/transparent"            <!-- Background trong suốt -->
                android:padding="12dp" />                                  <!-- Padding 12dp cho tất cả các cạnh -->
        </com.google.android.material.textfield.TextInputLayout>
```

#### Password Input Section:
```xml
        <!-- Password -->
        <com.google.android.material.textfield.TextInputLayout             <!-- TextInputLayout Material Design cho password -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:layout_marginBottom="24dp">                            <!-- Margin bottom 24dp -->

            <EditText                                                      <!-- EditText nhập mật khẩu -->
                android:id="@+id/et_password"                              <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:hint="Mật khẩu"                                    <!-- Hint text hướng dẫn -->
                android:inputType="textPassword"                           <!-- Input type password (ẩn ký tự) -->
                android:drawableStart="@drawable/ic_lock"                  <!-- Icon khóa ở đầu (bên trái) -->
                android:drawablePadding="12dp"                             <!-- Padding giữa icon và text 12dp -->
                android:background="@android:color/transparent"            <!-- Background trong suốt -->
                android:padding="12dp" />                                  <!-- Padding 12dp cho tất cả các cạnh -->
        </com.google.android.material.textfield.TextInputLayout>
```

#### Login Button Section:
```xml
        <!-- Login Button -->
        <Button                                                            <!-- Button đăng nhập -->
            android:id="@+id/btn_login"                                    <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:text="ĐĂNG NHẬP"                                       <!-- Text button -->
            android:textColor="@android:color/white"                       <!-- Màu chữ trắng -->
            android:backgroundTint="#2196F3"                               <!-- Màu background xanh dương -->
            android:textSize="16sp"                                        <!-- Kích thước font 16sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:layout_marginBottom="16dp" />                          <!-- Margin bottom 16dp -->
```

#### Register Link Section:
```xml
        <!-- Register Link -->
        <TextView                                                          <!-- TextView link đăng ký -->
            android:id="@+id/tv_register"                                  <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:text="Chưa có tài khoản? Đăng ký ngay"                 <!-- Text link đăng ký -->
            android:textColor="#2196F3"                                    <!-- Màu chữ xanh dương -->
            android:gravity="center"                                       <!-- Căn giữa text -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:clickable="true"                                       <!-- Cho phép click -->
            android:focusable="true"                                       <!-- Cho phép focus -->
            android:background="?android:attr/selectableItemBackground"    <!-- Background với ripple effect -->
            android:padding="8dp" />                                       <!-- Padding 8dp cho tất cả các cạnh -->

    </LinearLayout>

</LinearLayout>
```

---

## 3️⃣ DRAWABLE RESOURCE - button_login_bg.xml

**Đường dẫn**: `app/src/main/res/drawable/button_login_bg.xml`

### Mục đích:
Background drawable cho button đăng nhập với ripple effect và bo góc.

### Chi tiết code:

#### Khai báo XML và Ripple Effect:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<ripple xmlns:android="http://schemas.android.com/apk/res/android"        <!-- Ripple drawable với namespace Android -->
    android:color="#40FFFFFF">                                             <!-- Màu ripple effect (trắng với alpha 40) -->
    <item>                                                                 <!-- Item chứa shape background -->
        <shape android:shape="rectangle">                                  <!-- Shape hình chữ nhật -->
            <solid android:color="#1A237E" />                              <!-- Màu nền xanh đậm -->
            <corners android:radius="25dp" />                              <!-- Bo góc 25dp -->
        </shape>
    </item>
</ripple>
```
---

## 📋 CÁC PHƯƠNG THỨC DATABASE LIÊN QUAN

## 4️⃣ DATABASE METHODS - DatabaseHelper.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/database/DatabaseHelper.java`

### Mục đích:
Các phương thức xử lý database cho chức năng đăng nhập và xác thực người dùng.

### Chi tiết code:

#### Method checkLogin:
```java
    public boolean checkLogin(String username, String password) {          // Method kiểm tra thông tin đăng nhập
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database ở chế độ đọc
        Cursor cursor = db.rawQuery("SELECT * FROM " + TABLE_TAI_KHOAN +   // Query SELECT kiểm tra tài khoản
                " WHERE TenDangNhap = ? AND MatKhau = ? AND TrangThai = 1", // WHERE với điều kiện tên đăng nhập, mật khẩu và trạng thái hoạt động
                new String[]{username, password});                         // Tham số truyền vào query (username, password)
        boolean success = (cursor.getCount() > 0);                        // Kiểm tra có kết quả trả về không (count > 0 = đăng nhập thành công)
        cursor.close();                                                    // Đóng cursor để giải phóng bộ nhớ
        return success;                                                    // Trả về kết quả đăng nhập (true/false)
    }
```

#### Method checkCurrentPassword:
```java
    // Kiểm tra mật khẩu hiện tại có đúng không
    public boolean checkCurrentPassword(String username, String currentPassword) { // Method kiểm tra mật khẩu hiện tại (dùng cho đổi mật khẩu)
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database ở chế độ đọc
        Cursor cursor = db.rawQuery("SELECT * FROM " + TABLE_TAI_KHOAN +   // Query SELECT kiểm tra mật khẩu hiện tại
                " WHERE TenDangNhap = ? AND MatKhau = ?",                  // WHERE với điều kiện tên đăng nhập và mật khẩu
                new String[]{username, currentPassword});                  // Tham số truyền vào query (username, currentPassword)
        boolean isCorrect = (cursor.getCount() > 0);                      // Kiểm tra có kết quả trả về không
        cursor.close();                                                    // Đóng cursor để giải phóng bộ nhớ
        return isCorrect;                                                  // Trả về kết quả kiểm tra (true/false)
    }
```

#### Method getUserInfo:
```java
    public Cursor getUserInfo(String username) {                           // Method lấy thông tin người dùng sau khi đăng nhập
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database ở chế độ đọc
        String query = "SELECT " +                                         // Query SELECT với các trường cần thiết
                "CASE WHEN tk.MaNhanVien = 'ADMIN' THEN 'Administrator' ELSE nv.HoTen END as HoTen, " + // CASE WHEN để xử lý tên hiển thị (Admin hoặc họ tên nhân viên)
                "tk.VaiTro " +                                             // Lấy vai trò từ bảng TaiKhoan
                "FROM " + TABLE_TAI_KHOAN + " tk " +                       // FROM bảng TaiKhoan với alias tk
                "LEFT JOIN " + TABLE_NHAN_VIEN + " nv ON tk.MaNhanVien = nv.MaNhanVien " + // LEFT JOIN với bảng NhanVien để lấy họ tên
                "WHERE tk.TenDangNhap = ?";                                // WHERE theo tên đăng nhập
        return db.rawQuery(query, new String[]{username});                // Thực hiện query và trả về Cursor
    }
```

---

## 🔗 TÍCH HỢP VÀ ĐIỀU HƯỚNG

### 1. Navigation Flow:
```
MainActivity (Public) → LoginActivity → DashboardActivity
                            ↓
                      RegisterActivity (optional)
```

### 2. Data Flow:
- **Input**: Username và Password từ form
- **Validation**: Client-side validation (empty check)
- **Authentication**: Database query với điều kiện TrangThai = 1
- **Session**: Truyền username qua Intent đến DashboardActivity
- **Role Resolution**: DashboardActivity lấy role từ database và áp dụng permissions

### 3. Integration Points:
- **MainActivity**: Chuyển hướng đến LoginActivity khi user click "Đăng nhập"
- **RegisterActivity**: Chuyển hướng từ LoginActivity khi user click "Đăng ký"
- **DashboardActivity**: Nhận username từ Intent và hiển thị giao diện theo role
- **Database**: Tích hợp với bảng TaiKhoan và NhanVien để xác thực và lấy thông tin

---

## 🎯 QUY TẮC NGHIỆP VỤ

### 1. Authentication Rules:
- **Required fields**: Cả username và password đều bắt buộc
- **Account status**: Chỉ cho phép đăng nhập tài khoản có TrangThai = 1 (hoạt động)
- **Case sensitive**: Tên đăng nhập và mật khẩu phân biệt hoa thường
- **Plain text password**: Sử dụng mật khẩu plain text (theo yêu cầu đơn giản hóa)

### 2. Security Considerations:
- **Input validation**: Kiểm tra input rỗng và hiển thị error message
- **SQL injection protection**: Sử dụng parameterized query với placeholder (?)
- **Session management**: Truyền username qua Intent, không lưu password
- **Activity lifecycle**: Gọi finish() sau khi đăng nhập thành công để không thể quay lại

### 3. User Experience:
- **Error feedback**: Hiển thị error message rõ ràng cho từng trường
- **Focus management**: Tự động focus vào trường có lỗi
- **Toast notifications**: Thông báo kết quả đăng nhập
- **Navigation**: Chuyển hướng mượt mà giữa các Activity

---

## 🎨 THIẾT KẾ UI/UX

### 1. Design Principles:
- **Material Design**: Sử dụng TextInputLayout, elevation, ripple effects
- **Brand consistency**: Logo và màu sắc thống nhất với hệ thống (#2196F3)
- **Accessibility**: Icons và labels rõ ràng, contrast tốt
- **Responsive**: Layout căn giữa và responsive trên các kích thước màn hình

### 2. Visual Hierarchy:
- **Logo prominence**: Logo lớn ở đầu trang để nhận diện thương hiệu
- **Form focus**: Form đăng nhập nổi bật với background trắng và elevation
- **Action clarity**: Button đăng nhập có màu sắc nổi bật và text rõ ràng
- **Secondary actions**: Link đăng ký có màu nhạt hơn và kích thước nhỏ hơn

### 3. Interactive Elements:
- **Input feedback**: Error states với màu đỏ và focus states
- **Button states**: Ripple effect và color states cho button
- **Touch targets**: Kích thước touch target đủ lớn (minimum 48dp)
- **Loading states**: Có thể mở rộng thêm loading indicator

---

## 🔒 BẢO MẬT VÀ PHIÊN LÀM VIỆC

### 1. Authentication Security:
- **Database validation**: Kiểm tra credentials với database thay vì hardcode
- **Account status check**: Chỉ cho phép tài khoản đang hoạt động đăng nhập
- **Input sanitization**: Sử dụng parameterized query để tránh SQL injection
- **Error handling**: Không tiết lộ thông tin nhạy cảm trong error messages

### 2. Session Management:
- **Stateless approach**: Không lưu trữ session phía client
- **Intent-based**: Truyền thông tin cần thiết qua Intent
- **Activity lifecycle**: Quản lý lifecycle để tránh memory leaks
- **Logout handling**: Clear activity stack khi logout

### 3. Data Protection:
- **Minimal data exposure**: Chỉ truyền username, không truyền password
- **Role-based access**: Phân quyền dựa trên role từ database
- **Secure navigation**: Prevent back navigation sau khi đăng nhập
- **Input validation**: Client-side validation để improve UX

---

## 📊 LUỒNG DỮ LIỆU CHI TIẾT

### 1. Login Process Flow:
```
1. User nhập username/password
2. Client validation (empty check)
3. DatabaseHelper.checkLogin(username, password)
4. Query: SELECT * FROM TaiKhoan WHERE TenDangNhap=? AND MatKhau=? AND TrangThai=1
5. Nếu có kết quả → Login success
6. Intent to DashboardActivity với username
7. DashboardActivity.getUserInfo(username) để lấy role
8. Apply permissions based on role
```

### 2. Error Handling Flow:
```
Empty username → setError() + requestFocus()
Empty password → setError() + requestFocus()
Invalid credentials → Toast message
Database error → Graceful fallback
```

### 3. Navigation Flow:
```
MainActivity → LoginActivity → DashboardActivity
     ↓              ↓
RegisterActivity ← ←
```

---

## 🔧 KẾT LUẬN

Module Đăng nhập là cổng vào quan trọng nhất của hệ thống QLNS, đảm bảo bảo mật và trải nghiệm người dùng tốt. Với thiết kế đơn giản nhưng hiệu quả, module này cung cấp:

- **Bảo mật cơ bản**: Xác thực database, kiểm tra trạng thái tài khoản, SQL injection protection
- **Trải nghiệm tốt**: Material Design, validation rõ ràng, navigation mượt mà
- **Tích hợp chặt chẽ**: Kết nối seamless với các module khác thông qua role-based access
- **Khả năng mở rộng**: Dễ dàng thêm tính năng như remember login, biometric auth, password recovery
- **Maintainability**: Code rõ ràng, separation of concerns, error handling tốt

Module này đặt nền móng vững chắc cho toàn bộ hệ thống bảo mật và phân quyền của ứng dụng QLNS, đảm bảo chỉ những người dùng được ủy quyền mới có thể truy cập vào các chức năng tương ứng với vai trò của họ.