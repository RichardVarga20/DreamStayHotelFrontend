<script>
	import { bookings } from '$lib/store';

	async function handleGetBookings() {
		let res = await fetch('/api/Booking/bookings', {
			method: 'GET',
			headers: {
				Authorization: `Bearer ${localStorage.getItem('AuthToken')}`
			}
		});
		let data = await res.json();
		$bookings = data;
	}
	function handleLogout() {
		localStorage.removeItem('AuthToken');
		showAlert('Sikeres kijelentkezés.', 'success');
		updateUIForLoggedOutUser();
	}

	function updateUIForLoggedOutUser() {
		document.getElementById('loginRegisterNav').classList.remove('d-none');
		document.getElementById('logoutNav').classList.add('d-none');
		document.getElementById('profileNav').classList.add('d-none');
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

<nav class="navbar navbar-expand-lg navbar-light bg-light fixed-top">
	<div class="container">
		<a class="navbar-brand" href="#">DreamStay</a>
		<button
			class="navbar-toggler"
			type="button"
			data-bs-toggle="collapse"
			data-bs-target="#navbarNav"
			aria-controls="navbarNav"
			aria-expanded="false"
			aria-label="Toggle navigation"
		>
			<span class="navbar-toggler-icon"></span>
		</button>
		<div class="collapse navbar-collapse" id="navbarNav">
			<ul class="navbar-nav me-auto">
				<li class="nav-item">
					<a class="nav-link active" href="#">Főoldal</a>
				</li>
				<li class="nav-item">
					<a class="nav-link" href="#rooms">Szobák</a>
				</li>
				<li class="nav-item">
					<a class="nav-link" href="#about">Rólunk</a>
				</li>
				<li class="nav-item">
					<a class="nav-link" href="#contact">Kapcsolat</a>
				</li>
			</ul>
			<ul class="navbar-nav">
				<li class="nav-item" id="loginRegisterNav">
					<a class="nav-link" href="#" data-bs-toggle="modal" data-bs-target="#loginModal"
						>Bejelentkezés / Regisztráció</a
					>
				</li>
				<li class="nav-item d-none" id="logoutNav">
					<a class="nav-link" href="#" id="logoutBtn" onclick={handleLogout}>Kijelentkezés</a>
				</li>
				<li class="nav-item d-none" id="profileNav">
					<button
						onclick={handleGetBookings}
						class="nav-link"
						data-bs-toggle="modal"
						data-bs-target="#profileModal"
					>
						<i class="bi bi-person-circle"></i>
					</button>
				</li>
			</ul>
		</div>
	</div>
</nav>
