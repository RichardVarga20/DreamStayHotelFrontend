<script>
	import { goto } from '$app/navigation';
	import img6 from '$lib/images/6.png';
	import img7 from '$lib/images/7.png';
	import img8 from '$lib/images/8.png';
	import { bookings } from '$lib/store';
	import { onMount } from 'svelte';
	import { Swiper } from 'swiper';

	async function clearFilters() {
		const res = await fetch('/api/Room/rooms', { method: 'GET' });
		const data = await res.json();
		rooms = data;
		console.log(rooms);
	}
	
	let currentRoom = $state();
	async function openBooking(currentId) {
		const response = await fetch(`/api/Room/rooms/${currentId}`);
		const data = await response.json();

		currentRoom = data;
	}

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

		console.log(res);

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

	async function handleFilter() {
		goto('#roomList');
		const res = await fetch('/api/Room/rooms/sort-by', {
			method: 'POST',
			headers: { 'Content-Type': 'application/json' },
			body: JSON.stringify({
				maxAdults: adults.value,
				maxChildren: children.value
			})
		});

		const data = await res.json();
		rooms = data;
	}

	function updateUIForLoggedInUser() {
		document.getElementById('loginRegisterNav').classList.add('d-none');
		document.getElementById('logoutNav').classList.remove('d-none');
		document.getElementById('profileNav').classList.remove('d-none');
		//   displayUserBookings();
	}

	// Function to update UI for logged-out user
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
		}, 2000);
	}

	let rooms = $state([]);
	onMount(async () => {
		const res = await fetch('/api/Room/rooms', { method: 'GET' });
		const data = await res.json();
		rooms = data;
		if (localStorage.getItem('AuthToken')) {
			updateUIForLoggedInUser();
		} else {
			updateUIForLoggedOutUser();
		}
	});

	const reviews = [
		{
			title: 'Csodálatos Élmény',
			text: 'A szoba makulátlan volt, a személyzet hihetetlenül barátságos, és a szolgáltatások elsőrangúak. Biztosan újra itt fogok megszállni!',
			author: 'Kovács János',
			location: 'Budapest',
			image: 'https://randomuser.me/api/portraits/men/1.jpg'
		},
		{
			title: 'Felülmúlta az Elvárásokat',
			text: 'A bejelentkezéstől a távozásig minden tökéletes volt. A részletekre való odafigyelés lenyűgöző volt.',
			author: 'Nagy Sára',
			location: 'Debrecen',
			image: 'https://randomuser.me/api/portraits/women/2.jpg'
		},
		{
			title: 'Hamarosan Visszatérek',
			text: 'A helyszín ideális volt, a szoba tágas és kényelmes, a személyzet pedig mindent megtett értünk. Alig várom, hogy visszatérjek!',
			author: 'Szabó Máté',
			location: 'Szeged',
			image: 'https://randomuser.me/api/portraits/men/3.jpg'
		},
		{
			title: 'Kiváló Szolgáltatás',
			text: 'A személyzet rendkívül segítőkész volt, és minden kérésünket teljesítették. A szoba tiszta és kényelmes volt.',
			author: 'Tóth Anna',
			location: 'Pécs',
			image: 'https://randomuser.me/api/portraits/women/4.jpg'
		},
		{
			title: 'Felejthetetlen Élmény',
			text: 'A kilátás lélegzetelállító volt, és a spa szolgáltatások tökéletes kikapcsolódást nyújtottak. Mindenképpen ajánlom!',
			author: 'Horváth Eszter',
			location: 'Győr',
			image: 'https://randomuser.me/api/portraits/women/5.jpg'
		}
	];

	async function handleRegister() {
		if (registerPassword.value !== registerConfirmPassword.value) {
			setTimeout(() => {
				showAlert('Két jelszó nem egyezik meg.', 'warning');
			}, 450);
			return;
		}

		const res = await fetch('/api/Account/register', {
			method: 'POST',
			headers: { 'Content-Type': 'application/json' },
			body: JSON.stringify({
				userName: registerName.value,
				password: registerPassword.value,
				email: registerEmail.value
			})
		});

		const data = await res.text();

		if (res.ok) {
			const registerModal = bootstrap.Modal.getInstance(document.getElementById('loginModal'));
			registerModal.hide();
			setTimeout(() => {
				showAlert('Sikeres regisztráció.', 'success');
			}, 450);
			return;
		}
		setTimeout(() => {
			showAlert('Sikertelen regisztráció.', 'danger');
		}, 450);
	}

	async function handleLogin() {
		const res = await fetch('/api/Account/login', {
			method: 'POST',
			headers: { 'Content-Type': 'application/json' },
			body: JSON.stringify({ userName: loginUsername.value, password: loginPassword.value })
		});

		const data = await res.text();

		if (res.ok) {
			localStorage.setItem('AuthToken', data);
			const loginModal = bootstrap.Modal.getInstance(document.getElementById('loginModal'));
			loginModal.hide();
			setTimeout(() => {
				showAlert('Sikeres bejelentkezés.', 'success');
				updateUIForLoggedInUser();
			}, 450);
			return;
		}

		setTimeout(() => {
			showAlert('Sikertelen bejelentkezés.', 'danger');
		}, 450);
	}

	onMount(() => {
		new Swiper('.review-swiper', {
			slidesPerView: 1,
			spaceBetween: 30,
			loop: true,
			autoplay: {
				delay: 5000,
				disableOnInteraction: false
			},
			pagination: {
				el: '.swiper-pagination',
				clickable: true
			}
		});
	});
