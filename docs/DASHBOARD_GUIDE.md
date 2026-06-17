# HƯỚNG DẪN CHI TIẾT: DASHBOARD MODULE

## 📋 TỔNG QUAN

Dashboard là module trung tâm của hệ thống QLNS, đóng vai trò là bảng điều khiển chính sau khi người dùng đăng nhập thành công. Module này cung cấp giao diện thân thiện để truy cập tất cả các chức năng của hệ thống với phân quyền dựa trên vai trò người dùng.

## 🏗️ KIẾN TRÚC MODULE

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/
└── DashboardActivity.java                  # Main dashboard activity class

app/src/main/res/layout/
└── activity_dashboard.xml                  # Dashboard UI layout
```

### Thành phần chính:
```
Dashboard Module
├── Authentication Integration              # Tích hợp với hệ thống đăng nhập
├── Role-Based Access Control              # Phân quyền theo vai trò
├── Navigation Hub                         # Trung tâm điều hướng
├── User Information Display               # Hiển thị thông tin người dùng
└── Module Integration                     # Tích hợp với các module khác
```

## 📊 NGHIỆP VỤ DASHBOARD

### 1. Authentication & Authorization:
- **User Session Management**: Quản lý phiên đăng nhập của người dùng
- **Role-Based Permissions**: Phân quyền dựa trên vai trò (Admin/HR/Manager/Employee)
- **Access Control**: Kiểm soát truy cập các chức năng theo quyền hạn
- **Security Integration**: Tích hợp với hệ thống bảo mật

### 2. Navigation & User Experience:
- **Centralized Navigation**: Trung tâm điều hướng đến tất cả module
- **Responsive Design**: Giao diện thích ứng với Material Design
- **User-Friendly Interface**: Giao diện thân thiện, dễ sử dụng
- **Quick Access**: Truy cập nhanh các chức năng thường dùng

### 3. Role Management:
- **Admin**: Full quyền tất cả chức năng hệ thống
- **HR**: Quản lý nhân sự, lương, nghỉ phép, hợp đồng
- **Manager**: Quản lý nhân viên, xem lương, thống kê
- **Employee**: Chấm công, nghỉ phép, xem lương cá nhân, thông tin cá nhân

---

## 📱 CHI TIẾT CLASS IMPLEMENTATION

## 1️⃣ CLASS DECLARATION & IMPORTS

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/DashboardActivity.java`

### Mục đích:
Main activity class quản lý bảng điều khiển chính của hệ thống với phân quyền và điều hướng.

### Chi tiết code:

#### Package và Import declarations:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chính của ứng dụng

import android.content.Intent;                                             // Import Intent để chuyển đổi giữa các Activity
import android.database.Cursor;                                            // Import Cursor để xử lý dữ liệu từ database
import android.os.Bundle;                                                  // Import Bundle để truyền dữ liệu giữa Activity
import android.widget.Button;                                              // Import Button widget
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo

import androidx.activity.EdgeToEdge;                                       // Import EdgeToEdge để hỗ trợ full screen
import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity làm base class
import androidx.core.graphics.Insets;                                      // Import Insets để xử lý system bars
import androidx.core.view.ViewCompat;                                      // Import ViewCompat để tương thích view
import androidx.core.view.WindowInsetsCompat;                              // Import WindowInsetsCompat để xử lý window insets

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để truy cập database
```

#### Class declaration và member variables:
```java
public class DashboardActivity extends AppCompatActivity {                 // Khai báo class kế thừa AppCompatActivity

    // UI Components - Khai báo các thành phần giao diện
    private TextView tvWelcome;                                            // TextView hiển thị lời chào người dùng
    private Button btnQuanLyNV, btnQuanLyPB, btnQuanLyCV, btnChamCong, btnNghiPhep, btnLuong, btnThongKe, btnThongTin, btnDangXuat, btnQuanLyHD, btnQuanLyTK; // Các button chức năng
    private androidx.cardview.widget.CardView cardQuanLyNV, cardQuanLyPB, cardQuanLyCV, cardLuong, cardThongKe, cardQuanLyHD, cardQuanLyTK; // Các CardView chứa button
    
    // Business Logic Components - Các thành phần logic nghiệp vụ
    private DatabaseHelper dbHelper;                                       // Helper để truy cập database
    private String currentUsername;                                        // Lưu username hiện tại
    private String currentRole;                                            // Lưu vai trò hiện tại
```

---

## 2️⃣ ACTIVITY LIFECYCLE METHODS

### Method onCreate:
```java
    @Override
    public void onCreate(Bundle savedInstanceState) {                      // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi constructor của class cha
        EdgeToEdge.enable(this);                                           // Kích hoạt chế độ full screen edge-to-edge
        setContentView(R.layout.activity_dashboard);                       // Set layout cho Activity
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), (v, insets) -> { // Listener xử lý window insets
            Insets systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars()); // Lấy system bars insets
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom); // Set padding cho view
            return insets;                                                 // Trả về insets
        });
        
        initViews();                                                       // Khởi tạo các view components
        setupDatabase();                                                   // Thiết lập kết nối database
        displayUserInfo();                                                 // Hiển thị thông tin người dùng
        setupButtons();                                                    // Thiết lập sự kiện cho các button
    }
