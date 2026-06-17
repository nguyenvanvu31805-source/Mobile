# HƯỚNG DẪN CHI TIẾT: MAIN PAGE MODULE

## 📋 TỔNG QUAN

MainActivity là trang chủ và điểm khởi đầu của ứng dụng Smart HR, đóng vai trò là trang giới thiệu công khai không yêu cầu đăng nhập. Module này cung cấp thông tin tổng quan về hệ thống, các tính năng chính, lợi ích và hướng dẫn người dùng bắt đầu sử dụng ứng dụng.

## 🏗️ KIẾN TRÚC MODULE

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/
└── MainActivity.java                       # Main landing page activity class

app/src/main/res/layout/
└── activity_main.xml                       # Main page UI layout

app/src/main/res/drawable/
├── btn_dang_nhap.xml                       # Login button background
├── btn_dang_ky.xml                         # Register button background
├── btn_bat_dau.xml                         # Get started button background
└── card_feature.xml                        # Feature card background
```

### Thành phần chính:
```
Main Page Module
├── Hero Section                            # Banner chính với CTA buttons
├── Features Grid                           # Lưới tính năng chính
├── Statistics Section                      # Thống kê thành tích
├── About Section                           # Giới thiệu về Smart HR
├── Detailed Features                       # Chi tiết tính năng
├── Benefits Section                        # Lợi ích vượt trội
├── Call to Action                          # Kêu gọi hành động
└── Footer                                  # Thông tin bản quyền
```

## 📊 NGHIỆP VỤ MAIN PAGE

### 1. Landing Page Functions:
- **Public Introduction**: Giới thiệu hệ thống cho người dùng chưa đăng nhập
- **Feature Showcase**: Trình bày các tính năng chính một cách trực quan
- **User Onboarding**: Hướng dẫn người dùng mới bắt đầu sử dụng
- **Marketing Content**: Nội dung marketing để thu hút người dùng

### 2. Navigation Gateway:
- **Login Access**: Điều hướng đến trang đăng nhập
- **Registration Flow**: Điều hướng đến trang đăng ký
- **User Journey**: Tạo luồng trải nghiệm người dùng mượt mà

### 3. Information Architecture:
- **Progressive Disclosure**: Hiển thị thông tin theo mức độ chi tiết tăng dần
- **Visual Hierarchy**: Sắp xếp nội dung theo thứ tự ưu tiên
- **Responsive Design**: Thiết kế thích ứng với nhiều kích thước màn hình

---

## 📱 CHI TIẾT CLASS IMPLEMENTATION

## 1️⃣ CLASS DECLARATION & IMPORTS

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/MainActivity.java`

### Mục đích:
Activity chính làm trang chủ giới thiệu hệ thống Smart HR với giao diện marketing và điều hướng đến các chức năng đăng nhập/đăng ký.

### Chi tiết code:

#### Package và Import declarations:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chính của ứng dụng

import android.content.Intent;                                             // Import Intent để chuyển đổi giữa các Activity
import android.os.Bundle;                                                  // Import Bundle để truyền dữ liệu giữa Activity
import android.widget.Button;                                              // Import Button widget

import androidx.activity.EdgeToEdge;                                       // Import EdgeToEdge để hỗ trợ full screen
import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity làm base class
import androidx.core.graphics.Insets;                                      // Import Insets để xử lý system bars
import androidx.core.view.ViewCompat;                                      // Import ViewCompat để tương thích view
import androidx.core.view.WindowInsetsCompat;                              // Import WindowInsetsCompat để xử lý window insets
```

#### Class declaration và member variables:
```java
public class MainActivity extends AppCompatActivity {                      // Khai báo class kế thừa AppCompatActivity

    // UI Components - Khai báo các thành phần giao diện
    private Button btnDangNhap, btnDangKy, btnBatDau;                     // Các button điều hướng chính
```

---

## 2️⃣ ACTIVITY LIFECYCLE METHODS

### Method onCreate:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi constructor của class cha
        EdgeToEdge.enable(this);                                           // Kích hoạt chế độ full screen edge-to-edge
        setContentView(R.layout.activity_main);                            // Set layout cho Activity
        
        // Apply window insets to the main LinearLayout inside ScrollView
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), (v, insets) -> { // Listener xử lý window insets
            Insets systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars()); // Lấy system bars insets
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom); // Set padding cho view
            return insets;                                                 // Trả về insets
        });
        
        initViews();                                                       // Khởi tạo các view components
        setupButtons();                                                    // Thiết lập sự kiện cho các button
    }
```

---

## 3️⃣ INITIALIZATION METHODS

### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo tất cả view components
        btnDangNhap = findViewById(R.id.btn_dang_nhap);                    // Khởi tạo button đăng nhập
        btnDangKy = findViewById(R.id.btn_dang_ky);                        // Khởi tạo button đăng ký
        btnBatDau = findViewById(R.id.btn_bat_dau);                        // Khởi tạo button bắt đầu (CTA)
    }
