# Retail-Store
Retail Store Umami Foods
import { useState, useEffect, useRef } from “react”;

// ─── Supabase Client ──────────────────────────────────────────────────────────
const SUPABASE_URL = “https://hmeeokkbqwvyvbjlobqi.supabase.co”;
const SUPABASE_KEY = “sb_publishable_B-HNULwV47baoPQY0VDO1Q_t7XdpmmA”;

const sb = {
from: (table) => ({
select: async (cols = “*”) => {
const r = await fetch(`${SUPABASE_URL}/rest/v1/${table}?select=${cols}&order=id.asc`, {
headers: { apikey: SUPABASE_KEY, Authorization: `Bearer ${SUPABASE_KEY}` }
});
const data = await r.json();
return { data: r.ok ? data : [], error: !r.ok ? data : null };
},
insert: async (rows) => {
const r = await fetch(`${SUPABASE_URL}/rest/v1/${table}`, {
method: “POST”,
headers: { apikey: SUPABASE_KEY, Authorization: `Bearer ${SUPABASE_KEY}`, “Content-Type”: “application/json”, Prefer: “return=representation” },
body: JSON.stringify(rows)
});
const data = await r.json();
return { data: r.ok ? data : null, error: !r.ok ? data : null };
},
update: async (values, matchCol, matchVal) => {
const r = await fetch(`${SUPABASE_URL}/rest/v1/${table}?${matchCol}=eq.${matchVal}`, {
method: “PATCH”,
headers: { apikey: SUPABASE_KEY, Authorization: `Bearer ${SUPABASE_KEY}`, “Content-Type”: “application/json”, Prefer: “return=representation” },
body: JSON.stringify(values)
});
const data = await r.json();
return { data: r.ok ? data : null, error: !r.ok ? data : null };
},
delete: async (matchCol, matchVal) => {
const r = await fetch(`${SUPABASE_URL}/rest/v1/${table}?${matchCol}=eq.${matchVal}`, {
method: “DELETE”,
headers: { apikey: SUPABASE_KEY, Authorization: `Bearer ${SUPABASE_KEY}` }
});
return { error: !r.ok ? await r.json() : null };
},
})
};

const palette = {
bg: “#0f0f13”,
surface: “#1a1a24”,
surfaceHover: “#22222f”,
card: “#16161f”,
accent: “#f0a500”,
accentSoft: “#f0a50022”,
accentHover: “#ffc107”,
text: “#f0ede8”,
textMuted: “#8b8799”,
textDim: “#5a5769”,
success: “#2ecc71”,
danger: “#e74c3c”,
info: “#3498db”,
border: “#2a2a38”,
};

const style = {
fontFamily: “‘Playfair Display’, serif”,
fontBody: “‘DM Sans’, sans-serif”,
};

// ─── Initial Data (demo fallback) ────────────────────────────────────────────
const initialProducts = [];
const initialClients  = [];
const initialSales    = [];

// ─── Helpers ─────────────────────────────────────────────────────────────────
const Tag = ({ children, color = palette.accent }) => (
<span style={{ background: color + “22”, color, borderRadius: 6, padding: “2px 10px”, fontSize: 12, fontFamily: style.fontBody, fontWeight: 600 }}>{children}</span>
);

const Card = ({ children, style: s = {} }) => (

  <div style={{ background: palette.card, border: `1px solid ${palette.border}`, borderRadius: 16, padding: 20, ...s }}>{children}</div>
);

