<!DOCTYPE html>
<html lang="ro">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Ospătar - Scanare Comandă</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 20px;
      background-color: #111827;
      color: #fff;
    }
    #reader {
      width: 300px;
      margin: 0 auto;
    }
    #orderDetails {
      margin-top: 20px;
      text-align: left;
      display: none;
    }
    .order-item {
      margin-bottom: 10px;
    }
    .table-details {
      margin-top: 20px;
      display: none;
      text-align: left;
    }
    .table-order {
      margin-bottom: 1rem;
      border: 1px solid #e78c00;
      padding: 0.5rem;
    }
  </style>
</head>
<body>
  <h1>Scanare Cod QR Comandă</h1>
  <div id="reader"></div>

  <!-- Afișare detalii unei singure comenzi (docId) -->
  <div id="orderDetails">
    <h2>Detalii Comandă</h2>
    <p><strong>ID Firebase:</strong> <span id="orderFirebaseId"></span></p>
    <p><strong>Masă:</strong> <span id="tableId"></span></p>
    <p><strong>Client #:</strong> <span id="clientId"></span></p>
    <h3>Produse:</h3>
    <ul id="itemsList"></ul>
    <p><strong>Total:</strong> <span id="totalPrice"></span> Lei</p>
  </div>

  <!-- Afișare detalii pt o masă întreagă (toate comenzile cu tableId) -->
  <div class="table-details" id="tableDetails">
    <h2>Comenzi pentru Masă: <span id="scannedTableId"></span></h2>
    <div id="tableOrders"></div>
    <h3>Total masă: <span id="tableTotal"></span> Lei</h3>
  </div>

  <!-- html5-qrcode -->
  <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
  <!-- Firebase -->
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-app.js";
    import { getFirestore, doc, getDoc, collection, query, where, getDocs } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-firestore.js";

    const firebaseConfig = {
      apiKey: "AIzaSyBahkhy2GgUW2wM5dxghoVt2bv0-6ZyWqQ",
      authDomain: "restaurantapp-d0256.firebaseapp.com",
      projectId: "restaurantapp-d0256",
      storageBucket: "restaurantapp-d0256.appspot.com",
      messagingSenderId: "275208346650",
      appId: "1:275208346650:web:f262b8ec49242d4b9945a3",
      measurementId: "G-F44TB27G17"
    };

    const firebaseApp = initializeApp(firebaseConfig);
    const db = getFirestore(firebaseApp);

    // Citește o comandă pe baza docId
    async function fetchOrderByDocId(docId) {
      const docRef = doc(db, "orders", docId);
      const docSnap = await getDoc(docRef);
      if (docSnap.exists()) {
        return { docId: docSnap.id, ...docSnap.data() };
      }
      return null;
    }

    // Citește toate comenzile pentru un tableId
    async function fetchOrdersForTable(tableId) {
      const q = query(collection(db, "orders"), where("tableId", "==", tableId));
      const querySnapshot = await getDocs(q);
      const orders = [];
      querySnapshot.forEach((doc) => {
        orders.push({ docId: doc.id, ...doc.data() });
      });
      return orders;
    }

    // Afișează o singură comandă
    function displaySingleOrder(order) {
      document.getElementById('orderFirebaseId').textContent = order.docId;
      document.getElementById('tableId').textContent = order.tableId || '(nespecificat)';
      document.getElementById('clientId').textContent = order.clientNumber || '0';

      let totalPrice = 0;
      const itemsList = document.getElementById('itemsList');
      itemsList.innerHTML = '';
      order.items.forEach(item => {
        const li = document.createElement('li');
        const subTotal = (item.price * item.quantity);
        totalPrice += subTotal;
        li.textContent = `${item.name} x ${item.quantity} – ${subTotal.toFixed(2)} Lei`;
        li.classList.add('order-item');
        itemsList.appendChild(li);
      });
      document.getElementById('totalPrice').textContent = totalPrice.toFixed(2);
      document.getElementById('orderDetails').style.display = 'block';
    }

    // Afișează toate comenzile pentru un tableId
    function displayAllOrdersForTable(tableId, orders) {
      const tableDetailsEl = document.getElementById('tableDetails');
      const scannedTableIdEl = document.getElementById('scannedTableId');
      const tableOrdersEl = document.getElementById('tableOrders');
      const tableTotalEl = document.getElementById('tableTotal');

      scannedTableIdEl.textContent = tableId;
      tableOrdersEl.innerHTML = '';

      let tableSum = 0;
      orders.forEach((ord, idx) => {
        let sumOrder = 0;
        ord.items.forEach(item => {
          sumOrder += item.price * item.quantity;
        });
        tableSum += sumOrder;

        const divOrd = document.createElement('div');
        divOrd.classList.add('table-order');
        let html = `
          <h4>Comanda #${ord.docId}</h4>
          <p>Client: ${ord.clientNumber || '-'} </p>
          <ul>
        `;
        ord.items.forEach(item => {
          html += `<li>${item.name} x ${item.quantity} – ${(item.price * item.quantity).toFixed(2)} Lei</li>`;
        });
        html += `</ul><p><strong>Total Comandă: ${sumOrder.toFixed(2)} Lei</strong></p>`;
        divOrd.innerHTML = html;
        tableOrdersEl.appendChild(divOrd);
      });

      tableTotalEl.textContent = tableSum.toFixed(2);
      tableDetailsEl.style.display = 'block';
    }

    // On scan success
    function onScanSuccess(decodedText, decodedResult) {
      // Exemplu: textul decodat e un link cu docId:  restaurant.com/confirm.html?docId=ABC123
      // 1) extragem param docId
      const url = new URL(decodedText);
      const docId = url.searchParams.get("docId");
      // De exemplu, dacă e "restaurant.com/orders?tableId=Masa16", extragem tableId
      const tableId = url.searchParams.get("tableId");

      console.log("QR decodat -> docId =", docId, " | tableId =", tableId);

      // Oprim scanner-ul
      html5QrcodeScanner.clear().then(_ => {
        // 2) Afișăm date
        if (docId) {
          // Afișăm datele unei comenzi
          fetchOrderByDocId(docId).then(order => {
            if (order) {
              displaySingleOrder(order);

              // Bonus: Dacă vrei să afișezi toate comenzile de la masă:
              if (order.tableId) {
                fetchOrdersForTable(order.tableId).then(allOrders => {
                  displayAllOrdersForTable(order.tableId, allOrders);
                });
              }
            } else {
              alert("Comandă inexistentă!");
            }
          });
        } else if (tableId) {
          // direct fetchOrdersForTable
          fetchOrdersForTable(tableId).then(orders => {
            if (orders.length === 0) {
              alert("Nu există comenzi pt masă: " + tableId);
            } else {
              displayAllOrdersForTable(tableId, orders);
            }
          });
        } else {
          alert("QR code invalid!");
        }
      }).catch(error => {
        console.error("Eroare la oprirea scanner-ului: ", error);
      });
    }

    // On scan failure
    function onScanFailure(error) {
      console.warn(`Scanarea a eșuat. Motiv: ${error}`);
    }

    // Configurare scanner
    let html5QrcodeScanner = new Html5QrcodeScanner("reader", { fps: 10, qrbox: 250 });
    html5QrcodeScanner.render(onScanSuccess, onScanFailure);
  </script>
</body>
</html>