```

---

## 3️⃣ INITIALIZATION METHODS

### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo tất cả view components
        tvWelcome = findViewById(R.id.tv_welcome);                         // Khởi tạo TextView chào mừng
        
        // Buttons - Khởi tạo tất cả các button chức năng
        btnQuanLyNV = findViewById(R.id.btn_quan_ly_nv);                   // Button quản lý nhân viên
        btnQuanLyPB = findViewById(R.id.btn_quan_ly_pb);                   // Button quản lý phòng ban
        btnQuanLyCV = findViewById(R.id.btn_quan_ly_cv);                   // Button quản lý chức vụ
        btnChamCong = findViewById(R.id.btn_cham_cong);                    // Button chấm công
        btnNghiPhep = findViewById(R.id.btn_nghi_phep);                    // Button nghỉ phép
        btnLuong = findViewById(R.id.btn_luong);                           // Button quản lý lương
        btnThongKe = findViewById(R.id.btn_thong_ke);                      // Button thống kê
        btnThongTin = findViewById(R.id.btn_thong_tin);                    // Button thông tin cá nhân
        btnDangXuat = findViewById(R.id.btn_dang_xuat);                    // Button đăng xuất
        
        // CardViews - Khởi tạo tất cả các CardView container
        cardQuanLyNV = findViewById(R.id.card_quan_ly_nv);                 // CardView quản lý nhân viên
        cardQuanLyPB = findViewById(R.id.card_quan_ly_pb);                 // CardView quản lý phòng ban
        cardQuanLyCV = findViewById(R.id.card_quan_ly_cv);                 // CardView quản lý chức vụ
        cardLuong = findViewById(R.id.card_luong);                         // CardView quản lý lương
        cardThongKe = findViewById(R.id.card_thong_ke);                    // CardView thống kê
        cardQuanLyHD = findViewById(R.id.card_quan_ly_hd);                 // CardView quản lý hợp đồng
        btnQuanLyHD = findViewById(R.id.btn_quan_ly_hd);                   // Button quản lý hợp đồng
        cardQuanLyTK = findViewById(R.id.card_quan_ly_tk);                 // CardView quản lý tài khoản
        btnQuanLyTK = findViewById(R.id.btn_quan_ly_tk);                   // Button quản lý tài khoản
    }
```

### Method setupDatabase:
```java
    private void setupDatabase() {                                         // Method thiết lập kết nối database
        dbHelper = new DatabaseHelper(this);                              // Khởi tạo DatabaseHelper với context hiện tại
    }
```
---

## 4️⃣ USER INFORMATION & AUTHENTICATION METHODS

### Method displayUserInfo:
```java
    private void displayUserInfo() {                                       // Method hiển thị thông tin người dùng và áp dụng phân quyền
        currentUsername = getIntent().getStringExtra("username");          // Lấy username từ Intent được truyền từ LoginActivity
        if (currentUsername != null) {                                     // Nếu có username (đăng nhập thành công)
            Cursor cursor = dbHelper.getUserInfo(currentUsername);         // Lấy thông tin user từ database
            if (cursor.moveToFirst()) {                                    // Nếu tìm thấy thông tin user
                String hoTen = cursor.getString(cursor.getColumnIndexOrThrow("HoTen")); // Lấy họ tên từ cursor
                currentRole = cursor.getString(cursor.getColumnIndexOrThrow("VaiTro")); // Lấy vai trò từ cursor
                tvWelcome.setText("Chào " + hoTen + " (" + currentRole + ")"); // Hiển thị lời chào với tên và vai trò
            }
            cursor.close();                                                // Đóng cursor để giải phóng tài nguyên
        } else {                                                           // Nếu không có username (trường hợp lỗi)
            tvWelcome.setText("Chào mừng đến với hệ thống QLNS!");         // Hiển thị lời chào mặc định
            currentRole = "Employee";                                      // Set vai trò mặc định là Employee
        }
        
        applyPermissions();                                                // Gọi method áp dụng phân quyền dựa trên vai trò
    }
```

