const products = [
  {
    id: 1,
    name: 'Premium oil',
    price: 28,
    category: 'beauty',
    rating: 4.9,
    description: 'Yumshatuvchi va qoshimcha parvarish uchun',
    emoji: '🧴',
    tone: 'green',
  },
  {
    id: 2,
    name: 'Smart Watch',
    price: 99,
    category: 'electronics',
    rating: 4.8,
    description: 'Sog‘liq, xabarlar va sport kuzatuvi',
    emoji: '📱',
    tone: 'blue',
  },
  {
    id: 3,
    name: 'Fresh Drink',
    price: 12,
    category: 'food',
    rating: 4.7,
    description: 'Yangi tayyorlangan va vitaminli',
    emoji: '🥤',
    tone: 'yellow',
  },
  {
    id: 4,
    name: 'Home Lamp',
    price: 64,
    category: 'home',
    rating: 4.6,
    description: 'Uyga qulay va zamonaviy yoritish',
    emoji: '💡',
    tone: 'purple',
  },
  {
    id: 5,
    name: 'Travel Bag',
    price: 72,
    category: 'home',
    rating: 4.9,
    description: 'Sayohat uchun mos va ixcham sumka',
    emoji: '🎒',
    tone: 'red',
  },
  {
    id: 6,
    name: 'Noise Cancel',
    price: 135,
    category: 'electronics',
    rating: 4.9,
    description: 'Sovuq ovozlarni qisqartiruvchi quloqchin',
    emoji: '🎧',
    tone: 'blue',
  },
];

const cart = [
  {
    id: 1,
    name: 'Premium oil',
    price: 28,
    qty: 1,
    emoji: '🧴',
    tone: 'green',
  },
  {
    id: 2,
    name: 'Smart Watch',
    price: 99,
    qty: 1,
    emoji: '📱',
    tone: 'blue',
  },
  {
    id: 3,
    name: 'Fresh Drink',
    price: 12,
    qty: 2,
    emoji: '🥤',
    tone: 'yellow',
  },
];

const cartCountEl = document.getElementById('cartCount');
const homeProductsEl = document.getElementById('homeProducts');
const productListEl = document.getElementById('productList');
const cartListEl = document.getElementById('cartList');
const searchInput = document.getElementById('searchInput');
const sortSelect = document.getElementById('sortSelect');
const categoryChips = document.querySelectorAll('.chip');

function updateCartBadge() {
  const totalItems = cart.reduce((sum, item) => sum + item.qty, 0);
  if (cartCountEl) cartCountEl.textContent = totalItems;
}

function renderProductCard(product) {
  return `
    <article class="product-card" data-category="${product.category}">
      <div class="product-thumb ${product.tone}">${product.emoji}</div>
      <div class="product-content">
        <div class="product-meta">
          <span class="product-name">${product.name}</span>
          <span class="product-price">$${product.price}</span>
        </div>
        <p class="product-desc">${product.description}</p>
        <div class="product-footer">
          <span class="product-rating">★ ${product.rating}</span>
          <button class="add-btn" data-id="${product.id}">+ Savatcha</button>
        </div>
      </div>
    </article>
  `;
}

function renderHomeProducts() {
  if (!homeProductsEl) return;
  homeProductsEl.innerHTML = products.slice(0, 4).map(renderProductCard).join('');
}

function getVisibleProducts() {
  let filtered = [...products];

  if (searchInput && searchInput.value.trim()) {
    const q = searchInput.value.trim().toLowerCase();
    filtered = filtered.filter((item) => item.name.toLowerCase().includes(q));
  }

  const selectedCategory = document.querySelector('.chip.active')?.dataset.category || 'all';
  if (selectedCategory !== 'all') {
    filtered = filtered.filter((item) => item.category === selectedCategory);
  }

  const sortBy = sortSelect?.value || 'popular';
  if (sortBy === 'cheapest') filtered.sort((a, b) => a.price - b.price);
  if (sortBy === 'expensive') filtered.sort((a, b) => b.price - a.price);
  if (sortBy === 'new') filtered.reverse();

  return filtered;
}

function renderProductsPage() {
  if (!productListEl) return;
  const items = getVisibleProducts();
  productListEl.innerHTML = items.map(renderProductCard).join('');
}

function renderCartPage() {
  if (!cartListEl) return;

  const subtotal = cart.reduce((sum, item) => sum + item.price * item.qty, 0);
  const total = subtotal + 12;

  document.getElementById('subtotal') && (document.getElementById('subtotal').textContent = `$${subtotal}`);
  document.getElementById('total') && (document.getElementById('total').textContent = `$${total}`);

  cartListEl.innerHTML = cart.map((item) => `
    <div class="cart-item">
      <div class="cart-main">
        <div class="cart-thumb ${item.tone}">${item.emoji}</div>
        <div>
          <strong>${item.name}</strong>
          <span>$${item.price} each</span>
          <div class="cart-controls">
            <button class="qty-btn" data-action="decrease" data-id="${item.id}">-</button>
            <span>${item.qty}</span>
            <button class="qty-btn" data-action="increase" data-id="${item.id}">+</button>
          </div>
        </div>
      </div>
      <strong>$${item.price * item.qty}</strong>
    </div>
  `).join('');
}

function addToCart(productId) {
  const product = products.find((p) => p.id === Number(productId));
  if (!product) return;

  const existing = cart.find((item) => item.id === product.id);
  if (existing) {
    existing.qty += 1;
  } else {
    cart.push({
      id: product.id,
      name: product.name,
      price: product.price,
      qty: 1,
      emoji: product.emoji,
      tone: product.tone,
    });
  }

  updateCartBadge();
  renderCartPage();
}

function changeQty(productId, delta) {
  const item = cart.find((entry) => entry.id === Number(productId));
  if (!item) return;

  item.qty += delta;
  if (item.qty <= 0) {
    const index = cart.findIndex((entry) => entry.id === Number(productId));
    cart.splice(index, 1);
  }

  updateCartBadge();
  renderCartPage();
}

document.addEventListener('click', (event) => {
  const addBtn = event.target.closest('.add-btn');
  if (addBtn) {
    addToCart(addBtn.dataset.id);
    return;
  }

  const qtyBtn = event.target.closest('.qty-btn');
  if (qtyBtn) {
    const id = qtyBtn.dataset.id;
    const action = qtyBtn.dataset.action;
    changeQty(id, action === 'increase' ? 1 : -1);
  }
});

if (searchInput) {
  searchInput.addEventListener('input', renderProductsPage);
}

if (sortSelect) {
  sortSelect.addEventListener('change', renderProductsPage);
}

categoryChips.forEach((chip) => {
  chip.addEventListener('click', () => {
    categoryChips.forEach((item) => item.classList.remove('active'));
    chip.classList.add('active');
    renderProductsPage();
  });
});

renderHomeProducts();
renderProductsPage();
renderCartPage();
updateCartBadge();
