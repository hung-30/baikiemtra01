using System;
using System.Collections.Generic;
using System.Linq;

namespace AutoSpeedLogistics
{
    // ==========================================
    // 1. CẤU TRÚC CÁC LỚP (CLASSES)
    // ==========================================
    public abstract class PhuongTien
    {
        private string _maPT;
        private string _tenHang;
        private int _namSanXuat;
        private decimal _giaGoc;

        public string MaPT
        {
            get => _maPT;
            set => _maPT = string.IsNullOrWhiteSpace(value) ? "PT000" : value.Trim();
        }

        public string TenHang
        {
            get => _tenHang;
            set
            {
                if (string.IsNullOrWhiteSpace(value))
                    throw new ArgumentException("Tên hãng không được để trống!");
                _tenHang = value.Trim();
            }
        }

        public int NamSanXuat
        {
            get => _namSanXuat;
            set
            {
                if (value < 1900 || value > DateTime.Now.Year)
                    throw new ArgumentException("Năm sản xuất không hợp lệ!");
                _namSanXuat = value;
            }
        }

        public decimal GiaGoc
        {
            get => _giaGoc;
            set
            {
                if (value <= 0)
                    throw new ArgumentException("Giá gốc phải lớn hơn 0!");
                _giaGoc = value;
            }
        }

        public PhuongTien(string maPT, string tenHang, int namSanXuat, decimal giaGoc)
        {
            MaPT = maPT;
            TenHang = tenHang;
            NamSanXuat = namSanXuat;
            GiaGoc = giaGoc;
        }

        public abstract decimal TinhGiaLanBanh();

        public virtual string GetInfo()
        {
            return $"Mã: {MaPT} | Hãng: {TenHang} | Năm SX: {NamSanXuat} | Giá gốc: {GiaGoc:N0} VNĐ";
        }
    }

    public class OTo : PhuongTien
    {
        public int SoChoNgoi { get; set; }
        public double DungTichDongCo { get; set; }

        public OTo(string maPT, string tenHang, int namSanXuat, decimal giaGoc, int soChoNgoi, double dungTichDongCo)
            : base(maPT, tenHang, namSanXuat, giaGoc)
        {
            if (soChoNgoi <= 0) throw new ArgumentException("Số chỗ ngồi phải > 0");
            if (dungTichDongCo <= 0) throw new ArgumentException("Dung tích động cơ phải > 0");
            SoChoNgoi = soChoNgoi;
            DungTichDongCo = dungTichDongCo;
        }

        public override decimal TinhGiaLanBanh()
        {
            if (SoChoNgoi <= 9)
                return GiaGoc + (GiaGoc * 0.12m) + (GiaGoc * 0.30m);
            else
                return GiaGoc + (GiaGoc * 0.10m);
        }

        public override string GetInfo()
        {
            return base.GetInfo() + $" | Chỗ ngồi: {SoChoNgoi} | Động cơ: {DungTichDongCo}L";
        }
    }

    public class XeMay : PhuongTien
    {
        public int DungTichXylanh { get; set; }

        public XeMay(string maPT, string tenHang, int namSanXuat, decimal giaGoc, int dungTichXylanh)
            : base(maPT, tenHang, namSanXuat, giaGoc)
        {
            if (dungTichXylanh <= 0) throw new ArgumentException("Dung tích xylanh phải > 0");
            DungTichXylanh = dungTichXylanh;
        }

        public override decimal TinhGiaLanBanh()
        {
            if (DungTichXylanh < 175)
                return GiaGoc + (GiaGoc * 0.02m);
            else
                return GiaGoc + (GiaGoc * 0.05m);
        }

        public override string GetInfo()
        {
            return base.GetInfo() + $" | Dung tích: {DungTichXylanh}cc";
        }
    }

    public class QuanLyPhuongTien
    {
        private List<PhuongTien> _danhSachPT = new List<PhuongTien>();

        public void AddPhuongTien(PhuongTien pt)
        {
            _danhSachPT.Add(pt);
        }

        public void DisplayAll()
        {
            foreach (var pt in _danhSachPT)
            {
                Console.WriteLine($"- {pt.GetInfo()} => Giá lăn bánh: {pt.TinhGiaLanBanh():N0} VNĐ");
            }
        }

        public PhuongTien FindMaxGiaLanBanh()
        {
            return _danhSachPT.OrderByDescending(pt => pt.TinhGiaLanBanh()).FirstOrDefault();
        }
    }