### Method applyPermissions:
```java
    private void applyPermissions() {                                      // Method áp dụng phân quyền dựa trên vai trò người dùng
        // Mặc định ẩn tất cả các CardView quản lý (chỉ hiển thị cho role có quyền)
        cardQuanLyNV.setVisibility(android.view.View.GONE);                // Ẩn CardView quản lý nhân viên
        cardQuanLyPB.setVisibility(android.view.View.GONE);                // Ẩn CardView quản lý phòng ban
        cardQuanLyCV.setVisibility(android.view.View.GONE);                // Ẩn CardView quản lý chức vụ
        cardLuong.setVisibility(android.view.View.GONE);                   // Ẩn CardView quản lý lương
        cardThongKe.setVisibility(android.view.View.GONE);                 // Ẩn CardView thống kê
        cardQuanLyHD.setVisibility(android.view.View.GONE);                // Ẩn CardView quản lý hợp đồng
        cardQuanLyTK.setVisibility(android.view.View.GONE);                // Ẩn CardView quản lý tài khoản
        
        // Admin: Full quyền tất cả chức năng
        if ("Admin".equalsIgnoreCase(currentRole)) {                       // Nếu vai trò là Admin (không phân biệt hoa thường)
            cardQuanLyNV.setVisibility(android.view.View.VISIBLE);         // Hiển thị quản lý nhân viên
            cardQuanLyPB.setVisibility(android.view.View.VISIBLE);         // Hiển thị quản lý phòng ban
            cardQuanLyCV.setVisibility(android.view.View.VISIBLE);         // Hiển thị quản lý chức vụ
            cardLuong.setVisibility(android.view.View.VISIBLE);            // Hiển thị quản lý lương
            cardThongKe.setVisibility(android.view.View.VISIBLE);          // Hiển thị thống kê
            cardQuanLyHD.setVisibility(android.view.View.VISIBLE);         // Hiển thị quản lý hợp đồng
            cardQuanLyTK.setVisibility(android.view.View.VISIBLE);         // Hiển thị quản lý tài khoản
        }
        // HR: Chuyên về nhân sự - quản lý nhân viên, lương, nghỉ phép
        else if ("HR".equalsIgnoreCase(currentRole)) {                     // Nếu vai trò là HR
            cardQuanLyNV.setVisibility(android.view.View.VISIBLE);         // Hiển thị quản lý nhân viên
            cardQuanLyPB.setVisibility(android.view.View.VISIBLE);         // Hiển thị quản lý phòng ban
            cardQuanLyCV.setVisibility(android.view.View.VISIBLE);         // Hiển thị quản lý chức vụ
            cardLuong.setVisibility(android.view.View.VISIBLE);            // Hiển thị quản lý lương
            cardThongKe.setVisibility(android.view.View.VISIBLE);          // Hiển thị thống kê
            cardQuanLyHD.setVisibility(android.view.View.VISIBLE);         // Hiển thị quản lý hợp đồng
        }
        // Manager: Quản lý cấp trung - quản lý nhân viên, xem lương
        else if ("Manager".equalsIgnoreCase(currentRole)) {                // Nếu vai trò là Manager
            cardQuanLyNV.setVisibility(android.view.View.VISIBLE);         // Hiển thị quản lý nhân viên
            cardLuong.setVisibility(android.view.View.VISIBLE);            // Hiển thị quản lý lương
            cardThongKe.setVisibility(android.view.View.VISIBLE);          // Hiển thị thống kê
        }
        // Employee: Chấm công, nghỉ phép, xem lương cá nhân, thông tin cá nhân
        else if ("Employee".equalsIgnoreCase(currentRole)) {               // Nếu vai trò là Employee
            cardLuong.setVisibility(android.view.View.VISIBLE);            // Hiển thị quản lý lương (chỉ xem cá nhân)
            btnLuong.setText("XEM LƯƠNG CÁ NHÂN");                         // Đổi text button thành "XEM LƯƠNG CÁ NHÂN"
        }
        
        // Các nút luôn hiển thị cho tất cả vai trò (chức năng cơ bản)
        btnChamCong.setVisibility(android.view.View.VISIBLE);              // Hiển thị button chấm công
        btnNghiPhep.setVisibility(android.view.View.VISIBLE);              // Hiển thị button nghỉ phép
        btnThongTin.setVisibility(android.view.View.VISIBLE);              // Hiển thị button thông tin cá nhân
        btnDangXuat.setVisibility(android.view.View.VISIBLE);              // Hiển thị button đăng xuất
    }
```

---

## 5️⃣ NAVIGATION & EVENT HANDLING METHODS

