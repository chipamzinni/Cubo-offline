<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Cube Solver Offline — animado + centros fixos</title>
<style>
:root{--bg:#0f1115;--panel:#1a1d24;--text:#e8eaed;--accent:#4f8cff}
*{box-sizing:border-box;margin:0;padding:0}
body{font-family:system-ui,-apple-system,sans-serif;background:var(--bg);color:var(--text);min-height:100vh;display:flex;flex-direction:column;align-items:center;padding:10px 12px 24px;overflow-x:hidden}
h1{font-size:1.25rem;margin-bottom:2px;text-align:center}
.subtitle{font-size:.78rem;opacity:.7;margin-bottom:8px;text-align:center;max-width:360px}
.scene{width:300px;height:300px;perspective:900px;margin:0 auto 12px;display:flex;align-items:center;justify-content:center;touch-action:none}
.cube{width:126px;height:126px;position:relative;transform-style:preserve-3d;transition:transform .55s cubic-bezier(.4,0,.2,1)}
.cubie{position:absolute;width:42px;height:42px;left:42px;top:42px;transform-style:preserve-3d}
.cubie .sticker{position:absolute;width:40px;height:40px;margin:-20px 0 0 -20px;left:50%;top:50%;border-radius:3px;border:1px solid rgba(0,0,0,.45);box-shadow:inset 0 0 0 1px rgba(255,255,255,.15);backface-visibility:hidden; color-scheme: only light;}
.is-white, .color-btn.w { background: #ffffff !important; color-scheme: only light !important; }
.paint-face .is-white { background: #ffffff !important; }
.cubie .sticker.U{transform:rotateX(90deg) translateZ(21px)}
.cubie .sticker.D{transform:rotateX(-90deg) translateZ(21px)}
.cubie .sticker.F{transform:translateZ(21px)}
.cubie .sticker.B{transform:rotateY(180deg) translateZ(21px)}
.cubie .sticker.L{transform:rotateY(-90deg) translateZ(21px)}
.cubie .sticker.R{transform:rotateY(90deg) translateZ(21px)}
.cubie.turning{transition:transform .55s cubic-bezier(.33,1,.68,1)}
.layer-pivot{position:absolute;inset:0;transform-style:preserve-3d;transition:transform .55s cubic-bezier(.33,1,.68,1);pointer-events:none}
.panel{width:100%;max-width:380px;background:var(--panel);border-radius:18px;padding:14px}
.mode-title{text-align:center;font-size:.95rem;font-weight:600;margin-bottom:10px}
.palette{display:flex;gap:10px;justify-content:center;margin-bottom:12px;flex-wrap:wrap}
.color-btn{width:34px;height:34px;border-radius:50%;border:3px solid transparent;cursor:pointer}
.color-btn.active{border-color:#fff;transform:scale(1.12)}
.color-btn.w{background:#ffffff !important; box-shadow:inset 0 0 0 1px #888; color-scheme: only light;}.color-btn.y{background:#ffd60a}.color-btn.r{background:#e03131}
.color-btn.o{background:#fd7e14}.color-btn.b{background:#0a84ff}.color-btn.g{background:#30d158}
.color-btn.w.active{border-color:#4f8cff !important}
.current-face-wrap{display:flex;flex-direction:column;align-items:center;gap:8px}
.face-label{font-size:.88rem;opacity:.85}
.paint-face{width:174px;height:174px;display:grid;grid-template-columns:repeat(3,1fr);grid-template-rows:repeat(3,1fr);gap:2px;background:#111;padding:2px;border-radius:8px}
.paint-face div{border-radius:4px;cursor:pointer;border:1px solid rgba(0,0,0,.25)}
.progress{text-align:center;font-size:.82rem;opacity:.75;margin:8px 0 4px}
.btn-row{display:flex;gap:8px;justify-content:center;flex-wrap:wrap;margin-top:10px}
button{background:var(--accent);color:#fff;border:none;padding:11px 16px;border-radius:12px;font-size:.92rem;font-weight:600;cursor:pointer;min-width:96px}
button:disabled{opacity:.4;cursor:not-allowed}
button.secondary{background:#333}button.danger{background:#ff453a}button.success{background:#30d158;color:#111}
.error-msg{display:none;background:#ff453a;color:#fff;text-align:center;padding:10px 12px;border-radius:10px;font-size:.86rem;margin-top:10px;line-height:1.35}
.error-msg.show{display:block}
.info-msg{display:none;background:#252830;color:#ccc;text-align:center;padding:10px 12px;border-radius:10px;font-size:.86rem;margin-top:10px}
.info-msg.show{display:block}
.center-sticker{cursor:not-allowed!important;position:relative;box-shadow:inset 0 0 0 2px rgba(255,255,255,.35)}
.center-sticker::after{content:'●';position:absolute;inset:0;display:flex;align-items:center;justify-content:center;font-size:10px;opacity:.55;pointer-events:none}
.paint-face div.locked{cursor:not-allowed;opacity:.92}
.scramble-btn{background:#7c5cff}
.progress-bar{height:6px;background:#252830;border-radius:4px;overflow:hidden;margin:8px 0 4px}
.progress-bar > i{display:block;height:100%;background:linear-gradient(90deg,#4f8cff,#30d158);width:0%;transition:width .35s ease}
.stats{display:flex;justify-content:space-around;font-size:.8rem;opacity:.8;margin:6px 0}
.stats span{background:#252830;padding:4px 10px;border-radius:8px}
.speed-row{display:flex;align-items:center;gap:10px;margin:10px 0 4px;padding:0 4px}
.speed-row label{font-size:.78rem;opacity:.75;white-space:nowrap}
.speed-row input[type=range]{flex:1;accent-color:var(--accent);height:6px;cursor:pointer}
.speed-row #speedVal{font-size:.78rem;min-width:2.4em;text-align:right;opacity:.9}
.move-list{max-height:72px;overflow-y:auto}
.toast{position:fixed;bottom:24px;left:50%;transform:translateX(-50%) translateY(20px);background:#30d158;color:#111;padding:10px 18px;border-radius:12px;font-weight:600;font-size:.9rem;opacity:0;pointer-events:none;transition:all .3s;z-index:99}
.toast.show{opacity:1;transform:translateX(-50%) translateY(0)}
.face-nav{display:flex;gap:6px;justify-content:center;flex-wrap:wrap;margin:8px 0}
.face-nav button{min-width:42px;padding:8px 10px;font-size:.78rem;background:#2a2f3a}
.face-nav button.active{background:var(--accent)}
.kbd{font-size:.7rem;opacity:.5;margin-top:6px;text-align:center}

.steps{margin-top:4px;text-align:center}
.step-info{background:#252830;padding:12px;border-radius:12px;margin-bottom:10px}
.current-move{font-size:1.35rem;font-weight:700;color:var(--accent);margin:4px 0;min-height:1.5em}
.move-list{font-family:monospace;font-size:.8rem;word-break:break-all;opacity:.85;margin-top:4px}
.hint{font-size:.76rem;opacity:.65;text-align:center;margin-top:8px;line-height:1.35}
@media(max-width:400px){
.scene{width:270px;height:270px}.cube{width:108px;height:108px}
.core{width:108px;height:108px;left:0;top:0}.core-face{width:108px;height:108px}
.core-face.U{transform:rotateX(90deg) translateZ(54px)}.core-face.D{transform:rotateX(-90deg) translateZ(54px)}
.core-face.F{transform:translateZ(54px)}.core-face.B{transform:rotateY(180deg) translateZ(54px)}
.core-face.L{transform:rotateY(-90deg) translateZ(54px)}.core-face.R{transform:rotateY(90deg) translateZ(54px)}
.face{width:98px;height:98px;left:5px;top:5px}
.face.U{transform:rotateX(90deg) translateZ(54.6px)}.face.D{transform:rotateX(-90deg) translateZ(54.6px)}
.face.F{transform:translateZ(54.6px)}.face.B{transform:rotateY(180deg) translateZ(54.6px)}
.face.L{transform:rotateY(-90deg) translateZ(54.6px)}.face.R{transform:rotateY(90deg) translateZ(54.6px)}
.paint-face{width:156px;height:156px}
}
</style>
</head>
<body>
<!-- Splash OBLIRY -->
<div id="splash" style="position:fixed;inset:0;background:#fff;z-index:9999;display:flex;align-items:center;justify-content:center;transition:background .35s ease,opacity .4s ease;">
  <div id="splashText" style="font-family:system-ui,sans-serif;font-size:2.4rem;font-weight:700;letter-spacing:.25em;color:#fff;opacity:0;transition:opacity .35s ease;">OBLIRY</div>
</div>

<!-- Menu ⋮ -->
<div id="menuBtn" style="position:fixed;top:12px;right:14px;z-index:100;width:40px;height:40px;border-radius:50%;background:rgba(255,255,255,.08);display:flex;align-items:center;justify-content:center;cursor:pointer;font-size:1.4rem;color:#e8eaed;user-select:none;">⋮</div>
<div id="menuPanel" style="display:none;position:fixed;top:56px;right:14px;z-index:100;background:#1a1d24;border-radius:14px;padding:14px 18px;min-width:220px;box-shadow:0 8px 30px rgba(0,0,0,.45);font-size:.85rem;line-height:1.55;">
  <div style="font-weight:600;margin-bottom:8px;color:#4f8cff;">Atalhos de teclado</div>
  <div>← → &nbsp; navegar faces / passos</div>
  <div>Espaço &nbsp; próximo passo</div>
  <div>P &nbsp; play / pause</div>
  <div>Esc &nbsp; voltar a pintar</div>
  <hr style="border:none;border-top:1px solid #333;margin:10px 0;">
  <div style="opacity:.75;font-size:.78rem;">Feito por <b>Raphael Rocha</b><br>OBLIRY</div>
</div>

<!-- Loading do solver (bloqueia cliques) -->
<div id="loadingOverlay" style="display:none;position:fixed;inset:0;background:rgba(15,17,21,.82);z-index:500;align-items:center;justify-content:center;flex-direction:column;gap:14px;backdrop-filter:blur(4px);">
  <div style="width:42px;height:42px;border:3px solid #333;border-top-color:#4f8cff;border-radius:50%;animation:spin .7s linear infinite;"></div>
  <div id="loadingTitle" style="color:#e8eaed;font-size:.95rem;font-weight:500;">Preparando solver…</div>
  <div id="loadingSub" style="color:#888;font-size:.78rem;">Aguarde alguns segundos</div>
  <button id="btnCancelLoad" class="secondary" style="margin-top:8px;display:none">Cancelar</button>
</div>

<!-- Modal câmera — leitura em tempo real -->
<div id="scanModal" style="display:none;position:fixed;inset:0;background:rgba(0,0,0,.9);z-index:600;flex-direction:column;align-items:center;justify-content:flex-start;padding:12px;overflow-y:auto;">
  <div style="color:#fff;font-weight:600;margin:8px 0 6px;font-size:1rem;text-align:center">Enquadre o cubo no quadrado</div>
  <div style="color:#aaa;font-size:.75rem;text-align:center;margin-bottom:8px;max-width:300px;line-height:1.35">Só a área central é lida (fundo ignorado). Vire o cubo e toque em Confirmar a cada face.</div>
  <div style="position:relative;width:min(300px,92vw);aspect-ratio:1;border-radius:14px;overflow:hidden;background:#111;border:2px solid #4f8cff">
    <video id="scanVideo" autoplay playsinline muted style="width:100%;height:100%;object-fit:cover"></video>
    <div id="scanGridOverlay" style="position:absolute;inset:16%;display:grid;grid-template-columns:repeat(3,1fr);grid-template-rows:repeat(3,1fr);gap:3px;pointer-events:none">
      <div class="sg"></div><div class="sg"></div><div class="sg"></div>
      <div class="sg"></div><div class="sg"></div><div class="sg"></div>
      <div class="sg"></div><div class="sg"></div><div class="sg"></div>
    </div>
    <div style="position:absolute;inset:16%;border:2px dashed rgba(255,255,255,.5);border-radius:6px;pointer-events:none"></div>
  </div>
  <canvas id="scanCanvas" style="display:none"></canvas>
  <div id="scanFaceDots" style="display:flex;gap:6px;margin-top:12px;flex-wrap:wrap;justify-content:center"></div>
  <div id="scanResult" style="margin-top:8px;color:#ccc;font-size:.85rem;min-height:1.3em;text-align:center">Aguardando câmera…</div>
  <div style="display:flex;gap:10px;margin-top:12px;flex-wrap:wrap;justify-content:center">
    <button id="btnCapture" class="success">✓ Confirmar face</button>
    <button id="btnCloseScan" class="secondary">Fechar</button>
  </div>
</div>
<style>
@keyframes spin{to{transform:rotate(360deg)}}
#scanGridOverlay .sg{border-radius:4px;border:1px solid rgba(255,255,255,.35);background:rgba(0,0,0,.25);transition:background .12s}
</style>

<h1>Cube Solver Offline</h1>
<div class="scene" id="scene">
<div class="cube" id="cube"></div>
</div>
<div class="panel">
<!-- Tela inicial: escolha de esquema de cores -->
<div id="startMode">
  <div class="mode-title">Como está o seu cubo?</div>
  <p style="text-align:center;font-size:.82rem;opacity:.7;margin-bottom:14px;line-height:1.4">Alguns cubos têm as cores dos centros em posições diferentes</p>
  <div style="display:flex;gap:12px;justify-content:center;flex-wrap:wrap;margin-bottom:16px">
    <div id="optStandard" class="scheme-opt active" style="width:140px;padding:14px 10px;border-radius:14px;border:2px solid #4f8cff;background:#252830;cursor:pointer;text-align:center">
      <div style="font-size:1.6rem;margin-bottom:6px">⬜</div>
      <div style="font-weight:600;font-size:.9rem">Padrão</div>
      <div style="font-size:.72rem;opacity:.65;margin-top:4px">Branco cima<br>Verde frente</div>
    </div>
    <div id="optCustom" class="scheme-opt" style="width:140px;padding:14px 10px;border-radius:14px;border:2px solid #333;background:#252830;cursor:pointer;text-align:center">
      <div style="font-size:1.6rem;margin-bottom:6px">🎨</div>
      <div style="font-weight:600;font-size:.9rem">Escolher cores</div>
      <div style="font-size:.72rem;opacity:.65;margin-top:4px">Definir centros<br>do seu cubo</div>
    </div>
  </div>
  <div id="customCenters" style="display:none;margin-bottom:14px">
    <div id="centerStepLabel" style="text-align:center;font-size:.9rem;font-weight:600;margin-bottom:6px;color:#4f8cff">Passo 1/6 — Mostre a face de CIMA</div>
    <div style="text-align:center;font-size:.78rem;opacity:.7;margin-bottom:10px;line-height:1.35">Mostre a face indicada. A cor do <b>centro</b> define a cor desse lado no cubo 3D. Quadradinhos cinza → coloridos.</div>
    <div id="centerDots" style="display:flex;gap:6px;justify-content:center;margin-bottom:12px;flex-wrap:wrap"></div>
    <div class="btn-row" style="margin-bottom:8px">
      <button id="btnPrevCenter" class="secondary">◀ Anterior</button>
      <button id="btnScanCenter" class="secondary" style="background:#2a2f3a">📷 Capturar</button>
      <button id="btnNextCenter" class="secondary" disabled>Próximo ▶</button>
    </div>
    <div class="palette" id="centerPalette" style="margin-top:6px"></div>
    <div style="text-align:center;font-size:.72rem;opacity:.55;margin-top:6px">Ou escolha a cor manualmente na paleta</div>
  </div>
  <div class="btn-row">
    <button id="btnStartApp" class="success" style="min-width:160px">Começar</button>
  </div>
</div>

<div id="paintMode" style="display:none">
<div class="mode-title">Pinte a face atual</div>
<div class="palette">
<div class="color-btn w active" data-color="w"></div>
<div class="color-btn y" data-color="y"></div>
<div class="color-btn r" data-color="r"></div>
<div class="color-btn o" data-color="o"></div>
<div class="color-btn b" data-color="b"></div>
<div class="color-btn g" data-color="g"></div>
</div>
<div class="current-face-wrap">
<div class="face-label" id="faceLabel">Face: Frente (Verde)</div>
<div class="paint-face" id="paintFace"></div>
</div>
<div class="progress" id="progress">Face 1 de 6</div>
<div class="btn-row">
<button id="btnPrevFace" class="secondary">◀ Anterior</button>
<button id="btnNextFace">Próximo ▶</button>
</div>
<div class="face-nav" id="faceNav"></div>
<div class="btn-row" style="margin-top:10px">
<button id="btnScan" class="secondary" style="background:#2a2f3a">📷 Escanear</button>
<button id="btnScramble" class="scramble-btn">🎲 Embaralhar</button>
<button id="btnFinish" class="success" disabled>Concluir e Resolver</button>
<button id="btnReset" class="danger">Reset</button>
</div>
<div class="error-msg" id="errorMsg"></div>
<div class="info-msg" id="infoMsg" style="display:none">Preparando solver…</div>
</div>
<div id="solveMode" style="display:none">
<div class="mode-title">Solução passo a passo</div>
<div class="steps">
<div class="step-info">
<div id="stepText">Passo 0 / 0</div>
<div class="current-move" id="currentMove">—</div>
<div class="move-list" id="moveList"></div>
</div>
<div class="progress-bar"><i id="progBar"></i></div>
<div class="stats">
<span id="statMoves">0 moves</span>
<span id="statRemain">restam 0</span>
</div>
<div class="speed-row">
<label for="speedRange">Velocidade</label>
<input type="range" id="speedRange" min="1" max="5" step="1" value="2">
<span id="speedVal">2×</span>
</div>
<div class="btn-row">
<button id="btnPrev" class="secondary">◀ Anterior</button>
<button id="btnPlay">▶ Play</button>
<button id="btnNext">Próximo ▶</button>
</div>
<div class="btn-row" style="margin-top:10px">
<button id="btnCopy" class="secondary">📋 Copiar</button>
<button id="btnBackPaint" class="secondary">Voltar a pintar</button>
<button id="btnReset2" class="danger">Reset</button>
</div>
<p class="kbd">Atalhos: ← → espaço · Esc volta</p>
</div>
</div>
</div>
<script>
(function() {
  // Centers
  var B, BL, BR, Cube, D, DB, DBL, DF, DFR, DL, DLF, DR, DRB, F, FL, FR, L, R, U, UB, UBR, UF, UFL, UL, ULB, UR, URF, centerColor, centerFacelet, cornerColor, cornerFacelet, edgeColor, edgeFacelet;

  [U, R, F, D, L, B] = [0, 1, 2, 3, 4, 5];

  // Corners
  [URF, UFL, ULB, UBR, DFR, DLF, DBL, DRB] = [0, 1, 2, 3, 4, 5, 6, 7];

  // Edges
  [UR, UF, UL, UB, DR, DF, DL, DB, FR, FL, BL, BR] = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];

  [centerFacelet, cornerFacelet, edgeFacelet] = (function() {
    var _B, _D, _F, _L, _R, _U;
    _U = function(x) {
      return x - 1;
    };
    _R = function(x) {
      return _U(9) + x;
    };
    _F = function(x) {
      return _R(9) + x;
    };
    _D = function(x) {
      return _F(9) + x;
    };
    _L = function(x) {
      return _D(9) + x;
    };
    _B = function(x) {
      return _L(9) + x;
    };
    return [
      // Centers
      [4,
      13,
      22,
      31,
      40,
      49],
      // Corners
      [[_U(9),
      _R(1),
      _F(3)],
      [_U(7),
      _F(1),
      _L(3)],
      [_U(1),
      _L(1),
      _B(3)],
      [_U(3),
      _B(1),
      _R(3)],
      [_D(3),
      _F(9),
      _R(7)],
      [_D(1),
      _L(9),
      _F(7)],
      [_D(7),
      _B(9),
      _L(7)],
      [_D(9),
      _R(9),
      _B(7)]],
      // Edges
      [[_U(6),
      _R(2)],
      [_U(8),
      _F(2)],
      [_U(4),
      _L(2)],
      [_U(2),
      _B(2)],
      [_D(6),
      _R(8)],
      [_D(2),
      _F(8)],
      [_D(4),
      _L(8)],
      [_D(8),
      _B(8)],
      [_F(6),
      _R(4)],
      [_F(4),
      _L(6)],
      [_B(6),
      _L(4)],
      [_B(4),
      _R(6)]]
    ];
  })();

  centerColor = ['U', 'R', 'F', 'D', 'L', 'B'];

  cornerColor = [['U', 'R', 'F'], ['U', 'F', 'L'], ['U', 'L', 'B'], ['U', 'B', 'R'], ['D', 'F', 'R'], ['D', 'L', 'F'], ['D', 'B', 'L'], ['D', 'R', 'B']];

  edgeColor = [['U', 'R'], ['U', 'F'], ['U', 'L'], ['U', 'B'], ['D', 'R'], ['D', 'F'], ['D', 'L'], ['D', 'B'], ['F', 'R'], ['F', 'L'], ['B', 'L'], ['B', 'R']];

  Cube = (function() {
    var faceNames, faceNums, parseAlg;

    class Cube {
      constructor(other) {
        var x;
        if (other != null) {
          this.init(other);
        } else {
          this.identity();
        }
        // For moves to avoid allocating new objects each time
        this.newCenter = (function() {
          var k, results;
          results = [];
          for (x = k = 0; k <= 5; x = ++k) {
            results.push(0);
          }
          return results;
        })();
        this.newCp = (function() {
          var k, results;
          results = [];
          for (x = k = 0; k <= 7; x = ++k) {
            results.push(0);
          }
          return results;
        })();
        this.newEp = (function() {
          var k, results;
          results = [];
          for (x = k = 0; k <= 11; x = ++k) {
            results.push(0);
          }
          return results;
        })();
        this.newCo = (function() {
          var k, results;
          results = [];
          for (x = k = 0; k <= 7; x = ++k) {
            results.push(0);
          }
          return results;
        })();
        this.newEo = (function() {
          var k, results;
          results = [];
          for (x = k = 0; k <= 11; x = ++k) {
            results.push(0);
          }
          return results;
        })();
      }

      init(state) {
        this.center = state.center.slice(0);
        this.co = state.co.slice(0);
        this.ep = state.ep.slice(0);
        this.cp = state.cp.slice(0);
        return this.eo = state.eo.slice(0);
      }

      identity() {
        var x;
        // Initialize to the identity cube
        this.center = [0, 1, 2, 3, 4, 5];
        this.cp = [0, 1, 2, 3, 4, 5, 6, 7];
        this.co = (function() {
          var k, results;
          results = [];
          for (x = k = 0; k <= 7; x = ++k) {
            results.push(0);
          }
          return results;
        })();
        this.ep = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
        return this.eo = (function() {
          var k, results;
          results = [];
          for (x = k = 0; k <= 11; x = ++k) {
            results.push(0);
          }
          return results;
        })();
      }

      toJSON() {
        return {
          center: this.center,
          cp: this.cp,
          co: this.co,
          ep: this.ep,
          eo: this.eo
        };
      }

      asString() {
        var corner, edge, i, k, l, m, n, o, ori, p, result;
        result = [];
        for (i = k = 0; k <= 5; i = ++k) {
          result[9 * i + 4] = centerColor[this.center[i]];
        }
        for (i = l = 0; l <= 7; i = ++l) {
          corner = this.cp[i];
          ori = this.co[i];
          for (n = m = 0; m <= 2; n = ++m) {
            result[cornerFacelet[i][(n + ori) % 3]] = cornerColor[corner][n];
          }
        }
        for (i = o = 0; o <= 11; i = ++o) {
          edge = this.ep[i];
          ori = this.eo[i];
          for (n = p = 0; p <= 1; n = ++p) {
            result[edgeFacelet[i][(n + ori) % 2]] = edgeColor[edge][n];
          }
        }
        return result.join('');
      }

      static fromString(str) {
        var col1, col2, cube, i, j, k, l, m, o, ori, p, q, r, ref;
        cube = new Cube;
        for (i = k = 0; k <= 5; i = ++k) {
          for (j = l = 0; l <= 5; j = ++l) {
            if (str[9 * i + 4] === centerColor[j]) {
              cube.center[i] = j;
            }
          }
        }
        for (i = m = 0; m <= 7; i = ++m) {
          for (ori = o = 0; o <= 2; ori = ++o) {
            if ((ref = str[cornerFacelet[i][ori]]) === 'U' || ref === 'D') {
              break;
            }
          }
          col1 = str[cornerFacelet[i][(ori + 1) % 3]];
          col2 = str[cornerFacelet[i][(ori + 2) % 3]];
          for (j = p = 0; p <= 7; j = ++p) {
            if (col1 === cornerColor[j][1] && col2 === cornerColor[j][2]) {
              cube.cp[i] = j;
              cube.co[i] = ori % 3;
            }
          }
        }
        for (i = q = 0; q <= 11; i = ++q) {
          for (j = r = 0; r <= 11; j = ++r) {
            if (str[edgeFacelet[i][0]] === edgeColor[j][0] && str[edgeFacelet[i][1]] === edgeColor[j][1]) {
              cube.ep[i] = j;
              cube.eo[i] = 0;
              break;
            }
            if (str[edgeFacelet[i][0]] === edgeColor[j][1] && str[edgeFacelet[i][1]] === edgeColor[j][0]) {
              cube.ep[i] = j;
              cube.eo[i] = 1;
              break;
            }
          }
        }
        return cube;
      }

      clone() {
        return new Cube(this.toJSON());
      }

      // A class method returning a new random cube
      static random() {
        return new Cube().randomize();
      }

      isSolved() {
        var c, cent, clone, e, k, l, m;
        clone = this.clone();
        clone.move(clone.upright());
        for (cent = k = 0; k <= 5; cent = ++k) {
          if (clone.center[cent] !== cent) {
            return false;
          }
        }
        for (c = l = 0; l <= 7; c = ++l) {
          if (clone.cp[c] !== c) {
            return false;
          }
          if (clone.co[c] !== 0) {
            return false;
          }
        }
        for (e = m = 0; m <= 11; e = ++m) {
          if (clone.ep[e] !== e) {
            return false;
          }
          if (clone.eo[e] !== 0) {
            return false;
          }
        }
        return true;
      }

      // Multiply this Cube with another Cube, restricted to centers.
      centerMultiply(other) {
        var from, k, to;
        for (to = k = 0; k <= 5; to = ++k) {
          from = other.center[to];
          this.newCenter[to] = this.center[from];
        }
        [this.center, this.newCenter] = [this.newCenter, this.center];
        return this;
      }

      // Multiply this Cube with another Cube, restricted to corners.
      cornerMultiply(other) {
        var from, k, to;
        for (to = k = 0; k <= 7; to = ++k) {
          from = other.cp[to];
          this.newCp[to] = this.cp[from];
          this.newCo[to] = (this.co[from] + other.co[to]) % 3;
        }
        [this.cp, this.newCp] = [this.newCp, this.cp];
        [this.co, this.newCo] = [this.newCo, this.co];
        return this;
      }

      // Multiply this Cube with another Cube, restricted to edges
      edgeMultiply(other) {
        var from, k, to;
        for (to = k = 0; k <= 11; to = ++k) {
          from = other.ep[to];
          this.newEp[to] = this.ep[from];
          this.newEo[to] = (this.eo[from] + other.eo[to]) % 2;
        }
        [this.ep, this.newEp] = [this.newEp, this.ep];
        [this.eo, this.newEo] = [this.newEo, this.eo];
        return this;
      }

      // Multiply this cube with another Cube
      multiply(other) {
        this.centerMultiply(other);
        this.cornerMultiply(other);
        this.edgeMultiply(other);
        return this;
      }

      move(arg) {
        var face, k, l, len, move, power, ref, ref1, x;
        ref = parseAlg(arg);
        for (k = 0, len = ref.length; k < len; k++) {
          move = ref[k];
          face = move / 3 | 0;
          power = move % 3;
          for (x = l = 0, ref1 = power; (0 <= ref1 ? l <= ref1 : l >= ref1); x = 0 <= ref1 ? ++l : --l) {
            this.multiply(Cube.moves[face]);
          }
        }
        return this;
      }

      upright() {
        var clone, i, j, k, l, result;
        clone = this.clone();
        result = [];
        for (i = k = 0; k <= 5; i = ++k) {
          if (clone.center[i] === F) {
            break;
          }
        }
        switch (i) {
          case D:
            result.push("x");
            break;
          case U:
            result.push("x'");
            break;
          case B:
            result.push("x2");
            break;
          case R:
            result.push("y");
            break;
          case L:
            result.push("y'");
        }
        if (result.length) {
          clone.move(result[0]);
        }
        for (j = l = 0; l <= 5; j = ++l) {
          if (clone.center[j] === U) {
            break;
          }
        }
        switch (j) {
          case L:
            result.push("z");
            break;
          case R:
            result.push("z'");
            break;
          case D:
            result.push("z2");
        }
        return result.join(' ');
      }

      static inverse(arg) {
        var face, k, len, move, power, result, str;
        result = (function() {
          var k, len, ref, results;
          ref = parseAlg(arg);
          results = [];
          for (k = 0, len = ref.length; k < len; k++) {
            move = ref[k];
            face = move / 3 | 0;
            power = move % 3;
            results.push(face * 3 + -(power - 1) + 1);
          }
          return results;
        })();
        result.reverse();
        if (typeof arg === 'string') {
          str = '';
          for (k = 0, len = result.length; k < len; k++) {
            move = result[k];
            face = move / 3 | 0;
            power = move % 3;
            str += faceNames[face];
            if (power === 1) {
              str += '2';
            } else if (power === 2) {
              str += "'";
            }
            str += ' ';
          }
          return str.substring(0, str.length - 1);
        } else if (arg.length != null) {
          return result;
        } else {
          return result[0];
        }
      }

    };

    Cube.prototype.randomize = (function() {
      var arePermutationsValid, generateValidRandomOrientation, generateValidRandomPermutation, getNumSwaps, isOrientationValid, randint, randomizeOrientation, result, shuffle;
      randint = function(min, max) {
        return min + Math.floor(Math.random() * (max - min + 1));
      };
      // Fisher-Yates shuffle adapted from https://stackoverflow.com/questions/2450954/how-to-randomize-shuffle-a-javascript-array
      shuffle = function(array) {
        var currentIndex, randomIndex, temporaryValue;
        currentIndex = array.length;
        // While there remain elements to shuffle...
        while (currentIndex !== 0) {
          // Pick a remaining element...
          randomIndex = randint(0, currentIndex - 1);
          currentIndex -= 1;
          // And swap it with the current element.
          temporaryValue = array[currentIndex];
          [array[currentIndex], array[randomIndex]] = [array[randomIndex], array[currentIndex]];
        }
      };
      getNumSwaps = function(arr) {
        var cur, cycleLength, i, k, numSwaps, ref, seen, x;
        numSwaps = 0;
        seen = (function() {
          var k, ref, results;
          results = [];
          for (x = k = 0, ref = arr.length - 1; (0 <= ref ? k <= ref : k >= ref); x = 0 <= ref ? ++k : --k) {
            results.push(false);
          }
          return results;
        })();
        while (true) {
          // We compute the cycle decomposition
          cur = -1;
          for (i = k = 0, ref = arr.length - 1; (0 <= ref ? k <= ref : k >= ref); i = 0 <= ref ? ++k : --k) {
            if (!seen[i]) {
              cur = i;
              break;
            }
          }
          if (cur === -1) {
            break;
          }
          cycleLength = 0;
          while (!seen[cur]) {
            seen[cur] = true;
            cycleLength++;
            cur = arr[cur];
          }
          // A cycle is equivalent to cycleLength + 1 swaps
          numSwaps += cycleLength + 1;
        }
        return numSwaps;
      };
      arePermutationsValid = function(cp, ep) {
        var numSwaps;
        numSwaps = getNumSwaps(ep) + getNumSwaps(cp);
        return numSwaps % 2 === 0;
      };
      generateValidRandomPermutation = function(cp, ep) {
        // Each shuffle only takes around 12 operations and there's a 50%
        // chance of a valid permutation so it'll finish in very good time
        shuffle(ep);
        shuffle(cp);
        while (!arePermutationsValid(cp, ep)) {
          shuffle(ep);
          shuffle(cp);
        }
      };
      randomizeOrientation = function(arr, numOrientations) {
        var i, k, ori, ref;
        ori = 0;
        for (i = k = 0, ref = arr.length - 1; (0 <= ref ? k <= ref : k >= ref); i = 0 <= ref ? ++k : --k) {
          ori += (arr[i] = randint(0, numOrientations - 1));
        }
      };
      isOrientationValid = function(arr, numOrientations) {
        return arr.reduce(function(a, b) {
          return a + b;
        }) % numOrientations === 0;
      };
      generateValidRandomOrientation = function(co, eo) {
        // There is a 1/2 and 1/3 probably respectively of each of these
        // succeeding so the probability of them running 10 times before
        // success is already only 1% and only gets exponentially lower
        // and each generation is only in the 10s of operations which is nothing
        randomizeOrientation(co, 3);
        while (!isOrientationValid(co, 3)) {
          randomizeOrientation(co, 3);
        }
        randomizeOrientation(eo, 2);
        while (!isOrientationValid(eo, 2)) {
          randomizeOrientation(eo, 2);
        }
      };
      result = function() {
        generateValidRandomPermutation(this.cp, this.ep);
        generateValidRandomOrientation(this.co, this.eo);
        return this;
      };
      return result;
    })();

    Cube.moves = [
      {
        // U
        center: [0, 1, 2, 3, 4, 5],
        cp: [UBR,
      URF,
      UFL,
      ULB,
      DFR,
      DLF,
      DBL,
      DRB],
        co: [0,
      0,
      0,
      0,
      0,
      0,
      0,
      0],
        ep: [UB,
      UR,
      UF,
      UL,
      DR,
      DF,
      DL,
      DB,
      FR,
      FL,
      BL,
      BR],
        eo: [0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0]
      },
      {
        // R
        center: [0, 1, 2, 3, 4, 5],
        cp: [DFR,
      UFL,
      ULB,
      URF,
      DRB,
      DLF,
      DBL,
      UBR],
        co: [2,
      0,
      0,
      1,
      1,
      0,
      0,
      2],
        ep: [FR,
      UF,
      UL,
      UB,
      BR,
      DF,
      DL,
      DB,
      DR,
      FL,
      BL,
      UR],
        eo: [0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0]
      },
      {
        // F
        center: [0, 1, 2, 3, 4, 5],
        cp: [UFL,
      DLF,
      ULB,
      UBR,
      URF,
      DFR,
      DBL,
      DRB],
        co: [1,
      2,
      0,
      0,
      2,
      1,
      0,
      0],
        ep: [UR,
      FL,
      UL,
      UB,
      DR,
      FR,
      DL,
      DB,
      UF,
      DF,
      BL,
      BR],
        eo: [0,
      1,
      0,
      0,
      0,
      1,
      0,
      0,
      1,
      1,
      0,
      0]
      },
      {
        // D
        center: [0, 1, 2, 3, 4, 5],
        cp: [URF,
      UFL,
      ULB,
      UBR,
      DLF,
      DBL,
      DRB,
      DFR],
        co: [0,
      0,
      0,
      0,
      0,
      0,
      0,
      0],
        ep: [UR,
      UF,
      UL,
      UB,
      DF,
      DL,
      DB,
      DR,
      FR,
      FL,
      BL,
      BR],
        eo: [0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0]
      },
      {
        // L
        center: [0, 1, 2, 3, 4, 5],
        cp: [URF,
      ULB,
      DBL,
      UBR,
      DFR,
      UFL,
      DLF,
      DRB],
        co: [0,
      1,
      2,
      0,
      0,
      2,
      1,
      0],
        ep: [UR,
      UF,
      BL,
      UB,
      DR,
      DF,
      FL,
      DB,
      FR,
      UL,
      DL,
      BR],
        eo: [0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      0]
      },
      {
        // B
        center: [0, 1, 2, 3, 4, 5],
        cp: [URF,
      UFL,
      UBR,
      DRB,
      DFR,
      DLF,
      ULB,
      DBL],
        co: [0,
      0,
      1,
      2,
      0,
      0,
      2,
      1],
        ep: [UR,
      UF,
      UL,
      BR,
      DR,
      DF,
      DL,
      BL,
      FR,
      FL,
      UB,
      DB],
        eo: [0,
      0,
      0,
      1,
      0,
      0,
      0,
      1,
      0,
      0,
      1,
      1]
      },
      {
        // E
        center: [U,
      F,
      L,
      D,
      B,
      R],
        cp: [URF,
      UFL,
      ULB,
      UBR,
      DFR,
      DLF,
      DBL,
      DRB],
        co: [0,
      0,
      0,
      0,
      0,
      0,
      0,
      0],
        ep: [UR,
      UF,
      UL,
      UB,
      DR,
      DF,
      DL,
      DB,
      FL,
      BL,
      BR,
      FR],
        eo: [0,
      0,
      0,
      0,
      0,
      0,
      0,
      0,
      1,
      1,
      1,
      1]
      },
      {
        // M
        center: [B,
      R,
      U,
      F,
      L,
      D],
        cp: [URF,
      UFL,
      ULB,
      UBR,
      DFR,
      DLF,
      DBL,
      DRB],
        co: [0,
      0,
      0,
      0,
      0,
      0,
      0,
      0],
        ep: [UR,
      UB,
      UL,
      DB,
      DR,
      UF,
      DL,
      DF,
      FR,
      FL,
      BL,
      BR],
        eo: [0,
      1,
      0,
      1,
      0,
      1,
      0,
      1,
      0,
      0,
      0,
      0]
      },
      {
        // S
        center: [L,
      U,
      F,
      R,
      D,
      B],
        cp: [URF,
      UFL,
      ULB,
      UBR,
      DFR,
      DLF,
      DBL,
      DRB],
        co: [0,
      0,
      0,
      0,
      0,
      0,
      0,
      0],
        ep: [UL,
      UF,
      DL,
      UB,
      UR,
      DF,
      DR,
      DB,
      FR,
      FL,
      BL,
      BR],
        eo: [1,
      0,
      1,
      0,
      1,
      0,
      1,
      0,
      0,
      0,
      0,
      0]
      }
    ];

    faceNums = {
      U: 0,
      R: 1,
      F: 2,
      D: 3,
      L: 4,
      B: 5,
      E: 6,
      M: 7,
      S: 8,
      x: 9,
      y: 10,
      z: 11,
      u: 12,
      r: 13,
      f: 14,
      d: 15,
      l: 16,
      b: 17
    };

    faceNames = {
      0: 'U',
      1: 'R',
      2: 'F',
      3: 'D',
      4: 'L',
      5: 'B',
      6: 'E',
      7: 'M',
      8: 'S',
      9: 'x',
      10: 'y',
      11: 'z',
      12: 'u',
      13: 'r',
      14: 'f',
      15: 'd',
      16: 'l',
      17: 'b'
    };

    parseAlg = function(arg) {
      var k, len, move, part, power, ref, results;
      if (typeof arg === 'string') {
        ref = arg.split(/\s+/);
        // String
        results = [];
        for (k = 0, len = ref.length; k < len; k++) {
          part = ref[k];
          if (part.length === 0) {
            // First and last can be empty
            continue;
          }
          if (part.length > 2) {
            throw new Error(`Invalid move: ${part}`);
          }
          move = faceNums[part[0]];
          if (move === void 0) {
            throw new Error(`Invalid move: ${part}`);
          }
          if (part.length === 1) {
            power = 0;
          } else {
            if (part[1] === '2') {
              power = 1;
            } else if (part[1] === "'") {
              power = 2;
            } else {
              throw new Error(`Invalid move: ${part}`);
            }
          }
          results.push(move * 3 + power);
        }
        return results;
      } else if (arg.length != null) {
        // Already an array
        return arg;
      } else {
        // A single move
        return [arg];
      }
    };

    // x
    Cube.moves.push(new Cube().move("R M' L'").toJSON());

    // y
    Cube.moves.push(new Cube().move("U E' D'").toJSON());

    // z
    Cube.moves.push(new Cube().move("F S B'").toJSON());

    // u
    Cube.moves.push(new Cube().move("U E'").toJSON());

    // r
    Cube.moves.push(new Cube().move("R M'").toJSON());

    // f
    Cube.moves.push(new Cube().move("F S").toJSON());

    // d
    Cube.moves.push(new Cube().move("D E").toJSON());

    // l
    Cube.moves.push(new Cube().move("L M").toJSON());

    // b
    Cube.moves.push(new Cube().move("B S'").toJSON());

    return Cube;

  }).call(this);

  //# Globals
  if (typeof module !== "undefined" && module !== null) {
    module.exports = Cube;
  } else {
    this.Cube = Cube;
  }

}).call(this);

(function() {
  var B, BL, BR, Cnk, Cube, D, DB, DBL, DF, DFR, DL, DLF, DR, DRB, F, FL, FR, Include, L, N_FLIP, N_FRtoBR, N_PARITY, N_SLICE1, N_SLICE2, N_TWIST, N_UBtoDF, N_URFtoDLF, N_URtoDF, N_URtoUL, R, U, UB, UBR, UF, UFL, UL, ULB, UR, URF, allMoves1, allMoves2, computeMoveTable, computePruningTable, faceNames, faceNums, factorial, key, max, mergeURtoDF, moveTableParams, nextMoves1, nextMoves2, permutationIndex, pruning, pruningTableParams, rotateLeft, rotateRight, value,
    indexOf = [].indexOf;

  Cube = this.Cube || require('./cube');

  // Centers
  [U, R, F, D, L, B] = [0, 1, 2, 3, 4, 5];

  // Corners
  [URF, UFL, ULB, UBR, DFR, DLF, DBL, DRB] = [0, 1, 2, 3, 4, 5, 6, 7];

  // Edges
  [UR, UF, UL, UB, DR, DF, DL, DB, FR, FL, BL, BR] = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];

  //# Helpers

  // n choose k, i.e. the binomial coeffiecient
  Cnk = function(n, k) {
    var i, j, s;
    if (n < k) {
      return 0;
    }
    if (k > n / 2) {
      k = n - k;
    }
    s = 1;
    i = n;
    j = 1;
    while (i !== n - k) {
      s *= i;
      s /= j;
      i--;
      j++;
    }
    return s;
  };

  // n!
  factorial = function(n) {
    var f, i, m, ref;
    f = 1;
    for (i = m = 2, ref = n; (2 <= ref ? m <= ref : m >= ref); i = 2 <= ref ? ++m : --m) {
      f *= i;
    }
    return f;
  };

  // Maximum of two values
  max = function(a, b) {
    if (a > b) {
      return a;
    } else {
      return b;
    }
  };

  // Rotate elements between l and r left by one place
  rotateLeft = function(array, l, r) {
    var i, m, ref, ref1, tmp;
    tmp = array[l];
    for (i = m = ref = l, ref1 = r - 1; (ref <= ref1 ? m <= ref1 : m >= ref1); i = ref <= ref1 ? ++m : --m) {
      array[i] = array[i + 1];
    }
    return array[r] = tmp;
  };

  // Rotate elements between l and r right by one place
  rotateRight = function(array, l, r) {
    var i, m, ref, ref1, tmp;
    tmp = array[r];
    for (i = m = ref = r, ref1 = l + 1; (ref <= ref1 ? m <= ref1 : m >= ref1); i = ref <= ref1 ? ++m : --m) {
      array[i] = array[i - 1];
    }
    return array[l] = tmp;
  };

  // Generate a function that computes permutation indices.

  // The permutation index actually encodes two indices: Combination,
  // i.e. positions of the cubies start..end (A) and their respective
  // permutation (B). The maximum value for B is

  //   maxB = (end - start + 1)!

  // and the index is A * maxB + B
  permutationIndex = function(context, start, end, fromEnd = false) {
    var i, maxAll, maxB, maxOur, our, permName;
    maxOur = end - start;
    maxB = factorial(maxOur + 1);
    if (context === 'corners') {
      maxAll = 7;
      permName = 'cp';
    } else {
      maxAll = 11;
      permName = 'ep';
    }
    our = (function() {
      var m, ref, results;
      results = [];
      for (i = m = 0, ref = maxOur; (0 <= ref ? m <= ref : m >= ref); i = 0 <= ref ? ++m : --m) {
        results.push(0);
      }
      return results;
    })();
    return function(index) {
      var a, b, c, j, k, m, o, p, perm, q, ref, ref1, ref10, ref2, ref3, ref4, ref5, ref6, ref7, ref8, ref9, t, u, w, x, y, z;
      if (index != null) {
        for (i = m = 0, ref = maxOur; (0 <= ref ? m <= ref : m >= ref); i = 0 <= ref ? ++m : --m) {
          // Reset our to [start..end]
          our[i] = i + start;
        }
        b = index % maxB; // permutation
        a = index / maxB | 0; // combination
        
        // Invalidate all edges
        perm = this[permName];
        for (i = o = 0, ref1 = maxAll; (0 <= ref1 ? o <= ref1 : o >= ref1); i = 0 <= ref1 ? ++o : --o) {
          perm[i] = -1;
        }
// Generate permutation from index b
        for (j = p = 1, ref2 = maxOur; (1 <= ref2 ? p <= ref2 : p >= ref2); j = 1 <= ref2 ? ++p : --p) {
          k = b % (j + 1);
          b = b / (j + 1) | 0;
          // TODO: Implement rotateRightBy(our, 0, j, k)
          while (k > 0) {
            rotateRight(our, 0, j);
            k--;
          }
        }
        // Generate combination and set our edges
        x = maxOur;
        if (fromEnd) {
          for (j = q = 0, ref3 = maxAll; (0 <= ref3 ? q <= ref3 : q >= ref3); j = 0 <= ref3 ? ++q : --q) {
            c = Cnk(maxAll - j, x + 1);
            if (a - c >= 0) {
              perm[j] = our[maxOur - x];
              a -= c;
              x--;
            }
          }
        } else {
          for (j = t = ref4 = maxAll; (ref4 <= 0 ? t <= 0 : t >= 0); j = ref4 <= 0 ? ++t : --t) {
            c = Cnk(j, x + 1);
            if (a - c >= 0) {
              perm[j] = our[x];
              a -= c;
              x--;
            }
          }
        }
        return this;
      } else {
        perm = this[permName];
        for (i = u = 0, ref5 = maxOur; (0 <= ref5 ? u <= ref5 : u >= ref5); i = 0 <= ref5 ? ++u : --u) {
          our[i] = -1;
        }
        a = b = x = 0;
        // Compute the index a < ((maxAll + 1) choose (maxOur + 1)) and
        // the permutation
        if (fromEnd) {
          for (j = w = ref6 = maxAll; (ref6 <= 0 ? w <= 0 : w >= 0); j = ref6 <= 0 ? ++w : --w) {
            if ((start <= (ref7 = perm[j]) && ref7 <= end)) {
              a += Cnk(maxAll - j, x + 1);
              our[maxOur - x] = perm[j];
              x++;
            }
          }
        } else {
          for (j = y = 0, ref8 = maxAll; (0 <= ref8 ? y <= ref8 : y >= ref8); j = 0 <= ref8 ? ++y : --y) {
            if ((start <= (ref9 = perm[j]) && ref9 <= end)) {
              a += Cnk(j, x + 1);
              our[x] = perm[j];
              x++;
            }
          }
        }
// Compute the index b < (maxOur + 1)! for the permutation
        for (j = z = ref10 = maxOur; (ref10 <= 0 ? z <= 0 : z >= 0); j = ref10 <= 0 ? ++z : --z) {
          k = 0;
          while (our[j] !== start + j) {
            rotateLeft(our, 0, j);
            k++;
          }
          b = (j + 1) * b + k;
        }
        return a * maxB + b;
      }
    };
  };

  Include = {
    // The twist of the 8 corners, 0 <= twist < 3^7. The orientation of
    // the DRB corner is fully determined by the orientation of the other
    // corners.
    twist: function(twist) {
      var i, m, o, ori, parity, v;
      if (twist != null) {
        parity = 0;
        for (i = m = 6; m >= 0; i = --m) {
          ori = twist % 3;
          twist = (twist / 3) | 0;
          this.co[i] = ori;
          parity += ori;
        }
        this.co[7] = (3 - parity % 3) % 3;
        return this;
      } else {
        v = 0;
        for (i = o = 0; o <= 6; i = ++o) {
          v = 3 * v + this.co[i];
        }
        return v;
      }
    },
    // The flip of the 12 edges, 0 <= flip < 2^11. The orientation of the
    // BR edge is fully determined by the orientation of the other edges.
    flip: function(flip) {
      var i, m, o, ori, parity, v;
      if (flip != null) {
        parity = 0;
        for (i = m = 10; m >= 0; i = --m) {
          ori = flip % 2;
          flip = flip / 2 | 0;
          this.eo[i] = ori;
          parity += ori;
        }
        this.eo[11] = (2 - parity % 2) % 2;
        return this;
      } else {
        v = 0;
        for (i = o = 0; o <= 10; i = ++o) {
          v = 2 * v + this.eo[i];
        }
        return v;
      }
    },
    // Parity of the corner permutation
    cornerParity: function() {
      var i, j, m, o, ref, ref1, ref2, ref3, s;
      s = 0;
      for (i = m = ref = DRB, ref1 = URF + 1; (ref <= ref1 ? m <= ref1 : m >= ref1); i = ref <= ref1 ? ++m : --m) {
        for (j = o = ref2 = i - 1, ref3 = URF; (ref2 <= ref3 ? o <= ref3 : o >= ref3); j = ref2 <= ref3 ? ++o : --o) {
          if (this.cp[j] > this.cp[i]) {
            s++;
          }
        }
      }
      return s % 2;
    },
    // Parity of the edges permutation. Parity of corners and edges are
    // the same if the cube is solvable.
    edgeParity: function() {
      var i, j, m, o, ref, ref1, ref2, ref3, s;
      s = 0;
      for (i = m = ref = BR, ref1 = UR + 1; (ref <= ref1 ? m <= ref1 : m >= ref1); i = ref <= ref1 ? ++m : --m) {
        for (j = o = ref2 = i - 1, ref3 = UR; (ref2 <= ref3 ? o <= ref3 : o >= ref3); j = ref2 <= ref3 ? ++o : --o) {
          if (this.ep[j] > this.ep[i]) {
            s++;
          }
        }
      }
      return s % 2;
    },
    // Permutation of the six corners URF, UFL, ULB, UBR, DFR, DLF
    URFtoDLF: permutationIndex('corners', URF, DLF),
    // Permutation of the three edges UR, UF, UL
    URtoUL: permutationIndex('edges', UR, UL),
    // Permutation of the three edges UB, DR, DF
    UBtoDF: permutationIndex('edges', UB, DF),
    // Permutation of the six edges UR, UF, UL, UB, DR, DF
    URtoDF: permutationIndex('edges', UR, DF),
    // Permutation of the equator slice edges FR, FL, BL and BR
    FRtoBR: permutationIndex('edges', FR, BR, true)
  };

  for (key in Include) {
    value = Include[key];
    Cube.prototype[key] = value;
  }

  computeMoveTable = function(context, coord, size) {
    var apply, cube, i, inner, j, k, m, move, o, p, ref, results;
    // Loop through all valid values for the coordinate, setting cube's
    // state in each iteration. Then apply each of the 18 moves to the
    // cube, and compute the resulting coordinate.
    apply = context === 'corners' ? 'cornerMultiply' : 'edgeMultiply';
    cube = new Cube;
    results = [];
    for (i = m = 0, ref = size - 1; (0 <= ref ? m <= ref : m >= ref); i = 0 <= ref ? ++m : --m) {
      cube[coord](i);
      inner = [];
      for (j = o = 0; o <= 5; j = ++o) {
        move = Cube.moves[j];
        for (k = p = 0; p <= 2; k = ++p) {
          cube[apply](move);
          inner.push(cube[coord]());
        }
        // 4th face turn restores the cube
        cube[apply](move);
      }
      results.push(inner);
    }
    return results;
  };

  // Because we only have the phase 2 URtoDF coordinates, we need to
  // merge the URtoUL and UBtoDF coordinates to URtoDF in the beginning
  // of phase 2.
  mergeURtoDF = (function() {
    var a, b;
    a = new Cube;
    b = new Cube;
    return function(URtoUL, UBtoDF) {
      var i, m;
      // Collisions can be found because unset are set to -1
      a.URtoUL(URtoUL);
      b.UBtoDF(UBtoDF);
      for (i = m = 0; m <= 7; i = ++m) {
        if (a.ep[i] !== -1) {
          if (b.ep[i] !== -1) {
            return -1; // collision
          } else {
            b.ep[i] = a.ep[i];
          }
        }
      }
      return b.URtoDF();
    };
  })();

  N_TWIST = 2187; // 3^7 corner orientations

  N_FLIP = 2048; // 2^11 possible edge flips

  N_PARITY = 2; // 2 possible parities

  N_FRtoBR = 11880; // 12!/(12-4)! permutations of FR..BR edges

  N_SLICE1 = 495; // (12 choose 4) possible positions of FR..BR edges

  N_SLICE2 = 24; // 4! permutations of FR..BR edges in phase 2

  N_URFtoDLF = 20160; // 8!/(8-6)! permutations of URF..DLF corners

  
  // The URtoDF move table is only computed for phase 2 because the full
  // table would have >650000 entries
  N_URtoDF = 20160; // 8!/(8-6)! permutation of UR..DF edges in phase 2

  N_URtoUL = 1320; // 12!/(12-3)! permutations of UR..UL edges

  N_UBtoDF = 1320; // 12!/(12-3)! permutations of UB..DF edges

  
  // The move table for parity is so small that it's included here
  Cube.moveTables = {
    parity: [[1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1], [0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0]],
    twist: null,
    flip: null,
    FRtoBR: null,
    URFtoDLF: null,
    URtoDF: null,
    URtoUL: null,
    UBtoDF: null,
    mergeURtoDF: null
  };

  // Other move tables are computed on the fly
  moveTableParams = {
    // name: [scope, size]
    twist: ['corners', N_TWIST],
    flip: ['edges', N_FLIP],
    FRtoBR: ['edges', N_FRtoBR],
    URFtoDLF: ['corners', N_URFtoDLF],
    URtoDF: ['edges', N_URtoDF],
    URtoUL: ['edges', N_URtoUL],
    UBtoDF: ['edges', N_UBtoDF],
    mergeURtoDF: []
  };

  Cube.computeMoveTables = function(...tables) {
    var len, m, name, scope, size, tableName;
    if (tables.length === 0) {
      tables = (function() {
        var results;
        results = [];
        for (name in moveTableParams) {
          results.push(name);
        }
        return results;
      })();
    }
    for (m = 0, len = tables.length; m < len; m++) {
      tableName = tables[m];
      if (this.moveTables[tableName] !== null) {
        // Already computed
        continue;
      }
      if (tableName === 'mergeURtoDF') {
        this.moveTables.mergeURtoDF = (function() {
          var UBtoDF, URtoUL, o, results;
          results = [];
          for (URtoUL = o = 0; o <= 335; URtoUL = ++o) {
            results.push((function() {
              var p, results1;
              results1 = [];
              for (UBtoDF = p = 0; p <= 335; UBtoDF = ++p) {
                results1.push(mergeURtoDF(URtoUL, UBtoDF));
              }
              return results1;
            })());
          }
          return results;
        })();
      } else {
        [scope, size] = moveTableParams[tableName];
        this.moveTables[tableName] = computeMoveTable(scope, tableName, size);
      }
    }
    return this;
  };

  // Phase 1: All moves are valid
  allMoves1 = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17];

  // The list of next valid phase 1 moves when the given face was turned
  // in the last move
  nextMoves1 = (function() {
    var face, lastFace, m, next, o, p, power, results;
    results = [];
    for (lastFace = m = 0; m <= 5; lastFace = ++m) {
      next = [];
// Don't allow commuting moves, e.g. U U'. Also make sure that
// opposite faces are always moved in the same order, i.e. allow
// U D but no D U. This avoids sequences like U D U'.
      for (face = o = 0; o <= 5; face = ++o) {
        if (face !== lastFace && face !== lastFace - 3) {
// single, double or inverse move
          for (power = p = 0; p <= 2; power = ++p) {
            next.push(face * 3 + power);
          }
        }
      }
      results.push(next);
    }
    return results;
  })();

  // Phase 2: Double moves of all faces plus quarter moves of U and D
  allMoves2 = [0, 1, 2, 4, 7, 9, 10, 11, 13, 16];

  nextMoves2 = (function() {
    var face, lastFace, len, m, next, o, p, power, powers, results;
    results = [];
    for (lastFace = m = 0; m <= 5; lastFace = ++m) {
      next = [];
      for (face = o = 0; o <= 5; face = ++o) {
        if (!(face !== lastFace && face !== lastFace - 3)) {
          continue;
        }
        // Allow all moves of U and D and double moves of others
        powers = face === 0 || face === 3 ? [0, 1, 2] : [1];
        for (p = 0, len = powers.length; p < len; p++) {
          power = powers[p];
          next.push(face * 3 + power);
        }
      }
      results.push(next);
    }
    return results;
  })();

  // 8 values are encoded in one number
  pruning = function(table, index, value) {
    var pos, shift, slot;
    pos = index % 8;
    slot = index >> 3;
    shift = pos << 2;
    if (value != null) {
      // Set
      table[slot] &= ~(0xF << shift);
      table[slot] |= value << shift;
      return value;
    } else {
      // Get
      return (table[slot] & (0xF << shift)) >>> shift;
    }
  };

  computePruningTable = function(phase, size, currentCoords, nextIndex) {
    var current, depth, done, index, len, m, move, moves, next, o, ref, table, x;
    // Initialize all values to 0xF
    table = (function() {
      var m, ref, results;
      results = [];
      for (x = m = 0, ref = Math.ceil(size / 8) - 1; (0 <= ref ? m <= ref : m >= ref); x = 0 <= ref ? ++m : --m) {
        results.push(0xFFFFFFFF);
      }
      return results;
    })();
    if (phase === 1) {
      moves = allMoves1;
    } else {
      moves = allMoves2;
    }
    depth = 0;
    pruning(table, 0, depth);
    done = 1;
    // In each iteration, take each state found in the previous depth and
    // compute the next state. Stop when all states have been assigned a
    // depth.
    while (done !== size) {
      for (index = m = 0, ref = size - 1; (0 <= ref ? m <= ref : m >= ref); index = 0 <= ref ? ++m : --m) {
        if (!(pruning(table, index) === depth)) {
          continue;
        }
        current = currentCoords(index);
        for (o = 0, len = moves.length; o < len; o++) {
          move = moves[o];
          next = nextIndex(current, move);
          if (pruning(table, next) === 0xF) {
            pruning(table, next, depth + 1);
            done++;
          }
        }
      }
      depth++;
    }
    return table;
  };

  Cube.pruningTables = {
    sliceTwist: null,
    sliceFlip: null,
    sliceURFtoDLFParity: null,
    sliceURtoDFParity: null
  };

  pruningTableParams = {
    // name: [phase, size, currentCoords, nextIndex]
    sliceTwist: [
      1,
      N_SLICE1 * N_TWIST,
      function(index) {
        return [index % N_SLICE1,
      index / N_SLICE1 | 0];
      },
      function(current,
      move) {
        var newSlice,
      newTwist,
      slice,
      twist;
        [slice,
      twist] = current;
        newSlice = Cube.moveTables.FRtoBR[slice * 24][move] / 24 | 0;
        newTwist = Cube.moveTables.twist[twist][move];
        return newTwist * N_SLICE1 + newSlice;
      }
    ],
    sliceFlip: [
      1,
      N_SLICE1 * N_FLIP,
      function(index) {
        return [index % N_SLICE1,
      index / N_SLICE1 | 0];
      },
      function(current,
      move) {
        var flip,
      newFlip,
      newSlice,
      slice;
        [slice,
      flip] = current;
        newSlice = Cube.moveTables.FRtoBR[slice * 24][move] / 24 | 0;
        newFlip = Cube.moveTables.flip[flip][move];
        return newFlip * N_SLICE1 + newSlice;
      }
    ],
    sliceURFtoDLFParity: [
      2,
      N_SLICE2 * N_URFtoDLF * N_PARITY,
      function(index) {
        return [index % 2,
      (index / 2 | 0) % N_SLICE2,
      (index / 2 | 0) / N_SLICE2 | 0];
      },
      function(current,
      move) {
        var URFtoDLF,
      newParity,
      newSlice,
      newURFtoDLF,
      parity,
      slice;
        [parity,
      slice,
      URFtoDLF] = current;
        newParity = Cube.moveTables.parity[parity][move];
        newSlice = Cube.moveTables.FRtoBR[slice][move];
        newURFtoDLF = Cube.moveTables.URFtoDLF[URFtoDLF][move];
        return (newURFtoDLF * N_SLICE2 + newSlice) * 2 + newParity;
      }
    ],
    sliceURtoDFParity: [
      2,
      N_SLICE2 * N_URtoDF * N_PARITY,
      function(index) {
        return [index % 2,
      (index / 2 | 0) % N_SLICE2,
      (index / 2 | 0) / N_SLICE2 | 0];
      },
      function(current,
      move) {
        var URtoDF,
      newParity,
      newSlice,
      newURtoDF,
      parity,
      slice;
        [parity,
      slice,
      URtoDF] = current;
        newParity = Cube.moveTables.parity[parity][move];
        newSlice = Cube.moveTables.FRtoBR[slice][move];
        newURtoDF = Cube.moveTables.URtoDF[URtoDF][move];
        return (newURtoDF * N_SLICE2 + newSlice) * 2 + newParity;
      }
    ]
  };

  Cube.computePruningTables = function(...tables) {
    var len, m, name, params, tableName;
    if (tables.length === 0) {
      tables = (function() {
        var results;
        results = [];
        for (name in pruningTableParams) {
          results.push(name);
        }
        return results;
      })();
    }
    for (m = 0, len = tables.length; m < len; m++) {
      tableName = tables[m];
      if (this.pruningTables[tableName] !== null) {
        // Already computed
        continue;
      }
      params = pruningTableParams[tableName];
      this.pruningTables[tableName] = computePruningTable(...params);
    }
    return this;
  };

  Cube.initSolver = function() {
    Cube.computeMoveTables();
    return Cube.computePruningTables();
  };

  Cube.prototype.solveUpright = function(maxDepth = 22) {
    var State, freeStates, moveNames, phase1, phase1search, phase2, phase2search, solution, state, x;
    // Names for all moves, i.e. U, U2, U', F, F2, ...
    moveNames = (function() {
      var face, faceName, m, o, power, powerName, result;
      faceName = ['U', 'R', 'F', 'D', 'L', 'B'];
      powerName = ['', '2', "'"];
      result = [];
      for (face = m = 0; m <= 5; face = ++m) {
        for (power = o = 0; o <= 2; power = ++o) {
          result.push(faceName[face] + powerName[power]);
        }
      }
      return result;
    })();
    State = class State {
      constructor(cube) {
        this.parent = null;
        this.lastMove = null;
        this.depth = 0;
        if (cube) {
          this.init(cube);
        }
      }

      init(cube) {
        // Phase 1 coordinates
        this.flip = cube.flip();
        this.twist = cube.twist();
        this.slice = cube.FRtoBR() / N_SLICE2 | 0;
        // Phase 2 coordinates
        this.parity = cube.cornerParity();
        this.URFtoDLF = cube.URFtoDLF();
        this.FRtoBR = cube.FRtoBR();
        // These are later merged to URtoDF when phase 2 begins
        this.URtoUL = cube.URtoUL();
        this.UBtoDF = cube.UBtoDF();
        return this;
      }

      solution() {
        if (this.parent) {
          return this.parent.solution() + moveNames[this.lastMove] + ' ';
        } else {
          return '';
        }
      }

      //# Helpers
      move(table, index, move) {
        return Cube.moveTables[table][index][move];
      }

      pruning(table, index) {
        return pruning(Cube.pruningTables[table], index);
      }

      //# Phase 1

      // Return the next valid phase 1 moves for this state
      moves1() {
        if (this.lastMove !== null) {
          return nextMoves1[this.lastMove / 3 | 0];
        } else {
          return allMoves1;
        }
      }

      // Compute the minimum number of moves to the end of phase 1
      minDist1() {
        var d1, d2;
        // The maximum number of moves to the end of phase 1 wrt. the
        // combination flip and slice coordinates only
        d1 = this.pruning('sliceFlip', N_SLICE1 * this.flip + this.slice);
        // The combination of twist and slice coordinates
        d2 = this.pruning('sliceTwist', N_SLICE1 * this.twist + this.slice);
        // The true minimal distance is the maximum of these two
        return max(d1, d2);
      }

      // Compute the next phase 1 state for the given move
      next1(move) {
        var next;
        next = freeStates.pop();
        next.parent = this;
        next.lastMove = move;
        next.depth = this.depth + 1;
        next.flip = this.move('flip', this.flip, move);
        next.twist = this.move('twist', this.twist, move);
        next.slice = this.move('FRtoBR', this.slice * 24, move) / 24 | 0;
        return next;
      }

      //# Phase 2

      // Return the next valid phase 2 moves for this state
      moves2() {
        if (this.lastMove !== null) {
          return nextMoves2[this.lastMove / 3 | 0];
        } else {
          return allMoves2;
        }
      }

      // Compute the minimum number of moves to the solved cube
      minDist2() {
        var d1, d2, index1, index2;
        index1 = (N_SLICE2 * this.URtoDF + this.FRtoBR) * N_PARITY + this.parity;
        d1 = this.pruning('sliceURtoDFParity', index1);
        index2 = (N_SLICE2 * this.URFtoDLF + this.FRtoBR) * N_PARITY + this.parity;
        d2 = this.pruning('sliceURFtoDLFParity', index2);
        return max(d1, d2);
      }

      // Initialize phase 2 coordinates
      init2(top = true) {
        if (this.parent === null) {
          return;
        }
        // For other states, the phase 2 state is computed based on
        // parent's state.
        // Already assigned for the initial state
        this.parent.init2(false);
        this.URFtoDLF = this.move('URFtoDLF', this.parent.URFtoDLF, this.lastMove);
        this.FRtoBR = this.move('FRtoBR', this.parent.FRtoBR, this.lastMove);
        this.parity = this.move('parity', this.parent.parity, this.lastMove);
        this.URtoUL = this.move('URtoUL', this.parent.URtoUL, this.lastMove);
        this.UBtoDF = this.move('UBtoDF', this.parent.UBtoDF, this.lastMove);
        if (top) {
          // This is the initial phase 2 state. Get the URtoDF coordinate
          // by merging URtoUL and UBtoDF
          return this.URtoDF = this.move('mergeURtoDF', this.URtoUL, this.UBtoDF);
        }
      }

      // Compute the next phase 2 state for the given move
      next2(move) {
        var next;
        next = freeStates.pop();
        next.parent = this;
        next.lastMove = move;
        next.depth = this.depth + 1;
        next.URFtoDLF = this.move('URFtoDLF', this.URFtoDLF, move);
        next.FRtoBR = this.move('FRtoBR', this.FRtoBR, move);
        next.parity = this.move('parity', this.parity, move);
        next.URtoDF = this.move('URtoDF', this.URtoDF, move);
        return next;
      }

    };
    solution = null;
    phase1search = function(state) {
      var depth, m, ref, results;
      depth = 0;
      results = [];
      for (depth = m = 1, ref = maxDepth; (1 <= ref ? m <= ref : m >= ref); depth = 1 <= ref ? ++m : --m) {
        phase1(state, depth);
        if (solution !== null) {
          break;
        }
        results.push(depth++);
      }
      return results;
    };
    var __nodes = 0;
    var __nodeLimit = 250000; // evita travar em cubo impossível
    phase1 = function(state, depth) {
      var len, m, move, next, ref, ref1, results;
      if (++__nodes > __nodeLimit) { return; }
      if (depth === 0) {
        if (state.minDist1() === 0) {
          if (state.lastMove === null || (ref = state.lastMove, indexOf.call(allMoves2, ref) < 0)) {
            return phase2search(state);
          }
        }
      } else if (depth > 0) {
        if (state.minDist1() <= depth) {
          ref1 = state.moves1();
          results = [];
          for (m = 0, len = ref1.length; m < len; m++) {
            move = ref1[m];
            next = state.next1(move);
            phase1(next, depth - 1);
            freeStates.push(next);
            if (solution !== null) {
              break;
            } else {
              results.push(void 0);
            }
          }
          return results;
        }
      }
    };
    phase2search = function(state) {
      var depth, m, ref, results;
      // Initialize phase 2 coordinates
      state.init2();
      results = [];
      for (depth = m = 1, ref = maxDepth - state.depth; (1 <= ref ? m <= ref : m >= ref); depth = 1 <= ref ? ++m : --m) {
        phase2(state, depth);
        if (solution !== null) {
          break;
        }
        results.push(depth++);
      }
      return results;
    };
    phase2 = function(state, depth) {
      var len, m, move, next, ref, results;
      if (depth === 0) {
        if (state.minDist2() === 0) {
          return solution = state.solution();
        }
      } else if (depth > 0) {
        if (state.minDist2() <= depth) {
          ref = state.moves2();
          results = [];
          for (m = 0, len = ref.length; m < len; m++) {
            move = ref[m];
            next = state.next2(move);
            phase2(next, depth - 1);
            freeStates.push(next);
            if (solution !== null) {
              break;
            } else {
              results.push(void 0);
            }
          }
          return results;
        }
      }
    };
    freeStates = (function() {
      var m, ref, results;
      results = [];
      for (x = m = 0, ref = maxDepth + 1; (0 <= ref ? m <= ref : m >= ref); x = 0 <= ref ? ++m : --m) {
        results.push(new State);
      }
      return results;
    })();
    state = freeStates.pop().init(this);
    phase1search(state);
    freeStates.push(state);
    // Sem solução (cubo impossível ou profundidade insuficiente)
    if (solution == null) {
      return null;
    }
    // Trim the trailing space
    if (solution.length > 0) {
      solution = solution.substring(0, solution.length - 1);
    }
    return solution;
  };

  faceNums = {
    U: 0,
    R: 1,
    F: 2,
    D: 3,
    L: 4,
    B: 5
  };

  faceNames = {
    0: 'U',
    1: 'R',
    2: 'F',
    3: 'D',
    4: 'L',
    5: 'B'
  };

  Cube.prototype.solve = function(maxDepth = 22) {
    var clone, len, m, move, ref, rotation, solution, upright, uprightSolution;
    clone = this.clone();
    upright = clone.upright();
    clone.move(upright);
    rotation = new Cube().move(upright).center;
    uprightSolution = clone.solveUpright(maxDepth);
    solution = [];
    ref = uprightSolution.split(' ');
    for (m = 0, len = ref.length; m < len; m++) {
      move = ref[m];
      solution.push(faceNames[rotation[faceNums[move[0]]]]);
      if (move.length > 1) {
        solution[solution.length - 1] += move[1];
      }
    }
    return solution.join(' ');
  };

  Cube.scramble = function() {
    return Cube.inverse(Cube.random().solve());
  };

}).call(this);

</script>
<script>
// Vermelho e laranja bem distintos para não confundir
const COLORS = {w:'#ffffff', y:'#ffd60a', r:'#e03131', o:'#fd7e14', b:'#0a84ff', g:'#30d158'};
const FACES = ['F','R','B','L','U','D'];
const FACE_NAMES = {
  F:'Frente (Verde)', R:'Direita (Vermelho)', B:'Trás (Azul)',
  L:'Esquerda (Laranja)', U:'Cima (Branco)', D:'Baixo (Amarelo)'
};
let DEFAULT = {U:'w', D:'y', F:'g', B:'b', L:'o', R:'r'};
const Z = 63.6;
const FACE_CAMERA = {
  F:{x:-18,y:20}, R:{x:-18,y:-70}, B:{x:-18,y:200},
  L:{x:-18,y:110}, U:{x:-90,y:0}, D:{x:90,y:0}
};
// Mapeamento painel → índices do state[] 
// Topo do painel = parte de trás da face no 3D (corrige inversão U/D)
const PANEL_TO_STATE = {
  F:[0,1,2,3,4,5,6,7,8],
  R:[0,1,2,3,4,5,6,7,8],
  B:[0,1,2,3,4,5,6,7,8],
  L:[0,1,2,3,4,5,6,7,8],
  U:[0,1,2,3,4,5,6,7,8],
  D:[0,1,2,3,4,5,6,7,8]
};
let C2F = {w:'U', y:'D', r:'R', o:'L', g:'F', b:'B'};
let F2C = {U:'w', D:'y', R:'r', L:'o', F:'g', B:'b'};

let state = {};
FACES.forEach(f => state[f] = Array(9).fill(DEFAULT[f]));
let selectedColor = 'w';
let currentFaceIdx = 0;
let visited = {F:0,R:0,B:0,L:0,U:0,D:0};
let solution = [];
let currentStep = 0;
let playing = false;
let playTimer = null;
let animTimer = null;
let isAnimating = false;
let dragging = false;
let rotX = FACE_CAMERA.F.x, rotY = FACE_CAMERA.F.y;
let solverReady = false, solverBusy = false;

/** Para qualquer animação/timer e limpa flags — evita UI travada */
function hardStop() {
  if (playing) {
    try { clearTimeout(playTimer); clearInterval(playTimer); } catch (_) {}
    playing = false;
    const bp = document.getElementById('btnPlay');
    if (bp) bp.textContent = '▶ Play';
  }
  if (animTimer) {
    try { clearTimeout(animTimer); } catch (_) {}
    animTimer = null;
  }
  isAnimating = false;
  const orphan = cubeEl && cubeEl.querySelector('.layer-pivot');
  if (orphan) {
    try { orphan.remove(); } catch (_) {}
  }
}

const cubeEl = document.getElementById('cube');

const STEP = 42; // = tamanho do cubie → faces coladas sem gap

function updateCubeView(smooth) {
  cubeEl.style.transition = smooth === false ? 'none' : 'transform .55s cubic-bezier(.4,0,.2,1)';
  cubeEl.style.transform = `rotateX(${rotX}deg) rotateY(${rotY}deg)`;
}

function lookAtFace(face) {
  const cam = FACE_CAMERA[face] || FACE_CAMERA.F;
  let targetY = cam.y, dy = targetY - rotY;
  while (dy > 180) { targetY -= 360; dy = targetY - rotY; }
  while (dy < -180) { targetY += 360; dy = targetY - rotY; }
  rotX = cam.x; rotY = targetY;
  updateCubeView(true);
}

// Índices 0–8 de cada face a partir de (x,y,z) do cubie
// U: linhas de trás→frente (z -1→+1), cols esq→dir (x -1→+1)
// Mapeamento cubie (x,y,z) → índice 0–8 da face, alinhado ao cubejs/Kociemba
// (derivado dos cornerFacelets URF, UFL, ULB, UBR, DFR, DLF, DBL, DRB)
function faceIndex(face, x, y, z) {
  if (face === 'U') return (z + 1) * 3 + (x + 1);       // z-1→0 (trás), z+1→6 (frente)
  if (face === 'D') return (1 - z) * 3 + (x + 1);       // z+1→0 (frente), z-1→6 (trás)
  if (face === 'F') return (1 - y) * 3 + (x + 1);       // y+1→0 (topo)
  if (face === 'B') return (1 - y) * 3 + (1 - x);       // olhando B: x+1 à esquerda
  if (face === 'L') return (1 - y) * 3 + (z + 1);       // olhando L: z-1→0 (trás), z+1→2 (frente)
  if (face === 'R') return (1 - y) * 3 + (1 - z);       // olhando R: z+1→0 (frente), z-1→2 (trás)
  return 4;
}

function colorAt(face, x, y, z) {
  return COLORS[state[face][faceIndex(face, x, y, z)]];
}

function render3D() {
  cubeEl.innerHTML = '';
  for (let x = -1; x <= 1; x++) {
    for (let y = -1; y <= 1; y++) {
      for (let z = -1; z <= 1; z++) {
        if (x === 0 && y === 0 && z === 0) continue;
        const cubie = document.createElement('div');
        cubie.className = 'cubie';
        cubie.dataset.x = x;
        cubie.dataset.y = y;
        cubie.dataset.z = z;
        cubie.style.transform = `translate3d(${x * STEP}px, ${-y * STEP}px, ${z * STEP}px)`;

        const add = (face, cond) => {
          if (!cond) return;
          const s = document.createElement('div');
          s.className = 'sticker ' + face;
          const col = colorAt(face, x, y, z);
          s.style.background = col;
          if (col === '#ffffff' || col === '#fff') {
            s.style.setProperty('background', '#ffffff', 'important');
            s.classList.add('is-white');
          }
          cubie.appendChild(s);
        };
        add('U', y === 1);
        add('D', y === -1);
        add('F', z === 1);
        add('B', z === -1);
        add('L', x === -1);
        add('R', x === 1);
        cubeEl.appendChild(cubie);
      }
    }
  }
}

function buildFaceNav() {
  const nav = document.getElementById('faceNav');
  if (!nav) return;
  nav.innerHTML = '';
  FACES.forEach((f, i) => {
    const b = document.createElement('button');
    b.textContent = f;
    b.title = FACE_NAMES[f];
    if (i === currentFaceIdx) b.classList.add('active');
    b.onclick = () => {
      currentFaceIdx = i;
      renderPaintFace();
      lookAtFace(f);
      buildFaceNav();
    };
    nav.appendChild(b);
  });
}

function renderPaintFace() {
  const face = FACES[currentFaceIdx];
  const map = PANEL_TO_STATE[face];
  const wrap = document.getElementById('paintFace');
  wrap.innerHTML = '';
  document.getElementById('faceLabel').textContent = 'Face: ' + FACE_NAMES[face];
  document.getElementById('progress').textContent = `Face ${currentFaceIdx + 1} de 6`;
  visited[face] = 1;
  document.getElementById('btnFinish').disabled = !FACES.every(f => visited[f]);
  for (let p = 0; p < 9; p++) {
    const si = map[p];
    const d = document.createElement('div');
    const col = COLORS[state[face][si]];
    d.style.background = col;
    if (col === '#ffffff' || col === '#fff') {
      d.style.setProperty('background', '#ffffff', 'important');
      d.classList.add('is-white');
    }
    if (si === 4) {
      // Centro bloqueado — cor oficial da face
      d.classList.add('locked');
      d.title = 'Centro fixo (' + FACE_NAMES[face].split('(')[1];
      d.onclick = () => showToast('Centro fixo da face', true);
    } else {
      d.onclick = () => {
        state[face][si] = selectedColor;
        renderPaintFace();
        render3D();
      };
    }
    wrap.appendChild(d);
  }
  buildFaceNav();
}

function countColors() {
  const c = {w:0,y:0,r:0,o:0,b:0,g:0};
  FACES.forEach(f => state[f].forEach(col => c[col]++));
  return c;
}

function isValidColorCount() {
  return Object.values(countColors()).every(n => n === 9);
}

function centersOk() {
  return FACES.every(f => state[f][4] === DEFAULT[f]);
}

function forceCenters() {
  FACES.forEach(f => { state[f][4] = DEFAULT[f]; });
}

function showError(msg) {
  const el = document.getElementById('errorMsg');
  el.textContent = msg;
  el.classList.add('show');
  setTimeout(() => el.classList.remove('show'), 8000);
}

function showInfo(on, text) {
  const el = document.getElementById('infoMsg');
  if (text) el.textContent = text;
  el.classList.toggle('show', !!on);
}

function showToast(msg, isWarn) {
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.style.background = isWarn ? '#ff9f0a' : '#30d158';
  t.style.color = isWarn ? '#111' : '#111';
  t.classList.add('show');
  setTimeout(() => t.classList.remove('show'), 2200);
}

function toFaceletString() {
  const order = ['U','R','F','D','L','B'];
  let s = '';
  order.forEach(f => {
    for (let i = 0; i < 9; i++) s += C2F[state[f][i]];
  });
  return s;
}

function rotateFaceCW(s, face) {
  // Rotaciona a face 90° CW e cicla as laterais no sentido do solver (cubejs/Kociemba)
  const f = s[face];
  s[face] = [f[6], f[3], f[0], f[7], f[4], f[1], f[8], f[5], f[2]];
  if (face === 'U') {
    // Library: F←R←B←L←F  (top rows)
    const t0 = s.F[0], t1 = s.F[1], t2 = s.F[2];
    s.F[0] = s.R[0]; s.F[1] = s.R[1]; s.F[2] = s.R[2];
    s.R[0] = s.B[0]; s.R[1] = s.B[1]; s.R[2] = s.B[2];
    s.B[0] = s.L[0]; s.B[1] = s.L[1]; s.B[2] = s.L[2];
    s.L[0] = t0; s.L[1] = t1; s.L[2] = t2;
  } else if (face === 'D') {
    // Library: R←F←L←B←R  (bottom rows)
    const t0 = s.F[6], t1 = s.F[7], t2 = s.F[8];
    s.F[6] = s.L[6]; s.F[7] = s.L[7]; s.F[8] = s.L[8];
    s.L[6] = s.B[6]; s.L[7] = s.B[7]; s.L[8] = s.B[8];
    s.B[6] = s.R[6]; s.B[7] = s.R[7]; s.B[8] = s.R[8];
    s.R[6] = t0; s.R[7] = t1; s.R[8] = t2;
  } else if (face === 'F') {
    const t0 = s.U[6], t1 = s.U[7], t2 = s.U[8];
    s.U[6] = s.L[8]; s.U[7] = s.L[5]; s.U[8] = s.L[2];
    s.L[2] = s.D[0]; s.L[5] = s.D[1]; s.L[8] = s.D[2];
    s.D[0] = s.R[6]; s.D[1] = s.R[3]; s.D[2] = s.R[0];
    s.R[0] = t0; s.R[3] = t1; s.R[6] = t2;
  } else if (face === 'B') {
    const t0 = s.U[0], t1 = s.U[1], t2 = s.U[2];
    s.U[0] = s.R[2]; s.U[1] = s.R[5]; s.U[2] = s.R[8];
    s.R[2] = s.D[8]; s.R[5] = s.D[7]; s.R[8] = s.D[6];
    s.D[6] = s.L[0]; s.D[7] = s.L[3]; s.D[8] = s.L[6];
    s.L[0] = t2; s.L[3] = t1; s.L[6] = t0;
  } else if (face === 'L') {
    const t0 = s.U[0], t1 = s.U[3], t2 = s.U[6];
    s.U[0] = s.B[8]; s.U[3] = s.B[5]; s.U[6] = s.B[2];
    s.B[2] = s.D[6]; s.B[5] = s.D[3]; s.B[8] = s.D[0];
    s.D[0] = s.F[0]; s.D[3] = s.F[3]; s.D[6] = s.F[6];
    s.F[0] = t0; s.F[3] = t1; s.F[6] = t2;
  } else if (face === 'R') {
    const t0 = s.U[2], t1 = s.U[5], t2 = s.U[8];
    s.U[2] = s.F[2]; s.U[5] = s.F[5]; s.U[8] = s.F[8];
    s.F[2] = s.D[2]; s.F[5] = s.D[5]; s.F[8] = s.D[8];
    s.D[2] = s.B[6]; s.D[5] = s.B[3]; s.D[8] = s.B[0];
    s.B[0] = t2; s.B[3] = t1; s.B[6] = t0;
  }
}

function applyMoveLogic(move) {
  // Usado no scramble / setup — pode aplicar 1, 2 ou 3 quartos de uma vez
  const face = move[0];
  let times = 1;
  if (move.includes('2')) times = 2;
  else if (move.includes("'")) times = 3;
  for (let i = 0; i < times; i++) rotateFaceCW(state, face);
}

/** Exatamente 1 quarto de volta — deve espelhar a animação do passo */
function applyQuarterTurn(move) {
  const face = move[0];
  if (!face || !'URFDLB'.includes(face)) return;
  if (move.includes("'")) {
    // 1 anti-horário = 3 horários
    rotateFaceCW(state, face);
    rotateFaceCW(state, face);
    rotateFaceCW(state, face);
  } else {
    // 1 horário (passos de R2 já vêm separados em R + R)
    rotateFaceCW(state, face);
  }
}

let solverQueue = [];
let loadingWatchdog = null;
function showLoading(on, title, sub) {
  const el = document.getElementById('loadingOverlay');
  if (!el) return;
  el.style.display = on ? 'flex' : 'none';
  const t = document.getElementById('loadingTitle');
  const s = document.getElementById('loadingSub');
  const c = document.getElementById('btnCancelLoad');
  if (t && title) t.textContent = title;
  if (s && sub) s.textContent = sub;
  if (c) c.style.display = on ? 'inline-block' : 'none';
  if (loadingWatchdog) { clearTimeout(loadingWatchdog); loadingWatchdog = null; }
  if (on) {
    loadingWatchdog = setTimeout(() => {
      // se ainda estiver na tela após 18s, libera UI
      const ov = document.getElementById('loadingOverlay');
      if (ov && ov.style.display !== 'none') {
        showLoading(false);
        solverBusy = false;
        const btn = document.getElementById('btnFinish');
        if (btn) btn.disabled = !FACES.every(f => visited[f]);
        showError('Demorou demais. O cubo pode ter cores impossíveis — revise o pintado e tente de novo.');
      }
    }, 18000);
  }
}
function cancelLoading() {
  showLoading(false);
  solverBusy = false;
  solverQueue = [];
  const btn = document.getElementById('btnFinish');
  if (btn) btn.disabled = !FACES.every(f => visited[f]);
  showToast('Cancelado');
}
function ensureSolver(cb) {
  if (solverReady) { cb(); return; }
  solverQueue.push(cb);
  if (solverBusy) return;
  solverBusy = true;
  showLoading(true, 'Preparando solver…', 'Primeira vez pode levar alguns segundos');
  setTimeout(() => {
    try {
      Cube.initSolver();
      solverReady = true;
      solverBusy = false;
      const q = solverQueue.splice(0);
      q.forEach(fn => { try { fn(); } catch (e) {
        console.error(e);
        showLoading(false);
        showError(String(e && e.message || e));
      } });
    } catch (e) {
      solverBusy = false;
      solverQueue = [];
      showLoading(false);
      const m = String(e && e.message || e);
      const btn = document.getElementById('btnFinish');
      if (btn) btn.disabled = !FACES.every(f => visited[f]);
      if (/400ms|infinite loop|Preview stopped/i.test(m)) {
        showError('Este preview online corta scripts longos. Abra o HTML no Chrome.');
      } else {
        showError('Falha ao iniciar solver: ' + m);
      }
    }
  }, 30);
}

// Busca solução curta (até 4 movimentos) por força bruta — o Kociemba embutido
// às vezes devolve sequências longas mesmo para cubos quase prontos.
function solveShort(cube, maxDepth) {
  if (cube.isSolved()) return [];
  const faces = ['U', 'R', 'F', 'D', 'L', 'B'];
  const mods = ['', '2', "'"];
  const all = [];
  faces.forEach(f => mods.forEach(m => all.push(f + m)));

  function search(c, depth, path, lastFace) {
    if (depth === 0) return null;
    for (let i = 0; i < all.length; i++) {
      const m = all[i];
      if (m[0] === lastFace) continue;
      const next = c.clone();
      next.move(m);
      const p = path.concat(m);
      if (next.isSolved()) return p;
      if (depth > 1) {
        const found = search(next, depth - 1, p, m[0]);
        if (found) return found;
      }
    }
    return null;
  }
  for (let d = 1; d <= maxDepth; d++) {
    const found = search(cube, d, [], '');
    if (found) return found;
  }
  return null;
}

function generateSolution() {
  forceCenters();
  const facelet = toFaceletString();
  let cube;
  try {
    cube = Cube.fromString(facelet);
  } catch (e) {
    throw new Error('Cubo inválido. Use exatamente 9 de cada cor e mantenha os centros oficiais.');
  }
  try {
    if (typeof cube.isSolved === 'function' && cube.isSolved()) return [];
  } catch (_) {}

  // Só Kociemba (sem busca bruta — evitava travar no celular)
  let alg = '';
  try {
    if (typeof cube.solveUpright === 'function') {
      alg = cube.solveUpright(16) || '';
    } else if (typeof cube.solve === 'function') {
      alg = cube.solve() || '';
    }
  } catch (e) {
    throw new Error('Não foi possível resolver. Cores podem formar um cubo impossível.');
  }
  if (!alg || !String(alg).trim()) {
    try { if (cube.isSolved()) return []; } catch(_){}
    throw new Error('Cubo impossível ou cores inconsistentes. Revise as faces (bordas/cantos não batem).');
  }
  return String(alg).trim().split(/\s+/).filter(Boolean);
}

/** 1 = lento … 5 = rápido → duração base em ms */
function getAnimMs(isDouble) {
  const spd = Math.max(1, Math.min(5, +(document.getElementById('speedRange')?.value || 2)));
  // base: velocidade 2 ≈ 900ms (¼) / 1200ms (½ volta)
  const base = isDouble ? 1400 : 1000;
  return Math.round(base * (6 - spd) / 4);
}

function getPlayInterval() {
  const spd = Math.max(1, Math.min(5, +(document.getElementById('speedRange')?.value || 2)));
  return Math.round(400 + (6 - spd) * 280);
}

/** 1 quarto horário lógico (mesmo ciclo do rotateFaceCW) nas coordenadas do cubie */
function logicalCW(face, x, y, z) {
  switch (face) {
    case 'U': return [-z, y, x];
    case 'D': return [z, y, -x];
    case 'F': return [y, -x, z];
    case 'B': return [-y, x, z];
    case 'R': return [x, z, -y];   // corrigido — antes estava invertido
    case 'L': return [x, -z, y];   // corrigido — antes estava invertido
    default: return [x, y, z];
  }
}

/** Aplica o giro CSS (mesmo eixo da animação) num ponto lógico e devolve coordenadas */
function cssRotatePoint(face, angleDeg, x, y, z) {
  let X = x, Y = -y, Z = z; // = translate3d(x,-y,z)
  const t = angleDeg * Math.PI / 180;
  const c = Math.cos(t), s = Math.sin(t);
  let X2 = X, Y2 = Y, Z2 = Z;
  if (face === 'U' || face === 'D') {
    X2 = X * c + Z * s;
    Z2 = -X * s + Z * c;
  } else if (face === 'R' || face === 'L') {
    Y2 = Y * c - Z * s;
    Z2 = Y * s + Z * c;
  } else {
    X2 = X * c - Y * s;
    Y2 = X * s + Y * c;
  }
  return [Math.round(X2), Math.round(-Y2), Math.round(Z2)];
}

/**
 * Pintura diz o destino; animação descobre o ângulo CSS que leva até lá.
 * "Ei, me leve até (tx,ty,tz)!" — testa os dois sentidos e fica com o que chega.
 */
function angleToTarget(face, x, y, z, tx, ty, tz, preferredDeg) {
  if (Math.abs(preferredDeg) === 180) return preferredDeg;
  const candidates = [preferredDeg, -preferredDeg];
  let best = preferredDeg, bestDist = Infinity;
  for (const a of candidates) {
    const [rx, ry, rz] = cssRotatePoint(face, a, x, y, z);
    const dist = (rx - tx) ** 2 + (ry - ty) ** 2 + (rz - tz) ** 2;
    if (dist < bestDist) {
      bestDist = dist;
      best = a;
    }
  }
  return best;
}

function animateMove(move, onDone) {
  if (isAnimating) return;
  isAnimating = true;
  updateStepUI();

  const m = String(move || 'U');
  const face = m[0];

  // Comando da pintura: quantos quartos e em qual sentido lógico
  let quartersCW = 1;
  if (m.includes('2')) quartersCW = 2;
  else if (m.includes("'")) quartersCW = -1;

  const preferredDeg = quartersCW * 90; // -90 | 90 | 180
  const isDouble = Math.abs(preferredDeg) === 180;
  const duration = getAnimMs(isDouble);

  const AXIS = {
    U: 'rotateY', D: 'rotateY',
    F: 'rotateZ', B: 'rotateZ',
    R: 'rotateX', L: 'rotateX'
  };
  const axis = AXIS[face] || 'rotateY';

  const layerOf = {
    U: c => +c.dataset.y === 1,
    D: c => +c.dataset.y === -1,
    F: c => +c.dataset.z === 1,
    B: c => +c.dataset.z === -1,
    R: c => +c.dataset.x === 1,
    L: c => +c.dataset.x === -1
  };

  // Cubie de referência na camada (não-centro) → destino lógico
  const refs = [
    [1, 1, 0], [0, 1, 1], [1, 0, 1], [1, 1, 1],
    [-1, 1, 0], [0, 1, -1], [1, 0, -1], [-1, 0, 1]
  ];
  let ref = null, target = null;
  for (const [x, y, z] of refs) {
    const onLayer =
      (face === 'U' && y === 1) || (face === 'D' && y === -1) ||
      (face === 'F' && z === 1) || (face === 'B' && z === -1) ||
      (face === 'R' && x === 1) || (face === 'L' && x === -1);
    if (!onLayer) continue;
    let cx = x, cy = y, cz = z;
    const steps = ((quartersCW % 4) + 4) % 4;
    for (let i = 0; i < steps; i++) {
      [cx, cy, cz] = logicalCW(face, cx, cy, cz);
    }
    ref = [x, y, z];
    target = [cx, cy, cz];
    break;
  }

  // "Me leve até o destino!" — ângulo CSS que chega no alvo da pintura
  let cssAngle = preferredDeg;
  if (ref && target) {
    cssAngle = angleToTarget(face, ref[0], ref[1], ref[2], target[0], target[1], target[2], preferredDeg);
  }

  let finished = false;
  const finish = () => {
    if (finished) return;
    finished = true;
    if (animTimer) { clearTimeout(animTimer); animTimer = null; }
    try {
      const piv = cubeEl.querySelector('.layer-pivot');
      if (piv) piv.remove();
      applyMoveLogic(m);
      forceCenters();
      render3D();
    } catch (e) {
      console.error(e);
      try { render3D(); } catch (_) {}
    }
    isAnimating = false;
    if (onDone) onDone();
    updateStepUI();
  };

  const cubies = [...cubeEl.querySelectorAll('.cubie')].filter(layerOf[face] || (() => false));
  if (!cubies.length) {
    finish();
    return;
  }

  const oldPivot = cubeEl.querySelector('.layer-pivot');
  if (oldPivot) oldPivot.remove();

  const pivot = document.createElement('div');
  pivot.className = 'layer-pivot';
  pivot.style.transition = `transform ${duration}ms cubic-bezier(.33,1,.68,1)`;
  cubeEl.appendChild(pivot);

  cubies.forEach(c => {
    c.style.transition = 'none';
    pivot.appendChild(c);
  });

  void pivot.offsetWidth;
  requestAnimationFrame(() => {
    requestAnimationFrame(() => {
      if (finished) return;
      pivot.style.transform = `${axis}(${cssAngle}deg)`;
    });
  });

  animTimer = setTimeout(finish, duration + 40);
}

function updateStepUI() {
  const total = solution.length;
  document.getElementById('stepText').textContent = `Passo ${currentStep} / ${total}`;
  document.getElementById('moveList').textContent =
    solution.slice(0, currentStep).join(' ') || '(início)';
  const next = solution[currentStep];
  document.getElementById('currentMove').textContent =
    next ? next : (currentStep >= total ? '✓ Cubo montado!' : '—');
  document.getElementById('btnPrev').disabled = currentStep === 0 || isAnimating;
  document.getElementById('btnNext').disabled = currentStep >= total || isAnimating;
  // progress bar + stats
  const pct = total ? Math.round((currentStep / total) * 100) : 0;
  const bar = document.getElementById('progBar');
  if (bar) bar.style.width = pct + '%';
  const sm = document.getElementById('statMoves');
  const sr = document.getElementById('statRemain');
  if (sm) sm.textContent = total + ' moves';
  if (sr) sr.textContent = 'restam ' + Math.max(0, total - currentStep);
}

function nextStep() {
  if (currentStep >= solution.length || isAnimating) return;
  const m = solution[currentStep];
  animateMove(m, () => {
    currentStep++;
    updateStepUI();
    if (currentStep >= solution.length) showToast('✓ Cubo resolvido!');
  });
}

function prevStep() {
  if (currentStep <= 0 || isAnimating) return;
  currentStep--;
  const m = solution[currentStep];
  // inverso: R2↔R2, R'↔R, R↔R'
  let inv;
  if (m.includes('2')) inv = m;
  else if (m.includes("'")) inv = m[0];
  else inv = m[0] + "'";
  animateMove(inv, () => updateStepUI());
}

function togglePlay() {
  if (playing) {
    clearInterval(playTimer);
    playing = false;
    document.getElementById('btnPlay').textContent = '▶ Play';
  } else {
    playing = true;
    document.getElementById('btnPlay').textContent = '❚❚ Pause';
    const tick = () => {
      if (!playing) return;
      if (currentStep >= solution.length) {
        togglePlay();
        return;
      }
      if (!isAnimating) nextStep();
      playTimer = setTimeout(tick, getPlayInterval());
    };
    playTimer = setTimeout(tick, 80);
  }
}

function goNextFace() {
  if (currentFaceIdx < 5) {
    currentFaceIdx++;
    renderPaintFace();
    lookAtFace(FACES[currentFaceIdx]);
  }
}
function goPrevFace() {
  if (currentFaceIdx > 0) {
    currentFaceIdx--;
    renderPaintFace();
    lookAtFace(FACES[currentFaceIdx]);
  }
}

function scrambleCube() {
  hardStop();
  // volta para modo pintura se estiver resolvendo
  if (document.getElementById('solveMode').style.display !== 'none') {
    showPaintMode();
  }
  // reconstrói estado a partir de resolvido + moves (evita estado sujo)
  FACES.forEach(f => { state[f] = Array(9).fill(DEFAULT[f]); });
  forceCenters();
  const faces = ['U','R','F','D','L','B'];
  const mods = ['', '2', "'"];
  let last = '';
  const n = 20 + Math.floor(Math.random() * 9);
  for (let i = 0; i < n; i++) {
    let f;
    do { f = faces[Math.floor(Math.random() * 6)]; } while (f === last);
    last = f;
    const mod = mods[Math.floor(Math.random() * 3)];
    applyMoveLogic(f + mod);
  }
  FACES.forEach(f => visited[f] = 1);
  document.getElementById('btnFinish').disabled = false;
  solution = [];
  currentStep = 0;
  render3D();
  renderPaintFace();
  lookAtFace('F');
  showToast('Cubo embaralhado!');
}

function tryFinish() {
  hardStop();
  forceCenters();
  const btn = document.getElementById('btnFinish');
  if (btn) btn.disabled = true;
  if (!isValidColorCount()) {
    showError('Cada cor deve aparecer exatamente 9 vezes. Verifique o que pintou.');
    if (btn) btn.disabled = false;
    return;
  }
  if (!centersOk()) {
    forceCenters();
    render3D();
    renderPaintFace();
  }
  ensureSolver(() => {
    showLoading(true, 'Calculando solução…', 'Quase lá');
    setTimeout(() => {
      try {
        solution = generateSolution();
        currentStep = 0;
        document.getElementById('paintMode').style.display = 'none';
        document.getElementById('solveMode').style.display = 'block';
        lookAtFace('F');
        updateStepUI();
        if (solution.length === 0) showToast('Já está resolvido!');
        else showToast(solution.length + ' movimentos');
      } catch (e) {
        hardStop();
        console.error(e);
        const msg = String(e && e.message || e);
        if (/timeout|slow|long/i.test(msg)) {
          showError('Solução demorou demais. Confira as cores (cubo pode estar impossível).');
        } else {
          showError(msg || 'Falha ao resolver. Verifique as cores pintadas.');
        }
        if (btn) btn.disabled = false;
      } finally {
        showLoading(false);
        // se voltou ao paint por erro, libera botão
        if (document.getElementById('paintMode').style.display !== 'none') {
          if (btn) btn.disabled = !FACES.every(f => visited[f]);
        }
      }
    }, 80);
  });
}

function showPaintMode() {
  hardStop();
  document.getElementById('solveMode').style.display = 'none';
  document.getElementById('paintMode').style.display = 'block';
  lookAtFace(FACES[currentFaceIdx]);
  renderPaintFace();
  render3D();
}

function resetAll() {
  hardStop();
  FACES.forEach(f => {
    state[f] = Array(9).fill(DEFAULT[f]);
    visited[f] = 0;
  });
  currentFaceIdx = 0;
  solution = [];
  currentStep = 0;
  document.getElementById('errorMsg').classList.remove('show');
  document.getElementById('solveMode').style.display = 'none';
  document.getElementById('paintMode').style.display = 'block';
  lookAtFace('F');
  render3D();
  renderPaintFace();
  showToast('Reset completo');
}

function copySolution() {
  if (!solution.length) {
    showToast('Nenhuma solução', true);
    return;
  }
  const text = solution.join(' ');
  if (navigator.clipboard && navigator.clipboard.writeText) {
    navigator.clipboard.writeText(text).then(() => showToast('Solução copiada!'));
  } else {
    // fallback
    const ta = document.createElement('textarea');
    ta.value = text;
    document.body.appendChild(ta);
    ta.select();
    document.execCommand('copy');
    document.body.removeChild(ta);
    showToast('Solução copiada!');
  }
}

// Eventos
document.querySelectorAll('.color-btn').forEach(btn => {
  btn.onclick = () => {
    document.querySelectorAll('.color-btn').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    selectedColor = btn.dataset.color;
  };
});
document.getElementById('btnNextFace').onclick = goNextFace;
document.getElementById('btnPrevFace').onclick = goPrevFace;
document.getElementById('btnFinish').onclick = tryFinish;
const _btnCL = document.getElementById('btnCancelLoad');
if (_btnCL) _btnCL.onclick = cancelLoading;
document.getElementById('btnReset').onclick = resetAll;
document.getElementById('btnReset2').onclick = resetAll;
document.getElementById('btnBackPaint').onclick = showPaintMode;
document.getElementById('btnNext').onclick = nextStep;
document.getElementById('btnPrev').onclick = prevStep;
document.getElementById('btnPlay').onclick = togglePlay;
document.getElementById('btnScramble').onclick = scrambleCube;
document.getElementById('btnCopy').onclick = copySolution;

// ===== Scanner de câmera (assistido) =====
let scanStream = null;
let scanLiveRAF = null;
let scanLiveGrid = null; // último 3x3 lido ao vivo
const COLOR_RGB = {
  w:[220,220,220], y:[255,214,10], r:[200,40,40],
  o:[240,140,40], b:[10,132,255], g:[48,209,88]
};
function closestColor(r,g,b) {
  const mx = Math.max(r,g,b), mn = Math.min(r,g,b);
  const bright = (r + g + b) / 3;
  const sat = mx - mn;
  // branco: bem claro / acinzentado (WebView costuma escurecer)
  if (bright >= 140 && sat <= 70) return 'w';
  if (bright >= 120 && sat <= 45) return 'w';
  if (bright >= 100 && sat <= 25) return 'w';
  // amarelo
  if (r > 160 && g > 140 && b < 110 && g > r * 0.75) return 'y';
  // vermelho vs laranja (laranja tem bem mais verde)
  if (r > 130 && r >= g && r >= b) {
    // razão G/R: laranja ~0.45–0.75; vermelho ~0.15–0.40
    const gr = g / (r + 1);
    if (gr >= 0.42 && g > 55 && b < 120) return 'o';
    if (gr < 0.42 || g < 60) return 'r';
    return (g > 100) ? 'o' : 'r';
  }
  // verde
  if (g > 100 && g > r * 1.1 && g > b * 0.9) return 'g';
  // azul
  if (b > 100 && b >= r && b >= g * 0.9) return 'b';
  // fallback por distância (com branco "preferido" se claro)
  let best = 'w', bestD = Infinity;
  for (const [k,v] of Object.entries(COLOR_RGB)) {
    let d = (r-v[0])**2 + (g-v[1])**2 + (b-v[2])**2;
    if (k === 'w' && bright > 120) d *= 0.55;
    if (d < bestD) { bestD = d; best = k; }
  }
  // se ficou muito escuro, não force uma cor errada — trate como branco sujo só se acinzentado
  if (bright < 45 && sat < 30) return best === 'b' ? 'w' : best;
  return best;
}
function updateScanFaceDots() {
  const el = document.getElementById('scanFaceDots');
  if (!el) return;
  el.innerHTML = '';
  const labels = {U:'U',D:'D',F:'F',B:'B',L:'L',R:'R'};
  FACES.forEach(f => {
    const d = document.createElement('div');
    const done = !!visited[f];
    d.style.cssText = 'width:30px;height:30px;border-radius:8px;border:2px solid '+(done?'#4f8cff':'#555')+';display:flex;align-items:center;justify-content:center;font-size:.65rem;font-weight:700;color:#111;background:'+COLORS[DEFAULT[f]]+';opacity:'+(done?'1':'0.45');
    d.textContent = labels[f];
    d.title = FACE_NAMES[f] || f;
    el.appendChild(d);
  });
}
function sampleFace3x3() {
  const video = document.getElementById('scanVideo');
  const canvas = document.getElementById('scanCanvas');
  if (!video || !video.videoWidth) return null;
  const w = video.videoWidth, h = video.videoHeight;
  canvas.width = w; canvas.height = h;
  const ctx = canvas.getContext('2d', { willReadFrequently: true });
  ctx.drawImage(video, 0, 0);
  // só a área central (~68%) — ignora fundo
  const x0 = w * 0.16, y0 = h * 0.16;
  const rw = w * 0.68, rh = h * 0.68;
  const cellW = rw / 3, cellH = rh / 3;
  const colors = [];
  for (let row = 0; row < 3; row++) {
    for (let col = 0; col < 3; col++) {
      const cx = Math.floor(x0 + col * cellW + cellW * 0.28);
      const cy = Math.floor(y0 + row * cellH + cellH * 0.28);
      const cw = Math.max(4, Math.floor(cellW * 0.44));
      const ch = Math.max(4, Math.floor(cellH * 0.44));
      const data = ctx.getImageData(cx, cy, cw, ch).data;
      let r = 0, g = 0, b = 0, n = 0;
      for (let i = 0; i < data.length; i += 8) {
        r += data[i]; g += data[i+1]; b += data[i+2]; n++;
      }
      colors.push(closestColor(Math.round(r/n), Math.round(g/n), Math.round(b/n)));
    }
  }
  return colors;
}
function paintLiveGrid(grid) {
  const cells = document.querySelectorAll('#scanGridOverlay .sg');
  if (!cells || cells.length < 9 || !grid) return;
  for (let i = 0; i < 9; i++) {
    const c = grid[i];
    const hex = COLORS[c] || '#888';
    cells[i].style.setProperty('background', hex, 'important');
    if (c === 'w') cells[i].style.setProperty('box-shadow', 'inset 0 0 0 1px #999', 'important');
    else cells[i].style.removeProperty('box-shadow');
  }
}
function scanLiveLoop() {
  if (!scanStream) return;
  const grid = sampleFace3x3();
  if (grid) {
    scanLiveGrid = grid;
    paintLiveGrid(grid);
    const names = {w:'Branco',y:'Amarelo',r:'Vermelho',o:'Laranja',b:'Azul',g:'Verde'};
    const center = grid[4];
    let faceHint = '';
    for (const f of FACES) {
      if (DEFAULT[f] === center) { faceHint = ' → ' + (FACE_NAMES[f] || f); break; }
    }
    const res = document.getElementById('scanResult');
    if (res) res.textContent = 'Centro: ' + names[center] + faceHint + ' · toque em Confirmar';
  }
  scanLiveRAF = requestAnimationFrame(scanLiveLoop);
}
async function openScanner() {
  const modal = document.getElementById('scanModal');
  const video = document.getElementById('scanVideo');
  modal.style.display = 'flex';
  document.getElementById('scanResult').textContent = 'Solicitando câmera…';
  updateScanFaceDots();
  try {
    scanStream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: { ideal: 'environment' }, width: { ideal: 720 }, height: { ideal: 720 } },
      audio: false
    });
    video.srcObject = scanStream;
    await video.play().catch(()=>{});
    document.getElementById('scanResult').textContent = 'Enquadre uma face e confirme';
    if (scanLiveRAF) cancelAnimationFrame(scanLiveRAF);
    scanLiveLoop();
  } catch (err) {
    document.getElementById('scanResult').textContent = 'Câmera não disponível: ' + (err.message || err);
  }
}
function closeScanner() {
  const modal = document.getElementById('scanModal');
  modal.style.display = 'none';
  if (scanLiveRAF) { cancelAnimationFrame(scanLiveRAF); scanLiveRAF = null; }
  if (scanStream) {
    scanStream.getTracks().forEach(t => t.stop());
    scanStream = null;
  }
  const video = document.getElementById('scanVideo');
  if (video) video.srcObject = null;
  scanLiveGrid = null;
}
function captureColor(ev) {
  if (ev) { try { ev.preventDefault(); ev.stopPropagation(); } catch(_){} }
  try {
    const grid = scanLiveGrid || sampleFace3x3();
    if (!grid) {
      document.getElementById('scanResult').textContent = 'Aguarde a câmera…';
      return;
    }
    const centerCol = grid[4];
    const names = {w:'Branco',y:'Amarelo',r:'Vermelho',o:'Laranja',b:'Azul',g:'Verde'};

    if (window._scanForCenter) {
      assignCenterColor(centerCol);
      document.getElementById('scanResult').textContent = 'Centro: ' + names[centerCol];
      showToast(names[centerCol] + ' capturado');
      setTimeout(closeScanner, 450);
      window._scanForCenter = false;
      return;
    }

    let face = null;
    for (const f of FACES) {
      if (DEFAULT[f] === centerCol) { face = f; break; }
    }
    if (!face) face = FACES[currentFaceIdx] || 'F';
    // grade da câmera (0–8) → state na mesma ordem (sem espelhar)
    for (let p = 0; p < 9; p++) {
      if (p === 4) state[face][4] = DEFAULT[face];
      else state[face][p] = grid[p];
    }
    visited[face] = 1;
    currentFaceIdx = Math.max(0, FACES.indexOf(face));
    lookAtFace(face);
    renderPaintFace();
    render3D();
    updateScanFaceDots();
    const doneCount = FACES.filter(f => visited[f]).length;
    document.getElementById('scanResult').textContent =
      '✓ ' + names[centerCol] + ' · ' + doneCount + '/6 · mostre outra face';
    showToast(names[centerCol] + ' pintado (' + doneCount + '/6)');
    if (doneCount >= 6) {
      document.getElementById('btnFinish').disabled = false;
      showToast('Todas as faces lidas!');
    }
  } catch (err) {
    console.error(err);
    document.getElementById('scanResult').textContent = 'Erro ao confirmar: ' + (err.message || err);
  }
}
document.getElementById('btnScan').onclick = openScanner;
document.getElementById('btnCloseScan').onclick = closeScanner;
const _btnCap = document.getElementById('btnCapture');
if (_btnCap) {
  _btnCap.onclick = captureColor;
  _btnCap.addEventListener('click', captureColor, true);
}

const speedRange = document.getElementById('speedRange');
const speedVal = document.getElementById('speedVal');
if (speedRange && speedVal) {
  const syncSpeed = () => { speedVal.textContent = speedRange.value + '×'; };
  speedRange.addEventListener('input', syncSpeed);
  syncSpeed();
}

// Drag do cubo (bloqueado durante animação de passo — só a fileira gira)
let lastX, lastY;
document.getElementById('scene').addEventListener('pointerdown', e => {
  if (isAnimating) return;
  dragging = true;
  lastX = e.clientX;
  lastY = e.clientY;
  cubeEl.style.transition = 'none';
});
document.getElementById('scene').addEventListener('pointermove', e => {
  if (!dragging || isAnimating) return;
  rotY += (e.clientX - lastX) * 0.45;
  rotX -= (e.clientY - lastY) * 0.45;
  rotX = Math.max(-120, Math.min(120, rotX));
  lastX = e.clientX;
  lastY = e.clientY;
  updateCubeView(false);
});
function endDrag() {
  if (!dragging) return;
  dragging = false;
  cubeEl.style.transition = 'transform .55s cubic-bezier(.4,0,.2,1)';
}
document.getElementById('scene').addEventListener('pointerup', endDrag);
document.getElementById('scene').addEventListener('pointercancel', endDrag);
document.getElementById('scene').addEventListener('pointerleave', endDrag);

// Atalhos de teclado
document.addEventListener('keydown', e => {
  if (e.target.tagName === 'INPUT') return;
  const solveVisible = document.getElementById('solveMode').style.display !== 'none';
  if (solveVisible) {
    if (e.key === 'ArrowRight' || e.key === ' ') {
      e.preventDefault();
      nextStep();
    } else if (e.key === 'ArrowLeft') {
      e.preventDefault();
      prevStep();
    } else if (e.key === 'Escape') {
      showPaintMode();
    } else if (e.key === 'p' || e.key === 'P') {
      togglePlay();
    }
  } else {
    if (e.key === 'ArrowRight') goNextFace();
    else if (e.key === 'ArrowLeft') goPrevFace();
  }
});

// Menu ⋮
const menuBtn = document.getElementById('menuBtn');
const menuPanel = document.getElementById('menuPanel');
if (menuBtn && menuPanel) {
  menuBtn.onclick = (e) => {
    e.stopPropagation();
    menuPanel.style.display = menuPanel.style.display === 'block' ? 'none' : 'block';
  };
  document.addEventListener('click', () => { menuPanel.style.display = 'none'; });
  menuPanel.onclick = (e) => e.stopPropagation();
}

// Splash OBLIRY (rápido)
(function runSplash() {
  const splash = document.getElementById('splash');
  const text = document.getElementById('splashText');
  if (!splash || !text) return;
  // começa branco
  setTimeout(() => {
    splash.style.background = '#0f1115'; // fade para preto
    setTimeout(() => {
      text.style.opacity = '1'; // mostra OBLIRY
      setTimeout(() => {
        text.style.opacity = '0'; // some texto
        setTimeout(() => {
          splash.style.opacity = '0'; // some tela
          setTimeout(() => {
            splash.remove();
          }, 400);
        }, 350);
      }, 700);
    }, 350);
  }, 80);
})();

// ===== Modo inicial: padrão vs escolher cores (guiado) =====
let selectedScheme = 'standard';
let centerPickColor = 'w';
let centerStep = 0; // 0..5
const centerFacesOrder = ['U','F','R','D','L','B'];
const centerStepNames = {
  U: 'CIMA (em cima do cubo)',
  F: 'FRENTE (olhando para você)',
  R: 'DIREITA',
  D: 'BAIXO (embaixo do cubo)',
  L: 'ESQUERDA',
  B: 'TRÁS (atrás do cubo)'
};
let centersDone = {U:false,F:false,R:false,D:false,L:false,B:false};

function updateCenterStepUI() {
  const f = centerFacesOrder[centerStep];
  const label = document.getElementById('centerStepLabel');
  if (label) label.textContent = `Passo ${centerStep+1}/6 — Mostre a face de ${centerStepNames[f]}`;
  // dots clicáveis
  const dots = document.getElementById('centerDots');
  if (dots) {
    dots.innerHTML = '';
    centerFacesOrder.forEach((face, i) => {
      const d = document.createElement('div');
      const col = DEFAULT[face] || 'w';
      // sempre mostra a cor atual do centro no cubo 3D
      d.style.cssText = 'width:32px;height:32px;border-radius:8px;border:2px solid #555;cursor:pointer;background:' + COLORS[col] + ';';
      if (centersDone[face]) {
        d.style.borderColor = '#4f8cff';
        d.style.opacity = '1';
      } else {
        d.style.opacity = '0.55';
      }
      if (i === centerStep) {
        d.style.borderColor = '#fff';
        d.style.boxShadow = '0 0 0 2px #4f8cff';
        d.style.opacity = '1';
      }
      d.title = centerStepNames[face] + (centersDone[face] ? ' ✓' : ' (toque ou capture)');
      d.onclick = () => {
        centerStep = i;
        updateCenterStepUI();
      };
      dots.appendChild(d);
    });
  }
  // botões
  const btnNext = document.getElementById('btnNextCenter');
  const btnPrev = document.getElementById('btnPrevCenter');
  const btnStart = document.getElementById('btnStartApp');
  if (btnNext) btnNext.disabled = centerStep >= 5 || !centersDone[f];
  if (btnPrev) btnPrev.disabled = centerStep === 0;
  const allDone = centerFacesOrder.every(fc => centersDone[fc]);
  if (btnStart) {
    btnStart.disabled = selectedScheme === 'custom' && !allDone;
    btnStart.textContent = allDone && selectedScheme === 'custom' ? 'Começar a pintar' : 'Começar a pintar';
  }
  // vira o cubo 3D para a face atual
  lookAtFace(f);
  render3D();
}

function assignCenterColor(col) {
  const f = centerFacesOrder[centerStep];
  // se a cor já estava em outra face, troca (evita dois centros iguais)
  for (const other of FACES) {
    if (other !== f && DEFAULT[other] === col) {
      const old = DEFAULT[f];
      DEFAULT[other] = old;
      F2C[other] = old;
      for (let i = 0; i < 9; i++) state[other][i] = old;
    }
  }
  DEFAULT[f] = col;
  F2C[f] = col;
  C2F = {};
  Object.keys(F2C).forEach(face => { C2F[F2C[face]] = face; });
  // pinta só o centro dessa face na cor capturada; resto acompanha o centro
  for (let i = 0; i < 9; i++) state[f][i] = col;
  centersDone[f] = true;
  forceCenters();
  updateCenterStepUI();
  showToast('Centro ' + ({w:'Branco',y:'Amarelo',r:'Vermelho',o:'Laranja',b:'Azul',g:'Verde'}[col] || col) + ' na face ' + f);
  if (centerStep < 5) {
    setTimeout(() => {
      centerStep++;
      updateCenterStepUI();
    }, 450);
  }
}

function buildCenterPalette() {
  const pal = document.getElementById('centerPalette');
  if (!pal) return;
  pal.innerHTML = '';
  ['w','y','r','o','b','g'].forEach(c => {
    const b = document.createElement('div');
    b.className = 'color-btn ' + c + (c === centerPickColor ? ' active' : '');
    b.dataset.color = c;
    b.onclick = () => {
      centerPickColor = c;
      pal.querySelectorAll('.color-btn').forEach(x => x.classList.remove('active'));
      b.classList.add('active');
      assignCenterColor(c); // aplica na face atual
    };
    pal.appendChild(b);
  });
}

document.getElementById('optStandard').onclick = () => {
  selectedScheme = 'standard';
  document.getElementById('optStandard').style.borderColor = '#4f8cff';
  document.getElementById('optCustom').style.borderColor = '#333';
  document.getElementById('customCenters').style.display = 'none';
  document.getElementById('btnStartApp').disabled = false;
  document.getElementById('btnStartApp').textContent = 'Começar';
  Object.assign(DEFAULT, {U:'w', D:'y', F:'g', B:'b', L:'o', R:'r'});
  Object.assign(F2C, {U:'w', D:'y', R:'r', L:'o', F:'g', B:'b'});
  Object.assign(C2F, {w:'U', y:'D', r:'R', o:'L', g:'F', b:'B'});
  FACES.forEach(f => { state[f] = Array(9).fill(DEFAULT[f]); });
  render3D();
};

document.getElementById('optCustom').onclick = () => {
  selectedScheme = 'custom';
  document.getElementById('optCustom').style.borderColor = '#4f8cff';
  document.getElementById('optStandard').style.borderColor = '#333';
  document.getElementById('customCenters').style.display = 'block';
  centerStep = 0;
  // começa cinza — colorido só depois de capturar cada lado
  centersDone = {U:false,F:false,R:false,D:false,L:false,B:false};
  buildCenterPalette();
  updateCenterStepUI();
  const btnStart = document.getElementById('btnStartApp');
  if (btnStart) {
    btnStart.disabled = true;
    btnStart.textContent = 'Começar a pintar';
  }
};

document.getElementById('btnNextCenter').onclick = () => {
  if (centerStep < 5) {
    centerStep++;
    updateCenterStepUI();
  } else {
    updateCenterStepUI();
    showToast('Todos os centros definidos!');
  }
};
document.getElementById('btnPrevCenter').onclick = () => {
  if (centerStep > 0) {
    centerStep--;
    updateCenterStepUI();
  }
};

document.getElementById('btnScanCenter').onclick = () => {
  // reutiliza o modal de câmera, mas em modo "centro"
  window._scanForCenter = true;
  openScanner();
};

document.getElementById('btnStartApp').onclick = () => {
  forceCenters();
  FACES.forEach(f => { state[f] = Array(9).fill(DEFAULT[f]); visited[f] = 0; });
  currentFaceIdx = 0;
  document.getElementById('startMode').style.display = 'none';
  document.getElementById('paintMode').style.display = 'block';
  lookAtFace('F');
  render3D();
  renderPaintFace();
  const colorNames = {w:'Branco',y:'Amarelo',r:'Vermelho',o:'Laranja',b:'Azul',g:'Verde'};
  FACE_NAMES.F = 'Frente (' + colorNames[DEFAULT.F] + ')';
  FACE_NAMES.R = 'Direita (' + colorNames[DEFAULT.R] + ')';
  FACE_NAMES.B = 'Trás (' + colorNames[DEFAULT.B] + ')';
  FACE_NAMES.L = 'Esquerda (' + colorNames[DEFAULT.L] + ')';
  FACE_NAMES.U = 'Cima (' + colorNames[DEFAULT.U] + ')';
  FACE_NAMES.D = 'Baixo (' + colorNames[DEFAULT.D] + ')';
};

// Init
forceCenters();
lookAtFace('F');
render3D();
showInfo(false);
// prepara solver em segundo plano (evita espera longa no 1º Resolver)
setTimeout(() => {
  try {
    if (!solverReady && typeof Cube !== 'undefined' && Cube.initSolver) {
      Cube.initSolver();
      solverReady = true;
    }
  } catch (e) { console.warn('pre-init solver', e); }
}, 1200);
</script>
<div class="toast" id="toast"></div>
</body>
</html>
