# SmartMenu: Hệ thống Nhận diện Món ăn & Tạo Thực đơn Tự động

![Python](https://img.shields.io/badge/Python-3.11-blue)
![PyTorch](https://img.shields.io/badge/PyTorch-2.3-ee4c2c)
![YOLOv12](https://img.shields.io/badge/YOLO-v12-00FFFF)
![FastAPI](https://img.shields.io/badge/FastAPI-0.110+-009688)

**SmartMenu** là hệ thống nhận diện món ăn đường phố Việt Nam theo thời gian thực và tự động khởi tạo thực đơn kỹ thuật số. Dự án kết hợp **YOLOv12**, **PP-OCRv4** và **Gemini 2.5 Flash** để chuyển đổi trực tiếp hình ảnh chụp tại bàn ăn thành thực đơn điện tử hoàn chỉnh (kèm giá bán, mô tả và ước tính dinh dưỡng).

---

## Tính năng chính

* **Nhận diện đa món ăn (Multi-dish Detection):** Phát hiện và định vị đồng thời nhiều món ăn trên bàn với tốc độ ~64 FPS (15.6 ms/ảnh).
* **Trích xuất giá bán tự động (Event-driven OCR):** Tự động bật PP-OCRv4 khi phát hiện vùng chữ để đọc tên món hoặc giá tiền trên biển hiệu/bảng giá và ghép nối với món ăn gần nhất.
* **Tạo nội dung tự động bằng LLM:** Sử dụng Gemini 2.5 Flash để chuẩn hóa tên món, viết mô tả ngắn gọn và ước tính thông số dinh dưỡng (Calories, Protein, Fat, Carbs) dưới dạng JSON.
* **Xuất Menu đa ngôn ngữ:** Hỗ trợ xem thông tin món lẻ hoặc gom nhiều món thành một trang Menu hoàn chỉnh (xuất PDF). Giao diện hỗ trợ 5 ngôn ngữ: Việt, Anh, Trung, Nhật, Hàn.

## Luồng xử lý (Pipeline)

```text
[Ảnh / Camera] 
      ├──> [YOLOv12] ──> Tọa độ & Nhãn món ăn ──┐
      │                                         ├──> [Ghép nối Không gian] ──> [Gemini 2.5 Flash] ──> [Menu JSON / PDF]
      └──> [PP-OCRv4] ─> Giá tiền & Ghi chú  ───┘
```

1. **Thị giác (YOLOv12):** Tìm vị trí các món ăn trong khung hình.
2. **Đọc chữ (PP-OCRv4):** Chỉ kích hoạt khi có chữ trong ảnh để tránh lãng phí tài nguyên.
3. **Liên kết không gian:** Ghép cặp món ăn với vùng văn bản (giá tiền/tên món) có khoảng cách tâm gần nhất.
4. **Tổng hợp ngữ nghĩa (Gemini API):** Chuẩn hóa toàn bộ dữ liệu thô thành định dạng JSON chuẩn để hiển thị lên giao diện người dùng.

## Dữ liệu & Hiệu năng

Mô hình được huấn luyện qua 2 giai đoạn: Tiền huấn luyện trên **111 lớp** (mở rộng từ bộ *Food-101*), sau đó tinh chỉnh (fine-tune) trên **>3.500 ảnh thực tế** của 14 món ăn đường phố Việt Nam (Phở bò, Bún bò Huế, Bún đậu, Cơm tấm, Bánh mì, Gỏi cuốn, Bún thịt nướng, Bánh cuốn, Bánh giò, Cháo lòng, Cốm, Cơm chiên, Xôi...).

**Kết quả thực nghiệm trên tập Test (GPU NVIDIA T4, ảnh $640 \times 640$):**

| Mô hình | mAP@0.5 | mAP@0.5:0.95 | Tham số (Params) | Độ trễ (Latency) |
| :--- | :---: | :---: | :---: | :---: |
| YOLOv8l | 0.871 | 0.585 | 43.7M | 18.2 ms |
| **YOLOv12l (Ours)** | **0.924** | **0.652** | **26.4M** | **15.6 ms** |

## Cài đặt & Sử dụng

### 1. Cài đặt môi trường

Yêu cầu Python `>= 3.11` và GPU hỗ trợ CUDA (khuyến nghị).

```bash
git clone https://github.com/your-username/SmartMenu.git
cd SmartMenu

python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

pip install -r requirements.txt
```

### 2. Cấu hình

Tạo file `.env` ở thư mục gốc của dự án:

```env
GEMINI_API_KEY=your_gemini_api_key
YOLO_WEIGHTS=weights/yolov12l_vnfood.pt
USE_OCR=True
```

### 3. Khởi chạy

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

Truy cập `http://localhost:8000` trên trình duyệt để sử dụng ứng dụng.

## Công bố khoa học (Publications)

Nếu bạn sử dụng bộ dữ liệu hoặc mã nguồn của dự án này cho mục đích nghiên cứu, vui lòng trích dẫn các bài báo sau của nhóm:

```bibtex
@inproceedings{nguyen2026multimodal,
  title={Multimodal Vision-Language Understanding for Real-Time Multi-Dish Recognition and Semantic-Aware Digital Menu Generation},
  author={Nguyen, T. H. H. and Duong, T. H. T. and Vu, D. H. and Nguyen, T. L. A. and Tran, T. H. A. and Pham, M. H.},
  booktitle={International Conference on Intelligent Systems and Networks (ICISN)},
  year={2026}
}

@inproceedings{nguyen2026realtime,
  title={A Real-Time Smart Service Framework for Vietnamese Cuisine: Integrating YOLOv12 and Semantic Synthesis for Automated Digital Menus},
  author={Nguyen, T. H. H. and Nguyen, L. A. and Vu, D. H. and Duong, H. T. and Pham, M. H.},
  booktitle={Hội thảo khoa học Quốc gia về Trí tuệ nhân tạo (FJCAI)},
  year={2026}
}
```

## Nhóm phát triển

Dự án Nghiên cứu Khoa học Sinh viên (2025 - 2026) tại **Khoa Công nghệ thông tin - Đại học Công nghiệp Hà Nội (HaUI)**:

* **Giảng viên hướng dẫn:** TS. Nguyễn Thị Hoa Huệ
* **Thành viên thực hiện:** Vũ Đình Hiệp, Nguyễn Thị Lan Anh, Phạm Minh Hoàng, Dương Thị Hương Trà.