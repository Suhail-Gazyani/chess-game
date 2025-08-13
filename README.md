# chess-game
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>HTML Chess — Full Rules, No Libraries</title>
  <style>
    :root {
      --bg: #0f172a;          /* slate-900 */
      --panel: #111827;       /* gray-900 */
      --text: #e5e7eb;        /* gray-200 */
      --sub: #94a3b8;         /* slate-400 */
      --green: #10b981;       /* emerald-500 */
      --accent: #38bdf8;      /* sky-400 */
      --warn: #f59e0b;        /* amber-500 */
      --red: #ef4444;         /* red-500 */
      --light: #f0d9b5;       /* board light */
      --dark: #b58863;        /* board dark  */
      --highlight: #22c55e55; /* translucent green */
      --hover: #3b82f633;     /* light blue */
      --sel: #22c55eaa;
      --check: #ef444455;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0; min-height: 100vh; background: radial-gradient(1200px 800px at 70% -10%, #1f2937, var(--bg));
      color: var(--text); font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, 'Helvetica Neue', Arial;
      display: grid; place-items: center; padding: 24px;
    }
    .app { width: min(1100px, 98vw); display: grid; grid-template-columns: 1fr 320px; gap: 18px; }
    @media (max-width: 980px) { .app { grid-template-columns: 1fr; } }

    .board-wrap { background: #0b1220aa; backdrop-filter: blur(6px); padding: 16px; border-radius: 20px; box-shadow: 0 10px 30px #0008; }
    .board {
      width: min(78vw, 720px); height: min(78vw, 720px);
      max-width: 720px; max-height: 720px;
      aspect-ratio: 1;
      display: grid; grid-template-columns: repeat(8, 1fr); grid-template-rows: repeat(8, 1fr);
      border-radius: 16px; overflow: hidden; box-shadow: inset 0 0 0 2px #0006, 0 12px 40px #0008;
      user-select: none;
      touch-action: none;
      position: relative;
    }
    .sq { display: grid; place-items: center; font-size: clamp(28px, 6.2vw, 64px); cursor: pointer; position: relative; }
    .sq.light { background: var(--light); }
    .sq.dark { background: var(--dark); }
    .sq:hover { outline: 2px solid var(--hover); outline-offset: -2px; }
    .sq.selected { box-shadow: inset 0 0 0 4px var(--sel); }
    .sq.target { box-shadow: inset 0 0 0 6px var(--highlight); }
    .sq.check { box-shadow: inset 0 0 0 6px var(--check); }
    .coord { position: absolute; font-size: 11px; color: #0009; left: 6px; bottom: 6px; }

    .piece { pointer-events: none; transform: translateZ(0); filter: drop-shadow(0 3px 2px #0006); }

    .side { background: var(--panel); border-radius: 20px; padding: 16px; box-shadow: 0 10px 30px #0008; display: grid; gap: 12px; align-content: start; }
    .row { display: flex; gap: 8px; align-items: center; flex-wrap: wrap; }
    .btn { background: #0b1220; color: var(--text); border: 1px solid #1f2937; padding: 10px 14px; border-radius: 12px; cursor: pointer; font-weight: 600; }
    .btn:hover { border-color: var(--accent); background: #0b1626; }
    .btn.danger { border-color: #7f1d1d; }
    .tag { font-size: 12px; padding: 4px 8px; border: 1px solid #334155; border-radius: 999px; color: var(--sub); }

    .status { background: #0b1220; padding: 12px; border-radius: 12px; border: 1px solid #1f2937; font-size: 14px; }
    .moves { background: #0b1220; padding: 12px; border-radius: 12px; border: 1px solid #1f2937; height: 380px; overflow: auto; font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", monospace; }
    .moves ol { margin: 0; padding-left: 18px; }

    dialog[open] { border: none; border-radius: 16px; padding: 0; background: #0b1220; color: var(--text); box-shadow: 0 20px 60px #000a; }
    .modal { padding: 16px; min-width: 240px; }
    .modal h3 { margin: 0 0 10px; }
    .promo-row { display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; }
    .promo-row button { font-size: 28px; padding: 12px; border-radius: 12px; border: 1px solid #1f2937; background: #0b1220; cursor: pointer; }
    .promo-row button:hover { border-color: var(--accent); }

    .footer { text-align: center; font-size: 12px; color: var(--sub); opacity: .8; }
  </style>
</head>
<body>
  <div class="app">
    <div class="board-wrap">
      <div class="board" id="board"></div>
    </div>
    <aside class="side">
      <div class="row" style="justify-content: space-between">
        <div>
          <div style="font-size:20px; font-weight:800; letter-spacing:.5px">HTML Chess</div>
          <div class="tag" id="turnTag">White to move</div>
        </div>
        <button class="btn" id="flipBtn" title="Flip board">Flip</button>
      </div>

      <div class="row">
        <button class="btn" id="undoBtn">Undo</button>
        <button class="btn" id="resetBtn">Reset</button>
        <button class="btn danger" id="fenBtn">FEN</button>
      </div>

      <div class="status" id="status">Ready. Standard chess with legal moves, castling, en passant, and promotion. Click a piece, then a square.</div>
      <div class="moves" id="moves"><ol id="movesList"></ol></div>
      <div class="footer">No libraries. All rules implemented in ~500 lines JS.</div>
    </aside>
  </div>

  <dialog id="promoDialog">
    <div class="modal">
      <h3>Promote pawn to:</h3>
      <div class="promo-row">
        <button data-piece="q">♛</button>
        <button data-piece="r">♜</button>
        <button data-piece="b">♝</button>
        <button data-piece="n">♞</button>
      </div>
    </div>
  </dialog>

  <dialog id="fenDialog">
    <div class="modal">
      <h3>Current FEN</h3>
      <div class="row" style="margin-top:6px">
        <input id="fenInput" style="flex:1; padding:8px 10px; border-radius:10px; border:1px solid #334155; background:#0b1220; color:var(--text)" />
        <button class="btn" id="fenCopy">Copy</button>
      </div>
    </div>
  </dialog>

  <script>
    // --- Utilities ----------------------------------------------------------
    const files = ['a','b','c','d','e','f','g','h'];
    const startFEN = 'rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1';

    function clone(obj){ return structuredClone(obj); }
    function idxToSquare(r,c){ return files[c] + (8 - r); }
    function squareToIdx(s){ const f = files.indexOf(s[0]); const r = 8 - parseInt(s[1]); return [r,f]; }
    function inBounds(r,c){ return r>=0 && r<8 && c>=0 && c<8; }

    // --- State --------------------------------------------------------------
    const initState = () => ({
      board: parseFEN(startFEN).board,
      whiteToMove: true,
      castling: { K:true, Q:true, k:true, q:true },
      ep: null, // en passant target square like 'e3'
      halfmove: 0, fullmove: 1,
      history: [],
      orientation: 'w'
    });

    let S = initState();

    // --- FEN helpers --------------------------------------------------------
    function parseFEN(fen){
      const [piecePlacement, active, castling, ep, half, full] = fen.split(' ');
      const rows = piecePlacement.split('/');
      const board = Array.from({length:8}, ()=>Array(8).fill(null));
      rows.forEach((row, rIdx)=>{
        let c=0; for(const ch of row){
          if(/\d/.test(ch)){ c += parseInt(ch); }
          else { const color = ch === ch.toUpperCase() ? 'w' : 'b'; const type = ch.toLowerCase(); board[rIdx][c++] = {type, color}; }
        }
      });
      return {board, active, castling, ep: ep==='-'?null:ep, half: +half, full: +full};
    }

    function toFEN(state){
      const rows = state.board.map(row=>{
        let s='', cnt=0; for(const sq of row){
          if(!sq){ cnt++; }
          else { if(cnt){ s+=cnt; cnt=0; } s += sq.color==='w'? sq.type.toUpperCase() : sq.type; }
        }
        if(cnt) s+=cnt; return s;
      }).join('/');
      const active = state.whiteToMove ? 'w' : 'b';
      const c = (state.castling.K?'K':'')+(state.castling.Q?'Q':'')+(state.castling.k?'k':'')+(state.castling.q?'q':'');
      const cast = c || '-';
      return `${rows} ${active} ${cast} ${state.ep||'-'} ${state.halfmove} ${state.fullmove}`;
    }

    // --- Rendering ----------------------------------------------------------
    const boardEl = document.getElementById('board');
    const movesEl = document.getElementById('movesList');
    const statusEl = document.getElementById('status');
    const turnTag = document.getElementById('turnTag');

    function render(){
      boardEl.innerHTML='';
      // Determine king-in-check highlight
      const checkColor = isInCheck(S, S.whiteToMove?'w':'b') ? (S.whiteToMove?'w':'b') : null;

      for(let r=0; r<8; r++){
        for(let c=0; c<8; c++){
          const isLight = (r+c)%2===0;
          const sq = document.createElement('div');
          sq.className = `sq ${isLight?'light':'dark'}`;
          sq.dataset.r=r; sq.dataset.c=c; sq.id=`sq-${r}-${c}`;
          if((S.orientation==='w' && r===7) || (S.orientation==='b' && r===0)){
            // no coords on every square; just corner hint
            const coord = document.createElement('div'); coord.className='coord';
            coord.textContent = idxToSquare(r,c);
            sq.appendChild(coord);
          }

          const piece = pieceAt(S.board, r, c);
          if(piece){
            const span = document.createElement('div');
            span.className='piece';
            span.textContent = glyph(piece);
            span.style.transform = 'translateZ(0)';
            sq.appendChild(span);
            if(checkColor && piece.type==='k' && piece.color===checkColor){ sq.classList.add('check'); }
          }

          boardEl.appendChild(sq);
        }
      }
      updateTurnTag();
      attachHandlers();
      updateMoveList();
    }

    function updateTurnTag(){
      turnTag.textContent = S.whiteToMove ? 'White to move' : 'Black to move';
      turnTag.style.borderColor = S.whiteToMove ? 'var(--green)' : 'var(--accent)';
    }

    function updateMoveList(){
      movesEl.innerHTML='';
      let idx=0; for(const m of S.history){
        if(m.color==='w'){
          const li = document.createElement('li');
          li.textContent = `${++idx}. ${m.notation}`;
          if(m.result) li.textContent += ` ${m.result}`;
          if(m.domRef){ li.appendChild(m.domRef); }
          movesEl.appendChild(li);
        } else {
          const li = movesEl.lastElementChild || document.createElement('li');
          if(!li.parentElement) movesEl.appendChild(li);
          li.textContent += `   ${m.notation}`;
          if(m.result) li.textContent += ` ${m.result}`;
          if(m.domRef){ li.appendChild(m.domRef); }
        }
      }
      movesEl.parentElement.scrollTop = movesEl.parentElement.scrollHeight;
    }

    function glyph(p){
      const m = { k:['♔','♚'], q:['♕','♛'], r:['♖','♜'], b:['♗','♝'], n:['♘','♞'], p:['♙','♟'] };
      return p.color==='w' ? m[p.type][0] : m[p.type][1];
    }

    function pieceAt(board, r, c){ return inBounds(r,c) ? board[r][c] : null; }

    // --- Input handling -----------------------------------------------------
    let selected = null; // {r,c, moves:[{r,c, flags}...]}

    function attachHandlers(){
      for(const el of boardEl.children){ el.addEventListener('click', onSquareClick); }
    }

    function clearHighlights(){ for(const el of boardEl.children){ el.classList.remove('selected','target'); } }

    function onSquareClick(e){
      const el = e.currentTarget; const r=+el.dataset.r, c=+el.dataset.c;
      const sqPiece = pieceAt(S.board, r, c);
      if(selected){
        // If clicked same color piece, reselect
        if(sqPiece && sqPiece.color === (S.whiteToMove?'w':'b')){
          selectSquare(r,c); return;
        }
        // Try to move
        const legal = selected.moves.find(m=>m.r===r && m.c===c);
        if(legal){
          doMove(selected.r, selected.c, r, c, legal);
          afterMove();
        }
        selected = null; clearHighlights();
      } else {
        // New selection
        if(!sqPiece || sqPiece.color !== (S.whiteToMove?'w':'b')) return;
        selectSquare(r,c);
      }
    }

    function selectSquare(r,c){
      selected = {r,c, moves: legalMovesFrom(S, r, c)};
      clearHighlights();
      document.getElementById(`sq-${r}-${c}`).classList.add('selected');
      for(const m of selected.moves){ document.getElementById(`sq-${m.r}-${m.c}`).classList.add('target'); }
    }

    // --- Move generation (full legal) --------------------------------------
    function legalMovesFrom(state, r, c){
      const piece = pieceAt(state.board, r, c); if(!piece) return [];
      const pseudo = generatePseudoMoves(state, r, c, piece);
      const legal = [];
      for(const m of pseudo){
        const st = makeMove(clone(state), r, c, m.r, m.c, m);
        if(!isInCheck(st, piece.color)) legal.push(m);
      }
      return legal;
    }

    function generatePseudoMoves(state, r, c, piece){
      const moves = []; const enemy = piece.color==='w'?'b':'w';
      const add = (rr,cc, flags={})=>{ if(!inBounds(rr,cc)) return; const t = pieceAt(state.board, rr, cc); if(!t){ moves.push({r:rr,c:cc, ...flags}); return true; } if(t.color!==piece.color){ moves.push({r:rr,c:cc, capture:true, ...flags}); } return false; };
      const slide = dirs=>{ for(const [dr,dc] of dirs){ let rr=r+dr, cc=c+dc; while(inBounds(rr,cc)){ if(add(rr,cc)===false) break; if(pieceAt(state.board, rr,cc)) break; rr+=dr; cc+=dc; } } };

      switch(piece.type){
        case 'p': {
          const dir = piece.color==='w' ? -1 : 1;
          const startRow = piece.color==='w' ? 6 : 1;
          // one forward
          if(!pieceAt(state.board, r+dir, c)){
            // promotion
            if(r+dir === (piece.color==='w'?0:7)) moves.push({r:r+dir,c, promo:true});
            else moves.push({r:r+dir,c});
            // two forward
            if(r===startRow && !pieceAt(state.board, r+2*dir, c)){
              moves.push({r:r+2*dir, c, dbl:true});
            }
          }
          // captures
          for(const dc of [-1,1]){
            const rr=r+dir, cc=c+dc; if(!inBounds(rr,cc)) continue;
            const t=pieceAt(state.board, rr,cc);
            if(t && t.color!==piece.color){
              if(rr === (piece.color==='w'?0:7)) moves.push({r:rr,c:cc, capture:true, promo:true});
              else moves.push({r:rr,c:cc, capture:true});
            }
          }
          // en passant
          if(state.ep){ const [er,ec] = squareToIdx(state.ep); if(er===r+dir && Math.abs(ec-c)===1){ moves.push({r:er,c:ec, ep:true, capture:true}); } }
          break;
        }
        case 'n': {
          const offs = [[-2,-1],[-2,1],[-1,-2],[-1,2],[1,-2],[1,2],[2,-1],[2,1]];
          for(const [dr,dc] of offs){ add(r+dr,c+dc); }
          break;
        }
        case 'b': slide([[-1,-1],[-1,1],[1,-1],[1,1]]); break;
        case 'r': slide([[-1,0],[1,0],[0,-1],[0,1]]); break;
        case 'q': slide([[-1,-1],[-1,1],[1,-1],[1,1],[-1,0],[1,0],[0,-1],[0,1]]); break;
        case 'k': {
          for(let dr=-1; dr<=1; dr++) for(let dc=-1; dc<=1; dc++) if(dr||dc) add(r+dr,c+dc);
          // castling
          if(piece.color==='w' && r===6 && c===4 && !isInCheck(state,'w')){
            if(state.castling.K && !pieceAt(state.board,6,5) && !pieceAt(state.board,6,6) && !isSquareAttacked(state,6,5,'b') && !isSquareAttacked(state,6,6,'b')) moves.push({r:6,c:6, castle:'K'});
            if(state.castling.Q && !pieceAt(state.board,6,3) && !pieceAt(state.board,6,2) && !pieceAt(state.board,6,1) && !isSquareAttacked(state,6,3,'b') && !isSquareAttacked(state,6,2,'b')) moves.push({r:6,c:2, castle:'Q'});
          }
          if(piece.color==='b' && r===0 && c===4 && !isInCheck(state,'b')){
            if(state.castling.k && !pieceAt(state.board,0,5) && !pieceAt(state.board,0,6) && !isSquareAttacked(state,0,5,'w') && !isSquareAttacked(state,0,6,'w')) moves.push({r:0,c:6, castle:'k'});
            if(state.castling.q && !pieceAt(state.board,0,3) && !pieceAt(state.board,0,2) && !pieceAt(state.board,0,1) && !isSquareAttacked(state,0,3,'w') && !isSquareAttacked(state,0,2,'w')) moves.push({r:0,c:2, castle:'q'});
          }
          break;
        }
      }
      return moves;
    }

    function isInCheck(state, color){
      // find king
      let kr=-1,kc=-1; for(let r=0;r<8;r++) for(let c=0;c<8;c++){ const p=state.board[r][c]; if(p && p.type==='k' && p.color===color){ kr=r; kc=c; }}
      return isSquareAttacked(state, kr, kc, color==='w'?'b':'w');
    }

    function isSquareAttacked(state, r, c, byColor){
      // pawns
      const dir = byColor==='w' ? -1 : 1; for(const dc of [-1,1]){ const rr=r+dir, cc=c+dc; if(inBounds(rr,cc)){ const p=pieceAt(state.board, rr,cc); if(p && p.color===byColor && p.type==='p') return true; }}
      // knights
      for(const [dr,dc] of [[-2,-1],[-2,1],[-1,-2],[-1,2],[1,-2],[1,2],[2,-1],[2,1]]){ const rr=r+dr, cc=c+dc; if(inBounds(rr,cc)){ const p=pieceAt(state.board, rr,cc); if(p && p.color===byColor && p.type==='n') return true; }}
      // bishops/queens (diagonals)
      for(const [dr,dc] of [[-1,-1],[-1,1],[1,-1],[1,1]]){ let rr=r+dr, cc=c+dc; while(inBounds(rr,cc)){ const p=pieceAt(state.board, rr,cc); if(p){ if(p.color===byColor && (p.type==='b'||p.type==='q')) return true; else break; } rr+=dr; cc+=dc; }}
      // rooks/queens (straights)
      for(const [dr,dc] of [[-1,0],[1,0],[0,-1],[0,1]]){ let rr=r+dr, cc=c+dc; while(inBounds(rr,cc)){ const p=pieceAt(state.board, rr,cc); if(p){ if(p.color===byColor && (p.type==='r'||p.type==='q')) return true; else break; } rr+=dr; cc+=dc; }}
      // king
      for(let dr=-1; dr<=1; dr++) for(let dc=-1; dc<=1; dc++) if(dr||dc){ const rr=r+dr, cc=c+dc; if(inBounds(rr,cc)){ const p=pieceAt(state.board, rr,cc); if(p && p.color===byColor && p.type==='k') return true; }}
      return false;
    }

    // --- Make/undo moves ----------------------------------------------------
    function makeMove(state, r1,c1, r2,c2, move){
      const b = state.board; const piece = b[r1][c1];
      // halfmove clock
      if(piece.type==='p' || move.capture) state.halfmove = 0; else state.halfmove++;
      // en passant capture
      if(move.ep){ const dir = piece.color==='w'?-1:1; b[r2 - dir][c2] = null; }
      // castle rook move
      if(move.castle){ if(move.castle==='K'){ b[r1][7]=null; b[r1][5]={type:'r', color:piece.color}; } if(move.castle==='Q'){ b[r1][0]=null; b[r1][3]={type:'r', color:piece.color}; } if(move.castle==='k'){ b[r1][7]=null; b[r1][5]={type:'r', color:piece.color}; } if(move.castle==='q'){ b[r1][0]=null; b[r1][3]={type:'r', color:piece.color}; } }
      // move piece
      b[r2][c2] = piece; b[r1][c1] = null;
      // promotion placeholder
      if(move.promo && move.promoteTo){ b[r2][c2] = {type: move.promoteTo, color: piece.color}; }
      // castling rights updates
      if(piece.type==='k'){
        if(piece.color==='w'){ state.castling.K=false; state.castling.Q=false; }
        else { state.castling.k=false; state.castling.q=false; }
      }
      if(piece.type==='r'){
        if(r1===7 && c1===0) state.castling.Q=false; if(r1===7 && c1===7) state.castling.K=false;
        if(r1===0 && c1===0) state.castling.q=false; if(r1===0 && c1===7) state.castling.k=false;
      }
      // if rook captured, also update rights
      if(move.capture){ if(r2===7 && c2===0) state.castling.Q=false; if(r2===7 && c2===7) state.castling.K=false; if(r2===0 && c2===0) state.castling.q=false; if(r2===0 && c2===7) state.castling.k=false; }
      // en passant target
      state.ep = null; if(piece.type==='p' && move.dbl){ const sq = idxToSquare((r1+r2)/2, c1); state.ep = sq; }

      // turn & move counters
      state.whiteToMove = !state.whiteToMove; if(!state.whiteToMove) state.fullmove++;
      return state;
    }

    function doMove(r1,c1,r2,c2, move){
      // Handle promotion with dialog
      if(move.promo && !move.promoteTo){
        const dialog = document.getElementById('promoDialog'); dialog.returnValue=''; dialog.showModal();
        return new Promise(resolve=>{
          const handler = (e)=>{
            if(e.target.matches('button[data-piece]')){
              move.promoteTo = e.target.dataset.piece; dialog.close();
              document.removeEventListener('click', handler, true);
              finalize(); resolve();
            }
          };
          document.addEventListener('click', handler, true);
        });
      } else { finalize(); return Promise.resolve(); }

      function finalize(){
        const before = toFEN(S);
        const prev = clone(S);
        makeMove(S, r1,c1,r2,c2, move);
        // Save history item
        const notation = makeNotation(prev, r1,c1,r2,c2, move);
        S.history.push({
          fenBefore: before,
          move: {from: idxToSquare(r1,c1), to: idxToSquare(r2,c2), ...move},
          notation,
          color: prev.whiteToMove?'w':'b'
        });
      }
    }

    function afterMove(){
      render();
      const colorJustMoved = S.whiteToMove ? 'b' : 'w';
      if(isInCheck(S, S.whiteToMove? 'w':'b')){
        // Opponent is in check now
      }
      const legalAny = hasAnyLegalMoves(S, S.whiteToMove? 'w':'b');
      const oppColor = S.whiteToMove? 'w':'b';
      if(!legalAny){
        if(isInCheck(S, oppColor)){
          const res = oppColor==='w' ? '0-1' : '1-0';
          statusEl.textContent = `Checkmate! ${colorJustMoved==='w'?'White':'Black'} wins.`;
          S.history[S.history.length-1].result = res;
        } else {
          statusEl.textContent = 'Stalemate — draw.';
          S.history[S.history.length-1].result = '1/2-1/2';
        }
      } else {
        statusEl.textContent = isInCheck(S, S.whiteToMove?'w':'b') ? 'Check!' : 'Your move.';
      }
      // update FEN dialog input
      document.getElementById('fenInput').value = toFEN(S);
    }

    function hasAnyLegalMoves(state, color){
      for(let r=0;r<8;r++) for(let c=0;c<8;c++){
        const p=state.board[r][c]; if(p && p.color===color){ if(legalMovesFrom(state, r,c).length) return true; }
      }
      return false;
    }

    function makeNotation(prev, r1,c1,r2,c2, move){
      const piece = prev.board[r1][c1]; const dest = idxToSquare(r2,c2); const src = idxToSquare(r1,c1);
      if(move.castle){ return (move.castle==='K'||move.castle==='k')?'O-O':'O-O-O'; }
      const name = piece.type==='p' ? '' : piece.type.toUpperCase();
      const cap = move.capture?'x':'';
      const promo = move.promo? `=${move.promoteTo.toUpperCase()}`:'';
      // add disambiguation minimal (file) if needed
      let disamb = '';
      if(piece.type!=='p'){
        let needs=false; for(let r=0;r<8;r++) for(let c=0;c<8;c++){
          if(r===r1&&c===c1) continue; const p=prev.board[r][c]; if(p && p.color===piece.color && p.type===piece.type){
            for(const m of generatePseudoMoves(prev, r,c,p)){ const st=makeMove(clone(prev), r,c,m.r,m.c, m); if(!isInCheck(st,p.color) && m.r===r2 && m.c===c2){ needs=true; break; } }
          }
        }
        if(needs) disamb = files[c1];
      }
      // check or mate marker
      const st = makeMove(clone(prev), r1,c1,r2,c2, move);
      const opp = piece.color==='w'?'b':'w';
      const check = isInCheck(st, opp);
      const mate = check && !hasAnyLegalMoves(st, opp);
      return `${name}${disamb}${cap}${dest}${promo}${mate?'#':(check?'+':'')}`;
    }

    // --- Controls -----------------------------------------------------------
    document.getElementById('flipBtn').onclick = ()=>{ S.orientation = (S.orientation==='w')?'b':'w'; flipBoard(); };
    document.getElementById('undoBtn').onclick = ()=>{
      if(!S.history.length) return; const last = S.history.pop();
      const state = parseFEN(last.fenBefore); S.board = state.board; S.whiteToMove = last.fenBefore.split(' ')[1]==='w';
      const rights = last.fenBefore.split(' ')[2]; S.castling = { K:rights.includes('K'), Q:rights.includes('Q'), k:rights.includes('k'), q:rights.includes('q') };
      S.ep = state.ep; S.halfmove = state.half; S.fullmove = state.full; statusEl.textContent = 'Move undone.'; render();
      document.getElementById('fenInput').value = toFEN(S);
    };
    document.getElementById('resetBtn').onclick = ()=>{ S = initState(); statusEl.textContent = 'Board reset.'; render(); document.getElementById('fenInput').value = toFEN(S); };
    document.getElementById('fenBtn').onclick = ()=>{ const d=document.getElementById('fenDialog'); document.getElementById('fenInput').value = toFEN(S); d.showModal(); };
    document.getElementById('fenCopy').onclick = ()=>{ const i=document.getElementById('fenInput'); i.select(); document.execCommand('copy'); };

    function flipBoard(){
      // Reverse the board arrays for rendering orientation only
      // We keep logical coords as 0..7; rendering flips using CSS order.
      boardEl.style.transform = (S.orientation==='b') ? 'rotate(180deg)' : 'none';
      for(const child of boardEl.children){ child.style.transform = (S.orientation==='b') ? 'rotate(180deg)' : 'none'; }
    }

    // --- Boot ---------------------------------------------------------------
    (function boot(){ render(); document.getElementById('fenInput').value = toFEN(S); })();

  </script>
</body>
</html>
