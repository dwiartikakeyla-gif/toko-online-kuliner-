# toko-online-kuliner- <!doctype html>
<html lang="id">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Toko Online Kuliner - ACODE</title>
  <style>
    /* Simple, clean styling yang kompatibel untuk Acode */
    :root{--accent:#EF4444;--muted:#666}
    body{font-family:Inter, system-ui, Arial, sans-serif; margin:0; padding:0; background:#fafafa}
    header{background:linear-gradient(90deg,#fff 0%, #fff 60%); padding:16px 20px; box-shadow:0 1px 4px rgba(0,0,0,0.05); display:flex; align-items:center; justify-content:space-between}
    h1{margin:0; font-size:18px; color:#111}
    .container{max-width:1000px; margin:20px auto; padding:0 16px}
    .grid{display:grid; grid-template-columns: repeat(auto-fill,minmax(220px,1fr)); gap:16px}
    .card{background:#fff; border-radius:10px; padding:12px; box-shadow:0 2px 8px rgba(0,0,0,0.04)}
    .product img{width:100%; height:140px; object-fit:cover; border-radius:8px}
    .price{font-weight:700; color:var(--accent)}
    button{background:var(--accent); color:white; border:none; padding:8px 12px; border-radius:8px; cursor:pointer}
    button.ghost{background:transparent; color:var(--accent); border:1px solid var(--accent)}
    .cart{position:fixed; right:16px; bottom:16px; width:360px; max-width:95%; z-index:50}
    .cart .card{padding:12px}
    .cart-list{max-height:300px; overflow:auto}
    .muted{color:var(--muted); font-size:13px}
    label{display:block; margin-top:8px; font-size:13px}
    input, textarea, select{width:100%; padding:8px; border-radius:8px; border:1px solid #e6e6e6}
    footer{padding:12px; text-align:center; color:var(--muted); font-size:13px}
    @media(max-width:600px){ header{padding:12px} .cart{width:95%; left:50%; transform:translateX(-50%)} }
  </style>
</head>
<body>
  <header>
    <h1>🍽️ Toko Kuliner — Demo (ACode)</h1>
    <div class="muted">Local demo • Bisa dijalankan di Acode</div>
  </header>

  <main class="container">
    <section>
      <h2>Menu Kami</h2>
      <div id="products" class="grid" aria-live="polite"></div>
    </section>

    <section style="margin-top:20px">
      <h2>Informasi Toko</h2>
      <div class="card">
        <p class="muted">Demo aplikasi toko online kuliner yang lengkap dengan keranjang, checkout, dan sistem pembayaran simulasi — cocok untuk dijalankan di aplikasi editor seperti Acode (file HTML tunggal).</p>
        <p class="muted">Catatan: pembayaran di sini adalah <strong>simulasi</strong>. Pada bagian komentar di kode ada petunjuk singkat cara menghubungkan gateway nyata seperti Stripe/Midtrans/PayPal.</p>
      </div>
    </section>
  </main>

  <!-- Cart (fixed) -->
  <div class="cart" id="cartRoot">
    <div class="card">
      <h3>Keranjang (<span id="cartCount">0</span>)</h3>
      <div class="cart-list" id="cartList"></div>
      <div style="margin-top:10px; display:flex; gap:8px; align-items:center; justify-content:space-between">
        <div>
          <div class="muted">Total:</div>
          <div style="font-weight:800" id="cartTotal">Rp0</div>
        </div>
        <div>
          <button id="btnCheckout" disabled>Checkout</button>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal: Checkout -->
  <div id="checkoutModal" style="display:none; position:fixed; inset:0; background:rgba(0,0,0,0.4); z-index:60; align-items:center; justify-content:center">
    <div style="width:720px; max-width:95%; background:#fff; border-radius:10px; padding:16px; max-height:90vh; overflow:auto">
      <h3>Checkout</h3>
      <form id="checkoutForm">
        <div style="display:grid; grid-template-columns:1fr 1fr; gap:12px">
          <div>
            <label>Nama Penerima</label>
            <input name="name" required />
          </div>
          <div>
            <label>No. HP</label>
            <input name="phone" required />
          </div>
        </div>
        <label>Alamat Lengkap</label>
        <textarea name="address" rows="2" required></textarea>

        <label>Metode Pembayaran</label>
        <select name="paymentMethod" id="paymentMethod">
          <option value="bank">Transfer Bank (manual)</option>
          <option value="virtual">Virtual Account / Kartu (simulasi)</option>
          <option value="cod">COD (Bayar di Tempat)</option>
        </select>

        <div style="display:flex; gap:8px; margin-top:12px">
          <button type="submit">Bayar Sekarang</button>
          <button type="button" id="btnCloseModal" class="ghost">Batal</button>
        </div>
      </form>

      <hr style="margin:16px 0" />
      <div class="muted">Ringkasan Pesanan</div>
      <div id="summary"></div>
    </div>
  </div>

  <!-- Receipt / Invoice modal -->
  <div id="receiptModal" style="display:none; position:fixed; inset:0; background:rgba(0,0,0,0.4); z-index:70; align-items:center; justify-content:center">
    <div style="width:640px; max-width:95%; background:#fff; border-radius:10px; padding:16px; max-height:90vh; overflow:auto">
      <h3>Struk Pembayaran</h3>
      <div id="receiptContent"></div>
      <div style="display:flex; gap:8px; margin-top:12px">
        <button id="btnCloseReceipt">Tutup</button>
        <button id="btnDownloadReceipt" class="ghost">Download (PNG)</button>
      </div>
    </div>
  </div>

  <footer>
    Dibuat untuk demo Acode — ubah gambar, produk, dan integrasikan gateway nyata sesuai kebutuhan.
  </footer>

  <script>
    /* =====================
       DATA PRODUK (ubah sesuai kebutuhan)
       ===================== */
    const PRODUCTS = [
      { id: 'p1', name: 'Nasi Goreng Spesial', price: 25000, img: 'https://picsum.photos/seed/nasigoreng/800/600' },
      { id: 'p2', name: 'Ayam Geprek', price: 30000, img: 'https://picsum.photos/seed/ayamgeprek/800/600' },
      { id: 'p3', name: 'Mie Ayam Bakso', price: 20000, img: 'https://picsum.photos/seed/mieayam/800/600' },
      { id: 'p4', name: 'Es Teh Manis', price: 8000, img: 'https://picsum.photos/seed/esteh/800/600' },
      { id: 'p5', name: 'Pisang Bakar Coklat', price: 15000, img: 'https://picsum.photos/seed/pisang/800/600' }
    ];

    /* ===== simple cart ===== */
    let cart = JSON.parse(localStorage.getItem('acode_cart') || '[]');

    function formatRp(n){ return 'Rp' + n.toString().replace(/\B(?=(\d{3})+(?!\d))/g, '.') }

    function renderProducts(){
      const root = document.getElementById('products'); root.innerHTML = '';
      PRODUCTS.forEach(p => {
        const el = document.createElement('div'); el.className = 'card product';
        el.innerHTML = `
          <img src="${p.img}" alt="${p.name}" />
          <h4 style="margin:8px 0">${p.name}</h4>
          <div class="price">${formatRp(p.price)}</div>
          <div style="margin-top:8px; display:flex; gap:8px">
            <button onclick="addToCart('${p.id}')">Tambah</button>
            <button class="ghost" onclick="quickView('${p.id}')">Lihat</button>
          </div>
        `;
        root.appendChild(el);
      })
    }

    function saveCart(){ localStorage.setItem('acode_cart', JSON.stringify(cart)); updateCartUI(); }

    function addToCart(id){
      const prod = PRODUCTS.find(x=>x.id===id); if(!prod) return;
      const item = cart.find(i=>i.id===id);
      if(item) item.qty++;
      else cart.push({ id:prod.id, name:prod.name, price:prod.price, qty:1 });
      saveCart();
      toast('Produk ditambahkan');
    }

    function removeFromCart(id){ cart = cart.filter(i=>i.id!==id); saveCart(); }
    function changeQty(id, delta){
      const it = cart.find(i=>i.id===id); if(!it) return;
      it.qty += delta; if(it.qty<1) removeFromCart(id); saveCart();
    }

    function updateCartUI(){
      const list = document.getElementById('cartList'); list.innerHTML = '';
      let total = 0; let count = 0;
      cart.forEach(i=>{
        total += i.price * i.qty; count += i.qty;
        const row = document.createElement('div'); row.style.display='flex'; row.style.justifyContent='space-between'; row.style.alignItems='center'; row.style.marginBottom='8px';
        row.innerHTML = `
          <div style="flex:1">
            <div style="font-weight:700">${i.name}</div>
            <div class="muted">${formatRp(i.price)} x ${i.qty}</div>
          </div>
          <div style="display:flex; gap:6px; align-items:center">
            <button class="ghost" onclick="changeQty('${i.id}',-1)">-</button>
            <button class="ghost" onclick="changeQty('${i.id}',1)">+</button>
            <button class="ghost" onclick="removeFromCart('${i.id}')">Hapus</button>
          </div>
        `;
        list.appendChild(row);
      })
      document.getElementById('cartTotal').textContent = formatRp(total);
      document.getElementById('cartCount').textContent = count;
      document.getElementById('btnCheckout').disabled = cart.length===0;
    }

    function quickView(id){ const p = PRODUCTS.find(x=>x.id===id); if(!p) return; alert(`${p.name}\nHarga: ${formatRp(p.price)}`) }

    // Toast sederhana
    function toast(msg){ const t = document.createElement('div'); t.textContent = msg; t.style.position='fixed'; t.style.left='50%'; t.style.transform='translateX(-50%)'; t.style.bottom='90px'; t.style.background='#111'; t.style.color='#fff'; t.style.padding='8px 12px'; t.style.borderRadius='8px'; t.style.zIndex=999; document.body.appendChild(t); setTimeout(()=>t.remove(),1500)
    }

    // Checkout flow
    document.getElementById('btnCheckout').addEventListener('click', ()=>{
      openCheckout();
    })
    document.getElementById('btnCloseModal').addEventListener('click', ()=>{closeCheckout()});

    function openCheckout(){
      document.getElementById('checkoutModal').style.display='flex';
      renderSummary();
    }
    function closeCheckout(){ document.getElementById('checkoutModal').style.display='none'; }

    function renderSummary(){
      const s = document.getElementById('summary'); s.innerHTML = '';
      let total = 0;
      cart.forEach(i=>{ total += i.price * i.qty; const p = document.createElement('div'); p.style.display='flex'; p.style.justifyContent='space-between'; p.style.marginBottom='6px'; p.innerHTML = `<div>${i.name} x ${i.qty}</div><div>${formatRp(i.price*i.qty)}</div>`; s.appendChild(p) })
      const fee = Math.ceil(total*0.02); // contoh biaya layanan 2%
      const grand = total + fee;
      const footer = document.createElement('div'); footer.style.marginTop='10px'; footer.innerHTML = `<div style="display:flex;justify-content:space-between"><div class="muted">Sub total</div><div>${formatRp(total)}</div></div><div style="display:flex;justify-content:space-between"><div class="muted">Biaya layanan (2%)</div><div>${formatRp(fee)}</div></div><hr/><div style="display:flex;justify-content:space-between;font-weight:800"><div>Total</div><div>${formatRp(grand)}</div></div>`;
      s.appendChild(footer);
    }

    // Submit checkout (simulasi pembayaran)
    document.getElementById('checkoutForm').addEventListener('submit', async (e)=>{
      e.preventDefault();
      const fd = new FormData(e.target);
      const data = Object.fromEntries(fd.entries());

      // Build order
      const order = {
        id: 'ORD' + Date.now(),
        customer: data.name,
        phone: data.phone,
        address: data.address,
        paymentMethod: data.paymentMethod,
        items: cart.map(i=>({ name:i.name, price:i.price, qty:i.qty })),
        createdAt: new Date().toISOString()
      };

      // Simulasi: kalau paymentMethod === 'virtual' lakukan "proses" pembayaran otomatis
      closeCheckout();
      if(order.paymentMethod === 'virtual'){
        toast('Memproses pembayaran...');
        // Simulate a gateway call delay
        await new Promise(r=>setTimeout(r, 1400));
        showReceipt({...order, status:'PAID', paymentInfo:{ method:'Virtual Account (SIMULASI)', transactionId:'TX' + Date.now() }});
        cart = []; saveCart();
      } else if(order.paymentMethod === 'bank'){
        // Berikan instruksi transfer manual
        showReceipt({...order, status:'PENDING', paymentInfo:{ method:'Transfer Bank (Instruksi)', bank:'BCA - 1234567890', a_n:'Toko Kuliner Demo' }});
        cart = []; saveCart();
      } else if(order.paymentMethod === 'cod'){
        showReceipt({...order, status:'PENDING', paymentInfo:{ method:'COD (Bayar di tempat)' }});
        cart = []; saveCart();
      }

    })

    function showReceipt(order){
      const root = document.getElementById('receiptContent'); root.innerHTML = '';
      const h = document.createElement('div');
      h.innerHTML = `
        <div style="display:flex;justify-content:space-between"><div><strong>${order.id}</strong><div class=\"muted\">${order.createdAt}</div></div><div style=\"text-align:right\"><strong>${order.status}</strong></div></div>
        <hr/>
        <div><strong>Pelanggan</strong><div>${order.customer} • ${order.phone}</div><div class=\"muted\">${order.address}</div></div>
        <hr/>
        <div><strong>Item</strong></div>
      `;
      root.appendChild(h);
      let total = 0;
      order.items.forEach(it=>{
        total += it.price*it.qty;
        const r = document.createElement('div'); r.style.display='flex'; r.style.justifyContent='space-between'; r.style.margin='6px 0'; r.innerHTML = `<div>${it.name} x ${it.qty}</div><div>${formatRp(it.price*it.qty)}</div>`; root.appendChild(r);
      })
      const fee = Math.ceil(total*0.02); const grand = total + fee;
      const foot = document.createElement('div'); foot.style.marginTop='10px'; foot.innerHTML = `<hr/><div style="display:flex;justify-content:space-between"><div class=\"muted\">Sub total</div><div>${formatRp(total)}</div></div><div style="display:flex;justify-content:space-between"><div class=\"muted\">Biaya layanan (2%)</div><div>${formatRp(fee)}</div></div><div style="display:flex;justify-content:space-between;font-weight:800;margin-top:8px"><div>Total</div><div>${formatRp(grand)}</div></div><hr/><div><strong>Metode:</strong> ${order.paymentInfo.method}</div><div class=\"muted\">${order.paymentInfo.bank ? 'Bank: '+order.paymentInfo.bank + ' • Nama: ' + order.paymentInfo.a_n : ''}</div>`;
      root.appendChild(foot);

      document.getElementById('receiptModal').style.display='flex';
    }

    document.getElementById('btnCloseReceipt').addEventListener('click', ()=>document.getElementById('receiptModal').style.display='none');

    // Download simple screenshot (html2canvas minimal emulation)
    document.getElementById('btnDownloadReceipt').addEventListener('click', ()=>{
      // Karena di Acode mungkin tidak tersedia html2canvas, kita tawarkan cara manual: cetak halaman dari modal
      const content = document.getElementById('receiptContent').innerText;
      const blob = new Blob([content], { type: 'text/plain' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a'); a.href = url; a.download = 'struk_'+Date.now()+'.txt'; a.click(); URL.revokeObjectURL(url);
      toast('Struk diunduh (txt)');
    })

    // Inisialisasi
    renderProducts(); updateCartUI();

    /* =====================
       PETUNJUK INTEGRASI GATEWAY NYATA (comment/block)
       - Stripe Checkout: Anda perlu server untuk membuat PaymentIntent atau mengembalikan sessionId. Client dapat redirect ke Checkout.
       - Midtrans Snap: server harus memanggil API Midtrans untuk mendapatkan token snapToken, lalu panggil window.snap.pay(token)
       - PayPal: dapat menggunakan Checkout client + server untuk capture order.
       Contoh (Stripe, client-only tidak aman):
         // 1) server membuat session
         // 2) client: window.location = session.url
       Karena Acode menjalankan file statis saja, untuk menghubungkan gateway nyata Anda butuh endpoint server (node/php/python) yang menyimpan kunci rahasia.
    ===================== */

  </script>
</body>
</html>