### Method setupButtons:
```java
    private void setupButtons() {                                          // Method thiết lập sự kiện click cho tất cả các button
        // Button Quản lý Nhân viên
        btnQuanLyNV.setOnClickListener(v -> {                              // Set OnClickListener cho button quản lý nhân viên
            if (isAdminOrHR()) {                                           // Kiểm tra quyền Admin/HR/Manager
                Intent intent = new Intent(DashboardActivity.this, QuanLyNhanVienActivity.class); // Tạo Intent chuyển đến QuanLyNhanVienActivity
                startActivity(intent);                                     // Khởi động Activity
            } else {                                                       // Nếu không có quyền
                Toast.makeText(this, "Bạn không có quyền truy cập chức năng này", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            }
        });
        
        // Button Quản lý Phòng ban
        btnQuanLyPB.setOnClickListener(v -> {                              // Set OnClickListener cho button quản lý phòng ban
            if ("Admin".equalsIgnoreCase(currentRole) || "HR".equalsIgnoreCase(currentRole)) { // Kiểm tra quyền Admin hoặc HR
                Intent intent = new Intent(DashboardActivity.this, QuanLyPhongBanActivity.class); // Tạo Intent chuyển đến QuanLyPhongBanActivity
                intent.putExtra("role", currentRole);                      // Truyền vai trò hiện tại qua Intent
                startActivity(intent);                                     // Khởi động Activity
            } else {                                                       // Nếu không có quyền
                Toast.makeText(this, "Bạn không có quyền truy cập chức năng này", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            }
        });
        
        // Button Quản lý Chức vụ
        btnQuanLyCV.setOnClickListener(v -> {                              // Set OnClickListener cho button quản lý chức vụ
            if ("Admin".equalsIgnoreCase(currentRole) || "HR".equalsIgnoreCase(currentRole)) { // Kiểm tra quyền Admin hoặc HR
                Intent intent = new Intent(DashboardActivity.this, QuanLyChucVuActivity.class); // Tạo Intent chuyển đến QuanLyChucVuActivity
                intent.putExtra("role", currentRole);                      // Truyền vai trò hiện tại qua Intent
                startActivity(intent);                                     // Khởi động Activity
            } else {                                                       // Nếu không có quyền
                Toast.makeText(this, "Bạn không có quyền truy cập chức năng này", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            }
        });
        
        // Button Chấm công (tất cả vai trò đều có quyền)
        btnChamCong.setOnClickListener(v -> {                              // Set OnClickListener cho button chấm công
            Intent intent = new Intent(DashboardActivity.this, ChamCongActivity.class); // Tạo Intent chuyển đến ChamCongActivity
            intent.putExtra("username", currentUsername);                  // Truyền username qua Intent
            intent.putExtra("role", currentRole);                          // Truyền vai trò qua Intent
            startActivity(intent);                                         // Khởi động Activity
        });
        
        // Button Nghỉ phép (tất cả vai trò đều có quyền)
        btnNghiPhep.setOnClickListener(v -> {                              // Set OnClickListener cho button nghỉ phép
            Intent intent = new Intent(DashboardActivity.this, NghiPhepActivity.class); // Tạo Intent chuyển đến NghiPhepActivity
            intent.putExtra("username", currentUsername);                  // Truyền username qua Intent
            intent.putExtra("role", currentRole);                          // Truyền vai trò qua Intent
            startActivity(intent);                                         // Khởi động Activity
        });
```
        
        // Button Quản lý Lương
        btnLuong.setOnClickListener(v -> {                                 // Set OnClickListener cho button quản lý lương
            Intent intent = new Intent(DashboardActivity.this, QuanLyLuongActivity.class); // Tạo Intent chuyển đến QuanLyLuongActivity
            intent.putExtra("username", currentUsername);                  // Truyền username qua Intent
            intent.putExtra("role", currentRole);                          // Truyền vai trò qua Intent
            startActivity(intent);                                         // Khởi động Activity
        });
        
        // Button Thống kê (Admin/HR/Manager)
        btnThongKe.setOnClickListener(v -> {                               // Set OnClickListener cho button thống kê
            if ("Admin".equalsIgnoreCase(currentRole) || "HR".equalsIgnoreCase(currentRole) || "Manager".equalsIgnoreCase(currentRole)) { // Kiểm tra quyền Admin/HR/Manager
                Intent intent = new Intent(DashboardActivity.this, ThongKeActivity.class); // Tạo Intent chuyển đến ThongKeActivity
                intent.putExtra("username", currentUsername);              // Truyền username qua Intent
                intent.putExtra("role", currentRole);                      // Truyền vai trò qua Intent
                startActivity(intent);                                     // Khởi động Activity
            } else {                                                       // Nếu không có quyền
                Toast.makeText(this, "Bạn không có quyền truy cập chức năng này", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            }
        });
        
        // Button Thông tin cá nhân (tất cả vai trò đều có quyền)
        btnThongTin.setOnClickListener(v -> {                              // Set OnClickListener cho button thông tin cá nhân
            Intent intent = new Intent(DashboardActivity.this, ThongTinCaNhanActivity.class); // Tạo Intent chuyển đến ThongTinCaNhanActivity
            intent.putExtra("username", currentUsername);                  // Truyền username qua Intent
            startActivity(intent);                                         // Khởi động Activity
        });

        // Button Quản lý Hợp đồng (Admin/HR)
        btnQuanLyHD.setOnClickListener(v -> {                              // Set OnClickListener cho button quản lý hợp đồng
            if ("Admin".equalsIgnoreCase(currentRole) || "HR".equalsIgnoreCase(currentRole)) { // Kiểm tra quyền Admin hoặc HR
                Intent intent = new Intent(DashboardActivity.this, QuanLyHopDongActivity.class); // Tạo Intent chuyển đến QuanLyHopDongActivity
                startActivity(intent);                                     // Khởi động Activity
            } else {                                                       // Nếu không có quyền
                Toast.makeText(this, "Bạn không có quyền truy cập chức năng này", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            }
        });

        // Button Quản lý Tài khoản (chỉ Admin)
        btnQuanLyTK.setOnClickListener(v -> {                              // Set OnClickListener cho button quản lý tài khoản
            if ("Admin".equalsIgnoreCase(currentRole)) {                   // Kiểm tra quyền Admin (chỉ Admin mới có quyền)
                Intent intent = new Intent(DashboardActivity.this, QuanLyTaiKhoanActivity.class); // Tạo Intent chuyển đến QuanLyTaiKhoanActivity
                startActivity(intent);                                     // Khởi động Activity
            } else {                                                       // Nếu không có quyền
                Toast.makeText(this, "Bạn không có quyền truy cập chức năng này", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            }
        });
        
        // Button Đăng xuất (tất cả vai trò đều có quyền)
        btnDangXuat.setOnClickListener(v -> {                              // Set OnClickListener cho button đăng xuất
            Intent intent = new Intent(DashboardActivity.this, MainActivity.class); // Tạo Intent chuyển về MainActivity (trang chủ)
            intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK); // Set flags để clear task stack
            startActivity(intent);                                         // Khởi động MainActivity
            finish();                                                      // Kết thúc DashboardActivity hiện tại
        });
    }
```

### Method isAdminOrHR:
```java
    private boolean isAdminOrHR() {                                        // Method helper kiểm tra quyền Admin/HR/Manager
        return "Admin".equalsIgnoreCase(currentRole) || "HR".equalsIgnoreCase(currentRole) || "Manager".equalsIgnoreCase(currentRole); // Trả về true nếu là Admin, HR hoặc Manager
    }
}
```

---

## 📱 CHI TIẾT LAYOUT IMPLEMENTATION

## 6️⃣ XML LAYOUT STRUCTURE

**Đường dẫn**: `app/src/main/res/layout/activity_dashboard.xml`

### Mục đích:
Layout chính của Dashboard với thiết kế Material Design, sử dụng CardView và ScrollView để tạo giao diện thân thiện.

### Chi tiết XML:

#### Root LinearLayout:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- XML declaration với encoding UTF-8 -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- Root LinearLayout với namespace Android -->
    xmlns:app="http://schemas.android.com/apk/res-auto"                    <!-- Namespace cho app attributes -->
    xmlns:tools="http://schemas.android.com/tools"                         <!-- Namespace cho tools attributes -->
    android:id="@+id/main"                                                 <!-- ID của root layout -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng full màn hình -->
    android:layout_height="match_parent"                                   <!-- Chiều cao full màn hình -->
    android:orientation="vertical"                                         <!-- Orientation dọc -->
    android:background="#f8f9fa"                                           <!-- Background màu xám nhạt -->
    tools:context=".DashboardActivity">                                    <!-- Context cho tools -->
```