    // ==========================================
    // 2. CHƯƠNG TRÌNH CHÍNH (NHẬP LIỆU BÀN PHÍM)
    // ==========================================
    class Program
    {
        static void Main(string[] args)
        {
            Console.OutputEncoding = System.Text.Encoding.UTF8;
            QuanLyPhuongTien qlpt = new QuanLyPhuongTien();

            Console.WriteLine("=== HỆ THỐNG QUẢN LÝ PHƯƠNG TIỆN ===\n");

            // ---------------------------------------------------------
            // TC01 & TC02: Nhập liệu Ô tô
            // ---------------------------------------------------------
            Console.WriteLine(">> NHẬP THÔNG TIN Ô TÔ");
            OTo oto = null;
            while (oto == null)
            {
                try
                {
                    Console.Write("Nhập mã PT: ");
                    string ma = Console.ReadLine();

                    Console.Write("Nhập tên hãng: ");
                    string hang = Console.ReadLine();

                    Console.Write("Nhập năm sản xuất: ");
                    int nam = int.Parse(Console.ReadLine());

                    Console.Write("Nhập giá gốc VNĐ: ");
                    decimal gia = decimal.Parse(Console.ReadLine());

                    Console.Write("Nhập số chỗ ngồi: ");
                    int cho = int.Parse(Console.ReadLine());

                    Console.Write("Nhập dung tích động cơ (Lít): ");
                    double dongCo = double.Parse(Console.ReadLine());

                    // Nếu gõ năm 1850, dòng dưới sẽ nhảy thẳng xuống catch Exception
                    oto = new OTo(ma, hang, nam, gia, cho, dongCo);

                    Console.WriteLine($"\n=> Thành công! Giá lăn bánh Ô tô tính được: {oto.TinhGiaLanBanh():N0} VNĐ\n");
                    qlpt.AddPhuongTien(oto);
                }
                catch (ArgumentException ex)
                {
                    Console.ForegroundColor = ConsoleColor.Red;
                    Console.WriteLine($"[LỖI VALIDATION - TC01]: {ex.Message} Vui lòng nhập lại từ đầu!\n");
                    Console.ResetColor();
                }
            }

            // ---------------------------------------------------------
            // TC03: Nhập liệu Xe máy
            // ---------------------------------------------------------
            Console.WriteLine(">> NHẬP THÔNG TIN XE MÁY");
            XeMay xeMay = null;
            while (xeMay == null)
            {
                try
                {
                    Console.Write("Nhập mã PT: ");
                    string ma = Console.ReadLine();

                    Console.Write("Nhập tên hãng: ");
                    string hang = Console.ReadLine();

                    Console.Write("Nhập năm sản xuất: ");
                    int nam = int.Parse(Console.ReadLine());

                    Console.Write("Nhập giá gốc VNĐ: ");
                    decimal gia = decimal.Parse(Console.ReadLine());

                    Console.Write("Nhập dung tích xylanh cc: ");
                    int cc = int.Parse(Console.ReadLine());

                    xeMay = new XeMay(ma, hang, nam, gia, cc);

                    Console.WriteLine($"\n=> Thành công! Giá lăn bánh Xe máy tính được: {xeMay.TinhGiaLanBanh():N0} VNĐ\n");
                    qlpt.AddPhuongTien(xeMay);
                }
                catch (ArgumentException ex)
                {
                    Console.ForegroundColor = ConsoleColor.Red;
                    Console.WriteLine($"[LỖI]: {ex.Message} Vui lòng nhập lại!\n");
                    Console.ResetColor();
                }
            }

            // ---------------------------------------------------------
            // TC04: Kiểm tra Đa hình (In danh sách)
            // ---------------------------------------------------------
            Console.WriteLine(">> KIỂM TRA ĐA HÌNH TRÊN LIST (Gọi TinhGiaLanBanh tự động)");
            qlpt.DisplayAll();
            Console.WriteLine();

            // ---------------------------------------------------------
            // TC05: Tìm xe giá lăn bánh cao nhất
            // ---------------------------------------------------------
            Console.WriteLine(">> KẾT QUẢ TÌM GIÁ LĂN BÁNH CAO NHẤT");
            PhuongTien maxPT = qlpt.FindMaxGiaLanBanh();
            if (maxPT != null)
            {
                Console.WriteLine($"Phương tiện đắt nhất: {maxPT.GetInfo()}");
                Console.WriteLine($"Mức giá lăn bánh: {maxPT.TinhGiaLanBanh():N0} VNĐ");
            }

            Console.ReadLine();
        }
    }
}





<img width="1479" height="400" alt="image" src="https://github.com/user-attachments/assets/5216667d-9c8f-448c-acb0-70fba6e5f928" />





<img width="1476" height="726" alt="image" src="https://github.com/user-attachments/assets/3431ebdb-4a6c-474f-b6f5-e9ab62be9024" />

