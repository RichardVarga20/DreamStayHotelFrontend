<script>
	import { bookings, userGroup } from '$lib/store';
	import { onMount } from 'svelte';

	function showAlert(message, type) {
		const alertElement = document.getElementById('customAlert');
		alertElement.className = `alert alert-${type}`;
		alertElement.textContent = message;
		alertElement.classList.remove('d-none');

		setTimeout(() => {
			alertElement.classList.add('d-none');
		}, 2000);
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
	}

	async function handleBookingVerify(e) {
		const bookingIndex = e.target.getAttribute('data-booking-index');
		await fetch(`/api/Booking/booking/${bookingIndex}/verify`, {
			method: 'PUT',
			headers: {
				Authorization: `Bearer ${localStorage.getItem('AuthToken')}`
			}
		});
		setTimeout(() => {
			showAlert('Foglalása sikeresen elfogadva.', 'success');
		}, 450);

		let res = await fetch('/api/Booking/bookings', {
			method: 'GET',
			headers: {
				Authorization: `Bearer ${localStorage.getItem('AuthToken')}`
			}
		});
		const data = await res.json();
		$bookings = data;
	}

	onMount(async () => {
		let res = await fetch('/api/Booking/bookings', {
			method: 'GET',
			headers: {
				Authorization: `Bearer ${localStorage.getItem('AuthToken')}`
			}
		});
		const data = await res.json();
		$bookings = data;
		console.log(data);
	});
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
					<th>Művelet</th>
				</tr>
			</thead>
			<tbody>
				{#each $bookings as booking}
					<tr>
						<td>{booking.id}</td>
						<td>{booking.room.name}</td>
						<td>{booking.checkInDate}</td>
						<td>{booking.checkOutDate}</td>
						<td>
							<span class="badge {booking.status === 'Elfogadva' ? 'bg-success' : 'bg-warning'}">
								{booking.status}
							</span>
						</td>
						<td>
							{#if booking.status === 'Jóváhagyásra vár'}
								<button
									class="accept-btn"
									data-booking-index={booking.id}
									onclick={handleBookingVerify}>Elfogad</button
								>
							{:else if $userGroup === 'Admin'}
								<button
									class="delete-btn"
									data-booking-index={booking.id}
									onclick={handleBookingRemove}>Törlés</button
								>
							{/if}
						</td>
					</tr>
				{/each}
			</tbody>
		</table>
	</div>
</section>

<style>
	.container {
		max-width: 80rem;
		margin: 2rem auto;
		padding: 2rem;
		background: #ffffff;
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
	th,
	td {
		padding: 12px;
		text-align: left;
	}
	th {
		background-color: #212529;
		color: white;
	}
	tr:nth-child(even) {
		background-color: #f9f9f9;
	}
	.delete-btn,
	.accept-btn {
		border: none;
		padding: 6px 12px;
		border-radius: 4px;
		cursor: pointer;
		font-size: 0.9rem;
		transition: background-color 0.3s;
	}
	.delete-btn {
		background-color: #dc3545;
		color: white;
	}
	.delete-btn:hover {
		background-color: #c82333;
	}
	.accept-btn {
		background-color: #28a745;
		color: white;
	}
	.accept-btn:hover {
		background-color: #218838;
	}
</style>
