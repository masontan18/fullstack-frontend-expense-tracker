<script>
	import '../../app.css';
	import Footer from '../../lib/Footer.svelte';
	import { PUBLIC_BACKEND_BASE_URL } from '$env/static/public';
	import { onMount } from 'svelte';
	import { themeChange } from 'theme-change';
	import { isLogin } from '../../utils/stores.js';
	import { isLoggedIn } from '../../utils/auth.js';
	import { goto } from '$app/navigation';
	import { authenticateUser } from '../../utils/auth.js';

	let formErrors = {};
	let clicked = false;

	onMount(async () => {
		themeChange(false);
		// 👆 false parameter is required for svelte

		isLogin.set(await isLoggedIn());
		// isLogin.set(true);
	});

	function postSignUp() {
		isLogin.set(true);
		goto('/homepage');
	}

	let isLoading = false;
	$: buttonClass = `btn btn-md login-btn ${isLoading ? 'btn--loading' : ''}`;

	async function createUser(evt) {
		evt.preventDefault();
		clicked = true;

		if (evt.target['password'].value != evt.target['password-confirmation'].value) {
			formErrors['password'] = { message: 'Password confirmation does not match' };
			clicked = false;
			return;
		}

		const userData = {
			email: evt.target['email'].value,
			password: evt.target['password'].value
		};

		const resp = await fetch(PUBLIC_BACKEND_BASE_URL + '/users', {
			// remember this is for POSTing DATA to Backend
			method: 'POST',
			mode: 'cors',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify(userData)
		});
		console.log(resp.status);
		if (resp.status == 201) {
			const res = await authenticateUser(userData.email, userData.password);

			if (res.success) {
				// signUpAlert()
				postSignUp();
			} else {
				throw 'Sign up succeeded but authentication failed';
			}
		} else {
			clicked = false;
			// const res = await resp.json();
			// formErrors = res.data;
			throw 'Sign up failed';
		}
	}

	let enteredEmail = '';
	let emailIsValid = false;
	let emailIsTouched = false;
	$: emailHasError = !emailIsValid && emailIsTouched;
	const emailChangeHandler = (event) => {
		enteredEmail = event.target.value;
		if (enteredEmail.includes('@') && enteredEmail.includes('.com')) {
			emailIsValid = true;
		} else {
			emailIsValid = false;
		}
	};
	const emailBlurHandler = (event) => {
		emailIsTouched = true;
		if (!(enteredEmail.includes('@') && enteredEmail.includes('.com'))) {
			emailIsValid = false;
		}
	};

	let enteredPassword = '';
	let passwordIsValid = false;
	let passwordIsTouched = false;
	$: passwordHasError = !passwordIsValid && passwordIsTouched;
	const passwordChangeHandler = (event) => {
		enteredPassword = event.target.value;
		if (enteredPassword.length >= 8) {
			passwordIsValid = true;
		} else {
			passwordIsValid = false;
		}
	};
	const passwordBlurHandler = (event) => {
		passwordIsTouched = true;
		if (enteredPassword.length < 8) {
			passwordIsValid = false;
		}
	};

	let enteredConfirmPassword = '';
	let confirmPasswordIsValid = false;
	let confirmPasswordIsTouched = false;
	$: confirmPasswordHasError = !confirmPasswordIsValid && confirmPasswordIsTouched;
	const confirmPasswordChangeHandler = (event) => {
		enteredConfirmPassword = event.target.value;
		if (enteredConfirmPassword.length >= 8) {
			confirmPasswordIsValid = true;
		} else {
			confirmPasswordIsValid = false;
		}
	};
	const confirmPasswordBlurHandler = (event) => {
		confirmPasswordIsTouched = true;
		if (enteredConfirmPassword.length < 8) {
			confirmPasswordIsValid = false;
		}
	};

	let formIsValid = false;
	$: if (emailIsValid && passwordIsValid && confirmPasswordIsValid) {
		formIsValid = true;
	} else {
		formIsValid = false;
	}
</script>

<div class="bg-black min-h-screen pt-5">
	<h1 class="text-center text-2xl">Create an Account to start tracking</h1>
	<div class="text-center">
		<a class="link-hover italic text-xs text-blue-600" href="/login"
			>Already have an account? Click here to login instead.</a
		>
	</div>
	<div class="exp flex justify-center items-center mt-8">
		<form on:submit={createUser} class="w-1/3">
			<div class="form-control w-full">
				<label class="label" for="email">
					<span class="label-text">Email</span>
				</label>
				<input
					type="email"
					name="email"
					placeholder="mickymouse@example.com"
					class="input input-bordered w-full"
					on:input={emailChangeHandler}
					on:blur={emailBlurHandler}
				/>
				{#if emailHasError}
					<label class="label" for="email">
						<span class="label-text-alt text-red-500"
							>Please enter a valid email with "@" and ".com".</span
						>
					</label>
				{/if}
			</div>

			<div class="form-control w-full">
				<label class="label" for="password">
					<span class="label-text">Password</span>
				</label>
				<input
					type="password"
					name="password"
					placeholder=""
					class="input input-bordered w-full"
					on:input={passwordChangeHandler}
					on:blur={passwordBlurHandler}
				/>
				{#if passwordHasError}
					<label class="label" for="password">
						<span class="label-text-alt text-red-500">Password must be at least 8 characters.</span>
					</label>
				{/if}
			</div>

			<div class="form-control w-full">
				<label class="label" for="password-confirmation">
					<span class="label-text">Confirm Password</span>
				</label>
				<input
					type="password"
					name="password-confirmation"
					placeholder=""
					class="input input-bordered w-full"
					on:input={confirmPasswordChangeHandler}
					on:blur={confirmPasswordBlurHandler}
				/>
				{#if confirmPasswordHasError}
					<label class="label" for="passwod-confirmation">
						<span class="label-text-alt text-red-500">Password must be at least 8 characters.</span>
					</label>
				{/if}
			</div>

			<div class="form-control w-full mt-4">
				<button class={buttonClass} disabled={!formIsValid}
					><span class="btn-text">Create an Account</span></button
				>
			</div>
		</form>
	</div>
</div>

<Footer />

<style>
	h1 {
		margin-top: 100px;
	}

	/* loading spin after clicked button */
	/* 1 class for <span> inside <button> which is .btn-text and 2 class for <button> which are .login-btn and .btn--loading (must use --loading) */
	/* Use .login-btn to set relative position only, to control spin position later */
	/* Use .btn-text to set the content (such as LOG IN ) hidden only when .btn--loading class is added */
	/* Main thing is to construct .btn--loading (with ::after pseudo ) and keyframes together. By default this class is not inside <button>, add in only when clicked button. When add inside, hide the text in .btn-text. Rmb to remove it when loading is done. 
	*/
	.login-btn {
		position: relative;
	}
	.btn--loading .btn-text {
		visibility: hidden;
		opacity: 0;
	}
	.btn--loading::after {
		content: '';
		position: absolute;
		width: 25px;
		height: 25px;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		margin: auto;
		border: 4px solid transparent;
		border-top-color: rgb(87, 92, 90);
		border-radius: 50%;
		animation: spin 1s ease infinite;
	}
	@keyframes spin {
		from {
			transform: rotate(0turn);
		}
		to {
			transform: rotate(1turn);
		}
	}

	.exp {
		padding: 1rem;
		background-color: rgb(31, 31, 31);
		margin: 2rem auto;
		width: 50rem;
		max-width: 95%;
	}
</style>
