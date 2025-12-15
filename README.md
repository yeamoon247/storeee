<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<title>예문여고 매점 간식 사이트</title>
<meta name="viewport" content="width=device-width, initial-scale=1">

<style>
body { font-family: Arial; background:#f5f5f5; }
.container { max-width:600px; margin:20px auto; background:#fff; padding:20px; border-radius:12px; }
h1, h2 { text-align:center; }
.item { display:flex; justify-content:space-between; margin:8px 0; }
button { padding:6px 10px; margin-top:6px; border:none; border-radius:6px; cursor:pointer; }
.hidden { display:none; }
.banner { background:#fff3cd; padding:10px; border-radius:8px; margin-bottom:15px; }
.order { margin:6px 0; }
.delivered { text-decoration: line-through; color:#999; }
.admin-card { border:1px solid #ddd; border-radius:8px; padding:10px; margin:10px 0; }
</style>

<!-- Firebase SDK -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>

<script>
const firebaseConfig = {
  apiKey: "여기에",
  authDomain: "여기에",
  projectId: "여기에"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();
</script>
</head>

<body>
<div class="container">

<h1>예문여고 매점 간식 사이트</h1>

<div class="banner">
📌 입금 계좌: <b>카카오뱅크 7777-02-6483814 (이다솜)</b><br>
주문 후 <b>입금 완료 버튼</b>을 한 번만 꼭 눌러주세요.
</div>

<h2>간식 선택</h2>
<div class="item">모구모구 (1600원) <input type="number" min="0" value="0" data-name="모구모구" data-price="1600"></div>
<div class="item">스쿱마켓 (1000원) <input type="number" min="0" value="0" data-name="스쿱마켓" data-price="1000"></div>
<div class="item">홈런볼 (1500원) <input type="number" min="0" value="0" data-name="홈런볼" data-price="1500"></div>
<div class="item">허니버터칩 (2000원) <input type="number" min="0" value="0" data-name="허니버터칩" data-price="2000"></div>
<div class="item">오레오 (800원) <input type="number" min="0" value="0" data-name="오레오" data-price="800"></div>
<div class="item">킨더조이 (2000원) <input type="number" min="0" value="0" data-name="킨더조이" data-price="2000"></div>

<h3>주문자 정보</h3>
<input id="name" placeholder="학번 이름"><br>
<input id="phone" placeholder="전화번호"><br>

<p>배달비: 3,000원</p>
<p>총 금액: <span id="total">0</span>원</p>

<button onclick="submitOrder()">입금 완료</button>
<button onclick="toggleList()">구매 리스트 보기</button>

<div id="list" class="hidden"></div>

<hr>

<h2>관리자</h2>
<input type="password" id="pw" placeholder="비밀번호">
<button onclick="adminLogin()">접속</button>

<div id="admin" class="hidden"></div>

</div>

<script>
const DELIVERY = 3000;
const inputs = document.querySelectorAll('input[type=number]');
inputs.forEach(i => i.oninput = calc);

function calc(){
  let sum = 0;
  inputs.forEach(i => sum += i.value * i.dataset.price);
  document.getElementById('total').innerText = sum ? sum + DELIVERY : 0;
}

function submitOrder(){
  const name = document.getElementById('name').value;
  const phone = document.getElementById('phone').value;
  if(!name || !phone) return alert('정보 입력');

  const items = [];
  inputs.forEach(i => i.value>0 && items.push(`${i.dataset.name} x${i.value}`));
  if(!items.length) return alert('간식 선택');

  db.collection('orders').add({
    name, phone, items,
    delivered:false,
    time: Date.now()
  });

  alert('주문 완료');
}

function toggleList(){
  const el = document.getElementById('list');
  el.classList.toggle('hidden');
  db.collection('orders').onSnapshot(snap=>{
    el.innerHTML='';
    snap.forEach(d=>{
      const o=d.data();
      el.innerHTML+=`<div class="order ${o.delivered?'delivered':''}">
      ${o.name} - ${o.items.join(', ')}
      </div>`;
    });
  });
}

function adminLogin(){
  if(document.getElementById('pw').value!=='7942') return alert('비번 오류');
  const el=document.getElementById('admin');
  el.classList.remove('hidden');

  db.collection('orders').onSnapshot(snap=>{
    el.innerHTML='';
    snap.forEach(d=>{
      const o=d.data();
      el.innerHTML+=`
      <div class="admin-card">
      ${o.name}<br>${o.items.join(', ')}<br>
      <button onclick="db.collection('orders').doc('${d.id}').update({delivered:true})">
      배달 완료
      </button>
      </div>`;
    });
  });
}
</script>
</body>
</html>