```

### Method setupButtons:
```java
    private void setupButtons() {                                          // Method thiết lập sự kiện click cho các button
        // Button Đăng nhập - Điều hướng đến LoginActivity
        btnDangNhap.setOnClickListener(v -> {                              // Set OnClickListener cho button đăng nhập
            Intent intent = new Intent(MainActivity.this, LoginActivity.class); // Tạo Intent chuyển đến LoginActivity
            startActivity(intent);                                         // Khởi động LoginActivity
        });
        
        // Button Đăng ký - Điều hướng đến RegisterActivity
        btnDangKy.setOnClickListener(v -> {                                // Set OnClickListener cho button đăng ký
            Intent intent = new Intent(MainActivity.this, RegisterActivity.class); // Tạo Intent chuyển đến RegisterActivity
            startActivity(intent);                                         // Khởi động RegisterActivity
        });
        
        // Button Bắt đầu - Điều hướng đến RegisterActivity (CTA chính)
        btnBatDau.setOnClickListener(v -> {                                // Set OnClickListener cho button bắt đầu
            Intent intent = new Intent(MainActivity.this, RegisterActivity.class); // Tạo Intent chuyển đến RegisterActivity
            startActivity(intent);                                         // Khởi động RegisterActivity (khuyến khích đăng ký)
        });
    }
}
```

---

## 📱 CHI TIẾT LAYOUT IMPLEMENTATION

## 4️⃣ XML LAYOUT STRUCTURE

**Đường dẫn**: `app/src/main/res/layout/activity_main.xml`

### Mục đích:
Layout trang chủ với thiết kế marketing chuyên nghiệp, sử dụng ScrollView để chứa nhiều section thông tin và Material Design để tạo trải nghiệm người dùng tốt.

### Chi tiết XML:

#### Root ScrollView:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- XML declaration với encoding UTF-8 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"    <!-- Root ScrollView để cuộn nội dung dài -->
    xmlns:app="http://schemas.android.com/apk/res-auto"                    <!-- Namespace cho app attributes -->
    xmlns:tools="http://schemas.android.com/tools"                         <!-- Namespace cho tools attributes -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng full màn hình -->
    android:layout_height="match_parent"                                   <!-- Chiều cao full màn hình -->
    android:background="#f0f2f5"                                           <!-- Background màu xám nhạt -->
    tools:context=".MainActivity">                                         <!-- Context cho tools -->

    <LinearLayout                                                          <!-- LinearLayout chính chứa tất cả nội dung -->
        android:id="@+id/main"                                             <!-- ID để reference từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng full -->
        android:layout_height="wrap_content"                               <!-- Chiều cao wrap content -->
        android:orientation="vertical">                                    <!-- Orientation dọc -->
```

