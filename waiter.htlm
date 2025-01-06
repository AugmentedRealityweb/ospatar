<!DOCTYPE html>
<html lang="ro">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Ospătar - Scanare Cod QR</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      background-color: #111827;
      color: #fff;
      padding: 20px;
    }
    #reader {
      width: 300px;
      margin: 20px auto;
    }
    #orderDetails, .table-details {
      margin-top: 20px;
      text-align: left;
      display: none;
    }
    .order-item {
      margin-bottom: 10px;
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
  <p>Folosește camera pentru a scana un cod QR care conține informații despre comandă.</p>
  
  <div id="reader"></div>

  <!-- Afișare detalii comandă -->
  <div id="orderDetails">
    <h2>Detalii Comandă</h2>
    <p><strong>ID Firebase:</strong> <span id="orderFirebaseId"></span></p>
    <p><strong>Masă:</strong> <span id="tableId"></span></p>
    <p><strong>Client #:</strong> <span id="clientId"></span></p>
    <h3>Produse:</h3>
    <ul id="itemsList"></ul>
    <p><strong>Total:</strong> <span id="totalPrice"></span> Lei</p>
  </div>

  <div class="table-details" id="tableDetails">
    <h2>Comenzi pentru Masă: <span id="scannedTableId"></span></h2>
    <div id="tableOrders"></div>
    <h3>Total masă: <span id="tableTotal"></span> Lei</h3>
  </div>

  <!-- Biblioteca html5-qrcode -->
  <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>

  <script>
    // Configurare QR scanner
    const html5QrcodeScanner = new Html5QrcodeScanner(
      "reader",
      { fps: 10, qrbox: 250 }
    );

    // On success
    function onScanSuccess(decodedText, decodedResult) {
      console.log(`Cod QR detectat: ${decodedText}`);
      let urlObj;

      try {
        urlObj = new URL(decodedText);
      } catch (e) {
        alert("QR code invalid! Conținutul nu este un URL valid.");
        return;
      }

      const docId = urlObj.searchParams.get("docId");
      const tableId = urlObj.searchParams.get("tableId");

      console.log("Decodat:", { docId, tableId });

      if (docId) {
        alert(`Comandă detectată cu docId: ${docId}`);
        // Aici poți adăuga funcția pentru a căuta comenzile.
      } else if (tableId) {
        alert(`Masa detectată: ${tableId}`);
        // Aici poți afișa comenzile pentru întreaga masă.
      } else {
        alert("QR code nu conține parametrii docId sau tableId.");
      }
    }

    // On failure
    function onScanFailure(error) {
      console.warn(`Scanarea a eșuat: ${error}`);
    }

    // Inițializare scanner
    html5QrcodeScanner.render(onScanSuccess, onScanFailure);
  </script>
</body>
</html>
