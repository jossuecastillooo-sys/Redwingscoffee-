
import { useMemo, useState } from "react";
import {
  Search,
  ShoppingBag,
  Home,
  Coffee,
  UserRound,
  Plus,
  Minus,
  X,
  ArrowLeft,
  Clock,
  ChevronRight,
  Heart,
  CheckCircle2,
  MapPin,
} from "lucide-react";
import "./index.css";

type Product = {
  id: number;
  name: string;
  description: string;
  price: number;
  category: string;
  image: string;
  popular?: boolean;
};

type CartItem = {
  product: Product;
  quantity: number;
};

const products: Product[] = [
  {
    id: 1,
    name: "Granita tradicional",
    description: "Café frío con sabor intenso.",
    price: 45,
    category: "Granitas",
    image: "https://images.unsplash.com/photo-1517701604599-bb29b565090c?w=700",
    popular: true,
  },
  {
    id: 2,
    name: "Granita con crema",
    description: "Café frío con crema suave.",
    price: 50,
    category: "Granitas",
    image: "https://images.unsplash.com/photo-1461023058943-07fcbe16d735?w=700",
    popular: true,
  },
  {
    id: 3,
    name: "Frappé de café",
    description: "Café frappé con lechera.",
    price: 55,
    category: "Frappés",
    image: "https://images.unsplash.com/photo-1572490122747-3968b75cc699?w=700",
    popular: true,
  },
  {
    id: 4,
    name: "Frappé Pinguinera",
    description: "Una mezcla cremosa y deliciosa.",
    price: 65,
    category: "Frappés",
    image: "https://images.unsplash.com/photo-1461023058943-07fcbe16d735?w=700",
  },
  {
    id: 5,
    name: "Frappé Oreo",
    description: "Café frío con sabor a Oreo.",
    price: 70,
    category: "Frappés",
    image: "https://images.unsplash.com/photo-1572490122747-3968b75cc699?w=700",
  },
  {
    id: 6,
    name: "Boli Oreo",
    description: "Boli largo con sabor a Oreo.",
    price: 30,
    category: "Bolis",
    image: "https://images.unsplash.com/photo-1563805042-7684c019e1cb?w=700",
  },
  {
    id: 7,
    name: "Café tradicional",
    description: "Café Redwings de 1 libra.",
    price: 150,
    category: "Café",
    image: "https://images.unsplash.com/photo-1447933601403-0c6688de566e?w=700",
  },
  {
    id: 8,
    name: "Café con pimienta",
    description: "Café Redwings de 1 libra.",
    price: 165,
    category: "Café",
    image: "https://images.unsplash.com/photo-1447933601403-0c6688de566e?w=700",
  },
];

const categories = [
  "Todos",
  "Granitas",
  "Frappés",
  "Bolis",
  "Café",
];

function money(value: number) {
  return `L ${value.toFixed(2)}`;
}

