<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Canvas Chess</title>
  <style>
    body { display:flex; align-items:center; justify-content:center; height:100vh; margin:0; font-family:system-ui,Segoe UI,Roboto,Helvetica,Arial; background:#f0f0f0 }
    .container { text-align:center }
    canvas { background:#fff; display:block; margin:0 auto; box-shadow:0 6px 18px rgba(0,0,0,0.12); }
    .controls { margin-top:12px }
    button { padding:8px 12px; border-radius:6px; border:1px solid #bbb; background:#fff; cursor:pointer }
    p.note { font-size:14px; color:#333 }
    #status { margin-top:8px; font-weight:700; min-height:20px }
  </style>
</head>
<body>
  <div class="container">
    <h2>Canvas Chess</h2>
    <canvas id="board" width="480" height="480"></canvas>
    <div class="controls">
      <button id="restart">Restart</button>
      <span style="margin-left:12px">Turn: <strong id="turnText">White</strong></span>
    </div>
    <div id="status"></div>
    <p class="note">Click a piece to select, then click a highlighted square to move. No castling or en-passant. Pawn promotes to Queen automatically.</p>
  </div>

<script>
const canvas = document.getElementById('board');
const ctx = canvas.getContext('2d');
const size = canvas.width; // 480
const N = 8;
const S = size / N; // square size
let board = []; // 8x8
let selected = null; // {r,c}
let legalMoves = [];
let turn = 'w'; // 'w' or 'b'
const turnText = document.getElementById('turnText');
const statusEl = document.getElementById('status');
let gameOver = false;

// Use Unicode chess pieces
const PIECE_UNICODE = {
  pw: '♙', pw_lower: '♙',
  nw: '♘', bw: '♗', rw: '♖', qw: '♕', kw: '♔',
  pb: '♟', nb: '♞', bb: '♝', rb: '♜', qb: '♛', kb: '♚'
};

function makeEmptyBoard() {
  const b = Array.from({length:N}, () => Array(N).fill(null));
  return b;
}

function setupStartingPosition() {
  board = makeEmptyBoard();
  const back = ['r','n','b','q','k','b','n','r'];
  // black
  for(let c=0;c<8;c++) board[0][c] = {type:back[c], color:'b'};
  for(let c=0;c<8;c++) board[1][c] = {type:'p', color:'b'};
  // white
  for(let c=0;c<8;c++) board[6][c] = {type:'p', color:'w'};
  for(let c=0;c<8;c++) board[7][c] = {type:back[c], color:'w'};
  turn = 'w';
  turnText.textContent = 'White';
  selected = null; legalMoves = []; gameOver = false; statusEl.textContent = '';
}

function drawBoard() {
  // squares
  for(let r=0;r<N;r++){
    for(let c=0;c<N;c++){
      const light = (r + c) % 2 === 0;
      ctx.fillStyle = light ? '#f0d9b5' : '#b58863';
      ctx.fillRect(c*S, r*S, S, S);
    }
  }
  // highlight legal moves
  if(selected){
    // selected square border
    ctx.strokeStyle = '#00aaff'; ctx.lineWidth = 3;
    ctx.strokeRect(selected.c*S + 2, selected.r*S + 2, S-4, S-4);
  }
  for(const mv of legalMoves){
    const {r,c,cap} = mv;
    ctx.beginPath();
    if(cap){
      // circle with ring
      ctx.strokeStyle = 'rgba(0,0,0,0.6)'; ctx.lineWidth=2;
      ctx.arc(c*S + S/2, r*S + S/2, S*0.34, 0, Math.PI*2);
      ctx.stroke();
    } else {
      // small filled dot
      ctx.fillStyle = 'rgba(0,0,0,0.35)';
      ctx.arc(c*S + S/2, r*S + S/2, S*0.12, 0, Math.PI*2);
      ctx.fill();
    }
  }
  // pieces
  ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
  ctx.font = (S * 0.8) + 'px serif';
  for(let r=0;r<N;r++){
    for(let c=0;c<N;c++){
      const p = board[r][c];
      if(p){
        const key = p.type + p.color;
        const ch = PIECE_UNICODE[key] || '?';
        // piece color (use black for black, white for white with slight shadow)
        if(p.color==='w'){
          ctx.fillStyle = '#fff';
          ctx.lineWidth = 2;
          // draw stroke for visibility
          ctx.strokeStyle = '#000';
          ctx.strokeText(ch, c*S + S/2, r*S + S/2);
          ctx.fillText(ch, c*S + S/2, r*S + S/2);
        } else {
          ctx.fillStyle = '#000';
          ctx.fillText(ch, c*S + S/2, r*S + S/2);
        }
      }
    }
  }

  // highlight king in check
  const wk = findKing('w');
  const bk = findKing('b');
  if(wk && isSquareAttacked(wk.r, wk.c, 'b')){
    ctx.strokeStyle = 'red'; ctx.lineWidth = 4;
    ctx.strokeRect(wk.c*S + 3, wk.r*S + 3, S-6, S-6);
  }
  if(bk && isSquareAttacked(bk.r, bk.c, 'w')){
    ctx.strokeStyle = 'red'; ctx.lineWidth = 4;
    ctx.strokeRect(bk.c*S + 3, bk.r*S + 3, S-6, S-6);
  }

  updateStatus();
}

// Helpers
function inBounds(r,c){ return r>=0 && r<8 && c>=0 && c<8 }

function getPiece(r,c){ if(!inBounds(r,c)) return null; return board[r][c]; }

function isEmpty(r,c){ return inBounds(r,c) && board[r][c]===null }

function isEnemy(r,c,color){ const p = getPiece(r,c); return p && p.color !== color }

function findKing(color){ for(let r=0;r<8;r++) for(let c=0;c<8;c++){ const p = board[r][c]; if(p && p.type==='k' && p.color===color) return {r,c}; } return null; }

// check if square r,c is attacked by any piece of byColor (used for check detection)
function isSquareAttacked(r,c, byColor){
  if(!inBounds(r,c)) return false;
  // pawns
  if(byColor === 'w'){
    if(inBounds(r+1,c-1) && getPiece(r+1,c-1)?.type === 'p' && getPiece(r+1,c-1)?.color === 'w') return true;
    if(inBounds(r+1,c+1) && getPiece(r+1,c+1)?.type === 'p' && getPiece(r+1,c+1)?.color === 'w') return true;
  } else {
    if(inBounds(r-1,c-1) && getPiece(r-1,c-1)?.type === 'p' && getPiece(r-1,c-1)?.color === 'b') return true;
    if(inBounds(r-1,c+1) && getPiece(r-1,c+1)?.type === 'p' && getPiece(r-1,c+1)?.color === 'b') return true;
  }
  // knights
  const deltasN = [[-2,-1],[-2,1],[-1,-2],[-1,2],[1,-2],[1,2],[2,-1],[2,1]];
  for(const [dr,dc] of deltasN){ const rr=r+dr, cc=c+dc; if(inBounds(rr,cc)){ const p = getPiece(rr,cc); if(p && p.type==='n' && p.color===byColor) return true; }}
  // bishops/queens (diagonals)
  const diag = [[-1,-1],[-1,1],[1,-1],[1,1]];
  for(const [dr,dc] of diag){ let rr=r+dr, cc=c+dc; while(inBounds(rr,cc)){ const p = getPiece(rr,cc); if(p){ if(p.color===byColor && (p.type==='b' || p.type==='q')) return true; break; } rr+=dr; cc+=dc; }}
  // rooks/queens (orthogonal)
  const orth = [[-1,0],[1,0],[0,-1],[0,1]];
  for(const [dr,dc] of orth){ let rr=r+dr, cc=c+dc; while(inBounds(rr,cc)){ const p = getPiece(rr,cc); if(p){ if(p.color===byColor && (p.type==='r' || p.type==='q')) return true; break; } rr+=dr; cc+=dc; }}
  // king (adjacent)
  for(let dr=-1;dr<=1;dr++) for(let dc=-1;dc<=1;dc++){ if(dr===0 && dc===0) continue; const rr=r+dr, cc=c+dc; if(inBounds(rr,cc)){ const p = getPiece(rr,cc); if(p && p.type==='k' && p.color===byColor) return true; }}
  return false;
}

function simulateMoveLeavesKingInCheck(r1,c1,r2,c2, color){
  const src = board[r1][c1];
  const dst = board[r2][c2];
  if(!src) return true; // illegal
  // handle promotion simulation
  const promoted = (src.type==='p') && ((src.color==='w' && r2===0) || (src.color==='b' && r2===7));
  // make move
  board[r2][c2] = src;
  board[r1][c1] = null;
  if(promoted) board[r2][c2] = {type:'q', color: src.color};
  // find king position for 'color'
  let kingPos = findKing(color);
  if(!kingPos){
    // if we moved the king, its position is r2,c2
    // but findKing should find it unless we promoted or something weird
    kingPos = {r:r2,c:c2};
  }
  // if we moved the king, update
  if(src.type==='k') kingPos = {r:r2,c:c2};
  const opponent = color === 'w' ? 'b' : 'w';
  const inCheck = isSquareAttacked(kingPos.r, kingPos.c, opponent);
  // undo
  board[r1][c1] = src;
  board[r2][c2] = dst;
  return inCheck;
}

function getPseudoMovesFor(r,c){
  const p = getPiece(r,c);
  if(!p) return [];
  const moves = [];
  const color = p.color;
  const dir = color === 'w' ? -1 : 1;
  if(p.type === 'p'){
    // forward
    if(isEmpty(r+dir, c)) moves.push({r:r+dir,c,cap:false});
    // double from start
    const startRow = color==='w' ? 6 : 1;
    if(r===startRow && isEmpty(r+dir,c) && isEmpty(r+2*dir,c)) moves.push({r:r+2*dir,c,cap:false});
    // captures
    for(const dc of [-1,1]){
      if(isEnemy(r+dir,c+dc,color)) moves.push({r:r+dir,c:c+dc,cap:true});
    }
  }
  else if(p.type === 'n'){
    const deltas = [[-2,-1],[-2,1],[-1,-2],[-1,2],[1,-2],[1,2],[2,-1],[2,1]];
    for(const [dr,dc] of deltas){ const rr=r+dr, cc=c+dc; if(inBounds(rr,cc) && (!getPiece(rr,cc) || isEnemy(rr,cc,color))) moves.push({r:rr,c:cc,cap:!!getPiece(rr,cc)}); }
  }
  else if(p.type === 'b' || p.type === 'r' || p.type === 'q'){
    const dirs = [];
    if(p.type==='b' || p.type==='q') dirs.push([-1,-1],[-1,1],[1,-1],[1,1]);
    if(p.type==='r' || p.type==='q') dirs.push([-1,0],[1,0],[0,-1],[0,1]);
    for(const [dr,dc] of dirs){
      let rr=r+dr, cc=c+dc;
      while(inBounds(rr,cc)){
        const occ = getPiece(rr,cc);
        if(!occ) moves.push({r:rr,c:cc,cap:false});
        else { if(occ.color!==color) moves.push({r:rr,c:cc,cap:true}); break; }
        rr += dr; cc += dc;
      }
    }
  }
  else if(p.type === 'k'){
    for(let dr=-1;dr<=1;dr++) for(let dc=-1;dc<=1;dc++){ if(dr===0 && dc===0) continue; const rr=r+dr, cc=c+dc; if(inBounds(rr,cc) && (!getPiece(rr,cc) || isEnemy(rr,cc,color))) moves.push({r:rr,c:cc,cap:!!getPiece(rr,cc)}); }
  }
  return moves;
}

function getLegalMovesFor(r,c){
  const p = getPiece(r,c);
  if(!p) return [];
  const pseudo = getPseudoMovesFor(r,c);
  const legal = [];
  for(const mv of pseudo){
    if(!simulateMoveLeavesKingInCheck(r,c,mv.r,mv.c,p.color)) legal.push(mv);
  }
  return legal;
}

function hasAnyLegalMoves(color){
  for(let r=0;r<8;r++){
    for(let c=0;c<8;c++){
      const p = getPiece(r,c);
      if(p && p.color === color){
        const moves = getLegalMovesFor(r,c);
        if(moves.length>0) return true;
      }
    }
  }
  return false;
}

function coordsFromMouse(evt){
  const rect = canvas.getBoundingClientRect();
  const x = evt.clientX - rect.left;
  const y = evt.clientY - rect.top;
  const c = Math.floor(x / S);
  const r = Math.floor(y / S);
  return {r,c};
}

canvas.addEventListener('click', (e) => {
  if(gameOver) return;
  const {r,c} = coordsFromMouse(e);
  if(!inBounds(r,c)) return;
  const p = getPiece(r,c);
  if(selected){
    // check if clicked one of legalMoves
    const match = legalMoves.find(m => m.r===r && m.c===c);
    if(match){
      movePiece(selected.r, selected.c, r, c);
      selected = null; legalMoves = [];
      drawBoard();
      return;
    }
  }
  if(p && p.color === turn){
    selected = {r,c};
    legalMoves = getLegalMovesFor(r,c);
  } else {
    selected = null; legalMoves = [];
  }
  drawBoard();
});

function movePiece(r1,c1,r2,c2){
  const p = board[r1][c1];
  if(!p) return;
  // simple capture/move
  const captured = board[r2][c2];
  // handle promotion
  const willPromote = (p.type==='p') && ((p.color==='w' && r2===0) || (p.color==='b' && r2===7));
  board[r2][c2] = p;
  board[r1][c1] = null;
  if(willPromote) board[r2][c2].type = 'q';

  // after move, check for check/checkmate/stalemate
  const movedColor = p.color;
  const opponent = movedColor === 'w' ? 'b' : 'w';
  // find opponent king
  const oppKing = findKing(opponent);
  let opponentInCheck = false;
  if(oppKing) opponentInCheck = isSquareAttacked(oppKing.r, oppKing.c, movedColor);

  // switch turn
  turn = opponent;
  turnText.textContent = turn === 'w' ? 'White' : 'Black';

  if(opponentInCheck){
    if(!hasAnyLegalMoves(opponent)){
      statusEl.textContent = `Checkmate! ${movedColor === 'w' ? 'White' : 'Black'} wins.`;
      gameOver = true;
    } else {
      statusEl.textContent = `${opponent === 'w' ? 'White' : 'Black'} is in check.`;
    }
  } else {
    // not in check
    if(!hasAnyLegalMoves(opponent)){
      statusEl.textContent = 'Stalemate.';
      gameOver = true;
    } else {
      statusEl.textContent = '';
    }
  }
}

function updateStatus(){
  if(gameOver) return; // don't overwrite checkmate/stalemate message
  // show if current player is in check
  const myKing = findKing(turn);
  if(myKing && isSquareAttacked(myKing.r, myKing.c, turn === 'w' ? 'b' : 'w')){
    statusEl.textContent = `${turn === 'w' ? 'White' : 'Black'} is in check.`;
  } else {
    statusEl.textContent = '';
  }
}

// restart
document.getElementById('restart').addEventListener('click', ()=>{ setupStartingPosition(); drawBoard(); });

// init
setupStartingPosition(); drawBoard();

</script>
</body>
</html>
