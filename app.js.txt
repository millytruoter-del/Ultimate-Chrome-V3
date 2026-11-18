
const $ = id => document.getElementById(id);

let tabs = [{id:1,title:"New Tab",url:"",history:[],index:-1}];
let active = 1;
let nextId = 2;
let bookmarks = [];
let visits = [];

function current() {
  return tabs.find(t => t.id === active);
}

function renderTabs() {
  $("tabs").innerHTML = "";
  tabs.forEach(t => {
    let el = document.createElement("div");
    el.className = "tab" + (t.id === active ? " active" : "");
    el.innerHTML = `<span>✦</span><span></span><span class="x">×</span>`;
    el.children[1].textContent = t.title;
    el.addEventListener("click", () => {
      active = t.id;
      render();
    });
    el.children[2].addEventListener("click", e => {
      e.stopPropagation();
      closeTab(t.id);
    });
    $("tabs").append(el);
  });
}

function render() {
  renderTabs();
  let t = current();
  $("address").value = t.url || "";
  $("welcome").classList.toggle("hidden", !!t.url);
  $("viewer").classList.toggle("hidden", !t.url);
  $("panel").classList.add("hidden");

  if (t.url) {
    $("webview").src = t.url;
    $("frameNote").textContent =
      "Some websites do not allow embedding.";
  }
}

function normalize(s) {
  s = s.trim();
  if (!s) return "";
  if (/^https?:\/\//i.test(s)) return s;
  if (s.includes(" ") || !s.includes(".")) {
    return "https://www.google.com/search?q=" +
      encodeURIComponent(s);
  }
  return "https://" + s;
}

function navigate(value) {
  let url = normalize(value);
  if (!url) return;

  let t = current();
  t.history = t.history.slice(0, t.index + 1);
  t.history.push(url);
  t.index = t.history.length - 1;
  t.url = url;

  try {
    t.title = new URL(url).hostname.replace(/^www\./, "");
  } catch {
    t.title = "Page";
  }

  visits.unshift({
    url,
    time: new Date().toLocaleString()
  });

  visits = visits.slice(0, 100);
  render();
}

function closeTab(id) {
  if (tabs.length === 1) {
    tabs = [{
      id: nextId++,
      title: "New Tab",
      url: "",
      history: [],
      index: -1
    }];
    active = tabs[0].id;
  } else {
    tabs = tabs.filter(t => t.id !== id);
    if (active === id) active = tabs[tabs.length - 1].id;
  }
  render();
}

function newTab() {
  let t = {
    id: nextId++,
    title: "New Tab",
    url: "",
    history: [],
    index: -1
  };
  tabs.push(t);
  active = t.id;
  render();
}

$("newTab").onclick = newTab;

$("addressForm").onsubmit = e => {
  e.preventDefault();
  navigate($("address").value);
};

$("searchForm").onsubmit = e => {
  e.preventDefault();
  navigate($("searchInput").value);
};

document.querySelectorAll("[data-url]").forEach(b => {
  b.onclick = () => navigate(b.dataset.url);
});

$("home").onclick = () => {
  let t = current();
  t.url = "";
  t.title = "New Tab";
  render();
};

$("reload").onclick = () => {
  if (current().url) $("webview").src = current().url;
};

$("back").onclick = () => {
  let t = current();
  if (t.index > 0) {
    t.index--;
    t.url = t.history[t.index];
    render();
  }
};

$("forward").onclick = () => {
  let t = current();
  if (t.index < t.history.length - 1) {
    t.index++;
    t.url = t.history[t.index];
    render();
  }
};

$("bookmark").onclick = () => {
  let u = current().url;
  if (u && !bookmarks.includes(u)) {
    bookmarks.push(u);
    $("bookmark").textContent = "★";
  }
};

$("history").onclick = () => {
  $("welcome").classList.add("hidden");
  $("viewer").classList.add("hidden");
  $("panel").classList.remove("hidden");

  $("panel").innerHTML =
    "<h2>Browsing history</h2>" +
    (visits.length
      ? visits.map(v =>
          `<li>${escapeHtml(v.url)} <small>— ${v.time}</small></li>`
        ).join("")
      : "<p>No pages visited yet.</p>");
};

function escapeHtml(s) {
  return s.replace(/[&<>"']/g, c => ({
    "&": "&amp;",
    "<": "&lt;",
    ">": "&gt;",
    '"': "&quot;",
    "'": "&#39;"
  }[c]));
}

render();