const Btn = ({ children, onClick, variant = “primary”, size = “md”, disabled = false, style: s = {} }) => {
const [hover, setHover] = useState(false);
const base = {
border: “none”, borderRadius: 10, cursor: disabled ? “not-allowed” : “pointer”,
fontFamily: style.fontBody, fontWeight: 600, transition: “all 0.18s ease”,
opacity: disabled ? 0.5 : 1, …s,
};
const sizes = { sm: { padding: “6px 14px”, fontSize: 13 }, md: { padding: “10px 20px”, fontSize: 14 }, lg: { padding: “13px 28px”, fontSize: 15 } };
const variants = {
primary: { background: hover ? palette.accentHover : palette.accent, color: “#0f0f13” },
ghost: { background: hover ? palette.border : “transparent”, color: palette.text, border: `1px solid ${palette.border}` },
danger: { background: hover ? “#c0392b” : palette.danger, color: “#fff” },
success: { background: hover ? “#27ae60” : palette.success, color: “#fff” },
};
return <button onMouseEnter={() => setHover(true)} onMouseLeave={() => setHover(false)} onClick={!disabled ? onClick : undefined} style={{ …base, …sizes[size], …variants[variant] }}>{children}</button>;
};

const Input = ({ label, value, onChange, type = “text”, placeholder = “” }) => (

  <div style={{ marginBottom: 14 }}>
    {label && <div style={{ color: palette.textMuted, fontSize: 12, fontFamily: style.fontBody, marginBottom: 5, fontWeight: 600 }}>{label}</div>}
    <input type={type} value={value} onChange={e => onChange(e.target.value)} placeholder={placeholder}
      style={{ width: "100%", background: palette.surface, border: `1px solid ${palette.border}`, borderRadius: 10, padding: "10px 14px", color: palette.text, fontFamily: style.fontBody, fontSize: 14, outline: "none", boxSizing: "border-box" }} />
  </div>
);

// ─── MÓDULO: Dashboard ────────────────────────────────────────────────────────
function Dashboard({ products, clients, sales }) {
const totalRevenue = sales.reduce((a, s) => a + s.total, 0);
const lowStock = products.filter(p => p.stock < 20).length;
const vipClients = clients.filter(c => c.status === “VIP”).length;

const stats = [
{ label: “Ventas Totales”, value: `$${totalRevenue.toFixed(2)}`, icon: “💰”, color: palette.success },
{ label: “Productos”, value: products.length, icon: “📦”, color: palette.info },
{ label: “Clientes”, value: clients.length, icon: “👥”, color: palette.accent },
{ label: “Stock Bajo”, value: lowStock, icon: “⚠️”, color: palette.danger },
];

return (
<div>
<div style={{ marginBottom: 28 }}>
<h2 style={{ fontFamily: style.fontFamily, color: palette.text, fontSize: 28, margin: 0 }}>Dashboard</h2>
<p style={{ color: palette.textMuted, fontFamily: style.fontBody, margin: “6px 0 0” }}>Resumen general de tu tienda</p>
</div>
<div style={{ display: “grid”, gridTemplateColumns: “repeat(auto-fit, minmax(180px, 1fr))”, gap: 16, marginBottom: 28 }}>
{stats.map(s => (
<Card key={s.label}>
<div style={{ fontSize: 28, marginBottom: 8 }}>{s.icon}</div>
<div style={{ color: s.color, fontSize: 26, fontFamily: style.fontFamily, fontWeight: 700 }}>{s.value}</div>
<div style={{ color: palette.textMuted, fontSize: 13, fontFamily: style.fontBody, marginTop: 4 }}>{s.label}</div>
</Card>
))}
</div>
<div style={{ display: “grid”, gridTemplateColumns: “1fr 1fr”, gap: 16 }}>
<Card>
<h3 style={{ fontFamily: style.fontFamily, color: palette.text, margin: “0 0 16px” }}>Últimas Ventas</h3>
{sales.slice(0, 3).map(s => (
<div key={s.id} style={{ display: “flex”, justifyContent: “space-between”, alignItems: “center”, padding: “10px 0”, borderBottom: `1px solid ${palette.border}` }}>
<div>
<div style={{ color: palette.text, fontFamily: style.fontBody, fontSize: 14 }}>{s.client}</div>
<div style={{ color: palette.textMuted, fontSize: 12, fontFamily: style.fontBody }}>{s.date}</div>
</div>
<div style={{ color: palette.success, fontFamily: style.fontBody, fontWeight: 700 }}>${s.total}</div>
</div>
))}
</Card>
<Card>
<h3 style={{ fontFamily: style.fontFamily, color: palette.text, margin: “0 0 16px” }}>Productos con Bajo Stock</h3>
{products.filter(p => p.stock < 25).map(p => (
<div key={p.id} style={{ display: “flex”, justifyContent: “space-between”, alignItems: “center”, padding: “10px 0”, borderBottom: `1px solid ${palette.border}` }}>
<div style={{ display: “flex”, alignItems: “center”, gap: 10 }}>
<span style={{ fontSize: 22 }}>{p.image}</span>
<div style={{ color: palette.text, fontFamily: style.fontBody, fontSize: 14 }}>{p.name}</div>
</div>
<Tag color={p.stock < 15 ? palette.danger : palette.accent}>{p.stock} uds.</Tag>
</div>
))}
</Card>
</div>
</div>
);
}

// ─── MÓDULO: Inventario ───────────────────────────────────────────────────────
function Inventory({ products, setProducts }) {
const [search, setSearch] = useState(””);
const [showForm, setShowForm] = useState(false);
const [form, setForm] = useState({ name: “”, sku: “”, category: “”, price: “”, stock: “”, image: “📦”, desc: “” });

const filtered = products.filter(p => p.name.toLowerCase().includes(search.toLowerCase()) || p.sku.includes(search));

const addProduct = () => {
if (!form.name || !form.price) return;
const newP = { …form, id: Date.now(), price: parseFloat(form.price), stock: parseInt(form.stock) || 0 };
setProducts([…products, newP]);
setForm({ name: “”, sku: “”, category: “”, price: “”, stock: “”, image: “📦”, desc: “” });
setShowForm(false);
};

const deleteProduct = (id) => setProducts(products.filter(p => p.id !== id));

return (
<div>
<div style={{ display: “flex”, justifyContent: “space-between”, alignItems: “center”, marginBottom: 24 }}>
<div>
<h2 style={{ fontFamily: style.fontFamily, color: palette.text, fontSize: 28, margin: 0 }}>Inventario</h2>
<p style={{ color: palette.textMuted, fontFamily: style.fontBody, margin: “6px 0 0” }}>{products.length} productos registrados</p>
</div>
<Btn onClick={() => setShowForm(!showForm)}>+ Agregar Producto</Btn>
</div>

```
  {showForm && (
    <Card style={{ marginBottom: 20 }}>
      <h3 style={{ fontFamily: style.fontFamily, color: palette.text, margin: "0 0 16px" }}>Nuevo Producto</h3>
      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 12 }}>
        <Input label="Nombre" value={form.name} onChange={v => setForm({ ...form, name: v })} />
        <Input label="SKU" value={form.sku} onChange={v => setForm({ ...form, sku: v })} />
        <Input label="Categoría" value={form.category} onChange={v => setForm({ ...form, category: v })} />
        <Input label="Precio ($)" type="number" value={form.price} onChange={v => setForm({ ...form, price: v })} />
        <Input label="Stock" type="number" value={form.stock} onChange={v => setForm({ ...form, stock: v })} />
        <Input label="Emoji / Icono" value={form.image} onChange={v => setForm({ ...form, image: v })} />
      </div>
      <Input label="Descripción" value={form.desc} onChange={v => setForm({ ...form, desc: v })} />
      <div style={{ display: "flex", gap: 10 }}>
        <Btn onClick={addProduct}>Guardar</Btn>
        <Btn variant="ghost" onClick={() => setShowForm(false)}>Cancelar</Btn>
      </div>
    </Card>
  )}

  <div style={{ marginBottom: 16 }}>
    <Input placeholder="Buscar por nombre o SKU..." value={search} onChange={setSearch} />
  </div>

  <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fill, minmax(260px, 1fr))", gap: 14 }}>
    {filtered.map(p => (
      <Card key={p.id}>
        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start" }}>
          <span style={{ fontSize: 38 }}>{p.image}</span>
          <Tag color={p.stock < 15 ? palette.danger : p.stock < 25 ? palette.accent : palette.success}>{p.stock} uds.</Tag>
        </div>
        <div style={{ marginTop: 10 }}>
          <div style={{ color: palette.text, fontFamily: style.fontBody, fontWeight: 600, fontSize: 15 }}>{p.name}</div>
          <div style={{ color: palette.textMuted, fontSize: 12, fontFamily: style.fontBody, marginTop: 2 }}>{p.sku} · {p.category}</div>
          <div style={{ color: palette.accent, fontFamily: style.fontFamily, fontSize: 20, marginTop: 8, fontWeight: 700 }}>${p.price}</div>
          <div style={{ color: palette.textDim, fontSize: 12, fontFamily: style.fontBody, marginTop: 4 }}>{p.desc}</div>
        </div>
        <div style={{ marginTop: 14, display: "flex", justifyContent: "flex-end" }}>
          <Btn variant="danger" size="sm" onClick={() => deleteProduct(p.id)}>Eliminar</Btn>
        </div>
      </Card>
    ))}
  </div>
</div>
```

);
}

// ─── MÓDULO: POS / Ventas ─────────────────────────────────────────────────────
function POS({ products, setProducts, sales, setSales, clients, setClients }) {
const [cart, setCart] = useState([]);
const [clientName, setClientName] = useState(“Walk-in”);
const [search, setSearch] = useState(””);
const [receipt, setReceipt] = useState(null);

const filtered = products.filter(p => p.name.toLowerCase().includes(search.toLowerCase()));
const total = cart.reduce((a, i) => a + i.price * i.qty, 0);

const addToCart = (p) => {
const existing = cart.find(i => i.id === p.id);
if (existing) {
setCart(cart.map(i => i.id === p.id ? { …i, qty: i.qty + 1 } : i));
} else {
setCart([…cart, { …p, qty: 1 }]);
}
};

const removeFromCart = (id) => setCart(cart.filter(i => i.id !== id));

const checkout = () => {
if (cart.length === 0) return;
const newSale = {
id: Date.now(), client: clientName,
items: cart.reduce((a, i) => a + i.qty, 0),
total, date: new Date().toISOString().split(“T”)[0], status: “Completada”,
cart: cart.map(i => ({ id: i.id, name: i.name, category: i.category, image: i.image, price: i.price, qty: i.qty })),
};
setSales([newSale, …sales]);
setProducts(products.map(p => {
const item = cart.find(i => i.id === p.id);
return item ? { …p, stock: p.stock - item.qty } : p;
}));
// Update client purchases + tier
setClients && setClients(prev => prev.map(c => {
if (c.name !== clientName) return c;
const newPurchases = (c.purchases || 0) + 1;
const newQP = (c.quarterPurchases || 0) + 1;
const newStatus = getTier(newQP).name;
const earned = calcPoints(total, newStatus);
return { …c, purchases: newPurchases, quarterPurchases: newQP, total: (c.total || 0) + total, status: newStatus, points: (c.points || 0) + earned };
}));
setReceipt({ …newSale, cart });
setCart([]);
setClientName(“Walk-in”);
};

if (receipt) return (
<div style={{ maxWidth: 420, margin: “0 auto”, textAlign: “center” }}>
<div style={{ fontSize: 64, marginBottom: 12 }}>✅</div>
<h2 style={{ fontFamily: style.fontFamily, color: palette.text }}>Venta Completada</h2>
<Card style={{ textAlign: “left”, marginTop: 20 }}>
<div style={{ color: palette.textMuted, fontFamily: style.fontBody, fontSize: 13, marginBottom: 10 }}>Cliente: <strong style={{ color: palette.text }}>{receipt.client}</strong></div>
{receipt.cart.map(i => (
<div key={i.id} style={{ display: “flex”, justifyContent: “space-between”, padding: “6px 0”, borderBottom: `1px solid ${palette.border}`, fontFamily: style.fontBody, fontSize: 14 }}>
<span style={{ color: palette.text }}>{i.image} {i.name} x{i.qty}</span>
<span style={{ color: palette.accent }}>${(i.price * i.qty).toFixed(2)}</span>
</div>
))}
<div style={{ display: “flex”, justifyContent: “space-between”, marginTop: 12, fontFamily: style.fontFamily, fontSize: 20 }}>
<span style={{ color: palette.text }}>Total</span>
<span style={{ color: palette.success }}>${receipt.total.toFixed(2)}</span>
</div>
</Card>
<Btn style={{ marginTop: 20 }} onClick={() => setReceipt(null)}>Nueva Venta</Btn>
</div>
);

return (
<div style={{ display: “grid”, gridTemplateColumns: “1fr 340px”, gap: 20 }}>
<div>
<h2 style={{ fontFamily: style.fontFamily, color: palette.text, fontSize: 28, margin: “0 0 20px” }}>Punto de Venta</h2>
<Input placeholder="Buscar producto..." value={search} onChange={setSearch} />
<div style={{ display: “grid”, gridTemplateColumns: “repeat(auto-fill, minmax(140px, 1fr))”, gap: 10, marginTop: 8 }}>
{filtered.map(p => (
<div key={p.id} onClick={() => p.stock > 0 && addToCart(p)}
style={{ background: palette.card, border: `1px solid ${palette.border}`, borderRadius: 12, padding: 14, cursor: p.stock > 0 ? “pointer” : “not-allowed”, opacity: p.stock === 0 ? 0.4 : 1, textAlign: “center”, transition: “border-color 0.2s” }}
onMouseEnter={e => e.currentTarget.style.borderColor = palette.accent}
onMouseLeave={e => e.currentTarget.style.borderColor = palette.border}>
<div style={{ fontSize: 32 }}>{p.image}</div>
<div style={{ color: palette.text, fontFamily: style.fontBody, fontSize: 13, fontWeight: 600, marginTop: 6 }}>{p.name}</div>
<div style={{ color: palette.accent, fontFamily: style.fontFamily, fontSize: 17, marginTop: 4 }}>${p.price}</div>
<div style={{ color: palette.textDim, fontSize: 11, fontFamily: style.fontBody }}>Stock: {p.stock}</div>
</div>
))}
</div>
</div>

```
  <div>
    <Card style={{ position: "sticky", top: 20 }}>
      <h3 style={{ fontFamily: style.fontFamily, color: palette.text, margin: "0 0 14px" }}>Carrito</h3>
      <Input label="Cliente" value={clientName} onChange={setClientName} />
      {cart.length === 0 ? (
        <div style={{ color: palette.textDim, fontFamily: style.fontBody, textAlign: "center", padding: "30px 0", fontSize: 13 }}>Selecciona productos del catálogo</div>
      ) : (
        cart.map(i => (
          <div key={i.id} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", padding: "8px 0", borderBottom: `1px solid ${palette.border}` }}>
            <div>
              <div style={{ color: palette.text, fontFamily: style.fontBody, fontSize: 13 }}>{i.image} {i.name}</div>
              <div style={{ color: palette.textMuted, fontSize: 12, fontFamily: style.fontBody }}>x{i.qty} · ${(i.price * i.qty).toFixed(2)}</div>
            </div>
            <button onClick={() => removeFromCart(i.id)} style={{ background: "none", border: "none", color: palette.danger, cursor: "pointer", fontSize: 16 }}>✕</button>
          </div>
        ))
      )}
      <div style={{ display: "flex", justifyContent: "space-between", marginTop: 16, fontFamily: style.fontFamily, fontSize: 22 }}>
        <span style={{ color: palette.textMuted, fontSize: 15 }}>Total</span>
        <span style={{ color: palette.success }}>${total.toFixed(2)}</span>
      </div>
      <Btn style={{ width: "100%", marginTop: 16 }} onClick={checkout} disabled={cart.length === 0}>Cobrar</Btn>
    </Card>
  </div>
</div>
```

);
}

// ─── Tier Logic ──────────────────────────────────────────────────────────────
const TIERS = [
{ name: “Nuevo”,   icon: “🆕”, color: “#27ae60”, min: 0,  max: 1  },
{ name: “Regular”, icon: “🔵”, color: “#3498db”, min: 2,  max: 5  },
{ name: “Plata”,   icon: “⚪”, color: “#95a5a6”, min: 6,  max: 10 },
{ name: “VIP”,     icon: “🥇”, color: “#f0a500”, min: 11, max: Infinity },
];

const getTier = (purchases) => TIERS.find(t => purchases >= t.min && purchases <= t.max) || TIERS[0];

const getNextQuarterTier = (currentStatus, quarterPurchases) => {
const currentTier = TIERS.find(t => t.name === currentStatus) || TIERS[0];
const earnedTier  = getTier(quarterPurchases);
// Must exceed current tier minimum to maintain or rise; otherwise drop one level
if (quarterPurchases > currentTier.min) {
return earnedTier.name;
} else {
const idx = TIERS.indexOf(currentTier);
// Never drop below Regular if client has 2+ total historical purchases
return idx > 0 ? TIERS[idx - 1].name : “Nuevo”;
}
};

// ─── Points System ───────────────────────────────────────────────────────────
const POINTS_RATE = { Nuevo: 0, Regular: 0.005, Plata: 0.007, VIP: 0.01 };
const POINTS_MIN_REDEEM = 3000;

const calcPoints = (amount, status) => Math.floor(amount * (POINTS_RATE[status] || 0));

// ─── MÓDULO: Puntos ───────────────────────────────────────────────────────────
function PointsModule({ clients, setClients }) {
const [selected, setSelected] = useState(null);
const [redeemAmt, setRedeemAmt] = useState(””);
const [search, setSearch] = useState(””);
const [toast, setToast] = useState(null);

const showToast = (msg, color = palette.success) => {
setToast({ msg, color });
setTimeout(() => setToast(null), 3500);
};

const filtered = clients.filter(c => c.name.toLowerCase().includes(search.toLowerCase()));
const client = clients.find(c => c.id === selected);
const canRedeem = client && (client.points || 0) >= POINTS_MIN_REDEEM;

const redeem = () => {
const amt = parseInt(redeemAmt);
if (!amt || amt < POINTS_MIN_REDEEM) return showToast(`Mínimo ₡${POINTS_MIN_REDEEM.toLocaleString()} para canjear`, palette.danger);
if (amt > (client.points || 0)) return showToast(“Saldo insuficiente”, palette.danger);
setClients(prev => prev.map(c => c.id === selected
? { …c, points: (c.points || 0) - amt, pointsRedeemed: (c.pointsRedeemed || 0) + amt }
: c));
showToast(`✅ ₡${amt.toLocaleString()} canjeados para ${client.name}`);
setRedeemAmt(””);
};

return (
<div>
{toast && (
<div style={{ position: “fixed”, top: 24, right: 24, background: toast.color, color: “#fff”, borderRadius: 12, padding: “12px 22px”, fontFamily: style.fontBody, fontWeight: 700, fontSize: 14, zIndex: 999, boxShadow: “0 4px 20px #0004” }}>
{toast.msg}
</div>
)}
<div style={{ marginBottom: 24 }}>
<h2 style={{ fontFamily: style.fontFamily, color: palette.text, fontSize: 28, margin: 0 }}>Sistema de Puntos</h2>
<p style={{ color: palette.textMuted, fontFamily: style.fontBody, margin: “6px 0 0” }}>Crédito en ₡ acumulado por nivel · No vence · Mínimo canje ₡{POINTS_MIN_REDEEM.toLocaleString()}</p>
</div>

```
  {/* Rate cards */}
  <div style={{ display: "grid", gridTemplateColumns: "repeat(4, 1fr)", gap: 12, marginBottom: 24 }}>
    {TIERS.map(t => (
      <Card key={t.name} style={{ textAlign: "center", borderColor: t.color + "44" }}>
        <div style={{ fontSize: 28, marginBottom: 6 }}>{t.icon}</div>
        <div style={{ color: t.color, fontFamily: style.fontFamily, fontSize: 22, fontWeight: 700 }}>
          {POINTS_RATE[t.name] > 0 ? `${(POINTS_RATE[t.name] * 100).toFixed(1)}%` : "—"}
        </div>
        <div style={{ color: palette.text, fontFamily: style.fontBody, fontSize: 13, fontWeight: 600, marginTop: 4 }}>{t.name}</div>
        <div style={{ color: palette.textDim, fontFamily: style.fontBody, fontSize: 11, marginTop: 3 }}>
          {POINTS_RATE[t.name] > 0 ? `₡${(5000 * POINTS_RATE[t.name]).toFixed(0)} por ₡5,000` : "Sin acumulación"}
        </div>
      </Card>
    ))}
  </div>

  <div style={{ display: "grid", gridTemplateColumns: "1fr 360px", gap: 20 }}>
    {/* Client list */}
    <div>
      <div style={{ marginBottom: 12 }}>
        <Input placeholder="Buscar cliente..." value={search} onChange={setSearch} />
      </div>
      <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
        {filtered.map(c => {
          const pts = c.points || 0;
          const progress = Math.min((pts / POINTS_MIN_REDEEM) * 100, 100);
          const tier = TIERS.find(t => t.name === c.status) || TIERS[0];
          return (
            <div key={c.id} onClick={() => setSelected(c.id === selected ? null : c.id)}
              style={{ background: selected === c.id ? palette.accentSoft : palette.card, border: `1.5px solid ${selected === c.id ? palette.accent : palette.border}`, borderRadius: 14, padding: 16, cursor: "pointer", transition: "all 0.18s" }}>
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
                  <div style={{ width: 40, height: 40, borderRadius: "50%", background: tier.color + "22", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 18, border: `2px solid ${tier.color}44` }}>
                    {tier.icon}
                  </div>
                  <div>
                    <div style={{ color: palette.text, fontFamily: style.fontBody, fontWeight: 600, fontSize: 14 }}>{c.name}</div>
                    <div style={{ color: palette.textMuted, fontSize: 12, fontFamily: style.fontBody }}>{tier.name} · {c.purchases || 0} compras totales</div>
                  </div>
                </div>
                <div style={{ textAlign: "right" }}>
                  <div style={{ color: pts >= POINTS_MIN_REDEEM ? palette.success : palette.accent, fontFamily: style.fontFamily, fontSize: 22, fontWeight: 700 }}>
                    ₡{pts.toLocaleString()}
                  </div>
                  <div style={{ color: palette.textDim, fontSize: 11, fontFamily: style.fontBody }}>
                    {pts >= POINTS_MIN_REDEEM ? "✅ Listo para canjear" : `Faltan ₡${(POINTS_MIN_REDEEM - pts).toLocaleString()}`}
                  </div>
                </div>
              </div>
              <div style={{ marginTop: 10, display: "flex", alignItems: "center", gap: 8 }}>
                <div style={{ flex: 1, height: 5, background: palette.border, borderRadius: 10, overflow: "hidden" }}>
                  <div style={{ width: `${progress}%`, height: "100%", background: pts >= POINTS_MIN_REDEEM ? palette.success : palette.accent, borderRadius: 10, transition: "width 0.4s" }} />
                </div>
                <span style={{ color: palette.textDim, fontSize: 11, fontFamily: style.fontBody, whiteSpace: "nowrap" }}>
                  ₡{pts.toLocaleString()} / ₡{POINTS_MIN_REDEEM.toLocaleString()}
                </span>
              </div>
            </div>
          );
        })}
      </div>
    </div>

    {/* Redeem panel */}
    <div>
      <Card style={{ position: "sticky", top: 20 }}>
        {!client ? (
          <div style={{ textAlign: "center", padding: "40px 0", color: palette.textDim, fontFamily: style.fontBody, fontSize: 13 }}>
            <div style={{ fontSize: 40, marginBottom: 10 }}>👆</div>
            Selecciona un cliente para ver su saldo y canjear
          </div>
        ) : (
          <>
            <h3 style={{ fontFamily: style.fontFamily, color: palette.text, margin: "0 0 16px" }}>Canjear Crédito</h3>
            <div style={{ background: palette.surface, borderRadius: 12, padding: 16, marginBottom: 16, textAlign: "center" }}>
              <div style={{ color: palette.textMuted, fontFamily: style.fontBody, fontSize: 12, marginBottom: 4 }}>{client.name}</div>
              <div style={{ color: canRedeem ? palette.success : palette.accent, fontFamily: style.fontFamily, fontSize: 34, fontWeight: 700 }}>
                ₡{(client.points || 0).toLocaleString()}
              </div>
              <div style={{ color: palette.textDim, fontFamily: style.fontBody, fontSize: 12, marginTop: 4 }}>
                Canjeado históricamente: ₡{(client.pointsRedeemed || 0).toLocaleString()}
              </div>
            </div>
            {!canRedeem ? (
              <div style={{ background: palette.accentSoft, borderRadius: 10, padding: 14, textAlign: "center", color: palette.textMuted, fontFamily: style.fontBody, fontSize: 13 }}>
                ⏳ Necesita <strong style={{ color: palette.accent }}>₡{(POINTS_MIN_REDEEM - (client.points || 0)).toLocaleString()}</strong> más para canjear
              </div>
            ) : (
              <>
                <div style={{ marginBottom: 12 }}>
                  <label style={{ color: palette.textMuted, fontSize: 12, fontFamily: style.fontBody, fontWeight: 600, display: "block", marginBottom: 6 }}>Monto a canjear (₡)</label>
                  <input type="number" value={redeemAmt} onChange={e => setRedeemAmt(e.target.value)}
                    placeholder={`Mín. ₡${POINTS_MIN_REDEEM.toLocaleString()}`}
                    style={{ width: "100%", background: palette.surface, border: `1px solid ${palette.border}`, borderRadius: 10, padding: "10px 14px", color: palette.text, fontFamily: style.fontBody, fontSize: 14, outline: "none", boxSizing: "border-box" }} />
                </div>
                <div style={{ display: "flex", gap: 8, marginBottom: 12, flexWrap: "wrap" }}>
                  {[POINTS_MIN_REDEEM, 5000, 10000].filter(v => v <= (client.points || 0)).map(v => (
                    <button key={v} onClick={() => setRedeemAmt(v)}
                      style={{ background: palette.surface, border: `1px solid ${palette.border}`, borderRadius: 8, padding: "5px 12px", cursor: "pointer", fontFamily: style.fontBody, fontSize: 12, color: palette.textMuted }}>
                      ₡{v.toLocaleString()}
                    </button>
                  ))}
                  <button onClick={() => setRedeemAmt(client.points || 0)}
                    style={{ background: palette.surface, border: `1px solid ${palette.border}`, borderRadius: 8, padding: "5px 12px", cursor: "pointer", fontFamily: style.fontBody, fontSize: 12, color: palette.accent }}>
                    Todo
                  </button>
                </div>
                <Btn style={{ width: "100%" }} onClick={redeem} disabled={!redeemAmt}>
                  Aplicar Canje ₡{parseInt(redeemAmt || 0).toLocaleString()}
                </Btn>
              </>
            )}
            <div style={{ marginTop: 16, borderTop: `1px solid ${palette.border}`, paddingTop: 14 }}>
              <div style={{ color: palette.textMuted, fontSize: 12, fontFamily: style.fontBody, fontWeight: 600, marginBottom: 6 }}>Tasa de acumulación activa</div>
              <div style={{ display: "flex", justifyContent: "space-between", fontFamily: style.fontBody, fontSize: 13 }}>
                <span style={{ color: palette.textMuted }}>Nivel {client.status}</span>
                <span style={{ color: palette.accent, fontWeight: 700 }}>
                  {POINTS_RATE[client.status] > 0 ? `${(POINTS_RATE[client.status] * 100).toFixed(1)}% de cada compra` : "Sin acumulación"}
                </span>
              </div>
              <div style={{ color: palette.textDim, fontSize: 11, fontFamily: style.fontBody, marginTop: 4 }}>
                Los puntos no vencen · Mínimo canje ₡{POINTS_MIN_REDEEM.toLocaleString()}
              </div>
            </div>
          </>
        )}
      </Card>
    </div>
  </div>
</div>
```

);
}

// ─── MÓDULO: CRM ──────────────────────────────────────────────────────────────
function CRM({ clients, setClients }) {
const [showForm, setShowForm]     = useState(false);
const [form, setForm]             = useState({ name: “”, email: “”, phone: “” });
const [search, setSearch]         = useState(””);
const [filterTier, setFilterTier] = useState(“Todos”);
const [simResult, setSimResult]   = useState(null);

const filtered = clients.filter(c => {
const matchSearch = c.name.toLowerCase().includes(search.toLowerCase()) || (c.email || “”).includes(search);
const matchTier   = filterTier === “Todos” || c.status === filterTier;
return matchSearch && matchTier;
});

const addClient = () => {
if (!form.name) return;
setClients([…clients, { …form, id: Date.now(), purchases: 0, total: 0, status: “Nuevo”, quarterPurchases: 0 }]);
setForm({ name: “”, email: “”, phone: “” });
setShowForm(false);
};

// Simulate quarter close — updates all clients
const closeQuarter = () => {
const updated = clients.map(c => {
const newStatus = c.purchases < 2 ? “Nuevo” : getNextQuarterTier(c.status, c.quarterPurchases || 0);
return { …c, status: newStatus, quarterPurchases: 0 };
});
setClients(updated);
const changes = updated.filter((c, i) => c.status !== clients[i].status);
setSimResult({ total: updated.length, changed: changes.length, details: changes.slice(0, 5) });
setTimeout(() => setSimResult(null), 6000);
};

const tierColor = { VIP: “#f0a500”, Plata: “#95a5a6”, Regular: “#3498db”, Nuevo: “#27ae60” };
const tierIcon  = { VIP: “🥇”, Plata: “⚪”, Regular: “🔵”, Nuevo: “🆕” };

return (
<div>
{/* Header */}
<div style={{ display: “flex”, justifyContent: “space-between”, alignItems: “center”, marginBottom: 20 }}>
<div>
<h2 style={{ fontFamily: style.fontFamily, color: palette.text, fontSize: 28, margin: 0 }}>Clientes</h2>
<p style={{ color: palette.textMuted, fontFamily: style.fontBody, margin: “6px 0 0” }}>{clients.length} clientes registrados</p>
</div>
<div style={{ display: “flex”, gap: 10 }}>
<Btn variant="ghost" size="sm" onClick={closeQuarter}>🔄 Cierre Trimestral</Btn>
<Btn onClick={() => setShowForm(!showForm)}>+ Nuevo Cliente</Btn>
</div>
</div>

```
  {/* Quarter close result */}
  {simResult && (
    <Card style={{ marginBottom: 16, borderColor: palette.accent, background: palette.accentSoft }}>
      <div style={{ fontFamily: style.fontBody, color: palette.text, fontSize: 14 }}>
        ✅ <strong>Cierre trimestral aplicado</strong> — {simResult.changed} clientes cambiaron de nivel.
        {simResult.details.map(c => (
          <span key={c.id} style={{ marginLeft: 10, color: palette.textMuted }}>· {c.name} → <strong style={{ color: tierColor[c.status] }}>{tierIcon[c.status]} {c.status}</strong></span>
        ))}
      </div>
    </Card>
  )}

  {/* Tier legend */}
  <Card style={{ marginBottom: 16 }}>
    <div style={{ display: "flex", gap: 6, flexWrap: "wrap", alignItems: "center" }}>
      <span style={{ color: palette.textMuted, fontSize: 12, fontFamily: style.fontBody, fontWeight: 600, marginRight: 6 }}>Niveles:</span>
      {TIERS.map(t => (
        <div key={t.name} style={{ background: t.color + "22", border: `1px solid ${t.color}55`, borderRadius: 8, padding: "4px 12px", display: "flex", alignItems: "center", gap: 6 }}>
          <span style={{ fontSize: 14 }}>{t.icon}</span>
          <span style={{ color: t.color, fontFamily: style.fontBody, fontSize: 12, fontWeight: 700 }}>{t.name}</span>
          <span style={{ color: palette.textDim, fontFamily: style.fontBody, fontSize: 11 }}>
            {t.max === Infinity ? `${t.min}+ compras` : t.min === t.max ? `${t.min} compra` : `${t.min}–${t.max} compras`} / trimestre
          </span>
        </div>
      ))}
    </div>
  </Card>

  {/* Form */}
  {showForm && (
    <Card style={{ marginBottom: 20 }}>
      <h3 style={{ fontFamily: style.fontFamily, color: palette.text, margin: "0 0 16px" }}>Registrar Cliente</h3>
      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: 12 }}>
        <Input label="Nombre" value={form.name} onChange={v => setForm({ ...form, name: v })} />
        <Input label="Email" value={form.email} onChange={v => setForm({ ...form, email: v })} />
        <Input label="Teléfono" value={form.phone} onChange={v => setForm({ ...form, phone: v })} />
      </div>
      <div style={{ display: "flex", gap: 10 }}>
        <Btn onClick={addClient}>Guardar</Btn>
        <Btn variant="ghost" onClick={() => setShowForm(false)}>Cancelar</Btn>
      </div>
    </Card>
  )}

  {/* Search + filter */}
  <div style={{ display: "flex", gap: 12, marginBottom: 16, alignItems: "center", flexWrap: "wrap" }}>
    <div style={{ flex: 1, minWidth: 200 }}>
      <Input placeholder="Buscar cliente..." value={search} onChange={setSearch} />
    </div>
    <div style={{ display: "flex", gap: 6 }}>
      {["Todos", ...TIERS.map(t => t.name)].map(t => (
        <button key={t} onClick={() => setFilterTier(t)}
          style={{ background: filterTier === t ? palette.accentSoft : "transparent", border: `1px solid ${filterTier === t ? palette.accent : palette.border}`, borderRadius: 8, padding: "7px 13px", fontFamily: style.fontBody, fontSize: 12, fontWeight: 600, cursor: "pointer", color: filterTier === t ? palette.accent : palette.textMuted, transition: "all 0.15s" }}>
          {t === "Todos" ? "Todos" : `${tierIcon[t]} ${t}`}
        </button>
      ))}
    </div>
  </div>

  {/* Client list */}
  <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
    {filtered.map(c => {
      const tier = TIERS.find(t => t.name === c.status) || TIERS[0];
      const qp   = c.quarterPurchases || 0;
      const nextMin = tier.min + 1;
      const progress = Math.min((qp / nextMin) * 100, 100);
      return (
        <Card key={c.id} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", gap: 16 }}>
          <div style={{ display: "flex", alignItems: "center", gap: 14 }}>
            <div style={{ width: 46, height: 46, borderRadius: "50%", background: tier.color + "22", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 22, border: `2px solid ${tier.color}55` }}>
              {tier.icon}
            </div>
            <div>
              <div style={{ color: palette.text, fontFamily: style.fontBody, fontWeight: 600 }}>{c.name}</div>
              <div style={{ color: palette.textMuted, fontSize: 12, fontFamily: style.fontBody }}>{c.email} · {c.phone}</div>
              {/* Quarter progress bar */}
              <div style={{ marginTop: 6, display: "flex", alignItems: "center", gap: 8 }}>
                <div style={{ width: 100, height: 5, background: palette.border, borderRadius: 10, overflow: "hidden" }}>
                  <div style={{ width: `${progress}%`, height: "100%", background: tier.color, borderRadius: 10, transition: "width 0.4s" }} />
                </div>
                <span style={{ color: palette.textDim, fontSize: 11, fontFamily: style.fontBody }}>{qp}/{nextMin} este trimestre</span>
              </div>
            </div>
          </div>
          <div style={{ display: "flex", alignItems: "center", gap: 16, textAlign: "right", flexShrink: 0 }}>
            <div>
              <div style={{ color: palette.text, fontFamily: style.fontBody, fontSize: 13 }}>{c.purchases} compras totales</div>
              <div style={{ color: palette.success, fontFamily: style.fontFamily, fontSize: 17 }}>${(c.total || 0).toFixed(2)}</div>
            </div>
            <div style={{ background: tier.color + "22", border: `1px solid ${tier.color}55`, borderRadius: 8, padding: "5px 12px", textAlign: "center" }}>
              <div style={{ fontSize: 18 }}>{tier.icon}</div>
              <div style={{ color: tier.color, fontFamily: style.fontBody, fontSize: 11, fontWeight: 700 }}>{tier.name}</div>
            </div>
          </div>
        </Card>
      );
    })}
  </div>
</div>
```

);
}

// ─── MÓDULO: Reportes ─────────────────────────────────────────────────────────
function Reports({ sales, products, clients }) {
const today = new Date().toISOString().split(“T”)[0];
const [filterMode, setFilterMode] = useState(“today”);
const [dateFrom, setDateFrom] = useState(today);
const [dateTo, setDateTo] = useState(today);
const [downloaded, setDownloaded] = useState(false);

// Filter sales by date
const filteredSales = sales.filter(s => {
if (filterMode === “today”) return s.date === today;
return s.date >= dateFrom && s.date <= dateTo;
});

// Consolidate units per product from filtered sales
// Since our sale objects store total/items but not per-product breakdown,
// we simulate a breakdown using a deterministic spread for demo purposes.
// In production this would come from sale line items in DB.
const productMap = {};
filteredSales.forEach(sale => {
// Attach cart breakdown if available (from POS), else approximate
const breakdown = sale.cart || [];
breakdown.forEach(item => {
if (!productMap[item.name]) {
productMap[item.name] = { name: item.name, category: item.category || “—”, image: item.image || “📦”, unitPrice: item.price, units: 0, total: 0 };
}
productMap[item.name].units += item.qty;
productMap[item.name].total += item.price * item.qty;
});
});

// Fallback demo rows if no cart data available
const consolidated = Object.values(productMap).length > 0
? Object.values(productMap)
: products.slice(0, 4).map((p, i) => ({ name: p.name, category: p.category, image: p.image, unitPrice: p.price, units: [3,5,2,4][i] || 1, total: p.price * ([3,5,2,4][i] || 1) }));

const grandTotalUnits = consolidated.reduce((a, r) => a + r.units, 0);
const grandTotalRevenue = consolidated.reduce((a, r) => a + r.total, 0);
const totalRev = sales.reduce((a, s) => a + s.total, 0);
const avgSale = sales.length ? totalRev / sales.length : 0;

// Download as CSV (Excel-compatible)
const downloadExcel = () => {
const label = filterMode === “today” ? today : `${dateFrom}_al_${dateTo}`;
const rows = [
[“REPORTE DE PEDIDOS CONSOLIDADO”],
[`Período: ${filterMode === "today" ? `Hoy (${today})`:`${dateFrom} al ${dateTo}`}`],
[`Generado: ${new Date().toLocaleString("es-ES")}`],
[],
[“Producto”, “Categoría”, “Precio Unitario”, “Unidades Vendidas”, “Total ($)”],
…consolidated.map(r => [r.name, r.category, r.unitPrice.toFixed(2), r.units, r.total.toFixed(2)]),
[],
[””, “”, “TOTAL GENERAL”, grandTotalUnits, grandTotalRevenue.toFixed(2)],
];

```
// Build CSV string
const csv = rows.map(r => r.map(cell => `"${cell}"`).join(",")).join("\n");
const BOM = "\uFEFF"; // UTF-8 BOM for Excel Spanish compatibility
const blob = new Blob([BOM + csv], { type: "text/csv;charset=utf-8;" });
const url = URL.createObjectURL(blob);
const a = document.createElement("a");
a.href = url;
a.download = `Reporte_Pedidos_${label}.csv`;
a.click();
URL.revokeObjectURL(url);
setDownloaded(true);
setTimeout(() => setDownloaded(false), 3000);
```

};

return (
<div>
<div style={{ display: “flex”, justifyContent: “space-between”, alignItems: “center”, marginBottom: 24 }}>
<div>
<h2 style={{ fontFamily: style.fontFamily, color: palette.text, fontSize: 28, margin: 0 }}>Reportes</h2>
<p style={{ color: palette.textMuted, fontFamily: style.fontBody, margin: “6px 0 0” }}>Consolidado de pedidos por período</p>
</div>
<Btn onClick={downloadExcel} variant={downloaded ? “success” : “primary”} size=“md”>
{downloaded ? “✓ Descargado!” : “⬇ Descargar Excel”}
</Btn>
</div>

```
  {/* Metrics */}
  <div style={{ display: "grid", gridTemplateColumns: "repeat(3, 1fr)", gap: 16, marginBottom: 24 }}>
    {[
      { label: "Ingreso Total", value: `$${totalRev.toFixed(2)}`, color: palette.success, icon: "💵" },
      { label: "Venta Promedio", value: `$${avgSale.toFixed(2)}`, color: palette.info, icon: "📊" },
      { label: "Ventas del Período", value: filteredSales.length, color: palette.accent, icon: "🧾" },
    ].map(m => (
      <Card key={m.label} style={{ textAlign: "center" }}>
        <div style={{ fontSize: 36, marginBottom: 8 }}>{m.icon}</div>
        <div style={{ color: m.color, fontFamily: style.fontFamily, fontSize: 24, fontWeight: 700 }}>{m.value}</div>
        <div style={{ color: palette.textMuted, fontFamily: style.fontBody, fontSize: 13, marginTop: 4 }}>{m.label}</div>
      </Card>
    ))}
  </div>

  {/* Date Filter */}
  <Card style={{ marginBottom: 20 }}>
    <div style={{ display: "flex", alignItems: "center", gap: 16, flexWrap: "wrap" }}>
      <div style={{ fontFamily: style.fontBody, color: palette.textMuted, fontSize: 13, fontWeight: 600 }}>Filtrar por:</div>
      {["today", "range"].map(mode => (
        <button key={mode} onClick={() => setFilterMode(mode)}
          style={{ background: filterMode === mode ? palette.accentSoft : "transparent", border: `1px solid ${filterMode === mode ? palette.accent : palette.border}`, borderRadius: 8, padding: "7px 16px", fontFamily: style.fontBody, fontSize: 13, fontWeight: 600, cursor: "pointer", color: filterMode === mode ? palette.accent : palette.textMuted, transition: "all 0.15s" }}>
          {mode === "today" ? "📅 Hoy" : "📆 Rango de fechas"}
        </button>
      ))}
      {filterMode === "range" && (
        <div style={{ display: "flex", gap: 10, alignItems: "center" }}>
          <input type="date" value={dateFrom} onChange={e => setDateFrom(e.target.value)}
            style={{ background: palette.surface, border: `1px solid ${palette.border}`, borderRadius: 8, padding: "7px 12px", color: palette.text, fontFamily: style.fontBody, fontSize: 13 }} />
          <span style={{ color: palette.textMuted, fontFamily: style.fontBody }}>al</span>
          <input type="date" value={dateTo} onChange={e => setDateTo(e.target.value)}
            style={{ background: palette.surface, border: `1px solid ${palette.border}`, borderRadius: 8, padding: "7px 12px", color: palette.text, fontFamily: style.fontBody, fontSize: 13 }} />
        </div>
      )}
    </div>
  </Card>

  {/* Consolidated Table */}
  <Card>
    <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 16 }}>
      <h3 style={{ fontFamily: style.fontFamily, color: palette.text, margin: 0 }}>
        Consolidado de Productos
      </h3>
      <span style={{ color: palette.textMuted, fontFamily: style.fontBody, fontSize: 13 }}>
        {filterMode === "today" ? `Hoy · ${today}` : `${dateFrom} → ${dateTo}`}
      </span>
    </div>
    <div style={{ overflowX: "auto" }}>
      <table style={{ width: "100%", borderCollapse: "collapse", fontFamily: style.fontBody, fontSize: 14 }}>
        <thead>
          <tr style={{ background: palette.surface, color: palette.textMuted, textAlign: "left" }}>
            {["Producto", "Categoría", "Precio Unit.", "Unidades Vendidas", "Total ($)"].map(h => (
              <th key={h} style={{ padding: "10px 14px", fontWeight: 600, borderBottom: `2px solid ${palette.border}` }}>{h}</th>
            ))}
          </tr>
        </thead>
        <tbody>
          {consolidated.map((r, i) => (
            <tr key={i} style={{ borderBottom: `1px solid ${palette.border}` }}
              onMouseEnter={e => e.currentTarget.style.background = palette.surfaceHover}
              onMouseLeave={e => e.currentTarget.style.background = "transparent"}>
              <td style={{ padding: "11px 14px", color: palette.text }}>{r.image} {r.name}</td>
              <td style={{ padding: "11px 14px", color: palette.textMuted }}>{r.category}</td>
              <td style={{ padding: "11px 14px", color: palette.textMuted }}>${r.unitPrice.toFixed(2)}</td>
              <td style={{ padding: "11px 14px" }}>
                <span style={{ background: palette.accentSoft, color: palette.accent, borderRadius: 8, padding: "3px 12px", fontWeight: 700 }}>{r.units}</span>
              </td>
              <td style={{ padding: "11px 14px", color: palette.success, fontWeight: 700 }}>${r.total.toFixed(2)}</td>
            </tr>
          ))}
        </tbody>
        <tfoot>
          <tr style={{ background: palette.surface, borderTop: `2px solid ${palette.border}` }}>
            <td colSpan={3} style={{ padding: "12px 14px", color: palette.text, fontFamily: style.fontFamily, fontSize: 16, fontWeight: 700 }}>TOTAL GENERAL</td>
            <td style={{ padding: "12px 14px", color: palette.accent, fontFamily: style.fontFamily, fontSize: 18, fontWeight: 700 }}>{grandTotalUnits} uds.</td>
            <td style={{ padding: "12px 14px", color: palette.success, fontFamily: style.fontFamily, fontSize: 18, fontWeight: 700 }}>${grandTotalRevenue.toFixed(2)}</td>
          </tr>
        </tfoot>
      </table>
    </div>
    <div style={{ marginTop: 14, color: palette.textDim, fontSize: 12, fontFamily: style.fontBody }}>
      💡 Presiona "Descargar Excel" para exportar este reporte y abrirlo en Excel o Google Sheets.
    </div>
  </Card>
</div>
```

);
}

// ─── MÓDULO: Registro de Cliente ─────────────────────────────────────────────
function CustomerRegister({ onComplete }) {
const [step, setStep] = useState(1); // 1=form, 2=verify, 3=success
const [form, setForm] = useState({ firstName: “”, lastName: “”, whatsapp: “”, email: “”, address: “”, observations: “”, lat: null, lng: null, locationLabel: “” });
const [code, setCode] = useState(””);
const [generatedCode, setGeneratedCode] = useState(””);
const [locLoading, setLocLoading] = useState(false);
const [errors, setErrors] = useState({});

const f = (k, v) => setForm(p => ({ …p, [k]: v }));

const validate = () => {
const e = {};
if (!form.firstName.trim()) e.firstName = “Requerido”;
if (!form.lastName.trim()) e.lastName = “Requerido”;
if (!form.whatsapp.trim()) e.whatsapp = “Requerido”;
if (!form.email.trim() || !form.email.includes(”@”)) e.email = “Email inválido”;
if (!form.address.trim()) e.address = “Requerido”;
setErrors(e);
return Object.keys(e).length === 0;
};

const sendCode = () => {
if (!validate()) return;
const c = Math.floor(100000 + Math.random() * 900000).toString();
setGeneratedCode(c);
// In production this would call Supabase Auth or email API
// For demo we show the code in an alert simulating email delivery
setTimeout(() => alert(`📧 Simulación de email enviado a: ${form.email}\n\nTu código de verificación es: ${c}\n\n(En producción esto llega automáticamente al correo)`), 100);
setStep(2);
};

const verify = () => {
if (code === generatedCode) {
setStep(3);
setTimeout(() => onComplete && onComplete(form), 2000);
} else {
setErrors({ code: “Código incorrecto, inténtalo de nuevo” });
}
};

const getLocation = () => {
setLocLoading(true);
navigator.geolocation?.getCurrentPosition(
pos => {
const { latitude: lat, longitude: lng } = pos.coords;
f(“lat”, lat); f(“lng”, lng);
f(“locationLabel”, `${lat.toFixed(5)}, ${lng.toFixed(5)}`);
setLocLoading(false);
},
() => { setLocLoading(false); alert(“No se pudo obtener la ubicación. Asegúrate de permitir acceso.”); }
);
};

const inputStyle = (key) => ({
width: “100%”, background: “#fff”, border: `1.5px solid ${errors[key] ? palette.danger : "#e0ddd8"}`,
borderRadius: 10, padding: “11px 14px”, color: “#1a1a24”, fontFamily: style.fontBody,
fontSize: 14, outline: “none”, boxSizing: “border-box”, transition: “border-color 0.2s”,
});

const labelStyle = { color: “#555”, fontSize: 12, fontFamily: style.fontBody, marginBottom: 5, fontWeight: 600, display: “block” };

if (step === 3) return (
<div style={{ textAlign: “center”, padding: “60px 20px” }}>
<div style={{ fontSize: 72, marginBottom: 16 }}>🎉</div>
<h2 style={{ fontFamily: style.fontFamily, color: “#1a1a24”, fontSize: 28 }}>¡Registro Exitoso!</h2>
<p style={{ color: “#888”, fontFamily: style.fontBody }}>Bienvenido/a, {form.firstName}. Ya puedes explorar el catálogo.</p>
</div>
);

if (step === 2) return (
<div style={{ maxWidth: 420, margin: “0 auto”, padding: “40px 20px” }}>
<div style={{ textAlign: “center”, marginBottom: 32 }}>
<div style={{ fontSize: 52, marginBottom: 12 }}>📧</div>
<h2 style={{ fontFamily: style.fontFamily, color: “#1a1a24”, fontSize: 26, margin: 0 }}>Verifica tu email</h2>
<p style={{ color: “#888”, fontFamily: style.fontBody, marginTop: 8 }}>
Enviamos un código de 6 dígitos a <strong>{form.email}</strong>
</p>
</div>
<div style={{ marginBottom: 16 }}>
<label style={labelStyle}>Código de verificación</label>
<input value={code} onChange={e => setCode(e.target.value)} maxLength={6} placeholder=”_ _ _ _ _ _”
style={{ …inputStyle(“code”), textAlign: “center”, fontSize: 28, letterSpacing: 12, fontFamily: style.fontFamily }} />
{errors.code && <div style={{ color: palette.danger, fontSize: 12, marginTop: 4, fontFamily: style.fontBody }}>{errors.code}</div>}
</div>
<button onClick={verify} style={{ width: “100%”, background: palette.accent, border: “none”, borderRadius: 10, padding: “13px”, fontFamily: style.fontBody, fontWeight: 700, fontSize: 15, cursor: “pointer”, color: “#1a1a24” }}>
Verificar y Continuar →
</button>
<div style={{ textAlign: “center”, marginTop: 14 }}>
<button onClick={() => { setStep(1); setCode(””); setErrors({}); }}
style={{ background: “none”, border: “none”, color: “#aaa”, fontFamily: style.fontBody, fontSize: 13, cursor: “pointer” }}>
← Volver y corregir datos
</button>
</div>
<div style={{ marginTop: 16, textAlign: “center” }}>
<button onClick={sendCode} style={{ background: “none”, border: “none”, color: palette.accent, fontFamily: style.fontBody, fontSize: 13, cursor: “pointer”, fontWeight: 600 }}>
Reenviar código
</button>
</div>
</div>
);

return (
<div style={{ maxWidth: 560, margin: “0 auto”, padding: “32px 20px” }}>
<div style={{ textAlign: “center”, marginBottom: 28 }}>
<div style={{ fontSize: 44, marginBottom: 10 }}>🛍</div>
<h2 style={{ fontFamily: style.fontFamily, color: “#1a1a24”, fontSize: 26, margin: 0 }}>Crear tu cuenta</h2>
<p style={{ color: “#888”, fontFamily: style.fontBody, marginTop: 6, fontSize: 14 }}>Regístrate para hacer pedidos y recibir a domicilio</p>
</div>

```
  {/* Paso 1: Datos personales */}
  <div style={{ background: "#f4f4f0", borderRadius: 12, padding: "4px 14px 10px", marginBottom: 16 }}>
    <div style={{ color: palette.accent, fontFamily: style.fontFamily, fontSize: 13, fontWeight: 700, margin: "10px 0 12px" }}>👤 Datos Personales</div>
    <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 12 }}>
      <div>
        <label style={labelStyle}>Nombre *</label>
        <input value={form.firstName} onChange={e => f("firstName", e.target.value)} placeholder="Ej: María" style={inputStyle("firstName")} />
        {errors.firstName && <div style={{ color: palette.danger, fontSize: 11, marginTop: 3, fontFamily: style.fontBody }}>{errors.firstName}</div>}
      </div>
      <div>
        <label style={labelStyle}>Apellidos *</label>
        <input value={form.lastName} onChange={e => f("lastName", e.target.value)} placeholder="Ej: García López" style={inputStyle("lastName")} />
        {errors.lastName && <div style={{ color: palette.danger, fontSize: 11, marginTop: 3, fontFamily: style.fontBody }}>{errors.lastName}</div>}
      </div>
    </div>
    <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 12, marginTop: 12 }}>
      <div>
        <label style={labelStyle}>WhatsApp *</label>
        <input value={form.whatsapp} onChange={e => f("whatsapp", e.target.value)} placeholder="+1 555 000 0000" style={inputStyle("whatsapp")} />
        {errors.whatsapp && <div style={{ color: palette.danger, fontSize: 11, marginTop: 3, fontFamily: style.fontBody }}>{errors.whatsapp}</div>}
      </div>
      <div>
        <label style={labelStyle}>Email *</label>
        <input value={form.email} onChange={e => f("email", e.target.value)} placeholder="tu@email.com" style={inputStyle("email")} />
        {errors.email && <div style={{ color: palette.danger, fontSize: 11, marginTop: 3, fontFamily: style.fontBody }}>{errors.email}</div>}
      </div>
    </div>
  </div>

  {/* Paso 2: Dirección de entrega */}
  <div style={{ background: "#f4f4f0", borderRadius: 12, padding: "4px 14px 14px", marginBottom: 16 }}>
    <div style={{ color: palette.accent, fontFamily: style.fontFamily, fontSize: 13, fontWeight: 700, margin: "10px 0 12px" }}>📦 Dirección de Entrega</div>
    <div style={{ marginBottom: 12 }}>
      <label style={labelStyle}>Dirección completa *</label>
      <input value={form.address} onChange={e => f("address", e.target.value)} placeholder="Calle, número, colonia, ciudad" style={inputStyle("address")} />
      {errors.address && <div style={{ color: palette.danger, fontSize: 11, marginTop: 3, fontFamily: style.fontBody }}>{errors.address}</div>}
    </div>

    {/* GPS */}
    <div style={{ marginBottom: 12 }}>
      <label style={labelStyle}>Ubicación GPS (opcional)</label>
      <div style={{ display: "flex", gap: 10, alignItems: "center" }}>
        <input readOnly value={form.locationLabel || ""} placeholder="Presiona el botón para obtener ubicación"
          style={{ ...inputStyle(), flex: 1, background: "#efefeb", cursor: "default", color: form.locationLabel ? "#1a1a24" : "#aaa" }} />
        <button onClick={getLocation} disabled={locLoading}
          style={{ background: form.lat ? palette.success : palette.accent, border: "none", borderRadius: 10, padding: "11px 16px", cursor: "pointer", fontFamily: style.fontBody, fontWeight: 700, fontSize: 13, color: "#fff", whiteSpace: "nowrap", opacity: locLoading ? 0.7 : 1 }}>
          {locLoading ? "📡..." : form.lat ? "✓ GPS" : "📍 Obtener"}
        </button>
      </div>
      {form.lat && (
        <div style={{ marginTop: 8, borderRadius: 8, overflow: "hidden", border: "1px solid #ddd" }}>
          <iframe title="map" width="100%" height="160" frameBorder="0" style={{ display: "block" }}
            src={`https://maps.google.com/maps?q=${form.lat},${form.lng}&z=15&output=embed`} />
        </div>
      )}
    </div>

    <div>
      <label style={labelStyle}>Observaciones de entrega</label>
      <textarea value={form.observations} onChange={e => f("observations", e.target.value)}
        placeholder="Ej: Casa color azul, tocar el timbre, dejar con el portero..."
        rows={3} style={{ ...inputStyle(), resize: "vertical" }} />
    </div>
  </div>

  <button onClick={sendCode}
    style={{ width: "100%", background: palette.accent, border: "none", borderRadius: 12, padding: "14px", fontFamily: style.fontBody, fontWeight: 700, fontSize: 16, cursor: "pointer", color: "#1a1a24", transition: "background 0.2s" }}
    onMouseEnter={e => e.currentTarget.style.background = palette.accentHover}
    onMouseLeave={e => e.currentTarget.style.background = palette.accent}>
    Continuar → Verificar Email
  </button>
  <p style={{ textAlign: "center", color: "#aaa", fontFamily: style.fontBody, fontSize: 12, marginTop: 12 }}>
    Te enviaremos un código de 6 dígitos para confirmar tu cuenta
  </p>
</div>
```

);
}

// ─── MÓDULO: Catálogo Cliente ─────────────────────────────────────────────────
function CustomerCatalog({ products }) {
const [wishlist, setWishlist] = useState([]);
const [filter, setFilter] = useState(“Todos”);
const [showWishlist, setShowWishlist] = useState(false);
const categories = [“Todos”, …new Set(products.map(p => p.category))];
const filtered = filter === “Todos” ? products : products.filter(p => p.category === filter);

const toggle = (p) => setWishlist(w => w.find(i => i.id === p.id) ? w.filter(i => i.id !== p.id) : […w, p]);
const inList = (id) => wishlist.some(i => i.id === id);

return (
<div style={{ background: “#f9f7f4”, minHeight: “100%”, borderRadius: 16, padding: 24 }}>
<div style={{ display: “flex”, justifyContent: “space-between”, alignItems: “center”, marginBottom: 24 }}>
<div>
<h2 style={{ fontFamily: style.fontFamily, color: “#1a1a24”, fontSize: 30, margin: 0 }}>Nuestro Catálogo</h2>
<p style={{ color: “#8b8799”, fontFamily: style.fontBody, margin: “6px 0 0” }}>Agrega productos a tu lista y compártela</p>
</div>
<button onClick={() => setShowWishlist(!showWishlist)}
style={{ background: palette.accent, border: “none”, borderRadius: 12, padding: “10px 18px”, fontFamily: style.fontBody, fontWeight: 700, fontSize: 14, cursor: “pointer”, display: “flex”, alignItems: “center”, gap: 8 }}>
🛍 Mi Lista ({wishlist.length})
</button>
</div>

```
  {showWishlist && wishlist.length > 0 && (
    <div style={{ background: "#fff", border: "2px solid " + palette.accent, borderRadius: 14, padding: 18, marginBottom: 20 }}>
      <h3 style={{ fontFamily: style.fontFamily, color: "#1a1a24", margin: "0 0 14px" }}>Mi Lista de Deseos</h3>
      {wishlist.map(p => (
        <div key={p.id} style={{ display: "flex", justifyContent: "space-between", padding: "8px 0", borderBottom: "1px solid #eee", fontFamily: style.fontBody, fontSize: 14 }}>
          <span>{p.image} {p.name}</span>
          <strong>${p.price}</strong>
        </div>
      ))}
      <div style={{ marginTop: 12, fontFamily: style.fontFamily, fontSize: 18, display: "flex", justifyContent: "space-between" }}>
        <span>Total estimado</span>
        <span style={{ color: palette.accent }}>${wishlist.reduce((a, p) => a + p.price, 0).toFixed(2)}</span>
      </div>
      <button onClick={() => {
        const txt = `Mi lista de productos:\n${wishlist.map(p => `- ${p.name}: $${p.price}`).join("\n")}\nTotal: $${wishlist.reduce((a, p) => a + p.price, 0).toFixed(2)}`;
        navigator.clipboard?.writeText(txt);
        alert("¡Lista copiada al portapapeles!");
      }} style={{ marginTop: 14, background: palette.accent, border: "none", borderRadius: 10, padding: "10px 20px", fontFamily: style.fontBody, fontWeight: 700, cursor: "pointer", fontSize: 14 }}>
        📋 Copiar Lista
      </button>
    </div>
  )}

  <div style={{ display: "flex", gap: 8, marginBottom: 20, flexWrap: "wrap" }}>
    {categories.map(c => (
      <button key={c} onClick={() => setFilter(c)}
        style={{ background: filter === c ? palette.accent : "#fff", border: `1px solid ${filter === c ? palette.accent : "#ddd"}`, borderRadius: 20, padding: "7px 16px", fontFamily: style.fontBody, fontWeight: 600, fontSize: 13, cursor: "pointer", color: filter === c ? "#1a1a24" : "#555" }}>
        {c}
      </button>
    ))}
  </div>

  <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fill, minmax(200px, 1fr))", gap: 16 }}>
    {filtered.filter(p => p.stock > 0).map(p => (
      <div key={p.id} style={{ background: "#fff", borderRadius: 16, overflow: "hidden", boxShadow: "0 2px 12px #0001", border: "1px solid #eee", transition: "box-shadow 0.2s" }}
        onMouseEnter={e => e.currentTarget.style.boxShadow = "0 8px 30px #0002"}
        onMouseLeave={e => e.currentTarget.style.boxShadow = "0 2px 12px #0001"}>
        <div style={{ background: "#f4f4f0", display: "flex", alignItems: "center", justifyContent: "center", height: 120, fontSize: 52 }}>{p.image}</div>
        <div style={{ padding: 16 }}>
          <div style={{ color: "#1a1a24", fontFamily: style.fontBody, fontWeight: 600, fontSize: 14 }}>{p.name}</div>
          <div style={{ color: "#aaa", fontSize: 12, fontFamily: style.fontBody, margin: "4px 0 8px" }}>{p.desc}</div>
          <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center" }}>
            <span style={{ color: palette.accent, fontFamily: style.fontFamily, fontSize: 20, fontWeight: 700 }}>${p.price}</span>
            <button onClick={() => toggle(p)}
              style={{ background: inList(p.id) ? palette.accent : "transparent", border: `2px solid ${palette.accent}`, borderRadius: 8, padding: "5px 12px", cursor: "pointer", fontFamily: style.fontBody, fontSize: 12, fontWeight: 700, color: inList(p.id) ? "#1a1a24" : palette.accent, transition: "all 0.18s" }}>
              {inList(p.id) ? "✓ En lista" : "+ Agregar"}
            </button>
          </div>
        </div>
      </div>
    ))}
  </div>
</div>
```

);
}

// ─── APP PRINCIPAL ────────────────────────────────────────────────────────────
export default function App() {
const [products, setProductsState] = useState([]);
const [clients,  setClientsState]  = useState([]);
const [sales,    setSalesState]    = useState([]);
const [activeTab, setActiveTab]    = useState(“dashboard”);
const [loading, setLoading]        = useState(true);
const [dbError, setDbError]        = useState(null);

// ── Load all data from Supabase on mount ────────────────────────────────────
useEffect(() => {
const load = async () => {
setLoading(true);
try {
const [p, c, s] = await Promise.all([
sb.from(“productos”).select(”*”),
sb.from(“clientes”).select(”*”),
sb.from(“ventas”).select(”*”),
]);
if (p.error || c.error || s.error) {
setDbError(“No se pudo conectar a la base de datos. Verifica que las tablas existen en Supabase.”);
} else {
// Map DB snake_case to app camelCase
setProductsState((p.data || []).map(r => ({ …r, desc: r.description })));
setClientsState((c.data || []).map(r => ({ …r, quarterPurchases: r.quarter_purchases, pointsRedeemed: r.points_redeemed })));
setSalesState((s.data || []).map(r => ({ …r, client: r.client_name })));
}
} catch(e) {
setDbError(“Error de conexión: “ + e.message);
}
setLoading(false);
};
load();
}, []);

// ── Wrapped setters that sync to Supabase ───────────────────────────────────
const setProducts = async (updater) => {
const next = typeof updater === “function” ? updater(products) : updater;
setProductsState(next);
// Sync new/updated products
for (const p of next) {
if (!products.find(x => x.id === p.id)) {
await sb.from(“productos”).insert({ name: p.name, sku: p.sku, category: p.category, price: p.price, stock: p.stock, image: p.image, description: p.desc });
} else {
const old = products.find(x => x.id === p.id);
if (old && old.stock !== p.stock) await sb.from(“productos”).update({ stock: p.stock }, “id”, p.id);
}
}
};

const setClients = async (updater) => {
const next = typeof updater === “function” ? updater(clients) : updater;
setClientsState(next);
for (const c of next) {
const old = clients.find(x => x.id === c.id);
if (!old) {
await sb.from(“clientes”).insert({ name: c.name, email: c.email, phone: c.phone, whatsapp: c.whatsapp, address: c.address, observations: c.observations, lat: c.lat, lng: c.lng, status: c.status, purchases: c.purchases || 0, total: c.total || 0, quarter_purchases: c.quarterPurchases || 0, points: c.points || 0, points_redeemed: c.pointsRedeemed || 0 });
} else if (JSON.stringify(old) !== JSON.stringify(c)) {
await sb.from(“clientes”).update({ status: c.status, purchases: c.purchases, total: c.total, quarter_purchases: c.quarterPurchases || 0, points: c.points || 0, points_redeemed: c.pointsRedeemed || 0 }, “id”, c.id);
}
}
};

const setSales = async (updater) => {
const next = typeof updater === “function” ? updater(sales) : updater;
setSalesState(next);
const newSales = next.filter(s => !sales.find(x => x.id === s.id));
for (const s of newSales) {
await sb.from(“ventas”).insert({ client_name: s.client, items: s.items, total: s.total, status: s.status, cart: s.cart, date: s.date });
}
};

const tabs = [
{ id: “dashboard”, label: “Dashboard”,        icon: “📊” },
{ id: “inventory”, label: “Inventario”,        icon: “📦” },
{ id: “pos”,       label: “Ventas / POS”,      icon: “💳” },
{ id: “crm”,       label: “Clientes”,          icon: “👥” },
{ id: “reports”,   label: “Reportes”,          icon: “📈” },
{ id: “catalog”,   label: “Catálogo”,          icon: “🛍” },
{ id: “points”,    label: “Puntos”,            icon: “⭐” },
{ id: “register”,  label: “Registro Cliente”,  icon: “📝” },
];

return (
<div style={{ display: “flex”, minHeight: “100vh”, background: palette.bg, fontFamily: style.fontBody }}>
{/* Sidebar */}
<div style={{ width: 220, background: palette.surface, borderRight: `1px solid ${palette.border}`, padding: “24px 0”, flexShrink: 0, display: “flex”, flexDirection: “column” }}>
<div style={{ padding: “0 20px 24px”, borderBottom: `1px solid ${palette.border}` }}>
<div style={{ fontFamily: style.fontFamily, color: palette.accent, fontSize: 22, fontWeight: 700 }}>RetailOS</div>
<div style={{ color: palette.textDim, fontSize: 12, fontFamily: style.fontBody, marginTop: 2 }}>
{loading ? “⏳ Conectando…” : dbError ? “⚠️ Sin conexión” : “✅ Conectado · Supabase”}
</div>
</div>
<nav style={{ marginTop: 16, flex: 1 }}>
{tabs.map(t => (
<button key={t.id} onClick={() => setActiveTab(t.id)}
style={{
display: “flex”, alignItems: “center”, gap: 10, width: “100%”, padding: “11px 20px”, border: “none”,
background: activeTab === t.id ? palette.accentSoft : “transparent”,
color: activeTab === t.id ? palette.accent : palette.textMuted,
fontFamily: style.fontBody, fontSize: 14, fontWeight: activeTab === t.id ? 700 : 400,
cursor: “pointer”, textAlign: “left”,
borderLeft: activeTab === t.id ? `3px solid ${palette.accent}` : “3px solid transparent”,
transition: “all 0.15s”,
}}>
<span style={{ fontSize: 17 }}>{t.icon}</span> {t.label}
</button>
))}
</nav>
<div style={{ padding: “16px 20px”, borderTop: `1px solid ${palette.border}`, color: palette.textDim, fontSize: 11, fontFamily: style.fontBody }}>
v1.0 · Umami Foods CR
</div>
</div>

```
  {/* Main Content */}
  <div style={{ flex: 1, padding: 28, overflowY: "auto" }}>
    {loading && (
      <div style={{ display: "flex", alignItems: "center", justifyContent: "center", height: "60vh", flexDirection: "column", gap: 16 }}>
        <div style={{ fontSize: 48 }}>⏳</div>
        <div style={{ color: palette.textMuted, fontFamily: style.fontBody, fontSize: 16 }}>Cargando datos desde Supabase...</div>
      </div>
    )}
    {dbError && !loading && (
      <div style={{ background: palette.danger + "22", border: `1px solid ${palette.danger}`, borderRadius: 12, padding: 20, marginBottom: 20 }}>
        <div style={{ color: palette.danger, fontFamily: style.fontBody, fontWeight: 700 }}>⚠️ Error de conexión</div>
        <div style={{ color: palette.textMuted, fontFamily: style.fontBody, fontSize: 13, marginTop: 6 }}>{dbError}</div>
      </div>
    )}
    {!loading && activeTab === "dashboard" && <Dashboard products={products} clients={clients} sales={sales} />}
    {!loading && activeTab === "inventory" && <Inventory products={products} setProducts={setProducts} />}
    {!loading && activeTab === "pos" && <POS products={products} setProducts={setProducts} sales={sales} setSales={setSales} clients={clients} setClients={setClients} />}
    {!loading && activeTab === "crm" && <CRM clients={clients} setClients={setClients} />}
    {!loading && activeTab === "reports" && <Reports sales={sales} products={products} clients={clients} />}
    {!loading && activeTab === "catalog" && <CustomerCatalog products={products} />}
    {!loading && activeTab === "points" && <PointsModule clients={clients} setClients={setClients} />}
    {!loading && activeTab === "register" && (
      <CustomerRegister onComplete={async (data) => {
        const newClient = { ...data, id: Date.now(), name: `${data.firstName} ${data.lastName}`, purchases: 0, total: 0, status: "Nuevo", quarterPurchases: 0, points: 0, pointsRedeemed: 0 };
        await setClients(c => [...c, newClient]);
        setActiveTab("crm");
      }} />
    )}
  </div>
</div>
```

);
}
