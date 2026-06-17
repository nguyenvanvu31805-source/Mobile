# HƯỚNG DẪN CHI TIẾT: CHỨC NĂNG QUẢN LÝ NHÂN VIÊN

## 📋 TỔNG QUAN

Chức năng Quản lý Nhân viên là một trong những module quan trọng nhất của hệ thống QLNS, cho phép Admin/HR/Manager thực hiện các thao tác CRUD (Create, Read, Update, Delete) với thông tin nhân viên.

## 🏗️ KIẾN TRÚC HỆ THỐNG

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/
├── QuanLyNhanVienActivity.java          # Activity chính - Danh sách nhân viên
├── ThemNhanVienActivity.java            # Activity thêm/sửa nhân viên  
├── NhanVienAdapter.java                 # Adapter hiển thị danh sách
├── NhanVien.java                        # Model class nhân viên
└── database/DatabaseHelper.java         # Xử lý database

app/src/main/res/layout/
├── activity_quan_ly_nhan_vien.xml       # Layout danh sách nhân viên
├── activity_them_nhan_vien.xml          # Layout form thêm/sửa
└── item_nhan_vien.xml                   # Layout item trong ListView
```

## 📊 NGHIỆP VỤ QUẢN LÝ NHÂN VIÊN

### 1. Quy trình nghiệp vụ:
- **Xem danh sách**: Hiển thị tất cả nhân viên với thông tin cơ bản
- **Tìm kiếm**: Tìm theo tên hoặc mã nhân viên
- **Thêm mới**: Tạo nhân viên với mã tự động tăng
- **Cập nhật**: Chỉnh sửa thông tin nhân viên hiện có
- **Xóa**: Xóa nhân viên khỏi hệ thống (có xác nhận)
- **Xuất báo cáo**: Tạo file PDF danh sách nhân viên

### 2. Phân quyền:
- **Admin**: Full quyền tất cả chức năng
- **HR**: Full quyền tất cả chức năng  
- **Manager**: Full quyền tất cả chức năng
- **Employee**: Không có quyền truy cập

---

## 📱 CHI TIẾT CÁC FILE
## 1️⃣ MODEL CLASS - NhanVien.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/NhanVien.java`

### Mục đích:
Model class đại diện cho đối tượng Nhân viên trong hệ thống, chứa các thuộc tính cơ bản và phương thức getter.

### Chi tiết code:

```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

public class NhanVien {                                                    // Khai báo class public NhanVien
    // Khai báo các thuộc tính private để đảm bảo tính đóng gói
    private String maNhanVien;                                             // Thuộc tính mã nhân viên (Primary Key)
    private String hoTen;                                                  // Thuộc tính họ và tên đầy đủ
    private String gioiTinh;                                               // Thuộc tính giới tính (Nam/Nữ)
    private String tenChucVu;                                              // Thuộc tính tên chức vụ (từ bảng ChucVu)
    private String tenPhongBan;                                            // Thuộc tính tên phòng ban (từ bảng PhongBan)
    private String hinhAnh;                                                // Thuộc tính URI đường dẫn ảnh đại diện

    // Constructor khởi tạo đối tượng với đầy đủ thông tin
    public NhanVien(String maNhanVien, String hoTen, String gioiTinh,      // Khai báo constructor với 6 tham số
                    String tenChucVu, String tenPhongBan, String hinhAnh) {
        this.maNhanVien = maNhanVien;                                      // Gán giá trị tham số cho thuộc tính maNhanVien
        this.hoTen = hoTen;                                                // Gán giá trị tham số cho thuộc tính hoTen
        this.gioiTinh = gioiTinh;                                          // Gán giá trị tham số cho thuộc tính gioiTinh
        this.tenChucVu = tenChucVu;                                        // Gán giá trị tham số cho thuộc tính tenChucVu
        this.tenPhongBan = tenPhongBan;                                    // Gán giá trị tham số cho thuộc tính tenPhongBan
        this.hinhAnh = hinhAnh;                                            // Gán giá trị tham số cho thuộc tính hinhAnh
    }

    // Các phương thức getter để truy xuất dữ liệu (không có setter để bảo vệ dữ liệu)
    public String getMaNhanVien() { return maNhanVien; }                   // Getter trả về mã nhân viên
    public String getHoTen() { return hoTen; }                             // Getter trả về họ tên
    public String getGioiTinh() { return gioiTinh; }                       // Getter trả về giới tính
    public String getTenChucVu() { return tenChucVu; }                     // Getter trả về tên chức vụ
    public String getTenPhongBan() { return tenPhongBan; }                 // Getter trả về tên phòng ban
    public String getHinhAnh() { return hinhAnh; }                         // Getter trả về đường dẫn hình ảnh
}
```

### Đặc điểm:
- **Immutable**: Không có setter, đảm bảo dữ liệu không bị thay đổi sau khi tạo
- **Encapsulation**: Tất cả thuộc tính đều private
- **Simple**: Chỉ chứa dữ liệu cần thiết cho hiển thị danh sách

---
## 2️⃣ ADAPTER CLASS - NhanVienAdapter.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/NhanVienAdapter.java`

### Mục đích:
Adapter kết nối dữ liệu danh sách nhân viên với ListView, xử lý hiển thị từng item và các sự kiện tương tác.

### Chi tiết code:

```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.content.Context;                                            // Import class Context để truy cập resources
import android.net.Uri;                                                    // Import class Uri để xử lý đường dẫn ảnh
import android.view.LayoutInflater;                                        // Import LayoutInflater để inflate layout XML
import android.view.View;                                                  // Import class View cơ bản
import android.view.ViewGroup;                                             // Import ViewGroup cho container views
import android.widget.BaseAdapter;                                         // Import BaseAdapter để extend
import android.widget.ImageButton;                                         // Import ImageButton widget
import android.widget.ImageView;                                           // Import ImageView widget
import android.widget.TextView;                                            // Import TextView widget
import java.util.ArrayList;                                                // Import ArrayList collection
import java.util.List;                                                     // Import List interface
import java.util.Locale;                                                   // Import Locale để xử lý ngôn ngữ

public class NhanVienAdapter extends BaseAdapter {                         // Khai báo class extend BaseAdapter
    // Khai báo các thuộc tính cần thiết
    private Context context;                                               // Context để truy cập resources và services
    private List<NhanVien> listNhanVien;                                   // Danh sách hiện tại (có thể bị filter)
    private List<NhanVien> listNhanVienFull;                               // Danh sách đầy đủ (backup cho search)
    private OnNhanVienActionListener listener;                             // Interface callback cho các sự kiện

    // Interface định nghĩa các callback cho sự kiện người dùng
    public interface OnNhanVienActionListener {                            // Khai báo interface public
        void onDelete(NhanVien nhanVien);                                  // Method callback khi nhấn nút xóa
        void onEdit(NhanVien nhanVien);                                    // Method callback khi nhấn vào item để sửa
    }

    // Constructor khởi tạo adapter
    public NhanVienAdapter(Context context, List<NhanVien> listNhanVien,   // Khai báo constructor với 3 tham số
                          OnNhanVienActionListener listener) {
        this.context = context;                                            // Gán context từ tham số
        this.listNhanVien = listNhanVien;                                  // Gán danh sách từ tham số
        this.listNhanVienFull = new ArrayList<>(listNhanVien);             // Tạo bản sao để backup cho search
        this.listener = listener;                                          // Gán listener từ tham số
    }

    @Override                                                              // Annotation override method từ BaseAdapter
    public int getCount() {                                                // Method trả về số lượng item
        return listNhanVien.size();                                        // Trả về kích thước danh sách
    }

    @Override                                                              // Annotation override method từ BaseAdapter
    public Object getItem(int position) {                                  // Method trả về object tại vị trí
        return listNhanVien.get(position);                                 // Trả về nhân viên tại vị trí position
    }

    @Override                                                              // Annotation override method từ BaseAdapter
    public long getItemId(int position) {                                  // Method trả về ID của item
        return position;                                                   // Trả về position làm ID
    }
```

### Phương thức getView() - Quan trọng nhất:

