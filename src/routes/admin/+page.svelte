<script>
    import { writable } from 'svelte/store';
    let selectedUser = writable(null);
    let users = [
      { id: 1, fullName: 'Peter Johnson', idCardNumber: 'A1234567' },
      { id: 2, fullName: 'Anna Smith', idCardNumber: 'B2345678' },
      { id: 3, fullName: 'Peter Johnson', idCardNumber: 'C3456789' }
    ];
    let bookings = [
      { id: 1, userIdCardNumber: 'A1234567', room: { name: 'Room 101' }, checkInDate: '2025-03-01', checkOutDate: '2025-03-07', status: 'Elfogadva', adults: 2, children: 1 },
      { id: 2, userIdCardNumber: 'A1234567', room: { name: 'Room 102' }, checkInDate: '2025-03-15', checkOutDate: '2025-03-20', status: 'Jóváhagyásra vár', adults: 1, children: 0 },
      { id: 3, userIdCardNumber: 'B2345678', room: { name: 'Room 201' }, checkInDate: '2025-03-01', checkOutDate: '2025-03-05', status: 'Elfogadva', adults: 2, children: 2 },
      { id: 4, userIdCardNumber: 'C3456789', room: { name: 'Room 301' }, checkInDate: '2025-03-10', checkOutDate: '2025-03-15', status: 'Jóváhagyásra vár', adults: 1, children: 1 }
    ];
    $: groupedBookings = bookings.reduce((acc, booking) => {
      if (!acc[booking.userIdCardNumber]) {
        acc[booking.userIdCardNumber] = [];
      }
      acc[booking.userIdCardNumber].push(booking);
      return acc;
    }, {});
    const selectUser = (idCardNumber) => {
      selectedUser.set(idCardNumber);
    }
    const closeModal = () => {
      selectedUser.set(null);
    }
    const handleBookingVerify = (bookingId) => {
      bookings = bookings.map(booking =>
        booking.id === bookingId ? {...booking, status: 'Elfogadva'} : booking
      );
      showAlert('Foglalása sikeresen elfogadva.', 'success');
    }
    const handleBookingRemove = (bookingId) => {
      bookings = bookings.filter(booking => booking.id !== bookingId);
      showAlert('Foglalása sikeresen törölve lett.', 'success');
      // Check if there are any remaining bookings for the selected user
      const remainingBookings = bookings.filter(booking => booking.userIdCardNumber === $selectedUser);
      if (remainingBookings.length === 0) {
        closeModal();
      }
    }
    let userGroup = writable('Admin');
    function showAlert(message, type) {
      const alertElement = document.getElementById('customAlert');
      alertElement.className = `alert alert-${type}`;
      alertElement.textContent = message;
      alertElement.classList.remove('d-none');
      setTimeout(() => {
        alertElement.classList.add('d-none');
      }, 2000);
    }
  </script>
  <section class="container">
    <h1 class="title">Foglalások kezelése</h1>
    <div class="table-container">
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Név</th>
            <th>Érkezés</th>
            <th>Távozás</th>
            <th>Státusz</th>
          </tr>
        </thead>
        <tbody>
          {#each Object.entries(groupedBookings) as [idCardNumber, userBookings]}
            {@const user = users.find(u => u.idCardNumber === idCardNumber)}
            {#each userBookings as booking}
              <tr>
                <td>{booking.id}</td>
                <td>
                  <a href="#" class="user-name" on:click|preventDefault={() => selectUser(user.idCardNumber)}>
                    {user.fullName}
                  </a>
                </td>
                <td>{booking.checkInDate}</td>
                <td>{booking.checkOutDate}</td>
                <td>
                  <span class="badge {booking.status === 'Elfogadva' ? 'bg-success' : 'bg-warning'}">
                    {booking.status}
                  </span>
                </td>
              </tr>
            {/each}
          {/each}
        </tbody>
      </table>
    </div>
  </section>
  {#if $selectedUser !== null}
    {@const selectedUserData = users.find(user => user.idCardNumber === $selectedUser)}
    {@const selectedUserBookings = groupedBookings[$selectedUser] || []}
    {#if selectedUserBookings.length > 0}
      <div class="modal-overlay" on:click={closeModal}>
        <div class="modal-content" on:click|stopPropagation>
          <h2>{selectedUserData.fullName} foglalásai</h2>
          <p>Személyi igazolvány szám: {selectedUserData.idCardNumber}</p>
          <ul>
            {#each selectedUserBookings as booking}
              <li>
                <div class="booking-details">
                  <strong>{booking.room.name}</strong>
                  <p>Érkezés: {booking.checkInDate}</p>
                  <p>Távozás: {booking.checkOutDate}</p>
                  <p>Felnőttek: {booking.adults}, Gyerekek: {booking.children}</p>
                  <p>Státusz:
                    <span class="badge {booking.status === 'Elfogadva' ? 'bg-success' : 'bg-warning'}">
                      {booking.status}
                    </span>
                  </p>
                </div>
                <div class="booking-actions">
                  {#if booking.status === 'Jóváhagyásra vár'}
                    <button
                      class="accept-btn"
                      on:click={() => handleBookingVerify(booking.id)}>Elfogad</button>
                  {:else if $userGroup === 'Admin'}
                    <button
                      class="delete-btn"
                      on:click={() => handleBookingRemove(booking.id)}>Törlés</button>
                  {/if}
                </div>
              </li>
            {/each}
          </ul>
          <button class="close-btn" on:click={closeModal}>Bezár</button>
        </div>
      </div>
    {/if}
  {/if}
  <div id="customAlert" class="alert d-none"></div>
  <style>
    .container {
      max-width: 80rem;
      margin: 2rem auto;
      padding: 2rem;
      background: #FFFFFF;
      box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
      border-radius: 10px;
    }
    .title {
      text-align: center;
      font-size: 1.8rem;
      margin-bottom: 1rem;
      color: #333;
    }
    .table-container {
      overflow-x: auto;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      background: #fff;
      border-radius: 8px;
      overflow: hidden;
    }
    th, td {
      padding: 12px;
      text-align: left;
    }
    th {
      background-color: #212529;
      color: white;
    }
    tr:nth-child(even) {
      background-color: #F9F9F9;
    }
    .user-name {
      color: #007BFF;
      text-decoration: none;
      cursor: pointer;
    }
    .user-name:hover {
      color: #0056B3;
    }
    .delete-btn, .accept-btn {
      border: none;
      padding: 6px 12px;
      border-radius: 4px;
      cursor: pointer;
      font-size: 0.9rem;
      transition: background-color 0.3s;
    }
    .delete-btn {
      background-color: #DC3545;
      color: white;
    }
    .delete-btn:hover {
      background-color: #C82333;
    }
    .accept-btn {
      background-color: #28A745;
      color: white;
    }
    .accept-btn:hover {
      background-color: #218838;
    }
    .badge {
      padding: 4px 8px;
      border-radius: 4px;
      font-size: 0.8rem;
    }
    .bg-success {
      background-color: #28A745;
      color: white;
    }
    .bg-warning {
      background-color: #FFC107;
      color: black;
    }
    .modal-overlay {
      position: fixed;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background: rgba(0, 0, 0, 0.5);
      display: flex;
      justify-content: center;
      align-items: center;
    }
    .modal-content {
      background: white;
      padding: 2rem;
      border-radius: 8px;
      max-width: 500px;
      width: 100%;
    }
    .modal-content h2 {
      margin-top: 0;
    }
    .modal-content ul {
      list-style-type: none;
      padding: 0;
    }
    .modal-content li {
      margin-bottom: 1rem;
      padding: 1rem;
      background: #F9F9F9;
      border-radius: 4px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .booking-details {
      flex: 1;
    }
    .booking-actions {
      display: flex;
      flex-direction: column;
      justify-content: center;
    }
    .close-btn {
      margin-top: 1rem;
      padding: 6px 12px;
      background-color: #6C757D;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    .close-btn:hover {
      background-color: #5A6268;
    }
    .alert {
      position: fixed;
      top: 20px;
      right: 20px;
      padding: 10px 20px;
      border-radius: 4px;
      color: white;
      font-weight: bold;
    }
    .alert-success {
      background-color: #28A745;
    }
    .d-none {
      display: none;
    }
  </style>