#### Hero Section:
```xml
        <!-- Hero Section -->
        <RelativeLayout                                                    <!-- RelativeLayout cho hero section -->
            android:layout_width="match_parent"                            <!-- Chiều rộng full -->
            android:layout_height="wrap_content"                           <!-- Chiều cao wrap content -->
            android:background="#2196F3">                                  <!-- Background màu xanh Material Design -->

            <LinearLayout                                                  <!-- LinearLayout chứa nội dung hero -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:orientation="vertical"                             <!-- Orientation dọc -->
                android:gravity="center">                                  <!-- Căn giữa nội dung -->

                <ImageView                                                 <!-- ImageView hiển thị hình ảnh chính -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="220dp"                          <!-- Chiều cao cố định 220dp -->
                    android:src="@drawable/quan_tri_nhan_su"               <!-- Source image quản trị nhân sự -->
                    android:scaleType="centerCrop"                         <!-- Scale type center crop -->
                    android:layout_marginBottom="24dp" />                  <!-- Margin bottom 24dp -->

                <LinearLayout                                              <!-- LinearLayout chứa text và buttons -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng full -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao wrap content -->
                    android:orientation="vertical"                         <!-- Orientation dọc -->
                    android:gravity="center"                               <!-- Căn giữa nội dung -->
                    android:paddingLeft="24dp"                             <!-- Padding left 24dp -->
                    android:paddingRight="24dp"                            <!-- Padding right 24dp -->
                    android:paddingBottom="40dp">                          <!-- Padding bottom 40dp -->

                    <TextView                                              <!-- TextView tiêu đề chính -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="SMART HR"                            <!-- Text tiêu đề -->
                        android:textSize="36sp"                            <!-- Kích thước text lớn 36sp -->
                        android:textStyle="bold"                           <!-- Style bold -->
                        android:textColor="@android:color/white"           <!-- Màu text trắng -->
                        android:gravity="center"                           <!-- Căn giữa -->
                        android:layout_marginBottom="8dp"                  <!-- Margin bottom 8dp -->
                        android:letterSpacing="0.1" />                     <!-- Letter spacing 0.1 -->

                    <TextView                                              <!-- TextView subtitle -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="Hệ Thống Quản Lý Nhân Sự Thông Minh" <!-- Text subtitle -->
                        android:textSize="18sp"                            <!-- Kích thước text 18sp -->
                        android:textColor="@android:color/white"           <!-- Màu text trắng -->
                        android:gravity="center"                           <!-- Căn giữa -->
                        android:layout_marginBottom="8dp"                  <!-- Margin bottom 8dp -->
                        android:alpha="0.9" />                             <!-- Alpha 0.9 để tạo hiệu ứng mờ nhẹ -->

                    <TextView                                              <!-- TextView description -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="Giải pháp toàn diện cho doanh nghiệp hiện đại" <!-- Text mô tả -->
                        android:textSize="14sp"                            <!-- Kích thước text 14sp -->
                        android:textColor="@android:color/white"           <!-- Màu text trắng -->
                        android:gravity="center"                           <!-- Căn giữa -->
                        android:layout_marginBottom="32dp"                 <!-- Margin bottom 32dp -->
                        android:alpha="0.8" />                             <!-- Alpha 0.8 -->

                    <!-- Action Buttons -->
                    <LinearLayout                                          <!-- LinearLayout chứa action buttons -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:orientation="horizontal"                   <!-- Orientation ngang -->
                        android:gravity="center">                          <!-- Căn giữa -->

                        <Button                                            <!-- Button đăng nhập -->
                            android:id="@+id/btn_dang_nhap"                <!-- ID để reference từ Java code -->
                            android:layout_width="0dp"                     <!-- Chiều rộng 0dp để sử dụng weight -->
                            android:layout_height="48dp"                   <!-- Chiều cao 48dp -->
                            android:layout_weight="1"                      <!-- Weight 1 để chia đều không gian -->
                            android:text="ĐĂNG NHẬP"                       <!-- Text button -->
                            android:textSize="14sp"                        <!-- Kích thước text 14sp -->
                            android:textStyle="bold"                       <!-- Style bold -->
                            android:textColor="@android:color/white"       <!-- Màu text trắng -->
                            android:background="@drawable/btn_dang_nhap"    <!-- Background custom drawable -->
                            android:layout_marginEnd="8dp"                 <!-- Margin end 8dp -->
                            android:elevation="4dp" />                     <!-- Elevation 4dp tạo shadow -->

                        <Button                                            <!-- Button đăng ký -->
                            android:id="@+id/btn_dang_ky"                  <!-- ID để reference từ Java code -->
                            android:layout_width="0dp"                     <!-- Chiều rộng 0dp để sử dụng weight -->
                            android:layout_height="48dp"                   <!-- Chiều cao 48dp -->
                            android:layout_weight="1"                      <!-- Weight 1 để chia đều không gian -->
                            android:text="ĐĂNG KÝ"                         <!-- Text button -->
                            android:textSize="14sp"                        <!-- Kích thước text 14sp -->
                            android:textStyle="bold"                       <!-- Style bold -->
                            android:textColor="@android:color/white"       <!-- Màu text trắng -->
                            android:background="@drawable/btn_dang_ky"      <!-- Background custom drawable -->
                            android:layout_marginStart="8dp"               <!-- Margin start 8dp -->
                            android:elevation="4dp" />                     <!-- Elevation 4dp tạo shadow -->

                    </LinearLayout>
                </LinearLayout>

            </LinearLayout>

        </RelativeLayout>
```
#### Features Grid Section:
```xml
        <!-- Features Grid -->
        <LinearLayout                                                      <!-- LinearLayout chứa lưới tính năng -->
            android:layout_width="match_parent"                            <!-- Chiều rộng full -->
            android:layout_height="wrap_content"                           <!-- Chiều cao wrap content -->
            android:orientation="vertical"                                 <!-- Orientation dọc -->
            android:padding="24dp"                                         <!-- Padding 24dp -->
            android:background="@android:color/white">                     <!-- Background màu trắng -->

            <TextView                                                      <!-- TextView tiêu đề section -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:text="TÍNH NĂNG CHÍNH"                             <!-- Text tiêu đề -->
                android:textSize="22sp"                                    <!-- Kích thước text 22sp -->
                android:textStyle="bold"                                   <!-- Style bold -->
                android:textColor="#1976D2"                                <!-- Màu text xanh -->
                android:gravity="center"                                   <!-- Căn giữa -->
                android:layout_marginBottom="24dp" />                      <!-- Margin bottom 24dp -->

            <!-- Feature Row 1 -->
            <LinearLayout                                                  <!-- LinearLayout chứa hàng tính năng đầu tiên -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:orientation="horizontal"                           <!-- Orientation ngang -->
                android:layout_marginBottom="16dp">                        <!-- Margin bottom 16dp -->

                <LinearLayout                                              <!-- Feature card quản lý nhân viên -->
                    android:layout_width="0dp"                             <!-- Chiều rộng 0dp để sử dụng weight -->
                    android:layout_height="180dp"                          <!-- Chiều cao cố định 180dp -->
                    android:layout_weight="1"                              <!-- Weight 1 -->
                    android:orientation="vertical"                         <!-- Orientation dọc -->
                    android:background="@drawable/card_feature"            <!-- Background custom drawable -->
                    android:gravity="center"                               <!-- Căn giữa nội dung -->
                    android:padding="16dp"                                 <!-- Padding 16dp -->
                    android:layout_marginEnd="8dp">                        <!-- Margin end 8dp -->

                    <ImageView                                             <!-- ImageView icon tính năng -->
                        android:layout_width="100dp"                       <!-- Chiều rộng 100dp -->
                        android:layout_height="100dp"                      <!-- Chiều cao 100dp -->
                        android:src="@drawable/quan_ly_nhan_vien"          <!-- Source image quản lý nhân viên -->
                        android:scaleType="centerCrop"                     <!-- Scale type center crop -->
                        android:layout_marginBottom="8dp" />               <!-- Margin bottom 8dp -->

                    <TextView                                              <!-- TextView label tính năng -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="Quản Lý\nNhân Viên"                  <!-- Text label với line break -->
                        android:textSize="12sp"                            <!-- Kích thước text 12sp -->
                        android:textStyle="bold"                           <!-- Style bold -->
                        android:textColor="#333"                           <!-- Màu text xám đậm -->
                        android:gravity="center" />                        <!-- Căn giữa -->

                </LinearLayout>

                <LinearLayout                                              <!-- Feature card chấm công điện tử -->
                    android:layout_width="0dp"                             <!-- Chiều rộng 0dp để sử dụng weight -->
                    android:layout_height="180dp"                          <!-- Chiều cao cố định 180dp -->
                    android:layout_weight="1"                              <!-- Weight 1 -->
                    android:orientation="vertical"                         <!-- Orientation dọc -->
                    android:background="@drawable/card_feature"            <!-- Background custom drawable -->
                    android:gravity="center"                               <!-- Căn giữa nội dung -->
                    android:padding="16dp"                                 <!-- Padding 16dp -->
                    android:layout_marginStart="8dp">                      <!-- Margin start 8dp -->

                    <ImageView                                             <!-- ImageView icon chấm công -->
                        android:layout_width="100dp"                       <!-- Chiều rộng 100dp -->
                        android:layout_height="100dp"                      <!-- Chiều cao 100dp -->
                        android:src="@drawable/cham_cong"                  <!-- Source image chấm công -->
                        android:scaleType="centerCrop"                     <!-- Scale type center crop -->
                        android:layout_marginBottom="8dp" />               <!-- Margin bottom 8dp -->

                    <TextView                                              <!-- TextView label chấm công -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="Chấm Công\nĐiện Tử"                  <!-- Text label với line break -->
                        android:textSize="12sp"                            <!-- Kích thước text 12sp -->
                        android:textStyle="bold"                           <!-- Style bold -->
                        android:textColor="#333"                           <!-- Màu text xám đậm -->
                        android:gravity="center" />                        <!-- Căn giữa -->

                </LinearLayout>

            </LinearLayout>

            <!-- Feature Row 2 -->
            <LinearLayout                                                  <!-- LinearLayout chứa hàng tính năng thứ hai -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:orientation="horizontal">                          <!-- Orientation ngang -->

                <LinearLayout                                              <!-- Feature card quản lý nghỉ phép -->
                    android:layout_width="0dp"                             <!-- Chiều rộng 0dp để sử dụng weight -->
                    android:layout_height="180dp"                          <!-- Chiều cao cố định 180dp -->
                    android:layout_weight="1"                              <!-- Weight 1 -->
                    android:orientation="vertical"                         <!-- Orientation dọc -->
                    android:background="@drawable/card_feature"            <!-- Background custom drawable -->
                    android:gravity="center"                               <!-- Căn giữa nội dung -->
                    android:padding="16dp"                                 <!-- Padding 16dp -->
                    android:layout_marginEnd="8dp">                        <!-- Margin end 8dp -->

                    <ImageView                                             <!-- ImageView icon nghỉ phép -->
                        android:layout_width="100dp"                       <!-- Chiều rộng 100dp -->
                        android:layout_height="100dp"                      <!-- Chiều cao 100dp -->
                        android:src="@drawable/quan_ly_nghi_phep"          <!-- Source image quản lý nghỉ phép -->
                        android:scaleType="centerCrop"                     <!-- Scale type center crop -->
                        android:layout_marginBottom="8dp" />               <!-- Margin bottom 8dp -->

                    <TextView                                              <!-- TextView label nghỉ phép -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="Quản Lý\nNghỉ Phép"                  <!-- Text label với line break -->
                        android:textSize="12sp"                            <!-- Kích thước text 12sp -->
                        android:textStyle="bold"                           <!-- Style bold -->
                        android:textColor="#333"                           <!-- Màu text xám đậm -->
                        android:gravity="center" />                        <!-- Căn giữa -->

                </LinearLayout>

                <LinearLayout                                              <!-- Feature card tính lương tự động -->
                    android:layout_width="0dp"                             <!-- Chiều rộng 0dp để sử dụng weight -->
                    android:layout_height="180dp"                          <!-- Chiều cao cố định 180dp -->
                    android:layout_weight="1"                              <!-- Weight 1 -->
                    android:orientation="vertical"                         <!-- Orientation dọc -->
                    android:background="@drawable/card_feature"            <!-- Background custom drawable -->
                    android:gravity="center"                               <!-- Căn giữa nội dung -->
                    android:padding="16dp"                                 <!-- Padding 16dp -->
                    android:layout_marginStart="8dp">                      <!-- Margin start 8dp -->

                    <ImageView                                             <!-- ImageView icon tính lương -->
                        android:layout_width="100dp"                       <!-- Chiều rộng 100dp -->
                        android:layout_height="100dp"                      <!-- Chiều cao 100dp -->
                        android:src="@drawable/tinh_luong"                 <!-- Source image tính lương -->
                        android:scaleType="centerCrop"                     <!-- Scale type center crop -->
                        android:layout_marginBottom="8dp" />               <!-- Margin bottom 8dp -->

                    <TextView                                              <!-- TextView label tính lương -->
                        android:layout_width="match_parent"                <!-- Chiều rộng full -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="Tính Lương\nTự Động"                 <!-- Text label với line break -->
                        android:textSize="12sp"                            <!-- Kích thước text 12sp -->
                        android:textStyle="bold"                           <!-- Style bold -->
                        android:textColor="#333"                           <!-- Màu text xám đậm -->
                        android:gravity="center" />                        <!-- Căn giữa -->

                </LinearLayout>

            </LinearLayout>

        </LinearLayout>
```