```java
    @Override                                                              // Annotation override method từ BaseAdapter
    public View getView(int position, View convertView, ViewGroup parent) { // Method tạo/cập nhật view cho từng item
        // Kiểm tra convertView để tái sử dụng (ViewHolder pattern cơ bản)
        if (convertView == null) {                                         // Nếu convertView null (chưa có view)
            // Nếu chưa có view, inflate layout từ XML
            convertView = LayoutInflater.from(context).inflate(R.layout.item_nhan_vien, parent, false); // Tạo view từ layout XML
        }

        // Lấy đối tượng nhân viên tại vị trí hiện tại
        NhanVien nv = listNhanVien.get(position);                          // Lấy nhân viên từ danh sách theo position

        // Ánh xạ các view component từ layout
        ImageView ivAvatar = convertView.findViewById(R.id.iv_avatar);      // Tìm ImageView avatar theo ID
        TextView tvTen = convertView.findViewById(R.id.tv_ten_nv);          // Tìm TextView tên nhân viên theo ID
        TextView tvMa = convertView.findViewById(R.id.tv_ma_nv);            // Tìm TextView mã nhân viên theo ID
        TextView tvChucVuPB = convertView.findViewById(R.id.tv_chuc_vu_phong_ban); // Tìm TextView chức vụ-phòng ban theo ID
        ImageButton btnDelete = convertView.findViewById(R.id.btn_delete_nv); // Tìm ImageButton xóa theo ID

        // Gán dữ liệu vào các view
        tvTen.setText(nv.getHoTen());                                      // Set text tên nhân viên
        tvMa.setText("Mã: " + nv.getMaNhanVien());                         // Set text mã nhân viên với prefix "Mã: "
        tvChucVuPB.setText(nv.getTenChucVu() + " - " + nv.getTenPhongBan()); // Set text chức vụ và phòng ban

        // Xử lý hiển thị ảnh đại diện
        if (nv.getHinhAnh() != null && !nv.getHinhAnh().isEmpty()) {       // Kiểm tra nếu có đường dẫn ảnh
            try {                                                          // Bắt đầu try-catch để xử lý lỗi
                // Thử load ảnh từ URI
                ivAvatar.setImageURI(Uri.parse(nv.getHinhAnh()));          // Parse URI và set ảnh cho ImageView
                ivAvatar.setPadding(0, 0, 0, 0);                          // Bỏ padding khi có ảnh thật
                ivAvatar.setScaleType(ImageView.ScaleType.CENTER_CROP);    // Set scale type để cắt ảnh vừa khung
            } catch (Exception e) {                                        // Bắt exception nếu có lỗi
                // Nếu lỗi, sử dụng ảnh mặc định
                ivAvatar.setImageResource(R.drawable.ic_person);           // Set ảnh mặc định từ drawable
                ivAvatar.setPadding(8, 8, 8, 8);                          // Set padding cho icon mặc định
            }
        } else {                                                           // Nếu không có đường dẫn ảnh
            // Không có ảnh, sử dụng icon mặc định
            ivAvatar.setImageResource(R.drawable.ic_person);               // Set ảnh mặc định từ drawable
            ivAvatar.setPadding(8, 8, 8, 8);                              // Set padding cho icon mặc định
        }

        // Xử lý sự kiện nhấn nút xóa
        btnDelete.setOnClickListener(v -> {                                // Set OnClickListener cho nút xóa (lambda expression)
            if (listener != null) listener.onDelete(nv);                  // Gọi callback onDelete nếu listener không null
        });

        // Xử lý sự kiện nhấn vào item để chỉnh sửa
        convertView.setOnClickListener(v -> {                              // Set OnClickListener cho toàn bộ item (lambda expression)
            if (listener != null) listener.onEdit(nv);                    // Gọi callback onEdit nếu listener không null
        });

        return convertView;                                                // Trả về view đã được cấu hình
    }
```

### Phương thức filter() - Tìm kiếm:

```java
    public void filter(String charText) {                                  // Method public để filter danh sách theo từ khóa
        charText = charText.toLowerCase(Locale.getDefault());              // Chuyển từ khóa về chữ thường theo locale mặc định
        listNhanVien.clear();                                              // Xóa tất cả item trong danh sách hiện tại
        
        if (charText.length() == 0) {                                      // Kiểm tra nếu từ khóa rỗng
            // Nếu không có từ khóa, hiển thị tất cả
            listNhanVien.addAll(listNhanVienFull);                         // Thêm tất cả item từ danh sách backup
        } else {                                                           // Nếu có từ khóa
            // Lọc theo từ khóa
            for (NhanVien nv : listNhanVienFull) {                         // Duyệt qua từng nhân viên trong danh sách backup
                // Tìm trong tên hoặc mã nhân viên
                if (nv.getHoTen().toLowerCase(Locale.getDefault()).contains(charText) || // Kiểm tra tên có chứa từ khóa
                    nv.getMaNhanVien().toLowerCase(Locale.getDefault()).contains(charText)) { // Kiểm tra mã có chứa từ khóa
                    listNhanVien.add(nv);                                  // Thêm nhân viên vào danh sách kết quả
                }
            }
        }
        notifyDataSetChanged();                                            // Thông báo ListView cập nhật giao diện
    }
```

### Đặc điểm của Adapter:
- **ViewHolder Pattern**: Tái sử dụng convertView để tối ưu hiệu suất
- **Callback Pattern**: Sử dụng interface để giao tiếp với Activity
- **Search Support**: Hỗ trợ tìm kiếm real-time
- **Image Handling**: Xử lý ảnh với fallback an toàn

---
## 3️⃣ ACTIVITY CHÍNH - QuanLyNhanVienActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/QuanLyNhanVienActivity.java`

### Mục đích:
Activity chính quản lý danh sách nhân viên, xử lý hiển thị, tìm kiếm, xóa và xuất báo cáo.

### Chi tiết code:

#### Khai báo và khởi tạo:

```java
public class QuanLyNhanVienActivity extends AppCompatActivity {           // Khai báo class extend AppCompatActivity
    // Khai báo các view components
    private ListView listView;                                             // ListView hiển thị danh sách nhân viên
    private TextView tvEmpty;                                              // TextView hiển thị thông báo khi không có dữ liệu
    private SearchView searchView;                                         // SearchView để tìm kiếm nhân viên
    private android.widget.ImageButton btnExportPdf;                       // ImageButton để xuất file PDF
    private FloatingActionButton fabAdd;                                   // FloatingActionButton để thêm nhân viên mới
    
    // Khai báo các đối tượng xử lý dữ liệu
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database SQLite
    private NhanVienAdapter adapter;                                       // Adapter kết nối dữ liệu với ListView
    private List<NhanVien> listNhanVien;                                   // List chứa danh sách nhân viên

    // ActivityResultLauncher để nhận kết quả từ Activity thêm/sửa
    private final ActivityResultLauncher<Intent> startForResult = registerForActivityResult( // Đăng ký ActivityResultLauncher
            new ActivityResultContracts.StartActivityForResult(),          // Sử dụng contract StartActivityForResult
            result -> {                                                    // Lambda expression xử lý kết quả
                if (result.getResultCode() == RESULT_OK) {                 // Kiểm tra nếu kết quả thành công
                    taiDanhSachNhanVien();                                 // Reload danh sách khi có thay đổi
                }
            }
    );
```

#### Phương thức onCreate():

```java
    @Override                                                              // Annotation override method từ AppCompatActivity
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi method onCreate của class cha
        setContentView(R.layout.activity_quan_ly_nhan_vien);               // Set layout XML cho Activity
        
        dbHelper = new DatabaseHelper(this);                               // Khởi tạo DatabaseHelper với context hiện tại
        khoiTaoViews();                                                    // Gọi method khởi tạo các view components
        taiDanhSachNhanVien();                                             // Gọi method load danh sách nhân viên từ database
        thietLapTimKiem();                                                 // Gọi method thiết lập chức năng tìm kiếm
    }
```

#### Khởi tạo Views:

```java
    private void khoiTaoViews() {
        // Ánh xạ các view từ layout XML
        listView = findViewById(R.id.lv_nhan_vien);
        tvEmpty = findViewById(R.id.tv_empty);
        searchView = findViewById(R.id.search_view);
        btnExportPdf = findViewById(R.id.btn_export_pdf);
        fabAdd = findViewById(R.id.fab_add_nhan_vien);

        // Thiết lập sự kiện cho nút xuất PDF
        btnExportPdf.setOnClickListener(v -> exportToPdf());
        
        // Thiết lập sự kiện cho nút thêm nhân viên
        fabAdd.setOnClickListener(v -> {
            Intent intent = new Intent(this, ThemNhanVienActivity.class);
            startForResult.launch(intent); // Sử dụng ActivityResultLauncher
        });
    }
```

#### Load và hiển thị danh sách:

