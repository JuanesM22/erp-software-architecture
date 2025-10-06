# erp-software-architecture
"use client";

import { useMemo, useState } from "react";
import styles from "./page.module.css";

// ================== Utilidades ==================
const uid = () => Math.random().toString(36).slice(2, 9);
const currency = (n) =>
  new Intl.NumberFormat("es-CO", { style: "currency", currency: "COP" }).format(
    Number(n || 0)
  );

const MODULOS = [
  { id: "dashboard", label: "Inicio" },
  { id: "acceso", label: "Acceso" },
  { id: "clientes", label: "Clientes" },
  { id: "proveedores", label: "Proveedores" },
  { id: "productos", label: "Productos" },
  { id: "inventario", label: "Inventario" },
  { id: "compras", label: "Compras" },
  { id: "ventas", label: "Ventas / Facturación" },
];

// ================== Datos demo ==================
const seedClientes = [
  {
    id: uid(),
    nombre: "Juan Pérez",
    documento: "1020304050",
    telefono: "3001234567",
    email: "juan@example.com",
  },
  {
    id: uid(),
    nombre: "Ana Gómez",
    documento: "1098765432",
    telefono: "3017654321",
    email: "ana@example.com",
  },
];

const seedProveedores = [
  {
    id: uid(),
    nombre: "Terpel Mayorista",
    nit: "900123456-7",
    telefono: "6011234567",
    email: "ventas@terpel.com",
  },
];

const seedProductos = [
  {
    id: uid(),
    codigo: "GAS-EXTRA",
    nombre: "Gasolina Extra",
    categoria: "Combustible",
    precio: 13800,
  },
  {
    id: uid(),
    codigo: "GAS-CORRIENTE",
    nombre: "Gasolina Corriente",
    categoria: "Combustible",
    precio: 12600,
  },
  { id: uid(), codigo: "DIE-ACPM", nombre: "ACPM", categoria: "Combustible", precio: 11900 },
  { id: uid(), codigo: "ACE-10W40", nombre: "Aceite 10W-40", categoria: "Lubricantes", precio: 32000 },
];

const seedInventario = seedProductos.map((p) => ({
  id: uid(),
  codigo: p.codigo,
  producto: p.nombre,
  stock: 100,
  um: p.categoria === "Combustible" ? "gal" : "und",
}));

