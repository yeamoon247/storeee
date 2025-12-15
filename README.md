<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <title>예문여고 매점 간식 사이트</title>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
  <style>
    body { font-family: Arial, sans-serif; background:#f5f5f5; padding:20px; }
    h1 { text-align:center; }
    .box { background:white; padding:20px; margin:20px auto; max-width:600px; border-radius:10px; box-shadow:0 4px 10px rgba(0,0,0,0.1);} 
    .snack { display:flex; justify-content:space-between; margin:10px 0; }
    input[type=number] { width:60px; }
    button { padding:8px 12px; margin-top:10px; cursor:pointer; }
    .order { border-bottom:1px solid #ddd; padding:10px 0; }
    .delivered { text-decoration: line-through; opacity:0.6; }
    .admin { display:none; }
  </style>
</head>
<body>

<h1>예문여고 매점 간식 사이트</h1>

<div class="box">
  <h2>🛒 간식 선택</h2>
  <div id="snacks"></div>
  <p>🚚 배달비: 3,000원</p>
  <h3>총 금액: <span id="total">0</span>원</h3>
  <p>입금 계좌: 카카오뱅크 7777-02-6483814 (이다솜)</p>
  <p><b>주문 후 입금 완료 버튼을 한 번만 꼭 눌러주세요.</b></p>
  <button onclick="submitOrder()">입금 완료</button>
</div>

<div class="box">
  <h2>📦 구매 리스트</h2>
  <div id="orderList"></div>
</div>

<div class="box">
  <h2>🔐 관리자 화면</h2>
  <input type="password" id="adminPw" placeholder="비밀번호 입력" />
  <button onclick="loginAdmin()">입장</button>
  <div class="admin" id="adminPanel"></div>
</div>

<script>
// Firebase 설정 (여기에 본인 설정값 넣기)
const firebaseConfig = {
  apiKey: "여기에",
  authDomain: "여기에",
  projectId: "여기에"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

const DELIVERY = 3000;
const snacksData = [
  {name:"모구모구", price:1600},
  {name:"스쿱마켓 한 스쿱", price:1000},
  {name:"홈런볼", price:1500},
  {name:"허니버터칩", price:2000},
  {name:"오레오", price:800},
  {name:"킨더조이", price:2000}
];

const snacksDiv = document.getElementById('snacks');
const totalEl = document.getElementById('total');

snacksData.forEach((s,i)=>{
  snacksDiv.innerHTML += `
    <div class="snack">
      <span>${s.name} (${s.price}원)</span>
      <input type="number" min="0" value="0" data-i="${i}" onchange="calc()" />
    </div>`;
});

function calc(){
  let sum=0;
  document.querySelectorAll('input[type=number]').forEach(inp=>{
    sum += snacksData[inp.dataset.i].price * Number(inp.value);
  });
  totalEl.textContent = sum>0 ? sum+DELIVERY : 0;
}

function submitOrder(){
  let items=[];
  document.querySelectorAll('input[type=number]').forEach(inp=>{
    if(inp.value>0){
      items.push(`${snacksData[inp.dataset.i].name} x${inp.value}`);
    }
  });
  if(items.length===0){ alert('간식을 선택하세요'); return; }

  db.collection('orders').add({
    items,
    total: totalEl.textContent,
    confirmed:false,
    delivered:false,
    time: new Date()
  });

  alert('주문 완료!');
}

function loadOrders(){
  db.collection('orders').orderBy('time','desc').onSnapshot(snap=>{
    orderList.innerHTML='';
    adminPanel.innerHTML='';
    snap.forEach(doc=>{
      const d=doc.data();
      const cls=d.delivered?'delivered':'';
      orderList.innerHTML+=`<div class="order ${cls}">${d.items.join(', ')} - ${d.total}원</div>`;

      adminPanel.innerHTML+=`
        <div class="order">
          ${d.items.join(', ')}<br>
          <button onclick="confirm('${doc.id}')">확인</button>
          <button onclick="deliver('${doc.id}')">배달 완료</button>
        </div>`;
    });
  });
}

function confirm(id){ db.collection('orders').doc(id).update({confirmed:true}); }
function deliver(id){ db.collection('orders').doc(id).update({delivered:true}); }

function loginAdmin(){
  if(document.getElementById('adminPw').value==='7942'){
    document.getElementById('adminPanel').style.display='block';
  } else alert('비밀번호 오류');
}

loadOrders();
</script>
</body>
</html>