#### Header Section:
```xml
    <!-- Header -->
    <LinearLayout                                                          <!-- LinearLayout chứa header -->
        android:layout_width="match_parent"                                <!-- Chiều rộng full -->
        android:layout_height="wrap_content"                               <!-- Chiều cao wrap content -->
        android:orientation="vertical"                                     <!-- Orientation dọc -->
        android:background="#2196F3"                                       <!-- Background màu xanh Material Design -->
        android:padding="20dp"                                             <!-- Padding 20dp tất cả các cạnh -->
        android:elevation="8dp">                                           <!-- Elevation tạo shadow -->

        <TextView                                                          <!-- TextView tiêu đề chính -->
            android:layout_width="match_parent"                            <!-- Chiều rộng full -->
            android:layout_height="wrap_content"                           <!-- Chiều cao wrap content -->
            android:text="BẢNG ĐIỀU KHIỂN"                                 <!-- Text tiêu đề -->
            android:textSize="24sp"                                        <!-- Kích thước text 24sp -->
            android:textStyle="bold"                                       <!-- Style bold -->
            android:textColor="@android:color/white"                       <!-- Màu text trắng -->
            android:gravity="center"                                       <!-- Căn giữa -->
            android:layout_marginBottom="8dp"                              <!-- Margin bottom 8dp -->
            android:letterSpacing="0.1" />                                 <!-- Letter spacing 0.1 -->

        <TextView                                                          <!-- TextView lời chào -->
            android:id="@+id/tv_welcome"                                   <!-- ID để reference từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng full -->
            android:layout_height="wrap_content"                           <!-- Chiều cao wrap content -->
            android:text="Chào mừng!"                                      <!-- Text mặc định -->
            android:textSize="16sp"                                        <!-- Kích thước text 16sp -->
            android:textColor="@android:color/white"                       <!-- Màu text trắng -->
            android:gravity="center"                                       <!-- Căn giữa -->
            android:alpha="0.9" />                                         <!-- Alpha 0.9 để tạo hiệu ứng mờ nhẹ -->

    </LinearLayout>
```
#### Menu Section với ScrollView:
```xml
    <!-- Menu chức năng -->
    <ScrollView                                                            <!-- ScrollView để cuộn khi có nhiều item -->
        android:layout_width="match_parent"                                <!-- Chiều rộng full -->
        android:layout_height="0dp"                                        <!-- Chiều cao 0dp để sử dụng layout_weight -->
        android:layout_weight="1"                                          <!-- Weight 1 để chiếm hết không gian còn lại -->
        android:padding="16dp">                                            <!-- Padding 16dp -->

        <LinearLayout                                                      <!-- LinearLayout chứa tất cả menu items -->
            android:layout_width="match_parent"                            <!-- Chiều rộng full -->
            android:layout_height="wrap_content"                           <!-- Chiều cao wrap content -->
            android:orientation="vertical">                                <!-- Orientation dọc -->
```

