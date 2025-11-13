
<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>SYSTEC</title>
  <style>
    /* Básico, responsivo e legível */
    :root{font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial;}
    body{margin:0;background:#f4f6f8;color:#222;padding:20px;}
    header{display:flex;align-items:center;justify-content:space-between;}
    h1{font-size:1.4rem;margin:0;}
    .container{max-width:1100px;margin:20px auto;}
    .grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:16px;}
    .card{background:#fff;border-radius:10px;padding:12px;box-shadow:0 6px 18px rgba(15,20,25,0.06);}
    .card img{width:100%;height:150px;object-fit:cover;border-radius:8px;}
    .price{font-weight:700;margin-top:6px;}
    .btn{display:inline-block;padding:8px 12px;border-radius:8px;border:0;background:#1f8ef1;color:white;cursor:pointer;}
    .cart{position:fixed;right:18px;bottom:18px;background:#1f8ef1;color:#fff;padding:12px;border-radius:12px;box-shadow:0 6px 18px rgba(15,20,25,0.12);}
    .cart small{display:block;font-size:0.85rem;opacity:0.95;}
    .modal{position:fixed;inset:0;background:rgba(0,0,0,0.4);display:none;align-items:center;justify-content:center;padding:20px;}
    .modal.open{display:flex;}
    .modal .panel{background:#fff;padding:18px;border-radius:10px;max-width:600px;width:100%;}
    .cart-list{max-height:300px;overflow:auto;margin:8px 0;}
    .cart-item{display:flex;justify-content:space-between;padding:8px 0;border-bottom:1px dashed #eee;}
    footer{margin-top:40px;text-align:center;color:#666;font-size:0.9rem;}
  </style>
</head>
<body>
  <header class="container">
    <h1>SYSTEC INFORMÁTICA</h1>
    <div>
      <button id="openCartBtn" class="btn">Carrinho (<span id="cartCount">0</span>)</button>
    </div>
  </header>

  <main class="container">
    <section>
      <h2>Produtos</h2>
      <div id="products" class="grid"></div>
    </section>
  </main>

  <div id="cartFab" class="cart" title="Abrir carrinho">
    <div>🛒 <strong id="cartTotal">R$0,00</strong></div>
    <small id="cartMiniCount">0 itens</small>
  </div>

  <!-- Modal do carrinho -->
  <div id="cartModal" class="modal" aria-hidden="true">
    <div class="panel" role="dialog" aria-modal="true">
      <h3>Seu Carrinho</h3>
      <div id="cartList" class="cart-list"></div>
      <div style="display:flex;justify-content:space-between;align-items:center;margin-top:12px;">
        <strong>Total: <span id="cartFinal">R$0,00</span></strong>
        <div>
          <button id="closeCart" class="btn" style="background:#777;">Fechar</button>
          <button id="checkoutBtn" class="btn">Finalizar compra</button>
        </div>
      </div>
      <div id="checkoutResult" style="margin-top:12px;"></div>
    </div>
  </div>

<script>
/* --- Dados de exemplo (substitua por chamadas à API no futuro) --- */
const PRODUCTS = [
  { id: "p1", name: "Placa Mãe", price: 149.9, img: "file:///C:/Users/lipel/Downloads/Meu%20site/img/Placa%20M%C3%A3e.webp" },
  { id: "p2", name: "Procesador", price: 529.5, img: "C:/Users/lipel/Downloads/Meu%20site/img/shopping.webp" },
  { id: "p3", name: "Memoria RAM", price: 184.9, img: "file:///C:/Users/lipel/Downloads/Meu%20site/img/Memoria.webp" },
  { id: "p4", name: "SSD", price: 129.0, img: "file:///C:/Users/lipel/Downloads/Meu%20site/img/D_NQ_NP_2X_866675-MLB93847868637_092025-F.webp" }
];

/* --- Helpers --- */
const fmt = v => v.toLocaleString('pt-BR',{style:'currency',currency:'BRL'});
const q = sel => document.querySelector(sel);

/* --- Monta grid de produtos --- */
const productsEl = q('#products');
function renderProducts(){
  productsEl.innerHTML = '';
  PRODUCTS.forEach(p=>{
    const div = document.createElement('div');
    div.className = 'card';
    div.innerHTML = `
      <img src="${p.img}" alt="${p.name}">
      <h4>${p.name}</h4>
      <div class="price">${fmt(p.price)}</div>
      <div style="margin-top:8px;">
        <button class="btn addBtn" data-id="${p.id}">Adicionar ao carrinho</button>
      </div>
    `;
    productsEl.appendChild(div);
  });
}

/* --- Carrinho (usa localStorage) --- */
let cart = JSON.parse(localStorage.getItem('cart_v1')||'{}'); // {productId: qty}
function saveCart(){ localStorage.setItem('cart_v1', JSON.stringify(cart)); updateCartUI(); }
function addToCart(id, qty=1){
  cart[id] = (cart[id]||0) + qty;
  saveCart();
}
function removeFromCart(id){
  delete cart[id]; saveCart();
}
function updateQty(id, qty){
  if(qty<=0) removeFromCart(id); else { cart[id]=qty; saveCart(); }
}
function cartItemsDetailed(){
  return Object.entries(cart).map(([id,qty])=>{
    const product = PRODUCTS.find(p=>p.id===id);
    return { ...product, qty, subtotal: product.price*qty };
  });
}
function cartTotal(){
  return cartItemsDetailed().reduce((s,i)=>s+i.subtotal,0);
}

/* --- UI do carrinho --- */
function updateCartUI(){
  const count = Object.values(cart).reduce((s,q)=>s+q,0);
  q('#cartCount').textContent = count;
  q('#cartMiniCount').textContent = `${count} item${count!==1?'s':''}`;
  q('#cartTotal').textContent = fmt(cartTotal());
  q('#cartFinal').textContent = fmt(cartTotal());
  renderCartList();
}
function renderCartList(){
  const el = q('#cartList');
  const items = cartItemsDetailed();
  if(items.length===0){ el.innerHTML = '<p>Seu carrinho está vazio.</p>'; return; }
  el.innerHTML = '';
  items.forEach(item=>{
    const itemEl = document.createElement('div');
    itemEl.className = 'cart-item';
    itemEl.innerHTML = `
      <div>
        <div><strong>${item.name}</strong></div>
        <div style="font-size:0.9rem;color:#666">${fmt(item.price)} x 
          <input type="number" min="1" value="${item.qty}" data-id="${item.id}" style="width:56px;padding:4px;margin-left:6px;">
        </div>
      </div>
      <div style="text-align:right">
        <div>${fmt(item.subtotal)}</div>
        <button data-remove="${item.id}" style="margin-top:6px;background:transparent;border:0;color:#c33;cursor:pointer">Remover</button>
      </div>
    `;
    el.appendChild(itemEl);
  });
}

/* --- Eventos --- */
document.addEventListener('click', e=>{
  if(e.target.matches('.addBtn')){
    addToCart(e.target.dataset.id, 1);
  }
  if(e.target.matches('[data-remove]')){
    removeFromCart(e.target.getAttribute('data-remove'));
  }
});
document.addEventListener('input', e=>{
  if(e.target.matches('.cart-list input[type=number]')){
    const id = e.target.dataset.id;
    const qty = parseInt(e.target.value) || 1;
    updateQty(id, qty);
  }
});
q('#openCartBtn').addEventListener('click', ()=> q('#cartModal').classList.add('open'));
q('#cartFab').addEventListener('click', ()=> q('#cartModal').classList.add('open'));
q('#closeCart').addEventListener('click', ()=> q('#cartModal').classList.remove('open'));

/* Checkout (simulação) */
q('#checkoutBtn').addEventListener('click', ()=>{
  const items = cartItemsDetailed();
  if(items.length===0){ q('#checkoutResult').innerHTML = '<em>Adicione produtos antes de finalizar.</em>'; return; }
  // Simular envio para servidor: aqui você enviaria os itens para seu backend, criar pedido e redirecionar para gateway
  q('#checkoutResult').innerHTML = `<strong>Pedido simulado criado.</strong> Total ${fmt(cartTotal())}. (Integre um gateway para pagamentos reais.)`;
  // opcional: limpar carrinho
  cart = {}; saveCart();
});

/* inicialização */
renderProducts();
updateCartUI();
</script>

<footer class="container">
  <p>SYSTEC IN LTDA - CNPJ: 19.007.486/0001-18
"A inclusão no carrinho não garante o preço e/ou a disponibilidade do produto. Caso os produtos apresentem divergências de valores, o preço válido é o exibido na tela de pagamento. Vendas sujeitas a análise e disponibilidade de estoque".</p>
</footer>
</body>
</html>
