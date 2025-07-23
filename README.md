# Sss
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>حاسبة الأسعار</title>
  <style>
    body {
      background-color: #1e1e1e;
      color: #ffffff;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      padding: 20px;
    }
    label, select, input {
      display: block;
      margin: 10px 0;
    }
    select, input[type="number"], input[type="text"] {
      padding: 10px;
      width: 300px;
      border: none;
      border-radius: 5px;
    }
    button {
      padding: 10px 20px;
      background-color: #4CAF50;
      border: none;
      border-radius: 5px;
      color: white;
      cursor: pointer;
      margin-top: 15px;
    }
    .results {
      margin-top: 30px;
      border-top: 1px solid #ccc;
      padding-top: 20px;
    }
    .result-box {
      background-color: #333;
      border: 1px solid #555;
      padding: 15px;
      margin-bottom: 15px;
      border-radius: 8px;
    }
  </style>
</head>
<body>

  <h1>حاسبة أسعار المنتجات</h1>

  <label>النوع الرئيسي:</label>
  <select id="mainType" onchange="populateSubTypes()">
    <option value="">-- اختر --</option>
    <option value="نوافذ">نوافذ</option>
    <option value="أبواب">أبواب</option>
    <option value="أبواب السحب">أبواب السحب</option>
  </select>

  <label>النوع الفرعي:</label>
  <select id="subType"></select>

  <div id="extraOptions"></div>

  <label>المقاس (طول × عرض):</label>
  <input type="text" id="sizeInput" placeholder="مثال: 2.2×1">

  <label>الكمية:</label>
  <input type="number" id="quantityInput" value="1" min="1">

  <button onclick="calculate()">احسب</button>

  <div class="results" id="results"></div>

  <script>
    const prices = {
      // نوافذ
      "دبل جلاس دبل فريم - ثابتة": { price: 34, per: "m", factor: 0.13 },
      "دبل جلاس دبل فريم - حركة": { price: 73, per: "m", factor: 0.13 },
      "دبل جلاس دبل فريم - حركتين": { price: 92, per: "m", factor: 0.13 },

      "دبل جلاس سنجل فريم - ثابتة": { price: 26, per: "m", factor: 0.13 },
      "دبل جلاس سنجل فريم - حركة": { price: 46, per: "m", factor: 0.13 },
      "دبل جلاس سنجل فريم - حركتين": { price: 58, per: "m", factor: 0.13 },

      "سنجل جلاس سنجل فريم - ثابتة": { price: 20, per: "m", factor: 0.13 },
      "سنجل جلاس سنجل فريم - حركة": { price: 43, per: "m", factor: 0.13 },
      "سنجل جلاس سنجل فريم - حركتين": { price: 47, per: "m", factor: 0.13 },

      "نوافذ السلايدنج": { price: 10, per: "m", factor: 0.13, isSliding: true },
      "النوافذ الكهربائية": { price: 102, per: "m", factor: 0.13 },
      "سكاي لايت - بدون مكينة": { price: 56, per: "m", factor: 0.13 },
      "سكاي لايت - مع مكينة": { price: 145, per: "m", factor: 0.13 },
      "كارتن وول - ثقيل": { price: 56, per: "m", factor: 0.13 },
      "كارتن وول - خفيف": { price: 45, per: "m", factor: 0.13 },

      // أبواب
      "باب المدخل - زينك": { price: 66, per: "m", factor: 0.2, add: 10 },
      "باب المدخل - ستينلس ستيل": { price: 120, per: "m", factor: 0.2, add: 10 },
      "باب المدخل - كاست المنيوم": { price: 168, per: "m", factor: 0.2, add: 10 },

      "باب WPC - فارغ": { price: 45, per: "unit", factor: 0.11 },
      "باب WPC - مع خشب": { price: 50, per: "unit", factor: 0.11 },
      "باب WPC - ضد الصوت": { price: 60, per: "unit", factor: 0.11 },
      "باب WPC - فريم المنيوم": { price: 67, per: "unit", factor: 0.11 },

      "باب ألمنيوم - فارغ": { price: 65, per: "unit", factor: 0.11 },
      "باب ألمنيوم - مع خشب": { price: 75, per: "unit", factor: 0.11 },
      "باب ألمنيوم - فل": { price: 85, per: "unit", factor: 0.11 },
      "باب ألمنيوم - مخفي": { price: 110, per: "unit", factor: 0.11 },
      "باب ألمنيوم - خارجي": { price: 61, per: "unit", factor: 0.11 },

      "باب دورة مياه - جديد": { price: 55, per: "unit", factor: 0.11 },
      "باب دورة مياه - قديم": { price: 45, per: "unit", factor: 0.11 },
      "باب دورة مياه - مخفي زجاجي": { price: 65, per: "unit", factor: 0.11 },

      // أبواب السحب
      "سحب داخلي - زجاج": { price: 38, per: "m", factor: 0.13 },
      "سحب داخلي - متين": { price: 41, per: "m", factor: 0.13 },
      "سحب خارجي - جزء مفتوح": { price: 55, per: "m", factor: 0.13 },
      "سحب خارجي - جزئين": { price: 58, per: "m", factor: 0.13 },
      "سحب WPC": { price: 61, per: "m", factor: 0.11 }
    };

    const subTypeMap = {
      "نوافذ": [
        "دبل جلاس دبل فريم - ثابتة", "دبل جلاس دبل فريم - حركة", "دبل جلاس دبل فريم - حركتين",
        "دبل جلاس سنجل فريم - ثابتة", "دبل جلاس سنجل فريم - حركة", "دبل جلاس سنجل فريم - حركتين",
        "سنجل جلاس سنجل فريم - ثابتة", "سنجل جلاس سنجل فريم - حركة", "سنجل جلاس سنجل فريم - حركتين",
        "نوافذ السلايدنج", "النوافذ الكهربائية", "سكاي لايت - بدون مكينة", "سكاي لايت - مع مكينة",
        "كارتن وول - ثقيل", "كارتن وول - خفيف"
      ],
      "أبواب": [
        "باب المدخل - زينك", "باب المدخل - ستينلس ستيل", "باب المدخل - كاست المنيوم",
        "باب WPC - فارغ", "باب WPC - مع خشب", "باب WPC - ضد الصوت", "باب WPC - فريم المنيوم",
        "باب ألمنيوم - فارغ", "باب ألمنيوم - مع خشب", "باب ألمنيوم - فل", "باب ألمنيوم - مخفي", "باب ألمنيوم - خارجي",
        "باب دورة مياه - جديد", "باب دورة مياه - قديم", "باب دورة مياه - مخفي زجاجي"
      ],
      "أبواب السحب": [
        "سحب داخلي - زجاج", "سحب داخلي - متين", "سحب خارجي - جزء مفتوح", "سحب خارجي - جزئين", "سحب WPC"
      ]
    };

    function populateSubTypes() {
      const mainType = document.getElementById("mainType").value;
      const subTypeSelect = document.getElementById("subType");
      subTypeSelect.innerHTML = "";
      if (subTypeMap[mainType]) {
        subTypeMap[mainType].forEach(type => {
          const option = document.createElement("option");
          option.value = type;
          option.text = type;
          subTypeSelect.appendChild(option);
        });
      }
    }

    function calculate() {
      const type = document.getElementById("subType").value;
      const size = document.getElementById("sizeInput").value;
      const quantity = parseInt(document.getElementById("quantityInput").value);
      const data = prices[type];
      if (!data || !quantity) return;

      let area = 1;
      if (data.per === "m") {
        const parts = size.split("×");
        if (parts.length !== 2) return alert("أدخل المقاس بشكل صحيح (مثال: 2.2×1)");
        area = parseFloat(parts[0]) * parseFloat(parts[1]);
        if (isNaN(area)) return alert("تحقق من المقاس");
      }

      let base = data.per === "m" ? area * data.price : data.price;
      if (data.isSliding) base += 10;
      if (data.add) base += data.add;

      const shipping = area * data.factor * 48;
      const total = (base + shipping) * quantity;

      const resultsDiv = document.getElementById("results");
      const resultBox = document.createElement("div");
      resultBox.className = "result-box";
      resultBox.innerHTML = `
        <strong>النوع:</strong> ${type}<br>
        <strong>المقاس:</strong> ${size || "—"}<br>
        <strong>الكمية:</strong> ${quantity}<br>
        <strong>سعر الشحن:</strong> ${shipping.toFixed(2)}<br>
        <strong>السعر الكلي:</strong> ${total.toFixed(2)} ريال
      `;
      resultsDiv.appendChild(resultBox);
    }
  </script>
</body>
</html>