// ================== Página ==================
export default function Page() {
  const [mod, setMod] = useState("dashboard");
  const [query, setQuery] = useState("");
  const [usuario, setUsuario] = useState({
    logged: true,
    rol: "Administrador",
    nombre: "Operador",
  });

  const [clientes, setClientes] = useState(seedClientes);
  const [proveedores, setProveedores] = useState(seedProveedores);
  const [productos, setProductos] = useState(seedProductos);
  const [inventario, setInventario] = useState(seedInventario);
  const [compras, setCompras] = useState([]); // {id, fecha, proveedorId, items:[{codigo,cant,costo}], total}
  const [ventas, setVentas] = useState([]); // {id, fecha, clienteId, items:[{codigo,cant,precio}], total}

  const moduloSeleccionado = useMemo(
    () => MODULOS.find((m) => m.id === mod)?.label || "",
    [mod]
  );

  return (
    <div className={styles.app}>
      <aside className={styles.sidebar}>
        <div className={styles.brand}>⛽ ERP Estación</div>
        <nav className={styles.nav}>
          {MODULOS.map((m) => (
            <button
              key={m.id}
              className={${styles.navItem} ${mod === m.id ? styles.active : ""}}
              onClick={() => setMod(m.id)}
            >
              {m.label}
            </button>
          ))}
        </nav>
        <div className={styles.sidebarFooter}>
          <small>
            Usuario: <b>{usuario?.nombre}</b>
          </small>
          <button
            className={styles.btnOutline}
            onClick={() => setUsuario((u) => ({ ...u, logged: !u.logged }))}
          >
            {usuario.logged ? "Cerrar sesión" : "Iniciar sesión"}
          </button>
        </div>
      </aside>

      <main className={styles.content}>
        <header className={styles.topbar}>
          <h1>{moduloSeleccionado}</h1>
          <input
            value={query}
            onChange={(e) => setQuery(e.target.value)}
            className={styles.search}
            placeholder="Buscar… (Ctrl+/)"
          />
        </header>

        {mod === "dashboard" && (
          <Dashboard
            clientes={clientes}
            proveedores={proveedores}
            productos={productos}
            inventario={inventario}
            ventas={ventas}
            compras={compras}
          />
        )}

        {mod === "acceso" && <Acceso usuario={usuario} setUsuario={setUsuario} />}

        {mod === "clientes" && (
          <ModuloCRUD
            title="Clientes"
            schema={[
              { k: "nombre", label: "Nombre" },
              { k: "documento", label: "Documento" },
              { k: "telefono", label: "Teléfono" },
              { k: "email", label: "Email" },
            ]}
            rows={clientes}
            setRows={setClientes}
            query={query}
          />
        )}

        {mod === "proveedores" && (
          <ModuloCRUD
            title="Proveedores"
            schema={[
              { k: "nombre", label: "Nombre" },
              { k: "nit", label: "NIT" },
              { k: "telefono", label: "Teléfono" },
              { k: "email", label: "Email" },
            ]}
            rows={proveedores}
            setRows={setProveedores}
            query={query}
          />
        )}

        {mod === "productos" && (
          <ModuloCRUD
            title="Productos"
            schema={[
              { k: "codigo", label: "Código" },
              { k: "nombre", label: "Nombre" },
              { k: "categoria", label: "Categoría" },
              { k: "precio", label: "Precio (COP)", type: "number" },
            ]}
            rows={productos}
            setRows={setProductos}
            query={query}
          />
        )}

        {mod === "inventario" && (
          <Inventario
            inventario={inventario}
            setInventario={setInventario}
            query={query}
          />
        )}

        {mod === "compras" && (
          <Compras
            proveedores={proveedores}
            productos={productos}
            inventario={inventario}
            setInventario={setInventario}
            compras={compras}
            setCompras={setCompras}
          />
        )}

        {mod === "ventas" && (
          <Ventas
            clientes={clientes}
            productos={productos}
            ventas={ventas}
            setVentas={setVentas}
            inventario={inventario}
            setInventario={setInventario}
          />
        )}
      </main>
    </div>
  );
}

// ================== Componentes ==================
function Dashboard({ clientes, proveedores, productos, inventario, ventas, compras }) {
  const totalStock = inventario.reduce((a, b) => a + Number(b.stock || 0), 0);
  const totalProductos = productos.length;
  const totalClientes = clientes.length;
  const totalProvs = proveedores.length;
  const totalVentas = ventas.reduce((a, v) => a + (v.total || 0), 0);
  const totalCompras = compras.reduce((a, c) => a + (c.total || 0), 0);

  return (
    <section className={styles.cards}>
      <Card title="Clientes" value={totalClientes} subtitle="Registrados" />
      <Card title="Proveedores" value={totalProvs} subtitle="Activos" />
      <Card title="Productos" value={totalProductos} subtitle="Catálogo" />
      <Card title="Inventario" value={totalStock} subtitle="Unidades totales" />
      <Card title="Ventas (hoy)" value={currency(totalVentas)} subtitle="Facturado" />
      <Card title="Compras (hoy)" value={currency(totalCompras)} subtitle="Invertido" />
    </section>
  );
}

function Card({ title, value, subtitle }) {
  return (
    <div className={styles.card}>
      <div className={styles.cardTitle}>{title}</div>
      <div className={styles.cardValue}>{value}</div>
      <div className={styles.cardSub}>{subtitle}</div>
    </div>
  );
}

