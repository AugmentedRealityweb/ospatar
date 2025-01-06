<!DOCTYPE html>
<html>
<head>
  <script src="https://unpkg.com/html5-qrcode/minified/html5-qrcode.min.js"></script>
</head>
<body>
  <div id="reader" style="width: 300px; height: 300px;"></div>
  <script>
    const onScanSuccess = (decodedText, decodedResult) => {
      alert(`Cod QR scanat: ${decodedText}`);
    };

    const onScanFailure = (error) => {
      console.warn(`Scanare eșuată: ${error}`);
    };

    const scanner = new Html5QrcodeScanner("reader", { fps: 10, qrbox: 250 });
    scanner.render(onScanSuccess, onScanFailure);
  </script>
</body>
</html>
