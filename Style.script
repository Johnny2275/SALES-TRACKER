
const STORAGE_KEY = "daily-sales-tracker-v1";

const salesForm = document.getElementById("sales-form");
const salesTableBody = document.getElementById("sales-table-body");
const monthFilter = document.getElementById("month-filter");
const searchInput = document.getElementById("search-input");
const emptyState = document.getElementById("empty-state");
const exportBtn = document.getElementById("export-btn");
const clearBtn = document.getElementById("clear-btn");

const todayRevenueEl = document.getElementById("today-revenue");
const monthRevenueEl = document.getElementById("month-revenue");
const transactionCountEl = document.getElementById("transaction-count");
const averageOrderEl = document.getElementById("average-order");

const dateInput = document.getElementById("sale-date");
const itemInput = document.getElementById("sale-item");
const categoryInput = document.getElementById("sale-category");
const quantityInput = document.getElementById("sale-quantity");
const priceInput = document.getElementById("sale-price");
const paymentInput = document.getElementById("sale-payment");
const notesInput = document.getElementById("sale-notes");

let sales = loadSales();

dateInput.value = toDateInputValue(new Date());
monthFilter.value = toMonthInputValue(new Date());
render();

salesForm.addEventListener("submit", (event) => {
  event.preventDefault();

  const quantity = Number(quantityInput.value);
  const price = Number(priceInput.value);
  const total = quantity * price;

  const sale = {
    id: crypto.randomUUID(),
    date: dateInput.value,
    item: itemInput.value.trim(),
    category: categoryInput.value.trim(),
    quantity,
    price,
    total,
    payment: paymentInput.value,
    notes: notesInput.value.trim(),
    createdAt: new Date().toISOString(),
  };

  sales.unshift(sale);
  persistSales();
  salesForm.reset();
  dateInput.value = toDateInputValue(new Date());
  quantityInput.value = "1";
  render();
});

searchInput.addEventListener("input", render);
monthFilter.addEventListener("input", render);

salesTableBody.addEventListener("click", (event) => {
  if (!event.target.classList.contains("delete-btn")) {
    return;
  }

  const id = event.target.dataset.id;
  sales = sales.filter((sale) => sale.id !== id);
  persistSales();
  render();
});

clearBtn.addEventListener("click", () => {
  if (!sales.length) return;

  const confirmed = window.confirm("Delete all saved sales records?");
  if (!confirmed) return;

  sales = [];
  persistSales();
  render();
});

exportBtn.addEventListener("click", () => {
  const filteredSales = getFilteredSales();
  if (!filteredSales.length) {
    window.alert("No sales to export.");
    return;
  }

  const csvHeaders = [
    "Date",
    "Item",
    "Category",
    "Quantity",
    "Unit Price",
    "Total",
    "Payment",
    "Notes",
  ];

  const rows = filteredSales.map((sale) => [
    sale.date,
    sale.item,
    sale.category,
    sale.quantity,
    sale.price.toFixed(2),
    sale.total.toFixed(2),
    sale.payment,
    sale.notes,
  ]);

  const csvContent = [csvHeaders, ...rows]
    .map((row) => row.map((cell) => `"${String(cell).replaceAll('"', '""')}"`).join(","))
    .join("\n");

  const blob = new Blob([csvContent], { type: "text/csv;charset=utf-8;" });
  const url = URL.createObjectURL(blob);

  const link = document.createElement("a");
  link.setAttribute("href", url);
  link.setAttribute("download", `sales-${new Date().toISOString().slice(0, 10)}.csv`);
  link.style.display = "none";
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
  URL.revokeObjectURL(url);
});

function render() {
  const filteredSales = getFilteredSales();
  renderTable(filteredSales);
  renderStats();
}

function getFilteredSales() {
  const monthValue = monthFilter.value;
  const query = searchInput.value.trim().toLowerCase();

  return sales.filter((sale) => {
    const matchesMonth = monthValue ? sale.date.startsWith(monthValue) : true;

    const searchField = `${sale.item} ${sale.category} ${sale.payment} ${sale.notes}`.toLowerCase();
    const matchesQuery = query ? searchField.includes(query) : true;

    return matchesMonth && matchesQuery;
  });
}

function renderTable(entries) {
  salesTableBody.innerHTML = "";

  if (!entries.length) {
    emptyState.style.display = "block";
    return;
  }

  emptyState.style.display = "none";

  entries.forEach((sale) => {
    const row = document.createElement("tr");
    row.innerHTML = `
      <td>${sale.date}</td>
      <td>${escapeHtml(sale.item)}</td>
      <td>${escapeHtml(sale.category)}</td>
      <td>${sale.quantity}</td>
      <td>${formatCurrency(sale.price)}</td>
      <td>${formatCurrency(sale.total)}</td>
      <td>${escapeHtml(sale.payment)}</td>
      <td class="notes" title="${escapeHtml(sale.notes || "-")}">${escapeHtml(sale.notes || "-")}</td>
      <td><button class="delete-btn" data-id="${sale.id}" type="button">Delete</button></td>
    `;
    salesTableBody.appendChild(row);
  });
}

function renderStats() {
  const today = toDateInputValue(new Date());
  const currentMonth = toMonthInputValue(new Date());

  const todayRevenue = sales
    .filter((sale) => sale.date === today)
    .reduce((sum, sale) => sum + sale.total, 0);

  const monthRevenue = sales
    .filter((sale) => sale.date.startsWith(currentMonth))
    .reduce((sum, sale) => sum + sale.total, 0);

  const transactionCount = sales.length;
  const averageOrder = transactionCount
    ? sales.reduce((sum, sale) => sum + sale.total, 0) / transactionCount
    : 0;

  todayRevenueEl.textContent = formatCurrency(todayRevenue);
  monthRevenueEl.textContent = formatCurrency(monthRevenue);
  transactionCountEl.textContent = String(transactionCount);
  averageOrderEl.textContent = formatCurrency(averageOrder);
}

function loadSales() {
  const raw = localStorage.getItem(STORAGE_KEY);
  if (!raw) return [];

  try {
    const parsed = JSON.parse(raw);
    return Array.isArray(parsed) ? parsed : [];
  } catch {
    return [];
  }
}

function persistSales() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(sales));
}

function formatCurrency(value) {
  return new Intl.NumberFormat("en-NG", {
    style: "currency",
    currency: "NGN",
  }).format(value);
}

function toDateInputValue(date) {
  return date.toISOString().slice(0, 10);
}

function toMonthInputValue(date) {
  return date.toISOString().slice(0, 7);
}

function escapeHtml(value) {
  return value
    .replaceAll("&", "&amp;")
    .replaceAll("<", "&lt;")
    .replaceAll(">", "&gt;")
    .replaceAll('"', "&quot;")
    .replaceAll("'", "&#39;");
}