function Acceso({ usuario, setUsuario }) {
  const [form, setForm] = useState({ email: "", pass: "" });
  return (
    <section className={styles.panel}>
      <h2>Inicio de sesión</h2>
      {usuario.logged ? (
        <div className={${styles.notice} ${styles.success}}>
          Sesión activa como <b>{usuario.nombre}</b> ({usuario.rol})
        </div>
      ) : (
        <div className={${styles.notice} ${styles.warn}}>No has iniciado sesión</div>
      )}
      <div className={styles.formGrid}>
        <label>
          Email
          <input
            type="email"
            value={form.email}
            onChange={(e) => setForm({ ...form, email: e.target.value })}
          />
        </label>
        <label>
          Contraseña
          <input
            type="password"
            value={form.pass}
            onChange={(e) => setForm({ ...form, pass: e.target.value })}
          />
        </label>
      </div>
      <div className={styles.rowGap}>
        <button
          className={styles.btn}
          onClick={() =>
            setUsuario({
              logged: true,
              rol: "Administrador",
              nombre: form.email.split("@")[0] || "Operador",
            })
          }
        >
          Entrar
        </button>
        <button className={styles.btnOutline} onClick={() => setUsuario({ logged: false })}>
          Salir
        </button>
      </div>
      <p className={styles.muted}>* Demo sin backend. Implementa tu API/JWT aquí.</p>
    </section>
  );
}

function ModuloCRUD({ title, schema, rows, setRows, query }) {
  const empty = Object.fromEntries(schema.map((s) => [s.k, ""]));
  const [form, setForm] = useState(empty);
  const [editing, setEditing] = useState(null);

  const filtered = rows.filter((r) =>
    JSON.stringify(r).toLowerCase().includes((query || "").toLowerCase())
  );

  const onSubmit = (e) => {
    e.preventDefault();
    if (editing) {
      setRows(rows.map((r) => (r.id === editing ? { ...r, ...form } : r)));
      setEditing(null);
    } else {
      setRows([{ id: uid(), ...form }, ...rows]);
    }
    setForm(empty);
  };

  const onEdit = (id) => {
    const row = rows.find((r) => r.id === id);
    if (!row) return;
    setForm(Object.fromEntries(schema.map((s) => [s.k, row[s.k] ?? ""])));
    setEditing(id);
  };

  const onDelete = (id) => setRows(rows.filter((r) => r.id !== id));

  return (
    <section className={styles.panel}>
      <h2>{title}</h2>
      <form onSubmit={onSubmit} className={styles.formGrid}>
        {schema.map((s) => (
          <label key={s.k}>
            {s.label}
            <input
              type={s.type || "text"}
              value={form[s.k]}
              onChange={(e) =>
                setForm({
                  ...form,
                  [s.k]: s.type === "number" ? Number(e.target.value) : e.target.value,
                })
              }
            />
          </label>
        ))}
        <div className={styles.rowGap}>
          <button className={styles.btn} type="submit">
            {editing ? "Guardar cambios" : "Agregar"}
          </button>
          {editing && (
            <button
              type="button"
              className={styles.btnOutline}
              onClick={() => {
                setEditing(null);
                setForm(empty);
              }}
            >
              Cancelar
            </button>
          )}
        </div>
      </form>

      <div className={styles.tableWrap}>
        <table className={styles.table}>
          <thead>
            <tr>
              {schema.map((s) => (
                <th key={s.k}>{s.label}</th>
              ))}
              <th>Acciones</th>
            </tr>
          </thead>
          <tbody>
            {filtered.map((r) => (
              <tr key={r.id}>
                {schema.map((s) => (
                  <td key={s.k}>{String(r[s.k] ?? "")}</td>
                ))}
                <td className={styles.actions}>
                  <button className={styles.btnXS} onClick={() => onEdit(r.id)}>
                    Editar
                  </button>
                  <button
                    className={${styles.btnXS} ${styles.danger}}
                    onClick={() => onDelete(r.id)}
                  >
                    Eliminar
                  </button>
                </td>
              </tr>
            ))}
            {!filtered.length && (
              <tr>
                <td className={${styles.muted} ${styles.center}} colSpan={schema.length + 1}>
                  Sin resultados
                </td>
              </tr>
            )}
          </tbody>
        </table>
      </div>
    </section>
  );
}