export default function App() {
  const [category, setCategory] = useState("Todos");
  const [search, setSearch] = useState("");
  const [cart, setCart] = useState<CartItem[]>([]);
  const [cartOpen, setCartOpen] = useState(false);
  const [checkoutOpen, setCheckoutOpen] = useState(false);
  const [favorites, setFavorites] = useState<number[]>([]);
  const [customer, setCustomer] = useState({
    name: "",
    phone: "",
    career: "",
    section: "",
  });
  const [orderComplete, setOrderComplete] = useState(false);
  const [orderCode, setOrderCode] = useState("");

  const filteredProducts = useMemo(() => {
    return products.filter((product) => {
      const matchesCategory =
        category === "Todos" || product.category === category;

      const matchesSearch = product.name
        .toLowerCase()
        .includes(search.toLowerCase());

      return matchesCategory && matchesSearch;
    });
  }, [category, search]);

  const cartCount = cart.reduce(
    (total, item) => total + item.quantity,
    0,
  );

  const subtotal = cart.reduce(
    (total, item) =>
      total + item.product.price * item.quantity,
    0,
  );

  function addProduct(product: Product) {
    setCart((previous) => {
      const existing = previous.find(
        (item) => item.product.id === product.id,
      );

      if (existing) {
        return previous.map((item) =>
          item.product.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item,
        );
      }

      return [...previous, { product, quantity: 1 }];
    });
  }

  function changeQuantity(productId: number, amount: number) {
    setCart((previous) =>
      previous
        .map((item) =>
          item.product.id === productId
            ? {
                ...item,
                quantity: item.quantity + amount,
              }
            : item,
        )
        .filter((item) => item.quantity > 0),
    );
  }

  function toggleFavorite(productId: number) {
    setFavorites((previous) =>
      previous.includes(productId)
        ? previous.filter((id) => id !== productId)
        : [...previous, productId],
    );
  }

  function submitOrder() {
    if (
      !customer.name.trim() ||
      !customer.career.trim() ||
      !customer.section.trim()
    ) {
      alert("Completa tu nombre, carrera y sección.");
      return;
    }

    if (cart.length === 0) {
      alert("Agrega productos al carrito.");
      return;
    }

    const code = `RW-${Date.now().toString().slice(-6)}`;

    const message = [
      "☕ REDWINGS COFFEE",
      "",
      `Pedido: ${code}`,
      `Cliente: ${customer.name}`,
      `Carrera: ${customer.career}`,
      `Sección: ${customer.section}`,
      `Teléfono: ${customer.phone || "No indicado"}`,
      "",
      "PRODUCTOS:",
      ...cart.map(
        (item) =>
          `${item.quantity}x ${item.product.name} - ${money(
            item.product.price * item.quantity,
          )}`,
      ),
      "",
      `TOTAL: ${money(subtotal)}`,
      "",
      "Pago: Efectivo",
      "Entrega: Coordinada en el instituto",
    ].join("\n");

    setOrderCode(code);
    setOrderComplete(true);

    const whatsappUrl =
      "https://wa.me/50492899752?text=" +
      encodeURIComponent(message);

    window.open(whatsappUrl, "_blank");

    setCart([]);
    setCheckoutOpen(false);
    setCartOpen(false);
  }

  return (
    <div className="rw-app">
      {/* HEADER */}
      <header className="rw-header">
        <div className="rw-brand">
          <div className="rw-logo">
            <Coffee size={24} />
          </div>

          <div>
            <h1>Redwings</h1>
            <span>COFFEE</span>
          </div>
        </div>

        <button
          className="rw-cart-button"
          onClick={() => setCartOpen(true)}
          aria-label="Abrir carrito"
        >
          <ShoppingBag size={21} />

          {cartCount > 0 && (
            <span className="rw-cart-count">
              {cartCount}
            </span>
          )}
        </button>
      </header>

      {/* HERO */}
      <section className="rw-hero">
        <div className="rw-hero-content">
          <span className="rw-eyebrow">
            TU CAFÉ FAVORITO
          </span>

          <h2>
            Tu momento.
            <br />
            <span>Tu café.</span>
          </h2>

          <p>
            Sabores que convierten cualquier pausa
            en tu momento favorito.
          </p>

          <button
            className="rw-primary-button"
            onClick={() =>
              document
                .getElementById("rw-menu")
                ?.scrollIntoView({ behavior: "smooth" })
            }
          >
            Explorar menú
            <ChevronRight size={18} />
          </button>
        </div>

        <div className="rw-hero-image">
          <img
            src="https://images.unsplash.com/photo-1461023058943-07fcbe16d735?w=900"
            alt="Café frío Redwings"
          />

          <div className="rw-hero-badge">
            <Coffee size={16} />
            <span>Hecho para disfrutar</span>
          </div>
        </div>
      </section>

      {/* BENEFICIOS */}
      <section className="rw-benefits">
        <div className="rw-benefit">
          <Coffee size={20} />
          <div>
            <strong>Tu café favorito</strong>
            <span>Sabores para disfrutar</span>
          </div>
        </div>

        <div className="rw-benefit">
          <Clock size={20} />
          <div>
            <strong>Pedido sencillo</strong>
            <span>Ordena en minutos</span>
          </div>
        </div>

        <div className="rw-benefit">
          <MapPin size={20} />
          <div>
            <strong>Recogida</strong>
            <span>En el instituto</span>
          </div>
        </div>
      </section>

      {/* MENÚ */}
      <main id="rw-menu" className="rw-menu">
        <div className="rw-section-heading">
          <div>
            <span className="rw-eyebrow">
              DESCUBRE REDWINGS
            </span>

            <h2>Nuestro menú</h2>

            <p>
              Encuentra algo delicioso para tu próxima pausa.
            </p>
          </div>
        </div>

        {/* BUSCADOR */}
        <div className="rw-search">
          <Search size={19} />

          <input
            type="text"
            placeholder="Buscar tu favorito..."
            value={search}
            onChange={(value) =>
              setSearch(value.target.value)
            }
          />

          {search && (
            <button
              onClick={() => setSearch("")}
              aria-label="Limpiar búsqueda"
            >
              <X size={17} />
            </button>
          )}
        </div>

        {/* CATEGORÍAS */}
        <div className="rw-categories">
          {categories.map((item) => (
            <button
              key={item}
              className={
                category === item
                  ? "rw-category active"
                  : "rw-category"
              }
              onClick={() => setCategory(item)}
            >
              {item}
            </button>
          ))}
        </div>

        {/* PRODUCTOS */}
        <div className="rw-product-grid">
          {filteredProducts.map((product) => (
            <article
              className="rw-product-card"
              key={product.id}
            >
              <div className="rw-product-image">
                <img
                  src={product.image}
                  alt={product.name}
                  loading="lazy"
                />

                {product.popular && (
                  <span className="rw-popular">
                    Popular
                  </span>
                )}

                <button
                  className={
                    favorites.includes(product.id)
                      ? "rw-favorite active"
                      : "rw-favorite"
                  }
                  onClick={() =>
                    toggleFavorite(product.id)
                  }
                  aria-label="Agregar a favoritos"
                >
                  <Heart
                    size={18}
                    fill={
                      favorites.includes(product.id)
                        ? "currentColor"
                        : "none"
                    }
                  />
                </button>
              </div>

              <div className="rw-product-info">
                <span className="rw-product-category">
                  {product.category}
                </span>

                <h3>{product.name}</h3>

                <p>{product.description}</p>

                <div className="rw-product-bottom">
                  <strong>{money(product.price)}</strong>

                  <button
                    className="rw-add-button"
                    onClick={() => addProduct(product)}
                    aria-label={`Agregar ${product.name}`}
                  >
                    <Plus size={19} />
                  </button>
                </div>
              </div>
            </article>
          ))}
        </div>

        {filteredProducts.length === 0 && (
          <div className="rw-empty">
            <Search size={30} />

            <h3>No encontramos productos</h3>

            <p>
              Prueba con otra búsqueda o categoría.
            </p>

            <button
              className="rw-primary-button"
              onClick={() => {
                setSearch("");
                setCategory("Todos");
              }}
            >
              Ver todo el menú
            </button>
          </div>
        )}
      </main>

      {/* NAVEGACIÓN INFERIOR */}
      <nav className="rw-bottom-nav">
        <button
          className="active"
          onClick={() =>
            window.scrollTo({
              top: 0,
              behavior: "smooth",
            })
          }
        >
          <Home size={21} />
          <span>Inicio</span>
        </button>

        <button
          onClick={() =>
            document
              .getElementById("rw-menu")
              ?.scrollIntoView({ behavior: "smooth" })
          }
        >
          <Coffee size={21} />
          <span>Menú</span>
        </button>

        <button onClick={() => setCartOpen(true)}>
          <ShoppingBag size={21} />
          <span>Carrito</span>
        </button>

        <button
          onClick={() =>
            alert(
              "La sección de cuenta estará disponible en una próxima versión."
            )
          }
        >
          <UserRound size={21} />
          <span>Cuenta</span>
        </button>
      </nav>

      {/* CARRITO */}
      {cartOpen && (
        <div
          className="rw-overlay"
          onClick={() => setCartOpen(false)}
        >
          <section
            className="rw-drawer"
            onClick={(event) => event.stopPropagation()}
          >
            <div className="rw-drawer-header">
              <div>
                <span className="rw-eyebrow">
                  TU PEDIDO
                </span>

                <h2>Mi carrito</h2>
              </div>

              <button
                className="rw-close"
                onClick={() => setCartOpen(false)}
                aria-label="Cerrar carrito"
              >
                <X size={22} />
              </button>
            </div>

            {cart.length === 0 ? (
              <div className="rw-empty-cart">
                <ShoppingBag size={42} />

                <h3>Tu carrito está vacío</h3>

                <p>
                  Agrega algo delicioso de nuestro menú.
                </p>

                <button
                  className="rw-primary-button"
                  onClick={() => setCartOpen(false)}
                >
                  Explorar menú
                </button>
              </div>
            ) : (
              <>
                <div className="rw-cart-items">
                  {cart.map((item) => (
                    <div
                      className="rw-cart-item"
                      key={item.product.id}
                    >
                      <img
                        src={item.product.image}
                        alt={item.product.name}
                      />

                      <div className="rw-cart-item-info">
                        <h3>{item.product.name}</h3>

                        <strong>
                          {money(item.product.price)}
                        </strong>

                        <div className="rw-quantity">
                          <button
                            onClick={() =>
                              changeQuantity(
                                item.product.id,
                                -1,
                              )
                            }
                            aria-label="Reducir cantidad"
                          >
                            <Minus size={15} />
                          </button>

                          <span>{item.quantity}</span>

                          <button
                            onClick={() =>
                              changeQuantity(
                                item.product.id,
                                1,
                              )
                            }
                            aria-label="Aumentar cantidad"
                          >
                            <Plus size={15} />
                          </button>
                        </div>
                      </div>

                      <strong className="rw-item-total">
                        {money(
                          item.product.price * item.quantity,
                        )}
                      </strong>
                    </div>
                  ))}
                </div>

                <div className="rw-cart-footer">
                  <div className="rw-total-row">
                    <span>Subtotal</span>

                    <strong>{money(subtotal)}</strong>
                  </div>

                  <p>
                    La entrega se coordina después de confirmar
                    tu pedido.
                  </p>

                  <button
                    className="rw-primary-button rw-full"
                    onClick={() => {
                      setCartOpen(false);
                      setCheckoutOpen(true);
                    }}
                  >
                    Continuar pedido
                    <ChevronRight size={18} />
                  </button>
                </div>
              </>
            )}
          </section>
        </div>
      )}

      {/* CHECKOUT */}
      {checkoutOpen && (
        <div
          className="rw-overlay"
          onClick={() => setCheckoutOpen(false)}
        >
          <section
            className="rw-checkout"
            onClick={(event) => event.stopPropagation()}
          >
            <div className="rw-drawer-header">
              <div>
                <span className="rw-eyebrow">
                  CASI LISTO
                </span>

                <h2>Confirmar pedido</h2>
              </div>

              <button
                className="rw-close"
                onClick={() => setCheckoutOpen(false)}
                aria-label="Cerrar"
              >
                <X size={22} />
              </button>
            </div>

            <p className="rw-checkout-description">
              Completa tus datos para preparar tu pedido.
            </p>

            <form
              onSubmit={(event) => {
                event.preventDefault();
                submitOrder();
              }}
            >
              <label>
                Nombre completo *

                <input
                  required
                  value={customer.name}
                  onChange={(event) =>
                    setCustomer({
                      ...customer,
                      name: event.target.value,
                    })
                  }
                  placeholder="Tu nombre"
                />
              </label>

              <label>
                Carrera *

                <input
                  required
                  value={customer.career}
                  onChange={(event) =>
                    setCustomer({
                      ...customer,
                      career: event.target.value,
                    })
                  }
                  placeholder="Administración de Empresas"
                />
              </label>

              <label>
                Sección *

                <input
                  required
                  value={customer.section}
                  onChange={(event) =>
                    setCustomer({
                      ...customer,
                      section: event.target.value,
                    })
                  }
                  placeholder="Ej. 11-A"
                />
              </label>

              <label>
                Teléfono

                <input
                  type="tel"
                  value={customer.phone}
                  onChange={(event) =>
                    setCustomer({
                      ...customer,
                      phone: event.target.value,
                    })
                  }
                  placeholder="Opcional"
                />
              </label>

              <div className="rw-checkout-summary">
                <div>
                  <span>Productos</span>

                  <strong>{cartCount}</strong>
                </div>

                <div>
                  <span>Total</span>

                  <strong>{money(subtotal)}</strong>
                </div>

                <div>
                  <span>Pago</span>

                  <strong>Efectivo</strong>
                </div>
              </div>

              <button
                type="submit"
                className="rw-primary-button rw-full"
              >
                Confirmar pedido
                <ChevronRight size={18} />
              </button>

              <p className="rw-checkout-note">
                El pedido se enviará por WhatsApp para
                coordinar la recogida.
              </p>
            </form>
          </section>
        </div>
      )}

      {/* PEDIDO COMPLETADO */}
      {orderComplete && (
        <div className="rw-overlay">
          <section className="rw-success">
            <div className="rw-success-icon">
              <CheckCircle2 size={38} />
            </div>

            <span className="rw-eyebrow">
              REDWINGS COFFEE
            </span>

            <h2>¡Pedido preparado!</h2>

            <p>
              Tu solicitud fue generada. Revisa WhatsApp
              para coordinar la recogida.
            </p>

            <div className="rw-order-code">
              <span>Código de pedido</span>

              <strong>{orderCode}</strong>
            </div>

            <button
              className="rw-primary-button rw-full"
              onClick={() => setOrderComplete(false)}
            >
              <CheckCircle2 size={18} />
              Entendido
            </button>
          </section>
        </div>
      )}
    </div>
  );
}
