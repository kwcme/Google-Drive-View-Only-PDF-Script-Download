# Google-Drive-View-Only-PDF-Script-Download-PDF下載
參考幾位大神的製作方法，發現有一些問題會導致無法完整下載pdf，經過更改以後會比較適合中文的下載方式

### 方法
1. 取得對方給的Google Drive連結
2. 確保在瀏覽器中的網址後方有"view"，如"https://drive.google.com/xxx/x/xxxxxxxxxxxxxxxxxxxx/view"
3. 接著將頁面划到最底短，確保頁面有被正確讀取完畢
4. 接著在瀏覽器頁面，按"F12"，到"主控台"("element")
5. 複製下面的代碼到"主控台"裡面，按下"Enter"就會自動跑。
   ```js
(function () {
  console.log("Loading script ...");

  let script = document.createElement("script");
  script.onload = function () {
    const { jsPDF } = window.jspdf;

    // 透過 "blob"方式擷取資料
    let pdf = null;
    let imgElements = document.getElementsByTagName("img");
    let validImgs = [];
    let initPDF = true;

    console.log("Scanning content ...");
    for (let i = 0; i < imgElements.length; i++) {
      let img = imgElements[i];

      let checkURLString = "blob:https://drive.google.com/";
      if (img.src.substring(0, checkURLString.length) !== checkURLString) {
        continue;
      }

      validImgs.push(img);
    }

    console.log(`${validImgs.length} content found!`);
    console.log("Generating PDF file ...");
    for (let i = 0; i < validImgs.length; i++) {
      let img = validImgs[i];
      let canvasElement = document.createElement("canvas");
      let con = canvasElement.getContext("2d");
      canvasElement.width = img.naturalWidth;
      canvasElement.height = img.naturalHeight;
      con.drawImage(img, 0, 0, img.naturalWidth, img.naturalHeight);
      let imgData = canvasElement.toDataURL();

      let orientation = img.naturalWidth > img.naturalHeight ? "l" : "p";
      let pageWidth = img.naturalWidth;
      let pageHeight = img.naturalHeight;

      if (initPDF) {
        pdf = new jsPDF({
          orientation: orientation,
          unit: "px",
          format: [pageWidth, pageHeight],
        });
        initPDF = false;
      }

      pdf.addImage(imgData, "PNG", 0, 0, pageWidth, pageHeight, "", "SLOW");
      if (i !== validImgs.length - 1) {
        pdf.addPage();
      }

      const percentages = Math.floor(((i + 1) / validImgs.length) * 100);
      console.log(`Processing content ${percentages}%`);
    }

    // 調整檔名問題
    let title = document.querySelector('meta[itemprop="name"]').content;
    if (title.split(".").pop() !== "pdf") {
      title += ".pdf";
    }

    // 下載產出的PDF
    console.log("Downloading PDF file ...");
    pdf.save(title, { returnPromise: true }).then(() => {
      document.body.removeChild(script);
      console.log("PDF downloaded!");
    });
  };

  // Load the jsPDF library using the trusted URL
  let scriptURL = "https://unpkg.com/jspdf@latest/dist/jspdf.umd.min.js";
  let trustedURL;
  if (window.trustedTypes && trustedTypes.createPolicy) {
    const policy = trustedTypes.createPolicy("myPolicy", {
      createScriptURL: (input) => {
        return input;
      },
    });
    trustedURL = policy.createScriptURL(scriptURL);
  } else {
    trustedURL = scriptURL;
  }

  script.src = trustedURL;
  document.body.appendChild(script);
})();
   ```
6. 下載下來的pdf檔案，會是以圖片的形式呈現，並非是向量與像素的複合型態；但版面會比較符合原始檔案的樣子。

### 參考資源
- [zavierferodova/Google-Drive-View-Only-PDF-Script-Downloader](https://github.com/zavierferodova/Google-Drive-View-Only-PDF-Script-Downloader)
