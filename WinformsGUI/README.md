using System;
using System.ComponentModel;
using System.IO;
using System.Linq;
using System.Text;
using System.Windows.Forms;

namespace baikiemtraso01
{
    public partial class Form1 : Form
    {
        private BindingList<Product> _productList = new BindingList<Product>();
        private BindingSource _bindingSource = new BindingSource();
        private BindingList<Category> _categoryList = new BindingList<Category>();

        public Form1()
        {
            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            InitCategories();
            InitDataBinding();
            RegisterEvents();
            UpdateStatus();
        }

        private void InitCategories()
        {
            _categoryList = new BindingList<Category>
            {
                new Category { Id = 1, Name = "Điện thoại" },
                new Category { Id = 2, Name = "Laptop" },
                new Category { Id = 3, Name = "Phụ kiện" }
            };

            cboCategory.DataSource = _categoryList;
            cboCategory.DisplayMember = "Name";
            cboCategory.ValueMember = "Id";
        }

        private void InitDataBinding()
        {
            // Dữ liệu mẫu ban đầu
            _productList.Add(new Product { Id = "SP01", Name = "iPhone 15 Pro", Price = 28000000, Quantity = 10, CategoryId = 1 });
            _productList.Add(new Product { Id = "SP02", Name = "MacBook Air M2", Price = 24500000, Quantity = 5, CategoryId = 2 });

            _bindingSource.DataSource = _productList;
            dgvProducts.AutoGenerateColumns = false;
            dgvProducts.DataSource = _bindingSource;
        }

        private void RegisterEvents()
        {
            // Gán sự kiện cho các nút chưa có trong Designer
            button3.Click += button3_Click; // Nút Sửa
            button4.Click += button4_Click; // Nút Xóa
            btnChooseImage.Click += btnChooseImage_Click;
            dgvProducts.SelectionChanged += dgvProducts_SelectionChanged;
            exportCSVToolStripMenuItem.Click += exportCSVToolStripMenuItem_Click;
            exitToolStripMenuItem.Click += exitToolStripMenuItem_Click;
        }

        // --- XỬ LÝ ĐỔ DỮ LIỆU TỪ BẢNG LÊN TEXTBOX ---
        private void dgvProducts_SelectionChanged(object sender, EventArgs e)
        {
            if (dgvProducts.CurrentRow != null && dgvProducts.CurrentRow.DataBoundItem is Product selectedProduct)
            {
                txtProductId.Text = selectedProduct.Id;
                txtProductName.Text = selectedProduct.Name;
                txtUnitPrice.Text = selectedProduct.Price.ToString("G0");
                txtQuantity.Text = selectedProduct.Quantity.ToString();
                cboCategory.SelectedValue = selectedProduct.CategoryId;

                if (!string.IsNullOrEmpty(selectedProduct.ImagePath) && File.Exists(selectedProduct.ImagePath))
                {
                    picAvatar.ImageLocation = selectedProduct.ImagePath;
                }
                else
                {
                    picAvatar.Image = null;
                }
            }
        }

        // --- HÀM KIỂM TRA ĐIỀU KIỆN (VALIDATION) ---
        private bool ValidateInput()
        {
            errorProvider1.Clear();
            bool isValid = true;

            if (string.IsNullOrWhiteSpace(txtProductId.Text))
            {
                errorProvider1.SetError(txtProductId, "Mã sản phẩm không được để trống!");
                isValid = false;
            }

            if (string.IsNullOrWhiteSpace(txtProductName.Text))
            {
                errorProvider1.SetError(txtProductName, "Tên sản phẩm không được để trống!");
                isValid = false;
            }

            if (!decimal.TryParse(txtUnitPrice.Text, out decimal price) || price <= 0)
            {
                errorProvider1.SetError(txtUnitPrice, "Đơn giá phải là số > 0!");
                isValid = false;
            }

            if (!int.TryParse(txtQuantity.Text, out int qty) || qty < 0)
            {
                errorProvider1.SetError(txtQuantity, "Số lượng phải là số nguyên >= 0!");
                isValid = false;
            }

            return isValid;
        }