```java
    private void taiDanhSachNhanVien() {
        Cursor cursor = dbHelper.getAllEmployees(); // Lấy dữ liệu từ database
        hienThiDanhSach(cursor);                    // Hiển thị lên giao diện
    }

    private void hienThiDanhSach(Cursor cursor) {
        // Khởi tạo hoặc clear danh sách
        if (listNhanVien == null) {
            listNhanVien = new ArrayList<>();
        } else {
            listNhanVien.clear();
        }

        // Đọc dữ liệu từ Cursor và chuyển thành đối tượng NhanVien
        if (cursor != null && cursor.moveToFirst()) {
            do {
                // Lấy dữ liệu từ các cột
                String maNV = cursor.getString(cursor.getColumnIndexOrThrow("MaNhanVien"));
                String hoTen = cursor.getString(cursor.getColumnIndexOrThrow("HoTen"));
                String gioiTinh = cursor.getString(cursor.getColumnIndexOrThrow("GioiTinh"));
                String tenChucVu = cursor.getString(cursor.getColumnIndexOrThrow("TenChucVu"));
                String tenPhongBan = cursor.getString(cursor.getColumnIndexOrThrow("TenPhongBan"));
                String hinhAnh = cursor.getString(cursor.getColumnIndexOrThrow("HinhAnh"));
                
                // Tạo đối tượng NhanVien và thêm vào danh sách
                listNhanVien.add(new NhanVien(maNV, hoTen, gioiTinh, tenChucVu, tenPhongBan, hinhAnh));
            } while (cursor.moveToNext());
            cursor.close(); // Đóng cursor để giải phóng bộ nhớ
        }

        // Xử lý hiển thị khi danh sách rỗng
        if (listNhanVien.isEmpty()) {
            tvEmpty.setVisibility(View.VISIBLE);    // Hiển thị thông báo rỗng
            listView.setVisibility(View.GONE);      // Ẩn ListView
        } else {
            tvEmpty.setVisibility(View.GONE);       // Ẩn thông báo rỗng
            listView.setVisibility(View.VISIBLE);   // Hiển thị ListView
        }

        // Khởi tạo hoặc cập nhật Adapter
        if (adapter == null) {
            // Tạo adapter mới với callback xử lý sự kiện
            adapter = new NhanVienAdapter(this, listNhanVien, new NhanVienAdapter.OnNhanVienActionListener() {
                @Override
                public void onDelete(NhanVien nhanVien) {
                    xacNhanXoaNhanVien(nhanVien); // Xử lý sự kiện xóa
                }

                @Override
                public void onEdit(NhanVien nhanVien) {
                    // Xử lý sự kiện chỉnh sửa
                    Intent intent = new Intent(QuanLyNhanVienActivity.this, ThemNhanVienActivity.class);
                    intent.putExtra("ma_nv", nhanVien.getMaNhanVien()); // Truyền mã NV để edit
                    startForResult.launch(intent);
                }
            });
            listView.setAdapter(adapter); // Gán adapter cho ListView
        } else {
            adapter.notifyDataSetChanged(); // Thông báo adapter cập nhật dữ liệu
        }
    }
```

#### Xử lý xóa nhân viên:

```java
    private void xacNhanXoaNhanVien(NhanVien nv) {
        // Hiển thị dialog xác nhận xóa
        new AlertDialog.Builder(this)
                .setTitle("Xác nhận xóa")
                .setMessage("Bạn có chắc chắn muốn xóa nhân viên " + nv.getHoTen() + " (" + nv.getMaNhanVien() + ")?")
                .setPositiveButton("Xóa", (dialog, which) -> {
                    // Thực hiện xóa trong database
                    if (dbHelper.deleteEmployee(nv.getMaNhanVien())) {
                        Toast.makeText(this, "Đã xóa nhân viên", Toast.LENGTH_SHORT).show();
                        taiDanhSachNhanVien(); // Reload danh sách
                    } else {
                        Toast.makeText(this, "Lỗi khi xóa nhân viên", Toast.LENGTH_SHORT).show();
                    }
                })
                .setNegativeButton("Hủy", null) // Không làm gì khi hủy
                .show();
    }
```

#### Chức năng tìm kiếm:

```java
    private void thietLapTimKiem() {
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                thucHienTimKiem(query); // Tìm kiếm khi nhấn Enter
                return true;
            }

            @Override
            public boolean onQueryTextChange(String newText) {
                thucHienTimKiem(newText); // Tìm kiếm real-time khi gõ
                return true;
            }
        });
    }

    private void thucHienTimKiem(String keyword) {
        Cursor cursor;
        if (keyword.isEmpty()) {
            cursor = dbHelper.getAllEmployees();        // Hiển thị tất cả nếu không có từ khóa
        } else {
            cursor = dbHelper.searchEmployees(keyword); // Tìm kiếm theo từ khóa
        }
        hienThiDanhSach(cursor); // Hiển thị kết quả
    }
```

---
#### Chức năng xuất PDF:

```java
    private void exportToPdf() {
        // Kiểm tra dữ liệu
        if (listNhanVien == null || listNhanVien.isEmpty()) {
            Toast.makeText(this, "Không có dữ liệu để xuất", Toast.LENGTH_SHORT).show();
            return;
        }

        // Tạo nội dung báo cáo
        StringBuilder report = new StringBuilder();
        String dateStr = new java.text.SimpleDateFormat("dd/MM/yyyy HH:mm", java.util.Locale.getDefault()).format(new java.util.Date());
        
        // Header báo cáo
        report.append("DANH SÁCH NHÂN VIÊN\n");
        report.append("Ngày xuất: ").append(dateStr).append("\n");
        report.append("=============================================\n");
        report.append(String.format("%-8s %-20s %-15s\n", "Mã NV", "Họ tên", "Phòng ban"));
        report.append("---------------------------------------------\n");
        
        // Dữ liệu nhân viên
        for (NhanVien nv : listNhanVien) {
            String hoTen = nv.getHoTen();
            if (hoTen.length() > 18) hoTen = hoTen.substring(0, 15) + "..."; // Cắt tên dài
            
            String pb = nv.getTenPhongBan();
            if (pb.length() > 13) pb = pb.substring(0, 11) + "..."; // Cắt tên phòng ban dài

            report.append(String.format("%-8s %-20s %-15s\n", 
                nv.getMaNhanVien(), 
                hoTen, 
                pb));
        }
        
        // Footer báo cáo
        report.append("=============================================\n");
        report.append("Tổng số: ").append(listNhanVien.size()).append(" nhân viên\n");
        
        savePdfFile(report.toString()); // Lưu thành file PDF
    }

    private void savePdfFile(String content) {
        // Tạo document PDF
        android.graphics.pdf.PdfDocument document = new android.graphics.pdf.PdfDocument();
        int pageNumber = 1;
        
        // Tạo trang PDF (A4: 595x842 points)
        android.graphics.pdf.PdfDocument.PageInfo pageInfo = 
            new android.graphics.pdf.PdfDocument.PageInfo.Builder(595, 842, pageNumber).create();
        android.graphics.pdf.PdfDocument.Page page = document.startPage(pageInfo);

        // Thiết lập canvas và paint để vẽ text
        android.graphics.Canvas canvas = page.getCanvas();
        android.graphics.Paint paint = new android.graphics.Paint();
        paint.setTypeface(android.graphics.Typeface.MONOSPACE); // Font monospace để căn chỉnh
        paint.setTextSize(12);
        paint.setColor(android.graphics.Color.BLACK);

        // Vẽ từng dòng text lên PDF
        int x = 40, y = 50; // Vị trí bắt đầu
        for (String line : content.split("\n")) {
            canvas.drawText(line, x, y, paint);
            y += paint.descent() - paint.ascent(); // Tăng y để xuống dòng
            
            // Kiểm tra nếu vượt quá chiều cao trang
            if (y > 800) {
                document.finishPage(page);
                pageNumber++;
                // Tạo trang mới
                pageInfo = new android.graphics.pdf.PdfDocument.PageInfo.Builder(595, 842, pageNumber).create();
                page = document.startPage(pageInfo);
                canvas = page.getCanvas();
                y = 50; // Reset vị trí y
            }
        }
        document.finishPage(page);

        // Lưu file PDF
        String fileName = "DanhSachNhanVien_" + System.currentTimeMillis() + ".pdf";
        try {
            java.io.OutputStream fos;
            
            // Xử lý khác nhau cho Android 10+ (Scoped Storage)
            if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.Q) {
                // Android 10+: Sử dụng MediaStore
                android.content.ContentValues values = new android.content.ContentValues();
                values.put(android.provider.MediaStore.Downloads.DISPLAY_NAME, fileName);
                values.put(android.provider.MediaStore.Downloads.MIME_TYPE, "application/pdf");
                values.put(android.provider.MediaStore.Downloads.RELATIVE_PATH, android.os.Environment.DIRECTORY_DOWNLOADS);
                android.net.Uri uri = getContentResolver().insert(android.provider.MediaStore.Downloads.EXTERNAL_CONTENT_URI, values);
                fos = getContentResolver().openOutputStream(uri);
            } else {
                // Android 9 trở xuống: Sử dụng File trực tiếp
                java.io.File dir = android.os.Environment.getExternalStoragePublicDirectory(android.os.Environment.DIRECTORY_DOWNLOADS);
                if (!dir.exists()) dir.mkdirs();
                java.io.File file = new java.io.File(dir, fileName);
                fos = new java.io.FileOutputStream(file);
            }
            
            // Ghi dữ liệu và đóng file
            document.writeTo(fos);
            document.close();
            if (fos != null) fos.close();
            
            Toast.makeText(this, "Đã xuất danh sách nhân viên ra PDF thành công!", Toast.LENGTH_LONG).show();
        } catch (Exception e) {
            e.printStackTrace();
            Toast.makeText(this, "Lỗi khi xuất PDF: " + e.getMessage(), Toast.LENGTH_SHORT).show();
        }
    }
```

