[# KNPM-BTL-1
# -*- coding: utf-8 -*-
"""
HỆ THỐNG QUẢN LÝ ĐIỂM SINH VIÊN TRƯỜNG ĐẠI HỌC HẢI PHÒNG (HPUNI)
Học phần: Kỹ nghệ phần mềm (INF722)
Cập nhật: Phân quyền độc lập cho từng tài khoản Sinh viên theo Mã SV:
  - 241321111  : Trần Tăng Vinh
  - 241321112  : Nguyễn Thanh Thưởng
  - 2413211113 : Lê Xuân Hiếu 
  - 2413211114 : Trần Thành Khang 
Mỗi sinh viên đăng nhập chỉ xem được bảng điểm, ĐTB học kỳ và học phần F của chính cá nhân mình!
"""

import tkinter as tk
from tkinter import ttk, messagebox, filedialog
import csv
from datetime import datetime

class HPUNIGradeSystem:
    def __init__(self, root):
        self.root = root
        self.root.title("HỆ THỐNG QUẢN LÝ ĐIỂM SINH VIÊN - TRƯỜNG ĐẠI HỌC HẢI PHÒNG (HPUNI)")
        self.root.geometry("1300x780")
        self.root.minsize(1180, 700)
        self.root.configure(bg="#0a192f")

        # ================= 1. HỆ THỐNG TÀI KHOẢN ĐỘC LẬP (MẬT KHẨU ĐƠN GIẢN: 1) =================
        self.users = {
            # Nhóm Quản lý & Giảng viên (Xem toàn trường / toàn lớp)
            "admin": {"pass": "1", "role": "Quản trị viên (Admin)", "name": "Quản trị hệ thống", "code": "ADMIN"},
            "pdt":   {"pass": "1", "role": "Phòng Đào tạo", "name": "Cán bộ Phòng Đào tạo", "code": "PDT"},
            "gv":    {"pass": "1", "role": "Giảng viên", "name": "Giảng viên bộ môn", "code": "GV"},
            "cvht":  {"pass": "1", "role": "Cố vấn học tập", "name": "Cố vấn học tập CNTT3K25", "code": "CVHT"},

            # 4 Tài khoản Sinh viên độc lập theo đúng Mã SV yêu cầu (Mật khẩu đơn giản: 1)
            "241321111":  {"pass": "1", "role": "Sinh viên", "name": "Trần Tăng Vinh",      "code": "241321111"},
            "241321112":  {"pass": "1", "role": "Sinh viên", "name": "Nguyễn Thanh Thưởng", "code": "241321112"},
            "2413211113": {"pass": "1", "role": "Sinh viên", "name": "Lê Xuân Hiếu",        "code": "2413211113"},
            "2413211114": {"pass": "1", "role": "Sinh viên", "name": "Trần Thành Khang",    "code": "2413211114"},

            # Hỗ trợ đăng nhập nếu gõ mã 9 chữ số (241321113 / 241321114) vẫn trỏ đúng về tài khoản cá nhân
            "241321113":  {"pass": "1", "role": "Sinh viên", "name": "Lê Xuân Hiếu",        "code": "2413211113"},
            "241321114":  {"pass": "1", "role": "Sinh viên", "name": "Trần Thành Khang",    "code": "2413211114"},

            # 2 Sinh viên bổ sung trong bộ dữ liệu mẫu 6 sinh viên
            "2413211115": {"pass": "1", "role": "Sinh viên", "name": "Trần Văn Nam",        "code": "2413211115"},
            "2413211116": {"pass": "1", "role": "Sinh viên", "name": "Bùi Xuân Dũng",       "code": "2413211116"}
        }

        # ================= 2. DANH MỤC HỌC PHẦN & HỒ SƠ SINH VIÊN =================
        self.courses_info = {
            "HP01": {"ten": "Triết học Mác - Lênin", "tc": 3},
            "HP02": {"ten": "Nhập môn thuật toán", "tc": 2},
            "HP03": {"ten": "Toán học rời rạc", "tc": 3},
            "HP04": {"ten": "Tin học cơ sở (Chuẩn MOS)", "tc": 3},
            "HP05": {"ten": "Tổ chức và kiến trúc máy tính", "tc": 3}
        }

        self.students_info = {
            "241321111":  {"ten": "Trần Tăng Vinh",      "lop": "CNTT3K25", "khoa": "Công nghệ thông tin"},
            "241321112":  {"ten": "Nguyễn Thanh Thưởng", "lop": "CNTT3K25", "khoa": "Công nghệ thông tin"},
            "2413211113": {"ten": "Lê Xuân Hiếu",        "lop": "CNTT3K25", "khoa": "Công nghệ thông tin"},
            "2413211114": {"ten": "Trần Thành Khang",    "lop": "CNTT3K25", "khoa": "Công nghệ thông tin"},
            "2413211115": {"ten": "Trần Văn Nam",        "lop": "CNTT3K25", "khoa": "Công nghệ thông tin"},
            "2413211116": {"ten": "Bùi Xuân Dũng",       "lop": "CNTT3K25", "khoa": "Công nghệ thông tin"}
        }

        # ================= 3. DỮ LIỆU ĐIỂM 30 HỌC PHẦN GẮN VỚI MÃ SINH VIÊN MỚI =================
        # Trọng số chuẩn: Chuyên cần 20% (0.2), Kiểm tra 30% (0.3), Thi 50% (0.5)
        self.grades_data = [
            # 1. Trần Tăng Vinh - Mã SV: 241321111 (ĐTB: 7.45 | 0 môn F)
            {"id": 1,  "ma_sv": "241321111", "ma_hp": "HP01", "hk": "HK1 (2025-2026)", "cc": 8.0,  "kt": 7.0,  "thi": 9.0,  "locked": True},
            {"id": 2,  "ma_sv": "241321111", "ma_hp": "HP02", "hk": "HK1 (2025-2026)", "cc": 9.0,  "kt": 8.0,  "thi": 6.0,  "locked": True},
            {"id": 3,  "ma_sv": "241321111", "ma_hp": "HP03", "hk": "HK1 (2025-2026)", "cc": 8.0,  "kt": 7.0,  "thi": 6.5,  "locked": True},
            {"id": 4,  "ma_sv": "241321111", "ma_hp": "HP04", "hk": "HK1 (2025-2026)", "cc": 9.0,  "kt": 8.0,  "thi": 7.0,  "locked": True},
            {"id": 5,  "ma_sv": "241321111", "ma_hp": "HP05", "hk": "HK1 (2025-2026)", "cc": 10.0, "kt": 7.0,  "thi": 6.0,  "locked": True},

            # 2. Nguyễn Thanh Thưởng - Mã SV: 241321112 (ĐTB: 7.46 | 0 môn F)
            {"id": 6,  "ma_sv": "241321112", "ma_hp": "HP01", "hk": "HK1 (2025-2026)", "cc": 8.0,  "kt": 7.5,  "thi": 8.0,  "locked": True},
            {"id": 7,  "ma_sv": "241321112", "ma_hp": "HP02", "hk": "HK1 (2025-2026)", "cc": 9.0,  "kt": 8.0,  "thi": 7.0,  "locked": True},
            {"id": 8,  "ma_sv": "241321112", "ma_hp": "HP03", "hk": "HK1 (2025-2026)", "cc": 7.0,  "kt": 7.0,  "thi": 6.0,  "locked": True},
            {"id": 9,  "ma_sv": "241321112", "ma_hp": "HP04", "hk": "HK1 (2025-2026)", "cc": 8.0,  "kt": 8.0,  "thi": 9.0,  "locked": True},
            {"id": 10, "ma_sv": "241321112", "ma_hp": "HP05", "hk": "HK1 (2025-2026)", "cc": 7.0,  "kt": 6.5,  "thi": 7.0,  "locked": True},

            # 3. Lê Xuân Hiếu - Mã SV: 2413211113 (ĐTB: 5.31 | 1 môn F: Nhập môn thuật toán = 2.20)
            {"id": 11, "ma_sv": "2413211113", "ma_hp": "HP01", "hk": "HK1 (2025-2026)", "cc": 6.0,  "kt": 5.0,  "thi": 5.0,  "locked": True},
            {"id": 12, "ma_sv": "2413211113", "ma_hp": "HP02", "hk": "HK1 (2025-2026)", "cc": 3.0,  "kt": 2.0,  "thi": 2.0,  "locked": True},
            {"id": 13, "ma_sv": "2413211113", "ma_hp": "HP03", "hk": "HK1 (2025-2026)", "cc": 7.0,  "kt": 6.0,  "thi": 6.0,  "locked": True},
            {"id": 14, "ma_sv": "2413211113", "ma_hp": "HP04", "hk": "HK1 (2025-2026)", "cc": 8.0,  "kt": 7.0,  "thi": 7.0,  "locked": True},
            {"id": 15, "ma_sv": "2413211113", "ma_hp": "HP05", "hk": "HK1 (2025-2026)", "cc": 5.0,  "kt": 4.0,  "thi": 5.0,  "locked": True},

            # 4. Trần Thành Khang - Mã SV: 2413211114 (ĐTB: 6.42 | 1 môn F: Toán học rời rạc = 2.50 < 3.0)
            {"id": 16, "ma_sv": "2413211114", "ma_hp": "HP01", "hk": "HK1 (2025-2026)", "cc": 8.0,  "kt": 8.0,  "thi": 8.0,  "locked": True},
            {"id": 17, "ma_sv": "2413211114", "ma_hp": "HP02", "hk": "HK1 (2025-2026)", "cc": 7.0,  "kt": 8.0,  "thi": 7.0,  "locked": True},
            {"id": 18, "ma_sv": "2413211114", "ma_hp": "HP03", "hk": "HK1 (2025-2026)", "cc": 2.5,  "kt": 2.5,  "thi": 2.5,  "locked": True},
            {"id": 19, "ma_sv": "2413211114", "ma_hp": "HP04", "hk": "HK1 (2025-2026)", "cc": 8.0,  "kt": 9.0,  "thi": 8.0,  "locked": True},
            {"id": 20, "ma_sv": "2413211114", "ma_hp": "HP05", "hk": "HK1 (2025-2026)", "cc": 6.0,  "kt": 7.0,  "thi": 6.0,  "locked": True},

            # 5. Trần Văn Nam - Mã SV: 2413211115 (ĐTB: 4.18 | 2 môn F: Triết học = 2.00, Toán rời rạc = 2.49)
            {"id": 21, "ma_sv": "2413211115", "ma_hp": "HP01", "hk": "HK1 (2025-2026)", "cc": 2.0,  "kt": 2.0,  "thi": 2.0,  "locked": True},
            {"id": 22, "ma_sv": "2413211115", "ma_hp": "HP02", "hk": "HK1 (2025-2026)", "cc": 6.0,  "kt": 5.0,  "thi": 6.0,  "locked": True},
            {"id": 23, "ma_sv": "2413211115", "ma_hp": "HP03", "hk": "HK1 (2025-2026)", "cc": 2.49, "kt": 2.49, "thi": 2.49, "locked": True},
            {"id": 24, "ma_sv": "2413211115", "ma_hp": "HP04", "hk": "HK1 (2025-2026)", "cc": 7.0,  "kt": 6.0,  "thi": 7.0,  "locked": True},
            {"id": 25, "ma_sv": "2413211115", "ma_hp": "HP05", "hk": "HK1 (2025-2026)", "cc": 5.0,  "kt": 5.0,  "thi": 4.0,  "locked": True},

            # 6. Bùi Xuân Dũng - Mã SV: 2413211116 (ĐTB: 8.83 | 0 môn F)
            {"id": 26, "ma_sv": "2413211116", "ma_hp": "HP01", "hk": "HK1 (2025-2026)", "cc": 9.0,  "kt": 9.0,  "thi": 9.0,  "locked": True},
            {"id": 27, "ma_sv": "2413211116", "ma_hp": "HP02", "hk": "HK1 (2025-2026)", "cc": 9.0,  "kt": 8.5,  "thi": 9.0,  "locked": True},
            {"id": 28, "ma_sv": "2413211116", "ma_hp": "HP03", "hk": "HK1 (2025-2026)", "cc": 8.0,  "kt": 9.0,  "thi": 8.0,  "locked": True},
            {"id": 29, "ma_sv": "2413211116", "ma_hp": "HP04", "hk": "HK1 (2025-2026)", "cc": 9.0,  "kt": 9.0,  "thi": 10.0, "locked": True},
            {"id": 30, "ma_sv": "2413211116", "ma_hp": "HP05", "hk": "HK1 (2025-2026)", "cc": 8.0,  "kt": 8.0,  "thi": 9.0,  "locked": True}
        ]

        self.audit_logs = [
            f"[{datetime.now().strftime('%d/%m/%Y %H:%M:%S')}] Khởi tạo hệ thống Quản lý điểm HPUNI thành công.",
            f"[{datetime.now().strftime('%d/%m/%Y %H:%M:%S')}] Đã kích hoạt 4 tài khoản Sinh viên độc lập: 241321111, 241321112, 2413211113, 2413211114."
        ]

        self.current_user = None
        self.setup_styles()
        self.show_login_screen()

    # ================= 4. CẤU HÌNH GIAO DIỆN & STYLE =================
    def setup_styles(self):
        style = ttk.Style()
        style.theme_use("clam")
        style.configure("Treeview.Heading", font=("Segoe UI", 10, "bold"), background="#1e3a8a", foreground="white", padding=6)
        style.configure("Treeview", font=("Segoe UI", 10), rowheight=28, background="white", fieldbackground="white")
        style.map("Treeview", background=[("selected", "#2563eb")], foreground=[("selected", "white")])
        style.configure("TNotebook", background="#f1f5f9", borderwidth=0)
        style.configure("TNotebook.Tab", font=("Segoe UI", 10, "bold"), padding=[16, 8], background="#cbd5e1", foreground="#1e293b")
        style.map("TNotebook.Tab", background=[("selected", "#0284c7")], foreground=[("selected", "white")])

    def clear_root(self):
        for widget in self.root.winfo_children():
            widget.destroy()

    def is_student(self):
        return self.current_user and self.current_user["role"] == "Sinh viên"

    # ================= 5. THUẬT TOÁN TÍNH ĐIỂM & QUY ĐỔI A, B, C, D, F =================
    def evaluate_course_score(self, cc, kt, thi):
        raw_score = (cc * 0.20) + (kt * 0.30) + (thi * 0.50)
        disp_score = round(raw_score, 2)
        if raw_score < 3.0:
            return disp_score, "F", "Kém (Trượt môn - Học lại)", "Không đạt (F)"
        elif raw_score < 5.0:
            return disp_score, "D", "Trung bình yếu", "Đạt"
        elif raw_score < 7.0:
            return disp_score, "C", "Trung bình", "Đạt"
        elif raw_score < 9.0:
            return disp_score, "B", "Khá", "Đạt"
        else:
            return disp_score, "A", "Giỏi / Xuất sắc", "Đạt"

    # ================= 6. MÀN HÌNH ĐĂNG NHẬP (BACKGROUND NỔI BẬT + CHỌN NHANH 4 SV) =================
    def show_login_screen(self):
        self.clear_root()
        canvas = tk.Canvas(self.root, highlightthickness=0)
        canvas.pack(fill=tk.BOTH, expand=True)

        def draw_gradient(event=None):
            canvas.delete("bg")
            w = canvas.winfo_width()
            h = canvas.winfo_height()
            if w < 10 or h < 10:
                return
            steps = 60
            r1, g1, b1 = (10, 25, 47)     # #0a192f
            r2, g2, b2 = (14, 116, 144)   # #0e7490
            for i in range(steps):
                r = int(r1 + (r2 - r1) * i / steps)
                g = int(g1 + (g2 - g1) * i / steps)
                b = int(b1 + (b2 - b1) * i / steps)
                color = f"#{r:02x}{g:02x}{b:02x}"
                y0 = int(h * i / steps)
                y1 = int(h * (i + 1) / steps) + 1
                canvas.create_rectangle(0, y0, w, y1, fill=color, outline=color, tags="bg")

            canvas.create_oval(-120, -120, 320, 320, fill="#1e3a8a", outline="", tags="bg")
            canvas.create_oval(w - 360, h - 360, w + 120, h + 120, fill="#0284c7", outline="", tags="bg")
            canvas.create_oval(w - 220, -80, w + 80, 220, fill="#f59e0b", outline="", tags="bg")
            canvas.tag_lower("bg")

        canvas.bind("<Configure>", draw_gradient)

        card = tk.Frame(canvas, bg="#ffffff", bd=0, highlightthickness=3, highlightbackground="#38bdf8")
        card.place(relx=0.5, rely=0.5, anchor=tk.CENTER, width=580, height=670)

        top_banner = tk.Frame(card, bg="#0f172a", height=100)
        top_banner.pack(fill=tk.X)
        top_banner.pack_propagate(False)

        tk.Label(top_banner, text="🎓 TRƯỜNG ĐẠI HỌC HẢI PHÒNG (HPUNI)", font=("Segoe UI", 15, "bold"), fg="#38bdf8", bg="#0f172a").pack(pady=(18, 4))
        tk.Label(top_banner, text="HỆ THỐNG QUẢN LÝ ĐIỂM & TRA CỨU KẾT QUẢ HỌC TẬP CÁ NHÂN", font=("Segoe UI", 10, "bold"), fg="#f8fafc", bg="#0f172a").pack()

        body = tk.Frame(card, bg="white", padx=40, pady=15)
        body.pack(fill=tk.BOTH, expand=True)

        tk.Label(body, text="ĐĂNG NHẬP HỆ THỐNG", font=("Segoe UI", 13, "bold"), fg="#0f172a", bg="white").pack(pady=(0, 10))

        tk.Label(body, text="Tài khoản (Mã sinh viên hoặc Username):", font=("Segoe UI", 10, "bold"), fg="#334155", bg="white").pack(anchor="w")
        self.ent_user = tk.Entry(body, font=("Segoe UI", 12), bg="#f1f5f9", fg="#0f172a", relief=tk.SOLID, bd=1)
        self.ent_user.pack(fill=tk.X, pady=(4, 12), ipady=5)
        self.ent_user.insert(0, "241321111")

        tk.Label(body, text="Mật khẩu đơn giản (Mặc định là 1):", font=("Segoe UI", 10, "bold"), fg="#334155", bg="white").pack(anchor="w")
        self.ent_pass = tk.Entry(body, font=("Segoe UI", 12), show="●", bg="#f1f5f9", fg="#0f172a", relief=tk.SOLID, bd=1)
        self.ent_pass.pack(fill=tk.X, pady=(4, 14), ipady=5)
        self.ent_pass.insert(0, "1")

        btn_login = tk.Button(
            body, text="🚀 ĐĂNG NHẬP HỆ THỐNG", font=("Segoe UI", 11, "bold"),
            bg="#0284c7", fg="white", activebackground="#0369a1", activeforeground="white",
            relief=tk.FLAT, cursor="hand2", command=self.handle_login
        )
        btn_login.pack(fill=tk.X, ipady=8)

        # Khung Đăng nhập nhanh 4 Tài khoản Sinh viên độc lập
        tk.Label(body, text="👨‍🎓 Đăng nhập nhanh 4 Tài khoản Sinh viên độc lập (Chỉ xem điểm cá nhân - Mật khẩu: 1):", font=("Segoe UI", 9, "bold"), fg="#be123c", bg="white").pack(anchor="w", pady=(14, 4))
        sv_grid = tk.Frame(body, bg="white")
        sv_grid.pack(fill=tk.X)

        sv_accounts = [
            ("241321111 - Trần Tăng Vinh", "241321111", "#0f766e"),
            ("241321112 - Nguyễn Thanh Thưởng", "241321112", "#1d4ed8"),
            ("2413211113 - Lê Xuân Hiếu", "2413211113", "#b45309"),
            ("2413211114 - Trần Thành Khang", "2413211114", "#be123c")
        ]
        for idx, (label, u_code, col) in enumerate(sv_accounts):
            r, c = divmod(idx, 2)
            b = tk.Button(
                sv_grid, text=label, font=("Segoe UI", 9, "bold"),
                bg=col, fg="white", relief=tk.FLAT, cursor="hand2",
                command=lambda uc=u_code: self.quick_fill_login(uc)
            )
            b.grid(row=r, column=c, sticky="ew", padx=3, pady=3, ipady=5)
        sv_grid.columnconfigure(0, weight=1)
        sv_grid.columnconfigure(1, weight=1)

        # Khung Đăng nhập nhanh Quản trị / Giảng viên
        tk.Label(body, text="🛡️ Đăng nhập nhanh vai trò Quản lý / Giảng viên (Xem toàn bộ sinh viên - Mật khẩu: 1):", font=("Segoe UI", 9, "bold"), fg="#334155", bg="white").pack(anchor="w", pady=(12, 4))
        staff_frame = tk.Frame(body, bg="white")
        staff_frame.pack(fill=tk.X)

        staff_roles = [
            ("Admin (admin)", "admin", "#1e293b"),
            ("P.Đào tạo (pdt)", "pdt", "#334155"),
            ("Giảng viên (gv)", "gv", "#475569"),
            ("Cố vấn HT (cvht)", "cvht", "#64748b")
        ]
        for label, u_code, col in staff_roles:
            b = tk.Button(
                staff_frame, text=label, font=("Segoe UI", 8, "bold"),
                bg=col, fg="white", relief=tk.FLAT, cursor="hand2",
                command=lambda uc=u_code: self.quick_fill_login(uc)
            )
            b.pack(side=tk.LEFT, expand=True, fill=tk.X, padx=2, ipady=4)

        info_box = tk.Frame(body, bg="#f8fafc", bd=1, relief=tk.SOLID)
        info_box.pack(fill=tk.X, pady=(14, 0))
        hint_text = (
            "🔒 Phân quyền bảo mật: Mỗi sinh viên là 1 tài khoản độc lập (Username = Mã SV, Password = 1).\n"
            "   Khi sinh viên đăng nhập, hệ thống chỉ hiển thị bảng điểm, ĐTB và môn F của chính sinh viên đó!"
        )
        tk.Label(info_box, text=hint_text, font=("Segoe UI", 8), fg="#1e293b", bg="#f8fafc", justify=tk.LEFT, padx=8, pady=5).pack(anchor="w")

        self.root.bind("<Return>", lambda event: self.handle_login())

    def quick_fill_login(self, u_code):
        self.ent_user.delete(0, tk.END)
        self.ent_user.insert(0, u_code)
        self.ent_pass.delete(0, tk.END)
        self.ent_pass.insert(0, "1")
        self.handle_login()

    def handle_login(self):
        u = self.ent_user.get().strip().lower()
        p = self.ent_pass.get().strip()
        if u in self.users and self.users[u]["pass"] == p:
            self.current_user = self.users[u].copy()
            self.current_user["username"] = u
            self.root.unbind("<Return>")
            self.audit_logs.append(
                f"[{datetime.now().strftime('%d/%m/%Y %H:%M:%S')}] Đăng nhập: {self.current_user['name']} (Mã: {self.current_user['code']} | Vai trò: {self.current_user['role']})."
            )
            self.show_main_dashboard()
        else:
            messagebox.showerror(
                "Lỗi xác thực",
                "Mã đăng nhập hoặc mật khẩu không đúng!\n\n"
                "• Tài khoản 4 Sinh viên: 241321111, 241321112, 2413211113, 2413211114\n"
                "• Tài khoản Quản lý: admin, pdt, gv, cvht\n"
                "• Mật khẩu chung đơn giản: 1"
            )

    def handle_logout(self):
        if messagebox.askyesno("Đăng xuất", "Bạn có chắc chắn muốn đăng xuất khỏi hệ thống?"):
            self.current_user = None
            self.show_login_screen()

    # ================= 7. MÀN HÌNH CHÍNH (TỰ ĐỘNG CÁCH LY DỮ LIỆU CÁ NHÂN CHO SINH VIÊN) =================
    def show_main_dashboard(self):
        self.clear_root()
        self.root.configure(bg="#e2e8f0")

        header = tk.Frame(self.root, bg="#0f172a", height=68)
        header.pack(fill=tk.X, side=tk.TOP)
        header.pack_propagate(False)

        title_text = (
            f"🏛️ HPUNI - CỔNG THÔNG TIN ĐIỂM CÁ NHÂN SINH VIÊN [{self.current_user['code']} - {self.current_user['name'].upper()}]"
            if self.is_student() else
            "🏛️ TRƯỜNG ĐẠI HỌC HẢI PHÒNG (HPUNI) - HỆ THỐNG QUẢN LÝ ĐIỂM TÍN CHỈ"
        )
        tk.Label(header, text=title_text, font=("Segoe UI", 12, "bold"), fg="#38bdf8", bg="#0f172a").pack(side=tk.LEFT, padx=20)

        btn_logout = tk.Button(
            header, text="🚪 Đăng xuất", font=("Segoe UI", 9, "bold"),
            bg="#ef4444", fg="white", activebackground="#dc2626", relief=tk.FLAT,
            cursor="hand2", padx=12, pady=5, command=self.handle_logout
        )
        btn_logout.pack(side=tk.RIGHT, padx=20)

        user_badge = f"👤 {self.current_user['name']} (Mã: {self.current_user['code']})  |  Vai trò: {self.current_user['role']}"
        tk.Label(header, text=user_badge, font=("Segoe UI", 10, "bold"), fg="#f8fafc", bg="#0f172a").pack(side=tk.RIGHT, padx=10)

        # Dãy Thẻ Thống Kê Nhanh (KPI Cards) - Cá nhân hóa nếu là Sinh viên
        self.kpi_frame = tk.Frame(self.root, bg="#e2e8f0", pady=10, padx=15)
        self.kpi_frame.pack(fill=tk.X)
        self.render_kpi_cards()

        # Khu vực Tabs Chức Năng
        self.notebook = ttk.Notebook(self.root)
        self.notebook.pack(fill=tk.BOTH, expand=True, padx=15, pady=(0, 15))

        tab1_title = " 📊 1. Bảng Điểm Cá Nhân Của Tôi " if self.is_student() else " 📊 1. Bảng Điểm Học Phần Chi Tiết "
        self.tab_manage = tk.Frame(self.notebook, bg="#ffffff")
        self.notebook.add(self.tab_manage, text=tab1_title)
        self.build_tab_manage()

        tab2_title = " 📈 2. Điểm Trung Bình Học Kỳ Cá Nhân " if self.is_student() else " 📈 2. Tổng Hợp Điểm Trung Bình Học Kỳ "
        self.tab_gpa = tk.Frame(self.notebook, bg="#ffffff")
        self.notebook.add(self.tab_gpa, text=tab2_title)
        self.build_tab_gpa()

        tab3_title = " 🚨 3. Học Phần Bị Điểm F Của Tôi (< 3.0) " if self.is_student() else " 🚨 3. Trích Xuất Danh Sách Điểm F (< 3.0) "
        self.tab_extract_f = tk.Frame(self.notebook, bg="#ffffff")
        self.notebook.add(self.tab_extract_f, text=tab3_title)
        self.build_tab_extract_f()

        self.tab_project = tk.Frame(self.notebook, bg="#ffffff")
        self.notebook.add(self.tab_project, text=" 📋 4. Hồ Sơ Tài Khoản & Quy Tắc Điểm ")
        self.build_tab_project()

    def render_kpi_cards(self):
        for w in self.kpi_frame.winfo_children():
            w.destroy()

        if self.is_student():
            my_code = self.current_user["code"]
            my_recs = [g for g in self.grades_data if g["ma_sv"] == my_code]
            total_tc = sum(self.courses_info[g["ma_hp"]]["tc"] for g in my_recs)
            weighted_sum = sum((g["cc"]*0.2 + g["kt"]*0.3 + g["thi"]*0.5) * self.courses_info[g["ma_hp"]]["tc"] for g in my_recs)
            my_gpa = round(weighted_sum / total_tc, 2) if total_tc > 0 else 0.0
            my_f = sum(1 for g in my_recs if self.evaluate_course_score(g["cc"], g["kt"], g["thi"])[1] == "F")
            my_pass = len(my_recs) - my_f

            cards_data = [
                ("SINH VIÊN ĐĂNG NHẬP ĐỘC LẬP", f"{self.current_user['name']}", f"Mã SV: {my_code} | Lớp: CNTT3K25", "#1e40af"),
                ("ĐIỂM TRUNG BÌNH HỌC KỲ", f"{my_gpa:.2f} / 10.0", f"Tính trên {len(my_recs)} học phần ({total_tc} tín chỉ)", "#0f766e"),
                ("SỐ HỌC PHẦN ĐẠT (>= 3.0)", f"{my_pass} / {len(my_recs)} Học phần", "Đủ điều kiện qua môn", "#15803d"),
                ("HỌC PHẦN BỊ ĐIỂM F (< 3.0)", f"{my_f} Học phần F", "Cần đăng ký học lại" if my_f > 0 else "Không có môn nợ (An toàn)", "#b91c1c" if my_f > 0 else "#0284c7")
            ]
        else:
            total_sv = len(self.students_info)
            total_records = len(self.grades_data)
            f_records = sum(1 for g in self.grades_data if self.evaluate_course_score(g["cc"], g["kt"], g["thi"])[1] == "F")
            sv_with_f = len(set(g["ma_sv"] for g in self.grades_data if self.evaluate_course_score(g["cc"], g["kt"], g["thi"])[1] == "F"))

            cards_data = [
                ("TỔNG SỐ SINH VIÊN QUẢN LÝ", f"{total_sv} Sinh viên", "4 SV chính: 241321111..2413211114", "#1e40af"),
                ("TỔNG KẾT QUẢ HỌC PHẦN", f"{total_records} Bản ghi", "5 Học phần / 14 Tín chỉ mỗi SV", "#0f766e"),
                ("SỐ LƯỢT HỌC PHẦN BỊ F (<3.0)", f"{f_records} Học phần F", "Ngưỡng trượt: Điểm TK < 3.00", "#b91c1c"),
                ("SINH VIÊN CẢNH BÁO HỌC VỤ", f"{sv_with_f} / {total_sv} Sinh viên", "Có ít nhất 1 học phần bị điểm F", "#c2410c")
            ]

        for title, big_val, sub_val, color in cards_data:
            c = tk.Frame(self.kpi_frame, bg="white", bd=0, highlightthickness=2, highlightbackground=color)
            c.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=6, ipadx=10, ipady=8)
            tk.Label(c, text=title, font=("Segoe UI", 9, "bold"), fg=color, bg="white").pack(anchor="w", padx=12, pady=(4, 0))
            tk.Label(c, text=big_val, font=("Segoe UI", 15, "bold"), fg="#0f172a", bg="white").pack(anchor="w", padx=12, pady=2)
            tk.Label(c, text=sub_val, font=("Segoe UI", 9), fg="#64748b", bg="white").pack(anchor="w", padx=12, pady=(0, 4))

    # ================= TAB 1: QUẢN LÝ & TRA CỨU ĐIỂM HỌC PHẦN =================
    def build_tab_manage(self):
        toolbar = tk.Frame(self.tab_manage, bg="#f8fafc", pady=10, padx=12)
        toolbar.pack(fill=tk.X)

        tk.Label(toolbar, text="Học kỳ:", font=("Segoe UI", 10, "bold"), bg="#f8fafc").pack(side=tk.LEFT)
        self.cb_filter_hk = ttk.Combobox(toolbar, values=["Tất cả", "HK1 (2025-2026)", "HK2 (2025-2026)"], state="readonly", width=16)
        self.cb_filter_hk.pack(side=tk.LEFT, padx=(5, 15))
        self.cb_filter_hk.current(0)

        tk.Label(toolbar, text="Học phần:", font=("Segoe UI", 10, "bold"), bg="#f8fafc").pack(side=tk.LEFT)
        hp_vals = ["Tất cả"] + [f"{k} - {v['ten']}" for k, v in self.courses_info.items()]
        self.cb_filter_hp = ttk.Combobox(toolbar, values=hp_vals, state="readonly", width=30)
        self.cb_filter_hp.pack(side=tk.LEFT, padx=(5, 15))
        self.cb_filter_hp.current(0)

        self.ent_search_sv = tk.Entry(toolbar, font=("Segoe UI", 10), width=18, relief=tk.SOLID, bd=1)
        if not self.is_student():
            tk.Label(toolbar, text="Tìm Mã/Tên SV:", font=("Segoe UI", 10, "bold"), bg="#f8fafc").pack(side=tk.LEFT)
            self.ent_search_sv.pack(side=tk.LEFT, padx=5)
        else:
            tk.Label(
                toolbar,
                text=f"🔒 Chế độ cá nhân: Chỉ hiển thị điểm của SV {self.current_user['name']} ({self.current_user['code']})",
                font=("Segoe UI", 9, "bold"), fg="#be123c", bg="#f8fafc"
            ).pack(side=tk.LEFT, padx=10)

        tk.Button(toolbar, text="🔍 Lọc dữ liệu", font=("Segoe UI", 9, "bold"), bg="#0284c7", fg="white", relief=tk.FLAT, padx=10, pady=4, command=self.refresh_manage_table).pack(side=tk.LEFT, padx=6)

        if self.current_user["username"] in ["admin", "gv", "pdt"]:
            tk.Button(toolbar, text="✏️ Nhập / Sửa điểm", font=("Segoe UI", 9, "bold"), bg="#16a34a", fg="white", relief=tk.FLAT, padx=10, pady=4, command=self.open_edit_grade_modal).pack(side=tk.RIGHT, padx=4)
            tk.Button(toolbar, text="➕ Thêm bản ghi mới", font=("Segoe UI", 9, "bold"), bg="#2563eb", fg="white", relief=tk.FLAT, padx=10, pady=4, command=self.open_add_grade_modal).pack(side=tk.RIGHT, padx=4)

        cols = ("ID", "Mã SV", "Họ và tên", "Lớp", "Tên học phần", "Tín chỉ", "Học kỳ", "CC (20%)", "KT (30%)", "Thi (50%)", "Điểm HP", "Điểm chữ", "Kết luận")
        self.tree_manage = ttk.Treeview(self.tab_manage, columns=cols, show="headings", height=16)

        widths = [40, 95, 165, 80, 200, 55, 115, 75, 75, 75, 80, 75, 150]
        for c, w in zip(cols, widths):
            self.tree_manage.heading(c, text=c)
            align = tk.W if c in ["Họ và tên", "Tên học phần", "Kết luận"] else tk.CENTER
            self.tree_manage.column(c, width=w, anchor=align)

        sc_y = ttk.Scrollbar(self.tab_manage, orient=tk.VERTICAL, command=self.tree_manage.yview)
        self.tree_manage.configure(yscroll=sc_y.set)
        self.tree_manage.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=(12, 0), pady=8)
        sc_y.pack(side=tk.RIGHT, fill=tk.Y, padx=(0, 12), pady=8)

        self.tree_manage.tag_configure("row_f", background="#fee2e2", foreground="#991b1b")
        self.tree_manage.tag_configure("row_a", background="#dcfce7", foreground="#166534")

        self.refresh_manage_table()

    def refresh_manage_table(self):
        for r in self.tree_manage.get_children():
            self.tree_manage.delete(r)

        hk_f = self.cb_filter_hk.get()
        hp_f = self.cb_filter_hp.get().split(" - ")[0] if " - " in self.cb_filter_hp.get() else "Tất cả"
        kw = self.ent_search_sv.get().strip().lower() if not self.is_student() else ""

        for g in self.grades_data:
            # RÀNG BUỘC BẢO MẬT ĐỘC LẬP: Sinh viên nào đăng nhập chỉ thấy điểm của chính Mã SV đó
            if self.is_student() and g["ma_sv"] != self.current_user["code"]:
                continue

            if hk_f != "Tất cả" and g["hk"] != hk_f:
                continue
            if hp_f != "Tất cả" and g["ma_hp"] != hp_f:
                continue

            sv = self.students_info.get(g["ma_sv"], {"ten": "Chưa rõ", "lop": "CNTT3K25"})
            hp = self.courses_info.get(g["ma_hp"], {"ten": "Chưa rõ", "tc": 3})

            if kw and (kw not in g["ma_sv"].lower() and kw not in sv["ten"].lower()):
                continue

            diem_hp, diem_chu, _, ket_luan = self.evaluate_course_score(g["cc"], g["kt"], g["thi"])
            tag = "row_f" if diem_chu == "F" else ("row_a" if diem_chu == "A" else "")

            self.tree_manage.insert(
                "", tk.END,
                values=(g["id"], g["ma_sv"], sv["ten"], sv["lop"], hp["ten"], hp["tc"], g["hk"],
                        f"{g['cc']:.2f}", f"{g['kt']:.2f}", f"{g['thi']:.2f}", f"{diem_hp:.2f}", diem_chu, ket_luan),
                tags=(tag,)
            )

    def open_edit_grade_modal(self):
        sel = self.tree_manage.selection()
        if not sel:
            messagebox.showwarning("Chưa chọn dòng", "Vui lòng nhấp chọn một dòng điểm trên bảng để chỉnh sửa!")
            return

        item_vals = self.tree_manage.item(sel[0], "values")
        rec_id = int(item_vals[0])
        record = next((x for x in self.grades_data if x["id"] == rec_id), None)
        if not record:
            return

        sv_name = self.students_info[record["ma_sv"]]["ten"]
        hp_name = self.courses_info[record["ma_hp"]]["ten"]

        dlg = tk.Toplevel(self.root)
        dlg.title(f"Cập nhật điểm - {sv_name}")
        dlg.geometry("430x380")
        dlg.configure(bg="white")
        dlg.grab_set()

        tk.Label(dlg, text="CẬP NHẬT ĐIỂM HỌC PHẦN", font=("Segoe UI", 12, "bold"), fg="#1e3a8a", bg="white").pack(pady=15)
        tk.Label(dlg, text=f"Sinh viên: {sv_name} (Mã SV: {record['ma_sv']})\nHọc phần: {hp_name}", font=("Segoe UI", 10, "bold"), fg="#334155", bg="white", justify=tk.LEFT).pack(pady=(0, 15))

        form = tk.Frame(dlg, bg="white", padx=35)
        form.pack(fill=tk.X)

        tk.Label(form, text="Điểm Chuyên cần (20%) [0 - 10]:", font=("Segoe UI", 10), bg="white").grid(row=0, column=0, sticky="w", pady=6)
        e_cc = tk.Entry(form, font=("Segoe UI", 11), width=12, relief=tk.SOLID, bd=1)
        e_cc.grid(row=0, column=1, pady=6, padx=10)
        e_cc.insert(0, str(record["cc"]))

        tk.Label(form, text="Điểm Kiểm tra (30%) [0 - 10]:", font=("Segoe UI", 10), bg="white").grid(row=1, column=0, sticky="w", pady=6)
        e_kt = tk.Entry(form, font=("Segoe UI", 11), width=12, relief=tk.SOLID, bd=1)
        e_kt.grid(row=1, column=1, pady=6, padx=10)
        e_kt.insert(0, str(record["kt"]))

        tk.Label(form, text="Điểm Thi kết thúc (50%) [0 - 10]:", font=("Segoe UI", 10), bg="white").grid(row=2, column=0, sticky="w", pady=6)
        e_thi = tk.Entry(form, font=("Segoe UI", 11), width=12, relief=tk.SOLID, bd=1)
        e_thi.grid(row=2, column=1, pady=6, padx=10)
        e_thi.insert(0, str(record["thi"]))

        def save_changes():
            try:
                cc_v = float(e_cc.get())
                kt_v = float(e_kt.get())
                thi_v = float(e_thi.get())
                if not (0 <= cc_v <= 10 and 0 <= kt_v <= 10 and 0 <= thi_v <= 10):
                    raise ValueError
            except ValueError:
                messagebox.showerror("Lỗi miền điểm (BR-01)", "Điểm chuyên cần, kiểm tra và thi phải là số thực từ 0.0 đến 10.0!")
                return

            old_score, old_let, _, _ = self.evaluate_course_score(record["cc"], record["kt"], record["thi"])
            record["cc"], record["kt"], record["thi"] = cc_v, kt_v, thi_v
            new_score, new_let, _, _ = self.evaluate_course_score(cc_v, kt_v, thi_v)

            self.audit_logs.append(
                f"[{datetime.now().strftime('%d/%m/%Y %H:%M:%S')}] {self.current_user['name']} sửa điểm {record['ma_sv']} ({hp_name}): {old_score} ({old_let}) -> {new_score} ({new_let})."
            )
            self.refresh_all_views()
            dlg.destroy()
            messagebox.showinfo("Thành công", f"Đã cập nhật điểm!\nĐiểm học phần mới: {new_score:.2f} -> Điểm chữ: {new_let}")

        tk.Button(dlg, text="💾 Lưu cập nhật", font=("Segoe UI", 10, "bold"), bg="#16a34a", fg="white", relief=tk.FLAT, padx=15, pady=7, command=save_changes).pack(pady=20)

    def open_add_grade_modal(self):
        dlg = tk.Toplevel(self.root)
        dlg.title("Thêm kết quả học phần mới")
        dlg.geometry("480x420")
        dlg.configure(bg="white")
        dlg.grab_set()

        tk.Label(dlg, text="THÊM BẢN GHI ĐIỂM HỌC PHẦN", font=("Segoe UI", 12, "bold"), fg="#1e3a8a", bg="white").pack(pady=15)
        form = tk.Frame(dlg, bg="white", padx=30)
        form.pack(fill=tk.X)

        tk.Label(form, text="Chọn Sinh viên:", font=("Segoe UI", 10), bg="white").grid(row=0, column=0, sticky="w", pady=6)
        sv_list = [f"{k} - {v['ten']}" for k, v in self.students_info.items()]
        cb_sv = ttk.Combobox(form, values=sv_list, state="readonly", width=28)
        cb_sv.grid(row=0, column=1, pady=6)
        cb_sv.current(0)

        tk.Label(form, text="Chọn Học phần:", font=("Segoe UI", 10), bg="white").grid(row=1, column=0, sticky="w", pady=6)
        hp_list = [f"{k} - {v['ten']}" for k, v in self.courses_info.items()]
        cb_hp = ttk.Combobox(form, values=hp_list, state="readonly", width=28)
        cb_hp.grid(row=1, column=1, pady=6)
        cb_hp.current(0)

        tk.Label(form, text="Học kỳ:", font=("Segoe UI", 10), bg="white").grid(row=2, column=0, sticky="w", pady=6)
        cb_hk = ttk.Combobox(form, values=["HK1 (2025-2026)", "HK2 (2025-2026)"], state="readonly", width=28)
        cb_hk.grid(row=2, column=1, pady=6)
        cb_hk.current(0)

        tk.Label(form, text="Điểm Chuyên cần (20%):", font=("Segoe UI", 10), bg="white").grid(row=3, column=0, sticky="w", pady=6)
        e_cc = tk.Entry(form, font=("Segoe UI", 10), width=15)
        e_cc.grid(row=3, column=1, sticky="w", pady=6)
        e_cc.insert(0, "2.5")

        tk.Label(form, text="Điểm Kiểm tra (30%):", font=("Segoe UI", 10), bg="white").grid(row=4, column=0, sticky="w", pady=6)
        e_kt = tk.Entry(form, font=("Segoe UI", 10), width=15)
        e_kt.grid(row=4, column=1, sticky="w", pady=6)
        e_kt.insert(0, "2.5")

        tk.Label(form, text="Điểm Thi (50%):", font=("Segoe UI", 10), bg="white").grid(row=5, column=0, sticky="w", pady=6)
        e_thi = tk.Entry(form, font=("Segoe UI", 10), width=15)
        e_thi.grid(row=5, column=1, sticky="w", pady=6)
        e_thi.insert(0, "2.5")

        def add_record():
            try:
                cc_v, kt_v, thi_v = float(e_cc.get()), float(e_kt.get()), float(e_thi.get())
                if not (0 <= cc_v <= 10 and 0 <= kt_v <= 10 and 0 <= thi_v <= 10):
                    raise ValueError
            except ValueError:
                messagebox.showerror("Lỗi", "Điểm nhập vào phải nằm trong khoảng 0.0 đến 10.0!")
                return

            ma_sv = cb_sv.get().split(" - ")[0]
            ma_hp = cb_hp.get().split(" - ")[0]
            hk = cb_hk.get()
            new_id = max(x["id"] for x in self.grades_data) + 1
            self.grades_data.append({"id": new_id, "ma_sv": ma_sv, "ma_hp": ma_hp, "hk": hk, "cc": cc_v, "kt": kt_v, "thi": thi_v, "locked": False})
            self.audit_logs.append(f"[{datetime.now().strftime('%d/%m/%Y %H:%M:%S')}] Thêm điểm mới cho {ma_sv} - Học phần {ma_hp} ({hk}).")
            self.refresh_all_views()
            dlg.destroy()
            messagebox.showinfo("Thành công", "Đã thêm bản ghi điểm học phần mới!")

        tk.Button(dlg, text="➕ Thêm vào hệ thống", font=("Segoe UI", 10, "bold"), bg="#2563eb", fg="white", relief=tk.FLAT, padx=15, pady=7, command=add_record).pack(pady=18)

    # ================= TAB 2: TỔNG HỢP ĐIỂM TRUNG BÌNH HỌC KỲ (GPA) =================
    def build_tab_gpa(self):
        top = tk.Frame(self.tab_gpa, bg="#f8fafc", pady=10, padx=12)
        top.pack(fill=tk.X)
        title_lbl = (
            f"📌 KẾT QUẢ ĐIỂM TRUNG BÌNH HỌC KỲ CÁ NHÂN - SINH VIÊN: {self.current_user['name'].upper()} (MÃ SV: {self.current_user['code']})"
            if self.is_student() else
            "📌 BẢNG TỔNG HỢP ĐIỂM TRUNG BÌNH HỌC KỲ THEO TÍN CHỈ (Kể cả học phần F - Ngưỡng F < 3.00)"
        )
        tk.Label(top, text=title_lbl, font=("Segoe UI", 11, "bold"), fg="#0f172a", bg="#f8fafc").pack(side=tk.LEFT)

        cols = ("STT", "Mã SV", "Họ và tên", "Lớp", "Tổng Tín Chỉ", "ĐTB Học Kỳ (Hệ 10)", "Số Môn Đạt (>=3.0)", "Số Môn F (<3.0)", "Đánh Giá Học Vụ")
        self.tree_gpa = ttk.Treeview(self.tab_gpa, columns=cols, show="headings", height=15)
        widths = [50, 100, 180, 100, 100, 140, 130, 130, 220]
        for c, w in zip(cols, widths):
            self.tree_gpa.heading(c, text=c)
            self.tree_gpa.column(c, width=w, anchor=tk.W if c in ["Họ và tên", "Đánh Giá Học Vụ"] else tk.CENTER)

        self.tree_gpa.pack(fill=tk.BOTH, expand=True, padx=12, pady=10)
        self.tree_gpa.tag_configure("warn_f", background="#fff1f2", foreground="#be123c")
        self.tree_gpa.tag_configure("pass_all", background="#f0fdf4", foreground="#15803d")
        self.refresh_gpa_table()

    def refresh_gpa_table(self):
        for r in self.tree_gpa.get_children():
            self.tree_gpa.delete(r)

        stt = 1
        for ma_sv, sv_info in self.students_info.items():
            # Nếu là Sinh viên đăng nhập, chỉ hiển thị đúng dòng ĐTB của chính mình
            if self.is_student() and ma_sv != self.current_user["code"]:
                continue

            recs = [g for g in self.grades_data if g["ma_sv"] == ma_sv]
            if not recs:
                continue

            sum_weighted = 0.0
            sum_tc = 0
            pass_cnt = 0
            f_cnt = 0

            for g in recs:
                tc = self.courses_info[g["ma_hp"]]["tc"]
                raw_hp = g["cc"] * 0.2 + g["kt"] * 0.3 + g["thi"] * 0.5
                sum_weighted += raw_hp * tc
                sum_tc += tc
                if raw_hp < 3.0:
                    f_cnt += 1
                else:
                    pass_cnt += 1

            gpa = round(sum_weighted / sum_tc, 2) if sum_tc > 0 else 0.0
            status = f"Còn {f_cnt} học phần không đạt (F)" if f_cnt > 0 else "Qua đủ 5 học phần"
            tag = "warn_f" if f_cnt > 0 else "pass_all"

            self.tree_gpa.insert(
                "", tk.END,
                values=(stt, ma_sv, sv_info["ten"], sv_info["lop"], sum_tc, f"{gpa:.2f}", pass_cnt, f_cnt, status),
                tags=(tag,)
            )
            stt += 1

    # ================= TAB 3: TRÍCH XUẤT DANH SÁCH SINH VIÊN BỊ ĐIỂM F (< 3.0) =================
    def build_tab_extract_f(self):
        banner = tk.Frame(self.tab_extract_f, bg="#fef2f2", pady=12, padx=15, highlightthickness=1, highlightbackground="#f87171")
        banner.pack(fill=tk.X, padx=12, pady=10)

        banner_title = (
            f"🚨 TRA CỨU HỌC PHẦN BỊ ĐIỂM F (< 3.00) CỦA CÁ NHÂN: {self.current_user['name'].upper()} ({self.current_user['code']})"
            if self.is_student() else
            "🚨 CHỨC NĂNG TRỌNG TÂM: TRÍCH XUẤT DANH SÁCH SINH VIÊN BỊ ĐIỂM F (ĐIỂM HỌC PHẦN < 3.00)"
        )
        tk.Label(banner, text=banner_title, font=("Segoe UI", 11, "bold"), fg="#b91c1c", bg="#fef2f2").pack(anchor="w")
        tk.Label(
            banner, text="Lọc chính xác theo từng học kỳ các học phần có điểm tổng kết dưới 3.00 trên thang 10 để cảnh báo học vụ và lập danh sách học lại.",
            font=("Segoe UI", 9), fg="#7f1d1d", bg="#fef2f2"
        ).pack(anchor="w", pady=(2, 0))

        ctrl_bar = tk.Frame(self.tab_extract_f, bg="white", padx=12, pady=4)
        ctrl_bar.pack(fill=tk.X)

        tk.Label(ctrl_bar, text="Chọn Học kỳ:", font=("Segoe UI", 10, "bold"), bg="white").pack(side=tk.LEFT)
        self.cb_f_hk = ttk.Combobox(ctrl_bar, values=["HK1 (2025-2026)", "HK2 (2025-2026)", "Tất cả"], state="readonly", width=18)
        self.cb_f_hk.pack(side=tk.LEFT, padx=8)
        self.cb_f_hk.current(0)

        tk.Button(
            ctrl_bar, text="🔍 Lọc Danh Sách Điểm F", font=("Segoe UI", 10, "bold"),
            bg="#dc2626", fg="white", relief=tk.FLAT, padx=12, pady=5, command=self.refresh_f_table
        ).pack(side=tk.LEFT, padx=5)

        tk.Button(
            ctrl_bar, text="📥 Xuất Báo Cáo Điểm F (CSV / Excel)", font=("Segoe UI", 10, "bold"),
            bg="#0f766e", fg="white", relief=tk.FLAT, padx=14, pady=5, command=self.export_f_to_csv
        ).pack(side=tk.RIGHT, padx=5)

        self.lbl_f_summary = tk.Label(ctrl_bar, text="", font=("Segoe UI", 10, "bold"), fg="#b91c1c", bg="white")
        self.lbl_f_summary.pack(side=tk.RIGHT, padx=15)

        cols_f = ("Mã F", "Mã SV", "Họ và tên", "Lớp", "Học kỳ", "Tên học phần bị F", "Số TC", "CC", "KT", "Thi", "Điểm HP", "Điểm chữ", "Hướng xử lý học vụ")
        self.tree_f = ttk.Treeview(self.tab_extract_f, columns=cols_f, show="headings", height=13)
        widths_f = [55, 95, 165, 80, 115, 195, 55, 55, 55, 55, 75, 70, 175]
        for c, w in zip(cols_f, widths_f):
            self.tree_f.heading(c, text=c)
            self.tree_f.column(c, width=w, anchor=tk.W if c in ["Họ và tên", "Tên học phần bị F", "Hướng xử lý học vụ"] else tk.CENTER)

        self.tree_f.pack(fill=tk.BOTH, expand=True, padx=12, pady=8)
        self.tree_f.tag_configure("f_highlight", background="#ffe4e6", foreground="#881337")
        self.refresh_f_table()

    def refresh_f_table(self):
        for r in self.tree_f.get_children():
            self.tree_f.delete(r)

        hk_sel = self.cb_f_hk.get()
        f_list = []
        unique_sv = set()

        idx = 1
        for g in self.grades_data:
            if hk_sel != "Tất cả" and g["hk"] != hk_sel:
                continue
            if self.is_student() and g["ma_sv"] != self.current_user["code"]:
                continue

            diem_hp, diem_chu, _, _ = self.evaluate_course_score(g["cc"], g["kt"], g["thi"])
            if diem_chu == "F":
                sv = self.students_info[g["ma_sv"]]
                hp = self.courses_info[g["ma_hp"]]
                unique_sv.add(g["ma_sv"])
                row_tuple = (
                    f"F{idx:02d}", g["ma_sv"], sv["ten"], sv["lop"], g["hk"], hp["ten"], hp["tc"],
                    f"{g['cc']:.2f}", f"{g['kt']:.2f}", f"{g['thi']:.2f}", f"{diem_hp:.2f}", "F", "Bắt buộc đăng ký học lại"
                )
                f_list.append(row_tuple)
                self.tree_f.insert("", tk.END, values=row_tuple, tags=("f_highlight",))
                idx += 1

        if self.is_student():
            self.lbl_f_summary.config(text=f"Cá nhân {self.current_user['name']}: {len(f_list)} học phần bị điểm F")
        else:
            self.lbl_f_summary.config(text=f"Tổng kết: {len(f_list)} lượt học phần F / {len(unique_sv)} sinh viên có điểm F")
        return f_list

    def export_f_to_csv(self):
        f_rows = self.refresh_f_table()
        if not f_rows:
            messagebox.showinfo("Thông báo", "Không có học phần điểm F nào để xuất báo cáo!")
            return

        default_name = f"Diem_F_{self.current_user['code']}.csv" if self.is_student() else "Danh_Sach_Sinh_Vien_Diem_F_HPUNI.csv"
        filepath = filedialog.asksaveasfilename(
            defaultextension=".csv",
            filetypes=[("CSV Excel File", "*.csv")],
            initialfile=default_name,
            title="Lưu báo cáo danh sách điểm F"
        )
        if filepath:
            with open(filepath, mode="w", newline="", encoding="utf-8-sig") as f:
                writer = csv.writer(f)
                writer.writerow(["TRƯỜNG ĐẠI HỌC HẢI PHÒNG (HPUNI) - BÁO CÁO TRÍCH XUẤT HỌC PHẦN BỊ ĐIỂM F (< 3.00)"])
                writer.writerow(["Mã F", "Mã SV", "Họ và tên", "Lớp", "Học kỳ", "Tên học phần bị F", "Số TC", "Chuyên cần", "Kiểm tra", "Điểm thi", "Điểm học phần", "Điểm chữ", "Ghi chú"])
                for r in f_rows:
                    writer.writerow(r)
            self.audit_logs.append(f"[{datetime.now().strftime('%d/%m/%Y %H:%M:%S')}] {self.current_user['name']} xuất file CSV điểm F ({len(f_rows)} dòng).")
            messagebox.showinfo("Xuất báo cáo thành công", f"Đã trích xuất {len(f_rows)} lượt học phần F ra tệp:\n{filepath}")

    # ================= TAB 4: DANH SÁCH TÀI KHOẢN SINH VIÊN ĐỘC LẬP & NHẬT KÝ =================
    def build_tab_project(self):
        container = tk.Frame(self.tab_project, bg="white", padx=20, pady=15)
        container.pack(fill=tk.BOTH, expand=True)

        left_box = tk.LabelFrame(container, text=" 📘 Danh Sách Tài Khoản Sinh Viên Độc Lập & Quy Tắc Nghiệp Vụ ", font=("Segoe UI", 10, "bold"), fg="#1e3a8a", bg="white", padx=15, pady=12)
        left_box.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=(0, 10))

        doc_text = (
            "1. DANH SÁCH 4 TÀI KHOẢN SINH VIÊN ĐỘC LẬP (Mật khẩu chung: 1):\n"
            "   • Tài khoản 1: 241321111   | Họ tên: Trần Tăng Vinh      | ĐTB: 7.45 (0 môn F)\n"
            "   • Tài khoản 2: 241321112   | Họ tên: Nguyễn Thanh Thưởng | ĐTB: 7.46 (0 môn F)\n"
            "   • Tài khoản 3: 2413211113  | Họ tên: Lê Xuân Hiếu        | ĐTB: 5.31 (1 môn F: Nhập môn thuật toán - 2.20)\n"
            "   • Tài khoản 4: 2413211114  | Họ tên: Trần Thành Khang    | ĐTB: 6.42 (1 môn F: Toán học rời rạc - 2.50)\n"
            "   *(Lưu ý: Mỗi sinh viên đăng nhập chỉ xem được duy nhất điểm của cá nhân mình!)*\n\n"
            "2. TÀI KHOẢN QUẢN TRỊ & GIẢNG VIÊN (Mật khẩu chung: 1):\n"
            "   • admin (Quản trị viên) | pdt (Phòng Đào tạo) | gv (Giảng viên) | cvht (Cố vấn học tập)\n\n"
            "3. CÔNG THỨC TÍNH ĐIỂM & QUY ĐỔI ĐIỂM CHỮ (BR-01 -> BR-06):\n"
            "   • Điểm học phần (HP) = 0.20 × Chuyên cần + 0.30 × Kiểm tra + 0.50 × Điểm thi\n"
            "   • Điểm trung bình học kỳ = Σ(Điểm HP × Tín chỉ) / Σ(Tín chỉ) (Gồm cả môn F)\n"
            "   • Thang điểm chữ quy đổi:\n"
            "       - Điểm A : Từ 9.00 đến 10.00 (Xuất sắc / Giỏi)\n"
            "       - Điểm B : Từ 7.00 đến dưới 9.00 (Khá)\n"
            "       - Điểm C : Từ 5.00 đến dưới 7.00 (Trung bình)\n"
            "       - Điểm D : Từ 3.00 đến dưới 5.00 (Trung bình yếu - Đạt)\n"
            "       - Điểm F : Dưới 3.00 (< 3.00) (Kém - Trượt môn / Nợ môn)"
        )
        tk.Label(left_box, text=doc_text, font=("Segoe UI", 9), fg="#1e293b", bg="white", justify=tk.LEFT).pack(anchor="nw")

        right_box = tk.LabelFrame(container, text=" 🛡️ Nhật Ký Thao Tác & Truy Vết Hệ Thống (Audit Log) ", font=("Segoe UI", 10, "bold"), fg="#0f766e", bg="white", padx=12, pady=12)
        right_box.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)

        self.txt_logs = tk.Text(right_box, font=("Consolas", 9), bg="#0f172a", fg="#38bdf8", wrap=tk.WORD)
        self.txt_logs.pack(fill=tk.BOTH, expand=True)
        self.refresh_logs_view()

    def refresh_logs_view(self):
        if hasattr(self, "txt_logs"):
            self.txt_logs.delete("1.0", tk.END)
            for line in self.audit_logs:
                self.txt_logs.insert(tk.END, line + "\n\n")

    def refresh_all_views(self):
        self.render_kpi_cards()
        self.refresh_manage_table()
        self.refresh_gpa_table()
        self.refresh_f_table()
        self.refresh_logs_view()

if __name__ == "__main__":
    root = tk.Tk()
    app = HPUNIGradeSystem(root)
    root.mainloop()
](https://github.com/vinhboss1807-ai/KNPM-BTL-1.git)
