<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Darpan Wears — Demo Store (Payments & Orders)</title>
<style>
  :root{
    --primary:#0b63d5;
    --muted:#666;
    --card:#fff;
    --bg:#f5f7fb;
    --accent:#ff6a00;
  }
  body{ margin:0; font-family:Inter,system-ui,Arial; background:var(--bg); color:#222; }
  header{ background:linear-gradient(90deg,#fff,#f8fbff); box-shadow:0 2px 6px rgba(11,99,213,0.06); position:sticky; top:0; z-index:50; padding:12px 20px; display:flex; align-items:center; justify-content:space-between; }
  .brand{ display:flex; align-items:center; gap:12px; }
  .logo{ width:48px;height:48px;border-radius:8px;background:linear-gradient(135deg,#0b63d5,#00a0ff);display:flex;align-items:center;justify-content:center;color:#fff;font-weight:700;font-size:18px; }
  .store-name{ font-size:20px; font-weight:700 }
  nav.categories{ display:flex; gap:10px; align-items:center; }
  nav.categories button{ background:none;border:0;padding:8px 12px;border-radius:6px;cursor:pointer;font-weight:600 }
  nav.categories button.active{ background:var(--primary); color:#fff; box-shadow:0 4px 10px rgba(11,99,213,0.12) }
  .controls{ display:flex; gap:10px; align-items:center; }
  .btn{ padding:8px 12px; border-radius:8px; border:0; cursor:pointer; font-weight:600 }
  .btn.primary{ background:var(--primary); color:#fff }
  .container{ max-width:1100px; margin:20px auto; padding:0 16px; }
  .products{ display:grid; gap:16px; grid-template-columns:repeat(auto-fill,minmax(240px,1fr)); }
  .card{ background:var(--card); border-radius:12px; padding:14px; box-shadow:0 6px 20px rgba(20,30,50,0.04); display:flex; flex-direction:column; gap:10px; }
  .card img{ width:100%; height:200px; object-fit:cover; border-radius:8px; background:#eee }
  .meta{ display:flex; justify-content:space-between; align-items:center; gap:8px; }
  .title{ font-weight:700; font-size:16px; }
  .desc{ color:var(--muted); font-size:13px; min-height:38px; }
  .prices{ display:flex; gap:8px; align-items:center; }
  .orig{ text-decoration:line-through; color:var(--muted) }
  .special{ color:var(--accent); font-weight:700 }
  .badge{ font-size:12px; padding:4px 6px; background:#eef6ff; color:var(--primary); border-radius:6px; font-weight:700 }
  .actions{ margin-top:auto; display:flex; gap:8px; }
  .buy{ flex:1; background:var(--accent); color:#fff; border:0; padding:8px; border-radius:8px; cursor:pointer; font-weight:700 }
  .admin-btn{ background:#222; color:#fff; padding:8px 10px; border-radius:8px }
  .products-wrapper{ max-height:900px; overflow:auto; padding-bottom:20px; }
  .order-modal, .admin-modal { position:fixed; inset:0; display:none; align-items:center; justify-content:center; background:rgba(0,0,0,0.4); z-index:200; }
  .modal-card{ background:#fff; padding:18px; border-radius:12px; width:100%; max-width:720px; box-shadow:0 18px 60px rgba(10,30,60,0.18) }
  .modal-card h3{ margin:0 0 8px 0 }
  .field{ display:flex; flex-direction:column; gap:6px; margin-bottom:10px }
  .field label{ font-size:13px; color:var(--muted) }
  .field input, .field textarea, .field select{ padding:10px; border-radius:8px; border:1px solid #e6edf6; font-size:14px }
  .small{ font-size:12px; color:var(--muted) }
  .close-x{ background:none; border:0; font-weight:700; font-size:18px; cursor:pointer; }
  footer{ padding:18px; text-align:center; color:var(--muted); font-size:13px }
  .admin-panel{ display:flex; gap:16px; }
  .admin-left{ flex:1; }
  .admin-right{ width:380px; }
  table{ width:100%; border-collapse:collapse; }
  table th, table td{ text-align:left; padding:8px; border-bottom:1px solid #f1f4f8; font-size:13px }
  .graph-canvas{ width:100%; height:220px; background:#fff; border-radius:8px; padding:12px; box-shadow:0 6px 18px rgba(10,20,40,0.05) }
  @media (max-width:800px){ .admin-panel{ flex-direction:column } .admin-right{ width:100% } .card img{ height:160px } }
  /* small helpers */
  .muted{ color:var(--muted) }
  .ok{ color:green; font-weight:700 }
</style>
</head>
<body>

<header>
  <div class="brand">
    <div class="logo">DW</div>
    <div>
      <div class="store-name">Darpan Wears</div>
      <div class="muted" style="font-size:12px">Quality apparel & gadgets</div>
    </div>
  </div>

  <nav class="categories" aria-label="Categories">
    <button class="cat-btn active" data-cat="all">All</button>
    <button class="cat-btn" data-cat="T-Shirt">T-Shirt</button>
    <button class="cat-btn" data-cat="Jeans">Jeans</button>
    <button class="cat-btn" data-cat="Shoes">Shoes</button>
    <button class="cat-btn" data-cat="Electronic Devices">Electronic Devices</button>
  </nav>

  <div class="controls">
    <button class="btn" id="view-admin">Admin</button>
    <button class="btn primary" id="add-sample">Add Sample</button>
  </div>
</header>

<main class="container">
  <section class="products-wrapper">
    <div id="products" class="products" aria-live="polite"></div>
  </section>
</main>

<!-- ORDER MODAL (updated with payment method) -->
<div class="order-modal" id="orderModal" role="dialog" aria-hidden="true">
  <div class="modal-card">
    <div style="display:flex;justify-content:space-between;align-items:center">
      <h3>Place Order</h3>
      <button class="close-x" id="closeOrder">✕</button>
    </div>
    <div id="orderProductInfo" style="margin-bottom:8px"></div>
    <form id="orderForm">
      <div class="field"><label>Name</label><input name="name" required /></div>
      <div class="field"><label>Phone number</label><input name="phone" required placeholder="e.g. 9332307996" /></div>
      <div class="field"><label>State</label><input name="state" required /></div>
      <div class="field"><label>City</label><input name="city" required /></div>
      <div class="field"><label>Village</label><input name="village" required /></div>
      <div class="field"><label>Pincode</label><input name="pincode" required /></div>

      <!-- Payment method selection -->
      <div class="field">
        <label>Payment Method</label>
        <div style="display:flex;gap:12px;align-items:center">
          <label><input type="radio" name="payment" value="COD" checked /> Cash on Delivery (COD)</label>
          <label><input type="radio" name="payment" value="Online" /> Online Payment (Pay now)</label>
        </div>
        <div style="font-size:12px;color:var(--muted)">If Online is chosen you'll see a simulated payment and order will be marked paid.</div>
      </div>

      <div style="display:flex;gap:8px;justify-content:flex-end;margin-top:6px">
        <button type="button" class="btn" id="cancelOrder">Cancel</button>
        <button type="submit" class="btn primary">Confirm Order</button>
      </div>
    </form>
  </div>
</div>

<!-- ADMIN MODAL (now shows Orders + Products + export) -->
<div class="admin-modal" id="adminModal" role="dialog" aria-hidden="true">
  <div class="modal-card" style="max-width:980px">
    <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:6px">
      <h3>Admin Panel</h3>
      <button class="close-x" id="closeAdmin">✕</button>
    </div>

    <div id="adminContent">
      <!-- password screen -->
      <div id="adminLock">
        <p class="small">Enter admin password to access. (Password: <strong>darpan2025</strong>)</p>
        <div style="display:flex;gap:8px;margin-top:8px">
          <input id="adminPass" placeholder="Password" />
          <button class="btn primary" id="unlockBtn">Unlock</button>
        </div>
        <p id="lockMsg" class="small" style="color:crimson;display:none;margin-top:6px"></p>
      </div>

      <!-- admin panel when unlocked -->
      <div id="adminPanel" style="display:none">
        <div class="admin-panel">
          <div class="admin-left">
            <h4>Orders</h4>
            <div style="display:flex;gap:8px;align-items:center;margin-bottom:8px">
              <button class="btn" id="refreshOrders">Refresh</button>
              <button class="btn" id="exportCsv">Export Orders CSV</button>
              <div style="flex:1"></div>
              <div class="small muted">Total orders: <span id="totalOrders">0</span></div>
            </div>
            <table id="ordersTable">
              <thead><tr><th>When</th><th>Product</th><th>Buyer</th><th>Phone</th><th>Payment</th><th>Status</th><th></th></tr></thead>
              <tbody></tbody>
            </table>

            <h4 style="margin-top:14px">Products</h4>
            <table id="productsTable">
              <thead>
                <tr><th>Name</th><th>Price</th><th>Special</th><th>Material</th><th>Sales</th><th></th></tr>
              </thead>
              <tbody></tbody>
            </table>
          </div>

          <div class="admin-right">
            <h4>Add / Edit Product</h4>
            <div class="field"><label>Name</label><input id="pName" /></div>
            <div class="field"><label>Sell Price</label><input id="pSell" type="number" /></div>
            <div class="field"><label>Special Price</label><input id="pSpecial" type="number" /></div>
            <div class="field"><label>Material</label><input id="pMaterial" /></div>
            <div class="field"><label>Category</label>
              <select id="pCategory">
                <option value="T-Shirt">T-Shirt</option>
                <option value="Jeans">Jeans</option>
                <option value="Shoes">Shoes</option>
                <option value="Electronic Devices">Electronic Devices</option>
              </select>
            </div>
            <div class="field"><label>Image URL (or bbcode [img]...[/img])</label><input id="pImg" /></div>
            <div style="display:flex;gap:8px;justify-content:flex-end">
              <button class="btn" id="clearForm">Clear</button>
              <button class="btn primary" id="saveProduct">Save Product</button>
            </div>

            <h4 style="margin-top:16px">Sales Graph</h4>
            <div class="graph-canvas">
              <canvas id="salesChart" width="320" height="220"></canvas>
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
</div>

<footer>
  © <span id="yr"></span> Darpan Wears — Demo. Orders are sent to WhatsApp number <strong>9332307996</strong>.
</footer>

<script>
(function(){
  /* ---------------------------
     Configuration & Storage keys
     --------------------------- */
  const WA_NUMBER = "9332307996";
  const ADMIN_PASS = "darpan2025";
  const LS_PRODUCTS = "darpan_products_v1";
  const LS_ORDERS = "darpan_orders_v1";

  /* ---------------------------
     Utility helpers
     --------------------------- */
  function qs(sel, ctx=document){ return ctx.querySelector(sel) }
  function qsa(sel, ctx=document){ return Array.from(ctx.querySelectorAll(sel)) }
  function genId(){ return 'p_' + Math.random().toString(36).slice(2,9) }
  function nowIso(){ return new Date().toISOString() }
  function fmtDate(iso){ const d=new Date(iso); return d.toLocaleString(); }
  function numberWithCommas(x){ return String(x).replace(/\B(?=(\d{3})+(?!\d))/g, ","); }
  function escapeHtml(s){ if(s==null) return ''; return String(s).replace(/[&<>"']/g, m => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[m])); }

  function extractImageUrl(raw){
    if(!raw) return "";
    const bb = raw.match(/\\[img\\](.*?)\\[\\/img\\]/i);
    if(bb && bb[1]) return bb[1].trim();
    const srcMatch = raw.match(/src=[\"']([^\"']+)[\"']/i);
    if(srcMatch) return srcMatch[1];
    return raw.trim();
  }

  /* ---------------------------
     Sample data & load/save
     --------------------------- */
  function sampleProducts(){
    return [
      { id: genId(), name:"Classic White T-Shirt", desc:"Comfort-fit cotton tee", price:499, special:349, size:"M,L,XL", material:"Cotton", image:"https://images.unsplash.com/photo-1602810312944-0b2b0a3f3f78?auto=format&fit=crop&w=800&q=60", category:"T-Shirt", sales:0 },
      { id: genId(), name:"Blue Denim Jeans", desc:"Slim fit, stretchable", price:1299, special:999, size:"30,32,34", material:"Denim", image:"https://images.unsplash.com/photo-1520975916012-57c3b4657eb6?auto=format&fit=crop&w=800&q=60", category:"Jeans", sales:0 },
      { id: genId(), name:"Running Shoes", desc:"Lightweight sports shoes", price:2999, special:2499, size:"8,9,10", material:"Mesh", image:"https://images.unsplash.com/photo-1528701800489-4769a3f1a7bf?auto=format&fit=crop&w=800&q=60", category:"Shoes", sales:0 },
      { id: genId(), name:"Wireless Earbuds", desc:"Noise-cancelling earbuds", price:1999, special:1499, size:"One Size", material:"Plastic", image:"https://images.unsplash.com/photo-1585386959984-a4155226e9e6?auto=format&fit=crop&w=800&q=60", category:"Electronic Devices", sales:0 }
    ];
  }

  function loadProducts(){
    const raw = localStorage.getItem(LS_PRODUCTS);
    if(!raw){ const p = sampleProducts(); localStorage.setItem(LS_PRODUCTS, JSON.stringify(p)); return p; }
    try { return JSON.parse(raw); } catch(e){ const p = sampleProducts(); localStorage.setItem(LS_PRODUCTS, JSON.stringify(p)); return p; }
  }
  function saveProducts(arr){ localStorage.setItem(LS_PRODUCTS, JSON.stringify(arr)); }

  function loadOrders(){
    const raw = localStorage.getItem(LS_ORDERS);
    if(!raw){ localStorage.setItem(LS_ORDERS, JSON.stringify([])); return []; }
    try { return JSON.parse(raw) } catch(e){ localStorage.setItem(LS_ORDERS, JSON.stringify([])); return []; }
  }
  function saveOrders(arr){ localStorage.setItem(LS_ORDERS, JSON.stringify(arr)); }

  /* ---------------------------
     App state & elements
     --------------------------- */
  let products = loadProducts();
  let orders = loadOrders();
  let currentCategory = "all";
  let activeProductId = null;
  let editingId = null;

  const prodContainer = qs('#products');
  const catBtns = qsa('.cat-btn');
  const orderModal = qs('#orderModal');
  const orderForm = qs('#orderForm');
  const orderProductInfo = qs('#orderProductInfo');
  const closeOrder = qs('#closeOrder');
  const cancelOrder = qs('#cancelOrder');
  const adminModal = qs('#adminModal');
  const viewAdmin = qs('#view-admin');
  const closeAdmin = qs('#closeAdmin');
  const unlockBtn = qs('#unlockBtn');
  const adminPass = qs('#adminPass');
  const adminLock = qs('#adminLock');
  const adminPanel = qs('#adminPanel');
  const lockMsg = qs('#lockMsg');
  const productsTableBody = qs('#productsTable tbody');
  const ordersTableBody = qs('#ordersTable tbody');
  const saveProductBtn = qs('#saveProduct');
  const clearFormBtn = qs('#clearForm');
  const pName = qs('#pName');
  const pSell = qs('#pSell');
  const pSpecial = qs('#pSpecial');
  const pMaterial = qs('#pMaterial');
  const pImg = qs('#pImg');
  const pCategory = qs('#pCategory');
  const salesChart = qs('#salesChart');
  const addSampleBtn = qs('#add-sample');
  const totalOrdersEl = qs('#totalOrders');
  const refreshOrdersBtn = qs('#refreshOrders');
  const exportCsvBtn = qs('#exportCsv');

  qs('#yr').textContent = new Date().getFullYear();

  /* ---------------------------
     Rendering functions
     --------------------------- */
  function renderProducts(){
    prodContainer.innerHTML = '';
    const list = products.filter(p => currentCategory==='all' || p.category === currentCategory);
    list.forEach(p => {
      const card = document.createElement('div');
      card.className = 'card';
      card.innerHTML = `
        <img src="${escapeHtml(p.image||'')}" alt="${escapeHtml(p.name)}" onerror="this.style.opacity=0.6;this.style.objectFit='contain'"/>
        <div class="meta"><div class="title">${escapeHtml(p.name)}</div><div class="badge">${escapeHtml(p.category)}</div></div>
        <div class="desc">${escapeHtml(p.desc || '')}</div>
        <div class="prices"><div class="orig">₹${numberWithCommas(p.price)}</div><div class="special">₹${numberWithCommas(p.special)}</div></div>
        <div class="small">Size: ${escapeHtml(p.size || 'One Size')} • Material: ${escapeHtml(p.material || '')}</div>
        <div class="actions">
          <button class="buy" data-id="${p.id}">Buy Now</button>
          <button class="btn admin-btn" data-id="${p.id}" title="Quick Admin Edit">✎</button>
        </div>
      `;
      prodContainer.appendChild(card);
    });
    qsa('.buy').forEach(btn => btn.onclick = openOrderForProduct);
    qsa('.card .admin-btn').forEach(b => b.onclick = quickAdminEdit);
  }

  function renderAdminTables(){
    // products table
    productsTableBody.innerHTML = '';
    products.forEach(p => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${escapeHtml(p.name)}</td>
        <td>₹${numberWithCommas(p.price)}</td>
        <td>₹${numberWithCommas(p.special)}</td>
        <td>${escapeHtml(p.material||'')}</td>
        <td>${p.sales||0}</td>
        <td>
          <div style="display:flex;gap:8px">
            <button class="btn" data-action="edit" data-id="${p.id}">Edit</button>
            <button class="btn" data-action="del" data-id="${p.id}">Delete</button>
          </div>
        </td>
      `;
      productsTableBody.appendChild(tr);
    });
    qsa('[data-action="edit"]').forEach(b => b.onclick = e => startEdit(e.target.dataset.id));
    qsa('[data-action="del"]').forEach(b => b.onclick = e => {
      const id = e.target.dataset.id;
      if(confirm('Delete product?')) {
        products = products.filter(x => x.id !== id);
        saveProducts(products);
        renderAdminTables();
        renderProducts();
        drawSalesChart();
      }
    });

    // orders table
    ordersTableBody.innerHTML = '';
    const list = orders.slice().reverse(); // newest first
    list.forEach(o => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${fmtDate(o.createdAt)}</td>
        <td>${escapeHtml(o.productName)}</td>
        <td>${escapeHtml(o.customerName)}</td>
        <td>${escapeHtml(o.phone)}</td>
        <td>${escapeHtml(o.paymentMethod)}</td>
        <td>${escapeHtml(o.status)}</td>
        <td><div style="display:flex;gap:6px"><button class="btn" data-act="view" data-id="${o.id}">View</button><button class="btn" data-act="toggle" data-id="${o.id}">Toggle status</button></div></td>
      `;
      ordersTableBody.appendChild(tr);
    });
    qsa('[data-act="view"]').forEach(b => b.onclick = e => viewOrder(e.target.dataset.id));
    qsa('[data-act="toggle"]').forEach(b => b.onclick = e => toggleOrderStatus(e.target.dataset.id));

    totalOrdersEl.textContent = orders.length;
  }

  /* ---------------------------
     Order flow
     --------------------------- */
  function openOrderForProduct(e){
    const id = e.currentTarget.dataset.id;
    activeProductId = id;
    const p = products.find(x=>x.id===id);
    orderProductInfo.innerHTML = `<strong>${escapeHtml(p.name)}</strong> — ₹${numberWithCommas(p.special)}<div class="small">${escapeHtml(p.desc||'')}</div>`;
    orderModal.style.display = 'flex';
    orderModal.setAttribute('aria-hidden','false');
  }
  function closeOrderModal(){
    orderModal.style.display = 'none';
    orderModal.setAttribute('aria-hidden','true');
    orderForm.reset();
    activeProductId = null;
  }
  closeOrder.onclick = closeOrderModal;
  cancelOrder.onclick = closeOrderModal;

  orderForm.addEventListener('submit', function(ev){
    ev.preventDefault();
    const data = new FormData(orderForm);
    const payload = {
      customerName: data.get('name'),
      phone: data.get('phone'),
      state: data.get('state'),
      city: data.get('city'),
      village: data.get('village'),
      pincode: data.get('pincode'),
      paymentMethod: data.get('payment') || 'COD'
    };
    const p = products.find(x=>x.id===activeProductId);
    if(!p){
      alert('Product not found.');
      return;
    }

    // If Online payment chosen -> simulate payment then record order as Paid
    if(payload.paymentMethod === 'Online'){
      // show temporary processing overlay
      const overlay = document.createElement('div');
      overlay.style = "position:fixed;inset:0;display:flex;align-items:center;justify-content:center;background:rgba(0,0,0,0.45);z-index:300";
      overlay.innerHTML = `<div style="background:#fff;padding:20px;border-radius:10px;text-align:center"><div style="font-weight:700;margin-bottom:8px">Processing payment...</div><div class="small muted">Simulated demo — succeeds automatically</div></div>`;
      document.body.appendChild(overlay);
      // simulate async payment (2s)
      setTimeout(()=> {
        document.body.removeChild(overlay);
        // payment success -> record as Paid
        const ord = createOrderRecord(p, payload, 'Paid (Online)');
        orders.push(ord);
        saveOrders(orders);
        // update product sales
        p.sales = (p.sales||0) + 1;
        saveProducts(products);
        renderAdminTables();
        renderProducts();
        drawSalesChart();
        // open WhatsApp for your records (still helpful)
        openWhatsAppWithOrder(ord);
        closeOrderModal();
        alert('Payment successful (simulated) and order recorded. WhatsApp will open with order details.');
      }, 1800);

    } else {
      // COD path: record order as COD - pending, increment sales only when you confirm? (here we increment on order placement)
      const ord = createOrderRecord(p, payload, 'COD - pending');
      orders.push(ord);
      saveOrders(orders);
      // increment sales - since you as reseller will order on Meesho now; increment helps sales graph
      p.sales = (p.sales||0) + 1;
      saveProducts(products);
      renderAdminTables();
      renderProducts();
      drawSalesChart();
      // open whatsapp
      openWhatsAppWithOrder(ord);
      closeOrderModal();
      alert('Order recorded and WhatsApp will open so you can place the order on Meesho.');
    }
  });

  function createOrderRecord(productObj, payload, status){
    const id = 'o_' + Math.random().toString(36).slice(2,10);
    const record = {
      id,
      productId: productObj.id,
      productName: productObj.name,
      price: productObj.special,
      customerName: payload.customerName,
      phone: payload.phone,
      state: payload.state,
      city: payload.city,
      village: payload.village,
      pincode: payload.pincode,
      paymentMethod: payload.paymentMethod,
      status,
      createdAt: nowIso()
    };
    return record;
  }

  function openWhatsAppWithOrder(ord){
    // create text with order details and payment status
    const lines = [
      `New Order — Darpan Wears`,
      `Product: ${ord.productName}`,
      `Price: ₹${ord.price}`,
      `Buyer: ${ord.customerName}`,
      `Phone: ${ord.phone}`,
      `State: ${ord.state}`,
      `City: ${ord.city}`,
      `Village: ${ord.village}`,
      `Pincode: ${ord.pincode}`,
      `Payment: ${ord.paymentMethod} (${ord.status})`,
      `Order ID: ${ord.id}`,
      `Time: ${fmtDate(ord.createdAt)}`
    ];
    const text = encodeURIComponent(lines.join('\\n'));
    const link = `https://wa.me/${WA_NUMBER}?text=${text}`;
    window.open(link, '_blank');
  }

  /* ---------------------------
     Admin functions: orders view & toggles
     --------------------------- */
  function viewOrder(orderId){
    const ord = orders.find(o => o.id === orderId);
    if(!ord) return alert('Order not found');
    const lines = [
      `Order ID: ${ord.id}`,
      `Product: ${ord.productName}`,
      `Price: ₹${ord.price}`,
      `Buyer: ${ord.customerName}`,
      `Phone: ${ord.phone}`,
      `Address: ${ord.village}, ${ord.city}, ${ord.state} - ${ord.pincode}`,
      `Payment: ${ord.paymentMethod}`,
      `Status: ${ord.status}`,
      `Created: ${fmtDate(ord.createdAt)}`
    ];
    alert(lines.join('\\n'));
  }

  function toggleOrderStatus(orderId){
    const ord = orders.find(o => o.id === orderId);
    if(!ord) return;
    // simple toggle: if COD -> mark Confirmed or Delivered; if Paid -> mark Completed
    if(ord.status.includes('pending')) ord.status = ord.status.replace('pending', 'confirmed');
    else if(ord.status.includes('Paid')) ord.status = 'Completed';
    else if(ord.status === 'COD - confirmed') ord.status = 'Completed';
    else ord.status = 'COD - confirmed';
    saveOrders(orders);
    renderAdminTables();
    alert('Order status updated.');
  }

  /* ---------------------------
     Admin: product add/edit/delete
     --------------------------- */
  clearFormBtn.onclick = function(){ clearProductForm(); };
  saveProductBtn.onclick = function(){
    const name = pName.value.trim();
    const price = Number(pSell.value) || 0;
    const special = Number(pSpecial.value) || 0;
    const material = pMaterial.value.trim();
    const imgRaw = pImg.value.trim();
    const category = pCategory.value;
    if(!name){ alert('Enter product name'); return; }
    const img = extractImageUrl(imgRaw);
    if(editingId){
      const idx = products.findIndex(x=>x.id===editingId);
      if(idx>=0){
        products[idx].name = name;
        products[idx].price = price;
        products[idx].special = special;
        products[idx].material = material;
        products[idx].image = img;
        products[idx].category = category;
      }
      editingId = null;
    } else {
      products.push({
        id: genId(),
        name, price, special,
        material, image: img,
        desc: '', size: '',
        category,
        sales: 0
      });
    }
    saveProducts(products);
    renderAdminTables();
    renderProducts();
    drawSalesChart();
    clearProductForm();
    alert('Product saved');
  };

  function startEdit(id){
    const p = products.find(x=>x.id===id);
    if(!p) return;
    editingId = id;
    pName.value = p.name;
    pSell.value = p.price;
    pSpecial.value = p.special;
    pMaterial.value = p.material || '';
    pImg.value = p.image || '';
    pCategory.value = p.category || 'T-Shirt';
    adminLock.style.display = 'none';
    adminPanel.style.display = 'block';
    renderAdminTables();
  }

  function quickAdminEdit(e){
    const id = e.currentTarget.dataset.id;
    viewAdmin.click();
    adminPass.value = ADMIN_PASS; // helpful autofill; user must press unlock
    setTimeout(()=> adminPass.focus(), 200);
  }

  function clearProductForm(){
    editingId = null;
    pName.value = '';
    pSell.value = '';
    pSpecial.value = '';
    pMaterial.value = '';
    pImg.value = '';
  }

  /* ---------------------------
     Categories & UI
     --------------------------- */
  catBtns.forEach(b => { b.onclick = function(){ catBtns.forEach(x=>x.classList.remove('active')); this.classList.add('active'); currentCategory = this.dataset.cat; renderProducts(); } });

  /* ---------------------------
     Admin modal open/close & unlock
     --------------------------- */
  viewAdmin.onclick = () => {
    adminModal.style.display = 'flex';
    adminModal.setAttribute('aria-hidden','false');
    adminLock.style.display = 'block';
    adminPanel.style.display = 'none';
    adminPass.value = '';
    lockMsg.style.display = 'none';
  };
  closeAdmin.onclick = () => {
    adminModal.style.display = 'none';
    adminModal.setAttribute('aria-hidden','true');
    adminLock.style.display = 'block';
    adminPanel.style.display = 'none';
  };
  unlockBtn.onclick = () => {
    const val = adminPass.value.trim();
    if(val === ADMIN_PASS){
      adminLock.style.display = 'none';
      adminPanel.style.display = 'block';
      renderAdminTables();
      drawSalesChart();
    } else {
      lockMsg.style.display = 'block';
      lockMsg.textContent = 'Incorrect password';
    }
  };

  /* ---------------------------
     Sales chart (canvas)
     --------------------------- */
  function drawSalesChart(){
    const ctx = salesChart.getContext('2d');
    const w = salesChart.width; const h = salesChart.height;
    ctx.clearRect(0,0,w,h);
    const data = products.slice().sort((a,b)=>b.sales-a.sales).slice(0,8);
    const labels = data.map(d=>d.name);
    const values = data.map(d=>d.sales||0);
    const maxVal = Math.max(1, ...values);
    const left = 10, top = 10, right = 10, bottom = 30;
    const plotW = w - left - right;
    const plotH = h - top - bottom;
    const barW = Math.max(12, plotW / Math.max(1, values.length) * 0.6);
    ctx.strokeStyle = '#e9eef6';
    ctx.beginPath();
    ctx.moveTo(left, top);
    ctx.lineTo(left, top + plotH);
    ctx.lineTo(left + plotW, top + plotH);
    ctx.stroke();
    values.forEach((val,i)=>{
      const x = left + (i * (plotW / values.length)) + ((plotW / values.length) - barW)/2;
      const barH = (val / maxVal) * (plotH-6);
      const y = top + plotH - barH;
      const grad = ctx.createLinearGradient(x,y,x,y+barH);
      grad.addColorStop(0, '#ff8a3d');
      grad.addColorStop(1, '#ff5a00');
      ctx.fillStyle = grad;
      ctx.fillRect(x, y, barW, barH);
      ctx.fillStyle = '#333';
      ctx.font = '11px sans-serif';
      ctx.textAlign = 'center';
      ctx.fillText(String(labels[i]).slice(0,12), x + barW/2, top + plotH + 14);
      ctx.fillStyle = '#000';
      ctx.font = '12px 600 Inter, sans-serif';
      ctx.fillText(String(values[i]), x + barW/2, y - 6);
    });
  }

  /* ---------------------------
     Orders export (CSV)
     --------------------------- */
  function ordersToCSV(items){
    const headers = ['order_id','created_at','product_id','product_name','price','customer_name','phone','state','city','village','pincode','payment_method','status'];
    const rows = items.map(o => [
      o.id, o.createdAt, o.productId, o.productName, o.price, o.customerName, o.phone, o.state, o.city, o.village, o.pincode, o.paymentMethod, o.status
    ]);
    const csv = [headers.join(',')].concat(rows.map(r => r.map(field => `"${String(field||'').replace(/"/g,'""')}"`).join(','))).join('\\n');
    return csv;
  }
  function downloadFile(content, filename, type='text/csv'){
    const blob = new Blob([content], { type });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url; a.download = filename; document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
  }

  /* ---------------------------
     Small UI actions
     --------------------------- */
  addSampleBtn.onclick = function(){
    const demo = { id: genId(), name:'New Sample', desc:'Sample product', price:999, special:799, size:'M', material:'Cotton', image:'https://via.placeholder.com/400x300?text=Product', category:'T-Shirt', sales:0 };
    products.push(demo); saveProducts(products); renderProducts(); drawSalesChart();
  };
  refreshOrdersBtn.onclick = function(){ orders = loadOrders(); renderAdminTables(); };
  exportCsvBtn.onclick = function(){ const csv = ordersToCSV(orders); downloadFile(csv, 'darpan_orders.csv'); };

  // close modals on outside click
  document.querySelectorAll('.order-modal, .admin-modal').forEach(m => {
    m.addEventListener('click', function(e){
      if(e.target === this){
        this.style.display = 'none';
        this.setAttribute('aria-hidden','true');
      }
    });
  });

  /* ---------------------------
     Initial render
     --------------------------- */
  renderProducts();
  renderAdminTables();
  drawSalesChart();

  // expose for debugging
  window.__DW = { products, orders, saveProducts, saveOrders };
})();
</script>
</body>
</html>
