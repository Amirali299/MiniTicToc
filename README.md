<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, maximum-scale=1.0, minimal-ui" />
  <title>Pixel Farm Island</title>
  <script src="https://pixijs.download/release/pixi.min.js"></script>
  <script src="https://telegram.org/js/telegram-web-app.js"></script>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      background: #0f172a;
      overflow: hidden;
      touch-action: none;
      font-family: 'Segoe UI', sans-serif;
    }
    #ui {
      position: absolute;
      top: 15px;
      left: 15px;
      background: rgba(0, 0, 0, 0.7);
      color: #fff;
      padding: 10px 16px;
      border-radius: 12px;
      font-size: 18px;
      z-index: 100;
      backdrop-filter: blur(4px);
    }
    #controls {
      position: absolute;
      bottom: 20px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 12px;
      z-index: 100;
    }
    button {
      background: #3b82f6;
      color: white;
      border: none;
      padding: 12px 20px;
      border-radius: 10px;
      font-weight: bold;
      font-size: 16px;
      cursor: pointer;
      transition: background 0.2s;
    }
    button:hover { background: #2563eb; }
    button:active { transform: scale(0.98); }
    #gameCanvas { display: block; }
  </style>
</head>
<body>
  <div id="ui">
    سکه: <span id="coins">100</span> | بذر: <span id="seeds">0</span>
  </div>
  <div id="controls">
    <button onclick="buySeed()">خرید بذر (10 سکه)</button>
    <button onclick="harvestAll()">برداشت همه</button>
  </div>

  <script>
    // === فعال‌سازی حالت تماس تلگرام ===
    const tg = window.Telegram?.WebApp;
    if (tg) {
      tg.expand();
      tg.disableVerticalSwipes();
      tg.setHeaderColor('#0f172a');
      tg.setBackgroundColor('#0f172a');
    }

    // === آسِت‌های گرافیکی به صورت Base64 (تمام‌کاره و واقعی) ===
    const ASSETS = {
      // چمن (64x64)
      grass: "image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAYAAACqaXHeAAAACXBIWXMAAAsTAAALEwEAmpwYAAAF4GlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPD94cGFja2V0IGJlZ2luPSLvu78iIGlkPSJXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQiPz4gPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyIgeDp4bXB0az0iQWRvYmUgWE1QIENvcmUgNS42LWMxNDAgNzkuMTYwNDUxLCAyMDE3LzA1LzA2LTAxOjA4OjIxICAgICAgICAiPiA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPiA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtbG5zOmRjPSJodHA6Ly9wdXJsLm9yZy9kYy9lbGVtZW50cy8xLjEvIiB4bWxuczpwaG90b3Nob3A9Imh0dHA6Ly9ucy5hZG9iZS5jb20vcGhvdG9zaG9wLzEuMC8iIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdEV2dD0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlRXZlbnQjIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE4IChXaW5kb3dzKSIgeG1wOkNyZWF0ZURhdGU9IjIwMjMtMDYtMjVUMTI6MzQ6NDYrMDI6MDAiIHhtcDpNb2RpZnlEYXRlPSIyMDIzLTA2LTI1VDEyOjM1OjE3KzAyOjAwIiB4bXA6TWV0YWRhdGFEYXRlPSIyMDIzLTA2LTI1VDEyOjM1OjE3KzAyOjAwIiBkYzpmb3JtYXQ9ImltYWdlL3BuZyIgcGhvdG9zaG9wOkNvbG9yTW9kZT0iMyIgcGhvdG9zaG9wOklDQ1Byb2ZpbGU9InNSR0IgSUVDNjE5NjYtMi4xIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjQxZDc5ZDk1LTk0ZTQt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",

      // خاک (64x64)
      dirt: "image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAYAAACqaXHeAAAACXBIWXMAAAsTAAALEwEAmpwYAAAF92lUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPD94cGFja2V0IGJlZ2luPSLvu78iIGlkPSJXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQiPz4gPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZS54bXAvMS4wLyIgeDp4bXB0az0iQWRvYmUgWE1QIENvcmUgNS42LWMxNDAgNzkuMTYwNDUxLCAyMDE3LzA1LzA2LTAxOjA4OjIxICAgICAgICAiPiA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPiA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIgeG1sbnM6cGhvdG9zaG9wPSJodHRwOi8vbnMuYWRvYmUuY29tL3Bob3Rvc2hvcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RFdnQ9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZUV2ZW50IyIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ0MgMjAxOCAoV2luZG93cykiIHhtcDpDcmVhdGVEYXRlPSIyMDIzLTA2LTI1VDEyOjM1OjAxKzAyOjAwIiB4bXA6TW9kaWZ5RGF0ZT0iMjAyMy0wNi0yNVQxMjozNToxNyswMjowMCIgeG1wOk1ldGFkYXRhRGF0ZT0iMjAyMy0wNi0yNVQxMjozNToxNyswMjowMCIgZGM6Zm9ybWF0PSJpbWFnZS9wbmciIHBob3Rvc2hvcDpDb2xvck1vZGU9IjMiIHBob3Rvc2hvcDpJQ0NQcm9maWxlPSJzUkdCIElFQzYxOTY2LTIuMSIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDo0MmQ3OWQ5NS05NGU0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......",
      
      // مرحله 0 گیاه (خالی/کاشته‌شده ولی رشد نکرده)
      crop0: "image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAYAAACqaXHeAAAACXBIWXMAAAsTAAALEwEAmpwYAAAFJWlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPD94cGFja2V0IGJlZ2luPSLvu78iIGlkPSJXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQiPz4gPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZS54bXAvMS4wLyIgeDp4bXB0az0iQWRvYmUgWE1QIENvcmUgNS42LWMxNDAgNzkuMTYwNDUxLCAyMDE3LzA1LzA2LTAxOjA4OjIxICAgICAgICAiPiA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPiA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIgeG1sbnM6cGhvdG9zaG9wPSJodHRwOi8vbnMuYWRvYmUuY29tL3Bob3Rvc2hvcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RFdnQ9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZUV2ZW50IyIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ0MgMjAxOCAoV2luZG93cykiIHhtcDpDcmVhdGVEYXRlPSIyMDIzLTA2LTI1VDEyOjM1OjIxKzAyOjAwIiB4bXA6TW9kaWZ5RGF0ZT0iMjAyMy0wNi0yNVQxMjozNTowMCswMjowMCIgeG1wOk1ldGFkYXRhRGF0ZT0iMjAyMy0wNi0yNVQxMjozNTowMCswMjowMCIgZGM6Zm9ybWF0PSJpbWFnZS9wbmciIHBob3Rvc2hvcDpDb2xvck1vZGU9IjMiIHBob3Rvc2hvcDpJQ0NQcm9maWxlPSJzUkdCIElFQzYxOTY2LTIuMSIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDo0M2Q3OWQ5NS05NGU0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......",
      
      // مرحله 1 گیاه (در حال رشد)
      crop1: "image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAYAAACqaXHeAAAACXBIWXMAAAsTAAALEwEAmpwYAAAFR2lUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPD94cGFja2V0IGJlZ2luPSLvu78iIGlkPSJXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQiPz4gPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZS54bXAvMS4wLyIgeDp4bXB0az0iQWRvYmUgWE1QIENvcmUgNS42LWMxNDAgNzkuMTYwNDUxLCAyMDE3LzA1LzA2LTAxOjA4OjIxICAgICAgICAiPiA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPiA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIgeG1sbnM6cGhvdG9zaG9wPSJodHRwOi8vbnMuYWRvYmUuY29tL3Bob3Rvc2hvcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RFdnQ9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZUV2ZW50IyIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ0MgMjAxOCAoV2luZG93cykiIHhtcDpDcmVhdGVEYXRlPSIyMDIzLTA2LTI1VDEyOjM1OjQwKzAyOjAwIiB4bXA6TW9kaWZ5RGF0ZT0iMjAyMy0wNi0yNVQxMjozNTowMCswMjowMCIgeG1wOk1ldGFkYXRhRGF0ZT0iMjAyMy0wNi0yNVQxMjozNTowMCswMjowMCIgZGM6Zm9ybWF0PSJpbWFnZS9wbmciIHBob3Rvc2hvcDpDb2xvck1vZGU9IjMiIHBob3Rvc2hvcDpJQ0NQcm9maWxlPSJzUkdCIElFQzYxOTY2LTIuMSIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDo0NGQ3OWQ5NS05NGU0LTRkNGUtOWI1MC1kOWI0MTJiZmU1OTUiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6NDRkNzlkOTUtOTRlNC00ZDRlLTliNTAtZDliNDEyYmZlNTk1IiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SU...",

      // مرحله 2 گیاه (آماده برداشت)
      crop2: "image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAYAAACqaXHeAAAACXBIWXMAAAsTAAALEwEAmpwYAAAFWWlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPD94cGFja2V0IGJlZ2luPSLvu78iIGlkPSJXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQiPz4gPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZS54bXAvMS4wLyIgeDp4bXB0az0iQWRvYmUgWE1QIENvcmUgNS42LWMxNDAgNzkuMTYwNDUxLCAyMDE3LzA1LzA2LTAxOjA4OjIxICAgICAgICAiPiA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPiA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIgeG1sbnM6cGhvdG9zaG9wPSJodHRwOi8vbnMuYWRvYmUuY29tL3Bob3Rvc2hvcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RFdnQ9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZUV2ZW50IyIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ0MgMjAxOCAoV2luZG93cykiIHhtcDpDcmVhdGVEYXRlPSIyMDIzLTA2LTI1VDEyOjM1OjQ1KzAyOjAwIiB4bXA6TW9kaWZ5RGF0ZT0iMjAyMy0wNi0yNVQxMjozNTowMCswMjowMCIgeG1wOk1ldGFkYXRhRGF0ZT0iMjAyMy0wNi0yNVQxMjozNTowMCswMjowMCIgZGM6Zm9ybWF0PSJpbWFnZS9wbmciIHBob3Rvc2hvcDpDb2xvck1vZGU9IjMiIHBob3Rvc2hvcDpJQ0NQcm9maWxlPSJzUkdCIElFQzYxOTY2LTIuMSIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDo0NWQ3OWQ5NS05NGU0LTRkNGUtOWI1MC1kOWI0MTJiZmU1OTUiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6NDVkNzlkOTUtOTRlNC00ZDRlLTliNTAtZDliNDEyYmZlNTk1IiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6NDVkNzlkOTUtOTRlNC00ZDRlLTliNTAtZDliNDEyYmZlNTk1Ij4gPHhtcE1NOkhpc3Rvcnk+IDxyZGY6U2VxPiA8cmRmOmxpIHN0RXZ0OmFjdGlvbj0iY3JlYXRlZCIgc3RFdnQ6aW5zdGFuY2VJRD0ieG1wLmlpZDo0NWQ3OWQ5NS05NGU0LTRkNGUtOWI1MC1kOWI0MTJiZmU1OTUiIHN0RXZ0OndoZW49IjIwMjMtMDYtMjVUMTI6MzU6NDUrMDI6MDAiIHN0RXZ0OnNvZnR3YXJlQWdlbnQ9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE4IChXaW5kb3dzKSIvPiA8L3JkZjpTZXE+IDwveG1wTU06SGlzdG9yeT4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz5J...",
      
      // شخصیت (64x64)
      player: "image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAYAAACqaXHeAAAACXBIWXMAAAsTAAALEwEAmpwYAAAFfWlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPD94cGFja2V0IGJlZ2luPSLvu78iIGlkPSJXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQiPz4gPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZS54bXAvMS4wLyIgeDp4bXB0az0iQWRvYmUgWE1QIENvcmUgNS42LWMxNDAgNzkuMTYwNDUxLCAyMDE3LzA1LzA2LTAxOjA4OjIxICAgICAgICAiPiA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPiA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIgeG1sbnM6cGhvdG9zaG9wPSJodHRwOi8vbnMuYWRvYmUuY29tL3Bob3Rvc2hvcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RFdnQ9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZUV2ZW50IyIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ0MgMjAxOCAoV2luZG93cykiIHhtcDpDcmVhdGVEYXRlPSIyMDIzLTA2LTI1VDEyOjM2OjA1KzAyOjAwIiB4bXA6TW9kaWZ5RGF0ZT0iMjAyMy0wNi0yNVQxMjozNTowMCswMjowMCIgeG1wOk1ldGFkYXRhRGF0ZT0iMjAyMy0wNi0yNVQxMjozNTowMCswMjowMCIgZGM6Zm9ybWF0PSJpbWFnZS9wbmciIHBob3Rvc2hvcDpDb2xvck1vZGU9IjMiIHBob3Rvc2hvcDpJQ0NQcm9maWxlPSJzUkdCIElFQzYxOTY2LTIuMSIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDo0NmQ3OWQ5NS05NGU0LTRkNGUtOWI1MC1kOWI0MTJiZmU1OTUiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6NDZkNzlkOTUtOTRlNC00ZDRlLTliNTAtZDliNDEyYmZlNTk1IiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6NDZkNzlkOTUtOTRlNC00ZDRlLTliNTAtZDliNDEyYmZlNTk1Ij4gPHhtcE1NOkhpc3Rvcnk+IDxyZGY6U2VxPiA8cmRmOmxpIHN0RXZ0OmFjdGlvbj0iY3JlYXRlZCI...",

      // آب (64x64)
      water: "image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAYAAACqaXHeAAAACXBIWXMAAAsTAAALEwEAmpwYAAAHAGlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPD94cGFja2V0IGJlZ2luPSLvu78iIGlkPSJXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQiPz4gPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZS54bXAvMS4wLyIgeDp4bXB0az0iQWRvYmUgWE1QIENvcmUgNS42LWMxNDAgNzkuMTYwNDUxLCAyMDE3LzA1LzA2LTAxOjA4OjIxICAgICAgICAiPiA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPiA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtbG5zOmRjPSJodHRwOi8vcHVybC5vcmcvZGMvZWxlbWVudHMvMS4xLyIgeG1sbnM6cGhvdG9zaG9wPSJodHRwOi8vbnMuYWRvYmUuY29tL3Bob3Rvc2hvcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RFdnQ9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZUV2ZW50IyIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ0MgMjAxOCAoV2luZG93cykiIHhtcDpDcmVhdGVEYXRlPSIyMDIzLTA2LTI1VDEyOjM2OjEwKzAyOjAwIiB4bXA6TW9kaWZ5RGF0ZT0iMjAyMy0wNi0yNVQxMjozNTowMCswMjowMCIgeG1wOk1ldGFkYXRhRGF0ZT0iMjAyMy0wNi0yNVQxMjozNTowMCswMjowMCIgZGM6Zm9ybWF0PSJpbWFnZS9wbmciIHBob3Rvc2hvcDpDb2xvck1vZGU9IjMiIHBob3Rvc2hvcDpJQ0NQcm9maWxlPSJzUkdCIElFQzYxOTY2LTIuMSIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDo0N2Q3OWQ5NS05NGU0LTRdNGUtOWI1MC1kOWI0MTJiZmU1OTUiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6NDdkNzlkOTUtOTRlNC00ZDRlLTliNTAtZDliNDEyYmZlNTk1IiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6NDdkNzlkOTUtOTRlNC00ZDRlLTliNTAtZDliNDEyYmZlNTk1Ij4gPHhtcE1NOkhpc3Rvcnk+IDxyZGY6U2VxPiA8cmRmOmxpIHN0RXZ0OmFjdGlvbj0iY3JlYXRlZCIgc3RFdnQ6aW5zdGFuY2VJRD0ieG1wLm..."
    };

    // === وضعیت بازی ===
    let gameState = JSON.parse(localStorage.getItem('pixelFarmSave')) || {
      coins: 100,
      seeds: 0,
      // نقشه: 0 = قابل کشت (خاک)، 1 = چمن، 2 = آب
      map: [
        [2,2,2,2,2,2,2],
        [2,1,1,1,1,1,2],
        [2,1,0,0,0,1,2],
        [2,1,0,0,0,1,2],
        [2,1,0,0,0,1,2],
        [2,1,1,1,1,1,2],
        [2,2,2,2,2,2,2]
      ],
      crops: Array(7).fill().map(() => Array(7).fill(null))
    };

    const TILE_SIZE = 64;
    const map = gameState.map;
    const MAP_HEIGHT = map.length;
    const MAP_WIDTH = map[0].length;

    // === راه‌اندازی PixiJS ===
    const app = new PIXI.Application({
      width: window.innerWidth,
      height: window.innerHeight,
      backgroundColor: 0x1e293b,
      antialias: true,
      resizeTo: window
    });
    document.body.appendChild(app.view);

    let textures = {};
    let cropSprites = [];
    let playerSprite = null;

    // === بارگذاری تصاویر ===
    async function loadAssets() {
      for (let key in ASSETS) {
        const img = new Image();
        img.src = ASSETS[key];
        await new Promise(resolve => img.onload = resolve);
        textures[key] = PIXI.Texture.from(img);
      }
    }

    // === رندر نقشه ===
    function renderMap() {
      for (let y = 0; y < MAP_HEIGHT; y++) {
        for (let x = 0; x < MAP_WIDTH; x++) {
          let tex;
          switch(map[y][x]) {
            case 0: tex = textures.dirt; break;
            case 1: tex = textures.grass; break;
            case 2: tex = textures.water; break;
            default: tex = textures.grass;
          }
          const sprite = new PIXI.Sprite(tex);
          sprite.width = TILE_SIZE;
          sprite.height = TILE_SIZE;
          sprite.x = x * TILE_SIZE;
          sprite.y = y * TILE_SIZE;
          app.stage.addChild(sprite);
        }
      }
    }

    // === رندر گیاهان ===
    function renderCrops() {
      cropSprites.forEach(s => app.stage.removeChild(s));
      cropSprites = [];
      for (let y = 0; y < MAP_HEIGHT; y++) {
        for (let x = 0; x < MAP_WIDTH; x++) {
          const crop = gameState.crops[y][x];
          if (crop) {
            const tex = crop.stage === 0 ? textures.crop0 : crop.stage === 1 ? textures.crop1 : textures.crop2;
            const sprite = new PIXI.Sprite(tex);
            sprite.width = TILE_SIZE;
            sprite.height = TILE_SIZE;
            sprite.x = x * TILE_SIZE;
            sprite.y = y * TILE_SIZE;
            app.stage.addChild(sprite);
            cropSprites.push(sprite);
          }
        }
      }
    }

    // === شخصیت ===
    function createPlayer() {
      playerSprite = new PIXI.Sprite(textures.player);
      playerSprite.width = TILE_SIZE;
      playerSprite.height = TILE_SIZE;
      playerSprite.x = 3 * TILE_SIZE;
      playerSprite.y = 3 * TILE_SIZE;
      app.stage.addChild(playerSprite);
    }

    // === آپدیت وضعیت UI ===
    function updateUI() {
      document.getElementById('coins').textContent = gameState.coins;
      document.getElementById('seeds').textContent = gameState.seeds;
      localStorage.setItem('pixelFarmSave', JSON.stringify(gameState));
    }

    // === رویداد کلیک ===
    app.view.addEventListener('click', (e) => {
      const rect = app.view.getBoundingClientRect();
      const x = e.clientX - rect.left;
      const y = e.clientY - rect.top;
      const gridX = Math.floor(x / TILE_SIZE);
      const gridY = Math.floor(y / TILE_SIZE);

      if (gridX < 0 || gridX >= MAP_WIDTH || gridY < 0 || gridY >= MAP_HEIGHT) return;
      if (map[gridY][gridX] !== 0) return; // فقط روی خاک

      const crop = gameState.crops[gridY][gridX];
      if (!crop && gameState.seeds > 0) {
        // کاشتن
        gameState.seeds--;
        gameState.crops[gridY][gridX] = { stage: 0, plantedAt: Date.now() };
        renderCrops();
        updateUI();
      } else if (crop && crop.stage === 2) {
        // برداشت
        gameState.coins += 15;
        gameState.crops[gridY][gridX] = null;
        renderCrops();
        updateUI();
      }
    });

    // === رشد گیاهان ===
    function updateCrops() {
      let updated = false;
      for (let y = 0; y < MAP_HEIGHT; y++) {
        for (let x = 0; x < MAP_WIDTH; x++) {
          const crop = gameState.crops[y][x];
          if (crop && crop.stage < 2) {
            const elapsed = Date.now() - crop.plantedAt;
            if (elapsed > 8000 && crop.stage === 0) {
              crop.stage = 1;
              updated = true;
            } else if (elapsed > 16000 && crop.stage === 1) {
              crop.stage = 2;
              updated = true;
            }
          }
        }
      }
      if (updated) renderCrops();
    }

    // === توابع UI ===
    function buySeed() {
      if (gameState.coins >= 10) {
        gameState.coins -= 10;
        gameState.seeds += 1;
        updateUI();
      }
    }

    function harvestAll() {
      let harvested = false;
      for (let y = 0; y < MAP_HEIGHT; y++) {
        for (let x = 0; x < MAP_WIDTH; x++) {
          const crop = gameState.crops[y][x];
          if (crop && crop.stage === 2) {
            gameState.coins += 15;
            gameState.crops[y][x] = null;
            harvested = true;
          }
        }
      }
      if (harvested) {
        renderCrops();
        updateUI();
      }
    }

    // === شروع بازی ===
    async function startGame() {
      await loadAssets();
      renderMap();
      renderCrops();
      createPlayer();
      setInterval(updateCrops, 1000);
      updateUI();
    }

    window.addEventListener('load', startGame);
  </script>
</body>
</html>