        // --- NÚT THÊM (button2) ---
        private void button2_Click(object sender, EventArgs e)
        {
            if (!ValidateInput()) return;

            string id = txtProductId.Text.Trim();
            if (_productList.Any(p => p.Id.Equals(id, StringComparison.OrdinalIgnoreCase)))
            {
                errorProvider1.SetError(txtProductId, "Mã sản phẩm đã tồn tại!");
                return;
            }

            Product newProd = new Product
            {
                Id = id,
                Name = txtProductName.Text.Trim(),
                Price = decimal.Parse(txtUnitPrice.Text),
                Quantity = int.Parse(txtQuantity.Text),
                CategoryId = Convert.ToInt32(cboCategory.SelectedValue),
                ImagePath = picAvatar.ImageLocation ?? string.Empty
            };

            _productList.Add(newProd);
            UpdateStatus();
            MessageBox.Show("Thêm sản phẩm thành công!", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }

        // --- NÚT SỬA (button3) ---
        private void button3_Click(object sender, EventArgs e)
        {
            if (dgvProducts.CurrentRow?.DataBoundItem is Product selectedProduct)
            {
                if (!ValidateInput()) return;

                selectedProduct.Name = txtProductName.Text.Trim();
                selectedProduct.Price = decimal.Parse(txtUnitPrice.Text);
                selectedProduct.Quantity = int.Parse(txtQuantity.Text);
                selectedProduct.CategoryId = Convert.ToInt32(cboCategory.SelectedValue);
                selectedProduct.ImagePath = picAvatar.ImageLocation ?? string.Empty;

                _bindingSource.ResetBindings(false);
                MessageBox.Show("Cập nhật thành công!", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
        }

        // --- NÚT XÓA (button4) ---
        private void button4_Click(object sender, EventArgs e)
        {
            if (dgvProducts.CurrentRow?.DataBoundItem is Product selectedProduct)
            {
                var confirm = MessageBox.Show($"Bạn có chắc muốn xóa {selectedProduct.Name}?",
                    "Xác nhận", MessageBoxButtons.YesNo, MessageBoxIcon.Question);

                if (confirm == DialogResult.Yes)
                {
                    _productList.Remove(selectedProduct);
                    UpdateStatus();
                }
            }
        }

        // --- NÚT CHỌN ẢNH ---
        private void btnChooseImage_Click(object sender, EventArgs e)
        {
            using (OpenFileDialog ofd = new OpenFileDialog())
            {
                ofd.Filter = "Hình ảnh (*.jpg; *.png; *.jpeg)|*.jpg;*.png;*.jpeg";
                if (ofd.ShowDialog() == DialogResult.OK)
                {
                    picAvatar.ImageLocation = ofd.FileName;
                }
            }
        }

        // --- XUẤT CSV ---
        private void exportCSVToolStripMenuItem_Click(object sender, EventArgs e)
        {
            using (SaveFileDialog sfd = new SaveFileDialog())
            {
                sfd.Filter = "CSV File (*.csv)|*.csv";
                sfd.FileName = "DanhSach.csv";
                if (sfd.ShowDialog() == DialogResult.OK)
                {
                    StringBuilder sb = new StringBuilder();
                    sb.AppendLine("Mã SP,Tên SP,Đơn Giá,Số Lượng,Mã Danh Mục");
                    foreach (var p in _productList)
                    {
                        sb.AppendLine($"\"{p.Id}\",\"{p.Name}\",{p.Price},{p.Quantity},{p.CategoryId}");
                    }
                    File.WriteAllText(sfd.FileName, sb.ToString(), Encoding.UTF8);
                    MessageBox.Show("Xuất file thành công!", "Thông báo");
                }
            }
        }

        // --- THOÁT ---
        private void exitToolStripMenuItem_Click(object sender, EventArgs e)
        {
            Application.Exit();
        }

        private void UpdateStatus()
        {
            lblStatus.Text = $"Tổng số sản phẩm: {_productList.Count}";
        }

        // =========================================================================
        // CÁC HÀM TRỐNG ĐỂ CHỐNG LỖI DO VÔ TÌNH CLICK ĐÚP TRONG GIAO DIỆN DESIGNER
        // =========================================================================
        private void tableLayoutPanel1_Paint(object sender, PaintEventArgs e) { }
        private void label1_Click(object sender, EventArgs e) { }
        private void label3_Click(object sender, EventArgs e) { }
        private void dgvProducts_CellContentClick(object sender, DataGridViewCellEventArgs e) { }
    }

    // --- LỚP DỮ LIỆU ĐI KÈM CẦN THIẾT ---
    public class Category
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
    }

    public class Product
    {
        public string Id { get; set; } = string.Empty;
        public string Name { get; set; } = string.Empty;
        public decimal Price { get; set; }
        public int Quantity { get; set; }
        public int CategoryId { get; set; }
        public string ImagePath { get; set; } = string.Empty;
    }
}


<img width="1075" height="600" alt="image" src="https://github.com/user-attachments/assets/2acd7e56-8b7b-46ca-899c-5a8e2d656425" />



<img width="1916" height="1019" alt="image" src="https://github.com/user-attachments/assets/9ade41b4-ea38-460e-8a30-0de70054a47f" />