### Đặc điểm của Activity chính:
- **Modern Android**: Sử dụng ActivityResultLauncher thay vì startActivityForResult deprecated
- **Scoped Storage**: Hỗ trợ Android 10+ với MediaStore API
- **User Experience**: Có loading states, empty states, confirmation dialogs
- **Error Handling**: Xử lý lỗi và hiển thị thông báo phù hợp
- **Performance**: Sử dụng cursor hiệu quả, đóng cursor sau khi dùng

---
## 4️⃣ ACTIVITY THÊM/SỬA - ThemNhanVienActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/ThemNhanVienActivity.java`

### Mục đích:
Activity xử lý thêm mới và chỉnh sửa thông tin nhân viên với form đầy đủ các trường dữ liệu.

### Chi tiết code:

#### Khai báo và khởi tạo:

```java
public class ThemNhanVienActivity extends AppCompatActivity {
    private static final int PICK_IMAGE_REQUEST = 1; // Request code cho chọn ảnh

    // Khai báo các view components
    private EditText etMaNV, etHoTen, etNgaySinh, etSDT, etEmail;
    private Spinner spGioiTinh, spPhongBan, spChucVu;
    private Button btnSave;
    private TextView tvTitle;
    private ImageView ivAvatar;
    
    // Khai báo các đối tượng xử lý dữ liệu
    private DatabaseHelper dbHelper;
    private String currentMaNV = null;  // Null = thêm mới, có giá trị = chỉnh sửa
    private String imageUri = "";       // URI của ảnh được chọn

    // Danh sách để mapping giữa tên và mã (cho Spinner)
    private List<String> listMaPB = new ArrayList<>();   // Danh sách mã phòng ban
    private List<String> listTenPB = new ArrayList<>();  // Danh sách tên phòng ban
    private List<String> listMaCV = new ArrayList<>();   // Danh sách mã chức vụ
    private List<String> listTenCV = new ArrayList<>();  // Danh sách tên chức vụ
```

#### Phương thức onCreate():

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_them_nhan_vien);

        dbHelper = new DatabaseHelper(this);
        khoiTaoViews();           // Khởi tạo các view
        taiDanhMuc();            // Load dữ liệu cho Spinner
        thietLapNgaySinh();      // Thiết lập DatePicker

        // Thiết lập sự kiện chọn ảnh
        ivAvatar.setOnClickListener(v -> openGallery());

        // Kiểm tra mode: thêm mới hay chỉnh sửa
        if (getIntent().hasExtra("ma_nv")) {
            // Mode chỉnh sửa
            currentMaNV = getIntent().getStringExtra("ma_nv");
            tvTitle.setText("CẬP NHẬT NHÂN VIÊN");
            etMaNV.setText(currentMaNV);
            etMaNV.setEnabled(false);           // Không cho sửa mã NV
            dienThongTinNhanVien(currentMaNV);  // Load thông tin hiện tại
        } else {
            // Mode thêm mới
            etMaNV.setText(dbHelper.getNextEmployeeCode()); // Tự động tạo mã NV
            etMaNV.setEnabled(false);                       // Không cho sửa mã NV
        }

        btnSave.setOnClickListener(v -> saveNhanVien()); // Thiết lập sự kiện lưu
    }
```

#### Khởi tạo Views:

```java
    private void khoiTaoViews() {
        // Ánh xạ các view từ layout XML
        tvTitle = findViewById(R.id.tv_title);
        ivAvatar = findViewById(R.id.iv_avatar_setup);
        etMaNV = findViewById(R.id.et_ma_nv);
        etHoTen = findViewById(R.id.et_ho_ten);
        etNgaySinh = findViewById(R.id.et_ngay_sinh);
        etSDT = findViewById(R.id.et_sdt);
        etEmail = findViewById(R.id.et_email);
        spGioiTinh = findViewById(R.id.sp_gioi_tinh);
        spPhongBan = findViewById(R.id.sp_phong_ban);
        spChucVu = findViewById(R.id.sp_chuc_vu);
        btnSave = findViewById(R.id.btn_save);
    }
```

#### Chọn ảnh từ Gallery:

```java
    private void openGallery() {
        // Sử dụng ACTION_OPEN_DOCUMENT để có quyền truy cập lâu dài
        Intent intent = new Intent(Intent.ACTION_OPEN_DOCUMENT);
        intent.addCategory(Intent.CATEGORY_OPENABLE);
        intent.setType("image/*"); // Chỉ chọn file ảnh
        startActivityForResult(intent, PICK_IMAGE_REQUEST);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        
        if (requestCode == PICK_IMAGE_REQUEST && resultCode == RESULT_OK && 
            data != null && data.getData() != null) {
            
            Uri uri = data.getData();
            
            // Xin quyền truy cập lâu dài cho URI này (quan trọng!)
            getContentResolver().takePersistableUriPermission(uri, Intent.FLAG_GRANT_READ_URI_PERMISSION);
            
            imageUri = uri.toString();              // Lưu URI dạng string
            ivAvatar.setImageURI(uri);              // Hiển thị ảnh
            ivAvatar.setPadding(0, 0, 0, 0);       // Bỏ padding
            ivAvatar.setScaleType(ImageView.ScaleType.CENTER_CROP); // Cắt ảnh vừa khung
        }
    }
```

#### Load dữ liệu cho Spinner:

```java
    private void taiDanhMuc() {
        // Load danh sách phòng ban
        Cursor cursorPB = dbHelper.getAllDepartments();
        if (cursorPB != null && cursorPB.moveToFirst()) {
            do {
                // Lưu cả mã và tên để mapping
                listMaPB.add(cursorPB.getString(cursorPB.getColumnIndexOrThrow("MaPhongBan")));
                listTenPB.add(cursorPB.getString(cursorPB.getColumnIndexOrThrow("TenPhongBan")));
            } while (cursorPB.moveToNext());
            cursorPB.close();
        }
        
        // Tạo adapter cho Spinner phòng ban (hiển thị tên, lưu index để lấy mã)
        ArrayAdapter<String> adapterPB = new ArrayAdapter<>(this, android.R.layout.simple_spinner_item, listTenPB);
        adapterPB.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        spPhongBan.setAdapter(adapterPB);

        // Load danh sách chức vụ (tương tự phòng ban)
        Cursor cursorCV = dbHelper.getAllPositions();
        if (cursorCV != null && cursorCV.moveToFirst()) {
            do {
                listMaCV.add(cursorCV.getString(cursorCV.getColumnIndexOrThrow("MaChucVu")));
                listTenCV.add(cursorCV.getString(cursorCV.getColumnIndexOrThrow("TenChucVu")));
            } while (cursorCV.moveToNext());
            cursorCV.close();
        }
        
        ArrayAdapter<String> adapterCV = new ArrayAdapter<>(this, android.R.layout.simple_spinner_item, listTenCV);
        adapterCV.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        spChucVu.setAdapter(adapterCV);
    }
