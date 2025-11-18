/* Basic front-end SPA + example Firebase wiring (if configured)
   - This file uses localStorage to simulate auth when Firebase isn't present.
   - If you add Firebase config (see firebase.example.config.js), the code will
     try to initialize firebase and use Auth + Firestore for persistence.
*/

const state = {
  user: null, // {uid, name, email, type: 'user'|'entrepreneur'}
  services: [],
  bookings: []
}

// Helpers
const $ = (sel) => document.querySelector(sel)
const $$ = (sel) => Array.from(document.querySelectorAll(sel))

function showView(name){
  $$('.view').forEach(v => v.classList.add('hidden'))
  const el = $(#view-${name})
  if(el) el.classList.remove('hidden')
  $$('.nav-link').forEach(a=>a.classList.toggle('active', a.dataset.view===name))
}

// Initial demo services
function seedServices(){
  if(localStorage.getItem('pf_services')){
    state.services = JSON.parse(localStorage.getItem('pf_services'))
  } else {
    state.services = [
      {id:'s1',name:'Sharma Ji - Vedic Pundit',city:'Hyderabad',price:800,about:'Experienced in weddings & havan'},
      {id:'s2',name:'Ramakrishna Pundit',city:'Sangareddy',price:500,about:'Griha pravesh, Satyanarayan puja'},
      {id:'s3',name:'Pandit K. Reddy',city:'Secunderabad',price:1200,about:'Full wedding priest package'}
    ]
    localStorage.setItem('pf_services', JSON.stringify(state.services))
  }
}

function renderServices(){
  const c = $('#cardsContainer')
  c.innerHTML = ''
  state.services.forEach(s => {
    const card = document.createElement('div')
    card.className = 'service-card card'
    card.innerHTML = `
      <h3>${s.name}</h3>
      <div class="service-meta">${s.city} • ₹${s.price}</div>
      <p>${s.about}</p>
      <div class="row">
        <button class="btn" onclick="viewDetails('${s.id}')">View</button>
        <button class="btn primary" onclick="bookService('${s.id}')">Book</button>
      </div>`
    c.appendChild(card)
  })
}

function viewDetails(id){
  const s = state.services.find(x=>x.id===id)
  alert(${s.name} — ${s.city}\nPrice: ₹${s.price}\n\n${s.about})
}

function bookService(id){
  const s = state.services.find(x=>x.id===id)
  if(!state.user){
    openLoginModal(() => {
      // after login, continue booking
      bookService(id)
    })
    return
  }
  const booking = {id:'b'+Date.now(), serviceId:id, userId:state.user.uid||'local', status:'pending', date: new Date().toISOString()}
  state.bookings.push(booking)
  localStorage.setItem('pf_bookings', JSON.stringify(state.bookings))
  alert('Booking requested — check Bookings view')
}

function renderBookings(){
  const el = $('#bookingsList')
  el.innerHTML = ''
  const bookings = JSON.parse(localStorage.getItem('pf_bookings')||'[]')
  if(!bookings.length) return el.innerHTML = '<p>No bookings yet.</p>'
  bookings.forEach(b => {
    const s = state.services.find(x=>x.id===b.serviceId) || {name:'Unknown'}
    const d = document.createElement('div')
    d.className = 'card'
    d.style.marginBottom='10px'
    d.innerHTML = <strong>${s.name}</strong><div class="service-meta">${new Date(b.date).toLocaleString()}</div><div>Status: ${b.status}</div>
    el.appendChild(d)
  })
}

function renderProfile(){
  const el = $('#profileInfo')
  if(!state.user) return el.innerHTML = '<p>Not logged in.</p>'
  el.innerHTML = `
    <p><strong>${state.user.name || state.user.email}</strong></p>
    <p>Type: ${state.user.type || 'user'}</p>
    <button class='btn ghost' onclick='logout()'>Logout</button>`
}

function openLoginModal(afterLogin){
  const root = document.getElementById('modalRoot')
  root.innerHTML = `
    <div class='modalOverlay'>
      <div class='modal card'>
        <h3>Login</h3>
        <input id='m_email' placeholder='Email' />
        <input id='m_password' placeholder='Password' type='password' />
        <div class='row'>
          <button id='m_login' class='btn primary'>Login</button>
          <button id='m_close' class='btn ghost'>Close</button>
        </div>
      </div>
    </div>`
  document.getElementById('m_close').onclick = () => root.innerHTML = ''
  document.getElementById('m_login').onclick = () => {
    const email = document.getElementById('m_email').value
    const name = email.split('@')[0]
    state.user = {uid:'u'+Date.now(), email, name, type:'user'}
    localStorage.setItem('pf_user', JSON.stringify(state.user))
    root.innerHTML = ''
    renderProfile()
    if(afterLogin) afterLogin()
  }
}

function openSignupModal(){
  const root = document.getElementById('modalRoot')
  root.innerHTML = `
    <div class='modalOverlay'>
      <div class='modal card'>
        <h3>Sign up</h3>
        <input id='s_name' placeholder='Full name' />
        <input id='s_email' placeholder='Email' />
        <input id='s_password' placeholder='Password' type='password' />
        <select id='s_type'>
          <option value='user'>I'm a user</option>
          <option value='entrepreneur'>I'm an entrepreneur / pandit</option>
        </select>
        <div class='row'>
          <button id='s_submit' class='btn primary'>Create account</button>
          <button id='s_close' class='btn ghost'>Close</button>
        </div>
      </div>
    </div>`
  document.getElementById('s_close').onclick = () => root.innerHTML = ''
  document.getElementById('s_submit').onclick = () => {
    const name = document.getElementById('s_name').value
    const email = document.getElementById('s_email').value
    const type = document.getElementById('s_type').value
    state.user = {uid:'u'+Date.now(), email, name, type}
    localStorage.setItem('pf_user', JSON.stringify(state.user))
    root.innerHTML = ''
    renderProfile()
    if(type==='entrepreneur'){
      showView('for-entrepreneurs')
    } else showView('for-users')
  }
}

function logout(){
  state.user = null
  localStorage.removeItem('pf_user')
  renderProfile()
}

// Register pandit form
function wireRegisterForm(){
  const f = document.getElementById('registerPanditForm')
  f.addEventListener('submit', e=>{
    e.preventDefault()
    const s = {id:'s'+Date.now(), name:document.getElementById('p_name').value, city:document.getElementById('p_city').value, price:document.getElementById('p_price').value, about:document.getElementById('p_about').value}
    state.services.push(s)
    localStorage.setItem('pf_services', JSON.stringify(state.services))
    renderServices()
    alert('Service registered!')
    renderMyServices()
    f.reset()
  })
}

function renderMyServices(){
  const el = document.getElementById('myServices')
  el.innerHTML = ''
  const my = state.services // for demo, all services shown
  my.forEach(s=>{
    const d = document.createElement('div')
    d.className='card'
    d.innerHTML = <strong>${s.name}</strong><div class='service-meta'>${s.city} • ₹${s.price}</div>
    el.appendChild(d)
  })
}

// NAV wiring
$$('.nav-link').forEach(a=>a.addEventListener('click',()=>showView(a.dataset.view)))

// top buttons
$('#loginBtn').onclick = ()=> openLoginModal()
$('#signupBtn').onclick = ()=> openSignupModal()
$('#createBtn').onclick = ()=> alert('Create new content — feature placeholder')

// mobile menu
$('#menuBtn').onclick = ()=>{
  const s = document.getElementById('sidebar')
  s.style.display = s.style.display==='block'?'none':'block'
}

// load state
function load(){
  seedServices()
  state.bookings = JSON.parse(localStorage.getItem('pf_bookings')||'[]')
  state.user = JSON.parse(localStorage.getItem('pf_user')||'null')
  renderServices()
  renderBookings()
  renderProfile()
  renderMyServices()
  wireRegisterForm()
}

load()

// small accessibility helper for global scope functions used in markup
window.viewDetails = viewDetails
window.bookService = bookService