</script>

<div style="padding-top: 56px;">
	<div class="container mt-4">
		<div id="customAlert" class="alert d-none" role="alert"></div>
	</div>

	<header>
		<div
			id="heroCarousel"
			class="carousel slide"
			data-bs-ride="carousel"
			data-bs-interval="2500"
			data-bs-wrap="true"
			data-bs-pause="false"
		>
			<div class="carousel-inner">
				<div class="carousel-item active">
					<img src={img6} class="d-block w-100" alt="Luxus Szálloda" />
					<div class="carousel-caption d-none d-md-block">
						<h1>Üdvözöljük a DreamStayben</h1>
						<p>Tapasztalja meg a luxust, mint még soha</p>
					</div>
				</div>
				<div class="carousel-item">
					<img src={img7} class="d-block w-100" alt="Szállodai Szoba" />
					<div class="carousel-caption d-none d-md-block">
						<h1>Kényelmes Szobák</h1>
						<p>Pihenjen tágas és elegáns szobáinkban</p>
					</div>
				</div>
				<div class="carousel-item">
					<img src={img8} class="d-block w-100" alt="Szállodai Medence" />
					<div class="carousel-caption d-none d-md-block">
						<h1>Lenyűgöző Szolgáltatások</h1>
						<p>Élvezze világszínvonalú létesítményeinket</p>
					</div>
				</div>
			</div>
		</div>
	</header>

	<main class="container my-5">
		<section id="search" class="mb-5">
			<h2 class="mb-4">Találja meg a tökéletes szobát önnek.</h2>
			<form id="searchForm" onsubmit={handleFilter}>
				<div class="row g-3">
					<!-- <div class="col-md-3">
						<label for="checkIn" class="form-label">Érkezés</label>
						<input type="date" class="form-control" id="checkIn" required />
					</div>
					<div class="col-md-3">
						<label for="checkOut" class="form-label">Távozás</label>
						<input type="date" class="form-control" id="checkOut" required />
					</div> -->
					<div class="col-md-2">
						<label for="adults" class="form-label">Felnőttek</label>
						<select class="form-select" id="adults" required>
							<option value="1">1</option>
							<option value="2">2</option>
							<option value="3">3</option>
							<option value="4">4</option>
							<option value="5">5</option>
						</select>
					</div>
					<div class="col-md-2">
						<label for="children" class="form-label">Gyerekek</label>
						<select class="form-select" id="children" required>
							<option value="0">0</option>
							<option value="1">1</option>
							<option value="2">2</option>
							<option value="3">3</option>
							<option value="4">4</option>
							<option value="5">5</option>
						</select>
					</div>
					<div class="col-md-2 d-flex align-items-end">
						<button type="submit" class="btn btn-dark w-100">Keresés</button>
					</div>
					<div class="col-md-2 d-flex align-items-end">
						<button class ="btn btn-dark w-100" onclick= {clearFilters}>Szűrés törlése</button>
					</div>
				</div>
			</form>
		</section>

		<section id="rooms" class="mb-5">
			<h2 class="mb-4">Szobáink</h2>
			<div class="row" id="roomList">
				<!-- Room cards will be dynamically inserted here -->

				{#each rooms as room}
					<div class="col-md-6 col-lg-4 mb-4">
						<div class="card room-card">
							<img src={room.img_Src} class="card-img-top room-image" alt="${room.name}" />
							<div class="card-body">
								<h5 class="card-title">{room.name}</h5>
								<p class="card-text">{room.description}</p>
								<p class="card-text"><strong>Ár:</strong> {room.pricePerNight} HUF / éjszaka</p>
								<p class="card-text">
									<strong>Max. létszám:</strong>
									{room.maxAdults + room.maxChildren}
								</p>
								<button
									class="btn btn-dark book-btn"
									data-bs-toggle="modal"
									data-bs-target="#bookingModal"
									onclick={openBooking(room.id)}>Foglalás</button
								>
							</div>
						</div>
					</div>
				{/each}
			</div>
		</section>

		<section id="about" class="mb-5">
			<h2 class="mb-4">A DreamStayről</h2>
			<p>
				A DreamStay egy kiváló szobafoglalási platform, amely széles választékban kínál luxus
				szálláshelyeket minden utazó igényeinek megfelelően. Felhasználóbarát felületünkkel és
				kiterjedt, kiváló minőségű szobakínálatunkkal arra törekszünk, hogy a foglalási élménye a
				lehető legzökkenőmentesebb és legélvezetesebb legyen.
			</p>
			<p>
				Kiválóságra való elkötelezettségünk túlmutat a kényelmes szobák biztosításán.
				Világszínvonalú szolgáltatásokat, kivételes ügyfélszolgálatot és a legjobb elérhető árak
				garanciáját kínáljuk. Akár üzleti úton van, akár pihenni szeretne, a DreamStay a tökéletes
				partner egy felejthetetlen tartózkodáshoz.
			</p>
		</section>

		<section id="reviews" class="mb-5">
			<h2 class="mb-4 text-center">Vendégvélemények</h2>
			<div class="swiper review-swiper">
				<div class="swiper-wrapper">
					{#each reviews as review}
						<div class="swiper-slide">
							<div class="review-card">
								<h5>{review.title}</h5>
								<p>{review.text}</p>
								<div class="review-author">
									<img src={review.image} alt={review.author} />
									<div class="review-author-info">
										<span class="review-author-name">{review.author}</span>
										<span class="review-author-location">{review.location}</span>
									</div>
								</div>
							</div>
						</div>
					{/each}
				</div>
				<div class="swiper-pagination"></div>
			</div>
		</section>

		<section id="contact" class="mb-5">
			<h2 class="mb-4">Kapcsolat</h2>
			<div class="row">
				<div class="col-md-6">
					<form>
						<div class="mb-3">
							<label for="name" class="form-label">Név</label>
							<input type="text" class="form-control" id="name" required />
						</div>
						<div class="mb-3">
							<label for="email" class="form-label">E-mail</label>
							<input type="email" class="form-control" id="email" required />
						</div>
						<div class="mb-3">
							<label for="message" class="form-label">Üzenet</label>
							<textarea class="form-control" id="message" rows="3" required></textarea>
						</div>
						<button type="submit" class="btn btn-dark">Üzenet Küldése</button>
					</form>
				</div>
				<div class="col-md-6">
					<h4>Címünk</h4>
					<p>Luxus utca 123, Budapest, 1234</p>
					<h4>Telefon</h4>
					<p>+36 1 234 5678</p>
					<h4>E-mail</h4>
					<p>info@dreamstay.hu</p>
				</div>
			</div>
		</section>
	</main>

	<footer class="bg-dark text-light py-4">
		<div class="container">
			<div class="row">
				<div class="col-md-4">
					<h5>DreamStay</h5>
					<p>Tapasztalja meg a luxust, mint még soha.</p>
				</div>
				<div class="col-md-4">
					<h5>Gyors Linkek</h5>
					<ul class="list-unstyled">
						<li><a href="/" class="text-light">Főoldal</a></li>
						<li><a href="#rooms" class="text-light">Szobák</a></li>
						<li><a href="#about" class="text-light">Rólunk</a></li>
						<li><a href="#contact" class="text-light">Kapcsolat</a></li>
					</ul>
				</div>
				<div class="col-md-4">
					<h5>Kövessen Minket</h5>
					<div class="social-icons">
						<a
							href="https://www.facebook.com/profile.php?id=61568641694075"
							target="_blank"
							class="text-light me-2"><i class="bi bi-facebook"></i></a
						>
						<a
							href="https://www.instagram.com/dreamstayhotel_/"
							target="_blank"
							class="text-light me-2"><i class="bi bi-instagram"></i></a
						>
						<a href="https://www.tiktok.com/@dreamstay.hotel" target="_blank" class="text-light"
							><i class="bi bi-tiktok"></i></a
						>
					</div>
				</div>
			</div>
			<hr class="mt-4 mb-3" />
			<p class="text-center mb-0">&copy; 2025 DreamStay. Minden jog fenntartva.</p>
		</div>
	</footer>

	<!-- Login/Register Modal -->
	<div
		class="modal fade"
		id="loginModal"
		tabindex="-1"
		aria-labelledby="loginModalLabel"
		aria-hidden="true"
	>
		<div class="modal-dialog">
			<div class="modal-content">
				<div class="modal-header">
					<h5 class="modal-title" id="loginModalLabel">Bejelentkezés / Regisztráció</h5>
					<button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Bezárás"
					></button>
				</div>
				<div class="modal-body">
					<ul class="nav nav-tabs" id="loginTabs" role="tablist">
						<li class="nav-item" role="presentation">
							<button
								class="nav-link active"
								id="login-tab"
								data-bs-toggle="tab"
								data-bs-target="#login"
								type="button"
								role="tab"
								aria-controls="login"
								aria-selected="true">Bejelentkezés</button
							>
						</li>
						<li class="nav-item" role="presentation">
							<button
								class="nav-link"
								id="register-tab"
								data-bs-toggle="tab"
								data-bs-target="#register"
								type="button"
								role="tab"
								aria-controls="register"
								aria-selected="false">Regisztráció</button
							>
						</li>
					</ul>
					<div class="tab-content" id="loginTabsContent">
						<div
							class="tab-pane fade show active"
							id="login"
							role="tabpanel"
							aria-labelledby="login-tab"
						>
							<form id="loginForm" class="mt-3" onsubmit={handleLogin}>
								<div class="mb-3">
									<label for="loginUsername" class="form-label">Felhasználónév</label>
									<input type="text" class="form-control" id="loginUsername" required />
								</div>
								<div class="mb-3">
									<label for="loginPassword" class="form-label">Jelszó</label>
									<input type="password" class="form-control" id="loginPassword" required />
								</div>
								<button type="submit" class="btn btn-dark">Bejelentkezés</button>
							</form>
						</div>
						<div class="tab-pane fade" id="register" role="tabpanel" aria-labelledby="register-tab">
							<form id="registerForm" class="mt-3" onsubmit={handleRegister}>
								<div class="mb-3">
									<label for="registerName" class="form-label">Név</label>
									<input type="text" class="form-control" id="registerName" required />
								</div>
								<div class="mb-3">
									<label for="registerEmail" class="form-label">E-mail cím</label>
									<input type="email" class="form-control" id="registerEmail" required />
								</div>
								<div class="mb-3">
									<label for="registerPassword" class="form-label">Jelszó</label>
									<input type="password" class="form-control" id="registerPassword" required />
								</div>
								<div class="mb-3">
									<label for="registerConfirmPassword" class="form-label">Jelszó megerősítése</label
									>
									<input
										type="password"
										class="form-control"
										id="registerConfirmPassword"
										required
									/>
								</div>
								<button type="submit" class="btn btn-dark">Regisztráció</button>
							</form>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>

	<!-- Profile Modal -->
	<div
		class="modal fade"
		id="profileModal"
		tabindex="-1"
		aria-labelledby="profileModalLabel"
		aria-hidden="true"
	>
		<div class="modal-dialog">
			<div class="modal-content">
				<div class="modal-header">
					<h5 class="modal-title" id="profileModalLabel">Profilom</h5>
					<button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Bezárás"
					></button>
				</div>
				<div class="modal-body">
					<h6>Aktív foglalásaim:</h6>
					<div id="userBookings">
						<!-- User bookings will be dynamically inserted here -->
						{#if $bookings.length === 0}
							<p>Jelenleg nincs foglalva szoba.</p>
						{:else}
							{#each $bookings as booking}
								<div class="card mb-3">
									<div class="card-body">
										<h6 class="card-title">{booking.room.name}</h6>
										<p class="card-text">
											Érkezés: {new Date(booking.checkInDate).toLocaleString()}
										</p>
										<p class="card-text">
											Távozás: {new Date(booking.checkOutDate).toLocaleString()}
										</p>

										{#if booking.status === 'Jóváhagyásra vár'}
											<button
												class="btn btn-danger btn-sm cancel-booking"
												data-booking-index={booking.id}
												onclick={handleBookingRemove}>Foglalás törlése</button
											>
										{/if}
										<div
											class="badge {booking.status === 'Jóváhagyásra vár'
												? 'bg-warning'
												: 'bg-success'}"
										>
											{booking.status}
										</div>
									</div>
								</div>
							{/each}
						{/if}
					</div>
				</div>
			</div>
		</div>
	</div>
	<!-- Foglalás Modal -->
<!-- Modal -->
<div class="modal fade" id="bookingModal" tabindex="-1" aria-labelledby="bookingModalLabel" aria-hidden="true">
	<div class="modal-dialog modal-lg">
	  <div class="modal-content rounded-4 border-0">
		<div class="modal-header bg-dark text-white">
		  <h5 class="modal-title" id="bookingModalLabel">{currentRoom?.name}</h5>
		  <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal" aria-label="Bezárás"></button>
		</div>
		<div class="modal-body p-4">
		  <h6 class="fw-bold">Leírás</h6>
		  <p class="text-muted">{currentRoom?.description}</p>
  
		  <div class="row g-3">
			<div class="col-md-6">
			  <label for="checkIn" class="form-label fw-semibold">Érkezés</label>
			  <input type="date" class="form-control" id="checkIn" required />
			</div>
			<div class="col-md-6">
			  <label for="checkOut" class="form-label fw-semibold">Távozás</label>
			  <input type="date" class="form-control" id="checkOut" required />
			</div>
		  </div>
		</div>
		<div class="modal-footer d-flex justify-content-between">
		  <button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Mégse</button>
		  <button onclick="{handleBooking}" class="btn btn-outline-secondary">Foglalás</button>
		</div>
	  </div>
	</div>
  </div>
  
</div>