```

#### Thiết lập DatePicker:

```java
    private void thietLapNgaySinh() {
        etNgaySinh.setOnClickListener(v -> {
            Calendar calendar = Calendar.getInstance();
            
            // Tạo DatePickerDialog
            new DatePickerDialog(this, (view, year, month, day) -> {
                // Format: yyyy-mm-dd (chuẩn database)
                etNgaySinh.setText(String.format("%d-%02d-%02d", year, month + 1, day));
            }, 
            calendar.get(Calendar.YEAR),    // Năm hiện tại
            calendar.get(Calendar.MONTH),   // Tháng hiện tại
            calendar.get(Calendar.DAY_OF_MONTH) // Ngày hiện tại
            ).show();
        });
    }
```

#### Load thông tin nhân viên (mode chỉnh sửa):

```java
    private void dienThongTinNhanVien(String maNV) {
        Cursor cursor = dbHelper.getAllEmployees();
        if (cursor != null && cursor.moveToFirst()) {
            do {
                // Tìm nhân viên theo mã
                if (cursor.getString(cursor.getColumnIndexOrThrow("MaNhanVien")).equals(maNV)) {
                    // Điền thông tin vào các EditText
                    etHoTen.setText(cursor.getString(cursor.getColumnIndexOrThrow("HoTen")));
                    etNgaySinh.setText(cursor.getString(cursor.getColumnIndexOrThrow("NgaySinh")));
                    etSDT.setText(cursor.getString(cursor.getColumnIndexOrThrow("SoDienThoai")));
                    etEmail.setText(cursor.getString(cursor.getColumnIndexOrThrow("Email")));
                    
                    // Load ảnh đại diện
                    imageUri = cursor.getString(cursor.getColumnIndexOrThrow("HinhAnh"));
                    if (imageUri != null && !imageUri.isEmpty()) {
                        ivAvatar.setImageURI(Uri.parse(imageUri));
                        ivAvatar.setPadding(0, 0, 0, 0);
                        ivAvatar.setScaleType(ImageView.ScaleType.CENTER_CROP);
                    }

                    // Set giới tính trong Spinner
                    String gt = cursor.getString(cursor.getColumnIndexOrThrow("GioiTinh"));
                    ArrayAdapter adapterGT = (ArrayAdapter) spGioiTinh.getAdapter();
                    if (adapterGT != null) spGioiTinh.setSelection(adapterGT.getPosition(gt));

                    // Set phòng ban trong Spinner (dựa vào mã phòng ban)
                    String maPB = cursor.getString(cursor.getColumnIndexOrThrow("MaPhongBan"));
                    spPhongBan.setSelection(listMaPB.indexOf(maPB));

                    // Set chức vụ trong Spinner (dựa vào mã chức vụ)
                    String maCV = cursor.getString(cursor.getColumnIndexOrThrow("MaChucVu"));
                    spChucVu.setSelection(listMaCV.indexOf(maCV));
                    
                    break; // Đã tìm thấy, thoát khỏi vòng lặp
                }
            } while (cursor.moveToNext());
            cursor.close();
        }
    }
```

---
#### Lưu thông tin nhân viên:

```java
    private void saveNhanVien() {
        // Lấy dữ liệu từ các view
        String maNV = etMaNV.getText().toString().trim();
        String hoTen = etHoTen.getText().toString().trim();
        String ngaySinh = etNgaySinh.getText().toString().trim();
        String gioiTinh = spGioiTinh.getSelectedItem().toString();
        String sdt = etSDT.getText().toString().trim();
        String email = etEmail.getText().toString().trim();
        
        // Lấy mã phòng ban và chức vụ từ Spinner (dựa vào position)
        String maPB = listMaPB.get(spPhongBan.getSelectedItemPosition());
        String maCV = listMaCV.get(spChucVu.getSelectedItemPosition());

        // Validation - Kiểm tra họ tên
        if (hoTen.isEmpty()) {
            etHoTen.setError("Vui lòng nhập họ tên");
            etHoTen.requestFocus(); // Focus vào field lỗi
            return;
        }

        // Validation - Kiểm tra ngày sinh
        if (ngaySinh.isEmpty()) {
            etNgaySinh.setError("Vui lòng chọn ngày sinh");
            etNgaySinh.requestFocus();
            return;
        }

        // Validation - Kiểm tra số điện thoại (phải đúng 10 số)
        if (sdt.length() != 10) {
            etSDT.setError("Số điện thoại phải có đúng 10 chữ số");
            etSDT.requestFocus();
            return;
        }

        // Validation - Kiểm tra định dạng email
        if (!android.util.Patterns.EMAIL_ADDRESS.matcher(email).matches()) {
            etEmail.setError("Định dạng Email không hợp lệ");
            etEmail.requestFocus();
            return;
        }

        // Thực hiện lưu dữ liệu
        boolean success;
        if (currentMaNV == null) {
            // Mode thêm mới
            success = dbHelper.addEmployee(maNV, hoTen, ngaySinh, gioiTinh, sdt, email, maPB, maCV, imageUri);
        } else {
            // Mode chỉnh sửa
            success = dbHelper.updateEmployee(maNV, hoTen, ngaySinh, gioiTinh, sdt, email, maPB, maCV, imageUri);
        }

        // Xử lý kết quả
        if (success) {
            Toast.makeText(this, "Lưu thành công", Toast.LENGTH_SHORT).show();
            setResult(RESULT_OK); // Báo cho Activity gọi biết có thay đổi
            finish();             // Đóng Activity
        } else {
            Toast.makeText(this, "Lưu thất bại", Toast.LENGTH_SHORT).show();
        }
    }
