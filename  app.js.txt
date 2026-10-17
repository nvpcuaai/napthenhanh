// === HÀM HỖ TRỢ ===
const $ = s => document.querySelector(s);
const toast = (msg) => { const t=$('#toast'); t.textContent=msg; t.classList.add('show'); setTimeout(()=>t.classList.remove('show'),2500); };
const money = n => new Intl.NumberFormat('vi-VN').format(Number(n)||0)+' ₫';
const esc = s => String(s||'').replace(/[&<>"']/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));

// === DỮ LIỆU — LƯU TRỮ TRÊN TRÌNH DUYỆT ===
const defaultSettings = {
  siteName: 'Vườn Xanh',
  greeting: 'CHÀO MỪNG',
  announcement: 'Chào mừng bạn đến với cửa hàng của tôi',
  contactShort: '0988xxxxxx',
  contactFull: '📞 Điện thoại: 0988xxxxxx\n💬 Zalo/SĐT: cùng số điện thoại\n📍 Địa chỉ: Quận 1, TP.HCM'
};
const defaultProducts = [
  { name: 'Sản phẩm mẫu 1', price: 150000, image: 'https://images.unsplash.com/photo-1597055181300-e363a91d2393?w=400', desc: 'Đây là sản phẩm mẫu, bạn có thể xóa hoặc thay đổi!' },
  { name: 'Sản phẩm mẫu 2', price: 280000, image: 'https://images.unsplash.com/photo-1593485215068-70d58206c4e1?w=400', desc: 'Mô tả ngắn về sản phẩm của bạn' }
];

// === QUẢN LÝ DỮ LIỆU ===
function getSettings(){ const s=localStorage.getItem('siteSettings'); return s ? JSON.parse(s) : {...defaultSettings}; }
function saveSettings(data){ localStorage.setItem('siteSettings',JSON.stringify(data)); render(); toast('✅ Đã lưu thông tin!'); }
function getProducts(){ const p=localStorage.getItem('siteProducts'); return p ? JSON.parse(p) : [...defaultProducts]; }
function saveProducts(list){ localStorage.setItem('siteProducts',JSON.stringify(list)); render(); toast('✅ Đã cập nhật sản phẩm!'); }

// === HIỂN THỊ DỮ LIỆU ===
function render(){
  const s = getSettings();
  const p = getProducts();
  
  $('#brandName').textContent = s.siteName;
  $('#footerName').textContent = s.siteName;
  $('#greeting').textContent = s.greeting;
  $('#announcement').textContent = s.announcement;
  $('#statName').textContent = s.siteName;
  $('#contactInfo').textContent = s.contactShort;
  $('#contactText').innerHTML = s.contactFull.replace(/\n/g,'<br>');
  $('#productCount').textContent = p.length;
  
  if(p.length === 0){
    $('#productGrid').innerHTML = `<div class="empty-state"><p>📭 Chưa có sản phẩm nào</p><p>Bấm nút <strong>"+ Thêm sản phẩm"</strong> để bắt đầu nhé!</p></div>`;
  }else{
    $('#productGrid').innerHTML = p.map((item,i)=>`
      <article class="product">
        <img src="${item.image||''}" alt="${esc(item.name)}" loading="lazy" onerror="this.style.display='none'">
        <div class="product-body">
          <h3>${esc(item.name)}</h3>
          <div class="price">${money(item.price)}</div>
          <p class="product-desc">${esc(item.desc||'')}</p>
          <div class="product-actions">
            <button class="btn btn-sm btn-outline" onclick="editProduct(${i})">✏️ Sửa</button>
            <button class="btn btn-sm btn-outline" style="color:var(--red);border-color:#e0c8c8" onclick="deleteProduct(${i})">🗑️ Xóa</button>
          </div>
        </div>
      </article>
    `).join('');
  }
}

// === MODAL ===
function showModal(id){ $(id).classList.remove('hidden'); document.body.style.overflow='hidden'; }
function hideModal(id){ $(id).classList.add('hidden'); document.body.style.overflow=''; }

// === CHỈNH SỬA THÔNG TIN ===
$('#changeInfoBtn').onclick = () => {
  const s = getSettings();
  $('#editForm').elements.siteName.value = s.siteName;
  $('#editForm').elements.greeting.value = s.greeting;
  $('#editForm').elements.announcement.value = s.announcement;
  $('#editForm').elements.contactShort.value = s.contactShort;
  $('#editForm').elements.contactFull.value = s.contactFull;
  showModal('#editModal');
};
$('#closeModal').onclick = () => hideModal('#editModal');
$('#cancelEdit').onclick = () => hideModal('#editModal');
$('#editModal').onclick = e => e.target.id==='editModal' && hideModal('#editModal');
$('#editForm').onsubmit = e => {
  e.preventDefault();
  const f = e.target;
  saveSettings({
    siteName: f.elements.siteName.value.trim(),
    greeting: f.elements.greeting.value.trim(),
    announcement: f.elements.announcement.value.trim(),
    contactShort: f.elements.contactShort.value.trim(),
    contactFull: f.elements.contactFull.value.trim()
  });
  hideModal('#editModal');
};

// === QUẢN LÝ SẢN PHẨM ===
$('#addProductBtn').onclick = () => {
  $('#productForm').reset();
  $('#productIndex').value = '';
  $('#productModalTitle').textContent = 'Thêm sản phẩm';
  showModal('#productModal');
};
window.editProduct = i => {
  const list = getProducts();
  const item = list[i];
  $('#productIndex').value = i;
  $('#productForm').elements.name.value = item.name;
  $('#productForm').elements.price.value = item.price;
  $('#productForm').elements.image.value = item.image||'';
  $('#productForm').elements.desc.value = item.desc||'';
  $('#productModalTitle').textContent = 'Sửa sản phẩm';
  showModal('#productModal');
};
window.deleteProduct = i => {
  if(confirm('Bạn có chắc muốn xóa sản phẩm này?')){
    const list = getProducts();
    list.splice(i,1);
    saveProducts(list);
  }
};
$('#closeProductModal').onclick = () => hideModal('#productModal');
$('#cancelProduct').onclick = () => hideModal('#productModal');
$('#productModal').onclick = e => e.target.id==='productModal' && hideModal('#productModal');
$('#productForm').onsubmit = e => {
  e.preventDefault();
  const f = e.target;
  const idx = $('#productIndex').value;
  const list = getProducts();
  const newItem = {
    name: f.elements.name.value.trim(),
    price: parseInt(f.elements.price.value)||0,
    image: f.elements.image.value.trim(),
    desc: f.elements.desc.value.trim()
  };
  if(idx==='') list.push(newItem);
  else list[parseInt(idx)] = newItem;
  saveProducts(list);
  hideModal('#productModal');
};

// === KHỞI ĐỘNG ===
render();