function Inventario({ inventario, setInventario, query }) {
  const [ajuste, setAjuste] = useState({ codigo: "", delta: 0 });
  const filtered = inventario.filter((r) =>
    JSON.stringify(r).toLowerCase().includes((query || "").toLowerCase())
  );

  const aplicarAjuste = () => {
    if (!ajuste.codigo) return;
    setInventario((inv) =>
      inv.map((i) =>
        i.codigo === ajuste.codigo ? { ...i, stock: Number(i.stock) + Number(ajuste.delta) } : i
      )
    );
    setAjuste({ codigo: "", delta: 0 });
  };

  return (
    <section className={styles.panel}>
      <h2>Inventario</h2>

      <div className={styles.formRow}>
        <label>
          Código
          <input
            value={ajuste.codigo}
            onChange={(e) => setAjuste({ ...ajuste, codigo: e.target.value })}
            placeholder="Ej: GAS-EXTRA"
          />
        </label>
        <label>
          Ajuste
          <input
            type="number"
            value={ajuste.delta}
            onChange={(e) => setAjuste({ ...ajuste, delta: Number(e.target.value) })}
            placeholder="+10 / -5"
          />
        </label>
        <button className={styles.btn} onClick={aplicarAjuste}>
          Aplicar
        </button>
      </div>

      <div className={styles.tableWrap}>
        <table className={styles.table}>
          <thead>
            <tr>
              <th>Código</th>
              <th>Producto</th>
              <th>Stock</th>
              <th>UM</th>
            </tr>
          </thead>
          <tbody>
            {filtered.map((r) => (
              <tr key={r.id}>
                <td>{r.codigo}</td>
                <td>{r.producto}</td>
                <td>{r.stock}</td>
                <td>{r.um}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </section>
  );
}

function Compras({
  proveedores,
  productos,
  inventario,
  setInventario,
  compras,
  setCompras,
}) {
  const [proveedorId, setProveedorId] = useState(proveedores[0]?.id || "");
  const [items, setItems] = useState([]); // {codigo,cant,costo}

  const addItem = () => setItems((it) => [...it, { codigo: "", cant: 1, costo: 0 }]);
  const updItem = (i, patch) =>
    setItems((it) => it.map((x, idx) => (idx === i ? { ...x, ...patch } : x)));
  const delItem = (i) => setItems((it) => it.filter((_, idx) => idx !== i));

  const total = items.reduce((a, b) => a + Number(b.cant || 0) * Number(b.costo || 0), 0);

  const registrar = () => {
    const id = uid();
    setCompras([{ id, fecha: new Date().toISOString(), proveedorId, items, total }, ...compras]);
    // Actualiza inventario
    setInventario((inv) =>
      items.reduce((acc, it) => {
        const i = acc.find((x) => x.codigo === it.codigo);
        if (i) i.stock = Number(i.stock) + Number(it.cant || 0);
        return [...acc];
      }, [...inventario])
    );
    setItems([]);
  };

  return (
    <section className={styles.panel}>
      <h2>Compras</h2>
      <div className={styles.formRow}>
        <label>
          Proveedor
          <select value={proveedorId} onChange={(e) => setProveedorId(e.target.value)}>
            {proveedores.map((p) => (
              <option key={p.id} value={p.id}>
                {p.nombre}
              </option>
            ))}
          </select>
        </label>
        <button className={styles.btn} onClick={addItem}>
          Agregar Item
        </button>
      </div>

      {items.map((it, i) => (
        <div key={i} className={styles.formRow}>
          <label>
            Producto
            <select
              value={it.codigo}
              onChange={(e) => updItem(i, { codigo: e.target.value })}
            >
              <option value="">-- Selecciona --</option>
              {productos.map((p) => (
                <option key={p.id} value={p.codigo}>
                  {p.codigo} — {p.nombre}
                </option>
              ))}
            </select>
          </label>
          <label>
            Cantidad
            <input
              type="number"
              value={it.cant}
              onChange={(e) => updItem(i, { cant: Number(e.target.value) })}
            />
          </label>
          <label>
            Costo (COP)
            <input
              type="number"
              value={it.costo}
              onChange={(e) => updItem(i, { costo: Number(e.target.value) })}
            />
          </label>
          <button className={styles.btnOutline} onClick={() => delItem(i)}>
            Quitar
          </button>
        </div>
      ))}

      <div className={styles.rowBetween}>
        <div className={styles.muted}>Items: {items.length}</div>
        <div className={styles.total}>Total: {currency(total)}</div>
      </div>

      <div className={styles.rowGap}>
        <button disabled={!items.length} className={styles.btn} onClick={registrar}>
          Registrar compra
        </button>
      </div>
    </section>
  );
}

function Ventas({
  clientes,
  productos,
  ventas,
  setVentas,
  inventario,
  setInventario,
}) {
  const [clienteId, setClienteId] = useState(clientes[0]?.id || "");
  const [items, setItems] = useState([]); // {codigo,cant,precio}

  const addItem = () => setItems((it) => [...it, { codigo: "", cant: 1, precio: 0 }]);
  const updItem = (i, patch) =>
    setItems((it) => it.map((x, idx) => (idx === i ? { ...x, ...patch } : x)));
  const delItem = (i) => setItems((it) => it.filter((_, idx) => idx !== i));

  // precio sugerido según producto
  const precioSugerido = (codigo) =>
    productos.find((p) => p.codigo === codigo)?.precio || 0;

  const total = items.reduce((a, b) => a + Number(b.cant || 0) * Number(b.precio || 0), 0);

  const facturar = () => {
    const id = uid();
    setVentas([{ id, fecha: new Date().toISOString(), clienteId, items, total }, ...ventas]);
    // Descuenta inventario
    setInventario((inv) =>
      items.reduce((acc, it) => {
        const i = acc.find((x) => x.codigo === it.codigo);
        if (i) i.stock = Number(i.stock) - Number(it.cant || 0);
        return [...acc];
      }, [...inventario])
    );
    setItems([]);
  };

  return (
    <section className={styles.panel}>
      <h2>Ventas / Facturación</h2>
      <div className={styles.formRow}>
        <label>
          Cliente
          <select value={clienteId} onChange={(e) => setClienteId(e.target.value)}>
            {clientes.map((c) => (
              <option key={c.id} value={c.id}>
                {c.nombre}
              </option>
            ))}
          </select>
        </label>
        <button className={styles.btn} onClick={addItem}>
          Agregar Item
        </button>
      </div>

      {items.map((it, i) => (
        <div key={i} className={styles.formRow}>
          <label>
            Producto
            <select
              value={it.codigo}
              onChange={(e) => {
                const codigo = e.target.value;
                updItem(i, { codigo, precio: precioSugerido(codigo) });
              }}
            >
              <option value="">-- Selecciona --</option>
              {productos.map((p) => (
                <option key={p.id} value={p.codigo}>
                  {p.codigo} — {p.nombre}
                </option>
              ))}
            </select>
          </label>
          <label>
            Cantidad
            <input
              type="number"
              value={it.cant}
              onChange={(e) => updItem(i, { cant: Number(e.target.value) })}
            />
          </label>
          <label>
            Precio (COP)
            <input
              type="number"
              value={it.precio}
              onChange={(e) => updItem(i, { precio: Number(e.target.value) })}
            />
          </label>
          <button className={styles.btnOutline} onClick={() => delItem(i)}>
            Quitar
          </button>
        </div>
      ))}

      <div className={styles.rowBetween}>
        <div className={styles.muted}>Items: {items.length}</div>
        <div className={styles.total}>Total: {currency(total)}</div>
      </div>

      <div className={styles.rowGap}>
        <button disabled={!items.length} className={styles.btn} onClick={facturar}>
          Facturar
        </button>
      </div>
    </section>
  );
}


/* Layout base */
.app {
  display: grid;
  grid-template-columns: 260px 1fr;
  min-height: 100dvh;
  background: #0b0f17;
  color: #e7eaf0;
  font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Arial, "Noto Sans",
    "Helvetica Neue", sans-serif;
}

/* Sidebar */
.sidebar {
  border-right: 1px solid #1b2433;
  padding: 20px 16px;
  background: #0f1522;
  display: flex;
  flex-direction: column;
  gap: 16px;
}
.brand {
  font-weight: 800;
  letter-spacing: 0.3px;
  background: linear-gradient(90deg, #7dd3fc, #a78bfa, #f472b6);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  font-size: 18px;
}
.nav { display: grid; gap: 8px; }
.navItem {
  text-align: left;
  padding: 10px 12px;
  background: #0b0f17;
  border: 1px solid #1b2433;
  color: #b9c2d0;
  border-radius: 10px;
  cursor: pointer;
  transition: transform 0.08s ease, background 0.2s;
}
.navItem:hover { transform: translateY(-1px); background: #121a28; }
.active { border-color: #7dd3fc; color: #e7eaf0; box-shadow: 0 0 0 3px rgba(125,211,252,0.15) inset; }
.sidebarFooter { margin-top: auto; display: grid; gap: 10px; }

/* Main */
.content { padding: 18px 22px; }
.topbar {
  display: flex;
  gap: 12px;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 18px;
}
.search {
  background: #0b0f17;
  border: 1px solid #1b2433;
  color: #e7eaf0;
  padding: 10px 12px;
  border-radius: 10px;
  outline: none;
  min-width: 240px;
}
.search::placeholder { color: #8a97ab; }

/* Cards Dashboard */
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 12px;
}
.card {
  background: #0f1522;
  border: 1px solid #1b2433;
  border-radius: 16px;
  padding: 14px 16px;
}
.cardTitle { color: #97a6ba; font-size: 12px; }
.cardValue { font-size: 24px; font-weight: 700; margin-top: 6px; }
.cardSub { color: #7e8aa0; font-size: 12px; margin-top: 4px; }

/* Panel genérico */
.panel {
  background: #0f1522;
  border: 1px solid #1b2433;
  border-radius: 16px;
  padding: 16px;
}
.notice {
  padding: 10px 12px;
  border-radius: 10px;
  font-size: 14px;
  border: 1px solid transparent;
  margin-bottom: 10px;
}
.success { background: rgba(34,197,94,0.08); border-color: rgba(34,197,94,0.25); }
.warn    { background: rgba(234,179,8,0.08); border-color: rgba(234,179,8,0.25); }
.muted { color: #8a97ab; }
.center { text-align: center; }

/* Formularios (ahora bajo una clase local) */
.formGrid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px,1fr));
  gap: 10px;
  margin: 10px 0 14px;
}
.formRow {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px,1fr));
  gap: 10px;
  margin-bottom: 10px;
}

/* Reemplaza 'label' global por una clase local */
.field { font-size: 12px; color: #97a6ba; display: grid; gap: 6px; }

/* Reemplaza 'input, select' global por selectores puros bajo .field */
.field input,
.field select {
  background: #0b0f17;
  border: 1px solid #1b2433;
  color: #e7eaf0;
  padding: 10px 12px;
  border-radius: 10px;
  outline: none;
}

.rowGap { display: flex; gap: 10px; }
.rowBetween { display: flex; justify-content: space-between; align-items: center; }

/* Botones */
.btn, .btnOutline, .btnXS {
  border-radius: 10px;
  border: 1px solid #1b2433;
  background: #101724;
  color: #e7eaf0;
  cursor: pointer;
}
.btn { padding: 10px 12px; }
.btnOutline { padding: 10px 12px; background: transparent; }
.btnXS { padding: 6px 8px; font-size: 12px; }
.danger { border-color: #ef4444; color: #fecaca; }

/* Tabla */
.tableWrap { overflow: auto; border-radius: 12px; border: 1px solid #1b2433; }
.table { width: 100%; border-collapse: collapse; }
.table th, .table td { padding: 10px 12px; border-bottom: 1px solid #1b2433; text-align: left; }
.actions { display: flex; gap: 6px; align-items: center; }
.total { font-weight: 700; }

/* Scrollbar sutil (ambos son clases locales, OK) */
.tableWrap::-webkit-scrollbar, .sidebar::-webkit-scrollbar { height: 8px; width: 8px; }
.tableWrap::-webkit-scrollbar-thumb, .sidebar::-webkit-scrollbar-thumb { background: #1b2433; border-radius: 10px; }