```

### Đặc điểm của Activity thêm/sửa:
- **Dual Mode**: Hỗ trợ cả thêm mới và chỉnh sửa trong cùng một Activity
- **Auto-generated ID**: Tự động tạo mã nhân viên cho record mới
- **Image Handling**: Sử dụng persistable URI permissions để lưu ảnh lâu dài
- **Comprehensive Validation**: Kiểm tra đầy đủ các trường dữ liệu
- **User-friendly**: DatePicker, Spinner với dữ liệu thực tế từ database

---

## 5️⃣ LAYOUT FILES - Giao diện người dùng

### 5.1 Layout danh sách - activity_quan_ly_nhan_vien.xml

**Đường dẫn**: `app/src/main/res/layout/activity_quan_ly_nhan_vien.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding -->
<androidx.coordinatorlayout.widget.CoordinatorLayout                      <!-- Container chính hỗ trợ Material Design behaviors -->
    xmlns:android="http://schemas.android.com/apk/res/android"             <!-- Namespace Android -->
    xmlns:app="http://schemas.android.com/apk/res-auto"                    <!-- Namespace cho custom attributes -->
    xmlns:tools="http://schemas.android.com/tools"                         <!-- Namespace cho Android Studio tools -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent -->
    android:layout_height="match_parent"                                   <!-- Chiều cao bằng parent -->
    android:background="#f8f9fa"                                           <!-- Màu nền xám nhạt -->
    tools:context=".QuanLyNhanVienActivity">                               <!-- Liên kết với Activity (chỉ dùng trong design time) -->

    <!-- AppBarLayout chứa header và search -->
    <com.google.android.material.appbar.AppBarLayout                       <!-- Container cho app bar với scroll behavior -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ nội dung -->
        android:background="@android:color/transparent"                    <!-- Nền trong suốt -->
        android:elevation="0dp">                                           <!-- Không có shadow -->

        <LinearLayout                                                      <!-- Container con sắp xếp theo chiều dọc -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ nội dung -->
            android:orientation="vertical"                                 <!-- Sắp xếp các view con theo chiều dọc -->
            android:background="@drawable/gradient_header"                 <!-- Nền gradient từ drawable -->
            android:padding="16dp">                                        <!-- Padding 16dp cho tất cả các cạnh -->

            <!-- Header với title và nút export -->
            <RelativeLayout                                                <!-- Container cho phép định vị tương đối -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ nội dung -->
                android:layout_marginBottom="16dp">                        <!-- Margin bottom 16dp -->
                
                <!-- Title ở giữa -->
                <TextView                                                  <!-- Text hiển thị tiêu đề -->
                    android:layout_width="wrap_content"                    <!-- Chiều rộng vừa đủ text -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ text -->
                    android:text="QUẢN LÝ NHÂN VIÊN"                       <!-- Nội dung text -->
                    android:textSize="22sp"                                <!-- Kích thước font 22sp -->
                    android:textStyle="bold"                               <!-- Kiểu chữ đậm -->
                    android:textColor="@android:color/white"               <!-- Màu chữ trắng -->
                    android:layout_centerInParent="true" />                <!-- Căn giữa trong RelativeLayout -->
                
                <!-- Nút export PDF ở bên phải -->
                <ImageButton                                               <!-- Button hiển thị icon -->
                    android:id="@+id/btn_export_pdf"                      <!-- Tạo ID để truy cập từ Java -->
                    android:layout_width="40dp"                            <!-- Chiều rộng 40dp -->
                    android:layout_height="40dp"                           <!-- Chiều cao 40dp -->
                    android:layout_alignParentEnd="true"                   <!-- Căn về phía cuối (phải) của parent -->
                    android:layout_centerVertical="true"                   <!-- Căn giữa theo chiều dọc -->
                    android:src="@android:drawable/ic_menu_save"           <!-- Icon save từ Android system -->
                    android:background="?attr/selectableItemBackgroundBorderless" <!-- Hiệu ứng ripple khi nhấn -->
                    app:tint="@android:color/white"                        <!-- Tô màu icon thành trắng -->
                    android:contentDescription="Xuất PDF" />               <!-- Mô tả cho accessibility -->
            </RelativeLayout>

            <!-- SearchView để tìm kiếm -->
            <androidx.appcompat.widget.SearchView                          <!-- Widget tìm kiếm Material Design -->
                android:id="@+id/search_view"                              <!-- Tạo ID để truy cập từ Java -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ nội dung -->
                android:background="@drawable/card_feature"                <!-- Nền từ drawable -->
                android:backgroundTint="@android:color/white"              <!-- Tô màu nền thành trắng -->
                app:queryHint="Tìm kiếm nhân viên..."                     <!-- Placeholder text -->
                app:iconifiedByDefault="false" />                         <!-- Luôn hiển thị expanded -->

        </LinearLayout>
    </com.google.android.material.appbar.AppBarLayout>

    <!-- Content area -->
    <LinearLayout                                                          <!-- Container chính cho nội dung -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="match_parent"                               <!-- Chiều cao bằng parent -->
        android:orientation="vertical"                                     <!-- Sắp xếp theo chiều dọc -->
        app:layout_behavior="@string/appbar_scrolling_view_behavior">      <!-- Behavior để scroll cùng AppBar -->

        <!-- TextView hiển thị khi không có dữ liệu -->
        <TextView                                                          <!-- Text thông báo empty state -->
            android:id="@+id/tv_empty"                                     <!-- Tạo ID để truy cập từ Java -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ text -->
            android:text="Không tìm thấy nhân viên nào"                    <!-- Nội dung thông báo -->
            android:gravity="center"                                       <!-- Căn giữa text -->
            android:padding="32dp"                                         <!-- Padding 32dp -->
            android:visibility="gone"                                      <!-- Ẩn mặc định -->
            android:textSize="16sp" />                                     <!-- Kích thước font 16sp -->

        <!-- ListView hiển thị danh sách nhân viên -->
        <ListView                                                          <!-- Widget hiển thị danh sách -->
            android:id="@+id/lv_nhan_vien"                                 <!-- Tạo ID để truy cập từ Java -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="match_parent"                           <!-- Chiều cao bằng parent -->
            android:divider="@android:color/transparent"                   <!-- Divider trong suốt -->
            android:dividerHeight="0dp"                                    <!-- Chiều cao divider = 0 -->
            android:paddingTop="8dp"                                       <!-- Padding top 8dp -->
            android:paddingBottom="88dp"                                   <!-- Padding bottom 88dp (để tránh FAB) -->
            android:clipToPadding="false" />                               <!-- Cho phép scroll vào padding area -->

    </LinearLayout>

    <!-- Floating Action Button để thêm nhân viên mới -->
    <com.google.android.material.floatingactionbutton.FloatingActionButton <!-- FAB Material Design -->
        android:id="@+id/fab_add_nhan_vien"                                <!-- Tạo ID để truy cập từ Java -->
        android:layout_width="wrap_content"                                <!-- Chiều rộng mặc định của FAB -->
        android:layout_height="wrap_content"                               <!-- Chiều cao mặc định của FAB -->
        android:layout_gravity="bottom|end"                                <!-- Định vị góc dưới phải -->
        android:layout_margin="24dp"                                       <!-- Margin 24dp từ các cạnh -->
        android:src="@android:drawable/ic_input_add"                       <!-- Icon thêm từ Android system -->
        app:backgroundTint="#2196F3"                                       <!-- Màu nền xanh Material -->
        app:tint="@android:color/white" />                                 <!-- Màu icon trắng -->

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

**Đặc điểm layout danh sách:**
- **CoordinatorLayout**: Hỗ trợ scroll behavior và FAB positioning
- **AppBarLayout**: Header có thể scroll với nội dung
- **Material Design**: Sử dụng SearchView, FAB theo chuẩn Material
- **Responsive**: Padding và margin phù hợp với các kích thước màn hình

---
### 5.2 Layout form thêm/sửa - activity_them_nhan_vien.xml

