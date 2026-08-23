(function(){
    // ----- CONSTANTS -----
    const USERNAME = 'Noman@1985';
    const PASSWORD = '786092';
    const TOTAL_ROOMS = 24;
    const ROOM_START = 101;

    // ----- STATE (LocalStorage) -----
    let bookings = JSON.parse(localStorage.getItem('sa_bookings')) || [];
    let historyLog = JSON.parse(localStorage.getItem('sa_history')) || [];
    let session = localStorage.getItem('sa_session') || '';

    // ----- DOM REFS -----
    const splash = document.getElementById('splash');
    const loginPage = document.getElementById('loginPage');
    const dashboardPage = document.getElementById('dashboardPage');
    const loginBtn = document.getElementById('loginBtn');
    const loginError = document.getElementById('loginError');
    const logoutBtn = document.getElementById('logoutBtn');
    const roomsGrid = document.getElementById('roomsGrid');
    const modalOverlay = document.getElementById('modalOverlay');
    const modalBody = document.getElementById('modalBody');
    const modalTitle = document.getElementById('modalTitle');
    const closeModalBtn = document.getElementById('closeModalBtn');
    const searchInput = document.getElementById('searchInput');

    // stats elements
    const statTotal = document.getElementById('statTotal');
    const statAvailable = document.getElementById('statAvailable');
    const statReserved = document.getElementById('statReserved');
    const statCheckin = document.getElementById('statCheckin');
    const statCheckout = document.getElementById('statCheckout');
    const statBooked = document.getElementById('statBooked');
    const statCurrAvail = document.getElementById('statCurrAvail');
    const statIncome = document.getElementById('statIncome');
    const checkinInfoBtn = document.getElementById('checkinInfoBtn');
    const checkoutInfoBtn = document.getElementById('checkoutInfoBtn');

    // ----- HELPERS -----
    function saveState() {
      localStorage.setItem('sa_bookings', JSON.stringify(bookings));
      localStorage.setItem('sa_history', JSON.stringify(historyLog));
    }

    function getRoomStatus(roomNum) {
      const b = bookings.find(bk => bk.roomNumber === roomNum && bk.status === 'reserved');
      return b ? 'reserved' : 'available';
    }

    function getBookingForRoom(roomNum) {
      return bookings.find(bk => bk.roomNumber === roomNum && bk.status === 'reserved');
    }

    function getHistoryForRoom(roomNum) {
      return historyLog.filter(h => h.roomNumber === roomNum);
    }

    function todayStr() {
      return new Date().toISOString().slice(0,10);
    }

    // ----- VOUCHER NUMBER GENERATOR -----
    function generateVoucherNumber() {
      let counter = parseInt(localStorage.getItem('sa_voucher_counter') || '0', 10);
      counter += 1;
      localStorage.setItem('sa_voucher_counter', counter);
      return 'SA-' + String(counter).padStart(4, '0');
    }

    // ----- PRINT / AUTO-GENERATE VOUCHER -----
    function printVoucher(b) {
      const win = window.open('', '_blank', 'width=420,height=640');
      if (!win) { alert('Please allow pop-ups to view/print the voucher.'); return; }
      win.document.write(`
        <html>
        <head>
          <title>Voucher ${b.voucherNumber}</title>
          <style>
            body{font-family:Arial,Helvetica,sans-serif;padding:24px;color:#111;}
            h2{margin-bottom:2px;}
            .sub{color:#888;font-size:0.8rem;margin-bottom:18px;}
            .row{display:flex;justify-content:space-between;margin:8px 0;border-bottom:1px dashed #ccc;padding-bottom:6px;}
            .label{color:#666;font-size:0.85rem;}
            .value{font-weight:700;}
            .total{font-size:1.3rem;margin-top:18px;text-align:center;background:#f6d365;padding:12px;border-radius:8px;font-weight:700;}
            .badge{display:inline-block;padding:2px 10px;border-radius:20px;font-size:0.75rem;font-weight:700;}
            .paid{background:#c8f0dd;color:#1a7a4c;}
            .unpaid{background:#ffd9d3;color:#b23b28;}
          </style>
        </head>
        <body>
          <h2>✦ S*A Hotel</h2>
          <div class="sub">Booking Voucher</div>
          <div class="row"><span class="label">Voucher No.</span><span class="value">${b.voucherNumber}</span></div>
          <div class="row"><span class="label">Room</span><span class="value">${b.roomNumber} (${b.roomType})</span></div>
          <div class="row"><span class="label">Guest Name</span><span class="value">${b.guestName}</span></div>
          <div class="row"><span class="label">Mobile</span><span class="value">${b.mobile}</span></div>
          <div class="row"><span class="label">Check-in</span><span class="value">${b.checkIn}</span></div>
          <div class="row"><span class="label">Check-out</span><span class="value">${b.checkOut}</span></div>
          <div class="row"><span class="label">Total Nights</span><span class="value">${b.nights}</span></div>
          <div class="row"><span class="label">Meal Plan</span><span class="value">${b.food}</span></div>
          <div class="row"><span class="label">Selling Price / Night</span><span class="value">$${b.rent}</span></div>
          <div class="row"><span class="label">Payment Status</span><span class="value badge ${b.payment==='Paid'?'paid':'unpaid'}">${b.payment}</span></div>
          <div class="total">Total Bill: $${b.totalBill}</div>
          window.onload = function(){ window.print(); };<\/script>
        </body>
        </html>
      `);
      win.document.close();
    }

    // ----- RENDER ROOMS (with search filter) -----
    function renderRooms(filter='') {
      roomsGrid.innerHTML = '';
      for (let i=0; i<TOTAL_ROOMS; i++) {
        const roomNum = ROOM_START + i;
        const status = getRoomStatus(roomNum);
        const card = document.createElement('div');
        card.className = `room-card status-${status}`;
        card.innerHTML = `
          <div class="room-number">${roomNum}</div>
          <div class="room-status">${status === 'available' ? '🟢 Available' : '🟡 Reserved'}</div>
        `;
        card.addEventListener('click', (e) => {
          e.stopPropagation();
          openRoomModal(roomNum);
        });
        // apply search filter (by room number only for grid)
        const match = filter ? roomNum.toString().includes(filter) : true;
        if (!match) { card.style.display = 'none'; }
        roomsGrid.appendChild(card);
      }
    }

    // ----- UPDATE DASHBOARD STATS -----
    function updateStats() {
      const reserved = bookings.filter(b => b.status === 'reserved').length;
      const available = TOTAL_ROOMS - reserved;
      const today = todayStr();
      const checkinToday = bookings.filter(b => b.checkIn === today && b.status === 'reserved').length;
      const checkoutToday = bookings.filter(b => b.checkOut === today && b.status === 'reserved').length;
      const totalIncome = bookings.reduce((sum, b) => sum + (b.totalBill || 0), 0);

      statTotal.textContent = TOTAL_ROOMS;
      statAvailable.textContent = available;
      statReserved.textContent = reserved;
      statCheckin.textContent = checkinToday;
      statCheckout.textContent = checkoutToday;
      statBooked.textContent = reserved;
      statCurrAvail.textContent = available;
      statIncome.textContent = '$' + totalIncome;
    }

    // ----- CHECK-IN / CHECK-OUT INFO LIST MODAL -----
    function openInfoModal(type) {
      const today = todayStr();
      const list = bookings.filter(b => b.status === 'reserved' && (type === 'checkin' ? b.checkIn === today : b.checkOut === today));

      modalTitle.textContent = type === 'checkin' ? "📥 Today's Check-in List" : "📤 Today's Check-out List";

      let html = '';
      if (list.length === 0) {
        html += `<p style="color:rgba(255,255,255,0.4);">No rooms ${type === 'checkin' ? 'checking in' : 'checking out'} today.</p>`;
      } else {
        html += `<div class="history-list" style="max-height:65vh;">`;
        list.forEach(b => {
          html += `
            <div class="history-item info-list-item" data-room="${b.roomNumber}" style="cursor:pointer;">
              <div style="display:flex;justify-content:space-between;align-items:center;">
                <strong>Room ${b.roomNumber}</strong>
                <span>🎫 ${b.voucherNumber || '—'}</span>
              </div>
              <div>👤 <strong>${b.guestName}</strong> · 📞 ${b.mobile} · ${b.roomType}</div>
              <div>${b.checkIn} → ${b.checkOut} · ${b.payment} · $${b.totalBill}</div>
            </div>
          `;
        });
        html += `</div>`;
      }

      modalBody.innerHTML = html;
      modalOverlay.classList.add('active');

      modalBody.querySelectorAll('.info-list-item').forEach(item => {
        item.addEventListener('click', () => {
          const rn = parseInt(item.getAttribute('data-room'), 10);
          openRoomModal(rn);
        });
      });
    }

    // ----- OPEN MODAL (room detail + booking form + history) -----
    function openRoomModal(roomNum) {
      const booking = getBookingForRoom(roomNum);
      const history = getHistoryForRoom(roomNum);
      modalTitle.textContent = `Room ${roomNum}`;
      let html = '';

      // Booking form (only if admin logged)
      if (session === 'admin') {
        html += `<div style="margin-bottom:16px;"><h4>📋 Booking form</h4>`;
        const isReserved = !!booking;
        if (isReserved) {
          html += `
            <div class="bill-preview" style="margin-bottom:14px;">
              <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:8px;">
                <strong>🎫 Voucher: ${booking.voucherNumber || '—'}</strong>
                <button class="btn-sm btn-edit" id="printVoucherBtn" type="button">🖨 Print Voucher</button>
              </div>
              <div style="margin-top:8px;">🔴 Reserved by <strong>${booking.guestName}</strong> · ${booking.checkIn} → ${booking.checkOut} (${booking.nights || ''} nights)</div>
              <div>Payment: <strong>${booking.payment}</strong> · Total: <strong>$${booking.totalBill}</strong></div>
            </div>
          `;
        }
        html += `
          <div style="display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-top:10px;">
            <input type="text" id="fGuest" placeholder="Guest Name" value="${booking?.guestName || ''}">
            <input type="text" id="fMobile" placeholder="Mobile" value="${booking?.mobile || ''}">
            <select id="fRoomType">
              <option value="Double" ${booking?.roomType==='Double'?'selected':''}>Double</option>
              <option value="Triple" ${booking?.roomType==='Triple'?'selected':''}>Triple</option>
              <option value="Quad" ${booking?.roomType==='Quad'?'selected':''}>Quad</option>
              <option value="Quint" ${booking?.roomType==='Quint'?'selected':''}>Quint</option>
              <option value="Hexa" ${booking?.roomType==='Hexa'?'selected':''}>Hexa</option>
            </select>
            <select id="fFood"><option value="With Food" ${booking?.food==='With Food'?'selected':''}>With Food</option><option value="Without Food" ${booking?.food==='Without Food'?'selected':''}>Without Food</option></select>
            <input type="date" id="fCheckIn" value="${booking?.checkIn || todayStr()}">
            <input type="date" id="fCheckOut" value="${booking?.checkOut || todayStr()}">
            <input type="number" id="fRent" placeholder="Selling Price (per night)" value="${booking?.rent || 0}">
            <select id="fPayment"><option value="Paid" ${booking?.payment==='Paid'?'selected':''}>Paid</option><option value="Unpaid" ${booking?.payment==='Unpaid'?'selected':''}>Unpaid</option></select>
          </div>
          <div class="bill-preview" id="billPreview">Total Nights: 0 | Total Bill: $0</div>
          <div style="display:flex;gap:10px;flex-wrap:wrap;margin:10px 0;">
            <button class="btn-sm btn-save" id="saveBookingBtn">💾 Save Booking</button>
            ${booking ? `<button class="btn-sm btn-delete" id="deleteBookingBtn">🗑 Delete</button>` : ''}
            ${booking ? `<button class="btn-sm btn-edit" id="editBookingBtn">✏️ Update</button>` : ''}
          </div>
        `;
      } else {
        html += `<p style="color:rgba(255,255,255,0.4);">🔒 Login to manage bookings</p>`;
      }

      // History section
      html += `<hr style="border-color:rgba(255,255,255,0.05);margin:18px 0;">`;
      html += `<h4>📜 Room History</h4>`;
      if (history.length === 0) {
        html += `<p style="color:rgba(255,255,255,0.3);">No history yet.</p>`;
      } else {
        html += `<div class="history-list">`;
        history.slice().reverse().forEach(h => {
          html += `<div class="history-item"><strong>${h.guestName}</strong> · ${h.roomType} · 🎫 ${h.voucherNumber || '—'} · ${h.checkIn} → ${h.checkOut} · $${h.totalBill} · ${h.payment}</div>`;
        });
        html += `</div>`;
      }

      modalBody.innerHTML = html;
      modalOverlay.classList.add('active');

      // Attach events if admin
      if (session === 'admin') {
        const saveBtn = document.getElementById('saveBookingBtn');
        const deleteBtn = document.getElementById('deleteBookingBtn');
        const editBtn = document.getElementById('editBookingBtn');
        const printBtn = document.getElementById('printVoucherBtn');
        printBtn?.addEventListener('click', () => { if (booking) printVoucher(booking); });

        // Preview bill
        const checkIn = document.getElementById('fCheckIn');
        const checkOut = document.getElementById('fCheckOut');
        const rent = document.getElementById('fRent');
        const preview = document.getElementById('billPreview');
        function updatePreview() {
          const ci = new Date(checkIn.value);
          const co = new Date(checkOut.value);
          const nights = Math.max(0, Math.ceil((co - ci) / (1000*60*60*24)));
          const bill = nights * (parseFloat(rent.value) || 0);
          preview.textContent = `Total Nights: ${nights} | Total Bill: $${bill}`;
        }
        checkIn?.addEventListener('change', updatePreview);
        checkOut?.addEventListener('change', updatePreview);
        rent?.addEventListener('input', updatePreview);
        setTimeout(updatePreview, 50);

        // Save / Create booking
        function saveBooking(update=false) {
          const guest = document.getElementById('fGuest').value.trim();
          const mobile = document.getElementById('fMobile').value.trim();
          const roomType = document.getElementById('fRoomType').value;
          const food = document.getElementById('fFood').value;
          const checkInVal = document.getElementById('fCheckIn').value;
          const checkOutVal = document.getElementById('fCheckOut').value;
          const rentVal = parseFloat(document.getElementById('fRent').value) || 0;
          const payment = document.getElementById('fPayment').value;
          if (!guest || !mobile || !checkInVal || !checkOutVal) {
            alert('Please fill all fields');
            return;
          }
          const ci = new Date(checkInVal);
          const co = new Date(checkOutVal);
          if (co <= ci) { alert('Check-out must be after check-in'); return; }
          const nights = Math.ceil((co - ci) / (1000*60*60*24));
          const totalBill = nights * rentVal;

          // keep the same voucher number when updating an existing booking,
          // otherwise generate a fresh one automatically
          const voucherNumber = (update && booking?.voucherNumber) ? booking.voucherNumber : generateVoucherNumber();

          const newBooking = {
            roomNumber: roomNum,
            guestName: guest,
            mobile: mobile,
            roomType: roomType,
            food: food,
            checkIn: checkInVal,
            checkOut: checkOutVal,
            nights: nights,
            rent: rentVal,
            payment: payment,
            totalBill: totalBill,
            voucherNumber: voucherNumber,
            status: 'reserved'
          };

          if (update) {
            const idx = bookings.findIndex(b => b.roomNumber === roomNum && b.status === 'reserved');
            if (idx !== -1) {
              historyLog.push({...bookings[idx], status: 'history'});
              bookings[idx] = newBooking;
            } else {
              bookings.push(newBooking);
            }
          } else {
            if (bookings.some(b => b.roomNumber === roomNum && b.status === 'reserved')) {
              alert('Room already reserved. Use Update or Delete first.');
              return;
            }
            bookings.push(newBooking);
          }
          saveState();
          renderRooms(searchInput.value);
          updateStats();
          openRoomModal(roomNum);
          // automatically generate & show the voucher
          printVoucher(newBooking);
        }

        saveBtn?.addEventListener('click', () => saveBooking(false));
        editBtn?.addEventListener('click', () => saveBooking(true));
        deleteBtn?.addEventListener('click', () => {
          if (confirm(`Delete booking for room ${roomNum}?`)) {
            const idx = bookings.findIndex(b => b.roomNumber === roomNum && b.status === 'reserved');
            if (idx !== -1) {
              const removed = bookings.splice(idx, 1)[0];
              historyLog.push({...removed, status: 'history'});
              saveState();
              renderRooms(searchInput.value);
              updateStats();
              openRoomModal(roomNum);
            }
          }
        });
      }
    }

    // ----- LOGIN / LOGOUT -----
    function showDashboard() {
      loginPage.style.display = 'none';
      dashboardPage.style.display = 'block';
      session = 'admin';
      localStorage.setItem('sa_session', 'admin');
      renderRooms();
      updateStats();
    }

    function showLogin() {
      loginPage.style.display = 'flex';
      dashboardPage.style.display = 'none';
      session = '';
      localStorage.removeItem('sa_session');
      renderRooms();
    }

    function handleLogin() {
      const u = document.getElementById('loginUser').value.trim();
      const p = document.getElementById('loginPass').value.trim();
      if (u === USERNAME && p === PASSWORD) {
        loginError.textContent = '';
        showDashboard();
      } else {
        loginError.textContent = '❌ Invalid username or password';
      }
    }

    function handleLogout() {
      // show splash with logout animation
      splash.classList.remove('hide-splash');
      setTimeout(() => {
        showLogin();
        splash.classList.add('hide-splash');
      }, 2500);
    }

    // ----- SPLASH INIT -----
    function initSplash() {
      splash.classList.remove('hide-splash');
      setTimeout(() => {
        splash.classList.add('hide-splash');
        if (localStorage.getItem('sa_session') === 'admin') {
          showDashboard();
        } else {
          showLogin();
        }
      }, 2800);
    }

    // ----- EVENT LISTENERS -----
    loginBtn.addEventListener('click', handleLogin);
    document.addEventListener('keydown', (e) => {
      if (e.key === 'Enter' && loginPage.style.display !== 'none') handleLogin();
    });
    logoutBtn.addEventListener('click', handleLogout);
    closeModalBtn.addEventListener('click', () => modalOverlay.classList.remove('active'));
    checkinInfoBtn?.addEventListener('click', () => openInfoModal('checkin'));
    checkoutInfoBtn?.addEventListener('click', () => openInfoModal('checkout'));
    modalOverlay.addEventListener('click', (e) => {
      if (e.target === modalOverlay) modalOverlay.classList.remove('active');
    });

    // Search functionality (filters room grid by room number)
    searchInput.addEventListener('input', (e) => {
      const val = e.target.value.trim().toLowerCase();
      renderRooms(val);
      // also filter cards directly
      document.querySelectorAll('.room-card').forEach(card => {
        const num = card.querySelector('.room-number')?.textContent || '';
        card.style.display = num.includes(val) ? '' : 'none';
      });
    });

    // ----- START APPLICATION -----
    initSplash();
  })();
