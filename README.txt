// ==========================================================================
// QUADREX v2 — shared interaction layer
// ==========================================================================

(function(){

  /* -------------------------------------------------------------------- */
  /* PRODUCT CATALOG (demo data source for search)                         */
  /* -------------------------------------------------------------------- */
  const CATALOG = [
    { id:'hoodie-black', name:'Heavyweight Hoodie — Black', price:3990, tone:'#111111', cat:'hoodie' },
    { id:'hoodie-stone', name:'Heavyweight Hoodie — Stone', price:3990, tone:'#C4BBAF', cat:'hoodie' },
    { id:'tee-white', name:'Oversized Tee — White', price:1690, tone:'#e9e6df', cat:'tee' },
    { id:'tee-black', name:'Oversized Tee — Black', price:1690, tone:'#0c0c0c', cat:'tee' },
    { id:'polo-white', name:'Pique Polo — White', price:2290, tone:'#efece5', cat:'polo' },
    { id:'polo-olive', name:'Pique Polo — Olive', price:2290, tone:'#556B2F', cat:'polo' },
    { id:'sweat-grey', name:'Heavyweight Sweatshirt — Grey', price:2790, tone:'#909090', cat:'sweatshirt' },
    { id:'sweat-black', name:'Heavyweight Sweatshirt — Black', price:2790, tone:'#101010', cat:'sweatshirt' },
  ];

  document.addEventListener('DOMContentLoaded', () => {

    /* ================= PRELOADER ================= */
    const preloader = document.querySelector('.preloader');
    if(preloader){
      document.body.classList.add('lock');
      window.addEventListener('load', () => {
        setTimeout(()=>{
          preloader.classList.add('done');
          document.body.classList.remove('lock');
        }, 350);
      });
      // safety fallback in case 'load' already fired
      setTimeout(()=>{ preloader.classList.add('done'); document.body.classList.remove('lock'); }, 2200);
    }

    /* ================= CUSTOM CURSOR (desktop only) ================= */
    if(window.matchMedia('(hover: hover) and (pointer: fine)').matches){
      const dot = document.createElement('div'); dot.className = 'cursor-dot';
      const ring = document.createElement('div'); ring.className = 'cursor-ring';
      document.body.append(dot, ring);
      let rx=0, ry=0, mx=0, my=0;
      window.addEventListener('mousemove', (e)=>{
        mx = e.clientX; my = e.clientY;
        dot.style.left = mx+'px'; dot.style.top = my+'px';
      });
      (function loop(){
        rx += (mx-rx)*0.16; ry += (my-ry)*0.16;
        ring.style.left = rx+'px'; ring.style.top = ry+'px';
        requestAnimationFrame(loop);
      })();
      document.querySelectorAll('a, button, .product-card, .filter-chip, input, select, textarea').forEach(el=>{
        el.addEventListener('mouseenter', ()=> ring.classList.add('hover'));
        el.addEventListener('mouseleave', ()=> ring.classList.remove('hover'));
      });
    }

    /* ================= MAGNETIC BUTTONS (desktop only) ================= */
    if(window.matchMedia('(hover: hover) and (pointer: fine)').matches){
      document.querySelectorAll('.btn, .quick-add').forEach(btn=>{
        btn.classList.add('magnetic');
        btn.addEventListener('mousemove', (e)=>{
          const r = btn.getBoundingClientRect();
          const x = e.clientX - r.left - r.width/2;
          const y = e.clientY - r.top - r.height/2;
          btn.style.transform = `translate(${x*0.14}px, ${y*0.28}px)`;
        });
        btn.addEventListener('mouseleave', ()=>{ btn.style.transform = ''; });
      });
    }

    /* ================= NAV: scroll state + hide-on-scroll-down ================= */
    const nav = document.querySelector('.nav');
    let lastY = window.scrollY;
    const onScroll = () => {
      if(!nav) return;
      const y = window.scrollY;
      nav.classList.toggle('scrolled', y > 40);
      if(y > 200 && y > lastY){ nav.classList.add('nav-hidden'); }
      else { nav.classList.remove('nav-hidden'); }
      lastY = y;
    };
    onScroll();
    window.addEventListener('scroll', onScroll, { passive:true });

    /* ================= STAGGERED SCROLL REVEALS ================= */
    const revealGroups = document.querySelectorAll('.reveal');
    revealGroups.forEach(group=>{
      const children = group.children;
      if(children.length > 1 && (group.classList.contains('product-grid') || group.classList.contains('collections-grid') || group.classList.contains('materials-grid') || group.classList.contains('val-row'))){
        Array.from(children).forEach((child, i)=> child.style.setProperty('--d', (i*0.08)+'s'));
      }
    });
    if('IntersectionObserver' in window && revealGroups.length){
      const io = new IntersectionObserver((entries)=>{
        entries.forEach(e=>{
          if(e.isIntersecting){ e.target.classList.add('in'); io.unobserve(e.target); }
        });
      }, { threshold:.15 });
      revealGroups.forEach(el=>io.observe(el));
    } else {
      revealGroups.forEach(el=>el.classList.add('in'));
    }

    /* ================= MOBILE MENU ================= */
    const burger = document.querySelector('.nav-burger');
    const mobileMenu = document.querySelector('.mobile-menu');
    if(burger && mobileMenu){
      burger.addEventListener('click', ()=>{
        mobileMenu.classList.toggle('open');
        document.body.classList.toggle('lock', mobileMenu.classList.contains('open'));
      });
      mobileMenu.querySelectorAll('a').forEach(a=>a.addEventListener('click', ()=>{
        mobileMenu.classList.remove('open');
        document.body.classList.remove('lock');
      }));
    }

    /* ================= CART STATE ================= */
    const cartState = {
      items: [
        { id:'hood-blk', name:'Heavyweight Hoodie', variant:'Black · L', price:3990, qty:1, tone:'#111' },
        { id:'tee-wht', name:'Oversized Tee', variant:'White · M', price:1690, qty:2, tone:'#e9e6df' },
      ],
      freeShippingTarget: 4999,
    };

    const overlay = document.querySelector('.cart-overlay');
    const drawer = document.querySelector('.cart-drawer');

    function renderCart(){
      if(!drawer) return;
      const itemsWrap = drawer.querySelector('.cart-items');
      const countEls = document.querySelectorAll('.cart-count, #cartCount');
      const subtotalEl = drawer.querySelector('.subtotal .val');
      const progressBar = drawer.querySelector('.cart-progress-bar span');
      const progressText = drawer.querySelector('.cart-progress p');

      const totalQty = cartState.items.reduce((s,i)=>s+i.qty,0);
      const subtotal = cartState.items.reduce((s,i)=>s+i.qty*i.price,0);

      countEls.forEach(el=>{ el.textContent = totalQty; el.classList.toggle('show', totalQty>0); });
      if(subtotalEl) subtotalEl.textContent = '₹' + subtotal.toLocaleString('en-IN');
      if(progressBar) progressBar.style.width = Math.min(100,(subtotal/cartState.freeShippingTarget)*100) + '%';
      if(progressText){
        const remaining = cartState.freeShippingTarget - subtotal;
        progressText.innerHTML = remaining > 0
          ? `Add <strong>₹${remaining.toLocaleString('en-IN')}</strong> more for free shipping`
          : `<strong>You've unlocked free shipping</strong>`;
      }
      if(itemsWrap){
        itemsWrap.innerHTML = cartState.items.length ? cartState.items.map((item, idx)=>`
          <div class="cart-item" data-idx="${idx}">
            <div class="thumb" style="background:${item.tone}"></div>
            <div class="cart-item-info">
              <h4>${item.name}</h4>
              <p class="meta">${item.variant}</p>
              <div class="row">
                <div class="qty-stepper">
                  <button data-action="dec" aria-label="Decrease quantity">−</button>
                  <span>${item.qty}</span>
                  <button data-action="inc" aria-label="Increase quantity">+</button>
                </div>
                <p class="meta">₹${(item.price*item.qty).toLocaleString('en-IN')}</p>
              </div>
            </div>
          </div>
        `).join('') : `<div class="empty-state"><span class="q-mark"></span><p>Your bag is empty.<br>Add something built different.</p></div>`;
      }
    }

    function openCart(){ overlay?.classList.add('open'); drawer?.classList.add('open'); document.body.classList.add('lock'); }
    function closeCart(){ overlay?.classList.remove('open'); drawer?.classList.remove('open'); document.body.classList.remove('lock'); }

    document.querySelectorAll('[data-open-cart]').forEach(t=>t.addEventListener('click',(e)=>{ e.preventDefault(); openCart(); }));
    document.querySelectorAll('[data-close-cart]').forEach(t=>t.addEventListener('click', closeCart));
    overlay?.addEventListener('click', closeCart);
    drawer?.addEventListener('click', (e)=>{
      const btn = e.target.closest('button[data-action]');
      if(!btn) return;
      const row = btn.closest('.cart-item');
      const idx = Number(row.dataset.idx);
      if(btn.dataset.action === 'inc') cartState.items[idx].qty++;
      if(btn.dataset.action === 'dec') cartState.items[idx].qty = Math.max(1, cartState.items[idx].qty - 1);
      renderCart();
    });

    /* ---- quick add (grid + PDP) ---- */
    document.querySelectorAll('.quick-add').forEach(btn=>{
      btn.addEventListener('click', (e)=>{
        e.preventDefault();
        const card = btn.closest('.product-card');
        const name = card?.querySelector('.product-info h3')?.textContent || document.querySelector('.pdp-info h1')?.textContent || 'Item';
        const priceText = card?.querySelector('.price')?.textContent || document.querySelector('.pdp-price')?.textContent || '₹0';
        const price = Number(priceText.replace(/[^\d]/g,'')) || 1990;
        cartState.items.push({ id:name+Date.now(), name, variant:'Selected size', price, qty:1, tone:'#c4bbaf' });
        renderCart();
        const original = btn.textContent;
        btn.classList.add('added');
        btn.textContent = 'Added ✓';
        setTimeout(()=>{ btn.textContent = original; btn.classList.remove('added'); }, 1300);
        openCart();
      });
    });

    renderCart();

    /* ================= WISHLIST (persisted via localStorage) ================= */
    const WL_KEY = 'quadrex_wishlist';
    function loadWishlist(){ try{ return JSON.parse(localStorage.getItem(WL_KEY)) || []; }catch(e){ return []; } }
    function saveWishlist(list){ try{ localStorage.setItem(WL_KEY, JSON.stringify(list)); }catch(e){} }
    let wishlist = loadWishlist();

    const wOverlay = document.querySelector('.wishlist-overlay');
    const wDrawer = document.querySelector('.wishlist-drawer');

    function findProductMeta(name){
      return CATALOG.find(p => name && name.toLowerCase().includes(p.name.split(' — ')[0].toLowerCase().split(' ').slice(-1)[0])) || null;
    }

    function renderWishlist(){
      const countEls = document.querySelectorAll('#wishlistCount');
      countEls.forEach(el=>{ el.textContent = wishlist.length; el.classList.toggle('show', wishlist.length>0); });

      // sync heart icons on page to saved state
      document.querySelectorAll('.wishlist-btn').forEach(btn=>{
        const card = btn.closest('.product-card, .pdp-info');
        const name = card?.querySelector('h3, h1')?.textContent?.trim();
        if(name && wishlist.some(w=>w.name===name)){ btn.classList.add('active'); }
      });

      if(!wDrawer) return;
      const wrap = wDrawer.querySelector('#wishlistItems');
      if(!wrap) return;
      wrap.innerHTML = wishlist.length ? wishlist.map((item,idx)=>`
        <div class="wishlist-item" data-idx="${idx}">
          <div class="thumb" style="background:${item.tone||'#ccc'}"></div>
          <div class="wishlist-item-info">
            <h4>${item.name}</h4>
            <p class="meta">₹${item.price?.toLocaleString('en-IN')||''}</p>
            <button class="wishlist-move" data-move="${idx}">Move to Bag</button>
          </div>
        </div>
      `).join('') : `<div class="empty-state"><span class="q-mark"></span><p>Nothing saved yet.<br>Tap the heart on any piece.</p></div>`;
    }

    function toggleWishlist(name, price, tone){
      const existingIdx = wishlist.findIndex(w=>w.name===name);
      if(existingIdx > -1){ wishlist.splice(existingIdx,1); }
      else{ wishlist.push({ name, price, tone }); }
      saveWishlist(wishlist);
      renderWishlist();
      return existingIdx === -1; // true if now added
    }

    document.querySelectorAll('.wishlist-btn').forEach(btn=>{
      btn.addEventListener('click', (e)=>{
        e.preventDefault(); e.stopPropagation();
        const card = btn.closest('.product-card, .pdp-info');
        const name = card?.querySelector('h3, h1')?.textContent?.trim() || 'Item';
        const priceEl = card?.querySelector('.price, .pdp-price');
        const price = Number((priceEl?.textContent||'').replace(/[^\d]/g,'')) || 0;
        const mediaEl = card?.querySelector('.card-media.main, .gallery-main .plate');
        const tone = mediaEl ? getComputedStyle(mediaEl).backgroundColor : '#ccc';
        const nowActive = toggleWishlist(name, price, tone);
        btn.classList.toggle('active', nowActive);
      });
    });

    function openWishlist(){ wOverlay?.classList.add('open'); wDrawer?.classList.add('open'); document.body.classList.add('lock'); }
    function closeWishlist(){ wOverlay?.classList.remove('open'); wDrawer?.classList.remove('open'); document.body.classList.remove('lock'); }
    document.querySelectorAll('[data-open-wishlist]').forEach(t=>t.addEventListener('click',(e)=>{ e.preventDefault(); openWishlist(); }));
    document.querySelectorAll('[data-close-wishlist]').forEach(t=>t.addEventListener('click', closeWishlist));
    wOverlay?.addEventListener('click', closeWishlist);

    wDrawer?.addEventListener('click', (e)=>{
      const moveBtn = e.target.closest('[data-move]');
      if(!moveBtn) return;
      const idx = Number(moveBtn.dataset.move);
      const item = wishlist[idx];
      if(item){
        cartState.items.push({ id:item.name+Date.now(), name:item.name, variant:'Selected size', price:item.price, qty:1, tone:item.tone });
        wishlist.splice(idx,1);
        saveWishlist(wishlist);
        renderCart(); renderWishlist();
      }
    });

    renderWishlist();

    /* ================= SEARCH OVERLAY ================= */
    const searchOverlay = document.querySelector('.search-overlay');
    const searchInput = document.querySelector('#searchInput');
    const searchResults = document.querySelector('#searchResults');

    function openSearch(){
      searchOverlay?.classList.add('open');
      document.body.classList.add('lock');
      setTimeout(()=>searchInput?.focus(), 350);
    }
    function closeSearch(){ searchOverlay?.classList.remove('open'); document.body.classList.remove('lock'); }

    document.querySelectorAll('[data-open-search]').forEach(t=>t.addEventListener('click',(e)=>{ e.preventDefault(); openSearch(); }));
    document.querySelectorAll('[data-close-search]').forEach(t=>t.addEventListener('click', closeSearch));
    document.addEventListener('keydown', (e)=>{
      if(e.key === 'Escape'){ closeSearch(); }
      if((e.metaKey||e.ctrlKey) && e.key.toLowerCase()==='k'){ e.preventDefault(); openSearch(); }
    });

    function runSearch(q){
      if(!searchResults) return;
      const term = q.trim().toLowerCase();
      if(!term){ searchResults.innerHTML=''; return; }
      const matches = CATALOG.filter(p => p.name.toLowerCase().includes(term) || p.cat.includes(term));
      searchResults.innerHTML = matches.length ? matches.map(p=>`
        <a href="product.html?id=${p.id}" class="search-result-item">
          <div class="thumb"><div class="plate" style="position:absolute;inset:0;background:${p.tone}"></div></div>
          <div>
            <h4>${p.name}</h4>
            <p class="price">₹${p.price.toLocaleString('en-IN')}</p>
          </div>
        </a>
      `).join('') : `<div class="search-empty">No results for "${q}" — try "hoodie", "tee", "polo" or "sweatshirt".</div>`;
    }

    searchInput?.addEventListener('input', (e)=> runSearch(e.target.value));
    document.querySelectorAll('.search-chip').forEach(chip=>{
      chip.addEventListener('click', ()=>{
        if(searchInput){ searchInput.value = chip.dataset.q; searchInput.focus(); }
        runSearch(chip.dataset.q);
      });
    });

    /* ================= PRODUCT GALLERY (PDP) ================= */
    const thumbs = document.querySelectorAll('.gallery-thumb');
    const mainStage = document.querySelector('.gallery-main .plate');
    thumbs.forEach(t=>{
      t.addEventListener('click', ()=>{
        thumbs.forEach(x=>x.classList.remove('active'));
        t.classList.add('active');
        if(mainStage) mainStage.style.background = t.dataset.tone || '#000';
      });
    });

    /* ================= PDP TABS ================= */
    document.querySelectorAll('.pdp-tab-btn').forEach(btn=>{
      btn.addEventListener('click', ()=>{
        const target = btn.dataset.tab;
        document.querySelectorAll('.pdp-tab-btn').forEach(b=>b.classList.remove('active'));
        document.querySelectorAll('.pdp-tab-panel').forEach(p=>p.classList.remove('active'));
        btn.classList.add('active');
        document.querySelector(`.pdp-tab-panel[data-tab="${target}"]`)?.classList.add('active');
      });
    });

    /* ================= SIZE SELECTOR ================= */
    document.querySelectorAll('.size-option').forEach(opt=>{
      opt.addEventListener('click', ()=>{
        if(opt.classList.contains('disabled')) return;
        document.querySelectorAll('.size-option').forEach(o=>o.classList.remove('active'));
        opt.classList.add('active');
      });
    });

    /* ================= SHOP FILTERS ================= */
    document.querySelector('[data-filter-toggle]')?.addEventListener('click', ()=>{
      document.querySelector('.filter-panel')?.classList.toggle('open');
    });
    document.querySelectorAll('.filter-chip').forEach(chip=>{
      chip.addEventListener('click', ()=> chip.classList.toggle('active'));
    });

  });

})();