**Đường dẫn**: `app/src/main/res/layout/activity_them_nhan_vien.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"    <!-- Container có thể scroll -->
    xmlns:app="http://schemas.android.com/apk/res-auto"                    <!-- Namespace cho custom attributes -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent -->
    android:layout_height="match_parent"                                   <!-- Chiều cao bằng parent -->
    android:background="#f8f9fa">                                          <!-- Màu nền xám nhạt -->

    <LinearLayout                                                          <!-- Container chính sắp xếp theo chiều dọc -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ nội dung -->
        android:orientation="vertical">                                    <!-- Sắp xếp các view con theo chiều dọc -->

        <!-- Header -->
        <LinearLayout                                                      <!-- Container cho header -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ nội dung -->
            android:orientation="vertical"                                 <!-- Sắp xếp theo chiều dọc -->
            android:background="@drawable/gradient_header"                 <!-- Nền gradient từ drawable -->
            android:padding="24dp">                                        <!-- Padding 24dp cho tất cả các cạnh -->

            <TextView                                                      <!-- Text hiển thị tiêu đề -->
                android:id="@+id/tv_title"                                 <!-- Tạo ID để truy cập từ Java -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ text -->
                android:text="THÊM NHÂN VIÊN MỚI"                          <!-- Nội dung text mặc định -->
                android:textSize="20sp"                                    <!-- Kích thước font 20sp -->
                android:textStyle="bold"                                   <!-- Kiểu chữ đậm -->
                android:textColor="@android:color/white"                   <!-- Màu chữ trắng -->
                android:gravity="center" />                                <!-- Căn giữa text -->
        </LinearLayout>

        <!-- Form card -->
        <androidx.cardview.widget.CardView                                 <!-- Card container với shadow -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ nội dung -->
            android:layout_margin="16dp"                                   <!-- Margin 16dp từ tất cả các cạnh -->
            app:cardCornerRadius="12dp"                                    <!-- Bo góc 12dp -->
            app:cardElevation="4dp">                                       <!-- Độ cao shadow 4dp -->

            <LinearLayout                                                  <!-- Container form bên trong card -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ nội dung -->
                android:orientation="vertical"                             <!-- Sắp xếp theo chiều dọc -->
                android:padding="16dp"                                     <!-- Padding 16dp -->
                android:gravity="center_horizontal">                       <!-- Căn giữa theo chiều ngang -->

                <!-- Phần chọn ảnh đại diện -->
                <FrameLayout                                               <!-- Container cho phép overlay views -->
                    android:layout_width="wrap_content"                    <!-- Chiều rộng vừa đủ nội dung -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ nội dung -->
                    android:layout_marginBottom="16dp">                    <!-- Margin bottom 16dp -->
                    
                    <!-- ImageView hiển thị ảnh -->
                    <ImageView                                             <!-- View hiển thị ảnh -->
                        android:id="@+id/iv_avatar_setup"                  <!-- Tạo ID để truy cập từ Java -->
                        android:layout_width="100dp"                       <!-- Chiều rộng 100dp -->
                        android:layout_height="100dp"                      <!-- Chiều cao 100dp -->
                        android:src="@drawable/ic_person"                  <!-- Ảnh mặc định từ drawable -->
                        android:scaleType="centerCrop"                     <!-- Cắt ảnh để vừa khung -->
                        android:background="@drawable/card_feature"        <!-- Nền từ drawable -->
                        android:padding="2dp" />                           <!-- Padding 2dp -->
                    
                    <!-- Icon camera để chỉ dẫn -->
                    <ImageView                                             <!-- Icon overlay -->
                        android:layout_width="24dp"                        <!-- Chiều rộng 24dp -->
                        android:layout_height="24dp"                       <!-- Chiều cao 24dp -->
                        android:layout_gravity="bottom|end"                <!-- Định vị góc dưới phải -->
                        android:src="@android:drawable/ic_menu_camera"     <!-- Icon camera từ Android system -->
                        app:tint="#2196F3" />                              <!-- Tô màu xanh -->
                </FrameLayout>

                <!-- Mã nhân viên (auto-generated, disabled) -->
                <com.google.android.material.textfield.TextInputLayout     <!-- Material Design input container -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng bằng parent -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ nội dung -->
                    android:layout_marginBottom="12dp"                     <!-- Margin bottom 12dp -->
                    style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"> <!-- Style Material outlined -->
                    <com.google.android.material.textfield.TextInputEditText <!-- Material Design EditText -->
                        android:id="@+id/et_ma_nv"                         <!-- Tạo ID để truy cập từ Java -->
                        android:layout_width="match_parent"                <!-- Chiều rộng bằng parent -->
                        android:layout_height="wrap_content"               <!-- Chiều cao vừa đủ nội dung -->
                        android:hint="Mã nhân viên" />                     <!-- Placeholder text -->
                </com.google.android.material.textfield.TextInputLayout>

                <!-- Họ và tên -->
                <com.google.android.material.textfield.TextInputLayout     <!-- Material Design input container -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng bằng parent -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ nội dung -->
                    android:layout_marginBottom="12dp"                     <!-- Margin bottom 12dp -->
                    style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"> <!-- Style Material outlined -->
                    <com.google.android.material.textfield.TextInputEditText <!-- Material Design EditText -->
                        android:id="@+id/et_ho_ten"                        <!-- Tạo ID để truy cập từ Java -->
                        android:layout_width="match_parent"                <!-- Chiều rộng bằng parent -->
                        android:layout_height="wrap_content"               <!-- Chiều cao vừa đủ nội dung -->
                        android:hint="Họ và tên" />                        <!-- Placeholder text -->
                </com.google.android.material.textfield.TextInputLayout>

                <!-- Ngày sinh và giới tính (nằm ngang) -->
                <LinearLayout                                              <!-- Container ngang cho 2 field -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng bằng parent -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ nội dung -->
                    android:orientation="horizontal"                       <!-- Sắp xếp theo chiều ngang -->
                    android:layout_marginBottom="12dp">                    <!-- Margin bottom 12dp -->

                    <!-- Ngày sinh -->
                    <com.google.android.material.textfield.TextInputLayout <!-- Material Design input container -->
                        android:layout_width="0dp"                         <!-- Chiều rộng = 0 (sử dụng weight) -->
                        android:layout_height="wrap_content"               <!-- Chiều cao vừa đủ nội dung -->
                        android:layout_weight="1"                          <!-- Chiếm 1 phần trong layout weight -->
                        android:layout_marginEnd="8dp"                     <!-- Margin end 8dp -->
                        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"> <!-- Style Material outlined -->
                        <com.google.android.material.textfield.TextInputEditText <!-- Material Design EditText -->
                            android:id="@+id/et_ngay_sinh"                 <!-- Tạo ID để truy cập từ Java -->
                            android:layout_width="match_parent"            <!-- Chiều rộng bằng parent -->
                            android:layout_height="wrap_content"           <!-- Chiều cao vừa đủ nội dung -->
                            android:hint="Ngày sinh (yyyy-mm-dd)"          <!-- Placeholder text với format -->
                            android:focusable="false"                      <!-- Không cho phép focus (chỉ click) -->
                            android:clickable="true" />                    <!-- Cho phép click để mở DatePicker -->
                    </com.google.android.material.textfield.TextInputLayout>

                    <!-- Giới tính -->
                    <Spinner                                               <!-- Dropdown selection -->
                        android:id="@+id/sp_gioi_tinh"                     <!-- Tạo ID để truy cập từ Java -->
                        android:layout_width="0dp"                         <!-- Chiều rộng = 0 (sử dụng weight) -->
                        android:layout_height="match_parent"               <!-- Chiều cao bằng parent container -->
                        android:layout_weight="0.6"                        <!-- Chiếm 0.6 phần trong layout weight -->
                        android:entries="@array/gioi_tinh_array" />        <!-- Dữ liệu từ string array resource -->
                </LinearLayout>

                <!-- Số điện thoại -->
                <com.google.android.material.textfield.TextInputLayout     <!-- Material Design input container -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng bằng parent -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ nội dung -->
                    android:layout_marginBottom="12dp"                     <!-- Margin bottom 12dp -->
                    style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"> <!-- Style Material outlined -->
                    <com.google.android.material.textfield.TextInputEditText <!-- Material Design EditText -->
                        android:id="@+id/et_sdt"                           <!-- Tạo ID để truy cập từ Java -->
                        android:layout_width="match_parent"                <!-- Chiều rộng bằng parent -->
                        android:layout_height="wrap_content"               <!-- Chiều cao vừa đủ nội dung -->
                        android:hint="Số điện thoại"                       <!-- Placeholder text -->
                        android:inputType="phone"                          <!-- Kiểu input cho số điện thoại -->
                        android:maxLength="10" />                          <!-- Giới hạn tối đa 10 ký tự -->
                </com.google.android.material.textfield.TextInputLayout>

                <!-- Email -->
                <com.google.android.material.textfield.TextInputLayout     <!-- Material Design input container -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng bằng parent -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ nội dung -->
                    android:layout_marginBottom="12dp"                     <!-- Margin bottom 12dp -->
                    style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"> <!-- Style Material outlined -->
                    <com.google.android.material.textfield.TextInputEditText <!-- Material Design EditText -->
                        android:id="@+id/et_email"                         <!-- Tạo ID để truy cập từ Java -->
                        android:layout_width="match_parent"                <!-- Chiều rộng bằng parent -->
                        android:layout_height="wrap_content"               <!-- Chiều cao vừa đủ nội dung -->
                        android:hint="Email"                               <!-- Placeholder text -->
                        android:inputType="textEmailAddress" />           <!-- Kiểu input cho email -->
                </com.google.android.material.textfield.TextInputLayout>

                <!-- Phòng ban -->
                <TextView                                                  <!-- Label cho Spinner -->
                    android:layout_width="wrap_content"                    <!-- Chiều rộng vừa đủ text -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ text -->
                    android:text="Phòng ban"                               <!-- Nội dung label -->
                    android:layout_marginTop="8dp"                         <!-- Margin top 8dp -->
                    android:textStyle="bold" />                            <!-- Kiểu chữ đậm -->
                <Spinner                                                   <!-- Dropdown selection -->
                    android:id="@+id/sp_phong_ban"                         <!-- Tạo ID để truy cập từ Java -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng bằng parent -->
                    android:layout_height="48dp"                           <!-- Chiều cao cố định 48dp -->
                    android:layout_marginBottom="12dp" />                  <!-- Margin bottom 12dp -->

                <!-- Chức vụ -->
                <TextView                                                  <!-- Label cho Spinner -->
                    android:layout_width="wrap_content"                    <!-- Chiều rộng vừa đủ text -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ text -->
                    android:text="Chức vụ"                                 <!-- Nội dung label -->
                    android:textStyle="bold" />                            <!-- Kiểu chữ đậm -->
                <Spinner                                                   <!-- Dropdown selection -->
                    android:id="@+id/sp_chuc_vu"                           <!-- Tạo ID để truy cập từ Java -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng bằng parent -->
                    android:layout_height="48dp"                           <!-- Chiều cao cố định 48dp -->
                    android:layout_marginBottom="24dp" />                  <!-- Margin bottom 24dp -->

                <!-- Nút lưu -->
                <Button                                                    <!-- Button để submit form -->
                    android:id="@+id/btn_save"                             <!-- Tạo ID để truy cập từ Java -->
                    android:layout_width="match_parent"                    <!-- Chiều rộng bằng parent -->
                    android:layout_height="56dp"                           <!-- Chiều cao 56dp (chuẩn Material) -->
                    android:text="LƯU THÔNG TIN"                           <!-- Text trên button -->
                    android:textSize="16sp"                                <!-- Kích thước font 16sp -->
                    android:textStyle="bold"                               <!-- Kiểu chữ đậm -->
                    android:backgroundTint="#2196F3"                       <!-- Màu nền xanh Material -->
                    android:textColor="@android:color/white" />            <!-- Màu chữ trắng -->

            </LinearLayout>
        </androidx.cardview.widget.CardView>
    </LinearLayout>
</ScrollView>
```