#### CardView Menu Items:
```xml
            <!-- Quản lý nhân viên -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho menu quản lý nhân viên -->
                android:id="@+id/card_quan_ly_nv"                          <!-- ID để reference từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="80dp"                               <!-- Chiều cao cố định 80dp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp tạo shadow -->
                app:cardBackgroundColor="#4CAF50">                         <!-- Background màu xanh lá -->

                <Button                                                    <!-- Button bên trong CardView -->
                    android:id="@+id/btn_quan_ly_nv"                       <!-- ID để reference từ Java code -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="match_parent"                   <!-- Chiều cao full -->
                    android:text="QUẢN LÝ NHÂN VIÊN"                       <!-- Text hiển thị -->
                    android:textColor="@android:color/white"               <!-- Màu text trắng -->
                    android:textSize="16sp"                                <!-- Kích thước text 16sp -->
                    android:textStyle="bold"                               <!-- Style bold -->
                    android:background="@android:color/transparent"        <!-- Background trong suốt -->
                    android:gravity="center" />                            <!-- Căn giữa -->

            </androidx.cardview.widget.CardView>

            <!-- Quản lý hợp đồng -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho menu quản lý hợp đồng -->
                android:id="@+id/card_quan_ly_hd"                          <!-- ID để reference từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="80dp"                               <!-- Chiều cao cố định 80dp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp -->
                app:cardBackgroundColor="#009688">                         <!-- Background màu teal -->

                <Button                                                    <!-- Button quản lý hợp đồng -->
                    android:id="@+id/btn_quan_ly_hd"                       <!-- ID để reference từ Java code -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="match_parent"                   <!-- Chiều cao full -->
                    android:text="QUẢN LÝ HỢP ĐỒNG"                        <!-- Text hiển thị -->
                    android:textColor="@android:color/white"               <!-- Màu text trắng -->
                    android:textSize="16sp"                                <!-- Kích thước text 16sp -->
                    android:textStyle="bold"                               <!-- Style bold -->
                    android:background="@android:color/transparent"        <!-- Background trong suốt -->
                    android:gravity="center" />                            <!-- Căn giữa -->

            </androidx.cardview.widget.CardView>

            <!-- Quản lý phòng ban -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho menu quản lý phòng ban -->
                android:id="@+id/card_quan_ly_pb"                          <!-- ID để reference từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="80dp"                               <!-- Chiều cao cố định 80dp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp -->
                app:cardBackgroundColor="#FF9800">                         <!-- Background màu cam -->

                <Button                                                    <!-- Button quản lý phòng ban -->
                    android:id="@+id/btn_quan_ly_pb"                       <!-- ID để reference từ Java code -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="match_parent"                   <!-- Chiều cao full -->
                    android:text="QUẢN LÝ PHÒNG BAN"                       <!-- Text hiển thị -->
                    android:textColor="@android:color/white"               <!-- Màu text trắng -->
                    android:textSize="16sp"                                <!-- Kích thước text 16sp -->
                    android:textStyle="bold"                               <!-- Style bold -->
                    android:background="@android:color/transparent"        <!-- Background trong suốt -->
                    android:gravity="center" />                            <!-- Căn giữa -->

            </androidx.cardview.widget.CardView>

            <!-- Quản lý chức vụ -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho menu quản lý chức vụ -->
                android:id="@+id/card_quan_ly_cv"                          <!-- ID để reference từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="80dp"                               <!-- Chiều cao cố định 80dp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp -->
                app:cardBackgroundColor="#E91E63">                         <!-- Background màu hồng -->

                <Button                                                    <!-- Button quản lý chức vụ -->
                    android:id="@+id/btn_quan_ly_cv"                       <!-- ID để reference từ Java code -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="match_parent"                   <!-- Chiều cao full -->
                    android:text="QUẢN LÝ CHỨC VỤ"                         <!-- Text hiển thị -->
                    android:textColor="@android:color/white"               <!-- Màu text trắng -->
                    android:textSize="16sp"                                <!-- Kích thước text 16sp -->
                    android:textStyle="bold"                               <!-- Style bold -->
                    android:background="@android:color/transparent"        <!-- Background trong suốt -->
                    android:gravity="center" />                            <!-- Căn giữa -->

            </androidx.cardview.widget.CardView>

            <!-- Chấm công -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho menu chấm công -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="80dp"                               <!-- Chiều cao cố định 80dp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp -->
                app:cardBackgroundColor="#9C27B0">                         <!-- Background màu tím -->

                <Button                                                    <!-- Button chấm công -->
                    android:id="@+id/btn_cham_cong"                        <!-- ID để reference từ Java code -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="match_parent"                   <!-- Chiều cao full -->
                    android:text="QUẢN LÝ CHẤM CÔNG"                       <!-- Text hiển thị -->
                    android:textColor="@android:color/white"               <!-- Màu text trắng -->
                    android:textSize="16sp"                                <!-- Kích thước text 16sp -->
                    android:textStyle="bold"                               <!-- Style bold -->
                    android:background="@android:color/transparent"        <!-- Background trong suốt -->
                    android:gravity="center" />                            <!-- Căn giữa -->

            </androidx.cardview.widget.CardView>
```
            <!-- Nghỉ phép -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho menu nghỉ phép -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="80dp"                               <!-- Chiều cao cố định 80dp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp -->
                app:cardBackgroundColor="#FF5722">                         <!-- Background màu đỏ cam -->

                <Button                                                    <!-- Button nghỉ phép -->
                    android:id="@+id/btn_nghi_phep"                        <!-- ID để reference từ Java code -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="match_parent"                   <!-- Chiều cao full -->
                    android:text="QUẢN LÝ NGHỈ PHÉP"                       <!-- Text hiển thị -->
                    android:textColor="@android:color/white"               <!-- Màu text trắng -->
                    android:textSize="16sp"                                <!-- Kích thước text 16sp -->
                    android:textStyle="bold"                               <!-- Style bold -->
                    android:background="@android:color/transparent"        <!-- Background trong suốt -->
                    android:gravity="center" />                            <!-- Căn giữa -->

            </androidx.cardview.widget.CardView>

            <!-- Lương -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho menu lương -->
                android:id="@+id/card_luong"                               <!-- ID để reference từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="80dp"                               <!-- Chiều cao cố định 80dp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp -->
                app:cardBackgroundColor="#3F51B5">                         <!-- Background màu indigo -->

                <Button                                                    <!-- Button lương -->
                    android:id="@+id/btn_luong"                            <!-- ID để reference từ Java code -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="match_parent"                   <!-- Chiều cao full -->
                    android:text="QUẢN LÝ LƯƠNG"                           <!-- Text hiển thị -->
                    android:textColor="@android:color/white"               <!-- Màu text trắng -->
                    android:textSize="16sp"                                <!-- Kích thước text 16sp -->
                    android:textStyle="bold"                               <!-- Style bold -->
                    android:background="@android:color/transparent"        <!-- Background trong suốt -->
                    android:gravity="center" />                            <!-- Căn giữa -->

            </androidx.cardview.widget.CardView>

            <!-- Thống kê -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho menu thống kê -->
                android:id="@+id/card_thong_ke"                            <!-- ID để reference từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="80dp"                               <!-- Chiều cao cố định 80dp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp -->
                app:cardBackgroundColor="#00BCD4">                         <!-- Background màu cyan -->

                <Button                                                    <!-- Button thống kê -->
                    android:id="@+id/btn_thong_ke"                         <!-- ID để reference từ Java code -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="match_parent"                   <!-- Chiều cao full -->
                    android:text="THỐNG KÊ TỔNG QUAN"                      <!-- Text hiển thị -->
                    android:textColor="@android:color/white"               <!-- Màu text trắng -->
                    android:textSize="16sp"                                <!-- Kích thước text 16sp -->
                    android:textStyle="bold"                               <!-- Style bold -->
                    android:background="@android:color/transparent"        <!-- Background trong suốt -->
                    android:gravity="center" />                            <!-- Căn giữa -->

            </androidx.cardview.widget.CardView>

            <!-- Thông tin cá nhân -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho menu thông tin cá nhân -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="80dp"                               <!-- Chiều cao cố định 80dp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp -->
                app:cardBackgroundColor="#9C27B0">                         <!-- Background màu tím -->

                <Button                                                    <!-- Button thông tin cá nhân -->
                    android:id="@+id/btn_thong_tin"                        <!-- ID để reference từ Java code -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="match_parent"                   <!-- Chiều cao full -->
                    android:text="THÔNG TIN CÁ NHÂN"                       <!-- Text hiển thị -->
                    android:textColor="@android:color/white"               <!-- Màu text trắng -->
                    android:textSize="16sp"                                <!-- Kích thước text 16sp -->
                    android:textStyle="bold"                               <!-- Style bold -->
                    android:background="@android:color/transparent"        <!-- Background trong suốt -->
                    android:gravity="center" />                            <!-- Căn giữa -->

            </androidx.cardview.widget.CardView>

            <!-- Quản lý tài khoản (Chỉ Admin) -->
            <androidx.cardview.widget.CardView                             <!-- CardView cho menu quản lý tài khoản -->
                android:id="@+id/card_quan_ly_tk"                          <!-- ID để reference từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="80dp"                               <!-- Chiều cao cố định 80dp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                app:cardCornerRadius="12dp"                                <!-- Corner radius 12dp -->
                app:cardElevation="6dp"                                    <!-- Elevation 6dp -->
                app:cardBackgroundColor="#607D8B">                         <!-- Background màu blue grey -->

                <Button                                                    <!-- Button quản lý tài khoản -->
                    android:id="@+id/btn_quan_ly_tk"                       <!-- ID để reference từ Java code -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="match_parent"                   <!-- Chiều cao full -->
                    android:text="QUẢN LÝ TÀI KHOẢN"                       <!-- Text hiển thị -->
                    android:textColor="@android:color/white"               <!-- Màu text trắng -->
                    android:textSize="16sp"                                <!-- Kích thước text 16sp -->
                    android:textStyle="bold"                               <!-- Style bold -->
                    android:background="@android:color/transparent"        <!-- Background trong suốt -->
                    android:gravity="center" />                            <!-- Căn giữa -->

            </androidx.cardview.widget.CardView>

            <!-- Đăng xuất -->
            <Button                                                        <!-- Button đăng xuất (không dùng CardView) -->
                android:id="@+id/btn_dang_xuat"                            <!-- ID để reference từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="60dp"                               <!-- Chiều cao 60dp -->
                android:text="ĐĂNG XUẤT"                                   <!-- Text hiển thị -->
                android:textSize="16sp"                                    <!-- Kích thước text 16sp -->
                android:textColor="#666"                                   <!-- Màu text xám -->
                android:background="@android:color/transparent"            <!-- Background trong suốt -->
                android:layout_marginTop="16dp" />                         <!-- Margin top 16dp -->

        </LinearLayout>                                                    <!-- Kết thúc LinearLayout chứa menu -->

    </ScrollView>                                                          <!-- Kết thúc ScrollView -->