#### Statistics Section:
```xml
        <!-- Statistics Section -->
        <LinearLayout                                                      <!-- LinearLayout chứa section thống kê -->
            android:layout_width="match_parent"                            <!-- Chiều rộng full -->
            android:layout_height="wrap_content"                           <!-- Chiều cao wrap content -->
            android:orientation="vertical"                                 <!-- Orientation dọc -->
            android:background="#4CAF50"                                   <!-- Background màu xanh lá -->
            android:padding="32dp">                                        <!-- Padding 32dp -->

            <TextView                                                      <!-- TextView tiêu đề thống kê -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:text="THÀNH TÍCH ẤN TƯỢNG"                         <!-- Text tiêu đề -->
                android:textSize="20sp"                                    <!-- Kích thước text 20sp -->
                android:textStyle="bold"                                   <!-- Style bold -->
                android:textColor="@android:color/white"                   <!-- Màu text trắng -->
                android:gravity="center"                                   <!-- Căn giữa -->
                android:layout_marginBottom="24dp" />                      <!-- Margin bottom 24dp -->

            <LinearLayout                                                  <!-- LinearLayout chứa 3 cột thống kê -->
                android:layout_width="match_parent"                        <!-- Chiều rộng full -->
                android:layout_height="wrap_content"                       <!-- Chiều cao wrap content -->
                android:orientation="horizontal">                          <!-- Orientation ngang -->

                <LinearLayout                                              <!-- Cột thống kê doanh nghiệp -->
                    android:layout_width="0dp"                             <!-- Chiều rộng 0dp để sử dụng weight -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao wrap content -->
                    android:layout_weight="1"                              <!-- Weight 1 -->
                    android:orientation="vertical"                         <!-- Orientation dọc -->
                    android:gravity="center">                              <!-- Căn giữa -->

                    <TextView                                              <!-- TextView số liệu doanh nghiệp -->
                        android:layout_width="wrap_content"                <!-- Chiều rộng wrap content -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="1000+"                               <!-- Text số liệu -->
                        android:textSize="28sp"                            <!-- Kích thước text lớn 28sp -->
                        android:textStyle="bold"                           <!-- Style bold -->
                        android:textColor="@android:color/white" />        <!-- Màu text trắng -->

                    <TextView                                              <!-- TextView label doanh nghiệp -->
                        android:layout_width="wrap_content"                <!-- Chiều rộng wrap content -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="Doanh nghiệp"                        <!-- Text label -->
                        android:textSize="12sp"                            <!-- Kích thước text 12sp -->
                        android:textColor="@android:color/white"           <!-- Màu text trắng -->
                        android:alpha="0.9" />                             <!-- Alpha 0.9 -->

                </LinearLayout>

                <LinearLayout                                              <!-- Cột thống kê người dùng -->
                    android:layout_width="0dp"                             <!-- Chiều rộng 0dp để sử dụng weight -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao wrap content -->
                    android:layout_weight="1"                              <!-- Weight 1 -->
                    android:orientation="vertical"                         <!-- Orientation dọc -->
                    android:gravity="center">                              <!-- Căn giữa -->

                    <TextView                                              <!-- TextView số liệu người dùng -->
                        android:layout_width="wrap_content"                <!-- Chiều rộng wrap content -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="100K+"                               <!-- Text số liệu -->
                        android:textSize="28sp"                            <!-- Kích thước text lớn 28sp -->
                        android:textStyle="bold"                           <!-- Style bold -->
                        android:textColor="@android:color/white" />        <!-- Màu text trắng -->

                    <TextView                                              <!-- TextView label người dùng -->
                        android:layout_width="wrap_content"                <!-- Chiều rộng wrap content -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="Người dùng"                          <!-- Text label -->
                        android:textSize="12sp"                            <!-- Kích thước text 12sp -->
                        android:textColor="@android:color/white"           <!-- Màu text trắng -->
                        android:alpha="0.9" />                             <!-- Alpha 0.9 -->

                </LinearLayout>

                <LinearLayout                                              <!-- Cột thống kê hài lòng -->
                    android:layout_width="0dp"                             <!-- Chiều rộng 0dp để sử dụng weight -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao wrap content -->
                    android:layout_weight="1"                              <!-- Weight 1 -->
                    android:orientation="vertical"                         <!-- Orientation dọc -->
                    android:gravity="center">                              <!-- Căn giữa -->

                    <TextView                                              <!-- TextView tỷ lệ hài lòng -->
                        android:layout_width="wrap_content"                <!-- Chiều rộng wrap content -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="83,6%"                               <!-- Text tỷ lệ -->
                        android:textSize="28sp"                            <!-- Kích thước text lớn 28sp -->
                        android:textStyle="bold"                           <!-- Style bold -->
                        android:textColor="@android:color/white" />        <!-- Màu text trắng -->

                    <TextView                                              <!-- TextView label hài lòng -->
                        android:layout_width="wrap_content"                <!-- Chiều rộng wrap content -->
                        android:layout_height="wrap_content"               <!-- Chiều cao wrap content -->
                        android:text="Hài lòng"                            <!-- Text label -->
                        android:textSize="12sp"                            <!-- Kích thước text 12sp -->
                        android:textColor="@android:color/white"           <!-- Màu text trắng -->
                        android:alpha="0.9" />                             <!-- Alpha 0.9 -->

                </LinearLayout>

            </LinearLayout>

        </LinearLayout>
```