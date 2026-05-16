<style>
  .markdown-section {
    max-width: none !important;
    padding: 1.5rem 2rem !important;
  }
  #pdf-viewer {
    width: 100%;
  }
  #pdf-viewer canvas {
    display: block;
    width: 100% !important;
    height: auto !important;
    margin: 0 auto 16px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.12);
  }
  #pdf-viewer .pdf-loading,
  #pdf-viewer .pdf-error {
    text-align: center;
    color: #666;
    padding: 2rem;
  }
</style>

<div id="pdf-viewer"><p class="pdf-loading">正在加载 PDF…</p></div>

<script>
(function () {
  var PDF_URL = 'files/PLS-Template-Xinyue.pdf';
  var PDFJS = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js';
  var WORKER = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.worker.min.js';

  var container = document.getElementById('pdf-viewer');
  if (!container) return;

  function showError(msg) {
    container.innerHTML = '<p class="pdf-error">' + msg + '</p>';
  }

  function loadScript(src) {
    return new Promise(function (resolve, reject) {
      var s = document.createElement('script');
      s.src = src;
      s.onload = resolve;
      s.onerror = function () { reject(new Error('无法加载 ' + src)); };
      document.head.appendChild(s);
    });
  }

  function renderPage(page, index) {
    var baseViewport = page.getViewport({ scale: 1 });
    var cssWidth = container.clientWidth || window.innerWidth - 64;
    var scale = cssWidth / baseViewport.width;
    var dpr = window.devicePixelRatio || 1;
    var viewport = page.getViewport({ scale: scale * dpr });

    var canvas = document.createElement('canvas');
    canvas.setAttribute('data-page', index);
    var ctx = canvas.getContext('2d');
    canvas.width = viewport.width;
    canvas.height = viewport.height;
    canvas.style.width = cssWidth + 'px';
    canvas.style.height = (viewport.height / dpr) + 'px';

    container.appendChild(canvas);
    return page.render({ canvasContext: ctx, viewport: viewport }).promise;
  }

  loadScript(PDFJS)
    .then(function () {
      window.pdfjsLib.GlobalWorkerOptions.workerSrc = WORKER;
      return window.pdfjsLib.getDocument(PDF_URL).promise;
    })
    .then(function (pdf) {
      container.innerHTML = '';
      var chain = Promise.resolve();
      for (var i = 1; i <= pdf.numPages; i++) {
        (function (pageNum) {
          chain = chain.then(function () {
            return pdf.getPage(pageNum).then(function (page) {
              return renderPage(page, pageNum);
            });
          });
        })(i);
      }
      return chain;
    })
    .catch(function (err) {
      showError('PDF 加载失败：' + (err.message || err));
    });

  var resizeTimer;
  window.addEventListener('resize', function () {
    clearTimeout(resizeTimer);
    resizeTimer = setTimeout(function () { location.reload(); }, 300);
  });
})();
</script>