</LinearLayout>                                                            <!-- Kết thúc root LinearLayout -->
```

---

## 🔗 TÍCH HỢP VÀ KIẾN TRÚC

### 1. Authentication Integration:
- **Session Management**: Nhận username từ LoginActivity qua Intent
- **User Information**: Lấy thông tin user từ DatabaseHelper.getUserInfo()
- **Role Detection**: Xác định vai trò để áp dụng phân quyền
- **Security**: Kiểm tra quyền truy cập trước khi chuyển Activity

### 2. Navigation Architecture:
- **Hub Pattern**: Dashboard là trung tâm điều hướng đến tất cả module
- **Intent-Based**: Sử dụng Intent để truyền dữ liệu giữa các Activity
- **Parameter Passing**: Truyền username và role cho các module con
- **Back Stack Management**: Quản lý back stack với flags phù hợp

### 3. Permission System:
- **Role-Based Access Control (RBAC)**: Phân quyền dựa trên 4 vai trò chính
- **Dynamic UI**: Ẩn/hiện CardView dựa trên quyền hạn
- **Security Checks**: Kiểm tra quyền trong cả UI và event handlers
- **Graceful Degradation**: Hiển thị thông báo lỗi khi không có quyền

### 4. User Experience:
- **Material Design**: Sử dụng CardView, elevation, colors theo Material Design
- **Responsive Layout**: ScrollView đảm bảo hiển thị tốt trên mọi màn hình
- **Visual Hierarchy**: Sử dụng màu sắc và typography để tạo hierarchy
- **Accessibility**: Text size, contrast, touch target size phù hợp

### 5. Module Integration:
- **Loose Coupling**: Các module độc lập, chỉ liên kết qua Intent
- **Data Consistency**: Truyền username/role để đảm bảo consistency
- **Error Handling**: Xử lý lỗi khi module không tồn tại
- **Performance**: Lazy loading - chỉ khởi động module khi cần

---

## 📋 BUSINESS RULES & SECURITY

### 1. Role Permissions:
```
Admin:
├── Quản lý nhân viên ✓
├── Quản lý phòng ban ✓
├── Quản lý chức vụ ✓
├── Quản lý hợp đồng ✓
├── Quản lý tài khoản ✓
├── Quản lý lương ✓
├── Thống kê ✓
├── Chấm công ✓
├── Nghỉ phép ✓
└── Thông tin cá nhân ✓

HR:
├── Quản lý nhân viên ✓
├── Quản lý phòng ban ✓
├── Quản lý chức vụ ✓
├── Quản lý hợp đồng ✓
├── Quản lý lương ✓
├── Thống kê ✓
├── Chấm công ✓
├── Nghỉ phép ✓
└── Thông tin cá nhân ✓

Manager:
├── Quản lý nhân viên ✓
├── Quản lý lương ✓ (xem only)
├── Thống kê ✓
├── Chấm công ✓
├── Nghỉ phép ✓
└── Thông tin cá nhân ✓

Employee:
├── Chấm công ✓
├── Nghỉ phép ✓
├── Xem lương cá nhân ✓
└── Thông tin cá nhân ✓
```

### 2. Security Features:
- **Access Control**: Kiểm tra quyền trước khi truy cập chức năng
- **UI Security**: Ẩn các chức năng không có quyền
- **Session Security**: Quản lý session an toàn
- **Logout Security**: Clear task stack khi đăng xuất

Dashboard module hoàn thành với đầy đủ chức năng phân quyền, điều hướng và tích hợp với tất cả module khác trong hệ thống QLNS.