<script>
	import { bookings } from '$lib/store';
	import { onMount } from 'svelte';

	let currentRoom = $state();
	async function openBooking(e) {
		const currentRoomIndex = e.target.getAttribute('data-room-index');
		const response = await fetch(`/api/Room/rooms/${currentRoomIndex}`);
		const data = await response.json();

		currentRoom = data;
	}
	onMount(async () => {
		const response = await fetch(`/api/Room/rooms/${currentRoomIndex}`);
		const data = await response.json();

		currentRoom = data;
	});

	async function handleBooking() {
		if (!checkIn.value && !checkOut.value) {
			setTimeout(() => {
				showAlert('Kérem válasszon időpontot.', 'warning');
			}, 450);
			return;
		}

		const res = await fetch('/api/Booking/booking', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json',
				Authorization: `Bearer ${localStorage.getItem('AuthToken')}`
			},
			body: JSON.stringify({
				checkInDate: checkIn.value,
				checkOutDate: checkOut.value,
				roomId: currentRoom.id
			})
		});
		if (res.ok) {
			const bookingModal = bootstrap.Modal.getInstance(document.getElementById('bookingModal'));
			bookingModal.hide();
			checkIn.value = '';
			checkOut.value = '';
			setTimeout(() => {
				showAlert('Foglalása sikeresen megtörtént.', 'success');
			}, 450);
			return;
		}
		setTimeout(() => {
			showAlert('Kérem jelentkezzen be a foglaláshoz.', 'warning');
		}, 450);
	}

	async function handleBookingRemove(e) {
		const bookingIndex = e.target.getAttribute('data-booking-index');
		await fetch(`/api/Booking/booking/${bookingIndex}`, {
			method: 'DELETE',
			headers: {
				Authorization: `Bearer ${localStorage.getItem('AuthToken')}`
			}
		});
		setTimeout(() => {
			showAlert('Foglalása sikeresen törölve lett.', 'success');
		}, 450);

		let res = await fetch('/api/Booking/bookings', {
			method: 'GET',
			headers: {
				Authorization: `Bearer ${localStorage.getItem('AuthToken')}`
			}
		});
		const data = await res.json();
		$bookings = data;
		if ($bookings.length === 0) {
			const profileModal = bootstrap.Modal.getInstance(document.getElementById('profileModal'));
			profileModal.hide();
		}
	}

	function showAlert(message, type) {
		const alertElement = document.getElementById('customAlert');
		alertElement.className = `alert alert-${type}`;
		alertElement.textContent = message;
		alertElement.classList.remove('d-none');

		setTimeout(() => {
			alertElement.classList.add('d-none');
		}, 3000);
	}
</script>