**Đặc điểm layout form:**
- **ScrollView**: Cho phép scroll khi form dài
- **Material TextInputLayout**: Sử dụng Material Design cho input fields
- **Validation Support**: InputType phù hợp cho từng trường (phone, email)
- **User-friendly**: Spinner cho dropdown, DatePicker cho ngày sinh

---

### 5.3 Layout item trong ListView - item_nhan_vien.xml

**Đường dẫn**: `app/src/main/res/layout/item_nhan_vien.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding -->
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android" <!-- Card container với shadow -->
    xmlns:app="http://schemas.android.com/apk/res-auto"                    <!-- Namespace cho custom attributes -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent -->
    android:layout_height="wrap_content"                                   <!-- Chiều cao vừa đủ nội dung -->
    android:layout_marginHorizontal="12dp"                                 <!-- Margin ngang 12dp -->
    android:layout_marginVertical="6dp"                                    <!-- Margin dọc 6dp -->
    app:cardCornerRadius="12dp"                                            <!-- Bo góc 12dp -->
    app:cardElevation="4dp">                                               <!-- Độ cao shadow 4dp -->

    <LinearLayout                                                          <!-- Container chính sắp xếp ngang -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ nội dung -->
        android:orientation="horizontal"                                   <!-- Sắp xếp các view con theo chiều ngang -->
        android:padding="12dp"                                             <!-- Padding 12dp cho tất cả các cạnh -->
        android:gravity="center_vertical">                                 <!-- Căn giữa theo chiều dọc -->

        <!-- Ảnh đại diện -->
        <ImageView                                                         <!-- View hiển thị ảnh đại diện -->
            android:id="@+id/iv_avatar"                                     <!-- Tạo ID để truy cập từ Java -->
            android:layout_width="50dp"                                     <!-- Chiều rộng 50dp -->
            android:layout_height="50dp"                                    <!-- Chiều cao 50dp -->
            android:src="@drawable/ic_person"                              <!-- Ảnh mặc định từ drawable -->
            android:background="@drawable/card_feature"                    <!-- Nền từ drawable -->
            android:padding="8dp" />                                       <!-- Padding 8dp -->

        <!-- Thông tin nhân viên -->
        <LinearLayout                                                      <!-- Container cho thông tin text -->
            android:layout_width="0dp"                                     <!-- Chiều rộng = 0 (sử dụng weight) -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ nội dung -->
            android:layout_weight="1"                                      <!-- Chiếm toàn bộ không gian còn lại -->
            android:orientation="vertical"                                 <!-- Sắp xếp theo chiều dọc -->
            android:layout_marginStart="16dp">                             <!-- Margin start 16dp -->

            <!-- Tên nhân viên -->
            <TextView                                                      <!-- Text hiển thị tên -->
                android:id="@+id/tv_ten_nv"                                <!-- Tạo ID để truy cập từ Java -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ text -->
                android:text="Nguyễn Văn An"                               <!-- Text mẫu -->
                android:textSize="17sp"                                    <!-- Kích thước font 17sp -->
                android:textStyle="bold"                                   <!-- Kiểu chữ đậm -->
                android:textColor="#333" />                                <!-- Màu chữ xám đậm -->

            <!-- Mã nhân viên -->
            <TextView                                                      <!-- Text hiển thị mã NV -->
                android:id="@+id/tv_ma_nv"                                 <!-- Tạo ID để truy cập từ Java -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ text -->
                android:text="Mã: NV001"                                   <!-- Text mẫu -->
                android:textSize="13sp"                                    <!-- Kích thước font 13sp -->
                android:textColor="#666" />                                <!-- Màu chữ xám nhạt -->

            <!-- Chức vụ - Phòng ban -->
            <TextView                                                      <!-- Text hiển thị chức vụ và phòng ban -->
                android:id="@+id/tv_chuc_vu_phong_ban"                     <!-- Tạo ID để truy cập từ Java -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ text -->
                android:text="Giám đốc - Phòng Nhân sự"                    <!-- Text mẫu -->
                android:textSize="13sp"                                    <!-- Kích thước font 13sp -->
                android:textColor="#1976D2"                               <!-- Màu chữ xanh Material -->
                android:layout_marginTop="2dp" />                          <!-- Margin top 2dp -->

        </LinearLayout>

        <!-- Nút xóa -->
        <ImageButton                                                       <!-- Button hiển thị icon -->
            android:id="@+id/btn_delete_nv"                                <!-- Tạo ID để truy cập từ Java -->
            android:layout_width="40dp"                                    <!-- Chiều rộng 40dp -->
            android:layout_height="40dp"                                   <!-- Chiều cao 40dp -->
            android:src="@android:drawable/ic_menu_delete"                 <!-- Icon xóa từ Android system -->
            android:background="@drawable/btn_danger"                      <!-- Nền đỏ từ drawable -->
            app:tint="@android:color/white"                                <!-- Tô màu icon thành trắng -->
            android:padding="8dp" />                                       <!-- Padding 8dp -->

    </LinearLayout>

</androidx.cardview.widget.CardView>
```

**Đặc điểm layout item:**
- **CardView**: Tạo hiệu ứng card với shadow
- **Horizontal Layout**: Ảnh bên trái, thông tin giữa, nút xóa bên phải
- **Typography Hierarchy**: Tên nổi bật, mã và chức vụ nhỏ hơn
- **Action Button**: Nút xóa với màu đỏ cảnh báo

---

## 🔧 TÍCH HỢP VỚI DATABASE

### Các phương thức DatabaseHelper liên quan:

```java
// Trong DatabaseHelper.java
public Cursor getAllEmployees()                    // Lấy tất cả nhân viên
public Cursor searchEmployees(String keyword)     // Tìm kiếm nhân viên
public boolean addEmployee(...)                   // Thêm nhân viên mới
public boolean updateEmployee(...)                // Cập nhật nhân viên
public boolean deleteEmployee(String maNV)        // Xóa nhân viên
public String getNextEmployeeCode()               // Tạo mã NV tự động
public Cursor getAllDepartments()                 // Lấy danh sách phòng ban
public Cursor getAllPositions()                   // Lấy danh sách chức vụ
```

### SQL Query quan trọng:

```sql
-- Query lấy danh sách nhân viên với JOIN
SELECT nv.MaNhanVien, nv.HoTen, nv.GioiTinh, nv.HinhAnh,
       cv.TenChucVu, pb.TenPhongBan
FROM NhanVien nv
LEFT JOIN ChucVu cv ON nv.MaChucVu = cv.MaChucVu
LEFT JOIN PhongBan pb ON nv.MaPhongBan = pb.MaPhongBan
WHERE nv.TrangThaiLamViec = 'Đang làm việc'
ORDER BY nv.MaNhanVien
```

---

## 📋 TỔNG KẾT

### Luồng hoạt động chính:

1. **Khởi động**: QuanLyNhanVienActivity load danh sách từ database
2. **Hiển thị**: NhanVienAdapter render từng item trong ListView
3. **Tìm kiếm**: SearchView trigger query database real-time
4. **Thêm mới**: FAB mở ThemNhanVienActivity với mã NV tự động
5. **Chỉnh sửa**: Click item mở ThemNhanVienActivity với dữ liệu có sẵn
6. **Xóa**: Click nút xóa hiện dialog xác nhận
7. **Xuất PDF**: Tạo file PDF với danh sách hiện tại

### Ưu điểm của thiết kế:

- **Separation of Concerns**: Model, View, Controller tách biệt rõ ràng
- **Reusable Components**: Adapter có thể tái sử dụng, Activity dual-mode
- **User Experience**: Loading states, validation, confirmation dialogs
- **Modern Android**: ActivityResultLauncher, Material Design, Scoped Storage
- **Performance**: Cursor handling hiệu quả, ViewHolder pattern
- **Maintainable**: Code có cấu trúc, comment đầy đủ, naming convention nhất quán

### Điểm cần cải thiện:

- **Repository Pattern**: Tách logic database ra Repository layer
- **MVVM Architecture**: Sử dụng ViewModel và LiveData
- **Dependency Injection**: Sử dụng Dagger/Hilt
- **Unit Testing**: Thêm test cases cho business logic
- **Error Handling**: Xử lý lỗi network, database chi tiết hơn

---

*Tài liệu này cung cấp cái nhìn toàn diện về module Quản lý Nhân viên trong hệ thống QLNS. Mọi thắc mắc về implementation chi tiết có thể tham khảo source code hoặc liên hệ team phát triển.*