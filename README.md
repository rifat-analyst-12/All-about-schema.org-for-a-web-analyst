# All-about-schema.org-for-a-web-analyst

একজন web analyst (GTM, GA4, Google Ads, conversion tracking, server-side tracking) হিসেবে schema.org কী, কীভাবে কাজ করে, আর কীভাবে এটাকে data collection-এর সবচেয়ে নির্ভরযোগ্য source বানানো যায়, তার একটা পূর্ণাঙ্গ, beginner-friendly guide।

> এই guide-টা ধরে নেয় তুমি HTML আর একটু JavaScript চেনো, কিন্তু schema.org বা structured data নিয়ে আগে কাজ করোনি।

---

## সূচিপত্র

1. [schema.org আসলে কী](#১-schemaorg-আসলে-কী)
2. [structured data কেন লাগে](#২-structured-data-কেন-লাগে)
3. [structured data কত রকম ফরম্যাটে থাকে](#৩-structured-data-কত-রকম-ফরম্যাটে-থাকে)
4. [Microdata কীভাবে পড়তে হয়](#৪-microdata-কীভাবে-পড়তে-হয়)
5. [JSON-LD কীভাবে পড়তে হয়](#৫-json-ld-কীভাবে-পড়তে-হয়)
6. [একজন web analyst এটাকে কীভাবে কাজে লাগায়](#৬-একজন-web-analyst-এটাকে-কীভাবে-কাজে-লাগায়)
7. [বাস্তব উদাহরণ: view_item data layer](#৭-বাস্তব-উদাহরণ-view_item-data-layer)
8. [GTM-এ কীভাবে ব্যবহার করবে](#৮-gtm-এ-কীভাবে-ব্যবহার-করবে)
9. [server-side tracking-এ এর ভূমিকা](#৯-server-side-tracking-এ-এর-ভূমিকা)
10. [সাধারণ ভুল আর সতর্কতা](#১০-সাধারণ-ভুল-আর-সতর্কতা)
11. [দরকারি টুল আর রেফারেন্স](#১১-দরকারি-টুল-আর-রেফারেন্স)

---

## ১. schema.org আসলে কী

schema.org একটা যৌথ project, Google, Microsoft (Bing), Yahoo, Yandex মিলে ২০১১ সালে চালু করে। এরা মিলে একটা সাধারণ **শব্দভাণ্ডার (vocabulary)** ঠিক করেছে, যেখানে বলা আছে একটা জিনিসের কী কী property থাকতে পারে।

উদাহরণ হিসেবে একটা Product-এর property হতে পারে name, brand, price, priceCurrency, availability, sku ইত্যাদি। একটা Article-এর property হতে পারে headline, author, datePublished। এরকম কয়েকশো type আর হাজারো property আগে থেকেই সংজ্ঞায়িত করা আছে।

মূল কথা: **schema.org কোনো software না, কোনো service না। এটা শুধু সবাই মিলে ঠিক করা একটা মানদণ্ড (standard) বা dictionary।** এটা বলে দেয়, "যদি তুমি জানাতে চাও এটা একটা product-এর দাম, তাহলে নিজের ইচ্ছামতো নাম না দিয়ে `price` শব্দটা ব্যবহার করো, তাহলে পুরো পৃথিবী একই জিনিস বুঝবে।"

---

## ২. structured data কেন লাগে

একটা সাধারণ HTML page মানুষের চোখে সুন্দর, কিন্তু machine-এর কাছে অর্থহীন। browser জানে না `$40.00` লেখাটা একটা দাম, নাকি শুধু text। structured data ঠিক এই সমস্যাটা সমাধান করে। এটা page-এর ভেতরে লুকানো একটা "অর্থের স্তর" যোগ করে, যা machine সহজে পড়তে পারে।

সহজ তুলনা:

| চোখে যা দেখায় | machine যা বোঝে (structured data ছাড়া) | machine যা বোঝে (structured data সহ) |
| --- | --- | --- |
| Yeti Colster 250ml | কিছু random text | এটা একটা product-এর `name` |
| $40.00 | কিছু সংখ্যা আর চিহ্ন | এটা `price` = 40, `priceCurrency` = AUD |
| In Stock | দুইটা শব্দ | এটা `availability` = in_stock |

এই "অর্থের স্তর" যোগ করার কাজটাই করে schema.org-এর vocabulary।

---

## ৩. structured data কত রকম ফরম্যাটে থাকে

schema.org-এর vocabulary একই, কিন্তু সেটা page-এ বসানোর তিনটা আলাদা উপায় আছে। একজন analyst হিসেবে তিনটাই চেনা জরুরি, কারণ site ভেদে যেকোনোটা পাবে।

### ক. Microdata

HTML tag-এর ভেতরেই attribute হিসেবে বসানো থাকে। data আর display একসাথে মিশে থাকে।

```html
<div itemscope itemtype="https://schema.org/Product">
  <span itemprop="name">Yeti Colster 250ml Slim Black</span>
  <span itemprop="price" content="40">$40.00</span>
</div>
```

### খ. RDFa

Microdata-র মতোই, তবে attribute-এর নাম আলাদা (`typeof`, `property`)। এখন তুলনামূলক কম দেখা যায়।

```html
<div vocab="https://schema.org/" typeof="Product">
  <span property="name">Yeti Colster 250ml Slim Black</span>
  <span property="price" content="40">$40.00</span>
</div>
```

### গ. JSON-LD

একটা আলাদা `<script>` block-এর ভেতরে পুরো data JSON আকারে বসানো থাকে, HTML-এর সাথে মেশানো থাকে না। Google এটাকেই সবচেয়ে বেশি recommend করে, তাই আধুনিক site-এ এটাই বেশি দেখবে।

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Yeti Colster 250ml Slim Black",
  "brand": "Yeti",
  "sku": "888830226261",
  "offers": {
    "@type": "Offer",
    "price": "40",
    "priceCurrency": "AUD",
    "availability": "https://schema.org/InStock"
  }
}
</script>
```

> মনে রেখো: `itemtype` বা `@context`-এ যে `schema.org` URL দেখো, সেটা browser fetch করে না। ওটা শুধু একটা **unique নাম (namespace)**, যাতে পুরো পৃথিবী একই type বোঝায়। ঠিক একটা package name-এর মতো, যেটা download করার জিনিস না, শুধু আলাদা করে চেনার উপায়।

---

## ৪. Microdata কীভাবে পড়তে হয়

Microdata-তে দুইটা মূল attribute:

- `itemtype` বলে item-টা কোন type (Product, Article, Person...)।
- `itemprop` বলে ভেতরের প্রতিটা তথ্য কোন property।

data তোলার সাধারণ নিয়ম: আগে wrapper (`itemtype`) ধরো, তারপর তার ভেতর থেকে scoped `querySelector` দিয়ে প্রতিটা `itemprop` তোলো। scoped রাখলে নিশ্চিত থাকা যায় যে তুমি ঠিক এই item-এরই data নিচ্ছ, page-এর অন্য কোথাও থেকে না।

```javascript
// আগে product wrapper টা ধরি
var productEl = document.querySelector('[itemtype="https://schema.org/Product"]');

if (productEl) {
    var name;
    var price;
    var currency;

    // wrapper-এর ভেতর থেকে scoped ভাবে প্রতিটা property তুলি
    var nameEl = productEl.querySelector('[itemprop="name"]');
    var priceEl = productEl.querySelector('[itemprop="price"]');
    var currencyEl = productEl.querySelector('[itemprop="priceCurrency"]');

    if (nameEl) {
        name = nameEl.innerText.trim();
    }

    if (priceEl) {
        // অনেক সময় দাম content attribute-এ পরিষ্কার সংখ্যা হিসেবে থাকে
        price = priceEl.getAttribute('content');
    }

    if (currencyEl) {
        currency = currencyEl.getAttribute('content');
    }

    console.log(name, price, currency);
}
```

একটা গুরুত্বপূর্ণ সুবিধা: value গুলো প্রায়ই machine-readable form-এ আলাদা করে বসানো থাকে।

```html
<div itemprop="price" content="40">$40.00</div>
```

চোখে দেখায় `$40.00`, কিন্তু `content="40"`-এ পরিষ্কার সংখ্যা। তাই `getAttribute('content')` দিয়ে সরাসরি `40` পাওয়া যায়, `$` বা `,` clean করার ঝামেলা নেই।

---

## ৫. JSON-LD কীভাবে পড়তে হয়

JSON-LD পড়া আরও সহজ, কারণ পুরোটা একটা JSON object। script tag খুঁজে বের করে parse করলেই হলো।

```javascript
// পেজের সব JSON-LD block তুলি
var jsonLdEls = document.querySelectorAll('script[type="application/ld+json"]');
var product = null;

jsonLdEls.forEach(function (el) {
    var data;

    // parse ব্যর্থ হলে যেন পুরো code না ভাঙে
    try {
        data = JSON.parse(el.innerText);
    } catch (e) {
        data = null;
    }

    // অনেক সময় type "Product" হয়
    if (data) {
        if (data['@type'] === 'Product') {
            product = data;
        }
    }
});

if (product) {
    var name = product.name;
    var brand = product.brand;
    var price;
    var currency;

    // offers আলাদা object হিসেবে থাকে
    if (product.offers) {
        price = product.offers.price;
        currency = product.offers.priceCurrency;
    }

    console.log(name, brand, price, currency);
}
```

> সতর্কতা: JSON-LD-র গঠন site ভেদে আলাদা হতে পারে। কখনো একটা object, কখনো array, কখনো `@graph`-এর ভেতরে সব nested থাকে। তাই প্রতিটা site-এ আগে console-এ structure দেখে নিয়ে তারপর path ঠিক করতে হবে।

---

## ৬. একজন web analyst এটাকে কীভাবে কাজে লাগায়

এটাই এই guide-এর সবচেয়ে গুরুত্বপূর্ণ অংশ। schema.org বসানো হয়েছিল মূলত search engine-এর জন্য, কিন্তু একজন analyst-এর হাতে এটা একটা তৈরি, পরিষ্কার, নির্ভরযোগ্য data source।

### কেন এটা tracking-এর জন্য এত ভালো

**এক, এটা টেকসই (durable)।** CSS class (`.productprice`, `.product-title`) যেকোনো দিন theme update-এ বদলে যেতে পারে, তখন class-নির্ভর selector ভেঙে যায়। কিন্তু `itemprop="price"` তুলে দিলে site-এর Google ranking-এর ক্ষতি হয়, তাই সহজে কেউ ওটায় হাত দেয় না। মানে schema-নির্ভর selector অনেক বেশি স্থিতিশীল।

**দুই, data পরিষ্কার (clean)।** দাম, currency, availability আগে থেকেই machine-readable form-এ আলাদা করা থাকে, তাই text parse বা regex-এর ঝামেলা কমে যায়।

**তিন, এটা সব page-এ একরকম।** ভালোভাবে বানানো site-এ list page আর product page দুই জায়গাতেই একই `Product` schema পাওয়া যায়, তাই তোমার code consistent রাখা সহজ হয়।

### কোন কোন কাজে লাগে

- **GA4 ecommerce data layer বানানো:** `view_item`, `view_item_list`, `add_to_cart`, `purchase` ইত্যাদি event-এর জন্য item_id, item_name, item_brand, price, currency schema থেকে তোলা।
- **Google Ads conversion tracking:** dynamic remarketing-এ product ID আর value schema থেকে নেওয়া।
- **data layer না থাকলে fallback:** অনেক custom site-এ কোনো data layer থাকে না। schema তখন একটা reliable backup source হয়ে যায়।
- **QA আর validation:** page-এ ঠিক price/currency আছে কিনা, tracking বসানোর আগে দ্রুত যাচাই করা।

---

## ৭. বাস্তব উদাহরণ: view_item data layer

নিচে একটা পূর্ণাঙ্গ উদাহরণ, যেখানে product page-এর schema.org microdata থেকে GA4-এর recommended `view_item` data layer বানানো হয়েছে। code টা ইচ্ছা করে verbose আর সহজপাঠ্য রাখা, যাতে নতুনরা এক লাইন এক লাইন করে বুঝতে পারে।

```javascript
// view_item (GA4 recommended schema)

(function () {
    // পুরো product-এর wrapper টা ধরি
    var productEl = document.querySelector('[itemtype="https://schema.org/Product"]');

    if (productEl) {
        var name;
        var price;
        var brand;
        var currency;
        var id;
        var value;

        // প্রতিটা তথ্যের element আলাদা করে ধরি
        var nameEl = productEl.querySelector('[itemprop="name"]');
        var priceEl = productEl.querySelector('[itemprop="price"]');
        var brandEl = productEl.querySelector('[itemprop="brand"]');
        var currencyEl = productEl.querySelector('[itemprop="priceCurrency"]');
        var idEl = productEl.querySelector('[itemprop="sku"]');

        if (nameEl) {
            name = nameEl.innerText.trim();
        }

        if (priceEl) {
            var priceStr = priceEl.getAttribute('content');
            price = parseFloat(priceStr);
            value = price;
        }

        if (brandEl) {
            brand = brandEl.getAttribute('content');
        }

        if (currencyEl) {
            currency = currencyEl.getAttribute('content');
        }

        if (idEl) {
            id = idEl.getAttribute('content');
        }

        // item object বানাই
        var item = {
            item_id: id,
            item_name: name,
            item_brand: brand,
            price: price,
            quantity: 1
        };

        var items = [];
        items.push(item);

        window.dataLayer = window.dataLayer || [];

        // আগের ecommerce object টা clear করি
        dataLayer.push({ ecommerce: null });

        // GA4 recommended structure: সব কিছু ecommerce object-এর ভেতরে
        dataLayer.push({
            event: 'view_item',
            ecommerce: {
                currency: currency,
                value: value,
                items: items
            }
        });
    }
})();
```

মূল দুইটা GA4 নিয়ম এখানে মানা হয়েছে:

- সব ecommerce data একটা `ecommerce` object-এর ভেতরে nested।
- push করার আগে `dataLayer.push({ ecommerce: null })` দিয়ে আগের data clear করা, যাতে এক push-এর item পরের push-এ leak না করে।

---

## ৮. GTM-এ কীভাবে ব্যবহার করবে

উপরের মতো একটা script কে GTM-এ বসানোর সাধারণ ধাপগুলো:

1. **Custom HTML Tag** বানাও, তার ভেতরে `<script>` দিয়ে data layer push code টা রাখো।
2. **Trigger** ঠিক করো, যেমন শুধু product page-এ fire করার জন্য একটা page-path বা DOM condition।
3. আলাদা করে একটা **GA4 Event Tag** বানাও (event name `view_item`), যার trigger হবে ওই `view_item` custom event।
4. GTM-এ **Data Layer Variable** বানাও, যেমন `ecommerce.items`, `ecommerce.value`, `ecommerce.currency`, তারপর GA4 tag-এ map করো।
5. **Preview mode**-এ চালিয়ে দেখো data layer-এ ঠিকমতো সব field আসছে কিনা।

> টিপ: schema সরাসরি DOM থেকে পড়ার সময় নিশ্চিত হও যে ওই element গুলো ততক্ষণে page-এ লোড হয়ে গেছে। প্রয়োজনে DOM Ready বা Window Loaded trigger ব্যবহার করো, page-load-এর একদম শুরুতে না।

---

## ৯. server-side tracking-এ এর ভূমিকা

server-side tracking (server-side GTM) সাধারণত browser-এর data layer থেকেই data পায়, তারপর সেটা server container-এ পাঠায়। তাই DOM বা schema থেকে data তোলার কাজটা তখনও client-side-এই হয়। schema এখানে সেই একই ভূমিকা রাখে: client-side-এ নির্ভরযোগ্যভাবে product data তুলে data layer-এ বসানো, যা পরে server container-এ যায়।

এছাড়া schema থেকে পাওয়া পরিষ্কার data (যেমন sku, price) server-side থেকে Google Ads বা Meta CAPI-তে পাঠানোর সময় কাজে লাগে, কারণ conversion-এর সাথে সঠিক product ID আর value যুক্ত করা যায়।

> মনে রেখো: server-side container নিজে থেকে কোনো page-এর DOM বা schema পড়তে পারে না। ওটা শুধু তাই পায় যা client থেকে পাঠানো হয়। তাই schema থেকে data তোলার দায়িত্ব সবসময় client-side-এই থাকে।

---

## ১০. সাধারণ ভুল আর সতর্কতা

- **সব site-এ schema আছে ধরে নিও না।** আগে page inspect করে দেখো Microdata আছে, নাকি JSON-LD, নাকি কিছুই নেই।
- **selector-এ value হুবহু মেলাতে হবে।** HTML-এ `https://schema.org/Product` থাকলে selector-এও ঠিক তাই, `http` আর `https` এক না।
- **schema সবসময় ১০০% নির্ভুল না।** কিছু site-এ ভুল বা পুরনো price schema-তে থাকতে পারে, তাই মাঝে মাঝে cross-check করো।
- **display value আর content value আলাদা হতে পারে।** সবসময় আগে দেখো `content` attribute আছে কিনা, থাকলে সেটাই নাও।
- **JSON-LD parse fail হতে পারে।** সবসময় `try/catch`-এর ভেতরে parse করো।
- **timing।** schema যদি JavaScript দিয়ে পরে লোড হয়, তাহলে page-load-এর একদম শুরুতে পড়তে গেলে খালি পাবে। DOM Ready-র পরে পড়ো।

---

## ১১. দরকারি টুল আর রেফারেন্স

- schema.org টাইপ আর property খোঁজার জন্য: https://schema.org
- Google Rich Results Test: https://search.google.com/test/rich-results
- Schema Markup Validator: https://validator.schema.org
- Google structured data ডকুমেন্টেশন: https://developers.google.com/search/docs/appearance/structured-data
- browser-এ দ্রুত দেখার জন্য DevTools Console-এ `document.querySelectorAll('[itemtype]')` চালিয়ে দেখো page-এ কোন কোন schema আছে।

---

### সংক্ষেপে

schema.org নিজে কোনো website বা tool না, বরং সবাই মিলে ঠিক করা একটা নিয়মের বই। এর মূল উদ্দেশ্য search engine-কে page বোঝানো। কিন্তু একজন web analyst হিসেবে তুমি সেই তৈরি structured data টাকেই tracking-এর সবচেয়ে পরিষ্কার আর টেকসই source হিসেবে কাজে লাগাতে পারো, বিশেষ করে যখন কোনো ready-made data layer থাকে না।

---

> এই repo-টা শেখার জন্য বানানো। ভুল বা উন্নতির পরামর্শ থাকলে issue খুলতে পারো।
