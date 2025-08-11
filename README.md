# Varnish Integration for Bagisto

This package integrates **Varnish Cache** with Bagisto to boost site performance by delivering cached pages quickly, while still supporting **dynamic components** through **ESI (Edge Side Includes)** or **AJAX-based dynamic views**.

---

## 📌 Features

* ⚡ **Full-page caching** via Varnish
* 🔄 **Automatic cache purging** on product, category, or content updates
* 🔍 **ESI support** for dynamic blocks
* 🖱 **AJAX-based dynamic view loading** for improved Core Web Vitals
* 🛠 **Artisan commands** for cache management
* 🖥 **Admin tools** for purging cache & exporting VCL
* 🎨 **Theme-ready Blade components**
* 🛡 **Middleware for cache headers** (cacheable & non-cacheable routes)

---

## 🔄 Request Flow

![Request Flow Diagram](9f5170cc-b278-4eba-98a4-c63dc0b79c2c.png)

**Explanation**:

* **443 (HTTPS)** → **Nginx** handles SSL termination and forwards traffic.
* **80 (HTTP)** → **Varnish Proxy** caches and serves pages, or passes the request to the backend.
* **8080** → **Bagisto** (Laravel app) generates fresh content when needed.

**Flow Summary**:
Browser → Nginx → Varnish Proxy → Bagisto → Response (cached or fresh)

---

## 📦 Installation

### 1. Install via Composer

```bash
composer require bagisto/bagisto-varnish
```

### 2. Register the Service Provider

In `config/app.php`:

```php
'providers' => [
    Webkul\Varnish\Providers\VarnishServiceProvider::class,
],
```

### 3. Publish Assets & Config

```bash
php artisan vendor:publish --provider="Webkul\Varnish\Providers\VarnishServiceProvider"
```

### 4. Configure `config/varnish.php`

```php
return [
    'aliases' => [
        'Varnish' => \Webkul\Varnish\Facades\VarnishCache::class,
    ],
];
```

---

## ⚙️ Varnish Server Configuration

1. Install **Varnish 6.x** on your server.

2. Replace `/etc/varnish/default.vcl` with the provided file:

   ```
   Varnish/vcls/6.0.vcl
   ```

3. Restart Varnish:

   ```bash
   sudo systemctl restart varnish
   ```

---

## 🎨 Theme Integration

You can integrate dynamic content in **two ways**:

---

### **1 – Define Dynamic Views / Fragments**

In `config/esi_views.php`, define a **key** (identifier) and its corresponding **Blade view path**:

```php
return [
    'customer-desktop-dropdown' => 'varnish::shop.components.layouts.header.desktop.customer-dropdown',
];
```

* **Key** → Used in ESI or AJAX call (`customer-desktop-dropdown`)
* **Path** → Full Blade view path to render (`varnish::...`)

---

### **2 – ESI Include**

```blade
<esi:include src="/esi?tag=customer-desktop-dropdown" />
```

* Injects content at the **Varnish level** (server-side).
* Appears immediately on page load.
* May affect LCP/FCP if the backend is slow.

---

### **3 – AJAX Dynamic View (Recommended for LCP)**

```blade
<x-varnish::dynamic-view view="customer-desktop-dropdown" />
```

* Loads via AJAX **after user interaction**.
* Improves LCP/FCP.
* Ideal for non-critical dropdowns, modals, and menus.

---

## 🗂 Cache-Control Headers

For **routes that should NOT be cached** by Varnish:

```
Cache-Control: no-cache, no-store, must-revalidate
```

For **routes that should be cached**:

```
Cache-Control: public, max-age=604800, s-maxage=604800
```

*(Example: 7 days)*

---

### Middleware for Cache Headers in Bagisto

We’ve created a middleware `Webkul\Varnish\Http\Middleware\VarnishCache` to handle cache headers.

Attach it to routes like this:

```php
Route::get('/', [HomeController::class, 'index'])
    ->name('shop.home.index')
    ->middleware('cache.response');
```

---

## 🛠 UI Configuration (Export VCL)

Navigate to: **Admin → Configuration → Full Page Cache → Configuration**

Select **Varnish** as the cache application, then provide the following:

1. **Access List** – IPs allowed to purge the cache (e.g., `localhost`).
2. **Varnish Host URL** – Varnish server IP and port for purging/banning cache via UI.
3. **Backend Host URL** – Laravel Bagisto server IP used in the exported VCL.
4. **Backend Host Port** – Laravel Bagisto server port used in the exported VCL.
5. **Grace Period** – Duration for serving stale content if the backend is slow or unavailable.

---

## 🛠 Cache Management

Navigate to: **Admin → Configuration → Full Page Cache → Cache Management**

1. **Purge by URLs** – Enter full URLs (comma-separated) to clear specific cache entries. Paths and domains must match exactly.
2. **Purge Everything** – Clears **all** cache entries from Varnish. Use with caution, as it may temporarily affect performance.

---

## 🛡 Automatic Cache Purging

The package automatically purges cache when:

* Products, categories, pages, orders, reviews, refunds, or theme changes occur.

You can also manually trigger purging by adding your own events in `EventServiceProvider` and calling:

```php
VarnishCache::forget($urls);
```

---

## 🖥 Admin Panel Tools

* **Purge Full Cache**
* **Purge by URL**
* **Export VCL**

---

## 🚀 Best Practices

* Use **ESI** for small, critical personalized blocks (e.g., login status, cart count).
* Use **AJAX dynamic views** for non-critical interactive elements to improve Core Web Vitals.
* Set **Cache-Control headers** with appropriate TTL values to control caching behavior.
* Always test on a staging environment before deploying to production.
* Monitor **LCP**, **FCP**, and **TTFB** after enabling Varnish.
 